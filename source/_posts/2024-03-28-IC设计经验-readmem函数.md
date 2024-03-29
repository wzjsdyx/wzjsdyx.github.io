---
title: IC设计经验-readmem函数
date: 2024-03-28 15:42:52
categories:
- IC设计经验
tags:
- readmemh
- readmemb
- readmem
---



> 在做RTL仿真的时候，需要对RAM以及ROM 设置初始值
>
> 

# 基本使用方法

txt文件中是十六进制数字就要用readmemh

txt文件中是二进制数字就要用readmemb





# 文件数据个数和mem深度不同



# 文件位宽和mem位宽不同

## 文件数据位宽>mem位宽

如果txt中的数字位宽大于mem的位宽，使用readmem函数时会自动从txt的低位开始填充mem，高位舍弃

## 文件数据位宽<mem位宽

---

参考博客：

1. [Verilog的系统任务----$readmemh和$readmemb](https://blog.csdn.net/wuzhikaidetb/article/details/125728219)
