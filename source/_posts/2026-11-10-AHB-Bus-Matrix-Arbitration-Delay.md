---
title: CMSDK-AHB Bus Matrix Arbitration Delay
date: 2026-11-10 21:12:39
categories:
- ARM
- CoreLink-CMSDK
tags:
- TBD
---



> CMSDK 的AHB-BUS Matrix
>
> 我之前就是认为，地址阶段1T就能够选中对应的Slave，然后下一T就能完成传输。CMSDK的BUS Matrix为什么要设计成这样？



![image-20261110211956921](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-AHB-Bus-Matrix-Arbitration-Delay\image-20261110211956921.png)





<font color=blue>你的理解适用于“Master 直连 Slave”，也适用于“矩阵内部连接已经建立”的情况。CMSDK 多出的那一拍，用来建立或切换矩阵内部的连接</font>

以下都假设：没有其他 Master 竞争，目标 Slave 零等待，前一笔事务已经结束



# 1、地址直接到达 Slave

![image-20261110211459440](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-AHB-Bus-Matrix-Arbitration-Delay\image-20261110211459440.png)

时序就是你说的：

| 信号/动作       | 周期 C0    | 周期 C1      |
| --------------- | ---------- | ------------ |
| Master          | 地址阶段   | 数据阶段     |
| Slave           | 接收地址   | 完成传输     |
| Master 在周期末 | 地址被接受 | **事务完成** |

**前提是：地址在 C0 内已经穿过所有组合逻辑，到达 Slave，并满足其采样要求。**



# 2、CMSDK 的连接未建立时：地址先到矩阵，再到 Slave

<font color=blue>CMSDK 的输出端不是只做地址译码，它还要确定：这个 Slave 现在连接哪个 Master？**</font>

![image-20261110212131200](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-AHB-Bus-Matrix-Arbitration-Delay\image-20261110212131200.png)

连接原本没有建立时，时序变成：

| 信号/动作        | 周期 C0                              | 周期 C1              | 周期 C2           |
| ---------------- | ------------------------------------ | -------------------- | ----------------- |
| Master           | 地址阶段                             | 数据阶段：等待       | 数据阶段：完成    |
| Matrix           | 译码、仲裁；周期末暂存请求并建立连接 | 将暂存地址送到 Slave | 返回 Slave 的结果 |
| Slave            | 尚未收到有效地址                     | 接收地址             | 完成传输          |
| Master 的 HREADY | 1¹                                   | **0**                | **1**             |

¹ C0 的 HREADY=1 表示上一数据阶段完成，矩阵可以接受当前地址；**不代表目标 Slave 已经接受了该地址。**

> 与你代码中的行为一致：输入级发现 `active_ip=0`，暂存请求并置位 `pend_tran_reg`，随后将 Master 的 HREADY 拉低一拍。

# 3、CMSDK 的连接已经建立时，就可以直接通过

矩阵并不是每笔请求都必须重新走一遍“寄存连接选择”。

如果目标输出已经连接到这个 Master，地址就可以沿现有连接直接通过：

![image-20261110212314538](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-AHB-Bus-Matrix-Arbitration-Delay\image-20261110212314538.png)

**这就是 HSEL 两种接法产生差异的地方：**

```
此前已有连接 → Master 进入 IDLE
                    │
          ┌─────────┴─────────┐
          │                   │
 HSEL=HTRANS[1]            HSEL=1
          │                   │
 IDLE 时 HSEL=0          IDLE 时 HSEL仍为1
          │                   │
 无其他请求时撤销连接       满足条件时保留连接
          │                   │
 下次访问同一目标           下次访问同一目标
 重新建立连接，多1T          沿已有连接直接通过
```

<font color=blue>这里的“保留连接”只是保留多路选择器的选择状态，不会把 IDLE 当成读写，也不妨碍仲裁器响应其他 Master 的请求。**</font>

# **4、CMSDK 为什么要付出这一拍？**

因为如果要求新连接也在同一个地址周期内完成，就需要把下面这些工作全部塞进一拍：

