---
title: 6 Low-power support（1）
date: 2024-07-02 20:49:45
categories:
- 基于Cortex-M的SoC设计
tags:
- low power
- TBD
---



# Overview of low-power Cortex-M features

现在的MCU追究非常机制的能效比；

<font color=blue>Cortex-M系列处理器提供了很多的低功耗特性：</font>

- sleep mode：sleep mode和deep sleep mode
- 多个模块级别的时钟门控，以及子模块级别的时钟门控（有些时候也称为架构门控时钟）
- 状态保持电源门口SRPG（State retention power gating  ）
- WIC
- sleep-on-exit：应用在中断驱动的场景中



<font color=blue>Cortex-M系列处理器还有很多能降低功耗的优点：</font>

- 代码密度高：降低存储空间要求；
- core的面积小：门数少
- 性能优，程序执行的快，减少运行时间；
- 低中断延时以及中断处理优化：能够降低中断处理时的功耗



<font color=blue>还有些系统级别的低功耗策略：</font>

例如将SRAM分成多个bank，某些场景下关闭不用的bank，来降低功耗；

# Low-power design basics

## clock gating

时钟门控clock gating是最基本的低功耗技术之一。

### 寄存器级别的时钟门控：

综合工具会将使能信号综合成寄存器时钟门控，对Verilog RTL的代码风格有要求

{% asset_img image-20240709093742706.png%}

<font color=blue></font>

### 模块级别的时钟门控

<font color=blue></font>

处理器内部各个功能模块的时钟信号受到时钟门控，通过例化时钟门控CELL，这种做法也叫做ACG（architectural clock gating  ）；

{% asset_img image-20240709094847853.png %}



很多的Cortex-M处理器有多个top_level级别的时钟信号，designer需要自行添加时钟门控；

{% asset_img image-20240709095233535.png %}

> 可以使用debug power control signals CDBGPWRUPREQ and CDBGPWRUPACK  去门控debug CLK
>
> 可以使用GATEHCLK signal   去门控system CLK



## power gating

有些场景下仅仅关闭时钟还是无法满足低功耗场景的要求，这时就可以考虑power gating；

power gating技术需要：

- power switch transistors  
- isolation  cell
- clamping  cell

> **电源开关晶体管 (Power Switch Transistors)**:
>
> - 这些晶体管用于实现电源门控，即通过控制它们的通断来控制电路的电源供应。当电源开关晶体管关闭时，相关电路部分与电源完全隔离，以减少静态功耗。
>
> **信号隔离 (Signal Isolation)**:
>
> - 在电源门控中，关闭电源时，需要确保电路输入和输出信号不会影响其他部分。为了实现这一点，通常会使用专门的电路单元或电路设计技巧，如电流隔离器或传输门，以保持断开电源时的信号完整性和稳定性。
>
> **信号夹持 (Signal Clamping)**:
>
> - 电源门控时，有时需要对电路输入或输出信号进行夹持，以防止信号超过安全电压范围。夹持电路通常包括二极管或其他电路元件，用于限制信号的振幅或保护相关电路部分不受损坏。
> - 信号夹持就是将对外的value输出为high或者low

{% asset_img image-20240709100104211.png %}

### SRPG（State Retention Power Gating  ）

<font color=blue></font>

power gating的一个最大的缺点就是掉点后状态会丢失，为了解决这个问题，引入了新的技术，SRPG（State Retention Power Gating  ），其在寄存器内部引入状态保持的逻辑，并且使用独立的电源供电。

- 寄存器除了状态保持部分，其余部分均会掉电，因此也会节省很多功耗；
- funciton功能下，此种寄存器的功耗比普通寄存器的功耗更高，因此不适合经常toggle的寄存器；



## 其他低功耗技术

纯粹的数字逻辑一般会用到上述的低功耗技术。在其他方面还有低功耗技术，例如

- memory macro的低功耗技术；
- 外设组件，例如ADC有其自身特有的低功耗技术



# Cortex-M low-power interfaces  

## Sleep status and GATEHCLK output  

<font color=blue>大多数的Cortex-M处理器具有如下的低功耗状态输出信号：</font>

{% asset_img image-20240712094613888.png %}

---

<font color=blue>==》SLEEPING和SLEEPDEEP信号的说明</font>

从架构上来说，处理器支持两种睡眠模式，sleep mode和deep sleep mode。

进入这两种睡眠模式的机制是相同的：

1、首先执行WFI或者WFE；

2、启用了sleep-on-exit   特性，处理器从异常处理程序返回到线程级别；

> 思考：
>
> 这里关于sleep-on-exit信号是不是说错了？sleep-on-exit特性使能的时候，中断处理结束会直接进入sleep mode。

Cortex-M处理器进入哪种睡眠模式，根据SCR（System Control Register  ）的配置。

处理器内部对这两种模式的处理是相同的，SLEEP和SLEEPDEEP信号的作用是，让system designer去定义系统级的低功耗策略。

WIC只在deepsleep mode的时候才会使用；

---

<font color=blue>==》GATEHCLK信号的说明</font>

