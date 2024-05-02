---
aliases: []
title: Flink流处理算子
date created: 2024-04-23 15:04:00
date modified: 2024-04-27 18:04:53
tags: [code/big-data]
---
### Source算子
Flink 可以使用 StreamExecutionEnvironment.addSource 来为我们的程序添加数据来源。  
Flink 已经提供了若干实现好了的 source functions，当然我们也可以通过实现 SourceFunction 来自定义非并行的 source 或者实现 ParallelSourceFunction 接口或者扩展 RichParallelSourceFunction 来自定义并行的 source。
#### 4类source
- 基于本地集合的 source(Collection-based-source)
- 基于文件的 source(File-based-source)- 读取文本文件，即符合TextInputFormat 规范的文件，并将其作为字符串返回
- 基于网络套接字的 source(Socket-based-source)- 从 socket 读取。元素可以用分隔符切分。
- 自定义的 source(Custom-source)
#### Kafka数据源
```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```
```scala
val properties = new Properties()  
properties.setProperty("bootstrap.servers", "localhost:9092") properties.setProperty("group.id", "consumer-group") properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.S tringDeserializer")  
properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization. StringDeserializer")  
properties.setProperty("auto.offset.reset", "latest")

val source = env.addSource(new FlinkKafkaConsumer011[String]("sensor", new SimpleSt ringSchema(), properties))
```
### Transform转换算子
#### KeyBy
>具有相同 Keys 的所有记录都分配给同一分区。
#### Fold
>具有初始值的被 Keys 化数据流上的“滚动”折叠。将当前数据元与最后折叠的值组合并发出新值
```scala
val result: DataStream[String] = keyedStream.fold("start")((str, i) => { str + "-" + i })
// 当上述代码应用于序列(1,2,3,4,5)，
// 输出结果为“start-1”，“start-1-2”，“start-1-2”，...
```
#### Window
可以在已经分区的 KeyedStream 上定义 Windows。Windows 根据某些特征(例如， 在最后 5 秒内到达的数据)对每个 Keys 中的数据进行分组。
```scala
dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5)));
```
#### WindowAll
>Windows 可以在常规 DataStream 上定义。Windows 根据某些特征(例如，在最后 5 秒内到达的数据)对所有流事件进行分组。
>注意:在许多情况下，这是非并行转换。所有记录将收集在 windowAll 算子的一个任务中。
```scala
dataStream.windowAll(TumblingEventTimeWindows.of(Time.seconds(5)))
```
#### Window Join
>在给定 Keys 和公共窗口上连接两个数据流
#### Connect
>“连接”两个保存其类型的数据流。连接允许两个流之间的共享状态
#### CoMap，CoFlatMap
>类似于连接数据流上的 map 和 flatMap
#### Split
>根据某些标准将流拆分为两个或更多个流
```scala
val split = someDataStream.split(
	(num: Int) =>
		(num % 2) match {
			case 0 => List("even")
			case 1 => List("odd")
		})
```
#### Select
>从拆分流中选择一个或多个流
```scala
SplitStream<Integer> split;
DataStream<Integer> even = split.select("even");
DataStre am<Integer> odd = split.select("odd");
DataStream<Integer> all = split.select("even", "odd")
```
### Sink算子
支持输出到以下内容：
- 本地文件(参考批处理)
- 本地集合(参考批处理)
- HDFS(参考批处理)
- 以及kafka、mysql、redis
```scala
// Kafka示例
val prop = new Properties()
prop.setProperty("bootstrap.servers", "node01:9092")
val myProducer = new FlinkKafkaProducer011[String](sinkTopic, new KeyedSerializ ationSchemaWrapper[String](new SimpleStringSchema()), prop)
studentStream.addSink(myProducer)
studentStream.print()
```