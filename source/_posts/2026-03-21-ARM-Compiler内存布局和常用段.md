---
title: ARM_Compiler内存布局和常用段
date: 2026-03-21 22:14:37
categories:
- 嵌入式C程序
- 编译链接
tags:
- TBD
---

# 内存布局文件

`.scat` = Scatter-loading description file,分散加载描述文件，把程序“分散放到不同内存区域”

```c
lr 0x00000000
{
    rom_s 0x00000000
    {
        boot_exectb_mcu.o(vectors,+FIRST)
        *(InRoot$$Sections)
        .ANY (+RO)
    }

    ram_s 0x20000000 {
        .ANY(+RW,+ZI)
    }

    ARM_LIB_HEAP +0 EMPTY 0x10000
    { }

    ARM_LIB_STACK 0x200FFFF0 EMPTY -0x8000
    { }

    ram_ns 0x20100000 EMPTY 0x100000
    { }
}
```

> 解释说明：
>
> Load Region（加载域）
>  ├── Execution Region（执行域）
>  ├── Execution Region
>  └── ...
>
> LR (Load Region)	整体镜像（烧进ROM的）
> ER (Execution Region)	运行时所在区域
>
> 
>
> lr 0x00000000 ，整个程序镜像从 `0x00000000` 开始
>
> 



# 常用的section段

