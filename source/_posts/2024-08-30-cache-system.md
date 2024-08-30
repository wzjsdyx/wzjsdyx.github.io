---
title: cache system
date: 2024-08-30 20:08:48
tags:
---



Agenda

- Why Cache?
- Cache Organization
- Replacement policy
- Write policy
- Cache Performance
- Cache Coherence
- Scratchpad Memory



## Why Do We Need Caches?



![image-20240830201109956](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240830201109956.png)

> 存储墙（Memory wall）：
>
> 处理器的速度和memory的速度差距越来越大；



==》程序的运行一般都符合程序局部性原理（指令的访问以及数据的访问都具有局部性）

- 时间局部性：当前访问的地址，之后还有可能访问。

- 空间局部性：当前访问的地址，其相邻地址之后有可能会访问。



因此，从局部性的角度来说，在不久的将来，只有一小部分地址空间会被访问到，例如

程序的顺序执行

子程序的调用；

循环语句等等；



==》一般来说，memory size越小，其性能可以做的越快，可以放到距离CPU比较近的位置；

自身的访问速度的限制；

走线的长度限制；



==》cache存储的是memory中的副本





## Memory Access Pattern & locality

1、当前访问的地址数据，可以认为它在短时间内还会再被使用（时间局部性）--》为什么将数据保存到cache中的原因；

2、当前访问地址的相邻数据，也可以认为在短时间内还会被再次使用（空间局部性）--》数据以block的形式加载到cache（cache-line）

3、按顺序访问memory（空间局部性）--》prefetch

4、循环和函数调用





## Cache operation

1、当CPU需要读取某个内存地址的时候，它首先会check想要访问的数据是否在Cache中，

- 如果数据在cache中，即cache hit，此时会把Cache中的数据直接返回到CPU中；
- 如果cache不在cache中，即cache miss，通常cache会分配一个cache line去保存新的memory数据
  - （也不是绝对，有些先进的处理器技术可以预判当前read操作的数据将来不会被使用，此时就不需要分配cache line去保存数据）
  - 还需要考虑，如果cache 满了，哪些cache-line会被替换

2、如果CPU write 某个地址，他同样会先check想要访问的数据是否在cache中，

- 如果数据在cache中，
- 如果数据不在cache中，即cache miss，是否一定要write-locate？不一定，通常来说有两种处理策略：
  - write-allocate（同样，有些先进的处理器技术可以预判当前write操作的数据将来不会被使用，此时就不需要分配cache line去保存数据）
  - write to main memory

- 命中率是衡量cache性能指标



##  Accessing the cache- address mapping





















































