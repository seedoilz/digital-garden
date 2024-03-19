---
aliases: [点对点模式]
date created: 四月 26日 2023, 4:52:32 下午
date modified: 三月 5日 2024, 4:07:11 下午
title: Peer-to-Peer Pattern
tags: [system-architecture/pattern]
---

1. 这一刻是提供者，下一刻就是消费者，是对等的。
2. 不单单提供服务，还能提供物流 (对于整个网络)
3. 对每一个 peer 可能会给他一个规定对的连接数

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/14.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/15.png)

### 优缺点
安全性: 节点既是 Client 又是 Server，被攻击的可能性提高。
可用性: 数据分布在不同节点上，相同数据多处拷贝，可能导致数据不一致，但是能保证个别数据出现问题不影响整体。
性能: 多个节点同时提供服务，性能好 (多个渠道获取数据，并行能力提高)
