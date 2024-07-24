---
title: tsmc auto eflash test chip
date: 2024-07-15 13:32:28
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---

> tsmc eflash Macro and BIST/SMWR的集成和仿真环境
>
> tsmc 提供了demo的示例==》testchip & testchip simulaiton

# integration

{% asset_img image-20240716110046441.png%}







# simulation

## 操作流程简介

> parameter TCLK1_CYC=13 (half cycle of bist clock)==》bist_clk frequency=38.46MHz
>
> parameter TCK_CYC =15   (half cycle of TCK           ) ==》TCK       frequency=33.33MHz



1. 电源PAD以及测试PAD处理；
   1. power on：power  on:`power_on`=(VDD_i===1)&&(VDIO_i===1)&&(VSS_i===0)&&(PDM_i===0）
   2. wake up：model wake up，`wakeup_flag`
2. smwr reset  &bist enable
   1. 释放SMWR IP的rst_n，以及BIST IP的bist enable
   2. DR-1, R_IP_CONFIG, to setup BIST general informationand mode selection.
3. bist operation
   1. setup internal register
   2. issue command index




## bist_cp_test分析

1. 电源PAD以及测试PAD处理；
   1. power on：power  on:`power_on`=(VDD_i===1)&&(VDIO_i===1)&&(VSS_i===0)&&(PDM_i===0）
   2. wake up：model wake up，`wakeup_flag`

2. smwr reset  &bist enable

   1. 释放SMWR IP的rst_n，以及BIST IP的bist enable

   2. DR-1, R_IP_CONFIG, to setup BIST general informationand mode selection.

> 设置SEL1寄存器（bist相关的配置寄存器）
>
> 1. R_IPSEL：用于选择最多4个Macro的测试，每2bits控制一个Macro的测试；此处2’b11表示: enable IP0 test, repair on
> 2. R_CONFIG：这是一些bist所需要的参数，bist clock运行在20MH；
>    1. 设置>= 1us pulse需要的参数配置；
>    2. 设置>=20ns需要的参数配置；
>    3. [15:11]：Used to adjust Tmcyc and Tmacc for PV/EV read cycles during SMW operation；
> 3. R_TEST
> 4. R_DBG
> 5. R_BIST_CLK_SEL















---

参考文献：

1. [fdisplay系统函数的使用](https://www.cnblogs.com/SYoong/p/5897150.html)
