---
title: ARM Dhrystone
date: 2025-03-06 15:36:02
categories:
- ISA和处理器
- 处理器性能指标和计算单元
tags:
- TBD
---

# 简介

第一个版本的Dhrystone benchmark  是1984年开发，主要是用来测试整数运算的性能；

1988年，Dhrystone有第二个版本V2，支持Ada, C and Pascal languages  

==》之前流行的原因：

- 代码非常简单
- 编译和运行快
- 比 MIPS 更准确地衡量 CPU 性能

==》现在，Dhrystone已经不在是代表性的基准测试，因为局限性非常明显：

- 编译器优化问题
- 不能代表真实应用，只能测试整数运算，无法测试浮点计算、向量运算、多线程等
- result report形式不统一：Dhrystones per second、DMIPS、DMIPS/MHz

==》新基准测试

CoreMark

✅ 代码优化难度更大，编译器无法轻易优化掉计算，测试 CPU 实际运算能力

SPEC（Standard Performance Evaluation Corporation）

✅ 涵盖整数、浮点、存储、内存等多个领域，测试更全面，适用于高端CPU



<font color=blue>ARM使用的是Dhrystone 2.1  的C语言版本</font>

<font color=blue>结果计算公式：Dhrystones per second = number of runs / execution time </font> 

<font color=blue>为了结果的有效性，ARM推荐运行至少20秒；</font>



# 编译选项

```
dhry_1.c, dhry_2.c and dhry.h
```

不允许使用如下编译选项：

• function inlining
• multifile compilation  

## Library functions required by Dhrystone

- `scanf() and printf() `to read the iteration count input by the user and to report back the benchmarking results
- `clock() `to compute the execution time of the benchmark  
- `strcpy() and strcmp()` to perform string assignment and comparison  

> 上述函数在如下两种库中均支持：
>
> - `standard C library ` ，included with the ARM Compiler toolchain and the Keil™ MDK  
> - `microlib  （size-optimized C library ）`，included with the Keil MDK toolkit.  



## Command-line options



### Speed-optimized

```
armcc -c -W --cpu=cpuname -O3 -Otime --no_inline --no_multifile -DMSC_CLOCK dhry_1.c dhry_2.c
armlink dhry_1.o dhry_2.o -o dhry.axf

The compiler switches used here are:
-c Performs the compilation and link steps separately.
-W Disables all warnings.
--cpu=cpunameSpecifies the name of the target processor, for example --cpu=Cortex-A8.
-O3 Applies maximum optimization.
-Otime Optimizes for time.
--no_inline Disables function inlining, because Dhrystone demands “No procedure merging”.
--no_multifileDisables multifile compilation, because Dhrystone demands that the two source files be
compiled independently.
-DMSC_CLOCK Ensures the C library function clock() is used for timing measurements.
<other options>Processor-specific tuning options
```



### Size-optimized  

```
armcc -c -W --cpu=cpuname --thumb -O3 -Ospace --no_inline --no_multifile -DMSC_CLOCK dhry_1.c dhry_2.c
armlink dhry_1.o dhry_2.o -o dhry.axf

The compiler switches used here are the same as in the speed-optimized example, with the
following differences:
--thumb Compiles for Thumb.
-Ospace Optimizes for size.
```





# Running Dhrystone  

Dhrystone is a very small integer benchmark that must be run multiple times in order to obtain
reproducible numbers. ARM recommends you run Dhrystone ten times with varying iteration
counts and that each run takes at least 20 seconds.

In order to minimize the variation caused by inconsistent processor states, ARM recommends
you disregard the first run and calculate the average for the remaining nine runs. In addition,
when running Dhrystone on a bare-metal system, before each run you must initialize the
processor by resetting, clearing and enabling the caches, TLBs, BTBs and other
micro-architectural features such as branch prediction.  



# Measurement characteristics  



DMIPS = Dhrystones per second / 1757  

A more commonly reported figure is DMIPS / MHz  





