---
title: AC7843 secure-boot flow总结
date: 2024-10-21 10:08:00
categories:
- 车规MCU
tags:
- secure boot
---



# secure boot flow

## block diagram

{% asset_img image-20241021101117881.png%}



## excute step

| step1：     hsm_status0=3_0100                          | step2：     hsm_status0=3_0101                               | step3：     hsm_status0=3_0105                               | step4：     hsm_status0=3_0115                               | step5：     hsm_status0=3_0115     h2s_info_0=a55a | step6：     hsm_status0=3_0115     h2s_info_0=a55a |
| ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- | -------------------------------------------------- |
| hsm romcode做自检，trng是否正确，算法是否能正常工作等等 | 时钟已经切换到160MHz     此时的状态是hsm romcode boot成功，然后对firmware进行校验 | hsm  boot successful，hsm romcode      will verify hsm firmware | hsm firmware verify success, wait host  command to verify pflash usercode | pflash  usercode 校验成功                          | 跳转到user  code执行                               |



# 仿真步骤

## 文件调整

1. host romcode --> host bootrom
2. hsm romcode --> hsm bootrom
3. user code --> pflash main
4. hsm firmware -->  dflash main
5. secure boot校验参数-->pflash info

> secure boot校验参数：
>
> - boot addr[31:0]
>
> - HSM veriry size[31:0]
> - HSM public key address
> - HSM code signature address
> - Header size
> - Header address
>
> 等等

4. OTP-->dflash info

> life Cycle[31:0]
> UID
> HW control
> FW control-eHSM
> FW control-SoC
> *key
> hsm enable
> secure boot enable



## 判断步骤

1、确认sota的配置（swap flag & swap flag address) 
2、host romcode从swapB读取校验参数，然后checksum；（0x0100_5000地址开始）
3、等到hsm ready ，hsm_status_0 value=0x****_0115;
4、host写寄存器s2h_info_0=32‘hdd_ff22,让hsm做校验；
5、对0x40_0000的pflash usercode进行校验，波形中确认是从bank0 main array中读取数据；
6、hsm完成对pflash usercode进行校验，h2s_info_0=0x0000_3c09;
7、host romcode跳转到用户程序执行



---

参考文档：

> /atcip/public/Poyang/OSR/20240529/Autochips_2023Q1-v1.6.9-a/doc/TRM/OSR_eHSM_LP_Technical_Reference_Manual.pdf
