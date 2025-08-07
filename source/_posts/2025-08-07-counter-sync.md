---
title: counter sync
date: 2025-08-07 14:19:52
categories:
- SoC设计
tags:
---



从慢domain==》快domain进行多bit counter同步；

>选取counter的一个bit作为pulse信号进行同步，然后latch counter数据；





```verilog
module sync_cnt_v2
    #(
        parameter WIDTH = 20
    )
    (
        input              frst_n       ,
        input              brst_n       ,
        input              fclk         ,
        input              bclk         ,
        input              enable       ,
        input  [      1:0] bit_sel      ,
        input  [WIDTH-1:0] multb_in     ,
        output [WIDTH-1:0] multb_out    ,
        output             chg_bpulse   
    );
    reg  bit_ind;
    reg [WIDTH-1:0] cnt_latch;
    reg [WIDTH-1:0] cnt_b;
    reg       chg_fd;
    reg [2:0] chg_bd;

    always @(*)
    begin
        case(bit_sel)
            2'd0 : bit_ind = multb_in[0];
            2'd1 : bit_ind = multb_in[1];
            2'd2 : bit_ind = multb_in[2];
            2'd3 : bit_ind = multb_in[3];
        endcase
    end
    //==============================================================
    // FCLK
    //==============================================================
    always@(posedge fclk or negedge frst_n)
    begin
        if(~frst_n)
        begin
            chg_fd <= 1'b0;
        end
        else
        begin
            chg_fd <= bit_ind;
        end
    end
    assign chg_fpulse = (chg_fd ^ bit_ind);
    always@(posedge fclk or negedge frst_n)
    begin
        if(!frst_n)
        begin
            cnt_latch <= {WIDTH{1'b0}};
        end
        else if(chg_fpulse)
        begin
            cnt_latch <= multb_in;
        end
    end

    //==============================================================
    // BCLK
    //==============================================================
    always@(posedge bclk or negedge brst_n)
    begin
        if(~brst_n)
        begin
            chg_bd <= 3'b0;
        end
        else
        begin
            chg_bd <= {chg_bd[1:0],chg_fd};
        end
    end

    assign chg_bpulse = (chg_bd[1] ^ chg_bd[2]);

    always@(posedge bclk or negedge brst_n)
    begin
        if(!brst_n)
        begin
            cnt_b <= {WIDTH{1'b0}};
        end
        else if(!enable)
            cnt_b <= multb_in;
        else if(chg_bpulse)
        begin
            cnt_b <= cnt_latch;
        end
    end

    assign multb_out = cnt_b;
endmodule


```

