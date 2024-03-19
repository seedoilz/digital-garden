---
aliases: 
title: NameNode和SecondaryNameNode
date created: 三月 19日 2024, 10:39:47 上午
date modified: 三月 19日 2024, 11:02:37 上午
tags: [code/big-data]
---
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
2. Secondary NameNode 请求执行CheckPoint。
3. NameNode 滚动正在写的Edits 日志。
4. 将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
5. Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。
6. 生成新的镜像文件 fsimage.chkpoint。
7. 拷贝fsimage.chkpoint 到NameNode。
8. NameNode 将 fsimage.chkpoint 重新命名成fsimage。
![CleanShot 2024-03-19 at 11.01.42@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-19%20at%2011.01.42%402x.png)
