---
title: mysql赋给用户权限
date: 2016-6-17 12:18:47
tags: [mysql]
categories: [技术]
---
```sql
grant all privileges on <dbname>.* to '<username>'@'localhost' identified by '<password>';
flush privileges;
```
<!--more-->

<dbname>为想要共享的数据库名称，如果想共享所有的数据库，则为*号
<username>为共享的用户名，如果该用户不存在，mysql会自动创建该用户，可以进入mysql数据库的user表查看，标识用户的信息除了用户名User，还包含有Host
localhost处可以替换为具体的ip地址，则代表只有该ip用该用户名才能访问，如果希望所有ip的用户都能通过该用户名访问，则将localhost替换为%号，identified by之后跟的是登录的密码
