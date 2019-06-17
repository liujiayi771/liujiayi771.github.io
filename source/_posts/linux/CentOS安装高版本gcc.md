---
title: CentOS安装高版本gcc
date: 2018-04-16 19:37:29
tags: [Linux]
---

最近需要在CentOS 7.2上安装matlab，发现matlabR2017b Linux版本安装需要gcc 4.9.x，但是CentOS 7.2 使用yum安装的gcc版本最高为4.8.5，于是决定将gcc版本进行升级。升级gcc一般建议采用编译安装的方式，但是这种方式比较麻烦，需要先编译安装mpfr、gmp、mpc等，于是在网上找到了一种通过yum比较方便的升级方式，而且可以随时在bash、zsh等当中切换各种gcc版本，特记录在此。

## gcc 4.9安装
```bash
sudo yum install centos-release-scl
sudo yum install devtoolset-3-toolchain
scl enable devtoolset-3 bash
```
这里scl enable就是用来切换不同版本的gcc的。这个切换是临时的，表示在bash中临时切换到gcc4.9的工作环境，当使用`exit`指令之后，就会回退到原始的gcc版本，可以使用`scl -l`来查看所有可以切换的开发工具集。

## gcc 5.2
```bash
sudo yum install centos-release-scl
sudo yum install devtoolset-4-toolchain
scl enable devtoolset-4 bash
```
