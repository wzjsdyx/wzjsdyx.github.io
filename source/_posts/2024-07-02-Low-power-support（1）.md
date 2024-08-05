---
title: 6 Low-power support（1）
date: 2024-07-02 20:49:45
categories:
- 基于Cortex-M的SoC设计
- 理论知识：基于Cortex-M的SoC设计指南
tags:
- low power
- TBD
---



# Overview of low-power Cortex-M features

现在的MCU追究非常机制的能效比；

<font color=blue>Cortex-M系列处理器提供了很多的低功耗特性：</font>

- sleep mode：sleep mode和deep sleep mode
- 多个模块级别的时钟门控，以及子模块级别的时钟门控（有些时候也称为架构门控时钟）
- 状态保持电源门口SRPG（State retention power gating  ）
- WIC
- sleep-on-exit：应用在中断驱动的场景中



<font color=blue>Cortex-M系列处理器还有很多能降低功耗的优点：</font>

- 代码密度高：降低存储空间要求；
- core的面积小：门数少
- 性能优，程序执行的快，减少运行时间；
- 低中断延时以及中断处理优化：能够降低中断处理时的功耗



<font color=blue>还有些系统级别的低功耗策略：</font>

例如将SRAM分成多个bank，某些场景下关闭不用的bank，来降低功耗；

# Low-power design basics

## clock gating

时钟门控clock gating是最基本的低功耗技术之一。

### 寄存器级别的时钟门控：

综合工具会将使能信号综合成寄存器时钟门控，对Verilog RTL的代码风格有要求

{% asset_img image-20240709093742706.png%}

<font color=blue></font>

### 模块级别的时钟门控

<font color=blue></font>

处理器内部各个功能模块的时钟信号受到时钟门控，通过例化时钟门控CELL，这种做法也叫做ACG（architectural clock gating  ）；

{% asset_img image-20240709094847853.png %}



很多的Cortex-M处理器有多个top_level级别的时钟信号，designer需要自行添加时钟门控；

{% asset_img image-20240709095233535.png %}

> 可以使用debug power control signals CDBGPWRUPREQ and CDBGPWRUPACK  去门控debug CLK
>
> 可以使用GATEHCLK signal   去门控system CLK



## power gating

有些场景下仅仅关闭时钟还是无法满足低功耗场景的要求，这时就可以考虑power gating；

power gating技术需要：

- power switch transistors  
- isolation  cell
- clamping  cell

> **电源开关晶体管 (Power Switch Transistors)**:
>
> - 这些晶体管用于实现电源门控，即通过控制它们的通断来控制电路的电源供应。当电源开关晶体管关闭时，相关电路部分与电源完全隔离，以减少静态功耗。
>
> **信号隔离 (Signal Isolation)**:
>
> - 在电源门控中，关闭电源时，需要确保电路输入和输出信号不会影响其他部分。为了实现这一点，通常会使用专门的电路单元或电路设计技巧，如电流隔离器或传输门，以保持断开电源时的信号完整性和稳定性。
>
> **信号夹持 (Signal Clamping)**:
>
> - 电源门控时，有时需要对电路输入或输出信号进行夹持，以防止信号超过安全电压范围。夹持电路通常包括二极管或其他电路元件，用于限制信号的振幅或保护相关电路部分不受损坏。
> - 信号夹持就是将对外的value输出为high或者low

{% asset_img image-20240709100104211.png %}

### SRPG（State Retention Power Gating  ）

<font color=blue></font>

power gating的一个最大的缺点就是掉点后状态会丢失，为了解决这个问题，引入了新的技术，SRPG（State Retention Power Gating  ），其在寄存器内部引入状态保持的逻辑，并且使用独立的电源供电。

- 寄存器除了状态保持部分，其余部分均会掉电，因此也会节省很多功耗；
- funciton功能下，此种寄存器的功耗比普通寄存器的功耗更高，因此不适合经常toggle的寄存器；



## 其他低功耗技术

纯粹的数字逻辑一般会用到上述的低功耗技术。在其他方面还有低功耗技术，例如

- memory macro的低功耗技术；
- 外设组件，例如ADC有其自身特有的低功耗技术



# Cortex-M low-power interfaces  

## Sleep status and GATEHCLK output  

<font color=blue>大多数的Cortex-M处理器具有如下的低功耗状态输出信号：</font>

{% asset_img image-20240712094613888.png %}

---

<font color=blue>==》SLEEPING和SLEEPDEEP信号的说明</font>

从架构上来说，处理器支持两种睡眠模式，sleep mode和deep sleep mode。

进入这两种睡眠模式的机制是相同的：

1、首先执行WFI或者WFE；

2、启用了sleep-on-exit   特性，处理器从异常处理程序返回到线程级别；

> 思考：
>
> 这里关于sleep-on-exit信号是不是说错了？sleep-on-exit特性使能的时候，中断处理结束会直接进入sleep mode。

Cortex-M处理器进入哪种睡眠模式，根据SCR（System Control Register  ）的配置。

处理器内部对这两种模式的处理是相同的，SLEEP和SLEEPDEEP信号的作用是，让system designer去定义系统级的低功耗策略。

