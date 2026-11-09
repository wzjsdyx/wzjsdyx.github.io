---
title: 培训笔记-19-Features-Programmability
date: 2026-11-09 16:50:33
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---

原文

> By default, the NIC-400 doesn’t have any programmable registers. However, to have more flexibility within your design certain features require you to have programmable registers. For example, to add a remap feature to the NIC-400, you need a programmable register within each ASIB that uses that address map. There is no way of changing the remap value without writing to the programmable registers. Fortunately, NIC-400 has optional programmable registers in various locations accessed through the global programmers view, or GPV. The GPV can be connected to the switch to be accessed from multiple masters or directly connected to an ASIB. Suppose you want to write to one of these registers. In that case, the transaction is routed to the GPV. From there, the transaction travels through an internal programming ring called the Register subsystem bus, which connects all the different registers within the system. This connection isn’t something you directly configure or implement. It exists already if you have programmable registers. If you, for example, perform a write to a register, the ring network effectively takes ownership of that Write ensuring it goes to the right register. When the BRESP is returned, it signals that the write has taken effect. The GPV map is always a fixed 1MB region. The bottom two 4KB entries are global register entries, including security, remap, and identification registers. The registers' placement within the memory map depends on where they are located within the system.The GPV memory map starts with the master interfaces 0 to 63. After that, slave interfaces 0 to 127, then there are the IB registers 0 to 61. For example, if the NIC-400 is configured with a programmable clock domain crossing and within slave interface 0, then to change these programmable registers, you have to access slave interface 0's region within the GPV memory map. The GPV memory map can be placed within the global memory map wherever you want, but it must be aligned to a 1MB address boundary.

![image-20261109174334765](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-19-Features-Programmability\image-20261109174334765-17942175749761.png)

![image-20261109174413580](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-19-Features-Programmability\image-20261109174413580.png)



![image-20261109174610833](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-19-Features-Programmability\image-20261109174610833.png)



# 原文翻译+图文理解

### 1. NIC-400 的可编程寄存器

默认情况下，NIC-400 不包含任何可编程寄存器。

但是，为了提高设计的灵活性，某些功能需要使用可编程寄存器。

例如，如果希望在 NIC-400 中增加地址重映射（Remap）功能，那么每个使用该地址映射的 ASIB 内都需要有相应的可编程寄存器。如果不写入这些寄存器，就无法改变 Remap 的配置值。

幸运的是，NIC-400 支持在多个位置配置可选的可编程寄存器，并通过 GPV（Global Programmers View，全局编程视图） 进行访问。

### 2. GPV 的连接方式

GPV 可以采用两种连接方式：

- 连接到 Switch（交换节点），允许多个 Master 访问。
- 直接连接到某个 ASIB。

假设需要写入某个可编程寄存器，那么访问事务首先会被路由到 GPV。

<font color=blue>随后，该事务通过一个称为 Register Subsystem Bus（寄存器子系统总线） 的内部编程环形网络传输。该环形网络连接 NIC-400 内部所有不同位置的可编程寄存器。这个连接不需要用户直接配置或实现。只要系统配置了可编程寄存器，该网络就会自动存在。</font>



例如，当执行一次寄存器写操作时，内部环形网络会接管该写事务，并确保事务被正确路由到目标寄存器。

当写响应 BRESP 返回时，表示对应的寄存器写操作已经生效。

### 3. GPV 的地址映射

GPV 的地址空间大小固定为 1MB。

其中，最开始的两个 4KB 地址区域用于全局寄存器，包括：

- Security（安全配置）寄存器
- Remap（地址重映射）寄存器
- Identification（标识）寄存器

其他寄存器在 GPV 地址空间中的位置，取决于它们在 NIC-400 内部所属的模块。

GPV 的地址映射顺序为：

1. Master Interface：编号 0～63。
2. Slave Interface：编号 0～127。
3. IB（Internal Block）：编号 0～61。

例如，如果 NIC-400 在 Slave Interface 0 中配置了可编程的跨时钟域（CDC）功能，那么要修改这些配置寄存器，就必须访问 GPV 地址空间中对应 Slave Interface 0 的寄存器区域。

GPV 的 1MB 地址空间可以放置在系统全局地址映射中的任意位置，但必须满足 1MB 地址对齐 的要求







# 原理解释

### 1. 为什么 NIC-400 需要 GPV？

NIC-400 的很多功能可以在生成 RTL 时静态配置，但有些功能需要在芯片运行过程中动态修改。

| 配置方式 | 特点               | 示例                             |
| -------- | ------------------ | -------------------------------- |
| 静态配置 | 在生成 RTL 时确定  | 接口数量、协议类型、拓扑         |
| 动态配置 | 软件通过寄存器修改 | Remap、部分 QoS 配置、可编程 CDC |

GPV 的核心作用就是提供统一的寄存器访问入口，让 CPU 能够修改 NIC-400 内部各个模块的运行参数。

注意：GPV 不是一个独立的 CPU，也不是通用的数据传输网络，而是 NIC-400 的配置寄存器访问接口。

### 2. GPV 如何访问分散在 NIC-400 内部的寄存器？

可以将 GPV 理解成一个统一的配置窗口。

![image-20261109173850409](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-19-Features-Programmability\image-20261109173850409.png)



假设 CPU 需要修改 ASIB0 的 Remap 寄存器：

1. CPU 向 GPV 对应的地址发起 AXI 写事务。
2. GPV 接收写请求。
3. Register Subsystem Bus 将写请求路由至 ASIB0。
4. ASIB0 内部的 Remap 寄存器完成更新。
5. 写响应 BRESP 返回 CPU。

<font color=blue>特别注意：BRESP 返回表示该寄存器写操作已经生效，但不代表所有之前发出的事务都已经完成。例如，在切换 Remap 前，通常需要由软件确保相关在途事务已经完成，避免新旧地址映射交替使用。</font>







## 三、总结

对于 NIC-400，需要掌握三个核心概念：

- GPV：统一的寄存器编程入口，CPU 通过它访问 NIC-400 内部配置寄存器。
- Register Subsystem Bus：NIC-400 自动生成的内部环形配置总线，负责将寄存器访问路由到对应模块。
- Programmable Register：分布在 ASIB、AMIB、IB 等模块中，实现 Remap、QoS 等可编程功能。

最重要的是：GPV 提供访问入口，Register Subsystem Bus 提供内部路由，真正的配置寄存器仍然位于各个功能模块内部。









































总结
