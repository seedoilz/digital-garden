---
aliases: 
tags: [code/big-data, code/database]
title: Doris架构
date created: 2024-05-16 14:05:00
date modified: 2024-07-28 12:07:59
---
## 架构
![CleanShot 2024-05-16 at 14.20.14.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-16%20at%2014.20.14.png)
主要有三个角色：
1. Leader 和Follower：主要是用来达到元数据的高可用，保证单节点宕机的情况下，元数据能够实时地在线恢复，而不影响整个服务。
2. Observer：用来扩展查询节点，同时起到元数据备份的作用。如果在发现集群压力非常大的情况下，需要去扩展整个查询的能力，那么可以加 observer 的节点。observer 不参与任何的写入，只参与读取。

### BackEnd（BE）
主要负责数据存储、查询计划的执行。

数据的可靠性由 BE 保证，BE 会对整个数据存储多副本或者是三副本。副本数可根据需求动态调整。

### MyQLClient
Doris借助MySQL协议，用户使用任意MySQL的ODBC/JDBC以及MySQL的客户端，都可以直接访问Doris。

### Broker
Broker 是 Doris 集群中一种可选进程，主要用于支持 Doris 读写远端存储上的文件和目录，如 HDFS、BOS 和 AFS 等。
