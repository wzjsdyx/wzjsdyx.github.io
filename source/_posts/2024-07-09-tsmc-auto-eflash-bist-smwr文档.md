---
title: tsmc auto eflash bist/smwr文档
date: 2024-07-09 20:41:10
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---

# doc1: Release note

## LIBRARY INFORMATION

{% asset_img image-20240709205651138.png%}

## DESIGN KIT VERSION DEPENDENCY TABLE

{% asset_img image-20240709205744503.png%}

# doc2: Application Note

## General Description

描述怎么集成BIST以及SMWT IP；

## Database Description

提供的database如下：

1. RTL
2. BENCH
3. MODEL
4. SIM
5. SYN
6. FORMAL
7. TESTER

### Test-Chip IO

1、disable bist功能的时候，eFlash macro是由user's logic控制；当enable bist功能的时候，eFlash macro是由bist控制

2、VPP和TM是机台CP测试时 使用的模拟PAD

### Test-Chip Block

{% asset_img image-20240709213236174.png%}

### Pattern Descriptionsunder BENCH

1. bist_cp_test
2. bist_slftest
3. smwr_test
4. smwr_test_abort

{% asset_img image-20240712112642868.png %}

{% asset_img image-20240712112658302.png %}



### TASK Descriptionsunder BENCH

1. set_ir_dir
2. set_dr_only
3. check_status
4. reset_enable_bist

{% asset_img image-20240712112825588.png %}



## Block Integration

{% asset_img image-20240710095327561.png%}

# doc3: BIST Design Spec

## General Description

提供BIST IP，用于CP测试

### Features

- 两个时钟域：

  1. JTAG时钟域 TCK：来自于机台tester

  2. BIST funciton时钟域CLK：可以来自于机台，也可以来自片上时钟源；

     <font color=blue>BIST CLK可以JTAG命令进行无毛刺的在TCK和片上时钟之间切换；要求是CLK≥TCK</font>

- BIST电路有配置寄存器

- BIST可以提供一些debug功能：

  1、用户可以收集failure bits 信息

  2、检测到错误的时候，BIST电路可以暂时停止

  

> 思考
>
> <font color=red>电路还有一些debug功能？</font>

### Block Diagramto integrate BIST_MUX

{% asset_img image-20240710103403532.png%}

> 说明：
>
> 1、CP测试阶段，大多数的bist 操作不会用到SMWR模块进行smart-write算法；少数情况下也是会发出smart-write请求；
>
> 2、用户逻辑可能会发出smart-write请求；（smart-erase和smart-program）

### Block Diagramof BIST_MUX

{% asset_img image-20240710104021779.png %}

## Pin Descriptions

### IO Description for BIST_MUX

详见文档

## BIST Operations

bist_enable信号可以来自于PAD，也可以来自于内部logic；

CLK可以来自于TCK或者片上时钟源；

CLK大于等于TCK，为了完成JTAG和BIST的handshake；

JTAG TCK的频率最好＜20Mhz，为了有一个好的IO质量；

### An example of BIST operation

{% asset_img image-20240710105722108.png %}

### JTAG Port

详见文档

<font color=blue>BIST使用JTAG接口完成和tester的通信；</font>

通过TAP（16个状态）去控制IR（指令寄存器）和DR（数据寄存器）的访问；

{% asset_img image-20240712133404519.png %}

 During Shift-IR and Shift-DR, the 1st TDI enters the LSB, and TDO shows the LSB of the shift register. 



IR的传输长度固定为8；IR[7:0]存储了JTAG的指令信息：

- IR[1:0]存储指令代码，`当指令代码是2'b10，表示PRELOAD/SAMPLE数据寄存器指令`，此时IR[7:2]表示BIST data寄存器的地址；
- IR[1:0]存储指令代码，当指令代码不是2'b10，表示`BYPASS指令`，接下来的所有数据会被bypass，直到加载下一次IR；

