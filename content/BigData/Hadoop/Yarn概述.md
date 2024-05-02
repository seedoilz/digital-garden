---
aliases: 
title: Yarn概述
date created: 2024-03-31 20:03:00
date modified: 2024-04-01 20:04:23
tags: [code/big-data]
---
### 组成
1. ResourceManager（RM）：整个集群资源（内存、CPU等）的老大
2. NodeManager（NM）：单个节点服务器资源老大
3. ApplicationMaster（AM）：单个任务运行的老大
4. Container：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。
![CleanShot 2024-03-31 at 20.13.18.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-31%20at%2020.13.18.png)

### 工作机制

![CleanShot 2024-03-31 at 20.23.44.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-31%20at%2020.23.44.png)
> [!NOTE] 解释
> 1. MapReduce 程序提交到客户端所在的节点。
> 2. YarnRunner 向 ResourceManager 申请一个Application。
> 3. ResourceManager 将该**应用程序的资源路径（存放位置）**返回给YarnRunner。
> 4. 该程序将运行所需资源提交到HDFS 上。
> 5. 程序资源提交完毕后，申请运行 mrAppMaster。
> 6. ResourceManager 将用户的请求初始化成一个 Task。
> 7. 其中一个NodeManager 领取到 Task 任务。
> 8. 该 NodeManager 创建容器Container，并产生 MRAppmaster。
> 9. Container 从HDFS 上拷贝资源到本地。
> 10. MRAppmaster 向RM 申请运行 MapTask 资源。
> 11. RM 将运行 MapTask 任务分配给另外两个 NodeManager， 另两个 NodeManager 分别领取任务并创建容器。
> 12. MR 向两个接收到任务的NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动MapTask，MapTask 对数据分区排序。
> 13. MrAppMaster 等待所有MapTask 运行完毕后，向RM 申请容器， 运行ReduceTask。
> 14. ReduceTask 向MapTask 获取相应分区的数据。
> 15. 程序运行完毕后，MR 会向 ResourceManager 申请注销自己。


