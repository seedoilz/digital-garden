---
aliases: 
title: MapReduce框架原理
date created: 2024-03-24 09:03:00
date modified: 2024-03-24 10:03:03
tags: [code/big-data]
---
### InputFormat 数据输入
- 数据块：Block 是 HDFS 物理上把数据分成一块一块。数据块是 HDFS 存储数据单位。
- 数据切片：只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。数据切片是MapReduce 程序计算输入数据的单位，一个切片会对应启动一个MapTask。

#### 数据切片与MapTask并行度决定机制
1. **一个Job的Map阶段并行度由客户端在提交Job时的切片数决定**
2. 每一个Split切片分配一个MapTask并行实例处理
3. 默认情况下，**切片大小=BlockSize**
4. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片