---
title: 远程调试Spark程序
date: 2018-10-26 18:40:25
tags: Spark

---

我们在写Spark程序的时候免不了要对我们的代码进行debug，在代码当中打上断点来查看程序执行过程中各个变量的变化情况。我一般使用Intellij IDEA来写Spark程序，可以直接在其中以local的方式运行Spark程序，也可以在其中打上断点进行调试，但这样做有一些问题：

1. 我们只能对Spark driver端的程序进行打断点debug；
2. Spark很多代码都是惰性执行的，很多代码都需要有action才能触发，在这之前打断点没有意义，真正的Task执行逻辑位于Executor当中；
3. 对于某些运算量比较大或者内存消耗比较多的程序来说，本地电脑不能运行；
4. 这样只能观察代码在local模式下运行的是否正确，无法对在集群中运行的代码进行调试。

之前所说的几点问题我在之前一般采用比较低效率的方式来进行debug，即在代码当中加入一些log信息，利用log信息来调试代码，这样当Spark程序在集群中运行时，可以在web UI的Executor的stdout和stderr中查看我们留下的log信息。这样做可以解决一些问题，但是十分低效，debug的信息需要添加改动时都需要重新编译程序，并且打印log信息的方式并不能很完整地观察到所有变量的变化情况。但这样做确实解决了一些问题，比如我们可以对Executor真正执行Task逻辑的代码进行调试，也无需考虑惰性执行的过程，Spark的所有RDD的transform的行为都会反映到每个Executor执行的stdout和stderr信息当中；这些代码都可以在服务器集群当中运行进行调试，不用担心本地电脑性能不够的问题，本地电脑只需要打开浏览器查看Spark的web UI即可，或者使用终端来查看一些信息；可以在集群当中调试代码，不需要局限于local模式下。

但我始终认为这样的debug方式是低效的，并且不是一个正常的程序员应该有的debug方式，之前有想过肯定有具体的方法来解决这样的调试问题，Intellij IDEA当中功能非常多，肯定有这样的功能来解决这个问题。近期又要写一些Spark程序，并且输入数据非常大，计算量也很大，我在本地电脑上根本没办法debug，于是想到了去查找一些资料来解决远程调试Spark程序的问题。其实Intellij IDEA或者Eclipse这样的IED都会有remote debug这样的功能，以Intellij IDEA为例，在Run-Edit Configurations菜单中我们可以添加一个Remote的Configurations，这就是Intellij IDEA为我们提供的远程调试的功能。这里对Spark程序的调试主要分两种——Driver程序调试和Executor程序调试。

#### Driver程序调试
Driver程序在远程进行调试时，需要在spark-submit的参数中增加一个配置：

```bash
--conf spark.driver.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
```
<!--more-->
或

```bash
--driver-java-options -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
```

我们队这个参数进行一些说明，首先spark.driver.exetraJavaOptions这个参数允许我们在Spark的driver程序在运行时传入一些额外的Java参数，`agentlib:jdwp`是Java Debug Wire Protocol选项，是一个远程调试的协议，后面紧跟的是以逗号进行分隔的子选项：
* `transport`定义了远程Debugger及Debuggee之间数据传输协议，可以选择socket和shared memory的方式，一般情况下都是选择socket的方式，即选项的值选择dt_socket
* `server`运行Java程序的进程是否作为服务端与客户端(Debugger)进行通信，通常情况下远程调试都需要有一个服务端和一个客户端，若运行Java程序端为服务端，则调试端为客户端，在调试driver程序时，选择Java程序为服务端，监听远端Debugger的连接，因此值为y(yes)
* `suspend`是否暂停执行直至有客户端(Debugger)成功连接到服务端，调试driver程序时将这个值设置为y(yes)使得driver程序会在刚启动时就暂停住，指到有远端Debugger客户端连接上来才继续执行，确保远端Debugger能在程序开始执行时已经连接完毕
* `address`监听的端口，即服务端socket监听的端口，可以设置为任意可用的没有被占用的端口号，只要确保客户端能够通过这个端口与服务端进行连接即可

进行了如上设置之后，在我们允许spark-submit指令之后，我们一般能够看到Spark程序暂停住了，并且会显示如下这样一句话：

```bash
Listening for transport dt_socket at address: 5005
```
这时候，我们的Spark程序已经做好准备在等待远程Debugger客户端连接了(一般称之为attach)，接下来就在Intellij IDEA中打开之前所说的那个菜单，点击“+”添加一个新的run/debug configuration，这里选择的是Remote类型，Debugger mode选择Attach to remote JVM，接下来填入Host和port并选择moudle classpath，之后使用这个配置点击debug按钮便可以进行远程调试了。

#### Executor程序调试
一般来说，我们进行Spark程序调试最主要的就是要对Task当中的执行逻辑进行调试，因为Executor中执行的才是真正的并行任务，也是程序最主要的部分。对Executor进行调试时我们一般将允许Java进程的服务器作为客户端，将Debugger所在的机器作为服务端，也就是在调试机器上启动一个socket服务器对指定端口进行监听，而Executor上运行的程序作为客户端与之进行连接。为什么不采用之前的那种模式呢？因为我们需要保证Executor上的程序一开始执行就与Debugger进行了连接，若采用之前的模式，就需要将Executor进行暂停，而Executor暂停会使得driver程序误认为其出现了故障，会不断的重启Executor，无法正常进行调试。一般情况下要这样进行调试需要将Executor设置为只有一个，因为多个Executor同时连接Debugger的服务端会出现问题，若一定要进行集群调试，则在指定Worker启动的时候加入参数，而不是在spark-submit的时候加入参数，首先说下参数：

```bash
--conf spark.executor.extraJavaOptions=-agentlib:jdwp=transport=dt_socket,server=n,address=10.0.0.171:5005,suspend=n
```
参数的具体意思不用多加说明，主要是address这里要写入具体的Debugger的地址以及端口号，如果要指定一个Worker上启动一个Executor来进行调试，可以在使用手动的方式启动该Worker

```bash
spark-class -agentlib:jdwp=transport=dt_socket,server=n,address=10.0.0.171:5005,suspend=n org.apache.spark.deploy.worker.Worker spark://10.0.0.1:7077
```
只要给该Worker分配一定的资源，确保在这个Worker上启动一个Executor即可。Intellij IDEA的设置上和之前的区别就是将Debugger mode改为Listen to remote JVM，然后首先需要启动Debugger的服务端，即先在Intellij IDEA当中点击debug开启服务，再运行Spark程序，这样远端的Spark Executor上运行的程序会自动进行连接，方便进行debug。这里需要注意的是，debug过程中不要让一行代码停留太长时间，否则driver会认为该Executor出现了故障，会不断进行重启Executor。


参考资料：
* https://stackoverflow.com/questions/30403685/how-can-i-debug-spark-application-locally
* https://stackoverflow.com/questions/29090745/debugging-spark-applications