WIC只在deepsleep mode的时候才会使用；

---

<font color=blue>==》GATEHCLK信号的说明</font>

当处理器是处于睡眠模式的时候，处理器的AHB-Lite总线接口仍然可能发出debug access访问；

因此，就需要一个额外的GATEHCLK信号用来表示处理器处于睡眠状态，并且此时没有AHB-Lite总线访问（例如debug access导致的AHB-Lite transaction）。

---

以上这些信号的使用特定于MCU，没有固定的使用规则；

例如，如果GATEHCLK信号是高电平的时候，可以：

- 将系统BUS clock gate off

- SRAM进入低功耗模式

- 部分外设被关闭；



## Q-channel low-power interface

> (Applicable to Cortex-M23, Cortex-M33,Cortex-M35P)

部分较新的处理器（Cortex-M23, Cortex-M33,Cortex-M35P）支持Q channel，这是AMBA 4 低功耗接口中定义的握手协议；

略





## Sleep hold interface

<font color=blue>sleep hold interface的作用</font>

当处理器从sleep mode唤醒的时候，可以使用这个接口延缓程序的执行；

<font color=blue>sleep hold interface的使用场景</font>

处理器的运行需要SRAM，但是SRAM可能需要一定的时间才能从低功耗状态退出，在SRAM退出低功耗状态时，CPU不应该执行程序，此时就可以使用这个端口；

当在deep sleep mode的时候，`如果采用WIC的方案，整个特性就很少会被使用`。因为当使用WIC的时候，此时的门控机制会关闭处理器的所有时钟，让其处于低功耗模式；

> 思考：
>
> <font color=red>没太懂，为啥使用WIC方案的时候，sleep hold 这个特性很少会用到？</font>

{% asset_img image-20240721172648556.png %}

> 这两个信号都是低电平有效，一般和功耗管理单元PMU交互

<font color=blue>sleep hold interface的使用方式</font>

1、当PMU检测到SLEEP/DEEPSLEEP信号的时候，会assert SLEEPHOLDREQn；

2、处理器返回SLEEPHOLDACKn

- 如果处理器 asset SLEEPHOLDACKn，此时PMU就可以关闭memory，外设，时钟等等；
- 如果处理器没有asset SLEEPHOLDACKn，此时意味着，处理器又接收到中断或者debugger request，直接就会被唤醒；在这种情况下，PMU就不应该进行下一步的行为，并且在SLEEPING or SLEEPDEEP  de-assert的时候，需要de-assert SLEEPHOLDACKn；

3、如果处理器 asset SLEEPHOLDACKn，唤醒中断到来的时候，处理器将会de-asset sleep signal，此时处理器并不会立刻执行程序，而是需要等到SLEEPHOLDREQn  deassert

{% asset_img image-20240722163855336.png %}

> 思考：
>
> 1、<font color=red>SLEEPHOLDACKn  会在什么时候从core返回？？</font>
>
> 2、在extended sleep  阶段，GATEHCLK有可能还是高电平，此时HCLK会被gating off；（特定于处理器，不同处理器的行为可能不相同）
>
> 3、目前接触的项目中，并没有使用这个特性；

<font color=blue>如果不需要这个sleep hold feature  ，需要将SLEEPHOLDREQn  = 1‘b1;</font>





## Wakeup Interrupt Controller (WIC)

<font color=blue>为什么需要WIC</font>

如果将CORE的所有时钟都关闭，或者CORE处于掉电状态（最大程度的降低CORE的功耗），那么NVIC此时就不能检测唤醒事件，此时就需要WIC。

### WIC接口

WIC是一个可选的模块，处于always-on power domain。当NVIC没有时钟或者掉电的时候，WIC起到检测中断事件的作用。

WIC接口特定于处理器，但是一般来说，：

- WIC和处理器的接口如下：

{% asset_img image-20240722165602006.png %}

- WIC和系统的接口如下：

{% asset_img image-20240723104002065.png %}

### WIC demo 代码

<font color=blue>Cortex-M产品包交付的WIC是一个示例逻辑，允许修改（如下面代码所示）。在某些情况下，designer会将WIC设计成基于锁存器的操作，这样的话，唤醒时间的检测和capture就无需任何时钟;</font>

