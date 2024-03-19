---
title: 项目实践-CortexM4 WIC约束
date: 2024-03-19 23:46:58
categories:
- DC synthesis
tags:
- 时序约束
- WIC约束
- TBD
---

> 设计需求

> WIC模块，其clock信号是reg输出产生，需要create clock
>
> 在创建时钟的时候，遇到一个困惑的问题，clk mux会切换function clock以及scan clk；
>
> - scan clock是core clock（200Mhz）
>
> - wic clock是需要约束的clock，之前的sdc是约束到90Mhz
>
>   这样约束会不会有问题，怎样写才会更好？

{% asset_img p2.png %}

{% asset_img p1.png %}

交流之后，答案是将wic 的时钟约束到200Mhz，和core clock一致；因为只按照function clock约束的话，scan以200Mhz进行，会有timing问题

之前约束到90Mhz，也不会有问题，因为DC先根据function clock的约束修timing，然后会自动根据scan clock的约束修timing

<font color=red>TBD：为什么会自动根据scan clock进行约束，机制是什么？</font>

