---
title: 培训笔记-15-Features-Interconnect Memory Map
date: 2026-11-09 16:32:07
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---



y

# 原文

> In the NIC-400, each ASIB can have its own address map, and you can set up the address map almost any way you want. The requirements are that address regions must not overlap within a single address map, and regions must be aligned to 4KB boundaries, as the AXI protocol does not allow transactions to cross 4KB boundaries. Undefined memory regions are routed to the internal default slave. The NIC-400 has an additional feature called remap. It allows you to switch between several predefined address maps during runtime. One of the most common uses of the remap feature is to reuse specific address ranges. For example, you may have a ROM region you boot from in the bottom 10 MB of memory. After booting, remap would allow you to reuse this address space for something else—for example, RAM. Remap is an eight-bit vector that you can set where each bit corresponds to a different remap state. If we set remap bit to zero the address map will look like the diagram on the left. When we de-assert remap bit 0 and assert remap bit 1, the top region has now changed the mapping as shown in the middle diagram. The top regions have gone from slave one to slave zero. The middle region has been unmapped by remap 1. The bottom region has now become slave one so the blank region in remap 0 now becomes a valid region. You can also combine the remap values by immediately asserting the remap bit. It creates a merged map of the different remap states, and whenever there is a conflict between the two remap states, the lower order remap bit gets priority. If both remap bits are asserted in the top region, there will be contention, as shown here. The lower order remap bit is selected, which means that slave one will occupy the top region in this remapped state.



![image-20261109165920293](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-15-Features-Interconnect-Memory-Map\image-20261109165920293.png)



# 原文翻译+图文理解

在 NIC-400 中，每个 ASIB（AXI Slave Interface Block）都可以拥有自己独立的地址映射（Address Map），并且你几乎可以按照任意方式配置地址映射。

但需要满足以下要求：

1. 在同一个 Address Map 内，不同的地址区域（Region）不能重叠。
2. 每个 Region 必须按照 4KB 边界对齐，因为 AXI 协议不允许事务跨越 4KB 边界。
3. 未定义的地址区域（Undefined Memory Regions）会被路由到 NIC-400 内部的默认从设备（Internal Default Slave）。

NIC-400 还提供了一项额外功能，称为 Remap（地址重映射）。

Remap 允许系统在运行期间（Runtime），在多个预先定义好的地址映射之间进行切换。

Remap 最常见的用途之一，就是复用特定的地址空间。

例如：

系统启动时，可能需要从地址空间底部 10MB 区域的 ROM 中执行启动程序。

启动完成后，通过 Remap 功能，可以将这段地址空间重新映射给其他存储器，例如 RAM。

Remap 是一个 8-bit 的向量（Remap Vector），其中每个 bit 对应一种不同的 Remap 状态。

当选择 Remap Bit 0 时，地址映射如左图所示。

当我们取消 Remap Bit 0，并使能 Remap Bit 1 时，地址映射将发生变化，如中间的图所示：

- 顶部 Region：原来映射到 Slave 1，现在映射到 Slave 0。
- 中间 Region：在 Remap 1 状态下取消映射（Unmapped）。
- 底部 Region：现在映射到 Slave 1，因此在 Remap 0 状态下没有映射的区域，现在变成了有效区域。

你还可以通过同时使能多个 Remap Bit，将不同 Remap 状态对应的地址映射合并。

当不同 Remap 状态之间存在地址映射冲突时，较低编号的 Remap Bit 具有更高优先级。

例如：

同时使能 Remap Bit 0 和 Remap Bit 1 时，顶部 Region 出现了地址映射冲突。

由于 Remap Bit 0 的优先级高于 Remap Bit 1，因此最终选择 Slave 1。

这意味着在合并后的地址映射状态中，顶部 Region 最终映射到 Slave 1。



# 原理解释

### 1. 每个 ASIB 都可以拥有独立的 Address Map

NIC-400 支持不同 Master 通过不同 ASIB 访问不同的地址空间。

例如：

| 地址区域    | CPU（ASIB0） | DMA（ASIB1） |
| ----------- | ------------ | ------------ |
| 0x0000_0000 | ROM          | RAM          |
| 0x2000_0000 | RAM          | RAM          |
| 0x4000_0000 | UART         | 未映射       |

这意味着：

- CPU 访问 `0x0000_0000`，可以被路由到 ROM。
- DMA 访问 `0x0000_0000`，可以被路由到 RAM。
- DMA 访问 `0x4000_0000`，如果没有定义映射，就会路由到内部 Default Slave。

重点：地址映射是从 ASIB 的视角定义的，不一定是整个 NIC-400 共享一套 Address Map。

### 2. Remap 是怎么工作的？

假设某个 ASIB 有三个地址 Region，并配置了两种 Remap 状态。

![image-20261109165659022](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-15-Features-Interconnect-Memory-Map\image-20261109165659022.png)



最终映射关系可以总结为：

| Remap[1:0] | Region A | Region B | Region C |
| ---------- | -------- | -------- | -------- |
| `01`       | Slave 1  | Slave 2  | 无效     |
| `10`       | Slave 0  | 无效     | Slave 1  |
| `11`       | Slave 1  | Slave 2  | Slave 1  |

注意：Remap Vector 并不是独热码（One-Hot）。

虽然每个 Bit 代表一组预定义地址映射，但允许多个 Bit 同时为 1。

因此，8-bit Remap Vector 不仅仅能选择 8 种独立状态，还可以组合不同状态的映射关系。

### 3. Remap 的典型应用：ROM 与 RAM 切换

例如 MCU 启动：

| 阶段      | 地址 0x0000_0000 映射到 | 作用                         |
| --------- | ----------------------- | ---------------------------- |
| Boot 阶段 | ROM                     | CPU 从 ROM 执行启动代码      |
| Boot 完成 | RAM                     | CPU 可以通过相同地址访问 RAM |

Remap 并不是将 ROM 的数据复制到 RAM，而是改变某段地址对应的目标 Slave。

对于软件而言，访问地址可以保持不变，但实际上访问的是不同的物理存储器。

实际进行 Remap 切换时，还需要考虑正在执行的指令和未完成事务，避免切换过程中访问错误的目标。

### 4. 为什么 Region 要按照 4KB 对齐？

AXI 协议规定，一个 Burst 不能跨越 4KB 地址边界。

例如：

- 从 `0x0000_0FF0` 开始，传输 16 Byte：合法。
- 从 `0x0000_0FF0` 开始，传输 32 Byte：不合法，因为跨越了 `0x0000_1000` 边界。

这里需要区分：

- AXI 协议要求：单个 Burst 不能跨越 4KB 边界。
- NIC-400 地址映射约束：Region 边界必须按照 4KB 对齐。

两者有关联，但不是完全相同的概念。







# 总结

| Feature                 | NIC-400 支持情况              |
| ----------------------- | ----------------------------- |
| Independent Address Map | 每个 ASIB 可以独立配置        |
| Region Overlap          | 同一映射状态内不允许重叠      |
| Region Alignment        | 4KB 对齐                      |
| Undefined Region        | 路由到 Internal Default Slave |
| Runtime Remap           | 支持运行时切换预定义映射      |
| Remap Vector            | 8-bit                         |
| Multiple Remap Bits     | 允许同时使能                  |
| Remap Priority          | Bit 0 最高，Bit 7 最低        |

最重要的理解：NIC-400 的 Remap 不只是切换多个地址表，它还支持将多个地址表合并；当映射冲突时，低编号 Remap Bit 优先。

这也是为什么你之前在 NIC-400 GUI 中看到的 Remap Vector 不一定是 One-Hot。



