---
title: Cortex-M4 Hardfault Handler示例
date: 2026-07-31 16:11:55
categories:
- ARM
- Cortex-M4
tags:
- TBD
---



> If you only want to implement HardFault hander, then your handler program should do the fault type triage and take action correspondingly based on different types of faults.==》怎么去实现Hardfault handler？



# HardFault handler example

> 思路：在处理器进入 HardFault 时，找到异常现场所在的栈帧，把故障发生前的寄存器、PC、LR，以及 Fault 状态寄存器打印出来，供调试定位。
>
> 流程：
>
> ↓ 程序发生 Fault    
>
> ↓ 处理器硬件自动将寄存器压入异常栈帧   
>
>  ↓ LR 被写成 EXC_RETURN   
>
>  ↓ 跳转到 HardFault_Handler    
>
> ↓ 汇编代码判断异常栈帧位于 MSP 还是 PSP    
>
> ↓ 把栈帧地址和 EXC_RETURN 传给 C 函数    
>
> ↓ C 函数读取栈帧及 SCB Fault 寄存器   
>
>  ↓ 打印现场并停在 while(1)

```c++
/*
1、程序发生 Fault-》处理器硬件自动将寄存器压入当前程序所在的栈帧-》LR 被写成 EXC_RETURN
2、汇编函数中：
- 首先判断LR(EXC_RETURN)，获取异常栈帧的起始地址并保存到R0;
- 将LR(EXC_RETURN)保存到R1
- 使用 B 而不是普通的 BL，不会重新生成一个普通函数返回地址覆盖当前LR
3、因此最终跳转到HardFault_Handler_C的时候，根据函数调用约定;
R0：第一个参数
R1：第二个参数
会传递给C函数
*/
__asm void HardFault_Handler(void)
{
    TST    LR, #4
    ITE    EQ
    MRSEQ  R0, MSP
    MRSNE  R0, PSP
    MOV    R1, LR
    B      __cpp(HardFault_Handler_C)
}

/*

*/
void HardFault_Handler_C(unsigned long * hardfault_args, unsigned int lr_value)
{
  unsigned long stacked_r0;
  unsigned long stacked_r1;
  unsigned long stacked_r2;
  unsigned long stacked_r3;
  unsigned long stacked_r12;
  unsigned long stacked_lr;
  unsigned long stacked_pc;
  unsigned long stacked_psr;
  unsigned long cfsr;
  unsigned long bus_fault_address;
  unsigned long memmanage_fault_address;
  
  bus_fault_address       = SCB->BFAR;
  memmanage_fault_address = SCB->MMFAR;
  cfsr                    = SCB->CFSR;
 
  stacked_r0  = ((unsigned long) hardfault_args[0]);
  stacked_r1  = ((unsigned long) hardfault_args[1]);
  stacked_r2  = ((unsigned long) hardfault_args[2]);
  stacked_r3  = ((unsigned long) hardfault_args[3]);
  stacked_r12 = ((unsigned long) hardfault_args[4]);
  stacked_lr  = ((unsigned long) hardfault_args[5]);
  stacked_pc  = ((unsigned long) hardfault_args[6]);
  stacked_psr = ((unsigned long) hardfault_args[7]);
 
  printf ("[HardFault]\n");
  printf ("- Stack frame:\n"); 
  printf (" R0  = %x\n", stacked_r0);
  printf (" R1  = %x\n", stacked_r1);
  printf (" R2  = %x\n", stacked_r2);
  printf (" R3  = %x\n", stacked_r3);
  printf (" R12 = %x\n", stacked_r12);
  printf (" LR  = %x\n", stacked_lr);
  printf (" PC  = %x\n", stacked_pc);
  printf (" PSR = %x\n", stacked_psr);
  printf ("- FSR/FAR:\n");  
  printf (" CFSR = %x\n", cfsr);
  printf (" HFSR = %x\n", SCB->HFSR);
  printf (" DFSR = %x\n", SCB->DFSR);
  printf (" AFSR = %x\n", SCB->AFSR);
  if (cfsr & 0x0080) printf (" MMFAR = %x\n", memmanage_fault_address);
  if (cfsr & 0x8000) printf (" BFAR = %x\n", bus_fault_address);
  printf ("- Misc\n"); 
  printf (" LR/EXC_RETURN= %x\n", lr_value);
    
  while(1); // endless loop
}
```



