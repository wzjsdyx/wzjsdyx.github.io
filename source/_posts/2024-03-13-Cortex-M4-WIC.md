---
title: Cortex-M4 WIC
date: 2024-03-13 17:09:15
categories:
- Cortex-M4
tags:
- TBD
---

> 因为NVIC工作功耗较高，使用WIC就可以将NVIC的时钟或者电源关闭

# WIC工作机制

## step1:enable WIC

`handshake结构`

{% asset_img image-20240523180934866.png %}

`handshake时序`

{% asset_img image-20240523181051633.png %}



## Step2:执行WFI进入低功耗模式(deepsleep)

{% asset_img image-20240523181235917.png %}

1)NVIC驱动WICLOAD进入deep sleep
WIC 从WICMASK[]输入接口 reload mask 配置，（最终能active的中断是WICSENSE[]）

2)Cortex-M4进入deep sleep

3)4)5)PMU对core关电

6)未屏蔽中断到来

7)WAKEUP通知PMU去唤醒
WICPEND与此同时pend到来的中断

8)9)10)PMU对core上电

11）

- core看到WIC的pending中断请求
我理解应该对pulse的中断会有作用，对于电平类型的中断没啥用；
因为外设的中断一直都在,WAKEUP通知PMU对core上电之后，仍然可以看到这个外部中断

- 清除WICSENSE[]，也就是WIC此时不会对中断进行响应

12）清除WIC pending中断请求，wakeup desserted



# WIC实现原理图

<font color=red>TBD：待补充</font>
