---
aliases: 
title: Kafka配置
date created: 2024-04-09 14:04:00
date modified: 2024-05-02 21:05:29
tags:
  - code/big-data
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]
3. `Kafka`运行在`JVM`上，需要安装`JDK`
4. `kafka`依赖`zookeeper`，需要安装`zookeeper`，可以参考我的另一篇文章[[Zookeeper配置]]

> 注意：下边的步骤都是在`hadoop1`这个节点上进行的操作，除特殊说明外。
## 集群搭建具体步骤
### 1、下载安装包
```shell
cd /opt/module
# 下载kafka安装包
wget https://archive.apache.org/dist/kafka/2.6.0/kafka_2.13-2.6.0.tgz
```

### 2、解压
```shell
# 解压kafka安装包
tar -zxvf kafka_2.13-2.6.0.tgz
mv kafka_2.13-2.6.0 kafka
```

### 3、创建存放kafka消息的目录
```shell
cd kafka
mkdir kafka-logs
```

### 4、修改配置文件
```shell
vim /opt/module/kafka/config/server.properties
# 修改如下参数
broker.id=0 
listeners=PLAINTEXT://hadoop1:9092
log.dirs=/opt/module/kafka/kafka-logs
zookeeper.connect=hadoop1:2181,hadoop2:2181,hadoop3:2181
```
> [!Note] 参数说明
> `broker.id` ： 集群内全局唯一标识，**每个节点上需要设置不同的值**
> `listeners`：这个`IP`地址也是与本机相关的，每个节点上设置为自己的`IP`地址
> `log.dirs` ：存放`kafka`消息的 
> `zookeeper.connect` ： 配置的是zookeeper集群地址

### 5、分发kafka安装目录
```shell
# 分发kafka安装目录给其他集群节点
scp -r /opt/module/kafka/ hadoop2:/opt/module
scp -r /opt/module/kafka/ hadoop3:/opt/module
```
> [!Warning] 注意⚠️
分发完成后，其他集群节点都需要修改配置文件`server.properties`中的 `broker.id` 和`listeners` 参数。

### 6、编写kafka集群操作脚本
```shell
cd /opt/module/kafka/bin
# 创建kafka启动脚本
vim kafka-cluster.sh
# 添加如下内容
#!/bin/bash
case $1 in
"start"){
	for i in hadoop1 hadoop2 hadoop3
	do 
		 echo -------------------------------- $i kafka 启动 ---------------------------
		ssh $i "source /etc/profile;/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
	done
}
;;
"stop"){
	for i in hadoop1 hadoop2 hadoop3
	do
		echo -------------------------------- $i kafka 停止 ---------------------------
		ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh"
	done
}
;;
esac

# 保存退出后，修改执行权限
chmod +x ./kafka-cluster.sh
```

脚本命令说明：

```shell
启动kafka集群命令
./kafka-cluster.sh start

停止kafka集群命令
./kafka-cluster.sh stop
```

### 7、启动kafka集群
首先启动`zookeeper`集群 然后执行`kafka`集群脚本启动命令
```shell
cd /opt/module/kafka/bin
./kafka-cluster.sh start
```

### 8、测试

`kafka`集群启动成功后，我们就可以对`kafka`集群进行操作了。

#### 创建主题
```shell
cd /opt/module/kafka
./bin/kafka-topics.sh --create --bootstrap-server hadoop1:9092 --replication-factor 3 --partitions 1 --topic test
```

#### 查看主题列表
```shell
./bin/kafka-topics.sh --list --bootstrap-server hadoop1:9092
```

#### 启动控制台生产者
```shell
./bin/kafka-console-producer.sh --broker-list hadoop1:9092 --topic test
```

#### 启动控制台消费者
```shell
./bin/kafka-console-consumer.sh --bootstrap-server hadoop1:9092 --topic test --from-beginning
```

在生产者控制台输入`hello kafka`，消费者控制台，就可以消费到生产者的消息，输出 `hello kafka`，表示消费端成功消费了生产者生产的消息！