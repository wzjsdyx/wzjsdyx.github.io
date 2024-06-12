---
title: secure boot
date: 2024-06-12 09:52:42
categories:
- 车规MCU
tags:
- secure boot
---

> 车规芯片中，有secure boot的选项，即对Flash中的user code进行校验，防止非法修改

大致的工作流程如下:

1、host romcode执行，此时的romcode会读取（dflash info/OTP区域）`secure boot配置`（hsm enable，secure boot enable等）；

2、如果secure boot enable，则读取（pflash info区域）`secure boot的校验参数`（verify size，boot address， HSM public key addr，HSM code signature addr等）与HSM交互，然后对Flash的特定区域进行校验；

