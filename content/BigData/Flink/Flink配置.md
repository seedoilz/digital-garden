---
aliases: 
title: Flink配置
date created: 2024-05-02 21:05:00
date modified: 2024-05-02 21:05:40
tags: [code/big-data]
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]

| **节点服务器** | hadoop1                   | hadoop2     | hadoop3     |
| --------- | ------------------------- | ----------- | ----------- |
| **角色**    | JobManager<br>TaskManager | TaskManager | TaskManager |
## 集群搭建具体步骤
### 1、下载并上传`Flink安装包`
```shell
# 在/opt/software目录下下载安装包
wget https://dlcdn.apache.org/flink/flink-1.17.2/flink-1.17.2-bin-scala_2.12.tgz
```
### 2、解压并重命名
```shell
tar -zxvf flink-1.17.1-bin-scala_2.12.tgz -C /opt/module/
cd /opt/module
mv flink-1.17.1-bin-scala_2.12 flink
```
### 3、修改集群配置
```shell
# 进入conf路径，修改flink-conf.yaml
vim flink-conf.yaml
# 添加以下内容
# JobManager节点地址.
jobmanager.rpc.address: hadoop102
jobmanager.bind-host: 0.0.0.0
rest.address: hadoop102
rest.bind-address: 0.0.0.0
# TaskManager节点地址.需要配置为当前机器名
taskmanager.bind-host: 0.0.0.0
taskmanager.host: hadoop1
```
### 4、修改workers
```shell
vim workers
# 指定hadoop1、hadoop2和hadoop3为TaskManager
hadoop1
hadoop2
hadoop3
```
### 5、修改masters
```shell
# vim masters
# 修改为以下内容
hadoop1:8081
```
> [!NOTE] 优化配置（可选）
> **jobmanager.memory**.process.size：对JobManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1600M，可以根据集群规模进行适当调整。
> **taskmanager.memory**.process.size：对TaskManager进程可使用到的全部内存进行配置，包括JVM元空间和其他开销，默认为1728M，可以根据集群规模进行适当调整。
> **taskmanager.numberOfTaskSlots**：对每个TaskManager能够分配的Slot数量进行配置，默认为1，可根据TaskManager所在的机器能够提供给Flink的CPU数量决定。所谓Slot就是TaskManager中具体运行一个任务所分配的计算资源。
> **parallelism.default**：Flink任务执行的并行度，默认为1。优先级低于代码中进行的并行度配置和任务提交时使用参数指定的并行度数量。

### 6、分发安装目录
```shell
# 在/opt/module下
xsync flink
```

### 7、修改hadoop2, hadoop3配置
>**切记修改**
```shell
vim flink-conf.yaml
# 修改如下内容
# TaskManager节点地址.需要配置为当前机器名
taskmanager.host: ${当前机器名}$
```

### 8、启动集群
在hadoop1节点服务器上执行start-cluster.sh启动Flink集群：
```shell
bin/start-cluster.sh
```
#### 查看进程情况
![CleanShot 2024-05-02 at 21.27.44.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-02%20at%2021.27.44.png)
#### 访问Web UI
启动成功后，同样可以访问[http://hadoop1:8081](http://hadoop1:8081)对flink集群和任务进行监控管理。
这里可以明显看到，当前集群的TaskManager数量为3；由于默认每个TaskManager的Slot数量为1，所以总Slot数和可用Slot数都为3。