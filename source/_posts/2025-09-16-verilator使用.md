---
title: verilator使用
date: 2025-09-16 01:02:13
categories:
- 一生一芯计划
- E阶段
tags:
---







# Verilator demo示例

## make_hello_binary

### Makefile文件

```makefile

# Check for sanity to avoid later confusion

ifneq ($(words $(CURDIR)),1)
 $(error Unsupported: GNU Make cannot build in directories containing spaces, build elsewhere: '$(CURDIR)')
endif

```

>`$(CURDIR)`：当前工作目录。
>
>`$(words $(CURDIR))`：统计目录路径里有几个“单词”，空格会被分隔成多个单词。
>
>如果目录路径里包含空格，就报错并退出。
> 👉 作用：**防止在带空格路径下构建**，否则后续命令可能解析错误。



```makefile
######################################################################

# This is intended to be a minimal example.  Before copying this to start a
# real project, it is better to start with a more complete example,
# e.g. examples/make_tracing_c.

# If $VERILATOR_ROOT isn't in the environment, we assume it is part of a
# package install, and verilator is in your path. Otherwise find the
# binary relative to $VERILATOR_ROOT (such as when inside the git sources).
ifeq ($(VERILATOR_ROOT),)
VERILATOR = verilator
else
export VERILATOR_ROOT
VERILATOR = $(VERILATOR_ROOT)/bin/verilator
endif

```

> ifeq (arg1, arg2)
>    ...   # 当 arg1 == arg2 时执行
> endif
>
> ifeq ($(VERILATOR_ROOT),)
>
> 如果环境变量 `VERILATOR_ROOT` 没设置,即为空：（默认 `verilator` 在系统路径 (`PATH`) 里），直接调用
>
> 如果设置了 `VERILATOR_ROOT`：使用它指定的目录中的 `bin/verilator`

```makefile

default:
	@echo "-- Verilator hello-world simple binary example"
	@echo "-- VERILATE & BUILD --------"
	$(VERILATOR) --binary -j 0 top.v
	@echo "-- RUN ---------------------"
	obj_dir/Vtop
	@echo "-- DONE --------------------"
	@echo "Note: Once this example is understood, see examples/make_hello_c."
	@echo "Note: See also https://verilator.org/guide/latest/examples.html"

```

> `default:` 定义默认目标（运行 `make` 就执行这里）。
>
> `@echo ...`：打印提示信息（`@` 表示不打印命令本身，只显示输出）。
>
> `--binary`：自动生成 C++ 仿真框架并编译成二进制
>
> `-j 0`：多线程编译，`0` 表示自动使用所有 CPU 核心

```makefile
######################################################################

maintainer-copy::
clean mostlyclean distclean maintainer-clean::
	-rm -rf obj_dir *.log *.dmp *.vpd core
```

> 这一段的作用就是：
>
> - 定义了常见的 **清理目标** (`clean`、`distclean` 等)。
> - 无论你输入 `make clean` 还是 `make distclean`，都会执行 **同样的删除命令**。
> - 这样做是模仿 GNU 工程里的习惯：
>   - `clean`：删除编译生成的临时文件。
>   - `distclean`：比 `clean` 更彻底，适合打包发布。
>   - `maintainer-clean`：给维护者准备的，通常最彻底

## make_hello_c

### Makefile文件

```makefile
default:
	@echo "-- Verilator hello-world simple example"
	@echo "-- VERILATE & BUILD --------"
	$(VERILATOR) -cc --exe --build -j top.v sim_main.cpp
	@echo "-- RUN ---------------------"
	obj_dir/Vtop
	@echo "-- DONE --------------------"
	@echo "Note: Once this example is understood, see examples/make_tracing_c."
	@echo "Note: See also https://verilator.org/guide/latest/examples.html"

```

>`--exe `：
>
>- 告诉 Verilator：
>   “我要<font color=blue>生成一个可以编译成 **可执行文件** 的工程</font>，请把 `sim_main.cpp` 一起加进去”。
>- 结果：Verilator 会在 `obj_dir/` 下<font color=blue>生成一个 **Makefile**，比如 `obj_dir/Vtop.mk`，</font>这个 Makefile 知道怎么编译 `Vtop.cpp + sim_main.cpp`。
>
>`--build`：
>
>- 在生成 `obj_dir/Vtop.mk` 之后，直接调用 `make -C obj_dir -f Vtop.mk` 来编译。
>
>- 最终结果：得到真正的二进制可执行文件 `obj_dir/Vtop`。



### Vtop_classes.mk

