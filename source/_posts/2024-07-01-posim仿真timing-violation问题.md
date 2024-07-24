---
title: posim仿真timing violation问题
date: 2024-07-01 23:41:58
categories:
- IC设计流程
- posim
tags:
- posim
- timing violation
- TBD
---



第一种：sync电路第一级不用care，添加async list





---



case1：sync电路第一级

不用care，添加async list

---



case2：module内部reg发生了timing，但是module输出没有X态

这种X太不会传播，不应影响到功能；

---



case3：module reg发生了timing violation，X态会发生传播

例如外部reset拉低的时候，可能会导致时钟产生毛刺（例如，dtc mux切换导致）

SMIC工艺的寄存器，碰到这样的毛刺时钟，就会导致timing violation，然后会有X态传播；
