---
aliases: 
title: Maxwell原理
date created: 2024-04-25 18:04:00
date modified: 2024-04-25 18:04:66
tags: [code/big-data]
---
## Maxwell工作原理
很简单，就是将自己伪装成slave，并遵循MySQL主从复制的协议，从master同步数据。
Maxwell的工作原理是实时读取MySQL数据库的二进制日志（Binlog），从中获取变更数据，再将变更数据以JSON格式发送至Kafka等流处理平台。

### MySQL二进制日志
二进制日志（Binlog）是MySQL服务端非常重要的一种日志，它会保存MySQL数据库的所有数据变更记录。Binlog的主要作用包括主从复制和数据恢复。Maxwell的工作原理和主从复制密切相关。

### MySQL主从复制
MySQL的主从复制，就是用来建立一个和主数据库完全一样的数据库环境，这个数据库称为从数据库。
1. 主从复制的应用场景如下
	1. 做数据库的热备：主数据库服务器故障后，可切换到从数据库继续工作。
	2. 读写分离：主数据库只负责业务数据的写入操作，而多个从数据库只负责业务数据的查询工作，在读多写少场景下，可以提高数据库工作效率。
2. 主从复制的工作原理如下
	1. Master主库将数据变更记录，写到二进制日志（binary log）中
	2. Slave从库向mysql master发送dump协议，将master主库的binary log events拷贝到它的中继日志（relay log）
	3. Slave从库读取并回放中继日志中的事件，将改变的数据同步到自己的数据库