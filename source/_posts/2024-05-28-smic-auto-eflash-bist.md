---
title: smic auto eflash bist
date: 2024-05-28 13:36:38
categories:
- eflash
- eflash bist
tags:
- smic auto eflash bist
---



<font color=blue>相关文档学习</font>

# DOC-->SMIC Serial Testing Bus

## Introduction

The Serial Testing Bus(STB) module 用于测试如下两个eflash Macro：

-  pfm_340sa128kx150_v1Flash Macro 
-  pfm_340sa8kx150_v1Flash Macro.

设计初衷是为了使用`JTAG接口(4PIN)`在`CP阶段`对Flash进行测试。

一个STB模块最多可以携带8个Eflash macro, SMIC能够提供测试向量和STB的netlist.

## Block Diagram

### STB in System

{% asset_img image-20240528134454779.png %}



> Note:
>
> 1.Dut_Num field selects flash macros,Dut_Num[7:0]  = 8’h01 means one Flash Macrois selected, Dut_Num[7:0]  = 8’hff means eight Flash Macros are selected;
>
> 2.N: The NthFlash.



### STB Block Diagram

{% asset_img image-20240528135925055.png %}



> Note:
>
> 1.JTAGController: Access all data registers (DR) and instruction registers (IR).
>
> 2.Register & Counter: Configure timing and create the proper timing for Function Controller.
>
> 3.Function Controller: Exports the proper waveform for flash to execute theerase/program/read/configure.
>
> 4.BIST Controller: Algorithmically  generates address/data, and create pass/fail log. 
>
> 5.Read NVR_CFG & Config:Auto configure Flash.
>
> 6.Erase Retry & Vread:Automatically  completes Erase Retry function  and Vread.
>
>  7.Cycling: Erase, program and read operation are cyclingexecuted.

### Pin Description

{% asset_img image-20240528140950126.png %}



{% asset_img image-20240528141012829.png %}



{% asset_img image-20240528141033520.png %}





## Block Function  Description of JTAG interface

TAG interface consists of four parts: test access port (TAP), TAP controller  (state machine), an instruction register and a group of data registers.

### Test access port (TAP)

### TAP controller  (state machine)

### Signal interface Settings

<font color=red>TBD</font>



## Operation  Description

### Instruction Group

There are sevenkindsof instructions: Setup  Instruction, FunctionInstruction, Erase Retry  Instruction, Cycling  Instruction,  Register Instruction,BISTInstruction,  Test  Mode Instruction.

### Internal Register

#### STB Register

{% asset_img image-20240528142039255.png %}



#### BIST_REG_SOC

{% asset_img image-20240528142048774.png %}





### Setup Instruction

The setup instruction is set the STB output pin high or low. The setup instruction may use together with other instruction for certain operation. 

The setup instruction is consist of 8 bits, 7 bits setup ID and 1 bits instruction parameter.

`All pins controlled by setup instruction  will not change until next setup instruction except “CEb”. If the chip is enable the “CEb” will change accordingly whiletest mode exit.`



#### MBIST Mux Setup

The MBIST mux setup instruction  will control the `BIST_SEL`pin. 



#### Flash Control Pin Setup

The flash control pin setup instruction include :

`PORb setup, standby setup, deep power down setup, recall setup, CFG lock setup, Vread0 setup, Vread1setup,Sread0 setup and Sread1 setup.` These instructions are used to` give the flash input a solid state`. Only  CEb will change while exiting the test mode. The other pins will remain constant until next setup instruction received.



#### STB Counter Setup

The STB counter setup instruction is used to switch on and off the STB counter. The execution for erase and program time is controlled by STB counter.  `By switching off the STB counter,  user can extend the time of erase and program, especially in test mode.` Please be noted that switching off the counter could only extend thetime but not shorten the time.

<font color=red>啥意思？不懂</font>



### Function Instruction

