---
aliases: 
title: HBase配置
date created: 2024-05-03 09:05:00
date modified: 2024-05-03 09:05:23
tags: [code/big-data]
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]
3. 首先保证[[Zookeeper配置|Zookeeper集群的正常部署]]，并**启动**。
4. [[Hadoop配置|Hadoop集群的正常部署]]并**启动**。

## 集群搭建具体步骤
### 1、下载`HBase`
```shell
wget 
```

### 2、解压重命名
```shell
tar -zxvf hbase-2.4.11-bin.tar.gz -C /opt/module/
mv /opt/module/hbase-2.4.11 /opt/module/hbase
```

### 3、配置环境变量
```shell
sudo vim /etc/profile.d/my_envs.sh
# 添加
#HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin

source /etc/profile.d/my_envs.sh
```

### 4、修改配置文件
#### hbase-env.sh修改内容，可以添加到最后
```shell
export HBASE_MANAGES_ZK=false
```
#### hbase-site.xml修改内容
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop1,hadoop2,hadoop3</value>
    <description>The directory shared by RegionServers.
    </description>
  </property>

  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop1:8020/hbase</value>
    <description>The directory shared by RegionServers.
    </description>
  </property>

  <property>
    <name>hbase.wal.provider</name>
    <value>filesystem</value>
  </property>
</configuration>
```
#### regionservers
```shell
hadoop1
hadoop2
hadoop3
```
> [!Warning] 解决HBase和Hadoop的log4j兼容性问题，修改HBase的jar包，使用Hadoop的jar包
> ```shell
> mv /opt/module/hbase/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar /opt/module/hbase/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar.bak
> ```

### 5、分发到其他节点
```shell
xsync /opt/module/hbase
```

### 6、HBase服务启动
#### 群起
```shell
bin/start-hbase.sh
```
#### 单点启动
```shell
bin/hbase-daemon.sh start master
bin/hbase-daemon.sh start regionserver
```
#### 停止服务
```shell
bin/stop-hbase.sh
```

### 7、查看HBase页面
启动成功后，可以通过“host:port”的方式来访问HBase管理页面，例如：[http://hadoop1:16010](http://hadoop1:16010)

### 8、高可用（可选）
在HBase中HMaster负责监控HRegionServer的生命周期，均衡RegionServer的负载，如果HMaster挂掉了，那么整个HBase集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对HMaster的高可用配置。

#### 关闭HBase集群（如果没有开启则跳过此步）
```shell
bin/stop-hbase.sh
```

#### 在conf目录下创建backup-masters文件
```shell
touch conf/backup-masters
```

#### 在backup-masters文件中配置高可用HMaster节点
```shell
echo hadoop2 > conf/backup-masters
```

#### 将整个conf目录scp到其他节点
```shell
xsync conf
```

#### 重启hbase,打开页面测试查看
[http://hadooo1:16010](http://hadooo1:16010)
