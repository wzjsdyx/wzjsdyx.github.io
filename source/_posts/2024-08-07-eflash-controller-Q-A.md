---
title: eflash controller Q&A
date: 2024-08-07 14:16:15
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---

> 记录Q&A

---

IP名字理解？



---

array选择？



---

地址译码？



---

==》Flash Macro的array是如何组织的？以及col-mux的作用？XADDR和YADDR寻址某一个WORD的过程



---

==》Flash Macro的erase操作如何实现？直接调用smart-write IP？具体如何实现？

> The smart write soft IP is need for this Flash IP. Smart write (word program & page erase) is a must for user’s application mode, including both 10K and 100K endurance array.  

<font color=blue>使用TSMC的eFlash Macro一定需要配合SMWR IP使用；因为word program以及page erase需要使用到SMWR IP</font>

当使用smart write/program算法的时候需要erase write/program操作

---

==》smart-write IP支持哪些write操作？以及这些操作对应的array？

- page erase
- <font color=red>word program？half-word program？</font>

smart-write执行这些操作的过程中需要执行program verify / erase verify

<font color=red>smart IP能够对那些array执行smart write</font>

---

Q3: 10K and 100K endurance array  这个怎么理解？

使用smart-write算法，其使用寿命是100K；
不使用smart-write算法，其使用寿命是10K；

<font color=red>可以这么理解吗？</font>

---

FMCSR (Flash Memory Command Status Register)寄存器中的具体含义：

- FMCCF
- FMCCFTF
- DBUF_LST
- DBUF_RDY
- CMD_BUSY

---

read & write rule？

1、通过option page配置

2、能保护的范围只有main区域

3、如果重新program option page区域，需要复位才能生效；

4、读写保护的配置有两种：一个是option page，另外一个是直接操作读写保护寄存器；

5、保护规则：

- 主要是让debugger不能访问；

---

smart write算法，比正常write的算法时间要长吗？



---

3’b010: SME2, to run one-shot mass-erase on main(with reden), or ifren + main+reden array

mass erase可以配置选择main或者main+reden
如何配置？



---

为什么没有用smart IP的mass erase算法？

为了节省功耗；





---