```makefile
# Verilated -*- Makefile -*-
# DESCRIPTION: Verilator output: Make include file with class lists
#
# This file lists generated Verilated files, for including in higher level makefiles.
# See Vtop.mk for the caller.

### Switches...
# C11 constructs required?  0/1 (always on now)
VM_C11 = 1
# Timing enabled?  0/1
VM_TIMING = 0
# Coverage output mode?  0/1 (from --coverage)
VM_COVERAGE = 0
# Parallel builds?  0/1 (from --output-split)
VM_PARALLEL_BUILDS = 0
# Tracing output mode?  0/1 (from --trace/--trace-fst)
VM_TRACE = 0
# Tracing output mode in VCD format?  0/1 (from --trace)
VM_TRACE_VCD = 0
# Tracing output mode in FST format?  0/1 (from --trace-fst)
VM_TRACE_FST = 0

### Object file lists...
# Generated module classes, fast-path, compile with highest optimization
VM_CLASSES_FAST += \
	Vtop \
	Vtop___024root__DepSet_h84412442__0 \
	Vtop___024root__DepSet_heccd7ead__0 \

# Generated module classes, non-fast-path, compile with low/medium optimization
VM_CLASSES_SLOW += \
	Vtop___024root__Slow \
	Vtop___024root__DepSet_heccd7ead__0__Slow \

# Generated support classes, fast-path, compile with highest optimization
VM_SUPPORT_FAST += \

# Generated support classes, non-fast-path, compile with low/medium optimization
VM_SUPPORT_SLOW += \
	Vtop__Syms \

# Global classes, need linked once per executable, fast-path, compile with highest optimization
VM_GLOBAL_FAST += \
	verilated \
	verilated_threads \

# Global classes, need linked once per executable, non-fast-path, compile with low/medium optimization
VM_GLOBAL_SLOW += \


# Verilated -*- Makefile -*-

```



### Vtop.mk

```makefile
# Verilated -*- Makefile -*-
# DESCRIPTION: Verilator output: Makefile for building Verilated archive or executable
#
# Execute this makefile from the object directory:
#    make -f Vtop.mk

default: Vtop

### Constants...
# Perl executable (from $PERL)
PERL = perl
# Path to Verilator kit (from $VERILATOR_ROOT)
VERILATOR_ROOT = /home/zj/ic_workspace/verilator/verilator
# SystemC include directory with systemc.h (from $SYSTEMC_INCLUDE)
SYSTEMC_INCLUDE ?= 
# SystemC library directory with libsystemc.a (from $SYSTEMC_LIBDIR)
SYSTEMC_LIBDIR ?= 

### Switches...
# C++ code coverage  0/1 (from --prof-c)
VM_PROFC = 0
# SystemC output mode?  0/1 (from --sc)
VM_SC = 0
# Legacy or SystemC output mode?  0/1 (from --sc)
VM_SP_OR_SC = $(VM_SC)
# Deprecated
VM_PCLI = 1
# Deprecated: SystemC architecture to find link library path (from $SYSTEMC_ARCH)
VM_SC_TARGET_ARCH = linux

### Vars...
# Design prefix (from --prefix)
VM_PREFIX = Vtop
# Module prefix (from --prefix)
VM_MODPREFIX = Vtop
# User CFLAGS (from -CFLAGS on Verilator command line)
VM_USER_CFLAGS = \

# User LDLIBS (from -LDFLAGS on Verilator command line)
VM_USER_LDLIBS = \

# User .cpp files (from .cpp's on Verilator command line)
VM_USER_CLASSES = \
	sim_main \

# User .cpp directories (from .cpp's on Verilator command line)
VM_USER_DIR = \
	. \


### Default rules...
# Include list of all generated classes
include Vtop_classes.mk
# Include global rules
include $(VERILATOR_ROOT)/include/verilated.mk

### Executable rules... (from --exe)
VPATH += $(VM_USER_DIR)

sim_main.o: sim_main.cpp
	$(OBJCACHE) $(CXX) $(CXXFLAGS) $(CPPFLAGS) $(OPT_FAST) -c -o $@ $<

### Link rules... (from --exe)
Vtop: $(VK_USER_OBJS) $(VK_GLOBAL_OBJS) $(VM_PREFIX)__ALL.a $(VM_HIER_LIBS)
	$(LINK) $(LDFLAGS) $^ $(LOADLIBES) $(LDLIBS) $(LIBS) $(SC_LIBS) -o $@


# Verilated -*- Makefile -*-

```



### verilated.mk

