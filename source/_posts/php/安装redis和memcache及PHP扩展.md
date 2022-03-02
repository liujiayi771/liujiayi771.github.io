---
title: 安装redis&memcache及PHP扩展
date: 2016-1-7 20:18:47
tags: [PHP]
categories: [技术]
---
## 安装redis
* 下载地址： http://redis.io/download
* 解压并进入解压目录，并进行编译安装。直接make & make install
* 拷贝配置文件到 /etc 目录，将其中的daemonize no改为daemonize yes
```bash
cp redis.conf /etc/
```
* 将解压目录中 src/ 中的redis-benchmark redis-cli redis-server 拷贝到 /usr/bin 中，方便之后启动
```bash
cp src/redis-benchmark redis-cli redis-server /usr/bin/
```
<!--more-->
* 启动redis
```bash
redis-server /etc/redis.conf
```
* 进入redis
```bash
redis-cli
```
* 关闭redis服务
```bash
redis-cli shutdown
```
## 安装redis的PHP扩展库
* 下载地址： http://pecl.php.net/package/redis
* 将phpize软链接到 /usr/bin
```bash
ln -s /usr/local/php/bin/phpize /usr/bin
```
* 解压并进入解压目录，使用如下命令进行安装
```bash
phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make
make install
```
* 查看输出信息，会告诉你redis.so放置在哪个文件夹下，并复制到PHP的扩展目录下，扩展目录一般为 /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/，在正常情况下，该文件已经被拷贝到正确的位置
* 在php.ini中添加一句extension=redis.so
* 重启PHP，在phpinfo界面会有redis信息显示则说明安装成功
## 安装memcache
* memcache需要使用libevent这个库用于socket的处理，需要安装libevent，下载地址： http://libevent.org/
* 编译安装libevent
```bash
./configure --prefix=/usr
make
make install
```
* memcache下载地址： http://memcached.org/
* 编译安装memcache
```bash
./configure --with-libevent=/usr
make
make install
```
* 启动memcache
```bash
memcached -d -m 2048 -u root -c 1024 -p 11211 -P /tmp/memcached1.pid
```
-p 指定端口号（默认11211）
-m 指定最大使用内存大小（默认64MB）
-t 线程数（默认4）
-l 连接的IP地址, 默认是本机
-d start 启动memcached服务
-d restart 重起memcached服务
-d stop|shutdown 关闭正在运行的memcached服务
-m 最大内存使用，单位MB。默认64MB
-M 内存耗尽时返回错误，而不是删除项
-c 最大同时连接数，默认是1024
-f 块大小增长因子，默认是1.25
-n 最小分配空间，key+value+flags默认是48
## 安装memcache的PHP扩展库
* 下载地址： http://pecl.php.net/package/memcache
* 用如下指令进行安装
```bash
phpize
./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config --with-zlib-dir
make
make install
```
* 在php.ini中增加一行：extension=memcache.so
* 查看phpinfo()的输出，观察memcache的扩展库是否存在

## PHP7安装memcache的PHP扩展库的问题
* 原生的memcache扩展库在PHP7中已经不能编译了，需要使用github上的pecl-memcache分支版本
https://github.com/websupport-sk/pecl-memcache

* 此外，还有一个扩展库叫memcached，与memcache不同，功能更强大，可以在https://github.com/rlerdorf/php-memcached 进行下载安装，编译需要安装libmemcached
```
sudo apt install libmemcached-dev
```
* 再提供一个redis扩展库的github地址https://github.com/phpredis/phpredis/ ，后续redis扩展库对PHP7支持不好可以从该地址下载进行扩展库安装
