---
title: Cortex-M4 Hardfault loop
date: 2026-07-30 10:17:05
categories:
- ARM
- Cortex-M4
tags:
- TBD
---



# Hardfault loop

针对ARM Cortex-M系类的处理器：有些时候Hardfault发生一次（hardfault once），有些时候Hardfault发生多次（hardfault loop）



 <font color=blue>通常的可能的原因是Bus上发生的Fault导致的（即BusFault）</font>

当发生错误时，在BusFault中读取CFSR.BusFault字段；
判断是imprecise error还是 Precise error;

- if PRECISERR bit set, such that `the PC value stacked for the exception return points to the instruction that caused the fault.`
-  IMPRECISERR bit set, where `the return address in the stack frame is not related to the instruction that caused the error` (and thus could continuing on the next instruction, so no repeated fault is triggered).可能是当前出错指令的下条指令，也可能是之后的指令，没法确定；















## Q&A

### “Strongly-ordered 不允许缓冲”与“M7 进入 Store Buffer”为什么不矛盾？



















<font color=red>strongly-order和Device的区别？是不是不同的架构定义的不一样？</font>



<font color=red>不管是Device还是strongly-order，Armv6/v7/v8-M架构均未定义不能进入CPU内部的Buffer？要看处理器的具体实现；？</font>





<font color=red>不同层级的缓冲机制？CPU内部的微架构缓冲？外部的缓冲？</font>



<font color=red>store buffer和write的区别？</font>





<font color=red>帮我明确下，CM0+,CM4，CM7是否有write buffer？</font>

| 处理器          | 架构     | 内部实现及故障行为                                           |
| --------------- | -------- | ------------------------------------------------------------ |
| Cortex-M0+ r0p0 | Armv6-M  | 处理器总线接口无 Write Buffer；访问按简单顺序模型完成        |
| Cortex-M4       | Armv7E-M | 有写缓冲机制；Strongly-ordered 写 Bus Error 为 precise       |
| Cortex-M7       | Armv7E-M | 四入口 Store Buffer；SO/Device 写可进入但不得合并；SO 写错误可能为 imprecise |



















# TBD

<font color=red>1、存在两个不同层次的“bufferable”。</font>

## 内存属性中的 Bufferable

它规定这笔访问在架构上的处理规则，例如：

- 能否进行某些写合并；
- 是否允许以较宽松的方式传播；
- 访问之间必须遵守什么顺序；
- 在什么操作前必须完成。

## CPU 内部 Store Buffer

这是处理器的微架构队列，用于暂存：

```
地址
写数据
字节选通
内存属性
事务状态
```

进入内部 Store Buffer 不代表处理器可以随意：

- 合并这笔 Strongly-ordered 写；
- 把后面的受约束访问越过它；
- 改变其对外可观察顺序。

M7 TRM 明确说明，Store Buffer 中的 Device/Strongly-ordered 写不会合并，并会按相应规则主动排空。



# 参考文档

1. Using Cortex-M3/M4/M7 Fault Exceptions, from ARM
