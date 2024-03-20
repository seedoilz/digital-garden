---
aliases: 
title: DataNode
date created: 三月 19日 2024, 2:12:03 下午
date modified: 三月 20日 2024, 10:21:18 上午
tags: [code/big-data]
---
### 工作流程
1. 一个数据块在DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
2. DataNode 启动后向NameNode 注册，通过后，周期性（6 小时）的向NameNode 上报所有的块信息。（默认6小时间隔后，汇报当前信息；默认6小时间隔后，扫面自己节点块信息）
3. 心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过 10 分钟没有收到某个 DataNode 的心跳，则认为该节点不可用。
4. 集群运行中可以安全加入和退出一些机器。
![CleanShot 2024-03-19 at 13.53.47@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-19%20at%2013.53.47%402x.png)

### 确保数据完整性
1. 当 DataNode 读取 Block 的时候，它会计算CheckSum。
2. 如果计算后的CheckSum，与Block 创建时值不一样，说明 Block 已经损坏。
3. Client 读取其他 DataNode 上的Block。
4. 常见的校验算法 crc(32)，md5(128)，sha1(160)
5. DataNode 在其文件创建后周期验证CheckSum。

### 掉线时限参数设置
![CleanShot 2024-03-19 at 14.11.21@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-19%20at%2014.11.21%402x.png)
