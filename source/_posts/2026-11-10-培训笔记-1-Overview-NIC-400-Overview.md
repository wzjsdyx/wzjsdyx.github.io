---
title: 培训笔记-1-Overview-NIC 400 Overview
date: 2026-11-09 10:19:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD

---



# 原文

> Systems used to be relatively straight forward regarding connections. For example, if you focus on the top left, you can see one clock domain, limited data widths, and support for limited protocols. For this relatively simple system, you might have used a switch that ran on a fixed frequency using a fixed data width. Then you may have added various bridges onto that switch to connect all your different components, so you might have added data resizing, protocol conversion, and so on. As systems got more complicated, we required more clock domains and switches of different sizes to efficiently optimize the network. If we look at our masters and slaves, we can see that we might also have components running at higher frequencies with more protocols and wider data widths. Connecting all of these components together becomes increasingly more complicated. With the need for multiple switches, you may need upsizing and downsizing blocks that loop between the different switches or clock domain crossing blocks of different varieties. The NIC-400 brings all this functionality together into one single top-level design. You use it to configure the external interfaces and internal topology, and then use Arm's Socrates tool to insert all of the various components such as upsizers, downsizers, and clock domain crossings into the appropriate places. This removes a lot of the manual connection work. So what you will get as the output of a single NIC-400 is virtually all of this area within the dark grey box shown here. Let's look at the Interface Block, IB. The IB is a very generic unit, and it implements all the various transfer functions such as protocol conversion, data width conversion, buffering, and clock domain crossing. Anywhere you have an IB component, or IB within the name of a component, it can implement one of these functions. The IB by itself is always implemented between two switches. The ASIB and AMIB blocks, as shown in the figure, can also implement the IB functionality. All the masters in our systems will always connect to an AMBA Slave Interface Block (ASIB). In the AXI specification, we always have a master interface in AXI connecting to a slave interface. So the input is the slave interface block. As well as being an interface block, the ASIB can also convert from AHB to AXI protocol as an additional function. On the output of the NIC-400, we have the AMBA Master Interface Block (AMIB ). On a master interface, it can also carry out all the IB functions, and in addition to that, the AMIB can also convert from AXI to AHB and AXI to APB protocol. The NIC-400 is an AXI interconnect, so anything that comes into the NIC-400 will be converted into AXI, and at the output of the NIC-400, if it needs to be another protocol, then the AMIB will convert it. To connect all the various units in the NIC-400, the main component we have is a switch. The switch is in charge of the routing and the arbitration between the inputs and outputs. Switches can have 1 to 12 slave interface inputs and 1 to 14 master interface outputs, so the largest a single Switch can be is 12 by 14. The Global Programmers view (GPV) is the entry point into the register space within the NIC-400. If you want to program the NIC-400, transactions go to the GPV and then get routed from the GPV to wherever they need to go. Another unit that we haven't highlighted is the little ground symbol in the figure that is the default slave. Whenever transactions go into a hole, or rather an unmapped region in a memory map, or fail security, the default slave will be in charge of returning an error response. Finally, the NIC-400 can have up to 128 input interfaces and 64 output interfaces.



![image-20261110103609342](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-1-Overview-NIC-400-Overview\image-20261110103609342.png)



![image-20261110103712565](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-1-Overview-NIC-400-Overview\image-20261110103712565.png)



![image-20261110103747500](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-1-Overview-NIC-400-Overview\image-20261110103747500.png)





# 原文翻译+图文理解

## 1. NIC-400 的设计背景

<font color=blue>过去，系统之间的连接相对简单。</font>

例如，观察图中的左上角，可以看到系统只有一个时钟域、有限的数据位宽，以及少量支持的总线协议。

<font color=blue>对于这种相对简单的系统，通常使用一个运行在固定频率、具有固定位宽的 Switch（交换机）。随后，可以在 Switch 上添加各种 Bridge（桥接模块），连接不同的组件，例如实现数据位宽转换、协议转换等。</font>

随着系统越来越复杂，我们需要更多的时钟域，以及不同规模的 Switch，以便高效地优化整个互连网络。

观察系统中的 Master 和 Slave，可以发现不同组件可能具有：

- 更高的工作频率。
- 更多种类的总线协议。
- 更宽的数据位宽。

将这些组件连接起来变得越来越复杂。

由于需要多个 Switch，系统中可能还需要各种 Upsizer（位宽扩展）、Downsizer（位宽缩减）模块，以及不同类型的 CDC（跨时钟域）模块，用来连接不同的 Switch。

<font color=blue>NIC-400 将所有这些功能整合到一个统一的顶层设计中。</font>

