---
aliases: 
title: MapReduce
date created: 2024-03-12 13:03:00
date modified: 2024-03-24 09:03:04
tags:
  - code/big-data
  - "#input"
---
## MapReduce概述
### 过程
MapReduce 将计算过程分为两个阶段：Map和Reduce
1. Map 阶段并行处理输入数据
2. Reduce 阶段对 Map 结果进行汇总
### 缺点
1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长 DAG（有向无环图）计算

### MapReduce 进程
1. MrAppMaster：负责整个程序的过程调度及状态协调。
2. MapTask：负责 Map 阶段的整个数据处理流程。
3. ReduceTask：负责 Reduce 阶段的整个数据处理流程。

## [[MapReduce框架原理]]

