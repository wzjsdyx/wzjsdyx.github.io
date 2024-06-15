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

作用是让hsm romcode对hsm的firmware进行校验





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



# doc->eHSM secure boot TRM

## Secure Boot, Install and Upgrade

The eHSM supports firmware stored in ROM, internal NVM or external NVM, 

and also supports the firmware excuted in IROM, internal IRAM or internal NVM.  

<font color=blue>This section describs one of the combinations that eHSM firmware is stored and excuted in internal NVM.</font>



### eHSM Secure Boot, Install and Upgrade

<font color=blue>eHSM Secure Boot: Verify eHSM firmware in NVM before excuting</font>

•Verify the eHSM firmware image w/ ”eHSM FW Verify Key”

•Excute the code in NVM

<font color=blue>eHSM Secure Install: Install eHSM firmware to NVM</font>

•Decrypt and verify the eHSM firmware upgrade image w/ ”eHSM Upgrade Encrypt Key” and”eHSM Upgrade Verify Key”

•Sign the eHSM firmware code w/ ”eHSM FW Verify Key”

•Save the plaintext code image to NVM

<font color=blue>eHSM Secure Upgrade: Upgrade eHSM firmware to NVM</font>

•Similar flow as secure install

> 一些思考：
>
> <font color=red>或者说install和upgrade的区别是什么?</font>

{% asset_img image-20240614134053043.png %}



#### eHSM Code Location

{% asset_img image-20240614134651421.png %}



#### Encryption Algorithm

{% asset_img image-20240614134840559.png %}



#### Encryption OTP Keys

{% asset_img image-20240614135113981.png %}



#### OTP Algorithm Selection Field

{% asset_img image-20240614135258415.png %}



### eHSM Image Format

#### eHSM Image Format

The eHSM image includes code image and upgrade image

{% asset_img image-20240614135539302.png %}



#### eHSM Image Fields Definition

- Code Image Fields Definition

{% asset_img image-20240614161410855.png %}

> Note:
>
> •The ”Code_Signature” covers 432+254K bytes, from ”Code_Valid_Flag” to the end of the ”Code"

- Upgrade Image Fields Definition

{% asset_img image-20240614161506218.png %}

> Note:
>
> •The ”Upgrade_Signature” covers 448+255K bytes data, including the whole upgrade imagefields except ”Upgrade_Signature” and ”Upgrade_Public_Key”.
>
> •The whole 255K bytes code image is encrypted with ”eHSM_UPGRADE_ENC_KEY”



### eHSM Secure Boot

#### Secure Boot Flow

1. HW loads OTP information, decrypts OTP keys(Optional) and initializes system before releasingeHSM’s CPU reset

2. FW decryption and verification can be skipped in lifecycle ”TEST_MODE” and ”DEVELOP_MODE”,controlled by OTP control fields

3. eHSM will report error if there is no available FW code to info host and wait host to burn FW code through mailbox

4. eHSM will reports error if secure boot fails to info host and wait for host to debug through mailboxor UART port, including:
   1. (a)Crypto IP self-test fail
   2. (b)Key (eHSM FW Verify Key) slot is not available
   3. (c)Program (NVM code) verification fail
   4. (d)Version number mismatch (OTP is newer than NVM)

{% asset_img image-20240614162500741.png %}

{% asset_img image-20240614163102775.png %}

{% asset_img image-20240614163132301.png %}



> Note:
>
> - Program verification flow depends on the algorithm, it’s CMAC for symmetric algorithm and Hash & RSA (or SM3 & SM2) for asymmetric algorithm.
>
> - BOOTDONE_SUCCESS and BOOTDONE_FAIL means:
>
>   - BOOTDONE_SUCCESS: 
>
>     - BOOTDONE=1 and BOOTERR=0
>
>     - BOOTDONE_FAIL: BOOTDONE=1 and BOOTERR=1
>
>       Both BOOTDONE and BOOTERR are register fields in ”SYS_BOOT_STA”, and is routedto eHSM top output pin ”o_ehsm_status”The ”Report Error” is defined in EMU registers, and will be monitored through the eHSM’soutput pin ”o_ehsm_fw_err” by host
>
> 一些思考&说明：
>
> <font color=red>crypto IP self test是什么意思？</font>
>
> 



