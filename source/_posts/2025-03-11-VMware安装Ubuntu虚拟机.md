---
title: VMware安装Ubuntu虚拟机
date: 2025-03-11 21:05:22
categories:
- 一生一芯计划
- 预学习阶段
tags:
- RISC-V CPU设计
---



# 虚拟机以及虚拟磁盘

> 1. **虚拟机配置文件**
>
> - **文件格式**：`.vmx` 文件
> - **作用**：
>   - 这是一个文本文件，用于存储虚拟机的配置信息，例如硬件设置（内存大小、CPU 数量、网络配置等）、虚拟设备（如 CD/DVD 驱动器、USB 控制器等）以及其他虚拟机相关的参数。
>   - 它是虚拟机的“元数据”文件，VMware 通过读取这个文件来加载虚拟机的配置。
> - **示例文件名**：`Ubuntu.vmx`
>
> ------
>
> 2. **虚拟磁盘文件**
>
> - **文件格式**：`.vmdk` 文件
> - **作用**：
>   - 这是虚拟机的虚拟磁盘文件，用于存储虚拟机的操作系统、应用程序和用户数据。
>   - 它模拟了一个物理硬盘，Ubuntu 系统会将其视为一个真实的磁盘设备，并在其中安装和运行。
>   - `.vmdk` 文件可以动态扩展（thin provisioned）或预先分配全部空间（thick provisioned），具体取决于创建虚拟机时的设置。
> - **示例文件名**：`Ubuntu.vmdk`
>
> 
>
> - **`.vmx` 文件**：虚拟机的配置文件，定义虚拟机的硬件和设置。
> - **`.vmdk` 文件**：虚拟磁盘文件，存储虚拟机的操作系统和数据。
>
> 这两个文件是 VMware 虚拟机的核心组成部分，缺一不可。如果需要迁移或备份虚拟机，通常需要同时复制这两个文件。



# 账号密码

用户名：`Zhijun Wang`

密码：`2025`



用户名：`root`

密码：`2025`





# 关闭开机时的软件更新

# linux初始root密码设置

# 配置中文输入法



# open_VM_Tools

```shell
sudo apt update
sudo apt install open-vm-tools-desktop
sudo reboot
vmware-toolbox-cmd -v
```

# 桥接模式无法连接网络

> 问题描述：
>
> 右上角网络连接出现`?`,游览器无法上网；
>
> Ubuntu虚拟机，配置的是桥接模式

问题解决：

step1：控制面板》网络和 Internet》网络和共享中心》更改适配器设置（Windows11系统）

{% asset_img image-20250910223822899.png %}

step2:打开VMware软件》编辑》虚拟网络编辑器》更改设置，将桥接模式选择上面的网卡即可；

{% asset_img image-20250910224040357.png %}

<font color=blue>VMware桥接模式的“自动”有时并不自动，有的时候需要手动配置桥接模式的网卡</font>

参考博客：

1. [VMware的Linux虚拟机桥接模式突然上不了网解决方法](https://cloud.tencent.com/developer/article/2093268)

# Ubuntu虚拟机 使用主机代理

ping不通，但是游览器能访问google；

git clone github可以使用



参考博客：

1. [vmware虚拟机解决vpn链接问题 win or linux系统](https://zhuanlan.zhihu.com/p/661644999)

---

参考博客：

1. [基于VMware虚拟机的Ubuntu22.04系统安装和配置](https://blog.csdn.net/qq_42417071/article/details/136327674)

2. [Open-VM-Tools：虚拟化环境中的开源工具套件](https://blog.csdn.net/weixin_35706067/article/details/142925696)
3. [ubuntu22.04安装vmware tools](https://blog.csdn.net/sofa_r/article/details/126060382)

3. [关闭 Ubuntu 开机软件更新弹窗通知](https://www.11343.com/8094.html)

4. [Linux初始root密码设置](https://www.cnblogs.com/lzj-0218-second/p/4292118.html)
5. [Ubuntu22.04无需命令行安装中文输入法](https://blog.csdn.net/zhg2546179328/article/details/134761713)
