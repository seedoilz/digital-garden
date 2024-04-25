---
aliases: 
title: Hadoop集群搭建
date created: 2024-04-09 14:04:00
date modified: 2024-04-25 20:04:41
tags: [code/big-data]
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]
3. [[Hadoop配置|一些补充内容]]
## 集群规划

我们使用三个虚拟机节点来搭建集群环境：

|ip|主机名|功能|
|---|---|---|
|192.168.36.121|hadoop1|NameNode DataNode ResourceManager NodeManager|
|192.168.36.122|hadoop2|DataNode NodeManager|
|192.168.36.123|hadoop3|SecondryNameNode DataNode NodeManager|

分别在上述的节点上修改hosts文件，增加IP和主机名的映射关系：

```shell
vim /etc/hosts

192.168.36.121   hadoop1
192.168.36.122   hadoop2
192.168.36.123   hadoop3
```

另外，`Hadoop` 集群运行需要 `Java` 运行环境，所以，在各个节点上需要安装 `JDK`！

## 集群搭建具体步骤

> 注意：以下步骤均在hadoop1节点上进行操作，特殊说明除外！

### 1、下载`hadoop-3.1.3.tar.gz`

hadoop官网下载：https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz

### 2、上传并解压

将下载好的 `hadoop-3.1.3.tar.gz` 上传到 `hadoop1` 虚拟机节点 `/opt/module` 目录下。

```shell
cd /opt/module

tar -zxvf hadoop-3.1.3.tar.gz

mv hadoop-3.1.3 hadoop
```

### 3、配置`path`变量

```shell
vim ~/.bashrc 
# 添加如下内容：
export PATH=$PATH:/opt/module/hadoop/bin:/opt/module/hadoop/sbin

# :wq! 保存退出后执行如下命令，使配置生效
source ~/.bashrc
```

### 4、修改配置文件

```shell
cd /opt/module/hadoop/etc/hadoop
```

#### 4.1 修改文件`hadoop-env.sh`

```shell
vim hadoop-env.sh

export JAVA_HOME=/usr/java/jdk1.8.0_131
```

#### 4.2 修改文件`workers`

```shell
vim workers
# 将localhost去掉添加如下内容
hadoop1
hadoop2
hadoop3
```

> 注意：需要把所有数据节点的主机名写入该文件，每行一个，默认为`localhost`（即把本机作为数据节点），所以，在伪分布式配置时，就采用了这种默认的配置，使得节点既作为名称节点也作为数据节点。在进行分布式配置时，可以保留`localhost`，让`hadoop1`节点同时充当名称节点和数据节点，或者也可以删掉`localhost`这行，让`hadoop1`节点仅作为名称节点使用。

#### 4.3 修改文件`core-site.xml`

```XML
<configuration>
	<property>
		<name>fs.defaultFS</name>
    	<value>hdfs://hadoop1:8020</value>
	</property>
	<property>
		 <name>hadoop.tmp.dir</name>						    
		 <value>file:/opt/module/hadoop/tmp</value>
		 <description>Abase for other temporary directories</description>
	</property>
</configuration>
```

#### 4.4 修改文件`hdfs-site.xml`

`dfs.replication`的值还是设置为 `3`, 也就是说，一份数据保存三份副本，`Hadoop`的分布式文件系统`HDFS`一般都是采用冗余存储。

```XML
<configuration>
  <property>
          <name>dfs.namenode.secondary.http-address</name>
          <value>hadoop1:50090</value>
  </property>
  <property>
          <name>dfs.replication</name>
          <value>3</value>
  </property>
  <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:/opt/module/hadoop/tmp/dfs/name</value>
  </property>
  <property>
          <name>dfs.datanode.data.dir</name>
          <value>file:/opt/module/hadoop/tmp/dfs/data</value>
  </property>
</configuration>
```

#### 4.5 修改文件`mapred-site.xml`

```XML
<configuration>
  <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
  </property>
  <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop1:10020</value>
  </property>
  <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>hadoop1:19888</value>
  </property>
  <property>
          <name>yarn.app.mapreduce.am.env</name>
          <value>HADOOP_MAPRED_HOME=/opt/module/hadoop</value>
  </property>
  <property>
          <name>mapreduce.map.env</name>
          <value>HADOOP_MAPRED_HOME=/opt/module/hadoop</value>
  </property>
  <property>
          <name>mapreduce.reduce.env</name>
          <value>HADOOP_MAPRED_HOME=/opt/module/hadoop</value>
  </property>
</configuration>
```

#### 4.6 修改文件 `yarn-site.xml`

```XML
<configuration>
  <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>hadoop1</value>
  </property>
  <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

#### 4.7 修改my_envs.sh
```shell
export HADOOP_USER_NAME=root
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

### 5、把`/opt/module/hadoop`复制到其他节点上

```shell
cd /opt/module
rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
rm -r ./hadoop/logs/*   # 删除日志文件
tar -zxcf hadoop.tar.gz ./hadoop   # 先压缩再复制
scp ./hadoop.tar.gz hadoop2:/opt/module
scp ./hadoop.tar.gz hadoop3:/opt/module
```

### 6、在其他节点上操作

```shell
cd /opt/module
rm -r ./hadoop    
tar -zxvf hadoop.tar.gz
```

### 7、名称节点的格式化

首次启动`Hadoop`集群时，需要先在`hadoop1`节点执行名称节点的格式化（只需要执行这一次，后面再启动`Hadoop`时，不要再次格式化名称节点）

```shell
hdfs namenode -format
```

### 8、启动`Hadoop`集群

需要在`hadoop1`节点上进行

```shell
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
```

### 9、验证是否启动成功

通过命令`jps`可以查看各个节点所启动的进程。如果已经正确启动，则在`hadoop1`节点上可以看到`NameNode`、`ResourceManager`、和`JobHistoryServer`以及`DataNode`和`NodeManager`进程  
在其他两个节点可以看到`DataNode`和`NodeManager`进程，在`hadoop3`节点上还可以看到`SecondryNameNode`进程  
缺少任一进程都表示出错。

### 10、查看运行实例

在执行过程中，可以在`Linux`系统中打开浏览器，在地址栏输入`http://hadoop1:8088/cluster`，通过`Web`界面查看任务进度，在`Web`界面点击 `Tracking UI` 这一列的`History`连接，可以看到任务的运行信息。

### 11、关闭`Hadoop`集群

关闭`Hadoop`集群，需要在`hadoop1`节点执行如下命令：

```shell
stop-yarn.sh
stop-dfs.sh
mr-jobhistory-daemon.sh stop historyserver
```

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
	ssh hadoop1 "/opt/module/hadoop/sbin/start-dfs.sh"
	echo " --------------- 启动 yarn ---------------"
	ssh hadoop2 "/opt/module/hadoop/sbin/start-yarn.sh"
	echo " --------------- 启动 historyserver ---------------"
	ssh hadoop3 "/opt/module/hadoop/bin/mapred --daemon start historyserver"
;;
"stop")
	echo " =================== 关闭 hadoop集群 ==================="
	echo " --------------- 关闭 historyserver ---------------"
	ssh hadoop1 "/opt/module/hadoop/bin/mapred --daemon stop
	historyserver"
	echo " --------------- 关闭 yarn ---------------"
	ssh hadoop2 "/opt/module/hadoop/sbin/stop-yarn.sh"
	echo " --------------- 关闭 hdfs ---------------"
	ssh hadoop3 "/opt/module/hadoop/sbin/stop-dfs.sh"
;;
*)
	echo "Input Args Error..."
;;
esac
```