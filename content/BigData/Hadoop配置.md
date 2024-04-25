---
aliases: 
title: Hadoop配置
date created: 2024-03-17 19:03:00
date modified: 2024-04-20 21:04:03
tags: [code/big-data]
---
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