---
title: ubuntu安装PHP5.6
date: 2016-04-11 16:18:57
tags: [PHP]
categories: [技术]
---
总体上来说，ubuntu安装PHP5.6与CentOS是差不多的，但是有一些库的名称在ubuntu和CentOS上是不一样的，安装过程中还是有一些细微的差别.

* 在安装PHP时，使用的configure指令如下所示：

```bash
./configure \
--prefix=/usr/local/php \
--with-curl \
--with-gd \
--enable-sockets \
--with-freetype-dir=/usr/local/freetype \
--enable-mbstring \
--enable-bcmath \
--with-gettext \
--with-jpeg-dir=/usr/local/libjpeg \
--with-config-file-path=/usr/local/php/etc \
--with-mysql=/usr/local/mysql \
--with-mysqli=/usr/local/mysql/bin/mysql_config \
--with-mysql-sock=/var/lib/mysql/mysql.sock \
--with-pdo-mysql=/usr/local/mysql \
--enable-fpm \
--disable-fileinfo
```
<!--more-->
其中with-curl需要安装curl、libcurl3、libcurl3-dev和php5-curl
需要指定jpeg-dir，采用apt-get安装的libjpeg不能被PHP的gd扩展识别，需要再次手动安装libjpeg，采用编译安装的方式，在 http://www.ijg.org/files/ 下载最新的libjpeg源码，进行编译安装：

```bash
./configure --prefix=/usr/local/libjpeg
make
make install
```
但是libpng是能被gd扩展库识别的，安装好之后可以在phpinfo函数的输出界面观察到支持的结果
freetype也需要手动编译安装，下载地址为 http://download.savannah.gnu.org/releases/freetype/ 下载最新的版本，采用如下指令进行安装：

```bash
./configure --prefix=/usr/local/freetype
make
make install
```

* 在安装PHP扩展库的时候，使用phpize指令可能会出现`Cannot find autoconf.`的错误。在ubuntu中，我们可以使用如下指令来解决这个问题：

```bash
sudo apt-get install m4
sudo apt-get install autoconf
```
