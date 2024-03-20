---
aliases: 
title: HDFS架构概述
date created: 2024-03-18 13:03:00
date modified: 2024-03-20 11:03:18
tags: [code/big-data]
---
>Hadoop Distributed File System，简称 HDFS，是一个分布式文件系统。
![CleanShot 2024-03-18 at 13.13.10@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2013.13.10%402x.png)
![CleanShot 2024-03-18 at 13.13.34@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2013.13.34%402x.png)
1. NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。
2. DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。
3. Secondary NameNode(2nn)：每隔一段时间对NameNode元数据备份。

**使用场景**：适合一次写入，多次读出的场景。一个文件经过创建、写入和关闭之后就不需要改变。

## 优缺点
### 优点
1. 高容错性
2. 适合处理大数据
3. 可构建在廉价机器上，通过多副本机制，提高可靠性。

### 缺点
1. 不适合低延时数据访问
2. 无法高效的对大量小文件进行存储
3. 不支持并发写入、文件随机修改。
	1. 一个文件只能有一个写，不允许多个线程同时写
	2. 仅支持数据append（追加），不支持文件的随机修改