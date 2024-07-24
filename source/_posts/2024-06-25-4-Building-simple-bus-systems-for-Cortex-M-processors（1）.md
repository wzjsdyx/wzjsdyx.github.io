---
title: 4 Building simple bus systems for Cortex-M processors（1）
date: 2024-06-25 17:44:15
categories:
- 基于Cortex-M的SoC设计
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



<font color=blue>有系统cache和上述组件的时候，如果CODE区域没有其他组件，架构可以考虑如下：</font>

{% asset_img image-20240626234311838.png %}

> 对于上述系统架构，I-CODE和D-CODE都是访问flash，在有系统cache的情况下，将I-CODE和D-CODE分开之后的收益不是很大了；
>
> 假设CODE区域没有其他Slave组件，其将I-CODE和D-CODE分开的初衷是，防止指令访问的wait state也阻碍了数据访问，大大降低系统性能；
>
> 但是现在有cache的情况下，指令访问不会有太多的wait state，因此就可以不用分开；
>
> 新一代的处理器，例如M33和M35P，I-CODE和D-CODE已经merge了，用于降低系统集成的复杂度以及low power。



<font color=blue>cpu cache VS flash cache：</font>

- cache如果放在cpu门口，那么只能CPU使用，其他master无法使用；
- cache放在flash附近，其和flash的接口可以使用更大的数据位宽；
- cache放在flash附近，cpu访问的时候就需要经过总线；





<font color=blue>NOTE：</font>

- 在Cortex-M3/M4处理器中，通过S-BUS取指令，需要寄存一拍才能送到instruction fetch interface  ；通过I-BUS获取指令，不需要寄存，可以直接送到instruction fetch interface ，因此如果使用S-BUS执行指令的时候，性能会下降；

- 系统外设一般系统通过S-BUS访问，而不是PPB；

  - PPB只能特权级别访问；
  - 永远是小端模式；
  - 访问行为是Strongly Ordered (no other data memory access can start until the current data access finished);  
  - 没有bit-band特性；
  - 只支持32bits访问，非对齐访问会出现未知结果；
  - 仅仅只有Core能访问；

  



> 思考：
>
> <font color=red>什么是Strongly Ordered？难道AHB不是strongly Ordered？</font>
>
> Strongly Ordered是一种存储器的访问模型；
>
> 



# Handling multiple bus masters  

==》在很多的微处理器中，其实会有多个Master，例如：

- DMA
- 高速外设：USB controller，Ethernet interface等等

这些模块有master interface，能主动发起transfer；也有slave interface，为了配置其工作参数；

==》为了处理器多个Master的情况，ARM提供了不同的IP

- Simple AHB master multiplexers  ，多个Master只能有一个占用总线，考虑到带宽的问题，通常只能支持2-3个master
- Configurable AHB Bus Matrix components（ 支持多个Master并发访问）

{% asset_img image-20240627212730972.png%}

> 这种设计适合对系统带宽要求不高的场景；
>
> 如果对系统带宽有着严格的要求，可以采用如下的设计；

{% asset_img image-20240627212849347.png%}

<font color=blue>Bus Matrix支持稀疏的配置，内部有default slave，灵活性很高，但是在进行Master间切换的时候，会引入latency。</font>

可以通过 减少arbiter切换master 来对latency进行优化：

- 定制化逻辑，例如定义默认选择的master
- 当bus idle的时候，强制将地址设置为特定的值

> NOTE:
>
> <font color=red>上面说的两种优化方式都不太懂，BusMatrix组件需要好好的研究下！！！</font>
>
> 

==》从安全的角度来考虑，例如DMA这种Master的Slave 接口，通常要保证特权级别访问；

不然非特权级别的程序可能会利用DMA，bypass memory protect



# Exclusive access support  

Armv7-M 和 Armv8-M  架构的处理器都支持互斥访问操作。

---

==》多核系统

<font color=blue>如果多核系统要支持互斥访问操作，需要集成global exclusive access monitors ；</font>

global exclusive access monitors需要放在bus matrix或者AHB master multiplexer 的下游；

总线组件也需要提供HMASTER，让global exclusive access monitor  知道是哪一个Master产生了transfer；

{% asset_img image-20240627220026567.png%}

<font color=blue>符合如下情况，需要添加global exclusive access monitors：</font>

1、可能存放信号量数据；

2、可能被多核互斥访问；

只有在多核系统中才需要global exclusive access monitors；

---

==》单核系统

单核系统中，即使有多个Master，软件也可以控制不会同时发生多个Master访问信号量数据的情况，因此不需要global exclusive access monitors

<font color=blue>单核系统中，如果使用的是M3/M4/M7，其会有专有的互斥访问handshake信号(EXREQ and EXRESP)  ：</font>

