---
title: DFT AU fault report分析
date: 2025-03-05 09:42:13
categories:
- IC设计流程
tags:
- DFT
- FT
---



# 打开Tessent工具

```shell
ANALYSIS>open_visualizer
```

# 查看某一个模块的覆盖率

```shell
Instance Browser界面查看
tab：
	Total
	Test Coverage
```



# 根据AU report在图形化界面分析

report

```shell
      1    AU.TC    /cpusys_par_wrap_0/TestAndSet_0/ts_0_cp_206621and_tpecs0_no201142_i/A1
      1    AU.TC    /cpusys_par_wrap_0/TestAndSet_0/PLACE_SETUP_FE_RC_1405_0/A1
      1    AU.TC    /cpusys_par_wrap_0/TestAndSet_0/PLACE_SETUP_FE_RC_1599_0/A1
      0    AU.TC    /PLACE_SETUP_FE_OFC1761041_ts_0_tpecs0/ZN
```

alalyze

```shell
analyze_fault -display -stuck_at 1 /cpusys_par_wrap_0/TestAndSet_0/ts_0_cp_206621and_tpecs0_no201142_i/A1
```

> Note:
>
> 可能需要open -> Hierarchical Schematic或者open-> Flat Schematic
>
> 然后在对应的界面下方输入instance路径，
>
> 例如/cpusys_par_wrap_0/TestAndSet_0/ts_0_cp_206621and_tpecs0_no201142_i/A1

{% asset_img image-20250305101356484.png  %}

> 针对上面的case做出分析：
>
> 向前trace，发现是DFT插入的function寄存器，用于提高coverage，
>
> 这对此case，可能是为了与门后面的逻辑可控、可观测，因此将A1 tie了一个DFT function寄存器，然后再推理ATPG pattern的时候，将此寄存器输出配置为1‘b1；





# DFT suppor logic

- 利用scan逻辑切换clock和reset

针对一些<font color=blue>慢速时钟</font>，可以使用scan_mode切换scan clock和function clock；（<font color=blue>此时的scan clock的速度能满足stuck-at测试和at-speed测试</font>）

针对一些<font color=blue>快速时钟</font>，需要进行at_speed测试（即transient fault测试），此时无需使用scan enable逻辑进行切换，直接使用occ的输出；

（occ IP会自己处理scan clock和function clock，本质上可以理解为一个mux）

<font color=blue>这种需要补充FT pattern进行测试下；</font>

- ICG

ICG的TE端口，连接的是`scan_en_cg_ctl IP`

<font color=blue>不需要补充FT pattern进行测试；</font>



- Note：

另外针对ATPG测试的时候，希望使用scan clock的同一个上升沿；

因此，红色框体的部分添加了一个clock mux，在scan_test（连接的是trapping test_mode）的时候，bypass flash_clk_inv；

缺点就是，此时只能对flash_clk_inv后面的逻辑进行stuck-at测试（因为使用的是scan_clock慢速时钟），无法进行at-speed测试；

7805 flash_clk_inv后面的逻辑仅有一个寄存器，因此影响不大；

{% asset_img image-20250305141650138.png  %}

