---
aliases: 
title: Flink Window
date created: 2024-04-26 11:04:00
date modified: 2024-07-16 20:07:49
tags: [code/big-data]
---
## 概念
流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而 window 是一种切割无限数据为有限块进行处理的手段。
所以 **Window 是无限数据流处理的核心**，Window 将一个无限的 stream 拆分成有限大小的”buckets”桶，我们可以在这些桶上做计算操作。

### 划分窗口的方式
- 根据时间进行截取(time-driven-window)，比如每 1 分钟统计一次或每 10 分钟统计一次。
- 根据数据进行截取(data-driven-window)，比如每 5 个数据统计一次或每 50 个数据统计一次。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-33-24.png)

### 窗口类型
#### 滚动窗口(Tumbling Windows)
将数据依据固定的窗口长度对数据进行切片。 
特点:时间对齐，窗口长度固定，没有重叠。 
滚动窗口分配器将每个元素分配到一个指定窗口大小的窗口中，滚动窗口有一个固定的大小，并且不会出现重叠。
使用场景：适合做 BI 统计等（做每个时间段的聚合计算）
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-39-23.png)
#### 滑动窗口（Sliding Windows）
滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的窗口长度和滑动间隔组成。
特点：时间对齐，窗口长度固定，可以有重叠。
滑动窗口分配器将元素分配到固定长度的窗口中，与滚动窗口类似，窗口的大小由窗口大小参数来配置，另一个窗口滑动参数控制滑动窗口开始的频率。
适用场景：对最近一个时间段内的统计（求某接口最近 5min 的失败率来决定是否要报警）。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-40-54.png)

#### 会话窗口（Session Windows）
由一系列事件组合一个指定时间长度的 timeout 间隙组成，类似于 web 应用的session，也就是一段时间没有接收到新数据就会生成新的窗口。
特点：**时间无对齐**。
当它在一个固定的时间周期内不再收到元素，即非活动间隔产生，那个这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-42-52.png)
