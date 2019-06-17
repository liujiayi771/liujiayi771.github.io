---
title: Spark对大规模数据进行分区
date: 2018-03-17 15:46:14
tags: [spark]
categories: [分布式]
---

# groupByKey

在编写处理大规模数据的Spark代码的时候遇到一个问题，对大规模数据进行groupByKey操作的时候时间非常长，而且很容易出现OOM的情况。这其中的主要原因有几点，一是groupByKey会造成大量的数据shuffle，大量的IO会影响程序的运行时间；二是每一个key下对应的数据非常不均匀，有的key对应的key非常多，在某个executor上的数据可能会超出内存大小，造成OOM的情况。

groupByKey这个API事实上不是一个非常高效的API，会造成大量的数据搬移，效率不高。在我的项目当中，我实际上要做的任务是将我的数据按照染色体序号进行group，将染色体号为1的记录归并到一起进行处理，将染色体序号为2的记录归并到一起处理，等等。由于记录的数量非常庞大，并且不同染色体号的记录数量又和染色体的长度相关，是十分不均匀的，例如1号染色体对应的记录数量就远远大于Y染色体，groupByKey会消耗非常多的时间在数据迁移以及数据的序列化反序列化上，同时，每一个group数据量的不均匀性又会导致某些executor上内存压力过大，出现OOM的情况。

# partitionBy

其实对于我的这种情况使用partitionBy会更为适合，可以自己实现一个partitioner，也可以直接使用HashPartitioner，以染色体号作为key进行重新分区，然后再使用mapPartition在每个分区内处理不同染色体，partitionBy的效率会比groupByKey高很多。

# repartitionAndSortWithinPartitions

在我的项目当中，对每个染色体号对应的记录进行的数据处理包括对数据的排序操作，在数据量很大的时候，这个排序操作也会很耗费时间，但是在partitionBy对数据进行shuffle的时候，已经对数据进行过遍历了，之后再次排序需要又一次遍历数据，十分浪费时间。于是，我又找到了一个非常适合我的程序的一个新的分区接口`repartitionAndSortWithinPartitions`，首先看下这个接口的使用方法：

```
def repartitionAndSortWithinPartitions(partitioner: Partitioner): RDD[(K, V)]

Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys.

This is more efficient than calling repartition and then sorting within each partition because it can push the sorting down into the shuffle machinery.
```

为了使用这个接口，我就必须将我自己定义的数据类型定义一个排序规则，即定义一个Ordering，具体的定义方法如下：

```scala
object MySAMRecord {
  implicit val samRecordOrdering: Ordering[MySAMRecord] = new Ordering[MySAMRecord] {
    override def compare(x: MySAMRecord, y: MySAMRecord): Int = {
      if (x.referenceIndex != y.referenceIndex) x.referenceIndex - y.referenceIndex
      else {
        if (x.startPos != y.startPos) {
          x.startPos - y.startPos
        } else {
          if (new String(x.originalStrByteArr) > new String(y.originalStrByteArr)) 1 else -1
        }
      }
    }
  }
}
```

同时，由于`repartitionAndSortWithinPartitions`接口的排序是按照key进行的，我就不能使用原有的HashPartitioner进行分区，需要自己定义Partitioner，使得我自己定义的数据类型作为key值时，仍然能够按照染色体序号进行分区，我自己的Partitioner函数定义如下：

```scala
import org.apache.spark.Partitioner

class MySAMRecordPartitioner(numParts: Int) extends Partitioner {
  override def numPartitions: Int = numParts

  override def getPartition(key: Any): Int = {
    val record: MySAMRecord = key.asInstanceOf[MySAMRecord]

    val code = record.regionId % numPartitions
    if (code < 0) {
      code + numPartitions
    } else {
      code
    }
  }

  override def equals(other: Any): Boolean = other match {
    case records: MySAMRecordPartitioner =>
      records.numPartitions == numPartitions
    case _ =>
      false
  }

  override def hashCode(): Int = numPartitions
}
```

当我的数据输入是`RDD[MySAMRecord]`时，为了使用`repartitionAndSortWithinPartitions`，需要将输入转换成键值对形式，RDD[(MySAMRecord, None)]，repartition之后，每个partition中的数据就会被自动排序完成，从源码注释当中我们也可以看到，这样子操作是比repartition之后再在每个分区中sorting是要快的，因为这个排序是在shffle的同时进行的，对数据的遍历在shffle的时候只进行了一次，效率当然会高很多。之后再使用mapPartition对每个分区进行操作的时候，每个partition所对应的Iterable就已经是有序的了，不需要再进行新的排序操作。

# 总结
1. 在处理的数据比较大的时候，尽量不要使用groupByKey操作，这个操作的效率很低，可以使用reduceByKey(当需要后续操作的时候)来代替，也可以使用partitionBy，repartition等接口进行替代。
2. 在处理的数据分区之后，如果还要进行排序操作的话，可以尝试使用repartitionAndSortWithinPartitions，这个接口能够在shuffle数据的同时进行排序，减少遍历数据的次数，节省程序运行时间。

# 参考资料
1. https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html