## exception entry，异常入口序列

HardFault 被接受后，处理器开始执行 exception entry，异常入口序列。

<font color=blue>Summary：</font>

```tex
1. 硬件确定应该使用哪个栈保存现场
2. 硬件自动创建异常栈帧
3. 硬件把当前 LR 改写为 EXC_RETURN
4. 硬件进入 Handler mode
5. 硬件从向量表取 Hardfault_Handler 地址
```



<font color=blue>真正执行hardfault_handler程序时的处理器状态：</font>

```tex
PC   = HardFault_Handler 的地址
LR   = EXC_RETURN
Mode = Handler mode
SP   = MSP

MSP 或 PSP 中的某一个
     = 指向硬件保存的异常栈帧
```





1、硬件确定应该使用哪个栈保存现场

硬件压栈时使用的是：异常发生之前，当前上下文正在使用的栈指针

注意：

压栈使用的是“进入异常前的栈”，而进入 HardFault Handler 后，处理器运行在 Handler mode，当前执行栈为 MSP。



> Note:
>
> 可以分成三种情况。
>
> 情况1：Thread mode 使用 MSP
>
> 例如裸机程序通常默认如此：
>
> ```
> 异常前：
>     Mode = Thread
>     CONTROL.SPSEL = 0
>     当前栈 = MSP
> ```
>
> 那么异常栈帧压入 MSP。
>
> ------
>
> 情况2：Thread mode使用 PSP
>
> RTOS 任务通常使用 PSP：
>
> ```
> 异常前：
>     Mode = Thread
>     CONTROL.SPSEL = 1
>     当前栈 = PSP
> ```
>
> 那么异常栈帧压入 PSP。
>
> ------
>
> 情况3：异常发生在另一个 Handler 中
>
> Handler mode 始终使用 MSP，因此：
>
> ```
> 异常前：
>     Mode = Handler
>     当前栈 = MSP
> ```
>
> 那么异常栈帧压入 MSP。



2、硬件自动创建`异常栈帧`

硬件自动保存下面 8 个寄存器

Arm 官方将这一过程称为自动 stacking。它使中断函数可以像普通 C 函数一样执行，而不必先用软件保存寄存器；

<font color=blue>压栈完成后，新的栈指针指向 R0：</font>

```text
低地址
                 +------------------------+
SP + 0x00  --->  | R0                     |
                 +------------------------+
SP + 0x04        | R1                     |
                 +------------------------+
SP + 0x08        | R2                     |
                 +------------------------+
SP + 0x0C        | R3                     |
                 +------------------------+
SP + 0x10        | R12                    |
                 +------------------------+
SP + 0x14        | LR，异常前的 LR        |
                 +------------------------+
SP + 0x18        | PC，异常返回地址       |
                 +------------------------+
SP + 0x1C        | xPSR                   |
                 +------------------------+
高地址
```

> Note:
>
> 1、按照 AAPCS 调用约定：
>
> - `R0-R3、R12、LR` 属于调用者保存寄存器；
> - `R4-R11` 属于被调用者保存寄存器。
>
> 如果后续 C 函数需要使用 `R4-R11`，编译器生成的函数会按需保存。
>
> 2、Arm 说明异常栈帧为了双字对齐，异常入口可能增加一个 padding word



3、硬件把当前 LR 改写为 EXC_RETURN

异常发生前，普通程序中的 `LR` 是函数返回地址。异常入口时，硬件做两件不同的事：

a) 原来的 LR 被压入栈

> 原始 LR 保存在：
>
> ```
> 异常栈帧 SP + 0x14
> ```
>
> 也就是：
>
> ```
> stacked_lr = hardfault_args[5];
> ```
>
> 这个 `stacked_lr` 才是故障前程序的 LR。



b) Handler 当前看到的 LR 被改写

> 进入 HardFault Handler 后，寄存器 `LR` 不再是原来的函数返回地址，而是一个特殊编码：
>
> ```
> EXC_RETURN
> ```
>
> 典型值包括：
>
> ```
> 0xFFFFFFF1：返回 Handler mode，使用 MSP
> 0xFFFFFFF9：返回 Thread mode，使用 MSP
> 0xFFFFFFFD：返回 Thread mode，使用 PSP
> ```



