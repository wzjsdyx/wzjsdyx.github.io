---
title: sram compiler中的column multiplexer配置
date: 2024-03-05 20:02:02
categories:
- SRAM
tags:
- sram compiler
- column multiplexer
---



通常接触到的sram size的表述都是words x bits，例如1024 x 8bits。但是在物理上，这个形状是非常狭长，不利于物理版图的实现，所以foundry实际做的时候会做`折叠`，这个折叠的参数就是column multiplexer（`直观上的理解就是调整SRAM Macro的长宽比例`），来调整SRAM Macro layout版图的形状。接下来就是重点说明：



## column multiplexer参数如何调整长宽比例的

> 假设需要的SRAM size是1024 x 8 bits，memory compiler中配置的column mux的参数是4 ,对应的型号就是(256x4x8 bits)-->256行32列
>
> ADDR总位宽是10 bits，在这种column mux配置下，X地址是8bits，Y地址是2bits
>
> 现在对地址10‘b00_0000_1100进行访问，经过X地址解码，会访问第3行，经过Y地址解码，会访问MUX=0的列
>
> 可以看到，如果将column mux设置为8，那么SRAM Macro会变宽，（128x8x8-->128行64列）

{% asset_img  column_multiplexer工作机制.jpg %}

<center><u>column_multiplexer工作机制</u> </center>

> 记住如下结论：
> 1)字长*字宽=行数*列数
> 2)行数=字长/MUX
> 3)列数=字宽*MUX
> 4)MUX越大，行数越少，列数越多，micro尺寸越矮、越宽



## column multiplexer还会影响时序和面积

column multiplexer这个参数主要作用是调整SRAM Macro layout的形状，此外还会一定程度上影响时序和面积。

很多情况下，column multiplexer这个参数越大，SRAM Macro layout的图形越宽，其面积越大，时序越好。

有些时候，这个规律并不使用，还需要以memory compiler实际产生的数据为准。

---

参考博客：

1. [ARM的memory Compiler总结](https://blog.csdn.net/lyfwill/article/details/81330081)

2. [ Wordline与Bitline](https://bbs.eetop.cn/thread-955010-1-1.html)

