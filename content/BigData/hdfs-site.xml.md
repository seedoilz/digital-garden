---
aliases: 
title: hdfs-site.xml
date created: 2024-03-17 19:03:00
date modified: 2024-04-09 15:04:27
tags: [code/big-data, code/snippet]
---
```XML
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <!-- nn web端访问地址-->
  <property>
    <name>dfs.namenode.http-address</name>
    <value>hadoop1:9870</value>
  </property>
  <!-- 2nn web端访问地址-->
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop103:9868</value>
  </property>
</configuration>
```