```makefile
# -*- Makefile -*-
######################################################################
# DESCRIPTION: Makefile commands for all verilated target files
#
# Copyright 2003-2023 by Wilson Snyder. This program is free software; you
# can redistribute it and/or modify it under the terms of either the GNU
# Lesser General Public License Version 3 or the Perl Artistic License
# Version 2.0.
# SPDX-License-Identifier: LGPL-3.0-only OR Artistic-2.0
######################################################################

AR = ar
CXX = g++
LINK = g++
OBJCACHE ?= ccache
PERL = /usr/bin/perl
PYTHON3 = /usr/bin/python3

CFG_WITH_CCWARN = no
CFG_WITH_LONGTESTS = no

# Compiler flags to enable profiling
CFG_CXXFLAGS_PROFILE =  -pg
# Select language required to compile (often empty)
CFG_CXXFLAGS_STD = 
# Select newest language (unused by this Makefile, for some test's Makefiles)
CFG_CXXFLAGS_STD_NEWEST = -std=gnu++17
# Compiler flags to use to turn off unused and generated code warnings, such as -Wno-div-by-zero
CFG_CXXFLAGS_NO_UNUSED =  -faligned-new -fcf-protection=none -Wno-bool-operation -Wno-sign-compare -Wno-uninitialized -Wno-unused-but-set-variable -Wno-unused-parameter -Wno-unused-variable -Wno-shadow
# Compiler flags that turn on extra warnings
CFG_CXXFLAGS_WEXTRA =  -Wextra -Wfloat-conversion -Wlogical-op
# Compiler flags that enable coroutine support
CFG_CXXFLAGS_COROUTINES = -fcoroutines
# Linker libraries for multithreading
CFG_LDLIBS_THREADS =  -pthread -lpthread -latomic

######################################################################
# Programs

VERILATOR_COVERAGE = $(PERL) $(VERILATOR_ROOT)/bin/verilator_coverage
VERILATOR_INCLUDER = $(PYTHON3) $(VERILATOR_ROOT)/bin/verilator_includer
VERILATOR_CCACHE_REPORT = $(PYTHON3) $(VERILATOR_ROOT)/bin/verilator_ccache_report

######################################################################
# Make checks

ifneq ($(words $(CURDIR)),1)
 $(error Unsupported: GNU Make cannot build in directories containing spaces, build elsewhere: '$(CURDIR)')
endif

######################################################################
# OS detection
UNAME_S := $(shell uname -s)

######################################################################
# C Preprocessor flags

# Add -MMD -MP if you're using a recent version of GCC.
VK_CPPFLAGS_ALWAYS += \
		-MMD \
		-I$(VERILATOR_ROOT)/include \
		-I$(VERILATOR_ROOT)/include/vltstd \
		-DVM_COVERAGE=$(VM_COVERAGE) \
		-DVM_SC=$(VM_SC) \
		-DVM_TRACE=$(VM_TRACE) \
		-DVM_TRACE_FST=$(VM_TRACE_FST) \
		-DVM_TRACE_VCD=$(VM_TRACE_VCD) \
		$(CFG_CXXFLAGS_NO_UNUSED) \

ifeq ($(CFG_WITH_CCWARN),yes)  # Local... Else don't burden users
VK_CPPFLAGS_WALL += -Wall $(CFG_CXXFLAGS_WEXTRA) -Werror
endif

CPPFLAGS += -I. $(VK_CPPFLAGS_WALL) $(VK_CPPFLAGS_ALWAYS)

VPATH += ..
VPATH += $(VERILATOR_ROOT)/include
VPATH += $(VERILATOR_ROOT)/include/vltstd

#OPT = -ggdb -DPRINTINITSTR -DDETECTCHANGE
#OPT = -ggdb -DPRINTINITSTR
CPPFLAGS += $(OPT)

CPPFLAGS += $(M32)
LDFLAGS  += $(M32)

# On macOS, specify all weak symbols as dynamic_lookup.
# Otherwise, you get undefined symbol errors.
ifeq ($(UNAME_S),Darwin)
	LDFLAGS += -Wl,-U,__Z15vl_time_stamp64v,-U,__Z13sc_time_stampv
endif

# Allow upper level user makefiles to specify flags they want.
# These aren't ever set by Verilator, so users are free to override them.
CPPFLAGS += $(USER_CPPFLAGS)
LDFLAGS  += $(USER_LDFLAGS)
LDLIBS   += $(USER_LDLIBS)

# Add flags from -CFLAGS and -LDFLAGS on Verilator command line
CPPFLAGS += $(VM_USER_CFLAGS)
LDFLAGS  += $(VM_USER_LDFLAGS)
LDLIBS   += $(VM_USER_LDLIBS)

######################################################################
# Optimization control.

# See also the BENCHMARKING & OPTIMIZATION section of the manual.

# Optimization flags for non performance-critical/rarely executed code.
# No optimization by default, which improves compilation speed.
OPT_SLOW =
# Optimization for performance critical/hot code. Most time is spent in these
# routines. Optimizing by default for improved execution speed.
OPT_FAST = -Os
# Optimization applied to the common run-time library used by verilated models.
# For compatibility this is called OPT_GLOBAL even though it only applies to
# files in the run-time library. Normally there should be no need for the user
# to change this as the library is small, but can have significant speed impact.
OPT_GLOBAL = -Os

#######################################################################
##### Profile builds

ifeq ($(VM_PROFC),1)
  CPPFLAGS += $(CFG_CXXFLAGS_PROFILE)
  LDFLAGS  += $(CFG_CXXFLAGS_PROFILE)
endif

#######################################################################
##### SystemC builds

ifeq ($(VM_SC),1)
  CPPFLAGS += $(SYSTEMC_CXX_FLAGS) $(addprefix -I, $(SYSTEMC_INCLUDE))
  LDFLAGS  += $(SYSTEMC_CXX_FLAGS) $(addprefix -L, $(SYSTEMC_LIBDIR))
  SC_LIBS   = -lsystemc
 ifneq ($(wildcard $(SYSTEMC_LIBDIR)/*numeric_bit*),)
  # Systemc 1.2.1beta
  SC_LIBS   += -lnumeric_bit -lqt
 endif
endif

#######################################################################
##### Threaded builds

CPPFLAGS += $(CFG_CXXFLAGS_STD)
LDLIBS += $(CFG_LDLIBS_THREADS)

ifneq ($(VM_TIMING),0)
 ifneq ($(VM_TIMING),)
  CPPFLAGS += $(CFG_CXXFLAGS_COROUTINES)
 endif
endif


#######################################################################
### Aggregates

VM_FAST += $(VM_CLASSES_FAST) $(VM_SUPPORT_FAST)
VM_SLOW += $(VM_CLASSES_SLOW) $(VM_SUPPORT_SLOW)

#######################################################################
### Overall Objects Linking

VK_FAST_OBJS = $(addsuffix .o, $(VM_FAST))
VK_SLOW_OBJS = $(addsuffix .o, $(VM_SLOW))

VK_USER_OBJS = $(addsuffix .o, $(VM_USER_CLASSES))

# Note VM_GLOBAL_FAST and VM_GLOBAL_SLOW holds the files required from the
# run-time library. In practice everything is actually in VM_GLOBAL_FAST,
# but keeping the distinction for compatibility for now.
VK_GLOBAL_OBJS = $(addsuffix .o, $(VM_GLOBAL_FAST) $(VM_GLOBAL_SLOW))

# Need to re-build if the generated makefile changes, as compiler options might
# have changed.
$(VK_GLOBAL_OBJS): $(VM_PREFIX).mk

ifneq ($(VM_PARALLEL_BUILDS),1)
  # Fast build for small designs: All .cpp files in one fell swoop. This
  # saves total compute, but can be slower if only a little changes. It is
  # also a lot slower for medium to large designs when the speed of the C
  # compiler dominates, which in this mode is not parallelizable.

  VK_OBJS += $(VM_PREFIX)__ALL.o
  $(VM_PREFIX)__ALL.cpp: $(addsuffix .cpp, $(VM_FAST) $(VM_SLOW))
	$(VERILATOR_INCLUDER) -DVL_INCLUDE_OPT=include $^ > $@
  all_cpp: $(VM_PREFIX)__ALL.cpp
else
  # Parallel build: Each .cpp file by itself. This can be somewhat slower for
  # very small designs and examples, but is a lot faster for large designs.

  VK_OBJS += $(VK_FAST_OBJS) $(VK_SLOW_OBJS)
endif

# When archiving just objects (.o), use single $(AR) run
#   1. Make .verilator_deplist.tmp file with list of objects so don't exceed
#      the command line limits when calling $(AR).
#      The approach to write the dependency file is compatible with GNU Make 3,
#      and can be simplified using the file function once GNU Make 4.x becomes
#      the minimum supported version.
# When merging objects (.o) and archives (.a) additionally:
#   1. Extract object files from .a
#   2. Create a new archive from extracted .o and given .o
%.a: | %.verilator_deplist.tmp
	$(info Archive $(AR) -rcs $@ $^)
	$(foreach L, $(filter-out %.a,$^), $(shell echo $L >>$@.verilator_deplist.tmp))
	@if test $(words $(filter %.a,$^)) -eq 0; then \
		$(RM) -f $@; \
		cat $@.verilator_deplist.tmp | xargs $(AR) -rc $@; \
		$(AR) -s $@; \
	else \
		$(RM) -rf $@.tmpdir; \
		for archive in $(filter %.a,$^); do \
			mkdir -p $@.tmpdir/$$(basename $${archive}); \
			cd $@.tmpdir/$$(basename $${archive}); \
			$(AR) -x ../../$${archive}; \
			cd ../..; \
		done; \
		$(RM) -f $@; \
		cat $@.verilator_deplist.tmp | xargs $(AR) -rc $@; \
		$(AR) -rcs $@ $@.tmpdir/*/*.o; \
	fi \
	; $(RM) -rf $@.verilator_deplist.tmp $@.tmpdir

# Truncate the dependency list file used in the %.a target above.
%.verilator_deplist.tmp:
	echo "" > $@

$(VM_PREFIX)__ALL.a: $(VK_OBJS) $(VM_HIER_LIBS)


######################################################################
### Compile rules

ifneq ($(VM_DEFAULT_RULES),0)
# Anything not in $(VK_SLOW_OBJS) or $(VK_GLOBAL_OBJS), including verilated.o
# and user files passed on the Verilator command line use this rule.
%.o: %.cpp
	$(OBJCACHE) $(CXX) $(CXXFLAGS) $(CPPFLAGS) $(OPT_FAST) -c -o $@ $<

$(VK_SLOW_OBJS): %.o: %.cpp
	$(OBJCACHE) $(CXX) $(CXXFLAGS) $(CPPFLAGS) $(OPT_SLOW) -c -o $@ $<

$(VK_GLOBAL_OBJS): %.o: %.cpp
	$(OBJCACHE) $(CXX) $(CXXFLAGS) $(CPPFLAGS) $(OPT_GLOBAL) -c -o $@ $<
endif

#Default rule embedded in make:
#.cpp.o:
#	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c -o $@ $<

######################################################################
### ccache report

ifneq ($(findstring ccache-report,$(MAKECMDGOALS)),)
  ifneq ($(OBJCACHE),ccache)
    $(error ccache-report requires OBJCACHE to equal 'ccache')
  endif
  VK_OTHER_GOALS := $(strip $(subst ccache-report,,$(MAKECMDGOALS)))
  ifeq ($(VK_OTHER_GOALS),)
    $(error ccache-report must be used with at least one other explicit target)
  endif

  # Report ccache behaviour for this invocation of make
  VK_CCACHE_LOGDIR := ccache-logs
  VK_CCACHE_REPORT := $(VM_PREFIX)__ccache_report.txt
  # Remove previous logfiles and report
  $(shell rm -rf $(VK_CCACHE_LOGDIR) $(VK_CCACHE_REPORT))

$(VK_CCACHE_LOGDIR):
	mkdir -p $@

$(VK_OBJS): | $(VK_CCACHE_LOGDIR)

$(VK_OBJS): export CCACHE_LOGFILE=$(VK_CCACHE_LOGDIR)/$@.log

$(VK_CCACHE_REPORT): $(VK_OBJS)
	$(VERILATOR_CCACHE_REPORT) -o $@ $(VK_CCACHE_LOGDIR)

.PHONY: ccache-report
ccache-report: $(VK_CCACHE_REPORT)
	@cat $<

# ccache-report runs last
ccache-report: $(VK_OTHER_GOALS)
endif

######################################################################
### Debugging

debug-make::
	@echo
	@echo CXXFLAGS: $(CXXFLAGS)
	@echo CPPFLAGS: $(CPPFLAGS)
	@echo OPT_FAST: $(OPT_FAST)
	@echo OPT_SLOW: $(OPT_SLOW)
	@echo VM_PREFIX:  $(VM_PREFIX)
	@echo VM_PARALLEL_BUILDS:  $(VM_PARALLEL_BUILDS)
	@echo VM_CLASSES_FAST: $(VM_CLASSES_FAST)
	@echo VM_CLASSES_SLOW: $(VM_CLASSES_SLOW)
	@echo VM_SUPPORT_FAST: $(VM_SUPPORT_FAST)
	@echo VM_SUPPORT_SLOW: $(VM_SUPPORT_SLOW)
	@echo VM_GLOBAL_FAST: $(VM_GLOBAL_FAST)
	@echo VM_GLOBAL_SLOW: $(VM_GLOBAL_SLOW)
	@echo VK_OBJS: $(VK_OBJS)
	@echo

######################################################################
### Detect out of date files and rebuild.

DEPS := $(wildcard *.d)
ifneq ($(DEPS),)
include $(DEPS)
endif

```





