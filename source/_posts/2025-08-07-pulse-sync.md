---
title: pulse sync
date: 2025-08-07 13:19:33
categories:
- SoC设计
tags:
---

# sync_pu(不带busy)

## 原理图

{% asset_img image-20250807135741977.png %}

## 时序图

> 如果同步的level过长，会出现多个pulse输出

{% asset_img image-20250807135920449.png %}

## RTL Code

```verilog
module sync_pu(
    prst_b,
    tx_ck,
    in,
    rx_ck,
    out
  );

  input  prst_b;// async. power-on reset
  input  tx_ck ;// clock of the source domain
  input  in    ;// asynchronous input signal
  input  rx_ck ;// clock of the destination domain
  output out   ;// synchronized output signal

  reg    sync_toggle;
  reg    sync_q1    ;
  reg    sync_q2    ;
  reg    sync_q3    ;
  reg    out   ;

  always @(posedge tx_ck or negedge prst_b)
    if(prst_b==1'b0)
      sync_toggle <= 1'b0        ;
    else
      sync_toggle <= sync_toggle ^ in ;

  always @(posedge rx_ck or negedge prst_b)
    if(prst_b==1'b0)
    begin
      sync_q1 <= 1'b0 ;
      sync_q2 <= 1'b0 ;
      sync_q3 <= 1'b0 ;
    end
    else
    begin
      sync_q1 <= sync_toggle ;
      sync_q2 <= sync_q1     ;
      sync_q3 <= sync_q2     ;
    end

  always @(sync_q2 or sync_q3)
    out = sync_q2 ^ sync_q3 ;

endmodule // end of module SYNC_PU

```

# sync_pu(带busy)

> 为了解决输入的pulse过长，出现多个pulse输出的问题；

## 原理图

{% asset_img image-20250807140608010.png %}

> 在busy期间，可以屏蔽pulse输入；

## 时序图

{% asset_img image-20250807140924666.png %}

## RTL Code

```verilog
module sync_pu_busy(
    prst_b,
    tx_ck,
    in,
    rx_ck,
    out
  );

  input  prst_b;// async. power-on reset
  input  tx_ck ;// clock of the source domain
  input  in    ;// asynchronous input signal
  input  rx_ck ;// clock of the destination domain
  output out   ;// synchronized output signal

  reg    sync_toggle;
  reg    in_dly1;
  reg    sync_q1    ;
  reg    sync_q2    ;
  reg    sync_q3    ;
  reg    sync_q4    ;
  reg    sync_q5    ;
  reg    sync_q6    ;
  reg    sync_busy_period;
  reg    out   ;

  wire sync_toggle_in = (in & ~in_dly1 & ~sync_busy_period) ^ sync_toggle;

  always @(posedge tx_ck or negedge prst_b)
    if(prst_b==1'b0)
    begin
      sync_q4 <= 1'b0;
      sync_q5 <= 1'b0;
      sync_q6 <= 1'b0;
    end
    else begin
      sync_q4 <= sync_q3;
      sync_q5 <= sync_q4;
      sync_q6 <= sync_q5;
    end

always @(posedge tx_ck or negedge prst_b)
    if(prst_b==1'b0)
    sync_busy_period <= 1'b0;
    else if (sync_q6 ^ sync_q5)
    sync_busy_period <= 1'b0;
    else if (sync_toggle ^ sync_toggle_in)
    sync_busy_period <= 1'b1;


  always @(posedge tx_ck or negedge prst_b)
    if(prst_b==1'b0)
    begin
      in_dly1 <= 1'b0        ;
      sync_toggle <= 1'b0        ;
    end
    else begin
      in_dly1 <= in;
      sync_toggle <= sync_toggle_in ;
    end

  always @(posedge rx_ck or negedge prst_b)
    if(prst_b==1'b0)
    begin
      sync_q1 <= 1'b0 ;
      sync_q2 <= 1'b0 ;
      sync_q3 <= 1'b0 ;
    end
    else
    begin
      sync_q1 <= sync_toggle ;
      sync_q2 <= sync_q1     ;
      sync_q3 <= sync_q2     ;
    end

  always @(sync_q2 or sync_q3)
    out = sync_q2 ^ sync_q3 ;

endmodule // end of module SYNC_PU_BUSY
```

