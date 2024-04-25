---
aliases: 
title: Flume集群搭建
date created: 2024-04-20 20:04:00
date modified: 2024-04-20 20:04:20
tags: [code/big-data]
---

前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]

> 注意：下边的步骤都是在`hadoop1`这个节点上进行的操作，除特殊说明外。

## 详细步骤
### 下载安装包
```shell
cd /opt/module
# 下载kafka安装包
wget https://archive.apache.org/dist/flume/1.10.1/apache-flume-1.10.1-bin.tar.gz
```

### 解压
```shell
# 解压flume安装包
tar -zxvf apache-flume-1.10.1-bin.tar.gz
mv apache-flume-1.10.1-bin flume
```

### 配置环境变量
```shell
vim ~/.bashrc
# 添加如下内容：
export PATH=$PATH:/opt/module/flume/bin
```

### 配置flume
```shell
# 进入conf目录
cd /opt/module/flume/conf
# 复制配置文件
cp flume-env.sh.template flume-env.sh
# 修改配置文件
vim flume-env.sh
# 添加Java环境变量
export JAVA_HOME=/opt/module/jdk1.8.0_212
```

### 检验安装
```shell
flume-ng version
```