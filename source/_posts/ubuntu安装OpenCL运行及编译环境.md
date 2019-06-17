---
title: ubuntu安装OpenCL运行及编译环境
date: 2019-01-18 09:19:46
tags: Linux
---

最近需要运行一个基于OpenCL开发的软件，服务器上的NVIDIA 1080Ti和Intel CPU都支持OpenCL，安装OpenCL之前可以安装一个clinfo的软件可以看到服务器上目前支持的OpenCL设备。

```
sudo apt install clinfo
```
一般安装NVIDIA的驱动的时候就会有选项选择是否安装OpenCL的驱动，若没有选择那个选项，clinfo是看不到有NVIDIA的cl设备的，接下来安装的几个软件一般也会将没有选择OpenCL的NVIDIA缺失的库作为依赖安装上，使得基于NVIDIA的cl设备能够正常使用。

```
sudo apt install ocl-icd-libopencl1
sudo apt install opencl-headers
```
若需要在服务器上编译或者调试OpenCL程序，还需要安装

```
sudo apt install ocl-icd-opencl-dev
```
要使得CPU也支持运行OpenCL程序，需要去intel官网下载opencl-sdk，地址是:http://software.intel.com/en-us/vcsource/tools/opencl-sdk ，选择runtime版本进行下载，目前最新的适合ubuntu的地址是： http://registrationcenter-download.intel.com/akdlm/irc_nas/vcp/12526/opencl_runtime_16.1.2_x64_rh_6.4.0.37.tgz ，适合CentOS的地址是： http://registrationcenter-download.intel.com/akdlm/irc_nas/vcp/13454/opencl_runtime_16.1.2_x64_rh_6.4.0.37.tgz 

安装前需要先安装`lsb-core`，sdk依赖于这个库，版本必须大于4.0

```
sudo apt install lsb-core
```
CentOS系统为

```
sudo yum install redhat-lsb-core
```
之后`sudo ./install`，检查操作系统版本时会提出ubuntu not support，这里可以忽略，这里说的只是不提供支持，而不是不能够使用，intel官方回复说对centos系统提供较好的支持，但ubuntu也是能使用的。

>1) The OpenCL™ implemenations themselves provided at the link are supported. Intel® CPU Runtime for OpenCL™ Applications on Ubuntu* OS is supported. This support is distinct from the 2017 SDK support.

>2) The 2017 SDK is supported on CentOS*. It is expected compatible with Ubuntu* OS.

>The semantic difference is that for supported there is an immediate paid path to get priority support and an intent to create service level agreements for CentOS* only. 'supported' is an overloaded word in the software industry... Hopefully the installer messaging will be more clear in the upcoming 2019 SDK.

具体可以看 https://software.intel.com/en-us/forums/opencl/topic/785262