---
title: verilog timescale
date: 2024-12-04 13:52:28
categories:
- verilog语法
tags:
- TBD
---







{% asset_img image-20241205133128279.png %}







<font color=blue>xrun工具编译的文件一定要有timescale的设置</font>

- 如果    有-timescale编译选项：如果verilog文件没有timescale定义，会使用-timescale指定的默认timescale配置；

- 如果没有-timescale编译选项：后面的verilog文件如果没有timescale定义，会继承前面verilog文件的timescale配置；



<font color=blue>xrun工具相关的命令选项：</font>

`-timescale`：

xrun -timescale '<time_unit/time_precision>'

如果verilog文件没有timescale定义，会使用-timescale指定的默认timescale配置



`-override_timescale`：

xrun -timescale <time_unit/time_precision> -override_timescale



`-vtimescale`:

xrun -vtimescale time_unit/time_precision



<font color=blue>xrun工具timescale配置优先级</font>

文件定义>-timescale编译命令>-vtimescale编译命令



<font color=blue>编译器timescale和module timescale</font>

编译器是以design中最小的timescale作为仿真精度；

但是模块内部还是以模块自身的timescale进行计算；

---

参考博客：

1. [Verilog基础：编译指令`timescale](https://blog.csdn.net/weixin_45791458/article/details/134804982)
2. [不同module中的timescale问题](https://bbs.eetop.cn/thread-886985-1-1.html)

