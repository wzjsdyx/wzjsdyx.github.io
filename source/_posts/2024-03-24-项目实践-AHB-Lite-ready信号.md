---
title: 项目实践 AHB-Lite ready信号
date: 2024-03-24 16:08:11
categories:
- AHB-Lite总线
tags:
- AHB-Lite ready信号
- TBD
---

> 前言：
>
> input看hready_in
>
> output看hready_out
>
> 
>
> •hready_in： When HIGH, the HREADY signal indicates to the master and all slaves, that the previous transfer is complete.
>
> •主要用于多个slave的情况，防止协议发生错误
>
> 
>
> •hready_out： When HIGH, the HREADYOUT signal indicates that a transfer has finished on the bus. 
>
> •表示slave的transfer已经传输完成，可以进行下一次transfer



# 单master和单slave

针对单master和单slave的情况，AHB-Lite slave侧的hready_in可以直接连接到hready_out。

如下图是Cortex-M4的IBUS和DBUS的连接原理图

{% asset_img image-20240324163320779.png %}



# 单master和多slave

针对单master和多slave的情况，需要`将所有的slave hready_out相与`然后送给master和各个slave。如下是连接原理图

<font color=red>TBD：从bus mastrix代码中验证这一点</font>

{% asset_img image-20240324165100671.png %}

## 实例说明

下图红色的ready信号是相与之后的结果

{% asset_img image-20240324164813564.png %}





# AHB-Lite组件ready接口说明

`基于CMSDK`

纯粹的AHB-Lite Master只有一个hready_in信号，例如CPU的DBUS接口；

{% asset_img image-20240324170333211.png %}



AHB-Lite Slave都会有hready_in和hready_out信号，例如sram-bridge；

{% asset_img image-20240324170337137.png %}



CMSDK中`BUS_mastrix`的Master port和Slave port都会有hready_in和hready_out信号。

{% asset_img image-20240324170234848.png %}



CMSDK中`sync_birdge`的Master Port就类似于CPU的DBUS接口，只有hready_in信号；Slave port会有hready_in和hready_out信号

{% asset_img image-20240324170227485.png %}



# AHB-Lite组件连接示意

{% asset_img image-20240324170428088.png %}

