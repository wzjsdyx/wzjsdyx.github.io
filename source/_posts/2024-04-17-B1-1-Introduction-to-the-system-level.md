---
title: B1.1 Introduction to the system level
date: 2024-04-17 08:33:34
categories:
- ARMv7-M架构
tags:
- System Level Architecture
- TBD
---



> ARMv7-M架构是为了低成本或低性能的应用场景，其和正常的架构演化有些偏离（如果为了兼容或者移植，不用使用v7架构特有的feature）。
>
> 
>
> 在深度嵌入式系统中，尤其是在低成本或低性能的应用场景下，操作系统与应用程序之间可能没有明显的区分。这意味着同一个程序中既有“操作系统级”的管理代码，也有“应用级”的业务逻辑代码。



<font color=red>将PDF文档的内容总结</font>



# Deprecated Features in ARMv7-M

<font color=blue>在技术文档中，当提到“已废弃的特性（`Deprecated features`）”时，这意味着某些功能虽然在当前版本的架构中仍然存在，主要是为了兼容以前的系统或软件，但是这些特性在未来版本的架构中可能不再被支持</font>

## Deprecated architectural features

### Four-byte stack alignment

架构是否实现4Byte栈对齐由具体实现确定

### Context switch optimization by not stacking LR

### Setting both DWT_CTRL.PCSAMPLENA and DWT_CTRL.CYCEVTENA to 1



## Deprecated feature of the ARMv7-M Thumb instruction set  



<font color=red>TBD</font>
