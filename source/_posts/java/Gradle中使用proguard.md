---
title: Gradle中使用proguard
date: 2018-05-23 21:04:45
tags: [Javai,技术]
---

最近做了一个Java项目，老板让我们将核心部分的代码进行混淆，防止jar包被反编译出来。Java项目是基于Gradle进行构建的，使用了shadowJar这个插件将源码生成的jar包和所有的依赖的jar包打包到一起，称为一个fat-jar。我之前单独使用过proguard的gui，也使用过maven的proguard plugin以及sbt的plugin，都踩了很多坑最终混淆成功了，以为这次应该很轻松能完成任务，但事实上我遇到了很多之前没有遇到过的问题，现在将我解决这个问题的每个阶段记录下来。
#### 阶段一
下载了最新的proguard6.0.3，执行proguardgui.sh，图形界面出来之后，写好一个配置文件并load进去，配置文件中将包含依赖的fat-jar作为输入，libraryjars只添加了`jre/lib/rt.jar`，因为其他库文件都包含在了fat-jar包当中，这样混淆有一个问题就是会去混淆依赖的库，虽然可以通过`keep class`来保持依赖的库不被混淆，但是proguard还是会去遍历所有的依赖库中的内容，导致混淆的时间非常长。这对于我来说是不能接受的，我现在都还不知道我写的混淆配置文件能不能让混淆后的jar包正常运行，如果测试一次要花这么长时间，肯定是不能按时完成任务的，而且整个调试的过程会非常痛苦。我看了一下jar包有200M左右，但实际上我们源码对应的jar包只有5M左右，其他的内容都是依赖的库，实际上我是不需要去混淆这些依赖，proguard花费时间去遍历这些依赖是没有意义的。
#### 阶段二
我先不用shadowJar进行打包，只使用jar任务编译出一个不包含依赖的jar包，只对这个不包含依赖的jar包进行混淆，把其依赖的库通过proguard配置文件中的`-libraryjars`参数添加进去（不添加进去会出现找不到依赖的库的问题）。这样proguard就只会混淆我们所写的代码，不会涉及到依赖的库，代码很快就混淆完了。混淆后的库文件中包含有`MAINIFEST.MF`文件，`Class-Path`中记录了所依赖的库文件的路径，使得独立的jar包也能正常运行。我写的混淆配置文件混淆的力度并不是很大，我以为程序能够正常运行，但是却并没有如我所愿。
#### 阶段三
独立混淆的jar包在混淆环节并没有出错，但是执行的时候却遇到了一个很奇怪的问题，我追踪代码发现在某一个地方使用了`ClassLoader.getResource(packageName)`方法去获取在packageName包下的所有资源，这个方法在jar包没有混淆之前是能正确找到packageName下的所有资源，但是混淆之后这个方法就什么都获取不到了。为了探究原因，我关闭了proguard的所有功能，包括optimize、obfuscate、shrink，相当于不对输入的jar包做任何处理，最后输出的jar包还是会有这个问题。同时，我把混淆前的jar包和不开启proguard任何功能输出的jar包使用JAPICC进行比较，发现里面的内容是完全一致的。查找资料发现proguard会对jar包进行优化，以期减少其大小。默认情况下，proguard会删除jar中的目录元素，导致ClassLoader().getResource（）方法找不到对应的资源，只需要在使用时加上`-keepdirectories`选项即可。附上官方文档的说明：
> **-keepdirectories** [_directory_filter_]
Specifies the directories to be kept in the output jars (or aars, wars, ears, zips, apks, or directories). By default, directory entries are removed. This reduces the jar size, but it may break your program if the code tries to find them with constructs like "com.example.MyClass.class.getResource("")". You'll then want to keep the directory corresponding to the package, "-keepdirectories com.example". If the option is specified without a filter, all directories are kept. With a filter, only matching directories are kept. For instance, "-keepdirectories mydirectory" matches the specified directory, "-keepdirectories mydirectory/*" matches its immediate subdirectories, and "-keepdirectories mydirectory/**" matches all of its subdirectories.

