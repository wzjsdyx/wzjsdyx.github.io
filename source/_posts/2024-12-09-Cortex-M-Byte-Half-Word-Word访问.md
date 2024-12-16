---
title: Cortex-M Byte/Half_Word/Word访问
date: 2024-12-09 17:02:40
categories:
- Cortex-M4
tags:
- Byte access
- Half_Word access
- Word access
---

> Cortex-M4 是基于 ARM 架构的处理器，支持字节（Byte）、半字（Half-word）和全字（Word）类型的数据访问。具体操作可以通过 **C 语言的指针类型** 来实现。

### 访问类型定义

| **访问类型** | **数据大小** | **C 语言类型**        | **访问说明** |
| ------------ | ------------ | --------------------- | ------------ |
| Byte         | 8 位         | `uint8_t` 或 `char`   | 访问一个字节 |
| Half-word    | 16 位        | `uint16_t` 或 `short` | 访问两个字节 |
| Word         | 32 位        | `uint32_t` 或 `int`   | 访问四个字节 |

### 实现方式

通过指针类型来实现不同的访问大小。例如，假设一个外设寄存器地址为 `0x40000000`。

#### **1. Byte 访问**

使用 `uint8_t` 指针：

```c
#define REGISTER_ADDR 0x40000000
uint8_t *byte_ptr = (uint8_t *)REGISTER_ADDR;
*byte_ptr = 0xFF;  // 写入一个字节
uint8_t value = *byte_ptr;  // 读取一个字节
```

#### **2. Half-word 访问**

使用 `uint16_t` 指针：

```c
#define REGISTER_ADDR 0x40000000
uint16_t *halfword_ptr = (uint16_t *)REGISTER_ADDR;
*halfword_ptr = 0xFFFF;  // 写入两个字节
uint16_t value = *halfword_ptr;  // 读取两个字节
```

#### **3. Word 访问**

使用 `uint32_t` 指针：

```c
#define REGISTER_ADDR 0x40000000
uint32_t *word_ptr = (uint32_t *)REGISTER_ADDR;
*word_ptr = 0xFFFFFFFF;  // 写入四个字节
uint32_t value = *word_ptr;  // 读取四个字节
```

### 注意事项

Cortex-M4 允许非对齐访问，但性能可能会受到影响，建议确保地址对齐到数据大小的边界（1 字节、2 字节或 4 字节）。
