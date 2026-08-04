---
title: normal/Device/Strongly-ordered memory
date: 2026-07-31 16:13:49
tags:
- TBD
---



# Device memory和Strongly-order memory

同一个物理外设地址，在 MPU 配置不同的情况下，处理器可能采用不同的访问规则。对于 Cortex-M3/M4/M7 所属的 ARMv7-M，主要有三类内存类型：

```
Normal memory
Device memory
Strongly-ordered memory
```

其中 Device 和 Strongly-ordered 主要用于**内存映射外设、控制寄存器、FIFO 等可能有副作用的地址**。

## Device memory 是什么

Device memory 表示：

> 这个地址不是普通 RAM，而是某个设备或外设寄存器；每一次读写都可能产生硬件副作用。



### Device memory访问的规则：

1. Device memory 保证的是架构规定的访问次数、访问大小、禁止推测，以及特定内存属性组合(Device和Strongly-ordered)之间的观察顺序；

> - 两次 Non-shareable Device 访问必须保持程序顺序
>
> - 两次 Shareable Device 访问必须保持程序顺序
>
> - Non-shareable Device 和 Shareable Device 之间没有架构保证的顺序
>
> - Device 与 Strongly-ordered 之间保持顺序
>
> - Device 与 Normal memory 之间没有自动顺序保证
>
> - 其他规则：
>
>   每次访问必须使用程序指定的访问大小。
>
>   不得推测访问 Device memory。
>
>   Device memory 不可缓存。
>
>   不得把多次同地址访问合并成更少的访问。
>
>   一次程序访问不能由实现擅自重复执行。
>
>   未对齐访问 Device memory 的行为是 UNPREDICTABLE。

2. 它不自动保证 Device 与 Normal memory 的顺序，

3. 也不保证每次 Device 写返回时外设副作用已经完成。



### "保持顺序"不代表处理器必须等待写操作完全结束

这是最容易混淆的地方。

Arm 官方对 Device memory 明确说明：

> Writes to Device memory can be buffered.

也就是说，<font color=blue>Device 写允许进入写缓冲区</font>。但写缓冲或合并操作必须保持：

- 访问次数；
- 访问顺序；
- 每次访问的大小。

并且对 Device memory 不允许把多次访问合并成少于程序指定次数的访问。Device memory 也不允许推测访问，不可缓存。

因此下面两个概念不能画等号：

```
保持访问观察顺序
≠
每条 STR 都阻塞处理器，直到外设内部副作用全部完成
```

<font color=blue>处理器可以把写放进 buffer，然后继续执行，只要整个实现最终满足架构规定的观察顺序。</font>



## Strongly-ordered memory 是什么

Strongly-ordered memory 也是用于有副作用的 I/O 地址，但它的要求更加严格：

> 所有对 Strongly-ordered memory 的访问都必须按照程序顺序发生。



### Strongly-ordered memory访问的规则

> - Strongly-ordered 与 Strongly-ordered 之间严格有序
>
> - Strongly-ordered 与 Device 之间，两个方向都有序
>
> - Strongly-ordered 与 Normal memory 之间不自动有序
>
> - 其他要求：
>
>   显式访问必须使用程序指定的访问大小
>
>   实际访问次数必须是程序指定的次数
>
>   不允许推测性数据访问
>
>   Strongly-ordered 地址不能被缓存
>
>   Strongly-ordered 地址始终被视为 Shareable
>
>   未对齐访问 Strongly-ordered memory 的行为是 `UNPREDICTABLE`
>
>   对可能因异常而重新开始并重复写访问的多访问指令，手册明确提示不要用来访问 Strongly-ordered memory
>
> 



Strongly-ordered 与 Device、Strongly-ordered 访问之间有严格顺序；与 Normal memory 访问之间没有 全局顺序保证。

<font color=blue>其中“严格有序”的官方含义是较早的访问必须严格先于较晚的访问被全局观察到。</font>

<font color=blue>它不能被理解为 前一访问对应的外设内部工作必须全部完成后，处理器才能执行后一条指令。</font>



## 不同架构上对内存属性的定义（第一层：Arm 架构规范）

