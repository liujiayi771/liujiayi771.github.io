---
title: Spark源码分析RDD缓存过程
date: 2018-04-05 13:43:39
tags: [spark]
categories: [Spark源码阅读]
---
Spark提供了一种将RDD持久化的方式(cache、persist)，这种方式适用于需要多次执行action操作的RDD，因为持久化之后的RDD中的内容不需要重新计算，可以直接使用，对于多次执行action的RDD来说，这样做能省下许多重复计算的时间。Task在启动之初读取一个分区的时候，会先判断这个分区是否已经被持久化，如果没有则会再去检查是否存在Checkpoint，还没有找到的话会根据血统重新计算。RDD的缓存是一种特殊的持久化操作，即`RDD.cache()`等同于`RDD.persist(MEMORY_ONLY)`即缓存是一种只将RDD持久化到内存当中的方式。本文基于Spark 2.1版本的源码对RDD的缓存过程进行了分析，文章中涉及到的源码文件主要有以下几个：
- [`spark/core/src/main/scala/org/apache/spark/storage/memory/MemoryStorage.scala`](https://github.com/apache/spark/blob/branch-2.1/core/src/main/scala/org/apache/spark/storage/memory/MemoryStore.scala)
- [`spark/core/src/main/scala/org/apache/spark/memory/StorageMemoryPool.scala`](https://github.com/apache/spark/blob/branch-2.1/core/src/main/scala/org/apache/spark/memory/StorageMemoryPool.scala)
- [`spark/core/src/main/scala/org/apache/spark/util/collection/SizeTrackingVector.scala`](https://github.com/apache/spark/blob/branch-2.1/core/src/main/scala/org/apache/spark/util/collection/SizeTrackingVector.scala)
- [`spark/core/src/main/scala/org/apache/spark/util/collection/SizeTracker.scala`](https://github.com/apache/spark/blob/branch-2.1/core/src/main/scala/org/apache/spark/util/collection/SizeTracker.scala)

## RDD缓存分析
RDD在缓存到内存之前，Partition中的数据一般以迭代器(Iterator)的数据结构来访问，通过Iterator可以获得分区中每一条序列化或者非序列化的Record，这些Record在访问的时候占用的是JVM堆内存中other部分的内存区域，同一个Partition的不同Record的空间并不是连续的。RDD被缓存之后，会由Partition转化为Block，并且存储位置变为了Storage Memory区域，并且此时Block中的Record所占用的内存空间是连续的。我们可以在Spark的源码当中多次看到unroll这个词，字面意思是展开，在Spark当中的意义就是将存储在Partition中的Record由不连续的存储空间转换为连续存储空间的过程。Unroll操作的时候需要在Storage Memory当中通过`reserveUnrollMemoryForThisTask`来申请Unroll操作所需要的内存，使用完毕之后，又通过`releaseUnrollMemoryForThisTask`方法来释放这部分内存。这与1.6.0版本之前固定Unroll内存的方式不同，是动态申请的，因为这部分内存只在Unroll的时候有用，动态申请这块内存能够在不需要Unroll的时候将这块内存区域用于其他的用途上，提升内存资源的利用率。Block有两种存储方式，分别为序列化存储和非序列化存储，这两种存储方式具有其对应的Entry，在MemoryStore类中通过一个`LinkedHashMap`来存储堆内和对外内存中的所有Block对象的实例：
```scala
// Note: all changes to memory allocations, notably putting blocks, evicting blocks, and
// acquiring or releasing unroll memory, must be synchronized on `memoryManager`!

private val entries = new LinkedHashMap[BlockId, MemoryEntry[_]](32, 0.75f, true)
```
通过这段源码的注释我们也可以知道，对这个map的数据结构进行操作的时候需要严格遵循同步的原则，因为一个Executor会对应一个MemoryStore，而一个Executor有多个core的时候会并行执行Task，就会有多个线程共享使用一块Storage Memory，即共享使用这一个LinkedHashMap，修改LinkedHashMap时需要做到同步。
因为不能保证存储空间可以一次容纳 Iterator 中的所有数据，当前的计算任务在 Unroll 时要向 MemoryManager 申请足够的 Unroll 空间来临时占位，空间不足则 Unroll 失败，空间足够时可以继续进行。至于为何要选择LinkedHashMap来存储也是有原因的，因为LinkedHashMap能够很好地支持LRU算法(最近最少使用，常用于页面置换算法)，我们可以看到定义LinkedHashMap的第三个参数`accessOrder=true`，即基于访问顺序，被访问到的元素会被加到LinkedHashMap的最后。基于这个特性，当新Block加入的时候发现内存空间不足的时候，会按照最近最少使用的顺序淘汰LinkedHashMap中的Block。
### 序列化存储
序列化存储使用了一个名为SerializedMemoryEntry的case class：
```scala
private case class SerializedMemoryEntry[T](
    buffer: ChunkedByteBuffer,
    memoryMode: MemoryMode,
    classTag: ClassTag[T]) extends MemoryEntry[T] {
  def size: Long = buffer.size
}
```
这里的主要存储结构为`ChunkedByteBuffer`，实际上这个类是Spark自己实现的用于存储`ByteBuffer`的数据结构，其本质为`Array[ByteBuffer]`，Array的每一个元素被称为一个chunk。对于已经序列化的Partition在转化为Block进行存储时，因为在存储时就已经知道序列化的ByteBuffer的size，其所需要的Unroll空间可以直接累加计算，一次申请。存储所使用的方法为`putBytes`，需要输入Block的ID、占用的内存空间大小、存储模式为堆内内存还是堆外内存以及存放序列化数据的ByteBuffer，其返回的内容指示了是否缓存成功：
```scala
def putBytes[T: ClassTag](
      blockId: BlockId,
      size: Long,
      memoryMode: MemoryMode,
      _bytes: () => ChunkedByteBuffer): Boolean
```
获取序列化缓存的内容可以直接使用`getBytes`方法，输入Block的ID，获取对应的ChunkByteBuffer。
<!--more-->
### 非序列化存储
非序列化存储使用了一个名为DeserializedMemoryEntry的case class：
```scala
private case class DeserializedMemoryEntry[T](
    value: Array[T],
    size: Long,
    classTag: ClassTag[T]) extends MemoryEntry[T] {
  val memoryMode: MemoryMode = MemoryMode.ON_HEAP
}
```
可以看到，非序列化的存储方式为将元素直接按照Array的方式进行存储，获取其中的存储内容时，可以直接返回装有数据的Iterator。非序列化的存储方式在遍历Record并申请内存时较为复杂，因为无法直接估计Block中所有Record的总空间大小，需要在遍历Record过程中一次申请，即每读取一条Record，采样估算其所需的Unroll空间并进行申请，空间不足时可以中断，释放已占用的Unroll空间。如果最终Unroll成功，当前Partition所占用的Unroll空间被转换为正常的缓存RDD的存储空间。采用估算使用的是`SizeTrackingVector`，其实现了`SizeTracker`接口，可以通过采样的方式，在O(1)时间估计出输入的Block的大小，若估计出的大小超出了申请的内存的临界值，便会再申请内存来存放新加入的Record。这种依次申请内存的方式，其占用的内存是通过周期性地采样近似估算而得，即并不是每次新增的数据项都会计算一次占用的内存大小，这种方法降低了时间开销但是有可能误差较大，导致某一时刻的实际内存有可能远远超出预期。但这也是一种权衡方式，即在计算以及申请内存的时间开销和申请内存准确度上的平衡，在大多数情况下，采样获得的内存大小估计还是准确的。
非序列化的Block有两种存储方式，一种是按照Array的方式存储原值，另一种为将原值进行序列化后存储，所调用的方法为`putIteratorAsValues`和`putIteratorAsBytes`：
```scala
/**
 * Attempt to put the given block in memory store as values.
 *
 * It's possible that the iterator is too large to materialize and store in memory. To avoid
 * OOM exceptions, this method will gradually unroll the iterator while periodically checking
 * whether there is enough free memory. If the block is successfully materialized, then the
 * temporary unroll memory used during the materialization is "transferred" to storage memory,
 * so we won't acquire more memory than is actually needed to store the block.
 *
 * @return in case of success, the estimated size of the stored data. In case of failure, return
 *         an iterator containing the values of the block. The returned iterator will be backed
 *         by the combination of the partially-unrolled block and the remaining elements of the
 *         original input iterator. The caller must either fully consume this iterator or call
 *         `close()` on it in order to free the storage memory consumed by the partially-unrolled
 *         block.
 */
private[storage] def putIteratorAsValues[T](
    blockId: BlockId,
    values: Iterator[T],
    classTag: ClassTag[T]): Either[PartiallyUnrolledIterator[T], Long]

private[storage] def putIteratorAsBytes[T](
    blockId: BlockId,
    values: Iterator[T],
    classTag: ClassTag[T],
    memoryMode: MemoryMode): Either[PartiallySerializedBlock[T], Long]
```
## RDD的持久化级别
前面既然说道了RDD缓存时会有序列化存储和非序列化存储两种方式，我们就顺便说一下RDD持久化的存储级别，这里涉及到的Spark源码当中的文件为`spark/core/src/main/scala/org/apache/spark/storage/StorageLevel.scala`，其中描述了Spark持久化存储级别当中5个重要的变量，这5个变量能组合称为RDD的11中存储级别：
```scala
private var _useDisk: Boolean,
private var _useMemory: Boolean,
private var _useOffHeap: Boolean,
private var _deserialized: Boolean,
private var _replication: Int = 1
```
这5个变量的前3个表示的是存储的位置，分别为磁盘、堆内内存和堆外内存。例如`DISK_ONLY`表示只存储在磁盘当中，`MEMORY_AND_DISK`表示可以存储在堆内内存和磁盘上，`OFF_HEAP`则只能存储在堆外内存上。
`_deserialized`表示是否序列化，例如`MEMORY_ONLY_SER`表示序列化存储在堆内内存上，不加SER的存储级别则为不进行序列化。
`_replication`表示备份的数量，默认不会进行冗余备份，`MEMORY_AND_DISK_2`则表示可以存储在堆内内存和磁盘上，并且创建一个副本进行备份，一般冗余备份会备份到其他的节点，保证持久化的数据的容错性。
## Block存储的淘汰和落盘
我们可以从Spark源码的一些注释当中看到，当有新的Block要存储，内存又不够用时，会通过LRU淘汰一部分最近不常使用的Block，释放其内存给新加入的Block进行存储，如果被淘汰的Block的持久化级别当中有DISK，则会将其写入磁盘当中，称为落盘。Block的淘汰也不是仅仅依赖于LRU算法，其具有一定的规则：
> - 被淘汰的旧 Block 要与新 Block 的 MemoryMode 相同，即同属于堆外或堆内内存
> - 新旧 Block 不能属于同一个 RDD，避免循环淘汰
> - 旧 Block 所属 RDD 不能处于被读状态，避免引发一致性问题
> - 遍历 LinkedHashMap 中 Block，按照最近最少使用（LRU）的顺序淘汰，直到满足新 Block 所需的空间。其中 LRU 是 LinkedHashMap 的特性

淘汰Block并释放其内存调用的方法为`evictBlocksToFreeSpace`：
```scala
/**
 * Try to evict blocks to free up a given amount of space to store a particular block.
 * Can fail if either the block is bigger than our memory or it would require replacing
 * another block from the same RDD (which leads to a wasteful cyclic replacement pattern for
 * RDDs that don't fit into memory that we want to avoid).
 *
 * @param blockId the ID of the block we are freeing space for, if any
 * @param space the size of this block
 * @param memoryMode the type of memory to free (on- or off-heap)
 * @return the amount of memory (in bytes) freed by eviction
 */
private[spark] def evictBlocksToFreeSpace(
    blockId: Option[BlockId],
    space: Long,
    memoryMode: MemoryMode): Long
```
我们也可以在这段代码的注释当中看到一些淘汰Block可能会出现fail的情况，包括新存储的Block所占用的空间大于总内存，或者需要替换处于同一个RDD的不同的Block。
## 总结
本文主要从Spark源码的角度分析了Storage Memory部分，RDD的缓存机制，同时简略介绍了RDD持久化的存储级别。Spark对堆内内存的管理是一种逻辑上的"规划式"的管理，因为对象实例占用内存的申请和释放都由JVM完成，Spark只能在申请后和释放前记录这些内存。所谓记录就是Spark不会有具体的释放内存的操作，只是记录内存变化而已(释放内存时会将不需要的对象置为null，让JVM的GC来回收)，从源码当中我们也可以看到对各种内存的释放和申请调用的方法只是在StorageManager当中进行一个数字的记录和改变，例如如下这段代码：
```scala
def transferUnrollToStorage(amount: Long): Unit = {
  // Synchronize so that transfer is atomic
  memoryManager.synchronized {
    releaseUnrollMemoryForThisTask(MemoryMode.ON_HEAP, amount)
    val success = memoryManager.acquireStorageMemory(blockId, amount, MemoryMode.ON_HEAP)
    assert(success, "transferring unroll memory to storage memory failed")
  }
}
// Acquire storage memory if necessary to store this block in memory.
val enoughStorageMemory = {
  if (unrollMemoryUsedByThisBlock <= size) {
    val acquiredExtra =
      memoryManager.acquireStorageMemory(
        blockId, size - unrollMemoryUsedByThisBlock, MemoryMode.ON_HEAP)
    if (acquiredExtra) {
      transferUnrollToStorage(unrollMemoryUsedByThisBlock)
    }
    acquiredExtra
  } else { // unrollMemoryUsedByThisBlock > size
    // If this task attempt already owns more unroll memory than is necessary to store the
    // block, then release the extra memory that will not be used.
    val excessUnrollMemory = unrollMemoryUsedByThisBlock - size
    releaseUnrollMemoryForThisTask(MemoryMode.ON_HEAP, excessUnrollMemory)
    transferUnrollToStorage(size)
    true
  }
}
```
unrollMemory在释放的时候，需要比较申请到的unrollMemory是否大于最后存储Block的entry的size，如果小于size，会有一个申请不足的storageMemory的过程，并将unrollMemroy转换(transfer)为storageMemory，而Storage Memory的申请还是释放最终都归结为StorageMemoryPool中一个Long类型的`_memoryUsed`变量的值的改变而已。例如如下StorageMemoryPool释放内存的代码：
```scala
def releaseMemory(size: Long): Unit = lock.synchronized {
  if (size > _memoryUsed) {
    logWarning(s"Attempted to release $size bytes of storage " +
      s"memory when we only have ${_memoryUsed} bytes")
    _memoryUsed = 0
  } else {
    _memoryUsed -= size
  }
}
```

对于Spark的内存管理，在被Spark标记为释放的对象实例，很有可能在实际上并没有被JVM回收，导致实际可用的内存小于Spark记录的可用内存，所以Spark并不能准确记录实际可用的堆内内存，从而也就无法完全OOM的异常，我们在平时管理Spark内存的时候要做好充分的内存监控以及调优，保证我们的程序能够在有限的内存环境下正常快速的运行。有时候为了调优可能还需要深刻了解JVM GC的机制，才能做到最大程度的内存充分理由和不出现OOM的异常。

### 参考资料
- https://www.ibm.com/developerworks/cn/analytics/library/ba-cn-apache-spark-memory-management/index.html
