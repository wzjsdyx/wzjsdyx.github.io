---
title: SOTA + secure boot方案
date: 2024-06-12 09:53:14
categories:
- 车规MCU
tags:
- SOTA
- secure boot
---



主要有如下几个部分：

- bank0 《==》bank0校验参数页

- bank1 《==》bank1校验参数页

- SOTA swap配置页

其中：

- bank0，bank1会做硬件swap；
- bank0校验参数页，bank1校验参数页没做硬件swap，由romcode根据SOTA swap配置去读取相应的校验参数页

{% asset_img image-20240612102802897.png %}

