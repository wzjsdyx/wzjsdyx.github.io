---
title: ARM coremark
date: 2025-03-06 13:41:01
categories:
- ISA和处理器
- 处理器性能指标和计算单元
tags:
- TBD
---

# CoreMark简介

CoreMark是更加复杂的benchmark程序用于衡量现代处理的性能；



CoreMark是在2009年开发，非常容易获取以及非常容易移植；<font color=blue>其开发的初衷就是为了替代Dhrystone benckmark。</font>

Dhrystone是在1984年开发，在很多方面都存在问题，CoreMark可以进行弥补，具体如下：

## 工作负载的现实性

> Dhrystone 是一个**合成基准测试**（synthetic benchmark）：
>
> 它通过执行一系列简单的整数运算和字符串操作来测试处理器的性能；<font color=blue>是人为设计的一组计算任务，不包含实际应用中的关键算法，它的测试结果与现实应用的性能关联度较低。</font>
>
> 
>
> CoreMark 提供更真实的工作负载<font color=blue>（采用了一些在嵌入式系统中常见的计算任务）</font>：
>
> 1、矩阵运算（Matrix Manipulation）
>
> - 作用：测试 CPU 处理数学运算的能力（如加法、乘法）。
> - 现实应用：图像处理、信号处理、科学计算。
>
> 2、链表操作（Linked List Manipulation）
>
> - 作用：测试指针操作和内存管理能力。
> - 现实应用：操作系统、数据库、动态数据结构（如哈希表、图）
>
> 3、状态机操作（State Machine Operation）
>
> - 作用：测试 CPU 在不同条件下执行不同代码路径的能力（数据相关的分支）。
> - 现实应用：协议解析（如 TCP/IP 协议栈）、控制系统（如电梯控制、嵌入式控制器）。
>
> 4、循环冗余校验（CRC，Cyclic Redundancy Check）
>
> - 作用：测试 CPU 计算数据校验码的能力。
> - 现实应用：网络通信（数据包校验）、存储设备（数据完整性检查）。

## 编译器优化

> Dhrystone 的问题：容易被编译器优化
>
> - Dhrystone 的代码结构简单，其中许多计算的结果在编译时就可以被确定（称为 `compile-time constants`）。
> - **现代编译器**（如 GCC、LLVM）在优化时，会尝试**消除不必要的计算**（称为 `dead code elimination`）。
>
> Dhrystone程序容易受到编译器的影响，当这些发生的时候，其结果就不可相信；
>
> 
>
> CoreMark 会避免被编译器优化：
>
> - CoreMark需要计算的数值在编译时无法获取，只有在运行时才能得到；
>
> - 因此编译器虽然会进行一定程度的优化，但是**无法在编译阶段预计算结果**，会强制让 CPU 运行完整的计算过程

✅ **示例：Dhrystone 被优化的情况**

```c
int a = 5;
int b = 10;
int result = a + b;  // 编译时就知道 result = 15
```

➡ **编译器优化后：**

```c
int result = 15;  // 计算直接替换，CPU 实际没有做加法
```

➡ **影响**：Dhrystone 可能会得到**虚高**的性能分数，因为 CPU 可能根本没执行预期的计算！



✅ **示例：CoreMark 代码避免优化**

```
c复制编辑int compute(int x, int y) {
    return x * y + (x - y);  // x, y 在运行时才知道
}

int main() {
    int a = rand();  // 运行时随机生成
    int b = rand();
    int result = compute(a, b);
}
```

➡ **编译器无法优化：** `a` 和 `b` 只有在程序运行时才知道值，因此 CPU 仍然需要执行 `compute(a, b)` 的完整计算过程。

➡ **影响**：CPU仍然会执行对应的运算；



## 计时部分库函数的调用

> Dhrystone 的问题:
>
> - Dhrystone **在计时阶段** 调用了 **标准库函数**（如 `printf()`、`strcpy()` 等）
>
> - 库函数的执行时间** 依赖于 **编译器和库实现**，不同平台上的 **库函数性能可能差异很大**。
>
> **不同系统的 Dhrystone 分数可能受库函数影响，而不是 CPU 的真实计算能力**
>
> CoreMark 的改进：不在计时部分调用库函数
>
> - CoreMark 设计时，**所有计算都由基准测试代码自己实现**，不依赖外部库函数。
> - 这样可以保证**所有平台运行完全相同的计算工作负载**，不会因库实现不同而影响结果。

✅ **示例：Dhrystone 受库函数影响**

```
void test() {
    char str1[] = "Hello";
    char str2[10];
    strcpy(str2, str1);  // 调用了标准库函数
}
```

➡ **问题：**

- `strcpy()` 的执行效率取决于 **所用 C 标准库的实现**，不同编译器可能优化程度不同。
- **测的可能是库的性能，而不是 CPU 本身的计算能力**。



