---
aliases:
  - Resilient Distributed Dataset
title: RDD
date created: 2024-04-03 10:04:00
date modified: 2024-05-15 16:05:73
tags: [code/big-data]
---
### 概念
RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

1. 弹性
	1. 存储的弹性：内存与磁盘的自动切换；
	2. 容错的弹性：数据丢失可以自动恢复；
	3. 计算的弹性：计算出错重试机制；
	4. 分片的弹性：可根据需要重新分片。
2. 数据集：RDD 封装了计算逻辑，并不保存数据
3. 数据抽象：RDD 是一个抽象类，需要子类具体实现
4. 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD 里面封装计算逻辑

### RDD编程
#### RDD创建
##### 从集合（内存）中创建 RDD
```scala
val rdd1 = sparkContext.parallelize(
	List(1,2,3,4)
)
val rdd2 = sparkContext.makeRDD(
	List(1,2,3,4)
)
```
##### 从外部存储创建RDD
```scala
val sparkConf =
	new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val fileRDD: RDD[String] = sparkContext.textFile("input")
fileRDD.collect().foreach(println)
sparkContext.stop()
```
#### RDD并行度与分区
默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建 RDD 时指定。
#### RDD转化算子
##### Value类型
###### map函数
>将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。
```scala
val dataRDD2: RDD[String] = dataRDD1.map(
	num => {
		"" + num
	}
)
```
###### mapPartitions函数
>将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。
```scala
val dataRDD1: RDD[Int] = dataRDD.mapPartitions(
	datas => {
		datas.filter(_==2)
	}
)
```

> [!NOTE] map和mapPartitions的区别
> ![截屏2024-04-03 上午11.45.58.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-03-11-46-01.png)
###### flatMap函数
> 将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射

###### glom函数
>将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

###### groupBy函数
>将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为shuffle。极限情况下，数据可能被分在同一个分区中
```scala
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
val dataRDD1 = dataRDD.groupBy(
	_%2
)
```
###### filter函数
>将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。

###### coalesce函数
>根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率。当spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少分区的个数，减小任务调度成本。
```scala
val dataRDD1 = dataRDD.coalesce(2)
```

###### repartition
>该操作内部其实执行的是 coalesce 操作，参数shuffle 的默认值为true。无论是将分区数多的RDD 转换为分区数少的RDD，还是将分区数少的 RDD 转换为分区数多的RDD，repartition操作都可以完成。
```scala
val dataRDD1 = dataRDD.repartition(4)
```

##### 双Value类型
- intersection
- union
- subtract
- zip

##### Key-Value类型
###### partitionBy函数
>将数据按照指定Partitioner 重新进行分区。Spark 默认的分区器是HashPartitioner
```scala
val rdd2: RDD[(Int, String)] =
	rdd.partitionBy(new HashPartitioner(2))
```

###### reduceByKey函数
>可以将数据按照相同的Key 对Value 进行聚合
```scala
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = dataRDD1.reduceByKey(_+_)
val dataRDD3 = dataRDD1.reduceByKey(_+_, 2)
```

###### groupByKey
>将数据源的数据根据 key 对 value 进行分组
```scala
val dataRDD2 = dataRDD1.groupByKey()
val dataRDD3 = dataRDD1.groupByKey(2)
val dataRDD4 = dataRDD1.groupByKey(new HashPartitioner(2))
```

> [!NOTE] reduceByKey 和 groupByKey 的区别？
> ![截屏2024-04-03 下午4.06.48.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-03-16-06-52.png)

###### aggregateByKey函数
>将数据根据不同的规则进行分区内计算和分区间计算
>第一个参数：如何在同一个分区中对同一个键的多个值进行聚合
>第二个参数：如何在不同分区之间聚合已经聚合过的结果

###### foldByKey函数
>当aggregateByKey函数的第一个参数和第二个参数相同时

