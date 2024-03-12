---
title: SRAM self test
date: 2024-03-12 09:31:26
categories:
- 随笔
tags:
- SRAM
- MBIST
---

> 了解SRAM的self test的机制

SRAM一般会配有MBIST电路进行自测，通常可以有两种方式就行调用：

- 第一种就是CP测试阶段，机台通过JTAG调用MBIST进行测试
- 第二种就是通过IST ROM调用MBIST进行测试（可以通过CPU下指令进行测试）

{% asset_img img_v3_028t_8bab661a-141b-453a-8a44-78b05b2a8f9g.jpg %}