The function instruction is used to execute certain operation, like erase, program, read and etc.

 It can also use together with setup instruction,  test mode instruction  and BIST instruction. The function instructions consist of instruction  ID and instruction  parameter.

{% asset_img image-20240528143303435.png %}





### Erase Retry Instruction

{% asset_img image-20240528143447615.png %}



### Cycling Instruction

{% asset_img image-20240528143512853.png %}



### Register Instruction

{% asset_img image-20240528143604662.png %}



<font color=red>啥意思？不懂</font>

### BIST Instruction

There are four kindsof BISTinstructions:  Chip BIST, Column BIST,SectorBIST and Row BIST. They can reador write all addressof the selected area. The Chip BIST andColumn BIST can only access the NVR/MAIN array. There are three patternsfor Chip BIST(Normal, CKBD and Diagonal).

> - 芯片BIST提供了三种模式来执行自测试：正常模式、CKBD模式（Checkerboard，棋盘格模式）和对角线模式。
> - 这三种模式可能会应用不同的测试算法或测试模式，但它们的目标都是对selected area进行操作，并检查读取的结果是否与预期一致

{% asset_img image-20240528144541432.png %}

{% asset_img image-20240528144548764.png %}



### Test Mode Instruction

Test mode instruction  is for flash going into the relative basic test mode. The test mode instruction can be set one by one and normal operation can be followed. When normal operation finished, the “TM Exit” instruction  should be followed.

> - 这句话表明测试模式指令用于将flash置于一种基本的测试模式中，以执行测试操作。
> - 测试模式指令可以逐个设置，然后可以继续进行正常的操作

<font color=red>啥意思？不懂</font>

# DOC-->BIST TEST BENCH USER GUIDE

## List of files required for the simulation

{% asset_img image-20240602112715254.png %}

<font color=blue>rtl sim和port sim使用的testbench文件不一样，需要注意</font>

## Testbenchnaming rules

{% asset_img image-20240602112747642.png %}



## Waived errors

{% asset_img image-20240602112850662.png %}



## Waived warnings

{% asset_img image-20240602112919497.png %}



## Necessary modifyfor interfacefile

> Reason: As the synthesis netlist use ideal clock network, there will be hold violation in `netlist simulation`. In order to solve this violation, the delay should be added to all flash ADDRports and RDEN ports.If do not modify in interfacefile, verilog model will report “ERRORX-010! Timing Violation@XXns:Tah,target=1.00ns;” and BIST read fail.

<font color=blue>netlist级别的仿真需要加上delay</font>

{% asset_img image-20240602113148104.png %}



# DOC-->eFlash Macro Datasheet

## Features

{% asset_img image-20240528204543207.png %}



## Product Description

### Function Block Diagram

{% asset_img image-20240528211323214.png %}

### Pin Description

{% asset_img image-20240528211417759.png %}

{% asset_img image-20240528211441687.png %}

### Operation Modes Selection

{% asset_img image-20240529090046913.png %}





## ECC

{% asset_img image-20240604213629620.png %}



## power mode

### Standby mode

{% asset_img image-20240604213701983.png %}



### Deep Power Down Mode

>  The device enters the Deep Power Down Mode when both DPD pin and CEb pin are high.
> Power consumption could be further reduced. However, a tDPD recovery time is needed to
> return the device back to `active mode` from deep power down mode.  



<font color=red>TBD:</font>active mode是指进入一些read/program的operation mode吗？

从deep power mode到其他operation mode，需要先进入Standby吗？还是直接可以进入？应该需要看时序图确认！！！



### Auto Low Power Mode  

> The device has the Auto Lower Power mode which puts it in a near standby mode `after data
> has been accessed with a valid Read operation`. This reduces the active Read current to the
> standby current.
> While CEb is low, the device exits Auto Low Power mode with CLOCK rising edge when
> RDEN=1 to initiate another Read cycle, `with no access time penalty`.  

