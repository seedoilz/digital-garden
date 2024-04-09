---
aliases: 
title: Zookeeper集群搭建
date created: 2024-04-09 14:04:00
date modified: 2024-04-09 15:04:06
tags:
  - code/big-data
---
前期准备：

1. 准备三个虚拟机：  
    `192.168.36.121 hadoop1`  
    `192.168.36.122 hadoop2`  
    `192.168.36.123 hadoop3`
2. 每台虚拟机都配置有`ssh`服务，可以进行[[集群SSH免密配置|免密登录]]

> 注意：下边的步骤都是在`hadoop1`这个节点上进行的操作，除特殊说明外。
## 详细步骤
### 1、下载安装包

```shell
cd /opt/module

wget http://archive.apache.org/dist/zookeeper/stable/apache-zookeeper-3.6.3-bin.tar.gz
```

### 2、解压

```shell
tar -zxvf apache-zookeeper-3.6.3-bin.tar.gz
mv apache-zookeeper-3.6.3-bin zookeeper
```

### 3、修改配置文件

```shell
cd ./zookeeper/conf
# 添加zookeeper配置文件
cp zoo_sample.cfg zoo.cfg
# 创建数据存放目录
mkdir /opt/module/zookeeper/conf/data

vim zoo.cfg
# 添加如下内容
server.1=hadoop1:2188:2888
server.2=hadoop2:2188:2888
server.3=hadoop3:2188:2888
# 修改dataDir
dataDir=/opt/module/zookeeper/conf/data

# 配置文件保存退出后，进入data目录
cd ../data
# 生成myid文件，指定myid服务号
echo "1" > myid
```

### 4、将zookeeper目录分发到其他节点

```shell
# 分发到其他集群节点
scp -r zookeeper/ hadoop2:/opt/module
scp -r zookeeper/ hadoop3:/opt/module
```

### 5、修改其他节点的myid文件

登录 `hadoop2` 节点：

```shell
cd /opt/module/zookeeper/conf/data
# 指定myid服务号为 2
vim myid
```

登录 `hadoop3` 节点：

```shell
cd /opt/module/zookeeper/conf/data
# 指定myid服务号为 3
vim myid
```

### 6、编写操作[zookeeper集群](https://so.csdn.net/so/search?q=zookeeper%E9%9B%86%E7%BE%A4&spm=1001.2101.3001.7020)的脚本

```shell
cd /opt/module/zookeeper/bin
# 创建zookeeper启动脚本
vim zk.sh
# 添加如下内容
#!/bin/bash
case $1 in
"start"){
	for i in hadoop1 hadoop2 hadoop3
	do 
		 echo -------------------------------- $i zookeeper 启动 ---------------------------
		ssh $i "/opt/module/zookeeper/bin/zkServer.sh start"
	done
}
;;
"stop"){
	for i in hadoop1 hadoop2 hadoop3
	do
		echo -------------------------------- $i zookeeper 停止 ---------------------------
		ssh $i "/opt/module/zookeeper/bin/zkServer.sh stop"
	done
}
;;
"status"){
	for i in hadoop1 hadoop2 hadoop3
	do
		echo -------------------------------- $i zookeeper 状态 ---------------------------
		ssh $i "/opt/module/zookeeper/bin/zkServer.sh status"
	done
}
;;
esac

# 保存退出后，修改zk.sh脚本执行权限
chmod +x ./zk.sh
```

脚本的命令说明：

```shell
# 启动集群命令
./zk.sh start
# 停止集群命令
./zk.sh stop
# 查看集群状态命令
./zk.sh status
```

### 7、启动集群

```shell
# 启动zookeeper集群
cd /opt/module/zookeeper
./bin/zk.sh start
```

### 8、连接zookeeper集群

```shell
# 连接zookeeper集群 
cd /opt/module/zookeeper 
./bin/zkCli.sh
```