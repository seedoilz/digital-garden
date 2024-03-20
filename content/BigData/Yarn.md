---
aliases: 
title: Yarn
date created: 2024-03-12 13:03:00
date modified: 2024-03-20 11:03:16
tags: [code/big-data]
---
## 组成
1. ResourceManager（RM）：整个集群资源（内存、CPU等）的老大
2. NodeManager（NM）：单个节点服务器资源老大
3. ApplicationMaster（AM）：单个任务运行的老大
4. Container：容器，相当一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。
![CleanShot 2024-03-12 at 13.57.58@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-12%20at%2013.57.58%402x.png)
