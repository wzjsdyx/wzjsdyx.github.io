---
title: Design of bus infrastructure components（1）
date: 2024-08-08 08:55:04
tags:
---



# Overview

在这个章节，需要初步掌握使用如下组件搭建AMBA系统架构的基本知识：

- Cortex-M3（AHB-Lite接口）

- AMBA 5 AHB

- AMBA 3 APB 



> 虽然 Cortex-M3 处理器在设计时使用了 AMBA 中 AHB 协议的 AHB Lite 版本，但在这里的示例中使用的是 AHB5 协议，因为它是最新的协议，更具有未来适应性。

<font color=blue>AMBA 5 AHB中和TrustZone 相关的机制并不会被使用，因为Cortex-M3处理器不支持；</font>

<font color=blue>如果AHB-Lite的Master想要使用AHB5总线的话，需要对总线加一层额外的wrapper，为了处理一些不需要的端口，例如互斥访问操作；</font>

## 总体架构

{% asset_img image-20240808090825499.png%}

> 1、有两个AHB5 wrapper，因此需要两个default slave
>
> 2、DNOTITRAN设置为1，IBUS和DBUS不会同时发出transaction，因此可以使用code_mux组件，无需arbiter
>
> 3、访问NVIC以及debug组件不会通过系统bus 进行路由route

## memory map

{% asset_img image-20240808091509023.png%}

> 每个外设模块分配的空间是4KB是比较常用的做法，因为在multiplex 的做法比较简单，统一用[15:12]进行decode



<font color=blue>另外需要注意，Cortex-M3和Cortex-M4处理器具有bitband特性，在利用这些特性的时候，注意冲突；</font>









































