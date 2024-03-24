---
title: 项目实践 AHB-Lite hsel和htrans
date: 2024-03-24 17:10:50
categories:
- AHB-Lite总线
tags:
- AHB-Lite hsel信号
- AHB-Lite htrans信号
---



只说明一点，HSEL信号和HTRANS的行为并不一致

纯粹的AHB-Lite Master，例如CPU IBUS，可以将HTRANS[1]连接到Slave的HSEL信号

但是针对BUS Matrix的HTRANS[1]信号和HSEL信号并不相同

{% asset_img image.png %}

连接错误可能会导致数据出错

{% asset_img 12yMGCLiuk.jpg %}

