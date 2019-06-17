---
title: mysql 导出database结构及数据的方法
date: 2016-03-25 15:43:14
tags: [mysql]
categories: [技术]
---
基本的使用方法：
```bash
mysqldump -u<user_name> -p<password> -d <database_name> <table_name> <file_name>.sql
```
#### 导出名称为dbname的database的所有表结构（导出database所有的表，只有结构，不包含数据）
```bash
mysqldump -u<user_name> -p<password> -d dbname > db.sql
```

#### 导出名称为dbname的database的名称为tbname的表的结构（导出databse的某张表，只有结构，不包含数据）
```bash
mysqldump -u<user_name> -p<password> -d dbname tbname > db.sql
```

#### 导出名称为dbname的database的所有表结构及数据（导出database所有的表，不仅有结构，还包含数据）
```bash
mysqldump -u<user_name> -p<password> dbname > db.sql
```

#### 导出名称为dbname的databases的名称为tbname的表的结构及数据（导出database的某张表，不仅又结构，还包含数据）
```bash
mysqldump -u<user_name> -p<password> dbname tbname > db.sql
```

从中可以看出，参数`-d`的作用在于是否只导出表结构，加`-d`之后只导出结构，不导出数据
<!--more-->

#### 将一个database中某个表中部分字段的数据导入另一个database的某个表的部分字段中
http://blog.163.com/l_tianwen/blog/static/35841670201431310363249/
```sql
INSERT INTO databaseA.table(field1,field2,field3) SELECT field1,field2,field3 FROM databaseB.table
```
