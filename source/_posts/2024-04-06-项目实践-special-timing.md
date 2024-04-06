---
title: 项目实践-special timing
date: 2024-04-06 15:58:39
categories:
- DC synthesis
tags:
- 时序约束
- special timing
- TBD
---

> 有些时序是DC SDC约束不到的，称为special timing，项目中通常分成两类：
>
> - skew
> - delay
>
> 涉及到PAD，analog等模块时，需要进行special timing的约束。

举例说明：

SPI会有SCLK, MISO, MOSI, CS信号

`skew`的含义就是，时钟边沿和信号边沿到PAD上时序要对齐，不然外部设备采样可能会出现问题；

`delay`的含义就是信号到PAD的延时，需要根据实际的波特率进行计算。



补充：

- JTAG/SWD时序约束

- Trace时序约束
