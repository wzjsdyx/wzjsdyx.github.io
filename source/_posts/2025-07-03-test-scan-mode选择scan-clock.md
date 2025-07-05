---
title: test_scan_mode选择scan clock
date: 2025-07-03 19:53:34
categories:
- DFT可测性设计
tags:
- DFT
- TBD
---







0、时钟只要有负载，就需要插入scan chain，换言之，就需要scan clock；

1、针对clock_mux，如果其选择信号来自于寄存器，需要使用test_scan_mode进行控制（因为在launch测试向量的时候，会导致clock乱动）

2、针对clock mux需要create clock？

- 如果不create clock，那么每一个时钟都会去check timing，带来的影响就是：

> 算是过约
>
> 影响clock tree（在每个create clock的点，clock都会被打断）；（一个clock去驱动）

- 在clock mux输出点，即function clock，选择最快的时钟输入进行时钟约束，同时需要注意跨时钟domain的处理



# 示例1：

{% asset_img image-20250703195619990.png %}



> #LSI 128K
> create_generated_clock  -name lsi128k_clk_sel_clk \
>                         -master_clock [get_clocks lsi128k_clk] \
>                         -source [get_pins ana1_wrap_inst/ana1_wrap_u1/MCU_TOP/AD_MCU_LPOSC_OSCOUT] \
>                         -divide_by 1 \
>                         -add \
>                        [get_pins top_design_ctl/ckgen_inst/u_ckgen_div_top/dtc_lsi_clk_mux/dtc_ckmux4_1_3/Z]



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
