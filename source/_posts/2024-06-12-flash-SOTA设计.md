---
title: flash SOTA设计
date: 2024-06-12 09:11:03
categories:
- 车规MCU
tags:
- SOTA
---

> flash的SOTA设计是一种在线升级方案，这里简述下其工作机制。
>
> SOTA即软件在线升级（Software updates Over The Air），是指在不连接烧写器的情况下，通过CAN、UART或其它通讯方式，实现应用程序的更新。



# SOTA实现机制

## 常规实现方法

在进行SOTA时，需要把旧的应用程序擦除，把新的应用程序写入。

常规的实现方式需要分别开发BootLoader程序和APP程序，MCU上电先运行BootLoader，BootLoader根据情况选择是否跳转到APP和是否进行程序更新。

具体来说有以下几种方式：

- 方案一：更新程序时，由APP接收更新数据并暂存于Flash，再更改APP更新标志位；MCU重启时，BootLoader检查更新标志位，如有效，则擦除旧的APP，再将暂存于Flash的新APP数据写入APP运行地址处。
  该方案的优点是更新数据的接收由APP完成，BootLoader不需要通讯协议栈，代码量更小，且数据传输中断时，原有APP不损坏。
  缺点是需要额外的Flash空间暂存更新数据。

- 方案二：BootLoader中内置通讯协议栈，更新时，先向MCU发送指令使其跳转到BootLoader，之后先擦除旧APP，在接收新APP的同时直接将其写入Flash的APP运行地址处。
  该方案的优点是不需要额外的Flash暂存数据。
  缺点是BootLoader代码更复杂，且如果数据传输发生中断，旧的APP将不能被恢复。该方案更适合Flash容量较小的MCU。

- 方案三：将方案一和方案二相结合，即在BootLoader程序中内置通讯协议栈，更新时，先向MCU发送指令使其跳转到BootLoader，之后接收更新数据的时候，采用方案一的方法，先将数据暂存于Flash，待数据全部接收完成后再擦除旧的APP，写入新的APP。
  该方案结合了方案一和方案二的优点，且能在没有APP或APP损坏的状态下实现程序更新。
  缺点是BootLoader代码量更大，Flash空间占用更大。

- 方案四：在Flash中划分出两块相同大小的区域，分为A区和B区，都用来存放APP，但同一时间下只有一个区的APP是有效的，分别设置一个标志位标识其有效性。初始状态下先将APP写入A区，更新的时候，将新的APP写入B区，再把A区的APP擦除，同时更新两个区的有效性标志位状态。BootLoader中判断哪个区的APP有效，就跳转到哪个区运行。
  这种方法优点不需要重复拷贝APP数据，但最大的一个缺点是AB区的APP程序运行地址不同，需要分别编译，从而使得可应用性大大降低。

经过上面的分析，可以看出来每种方案都有其优缺点，对于Flash容量较小的MCU，通常采用方案二，因为没有过多的空间暂存APP更新数据。但对于MCU的Flash容量很大的，通常要先把APP暂存下来再进行更新，防止数据传输中断导致APP不可用。上面的方案一、三、四都能实现，但并不完美。

<font color=blue>如果实现的SOTA机制更类似于方案四，且Flash支持两种地址映射方式，从而使得APP编译时不需要区分AB区，使用相同的地址即可，从而避免了方案四的硬伤，就可以提供一种最佳的SOTA方案，下面描述的就是此种实现方法</font>

## 项目实现方法

程序上不用对地址做改动，硬件做地址映射

{% asset_img image-20240612095815405.png %}

在升级的时候（假设当前程序运行在bank0，`bank0处于0x0020_000,bank1处于0x0040_0000`）：

1、先擦除bank1的内容

2、更新新的APP到bank1

3、更新SOTA swap参数

4、复位，对SOTA swap参数解析，rom code跳转到bank1（`bank1处于0x0020_0000,bank0处于0x0040_0000`）执行

---



参考博客：

1. [SOTA机制详解](https://blog.csdn.net/u014157109/article/details/121006232)
