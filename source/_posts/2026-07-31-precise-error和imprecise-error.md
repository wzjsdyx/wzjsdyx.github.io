---
title: precise error和imprecise error
date: 2026-07-31 16:13:10
tags:
- TBD
---



# imprecise和precise

PRECISERR 和 IMPRECISERR 的区别是 <font color=blue>CPU 能不能准确定位“是哪一条数据访问指令”引起了总线错误</font>

## 什么是 PRECISERR

`PRECISERR` 是 **Precise Data Bus Error，精确数据总线错误**。它表示：

> CPU 收到总线错误时，仍然清楚地知道是哪一条 `Load` 或 `Store` 指令、访问哪个数据地址导致了错误

例如：

```
LDR R0, [R1]        ; R1 = 0x01542000
ADD R2, R2, #1
```

CPU 执行 `LDR` 时向总线发起读操作：`Address = 0x01542000`

假设总线返回：`AXI RRESP = DECERR/SLVERR`

或者 AHB 返回：`HRESP = ERROR`

由于 `LDR` 必须等待读数据返回才能完成，所以 CPU 在错误发生时，通常仍停留在这条 `LDR` 指令上，可以精确定位故障来源。

```assembly
# 这时会设置：
CFSR.BFSR.PRECISERR = 1
# 并且通常还会设置：只有当 BFARVALID=1 时，BFAR 中的地址才可信
CFSR.BFSR.BFARVALID = 1
BFAR = 0x01542000
```

<font color=blue>Arm 对 `PRECISERR` 的定义就是由明确的数据读写访问造成、可以定位到具体访问地址的 BusFault</font>

## 什么是 IMPRECISERR

`IMPRECISERR` 是 **Imprecise Data Bus Error，非精确数据总线错误**。它表示：

> 总线确实报告了错误，但错误返回得比较晚，CPU 已经执行了其他指令，无法确定究竟是哪一条数据访问指令引起的。

例如：

```
STR R0, [R1]        ; R1 = 0x01542000
ADD R2, R2, #1
SUB R3, R3, #1
BL  next_function

```

在 Cortex-M7 中，`STR` 产生的写事务可能先进入：

- Store Buffer；
- Write Buffer；
- Data Cache；
- AXI Outstanding Transaction;

从 CPU 执行流水线的角度看，`STR` 可能已经完成，CPU 继续执行后面的 `ADD`、`SUB` 甚至函数调用；但是外部总线上的写操作仍在进行。

过了若干周期，总线才返回：`AXI BRESP = DECERR/SLVERR`

此时 CPU 可能已经执行到后面的指令，因此只能知道：“之前的某个写事务发生了错误”

却不能准确知道是哪一条 `STR`。

````assembly
# 这时会设置：
CFSR.BFSR.IMPRECISERR = 1
# 对于该错误，BFAR 不会给出可靠的故障地址，不能因为调试器中看到某个 BFAR 旧值，就把它当成当前故障地址
````

<font color=blue>Arm 明确指出，非精确 BusFault 通常来自较早完成的指令，异常发生时具体故障指令已经无法直接确定</font>

## 两者直观对比



| 项目                                 | PRECISERR                                                    | IMPRECISERR                                                  |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 中文含义                             | 精确数据总线错误                                             | 非精确数据总线错误                                           |
| <font color=blue>定义</font>         | Arm 对 `PRECISERR` 的定义就是由明确的数据读写访问造成、可以定位到具体访问地址的 BusFault | Arm 明确指出，非精确 BusFault 通常来自较早完成的指令，异常发生时具体故障指令已经无法直接确定 |
| <font color=blue>常见访问类型</font> | 读操作；未缓冲写操作                                         | 缓冲写、Cache写回等                                          |
| <font color=blue>异常发生时机</font> | 与故障指令同步                                               | 可能延迟若干周期                                             |
| 异常栈中的 PC                        | 通常能定位故障指令                                           | 可能已经指向后续指令                                         |
| 异常响应方式                         | 紧跟出错传输，立即响应                                       | 设置 Pending，判断优先级                                     |
| BFAR 是否可用                        | 检查 `BFARVALID`                                             | 通常不能用于定位地址                                         |
| 调试难度                             | 较低                                                         | 较高，可以通过ISB，DSB缩小范围                               |





## imprecise和precise深入理解



### 能否根据memory device类型直接判断

<font color=blue>是否产生 IMPRECISERR，关键看写操作是否在总线错误返回之前就被 CPU 认为已经完成，而不能只根据 Normal/Device/Strongly-order Memory 类型判断</font>