###### combineByKey函数
>createCombiner：当一个键第一次出现时，它会使用这个函数来创建该键的累加器的初始值。这个函数会对键的第一个值应用，并返回一个新的值（可能是转换过的），这个新的值会作为累加器的初始值。
>mergeValue：对于同一个键，在同一个分区中，当这个键后续出现时，使用这个函数来将该键的新值与累加器的当前值合并。
>mergeCombiners：在不同分区中，当同一个键的累加器需要合并时，使用这个函数来合并这些累加器的值。

###### cogroup
>在类型为(K,V)和(K,W)的RDD 上调用，返回一个(K,(Iterable\<V\>,Iterable\<W\>))类型的RDD
```scala
val dataRDD2 = dataRDD1.groupByKey()
val dataRDD3 = dataRDD1.groupByKey(2)
val dataRDD4 = dataRDD1.groupByKey(new HashPartitioner(2))
```

#### RDD行动算子
##### reduce函数
>聚集RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据

##### collect函数
>在驱动程序中，以数组Array 的形式返回数据集的所有元素

##### count函数
>返回RDD 中元素的个数

##### first函数
>返回RDD 中的第一个元素

##### take函数
>返回一个由RDD 的前 n 个元素组成的数组

##### fold函数
>aggregate的简化操作

##### save相关算子
```scala
def saveAsTextFile(path: String): Unit
def saveAsObjectFile(path: String): Unit
def saveAsSequenceFile(
	path: String,
	codec: Option[Class[_ <: CompressionCodec]] = None): Unit
```

```scala
// 保存成 Text文件
rdd.saveAsTextFile("output")
// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")
// 保存成 Sequencefile文件
rdd.map((_,1)).saveAsSequenceFile("output2")
```

#### RDD序列化
##### 1. Java自带序列化
笨重，慢

##### 2.Kryo序列化
```scala
val conf: SparkConf = new SparkConf()
	.setAppName("SerDemo")
	.setMaster("local[*]")
	// 替换默认的序列化机制
	.set("spark.serializer",
	"org.apache.spark.serializer.KryoSerializer")
	// 注册需要使用 kryo 序列化的自定义类
	.registerKryoClasses(Array(classOf[Searcher]))
```
> [!attention] 注意
> 注意：即使使用Kryo 序列化，也要继承Serializable 接口。

#### RDD依赖关系
>将创建RDD的一系列Lineage（血统）记录下来，以便恢复丢失的分区。
##### 窄依赖
窄依赖表示每一个父(上游)RDD 的 Partition 最多被子（下游）RDD 的一个Partition 使用，窄依赖我们形象的比喻为独生子女。
##### 宽依赖
宽依赖表示同一个父（上游）RDD 的 Partition 被多个子（下游）RDD 的Partition 依赖，会引起Shuffle，总结：宽依赖我们形象的比喻为多生。
##### RDD阶段划分
DAG有向无环图是由点和线组成的拓扑图形，该图形具有方向，不会闭环。例如，DAG 记录了RDD 的转换过程和任务的阶段。
##### RDD任务划分
RDD 任务切分中间分为：Application、Job、Stage 和Task
- Application：初始化一个 SparkContext 即生成一个Application；
- Job：一个Action 算子就会生成一个Job；
- Stage：Stage 等于宽依赖(ShuffleDependency)的个数加1；
- Task：一个 Stage 阶段中，最后一个RDD 的分区个数就是Task 的个数。
注意：Application->Job->Stage->Task 每一层都是1 对n 的关系。