### 仿真文件sim_main.cpp

```c++
// verilated.h：包含一些通用函数和类，比如时间管理、命令行参数解析
#include <verilated.h>

// Vtop.h：由 Verilator 根据你的 Verilog 顶层模块 module top(...) 自动生成的 C++ 类
#include "Vtop.h"

int main(int argc, char** argv) {

// VerilatedContext：保存仿真环境的状态（比如当前时间、是否收到 $finish 信号）
    VerilatedContext* contextp = new VerilatedContext;

// commandArgs(...)：把命令行参数传给 Verilated 模型，以便在 Verilog 里用 $value$plusargs 获取
    contextp->commandArgs(argc, argv);
/*
上面两句可以理解为：搭建一个仿真环境的控制器
*/

// 相当于 把 Verilog 模块 top 翻译成一个 C++ 对象，现在你可以调用它的 eval() 来执行仿真
    Vtop* top = new Vtop{contextp};

// contextp->gotFinish()：检测 Verilog 仿真里是否调用了 $finish
    while (!contextp->gotFinish()) {

// 执行一次设计的计算（就像时钟推进或信号变化后更新电路状态）
        top->eval();
    }

// 模型收尾，比如执行 Verilog `final` 块（如果有）
    top->final();

    // Destroy model
    delete top;

    // Return good completion status
    return 0;
}

```

