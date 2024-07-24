---
title: 5 Debug integration with Cortex-M processor systems（1）
date: 2024-06-28 17:13:03
categories:
- 基于Cortex-M的SoC设计
tags:
- debug integration 
- TBD
---

# Overview of debug and trace features  



































大多数的Cortex-M系统，都会集成debug和trace特性，不仅方便软件开发做调试，也对flash program调试和诊断数据的生成很有用。

debug功能有如下特性：

- 访问memory space。处理器正在运行的时候，debug也能访问
- 断点事件可以用于halt处理器
  - 硬件断点：使用的是硬件比较器去比较PC和断点地址，硬件断点的数量是有限的；
  - 软件断点：使用BKPT指令去触发断点中断，然后通过debug monitor进行捕获；





> 思考：
>
> <font color=red>软件断点的工作机制？</font>



























