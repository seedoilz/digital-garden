---
aliases: [代理模式, Broker]
date created: 2023-04-26 16:04:00
date modified: 2024-03-20 11:03:97
title: Broker Pattern
tags: [system-architecture/pattern]
---

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/6.png)

### 概述
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/7.png)

### 优缺点
#### 优点
1. Interoperability: 根本目的，提高 Server-Client 之间的交互性 
2. Scalability: 可伸缩和扩展
3. Modifiability
4. 两面性:
	1. Security: 代理对象屏蔽了系统内部的具体实现 
	2. Reliability: 服务降级和实例重启
	3. Availability:

#### 缺点
1. Security: 成为被攻击的对象
2. Reliability: 可靠性会降低
3. 两面性:
	1. Performance: 整体大集群的性能可能会提高 (QPS 等提高)，但是局部单点性能会下降，多次网络请求、多次匹配，有可能会抵消。