> 
>
> 注释1：
>
> 在 **Verilator** 里，生成的 C++ 顶层类名字是根据 **Verilog 顶层模块名** 来的
>
> 如果你的 Verilog 顶层模块是`top`;Verilator 会生成一个 C++ 类,`Vtop`;
>
> 假设顶层模块是 `top`，你会在 `obj_dir/` 里看到类似文件：
>
> - `Vtop.h` —— 类的声明（接口）。
> - `Vtop.cpp` —— 类的实现。
> - `Vtop__Syms.h / .cpp` —— 内部符号表（信号、子模块）。
> - `Vtop.mk` —— 用来编译的 Makefile 片段
>
> 
>
> 注释2：
>
>  `$value$plusargs` 是什么？怎么用？
>
> - `$value$plusargs` 是 Verilog/SystemVerilog 的 **系统函数**，用于从命令行读取参数。
> - 它常和 **Verilator 的 `commandArgs(argc, argv)`** 配合使用
>
> ```verilog
> module top;
>     integer cycles;
>     initial begin
>         if ($value$plusargs("cycles=%d", cycles)) begin
>             $display("Simulation cycles = %0d", cycles);
>         end else begin
>             cycles = 10; // 默认值
>         end
>     end
> endmodule
> ```
>
> ```shell
> // shell
> ./Vtop +cycles=100
> 
> // 输出
> Simulation cycles = 100
> ```
>
> 注释3：
>
> 概念
>
> - `final` 块是 **SystemVerilog** 引入的一种过程块。
> - 它在 **仿真结束时（遇到 `$finish`）** 执行一次，用来做收尾工作。
> - 类似 C++/Java 的析构函数。
>
> ```verilog
> module top;
>     integer count = 0;
> 
>     always #10 count++;
> 
>     initial begin
>         #100 $finish;   // 100 时间单位后结束仿真
>     end
> 
>     final begin
>         $display("Final count = %0d at time %0t", count, $time);
>     end
> endmodule
> 
> ```



