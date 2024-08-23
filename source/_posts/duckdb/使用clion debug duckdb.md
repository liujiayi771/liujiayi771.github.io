---
title: 使用clion debug duckdb
date: 2024-07-18 12:47:48
tags: [DB]
---

duckdb的代码实现非常值得DB领域的从业人员学习，最近在实现parquet v2的编码解析功能，看到duckdb的代码中已经支持了parquet v2中的各种新增的编码格式，因此就想先debug看下duckdb的实现代码，方便自己后续的开发。本文就将如何使用clion debug duckdb代码进行一个简单的记录。

在github clone duckdb最新的代码后，使用clion打开，由于duckdb默认就会加载parquet extension，因此我们不需要额外配置cmake参数就可以load parquet的相关代码。之后我们可以在右上角Run Configuration中找到名称为duckdb的Target，将其Executable选择为shell。在Environment Variables中填入：

```shell
LD_PRELOAD=/usr/lib64/libasan.so.6
```

这种运行方式无法进行交互式输入，因此我们需要直接加入需要执行的query，例如：

```shell
-c
"select * from parquet_scan('test.parquet')
```

这样在代码中打断点后执行就可以进行debug了，可以学习duckdb优秀的代码。