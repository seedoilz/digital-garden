---
aliases: [分层模式,分层架构]
date created: 四月 26日 2023, 4:47:28 下午
date modified: 三月 5日 2024, 4:07:11 下午
title: Layered Pattern
tags: [system-architecture/pattern]
---
### 定义
层间访问必须按照逐层进行访问，核心是关注点分离。
影响的质量属性: 可修改性、可模块化、可维护性、可复用性。

![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/2.png)

### 实例
OSI 的七层网络模型

### 变体
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20230426161310.png)
>左侧不是分层模式，形成环形依赖 (**没有实现关注点分离**)，是软件的坏味道。
>右侧在 D 不期待 A 的结果且 D 不期待 B 的结果时是分层模式，但是在严格意义和其他场景中不是分层模式。
