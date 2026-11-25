---
title: 培训笔记-4-Features-IB Data width
date: 2026-11-09 10:22:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

# 原文

> The data width conversion is either going to be upsizing or downsizing. The primary purpose of these two blocks is to allow wider or narrower buses to connect into the rest of the network. Other than the fact that one is widening the data bus and the other is narrowing it, their functionality is very similar. Both of them will attempt to optimize the number of transactions that go through the sizing logic. This optimization will only apply to modifiable transactions, meaning transactions to normal memory. If we have a 32-bit bus coming in and a 128-bit bus coming out, the Upsizer will repack the data to efficiently use the full data bus if the transaction is marked as modifiable. Here we have an incoming 32-bit transaction of length 4 that takes 4 cycles to transfer, and at the output, we have a 128-bit transaction of length 1, taking 1 cycle to transfer. The Downsizer also has the same functionality, but because we are going from a wider data bus to a narrower data bus, optimization can only occur when sending in transactions that are sub data bus word. Considering this example, let's say you send in a 32-bit transaction of length 4, that would get optimized by the Downsizer to be a 64-bit transaction of length 2, even though it's downsizing.



![image-20261125154223834](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-4-Features-IB-Data-width\image-20261125154223834.png)

{% asset_img image-20261125154223834.png %}

# 原文翻译+图文理解

数据位宽转换分为Upsizing和Downsizing，用于将不同位宽的总线接入互连网络。两者的功能很相似，都会尝试重新打包数据，以减少传输拍数。<font color=blue>这种优化只适用于标记为 **Modifiable（可修改）** 的事务，例如普通内存访问。</font>



假设输入总线为 32 bit，输出总线为 128 bit。如果事务可修改，Upsizer 就可以把输入端 **4 拍、每拍 32 bit** 的数据合成输出端 **1 拍、每拍 128 bit**，充分利用输出总线的位宽。

Downsizer 也能进行类似的优化。虽然输出总线更窄，但如果输入每拍的数据量足够小，输出的一拍仍能装下输入的多拍。例如，输入端 **4 拍、每拍 32 bit** 的事务，可以合成输出端 **2 拍、每拍 64 bit**。



# 原理解释

- 不是简单改变 Data Bus 位宽，而是：通过调整 AXI Burst 的每拍传输大小和长度，尽可能提高数据总线的利用率。

- 只有 Modifiable Transaction 才能优化，这是由 AXI 协议的 Memory Attribute 决定的。AXI4 通过 `AxCACHE[1]` 标识事务是否为 Modifiable。

  - `AxCACHE[1] = 1`：允许互联在协议规定范围内调整事务属性，例如 Burst 长度、Size。
  - `AxCACHE[1] = 0`：不能随意改变这些属性。

  例如访问普通 SRAM/DDR 的可修改事务，可以通过合并连续地址上的数据提高效率。但是访问具有副作用的外设寄存器时，不能任意合并访问，否则可能改变设备行为。





# 总结

