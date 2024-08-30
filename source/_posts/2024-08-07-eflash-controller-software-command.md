---
title: eflash controller software command
date: 2024-08-07 09:47:09
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---

# page erase

1、page erase操作会调用SMWR IP，SME操作会遵循SME操作流程图；

> 1. **开始擦除 (Start Erase)**:
>
>    - 流程开始，初始化擦除操作。
>
> 2. **初始化计数器**:
>
>    - 擦除脉冲计数器 NNN 被设为 0。
>    - 后擦除计数器 MMM 被设为 0。
>
> 3. **设置擦除电压**:
>
>    - 擦除电压设置为高电平 (EV=H)。
>    - 设置写入电压寄存器 (WHV3:03:03:0)。
>
> 4. **执行擦除 (Do Erase)**:
>   - 进行实际的擦除操作。
>    
>5. **执行后擦除读取 (Do Post-Erase Read)**:
> 
>   - <font color=blue>初步读取和检查步骤，用于获得擦除后的数据状态，并为后续的擦除验证提供基础数据。</font>
> 
>6. **增加擦除脉冲计数器**:
> 
>   - 擦除脉冲计数器 N 增加 1。
> 
>7. **擦除验证 (Erase Verify)**:
> 
>   - 验证擦除是否成功。
> 
>8. **验证成功**:
> 
>   - <font color=blue>即使擦除验证通过，为了确保擦除的彻底性，可能还需要进行额外擦除操作。这可能是为了应对一些边缘情况，确保数据完全擦除。==>大概率和Macro的自身设计有关，Erase Verify并不能cover到所有的情况，因此需要足够的额外擦除操作（M >= POST_ERS ），以保证数据完全擦除。</font>
> 
>   - 如果 M 大于等于 POST_ERS，流程结束。
>    - 如果 M 小于 POST_ERS，执行额外擦除操作，后擦除计数器 MMM 增加 1，然后返回步骤 4 继续擦除。
> 
>9. **失败处理**:
> 
>   - 如果擦除失败，检查擦除脉冲计数器 N 是否大于 N_me。
>      - 如果 N 大于 N_me，擦除操作失败，流程结束。
>      - 如果 N 小于 N_me，返回步骤 4 继续擦除。
> 
>术语解释
> 
>- **N_me**: 最大擦除脉冲次数，通常是基于特定芯片的耐久性限制。
> - **T_erase**: 总擦除时间。
> - **T_sme**: 智能擦除操作的时间。

2、SWMR的工作状态可以通过端口信号判断；

> 1、User assert `SMW_OP`
>
> - SME2：3’b010, one-shot mass-erase on MAIN array or MAIN, IFREN, and REDEN arrays, Tme is fixed based on macro specs
> - SME3：3’b011, smart-page-erase on selected array
>
> 2、TSMWR asserts 
>
> - `SMWR_BUSY` signal high until smart-erase cycle is completed.   
> - `SMWR_ERR` indicates PASS(0) or FAIL(1) of the cycle, 
> - `SMWR_LASTSET`, and `SMWR_LOOP` indicate detailed internal setting to complete the “smart-write” cycle.



> Q&A：
>
> Q1、page erase operation，到底是什么信号触发了是让SMWR动起来？SMWR动起来之后，会拉高swm_busy信号，使得smw_mux输出的是smw_top的信号而不是user信号；
>
> A1：通过SME操作时序图，可以判断是因为前一级触发了smw_op，然后让SMWR动起来；
>
> Q2、SME操作流程图怎么理解？
>
> A2：如上解释



# mass erase命令

pflash main区域和dflash main区域是分开操作的，需要不同的指令



操作的地址只要是在对应的main区域中，即可？？？





// try遇到的问题

eflash boot，debugger不能erase flash main区域？

1、先确认读写保护寄存器；

2、根据读写保护规则确定行为；



## Macro mass erase的操作范围：





# reference cell erase

空片必须要执行一次reference cell erase；

## 软件流程

首先写FMUKR 0x00ac7805 ->  0x01234567 unlock；

然后针对FMSPCLCK写32'h3322 1100 -> 32'hffee ddcc；

最后写FMCCR寄存器，执行reference erase 命令；

## 硬件流程

仿真流程

1、user logic控制pflash Macro执行reference cell erase

2、user logic控制dflash Macro执行reference cell erase

3、user logic控制pflash Macro执行mass erase==》pflash main array

4、smart-write IP采用SM3对pflash Macro区域执行page erase==》pflash info page0和page1

5、user logic控制dflash Macro执行mass erase==》dflash main array



实际芯片的流程





时序图





遇到的问题

{% asset_img image-20240809141508322.png%}

解决思路



- reference cell erase执行完之后，怎么判断有没有执行成功

没办法判断，只能按照时序图来，最好多添加一些余量；

- 对pflash info区域进行erase操作（利用smart ip）：

1、smart-ip flow怎么和波形对应？==》这个其实可以不用管，只要给command到smwr IP，其能够正确执行page erase操作即可；

2、怎么看是对应哪一个page进行操作？==》通过地址解码table

- reference cell erase命令，执行的范围怎么定？



# page program



## software page program的使用方式

{% asset_img image-20240809103335826.png%}

> NOTE：
>
> 1、硬件首先会verify判断当前地址是否有valid data
>
> 2、如果有valid data的话，需要先erase；

## 硬件执行机制



### SMP flow chart

{% asset_img image-20240814134414239.png%}



> Note: Once “prog_mask” is set to’1’ for existing ‘0’ bit, it will keep at ‘1’ till end of SMP by
> default.

### Smart-Program Operation

{% asset_img image-20240814135309547.png%}



> NOTE：
>
> SMW_OP
>
> SWM_BUSY
>
> SMW_ERR
>
> SMW_LATESET
>
> SMW_LOOP
>
> 思考：
>
> <font color=red>SMWR burst-write??</font>





## 时序图





# option page program

## software option page program的使用方式

![image-20240814152102767](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-07-eflash-controller-software-command\image-20240814152102767.png)

![image-20240814152118901](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-07-eflash-controller-software-command\image-20240814152118901.png)

> NOTE：
>
> 1、当SEGWREN是enable的时候，pflash info page2 和option page的属性相同；
>
> 2、当做read protect==》non read protect，此SW命令会对pflash main 和 dflash main做mass erase；
>
> 3、program之后，SMWR IP会做program verify操作；

## 硬件执行机制





# page verify

