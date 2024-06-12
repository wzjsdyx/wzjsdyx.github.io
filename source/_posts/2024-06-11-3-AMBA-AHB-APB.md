---
title: 3 AMBA/AHB/APB
date: 2024-06-11 21:26:50
categories:
- SoC设计
tags:
- AMBA
- AHB
- APB
- TBD
---



# AHB概述

## AHB版本

## AHB信号

## AHB基本操作

在单Master和多Slave的简单设计中，AHB总线系统可以按照下图设计：

{% asset_img image-20240611215804055.png %}

<font color=red>data phase mux control?什么意思？怎么实现？</font>

如果遵循`AMBA 5 AHB`规范，可以按照如下方式连接：

- 多个`AHB Master`共享一个bus，需要使用`Master Multiplexer`和`Master arbiter`
- 多个`AHB Slave`返回的data和response信号需要使用`Slave Multiplexer` feedback to `AHB Master`

---

信号可以被划分为`address phase `信号和`data phase `信号：

`address phase` signals include:

- HADDR, HTRANS, HSEL, HWRITE, HSIZE.
- Optional: HPROT, HBURST, HMASTLOCK, HEXCL, HAUSER  

`data phase` signals include:

- HWDATA, HRDATA, HRESP, HREADY (and HREADYOUT).

- Optional: HEXOKAY, HWUSER, HRUSER.  

<font color=blue>每一个transfer都是有address phase和data phase</font>

<font color=blue>transfer是pipelined，意味着当前transfer的address phase可以和上一次transfer的data phase overlap</font>

{% asset_img image-20240611221656976.png %}

---

<font color=blue>每个phase的都是由`当前activated AHB Slave data phase的hreadyout assert`而结束;</font>

<font color=red>这一块不是很明白他的描述，还需要再理解一下</font>

来自`AHB Slave的HREADYOUT信号`经过`Slave Multiplexer`，形成了`system-wide HREADY`信号；

此Slave Multiplexer在transfer的数据阶段工作，其控制信号可以通过AHB decoder/HSEL以及HREADY信号生成；

{% asset_img image-20240611223909464.png %}

<font color=red>这一块电路对应的时序是什么样子？</font>

如果某一个AHB Slave当前没有被select，其HREADYOUT应该是h

igh，表明其状态是ready的。

该从机仅仅只能在上一次transfer完成的时候（由system-wide的HREADY high表示），再进行新的transfer

{% asset_img image-20240611230046424.png %}



<font color=red>没太懂这两段想要表达什么意思？需要结合具体的代码实现看</font>
