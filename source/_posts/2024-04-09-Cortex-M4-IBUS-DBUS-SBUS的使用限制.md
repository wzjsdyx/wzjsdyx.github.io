---
title: Cortex-M4 IBUS/DBUS/SBUS的使用限制
date: 2024-04-09 08:24:54
categories:
- Cortex-M4
tags:
- IBUS
- DBUS
- SBUS
- TBD
---

> Cortex-M4 处理器中，有三个AHB-Lite总线接口，分别是IBUS，DBUS，SBUS。对其使用限制做描述

#  IBUS

{% asset_img image-20240409082630403.png %}

- IBUS只能发出core指令访问（外部调试器不能通过IBUS发出transatcion）
- 访问空间是0x0000_0000 - 0x1FFF_FFFF
- Word对齐的访问，每次访问可能取出多条指令（Thumb-2指令集，支持16bit和32bit指令的混合使用）

 

# DBUS

{% asset_img image-20240409083158653.png %}

{% asset_img image-20240409083651092.png %}



- DBUS能发出core数据访问和调试器访问
- 访问优先级：core数据访问 > debugger访问
- 访问空间是0x0000_0000 - 0x1FFF_FFFF
- DBUS控制逻辑会将`非对齐的数据访问以及debugger访问转化为对齐的访问`
- 如果外部对DBUS和IBUS做仲裁，ARM强烈建议DBUS优先级高于IBUS优先级<font color=red>为什么要这样建议？有什么考量？发个mail，问问ARM</font>

{% asset_img image-20240409084132148.png %}



# SBUS

{% asset_img image-20240409084948169.png %}

- SBUS能发出core的指令/数据访问，以及debugger访问
- 访问地址空间0x2000_0000  - 0xDFFFFFFF 以及 0xE010_0000 - 0xFFFF_FFFF  (之间空缺的空间是私有外设空间，通过PPB访问)

- 访问优先级：core数据访问> core指令访问以及vector fetches访问 > debugger访问



---

参考博客：

1. Cortex®-M4 Technical Reference Manual  
