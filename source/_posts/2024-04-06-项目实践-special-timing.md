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

# delay固定的情况下，分析最大波特率

## Master 模式

{% asset_img image-20250807101009297.png %}

> Master模式
> 下最大波特率受RX限制，Master需要正确采到slave发来的数据
> T1 + T2 + T3 + Tfclk*2  < Tclock_out/2 + T4
> 不考虑延时的情况下，单纯从设计的角度考虑， Tfclk*4 < Tclock_out  ，PLL没有5分频，因此最小6分频
> 考虑延时的情况相爱，最大同样为为6分频

{% asset_img image-20250807101045479.png %}

## Slave 模式

{% asset_img image-20250807101114808.png %}

> Slave模式
> 下最大波特率受TX限制，Slave需要保证发送的数据能让master采到
> T1 + T2 + T3 + Tfclk*3(2sync+1edge det) + T4 < Tclock_in/2
> PLL没有奇数分频最大为8分频

{% asset_img image-20250807101159633.png %}

# 设计上，优化时序的方法

## 将外部时钟作为data的方式

{% asset_img image-20250807101627214.png %}

> 假设T3和T4已经包括了信号在slave芯片内部的延迟
> T1已经除去SCK到CLK_SP的延迟
>
> 
>
> 假设主从设备都是上升沿发送数据，下降沿采样数据
>
> 双方约定配置为
> CPHA = 1, CPOL = 0, LSB, 4bit传输，传输值为0x5（0101）
> 如下图，奇数cycle为发送沿，偶数cycle为采样沿
> 以下以第二个bit传输为例，3为发送沿，4为采样
>
> 几段delay是真实固定存在的：
>
> 如图中由于delay过大，用原本的采样点（蓝线）去采会采错（slave实际发送的是0，Master采样到1）
>
> 
>
> 时钟延时+数据延时（总延时）<Tsck/2
>
> special timing约束：
> T1+T2<Tsck/4
> MCU外部device各站一半的margin

{% asset_img image-20250807101832157.png %}

> 解决方法：（从设计上缓解Timing上的压力，但是仍然需要卡specila timing进行约束）
> 将采样点delay（绿线）一个apb clk后能正确采到数据

## 直接使用外部时钟的方式

针对I2S这种协议，<font color=blue>外部时钟一直是toggle的</font>，设计上比较好处理；

但是针对SPI这种，<font color=blue>只有数据传输的时候，外部时钟才是toggle</font>，设计上比较好处理























# special timing约束举例说明：

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

> special timing约束的时候，注意：
>
> - 时钟周期： 按照10MHz去约束；
> - `主机和从机`的`采样沿/发送沿`
> - 外部调试器的实际行为、仿真模型的实际行为；

### CHIP SWD作为RX

{% asset_img image-20250807110208514.png %}

{% asset_img image-20250807110214627.png %}



> `外部调试器`：下降沿发送数据
> 内部芯片：上升沿采样数据
> ==》有0.5T的时间
>
> `仿真模型`：下降沿发送数据??-TBD
> 内部芯片：上升沿采样数据
> ==》有0.5T的时间
>
> chip SWD read：
> 卡skew
> PA13_SWDCLK -> reg/CK ≈ PA10_SWDIO->reg/D

### CHIP SWD作为TX

{% asset_img image-20250807110401287.png %}

{% asset_img image-20250807110406178.png %}



> `内部芯片`：上升沿发送数据
> 外部调试器：上升沿采样
> ==》有1T的时间
>
> `仿真模型`：上升沿发送数据？？-TBD
> 外部调试器：上升沿采样
> ==》有1T的时间
>
> 
>
> chip SWD write：
> 卡skew：
> sw_data_reg_0_/Q->PAD ≈ swdoen_reg/Q->PAD
>
> 卡latency
> PA13_SWDCLK -> reg/CK + reg/D->PA10_SWDIO < 1T（实际）
> PA13_SWDCLK -> reg/CK + reg/D->PA10_SWDIO < 1T（严格）
> 实际是1T的delay；
> 这里约束的严格点，可以约束为0.5T

### special timing