```verilog
//------------------------------------------------------------------------------
// The confidential and proprietary information contained in this file may
// only be used by a person authorised under and to the extent permitted
// by a subsisting licensing agreement from ARM Limited.
//
//            (C) COPYRIGHT 2004-2010 ARM Limited.
//                ALL RIGHTS RESERVED
//
// This entire notice must be reproduced on all copies of this file
// and copies of this file may only be made by a person if such person is
// permitted to do so under the terms of a subsisting license agreement
// from ARM Limited.
//
//  Revision            : $Revision: 141802 $
//  Release information : cortexm4_r0p1_00rel0
//
//------------------------------------------------------------------------------
// Purpose: Wake-Up Interrupt Controller (WIC)
//------------------------------------------------------------------------------

module cm4_wic
  (// Inputs
   FCLK, RESETn, WICLOAD, WICCLEAR, WICINT, WICMASK, WICENREQ,
   WICDSACKn,
   // Outputs
   WAKEUP, WICSENSE, WICPEND, WICDSREQn, WICENACK
  );

  //----------------------------------------------------------------------------
  // Parameters
  //----------------------------------------------------------------------------
  parameter WIC_PRESENT = 0;  // WIC present if 1.
  parameter WIC_LINES   = 8;  // Number of WIC lines. Min value is 3.

  //----------------------------------------------------------------------------
  // Port declarations
  //----------------------------------------------------------------------------

  // Clocks and resets
  input                  FCLK;
  input                  RESETn;

  // WIC interface
  input                  WICLOAD;     // WIC mask load from core
  input                  WICCLEAR;    // WIC mask clear from core
  input  [WIC_LINES-1:0] WICINT;      // Interrupt request from system
  input  [WIC_LINES-1:0] WICMASK;     // Mask from core
  input                  WICENREQ;    // WIC enable request from PMU
  input                  WICDSACKn;   // WIC enable ack from core
  output [WIC_LINES-1:0] WICSENSE;    // Input lines that can generate WAKEUP
  output [WIC_LINES-1:0] WICPEND;     // Pended interrupt request
  output                 WICDSREQn;   // WIC enable request to core
  output                 WICENACK;    // WIC enable ack to PMU
  output                 WAKEUP;      // Wake up request to PMU

  //----------------------------------------------------------------------------
  // Signal declarations
  //----------------------------------------------------------------------------

  // Pending and status registers
  reg    [WIC_LINES-1:0] wic_sense;        // Interrupt mask register
  wire   [WIC_LINES-1:0] nxt_wic_sense;
  wire                   wic_sense_we;
  reg    [WIC_LINES-1:0] wic_pend;         // Interrupt pend register
  wire   [WIC_LINES-1:0] nxt_wic_pend;
  wire                   wic_pend_we;
  reg                    wic_active;       // Active status register
  wire                   wic_active_we;

  // WIC enable handshake signals
  reg                    wic_ds_req;
  wire                   set_wic_ds_req;
  wire                   clr_wic_ds_req;
  wire                   wic_ds_req_we;
  reg                    wic_en_ack;
  wire                   set_wic_en_ack;
  wire                   clr_wic_en_ack;
  wire                   wic_en_ack_we;

  // PMU wakeup
  wire                   wakeup_pmu;       // Request wakeup to PMU

  // Logic removal terms
  wire                   opt_mst_wic_en;
  wire                   opt_wakeup;
  wire   [WIC_LINES-1:0] opt_wic_sense;
  wire   [WIC_LINES-1:0] opt_wic_pend;
  wire                   opt_wic_ds_req_n;
  wire                   opt_wic_en_ack;

  //----------------------------------------------------------------------------
  // Logic removal
  //----------------------------------------------------------------------------
  assign opt_wakeup       = (WIC_PRESENT != 0) ? wakeup_pmu  : 1'b0;
  assign opt_wic_sense    = (WIC_PRESENT != 0) ? wic_sense   : {WIC_LINES{1'b0}};
  assign opt_wic_pend     = (WIC_PRESENT != 0) ? wic_pend    : {WIC_LINES{1'b0}};
  assign opt_wic_ds_req_n = (WIC_PRESENT != 0) ? ~wic_ds_req : 1'b1;
  assign opt_wic_en_ack   = (WIC_PRESENT != 0) ? wic_en_ack  : 1'b0;
  assign opt_mst_wic_en   = (WIC_PRESENT != 0) ? 1'b1        : 1'b0;

  //----------------------------------------------------------------------------
  // WIC enable handshake
  //----------------------------------------------------------------------------

  // Request WIC mode sleep if PMU request is asserted and debugger is not
  // connected
  assign set_wic_ds_req = WICENREQ & opt_wic_ds_req_n;
  assign clr_wic_ds_req = ~opt_wic_ds_req_n & ~WICENREQ;
  assign wic_ds_req_we  = opt_mst_wic_en & (set_wic_ds_req | clr_wic_ds_req);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_ds_req <= 1'b0;
    else if (wic_ds_req_we)
      wic_ds_req <= set_wic_ds_req;

  // Acknowledge PMU request if core accepts and the debugger is not connected
  assign set_wic_en_ack = ~WICDSACKn & ~opt_wic_en_ack;
  assign clr_wic_en_ack = opt_wic_en_ack & WICDSACKn;
  assign wic_en_ack_we  = opt_mst_wic_en & (set_wic_en_ack | clr_wic_en_ack);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_en_ack <= 1'b0;
    else if (wic_en_ack_we)
      wic_en_ack <= set_wic_en_ack;

  //----------------------------------------------------------------------------
  // LOAD/CLEAR WIC sensitivity
  //----------------------------------------------------------------------------

  // Set mask value depending on operation
  assign nxt_wic_sense = {WIC_LINES{WICLOAD}} & WICMASK;
  assign wic_sense_we  = opt_mst_wic_en & (WICCLEAR | WICLOAD);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_sense <= {WIC_LINES{1'b0}};
    else if (wic_sense_we)
      wic_sense <= nxt_wic_sense;

  //----------------------------------------------------------------------------
  // Pend interrupts
  //----------------------------------------------------------------------------

  assign nxt_wic_pend = {WIC_LINES{~WICCLEAR}} & (WICINT | wic_pend);
  assign wic_pend_we  = opt_mst_wic_en & (WICLOAD | wic_active) & (WICCLEAR | (|WICINT));

  always @( posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_pend <= {WIC_LINES{1'b0}};
    else if (wic_pend_we)
      wic_pend <= nxt_wic_pend;

  //----------------------------------------------------------------------------
  // Active status register
  //----------------------------------------------------------------------------

  assign wic_active_we = opt_mst_wic_en & (WICLOAD | WICCLEAR);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_active <= 1'b0;
    else if (wic_active_we)
      wic_active <= WICLOAD;

  //----------------------------------------------------------------------------
  // Request wake-up
  //----------------------------------------------------------------------------

  // Wake up if something has been pended that is observed whilst powered down
  assign wakeup_pmu = (|(wic_pend & wic_sense));

  //----------------------------------------------------------------------------
  // Output assignments
  //----------------------------------------------------------------------------
  assign WAKEUP    = opt_wakeup;
  assign WICSENSE  = opt_wic_sense;
  assign WICPEND   = opt_wic_pend;
  assign WICDSREQn = opt_wic_ds_req_n;
  assign WICENACK  = opt_wic_en_ack;


  //----------------------------------------------------------------------------
  // OVL assertions
  //----------------------------------------------------------------------------

`ifdef ARM_ASSERT_ON
  assert_never_unknown #(0,1,0, "wic_ds_req_we must not be unknown")
  ts_wic_ds_req_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_ds_req_we));

  assert_never_unknown #(0,1,0, "wic_sense_we must not be unknown")
  ts_wic_sense_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_sense_we));

  assert_never_unknown #(0,1,0, "wic_pend_we must not be unknown")
  ts_wic_pend_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_pend_we));

  assert_never_unknown #(0,1,0, "wic_active_we must not be unknown")
  ts_wic_active_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_active_we));