#### RDD持久化
##### RDD Cache 缓存
RDD 通过Cache 或者Persist 方法将前面的计算结果缓存，默认情况下会把数据以缓存在JVM 的堆内存中。但是并不是这两个方法被调用时立即缓存，而是触发后面的 action 算子时，该RDD 将会被缓存在计算节点的内存中，并供后面重用。
```scala
// cache操作会增加血缘关系，不改变原有的血缘关系
println(wordToOneRdd.toDebugString)
// 数据缓存。
wordToOneRdd.cache()
// 可以更改存储级别
mapRdd.persist(StorageLevel.MEMORY_AND_DISK_2)
```
##### RDD CheckPoint检查点
所谓的检查点其实就是通过将RDD 中间结果写入磁盘
由于血缘依赖过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果检查点之后有节点出现问题，可以从检查点开始重做血缘，减少了开销。
##### 缓存和检查点区别
1. Cache 缓存只是将数据保存起来，不切断血缘依赖。Checkpoint 检查点切断血缘依赖。
2. Cache 缓存的数据通常存储在磁盘、内存等地方，可靠性低。Checkpoint 的数据通常存储在HDFS 等容错、高可用的文件系统，可靠性高。
3. 建议对checkpoint()的RDD 使用Cache 缓存，这样 checkpoint 的job 只需从 Cache 缓存中读取数据即可，否则需要再从头计算一次RDD。

#### RDD 分区器
>Spark 目前支持Hash 分区和Range 分区，和用户自定义分区。Hash 分区为当前的默认分区。分区器直接决定了RDD 中分区的个数、RDD 中每条数据经过Shuffle 后进入哪个分区，进而决定了Reduce 的个数。

- 只有Key-Value 类型的RDD 才有分区器，非 Key-Value 类型的RDD 分区的值是 None
- 每个RDD 的分区 ID 范围：0 ~ (numPartitions - 1)，决定这个值是属于那个分区的。
##### Hash 分区
对于给定的 key，计算其hashCode,并除以分区个数取余
##### Range 分区
将一定范围内的数据映射到一个分区中，尽量保证每个分区数据均匀，而且分区间有序

#### RDD 文件读取与保存
>Spark 的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。
>文件格式分为：text 文件、csv 文件、sequence 文件以及Object 文件；
>文件系统分为：本地文件系统、HDFS、HBASE 以及数据库。

##### sequence文件
SequenceFile 文件是Hadoop 用来存储二进制形式的key-value 对而设计的一种平面文件(FlatFile)。在 SparkContext 中，可以调用sequenceFile。
##### object对象文件
对象文件是将对象序列化后保存的文件，采用Java 的序列化机制。

#### 累加器
#####  实现原理
累加器用来把Executor 端变量信息聚合到Driver 端。在Driver 程序中定义的变量，在Executor 端的每个Task 都会得到这个变量的一份新的副本，每个task 更新这些副本的值后，传回Driver 端进行merge。

##### 自定义累加器
```scala
// 自定义累加器
// 1. 继承 AccumulatorV2，并设定泛型
// 2. 重写累加器的抽象方法
class WordCountAccumulator extends AccumulatorV2[String, mutable.Map[String,Long]]{
var map : mutable.Map[String, Long] = mutable.Map()
// 累加器是否为初始状态
override def isZero: Boolean = {
	map.isEmpty
}
// 复制累加器
override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = {
	new WordCountAccumulator
}
// 重置累加器
override def reset(): Unit = {
	map.clear()
}
// 向累加器中增加数据 (In)
override def add(word: String): Unit = {
	// 查询 map中是否存在相同的单词
	// 如果有相同的单词，那么单词的数量加 1
	// 如果没有相同的单词，那么在 map中增加这个单词
	map(word) = map.getOrElse(word, 0L) + 1L
}
// 合并累加器
override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]): Unit = {
	val map1 = map
	val map2 = other.value
	// 两个 Map的合并
	map = map1.foldLeft(map2)(
		( innerMap, kv ) => {
			innerMap(kv._1) = innerMap.getOrElse(kv._1, 0L) + kv._2
			innerMap
		}
	)
}
// 返回累加器的结果 （Out）
override def value: mutable.Map[String, Long] = map
}
```

#### 广播变量
>广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个Spark 操作使用。比如，如果你的应用需要向所有节点发送一个较大的只读查询表，广播变量用起来都很顺手。在多个并行操作中使用同一个变量，但是 Spark 会为每个任务分别发送。