DR的传输长度固定为40；DR[39:0]用于存储BIST data寄存器的内容；

- 当处于`CAPTURE-DR状态`，BIST data寄存器的数值会被load到DR[39:0]

- 在`SHIFT-DR状态`被shift out

- 在`UPDATA-DR状态`DR[39:0]的数值会被update到BIST data register中（该寄存器需要writeable）



> 思考：
>
> 1、JTAG指令寄存器和数据寄存器到底有啥作用？
>
> IR寄存器用于判断当前是否是有效的指令，以及选择哪一个BIST data reg；
>
> DR寄存器用于将BIST data reg shift out以及update；
>
> <font color=blue>具体可以看TAP接口就可以理解！！！</font>

#### IR & DR控制

详见文档

{% asset_img image-20240712144000804.png %}

{% asset_img image-20240712144028148.png %}

{% asset_img image-20240712144132209.png %}

{% asset_img image-20240712144150652.png %}



#### Interface between TAP controller and BIST

{% asset_img image-20240712141533341.png %}

{% asset_img image-20240712141554325.png %}



### BIST Command Table

详见文档

开始BIST command：

1、load command index==》R_TEST_CTRL[17:8]

2、set 1'b1 to R_TEST_CTRL[7]

3、R_TEST_CTRL[0]表示BIST操作是否完成





### BIST Debug Flow

详见文档

> 思考：
>
> <font color=red>什么是bist debug flow？</font>
>
> enable debug feature to collect failure address and data

{% asset_img image-20240712151949395.png %}



### ECC Operation

BIST操作是否需要添加ECC

{% asset_img image-20240710110603932.png %}

> 思考：
>
> Q1:项目中bist电路是否有添加ECC功能？
>
> 项目中并没有在bist测试的时候，使用ECC功能

## BIST Register Map

详见文档

# doc4: SMWR Design Spec

## General Description

可以用来调整 program和erase期间的HV脉冲电压，脉冲计数，脉冲宽度。有了这个算法，可以在最小的HV应力下将单元编程为“0”或擦除为“1”，从而提高Macro阵列耐久性和可靠性。

### Features

1. 单时钟域：内部有寄存器可以调整时钟频率

2. smart-write操作期间的操作码

{% asset_img image-20240710133603809.png%}

3. 针对SMP操作，提供burst-write：通过添加row buff来实现

4. SMP1/SMP2操作会调用PV-read，为了防止对cell的over-stress

5. SMP1和SMP2不能混合使用

6. 使用SMWR会有面积开销：SMWR IP的逻辑占用面积很小，主要是row buffer；

### Block Diagram to integrate TSMWR with eFlash macro on chip

{% asset_img image-20240710132116911.png%}



### SME Flowchart

{% asset_img image-20240710132559555.png%}

> 思考：
>
> SME流程没懂？

### SMP Flowchart

{% asset_img image-20240710132646627.png%}

> 思考：
>
> SMP流程没懂？



## Pin Description

详见文档



## Operation Descriptions
### Smart-Erase Operation

{% asset_img image-20240806172954142.png%}

1、User assert `SMW_OP`

- SME2：3’b010, one-shot mass-erase on MAIN array or MAIN, IFREN, and REDEN arrays, Tme is fixed based on macro specs
- SME3：3’b011, smart-page-erase on selected array

2、TSMWR asserts 

- `SMWR_BUSY` signal high until smart-erase cycle is completed.   
- `SMWR_ERR` indicates PASS(0) or FAIL(1) of the cycle, 
- `SMWR_LASTSET`, and `SMWR_LOOP` indicate detailed internal setting to complete the “smart-write” cycle.

对于10K-enduranced 的array，SM3 run with SMW-HEM disable

对于100K-enduranced的array，SM3 run with SMW-HEM enable

SMWR会更具SMW_HEM的设置，自动控制array的最大擦除次数；

### Smart-Program Operation

### Abort Operation

abort正在运行的smart-write操作或者abort被pending的smart-write操作；































































