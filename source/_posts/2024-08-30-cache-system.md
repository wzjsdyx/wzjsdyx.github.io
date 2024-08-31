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

![image-20240831103935303](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831103935303.png)

tag：数据标识符；

index：表示哪一个cache line；

offset：一个cache line中的地址；



## Direct-mapped Cache



![image-20240831104025717](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831104025717.png)



如果对某一个地址访问：

0、cache Initialization：clear all valid bit

1、会通过index选择哪一个cache line

2、会通过tags以及valid比较是否hit，（tags和data同时访问，效率更高，功耗更大）



直接映射方式的优点：

- 设计简单；
- access速度快（因为tags和data是同时访问的，不用先访问tags再访问data）；

直接映射方式的缺点：

- 命令率比较低；（很容易造成地址冲突，如下图所示：）



![image-20240831105329727](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831105329727.png)



> 0x4  ==> 001 00
>
> 0x24==> 001 00
>
> 两者的index是相同的，会不断地替换；





## Set-associative Cache

一个memory block可以存放在不同的way中；

相对于直接映射方式，它的命中率更高，因为对于同样的cache index，其可以映射到cache中的不同位置；



优点：

相比较于直接映射， 它的灵活性更高；



缺点：

硬件实现更复杂（寻找cache line更复杂）



![image-20240831111257318](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831111257318.png)

如图所示：如果cache size比较小的情况下，组相连映射的差距比较大；

![image-20240831111540247](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831111540247.png)



## Full-Associative Cache

一个memory block可以存放在任意一个cache line中；



优点：

命中率更高；

缺点；

硬件实现最复杂；



也是有他的应用场景在的：

TLB，其size非常小，可能只有32-64个entry





## Replacement Policies

如果是直接映射，一个memory block只有一个对应的cache line，没有什么替换策略；

如果是全相连映射，在cache line已经被塞满的情况下，可以选择多所有的cache-line进行替换；

如果是组相连映射，在cache line已经被塞满的情况下，可以选择替换的cache-line是ways的数量决定；



如果有多个cache-line可供选择：会有如下的替换策略：

- round robin：
  - cycle round
  - first in  first out
  - 简单，但是不是最高效的方式
- LRU（least-recently used）最不经常使用算法：（最合理）
  - 需要额外的逻辑对cache line进行追踪，
  - <font color=blue>通常使用的是pseudo-LRU算法</font>
- FIFO
- random

> Pseudo-LRU (Least Recently Used) 是一种近似于最少最近使用的缓存替换算法。LRU算法会记录每一个缓存块的使用顺序，当需要替换缓存块时，会选择最久未使用的块进行替换。但LRU算法的实现复杂度较高，尤其是在硬件实现中，需要较多的存储和计算资源。
>
> **Pseudo-LRU** 是一种简化的LRU算法，通过减少记录的精确度来降低实现复杂度。它不需要完全记录每个缓存块的使用顺序，而是通过使用额外的逻辑或位标记来大致估算哪些块最近被使用过。常见的pseudo-LRU实现包括以下几种方法：
>
> 1. **Tree-based Pseudo-LRU**: 用一棵二叉树来表示缓存块的状态，每个节点保存一个标志位，指示左子树或右子树中的块是否最近被使用过。通过遍历树的标志位，算法可以找到一个大致的最少最近使用的块来进行替换。
> 2. **Bit-based Pseudo-LRU**: 通过为每个缓存块分配一位标志位（或者多位），记录是否最近被访问。更新标志位时不精确记录顺序，而是通过简单规则更新（如最近访问的块标志位置1，其它块标志位置0），从而近似实现LRU行为。
>
> **优点**:
>
> - 实现复杂度低：相比于精确的LRU算法，pseudo-LRU使用较少的硬件资源。
> - 提高效率：在大多数情况下能够接近LRU算法的效果，降低缓存冲突。
>
> **缺点**:
>
> - 不如精确的LRU算法那么准确：在某些情况下，可能不会选择最佳的缓存块进行替换，导致性能下降。
>
> 总结来说，pseudo-LRU是一种在性能和实现复杂度之间做出权衡的缓存替换算法，适用于硬件资源有限但仍需较好缓存管理的场景。
>
> <font color=blue>硬件实现的时候，如果复杂度较高，在合理范围内可以使用近似的方式去实现，复杂度更低，对硬件更加友好；</font>





## LRU实现方式

![image-20240831114206050](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831114206050.png)



