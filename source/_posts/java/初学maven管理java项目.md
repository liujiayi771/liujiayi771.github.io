---
title: 初学maven管理java项目
date: 2016-03-24 20:04:35
tags: [maven,Java]
categories: [技术]
---

* 今天很粗浅地学会了如何使用maven来管理我的java项目，也就初步了解了一些项目的依赖如何写入`pom.xml`文件中，以及几个maven的插件。项目的依赖包可以到maven的仓库中进行查询，并将依赖的xml代码复制到自己项目的`pom.xml`文件中，maven仓库的地址为：
http://mvnrepository.com/
* 使用maven首先需要在系统上安装maven，对于linux系统而言，直接到maven官网下载tar.gz文件，解压到一个目录，例如`/usr/local/maven`，在环境变量中加入`M2_HOME`的值为maven的解压目录`/usr/local/maven`。在`PATH`变量中加入`${M2_HOME}/bin`，修改变量完毕之后不要忘记`source /etc/profile`。
* 在我的项目当中

```java
import com.sun.image.codec.jpeg.JPEGCodec;
import com.sun.image.codec.jpeg.JPEGImageEncoder;
```
这两个包是sun公司的，并不对外公开，maven的仓库中当然也没有这两个包，导致maven项目不能成功编译。这两个包实际上是jdk和jre的lib中自带的，路径分别为`${JAVA_HOME}/lib/rt.jar`和`${JAVA_HOME}/jre/lib/jce.jar`，也就是说我们有这两个库，现在需要加入到maven项目当中。在Google查找资料发现一种可行的办法，就是在`pom.xml`中加入一个maven的插件(plugin)，具体的内容如下所示：

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <encoding>UTF-8</encoding>
        <compilerArguments>
            <verbose />
            <bootclasspath>${java.home}/lib/rt.jar:${java.home}/lib/jce.jar</bootclasspath>
        </compilerArguments>
    </configuration>
</plugin>
```
<!--more-->
这样做就能成功编译了。
* maven编译可以在idea中很简单地实现，也可以在外部运行`mvn compile`指令，这样之后会产生编译后的class文件，但这些class文件一般不能运行，因为还依赖很多的jar包，这些jar包并不会在java执行的时候自动引入。目前我知道的方法只有说利用maven将整个项目打包成一个jar包，然后采用`java -jar <project_name>.jar`的方式来运行。maven本身并不支持打包，需要在`pom.xml`中加入插件的代码，如下所示：

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <appendAssemblyId>false</appendAssemblyId>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <mainClass>WormImage</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>assembly</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
这样就能通过idea中的maven项目管理中的package按钮来实现打包操作，也可以在外部终端中输入`mvn package`指令来实现打包操作。需要说明的是`<plugin>`标签需要插入在<build><plugins></plugins></build>标签之内，否则`pom.xml`文件会报错。当项目需要被push的时候，可以输入`mvn clean`指令来清除编译产生的文件，方便上传。
* 使用maven来管理项目的一个好处是不需要将依赖的jar包也一起上传到github中，只是源码部分被上传了。依赖的包在编译项目的时候才被下载下来。当然，maven管理java项目的好处还有很多，今后也会制定计划对maven进一步学习。
