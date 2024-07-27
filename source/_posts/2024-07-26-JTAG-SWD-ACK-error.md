---
title: JTAG/SWD ACK=error
date: 2024-07-26 13:52:50
tags:
- TBD
---

> 在对JTAG/SWD仿真的时候，经常会遇到ACK error的情况，现在对一些会导致ACK error的场景做总结；
>
> 一般来说不是IP本身的问题，需要对时钟，timing等等做check

错误case1：判断DAP是否有时钟



错误case2：判断DP是否有transaction到AP



错误case3：判断AP是否有transaction通过总线打出



错误case4：判断JTAG/SWD的约束timing是否满足
