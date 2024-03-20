---
aliases: 
title: MapReduce
date created: 三月 12日 2024, 1:58:54 下午
date modified: 三月 19日 2024, 9:44:26 晚上
tags: [code/big-data]
---
## MapReduce概述
### 过程
MapReduce 将计算过程分为两个阶段：Map 和Reduce
1. Map 阶段并行处理输入数据
2. Reduce 阶段对 Map 结果进行汇总
### 缺点
1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长 DAG（有向无环图）计算

### MapReduce 进程
1. MrAppMaster：负责整个程序的过程调度及状态协调。
2. MapTask：负责Map 阶段的整个数据处理流程。
3. ReduceTask：负责Reduce 阶段的整个数据处理流程。