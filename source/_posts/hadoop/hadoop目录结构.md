---
title: hadoop目录结构
date: 2015-12-24 19:39:29
tags: [hadoop]
categories: [分布式]
---
## **bin**
hadoop最基本的管理脚本和使用脚本所在的目录，这些脚本是sbin目录下管理脚本的基础实现，用户可以直接使用这些脚本管理和使用hadoop
## **etc**
hadoop配置文件所在的目录，包括core-site.xml、hdfs-site.xml、mapred-site.xml等从hadoop 1.0继承而来的配置文件和yarn-site.xml等hadoop 2.0新增的配置文件
## **include**
对外提供的编程库头文件（具体动态库和静态库在lib目录中，这些头文件均是用C++定义的，通常用于C++程序访问HDFS或者编写MapReduce程序）
<!--more-->
## **lib**
该目录包含了hadoop对外提供的编程动态库和静态库，与include目录中的头文件结合使用
## **libexec**
各个服务对应的shell配置文件所在目录，可用于配置日志输出目录、启动参数（比如JVM参数）等基本信息
## **sbin**
hadoop管理脚本所在目录，主要包含HDFS和YARN中各类服务的启动/关闭脚本
## **share**
hadoop各个模块编译后的jar包所在目录
