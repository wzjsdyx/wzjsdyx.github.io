---
title: 培训笔记-5-Features-IB-Buffering
date: 2026-11-09 10:23:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

# 原文

> Buffering can be instantiated in any of the Interface Blocks. They are implemented as FIFOs that can have a depth of up to 32 entries, where each entry can buffer one transfer. You can configure it on a per-channel basis. For example, you can choose only to have buffering on the R and the W channels and not have buffering on any of the other channels. One of the functions that buffering offers is a tidemark option.  Tidemarks stop the FIFO from issuing the write transaction downstream until a certain amount of write data has been received into the FIFO. The main idea of the tidemark option is to gather up multiple beats of write data and then issue them downstream simultaneously instead of allowing an upstream upstream device to issue the write data very slowly. In this example the Tidemark is set to 4, so four beats of write data is gathered before being sent simultaneously. Because of the NIC-400 not having support for write data interleaving, when an AW request is Because of the NIC-400 not having support for write data interleaving, when an AW request is arbitrated downstream, it effectively blocks that path for any other writes to that slave. This is because the AXI rules state that all write data must be issued to that slave before anyone else can write to that slave. If a master sends out an AW request and then sends out the data slowly, that could effectively block a path. You can use the tidemark option to gather up and send all that write data to prevent that path from being blocked. One thing to be careful about with the tidemark option is that you should not enable it unless you have a definite use for it. The reason for that is the tidemark can reduce the overall write data along a particular path. The tidemark will end up introducing bubbles in the write data stream along a path, so if you have a very slow master that is issuing write data slowly, then the tidemark can be beneficial for you. But if you are attempting to get full write data bandwidth and you would not have a need for the tidemark, you should not enable it because it will reduce the write bandwidth across a particular link.

![image-20261125155117157](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-5-Features-IB-Buffering\image-20261125155117157.png)

{% asset_img image-20261125155117157.png %}



# 原文翻译+图文理解



1、NIC-400 的各类接口块（ASIB、IB、AMIB）都可以配置缓冲。

缓冲由 FIFO 实现，最大深度为 32  entry，每个entry存放该通道的一次transfer。

可以按照通道进行设置，例如只在R和W通道设置Buffer；



2、写数据 FIFO 提供 Tidemark（水位门限）功能。它先缓存若干个写数据 beat，再把该笔写事务向下游放行。课件以 4 个 beat 为例，先把数据攒起来，再尽量连续地发出去，避免上游每隔几拍才送来一个 beat

NIC-400 不支持写数据交织。某笔写请求在下游赢得 AW 仲裁并排在前面后，发往同一 Slave 的后续写数据就要等它的 W burst 发完。如果 Master 很早就发出 AW，却迟迟送不齐 W，就可能让共享写通道空着，同时堵住后面的写事务。Tidemark 可以让这笔事务先攒数据，缩短它占住下游写通道的时间。



<font color=blue>Tidemark 也有代价。若 Master 原本就能连续提供写数据，等待攒数据可能引入空拍，降低链路的有效写带宽。它主要适合写数据来得慢、且会拖住其他 Master 的场景。</font>





# 原理解释



# 重点&总结

<font color=blue>Tidemark 的价值是让慢 Master 晚一点占住共享 W 通道。</font>

假设 M0 每隔 4 拍才产生一个 W beat，M1 的数据已经准备好，两者都写同一个 Slave。若 M0 先取得下游写顺序，M1 的 W 就得等 M0 的 `WLAST` 握手，即使中间有空拍也插不进去。



<font color=blue>若在共享仲裁前给 M0 配置 Tidemark，M0 攒数据期间，不会让M0的AW参与仲裁</font>。

M1 就有机会先发。下游 `WREADY` 持续为 1 时，M0 攒齐的 4 个 beat 可以在 **连续 4 拍**发完。

AXI4 允许后续 AW 提前握手并排队，限制的是 W beat 不能跨 burst 交织。

