---
aliases: 
tags: [code/big-data, code/snippet]
title: core-site.xml
date created: 2024-03-17 19:03:00
date modified: 2024-03-20 11:03:16
---
```XML
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- 指定NameNode的地址 -->
  <property>
    <name>fs.defaultFS</name>
  <value>hdfs://hadoop102:8020</value>
  </property>
  <!-- 指定hadoop数据的存储目录 -->
  <property>
    <name>hadoop.tmp.dir</name>
  <value>/opt/module/hadoop-3.1.3/data</value>
  </property>
  <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
  <property>
    <name>hadoop.http.staticuser.user</name>
    <value>root</value>
  </property>
</configuration>
```