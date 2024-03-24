---
title: 项目实践-CortexM4 Halt信号约束
date: 2024-03-24 15:58:15
categories:
- DC synthesis
tags:
- 时序约束
- HALTED信号约束
---



> 设计需求：CPU HALTED信号，是通过外部Debugger控制
>
> 外设模块也会引用这个信号进行halt，进而让外部调试器进行读取模块内部状态

{% asset_img image-20240324160122407.png %}

RTL上：外设需要将此信号作为一部信号处理，进行同步；

时序约束上：直接设置false path即可
