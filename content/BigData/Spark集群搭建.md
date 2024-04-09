---
aliases: 
title: Spark集群搭建
date created: 2024-04-09 14:04:00
date modified: 2024-04-09 15:04:98
tags:
  - code/big-data
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]
## 集群搭建具体步骤

> 注意：以下步骤均在`hadoop1`节点上进行操作，特殊说明除外！

### Standalone模式

#### 1、下载`spark-3.0.0`的jar包

下载地址：  
[https://archive.apache.org/dist/spark/spark-3.0.0/spark-3.0.0-bin-hadoop3.2.tgz](https://archive.apache.org/dist/spark/spark-3.0.0/spark-3.0.0-bin-hadoop3.2.tgz)

#### 2、上传并解压

将下载好的 `spark-3.0.0-bin-hadoop3.2.tgz` 上传到 `hadoop1` 虚拟机节点`/opt/module`目录下。

```shell
tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz
```

#### 3、配置SPARK_HOME环境变量

```shell
vim ~/.bashrc 

export SPARK_HOME=/opt/module/spark-3.0.0-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

source ~/.bashrc 
```

#### 4、修改配置

```shell
cd /opt/module/spark-3.0.0-bin-hadoop3.2/conf

cp spark-defaults.conf.template spark-defaults.conf
cp spark-env.sh.template spark-env.sh
cp slaves.template slaves
```

##### 4.1 修改 spark-defaults.conf

```shell
vim spark-defaults.conf

spark.master                     spark://hadoop1:7077
spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.driver.memory              1g
spark.executor.memory            1g
```

##### 4.2 修改spark-env.sh

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_131
export HADOOP_HOME=/opt/module/hadoop
export HADOOP_CONF_DIR=/opt/module/hadoop/etc/hadoop
export SPARK_DIST_CLASSPATH=$(/opt/module/hadoop/bin/hadoop classpath)
export SPARK_MASTER_HOST=hadoop1
export SPARK_MASTER_PORT=7077
```

##### 4.3 修改slaves文件

```shell
vim slaves

hadoop1
hadoop2
hadoop3
```

#### 5、将spark-3.0.0-bin-hadoop3.2 目录分发到其他节点

```shell
scp -r ./spark-3.0.0-bin-hadoop3.2 hadoop2:/opt/module/
scp -r ./spark-3.0.0-bin-hadoop3.2 hadoop3:/opt/module/
```

#### 6、启动Spark集群

```shell
start-all.sh
```

#### 7、在web界面查看Spark UI

在`Linux`系统上浏览器上查看`Spark UI`：  
`http://hadoop1:8080/`

#### 8、测试

运行`SparkPI`进行案例测试：

```shell
spark-submit --class org.apache.spark.examples.SparkPi \
$SPARK_HOME/examples/jars/spark-examples_2.12-3.0.0.jar 10
```

### Yarn模式

上面默认是用`standalone`模式启动的服务，如果想要把资源调度交给`yarn`来做，则需要配置为`yarn`模式：  
需要启动的服务：`hdfs服务`、`yarn服务`  
需要关闭 `Standalone` 对应的服务(即集群中的`Master`、`Worker`进程)。  
在`Yarn`模式中，`Spark`应用程序有两种运行模式：  
`yarn-client`：`Driver`程序运行在客户端，适用于交互、调试，希望立即看到app的输出  
`yarn-cluster`：`Driver`程序运行在由`RM`启动的 `AppMaster`中，适用于生产环境  
二者的主要区别：`Driver`在哪里！

#### 1、开启 [hdfs](https://so.csdn.net/so/search?q=hdfs&spm=1001.2101.3001.7020)、yarn服务

```shell
start-dfs.sh
start-yarn.sh
```

#### 2、修改Hadoop中的 yarn-site.xml 配置

在`$HADOOP_HOME/etc/hadoop/yarn-site.xml`中增加如下配置，然后分发到集群其他节点，重启`yarn` 服务。

```shell
vim /opt/module/hadoop/etc/hadoop/yarn-site.xml 
```

```XML
<property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
</property>
<property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
</property>
```

> 说明：  
> `yarn.nodemanager.pmem-check-enabled` ： 是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是`true`。  
> `yarn.nodemanager.vmem-check-enabled` ：是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是`true`。

#### 3、修改Spark配置，分发到集群

```shell
# spark-env.sh 中这一项必须要有
# cd /opt/module/spark/conf
# 添加如下内容
export HADOOP_CONF_DIR=/opt/module/hadoop/etc/hadoop

# spark-default.conf(以下是优化)
# vim spark-defaults.conf
# 添加如下内容
spark.yarn.jars                    hdfs://hadoop1:8020/spark-jars/*.jar
```

#### 4、向hdfs上传spark纯净版jar包

下载`spark-3.0.0-bin-without-hadoop.tgz`  
下载地址：[https://archive.apache.org/dist/spark/spark-3.0.0/spark-3.0.0-bin-without-hadoop.tgz](https://archive.apache.org/dist/spark/spark-3.0.0/spark-3.0.0-bin-without-hadoop.tgz)

```
# 上传并解压spark-3.0.0-bin-without-hadoop.tgz
tar -zxvf spark-3.0.0-bin-without-hadoop.tgz
```

上传spark纯净版jar包到hdfs

```
hdfs dfs -mkdir /spark-jars
hdfs dfs -put spark-3.0.0-bin-without-hadoop/jars/* /spark-jars
```

> 说明：  
> 1）`Spark`任务资源分配由`Yarn`来调度，该任务有可能被分配到集群的任何一个节点。所以需要将`spark`的依赖上传到`hdfs`集群路径，这样集群中任何一个节点都能获取到，依此达到`Spark`集群的`HA`。  
> 2） `Spark`纯净版`jar`包，不包含`hadoop`和`hive`相关依赖，避免和后续安装的`Hive`出现兼容性问题。

#### 5、测试

记得，先把`Master`与`worker`进程停掉，否则会走`standalone`模式。

```
# 停掉standalone模式的服务
stop-all.sh
```

##### client运行模式

```
# client
spark-submit --master yarn \
--deploy-mode client \
--class org.apache.spark.examples.SparkPi \
$SPARK_HOME/examples/jars/spark-examples_2.12-3.0.0.jar 20
```

这种模式可以看见：程序计算的结果（即可以看见计算返回的结果）！

##### cluster运行模式

```
# cluster
spark-submit --master yarn \
--deploy-mode cluster \
--class org.apache.spark.examples.SparkPi \
$SPARK_HOME/examples/jars/spark-examples_2.12-3.0.0.jar 20
```

这种模式就看不见最终的结果！