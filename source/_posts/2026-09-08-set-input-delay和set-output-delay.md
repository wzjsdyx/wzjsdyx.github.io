---
title: set_input_delay和set_output_delay
date: 2026-09-08 21:26:48
categories:
- DC synthesis
tags:
- TBD
---





# input delay

input_delay是指输入的数据到达FPGA的pad时相对于时钟边沿的延迟有多大

![img](https://pic1.zhimg.com/v2-9caf3d480bff06646f67f58cba8ae87c_1440w.jpg)





![img](https://pica.zhimg.com/v2-74e64d8e4e422f510b29ad4d6d081c82_1440w.jpg)





# output delay



![img](https://pica.zhimg.com/v2-ee6d3272ea9a954a475bcf6fc5fc6430_1440w.jpg)

<font color=red>针对图中的关系我也比较困惑？</font>



# 实战

```tcl
## in/output delay (60%)
if { ${VOL_DEFINE} == "1P1V" } {
puts "the voltage is 1.1v for normal mode constraints"
## TMS
set_input_delay -max 19 [get_ports PAD_PA4] -clock [get_clocks cpu_jtag_clk] -clock_fall -add
set_input_delay -min 0  [get_ports PAD_PA4] -clock [get_clocks cpu_jtag_clk] -clock_fall -add
# 可以近似把这个下降沿理解成“外部 launch/reference edge”
# 内部到底在上升沿还是下降沿 capture TMS，不是 -clock_fall 决定的，而是内部设计决定的


## TMS
set_output_delay -max 19 [get_ports PAD_PA4] -clock [get_clocks cpu_jtag_clk] -add
set_output_delay -min 0  [get_ports PAD_PA4] -clock [get_clocks cpu_jtag_clk] -add

## TDI
set_input_delay -max 19 [get_ports PAD_PC5] -clock [get_clocks cpu_jtag_clk]  -clock_fall -add
set_input_delay -min 0  [get_ports PAD_PC5] -clock [get_clocks cpu_jtag_clk]  -clock_fall -add

## TDO
set_output_delay -max 8 [get_ports PAD_PA10] -clock [get_clocks cpu_jtag_clk] -add
set_output_delay -min 0  [get_ports PAD_PA10] -clock [get_clocks cpu_jtag_clk] -add
}
```



# Q&A:

set_input_delay中：

-clock_fall和不设置有啥区别？



set_output_delay中：



---



参考博客

1. [set_input_delay如何使用？](https://zhuanlan.zhihu.com/p/561936760)
