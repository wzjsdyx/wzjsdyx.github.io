---
title: 培训笔记-3-Features-ASIB AHB-AXI
date: 2026-11-09 10:21:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

# 原文

> There are two different options you can select when converting from AHB to AXI using an AXI slave interface block, or ASIB. The Advanced High-performance Bus, AHB,  external interface can be configured as an AHB-lite mirrored master, or it can be configured as an AHB-lite slave, and the difference between these two options is how the NIC-400 connects to the rest of your AHB-lite system. If your NIC-400 connects directly to an AHB-lite master, you would need to use the AHB-lite mirrored master option. If the NIC-400 connects into a larger AHB-lite system as a slave, you would select the AHB-lite slave. The reason being that AHB exposes different signals depending on whether you’re connecting directly to a master or whether you’re connecting to the output of an AHB-lite switch. Aside from the signal differences, there are a couple of different configurable options. Both configurations share the top two options shown here. One of them allows you to promote AHB INCR transactions to AXI INCR transactions of length 4 to improve efficiency with memory controllers. The second function that the ASIB conversion would do is to provide an early OK response to write transactions. So, for example, if you send in a transaction, the early write response would provide an immediate OK response, even though the write has not made progress into the system. Still, if the slave returned an error or returned another error somewhere in the design, this error response would be lost for this kind of transaction. The AHB-lite slave configuration will have exactly the same as the previous two options, like the AHB-lite mirrored master. AHB-lite slave configuration also supports a so-called broken bursts option that adds additional logic to handle the AHB-lite subsystem changing targets halfway through the burst.

![image-20261110195236813](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-8-Features-AXI-AHB-APB-conversion\image-20261110195236813.png)





# 原文翻译+图文理解

## 接口类型配置

使用 AXI 从接口模块（AXI Slave Interface Block，简称 **ASIB**）将 AHB 转换为 AXI 时，外部 AHB（Advanced High-performance Bus，高级高性能总线）接口有两种可选配置：**AHB-Lite 镜像主接口（mirrored master）**和 **AHB-Lite 从接口（slave）**。

这两种配置的区别，在于 NIC-400 如何连接到其余的 AHB-Lite 系统。

<font color=blue>如果 NIC-400 直接连接到一个 AHB-Lite 主设备，就需要选择 AHB-Lite 镜像主接口；如果 NIC-400 作为一个从设备接入较大的 AHB-Lite 系统，则应选择 AHB-Lite 从接口。这是因为，直接连接主设备与连接 AHB-Lite 交换结构的输出端时，接口所需的信号有所不同。</font>

![image-20261110202715257](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-3-Features-ASIB-AHB-AXI\image-20261110202715257.png)

## 功能配置

除了信号差异，上述两种接口配置还提供一些可配置功能。

- <font color=blue>第一项允许将 AHB INCR 事务提升为长度为 4 的 AXI INCR 事务</font>，以提高访问存储控制器时的效率。

- <font color=blue>第二项允许 ASIB 在转换过程中，对写事务提前返回 OKAY 响应</font>。例如，收到一个写事务后，即使该写事务尚未继续向系统内部推进，也可以立即返回 OKAY。不过，如果目标从设备随后返回错误，或者设计中的其他位置出现错误，这类事务的错误响应就无法再传回发起方。

AHB-Lite 从接口配置同样支持上述两项功能。此外，它还支持一个称为 <font color=blue>Allow Broken Bursts（允许突发中断）的选项。该选项通过增加额外逻辑，处理 AHB-Lite 子系统在突发传输中途切换目标的情况。</font>



# 原理解释

## **1. “镜像主接口”不代表 NIC-400 在这里主动发起请求**

这是最容易混淆的地方。两种配置下，事务流向都是：

```
外部 AHB 主设备 → NIC-400 的 ASIB → 内部 AXI 网络 → 目标设备
```

区别只是 NIC-400 的 AHB 入口连接到哪里：

| 配置                     | 连接关系                                 | 理解方式                                |
| ------------------------ | ---------------------------------------- | --------------------------------------- |
| AHB-Lite mirrored master | AHB 主设备 → NIC-400                     | NIC-400 的端口直接与主设备信号配对      |
| AHB-Lite slave           | AHB 主设备 → AHB 互连/交换结构 → NIC-400 | NIC-400 是 AHB 系统中被选中的一个从设备 |

这里的 **mirrored** 可以理解为：接口信号方向与外部主设备相对应。主设备输出地址和控制信号，NIC-400 接收；NIC-400 返回读数据和响应，主设备接收。

普通 AHB-Lite 从接口则还需要配合上游互连的选择、就绪等机制，例如通过 `HSEL` 判断当前访问是否选中了自己。

## **2. INCR promotion：把不确定长度的传输组织成 AXI 突发**

AHB 中的 `INCR` 表示地址递增，但**没有预先规定总共传多少拍**；AXI 的 INCR 突发则需要在发出地址时给出长度。

原文所说的 promotion，就是让桥接逻辑把符合条件的 AHB INCR 传输组织成 **4 拍的 AXI INCR 突发**：

```
AHB：  地址 A → A+4 → A+8 → A+12 → ……
                      ↓
AXI：  一个包含 4 拍数据的 INCR 突发
```

上例假设每拍传输 4 字节。这里“长度为 4”指的是 **4 拍，不是 4 字节**。

这样可以减少逐笔发出 AXI 请求的开销，也有利于存储控制器连续处理数据。具体适用条件需要结合 NIC-400 的配置和事务属性。

## **3. Early Write Response：提前确认收到，不等于目标已经写完**

正常情况下，可以把写操作理解为：

