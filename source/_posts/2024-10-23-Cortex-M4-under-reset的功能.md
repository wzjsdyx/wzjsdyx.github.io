---
title: Cortex-M4 under reset的功能
date: 2024-10-23 15:00:06
categories:
- ARM
- Cortex-M4
tags:
---

under reset功能简介：*under reset功能是指debugger会在芯片在reset状态下连接芯片，一般是debugger拉住芯片的reset pin，然后通过jtag与芯片进行基本的通讯建立，如通过dp获取芯片内核型号，通过配置 core debug 寄存器在reset释放后halt住。*

*应用场景主要是程序逻辑有问题，导致芯片一执行程序就会无法连接芯片（jtag pin被复用为其他功能，或芯片进入低功耗模式），通过under reset可以在程序执行前设置reset catch，使得reset释放后，程序pc停在一开始的地方，不会执行后面的程序。此时可以单步调试程序或者直接擦除程序。*
