---
title: eFlash controller学习路线
date: 2024-08-15 09:21:22
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---



1、eFlash user mode

各种工作机制 & RTL仿真

RTL详细分析



2、eFlash test mode





# design review学习整理

DRS文档；

详细设计文档；

design review PPT；



---

## DRS

pflash drs

1、软件的verify命令是：

page erase verify；

mass erase verify；



2、

page program可以配置的长度（最大长度是一个page，最小长度是一个word）

page erase verify命令可以配置的长度（最大长度是一个page，最小长度是一个word）





3、page program

program之前会有verify命令执行，可以通过软件寄存器配置关闭，但是关闭之后，无法保证program是否成功；



4、读写保护规则

可以通过option byte区域，也可以通过寄存器配置



5、external reset pin control



dflash DRS和pflash DRS基本相同





## Flash IP

PFlash  256KB, DFlash 128KB

{% asset_img image-20240815201154560.png%}

![image-20240815201154560](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-15-eFlash-controller学习路线\image-20240815201154560.png)



### read操作

电压范围有要求：正常工作电压0.81V~1.21V

以0.99为档位：

- 小于0.99V，Tcyc=40ns
- 大于0.99V，Tcyc=25ns；

> 项目中eFlash的使用场景是大于0.99V的（ss corner）；
>
> 进入低功耗mode的时候，eFlash controller和eFlash Macro都会掉电；







### program操作

Macro的program操作每次支持的是36bits

因此一个word program需要执行两次macro program



### page erase

### mass erase

mass erase和page erase的主要区别是需要将WHV参数设置成0xC





### RECALL操作

flash实际上可以看成是一个模拟的macro，recall操作就是将flash 的trim值读取出来



### reference cell erase操作

针对样片进行操作；



# smart-write IP

可以直接实现page program和page_erase操作

可以提高endurance

{% asset_img image-20240815203928579.png%}

![image-20240815203928579](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-15-eFlash-controller学习路线\image-20240815203928579.png)



> 思考：
>
> <font color=red>为什么只用这几种smart-write算法？</font>
>
> 可能是直接follow原来项目的做法，也可能是有其他原因；

## workspaceflow

SME、SMP

# BIST IP





# Architecture Design

<font color=red>flash组织中，info0的page3 ？？</font>

## eFlash组织规划

## PFlash Info规划



## DFlash Info规划

dflash info区域更多的是和HSM相关，首先看HSM对Flash的要求：

1、hsm在Flash中会占用一些区域

- OTP：
  - 存放一些key，校验参数（image start ，image size）以及chip lock state；
  - 存放在dflash info0 page0区域

- Key area：
  - 存放一些custormer key
  - 存放在dflash info0 page1和page2
- romcode area：PFlash main array
- image area：PFlash main array

![image-20240815210009026](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-15-eFlash-controller学习路线\image-20240815210009026.png)

2、这些区域，在不同的生命周期中，对各个Master有着不同的访问要求



3、mass erase

如果pflash中存放了romcode以及image area，如果执行mass erase：

- RAW生命周期，mass erase命令可以通过执行macro 的mass erase实现
- 其他life cycle，mass erase命令可以通过macro的page erase实现（因为pflash存放的rom以及image区域不能被擦除）

> 各个生命周期是什么意思？

romcode起始地址always是0地址；

key area有两个，通过valid flag标记有效位，

## Flash支持的命令



## 读写保护机制

核心就是防止jtag修改或者读取flash main区域的内容



## Flash FS（function support）

- 命令支持
- 读写保护机制
  - 写保护可寄存器配置
  - 读保护-》非读保护：触发Dflash/Pflash mass erase
  - 
- read while write
  - 读pflash的时候可以写dflash
  - 反之也可
- flash size可配置
  - pflash：256/128/64KB
  - dflash：128/64KB
- LVD检测，低电压检测，当SPM给flash一个低电压信号的话，<font color=red>当前的program或者erase的操作会直接结束，因为这些操作对电压有要求；datasheet有要求</font>
- abort command
- HSM相关区域权限管控
- cache，硬件自动清除cache（当发生flash write操作的时候）
- ECC
- EIM
- BIST

> 思考：
>
> 当LVD的时候，状态机需要停止当前的操作，因为Macro database上，program或者erase操作，对工作电压是有要求的；

## 硬件架构

flash是放在MTCMOS电源域，因此在vlps或者standy mode的时候会断电（controller和Macro都会断电）



flash top ctrl：主要是处理顶层的信息

AHB itf：处理bus的transaction，主要有两个cache

apb itf：控制和状态寄存器

pflash_ctrl主要是处理除了flash接口时序相关的之外的所有控制（读写管控，命令控制）

pflash_itf主要是实现macro级别的时序



## Design and Implementation



cache是2路组相连的设计？

自研？添加了很多断言？



flash寄存器读写都是脚本生成的？









































































