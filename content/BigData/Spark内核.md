---
aliases: 
title: Spark内核
date created: 2024-04-11 14:04:00
date modified: 2024-04-12 09:04:99
tags: [code/big-data]
---
## [[Spark运行机制]]
### 通用运行流程
1. 任务提交后，都会先启动 Driver 程序；
2. 随后 Driver 向集群管理器注册应用程序；
3. 之后集群管理器根据此任务的配置文件分配 Executor 并启动；
4. Driver 开始执行 main 函数，Spark 查询为懒执行，当执行到 Action 算子时开始反向推算，根据宽依赖进行 Stage 的划分，随后每一个 Stage 对应一个 Taskset，Taskset 中有多个 Task，查找可用资源 Executor 进行调度；
5. 根据本地化原则，Task 会被分发到指定的 Executor 去执行，在任务执行的过程中，Executor 也会不断与 Driver 进行通信，报告任务运行情况。

### YARN Cluster 模式
1. 执行脚本提交方法，实际启动了一个 SparkSubmit 的 JVM 进程；
2. SparkSubmit 调用了 YarnClusterApplication 的 main 方法
3. YarnClusterApplication 创建了 Yarn 客户端，然后向 Yarn 服务器发送执行启动 ApplicationMaster 的指令。
4. Yarn 框架收到指令后会在指定的 NodeManager 中启动 ApplicationMaster
5. ApplicationMaster 启动 Driver 线程，执行用户的作业
6. AM 向 RM 注册，申请资源启动 NM；
7. 获取资源后 AM 向 NM 发送指令: `bin/java YarnCoarseGrainedExecutorBackend`
8. CoarseGrainedExecutorBackend 进程会接受消息，跟 Driver 通信，注册已经启动的 Executor；然后启动计算对象 Executor 等待接收任务。
9. Driver 线程继续执行完成作业的调度和任务的执行。
10. Driver 分配任务并监控任务的执行。
![截屏2024-04-11 下午3.22.16.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-15-22-20.png)

### YARN Client 模式
1. 执行脚本提交任务，实际是启动一个 SparkSubmit 的 JVM 进程；
2. SparkSubmit 类中的 main 方法反射调用用户代码的 main 方法；
3. 启动 Driver 线程，执行用户的作业，并创建 ScheduleBackend；
4. YarnClientSchedulerBackend 向 RM 发送指令：bin/java ExecutorLauncher；
5. Yarn 框架收到指令后会在指定的 NM 中启动 ExecutorLauncher（实际上还是调用 ApplicationMaster 的 main 方法）
6. AM 向 RM 注册，申请资源
7. 获取资源后 AM 向 NM 发送指令：bin/java CoarseGrainedExecutorBackend；
8. CoarseGrainedExecutorBackend 进程会接收消息，跟 Driver 通信，注册已经启动的 Executor；然后启动计算对象 Executor 等待接收任务
9. Driver 分配任务并监控任务的执行。
![截屏2024-04-11 下午3.36.41.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-15-37-58.png)

### Standalone Client 模式
>在 Standalone Client 模式下，Driver 在任务提交的本地机器上运行。
![截屏2024-04-11 下午3.43.40.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-15-43-44.png)
### Standalone Cluster 模式
>在 Standalone Cluster 模式下，任务提交后，Master 会找到一个 Worker 启动 Driver。
![截屏2024-04-11 下午3.43.59.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-15-44-20.png)

## [[Spark通讯架构]]
### 通讯架构概述
Spark2.x 版本使用 Netty 通讯框架作为内部通讯组件，借鉴了Akka的设计，基于Actor模型。
Spark通讯框架中各个组件被认为是一个个独立的实体，各个实体之间通过消息来进行通信。
![截屏2024-04-11 下午3.53.39.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-15-53-49.png)

Endpoint （Client/Master/Worker）有**1 个InBox 和N 个OutBox** （N>=1， N 取决于当前 Endpoint与多少其他的Endpoint 进行通信，一个与其通讯的其他Endpoint 对应一个OutBox）， Endpoint接收到的消息被写入InBox，发送出去的消息写入OutBox 并被发送到其他Endpoint 的InBox中。
### 通讯架构解析
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-16-08-44.png)

