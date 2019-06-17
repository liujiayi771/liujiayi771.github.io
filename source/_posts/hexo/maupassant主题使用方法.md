---
title: maupassant主题使用方法
date: 2016-04-11 16:38:13
tags: hexo
---
近期将操作系统重装之后需要重新部署hexo，安装maupassant主题时出了点问题，主题安装后所有网页输出均为空。报错为deploy方法git不存在和jade文件不能正常产生。解决该问题的方法如下：

* 在hexo目录下运行

```bash
npm install hexo-deployer-git --save
npm install hexo-renderer-jade --save
npm install hexo-renderer-sass --save
```
<!--more-->
* `npm install hexo-renderer-sass`安装时会报错，是因为国内网络问题。需要使用代理或者切换至淘宝NPM镜像安装，地址为 http://npm.taobao.org/ 
* 淘宝NPM镜像安装方法为将npm指令重写为cnpm指令：

```bash
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
```
用cnpm指令替代npm指令即可成功安装`hexo-renderer-sass`

