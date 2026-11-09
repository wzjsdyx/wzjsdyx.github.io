---
title: 培训笔记-12-Features-Timing Closure
date: 2026-11-09 11:10:57
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---





# 原文

> The NIC-400 supports timing register slices inserted at several different points within the NIC-400 to break the timing paths. Due to the READY-VALID handshake, a register slice does not simply just put flops in the path, but it also implements some logic to make sure the handshake works correctly. Within the input and the output of all the ASIBs, IBs, and AMIBs, you can insert a register slice. 
>
> You are able to trap all the logic internally within each block between two register slices if you are required to, and these are referred to as slave port register slices and master port register slices on each block. The upstream interface on the block is the slave port, and the downstream interface on the block is the master port. Within each IB block, it is possible to implement a boundary FIFO, which also acts as a retiming register slice since it will break the time into the block and sit right in the middle. Additionally, within ASIBs, inserting a Single Slave per ID register slice after the cyclic dependency logic is possible for certain configurations. You may need to use this register slice if the cyclic dependency logic is very large or complicated or the NIC-400 needs to run particularly fast. And lastly, the Switches' output port can also support register slices, referred to as post-arbitration register slices. The reasoning for these slices is because we expect the arbitration logic to be some of the most timing critical for the Switch so that the post-arbitration register slices can be useful for that. Each of the register slices comes with four different options. The first one is the Full register slice. For a particular channel, this register slice will register all the signals that are passing through that point. For an AW, W, AR, R, or B channel, a full register slice will register all the signals, the VALID, the PAYLOAD, and the READY. The full register slice will always add one cycle of latency and can buffer two transfers meaning it consumes more area and power. The next one is the Forward register slice, which will only register the VALID and the PAYLOAD signal. The advantage the forward register slice has over the full register slice is size. It is half the size of a full register slice. The forward register slice has only one payload entry slot compared to the full register slice's two entry slots. The next option is the reverse register slice, which only registers the READY direction. It has the same single payload slot as a forward register slice, but the most significant advantage of the reverse register slice is it does not add any forward latency to a path. The last option is a static bypass, where there is no actual slice.



- 插入位置

![image-20261109154401993](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109154401993.png)



![image-20261109154227422](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109154227422.png)

![image-20261109154322816](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109154322816.png)



![image-20261109154517862](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109154517862.png)



- 每个register-slice都可以有四种配置

![image-20261109154749773](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109154749773.png)



# 一、原文翻译+图文理解



NIC-400 支持在内部多个不同位置插入 Timing Register Slice（时序寄存器切片），用于切断较长的时序路径。

<font color=red>由于 AXI 采用 READY-VALID 握手机制，Register Slice 并不是简单地在信号路径上插入触发器（Flops），还需要实现一些额外逻辑，以确保握手协议能够正确工作。</font>

## 1. Register Slice 的插入位置

<font color=blue>在所有 ASIB、IB 和 AMIB 模块的输入端和输出端，都可以插入 Register Slice。</font>

如果有需要，可以通过两个 Register Slice 将每个模块内部的全部组合逻辑包围起来，从而切断跨模块的组合时序路径。

这些 Register Slice 分别称为：

- `Slave Port Register Slice`：位于每个模块的 Slave Port。
- `Master Port Register Slice`：位于每个模块的 Master Port。

模块的上游接口称为 Slave Port，下游接口称为 Master Port。



其中：在每个 IB 模块内部，还可以实现` Boundary FIFO`（边界 FIFO）。

<font color=blue>Boundary FIFO 同时也能起到 Retiming Register Slice（重定时寄存器切片）的作用，因为它位于 IB 模块的中间位置，可以切断穿过该模块的时序路径。</font>



<font color=red>另外，在某些配置下，ASIB 内部还可以在 Cyclic Dependency Logic（循环依赖处理逻辑）之后插入 Single Slave per ID Register Slice。如果循环依赖处理逻辑非常庞大或复杂，或者 NIC-400 需要运行在特别高的频率下，就可能需要使用这种 Register Slice。</font>



最后，Switch 的输出端口也支持插入 Register Slice，称为 Post-Arbitration Register Slice（仲裁后寄存器切片）。设置这种 Register Slice 的原因是，我们预计仲裁逻辑会是 Switch 内部时序最关键的逻辑之一，因此 Post-Arbitration Register Slice 对改善时序非常有用。



## 2. Register Slice 的四种类型

每个 Register Slice 都提供四种不同的配置选项。

### ① Full Register Slice（完整寄存器切片）

对于特定的 AXI Channel，Full Register Slice 会对经过该位置的所有信号进行寄存。

无论是 AW、W、AR、R 还是 B Channel，Full Register Slice 都会寄存：

- VALID 信号
- PAYLOAD（有效载荷）信号
- READY 信号

<font color=blue>Full Register Slice 始终会增加一个时钟周期的延迟，并且可以缓存两笔传输。</font>

这也意味着它需要消耗更多的面积和功耗。

### ② Forward Register Slice（前向寄存器切片）

Forward Register Slice 只对以下信号进行寄存：

- VALID
- PAYLOAD

相比 Full Register Slice，Forward Register Slice 最大的优势是面积更小。

它的面积约为 Full Register Slice 的一半。

<font color=blue>Forward Register Slice 只有一个 Payload 缓存槽位，而 Full Register Slice 有两个缓存槽位。</font>

### ③ Reverse Register Slice（反向寄存器切片）

