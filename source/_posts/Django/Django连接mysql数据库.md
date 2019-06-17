---
title: Django连接mysql数据库
date: 2016-10-17 10:04:42
tags: [python]
categories: [技术]
---
### 使用pymysql替代MySQLdb
Django与mysql进行连接使用的是MySQLdb模块，但是该模块并不支持python3，需要使用别的库来替代MySQLdb，这里采用的是pymysql，可以使用pip来安装

```bash
pip3 install pymysql
```
之后在Django的站点文件夹中的__init__.py文件夹中添加如下代码即可

```python
import pymysql
pymysql.install_as_MySQLdb()
```
<!--more-->
### 修改setting.py文件
在setting.py文件中修改DATABASES中的内容如下

```python
DATABASES = {
	'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': '<database_name>',
        'USER': '<user_name>',
        'PASSWORD': '<password>',
        'HOST': '<ip_address>',
        'PORT': '<port_num>',
    }
}
```

### 添加测试表
使用startapp users命令来新建一个名称为users的app，并在users文件夹中的models.py文件中添加如下内容进行测试

```python
from django.db import models


class User(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=32)
    nickname = models.CharField(max_length=200)

```
通过以上代码，会在数据库中创建一个叫users_user的表，表中有三个字段为username、password和nickname，字段的类型、特性可以在新建字段时进行设置

### manage.py运行命令
首先运行如下命令

```bash
python3 manage.py makemigrations users
```
通过运行makemigrations，可以告诉Django框架在users中model有所改变，使得这些改变会被作为migration进行存储。Migration是Django用于存储model的变化，只是在硬盘上存储的文件，可以从硬盘当中读取migration文件进行查看。例如上例中产生的migration文件存储在`users/migrations/0001_initial.py`当中，可以通过如下命令查看其中的SQL语句内容

```bash
python3 manage.py sqlmigrate users 0001
```
要想在数据库中真正创建上例中的表，需要运行如下命令

```bash
python3 manage.py migrate
```

### 总结
三个步骤来改变数据库结构

* 在model.py中改变数据库结构
* 运行命令`python3 manage.py makemigrations`来对改变创建migration文件
* 运行命令`python3 manage.py migrate`来应用这些改变到数据库中
