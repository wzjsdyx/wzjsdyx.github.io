---
title: AHB-Lite lock传输
date: 2024-12-09 10:46:52
categories:
- AHB-Lite总线
tags:
- lock传输
- TBD
---



是由Slave去实现lock传输功能；



多个Master同时访问同一个share memory的时候，一般会实现此功能，防止数据不同步；

>  典型的使用场景是多个Master访问信号量
>
> 处理器使用SWP指令



Q&A:

indivisiable不可分割怎么理解？