✅ **示例：CoreMark 避免库函数**

```
void my_strcpy(char *dst, const char *src) {
    while ((*dst++ = *src++));  // 手动实现字符串复制
}
```

➡ **优点：**

- **确保所有 CPU 运行相同的代码**，不受不同库函数实现的影响。
- **测试 CPU 计算能力，而不是测试库的优化程度**。



## 版本问题

Dhrystone没有一个固定的官方版本；

CoreMark有官方版本：http://www.coremark.org/  

- ANSI C 兼容性

>  ANSI C（American National Standards Institute C）指的是**标准化的 C 语言**。
>
> CoreMark 代码严格遵循 ANSI C 标准，确保**在所有符合 ANSI C 的编译器上都能正确编译和运行**。
>
> - **ANSI C（C89/C90）**：1989 年标准化，保证代码可以在不同编译器和系统上运行。
> - **非 ANSI C 代码** 可能会使用 **编译器特定的语法**，导致在某些平台上无法编译或运行。
>
> Dhrystone 的最早版本**在 ANSI C 标准发布前编写**，使用了一些 **非标准 C 语法**
>
> CoreMark 代码严格遵循 ANSI C 标准，确保**在所有符合 ANSI C 的编译器上都能正确编译和运行**。

## 结果报告

> Dhrystone 的问题:
>
> **测试方法不统一**：Dhrystone **虽然提供了一些测试指南**，但这些指南**不是强制执行的**，不同测试者可能会**以不同方式运行基准测试**。
>
> 结果格式不统一：Dhrystones per second, DMIPS, or DMIPS/MHz  
>
> CoreMark 的改进：
>
> 统一的测试和报告标准

<font color=blue>ARM使用CoreMark 1.0版本的程序：</font>

```
CoreMark 1.0 : N / C [/ P] [/ M]
where:
N displays the number of iterations per second with seeds 0,0,0x66,size=2000.
C displays the compiler version and flags specified when compiling the benchmark.
P displays parameters such as data and code allocation specifics.
• This parameter can be omitted if all data was allocated on the heap in RAM.
• This parameter cannot be omitted when reporting CoreMark/MHz.
M shows the type of parallel algorithm execution (if used) and number of contexts.
• This parameter can be omitted if parallel execution is not used.
For example:
CoreMark 1.0 : 256.344527 / ARM C/C++ Compiler, 5.03 [Build 24] -O3 -Otime / STACK
```

<font color=blue>ARM推荐运行10000次迭代或者30秒</font>



# 编译CoreMark

CoreMark包含如下文件：

```
• coremark.h
• core_main.c
• core_list_join.c
• core_matrix.c
• core_state.c
• core_util.c
• simple/core_portme.c
• simple/core_portme.h
```

需要移植的话，就修改core_portme文件

## 使用到的库函数

- `printf()`
- 特定于平台的计时函数，用于计算benchmark执行时间，通常是C library的`clock()`函数，或者使用core_portme.c中的`start_time() `and `stop_time()  `

> printf()函数以及clock()函数通常在如下的library中：
>
> - `standard C library ` ，included with the ARM Compiler toolchain and the Keil™ MDK  
> - `microlib  （size-optimized C library ）`，included with the Keil MDK toolkit.  

当编译CoreMark的在bare-metal system时，就需要移植这两个函数，`printf() 和clock()`

现在和很多处理器中，包含性能计数器，可以重定向clock函数即精确的统计执行benchmark所需要的时间；

## 内存布局

通过link文件，将代码和数据放在运行最高效的位置。通过使用link文件来进行内存布局；



## 编译命令行选项

优化执行速度：-Otime, -O3, and --loop_optimization_level=2  

优化代码size：use microlib instead of the standard C library  

### Speed-optimized

```
armcc -c -W <cflags> --cpu=name -O3 -Otime --loop_optimization_level=2
-I./ -Isimple -DITERATIONS=0 -DSEED_METHOD=SEED_ARG
-DCOMPILER_FLAGS=\”<cflags> --cpu=name -O3 -Otime --loop_optimization_level=2\”
core_main.c core_list_join.c core_matrix.c core_state.c core_util.c
simple/core_portme.c

armlink core_main.o core_list_join.o core_matrix.o core_state.o core_util.o
core_portme.o -o coremark.axf

The compiler switches used here are:

-c Performs the compilation and link steps separately.

-W Disables all warnings. This is optional.

<cflags> Any additional compiler flags you require.

--cpu=name Specifies the name of the target processor, for example --cpu=Cortex-M4.

-O3 Applies maximum optimization.

-Otime Optimizes for time.

-I./ -Isimple
Lists the directories to search for source files.

--loop_optimization_level=2
Specifies that the compiler performs high-level optimization, including
aggressive loop optimization. This option is usually best for performance.

-DSEED_METHOD=SEED_ARG
Specifies that randomization seeds are passed as command-line arguments at
runtime, rather than being automatically generated.

-DITERATIONS=0
Specifies that the number of iterations is passed as a command-line argument at
runtime.
Note
If you prefer, you can specify the number of iterations here rather than passing the
iteration count as an argument. See Running CoreMark on page 10 for more
information.

-DCOMPILER_FLAGS=\”<cflags> --cpu=name -O3 -Otime --loop_optimization_level=2\”
Specifies the compiler flags, as provided to the compiler earlier on the command
line. This allows the compiler flags to be included in the output of the CoreMark
results.
This example uses UNIX-style escaped quotes. On Windows, use
-DCOMPILER_FLAGS=”””<cflags> --cpu=name...”””
```



