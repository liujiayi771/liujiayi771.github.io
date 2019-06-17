---
title: CentOS 7.0 安装PHP 5.6
date: 2016-1-7 12:18:47
tags: [PHP]
categories: [技术]
---
## 1. 下载和安装PHP 5.6
* 下载地址： http://php.net/downloads.php
* 解压之后使用如下指令进行安装
```bash
./configure \
--prefix=/usr/local/php \
--with-curl \
--with-gd \
--enable-sockets \
--with-freetype-dir=/usr/local/freetype \
--enable-mbstring \
--enable-bcmath \
--enable-gettext \
--with-jpeg-dir=/usr/local/jpeg \
--with-config-file-path=/usr/local/php/etc \
--with-mysql=/usr/local/mysql \
--with-mysqli=/usr/local/mysql/bin/mysql_config \
--with-mysql-sock=/var/lib/mysql/mysql.sock \
--with-pdo-mysql=/usr/local/mysql \
--enable-fpm \
--disable-fileinfo

make
make install
```
<!--more-->
* 对于内存小于1G的电脑来说，make时可能会出现内存溢出的问题，解决方法为./configure加上选项--disable-fileinfo
## 2. 配置PHP
* 复制PHP配置文件到安装目录
```bash
cp php.in-production /usr/local/php/etc/php.ini
```
* 修改php.ini文件，将其中的date.timezone = PRC
* 删除系统自带的PHP配置文件
```bash
rm -rf /etc/php.ini
```
* 添加软连接到 /etc 目录
```bash
ln -s /usr/local/php/etc/php.ini /etc/php.ini
```
* 拷贝模板文件为php-fpm配置文件
```bash
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
```
* 添加软连接到 /etc 目录
```bash
ln -s /usr/local/php/etc/php-fpm.conf /etc/php-fpm.conf
```
* 修改php-fpm.conf文件，在user和group处添加当前用户名和组，取消pid=run/php-fpm.pid前的分号
## 3. 启动、关闭和重启PHP
* 启动PHP
```bsh
/usr/local/php/sbin/php-fpm
```
* 关闭PHP
```bash
kill -INT `cat /usr/local/php/var/run/php-fpm.pid`
```
* 重启PHP
```bash
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid`
```
