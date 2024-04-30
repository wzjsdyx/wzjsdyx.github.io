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



# skew和delay

`skew`就是多个信号到PAD上的时钟偏斜

`delay`的含义就是信号到PAD的延时

# 举例说明：

## SPI接口

会有SCLK, MISO, MOSI, CS信号

假设内部上升沿发送，外部下降沿接收

`skew要求`：时钟边沿和信号边沿到PAD上时序要对齐，不然外部设备采样可能会出现问题；

`delay要求`：

## Trace接口

会有Trace_clk以及Trace data，外部设备上升沿接收数据

`skew要求`：clk要在data中间，保证外部时钟采样数据正确；

`delay要求`：无delay要求

{% asset_img image-20240419143152809.png %}



##  JTAG接口

CDC check report会有Error

错误如下：

{% asset_img image-20240419144437661.png %}

代码追踪：

{% asset_img img_v3_02a3_14bfe2d7-14a0-4306-bb92-239c72d7b0ag.jpg %}

PA4是TMS端口，TCK是已经进行了时钟约束；

CDC在做check的时候，会认为PAD上的TMS和TCK不是同一个domain

<font color=blue>实际解决方法就是做special timing约束，保证TCK能够采样到TMS即可</font>



master和slave规定好：

- 从上升沿还是下降沿采样

- 从PAD上送进来的数据信号的时序



JTAG内部是上升沿采样，外部数据送进来会保证tck和tdi的相位正确，即TCK能采样到TDI，这样

`skew要求`：

{% asset_img image-20240419155458371.png %}



`delay要求`：

<font color=red>TBD</font>

## SWD接口

