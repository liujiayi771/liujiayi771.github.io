---
title: Linux搭建NAT共享网络
date: 2017-06-14 10:32:20
tags: [Linux]
categories: [技术]
---
### 1. 配置双网卡静态IP信息

`sudo vim /etc/sysconfig/network-scripts/ifcfg-enp15s0`

配置该网卡为外网IP，需要添加的内容如下

```bash
BOOTPROTO=static
ONBOOT=yes
DNS1=202.114.0.242
IPADDR=XXX.XXX.XXX.XXX
PREFIX=23
GATEWAY=115.156.163.254
```

`sudo vim /etc/sysconfig/network-scripts/ifcfg-eno1`

配置该网卡转发enp15s0的网络包，需要添加的内容如下

```bash
BOOTPROTO=static
ONBOOT=yes
DNS1=202.114.0.242
IPADDR=10.0.0.1
PREFIX=24
GATEWAY=10.0.0.1
```

### 2. 配置系统内核实现ipv4转发

`sudo vim /etc/sysctl.conf`

添加如下内容

```bash
# Controls IP packet forwarding
net.ipv4.ip_forward = 1
```

如果需要不重启实现ipv4转发功能可以使用如下指令

`sysctl -w net.ipv4.ip_forward=1`

### 3. 修改iptables转发规则
运行如下指令

`sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o enp15s0 -j SNAT --to XXX.XXX.XXX.XXX`

`iptables -A OUTPUT -p tcp --dport 53 -j ACCEPT`

`iptables -A OUTPUT -p udp --dport 53 -j ACCEPT`

保存iptables配置的结果防止重启失效

`sudo iptables-save`


### 4. 其他一些关于网络的问题
* 多网卡机器1号网卡作为内网网段，2号网卡作为外网网段不能上网的问题，可以通过修改默认路由来实现。一般来说，默认路由为1号网卡（如果没有配置的话），设置路由使用route指令，route指令说明如下：

```
Usage: route [-nNvee] [-FC] [<AF>]           List kernel routing tables
       route [-v] [-FC] {add|del|flush} ...  Modify routing table for AF.

       route {-h|--help} [<AF>]              Detailed usage syntax for specified AF.
       route {-V|--version}                  Display version/author and exit.

        -v, --verbose            be verbose
        -n, --numeric            don't resolve names
        -e, --extend             display other/more information
        -F, --fib                display Forwarding Information Base (default)
        -C, --cache              display routing cache instead of FIB

  <AF>=Use -4, -6, '-A <af>' or '--<af>'; default: inet
  List of possible address families (which support routing):
    inet (DARPA Internet) inet6 (IPv6) ax25 (AMPR AX.25)
    netrom (AMPR NET/ROM) ipx (Novell IPX) ddp (Appletalk DDP)
    x25 (CCITT X.25)
```
使用如下指令来添加默认路由：

```
sudo route add default gw 10.0.0.1
```
将默认网关设置为10.0.0.1这个上网的网段即可上网
