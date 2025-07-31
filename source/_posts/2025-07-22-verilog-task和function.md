---
title: verilog task和function
date: 2025-07-22 19:36:59
categories:
- verilog语法
---

# function

```verilog
    function [4:0] crc5_serial;
        input [4:0] crc;
        input data;
        begin
            crc5_serial[0] = crc[4] ^ data;
            crc5_serial[1] = crc[0];
            crc5_serial[2] = crc[1] ^ crc[4] ^ data;
            crc5_serial[3] = crc[2];
            crc5_serial[4] = crc[3];
        end
    endfunction
```

关键点：Verilog 函数用于描述组合逻辑，通过**对函数名赋值**返回结果，适合实现数学运算或状态转换（如 CRC、加密算法等）



# task

> 