## make_tracing_c



### makefile

```makefile
ifeq ($(VERILATOR_ROOT),)
VERILATOR = verilator
VERILATOR_COVERAGE = verilator_coverage
else
export VERILATOR_ROOT
VERILATOR = $(VERILATOR_ROOT)/bin/verilator
VERILATOR_COVERAGE = $(VERILATOR_ROOT)/bin/verilator_coverage
endif

# Generate C++ in executable form
VERILATOR_FLAGS += -cc --exe
# Generate makefile dependencies (not shown as complicates the Makefile)
#VERILATOR_FLAGS += -MMD
# Optimize，将所有显式的X态转化为0（此种方式仿真速度最快）
VERILATOR_FLAGS += -x-assign fast
# Warn abount lint issues; may not want this on less solid designs
VERILATOR_FLAGS += -Wall
# Make waveforms
VERILATOR_FLAGS += --trace
# Check SystemVerilog assertions
VERILATOR_FLAGS += --assert
# Generate coverage analysis
VERILATOR_FLAGS += --coverage
# Input files for Verilator
VERILATOR_INPUT = -f input.vc top.v sim_main.cpp

######################################################################
default: run

run:
	@echo
	@echo "-- Verilator tracing example"

	@echo
	@echo "-- VERILATE ----------------"
	$(VERILATOR) $(VERILATOR_FLAGS) $(VERILATOR_INPUT)

	@echo
	@echo "-- BUILD -------------------"
# To compile, we can either
# 1. Pass --build to Verilator by editing VERILATOR_FLAGS above.
# 2. Or, run the make rules Verilator does:
#	$(MAKE) -j -C obj_dir -f Vtop.mk
# 3. Or, call a submakefile where we can override the rules ourselves:
	$(MAKE) -j -C obj_dir -f ../Makefile_obj

	@echo
	@echo "-- RUN ---------------------"
	@rm -rf logs
	@mkdir -p logs
	obj_dir/Vtop +trace

	@echo
	@echo "-- COVERAGE ----------------"
	@rm -rf logs/annotated
	$(VERILATOR_COVERAGE) --annotate logs/annotated logs/coverage.dat

	@echo
	@echo "-- DONE --------------------"
	@echo "To see waveforms, open vlt_dump.vcd in a waveform viewer"
	@echo


######################################################################
# Other targets

show-config:
	$(VERILATOR) -V

maintainer-copy::
clean mostlyclean distclean maintainer-clean::
	-rm -rf obj_dir logs *.log *.dmp *.vpd coverage.dat core
```

