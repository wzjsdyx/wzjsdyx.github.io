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





0、时钟只要有负载，就需要插入scan chain，换言之，就需要scan clock；

1、针对clock_mux，如果其选择信号来自于寄存器，需要使用test_scan_mode进行控制（因为在launch测试向量的时候，会导致clock乱动）

## DFT测试相关的PAD

{% asset_img image-20250806201827682.png %}





## 内部设计中涉及到的相关信号

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