4、硬件进入 Handler mode

完成压栈后，处理器切换到：

```tex
Mode = Handler mode
Privilege = Privileged
当前执行栈 = MSP
IPSR = HardFault 异常号
```



- <font color=blue>PSP->指向异常栈帧，Hardfault_Handler的栈指针使用的是MSP</font>

- <font color=blue>MSP->指向异常栈帧，Hardfault_Handler的栈指针使用的是MSP</font>



5、硬件从向量表取 Hardfault_Handler 地址

跳转到Hardfault_Handler 



Q&A：

<font color=red>M4有PSP和MSP两个寄存器？</font>

<font color=red>程序运行的过程中主栈和程序栈是分配的？都是在程序编译的时候指定的栈空间中？</font>





## __asm void HardFault_Handler

```assembly
TST    LR, #4
ITE    EQ
MRSEQ  R0, MSP
MRSNE  R0, PSP
    
# 等价于
if ((LR & 4) == 0)
    r0 = MSP;
else
    r0 = PSP;
```



因此这个汇编函数的作用就是:

- 将数据异常栈帧的指针保存到R0;

- 将LR保存到R1;



<font color=red>为什么要用汇编而不直接用C函数？</font>



### <font color=blue>使用`b`而不是`bl`进行跳转</font>

> Wrapper 最后使用：`b HardFault_HandlerC`,而不是：`bl HardFault_HandlerC`
>
> 原因是 `b` 不会修改 LR。因此进入 C Handler 时：`LR 仍然是 EXC_RETURN`
>
> 当 `HardFault_HandlerC()` 执行：`return;`
>
> 编译器最终会使用类似：`bx lr`
>
> 由于 LR 是 EXC_RETURN，处理器识别出这是一次异常返回，而不是普通函数返回。
>
> 处理器随后自动：
>
> 1. 从 MSP 或 PSP 中恢复寄存器；
> 2. 恢复修改后的 PC；
> 3. 返回 Thread mode 或原异常上下文；
> 4. 从新的 PC 地址继续运行。



### 变量传递

`R0`和`R1`最终传递给`HardFault_HandlerC(exc_stack_t *f, uint32_t exc_return)`，即

```text
R0 = 异常栈帧地址
R1 = EXC_RETURN
```







<font color=red>最后的函数跳转，变量是如何传递的，有什么规则？</font>



## void HardFault_Handler_C

- 打印异常栈帧的内容；
- 打印错误状态和地址；













## Q&A-TBD



==》为什么先写一个汇编包装函数

在执行Hardfault的时候，MSP还是异常栈帧的MSP嘛？



==》发生exception时的寄存器保存；



==》函数调用时的寄存器保存?

==》函数调用时的参数约定？





# Fault Handler中如何修改返回地址



