---
aliases: 
title: DataX原理
date created: 2024-04-25 20:04:00
date modified: 2024-04-25 20:04:44
tags: [code/big-data]
---
## 架构原理
### 设计理念
为了解决异构数据源同步问题，DataX将复杂的网状的同步链路变成了星型数据链路，DataX作为中间传输载体负责连接各种数据源。当需要接入一个新的数据源的时候，只需要将此数据源对接到DataX，便能跟已有的数据源做到无缝数据同步。
![CleanShot 2024-04-25 at 20.56.55.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-04-25%20at%2020.56.55.png)

### 框架设计
DataX 本身作为离线数据同步框架，采用 Framework + plugin 架构构建。将数据源读取和写入抽象成为 Reader/Writer 插件，纳入到整个同步框架中。
![CleanShot 2024-04-25 at 20.58.04.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-04-25%20at%2020.58.04.png)

### 运行流程
![CleanShot 2024-04-25 at 20.58.54.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-04-25%20at%2020.58.54.png)
