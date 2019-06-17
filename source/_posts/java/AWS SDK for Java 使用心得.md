---
title: AWS SDK for Java 使用心得
date: 2016-3-17 21:18:47
tags: [Java]
categories: [技术]
---
AWS SDK 是一套用于开发者与 Amazon Web Services进行交互的系统，其功能繁多，对于普通开发者来说，本人主要使用了其针对Java开发的向S3云存储上传文件和文件夹的功能。AWS开发工具包的下载地址：
https://aws.amazon.com/cn/tools/
其中的Java开发工具包的API文档地址：
http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/index.html
进行开发之前建议自习阅读其官方的开发引导：
http://docs.aws.amazon.com/AWSSdkDocsJava/latest/DeveloperGuide/welcome.html

### 心得
* 开发AWS应用仅靠API接口的阅读是不够的，最好能够仔细阅读官方SDK中的Sample文件夹中的程序，熟悉整个SDK开发的代码流程，也可以到论坛和GitHub中找与AWS相关的代码进行阅读
* AWS SDK for Java 中有两种上传文件的方法，一种是构造一个PutObjectRequest，调用其中的构造函数：

```
PutObjectRequest(String bucketName, String key, File file)
Constructs a new PutObjectRequest object to upload a file to the specified bucket and key.
```
构造一个上传文件的PutObjectRequest对象，然后使用AmazonS3Client类中的putObject方法来实现上传文件的操作：
```
public PutObjectResult putObject(PutObjectRequest putObjectRequest)
                          throws AmazonClientException,
                                 AmazonServiceException
```
<!--more-->
还有一种方法是使用TransferManager类中的upload方法：
```
TransferManager(AmazonS3 s3)
Constructs a new TransferManager, specifying the client to use when making requests to Amazon S3.
```
```
public Upload upload(String bucketName,
            String key,
            File file)
              throws AmazonServiceException,
                     AmazonClientException
```
upload方法同样也可以传入一个PutObjectRequest对象，作用与之前描述的相同。
这两种方法都可以成功上传文件，但是前一种方法传输文件的速度比较慢但是不会出现timeout的问题，是一种简单并且稳定的传输方式，缺点就是传输速度很慢，并且只能传单个文件，不能传文件夹。传文件夹可以使用递归的方式通过深度搜索来传输文件夹中的每一个文件，但是在文件夹中文件数量很多的情况下，这样传输的方式会十分缓慢。后一种方法可以使用uploadDirectory方法来传输一个文件夹：
```
public MultipleFileUpload uploadDirectory(String bucketName,
                                 String virtualDirectoryKeyPrefix,
                                 File directory,
                                 boolean includeSubdirectories)
```
并且这种方式的传输在SDK内部是通过多线程的方式实现的，将文件传输分为多个线程来进行，传输速度慢，在传输一段时间之后会出现timeout的情况。
关于timeout的清空，网上众说纷纭，有人说通过设置AmazonS3Client的配置文件可以解决，有人说可以通过修改jre的networkaddress.cache.ttl时间为60秒可以解决。
```java
java.security.Security.setProperty("networkaddress.cache.ttl", "60");
```
但是经过本人的尝试，这个timeout的出现还是与你所在的地区以及你的网速决定的，当我将代码在EC2服务器上运行时，是不会出现timeout的情况的，并且运行速度很快。
* 在建立transfer连接之后一定要记得使用shudownNow函数来关闭连接，否则程序会一直等待连接的断开。
* 使用SDK与Amazon Web Service 进行连接需要提供 access key 和 secret access key，这些内容需要放到默认的~/.aws/credentials文件中去。
* 各种可配置的实例的配置文件一般在创建实例的时候传入。
* 使用upload和uploadDirectory方法传输文件时，要使用Upload.waitForCompletion函数来阻塞程序运行，等待传输完毕。