#### 阶段四
最后的要求还是需要将源码和依赖的库打包到一起，需要在shadowJar打包之前先将源码产生的jar包进行混淆，shadowJar任务的输入改成这个混淆后的jar包即可。proguard实际上也能作为gradle的一个插件进行使用，可以在`build.gradle`当中加入一个proguard的task进行混淆，proguard官网提供了一种使用方法：
```groovy
buildscript {
  repositories {
    flatDir dirs: '/usr/local/java/proguard/lib'
  }
  dependencies {
    classpath ':proguard:'
  }
}
```
定义task的方式如下：
```groovy
task myProguardTask(type: proguard.gradle.ProGuardTask) {
  .....
}
```
但这样需要自己手动下载proguard，并存放在编译gradle的服务器上，十分不方便。还有一种方式比较方便，每次会自动下载需要的jar包：
```groovy
buildscript {
  repositories {
    mavenCentral()
    jcenter() // for shadow plugin
  }
  dependencies {
    classpath 'net.sf.proguard:proguard-gradle:6.0.3'
  }
}
```
我所定义的proguard的混淆任务如下：
```groovy
task obfuscate(type: proguard.gradle.ProGuardTask) {
  injars jar
  outjars "$buildDir/libs/${project.name}-pg.jar"
  libraryjars "${System.getProperty('java.home')}/lib/rt.jar"
  libraryjars files(configurations.compile.collect())

  useuniqueclassmembernames

  dontshrink
  dontoptimize
  dontnote
  dontwarn

  //keepnames 'class ** { *; }'
  configuration 'proguard.pro'
}
```
这里`injars`直接写jar即可，会得到jar任务的输出（即源码编译产生的jar），`outjars`输出到`build/libs`路径下，`rt.jar`也许要添加，jre的路径可以使用`${System.getProperty('java.home')}`获得。另外，依赖的所有库可以通过一种很简洁的方式表述出来，不需要一个依赖一个依赖的添加，`libraryjars files(configurations.compile.collect())`，这句话会把compile环节所依赖的所有库文件的获取到，并添加到libraryjar当中。proguard的配置参数可以直接在gradle的task中写，一般来说是将普通的proguard参数去掉前面的-，参数的值需要写到一个字符串当中，遇到配置字符串需要换行的配置情况需要在最后加上一个\。
同时，还需要将混淆产生的jar包作为shadowJar任务的输入才能将这个混淆的jar包和依赖打包到一起，具体写法如下：
```groovy
task myShadow(type: ShadowJar) {
  baseName = jar.baseName
  from obfuscate
  configurations = [project.configurations.runtime]
  classifier = 'shadow'
  ...
} 
```
from指明了需要打包的jar的来源，这里指定obfuscate就是之前写的obfuscate任务的输出，configurations指定了配置文件，指定之后会根据这个配置文件找到所有的依赖库文件，这里指定的是打包compile环节依赖的库文件，并且`[project.configurations.runtime]`实际上是default `shadowJar` task的默认配置。

这里有一个坑需要注意，如果你使用了默认的shadowJar任务（shadowJar），最后生成的fat-jar会包含有依赖库、没混淆的代码、混淆的代码三部分，正如Stack Overflow上这个问题所描述的一样：https://stackoverflow.com/questions/43643609/gradle-shadowjar-output-contains-obfuscated-and-non-obfuscated-classes?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa
这里产生这种情况的原因是，默认的shadowJar任务总会将main文件夹中的源文件添加到输入当中，要解决这个问题就是自己定义一个type为shadowJar的task，不要去使用默认的shadowJar任务，其实这个问题在shadowJar官方说明文档当中也写到了：
> The built in shadowJar task only provides an output for the main source set of the project. It is possible to add arbitrary ShadowJar tasks to a project. When doing so, ensure that the configurations property is specified to inform Shadow which dependencies to merge into the output.

官方提供了一个例子可以将test中的源文件与testRuntime中依赖的库文件进行打包的方法，也说到了默认的shadowJar任务只能将main中的源文件进行打包，也提示了我们如果要用proguard混淆之后的jar作为输入需要自己定义shadowJar任务，不能使用默认的shadowJar任务。
```groovy
task testJar(type: ShadowJar) {
  classifier = 'tests'
  from sourceSets.test.output
  configurations = [project.configurations.testRuntime]
}
```

### 参考资料
* https://www.guardsquare.com/en/proguard/manual/usage
* https://www.oschina.net/question/237480_166440
* https://stackoverflow.com/questions/43643609/gradle-shadowjar-output-contains-obfuscated-and-non-obfuscated-classes?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa
* http://imperceptiblethoughts.com/shadow/
* https://www.huangyunkun.com/2013/12/23/gradle_with_proguard/
