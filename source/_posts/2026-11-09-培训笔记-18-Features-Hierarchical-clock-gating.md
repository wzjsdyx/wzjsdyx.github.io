---
title: 培训笔记-18-Features-Hierarchical clock gating
date: 2026-11-09 16:49:59
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

原文

> Hierarchical clock gating is where you wish to clock gate the clock externally to the NIC-400 to reduce clock tree power when the interconnect is idle. It exposes a C-channel low power interface per clock domain. The NIC-400 implementation of the C-channel is compatible with the Q-channel controller, so if you have a Q-channel controller, you can connect it to the NIC-400. The way Hierarchical clock gating works is that effectively, you have a transaction monitoring logic within each clock domain, so if a clock is required, it will be indicated through the low power interface.  While the clock domain is in a clock gated state, the external and internal interfaces will be gated by de-asserting the outgoing Ready signals to prevent transactions from being accepted and lost. For example, let’s say the CLKA domain is clock gated, then Master 0 sends a transaction into that clock domain. The transaction will be stalled on the interface, but the Transaction Monitor will assert CACTIVE and request that the clock returns to that clock domain. When the clock is returned, the interface signals will be un-gated, and the transaction will be allowed to make progress through that clock domain.

![image-20261109172704101](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-18-Features-Hierarchical-clock-gating\image-20261109172704101.png)



原文翻译+图文理解

Hierarchical Clock Gating（层次化时钟门控） 是 NIC-400 提供的一种低功耗机制。

它允许在 NIC-400 外部对时钟进行门控。当互联总线处于空闲状态时，可以关闭对应的时钟，从而降低时钟树的动态功耗。

NIC-400 为每个时钟域提供一个 C-channel 低功耗接口。NIC-400 实现的 C-channel 与 Q-channel 控制器兼容，因此，如果系统中存在 Q-channel 控制器，可以将其连接到 NIC-400。

层次化时钟门控的工作原理如下：

1. 事务监控：每个时钟域内部都具有事务监控逻辑（Transaction Monitor）。当某个时钟域需要时钟时，监控逻辑会通过低功耗接口发出指示。
2. 时钟关闭：当某个时钟域处于 Clock Gated 状态时，NIC-400 会通过拉低输出的 READY 信号，对内部及外部接口进行门控，防止事务被接受后丢失。
3. 时钟唤醒：假设 CLKA 时钟域已经关闭，此时 Master 0 向该时钟域发起一笔事务。
4. 事务暂停：该事务会被阻塞在接口上，同时 Transaction Monitor 会将 `CACTIVE` 拉高，请求恢复该时钟域的时钟。
5. 事务恢复：时钟恢复后，接口解除门控，事务便可以继续通过该时钟域。

原理解释

### 1. Hierarchical Clock Gating 的核心作用

NIC-400 可以划分为多个时钟域，例如：

![image-20261109172420803](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-18-Features-Hierarchical-clock-gating\image-20261109172420803.png)



- CLKA 空闲时，外部 Clock Controller 可以关闭 CLKA。
- CLKA 有事务需要处理时，NIC-400 通过 `CACTIVE` 请求恢复时钟。
- 其他时钟域可以保持运行，不需要关闭整个 NIC-400。

这就是 Hierarchical（层次化） 的含义：按时钟域独立进行门控。

### 2. 为什么关闭时钟后，事务不会丢失？

关键是 AXI 的 `VALID/READY` 握手机制。

例如，CLKA 已经关闭，此时 Master 0 发起读事务：

| 信号    | 状态 | 含义                  |
| ------- | ---- | --------------------- |
| ARVALID | 1    | Master 发出读地址请求 |
| ARREADY | 0    | NIC-400 暂时不接收    |
| CACTIVE | 1    | 请求恢复 CLKA         |
| ARREADY | 1    | 时钟恢复后允许接收    |

当 `ARVALID=1`、`ARREADY=0` 时，握手没有完成，Master 必须保持有效请求，因此事务不会因为时钟关闭而丢失。

### 3. C-channel 的作用

NIC-400 使用三个主要信号：

| 信号      | 方向             | 作用                     |
| --------- | ---------------- | ------------------------ |
| `CACTIVE` | NIC → Controller | 指示有活动或时钟需求     |
| `CSYSREQ` | Controller → NIC | 请求进入或退出低功耗状态 |
| `CSYSACK` | NIC → Controller | 确认低功耗状态转换       |

注意：NIC-400 本身不负责直接关闭外部时钟，而是通过 C-channel 配合外部 Clock Controller 实现。



### 4. 一个重要限制：AHB-Lite

上述 READY 阻塞方式主要适用于 AXI。

AHB-Lite 接口不能完全支持相同的层次化门控机制，因为它无法像 AXI 一样利用地址通道的 READY/VALID 握手来阻止地址阶段被接受。

因此，ARM 要求相关 AHB-Lite Slave Interface 使用独立时钟域，并由系统设计者确保关闭时钟时不会丢失事务。





总结：Hierarchical Clock Gating = 按时钟域关闭空闲时钟 + 使用事务监控自动请求唤醒 + 利用协议握手机制避免事务丢失。









总结

