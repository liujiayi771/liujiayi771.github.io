---
title: sudo不需要使用密码
date: 2016-06-17 16:36:56
tags: [Linux]
categories: [技术]
---
在使用linux时，sudo指令是经常会用到的，但是这个加sudo之后经常需要输入密码十分不便，可以通过设置来使加sudo的指令不需要密码，编辑`/etc/sudoer`文件
```
sudo vim /etc/sudoers
```
在其中`root    ALL=(ALL:ALL) ALL`后加一行` hadoop    ALL=NOPASSWD:ALL`，即可使hadoop用户sudo不需要密码，其他用户的设置类似
