---
aliases: 
title: UDP
date created: 2024-08-17 15:08:00
date modified: 2024-08-17 15:08:84
---

主要讲述 TCP 和 UDP 的区别

总结
1. TCP是面向连接的，UDP是无连接的
2. TCP是可靠的，UDP是不可靠的
3. TCP是面向字节流的，UDP是面向数据报文的
4. TCP只支持点对点通信，UDP支持一对一，一对多，多对多
5. TCP报文首部20个字节，UDP首部8个字节
6. TCP有拥塞控制机制，UDP没有
7. TCP协议下双方发送接受缓冲区都有，UDP并无实际意义上的发送缓冲区，但是存在接受缓冲区