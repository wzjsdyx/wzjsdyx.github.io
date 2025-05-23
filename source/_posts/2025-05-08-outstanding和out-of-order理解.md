---
title: outstanding和out-of-order理解
date: 2025-05-08 11:22:40
categories:
- ARM
- AMBA
- AXI
tags:
- outstanding
- out-of-order
---



outstanding的概念：

**支持多个超发事务（Multiple Outstanding）**

- **含义**：总线允许在未收到前一个事务的响应之前，连续发起多个新的事务请求。
- **优势**：提升总线利用率，避免因等待响应而空闲。例如，在访问高延迟设备（如内存）时，总线可以持续发送读写请求，无需阻塞。
- **实现**：<font color=blue>通过独立的地址/数据通道（如AXI的读/写地址、读/写数据通道）分离请求和响应，实现流水线化操作。</font>

 >举例说明：
 >
 >
 >
 >

Q&A：

outstanding和out-of-order的概念

outstanding能力是out-of-order的基础？

out-of-order的使用场景

ID在out-of-order的应用？



参考博客：

1. [AXI协议中Outstanding|Out of order|Interleave的区别和联系](https://www.cnblogs.com/Alfred-HOO/articles/17642770.html)
2. 
