---
aliases: 
title: Spark运行架构
date created: 2024-04-03 10:04:00
date modified: 2024-04-03 10:04:24
tags: [code/big-data]
---
### 运行结构
还是标准的master-slave架构，外加一个cluster manager来做资源分配。
其中，Driver就是master，Executor就是slave。
![截屏2024-04-03 上午9.26.23.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-03-09-26-25.png)

### 核心组件
#### Driver
- 将用户程序转化为作业（job）
- 在 Executor 之间调度任务 (task)
- 跟踪Executor 的执行情况
- 通过UI 展示查询运行情况
>在这个编程过程中没有看到任何关于Driver的字眼。

#### Executor
Spark 是 集群中 Worker 中的一个JVM进程，负责运行具体任务，任务间彼此独立。Spark启动时就启动，并且伴随整个Spark应用的生命周期存在。
- 负责运行组成Spark 应用的任务，并将结果返回给驱动器进程
- 它们通过自身的块管理器（Block Manager）为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存数据加速运算。

#### ApplicationMaster
Hadoop 用户向YARN 集群提交应用程序时, 提交程序中应该包含ApplicationMaster，用于向资源调度器申请执行任务的资源容器Container，运行用户自己的程序任务 job，监控整个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。

### 提交流程
>共两种模式，主要区别在于：**Driver 程序的运行节点位置**。
#### Yarn Client模式
Client 模式将用于监控和调度的Driver 模块在客户端执行，而不是在Yarn 中，所以一般用于测试。
*Driver在任务提交的本地机器上运行。*

#### Yarn Cluster模式
Cluster 模式将用于监控和调度的 Driver 模块启动在Yarn 集群资源中执行。一般应用于实际生产环境。
本地机器任务提交后向ResourceManager申请启动ApplicationMaster，Resource分配container后在合适的NodeManager上启动ApplicationMaster，*此时ApplicationMaster就是Driver*。