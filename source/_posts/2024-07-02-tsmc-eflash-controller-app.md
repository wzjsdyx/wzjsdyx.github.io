---
title: tsmc eflash controller-app
date: 2024-07-02 09:54:50
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---



#  Function Description

## features

{% asset_img image-20240702101133044.png %}

## memory organization

{% asset_img image-20240702140643117.png %}

{% asset_img image-20240702140658479.png %}

通过上述地址空间，可以访问到flash；

NOTE：

- 在flash boot mode的时候，也会将pflash地址映射到0x0000_0000地址，此时可以通过两块地址空间访问Flash；

- 在sram boot mode的时候，会将SRAM地址映射到0x0000_0000地址，此时可以通过两块地址访问SRAM；

## pflash info 0/1: option page

pflash info page0和info page 1被称为option byte；用来配置pflash和dflash main区域各个page的read以及write protect属性；

> 思考：
>
> <font color=red>memory map中提到的pflash info 2的作用是什么？</font>

## dflash info 0: OTP

dflash info page0作为OTP（one time program区域）

存放：

- lifecycle
- UID
- 秘钥
- pflash user code的校验参数
- hsm en

## dflash info 1/2: Key area

> 思考：
>
> 1、存放什么内容？作用是什么？
>
> 存放的是给客户用的密钥区域；
>
> 本质上就是划分一些page 给客户使用，之所以不用main array的原因是，flash常用mass erase操作，会将main array的数据全部擦除，处于这些密钥数据不会经常修改的角度考虑，将其划分到pflash info区域存储







## read and write protection rules



读保护说明：

- 读保护主要是防止debugger read flash；

- 正常情况下，CPU可以read flash，但是JTAG访问SRAM之后，CPU就无法访问flash（防止恶意程序在SRAM中执行，去访问flash）





> <font color=red>弄清楚读写保护条件以及读写保护规则：</font>
>
> - read protect only
> - write protect only
> - read and write protect only



# Program Guide



> 思考：
>
> 几种软件操作对应的flash macro上是哪种operation？
>
> 



## command summary

{% asset_img image-20240704104414873.png%}

{% asset_img image-20240704104447692.png%}



## program sequence

1、判断lock？

2、command is busy?

3、配置CMD_TYPE以及FMADDR

4、CMD_START：trigger

5、check command状态：是否finish以及是否有error

{% asset_img image-20240704110902369.png%}





