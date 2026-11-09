---
title: 培训笔记-14-Features-Arbitration
date: 2026-11-09 16:23:27
categories:
- ARM
- CoreLink-NIC
tags:
- TBD

---









# 原文

> The arbitration scheme in the NIC-400 is always Least Recently Granted everywhere. The traffic is split into different priority groups based on the AXI QoS value assigned to the transaction. Within these priority groups, we are going to use the Least Recently Granted to pick between them. You can assign QoS priority values in several different ways. You can fix them during configuration time. You can assign a value but make it programmable during runtime, so through writes to NIC-400 registers. And lastly, you can change the QoS value for a particular ASIB.  Another option is to allow the QoS value to be passed externally from the interface by the master, so the master is allowed to drive its own QoS value. There are also QoS regulators that can dynamically change the QoS based on latency or other traffic priority requirements, which we will explain in detail later. For the read data and write response channels, they arbitrate using the Least Recently Granted scheme. Since NIC-400 does not support write data interleaving, the write channel always follows what the address channels are doing.



![image-20261109164308004](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-14-Features-Arbitration\image-20261109164308004.png)



# 原文翻译+图文理解

<font color=blue>NIC-400 中所有仲裁位置均采用 Least Recently Granted（LRG，最久未获得授权优先） 仲裁算法。</font>



根据事务携带的 AXI QoS 值，将流量划分为不同的优先级组（Priority Groups）。在同一优先级组内部，使用 LRG 算法选择获得授权的事务。

QoS 优先级可以通过以下几种方式设置：

1. 固定配置（Fixed QoS）：在 NIC-400 配置阶段，为事务分配固定的 QoS 值。
2. 运行时可编程（Programmable QoS）：配置一个初始 QoS 值，并允许软件在运行期间通过写入 NIC-400 的寄存器修改该值。
3. ASIB QoS 调整：可以针对特定 ASIB 修改其 QoS 值。
4. 外部传入（External QoS）：允许 Master 通过接口直接提供 QoS 值，即由 Master 自己决定事务的 QoS 优先级。
5. QoS 调节器（QoS Regulator）：根据延迟或其他流量优先级要求，动态调整 QoS 值。后面会详细介绍该功能。



对于读数据通道（R）和写响应通道（B），同样使用 LRG 算法进行仲裁。

由于 NIC-400 不支持写数据交织（Write Data Interleaving），因此写数据通道（W）的调度始终遵循写地址通道（AW）所确定的事务顺序。



# 原理解释

### 1. 什么是 LRG 仲裁？

LRG = Least Recently Granted，即最久没有获得授权的请求优先。

例如，三个 Master 同时访问同一个 Slave，且 QoS 优先级相同。

| 仲裁轮次 | 授权 Master | 说明                        |
| -------- | ----------- | --------------------------- |
| 第1次    | M0          | 假设初始选择 M0             |
| 第2次    | M1          | M1、M2 比 M0 更久未获得授权 |
| 第3次    | M2          | M2 最久未获得授权           |
| 第4次    | M0          | 重新选择 M0                 |
| 第5次    | M1          | 继续轮转                    |

上述示例假设三个 Master 始终有有效请求、仲裁状态初始顺序为 M0→M1→M2。

LRG 的主要作用：保证相同优先级下的公平性，避免某个 Master 长期得不到授权。

### 2. QoS 和 LRG 如何配合？

假设有四个 Master：

| Master | QoS 值 | 优先级组 |
| ------ | ------ | -------- |
| M0     | 8      | 高       |
| M1     | 8      | 高       |
| M2     | 4      | 低       |
| M3     | 4      | 低       |

假设高优先级组优先于低优先级组，那么仲裁逻辑如下：

![image-20261109163827719](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-14-Features-Arbitration\image-20261109163827719.png)

这里实际上存在两个层次：

- 组间仲裁：根据 QoS 优先级决定哪个组先获得服务。
- 组内仲裁：通过 LRG 选择该组内的 Master。

注意：如果高优先级请求持续存在，低优先级请求可能长时间等待。LRG 本身不能保证不同 QoS 组之间的公平性。



### 3. AXI 五个通道分别怎么仲裁？

| AXI 通道  | 仲裁机制  | 说明                     |
| --------- | --------- | ------------------------ |
| AW 写地址 | QoS + LRG | 根据优先级和 LRG 仲裁    |
| AR 读地址 | QoS + LRG | 根据优先级和 LRG 仲裁    |
| W 写数据  | 跟随 AW   | 不支持写数据交织         |
| R 读数据  | LRG       | 多个读响应来源竞争时仲裁 |
| B 写响应  | LRG       | 多个写响应来源竞争时仲裁 |

重点理解 W 通道：

假设 M0、M1 都要向同一个 Slave 写数据。

- AW 仲裁先选择 M0，再选择 M1。
- W 通道先传输 M0 当前写事务的数据，直到该事务最后一个数据 Beat。
- 然后才能传输 M1 的写数据。

不能在同一个写 Burst 中间切换到另一个 Master 的数据。

注意，这不代表 AW 和 W 必须在同一个时钟周期传输，只是要求写数据调度遵守对应事务的顺序约束。





## 三、核心总结

NIC-400 仲裁机制可以归纳为：

QoS 决定优先服务哪一组，LRG 决定同组内优先服务哪个 Master。

其中：

- AW、AR：QoS 优先级分组 + LRG。
- R、B：LRG。
- W：跟随 AW 的写事务顺序，不进行写数据交织。
- QoS 可以静态设置、运行时编程、从 Master 传入，也可以由 QoS Regulator 动态调节。

补充： LRG 并不完全等同于 Round-Robin（轮询仲裁）。在所有请求持续有效时，两者可能表现出相似的轮转结果，但 LRG 是依据历史授权顺序决定优先级。