> If you would like to directly ignore the repeatedly error response original fault or escalated-HardFault, then you could directly modify the return address on the stack frame (but this doesn't resolve the faulting cause).==》怎么直接修改栈中的内容？
>
> HardFault 发生时，硬件把返回地址 PC 压入当前栈；Handler 直接修改栈中的 PC。
>
> 异常返回时，处理器从栈中恢复这个被修改后的 PC，于是跳过导致 Fault 的指令，避免再次执行同一条指令而重复 HardFault。



```c++
  #include <stdint.h>
  #include "cm4ikmcu.h"
  #include "IKtests.h"

  /* Optional: enable BusFault during bring-up to avoid immediate escalation */
  #define SHCSR_BUSFAULTENA_Msk   (1UL << 17)

  /* SCB fault bits (CFSR/HFSR) */
  #define HFSR_FORCED_Msk         (1UL << 30)
  #define CFSR_BFSR_Msk           (0xFFUL << 8)
  #define CFSR_BFARVALID_Msk      (1UL << 15)
  #define CFSR_PRECISERR_Msk      (1UL << 9)
  #define CFSR_IMPRECISERR_Msk    (1UL << 10)

  /* Stacked frame pushed by hardware on exception entry */
  typedef struct {
    uint32_t r0, r1, r2, r3, r12, lr, pc, xpsr;
  } exc_stack_t;

  /* Control flags for "skip once" behavior */
  volatile uint32_t g_skip_fault_addr = 0;
  volatile uint32_t g_skip_fault_once = 0;
  volatile uint32_t g_fault_recovered = 0;
  volatile uint32_t g_last_cfsr = 0, g_last_hfsr = 0, g_last_bfar = 0;

  static uint32_t thumb_instr_size(uint16_t hw1)
  {
    /* 32-bit Thumb encodings start with 11101/11110/11111 */
    if ((hw1 & 0xE000u) == 0xE000u && (hw1 & 0x1800u) != 0x0000u) return 4u;
    return 2u;
  }

  /* Naked wrapper: pass active SP frame (MSP/PSP) into C handler */
  __attribute__((naked)) void HardFault_Handler(void)
  {
    __asm volatile(
      "tst lr, #4      \n"
      "ite eq          \n"
      "mrseq r0, msp   \n"
      "mrsne r0, psp   \n"
      "mov r1, lr      \n"
      "b HardFault_HandlerC \n");
  }

void HardFault_HandlerC(exc_stack_t *f, uint32_t exc_return)
  {
    (void)exc_return;

    uint32_t cfsr = SCB->CFSR;
    uint32_t hfsr = SCB->HFSR;
    uint32_t bfar = SCB->BFAR;

    g_last_cfsr = cfsr;
    g_last_hfsr = hfsr;
    g_last_bfar = bfar;

    /*
    跳过的条件：
    	BusFault 确实升级成了 HardFault
		它是精确错误，因此保存的 PC 可信
		BFAR 中的故障地址有效
		当前确实允许跳过一次
		实际故障地址与预期探测地址一致
	
    */
    /* Typical case: BusFault escalated to HardFault (FORCED=1) */
    if ((hfsr & HFSR_FORCED_Msk) &&
        (cfsr & CFSR_PRECISERR_Msk) &&
        (cfsr & CFSR_BFARVALID_Msk) &&
        g_skip_fault_once &&
        (bfar == g_skip_fault_addr)) {

      uint16_t hw1 = *(uint16_t *)(uintptr_t)f->pc;
      f->pc += thumb_instr_size(hw1);    /* skip faulting instruction */
      g_skip_fault_once = 0;
      g_fault_recovered = 1;

      /* sticky/W1C cleanup */
      SCB->CFSR = (cfsr & CFSR_BFSR_Msk);
      SCB->HFSR = HFSR_FORCED_Msk;
      return;
    }

    /* Imprecise bus fault: PC may not point to faulting instruction */
    if ((hfsr & HFSR_FORCED_Msk) && (cfsr & CFSR_IMPRECISERR_Msk)) {
      SCB->CFSR = (cfsr & CFSR_BFSR_Msk);
      SCB->HFSR = HFSR_FORCED_Msk;
      return;
    }

    MSG(("Unhandled HardFault CFSR=%08x HFSR=%08x BFAR=%08x PC=%08x\n",
         cfsr, hfsr, bfar, f->pc));
    while (1) { }
  }

/* Example usage around an expected single bad access */
  static void probe_reserved_addr_once(uint32_t addr)
  {
    volatile uint32_t data = 0xDEADBEEFu;

    g_skip_fault_addr = addr;
    g_skip_fault_once = 1;
    g_fault_recovered = 0;

    data = *(volatile uint32_t *)addr; /* may HardFault */

    if (g_fault_recovered) {
      MSG(("Recovered from faulting access at %08x\n", addr));
    } else {
      MSG(("Read completed: %08x\n", data));
    }
  }
```



# Final使用版本-for gcc compiler

```c++
#include "core_cm4.h" # SCB

  /* SCB fault bits (CFSR/HFSR) */
  #define HFSR_FORCED_Msk         (1UL << 30)
  #define CFSR_BFSR_Msk           (0xFFUL << 8)
  #define CFSR_BFARVALID_Msk      (1UL << 15)
  #define CFSR_PRECISERR_Msk      (1UL << 9)
  #define CFSR_IMPRECISERR_Msk    (1UL << 10)

  /* Stacked frame pushed by hardware on exception entry */
  typedef struct {
    uint32_t r0, r1, r2, r3, r12, lr, pc, xpsr;
  } exc_stack_t;


  static uint32_t thumb_instr_size(uint16_t hw1)
  {
    /* 32-bit Thumb encodings start with 11101/11110/11111 */
    if ((hw1 & 0xE000u) == 0xE000u && (hw1 & 0x1800u) != 0x0000u) return 4u;
    return 2u;
  }

  /* Naked wrapper: pass active SP frame (MSP/PSP) into C handler */
  __attribute__((naked)) void HardFault_Handler(void)
  {
    __asm volatile(
      "tst lr, #4      \n"
      "ite eq          \n"
      "mrseq r0, msp   \n"
      "mrsne r0, psp   \n"
      "mov r1, lr      \n"
      "b HardFault_Handler_C \n");
  }

void HardFault_Handler_C(exc_stack_t *f, uint32_t exc_return)
  {

    uint32_t shcsr = SCB->SHCSR;
    uint32_t cfsr = SCB->CFSR;
    uint32_t hfsr = SCB->HFSR;
    uint32_t dfsr = SCB->DFSR;
    uint32_t afsr = SCB->AFSR;
    uint32_t mmfar= SCB->MMFAR;
    uint32_t bfar = SCB->BFAR;

    // ****************************
    // exception stack
    // ****************************
    user_fputc("\033[1;34m exception stack\033[0m\n");
    user_fputc(" R0   = ");user_fputint32(f->r0    );user_fputc("\n");
    user_fputc(" R1   = ");user_fputint32(f->r1    );user_fputc("\n");
    user_fputc(" R2   = ");user_fputint32(f->r2    );user_fputc("\n");
    user_fputc(" R3   = ");user_fputint32(f->r3    );user_fputc("\n");
    user_fputc(" R12  = ");user_fputint32(f->r12   );user_fputc("\n");
    user_fputc(" LR   = ");user_fputint32(f->lr    );user_fputc("\n");
    user_fputc(" PC   = ");user_fputint32(f->pc    );user_fputc("\n");
    user_fputc(" xPSR = ");user_fputint32(f->xpsr  );user_fputc("\n");

    // ****************************
    // Hardfault Handler lr
    // ****************************
    user_fputc("\033[1;34m Hardfault Handler lr\033[0m\n");
    user_fputc(" EXC_RETURN = ");user_fputint32(exc_return  );user_fputc("\n");

    // ****************************
    // error control and status
    // ****************************
    user_fputc("\033[1;34m error control and status\033[0m\n");
    user_fputc(" SHCSR  = ");user_fputint32(shcsr   );user_fputc("\n");
    user_fputc(" CFSR   = ");user_fputint32(cfsr    );user_fputc("\n");
    user_fputc(" HFSR   = ");user_fputint32(hfsr    );user_fputc("\n");
    user_fputc(" DFSR   = ");user_fputint32(dfsr    );user_fputc("\n");
    user_fputc(" AFSR   = ");user_fputint32(afsr    );user_fputc("\n");
    user_fputc(" MMFAR  = ");user_fputint32(mmfar   );user_fputc("\n");
    user_fputc(" BFSR   = ");user_fputint32(bfar    );user_fputc("\n");

    // ****************************
    // precise error
    // ****************************
    if ((hfsr & HFSR_FORCED_Msk) &&
        (cfsr & CFSR_PRECISERR_Msk) &&
        (cfsr & CFSR_BFARVALID_Msk)) {

            user_fputc("\033[1;34m precise error\033[0m\n");
      uint16_t hw1 = *(uint16_t *)(uintptr_t)f->pc;
      f->pc += thumb_instr_size(hw1);    /* skip faulting instruction */

      /* sticky/W1C cleanup */
      SCB->CFSR = (cfsr & CFSR_BFSR_Msk);
      SCB->HFSR = HFSR_FORCED_Msk;
      return;
    }
    // ****************************
    // imprecise error
    // ****************************
    if ((hfsr & HFSR_FORCED_Msk) && (cfsr & CFSR_IMPRECISERR_Msk)) {
        user_fputc("\033[1;34m imprecise error\033[0m\n");
      SCB->CFSR = (cfsr & CFSR_BFSR_Msk);
      SCB->HFSR = HFSR_FORCED_Msk;
      return;
    }

    // ****************************
    // Unhandled HardFault
    // ****************************
    while (1) {
        /* 停留在HardFault处理程序中 */
        user_fputc("Unhandled HardFault, loop.....\n");
    }
}
```

