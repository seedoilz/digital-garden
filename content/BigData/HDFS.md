---
aliases: 
title: HDFS
date created: 三月 12日 2024, 1:36:05 下午
date modified: 三月 19日 2024, 2:11:59 下午
tags:
  - code/big-data
  - input
---
## [[HDFS架构概述]]
>[[Hadoop]] Distributed File System，简称 HDFS，是一个分布式文件系统。
![CleanShot 2024-03-18 at 13.13.10@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2013.13.10%402x.png)
![CleanShot 2024-03-18 at 13.13.34@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2013.13.34%402x.png)
1. NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。
2. DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。
3. Secondary NameNode(2nn)：每隔一段时间对NameNode元数据备份。

**使用场景**：适合一次写入，多次读出的场景。一个文件经过创建、写入和关闭之后就不需要改变。

## HDFS优缺点
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

## [[HDFS文件块]]
>[[HDFS]]中的文件在物理上是分块存储 (Block)，块的大小可以通过配置参数(dfs.blocksize)来规定，默认大小在Hadoop2.x/3.x版本中是128M，1.x版本中是64M。

***寻址时间为传输时间的1%时，则为最佳状态。（专家）***

### 为什么块的大小不能设置太小，也不能设置太大？
1. HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置；
2. 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢。

## [[HDFS读写流程]]
### 文件写入
![CleanShot 2024-03-18 at 17.05.47@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2017.05.47%402x.png)
#### 步骤
1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
4. NameNode 返回3个DataNode 节点，分别为dn1、dn2、dn3。
5. 客户端通过 FSDataOutputStream 模块请求dn1 上传数据，dn1 收到请求会继续调用dn2，然后dn2 调用dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。
7. 客户端开始往dn1 上传第一个Block （先从磁盘读取数据放到一个本地内存缓存），以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet会放入一个应答队列等待应答。
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。

### 网络拓扑-节点距离计算
![CleanShot 2024-03-18 at 17.32.06@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2017.32.06%402x.png)

### 副本节点选择
![CleanShot 2024-03-18 at 18.47.01@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2018.47.01%402x.png)

### HDFS读流程
![CleanShot 2024-03-18 at 18.57.41@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2018.57.41%402x.png)
1. 客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的DataNode 地址。
2. 挑选一台 DataNode（*就近原则，然后随机*）服务器，请求读取数据。
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## [[NameNode和SecondaryNameNode]]
### NN 和 2NN 工作机制
#### NameNode 中的元数据存储在哪？
如果存储在 NameNode 节点的**磁盘**里，那么一定会带来*效率过低*的问题。
但如果存储在 NameNode 节点的**内存**中，那么又会带来*低可靠性*的问题
因此引出**在磁盘中备份元数据**的***FsImage***

然而如果内存中的元数据和 FsImage 同步更新，就会导致效率过低（由于磁盘的随机读写能力弱）。
因此引入 **Edits** 文件（只进行追加操作，效率很高）。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到 Edits 中。

如果长时间**追加**数据到 Edits 文件中，则会导致该文件数据过大，效率降低；且一旦断电，恢复元数据需要的时间过长。因此，需要*定期进行 FsImage 和 Edits 的合并*，交由 **SecondaryNameNode** 来完成。（如果由 NameNode 完成，效率过低。）

#### 工作阶段
##### 第一阶段：NameNode 启动
1. 第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
2. 客户端对元数据进行增删改的请求。
3. NameNode 记录操作日志，更新滚动日志。
4. NameNode 在内存中对元数据进行增删改。
##### 第二阶段：SecondaryNameNode 启动

1. Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode是否检查结果。
	1. 触发条件1: 定时时间到了
	2. 触发条件2: Edits 中的数据满了
2. Secondary NameNode 请求执行CheckPoint。
3. NameNode 滚动正在写的Edits 日志。
4. 将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
5. Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。
6. 生成新的镜像文件 fsimage.chkpoint。
7. 拷贝fsimage.chkpoint 到NameNode。
8. NameNode 将 fsimage.chkpoint 重新命名成fsimage。
![CleanShot 2024-03-19 at 11.01.42@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-19%20at%2011.01.42%402x.png)

### Fsimage 和Edits 解析
1. Fsimage文件：HDFS文件系统元数据的一个**永久性的检查点**，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。
2. Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到Edits文件中。
3. seen_txid文件保存的是一个数字，就是最后一个edits_的数字
4. 每次NameNode启动的时候都会将Fsimage文件读入内存，加载Edits里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成NameNode启动的时候就将Fsimage和Edits文件进行了合并。

## [[DataNode]]
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
