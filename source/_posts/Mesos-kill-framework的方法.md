---
title: Mesos kill framework的方法
date: 2017-11-16 15:32:39
tags: [Mesos]
categories: [分布式]
---
最近将Spark集群部署到了Mesos之上，运行Spark程序时，能够方便的在Mesos的sandbox中查看程序运行时产生的stderr和stdout，并且这两个文件可以动态加载，不像在Spark管理界面上需要手动点击load more才能加载出来，十分方便。但是也发现了一个问题，当调试Spark代码时手动kill掉spark-submit的进程，或者Ctrl-c时，spark的程序已经停止了，但是在Mesos的管理界面上有时候还会看到有Job在运行，会占用资源，影响之后提交的程序的资源获取。但是Mesos网页管理界面上又没有像Spark管理界面上的kill按钮，就一时不知道如何kill掉这些实际已经不在运行的Job。在Stack Overflow上查到了几种方法，其中我觉得比较好用的一个贴在这里方便之后查看。


```bash
curl -XPOST http://mesos-master-ip-address:5050/master/teardown -d 'frameworkId=<frameId-you-want-to-kill>'
```
这里使用到了Mesos的HTTP Endpoint teardown，有很多人可能会查到是shutdown，但是从Mesos某个版本之后，已经从shutdown改为了teardown，具体的有关HTTP Endpoint的官方文档说明在参考文档当中。

指令发送了一个POST请求给Mesos的一个HTTP Endpoint，要求关闭指定frameworkId的framework。可以将这个指令写在`~/.bashrc`中，方便之后使用，如下：

```bash
killmesostask(){ curl -XPOST http://gpu-server5:5050/master/teardown -d 'frameworkId='$@''; } ;
```
之后就可以使用`killmesostask <frameworkId>`的方式来kill指定Id的framework，需要注意的是这个指令一定要在mesos的master所在的服务器上运行，如果使用了zookeeper，master有时候会发生变化，需要在每个运行mesos-master进程的机器上都加上这个语句，需要kill的时候在对应那个时刻为master的机器上运行指令。
<!--more-->
还有一种比较复杂的方法，但可以不需要在mesos master所在的机器上运行，可以在任何机器上使用Mesos的http api来kill指定的framework，参考官方说明的如下json包：

```json
TEARDOWN Request (JSON):
POST /api/v1/scheduler  HTTP/1.1

Host: masterhost:5050
Content-Type: application/json
Mesos-Stream-Id: 130ae4e3-6b13-4ef4-baa9-9f2e85c3e9af

{
  "framework_id"    : {"value" : "12220-3440-12532-2345"},
  "type"            : "TEARDOWN"
}

TEARDOWN Response:
HTTP/1.1 202 Accepted
```
可以通过postman模拟这样一个json包发送到指定的地址，就同样能够实现kill指定frameworkId的framework的功能。

参考文档：

* http://mesos.apache.org/documentation/latest/endpoints/
* http://mesos.apache.org/documentation/latest/scheduler-http-api/#teardown
