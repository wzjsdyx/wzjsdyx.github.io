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

## I-CODE和D-CODE的处理方式1：无系统cache

<font color=blue>将I-CODE和D-CODE分开的目的，添加一个常量数据cache，当指令访问flash，由于wait state而等待的时候，常量数据仍然可以从cache中read；</font>

<font color=blue>解释如下：</font>

<font color=blue>==》首先只有一个总线的方案</font>

- 一般来说，flash的速度为30MHz-50MHz，远小于处理器的速度(超过100MHz)。
- 通常会选择位宽更大以及有prefetch buffer的flash，<font color=blue>顺序执行的指令会被prefetch</font>，处理器更多时候会从prefetch buffer中读取指令，从而提供性能；

{% asset_img image-20240626201216548.png%}

但是程序中还有很多常量，这些常量的访问往往是非顺序的，因此prefetch的性能会大大降低；

往极端的方向思考：如果在prefetch刚开始的时候，发生了读常量的操作，需要等待的时间很更久；

{% asset_img image-20240626223223336.png %}

<font color=blue>==》过渡到两个总线的方案</font>

<font color=blue>为了解决上述的问题，一种解决方案是，将数据访问和指令访问分开，然后在数据访问总线上添加一个小的data cache；</font>

> 数据访问一开始的时候会将变量搬运到SRAM中；
>
> 之后程序的运行，即访问常量的时候，会通过D-CODE访问；

{% asset_img image-20240626224644608.png %}

> 这是将一条程序总线分成I-CODE和D-CODE两条总线的初衷；

## I-CODE和D-CODE的处理方式2：有系统cache

<font color=blue>上述方案没有考虑系统级别的cache，如果有系统级别的cache，指令访问就不会因为flash的速度低而等待很久，因此可以不需要上述flash访问加速的机制，</font>

- flash的prefetch机制
- 常量cache机制

<font color=blue>可以考虑直接将I-CODE和D-CODE总线进行merge</font>



为了进行merge，Cortex-M3和Cortex-M4的产品交付包中包含两个组件：

### Code mux component  

门数量非常少，为了使用这个组件，I-CODE和D-CODE只能同时有一个active，通过配置DNOTITRANS  输入端口需要为1

（防止在D-CODE进行transfer的时候，I-CODE再生成transfer）；

{% asset_img image-20240626231517758.png %}

### Flash mux component

这个组件内部有仲裁机制；这个组件在CODE region还有其他slave组件的时候，非常有用，因为I-CODE和D-CODE可以同时发出访问；

{% asset_img image-20240626233511046.png %}



<font color=blue>有系统cache和上述组件的时候，<font color=green>如果CODE区域没有其他组件</font>，架构可以考虑如下：</font>

{% asset_img image-20240626234311838.png %}

> 对于上述系统架构，I-CODE和D-CODE都是访问flash，在有系统cache的情况下，将I-CODE和D-CODE分开之后的收益不是很大了；
>
> 假设CODE区域没有其他Slave组件，其将I-CODE和D-CODE分开的初衷是，防止指令访问的wait state也阻碍了数据访问，大大降低系统性能；
>
> 但是现在有cache的情况下，指令访问不会有太多的wait state，因此就可以不用分开；
>
> 新一代的处理器，例如M33和M35P，I-CODE和D-CODE已经merge了，用于降低系统集成的复杂度以及low power。









