<font color=red>TBD:</font>应该是read 操作之后自动进入low power mode，此时的功耗类似于standby mode。



## eflash operation

### read

> The Read operation of the device is triggered by CLOCK rising edge during RDEN=1.  Output data width is 150 Bits.  
>
> `All non-Read operations do not care the status of CLOCK.  `
>
> CEb must be low for the system to obtain data from the device. When CEb is high, the device
> is de-selected and only standby power is consumed.（这里说的应该就是Auto Low Power Mode）
> `The data output is reset to be all zero by Power on Reset, or CEb high  `

<font color=red>所以我理解在实现的时候，read操作之后不需要进入主动进入standby mode；这样连续读写可以节省进入和退出standby mode的时间；</font>

### Recall Read

Recall Read is a special slow read mode to obtain the configuration bits from the NVR and
NVR_CFG sectors before VREF can be accurately configured.  

<font color=red>TBD:应该只有NVF_CFG也就是NVR_1需要用recall read吧？</font>



#### Read Cycle Timing Parameters (Cload=0.15pF)

{% asset_img image-20240605092527921.png %}



#### Read Cycle Timing Diagram

{% asset_img image-20240529104433588.png %}



### Program

> The device is programmed on 75 bits basis. DINSEL selects one of 75 bits out of 150 bits
> word.  

program的最小单位是75bits；

#### Program/Erase Cycle Timing Parameters

{% asset_img image-20240605094745072.png %}

#### Program Cycle Timing Diagram  

{% asset_img image-20240529104819187.png %}

{% asset_img image-20240529104843799.png %}





### Sector Erase

> The sector erase operation erases the device on a sector-by-sector basis.  



### Block Erase

>  Block erase operation could erase total 16 sectors organized as block for main array.   
>
> During a block erase, users could also specify a combination of multiple redundancy sectors
> to be erased with main array sectors.   

7843项目中flash controller没有实现这个操作；

{% asset_img image-20240528210008315.png %}

### Chip Erase

{% asset_img image-20240528205914524.png %}

{% asset_img image-20240528210021298.png %}



### Charge Pump Sharing

> Its charge pump block (macro name: bias_340sa128kx150_v1) can be shared among up to 2
> Flash macros. Before the signals connecting to the pump, the logic signals from different
> devices should be `multiplexed`, whereas the analog signals from different devices must be
> wire-connected.
> The read while write and the read while read operations can be done in two different
> instances with a shared charge pump simultaneously, but `the concurrent writes cannot be
> allowed`.
> `Bias IP must be in power off mode when any shared flash IP enter power off mode.  `

<font color=blue>design上是对多个Macro信号或操作连接到bias；只使用一种bias IP，能够对不同size的Flash macro进行操作</font>





### Power On Reset (PORb)

> The Power On Reset pin (PORb) is used to reset the device during power-on. When the
> PORb pin is held low, any in-progress operation will be terminated, and all internal registers
> are cleared.  



### NVR Sectors

> NVR array is a group of special sectors outside the main array memory space. NVR array has
> 32 sectors which can be selected by the NVR pin.  



### NVR_CFG Sector

> The device’s configuration information is written into a special NVR_CFG sector during wafer
> testing. User can access its contents with NVR_CFG=1 and RECALL=1.
> To prevent configuration information being inadvertently damaged, a pin LCK_CFG=1 is used
> to protect this NVR_CFG sector.
> NVR_CFG sector can be programmed or erased for testing purpose only.  
>
> In NVR_CFG sector, every even and odd row pair (row 0/1, row 2/3 …) should be
> programmed with same data pattern. In read mode, the data of even/odd pair is read out  simultaneously as one word, so its sector size is only a half of 128 x 150 bits.   





### Pre-Program  

> Before a program operation, same data should be programmed onto the target address with a
> shorter program pulse. A program operation with PREPG high indicates a Pre-Program
> operation.
> Pre-Program multiple words in the same row before the program operations can reduce the
> Pre-program time overhead.  



