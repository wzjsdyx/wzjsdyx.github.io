---
title: 3 AMBA/AHB/APB
date: 2024-06-11 21:26:50
categories:
- SoC设计
tags:
- AMBA
- AHB
- APB
- TBD
---



# AHB概述

## AHB版本

## AHB信号

## AHB基本操作

### 组件的连接

在单Master和多Slave的简单设计中，AHB总线系统可以按照下图设计：

{% asset_img image-20240611215804055.png %}

<font color=red>1、data phase mux control?什么意思？怎么实现？</font>

<font color=red>2、一般接触到的hready应该是将所有的hreadyout信号进行与操作实现</font>

<font color=red>这两种方式均可</font>

如果遵循`AMBA 5 AHB`规范，可以按照如下方式连接：

- 多个`AHB Master`共享一个bus，需要使用`Master Multiplexer`和`Master arbiter`，arbiter仲裁之后产生选通信号，通过Multiplexer选通，占用总线
- 多个`AHB Slave`返回的data和response信号需要使用`Slave Multiplexer` feedback to `AHB Master`，其中Slave Multiplexer通过Decoder产生的信号进行选通

---

### 信号基本时序

信号可以被划分为`address phase singals `和`data phase singals  `：

`address phase` signals include:

- HADDR, HTRANS, HSEL, HWRITE, HSIZE.
- Optional: HPROT, HBURST, HMASTLOCK, HEXCL, HAUSER  

`data phase` signals include:

- HWDATA, HRDATA, HRESP, HREADY (and HREADYOUT).

- Optional: HEXOKAY, HWUSER, HRUSER.  

<font color=blue>每一个transfer都是有address phase和data phase</font>

<font color=blue>transfer是pipelined，意味着当前transfer的address phase可以和上一次transfer的data phase overlap</font>

{% asset_img image-20240611221656976.png %}

---

### data phase select control生成

> Each phase is terminated by the assertion of HREADYOUT (HREADY) from `the currently activated AHB slave in the data phase`. 
>
> The HREADYOUT from the AHB slaves are multiplexed by the slave multiplexer, forming the system-wide HREADY signal.
>
>  The multiplexer is operating at the data phase of each transfer. Control of the multiplexer can be generated from the AHB decoder, or the HSEL
> signals and the HREADY signal.  

结合上面的波形图，来理解这段话：

> 如果当前看的是Address Phase N，则当前有效的ahb slave data phase 就是上图的data phase(N-1)；
>
> hreadyout通过slave multiplexer形成了系统范围的HREADY信号；
>
> slave multiplexer在一个transfer的data phase进行mux，现在问题就是如何生成slave multiplexer的控制信号data phase select？
>
> ==》通过HSEL以及HREADY信号

{% asset_img image-20240611223909464.png %}





> If an AHB slave is not currently selected, its HREADYOUT should be high to indicate it is ready.
> However, it can only accept a transfer from the bus master when the previous transfer is completed,
> indicated by a high level in the system-wide HREADY.   
>
> 注意两个词语，结合下面的波形图理解：
>
> (Address  phase)  AHB Slave A selected
>
> (Data pahse       )  AHB Slave A active
>
> 以Slave B为例，如果开始时，Slave B未被选中进行transfer，那么下次Slave B的transfer需要等到系统范围的HREADY HIGH才可以进行；

{% asset_img image-20240612222936437.png %}



## Minimal AHB systems  

最小的AHB system有一个bus master，多个bus slave以及接下来的AHB 组件：

{% asset_img image-20240612225507780.png %}



address decoder应根据系统的memory map进行设计，HSEL是一个地址阶段的信号，由HADDR通过组合逻辑产生，其不应该有复杂的decode逻辑，否则综合的时候timing会受到影响；



AHB slave multiplexer会更加通用一点，而且能够在各种ARM AHB IP bundles中找到。不同的AHB Slave有着不同的read data outputs和response outputs，因此需要一个Slave Multiplexer 在current active slave阶段返回data和response；









