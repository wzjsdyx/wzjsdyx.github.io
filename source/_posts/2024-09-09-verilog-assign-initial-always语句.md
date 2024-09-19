---
title: verilog assign/initial/always语句
date: 2024-09-09 20:27:37
categories:
- verilog语法
tags:
- TBD
---





assign initial always语句三者是`并列`的关系；

其中:

- assign是数据流描述；

- initial和always都可以看成是行为级描述，只不过initial只执行一次，通常用来仿真的时候初始化信号；



变量的定义不能放在initial和always语句中；

assign也不能放在initial和always语句中；





---

参考博客：

1. [Verilog 之 initial 模块与always 模块的用法与差异](https://blog.csdn.net/weixin_74850661/article/details/134303759)
