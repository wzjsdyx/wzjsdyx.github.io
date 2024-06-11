---
title: 车规MCU bootrom 验证
date: 2024-06-07 10:36:27
categories:
- 车规MCU
tags:
- host bootrom
---

> 车规MCU需要HSM信息安全，其实现方式有可能会利用到host bootrom
>
> 一个不懂的东西，怎么去快速了解上手？
>
> 1、需要自己看看已有的资料有一定的认知；
>
> 2、私下问下同事，询问不懂的问题；
>
> 2、开会和其他同事同步，进一步加深理解；
>
> 3、总结思考实践-->有自己的认知；



# host bootrom功能描述

主要提供normal boot和secure boot的功能

BootRom需要使用GPIO，Uart，HSM message box等模块。



# 测试流程

==》load config只是从flash info区域读取数据，不care OTP区域；

<font color=blue>hsm enable以及secure boot是通过读取寄存器</font>？也不用care OTP区域；

<font color=red>读取寄存器？读取什么寄存器？</font>

==》init debug表明是否需要打印log（optional，默认disable）

要打印log就初始化UART串口；不需要打印log就不初始化UART串口；



==》rom code的执行是否需要切换成系统时钟（optional，默认enable）

切换成PLL提供的系统时钟，处理速度就比较快，加速启动时间

启动PLL也就是初始化PLL大概需要100us的时间

<font color=red>系统时钟：从什么时钟切换到PLL时钟？？？</font>

## non-security boot

==》两个secure boot check？

security boot分成两部分：一个是启动校验，一个是等待校验结果

所以流程图上有两个security boot check框



==》Restore debug and clock(Optional)

关闭UART以及将系统时钟切回去



==》jump to user code

跳转到pflash的user code（default是0x20_0000）

- user code可以配置从pflash的info区域执行；

- user code可以配置从pflash的main区域执行；

执行user code



## security boot

==》 security boot需要很多的配置信息，首先需要确认配置信息是否正确（主要是对地址以及size的有效性进行检查）



==》等待HSM ready



==》执行check code（<font color=blue>校验用户代码的代码</font>），通过发命令到HSM，（主要是校验地址，校验size以及校验算法），让其开始校验<font color=blue>用户代码</font>



==》等待check结果（done，pass/fail）

正确就继续执行下面的流程，即走启动用户代码的流程；

错误就执行while 1 （default的行为），也可配置成system reset，通过watch dog进行复位；

吴奎那边有现成的工具可以做对应的测试；





# Q&A



==》什么时候出现check error

将需要校验的代码，手动修改成错误数据

> 校验算法是通过OTP配置好的
>
> 校验的地址和size都是在info区域配置的



==》standby boot

pflash user code进入standby模式之后，默认还会执行security boot校验（optional可配置）



==》用户代码和被校验的代码

安全启动的时候就是被校验的代码

没有安全启动的时候就是未被校验的代码，即用户代码



==》默认是不swap的

swap之后，如下图这些配置起始地址就会从0x01004800-->0x01005000，offset不变

{% asset_img image-20240611113255222.png %}



==》security boot的参数比较多

hsm security boot的各个参数是绑定到一起的，通常来说错一个都会导致HSM check error



==》SOTA功能测试

有两个配置区域，一个是正确的配置，一个是错误的配置；

通过配置SOTA enable还是disable可以进行不同的测试；



==》HSM安全启动测试

不同的校验算法都校验成功；

根据hsm的算法，去校验什么东西，然后做对应的配置；

询问HSM的人，怎么把算法用起来；

例如：key需要配置什么东西，验签signature需要设置什么东西和算法相关，使用什么算法，就需要根据对应的算法做相应的配置





## pflash info:HSM security boot各项配置参数的含义



## dflash info:OTP各项配置参数的含义



## SOTA机制





## boot flow/数据流

例如，host core先动起来，执行host romcode代码，然后加载pflash info区域的值；

然后判断是否为security boot，如果是的话，发送command，去调动hsm做校验；

<font color=red>hsm根据OTP的value去使用相应的算法做校验？？？具体的流程是什么样？</font>





## 测试item

配置选项/输入条件

check point？





