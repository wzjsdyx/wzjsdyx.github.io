---
title: tsmc eflash controller-应用文档
date: 2024-07-02 09:54:50
categories:
- eflash
- tsmc auot eflash controller
tags:
- eflash controller
- TBD
---



#  Function Description

## features

{% asset_img image-20240702101133044.png %}

## memory organization

{% asset_img image-20240702140643117.png %}

{% asset_img image-20240702140658479.png %}

通过上述地址空间，可以访问到flash；

NOTE：

- 在flash boot mode的时候，也会将pflash地址映射到0x0000_0000地址，此时可以通过两块地址空间访问Flash；

- 在sram boot mode的时候，会将SRAM地址映射到0x0000_0000地址，此时可以通过两块地址访问SRAM；

## pflash info page0/1: option page

pflash info page0和info page 1被称为option byte；用来配置pflash和dflash main区域各个page的read以及write protect属性；

> 思考：
>
> <font color=red>memory map中提到的pflash info 2的作用是什么？</font>

## pflash info page2: Hardware Segment

- SGWREN enable
  - read
  - option page program
  - option page erase
- SGWREN disable
  - read

{% asset_img image-20240802111211903.png%}

> 思考：
>
> 默认芯片制造出来的时候，是空片，也就是说flash中的数据为全F；
>
> 交付给客户的时候，是通过FT测试，将数据program到pflash info中去配置effuse信息；
>
> 所以完整的使用流程是：
>
> 1、生产出来的时候，SEGWREN默认是enable；
>
> 2、然后机台FT测试的时候将effuse信息program进去
>
> 3、将SEGWREN配置成disable，此时effuse信息就只有read属性，无法进行修改；

## dflash info 0: OTP

dflash info page0作为OTP（one time program区域）

存放：

- lifecycle
- UID
- 秘钥
- pflash user code的校验参数
- hsm en

## dflash info 1/2: Key area

> 思考：
>
> 1、存放什么内容？作用是什么？
>
> 存放的是给客户用的密钥区域；
>
> 本质上就是划分一些page 给客户使用，之所以不用main array的原因是，flash常用mass erase操作，会将main array的数据全部擦除，处于这些密钥数据不会经常修改的角度考虑，将其划分到pflash info区域存储







## read and write protection rules



读保护说明：

- 读保护主要是防止debugger read flash；

- 正常情况下，CPU可以read flash，但是JTAG访问SRAM之后，CPU就无法访问flash（防止恶意程序在SRAM中执行，去访问flash）





> <font color=red>弄清楚读写保护条件以及读写保护规则：</font>
>
> - read protect only
> - write protect only
> - read and write protect only



# Program Guide



> 思考：
>
> 几种软件操作对应的flash macro上是哪种operation？
>
> 



## command summary

{% asset_img image-20240704104414873.png%}

{% asset_img image-20240704104447692.png%}



## program sequence

1、在操作Flash之前（`FMGCR(Flash Memory Global Control Register)`以及`FMCCR(Flash Memory Command Control Register) `），需要判断这些寄存器是否被lock（<font color=blue>防止误操作</font>）

```C
void eflash_unlock (void)
{
    if( (apb_read(FMGSR_ADR) & 0x1) == 1){
        #ifdef PRINT_LOG
            user_fputc("\ncurretn eflash controller register is lock(writing protect), so execute unlock operation\n");
        #endif
        WRITEMEM32(FMUCK_ADR,0x00ac7805);
        WRITEMEM32(FMUCK_ADR,0x01234567);
    } else{
        #ifdef PRINT_LOG
            user_fputc("\ncurretn eflash controller register is unlock\n");
        #endif
    }
}
```

2、解锁之后，最好也check下是否有未完成的command

```C
void wait_eflash_operation_done(void)
{
    int tmp = apb_read(FMGSR_ADR) ;
    while( tmp & 0x2 == 2){
        #ifdef PRINT_LOG
            user_fputc("\ncurretn eflash controller is busy, so execute wait operation done \n");
        #endif
        tmp = apb_read(FMGSR_ADR) ;
    } 
    #ifdef PRINT_LOG
        user_fputc("\ncurretn eflash controller operation is done\n");
    #endif
}
```

3、配置操作地址FMADDR、命令类型CMD_TYPE并触发开始CMD_START

```C
apb_write(FMADDR_ADR,ADDR);
apb_write(FMCCR_ADR ,{CMD_TYPE,CMD_START});
```

4、check command状态：是否finish以及是否有error

```C
    rdata   = apb_read(FMCSR_ADR);
    fmccf 	= (rdata>>21)&0x1;
    while(fmccf == 0x0) {
        rdata = apb_read(FMCSR_ADR);
        fmccf = (rdata>>21)&0x1;
    }  
    pflash_wp_err = (rdata>>4)&0x1;
    dflash_wp_err = (rdata>>5)&0x1;
    erase_err = (rdata>>10)&0x1;
    acc_err = (rdata>>11)&0x1;
    cmd_conflict_err = (rdata>>20)&0x1;

   if(pflash_wp_err ||dflash_wp_err || erase_err || acc_err || cmd_conflict_err) {
    err=1 ;
   }
```

{% asset_img image-20240704110902369.png%}





