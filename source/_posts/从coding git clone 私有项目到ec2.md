---
title: 从coding git clone 私有项目到ec2
date: 2016-03-24 13:04:40
tags: [EC2,git]
---
之前从 coding git clone 私有项目到EC2服务器一直出现400的网页错误，不知道如何解决，今天查阅资料后发现需要以一下格式来 git clone 才能将私有项目成功 clone 下来。
```bash
git clone -b sci-demo https://<coding_username>:<coding_password>@git.coding.net/<coding_username>/<repository_name>.git
```
<coding_username> 为coding的账户名
<coding_password> 为coding的密码
<coding_repository> 为 clone 的项目的仓库名
实际上只需要将用户名和密码部分加上，其他部分的内容直接拷贝 git 地址即可
