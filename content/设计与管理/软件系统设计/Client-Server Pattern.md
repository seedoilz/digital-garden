---
aliases: [客户端-服务端模式]
date created: 四月 26日 2023, 4:51:31 下午
date modified: 三月 5日 2024, 4:07:11 下午
title: Client-Server Pattern
tags: [system-architecture/pattern]
---

1. 包含两类不同的 component
2. 请求发起 client、server 接收请求，这里没有 broker，不能动态改变 client 和 server 的关系，相对更固定，但是一个 client 可以连接多个 server
3. 一个 component 在一个关系中可以是 client，也可能是 server，非绝对，但是成对的关系相对固定。
4. 会受到负载的限制。
5. Server 可能有性能瓶颈，但是可以通过事先规划避免。
6. Server 可能单点失效，但是 broker 可以控制

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/12.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/13.png)
