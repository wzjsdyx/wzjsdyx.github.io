---
title: system_verilog基础语法
date: 2026-03-21 17:31:19
categories:
- verilog语法
tags:
- TBD
---

# 基础概念

## package及其使用方式

> SystemVerilog package 是一个“共享定义库”，用于集中管理参数、类型和函数，供多个模块统一使用，避免重复和不一致。

定义

```verilog
package olympus_dcu_pkg;

localparam OLYMPUS_DCU_REQ_NONE      = 4'b0000;
localparam OLYMPUS_DCU_REQ_ALLOC     = 4'b0001;

typedef enum logic [3:0] {
     CACHE_SIZE_4KB  = 4'b0000,
     CACHE_SIZE_8KB  = 4'b0001,
     CACHE_SIZE_16KB = 4'b0011,
     CACHE_SIZE_32KB = 4'b0111,
     CACHE_SIZE_64KB = 4'b1111
     } cache_size_t;
    
  function automatic [3:0] next_way( input logic [3:0] ways,
                                     input logic [1:0] start );
    logic       seen;
    logic [1:0] tmp_way;
    begin
      seen = 1'b0;
      tmp_way = 2'b00;
      next_way = 4'b0000;

      for( int i = 0; i < 4; i = i + 1 ) begin
        tmp_way = start + i[1:0];

        if( ~seen & ways[tmp_way] ) begin
          seen = 1'b1;
          next_way = 4'b0001 << tmp_way;
        end
      end
    end
  endfunction

endpackage
```



使用

```verilog
import olympus_dcu_pkg::*;

if (req == OLYMPUS_DCU_REQ_ALLOC) begin
    ...
end
```



package vs include

| 方式        | 特点                            |
| ----------- | ------------------------------- |
| `package`   | ✅ 结构化、可复用、支持类型/函数 |
| ` `include` | ❌ 纯文本替换，容易重复定义      |