| 对比项                                | Armv6-M                                                      | Armv7-M / Armv7E-M                                           | Armv8-M                                                      |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **顶层内存类型**                      | `Normal`、`Device`、`Strongly-ordered`                       | `Normal`、`Device`、`Strongly-ordered`                       | `Normal`、`Device`；不再把 Strongly-ordered 作为独立命名的顶层类型 |
| **Strongly-ordered 是否为独立类型**   | 是                                                           | 是                                                           | 否；其语义由 `Device-nGnRnE` 表示                            |
| **旧架构 Device 的对应关系**          | 独立的 `Device` 类型                                         | 独立的 `Device` 类型                                         | 原 Armv7-M Device 对应 `Device-nGnRE`                        |
| **Device 子类型**                     | Device 可区分 Shareable/Non-shareable                        | Device 可区分 Shareable/Non-shareable                        | 四种：`nGnRnE`、`nGnRE`、`nGRE`、`GRE`                       |
| **Normal memory 用途**                | RAM、ROM、Flash 等普通存储器                                 | RAM、ROM、Flash 等普通存储器                                 | RAM、ROM、Flash 等普通存储器                                 |
| **Normal 是否可缓存**                 | 可以定义为 Cacheable 或 Non-cacheable；具体处理器可以不实现 Cache | 可以定义为 Cacheable 或 Non-cacheable，并可具有 Write-through、Write-back、分配策略等属性 | 仍支持 Cacheable/Non-cacheable，并通过 Inner/Outer 属性表达缓存策略 |
| **Normal 是否可共享**                 | Shareable 或 Non-shareable                                   | Shareable 或 Non-shareable                                   | Shareability 与 Normal 的缓存属性分别配置                    |
| **Normal 是否允许优化**               | 可进行缓存、预取、推测访问以及满足架构规则的重排             | 可进行缓存、预取、推测访问以及满足架构规则的重排             | 同样是允许优化的普通内存类型                                 |
| **Device 典型用途**                   | 存储器映射外设、访问有副作用的位置                           | 存储器映射外设、访问有副作用的位置                           | 存储器映射外设；通过 G/R/E 明确表示允许的优化                |
| **Device 是否可缓存**                 | 不可缓存                                                     | 不可缓存                                                     | 不可缓存                                                     |
| **Device 是否允许推测性数据访问**     | 不允许                                                       | 不允许                                                       | 不允许按 Normal memory 方式推测访问                          |
| **Device 是否必须保持访问次数和大小** | 是；不能删除或复制程序指定的访问                             | 是；必须维持程序指定的访问次数、顺序和大小要求               | 由 `G/nG` 属性明确控制是否允许 Gathering                     |
| **Device write 是否允许提前完成**     | Device write 可以被缓冲，但必须保持架构规定的访问语义        | Device write 可以被缓冲，但必须保持访问次数、顺序和大小      | 由 `E/nE` 明确表示是否允许 Early Write Acknowledgement       |
| **Strongly-ordered 是否可缓存**       | 不可缓存                                                     | 不可缓存                                                     | `Device-nGnRnE` 不可缓存                                     |
| **Strongly-ordered 的共享属性**       | 始终视为 Shareable                                           | 始终视为 Shareable                                           | 由 `Device-nGnRnE` 提供对应的严格 Device 语义                |
| **Strongly-ordered 的访问顺序**       | 显式访问遵守 Strongly-ordered 排序规则                       | 显式访问遵守 Strongly-ordered 排序规则；Strongly-ordered 访问按架构规定的程序顺序被观察 | `nR` 表示不允许重排                                          |
| **Strongly-ordered write 的提前确认** | 不允许表现成普通 Device 的提前完成写                         | 不允许表现成普通 Device 的提前完成写                         | `nE` 明确表示 No Early Write Acknowledgement                 |
| **访问合并**                          | Device/SO 不能改变程序规定的访问次数和大小                   | Device/SO 不能改变程序规定的访问次数和大小                   | `nG` 禁止 Gathering；`G` 允许 Gathering                      |
| **主要特点**                          | Armv7-M 内存模型的精简版本；不支持用多字 load/store 指令访问 Device 或 Strongly-ordered memory | 保留三种传统内存类型，并给出更完整的排序、可观察性和屏障规则 | 把旧 Device/SO 统一为 Device 子类型，用 G、R、E 三个维度精确描述行为 |
| **与下一代的对应关系**                | SO → v8-M `Device-nGnRnE`；Device → v8-M `Device-nGnRE`      | SO → v8-M `Device-nGnRnE`；Device → v8-M `Device-nGnRE`      | `nGnRnE` 和 `nGnRE` 分别继承旧 SO 和 Device 语义；`nGRE`、`GRE` 为新增的更宽松类型 |



## 具体处理器的处理方式是否不同（第一层：Arm 架构规范）



## 外部存储系统对不同memory属性的处理（第三层：SoC 外部内存系统）

