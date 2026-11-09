---
title: 培训笔记-22-ThinLinks-Thin links
date: 2026-11-09 17:29:54
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

# 原文

> Thin Links is another add-on to the NIC-400, and it is used for longer distances or going off-chip. AXI is not very efficient for long connections because it has many wires that are not always fully utilized, especially if you have a fairly low bandwidth connection. Thin-Links takes the AXI interface and packetizes it into a narrower link that will take up less space and be subject to less congestion. It will also be easier to time across a larger distance or one going off-chip when there is a limited number of pins available. Thin Links is configured as a NIC-400 bridge, which has one input interface and one output interface.As shown here, the AW, AR, W channels will get packetized down from 174 wires running at 400 MHz into a length of only 64 wires running on 800 MHz. For example, if there is a mixture of reads and writes for a particular traffic profile, it is possible to achieve 100 percent bandwidth but with 63 percent fewer wires. The main idea here is because things like the address bus are not very well utilized; if you are, for example, using length 4 transactions, you are only ever going to be using the address bus one in every four cycles. By re-packetizing it down, you can make use of these wires more efficiently and will be able to reduce the number of wires needed to cross a particular link.

![image-20261109202501009](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-22-ThinLinks-Thin-links\image-20261109202501009.png)

# 原文翻译+图文理解

Thin Links 是 NIC-400 的另一个附加功能，主要用于长距离互连或芯片间（Off-chip）通信。

AXI 接口并不适合长距离连接，因为它包含大量信号线，而这些信号线并不总是得到充分利用，尤其是在带宽需求较低的连接中。

Thin Links 将 AXI 接口上的事务进行数据包化（Packetization），然后通过更窄的链路进行传输，从而减少布线资源和布线拥塞。

对于长距离连接或者引脚数量受限的芯片间连接，这种方式也更容易满足时序要求。

Thin Links 在 NIC-400 中被配置为一个 Bridge（桥接器），包含一个输入接口和一个输出接口。

如图所示，AXI 的 AW、AR、W 通道经过数据包化处理后，可以将原本运行在 400MHz、需要 174 根信号线的接口，转换为运行在 800MHz、仅需要 64 根信号线的窄链路。

例如，对于特定读写混合的流量模型（Traffic Profile），Thin Links 可以在减少 63% 信号线的情况下，实现 100% 的原始带宽。

其核心思想是：AXI 的地址总线利用率通常不高。例如，使用 Burst Length = 4 的传输时，地址总线可能平均每四个周期才使用一次。

通过将这些事务重新打包，可以更充分地利用传输链路，减少跨越特定连接所需要的信号线数量。



# 原理解释

## 1. 为什么传统 AXI 接口存在浪费？

AXI 具有五个独立通道：

| 通道 | 功能   | 使用特点                 |
| ---- | ------ | ------------------------ |
| AW   | 写地址 | 每个 Burst 传输一次      |
| W    | 写数据 | 每个 Burst 传输多个 Beat |
| B    | 写响应 | 每个 Burst 响应一次      |
| AR   | 读地址 | 每个 Burst 传输一次      |
| R    | 读数据 | 每个 Burst 传输多个 Beat |

例如，Master 发起一个 Burst Length = 4 的写事务。

![image-20261109201427261](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-22-ThinLinks-Thin-links\image-20261109201427261.png)

可以看到：

- AW 通道利用率只有 25%。
- W 通道利用率可以达到 100%。
- 尽管 AW 空闲，其对应的物理信号线仍然存在。

这就是 AXI 并行独立通道在物理连接上的资源浪费。

## 2. Thin Links 如何解决？

核心方法：将多个 AXI 通道的信息打包，通过共享的窄链路进行传输。

![image-20261109201540901](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-22-ThinLinks-Thin-links\image-20261109201540901.png)

相当于：

- 原本为 AW、AR、W 分别铺设独立的物理总线。
- 现在利用空闲传输周期，将不同通道的信息复用到同一条较窄的链路。
- 接收端解包，重新恢复成 AXI 接口事务。

注意：Thin Links 不只是简单地把总线宽度减小，而是进行了事务打包与通道复用。

## 3. 为什么减少 63% 的信号线，还能保持 100% 带宽？

原始接口：

- 174 根线
- 400MHz

Thin Links：

- 64 根线
- 800MHz

信号线减少比例：

![image-20261109201656714](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-22-ThinLinks-Thin-links\image-20261109201656714.png)

虽然线宽减少，但工作频率提高了一倍，同时通过打包减少了空闲传输资源。

因此，对于特定的流量模型，可以维持原始接口的有效吞吐量。

注意： 这里的 100% 带宽并不意味着所有业务模式都能达到。实际吞吐量取决于读写比例、Burst Length、包头开销以及链路调度效率。





## 4. Thin Links 的优缺点

| 对比项     | 传统 AXI       | Thin Links         |
| ---------- | -------------- | ------------------ |
| 物理信号线 | 多             | 少                 |
| 工作频率   | 相对较低       | 可以更高           |
| 传输方式   | 多通道并行     | Packet 化、复用    |
| 布线拥塞   | 较大           | 较小               |
| 长距离连接 | 不够经济       | 更合适             |
| 传输延迟   | 无额外打包延迟 | 可能增加           |
| 实现复杂度 | 较低           | 需要打包、解包逻辑 |

# 总结

Thin Links 本质上是一种 AXI 链路压缩与序列化程度提升的技术，但不一定是单比特串行传输。

它解决的是 NIC-400 在长距离互连时，AXI 物理信号线数量过多、利用率偏低的问题。

需要特别区分：

- QoS/QVN： 解决多个 Master 之间的带宽分配、优先级及阻塞问题。
- Thin Links： 解决互连物理信号线数量和布线效率问题。

因此，Thin Links 主要是一种物理互连优化手段，而不是用于提高 AXI 事务仲裁能力的功能。
