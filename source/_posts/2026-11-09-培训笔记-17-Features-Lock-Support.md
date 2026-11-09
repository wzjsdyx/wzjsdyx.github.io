---
title: 培训笔记-17-Features-Lock Support
date: 2026-11-09 16:49:17
categories:
- ARM
- CoreLink-NIC
tags:
- TBD

---







# 原文

> Lock support is a function in AXI3 where a particular master can effectively block access to a specific slave until it decides to unlock it. When configuring the tooling, the user will have an option for a particular interface to enable lock support. If enabled, the tooling will create the right arbitration logic in the switches to enforce this lock behavior. If the interface does not have lock support, then the tooling does not generate the lock support logic.

![image-20261109171806375](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-17-Features-Lock-Support\image-20261109171806375.png)

# 原文翻译+图文理解

Lock Support（锁定支持） 是 AXI3 协议中的一项功能，允许某个 Master（主设备）锁定对特定 Slave（从设备）的访问，从而有效阻止其他 Master 访问该 Slave，直到该 Master 决定解除锁定。

在使用 NIC-400 配置工具时，用户可以选择是否为某个接口启用 Lock Support。

- 启用 Lock Support：工具会在内部 Switch（交换节点）中生成相应的仲裁逻辑，以确保锁定行为得到正确执行。
- 禁用 Lock Support：工具不会生成用于支持锁定功能的逻辑，从而减少硬件资源消耗。



# 原理解释

### 1. Lock Support 的作用是什么？

假设 NIC-400 连接两个 Master 和一个 Slave：

![image-20261109171541818](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-17-Features-Lock-Support\image-20261109171541818.png)

正常情况下：

- Master A 和 Master B 都可以访问 Slave，NIC-400 根据仲裁规则分配访问权限。
- 当 Master A 获得 Slave 的 Lock 后，Master B 即使发起访问请求，也无法在锁定期间获得对该 Slave 的访问权限。
- 当 Master A 解除 Lock 后，Master B 才能继续访问。

核心目的：保证 Master A 对 Slave 的一组操作不被其他 Master 的访问插入。



### 2. AXI3 如何实现 Lock？

AXI3 使用 `AxLOCK[1:0]` 表示事务的锁定属性：

| AxLOCK  | 含义                |
| ------- | ------------------- |
| `2'b00` | Normal，普通访问    |
| `2'b01` | Exclusive，独占访问 |
| `2'b10` | Locked，锁定访问    |
| `2'b11` | Reserved，保留      |

注意：Locked 和 Exclusive 是两种不同的机制。

- Locked：通过互联仲裁机制阻止其他 Master 访问相关 Slave。
- Exclusive：通过独占访问监视机制支持原子读改写操作，不要求阻止其他 Master 访问。

AXI4 已经取消了 Locked Transaction，但仍然支持 Exclusive Access。

### 3. NIC-400 为什么需要专门配置 Lock Support？

原因在于：Lock 不只是 Master 发出一个 AxLOCK 信号，还需要 NIC-400 的仲裁器配合。

假设 Master A 发起 Locked Transaction：

1. NIC-400 识别 `AxLOCK = 2'b10`。
2. Switch 的仲裁逻辑记录锁定关系，保证 Master A 的锁定访问不被其他 Master 插入。
3. Master A 继续执行锁定事务。
4. Master A 按 AXI3 规则发出解锁事务，NIC-400 完成相应的解除锁定处理。
5. 其他 Master 恢复对 Slave 的正常访问。

因此，启用 Lock Support 会增加一定的仲裁控制逻辑。

## 三、设计上的注意事项

对于 NIC-400：

| 配置                 | 硬件影响             | 适用场景                         |
| -------------------- | -------------------- | -------------------------------- |
| Enable Lock Support  | 增加锁定仲裁逻辑     | 需要支持 AXI3 Locked Transaction |
| Disable Lock Support | 减少逻辑面积和复杂度 | 不需要 Locked Transaction        |

特别注意：

1. AXI3 的 Locked Transaction 与 AXI4 的 Exclusive Access 不同。
2. 不支持 Locked Transaction 的 AXI4 Master，通常不需要启用此功能。
3. Master 锁住的是目标 Slave 的访问仲裁，并不是锁住整个 NIC-400 网络。

总结：NIC-400 的 Lock Support 本质上是在 Switch 仲裁器中增加 AXI3 Locked Transaction 所需的控制逻辑，确保某个 Master 在锁定期间获得对目标 Slave 的受保护访问。







总结



