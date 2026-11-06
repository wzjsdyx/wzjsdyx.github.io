---
title: AXI协议的sparse write
date: 2026-11-06 17:04:31
categories:
- ARM
- AMBA-AXI
tags:
- TBD
---



# sparse write

<font color=blue>稀疏写（Sparse Write）就是 AXI 写操作中，利用 WSTRB 指定只更新部分字节，而不是全部字节</font>

举个例子，假设 AXI 数据总线是 32 bits。

## 1. 正常写和稀疏写有什么区别？

假设：

```
AWADDR = 32'h0000_1000;
AWSIZE = 3'b010;       // 32-bit
WDATA  = 32'h11223344;
```

WSTRB 是 4 bits，每个 bit 对应 8 bits 数据

![image-20261106175034089](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-06-AXI协议的sparse-write\image-20261106175034089.png)



## 2. 通过一个实际写操作理解

假设 SRAM 地址 `0x1000` 原本存储：

```
原始数据：0xAABBCCDD

AXI写入：
WDATA = 0x11223344
WSTRB = 4'b0101
```

正确的 AXI 写入结果应该是：

| 字节地址 | 原始数据 | 新数据 | 是否更新 | 最终数据 |
| -------- | -------- | ------ | -------- | -------- |
| 0x1003   | AA       | 11     | 否       | AA       |
| 0x1002   | BB       | 22     | 是       | 22       |
| 0x1001   | CC       | 33     | 否       | CC       |
| 0x1000   | DD       | 44     | 是       | 44       |

正确写入后的 SRAM 数据 `0xAA22CC44`



























<font color=red>WSTRB和SIZE的关系</font>



# narrow Transfer



<font color=red>AHB-Lite的Byte Lane</font>

