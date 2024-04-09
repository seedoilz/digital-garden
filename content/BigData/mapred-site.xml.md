---
aliases: 
title: mapred-site.xml
date created: 2024-03-17 19:03:00
date modified: 2024-04-09 15:04:06
tags: [code/big-data, code/snippet]
---
```XML
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <!-- 指定MapReduce程序运行在 Yarn上 -->
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <!-- 历史服务器端地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop1:10020</value>
  </property>
  <!-- 历史服务器 web端地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop1:19888</value>
  </property>
  <property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/module/hadoop-3.1.3/etc/hadoop:/opt/module/hadoop-3.1.3/share/hadoop/common/lib/*:/opt/module/hadoop-3.1.3/share/hadoop/common/*:/opt/module/hadoop-3.1.3/share/hadoop/hdfs:/opt/module/hadoop-3.1.3/share/hadoop/hdfs/lib/*:/opt/module/hadoop-3.1.3/share/hadoop/hdfs/*:/opt/module/hadoop-3.1.3/share/hadoop/mapreduce/lib/*:/opt/module/hadoop-3.1.3/share/hadoop/mapreduce/*:/opt/module/hadoop-3.1.3/share/hadoop/yarn:/opt/module/hadoop-3.1.3/share/hadoop/yarn/lib/*:/opt/module/hadoop-3.1.3/share/hadoop/yarn/*</value>
  </property>
</configuration>

```