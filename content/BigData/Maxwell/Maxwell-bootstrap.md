---
aliases: 
title: Maxwell-bootstrap
date created: 2024-04-25 18:04:00
date modified: 2024-04-25 19:04:04
tags: [code/big-data]
---
>我们可能需要使用到MySQL数据库中从历史至今的一个完整的数据集。这就需要我们在进行增量同步之前，先进行一次历史数据的**全量同步**。这样就能保证得到一个完整的数据集。

Maxwell*提供了bootstrap功能*来进行历史数据的全量同步，命令如下：
```shell
/opt/module/maxwell/bin/maxwell-bootstrap --database gmall --table user_info --config /opt/module/maxwell/config.properties
```
