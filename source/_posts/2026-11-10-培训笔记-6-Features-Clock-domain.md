---
title: 培训笔记-6-Features-IB Clock domain
date: 2026-11-09 10:24:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

# 原文

> With the NIC-400, we have several different clock relationships. So when you're configuring a design, you will specify clock domains. Between each of the domains, you will determine the relationship between those domains, which can be one of these options shown here, such as asynchronous relationships or various types of synchronous relationships. These relationships will change the type of clock domain crossing or change some of the clock domain crossing details whenever we need to perform clock domain crossing within the NIC-400. All the clock domain crossing bridges are implemented using a FIFO-pointer clock domain structure. The difference between the relationships is how many synchronization stages are inserted on the pointer paths, affecting how much latency the clock domain crossing adds. The asynchronous crossing is the worst case in terms of latency, as it requires two-pointer synchronization stages to be inserted in each direction. This option effectively gives the asynchronous crossing a round trip latency of about six cycles. We can insert fewer pointer synchronization stages for the different synchronous options, which results in less latency than in the asynchronous case. One particular case is synchronous 1:1, which doesn't require a clock domain crossing because the clock relationship is synchronous and of the same clock speed on both sides. The reason for synchronous 1:1 is if you wanted to break your design into more clock domains allowing you to do fine-grained clock gating. There is one other option here: you can specify the relationship between two clock domains as programmable. It allows you to change the clock relationship during runtime by writing to one of the registers, and it will allow some of the pointers to bypass some of the synchronization stages. So if your clock relationship changes during runtime, you can reduce the latency for a particular path by having the clock relationship programmable. For each above clock domain crossing relationship, you can achieve full bandwidth across the bridge by implementing enough buffering to cover the pointers' round-trip latency.  For example, an asynchronous crossing will have around 6 cycles of round-trip latency. So, a buffering depth of 6 will ensure that the clock domain crossing bridge is not the bottleneck.

![image-20261125161729519](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-6-Features-Clock-domain\image-20261125161729519.png)

{% asset_img image-20261125161729519.png %}

# 原文翻译+图文理解

<font color=blue>1、配置 NIC-400 时，需要先划分时钟域，再指定各时钟域之间的关系，可以是异步，也可以是几种不同的同步关系。NIC-400 会根据这个关系决定如何实现跨时钟域传输（CDC）。</font>

2、NIC-400 的跨时钟域桥都采用 FIFO 结构，通过在两个时钟域之间传递读写指针来工作。不同配置的主要区别是**指针路径上插入多少级同步器**，这会影响跨域延迟。

- 异步模式的延迟最大，因为两个方向的指针都需要经过两级同步，指针往返大约需要 6 个周期。

- 对于同步时钟，可以减少同步级数，从而降低延迟。同步 `1:1` 是一个特例。<font color=blue>两侧时钟同频且同步，因此无需额外的跨域同步。设计上仍然可以把它们划分成两个时钟域，以便分别进行时钟门控</font>



3、<font color=red>还可以把时钟关系配置为可编程。如果运行时时钟关系发生变化，软件可以写寄存器，调整指针经过的同步级数。在时钟关系允许时，指针可以旁路部分同步级，从而降低这条路径的延迟。</font>



4、对于上述各种时钟关系，只要 FIFO 有足够的缓冲空间，能够覆盖指针往返期间的传输，跨域桥就可以达到满带宽。以原文的异步场景为例，指针往返约 6 个周期，配置 6 项缓冲即可避免跨域桥成为吞吐瓶颈。



# 原理解释



# 总结

**时钟关系决定指针需要几级同步，指针往返时间决定一个 FIFO 位置多久能重用，FIFO 深度决定等待期间还能连续传多少个 beat。** 原文中“约 6 拍、深度 6”的联系，就是用 6 个位置覆盖指针反馈延迟，从而维持每拍一个 beat 的吞吐
