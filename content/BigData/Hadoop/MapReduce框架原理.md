---
aliases: 
title: MapReduce框架原理
date created: 2024-03-24 09:03:00
date modified: 2024-03-30 22:03:95
tags: [code/big-data]
---
## InputFormat 数据输入
- 数据块：Block 是 [[HDFS]] 物理上把数据分成一块一块。数据块是 HDFS 存储数据单位。
- 数据切片：只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。数据切片是MapReduce 程序计算输入数据的单位，一个切片会对应启动一个MapTask。

### 数据切片与MapTask并行度决定机制
1. **一个Job的Map阶段并行度由客户端在提交Job时的切片数决定**
2. 每一个Split切片分配一个MapTask并行实例处理
3. 默认情况下，**切片大小=BlockSize**
4. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

#### 面试重点
1. 切片时不考虑数据集整体，而是逐个对于每一个**文件** *单独*切片。
2. 切片大小可以调整，因为 `切片大小=max(minSize, min(blockSize, maxSize))`。
3. **切片大小**可能会超过*设置的切片大小*，设置了1.1倍（32.1MB切一片，避免产生小文件）

#### 源码解析
![Capture 2024-03-24 at 19.35.52@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/Capture%202024-03-24%20at%2019.35.52%402x.png)

### 不同的实现类
#### TextInputFormat
>默认FileInputFormat实现类

键是存储该行在整个文件中的起始字节偏移量， LongWritable 类型。值是这行的内容，不包括任何行终止符（换行符和回车符），Text 类型。

#### CombineTextInputFormat
>TextinputFormat会让**小文件**也产生一个单独的切片，在大量小文件的情况下，会产生大量的MapTask，导致浪费资源。
>而CombineTextInputFormat可以解决此问题，多个小文件在逻辑上划分到一个切片中。

##### 虚拟存储切片最大值设置
```java
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
```
##### 切片机制
生成切片过程包括：虚拟存储过程和切片过程二部分。
![Capture 2024-03-24 at 20.16.30.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/Capture%202024-03-24%20at%2020.16.30%402x.png)

###### 虚拟存储过程
将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2 倍，此时将文件均分成 2 个虚拟存储块（防止出现太小切片）。

###### 切片过程
1. 判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独形成一个切片。
2. 如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。

### MapReduce工作原理
![CleanShot 2024-03-25 at 09.24.43.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-25%20at%2009.24.43.png)
![CleanShot 2024-03-25 at 09.47.54.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-25%20at%2009.47.54.png)

#### Shuffle机制
>Map 方法之后，Reduce 方法之前的数据处理过程称之为Shuffle。

1. Map方法执行完成后，context.write(), 直接输出我们的Key-value
2. 环形缓冲区默认大小是100MB，在80%的时候进行反向，来进行溢写（写入磁盘），相当于一个线程在写入内存，而另一个线程在写入磁盘。
3. 在溢写的时候，会按照key进行分区，对80MB（80%）空间内的数据进行Key排序（**快排**），只修改索引。

#### Partition分区
>默认情况下，分区是根据key的hashCode对ReduceTasks的数量进行取模得到的。
>在此情况下，有时可能会出现数据倾斜的问题，所以需要自定义分区

##### 自定义Partition的步骤
###### 1. 自定义类继承Partitioner，重写getPartition() 方法
```java
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions){
		// 控制分区代码逻辑
		… …
		return partition;
	}
}
```
###### 2. 在Job驱动中，设置自定义Partitioner
```java
job.setPartitionerClass(CustomPartitioner.class);
```
###### 3. 自定义Partition后，要根据自定义Partitioner的逻辑设置相应数量的ReduceTask
```java
job.setNumReduceTasks(5);
```

##### 总结
1. 如果ReduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx
2. 如果1<ReduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception
3. 如果ReduceTask的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个ReduceTask，最终也就只会产生一个结果文件 part-r-00000
4. 分区号必须从零开始，逐一累加。

#### 排序
>默认排序是按照字典顺序排序，实现方式是**快排**。
>任何应用程序中的数据均会被排序，不管逻辑上是否需要。

##### 概述
对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使用率达到80%后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行[[归并排序]]。

对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次[[归并排序]]以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次[[归并排序]]。

#### Combiner
>除了Mapper和Reducer之外的一种组件，但**其父类就是Reducer**。
>就是与Reducer相同的作用与函数，只是作用的时间点不同。


## OutputFormat数据输出
>默认输出格式TextOutputFormat

### 自定义OutputFormat
>场景：输出数据到数据库中

#### 自定义OutputFormat步骤
1. 自定义一个类继承FileOutputFormat。
2. 改写RecordWriter，具体改写输出数据的方法write()。
3. 在自己所写的继承FileOutputFormat的类中新建RecordWriter并返回
```java
@Override
public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job) throws IOException,
InterruptedException {
	//创建一个自定义的 RecordWriter返回
	LogRecordWriter logRecordWriter = new LogRecordWriter(job);
	return logRecordWriter;
}
```

