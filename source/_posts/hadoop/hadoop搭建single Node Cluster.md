---
title: Hadoop搭建single Node Cluster
date: 2015-12-16 16:18:47
tags: [hadoop]
categories: [分布式]
---
### 1. 添加Hadoop系统用户组和用户
* 使用以下命令在终端中执行先来创建一个用户组：
```sh
sudo addgroup hadoop
```
* 使用以下命令来添加用户
```sh
sudo adduser --ingroup hadoop hadoop
```
<!--more-->
### 2. 配置SSH
* 首先需要切换到hadoop用户，输入以下命令
```sh
su hadoop
```
* 创建一个新的秘钥
```sh
ssh-keygen -t rsa -P ""
```
* 使用此秘钥启用SSH访问本地计算机
```sh
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
* 也可以采用如下语句
```sh
ssh-copy-id hadoop@localhost
```
* 测试是否配置正确
```sh
ssh localhost
```
* 若能正确连接localhost则说明SSH配置正确
* 若看到如下错误响应，可能SSH在此系统不可用，可以purge掉openssh-server之后重新进行安装

### 3. 安装、配置Java
* 在浏览器中打开网址： http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
* 下载最新的稳定的jdk(Java SE Developmet Kit)
* 将下载的文件解压到目录`/opt/java/jdk`，并改名为`jdk1.8`（我下载的是1.8版本的JDK）
* 修改`/etc/bash.bashrc`文件，在最后加上如下内容
```bash
export JAVA_HOME=/opt/java/jdk/jdk1.8
export CLASSPATH=${JAVA_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
* 在终端中输入命令
```bash
java -version
```
* 若输出如下内容，则说明安装、配置正确
```bash
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
```
### 4. 下载Hadoop
* 在浏览器中打开网址：  http://hadoop.apache.org/releases.html
* 下载新版本的二进制包，下载完成后解压到目录`/usr/local`，并改名为`hadoop`
* 更改文件权限，使hadoop用户能够拥有该文件夹的权限
```bash
cd /usr/local
sudo chown -R hadoop:hadoop hadoop
```
### 5. 配置Hadoop
* 修改`/etc/bash.bashrc`文件，在最后加上如下内容
```sh
#Set HADOOP_HOME
export HADOOP_HOME=/usr/local/hadoop
#Set JAVA_HOME
export JAVA_HOME=/usr/local/jdk1.8.0_60
# Add bin/ directory of Hadoop to PATH
export PATH=$PATH:$HADOOP_HOME/bin
```
* 在文件`/usr/local/hadoop/etc/hadoop/hadoop-env.sh`中加入JAVA_HOME的完整路径，如下所示
```bash
export JAVA_HOME=/opt/java/jdk/jdk1.8
```
* 在`/usr/local/hadoop/etc/hadoop/core-site.xml`文件中还有两个参数需要设置：
    1. 'hadoop.tmp.dir'-用于制定目录让Hadoop来存储其数据文件
    2. 'fs.default.name'-指定默认的文件系统
```xml
<configuration>
	<property>
  		<name>hadoop.tmp.dir</name>
  		<value>/app/hadoop/tmp</value>
  		<description>A base for other temporary directories.</description>
	</property>

	<property>
  		<name>fs.default.name</name>
  		<value>hdfs://localhost:54310</value>
  		<description>The name of the default file system.  A URI whose
  		scheme and authority determine the FileSystem implementation.  The
  		uri's scheme determines the config property (fs.SCHEME.impl) naming
		the FileSystem implementation class.  The uri's authority is used to
		determine the host, port, etc. for a filesystem.</description>
	</property>
</configuration>
```
* 创建目录，如上面配置core-site.xml中使用的目录：`/app/hadoop/tmp`，并授予权限
```bash
sudo mkdir -p /app/hadoop/tmp
sudo chown -R hadoop:hadoop /app/hadoop/tmp
sudo chmod 750 /app/hadoop/tmp
```
* 配置文件`/usr/local/hadoop/etc/hadoop/mapred-site.xml`
```bash
sudo cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml
```
* 在mapred-site.xml文件中添加以下内容
```xml
<configuration>
	<property>
	  <name>mapred.job.tracker</name>
	  <value>localhost:54311</value>
	  <description>The host and port that the MapReduce job tracker runs
	  at.  If "local", then jobs are run in-process as a single map
	  and reduce task.
	  </description>
	</property>
</configuration>
```
* 配置`/usr/local/hadoop/etc/hadoop/hdfs-site.xml`
```xml
<configuration>
	<property>
	  <name>dfs.replication</name>
	  <value>1</value>
	  <description>Default block replication.
	  The actual number of replications can be specified when the file is created.
	  The default is used if replication is not specified in create time.
	  </description>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/home/hadoop/hdfs</value>
	</property>
</configuration>
```
### 6. 使用Hadoop
* 在第一次使用Hadoop之前，需要先格式化HDFS，使用如下命令
```bash
/usr/local/hadoop/bin/hadoop namenode -format
```
* 可能会遇到创建文件夹权限问题，可以自行使用sudo进行文件夹的创建，或使用sudo来执行上面的命令
* 使用以下命令启动Hadoop的单节点集群（使用hadoop用户来启动）
    ```bash
    /usr/local/hadoop/sbin/start-dfs.sh
    /usr/local/hadoop/sbin/start-yarn.sh
    ```
* 也可以使用`/usr/local/hadoop/sbin/start-all.sh`来启动所有功能
* 使用jps验证是否所有Hadoop相关的Java进程正在运行`/opt/java/jdk/jdk1.8/bin/jps`
* 关闭Hadoop相关功能使用`stop-*.sh`，具体可以参照`/usr/local/hadoop/sbin`中的文件内容
* 测试
  * ALL Application: http://localhost:8088/
  * DataNode: http://localhost:50070/
