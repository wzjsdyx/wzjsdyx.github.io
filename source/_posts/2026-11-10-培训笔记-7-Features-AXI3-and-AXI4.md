---
title: 培训笔记-7-Features-IB AXI3 and AXI4
date: 2026-11-09 10:25:51
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

原文

> The NIC-400 can be configured almost entirely as AXI3 or AX4, or a mix of the two. AXI4 extends the AXI3. AXI4 allows you to have longer transactions, it removes many ports, has a new write response dependency, and AXI4 has no write data interleaving, meaning no WID signals. When converting from AXI4 to AXI3, we need the ability to break bursts, as AXI4 bursts might be longer than 16 transfers. So the incoming AXI4 transactions would need to be broken up to meet AXI3 rules. The AXI3 output is going to expect the WID signal, so the conversion logic will generate it. And there is also a logic here to prevent early B response, so it would ensure that the address has been accepted, as far as the master is concerned, before the response goes back. When converting from AXI3 to AXI4, it is a lot simpler. With AXI3 to AXI4 conversion, the transactions will always be small enough, allowing you to tie off the upper AxLEN bits. WID is left disconnected since the AXI4 does not support write data interleaving, and the NIC-400 does not accept interleaved write data on its AXI3 interfaces. AxLOCK[1] bit is left disconnected, meaning that all AXI3 locked transactions will appear to the AXI4 side as a normal transaction and return a normal OKAY response.

![image-20261125162425493](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-培训笔记-7-Features-AXI3-and-AXI4\image-20261125162425493.png)

{% asset_img image-20261125162425493.png %}



原文翻译+图文理解

1、NIC-400 可以配置为主要使用 AXI3、主要使用 AXI4，也可以混合使用两种协议。

2、AXI4 在 AXI3 的基础上扩展了突发长度，精简了部分接口信号，并增加了写响应的依赖条件。AXI4 还取消了写数据交织，因此移除了 `WID` 信号。

3、AXI4 → AXI3** 时，转换逻辑需要能够拆分突发。

- AXI4 的一次突发可能超过 16 拍，而 AXI3 的一次突发最多只有 16 拍，因此需要将长突发拆成多个符合 AXI3 规范的短突发。
- AXI3 输出接口还需要 `WID`，由转换逻辑生成。
- 此外，转换逻辑需要防止 B 通道过早返回写响应，确保从上游主设备来看，写地址已经被接收。

4、AXI3 → AXI4** 时，转换较简单。

- AXI3 的突发长度都能用 AXI4 表示，因此可以将 `AxLEN` 的高 4 位绑为 0。
- <font color=blue>AXI4 不支持写数据交织，而 NIC-400 的 AXI3 接口也不接受交织写数据，因此输入的 `WID` 可以不连接。</font >
- `AxLOCK[1]` 同样不连接，所以 AXI3 的锁定访问（locked）在 AXI4 侧表现为普通访问，成功时返回 `OKAY`。



# 原理解释





# 总结