> 1. `$(MAKE)`
>
> - 这是 GNU Make 内置变量，代表 **当前正在执行的 make 程序**（比如 `/usr/bin/make`）。
>
> 2. `-j`
>
> - 表示 **并行编译**。
> - `-j` 后面可以加数字，例如 `-j4` 表示最多同时跑 4 个任务。
> - 这里只写了 `-j`，相当于“尽可能多并行”，由系统决定。
>
> 3.  `-C obj_dir`
>
> - 表示 **切换目录**，进入 `obj_dir` 目录后再执行 make
>
> 4. `-f ../Makefile_obj`
>
> - `-f` 指定要用的 Makefile 文件
>
> 因此作用是：
>
> 在 `obj_dir` 目录下，调用 make，并行编译，使用 `../Makefile_obj` 作为规则文件





### input.vc

```shell
// This file typically lists flags required by a large project, e.g. include directories
+librescan +libext+.v+.sv+.vh+.svh -y .
```



> 这是几条和 **文件搜索路径 / 库解析** 有关的选项。
>
>  (1) `+librescan`
>
> - 意思是：**允许重复搜索库文件**。
> - 默认情况下，Verilog 编译器（包括 Verilator）在 `-y` 指定的目录下找到一个匹配的模块文件后就停止，不再继续找同名模块。
> - 加上 `+librescan`，会继续扫描目录中剩下的文件，避免因为库里有多个定义而漏掉。
>    👉 一般用于 **大 IP 库目录**，确保所有依赖模块都能找到。
>
> ------
>
>  (2) `+libext+.v+.sv+.vh+.svh`
>
> - 指定 **允许的文件扩展名**。
> - `+libext+.v+.sv+.vh+.svh` 表示依次尝试：
>   - `.v` (Verilog)
>   - `.sv` (SystemVerilog)
>   - `.vh` (Verilog header)
>   - `.svh` (SystemVerilog header)
> - 当在 `-y` 指定的目录中搜索某个模块时，编译器会自动尝试这些扩展名。
>    👉 这样就不用手动写出每个文件名，模块引用时只要写 `module_name`，工具会自动补齐后缀。
>
> ------
>
>  (3) `-y .`
>
> - 指定库搜索路径，`.` 表示`当前目录`。
> - 当 Verilator 找不到某个 `module xxx` 的定义时，就会去 `.` 目录下扫描所有符合 `+libext` 的文件。
>    👉 常用来放 IP 库或者模块集合。
>
> <font color=blue>最终形成的效果：</font>
>
> 综合起来，`input.vc` 的意思是：
>
> 1. 当仿真编译时，如果遇到未定义的模块，Verilator 会去 **当前目录 (`.`)** 搜索。
> 2. 搜索时，会尝试 **.v / .sv / .vh / .svh** 这些扩展名。
> 3. 即使找到一个匹配文件，也会继续扫描，避免漏掉别的依赖。



### Makefile_obj

```makefile

default: Vtop

# Include the rules made by Verilator
include Vtop.mk

# Use OBJCACHE (ccache) if using gmake and its installed
COMPILE.cc = $(OBJCACHE) $(CXX) $(CXXFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c

#######################################################################
# Compile flags

# Turn on creating .d make dependency files
CPPFLAGS += -MMD -MP

# Compile in Verilator runtime debugging, so +verilator+debug works
CPPFLAGS += -DVL_DEBUG=1

# Turn on some more compiler lint flags (when configured appropriately)
# For testing inside Verilator, "configure --enable-ccwarn" will do this
# automatically; otherwise you may want this unconditionally enabled
ifeq ($(CFG_WITH_CCWARN),yes)  # Local... Else don't burden users
USER_CPPFLAGS_WALL += -W -Werror -Wall
endif

# See the benchmarking section of bin/verilator.
# Support class optimizations.  This includes the tracing and symbol table.
# SystemC takes minutes to optimize, thus it is off by default.
OPT_SLOW =

# Fast path optimizations.  Most time is spent in these classes.
OPT_FAST = -Os -fstrict-aliasing
#OPT_FAST = -O
#OPT_FAST =

######################################################################
######################################################################
# Automatically understand dependencies

DEPS := $(wildcard *.d)
ifneq ($(DEPS),)
include $(DEPS)
endif

```

