---
title: 培训笔记-8-Features-AMIB AXI_AHB_APB conversion
date: 2026-11-09 10:26:29
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---



# 原文

> On the AMIB side, the AXI to AHB-Lite conversion is the same as the ASIB side protocol conversion, and in that, we have two different options, AXI to AHB-Lite mirrored slave and AXI to AHB-Lite master. The difference between these two options is how you connect the NIC-400 to the rest of your AHB system. If the NIC-400 is acting as an AHB interconnect and connecting directly to an AHB slave, you will choose the AHB-Lite mirrored slave option. If the NIC-400 is acting as an AHB master connecting to an AHB interconnect, you will have to choose the AHB-Lite master option. The difference between these two options is just the signals that are exposed. When the AMIB is pretending to be an AHB-Lite interconnect, it needs to expose the HSEL signal and have an HREADY input and HREADY output, which would not be required in the AHB-Lite master case. For both of these two, there are two configurable options. The first one is to return a Decode error when unaligned addresses or sparse write transfers are received. The second option is to convert all transactions into undefined length INCR transfers. One key point to mention here for AXI to AHB-Lite conversion is that the NIC-400 intentionally does not split sparse write strobes into separate AHB-Lite transfers. If you send a sparse write transaction to an AHB-Lite slave, the NIC-400 will write out the entire transfer to the AHB-Lite slave. AHB-Lite does not support write strobes, so for example, if it sees a transaction with a size of 32 bits, it will assume that all 32 bits are being written, and the NIC-400 by itself does not take care of this. It is the reason why the AMIB has this option for returning a decode error when unaligned addresses or sparse write transfers are received. XHB-400 is a bridge that can be used instead of a NIC-400 to perform AXI to AHB-Lite if sparse write strobes are expected to target an AHB-Lite device. Finally, on conversion functionality, the AMIB has an AXI to APB conversion setting. The unique thing about the AXI to APB conversion is that you can connect up to 16 APB slaves to a single AMIB. Each of these slaves can be a different APB version, so they can be APB2, APB3, or APB4, and the NIC-400 will use the PSEL signal to choose which slave transactions are being sent to which slave. If you connect multiple APB slaves to the same AMIB, the advantage is that you get to share the conversion logic. But the downside of this is that you can only access one APB slave at a time.





![image-20261125170447148](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-8-Features-AXI-AHB-APB-conversion\image-20261125170447148.png)

{% asset_img image-20261125170447148.png %}

![image-20261125170656879](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-8-Features-AXI-AHB-APB-conversion\image-20261125170656879.png)

{% asset_img image-20261125170656879.png %}

# 原文翻译+图文理解

1、AXI 转 AHB-Lite。与 ASIB 侧类似，AMIB 侧的协议转换也提供两种接口形式。具体选择取决于 NIC-400 如何连接下游 AHB 系统。

- 如果 NIC-400 直接连接一个 AHB-Lite 从设备，由 NIC-400 承担互连侧的职责，应选择 **AHB-Lite mirrored slave（镜像从接口）**。此时，AMIB 需要输出 `HSEL`，并提供就绪信号的输入和输出。
- 如果 NIC-400 作为 AHB-Lite 主设备，接入已有的 AHB 互连，应选择 **AHB-Lite master（主接口）**。此时，由下游互连负责选择从设备，AMIB 使用标准 AHB-Lite 主接口信号。

<font color=blue>两种接口都支持两项配置：</font>

- 一项是在收到非对齐地址或稀疏写传输时，向 AXI 侧返回译码错误 `DECERR`。

- 一项是把输出到 AHB-Lite 的所有事务转换为未指定长度的递增突发 `INCR`

NIC-400 不会根据稀疏的 AXI 写字节选通 `WSTRB`，自动将一次写拆成多笔 AHB-Lite 传输。AHB-Lite 没有对应的写字节选通信号，因此，如果下游收到一笔 32 位写，就会按 32 位写入，即使 AXI 主设备原本只想更新其中几个字节。这也是 AMIB 提供 `DECERR` 报错选项的原因。如果系统需要支持发往 AHB-Lite 设备的稀疏写，可以使用 XHB-400 桥完成转换。



2、AXI 转 APB。 一个 AMIB 最多可连接 16 个 APB 从设备，每个从设备可以独立配置为 APB2、APB3 或 APB4。NIC-400 通过 `PSEL` 选中本次访问的目标。多个外设共享一套转换逻辑，可以节省面积，代价是同一 AMIB 的 APB 侧一次只能访问一个外设



# 原理解释



# 总结





