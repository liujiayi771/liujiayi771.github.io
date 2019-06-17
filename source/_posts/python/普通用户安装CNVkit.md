---
title: 普通用户安装CNVkit
date: 2017-08-08 10:49:54
tags: [python]
categories: [技术]
---
安装python的library的时候经常需要sudo权限，原因是许多机器上安装的python都是使用apt-get(ubuntu)、yum(CentOS)安装的，在这种情况下python被安装到了root用户目录下，再使用pip安装各种库的时候，有时候库需要被安装到这些root用户目录下，就会出现权限问题。解决这个问题的方法是通过源码安装python到普通用户目录下，再通过源码安装setuptools和pip便能在普通用户下安装python的各种库。

## 1.编译安装python
下载python源码，https://www.python.org/downloads/source/
这里选择python2.7.13进行测试，https://www.python.org/downloads/release/python-2713/
下载xz压缩格式的源码，使用`tar -xvf Python-2.7.13.tar.xz`解压后进入目录，`./configure --prefix=/home/spark/Softwares/python2`通过`--prefix`指定安装目录为普通用户目录，`make`进行编译，`make install`进行安装。之后在`~/.bashrc`文件内添加PATH指定python的安装路径`export PATH=/home/spark/Softwares/python2/bin:$PATH`，然后`source ~/.bashrc`，此时python便被指定为编译安装的python了。

## 2.编译安装setuptools和pip
下载setuptools源码，https://pypi.python.org/pypi/setuptools

下载zip压缩格式源码，使用`unzip setuptools-36.2.7.zip`进行解压，进入目录之后使用`python setup.py install`进行安装，此时需要确保python指令指向的是之前编译安装的python。

下载pip源码，https://pypi.python.org/pypi/pip

下载tar.gz格式的源码，使用`tar -zxvf pip-9.0.1.tar.gz`进行解压，进入目录后，首先使用`python setup.py build`进行编译，然后使用`python setup.py install`进行安装，这之后会发现在编译安装python的路径的bin目录中会有pip和setuptools程序。由于之前已经添加该bin目录到PATH环境变量当中，此时使用pip安装即为普通用户目录下的pip，可以不需要sudo权限安装各种library。
