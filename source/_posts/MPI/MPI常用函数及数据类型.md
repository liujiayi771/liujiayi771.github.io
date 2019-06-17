---
title: MPI常用函数及数据类型
date: 2016-10-21 17:21:18
tags: [MPI]
categories: [分布式]
---
其他MPI中会用到的函数数据类型：

* `MPI_Barrier` - `int MPI_Barrier( MPI_Comm comm )`阻塞执行到当前位置的处理程序，直到communicator中所有处理程序都到达该位置
* `MPI_Wtime` - `double MPI_Wtime( void )`返回调用这个函数的进程节点从创建开始经过的时间
* `MPI_Type_size` - `int MPI_Type_size(MPI_Datatype datatype, int *size)`返回datatype所占有的字节数
* `MPI_Send` - `int MPI_Send(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm)`MPI发送数据函数，buf为发送缓冲区的起始地址，count为将发送的数据的个数，datatype为发送数据的数据类型，dest为目的进程的标识号，tag为消息标志，comm为通信域，该函数的返回值为MPI_SUCCESS时表示发送成功
* `MPI_Irecv` = `int MPI_Irecv(void *buf, int count, MPI_Datatype datatype, int source, int tag, MPI_Comm comm, MPI_Status *status)`MPI接受数据函数，buf为接受缓冲区的起始地址，count为最多可接受的数据个数，datatype为接收数据的数据类型，source为接受数据的来源即发送数据的进程的进程标识号，tag为消息标识，与相应的发送操作的表示相匹配，comm为本进程和发送进程所在的通信域，status为返回状态
<!--more-->
* `MPI_Bcast` - `int MPI_Bcast( void *buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm )`从进程rank号为root的进程向通信域中的其他进程发送一条广播消息
* `MPI_Datatype` - MPI数据类型枚举

```C++
typedef enum _MPI_Datatype {
  MPI_DATATYPE_NULL          = 0x0c000000,
  MPI_CHAR                   = 0x4c000101,
  MPI_UNSIGNED_CHAR          = 0x4c000102,
  MPI_SHORT                  = 0x4c000203,
  MPI_UNSIGNED_SHORT         = 0x4c000204,
  MPI_INT                    = 0x4c000405,
  MPI_UNSIGNED               = 0x4c000406,
  MPI_LONG                   = 0x4c000407,
  MPI_UNSIGNED_LONG          = 0x4c000408,
  MPI_LONG_LONG_INT          = 0x4c000809,
  MPI_LONG_LONG              = MPI_LONG_LONG_INT,
  MPI_FLOAT                  = 0x4c00040a,
  MPI_DOUBLE                 = 0x4c00080b,
  MPI_LONG_DOUBLE            = 0x4c00080c,
  MPI_BYTE                   = 0x4c00010d,
  MPI_WCHAR                  = 0x4c00020e,
  MPI_PACKED                 = 0x4c00010f,
  MPI_LB                     = 0x4c000010,
  MPI_UB                     = 0x4c000011,
  MPI_C_COMPLEX              = 0x4c000812,
  MPI_C_FLOAT_COMPLEX        = 0x4c000813,
  MPI_C_DOUBLE_COMPLEX       = 0x4c001614,
  MPI_C_LONG_DOUBLE_COMPLEX  = 0x4c001615,
  MPI_2INT                   = 0x4c000816,
  MPI_C_BOOL                 = 0x4c000117,
  MPI_SIGNED_CHAR            = 0x4c000118,
  MPI_UNSIGNED_LONG_LONG     = 0x4c000819,
  MPI_CHARACTER              = 0x4c00011a,
  MPI_INTEGER                = 0x4c00041b,
  MPI_REAL                   = 0x4c00041c,
  MPI_LOGICAL                = 0x4c00041d,
  MPI_COMPLEX                = 0x4c00081e,
  MPI_DOUBLE_PRECISION       = 0x4c00081f,
  MPI_2INTEGER               = 0x4c000820,
  MPI_2REAL                  = 0x4c000821,
  MPI_DOUBLE_COMPLEX         = 0x4c001022,
  MPI_2DOUBLE_PRECISION      = 0x4c001023,
  MPI_2COMPLEX               = 0x4c001024,
  MPI_2DOUBLE_COMPLEX        = 0x4c002025,
  MPI_REAL2                  = MPI_DATATYPE_NULL,
  MPI_REAL4                  = 0x4c000427,
  MPI_COMPLEX8               = 0x4c000828,
  MPI_REAL8                  = 0x4c000829,
  MPI_COMPLEX16              = 0x4c00102a,
  MPI_REAL16                 = MPI_DATATYPE_NULL,
  MPI_COMPLEX32              = MPI_DATATYPE_NULL,
  MPI_INTEGER1               = 0x4c00012d,
  MPI_COMPLEX4               = MPI_DATATYPE_NULL,
  MPI_INTEGER2               = 0x4c00022f,
  MPI_INTEGER4               = 0x4c000430,
  MPI_INTEGER8               = 0x4c000831,
  MPI_INTEGER16              = MPI_DATATYPE_NULL,
  MPI_INT8_T                 = 0x4c000133,
  MPI_INT16_T                = 0x4c000234,
  MPI_INT32_T                = 0x4c000435,
  MPI_INT64_T                = 0x4c000836,
  MPI_UINT8_T                = 0x4c000137,
  MPI_UINT16_T               = 0x4c000238,
  MPI_UINT32_T               = 0x4c000439,
  MPI_UINT64_T               = 0x4c00083a,
  MPI_AINT                   = 0x4c00083b (_WIN64), 0x4c00043b,
  MPI_OFFSET                 = 0x4c00083c,
  MPI_FLOAT_INT              = 0x8c000000,
  MPI_DOUBLE_INT             = 0x8c000001,
  MPI_LONG_INT               = 0x8c000002,
  MPI_SHORT_INT              = 0x8c000003,
  MPI_LONG_DOUBLE_INT        = 0x8c000004
} MPI_Datatype;
```
