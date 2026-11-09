---
title: 培训笔记-13-Features-Cyclic dependency avoidance schemes
date: 2026-11-09 15:50:30
categories:
- ARM
- CoreLink-NIC
tags:
- TBD

---







# 原文

> On the ASIBs, it is always required to implement one of the two Cyclic Dependency Avoidance Schemes. The choices you have are Single Slave, which is where a master can only have outstanding transactions to one slave at a time. However, when it tries to address a different slave, while the transaction is still outstanding, the different slave transaction will be stalled. It will block all the transactions on that channel behind it. The other option is to have a Single Slave per ID, which is the same as a Single Slave but only on a per ID basis. Transactions cannot be issued with the same ID to multiple slaves at the same time. But if you are using transactions with different IDs, you can target multiple different slaves. We have the trade-offs between those two options: Single Slave is the lowest performance because you will have to introduce a lot of latency if you are trying to address different targets simultaneously. But it has the smallest area and least timing impact since it is straightforward to implement and is immune to system-level dependency loops. The Single Slave per ID is the opposite. It has a far higher performance because you can target many slaves simultaneously. Still, it comes at a price of increased area, and it is vulnerable to system-wide dependency loops. Either of these is always implemented in the ASIB. You are forced to choose one of the two options. For the system-wide cyclic dependency loops, if it is within a single NIC-400, the tooling can flag this for you automatically. If it is across multiple NIC-400s, you have to mark which interfaces are dependent or independent of each other. Another option is to work it out manually by working out where the possible loops could be. If either the tooling finds the loops or you find it manually, what you can do is add the Single-Active Slave option, which is applied at the Switch's input port, which has the effect of breaking the system-wide dependency loops. The Single-Active Slave ensures that all write data has been issued past this point in the Switch before a write request is allowed to go to a different destination. The Single-Active Slave scheme doesn't significantly impact the system's performance, but it can slightly reduce your write bandwidth if you often change targets.

![image-20261109162653478](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-13-Features-Cyclic-dependency-avoidance-schemes\image-20261109162653478.png)

![image-20261109162902954](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-13-Features-Cyclic-dependency-avoidance-schemes\image-20261109162902954.png)





# 原文翻译+图文理解

【主要内容】这段内容介绍的是 NIC-400 的循环依赖避免机制（Cyclic Dependency Avoidance Schemes），

<font color=red>【核心目的】是防止多个 AXI 事务之间形成循环等待，最终导致 Deadlock（死锁）</font>

主要涉及三个机制：

1. Single Slave
2. Single Slave per ID
3. Single-Active Slave

### 1. ASIB 上的两种循环依赖避免机制

在 NIC-400 的 ASIB 中，必须实现两种循环依赖避免机制中的一种。

方案一：Single Slave（单目标 Slave）

一个 Master 在同一时刻，只允许对一个 Slave 存在未完成的事务（Outstanding Transactions）。

如果 Master 在前一个 Slave 的事务尚未完成时，尝试访问另一个 Slave，那么访问新 Slave 的事务会被暂停。

<font color=red>而且，该事务还会阻塞同一通道中排在它后面的所有事务。</font>

方案二：Single Slave per ID（每个 ID 对应单个 Slave）

该方案与 Single Slave 类似，但限制是针对每个 Transaction ID 独立实施的。

同一个 ID 的事务不能同时访问多个不同的 Slave。

但是，如果事务使用不同的 ID，就允许同时访问多个不同的 Slave。



### 2. 两种方案的性能权衡

两种方案各有优缺点：

Single Slave：

- 性能最低。如果 Master 需要同时访问不同目标 Slave，就可能引入大量等待延迟。
- 面积最小。
- 对时序的影响最小，因为实现逻辑比较简单。
- 能够避免系统级循环依赖问题。

Single Slave per ID：

- 性能更高，因为允许同时访问多个 Slave。
- 需要更多硬件资源，因此面积增加。
- <font color=red>容易受到系统级循环依赖的影响。</font>

这两种方案必须在 ASIB 中选择一种实现，不能全部关闭。

### 3. 系统级循环依赖检测

对于系统级的循环依赖：

- 如果循环依赖发生在单个 NIC-400 内部，配置工具可以自动检测并报告。
- <font color=red>如果循环依赖跨越多个 NIC-400，则需要标记不同接口之间是否存在依赖关系，或者是否彼此独立。</font>
- 另一种方式是人工分析系统中可能出现的循环依赖路径。



### 4. Single-Active Slave 机制

如果配置工具或者人工分析发现了循环依赖，可以增加 Single-Active Slave 机制。

该机制部署在 Switch 的输入端口，其作用是打破系统级循环依赖。

Single-Active Slave 保证：

在允许新的写请求切换到另一个目标 Slave 之前，之前所有的写数据必须已经通过当前 Switch 的相应位置。

这种机制一般不会明显降低系统性能，但如果写事务频繁切换目标 Slave，可能会轻微降低写带宽。

<font color=red>Single-Active Slave 机制和ASIB 上的两种循环依赖避免机制有联系吗？</font>



# 原理解释







