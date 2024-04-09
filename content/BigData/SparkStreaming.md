---
aliases:
  - Spark Streaming
title: SparkStreaming
date created: 2024-04-09 11:04:00
date modified: 2024-04-09 20:04:78
tags: [code/big-data]
---
## 概念
>SparkStreaming是**准实时**，**微批次**的数据处理框架。

和 Spark 基于 RDD 的概念很相似，Spark Streaming 使用离散化流(discretized stream)作为抽象表示，叫作DStream。DStream 是随时间推移而收到的数据的序列。在内部，每个时间区间收到的数据都作为 RDD 存在，而DStream 是由这些RDD 所组成的序列(因此得名“离散化”)。
### 整体架构
![CleanShot 2024-04-09 at 12.45.47.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-04-09%20at%2012.45.47.png)

### 背压机制
为了协调数据接收速率与资源处理能力，SparkStreaming可以动态控制数据接收速率来适配集群数据处理能力。即背压机制（即 Spark Streaming Backpressure）: 根据JobScheduler 反馈作业的执行信息来动态调整Receiver 数据接收率。
>通过属性“spark.streaming.backpressure.enabled”来控制是否启用 backpressure 机制，默认值false，即不启用。

## DStream
### RDD队列
#### 用法及说明
测试过程中，可以通过使用 ssc.queueStream(queueOfRDDs)来创建DStream，每一个推送到这个队列中的RDD，都会作为一个DStream 处理。
#### 实操
```scala
val rddQueue = new mutable.Queue[RDD[Int]]()
//4.创建 QueueInputDStream
val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)
//5.处理队列中的 RDD数据
val mappedStream = inputStream.map((_,1))
val reducedStream = mappedStream.reduceByKey(_ + _)
//6.打印结果
reducedStream.print()
```

### 自定义数据源
#### 用法及说明
需要继承Receiver，并实现 onStart、onStop 方法来自定义数据源采集。
#### 实操
```scala
class CustomerReceiver(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {
	//最初启动的时候，调用该方法，作用为：读数据并将数据发送给 Spark
 	override def onStart(): Unit = {
 		new Thread("Socket Receiver") {
 			override def run() {
 				receive()
 			}
 		}.start()
 	}
	 //读数据并将数据发送给 Spark
	 def receive(): Unit = {
		 //创建一个 Socket
		 var socket: Socket = new Socket(host, port)
		 //定义一个变量，用来接收端口传过来的数据
		 var input: String = null
		 //创建一个 BufferedReader用于读取端口传来的数据
		 val reader = new BufferedReader(new InputStreamReader(socket.getInputStream, StandardCharsets.UTF_8))
	 	//读取数据
	 	input = reader.readLine()
	 	//当 receiver没有关闭并且输入数据不为空，则循环发送数据给 Spark
		while (!isStopped() && input != null) {
 			store(input)
 			input = reader.readLine()
 		}
 		//跳出循环则关闭资源
 		reader.close()
 		socket.close()
 		//重启任务
 		restart("restart")
 	}
 	override def onStop(): Unit = {}
 }
```

### [[Kafka|Kafka数据源]]
>DirectAPI：是由计算的Executor 来主动消费Kafka 的数据，速度由自身控制。
>ReceiverAPI已经不适用了。

```scala
val kafkaPara: Map[String, Object] = Map[String, Object](
	ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG ->
	"hadoop1:9092,hadoop2:9092,hadoop3:9092",
	ConsumerConfig.GROUP_ID_CONFIG -> "atguigu",
	"key.deserializer" ->
	"org.apache.kafka.common.serialization.StringDeserializer",
	"value.deserializer" ->
	"org.apache.kafka.common.serialization.StringDeserializer"
	)
//4.读取 Kafka数据创建 DStream
val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
KafkaUtils.createDirectStream[String, String](ssc,
	LocationStrategies.PreferConsistent,
	ConsumerStrategies.Subscribe[String, String](Set("atguigu"), kafkaPara))
```

