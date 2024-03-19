---
aliases: [发布-订阅模式]
date created: 四月 26日 2023, 4:53:16 下午
date modified: 三月 5日 2024, 4:07:11 下午
title: Publish-Subscribe Pattern
tags: [system-architecture/pattern]
---

Subscribe 对 Publish 进行注册，某个 Publisher 发布自己的消息可能订阅其他的消息 (朋友圈微博)。随着订阅的增加，可能会导致性能的延迟 (Subscriber 数量订阅越多性能下降)
传统操作系统使用事件驱动方式来管理。

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/18.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/19.png)