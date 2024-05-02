---
aliases: 
title: Watermark
date created: 2024-04-26 11:04:00
date modified: 2024-04-27 18:04:44
tags: [code/big-data]
---
## 基本概念
### 使用原因
流处理从事件产生，到流经 source，再到 operator，虽然大部分情况下，流到 operator 的数据都是按照事件产生的时间顺序来的，但是由于网络、分布式等原因，导致乱序的产生。所谓乱序，就是指 Flink 接收到的事件的先后顺序不是严格按照事件的 Event Time 顺序排列的。
一旦出现乱序，如果只根据 eventTime 决定 window 的运行，我们不能明确数据是否全部到位，但又不能无限期的等下去，可以把Watermark看作是一种告诉Flink一个消息延迟多少的方式。定义了什么时候不再等待更早的数据。
### 概念
- Watermark 是一种衡量 Event Time 进展的机制。
- Watermark 是用于处理乱序事件的，而正确的处理乱序事件，通常用 Watermark 机制结合 window 来实现。
- 可以把Watermark看作是一种告诉Flink一个消息延迟多少的方式。定义了什么时候不再等待更早的数据。
- 数据流中的 Watermark 用于表示 timestamp 小于 Watermark 的数据，都已经到达了，因此，window 的执行也是由 Watermark 触发的。
- Watermark 可以理解成一个延迟触发机制，我们可以设置 Watermark 的延时时长 t，每次系统会校验已经到达的数据中最大的 maxEventTime，然后认定 eventTime 小于 maxEventTime - t 的所有数据都已经到达，如果有窗口的停止时间等于 maxEventTime – t，那么这个窗口被触发执行。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/2024-04-26-11-57-34.png)
### 窗口触发条件
#### 对于out-of-order及正常的数据而言
- watermark的时间戳 > = window endTime
- 在 [window_start_time,window_end_time] 中有数据存在。
#### 对于late element太多的数据而言
- Event Time > watermark的时间戳
WaterMark相当于一个EndLine，一旦Watermarks大于了某个window的end_time，就意味着windows_end_time时间和WaterMark时间相同的窗口开始计算执行了。
就是说，我们**根据一定规则**，**计算出Watermarks**，并且**设置一些延迟**，**给迟到的数据一些机会**，也就是说正常来讲，对于迟到的数据，我只等你一段时间，再不来就没有机会了。

### WaterMark设定方法
#### 标点水位线(Punctuated Watermark)
标点水位线（Punctuated Watermark）通过数据流中某些特殊标记事件来触发新水位线的生成。这种方式下窗口的触发与时间无关，而是决定于何时收到标记事件。

在实际的生产中Punctuated方式在TPS很高的场景下会产生大量的Watermark在一定程度上对下游算子造成压力，所以只有在实时性要求非常高的场景才会选择Punctuated的方式进行Watermark的生成。

#### 定期水位线(Periodic Watermark)
周期性的（允许一定时间间隔或者达到一定的记录条数）产生一个Watermark。水位线提升的时间间隔是由用户设置的，在两次水位线提升时隔内会有一部分消息流入，用户可以根据这部分数据来计算出新的水位线。

在实际的生产中Periodic的方式必须结合时间和积累条数两个维度继续周期性产生Watermark，否则在极端情况下会有很大的延时。

### 迟到事件
虽说水位线表明着早于它的事件不应该再出现，但是上如上文所讲，接收到水位线以前的的消息是不可避免的，这就是所谓的迟到事件。实际上迟到事件是乱序事件的特例，和一般乱序事件不同的是它们的乱序程度超出了水位线的预计，导致窗口在它们到达之前已经关闭。

迟到事件出现时窗口已经关闭并产出了计算结果，因此处理的方法有3种：
- 重新激活已经关闭的窗口并重新计算以修正结果。
- 将迟到事件收集起来另外处理。
- 将迟到事件视为错误消息并丢弃。
Flink 默认的处理方式是第3种直接丢弃，其他两种方式分别使用 `Side Output` 和 `Allowed Lateness`。

`Side Output`机制可以将迟到事件单独放入一个数据流分支，这会作为 window 计算结果的副产品，以便用户获取并对其进行特殊处理。
`Allowed Lateness`机制允许用户设置一个允许的最大迟到时长。Flink 会在窗口关闭后一直保存窗口的状态直至超过允许迟到时长，这期间的迟到事件不会被丢弃，而是默认会触发窗口重新计算。因为保存窗口状态需要额外内存，并且如果窗口计算使用了 `ProcessWindowFunction` API 还可能使得每个迟到事件触发一次窗口的全量计算，代价比较大，所以允许迟到时长不宜设得太长，迟到事件也不宜过多，否则应该考虑降低水位线提高的速度或者调整算法。