| 内存属性及访问行为           | 写操作是否可能提前完成 | BusFault 类型                  |
| ---------------------------- | ---------------------- | ------------------------------ |
| Bufferable Normal Memory     | 是                     | 通常 `IMPRECISERR`             |
| Non-bufferable Normal Memory | 否                     | 通常 `PRECISERR`               |
| Device Memory                | 可能                   | 可能 Precise，也可能 Imprecise |
| Strongly-ordered Memory      | 不允许缓冲             | 通常 `PRECISERR`               |
| 任意类型的读操作             | CPU 通常必须等待读数据 | 通常 `PRECISERR`               |

<font color=blue>Arm 对内存类型的说明明确指出：Device Memory 的写可以被内存系统缓冲，而 Strongly-ordered Memory 的写不能被缓冲</font>



==》<font color=blue>为什么读错误通常是精确的，写错误通常不精确</font>

对于读指令：

```
LDR r0, [r1]
```

CPU 后续指令通常依赖 `r0` 的结果，所以必须等待数据返回。

如果读失败，CPU 很快就能知道是哪条 `LDR` 出错。

而对于写指令：

```
STR r0, [r1]
```

写操作通常没有需要返回给寄存器的数据。

为了提升性能，处理器可以：

```
把写操作交给 Write Buffer
CPU 立即继续执行
Write Buffer 在后台完成总线写入
```

因此错误响应可能延迟回来。

这就是 IMPRECISERR 常见于写操作的根本原因。

==》<font color=blue>Cortex-M4是否有write buffer？</font>

Cortex-M4 的 DCode Bus 和 System Bus 上，符合缓冲条件的 Store 可以经过一个 one-entry write buffer



==》<font color=blue>需要注意Cortex-M7和Cortex-M3/M4在strongly-order memory的不同</font>

> Using Cortex-M3/M4/M7 Fault Exceptions  原文：
>
> ARM M3/M4：
> 写Strongly-ordered区域发生总线错误
> → precise
>
> M7：
> 相同写操作
> → 可能imprecise



对 Cortex-M3/M4 来说，Strongly-ordered 写<font color=blue>不会使用内部 Write Buffer 来让 Store 指令提前完成</font>。CPU 必须等待总线写响应，因此 ERROR 能绑定到原始 Store，表现为 `PRECISERR`。



Cortex-M7 的 Strongly-ordered 写确实可以进入处理器内部 Store Buffer。
 “Strongly-ordered / non-bufferable”不等于“物理上不能进入任何内部缓冲区”，而是规定这笔访问必须满足更严格的顺序、合并和可观察性规则。

Cortex-M7 TRM 明确提到：

- Store Buffer 中可以包含对 **Device 或 Strongly-ordered memory** 的 Store；
- 这类 Store 不进行合并；
- 在执行后续 Device/Strongly-ordered Load 之前，要排空相关 Store。



| 项目                    | Cortex-M3/M4                        | Cortex-M7                                      |
| ----------------------- | ----------------------------------- | ---------------------------------------------- |
| Strongly-ordered 写错误 | 保证 precise                        | 可能 imprecise                                 |
| stacked PC              | 保持原访问代码上下文                | 可能属于后来的 IRQ 上下文                      |
| 内部物理路径            | 不应仅根据 precise 推断完全绕过缓冲 | 明确存在可保存 SO/Device Store 的 Store Buffer |



==》<font color=blue>memory type和imprecise/precise是两个概念</font>

不能通过memory type类型来判断是否是imprecise/precise

不同的处理器，其write buffer的设计逻辑不同,有些memory type的访问可以经过store buffer，有些memory type的访问不经过store buffer

而且经过write buffer，也不一定是imprecise，可能error返回的时候，`STR`还在当前上下文；









### 经过 Write Buffer也不代表一定是 IMPRECISERR

Write Buffer 只是产生非精确错误的必要背景之一，不代表只要进入 Write Buffer 就一定设置 `IMPRECISERR`。

例如：

```
STR r0, [r1]
```

<font color=blue>如果总线很快返回错误，而 CPU 仍能把错误关联到这条 `STR`，可能仍报告imprecise error：</font>

```
PRECISERR = 1
BFARVALID = 1
BFAR      = r1 对应的地址
```

只有当下面的情况发生时，才属于典型的非精确错误：

```
STR 已经从处理器流水线退休
        ↓
CPU 已继续执行其他指令
        ↓
总线错误随后才返回
        ↓
CPU 无法准确关联原始 STR
        ↓
IMPRECISERR = 1
```

所以真正判断标准是：

<font color=blue>错误返回时，CPU 能否把错误精确关联到原始写指令。</font>

而不是单纯判断：是否进入过 Write Buffer



### Cortex-M4 的 DISDEFWBUF

Cortex-M4 的 `ACTLR` 中通常有：

```
ACTLR.DISDEFWBUF
```

该位用于禁止默认内存映射访问使用 Write Buffer。

典型位置是：

