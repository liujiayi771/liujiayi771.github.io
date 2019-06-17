---
title: 在pycharm上运行pyspark程序
date: 2016-06-28 13:26:28
tags: [python,spark]
categories: [分布式]
---
### 配置pycharm
1. 在pycharm上创建project之后，在Run->Edit Configurations->Environment variables中添加两个环境变量：
`PYTHONPATH = /opt/spark/spark1.6.2/python`
`SPARK_HOME = /opt/spark/spark1.6.2`
<!--more-->
2. 将`spark/python`目录中的pyspark目录拷贝到工程文件夹中

### 编写pyspark程序
1. pyspark程序中经常会用到一个变量sqlContext，这个变量一般是通过`sqlContext = SQLContext(sc)`生成的，变量`sc`是一个pyspark的shenll自动生成的helper变量
2. 在pyspark的shell中，这个变量是会自动生成作为helper变量的，但是在pycharm编写的程序中需要自己去定义这个变量：
```python
from pyspark import SparkConf, SparkContext
from pyspark.sql import SQLContext
conf = SparkConf().setAppName("berkeleySpark")
sc = SparkContext(conf=conf)
sqlContext = SQLContext(sc)
```
