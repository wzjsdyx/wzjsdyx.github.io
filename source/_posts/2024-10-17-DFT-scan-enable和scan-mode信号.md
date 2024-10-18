---
title: DFT scan_enable和scan_mode信号
date: 2024-10-17 15:20:14
tags:
---

- scan_enable 是scan chain的组成部分，连到scan DFF的SE端，由于切换scanchain的shift mode和capture mode。  

- scan_mode不是scan chain的组成部分，而是控制芯片在正常工作模式和测试模式的切换，通常用于选择clock，reset等信号的来源。

---

项目中，正常来说，ICG的TE端连接的是scan_enable（shift阶段是打开的，capture阶段是关闭的）

但是为了提高覆盖率，ICG的TE端连接的是一个DFT IP

这是为了解决，capture阶段，寄存器 clock端口无法toggle的问题

---

参考博客：

1. [Clock gating cell 在DFT的过程中如何连接？](https://blog.eetop.cn/blog-458155-6946521.html)
1. [scan_enable和scan_mode](https://bbs.eetop.cn/thread-311203-1-1.html)

