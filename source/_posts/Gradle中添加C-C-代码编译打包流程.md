---
title: Gradle中添加C/C++代码编译打包流程
date: 2018-10-29 15:10:41
tags:
---
写过JNI代码的同学应该都有遇到过这样的问题，我们可以很方便的使用Maven、Gradle等工具将我们写好的Java代码打包成一个jar包进行发布，但是当我们的代码当中包含有C/C++的代码时，我们可能需要额外再去运行一次make指令去编译C/C++的代码，并将编译出的库和jar包一起发布（在大多数情况下so文件和jar包是分开的）。实际上，我们可以使用Gradle在打包的同时自动编译我们的C/C++代码，同时将so文件打包到jar包之中，这样在运行jar包时不需要指定`java.library.path`也能正确读取到so文件，并且发布时只有一个文件，更为方便。

#### Exec Task
我们知道，Gradle的执行过程可以分为很多的Task进行，要实现Gradle自动编译C/C++代码我们需要首先了解一下Exec Task。Exec Task是用来执行一个命令行语句的，例如：

```gradle
task stopTomcat(type: Exec) {
	workingDir '../tomcat/bin'
	
	//on windows:
	commandLine 'cmd', '/c', 'stop.bat'
	
	//on linux:
	commandLine './stop.sh'
	
	//store the output instead of printing to the console:
	standardOutput = new ByteArrayOutputStream()
	
	//extension method stopTomcat.output() can be used to obtain the output:
	ext.output = {
		return standardOutput.toString()
	}
}
```
具体有关这个Task的详细说明可以看这个：https://docs.gradle.org/current/dsl/org.gradle.api.tasks.Exec.html

#### 使用Exec Task来编译C/C++代码
当然，Exec Task用来编译C/C++代码使用的是make指令，如果使用的是cmake，也可以先执行cmake再执行make，无非是多加几个Exec Task。这里具体的写法如下：

```gradle
String cpath = "native"
String libname = "libpairhmm"
task buildLib(type: Exec) {
	workingDir "$cpath"
	outputs.files "$cpath/libpairhmm*"
	outputs.dir "$cpath/pairhmm"
	commandLine "make all"
	String home = System.properties."java.home"
	//strip the trailing jre
	String corrected = home.endsWith("jre") ? home.substring(0, home.length() - 4) : home
	environment JAVA_HOME : corrected
	doFirst { println "using $home -> $corrected as JAVA_HOME" }
}

clean {
	delete "$cpath/pairhmm"
	delete "$cpath/$libname*"
	delete fileTree("$cpath") { include "$libname*", "*.o" }
}

processResources {
	dependsOn buildLib
	from cpath
	include "$libname*"
}
```

这里buildLib这个task就是一个Exec Task，其中定义的流程就是执行make all指令来编译C/C++代码，clean的作用就是清除编译产生的lib文件和o文件（在调用gradle clean的时候调用清除工作），processResources即处理资源文件，其作用就是将编译产生的库文件拷贝到jar包当中。
