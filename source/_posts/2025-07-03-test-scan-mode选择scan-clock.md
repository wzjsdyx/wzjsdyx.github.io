---
title: test_scan_mode选择scan clock
date: 2025-07-03 19:53:34
categories:
- DFT可测性设计
tags:
- DFT
- TBD
---



# 基础知识





时钟只要有负载，就需要插入scan chain，换言之，就需要scan clock；

0、DFT相关的PAD

{% asset_img image-20250806201827682.png %}



1、针对clock_mux，如果其选择信号来自于寄存器，需要使用test_scan_mode进行控制（因为在launch测试向量的时候，会导致clock乱动）



{% asset_img image-20250806195954895.png %}





> 1、test_se 连接外部PAD；即scan enable;用于控制shift和capture;
> 一般由工具自动trace并连接到test point上；
>
> - dft use to connect DFF SE PIN;
> - RTL not use;
> 一般由工具自动trace并连接到test point上
>
> 2、test mode 来自trapping；
> - 控制测试mode下的电压等；
> - 控制cm0p_reset_release
> - top_ckgen中切换到scan_clk;
> - 控制pinmux
>
> 3、dft会使用这个test mode信号吗？
> 会使用到这个信号，需要切换pinmux，控制PAD的功能为DFT相关；
>
> 4、为什么有些用test mode切换？有些用test scan mode切换？
> - 只有ATPG测试的时候，test scan mode才会拉高；
> - mbist测试的时候（test mode会拉高，test scan mode不会拉高）也需要使用DFT的时钟，使用test mode进行切换；
> - module内部寄存器如果只是为了ATPG测试的话，一般用test scan mode进行时钟切换，而不是test mode；（一般只有ckgen才会用test mode）
> 顺序为：test mode-》IJTAG-》TDR寄存器-》test_scan_mode在整个ATPG阶段一直为高电平；
>
> 5、test scan mode信号一般由DFT logic 生成





2、针对clock mux需要create clock？

- 如果不create clock，那么每一个时钟都会去check timing，带来的影响就是：

> 算是过约
>
> 影响clock tree（在每个create clock的点，clock都会被打断）；（一个clock去驱动）

- 在clock mux输出点，即function clock，选择最快的时钟输入进行时钟约束，同时需要注意跨时钟domain的处理



# 示例1：

{% asset_img image-20250703195619990.png %}



```bash
#LSI 128K
create_generated_clock  -name lsi128k_clk_sel_clk \
                     -master_clock [get_clocks lsi128k_clk] \
                     -source [get_pins ana1_wrap_inst/ana1_wrap_u1/MCU_TOP/AD_MCU_LPOSC_OSCOUT] \
                     -divide_by 1 \
                     -add \
                    [get_pins top_design_ctl/ckgen_inst/u_ckgen_div_top/dtc_lsi_clk_mux/dtc_ckmux4_1_3/Z]
```

> create_generated_clock：会生成一个新时钟，与一个已存在的主时钟 `master_clock` 有直接的相位和频率关系
>
> -master_clock：指定新生成的时钟 `lsi128k_clk_sel_clk` 的**参考主时钟**是 `lsi128k_clk`
>
> -source：新生成时钟的源点，（即master clock设置点--master clock get_pins设置的点）
>
> -divide_by：定义新时钟 `lsi128k_clk_sel_clk` 相对于主时钟 `lsi128k_clk` 的**分频比**
>
> -add：选项用于在同一个端口或引脚上添加多个时钟源。一般当需要为同一物理引脚定义多个时钟约束时，该选项允许指定附加的时钟源
>
> [get_pins xxx]：新时钟设置在哪一个对象上；



# 示例2：

{% asset_img image-20250704153915179.png %}

> #max sys clk
>
> create_generated_clock  -name wdg_fclk \
>                         -master_clock [get_clocks bus_clk] \
>                         -source [get_pins top_design_ctl/ckgen_inst/u_ckgen_div_top/u_ckgen_div/ahb_clk_div/DTC_div_cg/dtc_icgd2/Q] \
>                         -divide_by 1 \
>                         -add \
>                        [get_pins wdg_whole_top/wdt_top/wdt_cnt_core/wdt_clk_ckmux4/clk_sync_icg/dtc_icgd2/Q]