### Erase Retry  

> Erase Retry uses a step-up voltage scheme with a maximum of 20 retry pulses. The
> RETRY[1:0] pins specify the retry voltage level for each retry pulse. RETRY[1:0] should be
> ’11 for single pulse sector/block erase and chip erase.
> The Verify Read (VREAD1=1) is used to check whether a sector/block has been successfully
> erased after each pulse. An extra retry operation is needed if the sector/block is not erased
> successfully.
> No verify Read is needed for the last pulse as the cumulative erase time is already sufficient.
> Erase retry scheme should be used for sector/block erase only, not chip erase.  

### Verify Read

> Verify read is a special read mode to guarantee a successful write operation. Verify read is
> controlled by the pin VREAD1 and VREAD0. The access time is slower than normal read.  

### Safety Read

> Safety read 0/1 is a special read mode to detect whether any flash bit has experienced
> unusual reduction in read margin. Safety read 0 is controlled by the pin SREAD0, and safety
> read 1 is controlled by the pin SREAD1. Their access time is the same as a normal read.  

### Row Redundancy

> There are 4 redundancy sectors: These 4 redundancy sectors can be used to replace the bad
> sectors of main array. See Table: NVR, Redundancy Sector Selection.
> The redundancy information is stored at several locations of NVR_CFG sector.   

### Set Configuration Register

> For obtain required performance, it is necessary to set configuration bits into the registers of
> the device.
> Before normal read/program/erase operation, eight groups of configuration data must be read
> from the device while RECALL and NVR_CFG are high and written to eight groups of
> configuration registers (16 bits for each group) in the device. These configuration registers are
> selected by A[2:0] when CONFEN is high. Only 16 LSB bits of each word are effective  configuration bits. 
>
> Word 0 maps to address 000, Word1 maps to address 001, etc.   
>
> After power on reset, all configuration registers are reset and need to be rewritten before any
> other operations  
>
> To set RECALL pin high during Set Configuration Registers operation, performing Recall
> Read and Set Configuration Registers sequentially can help reduce time overhead.  



### Configuration Register Read

> After all configuration registers are written, the contents of the configuration registers can be
> read out to device pins by configuration register read.
> Those configuration registers are selected by A[2:0] when CONFEN=1 and RDEN=1 at the
> rising edge of CLOCK. Only 16 LSB bits of each word are read out to DOUT[15:0].  



### Power Off  

> The device can be completely turned off by an internal power switch when SUPPLYON=0.
> This power switch should be turned on (SUPPLYON=1) after power up and constantly
> remained for any operations. Flash IPs and their shared Bias IP must go to power off mode
> together.
> Power on reset should be activated during supply switching off, therefore all configuration
> registers will lose the current value and they must be reloaded again before the next
> read/write operation.
> When the device is switched off, all output pins from pfm are floated.  



## 其他说明



### NVR,Redundancy Sector Selection

{% asset_img image-20240529090244081.png %}

### Memory Array Configuraton

{% asset_img image-20240529090349066.png %}

### Configuration Data and Redundancy Storage in NVR_CFG

{% asset_img image-20240606142930577.png %}

### Configuration Data Mapping Table

{% asset_img image-20240606144800615.png %}

### Connection between multiple Flash IP and Pump IP

{% asset_img image-20240602144959234.png %}





<font color=blue>testbench学习理解</font>

# Testbench-->STB operation simulaiton

## chip erase

> 含义：erase entire main arrary, redundancy sectors and NVR sectors all together or seprately

<font color=red>TBD</font>

观察STB输出的控制信号即可，不用太过关注testbench 指令



## sector erase









# Q&A

## STB in System 章节中，SoC REG的作用？

首先这是一个输出端口，是SoC控制寄存器，用于在flash test mode下，输出信息，让外部调节电压

7843项目中，将此信号连接到Standby 模块；

<font color=red>TBD</font>

## 仿真地址问题

