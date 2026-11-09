---
title: 培训笔记-11-Features-Master Interface ID width
date: 2026-11-09 10:52:05
categories:
- ARM
- CoreLink-NIC
tags:
- TBD

---



原文

> In this video, we are going to look at how the master interface ID is calculated. 
>
> The interconnect will append bits onto the input ID bits and ensure that they are consistent internally and ready to route transactions back to the originating masters by using the ID. Three components determine the master interface ID width. 
>
> - Firstly, a Virtual ID is the input ID from your master device. If the master has an output ID with 5 bits, you set the Virtual ID to 5. The virtual ID width used for calculating the output ID width is determined by the largest VID of all the master interfaces. 
> - We then concatenate that with the interconnect ID, which is the number of switches that have connecting ASIBs. If we look at the bm0, bm2, bm3 from the figure in the slide, three switches have connecting ASIBs, which results in an interconnect ID width of 2. 
> - The third component is the slave interface ID, which is the number of bits required to store the largest Switch size that has connecting ASIBs. In our example, bm3 is the largest in that aspect, with four input ports connected to ASIBs; 2 bits would be required to cover this. Adding these three components gives us, by default, without the ID reduction, a master interface ID width of 9. However, the NIC-400 can also apply an ID reduction by optimizing away static bits or bits that otherwise would not be used at the output, thus reducing the ID width of the master interface. For example, the master with the largest Virtual ID may only connect to a specific subset of slaves within the system. The AMIBs that this master does not have a path to do not need as large an output width, and so the NIC-400 can correspondingly reduce the output ID width on those interfaces.Please note that ID reduction should not be confused with ID compression. ID reduction cannot output an arbitrary ID width.

在本视频中，我们将介绍如何计算 Master Interface ID（主接口 ID）。

<font color=blue>互联网络（Interconnect）会在输入 ID 的基础上附加一些额外的 ID 位，以确保 ID 在内部保持一致，并能够利用 ID 将事务响应正确路由回发起事务的 Master。</font>

Master Interface ID 的位宽由三个部分决定。

1. Virtual ID（VID，虚拟 ID）

首先，Virtual ID 是来自 Master 设备的输入 ID。

如果某个 Master 输出的 ID 位宽为 5 bits，那么就将 Virtual ID 设置为 5。

在计算输出 ID 位宽时，使用的 Virtual ID 位宽取决于所有 Master Interface 中最大的 VID。

2. Interconnect ID（IID，互联 ID）

然后，我们将 Virtual ID 与 Interconnect ID 进行拼接（Concatenate）。

Interconnect ID 的位宽由连接了 ASIB 的 Switch 数量决定。

如果观察幻灯片中的 bm0、bm2、bm3，可以发现共有三个 Switch 连接了 ASIB，因此 Interconnect ID 的位宽为 2 bits。

3. Slave Interface ID（SID，从接口 ID）

第三部分是 Slave Interface ID。

其位宽取决于连接了 ASIB 的 Switch 中，最大 Switch 所需要的 ID 位数。

在我们的例子中，bm3 在这方面是最大的，它有四个输入端口连接到 ASIB，因此需要 2 bits 来表示这些端口。

将这三个部分相加，在默认情况下，如果不使用 ID Reduction（ID 缩减），Master Interface ID 的位宽就是 9 bits。

4. ID Reduction（ID 缩减）

不过，NIC-400 还可以应用 ID Reduction。

它通过优化并移除固定不变的 ID 位，或者在输出端不会被使用的 ID 位，从而减小 Master Interface 的 ID 位宽。

例如，具有最大 Virtual ID 位宽的 Master，可能只能连接到系统中特定的一部分 Slave。

对于那些与该 Master 之间不存在访问路径的 AMIB，就不需要使用如此大的输出 ID 位宽。

因此，NIC-400 可以相应地缩减这些接口上的输出 ID 位宽。

5. ID Reduction 与 ID Compression 的区别

请注意，不要将 ID Reduction 与 ID Compression（ID 压缩）混淆。

ID Reduction 无法生成任意指定的 ID 位宽。





![image-20261109105648813](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-11-Features-Master-Interface-ID-width\image-20261109105648813.png)

![image-20261109105734369](D:\blog\wzjsdyx.github.io\source\_posts\2026-11-09-培训笔记-11-Features-Master-Interface-ID-width\image-20261109105734369.png)

理解