> - **MRU（Most Recently Used）**: 指的是缓存中最近使用的块。图中的“MRU”标记在每次缓存命中（Hit）或缺失（Miss）时，都会被更新到最新的访问位置。这意味着，MRU是最不可能被替换的块，因为它是最近使用过的。
> - **LRU（Least Recently Used）**: 指的是缓存中最久未使用的块。图中的“LRU”标记代表最可能被替换的缓存块，因为它已经有一段时间未被访问。
>
> **图的说明**:
>
> 1. **Hit (命中)**:
>    - **块B被访问**时，它已经在缓存中（命中）。
>    - 缓存将块B的LRU值设置为0（MRU位置），表示它现在是最近使用的块。
>    - 在B和MRU之间的所有块（即块C）的LRU值增加，表示这些块的使用顺序向后移动了一位。
>    - 块A和块D的LRU值保持不变，因为它们的顺序并未被影响。
> 2. **Miss (缺失)**:
>    - 当访问块E时（如图下部分），它不在缓存中，因此会发生缓存缺失（Miss）。
>    - 系统会查找LRU值最大的块（即当前的LRU块），然后将其替换为新的块E。
>    - 块E被放入MRU位置（LRU值设置为0），表示它是最新使用的块，而其他块的LRU值则相应增加。
>
> 这个过程通过更新块的LRU值（越大表示越少使用），实现了自动替换缓存中最不常用的块，确保经常使用的数据优先保存在缓存中。



## cache policies- write hit, write-through,write-back

当write-hit的时候，write策略有如下两种：

write-through：同时更新cache和外部memory

write-back	  ：只更新cache，不更新外部memory

（write-back的好处是不需要频繁的访问外部memory，但是控制变复杂；）

write-once      :write-through on the first write, write-back on all subsequent write(cache coherence)

（当系统有多个cache并且采用write-back策略的时候，此时为了cache一致性，需要将第一笔数据write-through）

![image-20240831124710411](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831124710411.png)



## cache policies-write miss：write-allocatioin，non-writre-allocation



![image-20240831124801722](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831124801722.png)



如果采用write-allocate策略，当发生write miss的时候，先`read-fill`,然后write-hit;

read-fill操作比较重要，因为并不是单纯的做read，还需要修改，需要通知别的cache做invalid(维护cache一致性)

`read-fill和read miss fill在bus transaction上看到的状态是不同的；`



> 一般来说：
>
> write-back搭配write-allocate
>
> write-through搭配non-write-allocate





## Multi-level caches

![image-20240831125558401](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831125558401.png)



