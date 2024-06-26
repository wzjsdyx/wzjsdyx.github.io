---
title: 4 Building simple bus systems for Cortex-M processors（1）
date: 2024-06-25 17:44:15
categories:
- SoC设计
tags:
- bus system
- TBD
---

> 研究怎样基于Cortex-M处理器构建bus system：
>
> Cortex-M0, Cortex-M0+,
> Cortex-M1, and Cortex-M3/M4   

# Introduction to the basics of bus design

system bus有几个基本的设计原则：

- 如果是哈佛结构，bus系统需要保证指令和数据的同时访问；
- 使用default slave IP去处理违法地址的访问；
- 很多Cortex-M处理器，其中断向量表的起始地址在0x0000_0000，因此需要保证程序被加载到这个地址；
- 访问mem的wait state数量要尽可能少，尤其是对于没有cache的处理器，访问mem的wait state会影响性能，功耗以及中断响应延时；
- 尽量保证主系统bus有尽可能少的slave；





































system bus和peripheral bus分开的好处：

- 降低对system bus的max clock frequency的影响；
- 外设bus和system bus可以跑在不同的频率；
- 外设协议更简单；

> NOTE:
>
> most of the peripherals that do not need low latency accesses (e.g., SPI, I2C, UART) can be placed in separated peripheral buses. Some peripherals like GPIO can gain the benefit of lower access latency, so some GPIO blocks are placed system AHB or single-cycle I/O port interface (available on
> Cortex-M0+ and Cortex-M23 processors).  



# Building a simple Cortex-M0 system

{% asset_img image-20240625175846130.png %}





# Building a simple Cortex-M0+ system

{% asset_img image-20240625180447010.png %}



<font color=blue>和Cortex-M0系统非常类似，在如下方面有所不同：</font>

- <font color=blue>Optional single-cycle I/O port (IOP) interface for low latency peripheral register accesses;  </font>
- <font color=blue>Optional Micro Trace Buffer (MTB).  </font>

> NOTE：
>
> ==>IOP
>
> 1、如果想要使用single-cycle I/O port interface  ，需要调整对应的外设时序；
>
> 2、需要有一个简单的IOP decoder，去告诉CPU哪些地址通过IOP访问，哪些地址通过AHB-Lite访问；
>
> ==>MTB
>
> 1、用于提供低成本的指令trace；在AHB和SRAM中间；
>
> norm情况下，就可以看成是一个AHB2SRAM的bridge；
>
> 在进行指令trace的时候，debugger会program MTB从SRAM中划分一块空间用于存储数据；
>
> 2、MTB有一个和处理器连接的trace 接口，用于接收处理器trace数据，以及生成debug事件（halting request）到处理器；
>
> 3、一般来说MTB可以配置成环形buffer从而保存最近执行的指令数据；



- <font color=blue>Cortex-M0+支持特权级访问和非特权级访问；系统集成者也可以考虑配置MPU去支持特权/非特权访问；</font>

> 这样就能够阻止非特权级别的代码去访问特权级别的memory；
>
> 从功能安全级别的角度考虑：
>
> - 用于系统控制的外设寄存器应该设置为特权级别访问；
> - 非特权级别的AHB access 特权级别的device，地址decoder应该select default slave去产生bus error；

# Building a simple Cortex-M1 system

Cortex-M1和Cortex-M0的类似的features：

- 不支持特权/非特权级别的操作；
- 仅仅只有一个AHB接口；

Cortex-M1和Cortex-M0的不同的features：

- Cortex-M1支持ITCM和DTCM；
- 没有sleep mode；



<font color=blue>Cortex-M1一般用作FPGA的开发；</font>

{% asset_img image-20240625203309033.png %}



# Building a simple Cortex-M3/Cortex-M4 system



Cortex-M3/Cortex-M4处理器采用哈佛架构，而且有3个AHB总线接口以及一个APB总线接口；

{% asset_img image-20240625204223785.png %}



<font color=blue>一个典型的Cortex-M3/M4 system design ：</font>

- <font color=blue>program image 被放在CODE区域；</font>
- <font color=blue>sram和外设通过系统总线连接；通常地址0x2000_0000分配SRAM，0x4000_0000分配外设；因为方便软件开发者使用side-band特性；</font>

{% asset_img image-20240625210218086.png %}

{% asset_img image-20240625210337572.png %}

{% asset_img image-20240625210426184.png %}



<font color=blue>将I-CODE和D-CODE分开的目的：添加一个常量数据cache，当指令访问flash，由于wait state而等待的时候，常量数据仍然可以从cache中read；</font>

- 一般来说，flash的速度为30MHz-50MHz，远小于处理器的速度(超过100MHz)。

- 通常会选择位宽更大以及有prefetch buffer的flash，<font color=blue>顺序执行的指令会被prefetch</font>，处理器更多时候会从prefetch buffer中读取指令，从而提供性能；

{% asset_img image-20240626201216548.png%}

























































