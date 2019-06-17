---
title: CentOS 7.0 安装Nginx
date: 2016-1-7 11:18:47
tags: [nginx]
categories: [技术]
---
## 1. 所需库的安装
* pcre库
	* 下载地址： ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/
	* 安装指令：
```bash
./configure
make
make install
```
<!--more-->
* zlib库
	* 下载地址： http://www.zlib.net/
	*安装指令：
```bash
./configure
make
make install
```
## 2. 安装nginx
* 下载地址： http://nginx.org/en/download.html
* 安装指令：
```bash
./configure --prefix=/usr/local/nginx
make
make install
```

* 安装完毕后开启nginx服务器：
```bash
/usr/local/nginx/sbin/nginx
```
* 启动之后在浏览器输入： http://localhost ，若能出现welcome to nginx！的界面则说明安装成功
* 重启、关闭nginx服务器
```bash
/usr/local/nginx/sbin/nginx -s reload
/usr/local/nginx/sbin/nginx -s stop
```