> `Makefile_obj` 的作用是 **给 Verilator 生成的核心规则 (`Vtop.mk`) 提供一个编译环境**。
>
> 核心逻辑在 `Vtop.mk`，而 `Makefile_obj` 只是：
>
> 1. 设置默认目标 (`Vtop`)；
> 2. 定义编译命令和依赖生成规则；
> 3. 启用调试宏和优化选项；
> 4. 自动处理头文件依赖。
>
> 所以它相当于一个 **薄包装层**，让你在 `obj_dir` 下敲 `make` 就能编译，而不用手动关心一堆参数。

### sub.v



```verilog
   always_ff @ (posedge clk) begin
      AssertionExample: assert (!reset_l || count_c<100);
   end
   // And example coverage analysis
   cover property (@(posedge clk) count_c==3);
```



> // Check SystemVerilog assertions
>
> VERILATOR_FLAGS += `--assert`，<font color=blue>使得编译出来的C++工程能执行assert的功能</font>
>
> 
>
> // Generate coverage analysis
>
> VERILATOR_FLAGS += `--coverage`，<font color=blue>使得编译出来的C++工程能执行覆盖率分析的功能</font>



### top.v

```verilog

   // Print some stuff as an example
   initial begin
      if ($test$plusargs("trace") != 0) begin
         $display("[%0t] Tracing to logs/vlt_dump.vcd...\n", $time);
         $dumpfile("logs/vlt_dump.vcd");
         $dumpvars();
      end
      $display("[%0t] Model running...\n", $time);
   end
```

> 可执行文件 +trace，传递参数；
>
> // Make waveforms
>
> VERILATOR_FLAGS += `--trace`，<font color=blue>使得编译出来的C++工程能执行trace的功能，即dump波形</font>
>
> 

### sim_main.cpp

```cpp
    // Using unique_ptr is similar to
    // "VerilatedContext* contextp = new VerilatedContext" then deleting at end.
    const std::unique_ptr<VerilatedContext> contextp{new VerilatedContext};

    // Pass arguments so Verilated code can see them, e.g. $value$plusargs
    // This needs to be called before you create any model
    contextp->commandArgs(argc, argv);

    // Randomization reset policy
    // May be overridden by commandArgs argument parsing
    contextp->randReset(2);

    // Verilator must compute traced signals
    contextp->traceEverOn(true);

    // Simulate until $finish
    while (!contextp->gotFinish()) {
        contextp->timeInc(1);  // 1 timeprecision period passes...

        // Toggle a fast (time/2 period) clock
        top->clk = !top->clk;

        if (!top->clk) {
            if (contextp->time() > 1 && contextp->time() < 10) {
                top->reset_l = !1;  // Assert reset
            } else {
                top->reset_l = !0;  // Deassert reset
            }
            // Assign some other inputs
            top->in_quad += 0x12;
        }

        // Evaluate model
        top->eval();
    }

    // Final model cleanup
    top->final();

    // Coverage analysis (calling write only after the test is known to pass)
#if VM_COVERAGE
    Verilated::mkdir("logs");
    contextp->coveragep()->write("logs/coverage.dat");
#endif
	top->delete(); //必须添加，不然生成的波形文件可能size为0；


```



### trace&覆盖率&断言

| 功能         | verilator选项 | verilog语句                                                  | C-testbench语句                                    | 备注                                                         |
| ------------ | ------------- | ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| trace功能    | --trace       | $dumpfile("logs/vlt_dump.vcd"); <br>        $dumpvars();     | contextp->traceEverOn(true);                       | gtkwave vlt_dump.vcd                                         |
| coverage功能 | --coverage    | // And example coverage analysis<br>   cover property (@(posedge clk) count_c==3); | contextp->coveragep()->write("logs/coverage.dat"); | verilator_coverage --annotate logs/annotated logs/coverage.dat |
| assert功能   | --assert      | always_ff @ (posedge clk) begin<br>AssertionExample: assert (!reset_l \|\| count_c<100);<br>end | NA                                                 | NA                                                           |
|              |               |                                                              |                                                    |                                                              |



# 进阶使用

verilator的仿真选项：

`timescale `仿真时间单位需要--timing参数？

如果没有timescale？？

`--timing`

contextp->timeInc()函数；

dump fsdb需要推进时间片;

contextp->traceOn(True);















---

参考博客&文档：

1. [Verilator_User_Guide](https://verilator.org/guide/latest/example_cc.html#example-c-execution)

