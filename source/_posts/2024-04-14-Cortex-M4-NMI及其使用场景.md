---
title: Cortex-M4 NMI及其使用场景
date: 2024-04-14 21:53:56
categories:
- Cortex-M4
tags:
- NMI
- 不可屏蔽中断
---

> Cortex-M4不可屏蔽中断的使用场景

{% asset_img image-20240414220214097.png %}

`NMI通常由看门狗定时器或者掉电检测以及PAD产生`

实际项目中，接触到的是将PAD接入到NMI中断

场景是，`当程序故障的时候，会通过按键GPIO使用最高优先级的中断功能，此时不希望被其他中断打断，然后内部进行一些操作`。



此时需要注意的是，`外部PAD在做function切换的时候，可能会有高电平，触发不希望的NMI中断`。

所以需要有个控制寄存器，enable/disable NMI，在切换的时候disable，切换完成时，enable。