Reverse Register Slice 只对反向传输的 READY 信号进行寄存。

<font color=blue>与 Forward Register Slice 一样，它也只有一个 Payload 缓存槽位。</font>

但 Reverse Register Slice 最显著的优势是：

它不会增加前向传输路径的延迟。

### ④ Static Bypass（静态旁路）

最后一种配置是 Static Bypass。

在这种模式下，实际上不会插入任何 Register Slice。



# 二、原理解释

## 1. 为什么 NIC-400 需要 Register Slice？

核心目的：改善 Timing，缩短组合逻辑路径，提高 NIC-400 能够达到的工作频率。





## 2. READY 和 VALID 为什么不能简单地全部打拍？

AXI 使用 READY-VALID 握手：



只有在时钟上升沿，VALID 和 READY 同时为 1，才表示一次有效传输。

如果简单地将 VALID、PAYLOAD 和 READY 分别加一级寄存器，就可能导致握手状态不一致，从而发生：

- 数据丢失
- 数据重复传输
- Buffer Overflow

因此，Register Slice 还需要实现缓存、流控和握手管理逻辑。



## 3. 四种 Register Slice 的关键区别

先理解 AXI Channel 的信号方向：

![image-20261109112706104](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-12-Features-Timing-Closure\image-20261109112706104.png)

四种配置对比如下：

| 特性                | Full     | Forward  | Reverse             | Bypass |
| ------------------- | -------- | -------- | ------------------- | ------ |
| VALID 寄存          | 是       | 是       | 否                  | 否     |
| PAYLOAD 寄存        | 是       | 是       | 否                  | 否     |
| READY 寄存          | 是       | 否       | 是                  | 否     |
| 缓存槽位            | 2        | 1        | 1                   | 0      |
| 前向延迟            | +1 cycle | +1 cycle | 0 cycle（无阻塞时） | 0      |
| 切断前向组合路径    | 是       | 是       | 否                  | 否     |
| 切断 READY 组合路径 | 是       | 否       | 是                  | 否     |
| 面积开销            | 最大     | 较小     | 较小                | 最小   |

其中 Reverse 的 0-cycle 延迟，是指在没有阻塞的情况下，VALID 和 PAYLOAD 可以组合透传。

## 4. Forward 和 Reverse 的区别是什么？

这是最值得重点理解的部分。

Forward Register Slice：

```
VALID   ──[寄存器]──>
PAYLOAD ──[寄存器]──>
READY   <────────────
```

- VALID/PAYLOAD 通过寄存器传输。
- READY 为组合逻辑路径。
- 可以改善前向路径 Timing。
- 不能完全切断反向 READY 路径。



Reverse Register Slice：

```
VALID   ────────────>
PAYLOAD ────────────>
READY   <--[寄存器]──
```

- VALID/PAYLOAD 可以组合透传。
- READY 通过寄存器路径反馈。
- 可以改善 READY 路径 Timing。
- 由于 READY 反馈存在延迟，需要一个 Payload 缓存槽位，避免数据丢失。



Full Register Slice：

```
VALID   ──[寄存器]──>
PAYLOAD ──[寄存器]──>
READY   <--[寄存器]──
```

- 同时切断前向和反向组合路径。
- 使用两个缓存槽位。
- Timing 改善最全面，但面积和功耗更高。

以上是逻辑示意，不代表实际 RTL 中每条信号仅经过一个触发器。

## 5. 为什么 Full 需要两个缓存槽位，而 Forward 和 Reverse 只需要一个？

关键在于 READY 信号是否存在一个周期的反馈延迟。

以 Full Register Slice 为例：

假设内部 Buffer 已经保存了一笔数据，此时下游突然将 READY 拉低。

由于 READY 经过寄存器反馈，上游可能还没有立即感知到下游停止接收数据。

因此，上游仍可能再发送一笔数据。

这就要求 Full Register Slice 具备额外缓存能力。

而 Forward Register Slice 的 READY 没有寄存，下游的反压可以通过组合逻辑及时传递给上游，因此只需要一个缓存槽位。

Reverse Register Slice 虽然前向可以透传，但 READY 反馈有延迟，所以同样需要一个缓存槽位来暂存可能多接收的数据。



##  6. NIC-400 中如何选择register-slice？

| 使用场景                             | 推荐配置                        |
| ------------------------------------ | ------------------------------- |
| VALID/PAYLOAD 前向路径 Timing 不满足 | Forward                         |
| READY 反向路径 Timing 不满足         | Reverse                         |
| 前向和反向 Timing 都不满足           | Full                            |
| 时序宽松，希望面积和功耗最低         | Static Bypass                   |
| Switch 仲裁输出路径过长              | Post-Arbitration Register Slice |
| IB 内部需要断开时序路径并增加缓存    | Boundary FIFO                   |

# 三、总结：

Register Slice 不是单纯的数据打拍，而是带有 AXI 握手管理能力的时序优化模块。

最核心的区别是：

- Forward：切前向 VALID/PAYLOAD 路径。
- Reverse：切反向 READY 路径。
- Full：前向、反向都切，Timing 优化最全面。
- Bypass：不插入寄存器，不增加额外延迟。

对于 NIC-400 的实际配置，建议首先根据 STA 报告确定 Critical Path 属于 VALID/PAYLOAD 方向还是 READY 方向，再选择相应的 Register Slice，而不是所有位置都配置 Full。









