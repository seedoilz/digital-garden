---
aliases: 
title: yarn-site.xml
date created: 2024-03-17 19:03:00
date modified: 2024-03-20 11:03:15
tags: [code/big-data, code/snippet]
---
```XML
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- 指定MR走 shuffle -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <!-- 指定ResourceManager的地址-->
  <property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop103</value>
  </property>
  <!-- 开启日志聚集功能 -->
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>
  <!-- 设置日志聚集服务器地址 -->
  <property>
    <name>yarn.log.server.url</name>
    <value>http://hadoop102:19888/jobhistory/logs</value>
  </property>
  <!-- 设置日志保留时间为 7天 -->
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
  </property>
  <!-- 环境变量的继承 -->
  <property>
	<name>yarn.nodemanager.env-whitelist</name>
	<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CO
NF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAP
RED_HOME</value>
  </property>
  <!-- 通过指令“hadoop classpath获得” -->
  <property>
    <name>yarn.application.classpath</name>
    <value>
    /opt/module/hadoop-3.1.3/etc/hadoop,
    /opt/module/hadoop-3.1.3/share/hadoop/common/*,
    /opt/module/hadoop-3.1.3/share/hadoop/common/lib/*,
    /opt/module/hadoop-3.1.3/share/hadoop/hdfs/*,
    /opt/module/hadoop-3.1.3/share/hadoop/hdfs/lib/*,
    /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/*,
    /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/lib/*,
    /opt/module/hadoop-3.1.3/share/hadoop/yarn/*,
    /opt/module/hadoop-3.1.3/share/hadoop/yarn/lib/*
    </value>
  </property>
</configuration>
```