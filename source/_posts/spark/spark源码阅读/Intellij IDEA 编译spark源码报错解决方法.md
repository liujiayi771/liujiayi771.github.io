---
title: Intellij IDEA 编译spark源码报错解决方法
date: 2017-07-05 17:09:46
tags: [spark]
categories: [Spark源码阅读]
---

### SparkFlumeProtocol
```
Error:(45, 66) not found: type SparkFlumeProtocol
  val transactionTimeout: Int, val backOffInterval: Int) extends SparkFlumeProtocol with Logging {
```
这个问题是由于flume-sink所需要的部分源文件idea不会自动下载，所以编译时不能通过。

#### 解决方式
在Intellij IDEA里面：

- 打开View -> Tool Windows -> Maven Projects
- 右击Spark Project External Flume Sink
- 点击Generate Sources and Update Folders
随后，Intellij IDEA会自动下载Flume Sink相关的包

### SqlBaseParser
```
Error:(36, 45) object SqlBaseParser is not a member of package org.apache.spark.sql.catalyst.parser
import org.apache.spark.sql.catalyst.parser.SqlBaseParser._
```
这个问题和之前的问题类似，是idea不会自动下载部分catalyst相关的源文件，导致编译时不能通过。

#### 解决办法
在Intellij IDEA里面：

- 打开View -> Tool Windows -> Maven Projects
- 右击Spark Project Catalyst
- 点击Generate Sources and Update Folders
随后，Intellij IDEA会自动下载Catalyst相关的包

## 总结
IDEA编译spark源码出错大多都是不能正确下载相关的源文件，在Maven的管理菜单内选择相对应的库选择下载源码并更新即可解决大多数问题。

spark build error集合：
https://www.mail-archive.com/search?l=user@spark.apache.org&q=subject:%22Build+error%22&o=newest&f=1
