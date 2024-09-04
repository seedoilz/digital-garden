---
aliases: 
title: Channel
date created: 2024-08-17 11:08:00
date modified: 2024-08-17 11:08:05
---
在标准的 IO 当中，都是基于字节流 / 字符流进行操作的，而在 NIO 中则是是基于 Channel 和 Buffer 进行操作，其中的 Channel 的虽然模拟了流的概念，实则大不相同。

|区别|Stream|Channel|
|---|---|---|
|支持异步|不支持|支持|
|是否可双向传输数据|不能，只能单向|可以，既可以从通道读取数据，也可以向通道写入数据|
|是否结合 Buffer 使用|不|必须结合 Buffer 使用|
|性能|较低|较高|

Channel 用于在字节缓冲区和位于通道另一侧的实体（通常是文件或者套接字）之间以便有效的进行数据传输。借助通道，可以用最小的总开销来访问操作系统本身的 I/O 服务。
![image.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/20240817114348.png)
