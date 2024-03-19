---
aliases: [面向服务的模式]
date created: 四月 26日 2023, 4:54:17 下午
date modified: 三月 5日 2024, 4:07:11 下午
title: SOA
tags: [system-architecture/pattern]
---

面向服务的模式是 Broker Pattern 的延伸，Component 包含服务提供者、服务消费者、ESB (企业服务总线)、企业服务组件、连接处理 (注册、发现)，Connector 包含 SOAP、REST、Asynchronous messaging connector。

### 解决方案
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/16.png)

### 概述图
![](https://spricoder.oss-cn-shanghai.aliyuncs.com/2021-Software-System-Design/img/lec14/17.png)

### 面向服务的模式的连接器
1. SOAP (简单对象访问协议): 服务提供者和提供者通过通常在 HTTP 之上交换请求/答复 XML 消息进行交互。
2. REST (代表性状态传输协议): 服务使用者依赖于四个基本状态 (POST、GET、 PUT、DELETE)的 HTTP 请求的 REST
3. 异步消息传递 (即忘即发): 参与者不必等待确认。

### SOA 和其他架构的区别
1. SOA 具备 Broker 的优势 (而又不继承 Broker，因此 Broker 消失)。
2. SOA 具有更高的互操作性和更高的伸缩性。
3. 出现类似基础设施的组件 (代替单一节点 broker)，解决单点失效的问题，满足互联网普及、参与人数规模较大的问题。
4. 商业模式的变化与技术可用性: 技术条件下能够提供的服务越来越多，随着社会分工越来越细，服务也可以被拆分成更细、更大、更差异化、更细粒度，通过这种拆分可以提供差异化的服务，实现服务的动态绑定和替换; 而微服务也是类似。