设计人员只需要配置外部接口和内部拓扑结构，然后利用 Arm 的 Socrates 工具，在适当的位置自动插入 Upsizer、Downsizer、CDC 等模块。

这样就减少了大量手动连接工作。

最终，一个 NIC-400 生成的设计，基本包含了图中深灰色区域内的所有功能。



## 2. IB（Interface Block，接口模块）

接下来介绍 Interface Block，简称 IB。

IB 是一种非常通用的模块，它能够实现各种传输相关的功能，例如：

- Protocol Conversion：协议转换。
- Data Width Conversion：数据位宽转换。
- Buffering：缓冲。
- Clock Domain Crossing：跨时钟域转换。

<font color=blue>只要某个模块包含 IB，或者其名称中带有 IB，它就可能实现上述某些功能。独立的 IB 始终位于两个 Switch 之间。</font>

<font color=blue>图中所示的 ASIB 和 AMIB 也能够实现 IB 的功能</font>

## 3. ASIB（AMBA Slave Interface Block）

系统中所有 Master 都需要连接到一个 ASIB。

根据 AXI 协议，Master Interface 必须连接到 Slave Interface。

因此，从 NIC-400 的角度来看，输入侧使用的是 Slave Interface Block，即 ASIB。

<font color=blue>除了具备普通 IB 的功能之外，ASIB 还能够额外实现：AHB → AXI 协议转换。</font>



## 4. AMIB（AMBA Master Interface Block）

NIC-400 的输出侧使用 AMBA Master Interface Block，简称 AMIB。

AMIB 不仅能够执行普通 IB 的全部功能，还能够额外实现：

- AXI → AHB 协议转换。
- AXI → APB 协议转换。

NIC-400 本质上是一个 AXI Interconnect（AXI 互连网络）。

<font color=blue>因此，所有进入 NIC-400 的事务都会被转换为 AXI 协议；如果输出接口需要使用其他协议，则由 AMIB 负责转换。</font>

## 5. Switch（交换机）

NIC-400 内部连接各个模块的核心组件是 Switch。

<font color=blue>Switch 主要负责事务路由（Routing）和仲裁（Arbitration）。</font>

一个 Switch 可以具有：

- 1～12 个 Slave Interface 输入端口。
- 1～14 个 Master Interface 输出端口。

因此，单个 Switch 最大规模为 12×14。

## 6. GPV（Global Programmers View）

GPV 是访问 NIC-400 内部寄存器空间的入口。

如果需要对 NIC-400 进行编程配置，访问事务首先进入 GPV，然后由 GPV 将事务路由到相应的内部寄存器。

## 7. Default Slave（默认从设备）

图中还有一个未特别标出的组件，它使用接地符号表示，称为 Default Slave。

当事务访问以下区域时：

- 地址映射中未定义的区域。
- 安全检查失败的区域。

Default Slave 将负责返回错误响应。

## 8. NIC-400 的接口规模

最后，整个 NIC-400 最多可以支持：

- 128 个输入接口。
- 64 个输出接口。





# 原理解释

NA

# 总结

这段资料重点介绍 NIC-400 的内部组成架构，记住以下关系即可：

NIC-400 = ASIB + AMIB + IB + Switch + GPV + Default Slave。

其中：

- ASIB/AMIB/IB：解决接口适配问题。
- Switch：解决事务路由和仲裁问题。
- GPV：解决内部寄存器配置问题。
- Default Slave：解决非法访问的错误响应问题。

此外，有一个设计上的重要区别：NIC-400 并不是一个固定拓扑的 Crossbar，而是一个可配置、可自动生成多级互连结构的 Interconnect IP。

这也是 NIC-400 相比手工搭建 Switch + Bridge 架构的主要优势。





# Q&A



## ASIB、IB以及AMIB的协议转换

在标准 NIC-400 架构中，内部 IB 的协议转换主要指 AXI3 ↔ AXI4，不负责 AHB-Lite 或 APB 协议转换。

具体分工如下：

| 模块 | 支持的协议转换                        |
| ---- | ------------------------------------- |
| ASIB | AHB-Lite → AXI3/AXI4、AXI3 ↔ AXI4     |
| IB   | AXI3 ↔ AXI4                           |
| AMIB | AXI3/AXI4 → AHB-Lite/APB、AXI3 ↔ AXI4 |

Arm NIC-400 TRM 对 AXI3/AXI4 协议转换的说明也涵盖 ASIB、内部 IB 和 AMIB。

核心原因：NIC-400 的内部 Switch 网络采用 AXI3/AXI4，而不是 AHB-Lite 或 APB。

所以 IB 不需要支持 AHB-Lite/APB 转换，主要负责内部 AXI 协议版本转换、位宽转换、CDC 和 Buffering 等功能。

