---
title: 培训笔记-16-Features-Security
date: 2026-11-09 16:48:42
categories:
- ARM
- CoreLink-NIC
tags:
- TBD
---



原文

> The NIC-400 supports TrustZone security on the ASIBs and the AMIBs. The ASIB overrides the incoming security of transactions, and the AMIB will enforce the security of transactions. If you set the ASIB as secure, all the transactions coming in from that master will be treated as secure transactions and have their AxPROT[1] value set as 0. If you set the ASIB as a non-secure, all the transactions coming from that master will be forced to be non-secure, and AxPROT[1] will be 1. These two options are useful if you do not trust the master to appropriately set the security of its transactions. Another option is to set it as per-access-security, where AxPROT values will depend on the master. This option is not available for AHB-Lite ASIBs. If the AMIB is configured as secure, only secure transactions will be allowed through to that slave, and everything else will go to the default slave. If the AMIB is configured as non-secure, then all the transactions can be allowed to go through to that slave, meaning it doesn’t care about security. The third option is boot secure, which means that the AMIB will only allow secure transactions through to that slave out of reset. After reset, you can reprogram the AMIB such that it can be non-secure, or you can leave it as secure.

![image-20261109171001834](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-16-Features-Security\image-20261109171001834.png)



# 原文翻译+图文理解

NIC-400 在 ASIB（Slave Interface Block）和 AMIB（Master Interface Block） 上都支持 TrustZone 安全机制。

其中：

- ASIB： 覆盖（Override）输入事务的安全属性。
- AMIB： 检查并强制执行（Enforce）事务的安全访问规则。

### 1. ASIB 的安全配置

ASIB 支持以下三种安全配置：

① Secure（安全模式）

如果将 ASIB 配置为 Secure，那么来自该 Master 的所有事务都将被视为安全事务，并且强制将 `AxPROT[1]` 设置为 0。

② Non-secure（非安全模式）

如果将 ASIB 配置为 Non-secure，那么来自该 Master 的所有事务都将被强制视为非安全事务，并且将 `AxPROT[1]` 设置为 1。

上述两种配置适用于不能完全信任 Master 正确设置事务安全属性的情况。

③ Per-access-security（逐事务安全模式）

在这种模式下，事务的 `AxPROT` 安全属性由 Master 自己决定，NIC-400 不会强制修改。

需要注意，AHB-Lite ASIB 不支持 Per-access-security 模式。

### 2. AMIB 的安全配置

AMIB 同样支持三种安全配置。

① Secure（安全模式）

如果 AMIB 配置为 Secure，那么只有安全事务才能访问对应的 Slave。

所有非安全事务都会被路由到 NIC-400 内部的 Default Slave。

② Non-secure（非安全模式）

如果 AMIB 配置为 Non-secure，那么所有事务（包括 Secure 和 Non-secure）都可以访问对应的 Slave。

也就是说，该 AMIB 不会根据事务的安全属性进行访问限制。

③ Boot Secure（启动安全模式）

如果 AMIB 配置为 Boot Secure，那么复位之后，AMIB 最初只允许安全事务访问对应的 Slave。

系统启动后，可以通过编程将 AMIB 重新配置为 Non-secure，也可以保持 Secure 状态。









# 原理解释

### 1. ASIB 和 AMIB 的本质区别



![image-20261109170628414](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-16-Features-Security\image-20261109170628414.png)

可以这样理解：

- ASIB 是安全属性的赋予者： 决定一笔 AXI Transaction 属于 Secure 还是 Non-secure。
- AMIB 是安全属性的检查者： 决定这笔 Transaction 是否有权限访问对应 Slave。

AXI 使用 `AxPROT[1]` 表示安全属性：

| AxPROT[1] | 含义                   |
| --------- | ---------------------- |
| `1'b0`    | Secure Transaction     |
| `1'b1`    | Non-secure Transaction |

这里 `AxPROT` 对应 `AWPROT`（写地址）和 `ARPROT`（读地址）。

### 2. 举例说明

假设 NIC-400 连接两个 Master 和两个 Slave：

- CPU：支持 TrustZone，能够发起 Secure/Non-secure 事务。
- DMA：不支持 TrustZone，无法可靠地指定安全属性。
- SRAM：存储安全数据。
- UART：普通外设。

我们可以这样配置：

| 接口      | 安全配置            | 作用                      |
| --------- | ------------------- | ------------------------- |
| CPU ASIB  | Per-access-security | 保留 CPU 发出的安全属性   |
| DMA ASIB  | Non-secure          | 强制 DMA 所有事务为非安全 |
| SRAM AMIB | Secure              | 只允许 Secure 事务访问    |
| UART AMIB | Non-secure          | 允许所有事务访问          |

最终访问结果：

| 访问路径   | 事务属性   | 结果 |
| ---------- | ---------- | ---- |
| CPU → SRAM | Secure     | 允许 |
| CPU → SRAM | Non-secure | 拒绝 |
| DMA → SRAM | Non-secure | 拒绝 |
| DMA → UART | Non-secure | 允许 |
| CPU → UART | Secure     | 允许 |

特别注意：AMIB 配置为 Non-secure，并不意味着只允许 Non-secure Transaction。

它实际上允许 Secure 和 Non-secure 两种事务通过。这一点很容易理解错误。

### 3. Boot Secure 的作用

Boot Secure 主要用于系统安全启动。

例如一个 SRAM，在系统刚刚复位时保存启动代码或安全数据，因此必须禁止非安全访问。

![image-20261109170858680](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-16-Features-Security\image-20261109170858680.png)

这里有两个关键点：

1. Boot Secure 并不是复位后自动切换为 Non-secure，而是需要软件主动配置。
2. 相应的安全配置寄存器通常通过 NIC-400 的 GPV（Global Programmers View）进行访问和编程，具体取决于生成配置。



## 三、总结

| 比较项     | ASIB                     | AMIB                      |
| ---------- | ------------------------ | ------------------------- |
| 作用       | 设置事务安全属性         | 检查事务访问权限          |
| Secure     | 强制 AxPROT[1]=0         | 只允许 Secure             |
| Non-secure | 强制 AxPROT[1]=1         | 允许 Secure 和 Non-secure |
| 第三种模式 | Per-access-security      | Boot Secure               |
| 核心目的   | 防止 Master 伪造安全属性 | 防止非法事务访问 Slave    |

一句话总结：ASIB 管理事务是什么安全级别，AMIB 管理这个安全级别能访问哪些 Slave。

另外需要明确：NIC-400 的 TrustZone 机制主要实现互联层面的访问控制，并不意味着 NIC-400 本身具备完整的 TrustZone 安全执行环境。











总结
