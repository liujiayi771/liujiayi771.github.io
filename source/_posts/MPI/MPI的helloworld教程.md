---
title: MPI的helloworld教程
date: 2016-10-21 15:56:47
tags: [MPI]
categories: [分布式]
---

```C
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialize the MPI environment
    MPI_Init(NULL, NULL);

    // Get the number of processes
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Get the rank of the process
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Get the name of the processor
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    // Print off a hello world message
    printf("Hello world from processor %s, rank %d"
           " out of %d processors\n",
           processor_name, world_rank, world_size);

    // Finalize the MPI environment.
    MPI_Finalize();
}
```
<!--more-->
MPI程序首先需要includeMPI的头文件`#include <mpi.h>`，之后，MPI环境需要使用如下语句进行初始化

```C
MPI_Init(
	int* argc,
	char*** argv)
```
`MPI_Init`之后，所有的MPI需要的全局和内部变量都会被创建完成，例如会产生一个通信域communicator负责所有产生的进程之间的通信，并且每一个MPI进程都会有一个唯一的rank值产生。
`MPI_Init`之后，一般会调用两个函数

```C
MPI_Comm_size(
	MPI_Comm communicator,
	int* size)
```
`MPI_Comm_size`会返回communicator的大小，`MPI_COMM_WORLD`（由MPI产生）封装了所有MPI job中的进程，因此这个函数的调用会返回MPI job中所需要的进程的数量。

```C
MPI_Comm_rank(
	MPI_Comm communicator,
	int* rank)
```
`MPI_Comm_rank`返回在一个communicator中某一个节点的rank值。在communicator中的每一个进程都会被赋予一个从0开始递增的rank值。所有进程的rank值主要作用为在发送和接受信息时起到标识的作用。
在这个例子中用到了一个使用很少的函数

```C
MPI_Get_processor_name(
	char* name,
	int* name_length)
```
`MPI_Get_processor_name`获得正在执行的进程的真实名字，这个程序最后会执行

```C
MPI_Finalize()
```
`MPI_Finalize`的作用是情理MPI环境，这个函数调用之后，MPI的其他操作将不能再继续调用