## Spark任务调度机制
### Spark任务调度概述
当 Driver 起来后，Driver 则会根据用户程序逻辑准备任务，并根据 Executor 资源情况逐步分发任务。在详细阐述任务调度前，首先说明下 Spark 里的几个概念。一个 Spark 应用程序包括 Job、Stage 以及 Task 三个概念：
1. Job 是以 Action 方法为界，遇到一个 Action 方法则触发一个 Job；
	1. Stage 是 Job 的子集，以 RDD 宽依赖 (即 Shuffle) 为界，遇到 Shuffle 做一次划分；
2. Task 是 Stage 的子集，以并行度 (分区数) 来衡量，分区数是多少，则有多少个 task。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-16-40-13.png)
Spark RDD 通过其Transactions 操作，形成了RDD 血缘（依赖）关系图，即DAG，最后通过Action 的调用，触发 Job 并调度执行， 执行过程中会创建两个调度器：DAGScheduler和TaskScheduler。
- DAGScheduler 负责 Stage 级的调度，主要是将 job 切分成若干 Stages，并将每个 Stage打包成TaskSet 交给TaskScheduler 调度。
- TaskScheduler 负责Task 级的调度，将DAGScheduler 给过来的TaskSet 按照指定的调度策略分发到 Executor 上执行，调度过程中 SchedulerBackend 负责提供可用资源，其中SchedulerBackend 有多种实现，分别对接不同的资源管理系统。

### Spark Stage级调度
Spark 的任务调度是从 DAG 切割开始，主要是由 DAGScheduler 来完成。当遇到一个Action 操作后就会触发一个 Job 的计算，并交给 DAGScheduler 来提交，下图是涉及到 Job提交的相关方法调用流程图。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-16-52-28.png)
1) Job 由最终的RDD 和Action 方法封装而成；
2) SparkContext 将 Job 交给 DAGScheduler 提交，它会根据 RDD 的血缘关系构成的 DAG 进行切分，将一个 Job 划分为若干 Stages，具体划分策略是，由最终的 RDD 不断通过依赖回溯判断父依赖是否是宽依赖，即以 Shuffle 为界，划分 Stage，窄依赖的 RDD 之间被划分到同一个 Stage 中，可以进行 pipeline 式的计算。划分的 Stages 分两类，一类叫做 ResultStage，为 DAG 最下游的 Stage，由 Action 方法决定，另一类叫做ShuffleMapStage，为下游Stage 准备数据，下面看一个简单的例子 WordCount。
### Spark Task级调度
Spark Task 的调度是由 TaskScheduler 来完成，由前文可知，DAGScheduler 将 Stage 打包到交给 TaskScheTaskSetduler，TaskScheduler 会将 TaskSet 封装为 TaskSetManager 加入到调度队列中，TaskSetManager 结构如下图所示。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-17-08-02.png)
> [!NOTE] 资源调度单位
> TaskSetManager 负责监控管理同一个 Stage 中的 Tasks ，TaskScheduler 就是以TaskSetManager 为单元来调度任务。

上图中，将 TaskSetManager 加入 rootPool 调度池中之后，调用 SchedulerBackend 的 riviveOffers 方法给 driverEndpoint 发送 ReviveOffer 消息； driverEndpoint 收到 ReviveOffer 消尚硅谷大数据技术之 Spark 内核息后调用 makeOffers 方法，过滤出活跃状态的 Executor（这些 Executor 都是任务启动时反向注册到 Driver 的 Executor），然后将 Executor 封装成 WorkerOffer 对象；准备好计算资源（WorkerOffer）后，taskScheduler 基于这些资源调用 resourceOffer 在Executor 上分配 task。

#### 调度策略
TaskScheduler 支持两种调度策略， 一种是FIFO，也是默认的调度策略，另一种是 FAIR。
在TaskScheduler 初始化过程中会实例化 rootPool，表示树的根节点，是 Pool 类型。
##### FIFO调度策略
如果是采用 FIFO 调度策略，则直接简单地将 TaskSetManager 按照先来先到的方式入队，出队时直接拿出最先进队的 TaskSetManager，其树结构如下图所示，TaskSetManager 保存在一个FIFO 队列中
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-17-23-40.png)
##### FAIR调度策略
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-17-24-27.png)
FAIR 模式中有一个 rootPool 和多个子 Pool，各个子 Pool 中存储着所有待分配的TaskSetMagager。
在 FAIR 模式中，需要先对子Pool 进行排序，再对子Pool 里面的TaskSetMagager 进行排序，因为 Pool 和 TaskSetMagager 都继承了 Schedulable 特质，因此使用相同的排序算法。
排序过程的比较是基于 Fair-share 来比较的，每个要排序的对象包含三个属性:
runningTasks 值（正在运行的Task 数）、 minShare 值、 weight 值，比较时会综合考量runningTasks值，minShare 值以及weight 值。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-11-17-28-19.png)