```bash
#=====================================
# SWD releated
# PAD_PA13: SWDCLK
# PAD_PA14: SWDIO
#=====================================

# debugger  launch data on the falling edge of SWDCLK,  capture data from SWDIO on the rising edge of SWDCLK.
# chip      launch data on the  rising edge of SWDCLK,  capture data from SWDIO on the rising edge of SWDCLK.


#----------------
# chip as SWD RX
#----------------
REQUIREMENTS: the following signals should be placed and rounted to obtain the same delay path, skew between the following paths should be less than 3ns
-from PAD_PA13 -to MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/gen_swdi_sw_0_u_dp_sw_swdi_capture/iregdo_reg/CK
-from PAD_PA14 -to MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/gen_swdi_sw_0_u_dp_sw_swdi_capture/iregdo_reg/D

#----------------
# chip as SWD TX
#----------------
REQUIREMENTS : the following signals should be placed and routed to obtain the small delay, the total latency SWDCLK in + SWDO/SWDOE OUT requirement is less than 33ns
-from PAD_PA13 -to MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/gen_swdi_sw_0_u_dp_sw_swdi_capture/iregdo_reg/CK
-from MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/swdoen_reg/Q     -to PAD_PA14
-from MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/sw_data_reg_0_/Q -to PAD_PA14

REQUIREMENTS : the following signals should be placed and rounted to obtain the same delay path, skew between the following paths should be less than 3ns
-from MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/swdoen_reg/Q     -to PAD_PA14        # SWDO
-from MTCMOS_wrap/i_MTCMOS/cm0p_core_top/u_cm0pintegration/gen_dap_u_dap/u_dap_top/u_dp/gen_sw_dp_u_cm0p_dap_dp_sw/sw_data_reg_0_/Q -to PAD_PA14        # SWDO EN
```

## CPU Model接口

> - 时钟周期10MHz
>
> - `主机和从机`的`发送沿/接收沿`
> - 仿真模型和外部机台的行为

### 相关的PAD

{% asset_img image-20250807111416689.png %}



### CPUM Write

{% asset_img image-20250807111139410.png %}



> write
> 地址和数据全是输入，因此只需要卡skew，skew尽量小(3ns);
> PAD_CLK->register.CK
> PAD_DATA->register.D

### CPUM Read

{% asset_img image-20250807111159175.png %}



> read
> 地址是输入，数据是输出
> PAD_CLK->register.CK
> PAD_DATA->register.D;
>
> register.D->PAD_DATA;
> 需要卡latency
> 需要卡skew

### special timing

```bash
#=====================================
# CPUM0 releated
# PAD_PA14: cpum0_clk
# PAD_PA10: D3
# PAD_PA11: D2
# PAD_PC8 : D1
# PAD_PC9 : D0
#=====================================


#----------------
# chip as CPUM0 RX
#----------------
REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, skew is less than 3ns
#0> ck
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK

#1> addr+write_en
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/D
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PC8  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA11  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA10  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D

#2> wdata
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PC8  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA11 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA10 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D


REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, latency is less than 10ns
#0> ck
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK

#1> addr+write_en
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/D
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PC8  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA11  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA10  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D

#2> wdata
-from PAD_PC9  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PC8  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA11 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA10 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D


#----------------
# chip as CPUM0 TX
#----------------
REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, skew is less than 3ns
#0> dout+dout_en
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PA10
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PA11
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PC8 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PC9 

-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PA10
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PA11
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PC8 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PC9 

REQUIREMENTS : the following signals should be placed and routed to obtain the small delay, the delay (0 + 1) requirement is less than 25ns/
#0> ck
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/CK
-from PAD_PA14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/CK

#1> dout+dout_en
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PA10
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PA11
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PC8 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PC9 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PA10
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PA11
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PC8 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PC9 


#=====================================
# CPUM1 releated
# PAD_PB14: cpum1_clk
# PAD_PB4: D3
# PAD_PB5: D2
# PAD_PA4: D1
# PAD_PA5: D0
#=====================================

#----------------
# chip as CPUM1 RX
#----------------
REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, skew is less than 3ns
#0> ck
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK

#1> addr+write_en
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/D
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PB5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PB4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D

#2> wdata
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PB5 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PB4 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D


REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, latency is less than 10ns
#0> ck
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK

#1> addr+write_en
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/D
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PA4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PB5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D
-from PAD_PB4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/D

#2> wdata
-from PAD_PA5  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PA4  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PB5 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D
-from PAD_PB4 -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/D



#----------------
# chip as CPUM1 TX
#----------------
REQUIREMENTS: the following signals should be placed as near as possible and to obtain the same delay path, skew is less than 3ns
#0> dout+dout_en
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PB4
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PB5
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PA4 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to PAD_PA5 

-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PB4
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PB5
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PA4 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to PAD_PA5 

REQUIREMENTS : the following signals should be placed and routed to obtain the small delay, the delay (0 + 1) requirement is less than 25ns/
#0> ck
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/cpum_write_reg/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_addr_reg_*/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/test_wdata_reg_*/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/CK
-from PAD_PB14  -to MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/CK

#1> dout+dout_en
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PB4
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PB5
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PA4 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/dout_reg_*_/Q -to  PAD_PA5 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PB4
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PB5
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PA4 
-from MTCMOS_wrap/i_MTCMOS/cm0_MTCMOS/cm0_infra_ctl/cpumif/TD_en_reg/Q -to  PAD_PA5 
```



