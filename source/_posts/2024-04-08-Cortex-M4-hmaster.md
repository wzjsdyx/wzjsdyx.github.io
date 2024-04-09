---
title: Cortex-M4 hmaster
date: 2024-04-08 22:15:17
categories:
- Cortex-M4
tags:
- Cortex-M4 hmaster信号
- TBD
---

> Cortex-M4 处理器中，有三个AHB-Lite总线接口，分别是IBUS，DBUS，SBUS。
>
> 其中DBUS和SBUS接口有hmaster信号，对其可能的一个使用场景进行分析



# IBUS的hmaster信号

无

`IBUS之所以没有hmaster信号，是因为IBUS只能发出core的指令访问`

# DBUS的hmaster信号

{% asset_img image-20240408222327461.png %}

Cortex-M4处理器的总线接口，hmaster[1:0]信号可以区分：

- core数据访问
- DAP访问
- core指令访问

`中断向量的读取有些特殊，虽然是为了抓取指令，但是总线层面上是数据访问，即HPROT[0]标记为数据访问`

`DBUS不能进行指令访问，因此hmasterd[1:0] value不可能是2`

# SBUS的hmaster信号

{% asset_img image-20240408223015766.png %}

Cortex-M4处理器的总线接口，hmaster[1:0]信号可以区分：

- core数据访问
- DAP访问
- core指令访问

`中断向量的读取有些特殊，虽然是为了抓取指令，但是总线层面上是数据访问，即HPROT[0]标记为数据访问`

<font color=red>These include vector fetches that  are marked as data by HPROTS[0]. 这是什么意思？发个mail问问ARM</font>

> ==>chatgpt
>
> 这里提到的“vector”指的是中断向量，它是在处理器的向量表中的条目，包含了中断服务例程（Interrupt Service Routine, ISR）的入口地址。当中断发生时，处理器会使用这些向量来快速定位到对应的中断服务例程并跳转到那里执行。
>
> <font color=blue>在 ARM Cortex-M4 处理器的文档中，提到了“这些包括被 HPROT[0] 标记为数据的向量抓取”，</font>
>
> <font color=blue>这意味着这些访问虽然是为了抓取指令，但在总线层面上它们被标记为数据访问。</font>
>
> <font color=blue>这通常涉及到对于系统中断向量表的读取，因为它们不是普通的指令抓取，而是需要特别的处理。</font>
>
> HMASTERS信号的作用是指示当前的系统总线主控单元。其中：
>
> - 0 表示核心的数据侧访问。
> - 1 表示调试访问端口（Debug Access Port, DAP）访问。
> - 2 表示核心的指令侧访问，包括被标记为数据的中断向量抓取。
> - 3 是保留值，不应出现在 HMASTERS 上。
>
> 在设计中确保DCode（数据访问）比ICode（指令抓取）有更高的优先级，特别是当涉及到中断处理时，是为了保证中断处理的及时性和数据访问的高效性。中断向量的抓取是关键的一步，因为它指向中断服务例程的开始，必须快速且可靠地完成以确保系统能及时响应中断

# 使用场景-MPU

系统MPU（不是Cortex-M4内部的MPU）可以根据hmaster做访问权限的管控

{% asset_img image-20240408223606948.png%}

例如：

- hmaster = 3'h0作为core访问（此处不区分是指令访问还是数据访问）

- hmaster = 3'h1作为debugger访问
- hmaster = 3'h2作为DMA访问
- hmaster = 3'h3作为HSM（包括hsm-host和hsm-dma）访问



针对上面的要求：

hmaster的处理方式是

- 总线连接IBUS端口的hmaster信号直接tie 4’h0(此处视情况而定，实践中BUS的hmaster位宽是4bits)
- 总线连接DBUS端口的hmaster信号{hmasterd[1],2'b0,hmasterd[0]};
- 总线连接SBUS端口的hmaster信号{hmasters[1],2'b0,hmasters[0]};
- 总线连接DMA端口的hmaster信号tie 4’h2；
- 总线连接hsm_host和hsm_dma端口的hmaster信号tie 4’h4；



这样的话，MPU看低3bits:

- core访问的话，MPU看到的是3'h0；debugger访问的话，MPU看到的是3'h1；
- dma访问，MPU看到的是3'h2；
- hsm_host和hsm_dma访问，MPU看到的是3'h4；

---

参考博客：

1. Cortex®-M4  Integration and Implementation Manual  