#### 本地化调度
DAGScheduler 切割 Job，划分 Stage, 通过调用 submitStage 来提交一个 Stage 对应的tasks， submitStage 会调用 submitMissingTasks， submitMissingTasks 确定每个需要计算的 task的preferredLocations，通过调用 getPreferrdeLocations()得到partition 的优先位置。
根据每个 Task 的优先位置，确定 Task 的 Locality 级别，Locality 一共有五种，优先级由高到低顺序：

| 名称            | 解析                                                              |
| ------------- | --------------------------------------------------------------- |
| PROCESS_LOCAL | 进程本地化，task 和数据在同一个 Executor 中，性能最好。                             |
| NODE_LOCAL    | 节点本地化，task 和数据在同一个节点中，但是 task 和数据不在同一个 Executor 中，数据需要在进程间进行传输。 |
| RACK_LOCAL    | 机架本地化， task 和数据在同一个机架的两个节点上，数据需要通过网络在节点之间进行传输。                  |
| NO_PREF       | 对于 task 来说，从哪里获取都一样，没有好坏之分。                                     |
| ANY           | task 和数据可以在集群的任何地方，而且不在一个机架中，性能最差。                              |

> [!NOTE] 降级规则
> 在调度执行时，Spark 调度总是会尽量让每个 task 以最高的本地性级别来启动，当一个task 以X 本地性级别启动，但是该本地性级别对应的所有节点都没有空闲资源而启动失败，此时并不会马上降低本地性级别启动而是在某个时间长度内再次以 X 本地性级别来启动该task，若超过限时时间则降级启动，去尝试下一个本地性级别，依次类推。

#### 失败重试与黑名单机制
>除了选择合适的 Task 调度运行外，还需要监控Task 的执行状态。

前面也提到，与外部打交道的是 SchedulerBackend，Task 被提交到 Executor 启动执行后，Executor 会将执行状态上报给 SchedulerBackend，SchedulerBackend 则告诉 TaskScheduler，TaskScheduler 找到该 Task 对应的 TaskSetManager，并通知到该 TaskSetManager，这样 TaskSetManager 就知道 Task 的失败与成功状态，对于失败的 Task，会记录它失败的次数，如果失败次数还没有超过最大重试次数，那么就把它放回待调度的 Task 池子中，否则整个 Application 失败。

在记录 Task 失败次数过程中，会记录它上一次失败所在的 Executor Id 和 Host，这样下次再调度这个 Task 时，会使用黑名单机制，避免它被调度到上一次失败的节点上，起到一定的容错作用。黑名单记录 Task 上一次失败所在的 Executor Id 和 Host，以及其对应的“拉黑”时间，“拉黑”时间是指这段时间内不要再往这个节点上调度这个Task 了。

## Spark Shuffle解析
### HashShuffle解析
#### 未优化的HashShuffle
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-12-14-31-28.png)
#### 优化后的HashShuffle
>优化的 HashShuffle 过程就是启用合并机制，合并机制就是复用 buffer，开启合并机制的配置是 spark.shuffle.consolidateFiles。该参数默认值为 false，将其设置为 true 即可开启优化机制。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-12-14-34-40.png)
### SortShuffle解析
#### 普通SortShuffle
>在该模式下，数据会先写入一个数据结构，reduceByKey 写入 Map，一边通过 Map 局部聚合，一遍写入内存。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-12-15-04-28.png)

#### bypass SortShuffle
>该过程的磁盘写机制其实跟未经优化的 HashShuffleManager 是一模一样的，因为都要创建数量惊人的磁盘文件，只是在最后会做一个磁盘文件的合并而已。
>该机制与普通 SortShuffleManager 运行机制的不同在于：不会进行排序。