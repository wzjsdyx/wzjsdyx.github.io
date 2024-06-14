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

AHB Slave multiplexer是被一个`data phase版本的HSEL信号`控制(对address phase的HSEL做一级pipelline)。具体的实现方法是，利用系统范围的HREADY信号做使能，然后对HSEL寄存一拍（可以参考前文的原理图）；<font color=blue>大多数情况下，HSEL的寄存行为是在AHB Slave Muxplexer中做的，所以Designer可以直接将地址阶段的HSEL信号连接到AHB Slave Multiplexer即可；</font>



AHB system中可能由多个decoder和AHB Slave Multiplexer。AHB system有多个AHB subsystem，各自包含了一块地址空间。在这样的场景下，子系统的decoder可能就要考虑到顶层decoder的HSEL信号，如下图所示：

{% asset_img image-20240613230353912.png %}



## Handling of multiple bus masters  

如果是基于AMBA2规范的AHB协议，多个AHB Master和AHB Slave之间的连接关系如下图所示：

{% asset_img image-20240613230916059.png %}

> 多个AHB Master通过Master multiplexer共享同一个总线，由master arbiter仲裁产生控制信号；
>
> read data和response通过Slave multiplexer返回，由decoder产生控制信号；
>
> 在Master发出transfer之前，首先通过`HBUSREQ (Bus Request) `以及`HGRANT (Bus Granted)  `信号和Master arbiter进行handshaking
>
> 1. A bus master must first assert a bus request (HBUSREQ) to the arbiter, then
> 2. After arbitration, the arbiter returns the bus granted signal (HGRANT) to one of the bus masters, then
> 3. The bus master can then generate transfers on the bus.
> 4. If the HGRANT signal is de-asserted, the bus master must stop issuing new transfers.  

<font color=blue>简单的AHB系统可以根据这种规则进行工作，但是如果是复杂的场景，BUS带宽就会受到限制，因为BUS上同时只能有一个Master和一个Slave进行通信；</font>

<font color=blue>`AMBA Design Kit (ADK) `产品有了一种新的技术（`multi-layer AHB`）去支持多个AHB Master同时使用总线的场景；</font>

ADK的`BusMatrix IP`是一个`AHB interconnect`组件（使用的是multi-layer AHB技术）,即支持多个Master和多个Slave互联；

如果有多个AHB Slave需要连接到总线，但是总线资源有限的情况下，可以在BusMatrix外面使用额外的Slave Muxplexer进行扩展；

{% asset_img image-20240613234213952.png %}

可以看到，BusMatrix IP<font color=blue>为了解决多个bus master去访问同一个bus slave时的冲突</font>，每个master port（connects to bus slave）都有一个arbiter；

如果某一个bus masterA想要传输数据到某个bus slave，但是此bus slave正在被另外一个bus masterB访问，那么bus masterA的transfer会保存到input stage的buffer中；在这种情形下，HBUSREQ 和HGRANT 就不再需要了；

> 一些思考：
>
> 1，原来的HBUSREQ 和HGRANT是为了解决多个bus master去访问同一个bus slave的场景；
>
> 现在使用input stage之后，transfer会被缓存到input stage中；就不会造成冲突，因此就不需要HBUSREQ 和HGRANT
>
> 2，文中只说明了一个bus masterB已经访问了bus slave，然后在访问期间，另外一个bus masterA有来访问bus slave的情况；如果是两个bus Master同时访问同一个bus Slave，处理的情况也是类似的；都会经过arbiter仲裁，然后有一个bus master transfer会被缓存到input stage；
>
> 3，如果bus masterA的transfer的被缓存到input stage中，此时slave port（connect to bus masterA）hreadyout应该是拉低的，此时bus masterA会hold<font color=red>这一步transfer吗?如果会的话，为什么要缓存？直接返回hreadyout，然bus MasterA一直hold不可以吗？</font>















<font color=red>需要研究下AHB组件 Slave Mux IP 的HSEL信号一级HREADYOUTx，HREADY实现</font>







