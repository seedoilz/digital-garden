---
aliases: 
title: Maxwell配置
date created: 2024-04-24 17:04:00
date modified: 2024-04-24 17:04:02
tags: [code/big-data]
---
## 详细步骤
### 1、下载安装包
```shell
cd /opt/software
wget https://github.com/zendesk/maxwell/releases/download/v1.29.2/maxwell-1.29.2.tar.gz
```
### 2、解压并重命名
```shell
tar -zxvf maxwell-1.29.2.tar.gz -C /opt/module/
mv maxwell-1.29.2/ maxwell
```
### 3、配置MySQL
```shell
# 修改MySQL配置文件/etc/my.cnf
sudo vim /etc/my.cnf
# 增加如下配置
#数据库id
server-id = 1
#启动binlog，该参数的值会作为binlog的文件名
log-bin=mysql-bin
#binlog类型，maxwell要求为row类型
binlog_format=row
#启用binlog的数据库，需根据实际情况作出修改
binlog-do-db=gmall

# 重启MySQL服务
sudo systemctl restart mysqld
```
> [!NOTE] MySQL Binlog模式
> Statement-based：基于语句，Binlog会记录所有写操作的SQL语句，包括insert、update、delete等。
> 优点：节省空间
> 缺点：有可能造成数据不一致，例如insert语句中包含now()函数。
> Row-based：基于行，Binlog会记录每次写操作后被操作行记录的变化。
> 优点：保持数据的绝对一致性。
> 缺点：占用较大空间。
> mixed：混合模式，默认是Statement-based，如果SQL语句可能导致数据不一致，就自动切换到Row-based。
> Maxwell要求Binlog采用Row-based模式。
### 4、创建Maxwell所素数据库和用户
```sql
-- 创建数据库
CREATE DATABASE maxwell;
-- 创建Maxwell用户并赋予其必要权限
CREATE USER 'maxwell'@'%' IDENTIFIED BY 'maxwell';
GRANT ALL ON maxwell.* TO 'maxwell'@'%';
GRANT SELECT, REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'maxwell'@'%';
```
### 5、配置Maxwell
```shell
# 修改Maxwell配置文件名称
cp config.properties.example config.properties

# 修改Maxwell配置文件
vim config.properties

#Maxwell数据发送目的地，可选配置有stdout|file|kafka|kinesis|pubsub|sqs|rabbitmq|redis
producer=kafka
# 目标Kafka集群地址
kafka.bootstrap.servers=hadoop1:9092,hadoop2:9092,hadoop3:9092
#目标Kafka topic，可静态配置，例如:maxwell，也可动态配置，例如：%{database}_%{table}
kafka_topic=topic_db

# MySQL相关配置
host=hadoop1
user=maxwell
password=maxwell
jdbc_options=useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true

# 过滤gmall中的z_log表数据，该表是日志数据的备份，无须采集
filter=exclude:gmall.z_log
# 指定数据按照主键分组进入Kafka不同分区，避免数据倾斜
producer_partition_by=primary_key
```

### 5、Maxwell启停
#### 启动停止Maxwell
```shell
# 启动
/opt/module/maxwell/bin/maxwell --config /opt/module/maxwell/config.properties --daemon
# 停止
ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
```
#### Maxwell启停脚本
```shell
#!/bin/bash

MAXWELL_HOME=/opt/module/maxwell

status_maxwell(){
    result=`ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep| wc -l`
    return $result
}


start_maxwell(){
    status_maxwell
    if [[ $? -lt 1 ]]; then
        echo "启动Maxwell"
        $MAXWELL_HOME/bin/maxwell --config $MAXWELL_HOME/config.properties --daemon
    else
        echo "Maxwell正在运行"
    fi
}


stop_maxwell(){
    status_maxwell
    if [[ $? -gt 0 ]]; then
        echo "停止Maxwell"
        ps -ef | grep com.zendesk.maxwell.Maxwell | grep -v grep | awk '{print $2}' | xargs kill -9
    else
        echo "Maxwell未在运行"
    fi
}


case $1 in
    start )
        start_maxwell
    ;;
    stop )
        stop_maxwell
    ;;
    restart )
       stop_maxwell
       start_maxwell
    ;;
esac

# 赋予权限
chmod 777 mxw.sh
```