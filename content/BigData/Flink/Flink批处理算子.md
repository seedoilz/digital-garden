---
aliases: []
title: Flink批处理算子
date created: 2024-04-23 15:04:00
date modified: 2024-06-28 10:06:69
tags: [code/big-data]
---
### Source算子
#### fromCollection
> 从本地集合读取数据
```scala
val env = ExecutionEnvironment.getExecutionEnvironment
val textDataSet: DataSet[String] = env.fromCollection(
	List("1,张三", "2,李四", "3,王五", "4,赵六") 
)
```

#### readTextFile
##### 从文件中读取
```scala
val textDataSet: DataSet[String] = env.readTextFile("/data/a.txt")
```
##### 遍历目录
>readTextFile 可以对一个文件目录内的所有文件，包括所有子目录中的所有文件的遍历访问方式
```scala
val parameters = new Configuration  
// recursive.file.enumeration 开启递归
parameters.setBoolean("recursive.file.enumeration", true)  
val file = env.readTextFile("/data").withParameters(parameters)
```
##### 读取压缩文件
> 对于以下压缩类型，不需要指定任何额外的 inputformat 方法，flink 可以自动识别并且解压。但是，压缩文件可能**不会并行读取**，可能是顺序读取的，这样可能*会影响作业的可伸缩性*。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-23-14-17-08.png)
```scala
val file = env.readTextFile("/data/file.gz")
```

### Transform算子
>类sql的就不过多介绍了
#### rebalance
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-23-14-37-14.png)
```scala
// 使用 rebalance 操作，避免数据倾斜  
val rebalance = filterData.rebalance()
```
#### partitionByHash
>将数据流 `collection` 按照其中的元素的第二个字段（下标为1）进行**哈希分区**
```scala
val collection = env.fromCollection(data)
val unique = collection.partitionByHash(1).mapPartition{
	line =>  
		line.map(x => (x._1 , x._2 , x._3))
}
```
#### partitionByRange
>将数据流 `collection` 按照其中的元素的第二个字段（下标为1）进行范围分区
```scala
val collection = env.fromCollection(data)  
val unique = collection.partitionByRange(x => x._1).mapPartition(line => line.map{
  x=>
    (x._1 , x._2 , x._3)
})
```
#### sortPartition
>根据指定的字段值进行分区的排序
```scala
val ds = env.fromCollection(data)
    val result = ds
		.map { x => x }.setParallelism(2)  
		.sortPartition(1, Order.DESCENDING)//第一个参数代表按照哪个字段进行分区
		.mapPartition(line => line)  
		.collect()
```
### Sink算子
#### collect
>将数据输出到本地集合
```scala
result.collect()
```
#### writeAsText
>将数据输出到文件
```scala
// 将数据写入本地文件  
result.writeAsText("/data/a", WriteMode.OVERWRITE)
// 将数据写入 HDFS  
result.writeAsText("hdfs://node01:9000/data/a", WriteMode.OVERWRITE)
```