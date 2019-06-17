---
title: CentOS 7.0 编译安装VIPS dzsave
date: 2016-1-13 15:07:47
tags: []
categories: [技术]
---
## 介绍
VIPS 是一个能够处理大型图片的很棒的图片处理系统。特别是它能通过dzsave很简单地生成deepzoon图片（.dzi）。从VIPS版本7.40开始，它需要使用libgsf库来激活对dzsave的支持。
## 安装libgsf库
libgsf库的作用在于能够使用dzsave指令，没有安装libgsf的话会报dzsave不存在的错误。安装libgsf之后需要重新编译安装vips才能使用dzsave。
```bash
yum install libgsf
```
ubuntu中安装libgsf库的方法为：

```bash
sudo apt-get install libgsf-1-dev
```
## 安装libjpeg和libpng库
vips要支持png和jpeg还需要安装libpng和libjpeg，在CentOS中安装的方法为：
```bash
sudo yum install libjpeg libjpeg-devel libpng libpng-devel

```
在ubuntu中安装方法为：

```bash
sudo apt-get install linpng12-0
sudo apt-get install libjpeg-dev
```
## 安装VIPS
简单的编译安装，采用`vips dzsave --version`来检查dzsave是否安装成功

## 使用VIPS
```bash
/usr/local/vips dzsave <source_file> <destination_file> --suffix .png
```
