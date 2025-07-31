---
title: vcs和verdi查看glitch
date: 2025-07-07 17:47:54
categories:
- ip开发环境搭建
- EDA工具使用
tags:
- vcs
- verdi
---





---

![image-20250707175100846](D:\blog\wzjsdyx.github.io\source\_posts\2025-07-07-vcs和verdi查看glitch\image-20250707175100846.png)

![image-20250707175043628](D:\blog\wzjsdyx.github.io\source\_posts\2025-07-07-vcs和verdi查看glitch\image-20250707175043628.png)

实际芯片中，PRESP肯定也会有毛刺，但是满足setup time和hold time；

因而这个毛刺会存在，但是不会被PCLK抓到；



添加检测glitch的选项：

```shell
+fsdb+glitch=0 \
+fsdb+delta \
```





---

参考博客：

1. [vcs编译选项--不常用](https://blog.csdn.net/qq_40571921/article/details/136985727)
2. [Verdi查看delta time 波形方法](https://blog.csdn.net/zya1314125/article/details/140004686)
