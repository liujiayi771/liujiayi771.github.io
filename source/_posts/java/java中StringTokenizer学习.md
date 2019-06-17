---
title: java中StringTokenizer学习
date: 2016-04-13 10:35:53
tags: [Java]
categories: [技术]
---
在平时编程中经常会遇到对字符串进行拆分的情况，例如有些字符串代表了许多组数据，这些数据以‘,’进行分隔，需要将这些数据单独提取出来，以前总是自己去写拆分函数，十分不方便。最近发现java中有一个类叫`StringTokenizer`可以很轻松地解决这个问题。

先来看看官方API文档对`StringTokenizer`的解释：

```console
The string tokenizer class allows an application to break a string into tokens.
```
<!--more-->
也就是说这个类能够使我们将一个字符串拆分成小部分。官方给出了一个使用该类的例子：

```java
StringTokenizer st = new StringTokenizer("this is a test");
     while (st.hasMoreTokens()) {
         System.out.println(st.nextToken());
     }
```
该段函数输出的内容为：

```console
this
is
a
test
```
StringTokenizer将英文句子以空格为分隔拆分成了单个单词。以上的这种方法在之前可以用字符串数组的`split`方法进行实现，如下所示：

```java
String[] result = "this is a test".split("\\s");
     for (int x=0; x<result.length; x++)
         System.out.println(result[x]);
```
这两种方法输出的结果是一致的，但是`StringTokenizer`的方法和功能远不止这一种，还有许多新的更方便的功能。
`StringTokenizer`一共有3种构造函数：

1. `StringTokenizer(String str)`
Constructs a string tokenizer for the specified string.
2. `StringTokenizer(String str, String delim)`
Constructs a string tokenizer for the specified string.
3. `StringTokenizer(String str, String delim, boolean returnDelims)`
Constructs a string tokenizer for the specified string.
这三种构造函数的功能显而易见，第一种使用了默认的分隔符对字符串进行分割，默认的分隔符有` \t\n\r\f`，分别为空格（space character）、Tab制表符（tab character）、换行符（newline character）、回车符（carriage-return character）、换页符（form-feed character）。第二种和第三种指定了分隔符，并且第三种返回的Token中可以选择是否包含分隔符，若选择包含分隔符，分割符将会被当做一个长度为1的字符串单独作为一个Token。

`StringTokenizer`有6种方法：
1. `public boolean hasMoreTokens()`
测试是否有更多的Token
2. `public String nextToken()`
返回下一个字符串Token
3. `public String nextToken(String delim)`
按照指定的分隔符返回下一个Token
4. `public boolean hasMoreElements`
功能与第一种方法完全相同，只是为了实现Enumeration的接口
5. `public Object nextElement()`
功能与第二种方法完全相同，只是为了实现Enumeration的接口
6. `public int countTokens()`
返回拆分的Token数量