- 总线上如果有SRAM，就需要将EXRESP tie low，不然OS系统的信号量功能会一直返回fail
- 总线上如果只有NVM，或者外设，此时可以将EXRESP tie high ，表示不支持互斥访问；

<font color=blue>单核系统中，如果使用的是M23/M33/M35P，使用的是标准AMBA5 AHB互斥访问信号，(HEXCL and HEXOKAY)  ：</font>

- 如果总线包含SRAM，需要对HEXOKAY做简单的glue逻辑；
- 如果总线上如果只有NVM，或者外设，可以将HEXOKAY tie low，表示不支持互斥访问；



# Address remap

Address remap是基于Cortex-M的MCU中一种常见的system design technique  ；一般用来支持多个boot阶段或者多种boot mode；

举例说明：

假设一个基于Cortex-M0的MCU可能需要支持boot loader；

==》address remap需要完成如下动作：

1、startup阶段将boot loader ROM放到0x0000_0000处；

2、之后将embedded flash映射到0x0000_0000，用于执行用户程序；

==》为了支持address remap，需要寄存器去控制系统address decoder的行为

{% asset_img image-20240628105739745.png%}

0、在boot的时候，REMAP是出于active的状态，即ROM会映射到0x0000_0000地址处；

1、处理器读取的是ROM的中断向量表（按照存放位置是0x0010_0000进行编译），执行的是ROM的reset handler（在0x0010_0000地址处执行）

2、ROM的reset handler执行结束之后：

- 将REMAP disable，（0x0000_0000映射的就是Flash usercode的中断向量表）；
- 读取Flash usercode vector table中的MSP并进行配置；
- 读取Flash usercode vector table中的reset handler的地址，并进行跳转；

有如下几点说明：

- Flash在boot阶段的时候，也可以REMAP，这样romcode是可以对Flash中的程序做一些操作的；否则，REMAP之后，Flash的code就不可见；
- remap控制寄存器：
  - 仅支持特权级别访问；
  - 通过power on reset控制，这样在boot的时候，仅执行一次；（例如，debugger在使用SYSRESETREQ field in AIRCR  进行复位的时候，不会再次执行；）
  - 有些系统中，REMAP只能软件被deactive而不能被active，或者说，bootrom仅仅在boot阶段可见，在REMAP关闭之后，rom就不可见，为了功能安全；



AHB BusMatrix IP是支持REMAP操作的；

- 但是如果处理器支持VTOR，那么就不需要使用REMAP，因为其可以通过将VTOR指向某块区域；

- Cortex-M7/M23/M33处理器支持启动时，中断向量表的位置是可配置的；

> 思考：
>
> 1、VTOR寄存器的作用？
>
> 指定中断向量表的位置；
>
> 2、<font color=red>一般在什么场景中会利用VTOR寄存器？</font>
>
> 

# AHB-based memory connection versus TCM  

<font color=blue>有些处理器可能会使用TCM；如果没有TCM接口，就需要使用AHB去访问系统SRAM；</font>

就性能而言（且不考虑AHB访问时的wait state），TCM接口和AHB接口访问SRAM，在read access上，latency是相同的；

写访问时间可能会有所不同，如下图所示：

{% asset_img image-20240628133549414.png %}

但是因为处理器的pipeline以及write buffer等技术，write access也可能被优化成一个Cycle完成；



<font color=blue>使用TCM的最大的优点是，不用担心总线的wait states；最大的缺点是只有Core能访问；SRAM资源利用不充分</font>



<font color=blue>在有些设计场景中，需要使用TCM去获得一个确定的中断响应时间；</font>



# Handling of embedded flash memories  

## IP requirements

嵌入eflash受限于工艺节点；

需要eflash controller IP；

通常也需要system cache IP；

## Flash programming  

eflash的memory是以page进行划分，因此eflash的操作也都是以page为单位；

做flash的program时，并不是直接将debugger和eflash controller连接，而是通过如下方式：

- 将flash programming algorithm  程序加载到SRAM中
- 将要写入flash中的数据，放到SRAM中；
- 配置flash programming algorithm需要的信息；
- PC设置到flash programming algorithm进行program；

- 一次program一个page，而且flash programming algorithm   可以对page内容进行verify；
- debugger重复上述过程，直到完成对flash的编程；



## Bringing up a new device without a valid program image  

如果eflash中没有任何程序，怎么启动Cortex-M MCU？

1、power on上电，此时flash中是空程序，会先发生hardfault，然后再lockup；

2、即使MCU处于lockup的状态，debugger仍然能够和MCU建立连接；

3、此时调试器可以使能reset vector catch  ，然后写AIRCR 寄存器进行软复位；处理捕获到reset release之后，进入halt状态；

4、debugger下载flash program algorithm以及程序镜像数据到SRAM，然后设置PC去启动flash program algorithm

5、当所有的flash程序program完毕之后，再次复位开始执行flash usercode；



































































