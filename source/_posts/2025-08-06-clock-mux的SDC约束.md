---
title: clock mux的SDC约束
date: 2025-08-06 20:16:36
categories:
- DC synthesis
tags:
- 时序约束
- PAD
- TBD
---



shi

示例

## 时钟架构图

{% asset_img image-20250806202137964.png %}

> 0、PAD无需special timing的约束；
>
> 1、RTC clk in需要create clock；
>
> 2、RTC CLK OUT是data属性还是clock属性？ 
> RTC是clock属性，因为外部有可能将这个PAD作为时钟，所以CHIP内部会作为时钟约束一下；
>
> 3、RTC_CLK_OUT无需再create clock，因为后面没有使用RTC_CLK_OUT作为时钟，进行驱动的逻辑；不需要考虑时钟balance的问题；因此这里不需要create clock；
>
> 4、如果func_clk不create clock？综合工具会如何综合？
> 首先func_clk也是时钟属性，并且RTC内部的寄存器需要和4个时钟源头做balance；（影响clock tree）
> 如果func_clk进行create generate clock，会对clock tree比较友好，此时只是RTC内部的寄存器和func_clk做balance即可，不用和4个时钟源头做balance；

## PAD_PA7结构

-TBD





## 时序约束

```bash
# STEP0: create PAD clock list
set io_clk_list "pinx_sel_clk1 pinx_sel_clk2 pinx_sel_clk3 monitor_ref_clk ef_TCLK cpu_jtag_clk cpum0_clk cpum1_clk rtc_clk_in"

# STEP1:create RTC clock && rtc_clk_list
set RTC_FCLK_HIERARCHY "u_standby_ctl_pwr_wrap/standby_ctl_u1/rtc_top/u_rtc_standby_ckmux4/dtc_ckmux4_1_3/Z"

create_clock -name rtc_clk_in -period 100 -waveform "0 50" \
             [get_ports PAD_PA7]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pad/gpio_in_buf_7__u_gpio_buf/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_cpum1_d/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_edt_update/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_pwm0_flt2/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_pwm3_ch3/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_scan_rst/dtc_ckbufd4/I]
set_sense -stop_propagation -clocks [get_clocks rtc_clk_in] [get_pins pin_top/imux_inst/u_pa7_trapping_d/dtc_ckbufd4/I]

create_generated_clock -name rtc_fclk -source [get_pins top_design_ctl/ckgen_inst/u_ckgen_div_top/u_ckgen_div/hse_clk_div1/o_clk_reg/Q] -master_clock [get_clocks hse_1_div2_clk] -div 1 -add \
                       [get_pins $RTC_FCLK_HIERARCHY]
		       
create_generated_clock -name rtc_psc_clk -source [get_pins $RTC_FCLK_HIERARCHY] -divide_by 2 -add -master_clock [get_clocks rtc_fclk] \
                       [get_pins u_standby_ctl_pwr_wrap/standby_ctl_u1/rtc_top/inst_rtc_cnt/toggle_psc_ov_reg/Q]

set rtc_clk_list "rtc_fclk rtc_psc_clk"

                       
                       
### whole_top level group_list
set_clock_groups -name ac7842e_clock_group -asynchronous \
    -group "$hse_clk_list" \
    -group "$vhsi_clk_list" \
    -group "$lsi_clk_list $acmp_abist_list" \
    -group "$pll_clk_list $sys_clk_list $out_clk_list $cmu_clk_list $flash_clk_list $spm_clk_list $uart_clk_list $acmp_clk_list $eio_clk_list $iic_clk_list $spi_clk_list $timer_clk_list $rtc_clk_list $wdt_clk_list" \
    -group "$pll_adc_list $can_clk_list $adc_clk_list" \
    -group "$spm_int_clk_list" \
    -group "$cm4_clk_list" \
    -group "$acmp_ana_list" \
    -group "$ctu_clk_list" \
    -group "$gpio_PA_clk_list" \
    -group "$gpio_PB_clk_list" \
    -group "$gpio_PC_clk_list" \
    -group "$gpio_PD_clk_list" \
    -group "$gpio_PE_clk_list" \
    -group "$io_clk_list"                       


set_clock_groups -name rtc_clk_group -asynchronous \
    -group bus_clk ahb_clk \
    -group rtc_fclk \
    -group rtc_psc_clk
```

> 说明：
>
> set_clock_groups后的group如果后面有子group，则按照子group的规则来；没有的话，则是按照同步clock进行约束；
>
> 
>
> create_generated_clock：会生成一个新时钟，与一个已存在的主时钟 `master_clock` 有直接的相位和频率关系
>
> -master_clock：指定新生成的时钟 `lsi128k_clk_sel_clk` 的**参考主时钟**是 `lsi128k_clk`
>
> -source：新生成时钟的源点，（即master clock设置点–master clock get_pins设置的点）
>
> -divide_by：定义新时钟 `lsi128k_clk_sel_clk` 相对于主时钟 `lsi128k_clk` 的**分频比**
>
> -add：选项用于在同一个端口或引脚上添加多个时钟源。一般当需要为同一物理引脚定义多个时钟约束时，该选项允许指定附加的时钟源
>
> [get_pins xxx]：新时钟设置在哪一个对象上；
