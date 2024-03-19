---
aliases: 
title: hdfs-site.xml
date created: 三月 17日 2024, 7:16:29 晚上
date modified: 三月 17日 2024, 7:17:13 晚上
tags: [code/big-data, code/snippet]
---
```XML
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- nn web端访问地址-->
  <property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop102:9870</value>
  </property>
  <!-- 2nn web端访问地址-->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop104:9868</value>
  </property>
</configuration>
```