```
ACTLR 地址：0xE000E008
DISDEFWBUF：bit[1]
```

设置为 `1` 后：

```
#define ACTLR (*(volatile uint32_t *)0xE000E008UL)

ACTLR |= (1UL << 1);
```

处理器写操作必须等待总线事务完成，因此 BusFault 可以被精确报告；代价是写操作会阻塞流水线，性能下降。Arm 的 Cortex-M4 文档说明，设置 `DISDEFWBUF` 会禁止默认内存映射访问使用写缓冲，使 BusFault 成为精确错误，但会降低存储写性能

<font color=red>为什么有write buffer?</font>



### `volatile` 不会关闭 Write Buffer

例如：

```
*(volatile uint32_t *)addr = data;
```

`volatile` 只是在编译器层面保证：

- 不删除该访问；
- 不把该访问长期保存在寄存器中；
- 按照编译器规定发出实际的 Load/Store。

它不会告诉 Cortex-M4：不要使用硬件 Write Buffer

因此即使使用了 `volatile`，仍然可能出现：STR → Write Buffer → 延迟 Bus Error → IMPRECISERR



<font color=red>嵌入式volatile的语法使用</font>



### DMB 和 DSB 能不能使错误变成 Precise

例如：

```
*(volatile uint32_t *)addr = data;
__DSB();
```

<font color=blue>`DSB` 会等待前面的显式内存访问完成，因此可以把错误限制在较小的代码范围内，便于定位。</font>

<font color=blue>但不建议直接理解成：加了 DSB，就一定把 IMPRECISERR 转换成 PRECISERR</font>



因为原始 `STR` 可能已经从流水线退休，错误是在清空 Write Buffer 时才被发现。此时异常位置可能靠近 `DSB`，但状态仍可能反映异步写错误。

两者区别是：

```
DMB：
保证访问顺序，不一定等待访问真正完成。

DSB：
等待之前的访问完成，再执行后续指令。
```

调试时可以采用：

```
*(volatile uint32_t *)addr = data;
__DSB();
__ISB();
```

缩小故障范围；要从根本上让 Cortex-M4 的默认写操作同步化，更直接的方法是临时设置 `ACTLR.DISDEFWBUF`。





### 同一个错误地址既可能是 PRECISERR，也可能是 IMPRECISERR

假设 `0x2004_0000` 是不存在的地址。

==>情况一：读不存在的地址

```
data = *(volatile uint32_t *)0x20040000;
```

对应：

```
LDR R0, [R1]
```

CPU 必须等待读响应。总线返回错误后，CPU 知道是这个 `LDR` 导致的，因此通常是：

```
PRECISERR = 1
BFARVALID = 1
BFAR      = 0x20040000
```

 ==>情况二：写不存在的地址

```
*(volatile uint32_t *)0x20040000 = 0x12345678;
```

对应：

```
STR R0, [R1]
```

如果写事务没有经过缓冲，错误及时返回，可能是：

```
PRECISERR = 1
```

如果写事务进入 Write Buffer，CPU 继续执行，稍后才收到总线错误，则可能是：

```
IMPRECISERR = 1
```

所以：

<font color=blue>同一个地址:</font>

- <font color=blue>读访问可能是精确错误；</font>
- <font color=blue>写访问既可能是精确错误，也可能是非精确错误；</font>

> Note:
>
> Cortex-M7 具有更复杂的总线接口、写缓冲和多笔未完成事务能力，因此相比简单的 Cortex-M 内核，更容易出现需要结合总线接口分析的异步 BusFault

### 有些BUS故障不是 PRECISERR/IMPRECISERR

<font color=blue>并不是所有非法内存访问都会设置这两个位。</font>

| 故障情况             | 主要状态位              |
| -------------------- | ----------------------- |
| 指令取指总线错误     | `BFSR.IBUSERR`          |
| 精确数据访问错误     | `BFSR.PRECISERR`        |
| 非精确数据访问错误   | `BFSR.IMPRECISERR`      |
| 异常入栈时总线错误   | `BFSR.STKERR`           |
| 异常返回出栈时错误   | `BFSR.UNSTKERR`         |
| 浮点延迟状态保存错误 | `BFSR.LSPERR`           |
| MPU访问权限违规      | MemManage Fault         |
| 非对齐访问并启用Trap | UsageFault `UNALIGNED`  |
| 未定义指令           | UsageFault `UNDEFINSTR` |

<font color=blue>CFSR 中的 BusFault 状态不仅包含 `PRECISERR` 和 `IMPRECISERR`，还分别记录取指、入栈、出栈和延迟浮点保存过程中发生的总线错误</font>

<font color=red>那么准备非PRECISE和IMPRECISE Error，hardfault之后的行为模式是？</font>

自己可以在程序中读取返回地址进行判断？