### Speed-optimized, with microlib  

```
armcc -c -W <cflags> --cpu=name -O3 -Otime --loop_optimization_level=2
--library_type=microlib -I./ -Isimple -DITERATIONS=0 -DSEED_METHOD=SEED_ARG
-DCOMPILER_FLAGS=\”<cflags> --cpu=name -O3 -Otime --loop_optimization_level=2
--library_type=microlib\”
core_main.c core_list_join.c core_matrix.c core_state.c core_util.c
simple/core_portme.c

armlink core_main.o core_list_join.o core_matrix.o core_state.o core_util.o
core_portme.o -o coremark.axf

The compiler and linker switches used here are the same as in the speed-optimized example,
with the following difference:
--library_type=microlib
Links with the size-optimized library
```



### Size-optimized, with microlib  

```

armcc -c -W <cflags> --cpu=cpuname -O3 -Ospace -I./ -Isimple -DITERATIONS=0
-DSEED_METHOD=SEED_ARG -DCOMPILER_FLAGS=\”<cflags> --cpu=cpuname -O3 -Ospace\”
--library_type=microlib --split_sections core_main.c core_list_join.c
core_matrix.c core_state.c core_util.c simple/core_portme.c

armlink core_main.o core_list_join.o core_matrix.o core_state.o core_util.o
core_portme.o -o coremark.axf

The compiler and linker switches used here are the same as in the speed-optimized example,
with the following differences:
--library_type=microlib
Links with the size-optimized library.
-Ospace Optimizes for size.
--split_sections
Generates one ELF section for each function in the source file.
Compilers usually collect functions and data together and emit one section for
each category. The linker can only eliminate a section if it is entirely unused. The
--split_sections option ensures that each function is in a separate section, so that
the linker can eliminate all unused functions.
```



# 运行CoreMark

CoreMark是一个小的benchmark程序，必须要运行多次才能获得可复制的分数

1. **减少处理器状态不一致带来的影响**

- **问题描述**：
  处理器的状态（如缓存状态、分支预测状态等）可能会影响测试结果。如果每次运行 CoreMark 时处理器的初始状态不一致，测试结果可能会有较大波动。
- **解决方法**：
  ARM 建议按照以下步骤运行 CoreMark：
  1. **两次验证运行（Validation Runs）**：
     在正式测试之前，先运行两次 CoreMark，以确保测试环境正确设置，并且处理器状态趋于稳定。
  2. **至少十次性能运行（Profile Runs）**：
     在验证运行之后，至少运行十次 CoreMark，以收集足够的数据。
  3. **计算平均值**：
     对十次性能运行的结果取平均值，作为最终的测试结果。
- **目的**：
  通过多次运行和取平均值，可以减少处理器状态不一致带来的波动，提高测试结果的可靠性。

------

2. **最小执行时间要求**

- **CoreMark 要求**：
  CoreMark 的官方要求是，每次运行的执行时间至少为 **10 秒**，以确保测试结果的准确性。
- **ARM 建议**：
  ARM 建议将最小执行时间延长至 **30 秒**，以进一步减少测试结果的波动性。
- **原因**：
  - 较长的执行时间可以平滑短时间内的性能波动（如中断、缓存未命中等）。
  - 较长的执行时间还可以更好地反映处理器的持续性能。

------

3. **执行时间与处理器频率的关系**

- **执行时间的影响因素**：
  CoreMark 的执行时间取决于处理器的频率和性能。频率越高，执行时间越短；频率越低，执行时间越长。
- **示例**：
  ARM 提供了一个参考示例：
  - 在 **Cortex-M4 处理器** 上，运行频率为 **168 MHz** 时，**15000 次迭代** 的执行时间约为 **30 秒**。
- **如何调整迭代次数**：
  如果处理器的频率不同，可以通过调整迭代次数来控制执行时间。例如：
  - 如果处理器频率较低，可以增加迭代次数以达到 30 秒的执行时间。
  - 如果处理器频率较高，可以减少迭代次数以避免执行时间过长。

# 测量特性

The CoreMark benchmark number is the number of iterations per second  。

A commonly reported figure is CoreMarks / MHz.  

