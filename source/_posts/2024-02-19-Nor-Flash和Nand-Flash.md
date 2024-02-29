---
title: Nor Flash和Nand Flash
date: 2024-02-19 21:59:56
categories:
- 随笔
tags:
- Nor Flash
- Nand Flash
---

# 基本概念

Nor Flash是一种非易失性存储器（non-volatile memory,NVM）技术。`存储密度低、读取速度快`和`支持随机访问`能力，适合用作程序存储器。

Nand Flash也是一种非易失性存储器技术，`存储密度高、读取速度较慢`，`只能支持顺序访问`，适合用于大存储容量的应用场景。

# Nor Flash和Nand Flash对比

|                      |         Nor Flash          |         Nand Flash         |
| :------------------: | :------------------------: | :------------------------: |
|       读写方式       |        **随机访问**        |        **顺序访问**        |
| XIP(excute in place) |             Y              |             N              |
|       读写性能       | **读取快**，写入慢，擦除慢 | **读取慢**，写入快，擦除快 |
|         容量         |       密度低，容量小       |       密度高，容量大       |
|         价格         |             高             |             低             |
|       应用场景       |   汽车电子，IoT，手机等    |     U盘，SSD固定硬盘等     |



# 参考博客

1. [Nor闪存和NAND闪存分别是什么意思？](http://www.360doc.com/content/23/0625/07/22355405_1086135834.shtml)
2. [Nor Flash和Nand Flash区别与共性](https://www.seccw.com/Document/detail/id/18321.html)

3. [FLASH XIP 概念理解](https://www.cnblogs.com/zhouxingxing7920/p/17469793.html)
