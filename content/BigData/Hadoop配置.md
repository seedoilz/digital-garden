---
aliases: 
title: Hadoop配置
date created: 三月 17日 2024, 7:09:19 晚上
date modified: 三月 18日 2024, 10:46:13 上午
tags: [code/big-data]
---
## 常用命令
### 各个模块分开启动/停止（配置ssh 是前提）
#### 整体启动/停止HDFS
```shell
start-dfs.sh/stop-dfs.sh
```

#### 整体启动/停止YARN
```shell
start-yarn.sh/stop-yarn.sh
```
### 各个服务组件逐一启动/停止
#### 分别启动/停止HDFS 组件
```shell
hdfs --daemon start/stop namenode/datanode/secondarynamenode
```
#### 启动/停止YARN
```shell
yarn --daemon start/stop resourcemanager/nodemanager
```
### Hadoop集群启停脚本
```shell
#!/bin/bash
if [ $# -lt 1 ]
then
	echo "No Args Input..."
	exit ;
fi

case $1 in
"start")
	echo " =================== 启动 hadoop集群 ==================="
	echo " --------------- 启动 hdfs ---------------"
	ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
	echo " --------------- 启动 yarn ---------------"
	ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
	echo " --------------- 启动 historyserver ---------------"
	ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
	echo " =================== 关闭 hadoop集群 ==================="
	echo " --------------- 关闭 historyserver ---------------"
	ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop
	historyserver"
	echo " --------------- 关闭 yarn ---------------"
	ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
	echo " --------------- 关闭 hdfs ---------------"
	ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
	echo "Input Args Error..."
;;
esac
```


### 常用端口号
![CleanShot 2024-03-18 at 10.45.12@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-18%20at%2010.45.12%402x.png)

### 注意事项
NameNode 和SecondaryNameNode 不要安装在同一台服务器
ResourceManager 也很消耗内存，不要和NameNode、SecondaryNameNode 配置在同一台机器上。
![CleanShot 2024-03-17 at 19.11.07@2x.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-03-17%20at%2019.11.07%402x.png)

### 配置
Hadoop 配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认
配置值时，才需要修改自定义配置文件，更改相应属性值。
#### （1）默认配置文件：
要获取的默认文件 文件存放在Hadoop 的 jar 包中的位置

| 要获取的默认文件              | 文件存放在Hadoop 的 jar 包中的位置                                   |
| --------------------- | --------------------------------------------------------- |
| \[core-default.xml\]  | hadoop-common-3.1.3.jar/core-default.xml                  |
| \[hdfs-default.xml]   | hadoop-hdfs-3.1.3.jar/hdfs-default.xml                    |
| \[yarn-default.xml]   | hadoop-yarn-common-3.1.3.jar/yarn-default.xml             |
| \[mapred-default.xml] | hadoop-mapreduce-client-core-3.1.3.jar/mapred-default.xml |

#### （2）自定义配置文件：
[[core-site.xml]]、[[hdfs-site.xml]]、[[yarn-site.xml]]、[[mapred-site.xml]] 四个配置文件存放在$HADOOP_HOME/etc/hadoop 这个路径上，用户可以根据项目需求重新进行修改配置。