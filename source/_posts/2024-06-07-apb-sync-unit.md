---
title: apb_sync_unit
date: 2024-06-07 09:27:24
categories:
- IC电路设计tips
tags:
- apb协议同步
---

> 项目中可能需要对APB协议进行CDC的设计，简单的方法是通过使用ready信号反压来实现；



# RTL代码实现

```verilog
module apb_sync_unit(
  input  psel       ,
  input  pwrite     ,
  input  pclk       ,
  input  fclk       ,
  input  prst_b     ,
  input  frst_b     ,
  output psel_sync  ,
  output pready     
  );

//apb valid from pclk
reg psel_d;
always@(posedge pclk or negedge prst_b)
  if(!prst_b)
    psel_d <= 1'b0;
  else
    psel_d <= psel;

wire psel_pos = (!psel_d) & psel;

reg ptrans;
always @(posedge pclk or negedge prst_b)
  if(!prst_b)
    ptrans <= 1'b0;
  else if (psel_pos)
    ptrans <= ~ptrans;

//ptrans sync to fclk domain
reg [2:0] ptrans_sync;
always @(posedge fclk or negedge frst_b)
  if(!frst_b)
    ptrans_sync <= 3'b0;
  else
    ptrans_sync <={ptrans_sync[1:0],ptrans};
assign psel_sync = ptrans_sync[2] ^ ptrans_sync[1];//both trigger

//ptrans sync back to pclk
reg [2:0] ptrans_sync_pclk;
always @(posedge pclk or negedge prst_b)
  if(!prst_b)
    ptrans_sync_pclk <= 3'b0;
  else
    ptrans_sync_pclk <={ptrans_sync_pclk[1:0],ptrans_sync[2]};
wire pready_comb  = ptrans_sync_pclk[2] ^ ptrans_sync_pclk[1];//bothedge trigger
assign pready = psel ? pready_comb : 1'b1;

endmodule
```

# 电路解析

{% asset_img image-20240607093602194.png %}



# 使用限制

> apb_sync_unit只使用了psel信号，  通过检测其上升沿从而进行同步，使用时需要保证连续的apb transaction之间，psel信号不会一直为高；
>
> （之前的项目中，外设APB接口协议实现的可能不标准，没有penable，因此sync电路只用了psel进行实现）；



==》如果连续多比apb transaction过程中psel一直为高，会导致pready信号一直拉低，如下图所示是一个先写后读的时序：

错误分析：第一笔写操作，能够回pready；

​             	  第二笔读操作，sync电路检测由于psel=1'b1是连续的，检测不到pos，因此当前读操作得不到处理，会导致pready一直拉低

{% asset_img image-20240607093744708.png %}



<font color=blue>为了处理 连续的apb transaction，psel一直为高  可以参考如下几种做法</font>

## 改进方式1

> APB接口有penable信号，不改动apb_sync_unit

{% asset_img image-20240607094323521.png %}

{% asset_img image-20240607094352996.png %}





## 改进方式2

> APB接口有penable信号，  改动apb_sync_unit

{% asset_img image-20240607094437852.png %}

{% asset_img image-20240607094442804.png %}



## 改进方式3

> APB接口没有penable信号， 改动apb_sync_unit

{% asset_img image-20240607094502525.png %}

{% asset_img image-20240607094506574.png %}



# 时序约束

> APB同步：根据同步之后的psel信号，去抓address phase的控制信号。
>
> 如果时钟频率很快，T=2ns，假设psel经过2拍同步，那么会经过4ns的时间，从clock domain1 到clock domain 2,；
>
> 假设psel sync之后，其他的控制信号例如pwrite，padd等，还没有到达clock domain2（绕线之后，可能有多达到10ns的delay）。那么就会出现错误状态！！！
>
> 可能出现在高制程，频率很快的设计中；例如BUS内部的信号可能走线会很远，导致这种情况的发生！

{% asset_img image-20240607095127922.png %}

