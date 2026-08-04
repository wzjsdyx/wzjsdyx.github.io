---
title: verilog testbench搭建
date: 2026-05-15 10:23:50
categories:
- verilog语法
tags:
- TBD
---



# Display Color文本

```verilog
    $display("\033[1;31m This is red text       \033[0m\n");    // 红色
    $display("\033[1;32m This is green text     \033[0m\n");    // 绿色
    $display("\033[1;33m This is yellow text    \033[0m\n");    // 黄色
    $display("\033[1;34m This is blue text      \033[0m\n");    // 蓝色
    $display("\033[1;35m This is magenta text   \033[0m\n");    // 品红
    $display("\033[1;36m This is cyan text      \033[0m\n");    // 青色
```



# Display TimeStamp

```verilog
`timescale 1ns/1ps
initial begin
    // 设置时间格式为 纳秒，3位小数，单位后缀为 ns，字段宽度 20
    $timeformat(-9, 3, " ns", 20);
    $display("Start simulation");
    // 无限打印
    forever begin
        #1ms;  // 延迟 1ms
        $display("Time: %t - Looping log...", $time);
    end
end
```

> 说明：
>
> 分别打印在默认格式下和设置时间格式为单位1ns，精度小数点后3位，打印字符串 ns，最小打印长度10的情况下的仿真时间
>
> $timeformat(-9,3,” ns”,20);









# 时钟和复位

