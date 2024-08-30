---
title: Cortex-M4 SoC设计实战-系统低功耗设计
date: 2024-07-18 09:39:40
categories:
- 基于Cortex-M的SoC设计
- Cortex-M4 SoC设计实战
tags:
- low power
- TBD
---

> 基于Cortex-M4的系统低功耗设计：
>
> 注意进入VLPS mode以及Standby mode的时候，除了关闭时钟和掉电之外，工作电压也从1.1V降低到0.9V

# 电源域以及低功耗模式

{% asset_img image-20240729172635279.png %}

- 数字部分有4个电源域：PD_VDD/TOP_VDD/AO_VDD/FLASH

- 有3种系统低功耗模式：RUN/VLPS/STANDBY

> 当处于VLPS模式的时候，此时唤醒的机制是使用WIC；
>
> 当处于STANDBY模式的时候，此时唤醒的机制是使用外部的PMU；
>
> <font color=blue>WIC是放在TOP power  domain</font>
>
> 思考：
>
> <font color=red>电源情况？时钟情况？复位情况？</font>

# VLPS模式下的低功耗唤醒机制

## 软件流程

0、使能中断唤醒源（有哪些唤醒源是由什么决定？？）

1、配置PMU寄存器，使能WIC

2、配置DEEPSLEEP，执行WFI

3、中断唤醒并处理

> 思考：
>
> <font color=red>有哪些唤醒源？</font>
>
> 第一点是看，在进入低功耗的时候，该模块是否是active；
>
> 第二点是看，中断是否使能；



## 硬件实现

### WIC handshake

{% asset_img image-20240729201435271.png%}

{% asset_img image-20240729201549271.png%}

### 时序图

{% asset_img  image-20240730113951925.png %}

> Q&A:
> 1、1==》2的过程
> 在这两个动作之间，PMU有做其他操作吗？PMU和各个外设的REQ/ACK是在什么阶段做的？
> 如果在这个ACK期间，有中断到来，是什么行为？会进入唤醒流程吗？
> ==》在这两个动作之间，PMU不会有其他操作
> ==》PMU和各个外设的sleep_req和sleep_ack是在PMU接收到DEEPSLEEPING之后做的；
> ==》如果在进入低功耗mode的过程中，有中断到来，其行为取决于这时候PMU的state machine在哪一个状态：
>
> - 有可能直接唤醒，
> - 也有可能会先忽略这个中断，然后等到进入睡眠状态之后再去处理中断
>
> 2、4==》5的过程？
> PMU接收到WAKEUP，内部状态机跳转进入唤醒流程，de-assert WICENREQ
> 3、5==》6的过程？
> PMU内部状态机跳转，active FCLK
>
> 4、时钟和WICDSREQ怎么释放的？
> ==》WICDSREQ怎么释放？
> 来自PMU的WICENREQ deassert之后，理论上WICDSREQn就应该同样de-assert，但是这时候没有时钟，需要等待FCLK时钟到来才能完成这个动作
> FCLK的时钟
> ==》时钟怎么释放？
> 在PMU那边控制；
>
> 5、SLEEP如何拉低？
> CORE的行为

### WIC & PMU交互

{% asset_img  image-20240730114034786.png %}



# STANDBY模式下的低功耗唤醒机制

## 软件流程

0、使能中断唤醒源（有哪些唤醒源是由什么决定？？）

1、配置DEEPSLEEP，执行WFI；（WFI处于TOP电源域，在standby模式下，也会掉电）

2、中断唤醒并通过PMU处理；

## 硬件流程

{% asset_img  image-20240806153805333.png %}







































































