---
title: wire和wreal的连接
date: 2025-05-29 19:54:51
categories:
- ip开发环境搭建
- EDA工具使用
tags:
- vcs
- xrun
---



# wire和wreal

> `wire`是`Verilog`的数据类型，表示`net`；
>
> `wreal` 是 `Verilog-AMS（Analog Mixed-Signal Verilog） `的语法扩展，表示一种双向实数信号（real-valued net），它可以携带一个实数值（例如模拟电压、电流），用于模拟与数字模块之间的接口。

```verilog
// 声明一个 wreal 类型的端口
wreal vout;

// 模拟电压行为建模
analog begin
  V(vout) <+ transition(1.2, 0, 1n); // 输出1.2V，过渡时间1ns
end

```

# wire和wreal的连接

将wreal和一个实数进行比较，然后生成wire类型数据；

```verilog
// 方式1
module digital_block (
  input wreal in_signal,
  output logic out_signal
);
  real threshold = 0.5;
  always @(*) begin
    if (in_signal > threshold)
      out_signal = 1;
    else
      out_signal = 0;
  end
endmodule

// 方式2
module wreal_to_logic (
  input  wreal analog_in,
  output logic digital_out
);
  assign digital_out = (analog_in > 1.0) ? 1'b1 : 1'b0;
endmodule
```

