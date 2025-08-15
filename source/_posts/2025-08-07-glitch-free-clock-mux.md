---
title: glitch free clock mux
date: 2025-08-07 14:27:26
categories:
- SoC设计
tags:
---







> 核心思想是将有glitch的那段时间gating掉

# 接口

{% asset_img image-20250807142929081.png %}



## 原理图

{% asset_img image-20250807142939116.png %}



## RTL Code

```verilog
module clk_sync_4_1 (clk0, clk1, clk2, clk3, clk_ref, test_scan_mode, scan_clk, test_mode, chg, sel, clkout, chg_ok, rst_b);
   input clk0, clk1, clk2, clk3;
   input chg;
   input [1:0] sel;
   input clk_ref;
   input test_scan_mode, scan_clk, test_mode;
   input rst_b;
   
   output clkout;
   output chg_ok;

reg chg_d, chg_2d, chg_3d;
reg trig, trig_d, trig_2d;
reg chg_ok;
reg [1:0] mux_sel;
wire mux_clk;


wire mux_clk_scan;
dtc_ckmux2 DTC_MUX_CLK_SCAN (.I0(mux_clk), .I1(scan_clk), .S(test_scan_mode), .Z(mux_clk_scan));

always @(posedge mux_clk_scan or negedge rst_b)
   if(~rst_b)
    begin
     chg_d <= 1'b0;
     chg_2d <= 1'b0;
     chg_3d <= 1'b0;
   end
  else
   begin
     chg_d <= chg;
     chg_2d <= chg_d;
     chg_3d <= chg_2d;
   end


dtc_ckmux2 DTC_REF_CLK_SCAN (.I0(clk_ref), .I1(scan_clk), .S(test_scan_mode), .Z(clk_ref_scan));

always @(posedge clk_ref_scan or negedge rst_b)
  if(~rst_b)
   begin
     trig <= 1'b0;
     trig_d <= 1'b0;
     trig_2d <= 1'b0;
     chg_ok <= 1'b0;
  end
 else     
   begin
     trig <= chg_3d;
     trig_d <= trig;
     trig_2d <= trig_d;
     chg_ok <= trig_2d;
   end

wire and_sel;
assign and_sel = trig_d & ~trig_2d;

wire [1:0] mux_of_sel;
assign mux_of_sel  = and_sel ? sel : mux_sel;

//always @(posedge clk_ref)
always @(posedge clk_ref_scan or negedge rst_b)
  if(~rst_b)
   mux_sel <= 2'b0;
  else
    mux_sel <= mux_of_sel;

wire mux_clk01, mux_clk23, mux_clk_1;
dtc_ckmux2 DTC_MUX_CLK01 (.I0(clk0), .I1(clk1), .S(mux_sel[0]|test_mode), .Z(mux_clk01));
dtc_ckmux2 DTC_MUX_CLK23 (.I0(clk2), .I1(clk3), .S(mux_sel[0]|test_mode), .Z(mux_clk23));
dtc_ckmux2 DTC_MUX_CLK_OUT (.I0(mux_clk01), .I1(mux_clk23), .S(mux_sel[1]|test_mode), .Z(mux_clk));


dtc_icg clk_sync_icg(.TE(1'b0), .E(~chg_2d|test_mode), .CP(mux_clk), .Q(clkout));

endmodule

```