#### Secure Boot Related OTP Fields

{% asset_img image-20240614163634621.png %}



### eHSM Secure Install

#### Secure Install Related Keys

{% asset_img image-20240614135113981.png %}



#### Secure Install Flow



The secure install only contains following three steps:

##### step1: Burn install FW image to NVM

1.Burn `install FW image` to NVM

(a)Host prepares the FW install image and sends an INSTALL command to eHSM to start the FW install

(b)Once eHSM bootloader receives the INSTALL command, it starts following steps 

​	i.Check the version with version counter in OTP

​	ii.Verify and decrypt the install image

​		(a) Verify the image with ”eHSM_UPGRADE_VERIFY_KEY”

​		(b)- For asymmetric algorithm

​				(i) Calculate the Hash of the image

​				(ii) Verify the signature of the image Hash value

​			- For symmetric algorithm

​				(i) Verify the signature of the image

​		(c)Decrypt the image with ”eHSM_UPGRADE_ENC_KEY”

​	iii.Compare the version counter between code image and upgrade image, report error if they are different

​	iv.(Optional) If Boot_Alg is symmetric algorithm: Calculate the code image MAC with”eHSM_VERIFY_KEY” and update the ”Code_Signature” field with the 		calculated MAC

> Note: This step is only needed when each part has its own ”eHSM_VERIFY_KEY”which is different from others’

​	v.Write image to internal NVM

​	vi.Return command response with ”UPGRADE_DONE” (or corresponding error code)

##### step2: Verify installed Program in NVM

2.Verify installed Program in NVM

(a)Host sends VERIFY_PROGRAM command to start the program verification

(b)Once eHSM receives the VERIFY_PROGRAM command, it starts to verify the program inNVM, including

​	i.Compare the program version with OTP version counter

​	ii.Verify the program with ”eHSM_VERIFY_KEY”

​	iii.Return command response with ”REV_MATCH” or expected warning code ”REV_NEW”(Verification pass, but NVM program is newer than or same as OTP version counter)

##### step3: Update OTP version counter

3.Update OTP version counter

(a)Host resets the eHSM

(b)Bootloader jumps to the internal NVM after verification

(c)eHSM runs the new FW.

(d)eHSM’s new FW always compare the image version with version counter in OTP, and burn the new version to OTP version counter bit filed if image version is newer than OTP versioncounter.

{% asset_img image-20240614172128526.png %}

{% asset_img image-20240614172219853.png %}





> Note:
>
> •Upgrade algorithm field in OTP and image will be compared in ”Is Upgrade_Alg asymmetric?”step



{% asset_img image-20240614172334022.png %}

{% asset_img image-20240614172349477.png %}



### eHSM Secure Upgrade

#### Secure Upgrade Related Keys

{% asset_img image-20240614135113981.png %}



#### Secure Upgrade Flow

In order to avoid stopping eHSM secure service during upgrade, the upgrade image will be split into several segments, e.g., 4KB for each segment, and eHSM will verify, decrypt and encrypt eachsegment individually with Crypto IPs in interleave mode. The secure upgrade contains following three steps

##### step1:Burn Upgrade FW image to NVM

{% asset_img image-20240614173335395.png %}

{% asset_img image-20240614173459441.png %}



##### step2:Verify Upgrade Program in NVM

{% asset_img image-20240614174827334.png %}

{% asset_img image-20240614174907336.png %}



##### step3:Update OTP version counter

Same as secure install

{% asset_img image-20240614172349477.png %}









# secure boot仿真build

更新OTP和FW

























































