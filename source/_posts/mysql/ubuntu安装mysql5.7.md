---
title: ubuntu安装mysql5.7
date: 2016-6-17 12:30:47
tags: [mysql]
categories: [技术]
---
### 准备工作
1.下载mysql5.7源码进行编译安装： http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.13.tar.gz
2.安装必备的包和工具

```
sudo apt install build-essential
sudo apt install cmake
sudo apt install bison
sudo apt install libncurses5-dev
```
  编译安装zlib（安装nginx时一般已经安装）
3.安装功能需要的包

```
sudo apt install libxml
sudo apt install openssl
```
4.下载Boost库
从mysql5.7.5开始Boost库是必须的，一般需要的版本为1_59_0，下载地址为： http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.10.tar.gz
下载后解压到与mysql源码相同的目录处
<!--more-->
5.直接安装mysql在后续操作中可能会出现启动mysql段错误的情况，需要在编译修改源代码来解决
在源码包中，找到`cmd-line-utils/libedit/terminal.c`，找到其中`area = buf`的语句，将其改为`area = NULL`即可

### 创建用户组和用户

```
sudo groupadd mysql
sudo useradd mysql
```

### 源码编译及安装
1.mysql采用cmake的方式进行编译，编译指令如下：

```
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
-DDEFAULT_COLLATION=utf8_general_ci \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=../boost_1_59_0
```
2.编译完成之后，使用`make`及`make install`指令进行安装

### 配置mysql
1.安装完成之后，将安装目录的所有者改为mysql
```
sudo chown -R mysql:mysql /usr/local/mysql
```
2.复制配置文件到系统`etc`目录下，并修改所有者为mysql

```
sudo cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
```
修改其中的内容如下所示：

```
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
```
3.配置mysql开机启动，将mysql.server文件复制到系统`etc/init.d`目录当中

```
sudo cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
sudo chmod +x /etc/init.d/mysql
sudo update-rc.d -f mysql defaults
```

### 初始化数据库
从mysql5.7开始，之前的mysql_install_db已经被废弃了，需采用新的指令进行数据库初始化操作

```
sudo /usr/local/mysql/bin/mysqld \
--initialize-insecure \
--user=mysql \
--basedir=/usr/local/mysql \
--datadir=/usr/local/mysql/data
```
需要注意的是：
--initialize会生成一个随机密码(~/.mysql_secret)
--initialize-insecure不生成密码
--datadir目标目录下不能有数据文件

### 开启mysql服务
开启mysql

```
sudo service mysql start
```

### 修改数据库密码
之后可以使用`mysql -uroot -p`输入空密码进入mysql，采用如下语句修改mysql的密码

```sql
set password for root@localhost = password('123');
```
