---
layout: post
title: Spark机制
categories: Engineer
description:
keywords: BigData
---



### Spark存储机制

#### 堆内存储

spark的内核由scala语言完成，其运行在JVM上，所以spark在运行时具有堆（heap）的概念。进一步，其将堆分为四部分

| **Storage Memory**   | 存储cached数据              |
| -------------------- | --------------------------- |
| **Execution Memory** | 用于存储shuffle操作时的数据 |
| **User Memory**      | 存储用户代码中的数据结构    |
| **Reserved Memory**  | 用于spark自身               |

![image-20230707113552275](http://pic.inoodles.online/imgimage-20230707113552275.png)

- 在spark 1.6以前，spark为[StaticMemoryManager](https://github.com/apache/spark/blob/branch-1.6/core/src/main/scala/org/apache/spark/memory/StaticMemoryManager.scala)，在此之后为 [UnifiedMemoryManager](https://github.com/apache/spark/blob/branch-1.6/core/src/main/scala/org/apache/spark/memory/UnifiedMemoryManager.scala)。在静态存储管理时，spark.storage.memoryFraction和spark.shuffle.memoryFraction分别用于控制storage memory和shuffle memory的比例，在此之后此参数失效（除非配置spark.memory.useLegacyMode）
- 在统一存储管理中，使用spark.memory.fraction控制storage和shuffle存储的总比例

#### 堆外存储

SPARK的堆外内存可分为两种

- 一种是用于JVM本身、字符串等开销
- 另一种是用于spark程序本身，存放数据等

![image-20230707114948882](http://pic.inoodles.online/imgimage-20230707114948882.png)

与之相关的主要有两组参数，一组是spark.yarn.executor.memoryOverhead，另一组是spark.memory.offHeap.enabled和spark.memory.offHeap.size，对于spark1.x和2.x，spark.yarn.executor.memoryOverhead控制整个堆外存储的大小，第二种堆外存储通过spark.memory.offHeap.enabled和spark.memory.offHeap.size显式配置；对于spark3.x，这两组参数独立控制两种堆外存储大小。此外，第一种堆外存储默认开启，大小为excutor的10%，且最小为384MB。



### SPARK

- executor-cores 表示一个executor的核数，也是一个executor最大并行的task数

- spark.dynamicAllocation.maxExecutors 最大使用饿的executor数目

  




## spark调优

### 代码优化
#### 避免创建重复RDD

#### 尽可能复用同一个rdd

#### 对多次使用的rdd进行持久化 

sc.textFile("hdfs://192.168.0.1:9000/hello.txt").cache()

sc.textFile("hdfs://192.168.0.1:9000/hello.txt").persist(StorageLevel.MEMORY_AND_DISK_SER)

#### 尽量避免使用shuffle类算子
例如reduceByKey、join、reparation等算子

#### 使用map-side预聚合的shuffle操作
如果一定要使用shuffle操作，尽量可以使用map-side预聚合的算子（所谓map-side，是指每个节点本地对相同的key进行一次聚合操作，预聚合后，每个节点本地就只会有一条相同的key）
建议使用reduceByKey或aggregateByKey替代groupByKey


### 配置优化
#### num-executors
该参数用于设置Spark作业总共要用多少个Executor进程来执行。如果不设置的话，默认只会给你启动少量的Executor进程，此时你的Spark作业的运行速度是非常慢

每个Spark作业的运行一般设置50~100个左右的Executor进程比较合适，设置太少或太多的Executor进程都不好。设置的太少，无法充分利用集群资源；设置的太多的话，大部分队列可能无法给予充分的资源。


#### executor-memory
该参数用于设置每个Executor进程的内存。Executor内存的大小，很多时候直接决定了Spark作业的性能，而且跟常见的JVM OOM异常，也有直接的关联。

每个Executor进程的内存设置4G~8G较为合适。但是这只是一个参考值，具体的设置还是得根据不同部门的资源队列来定。可以看看自己团队的资源队列的最大内存限制是多少，num-executors乘以executor-memory，是不能超过队列的最大内存量的。此外，如果你是跟团队里其他人共享这个资源队列，那么申请的内存量最好不要超过资源队列最大总内存的1/3~1/2，避免你自己的Spark作业占用了队列所有的资源，导致别的同学的作业无法运行。

#### executor-cores
该参数用于设置每个Executor进程的CPU core数量。这个参数决定了每个Executor进程并行执行task线程的能力。因为每个CPU core同一时间只能执行一个task线程，因此每个Executor进程的CPU core数量越多，越能够快速地执行完分配给自己的所有task线程。

Executor的CPU core数量设置为2~4个较为合适。同样得根据不同部门的资源队列来定，可以看看自己的资源队列的最大CPU core限制是多少，再依据设置的Executor数量，来决定每个Executor进程可以分配到几个CPU core。同样建议，如果是跟他人共享这个队列，那么num-executors * executor-cores不要超过队列总CPU core的1/3~1/2左右比较合适，也是避免影响其他同学的作业运行。

#### driver-memory
该参数用于设置Driver进程的内存。

Driver的内存通常来说不设置，或者设置1G左右应该就够了。唯一需要注意的一点是，如果需要使用collect算子将RDD的数据全部拉取到Driver上进行处理，那么必须确保Driver的内存足够大，否则会出现OOM内存溢出的问题。

#### spark.default.parallelism
该参数用于设置每个stage的默认task数量。这个参数极为重要，如果不设置可能会直接影响你的Spark作业性能。

Spark作业的默认task数量为500~1000个较为合适。Spark官网建议的设置原则是，设置该参数为num-executors * executor-cores的2~3倍较为合适。

#### spark.storage.memoryFraction
该参数用于设置RDD持久化数据在Executor内存中能占的比例，默认是0.6

如果Spark作业中，有较多的RDD持久化操作，该参数的值可以适当提高一些，保证持久化的数据能够容纳在内存中。避免内存不够缓存所有的数据，导致数据只能写入磁盘中，降低了性能。但是如果Spark作业中的shuffle类操作比较多，而持久化操作比较少，那么这个参数的值适当降低一些比较合适。此外，如果发现作业由于频繁的gc导致运行缓慢（通过spark web ui可以观察到作业的gc耗时），意味着task执行用户代码的内存不够用，那么同样建议调低这个参数的值。

#### spark.shuffle.memoryFraction
该参数用于设置shuffle过程中一个task拉取到上个stage的task的输出后，进行聚合操作时能够使用的Executor内存的比例，默认是0.2。也就是说，Executor默认只有20%的内存用来进行该操作。shuffle操作在进行聚合时，如果发现使用的内存超出了这个20%的限制，那么多余的数据就会溢写到磁盘文件中去，此时就会极大地降低性能。


#### 示例
./bin/spark-submit \
  --master yarn-cluster \
  --num-executors 100 \
  --executor-memory 6G \
  --executor-cores 4 \
  --driver-memory 1G \
  --conf spark.default.parallelism=1000 \
  --conf spark.storage.memoryFraction=0.5 \
  --conf spark.shuffle.memoryFraction=0.3 \

