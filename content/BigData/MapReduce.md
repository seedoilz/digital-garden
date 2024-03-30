---
aliases: 
title: MapReduce
date created: 2024-03-12 13:03:00
date modified: 2024-03-30 21:03:55
tags:
  - code/big-data
  - input
---
## MapReduce概述
### 过程
MapReduce 将计算过程分为两个阶段：Map和Reduce
1. Map 阶段并行处理输入数据
2. Reduce 阶段对 Map 结果进行汇总
### 缺点
1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长 DAG（有向无环图）计算

### MapReduce 进程
1. MrAppMaster：负责整个程序的过程调度及状态协调。
2. MapTask：负责 Map 阶段的整个数据处理流程。
3. ReduceTask：负责 Reduce 阶段的整个数据处理流程。

## [[MapReduce框架原理]]
### 总流程
#### MapTask
1. Read 阶段：MapTask 通过 InputFormat 获得的RecordReader，从输入InputSplit 中解析出一个个key/value。
2. Map 阶段：该节点主要是将解析出的 key/value 交给用户编写 map()函数处理，并产生一系列新的 key/value。
3. Collect 收集阶段：在用户编写 map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的 key/value 分区（调用Partitioner），并写入一个环形内存缓冲区中。
4. Spill 阶段：即“溢写”， 当环形缓冲区满后， MapReduce 会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
	1. 溢写阶段详情：
	2. 步骤 1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition 进行排序，然后按照 key 进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序。
	3. 步骤 2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件 output/spillN.out（N 表示当前溢写次数）中。如果用户设置了 Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
	4. 步骤 3：将分区数据的元信息写到内存索引数据结构SpillRecord 中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件 output/spillN.out.index 中。
5. Merge 阶段：当所有数据处理完成后，MapTask 对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。当所有数据处理完后，MapTask 会将所有临时文件合并成一个大文件，并保存到文件output/file.out 中，同时生成相应的索引文件 output/file.out.index。在进行文件合并过程中，MapTask 以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合mapreduce.task.io.sort.factor （默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。让每个 MapTask 最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

#### ReduceTask
1. Copy 阶段：ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
2. Sort 阶段：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一起，Hadoop 采用了基于排序的策略。由于各个MapTask 已经实现对自己的处理结果进行了局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。
3. Reduce 阶段：reduce()函数将计算结果写到HDFS 上。

## Join应用
### Reduce Join
#### 主要方法
1. Map 端的主要工作：为来自不同表或文件的 key/value 对，**打标签以区别不同来源的记录**。然后用连接字段作为key，其余部分和新加的标志作为value，最后进行输出。
2. Reduce 端的主要工作：在Reduce 端**以连接字段作为key 的分组已经完成**，我们只需要在每一个分组当中将那些来源于不同文件的记录（在 Map 阶段已经打标志）分开，最后进行合并就ok 了。
#### 缺点
这种方式中，合并的操作是在Reduce 阶段完成， Reduce 端的处理压力太大， Map节点的运算负载则很低，资源利用率不高，且在Reduce 阶段极易产生数据倾斜。

### Map Join
#### 使用场景
Map Join 适用于*一张表十分小、一张表很大*的场景。
#### 优点
在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能的减少数据倾斜。
#### 具体方法
>采用 *DistributedCache*
1. Mapper 的setup 阶段，将文件读取到缓存集合中。
2. 在Driver 驱动类中加载缓存。
```java
//缓存普通文件到 Task运行节点。
job.addCacheFile(new URI("file:///e:/cache/pd.txt"));
//如果是集群运行,需要设置 HDFS路径
job.addCacheFile(new URI("hdfs://hadoop102:8020/cache/pd.txt"));
```
