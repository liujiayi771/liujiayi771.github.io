---
title: 导入hadoop源码至Eclipse
date: 2015-12-24 20:04:40
tags: [hadoop]
categories: [分布式]
---
从hadoop官网下载hadoop源码，解压后进入目录，阅读BUILDING.txt文件，文件中讲到了如何将hadoop源码工程文件导入到Eclipse中。首先需要给Eclipse安装hadoop-maven-plugins插件，进入hadoop-maven-plugins目录执行指令
```bash
cd hadoop-maven-pligins
mvn install
mvn eclipse:eclipse -DskipTests
```
最后，在Eclipse中点击[File] > [Import] > [Existing Projects into Workspace]