## DStream转换
DStream 上的操作与 RDD 的类似，分为 Transformations（转换）和 Output Operations（输出）两种。
### 无状态转化操作
无状态转化操作就是把简单的RDD 转化操作应用到每个批次上，也就是转化DStream 中的每一个RDD
![CleanShot 2024-04-09 at 15.26.15.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-04-09%20at%2015.26.15.png)
#### transform
>可以将底层RDD获取到后进行操作
##### 使用场景
1. DStream功能不完善  
2. 需要代码周期性的执行
```scala
// 按批次更新
val newDS: DStream[String] = lines.transform(  
    rdd => {  
        // Code : Driver端，（周期性执行）  
        rdd.map(  
            str => {  
                // Code : Executor端  
                str  
            }  
        )  
    }  
)
```
#### join
>所谓的DStream的Join操作，其实就是两个RDD的join
```scala
val data9999 = ssc.socketTextStream("localhost", 9999)  
val data8888 = ssc.socketTextStream("localhost", 8888)  

val map9999: DStream[(String, Int)] = data9999.map((_,9))  
val map8888: DStream[(String, Int)] = data8888.map((_,8))  
// 所谓的DStream的Join操作，其实就是两个RDD的join  
val joinDS: DStream[(String, (Int, Int))] = map9999.join(map8888)
```
### 有状态转化操作
#### UpdateStateByKey
>UpdateStateByKey 原语用于记录历史记录，有时，我们需要在 DStream 中跨批次维护状态(例如流计算中累加wordcount)。

##### 使用
1. 定义状态
2. 定义状态更新函数，用此函数阐明如何使用之前的状态和来自输入流的新值对状态进行更新。
使用updateStateByKey 需要对检查点目录进行配置，会使用检查点来保存状态。

#### WindowOperations
Window Operations 可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming 的允许状态。所有基于窗口的操作都需要两个参数，分别为窗口时长以及滑动步长。
- 窗口时长：计算内容的时间范围；
- 滑动步长：隔多久触发一次计算。
注意：这两者都必须为采集周期大小的整数倍。
```scala
val windowDS: DStream[(String, Int)] = wordToOne.window(Seconds(6), Seconds(6))
val wordToCount = windowDS.reduceByKey(_+_)
```
##### 其他衍生方法
1. window(windowLength, slideInterval): 基于对源DStream 窗化的批次进行计算返回一个新的Dstream；
2. countByWindow(windowLength, slideInterval): 返回一个滑动窗口计数流中的元素个数；
3. reduceByWindow(func, windowLength, slideInterval): 通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流；
4. reduceByKeyAndWindow(func, windowLength, slideInterval, \[numTasks\]): 当在一个(K,V)对的DStream 上调用此函数，会返回一个新(K,V)对的 DStream，此处通过对滑动窗口中批次数据使用reduce 函数来整合每个 key 的 value 值。
5. reduceByKeyAndWindow(func, invFunc, windowLength, slideInterval, \[numTasks\]): 这个函数是上述函数的变化版本，每个窗口的reduce 值都是通过用前一个窗的 reduce 值来递增计算。

## DStream输出
输出操作指定了对流数据经转化操作得到的数据所要执行的操作(例如把结果推入外部数据库或输出到屏幕上)。与RDD 中的惰性求值类似，如果一个DStream 及其派生出的DStream 都没有被执行输出操作，那么这些DStream 就都不会被求值。*如果 StreamingContext 中没有设定输出操作，整个context 就都不会启动。*

### 输出操作
- print()：在运行流程序的驱动结点上打印DStream 中每一批次数据的最开始 10 个元素。这用于开发和调试。在 Python API 中，同样的操作叫 print()。
- saveAsTextFiles(prefix, \[suffix\])：以 text 文件形式存储这个 DStream 的内容。每一批次的存储文件名基于参数中的 prefix 和suffix。”prefix-Time_IN_MS\[.suffix\]”。
- saveAsObjectFiles(prefix, \[suffix\])：以Java 对象序列化的方式将 Stream 中的数据保存为SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS\[.suffix\]". Python中目前不可用。
- saveAsHadoopFiles(prefix, \[suffix\])：将 Stream 中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS\[.suffix\]"。Python API 中目前不可用
- foreachRDD(func)：这是最通用的输出操作，即将函数 func 用于产生于 stream 的每一个RDD。其中参数传入的函数 func 应该实现将每一个RDD 中数据推送到外部系统，如将RDD 存入文件或者通过网络将其写入数据库。

## 优雅关闭
使用外部文件系统来控制内部程序关闭。
```scala
// 如果想要关闭采集器，那么需要创建新的线程  
// 而且需要在第三方程序中增加关闭状态  
new Thread(  
    new Runnable {  
        override def run(): Unit = {  
            // 优雅地关闭  
            // 计算节点不在接收新的数据，而是将现有的数据处理完毕，然后关闭  
            // Mysql : Table(stopSpark) => Row => data  
            // Redis : Data（K-V）  
            // ZK    : /stopSpark  
            // HDFS  : /stopSpark            while ( true ) {  
                if (true) {  
                    // 获取SparkStreaming状态  
                    val state: StreamingContextState = ssc.getState()  
                    if ( state == StreamingContextState.ACTIVE ) {  
                        ssc.stop(true, true)  
                    }  
                }  
                Thread.sleep(5000)  
            }  
        }  
    }  
).start()
```