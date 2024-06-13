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



---

# 同事1

## hsm的程序分成两个部分

- 第一部分是hsm romcode程序（存放在hsm rom中）

- 第二部分是hsm firmware程序（存放在dflash bank1中）

工作机制大致是：hsm reset释放之后，先执行hsm romcode的程序，用来校验firmware中的code



## build secure boot所需要的文件

- host romcode & hsm romcode

- dflash info: OTP(存放秘钥)

- dflash main: FW/firmware/hsm image/

不同的算法，就会生成不同的fw和otp



## 想要测试的点

- 能否正确等到HSM的状态/ready信号
- 能否下命令调动HSM
- host能否boot起来



---

# 同事2

## pflash info区域的参数

- verify size[31:0]：hsm需要验证的代码大小

- boot addr[31:0]：hsm需要验证的代码的起始地址

- HSM public key address：如果使用非对称HSM算法，就需要提供公钥所在的地址

- HSM code signature address：签名地址，将pflash usercode计算出来的MAC值（非对称，直接利用软件/工具计算出来的结果） 存放在某一个地址处，例如pflash info或者pflash main等等

> 签名值：
>
> - 非对称HSM算法计算出来的签名值-->签名
>
> - ​     对称HSM算饭计算出来的签名值-->MAC值

- Header size：如果不需要osr处理之后的head参与计算，此size配置为0

- Header address：

> 关于head的理解，一般编译之后，会生成pflash main区域的user code；OSR会对这个user code进行处理，
>
> 然后会在签名值和code之间添加很多其他信息，例如Code public key，code size等等--》OSR 命名为head
>
> 计算最终的code_Signature签名值的时候，有可能将head的一部分和usercode一起参与计算

{% asset_img image-20240613193648354.png %}



- Version address

> head中会有一个版本号，如果head中的版本号小于pflash info中提供的版本号，<font color=red>就不会去更新版本号??</font>

- reset disable

- Version Update enable

- CheckSum

{% asset_img image-20240613192251191.png %}



## romcode需要将整个secure boot框架搭建好，会调用HSM进行校验









## 测试步骤（FPGA侧）



1. 确定使用rsa sm2 aes sm4使用哪种HSM算法

   首先确定使用rsa sm2 aes sm4使用哪种算法，此时需要配置OTP的FW control SoC

   {% asset_img image-20240613202913302.png %}



具体32bits对应的含义，如下：

{% asset_img image-20240613203755205.png %}

{% asset_img image-20240613204224175.png %}



2. 更新HSM的秘钥



<font color=red>这个秘钥是指？作用是什么？</font>我理解是OTP的KEY区域



3. 更新固件

因为HSM默认是没有固件的（dflash main区域的code）；可以backdoor的方式放进去



4. 升级pflash usercode代码

此时host的pflash侧是没有usercode，需要编译pflash usercode，并需要将此usercode烧录到pflash中；

此时还需要利用OSR的工具计算出一个签名值（可以选择用什么算法 计算签名），得到一个`签名值+head+usercode`的pack,一起烧录到pflash 中；

然后就能得到校验参数，需要烧写到pflash info区域；



5. 烧写host romcode信息（烧写到flash进行模拟）

6. 复位
7. 读取pflash info校验参数
8. mail通信，调用hsm校验pflash user code
9. 跳转到pflash user code执行





# secure boot仿真

更新OTP和FW

























