### 1. Single Slave 与 Single Slave per ID 的区别

假设 NIC-400 连接一个 Master 和三个 Slave：

![image-20261109160725028](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-13-Features-Cyclic-dependency-avoidance-schemes\image-20261109160725028.png)

假设 Master M0 连续发出三个 AXI 读事务：

| 事务 | ARID | 目标    |
| ---- | ---- | ------- |
| T1   | 0    | Slave A |
| T2   | 1    | Slave B |
| T3   | 2    | Slave C |

情况 A：Single Slave

- T1 发给 Slave A，尚未完成。
- T2 想访问 Slave B，但必须等待 T1 完成。
- T3 被排在 T2 后面，也无法继续。

也就是说，虽然三个事务使用了不同 ID，但仍然无法同时访问三个 Slave。

情况 B：Single Slave per ID

- T1：ID=0，访问 Slave A。
- T2：ID=1，访问 Slave B。
- T3：ID=2，访问 Slave C。

三个事务可以同时 Outstanding，充分利用多个 Slave 的并行处理能力。

但是，如果 T2 也使用 ID=0，那么由于 ID=0 已经有一个访问 Slave A 的未完成事务，T2 就必须等待。

<font color=red>如果采用Single Slave per ID，实际应用中怎么保证访问不同的Slave，使用不同的ID，如果不能保证的话，即使实现Single Slave per ID，也只是面积浪费？</font>



### 2. 为什么 Single Slave per ID 可能引入死锁？

<font color=red>什么操作会引入死锁？这个是NIC IP的问题还是总线协议的问题？</font>

关键是：AXI 事务可能在不同模块、不同通道之间形成相互等待的依赖关系。

例如，概念性地假设系统存在两条依赖路径：

![image-20261109161526075](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-13-Features-Cyclic-dependency-avoidance-schemes\image-20261109161526075.png)





- 事务 A 持有部分资源，但需要等待 B 释放资源。
- 事务 B 也持有部分资源，但需要等待 A 释放资源。
- 两者互相等待，最终死锁。

以上是简化的依赖环示意，不是 NIC-400 某个固定拓扑的具体死锁案例。

Single Slave 可以通过限制访问不同 Slave 的并发性，避免这类系统级依赖环。

而 Single Slave per ID 允许不同 ID 同时访问不同 Slave，因此需要进一步检查系统级依赖关系。



### 3. Single-Active Slave 的作用是什么？

<font color=red>完全没看懂</font>

这个机制非常容易与 Single Slave 混淆，但它们不一样。

假设 Master 连续向两个不同 Slave 写数据：

| 事务    | 目标    | 数据量  |
| ------- | ------- | ------- |
| Write A | Slave A | 4 Beats |
| Write B | Slave B | 4 Beats |

Single-Active Slave 的行为：

1. 允许 Write A 的 AW 请求发往 Slave A。
2. Write A 的 W 数据依次通过 Switch 输入端口。
3. 在 Write A 的全部写数据通过该位置之前，阻止 Write B 的 AW 请求切换到 Slave B。
4. Write A 的全部写数据通过后，允许 Write B 的 AW 请求前往 Slave B。

这里有一个关键点：

Single-Active Slave 不需要等待 Write A 的 B Response 返回，而是等待写数据通过相应的 Switch 位置。

因此，相比于 Single Slave，Single-Active Slave 的限制更小，对写性能的影响也相对较低。



### 4. 三个机制的核心区别

| 对比项             | Single Slave               | Single Slave per ID      | Single-Active Slave    |
| ------------------ | -------------------------- | ------------------------ | ---------------------- |
| 实现位置           | ASIB                       | ASIB                     | Switch 输入端          |
| 限制粒度           | 每个 Master 接口的对应通道 | 每个 Transaction ID      | 写数据目标切换         |
| 同时访问多个 Slave | 不允许                     | 不同 ID 允许             | 受写数据通过情况限制   |
| 是否等待事务完成   | 是                         | 同 ID 跨 Slave 时需要    | 不需要等待 B 响应      |
| 性能影响           | 较大                       | 较小                     | 较小                   |
| 硬件面积           | 较小                       | 较大                     | 额外逻辑               |
| 主要目的           | 避免循环依赖               | 在满足约束下提高并发性能 | 打破系统级写事务依赖环 |





---



# 总结

NIC-400 实际上提供了两级循环依赖控制：

- ASIB 级别：必须在 Single Slave 与 Single Slave per ID 之间选择一种。
- Switch 级别：当系统存在潜在循环依赖时，可以进一步加入 Single-Active Slave。

因此，一个值得关注的配置组合是：

Single Slave per ID + 必要位置的 Single-Active Slave

既尽可能保留不同 ID 之间的并行能力，又通过局部限制写请求目标切换，降低死锁风险。

但需要强调：Single-Active Slave 不是对所有系统级死锁的通用修复手段，具体部署位置及是否能够消除依赖环，仍需要结合 NIC-400 的实际互联拓扑进行分析。



---



Q&A

Q&A:AXI的死锁是什么造成的？



Q&A:循环依赖？这里面提到的系统级循环依赖是什么意思？