```
Master 输出地址
  → 地址译码
  → 路由到目标输出端口
  → 检查多个 Master 的竞争
  → 计算仲裁结果
  → 控制输出多路选择器
  → 地址到达 Slave，满足采样建立时间
```

<font color=blue>Master、Slave 越多，这条组合路径越容易变长，从而限制最高时钟频率。</font>

CMSDK 把连接选择寄存下来，让建立新连接的工作跨到下一周期，降低这条路径的时序压力。Arm 工程师对 CMSDK 的解释也明确指出，这一拍用于避免译码、路由、仲裁、输出到目标全部挤在同一条关键路径上。[Arm：AHB Bus Matrix Arbitration Delay](https://community.arm.com/forums/f/architectures-and-processors-forum/54568/ahb-bus-matrix-arbitration-delay)

所以，这是一项取舍：

| 设计方式                     | 新连接的访问延迟 | 单周期内的组合逻辑           |
| ---------------------------- | ---------------- | ---------------------------- |
| 同周期完成译码、仲裁和转发   | 可以少一拍       | 更长，时序更难满足           |
| CMSDK 这种寄存连接选择的方式 | 通常多一拍       | 避免整条路径必须在同周期完成 |



# 5、CMSDK RTL关于Delay 1T的机制



```verilog
//===============================
// 连接建立
//===============================
输入 HSELS
    ↓
输入级 sel_ip
    ↓
译码级 sel_decX
    ↓
输出级 i_hselm
    ↓
仲裁器决定保留连接，还是置位 no_port

// 没有新请求时，保不保留当前连接，取决于输出端的 HSELM。 然后在 HREADYM=1 的时钟沿更新仲裁状态。
else if (HSELM)
  next_addr_in_port = 2'b01;
else
  next_no_port = 1'b1;

//========================================
// 连接断开以后，额外的一拍具体从哪里来？
//========================================

// no_port=1 → active_opX=0 → 输入级 active_ip=0
assign load_reg = addr_valid & HREADYS;

assign pend_tran =
    (load_reg & ~active_ip) ? 1'b1 :
    (active_ip & readyout_ip) ? 1'b0 :
    pend_tran_reg;
// 关键是第一项：“请求有效，但输出端尚未连接到我”，就将这笔请求标记为 pending。

// 时钟沿之后：
pend_tran_reg <= pend_tran;

// 返回就绪逻辑则明确规定：
else if (pend_tran_reg)
begin
  HREADYOUTS = 1'b0;
  HRESPS    = `RSP_OKAY;
end
 //这就是额外等待周期的来源。 同时，仲裁器在该时钟沿建立连接；下一周期，暂存的地址才送到目标 Slave。
```



# 7、Slave port HSEL的连接

假设：

- 此前已经建立到目标 Slave 的连接；
- 中间有一个完成的 IDLE 周期；
- 下一笔仍访问相同目标；
- 没有其他 Master 竞争，目标 Slave 零等待。

那么周期关系如下。表中每列表示一个时钟周期内的行为：

| 接法             | IDLE 周期                  | 下一笔地址周期                               | 后续周期                        | 再下一周期   |
| ---------------- | -------------------------- | -------------------------------------------- | ------------------------------- | ------------ |
| `HSEL=HTRANS[1]` | 撤销选择，在周期末断开连接 | 输入级接收请求，周期末重新建立连接并暂存请求 | 暂存地址送到 Slave，Master 等待 | 数据阶段完成 |
| `HSEL=1`         | 保留连接                   | 地址直接送到 Slave                           | 数据阶段完成                    | —            |

**前者多出的正好是 1T。**

注意，这个优化是**保留已经建立的连接**，不是“在 IDLE 时提前为所有未来请求仲裁”。复位后连接尚未建立，或者下一笔换了目标，接高也不能保证省掉第一拍。





# 附注



https://community.arm.com/forums/f/architectures-and-processors-forum/54568/ahb-bus-matrix-arbitration-delay?ReplySortBy=CreatedDate&ReplySortOrder=Ascending



![image-20261110213743921](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-10-AHB-Bus-Matrix-Arbitration-Delay\image-20261110213743921.png)



<font color=blue>这张图讨论的正是你遇到的问题：为什么没有其他 Master 竞争，CMSDK AHB Bus Matrix 仍会插入一个等待周期？**</font>

**1、提问者观察到了什么？**

提问者说：

> 我发现，通过 AHB Bus Matrix 的每笔事务，开始时都会遇到 `!HREADY`，即使没有其他 Master，也没有访问竞争。
>
> 我追踪后发现，延迟来自输出级仲裁器。我尝试了轮询和固定优先级两种仲裁方式。
>
> 这是正常现象吗？对于没有竞争的端口，有没有办法消除这个初始仲裁延迟？

这里的 `!HREADY` 表示 **HREADY 为低，Master 需要等待**

**2、Arm 工程师为什么说“无法避免”？**

他的回答是：

> 对于 AHB-Lite CMSDK BusMatrix，无论选择 round、fixed 还是 burst 仲裁，都存在这个延迟。

<font color=blue>这句话的重点是：**换仲裁策略，不能消除建立新连接需要的这一拍。**</font>

三种策略决定的是“谁先使用目标 Slave”：

| 仲裁方式 | 主要作用                             |
| -------- | ------------------------------------ |
| round    | 轮流分配访问机会                     |
| fixed    | 按固定优先级选择                     |
| burst    | 仲裁时考虑突发边界，避免打断相应突发 |

<font color=blue>而你看到的延迟，还涉及把仲裁结果写入连接选择寄存器**。即使只有一个请求者，这个寄存器仍然需要时钟沿更新</font>

因此：

```
没有竞争
   ↓
可以立即算出应该选择谁
   ↓
但建立连接仍要经过寄存器更新
   ↓
仍可能增加一拍
```

**“没有竞争”并不等于“连接已经建立”。**

**3、为什么要增加这一拍？**

工程师明确解释：当 Master 访问一个新的目标 Slave 时，需要完成：

```verilog
地址译码
   ↓
路由到对应输出级
   ↓
检查并执行仲裁 // 参考CMSDK BUS Matrix的block diagram
   ↓
通过输出选择逻辑
   ↓
将地址、控制送到目标 Slave
   ↓
满足 Slave 的建立时间要求
```

如果全部放在同一个周期里，组合路径可能很长，成为限制最高时钟频率的关键路径。

CMSDK 用额外一拍缓解这个问题。因此，这一拍是**实现时序上的设计取舍**，并非 AHB-Lite 协议强制要求。

**4、“被上一笔等待掩盖”是什么意思？**

原文还说：

> 如果当前目标 Slave 在上一笔传输的最后一个数据阶段插入等待，那么新增的仲裁周期可能被这个等待掩盖，因此不一定增加 Master 实际看到的延迟。

意思是，AHB 的地址阶段与上一笔数据阶段可以重叠，所以有些内部处理时间可以与已经存在的等待重叠：

```
情况 A：没有可重叠的等待

上一笔完成 → 建立新连接 → 下一笔继续
                 ↑
          Master 额外等这一拍


情况 B：存在可重叠的等待

上一笔：    [ 尚未完成，正在等待 ]
矩阵处理：       [ 部分切换处理 ]
                         ↓
             不一定再多出完整一拍
```

这是示意，具体重叠时机要看波形。**不能简单地把“Slave 等待拍数”和“矩阵仲裁拍数”永远直接相加。**

**6、SIE-200 和 NIC-400 那两段是什么意思？**

<font color=blue>工程师说，较新的 SIE-200 AHB5 BusMatrix** 提供了部分零延迟仲裁选项。其代价是更长的组合路径，时序更难满足。</font>

这里“零延迟仲裁”指的是**不额外插入仲裁周期**，并不代表整个读写事务零周期完成。

截图最后，提问者又问：

<font color=red>NIC-400 是否也有同样的问题？点对点、没有其他流量时，我仍然观察到了延迟。有没有高性能配置？</font>

**截图没有包含这个问题的回答。** 因此，这张图能够支持 CMSDK 的上述解释，不能用来证明 NIC-400 的延迟来源或配置方法也完全相同。

