---
aliases: 
title: HBase报错Master is initializing
date created: 2024-05-04 13:05:00
date modified: 2024-05-04 13:05:03
tags: [bug]
---
## 错误概述
>启动服务后，进入hbase shell，准备创建一个表看看能不能正常运行，然后就出错了。

### 详细信息
执行status等命令不出错，但是在create的时候会报错。
![CleanShot 2024-05-04 at 13.36.46.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-04%20at%2013.36.46.png)

## 解决方法
进入zk客户端，把hbase目录删除，重启hbase即可。

### 解决步骤
#### 进入zookeeper目录
```shell
./bin/zkCli.sh
```
#### 查看目录
```shell
ls /
```
![CleanShot 2024-05-04 at 13.40.28.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-05-04%20at%2013.40.28.png)
#### 删除hbase目录
```shell
deleteall /hbase
```

问题就解决了。