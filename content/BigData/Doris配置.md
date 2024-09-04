---
aliases: 
title: Doris配置
date created: 2024-05-03 14:05:00
date modified: 2024-06-05 13:06:46
tags: [code/big-data, code/database]
---

| hadoop1 | hadoop2 | hadoop3 |
| --------------- | ----------------- | ----------------- |
| FE(LEADER)      | FE(FOLLOWER)      | FE(FOLLOWER)      |
| BE              | BE                | BE                |
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]

## 集群搭建具体步骤
### 0、设置操作系统
```shell
# 操作系统安装要求
# 设置系统最大打开文件句柄数(注意这里的*不要去掉)
sudo vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
# 设置最大虚拟块的大小
sudo vim /etc/sysctl.conf
vm.max_map_count=2000000
# 重启生效
```
### 1、下载安装包
```shell
wget https://doris.apache.org/download/...
```
> [!NOTE] avx2
> Contentx86_64架构 cpu（intel，amd),执行命令：
> cat /proc/cpuinfo | grep avx2
> 如果能看到avx2 字样选择带 avx2 的包，否则选择不带 avx2

### 2、解压
```shell
mkdir -p /opt/module/doris
tar -xvf apache-doris-fe-1.2.4.1-bin-arm.tar.xz -C /opt/module/doris
mv /opt/module/doris/apache-doris-fe-1.2.4.1-bin-arm /opt/module/doris/fe
tar -xvf apache-doris-be-1.2.4.1-bin-arm.tar.xz -C /opt/module/doris
mv /opt/module/doris/apache-doris-be-1.2.4.1-bin-arm /opt/module/doris/be
tar -xvf apache-doris-dependencies-1.2.4.1-bin-arm.tar.xz -C /opt/module/doris
mv /opt/module/doris/apache-doris-dependencies-1.2.4.1-bin-arm /opt/module/doris/dependencies
cp /opt/module/doris/dependencies/java-udf-jar-with-dependencies.jar /opt/module/doris/be/lib
```

### 3、配置FE（前端）
#### 修改配置文件
```shell
vim /opt/module/doris/fe/conf/fe.conf

# web 页面访问端口
http_port = 7030
# 配置文件中指定元数据路径：默认在 fe 的根目录下，可以不配
# meta_dir = /opt/module/doris/fe/doris-meta
# 修改绑定 ip
priority_networks = 192.168.36.102/24
```

#### 启动FE
```shell
/opt/module/doris/fe/bin/start_fe.sh --daemon
```

#### 登录FE Web页面
地址: http://hadoop1:7030/login
用户:root
密码:无

### 4、配置BE（后端）
#### 修改配置文件
```shell
vim /opt/module/doris/be/conf/be.conf

webserver_port = 7040
# 不配置存储目录， 则会使用默认的存储目录
storage_root_path = /opt/module/doris/doris-storage1;/opt/module/doris/doris-storage2.SSD,10
priority_networks = 192.168.36.0/24

mem_limit=40%
```

#### 使用 Mysql 客户端连接到 FE
```shell
# 没写错 放心用
mysql -hhadoop1 -P9030 -uroot
# 设置密码
SET PASSWORD FOR 'root' = PASSWORD('aaaaaa');
```

#### 启动BE
```shell
/opt/module/doris/be/bin/start_be.sh --daemon
```

#### 查询BE状态
```shell
mysql -h hadoop1 -P 9030 -uroot -paaaaaa
show proc '/backends';
```


### 扩容
#### FE扩容
##### 修改配置
```sql
-- 在hadoop1中mysql执行 mysql -h hadoop1 -P 9030 -uroot -paaaaaa
ALTER SYSTEM ADD OBSERVER "hadoop2:9010";
ALTER SYSTEM ADD OBSERVER "hadoop3:9010";
```
##### 分发FE
```shell
xsync /opt/module/doris/fe
rm -rf /opt/module/doris/fe/doris-meta/*
```
##### 启动FE
###### 在 hadoop2 和 hadoop3 启动 FE
>第一次启动时，启动命令需要添加参 --helper leader主机: edit_log_port

###### 分别在 hadoop2 和 hadoop3 执行:
```shell
/opt/module/doris/fe/bin/start_fe.sh --daemon --helper hadoop1:9010
```
###### 在 mysql 客户端查看 FE 状态
```sql
show proc '/frontends'\G;
```

#### BE扩容
##### 修改配置
```sql
ALTER SYSTEM ADD BACKEND "hadoop2:9050";
ALTER SYSTEM ADD BACKEND "hadoop3:9050";
```
##### 启动BE
```shell
# 分别在三个节点执行如下命令
/opt/module/doris/be/bin/start_be.sh --daemon
```
##### 查询BE状态
```sql
SHOW PROC '/backends'\G
```

### 缩容
#### FE缩容
```sql
ALTER SYSTEM DROP FOLLOWER[OBSERVER] "fe_host:edit_log_port";
```
> [!warning] 注意
> 注意：删除 Follower FE 时，确保最终剩余的 Follower（包括 Leader）节点为奇数

#### BE缩容
##### DECOMMISSION 方式删除BE节点（推荐）
```sql
ALTER SYSTEM DECOMMISSION BACKEND "be_host:be_heartbeat_service_port";
```
> [!NOTE] 安全删除
> 该命令用于安全删除BE节点。命令下发后，Doris 会尝试将该BE上的数据向其他BE节点迁移，当所有数据都迁移完成后，Doris会自动删除该节点。
> 该命令是一个异步操作。执行后，可以通过 SHOW PROC '/backends'; 看到该 BE 节点的isDecommission状态为true。表示该节点正在进行下线。
> 该命令不一定执行成功。比如剩余BE存储空间不足以容纳下线BE上的数据，或者剩余机器数量不满足最小副本数时，该命令都无法完成，并且BE会一直处于 isDecommission为true的状态。
> DECOMMISSION的进度，可以通过SHOW PROC '/backends'; 中的TabletNum查看，如果正在进行，TabletNum将不断减少。
> 该操作可以通过如下命令取消：
> CANCEL DECOMMISSION BACKEND "be_host:be_heartbeat_service_port";

##### DROP方式删除BE节点（不推荐）
```sql
ALTER SYSTEM DROP BACKEND "be_host:be_heartbeat_service_port";
```

### 群起脚本
```shell
#!/bin/bash
case $1 in
    "start")
        for host in hadoop1 hadoop2 hadoop3 ; do
            echo "========== 在 $host 上启动 fe  ========="
            ssh $host "source /etc/profile; /opt/module/doris/fe/bin/start_fe.sh --daemon"
        done
        for host in hadoop1 hadoop2 hadoop3 ; do
            echo "========== 在 $host 上启动 be  ========="
            ssh $host "source /etc/profile; /opt/module/doris/be/bin/start_be.sh --daemon"
        done

       ;;
    "stop")
            for host in hadoop1 hadoop2 hadoop3 ; do
                echo "========== 在 $host 上停止 fe  ========="
                ssh $host "source /etc/profile; /opt/module/doris/fe/bin/stop_fe.sh "
            done
            for host in hadoop1 hadoop2 hadoop3 ; do
                echo "========== 在 $host 上停止 be  ========="
                ssh $host "source /etc/profile; /opt/module/doris/be/bin/stop_be.sh "
            done

           ;;

    *)
        echo "你启动的姿势不对"
        echo "  start   启动doris集群"
        echo "  stop    停止stop集群"

    ;;
esac
```