`endif
endmodule
```

### WIC修改后基于锁存的代码

项目中的修改：

```verilog
//------------------------------------------------------------------------------
// The confidential and proprietary information contained in this file may
// only be used by a person authorised under and to the extent permitted
// by a subsisting licensing agreement from ARM Limited.
//
//            (C) COPYRIGHT 2004-2010 ARM Limited.
//                ALL RIGHTS RESERVED
//
// This entire notice must be reproduced on all copies of this file
// and copies of this file may only be made by a person if such person is
// permitted to do so under the terms of a subsisting license agreement
// from ARM Limited.
//
//  Revision            : $Revision: 141802 $
//  Release information : cortexm4_r0p1_00rel0
//
//------------------------------------------------------------------------------
// Purpose: Wake-Up Interrupt Controller (WIC)
//------------------------------------------------------------------------------

module cm4_wic
  (// Inputs
   TESTMODE, SCANCLK, FCLK, RESETn, WICLOAD, WICCLEAR, WICINT, WICMASK, WICENREQ,
   WICDSACKn,
   // Outputs
   WAKEUP, WICSENSE, WICPEND, WICDSREQn, WICENACK
  );

  //----------------------------------------------------------------------------
  // Parameters
  //----------------------------------------------------------------------------
  parameter WIC_PRESENT = 0;  // WIC present if 1.
  parameter WIC_LINES   = 8;  // Number of WIC lines. Min value is 3.

  //----------------------------------------------------------------------------
  // Port declarations
  //----------------------------------------------------------------------------
  input                  TESTMODE;

  // Clocks and resets
  input                  SCANCLK;
  input                  FCLK;
  input                  RESETn;
  
  // WIC interface
  input                  WICLOAD;     // WIC mask load from core
  input                  WICCLEAR;    // WIC mask clear from core
  input  [WIC_LINES-1:0] WICINT;      // Interrupt request from system
  input  [WIC_LINES-1:0] WICMASK;     // Mask from core
  input                  WICENREQ;    // WIC enable request from PMU
  input                  WICDSACKn;   // WIC enable ack from core
  output [WIC_LINES-1:0] WICSENSE;    // Input lines that can generate WAKEUP
  output [WIC_LINES-1:0] WICPEND;     // Pended interrupt request
  output                 WICDSREQn;   // WIC enable request to core
  output                 WICENACK;    // WIC enable ack to PMU
  output                 WAKEUP;      // Wake up request to PMU

  //----------------------------------------------------------------------------
  // Signal declarations
  //----------------------------------------------------------------------------

  // Pending and status registers
  reg    [WIC_LINES-1:0] wic_sense;        // Interrupt mask register
  wire   [WIC_LINES-1:0] nxt_wic_sense;
  wire                   wic_sense_we;
  reg    [WIC_LINES-1:0] wic_pend;         // Interrupt pend register
  wire   [WIC_LINES-1:0] nxt_wic_pend;
  wire                   wic_pend_we;
  reg                    wic_active;       // Active status register
  wire                   wic_active_we;

  // WIC enable handshake signals
  reg                    wic_ds_req;
  wire                   set_wic_ds_req;
  wire                   clr_wic_ds_req;
  wire                   wic_ds_req_we;
  reg                    wic_en_ack;
  wire                   set_wic_en_ack;
  wire                   clr_wic_en_ack;
  wire                   wic_en_ack_we;

  // PMU wakeup
  wire                   wakeup_pmu;       // Request wakeup to PMU

  // Logic removal terms
  wire                   opt_mst_wic_en;
  wire                   opt_wakeup;
  wire   [WIC_LINES-1:0] opt_wic_sense;
  wire   [WIC_LINES-1:0] opt_wic_pend;
  wire                   opt_wic_ds_req_n;
  wire                   opt_wic_en_ack;

  //----------------------------------------------------------------------------
  // Logic removal
  //----------------------------------------------------------------------------
  assign opt_wakeup       = (WIC_PRESENT != 0) ? wakeup_pmu  : 1'b0;
  assign opt_wic_sense    = (WIC_PRESENT != 0) ? wic_sense   : {WIC_LINES{1'b0}};
  assign opt_wic_pend     = (WIC_PRESENT != 0) ? wic_pend    : {WIC_LINES{1'b0}};
  assign opt_wic_ds_req_n = (WIC_PRESENT != 0) ? ~wic_ds_req : 1'b1;
  assign opt_wic_en_ack   = (WIC_PRESENT != 0) ? wic_en_ack  : 1'b0;
  assign opt_mst_wic_en   = (WIC_PRESENT != 0) ? 1'b1        : 1'b0;

  //----------------------------------------------------------------------------
  // WIC enable handshake
  //----------------------------------------------------------------------------

  // Request WIC mode sleep if PMU request is asserted and debugger is not
  // connected
  assign set_wic_ds_req = WICENREQ & opt_wic_ds_req_n;
  assign clr_wic_ds_req = ~opt_wic_ds_req_n & ~WICENREQ;
  assign wic_ds_req_we  = opt_mst_wic_en & (set_wic_ds_req | clr_wic_ds_req);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_ds_req <= 1'b0;
    else if (wic_ds_req_we)
      wic_ds_req <= set_wic_ds_req;

  // Acknowledge PMU request if core accepts and the debugger is not connected
  assign set_wic_en_ack = ~WICDSACKn & ~opt_wic_en_ack;
  assign clr_wic_en_ack = opt_wic_en_ack & WICDSACKn;
  assign wic_en_ack_we  = opt_mst_wic_en & (set_wic_en_ack | clr_wic_en_ack);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_en_ack <= 1'b0;
    else if (wic_en_ack_we)
      wic_en_ack <= set_wic_en_ack;

  //----------------------------------------------------------------------------
  // LOAD/CLEAR WIC sensitivity
  //----------------------------------------------------------------------------

  // Set mask value depending on operation
  assign nxt_wic_sense = {WIC_LINES{WICLOAD}} & WICMASK;
  assign wic_sense_we  = opt_mst_wic_en & (WICCLEAR | WICLOAD);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_sense <= {WIC_LINES{1'b0}};
    else if (wic_sense_we)
      wic_sense <= nxt_wic_sense;

  //----------------------------------------------------------------------------
  // Pend interrupts
  //----------------------------------------------------------------------------

  assign nxt_wic_pend = {WIC_LINES{~WICCLEAR}} & (WICINT | wic_pend);

  //AWIC modify by zhbin 20200826
  //assign wic_pend_we  = opt_mst_wic_en & (WICLOAD | wic_active) & (WICCLEAR | (|WICINT));
  //always @(posedge FCLK or negedge RESETn)
  //  if (!RESETn)
  //    wic_pend <= {WIC_LINES{1'b0}};
  //  else if (wic_pend_we)
  //    wic_pend <= nxt_wic_pend;
  wire  wicclear_tmp;
  dtc_buf  dtc_dly_wicclr_tmp( .I( !WICCLEAR ), .Z( wicclear_tmp ) );
  //assign wic_pend_we  = opt_mst_wic_en & (WICLOAD | wic_active) & (|WICINT);
  assign wic_pend_we  = opt_mst_wic_en & (WICLOAD | wic_active) ; //20210731 atc1010 update
  //wire wic_pend_clr_n = RESETn & !WICCLEAR;
  wire wic_pend_clr_n = RESETn & wicclear_tmp; //20220629 atc1010 just add dtc buf;


  wire  [WIC_LINES-1:0] wic_pend_clk;
  wire  [WIC_LINES-1:0] wic_pend_clk_mux;
  wire                  wic_pend_clr_n_mux;
  dtc_ckmux2 dtc_wic_pend_rst (.S(TESTMODE), .I0(wic_pend_clr_n), .I1(RESETn), .Z(wic_pend_clr_n_mux));

  genvar i;
  generate
     for(i=0;i<WIC_LINES;i=i+1)
     begin:wic_pend_int

       dtc_ckbuf  dtc_ckbuf_int( .I(wic_active & nxt_wic_pend[i]), .Z(wic_pend_clk[i]) );
       dtc_ckmux2 dtc_wic_pend_mux (.S(TESTMODE), .I0(wic_pend_clk[i]), .I1(SCANCLK), .Z(wic_pend_clk_mux[i]));
       always @(posedge wic_pend_clk_mux[i] or negedge wic_pend_clr_n_mux)
       if (!wic_pend_clr_n_mux)
       //wic_pend[i] <= {WIC_LINES{1'b0}};
         wic_pend[i] <= 1'b0;
       else if (wic_pend_we)
       //wic_pend[i] <= nxt_wic_pend[i];       
         wic_pend[i] <= 1'b1;
     end
  endgenerate
  //----------------------------------------------------------------------------
  // Active status register
  //----------------------------------------------------------------------------

  assign wic_active_we = opt_mst_wic_en & (WICLOAD | WICCLEAR);

  always @ (posedge FCLK or negedge RESETn)
    if (!RESETn)
      wic_active <= 1'b0;
    else if (wic_active_we)
      wic_active <= WICLOAD;

  //----------------------------------------------------------------------------
  // Request wake-up
  //----------------------------------------------------------------------------

  // Wake up if something has been pended that is observed whilst powered down
  assign wakeup_pmu = (|(wic_pend & wic_sense));

  //----------------------------------------------------------------------------
  // Output assignments
  //----------------------------------------------------------------------------
  assign WAKEUP    = opt_wakeup;
  assign WICSENSE  = opt_wic_sense;
  assign WICPEND   = opt_wic_pend;
  assign WICDSREQn = opt_wic_ds_req_n;
  assign WICENACK  = opt_wic_en_ack;


  //----------------------------------------------------------------------------
  // OVL assertions
  //----------------------------------------------------------------------------

`ifdef ARM_ASSERT_ON
  assert_never_unknown #(0,1,0, "wic_ds_req_we must not be unknown")
  ts_wic_ds_req_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_ds_req_we));

  assert_never_unknown #(0,1,0, "wic_sense_we must not be unknown")
  ts_wic_sense_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_sense_we));

  assert_never_unknown #(0,1,0, "wic_pend_we must not be unknown")
  ts_wic_pend_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_pend_we));

  assert_never_unknown #(0,1,0, "wic_active_we must not be unknown")
  ts_wic_active_we_unknown
    (.clk       (FCLK),
     .reset_n   (RESETn),
     .qualifier (1'b1),
     .test_expr (wic_active_we));

`endif
endmodule

```

{% asset_img image-20240723104805892.png %}

> 项目中是如上图这么做的
>
> <font color=red>有个问题，clock后面需要弄一个clock mux，用于切换scan和function clock？为什么这么做？</font>



### WIC的工作流程

1、当进入睡眠模式的时候，wakeup event mask将会从NVIC--》WIC，通过(WICMASK[] and WICLOAD)。

2、当唤醒事件到来的时候，WIC将会发送wake up request到PMU

3、PMU控制处理器的power和clock恢复；此时处理器可以接收中断请求（或其他唤醒事件）；

4、当处理器被唤醒之后，WIC的wake-up mask信息以及pending wake-up event将会被自动清除(WICCLEAR)  

{% asset_img image-20240723111728699.png %}

### WIC的详细时序图

{% asset_img image-20240730094308493.png %}



### WIC的集成

Cortex-M处理器，部分是将WIC放在处理器内部，有些是将WIC放在处理器外部；

放在外部的WIC，集成上略有不同：

{% asset_img image-20240723111937842.png %}



### EDBGRQ信号

Armv7-M 和 Armv8-M Mainline 处理器系统中，会将EDBGRQ signal (external debug request)  作为WIC的唤醒源之一，因为外部debug请求会触发Debug Monitor exception；

但是在Armv6-M 和 Armv8-M baseline 系统（如 Cortex-M23 处理器系统）中，因为没有Debug Monitor exception，因此就没有外部debug请求；

> 思考：
>
> 1、什么是Mainline？什么是baseline？
>
> 在Arm架构中，术语“Mainline”和“baseline”通常用来区分不同的处理器配置或特性水平。这些术语可以根据具体的处理器系列和架构版本来理解：
>
> 1. Mainline：
>    - 在Arm架构中，“Mainline”通常指的是支持全套标准特性和功能的处理器配置。这些处理器通常具有较高的性能和更广泛的功能集，能够满足多种应用需求。例如，在Cortex-M系列中，Mainline处理器通常支持较多的调试和安全功能，如调试监视器异常和硬件安全扩展。
> 2. Baseline：
>    - 相比之下，“Baseline”处理器配置则是指针对低成本、低功耗以及对特定应用需求的优化。这些处理器可能会限制一些高级特性或功能，以降低成本和功耗，同时仍提供足够的性能来处理典型的嵌入式应用。在Arm的术语中，Baseline配置可能会限制或不支持一些高级的调试功能或安全扩展。
>
> <font color=red>2、debug monitor异常？</font>



### WIC可以通过handshake信号enable或者disable

在M3和M4系统中，其handshake信号有两对：

{% asset_img image-20240724150409817.png %}

在程序开始的时候，软件去写PMU模块，让其和WIC进行handshake，使能WIC；这也使能了PMU去处理state retention  power gating；

WIC的enable 顺序如下：

{% asset_img image-20240724151628321.png %}

> 思考：
>
> <font color=red>PMU什么时候去处理power gating？怎么去处理？</font>

### SRPG support端口

如果使用SRPG（State Retention Power Gating  ），需要去控制一系列的信号，并且需要看特定的工艺（cell的控制端口），但是通常来说，需要有如下信号：

{% asset_img image-20240724152606630.png %}

### Demo PMU state machine

系统设计者需要通过state machine去控制power-down和power-up的状态，并且需要一个表示是否上电完成的信号PWRUPREADY（模拟LDO给数字供电需要一个过程），大概的流程如下所示：

{% asset_img image-20240724153806629.png %}

> 假设当前处于POWER-DOWN的状态：
>
> 1、当WIC检测到唤醒事件的时候，会有WAKEUP信号送到PMU；
>
> 2、PMU不在对状态保持逻辑做power gating，并且等到上电完成PWRUPREADY；
>
> 3、恢复状态保持寄存器的值；
>
> 4、关闭ISO单元；
>
> 5、打开时钟，关闭睡眠保持请求；
>
> 6、上电完成；



还可以进一步设计，允许power down顺序被中断，从而减少中断响应的时间：

{% asset_img image-20240724154409712.png %}



> <font color=red>启用WIC的软件流程是什么？</font>



## SRPG’s impact on software

SRPG对软件有一定的影响，需要考虑如下方面：

1、systick timer将会被关闭。此时如果还需要定时操作的话，需要使用外部时钟；

2、中断延时将会增加，当WIC或者SRPG使用的时候；

3、当有外部调试器连接的时候，通常禁用掉电功能；这是因为在处理器内核处于睡眠模式的时候，外部的调试器也需要访问处理器；





## Software power-saving approach  

软件开发的过程中，需要对如下场景做决策：

- 程序快速运行，并尽可能的进入低功耗模式;

- 程序慢速运行；

对于这个问题的分析，并没有一个固定的准则，原因如下：

1、如果振荡器的功耗很高，那么慢速时钟可能是减低功耗的方法，但是这会增加中断延时以及整体的漏电流；

2、如果eflash的漏电流很大，在程序快速运行然后进入低功耗模式的时候，可能是降低功耗的一种方法，但是其peak power会更高；



# Cortex-M processor characteristics that enable low-power designs



## High code density

Cortex-M系列的处理器使用的是16-bits/32bits混合指令集，其代码密度更高，在低功耗方面带来的好处就是，可以使用更小的ROM/Flash，从而降低功耗；

除了功耗方面的好处之外，更小的ROM/eflash可以降低成本；



## Short pipeline

除了Cortex-M7之外的大多数Cortex-M系列处理器，其流水线大多数都很短（2-3级），这样的处理器其好处就是（假设不考虑分支预测逻辑）：

- 具有较低的分支代价branch penalty；
- 减少了分支阴影branch shadow；

{% asset_img image-20240724173005520.png %}

> 分支惩罚是指当分支条件不满足时，处理器需要清空流水线并重新开始执行的时间成本
>
> 分支阴影（Branch shadows）指的是在处理器执行分支指令后，预先获取的指令
>
> 如图所示，Cortex-M0+ 和 Cortex-M23  处理器中，流水线只有2级，分支阴影会被降到只有一个word大小；
>
> 分支阴影的size可能和流水线的级数有关； 

## Instruction fetch optimizations

虽然有些指令是16位，但是Cortex-M处理器在大多数情况下以32bits的方式获取指令。这意味着获取每条指令的时候，Cortex-M处理器最多可以获取两条指令，并且指令fetch接口可以在不需要时处于非活动状态，这样可以降低程序存储器访问的功耗；

{% asset_img image-20240724194003374.png %}





# System-level design considerations  

## Low-power designs overview

低功耗本身是一个非常大的话题。除了利用CORE本身的睡眠模式之外，芯片的其他部分都会对低功耗产生影响。例如，可以用clock gating技术，此外如果某些外设不用的情况下，还可以关闭电源。



## Clock sources  

<font color=blue>有一个低功耗的时钟源非常重要</font>。很多设计都需要一个持续工作的32KHz常开时钟源，如果这个时钟源的功耗很高，那么将会对系统功耗产生很严重的影响。理想情况下，32KHz需要超低功耗的特性，在工作电压变化很大的情况下也能正常工作。

<font color=blue>时钟晶振的工作范围选择也很重要</font>，如果微处理器产品运行在100MHz，如果使用一个100Mhz的晶振，它意味着会以100Mhz running all the time因此会消耗大量的power。一次通常会使用一个低频率的时钟晶振+PLL去产生高频率的时钟源。



## Low-power memories

- memory macros 在设计中考虑了多种睡眠/保持模式，并提供了各种`“钩子”（hooks）`来允许系统设计者<font color=blue>将内存的低功耗状态与系统的睡眠模式进行关联。但是需要注意睡眠模式功耗和唤醒延迟之间存在的权衡。</font>

- eflash 可以选择关闭电源，因为其数据不会被丢失。<font color=blue>但是需要注意当闪存重新上电时，会产生一个瞬时的电流峰值，这被称为 in-rush current 或 current spike；这个电流峰值可能会导致电源线上的电压瞬时下降；电压下降可能会导致芯片中其他电路或模块出现工作不正常的问题，例如可能导致微处理器（或其他处理器）在重新启动时失去部分电源；</font>
- 某些处理器允许软件在掉电之前向闪存写入数据，目的是让SRAM中的关键数据在上电会被恢复。当替换电池的时候，可能需要这样的操作。

## cache

系统中加入cache，虽然会增加芯片面积，漏电流以及对电源的需求，但是在一些情况下

- 可以降低功耗，因为避免频繁访问eflash，访问flash的开销是非常巨大的；
- 可以增加系统的整体性能，因为访问flash的速度是很慢的；

## Low-power analog components

在睡眠模式的时候，有很多的模拟组件是开着的：

- 32kHz振荡器：这是一种低频振荡器，通常用于实时时钟（RTC，Real-Time Clock）和定时器。它的低频率特性使其在功耗上比高频率振荡器更为节能，适合在系统的低功耗模式（如睡眠模式）下运行。

- 实时时钟（RTC）：RTC是嵌入式系统中用于跟踪日期和时间的设备或模块。即使系统进入睡眠模式，RTC通常也需要保持运行，以便能够在系统唤醒时继续提供准确的时间信息。

- 低电压检测器（brown-out detector）：这是一种检测电源电压是否低于安全工作范围的电路。它通常会在电源电压低于规定值时发出警告或者复位系统，以防止芯片和周边电路因电压不稳定而受损。

- 部分I/O引脚：在睡眠模式下，有些输入/输出引脚可能需要保持活动状态，特别是当它们用于外部中断检测时。这意味着即使系统处于低功耗模式，这些引脚需要能够接收并响应来自外部的信号变化。



很多I/O引脚具有可配置的功耗模式，可以通过调整驱动强度和翻转速度来降低功耗。系统设计人员可以通过引入寄存器来控制这些配置信号；

## Maximizing clock gating opportunities  

在系统设计中，也可以通过clock gating增加时钟门控单元以降低动态功耗。ARM的Corstone Foundation IP / Cortex-M System Design Kit   中提供的AHB to APB bridge  就有一个`APBACTIVE `信号，当bridge上没有transaction时，就可以利用此信号去做门控，关闭APB外设时钟；

{% asset_img image-20240725171927479.png %}



## Sleep mode that completely powers down the processor  

Cortex-M处理器允许完全掉电，并且能被某些时间唤醒，但是需要注意如下两点：

1、处理器的状态会丢失，因此软件在执行掉电之前必须将处理器中关键的数据保存的状态保持SRAM中；

2、系统设计需要额外的硬件逻辑去处理硬件唤醒时间；

硬件逻辑需要完成如下任务：

1、向PMU发送唤醒信号，恢复处理器的供电；

2、复位处理器系统，（不对状态保持memory和寄存器进行复位）；

3、释放reset，让处理器boot-up并执行软件程序；

这样的一个系统，其框架大致如图所示：

{% asset_img image-20240725174130100.png %}

1、电源控制，允许软件主动选择进入哪一种睡眠模式（例如是否在低功耗的时候掉电）

2、复位信息寄存器，允许软件决定是否进入冷启动或者暖启动；

> 冷启动（Cold Boot）和暖启动（Warm Boot）是两种不同的系统启动方式，它们的主要区别在于系统在启动过程中是否经历了完全的电源断电。
>
> - 冷启动指的是系统在完全断电后重新上电启动
>
> - 暖启动是指系统在不断电的情况下重新启动

3、状态保持SRAM，用于保存关键数据；

4、事件唤醒检测；



<font color=blue>由于唤醒的过程需要时间，因此必须保存wake-up event ，以便软件有时间启动NIVC，然后由NVIC处理；</font>

DAP(debug access port)有两种使用方法：

1、将DAP放在always-on power-domain，允许调试器连接的时候去唤醒处理器；

2、先使用其他的方式去唤醒，然后再连接调试器；



这种掉电的方式，其缺点是处理器必须先boot-up，然后才能处理中断。可以通过将处理器关键的状态保存在retention 的sram中，这种方式，就可以跳过C startup就可以及时响应中断。一般来说，关键的信息包括如下内容：

- NVIC的配置
- MPU的配置
- ststick的配置
- MPS和PSP的配置
- 特殊寄存器的配置，PRIMASK, FAULTMASK, BASEPRI, etc.  
- FPU的配置



> <font color=red>软件流程上具体是如何做的？？</font>











































