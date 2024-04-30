---
title: B1.2 About the ARMv7-M memory mapped architecture
date: 2024-04-17 09:37:57
categories:
- ARMv7-M架构
tags:
- System Level Architecture
- TBD
---

# 内存映射架构

ARMv7-M 架构是一种<font color=blue>内存映射（memory-mapped）架构</font>。在这种架构中，<font color=blue>为处理器的内部寄存器赋予了物理地址</font>，从而提供：

- 外部中断和系统异常的入口点，即vectors
- 系统控制和配置

ARMv7-M架构通过一个中断向量表去维护vectors。

# 架构保留空间

{% asset_img image-20240418093510959.png %}

## PPB

架构reserve的地址空间0xE0000000 to 0xFFFFFFF ，本意是为system-level的软件使用。

- 第一个1MB的地址空间0xE0000000 to 0xE00FFFFF, 作为 `Private Peripheral Bus (PPB)  `使用。

See The system address map on page B3-592 for more information  

### SCS

在PPB地址空间中，架构又分配了4KB（0xE0000000 to 0xFFFFFFFF  ）的地址作为系统控制空间，支持：

- Processor ID registers  
- General control and configuration, including the vector table base address  
- System handler support, for system interrupts and exceptions  
- A system timer, SysTick.  
- A Nested Vectored Interrupt Controller<font color=blue> (NVIC)</font>, that supports up to 496 discrete external interrupts.  
- Fault status and control registers  
- The Protected Memory System Architecture, PMSAv7  
- Cache and branch predictor control  
- Processor debug  

See System Control Space (SCS) on page B3-595 for more details  

## System/Verder Specific

剩余的地址空间0xE0100000-0xFFFFFFFF，是由系统具体实现所定义，有<font color=red>some memory attribute restrictions.  </font>