> <font color=blue>snoop操作</font>
>
>  **Snoop 操作的背景**
>
> 在多核处理器中，每个核心通常都有自己的私有缓存（如L1缓存和L2缓存）。为了确保多个核心之间的数据一致性，需要使用一种机制来保证当一个核心修改了数据，其他核心能够及时获知并更新它们的缓存。这就涉及到了 **缓存一致性协议**，如MESI协议（Modified, Exclusive, Shared, Invalid）。
>
> **Snoop 操作** 是这种缓存一致性协议中的一部分，它的目的是在一个核心对内存位置进行读写操作时，通知其他核心检查它们的缓存状态，并根据需要更新或无效化它们缓存中的数据。
>
>  **举个具体例子**
>
> 假设我们有一个四核处理器（核心A、B、C、D），每个核心都有自己的L1缓存。现在有一个内存位置X，它最初存储在主存储器中。
>
> **步骤 1: 核心A读取内存位置X**
>
> - **读取操作**: 核心A需要访问内存位置X。首先，它会检查自己的L1缓存，看是否已经有X的缓存行。如果没有（缓存未命中），它会将请求发送到L2缓存，L2缓存也没有（再未命中），最终它会从主存储器中加载X。
> - **缓存X**: 一旦X被加载，核心A会将其存储在自己的L1缓存中，并将状态标记为 **"Exclusive"**（意味着只有核心A有这个缓存行，且数据与主存储器一致）。
>
> **步骤 2: 核心B写入内存位置X**
>
> - **写入操作**: 现在，核心B决定要写入内存位置X。为了保持一致性，核心B需要先检查其他核心是否也缓存了X（即是否有核心A、C或D缓存了X）。
> - **Snoop操作**: 核心B的写请求会触发 **snoop 操作**。在MESI协议下，核心B会发送一个 **snoop 请求**，询问其他核心（A、C、D）是否缓存了X。
>
> **步骤 3: 其他核心响应snoop请求**
>
> - **核心A响应**: 核心A检查自己的缓存，发现它确实有X的缓存行（状态为Exclusive）。根据协议，核心A必须将X的缓存行状态设置为 **"Invalid"**（无效），表示它现在不能再使用这个缓存行，因为核心B要更新X。
> - **核心C和D**: 如果核心C和D没有X的缓存行，它们会简单地忽略snoop请求或返回“未缓存”的响应。
>
> **步骤 4: 核心B完成写入**
>
> - **写入数据**: 在核心B收到所有核心的snoop响应后（A将其缓存行无效化），核心B将X的更新后的数据写入自己的L1缓存，并将状态标记为 **"Modified"**（修改过的，且数据与主存储器不一致）。
>
> **后续操作**
>
> - **其他核心读取X**: 如果核心C或D之后试图读取内存位置X，它们会发现自己的缓存中没有有效的X（如果有，状态会是Invalid）。因此，它们会发出读取请求，最终从核心B的缓存中获得最新的X数据。
>
> 
>
> <font color=green>可以看到这个overhead非常大；</font>
>
> <font color=green>希望在last level cache中，就可以判断数据是否在cache中:</font>
>
> - <font color=green>如果不在的话，就不发出snoop请求;</font>
>
> - <font color=green>如果在，具体在哪个cache中，只在需要更新的cache中，发出snoop请求；</font>
>
> 
>
> <font color=blue>LLC （last level cache）+  snoop filter机制</font>
>
> **Snoop Filter 在有 LLC 的情况下的工作机制**
>
> 在多核处理器中，LLC作为最后一级缓存，由多个核心共享。在这种架构下，Snoop Filter的主要功能是减少不必要的snoop操作，从而优化系统的缓存一致性管理。
>
> **基本机制：**
>
> 1. **跟踪缓存行状态**：
>    - Snoop Filter会跟踪每个缓存行的状态，尤其是跟踪哪些缓存行可能在某个核心的私有缓存（如L1或L2缓存）中存在。这些信息通常存储在一个表格或目录结构中，这个表格记录了哪些核心正在缓存哪些数据。
> 2. **过滤snoop请求**：
>    - 当一个核心需要访问或修改某个数据（如一个缓存行）时，首先会检查LLC。如果在LLC中未命中，或需要更新数据，通常需要发起snoop请求，通知其他核心检查它们的私有缓存。
>    - 在有Snoop Filter的情况下，Snoop Filter会根据其记录的缓存行信息决定是否需要向所有核心广播snoop请求，还是只向特定的核心发送请求。这样可以显著减少不必要的通信开销。
> 3. **优化缓存一致性**：
>    - 通过减少无关核心的snoop请求，Snoop Filter不仅降低了总线上的通信负担，还减少了核心的响应延迟，提升了系统整体的效率。
>
> **具体例子说明：**
>
> 假设我们有一个四核处理器（核心A、B、C、D），每个核心都有自己的L1缓存，还有一个共享的LLC。Snoop Filter与LLC集成在一起，用于管理缓存一致性。
>
> **场景设置：**
>
> 1. **初始状态**：
>    - 核心A从主存储器加载了数据块X，并将其存储在L1缓存中，同时LLC中也缓存了这块数据。Snoop Filter记录了“数据块X在核心A的L1缓存中”。
>    - 核心B、C、D的缓存中没有数据块X。
> 2. **核心B尝试修改数据块X**：
>    - 核心B决定修改数据块X。这意味着它需要确保其他核心不能继续使用旧的数据块X（以防止数据不一致）。根据MESI协议，这通常会触发snoop请求。
>    - 核心B检查LLC。假设LLC中的数据块X被标记为“共享状态”，那么核心B的请求将被发送给Snoop Filter。
> 3. **Snoop Filter的作用**：
>    - Snoop Filter检查其记录，发现数据块X的共享记录表明它只在核心A的L1缓存中存在。
>    - Snoop Filter决定**只向核心A发送snoop请求**，而不向核心C和D发送请求，因为它知道C和D并没有缓存数据块X。
> 4. **核心A响应**：
>    - 核心A收到snoop请求后，将数据块X的状态更新为“无效”，因为核心B要对其进行修改。
>    - 核心B随后将数据块X从LLC加载到自己的缓存中，并执行修改操作。此时，数据块X在B的缓存中被标记为“修改”状态。
> 5. **后续操作**：
>    - 如果之后核心C或D尝试访问数据块X，它们将会查询LLC。如果LLC中数据块X已经被核心B修改，且其他核心的缓存中没有有效的副本，C或D将从B的缓存中读取最新的数据块X。



## Inclusive/Exclusive Cache

![image-20240831133616501](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831133616501.png)





![image-20240831134213174](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831134213174.png)

![image-20240831134612895](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831134612895.png)

从AMAT公式，可以看到，有三个方向可以做优化，去提高Cache性能；

![image-20240831134732393](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831134732393.png)

![image-20240831134908989](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831134908989.png)



软件很清楚自己用到的数据在哪里，可以使用prefetch指令

![image-20240831135507425](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831135507425.png)



---



![image-20240831135711333](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831135711333.png)





![image-20240831135809264](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831135809264.png)

![image-20240831135839085](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831135839085.png)



略：



![image-20240831140112738](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831140112738.png)

![image-20240831140145060](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831140145060.png)



write操作一般对latency不太敏感，因为有write buffer；

read操作对latency比较敏感；



![image-20240831140545084](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831140545084.png)



![image-20240831140831090](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831140831090.png)









投机访问





---

![image-20240831140902321](D:\blog\wzjsdyx.github.io\source\_posts\2024-08-30-cache-system\image-20240831140902321.png)





---

参考博客：

1. [cache system](https://www.bilibili.com/video/BV13T421k7TS/?spm_id_from=333.999.0.0&vd_source=c76cab13994e90aef30da628f94d99e8)
