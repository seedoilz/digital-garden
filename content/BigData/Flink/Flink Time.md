---
aliases: 
title: Flink Time
date created: 2024-04-26 11:04:00
date modified: 2024-04-26 11:04:71
tags: [code/big-data]
---
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-27-34.png)
- **Event Time**:是事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间，Flink 通过时间戳分配器访问事件时间戳。
- Ingestion Time:是数据进入 Flink 的时间。
- Processing Time:是每一个执行基于时间操作的算子的本地系统时间，与机器相关，默认的时间属性就是 Processing Time。

在 **Flink** 的流式处理中，绝大部分的业务都会使用 **eventTime**，一般只在eventTime 无法使用时，才会被迫使用 ProcessingTime 或者 IngestionTime。