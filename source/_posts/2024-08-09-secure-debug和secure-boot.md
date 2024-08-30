---
title: secure-debug和secure-boot
date: 2024-08-09 15:39:55
tags:
---

> secure debug

在hsm enable的时候，此时必须要等到hsm debug en信号assert之后，外部调试器才能访问MCU资源；

hsm debug en assert的条件是调用hsm算法完成鉴权；



这部分鉴权操作和secure boot中，利用hsm算法对pflash usercode进行校验是独立的；