```
收到 AHB 写请求 → 执行 AXI 写操作 → 收到结果 → 返回 AHB 响应
```

提前写响应则是：

```
收到并接受 AHB 写请求 → 提前返回 OKAY
                    → 后续继续执行 AXI 写操作
```

它的收益是减少 AHB 主设备等待的时间。

代价是：**主设备已经收到成功响应后，后续发生的写错误无法再通过这笔 AHB 事务报告给它。** 因而，这个 OKAY 不能被理解为“目标存储器已经成功完成写入”。

原文用“立即返回”来说明提前响应的含义，不宜理解为任何情况下都保证零等待；实际仍取决于桥接逻辑能否接受请求。

## **4. Broken Bursts：从 NIC-400 看，一串传输可能只到了一部分**

假设上游原本有一串突发传输，但 AHB 子系统在中途改变连接或选择，使 NIC-400 只看到了前几拍：

```
原计划：第 1 拍 → 第 2 拍 → 第 3 拍 → 第 4 拍
NIC看到：第 1 拍 → 第 2 拍 → 后续传输不再到达
```

如果桥接逻辑仍然等着后两拍，就可能无法正常结束对应处理。因此，`Allow Broken Bursts` 用来处理这种突发被截断的情况。



直接连接主设备时，NIC-400 不提供这一配置；

Arm 手册也明确说明，镜像主接口不能启用 `Allow Broken Bursts`。



==》详见后面的Q&A 的 0 和 1分析；



# 总结



# Q&A



## **0、AHB-Lite Master 是否允许 broken burst？**-- done

需要区分三种情况：

| 情况                                                         | 是否允许 | 如何理解                             |
| ------------------------------------------------------------ | -------- | ------------------------------------ |
| 不定长 `INCR` 在若干拍后结束                                 | 允许     | 没有承诺总拍数，是正常结束           |
| `INCR4/8/16`、`WRAP4/8/16` 在没有 ERROR 时，由 Master 自行提前结束 | 不允许   | 已经声明了固定拍数，需要完成         |
| 收到从设备的 `ERROR` 后，Master 取消剩余传输                 | 允许     | 协议明确允许，也允许继续完成剩余传输 |

<font color=blue>此外，多层互连可以因仲裁切换 Master，让下游 Slave 看到一个未完成的突发。这与原始 Master 自己无故中断定长突发是不同的情况。</font>

参见 [AHB-Lite 协议 §3.5.2](https://documentation-service.arm.com/static/5f914801f86e16515cdc2a27)

## 1、Cortex-M4 或 Cortex-M7 会自行 broken burst 吗？-- TBD

需要确认下两个处理器发出的Burst类型；



## 2、纯AHB-Lite Master对Mirror Master和Slave接口 --Done



> Q:外部是纯AHB-Lite Master，ASIB配置成镜像Master还是Slave其实都是兼容的吧？</font>
>
> 例如外面是纯AHB-Lite Master，ASIB是Slave接口。没有HSEL信号，其实可以用HTRANS[1]，Slave接口的Hreadyin可以直接连接到Slave的Hreadyout。

- 对于Mirror Master的配置肯定是没问题；

- 对于Salve接口的配置，分析如下：

对于这个连接：

```
唯一的 AHB-Lite Master → 唯一的 ASIB Slave 接口
```

并且 Master 的所有有效访问都交给这个 ASIB 处理，可以这样连接：

```
ASIB.HSEL       = 1'b1
ASIB.HREADYIN   = ASIB.HREADYOUT
Master.HREADY  = ASIB.HREADYOUT
```

地址、控制、写数据和返回的读数据、响应分别对应连接即可。

<font color=blue>这里将 `HREADYOUT` 反馈到 `HREADYIN` 是合理的：**整个 AHB 连接只有这一个 Slave，其输出的就绪状态就是总线的全局就绪状态。** 不需要人为增加一拍。</font>

你提出的 `HSEL = HTRANS[1]`，从**有效传输接收条件**看也是成立的：

```
接收地址阶段 = HREADYIN && HSEL && HTRANS[1]
```

代入后就是：

```
接收地址阶段 = HREADYIN && HTRANS[1]
```

<font color=blue>不过，在这种唯一目标的直连结构里，建议直接把 HSEL 接高</font>

> `HSEL` 的含义是“地址选中了这个 Slave”（接高更直接），而 `HTRANS[1]` 的含义是“当前是有效传输”；

<font color=blue>如果HSEL接HTRANS[1]，可能会有性能上的影响，具体看BUS IP。例如CMSDK BUSMatrix，会打拍（Arbiter之后，将Arbiter结果寄存==》性能的权衡，为了更高的频率）</font>

### 使用两种不同的配置，有PPA上的差异吗？--TBD

需要看实际RTL的行为和综合结果！



## 3、总线AHB-Lite Master对Mirror Master和Slave接口--done



> Q:外部是AHB-Lite总线的Master，ASIB配置成镜像Master还是Slave其实都是兼容的吧？</font>



- 使用Slave接口的配置肯定没问题；
- 使用Mirror Master配置，分析如下：



接在 AHB 互连后面时，接在 AHB 互连后面的端口，不能无条件换成 mirrored master

此时：

- `HSEL` 来自地址译码或互连选择。
- `HREADYIN` 是总线当前数据阶段的全局就绪状态。
- `HREADYOUT` 只表示这个 Slave 自己的就绪状态。

例如，当前数据阶段属于 Slave A，下一地址阶段已经选择 NIC-400；NIC-400 必须通过全局 `HREADYIN` 判断这一地址阶段能否被接收，不能只看自己的 `HREADYOUT`。













