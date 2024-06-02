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

### NVR,Redundancy Sector Selection

{% asset_img image-20240529090244081.png %}

### Memory Array Configuraton

{% asset_img image-20240529090349066.png %}

### Connection between multiple Flash IP and Pump IP

{% asset_img image-20240602144959234.png %}



### ECC

### Chip Erase

{% asset_img image-20240528205914524.png %}

{% asset_img image-20240528210021298.png %}



### Sector/Block Erase

{% asset_img image-20240528210008315.png %}



### Program

{% asset_img image-20240529104657371.png %}

{% asset_img image-20240529104819187.png %}

{% asset_img image-20240529104843799.png %}





### read

{% asset_img image-20240529104433588.png %}



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

