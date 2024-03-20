---
aliases: 
date created: 2023-04-26 16:04:00
date modified: 2024-03-20 11:03:92
title: MVC Pattern
tags: [system-architecture/pattern]
---

使用运行时、动态、相互之间的关系来审视，集成到了开发框架中，也是[[Layered Pattern|分层架构]]的变种 (强调模块间约束关系，**model 不可以直接返回到 controller**)，分为 model (业务逻辑)、view (处理用户展示，接收用户操作)、controller (对用户操作进行处理，将信息通知给 model)

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/8.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/9.png)