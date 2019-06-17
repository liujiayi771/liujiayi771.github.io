---
title: CentOS 7安装Facebook folly库
date: 2019-04-26 14:47:01
tags: Linux
---
folly是Facebook提出的一个基于C++14的C++组件库，包含有非常多具有实用性和效率性的数据结构。其github地址为： https://github.com/facebook/folly 。在其README当中提供了在ubuntu和MacOS中的安装教程，没有CentOS中的安装教程，并且在CentOS当中安装会遇到很多问题，本文对在CentOS 7系统中安装folly库进行了过程记录。

最新的folly需要gcc的版本高于5.1，但是CentOS 7中的gcc版本为4.8，直接升级安装gcc非常麻烦，并且可能会对其他项目使用的gcc造成影响，以采用我另一篇博客中提及的方法切换到高版本的gcc： http://joey771.cn/2018/04/16/linux/CentOS%E5%AE%89%E8%A3%85%E9%AB%98%E7%89%88%E6%9C%ACgcc/ 

从官网提供的ubuntu安装教程中可以看出，其依赖于boost-devel、glog-devel、gflags-devel、double-conversion等库，有些库可以通过yum来进行安装，有些则需要通过源码编译安装。首先是可以使用yum安装的库：

```bash
sudo yum install boost-devel \
double-conversion-devel \
libevent-devel \
openssl-devel \
lz4-devel \
xz-devel \
snappy-devel \
zlib-devel \
jemalloc-devel \
libtool \
scons
```

编译安装gflags、glog

```bash
git clone https://github.com/schuhschuh/gflags.git
mkdir gflags/build
cd gflags/build
cmake -DGFLAGS_NAMESPACE=google -DBUILD_SHARED_LIBS=on ..
make -j 8
sudo make install

git clone https://github.com/google/glog.git
cd glog
autoreconf -ivf
./configure --with-gflags=/usr/local/lib
make -j 8
sudo make install
```

编译安装folly

```bash
git clone https://github.com/facebook/folly.git
cd folly
mkdir _build && cd _build
cmake ..
make -j 8
sudo make install
```