当处理器是处于睡眠模式的时候，处理器的AHB-Lite总线接口仍然可能发出debug access访问；

因此，就需要一个额外的GATEHCLK信号用来表示处理器处于睡眠状态，并且此时没有AHB-Lite总线访问（例如debug access导致的AHB-Lite transaction）。

---

以上这些信号的使用特定于MCU，没有固定的使用规则；

例如，如果GATEHCLK信号是高电平的时候，可以：

- 将系统BUS clock gate off

- SRAM进入低功耗模式

- 部分外设被关闭；



## Q-channel low-power interface

> (Applicable to Cortex-M23, Cortex-M33,Cortex-M35P)

部分较新的处理器（Cortex-M23, Cortex-M33,Cortex-M35P）支持Q channel，这是AMBA 4 低功耗接口中定义的握手协议；

略





## Sleep hold interface

<font color=blue>sleep hold interface的作用</font>

当处理器从sleep mode唤醒的时候，可以使用这个接口延缓程序的执行；

<font color=blue>sleep hold interface的使用场景</font>

处理器的运行需要SRAM，但是SRAM可能需要一定的时间才能从低功耗状态退出，在SRAM退出低功耗状态时，CPU不应该执行程序，此时就可以使用这个端口；

当在deep sleep mode的时候，`如果采用WIC的方案，整个特性就很少会被使用`。因为当使用WIC的时候，此时的门控机制会关闭处理器的所有时钟，让其处于低功耗模式；

> 思考：
>
> <font color=red>没太懂，为啥使用WIC方案的时候，sleep hold 这个特性很少会用到？</font>

{% asset_img image-20240721172648556.png %}

> 这两个信号都是低电平有效，一般和功耗管理单元PMU交互

<font color=blue>sleep hold interface的使用方式</font>

1、当PMU检测到SLEEP/DEEPSLEEP信号的时候，会assert SLEEPHOLDREQn；

2、处理器返回SLEEPHOLDACKn

- 如果处理器 asset SLEEPHOLDACKn，此时PMU就可以关闭memory，外设，时钟等等；
- 如果处理器没有asset SLEEPHOLDACKn，此时意味着，处理器又接收到中断或者debugger request，直接就会被唤醒；在这种情况下，PMU就不应该进行下一步的行为，并且在SLEEPING or SLEEPDEEP  de-assert的时候，需要de-assert SLEEPHOLDACKn；

3、如果处理器 asset SLEEPHOLDACKn，唤醒中断到来的时候，处理器将会de-asset sleep signal，此时处理器并不会立刻执行程序，而是需要等到SLEEPHOLDREQn  deassert

{% asset_img image-20240722163855336.png %}

> 思考：
>
> 1、<font color=red>SLEEPHOLDACKn  会在什么时候从core返回？？</font>
>
> 2、在extended sleep  阶段，GATEHCLK有可能还是高电平，此时HCLK会被gating off；（特定于处理器，不同处理器的行为可能不相同）
>
> 3、目前接触的项目中，并没有使用这个特性；

<font color=blue>如果不需要这个sleep hold feature  ，需要将SLEEPHOLDREQn  = 1‘b1;</font>





## Wakeup Interrupt Controller (WIC)  

<font color=blue>为什么需要WIC</font>

如果将CORE的所有时钟都关闭，或者CORE处于掉电状态（最大程度的降低CORE的功耗），那么NVIC此时就不能检测唤醒事件，此时就需要WIC。

<font color=blue>WIC接口</font>

WIC是一个可选的模块，处于always-on power domain。当NVIC没有时钟或者掉电的时候，WIC起到检测中断事件的作用。

WIC接口特定于处理器，但是一般来说，：

- WIC和处理器的接口如下：

{% asset_img image-20240722165602006.png %}

- WIC和系统的接口如下：

{% asset_img image-20240723104002065.png %}



<font color=blue>Cortex-M产品包交付的WIC是一个示例逻辑，允许修改。在某些情况下，designer会将WIC设计成基于锁存器的操作，这样的话，唤醒时间的检测和capture就无需任何时钟;</font>

{% asset_img image-20240723104805892.png %}



> 项目中是如上图这么做的
>
> <font color=red>有个问题，clock后面需要弄一个clock mux，用于切换scan和function clock？为什么这么做？</font>



<font color=blue>WIC的工作机制</font>

1、当进入睡眠模式的时候，wakeup event mask将会从NVIC--》WIC，通过(WICMASK[] and WICLOAD)。

2、当唤醒事件到来的时候，WIC将会发送wake up request到PMU

3、PMU控制处理器的power和clock恢复；此时处理器可以接收中断请求（或其他唤醒事件）；

4、当处理器被唤醒之后，WIC的wake-up mask信息以及pending wake-up event将会被自动清除(WICCLEAR)  

{% asset_img image-20240723111728699.png %}



<font color=blue>WIC的集成</font>

Cortex-M处理器，部分是将WIC放在处理器内部，有些是将WIC放在处理器外部；

放在外部的WIC，集成上略有不同：

{% asset_img image-20240723111937842.png %}



























