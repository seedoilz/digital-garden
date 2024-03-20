---
aliases: [管道和过滤模式]
date created: 2023-04-26 16:04:00
date modified: 2024-03-20 11:03:92
title: Pipe-and-Filter Pattern
tags: [system-architecture/pattern]
---

Filter: 相当于 Component，起数据处理、计算作用，每个 Filter 有 input 和多个 output，将数据处理后传递给后续部分。
Pipe: 相当于 Connector，连接 filter，将 output 导入到其他的 filter 的 input 中去，不会独立存在。

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/10.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/11.png)
