---
aliases: 
title: DataX配置
date created: 2024-04-25 20:04:00
date modified: 2024-04-25 21:04:16
tags: [code/big-data]
---
前期准备：
1. 准备三个虚拟机： `192.168.36.121 hadoop1` `192.168.36.122 hadoop2` `192.168.36.123 hadoop3`
2. 虚拟机上配置有`ssh`服务，可以进行[[集群SSH免密配置|集群免密登陆]]

> 注意：下边的步骤都是在`hadoop1`这个节点上进行的操作，除特殊说明外。

## 详细步骤
### 1、下载安装包
```shell
cd /opt/module
# 下载DataX安装包
wget http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz
```
### 2、解压安装包
```shell
tar -zxvf datax.tar.gz -C /opt/module/
# 可执行自检
python /opt/module/datax/bin/datax.py /opt/module/datax/job/job.json
```
