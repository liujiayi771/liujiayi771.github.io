---
title: CentOS 7.0 安装MySQL 5.6
date: 2016-1-7 12:18:47
tags: [mysql]
categories: [技术]
---
## 1. 安装依赖
* 使用yum安装一些依赖
```bash
yum -y install make gcc gcc-c++ cmake bison-devel ncurses-devel
yum -y install kernel-devel readline-devel openssl-devel openssl
```
## 2. 设置MySQL用户和组
* 新增mysql用户组
```bash
groupadd mysql
```
* 新增mysql用户
```bash
useradd -r -g mysql mysql
```
<!--more-->
## 3. 新建MySQL所需要的目录
```bash
cd /usr/local/
mkdir mysql
cd mysql/
mkdir data
```
## 4. 下载安装MySQL
* 下载mySQL安装包
	* 下载地址： http://dev.mysql.com/downloads/mysql/5.6.html#downloads
	* 选择版本号，Platform选择Source Code，下载.tar.gz文件

* 编译安装MySQL
	* 从MySQL 5.5起，MySQL源码安装开始使用cmake了，设置源码编译配置脚本
```bash
-DCMAKE_INSTALL_PREFIX=dir_name	//设置MySQL安装目录
-DMYSQL_UNIX_ADDR=file_name	//设置监听套接字路径，这必须是一个绝对路径名，默认为/tmp/r
-DDEFAULT_CHARSET=charset_name	//设置服务器的字符集，缺省情况下，MySQL使用latin1的（CP1252西欧）字符集，cmake/character_sets.cmake文件包含允许的字符集名称列表
-DDEFAULT_COLLATION=collation_name	//设置服务器的排序规则
-DMYSQL_DATADIR=dir_name	//设置MySQL数据库文件目录
-DMYSQL_TCP_PORT=port_num		//设置MySQL服务器监听端口，默认为3306
-DSYSCONFDIR=dir_name	//配置文件 my.cnf 目录
-DENABLED_LOCAL_INFILE=1	//启用加载本地数据
-DEXTRA_CHARSETS=all	//扩展字符支持，默认值为all
-DDEFAULT_COLLATION=utf8_general_ci	//默认字符校对
```
	* cmake编译指令
```bash
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci


make
make install
```
## 5. 修改安装目录所有者
```bash
cd /usr/local/mysql
chown -R mysql:mysql .
```

## 6. 初始化和启动MySQL
* 进入安装目录执行脚本，启动服务
```bash
cd /usr/local/mysql
scripts/mysql_install_db --bashdir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
cp support-files/mysql.server /etc/init.d/mysql
rm /etc/my.cnf
cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
chkconfig mysql on
service mysql start
chkconfig --level 35 mysql on	//可选，设置后MySQL会开机自启动
```
* 设置之前，需先设置PATH要不然不能直接调用mysql这个指令
```bash
ln -s /usr/local/mysql/bin/mysql /usr/bin/
```
## 7. 检查MySQL服务是否启动
```bash
mysql -u root -p
```
* 密码为空，如果能登陆上，则安装成功
* 修改密码，登陆mysql，依次输入如下指令
```SQL
use mysql;
update user set password=password('your_passward') where user='root';
flush privileges;
```
## 8. 可能会出现的问题
```bash
问题：
Starting MySQL..The server quit without updating PID file ([FAILED]/mysql/Server03.mylinux.com.pid).
解决：
修改/etc/my.cnf 中datadir,指向正确的mysql数据库文件目录
```
```bash
问题：
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
解决：
新建一个链接或在mysql中加入-S参数，直接指出mysql.sock位置
ln -s /usr/local/mysql/data/mysql.sock /tmp/mysql.sock
/usr/local/mysql/bin/mysql -u root -S /usr/local/mysql/data/mysql.sock
```
