---
title: AHB-Lite总线上的数据valid
date: 2024-04-09 08:49:25
categories:
- AHB-Lite总线
tags:
- AHB-Lite data valid
---



> 在做BUS E2E逻辑时，需要明确数据什么时候有效，数据包括haddr[31:0],hwdata[31:0],hrdata[31:0]

{% asset_img image-20240409093444241.png %}

# Master侧发出的haddr在Slave侧做decode

- *_hsel
- *_htrans[1]
- *_hreadyin

同时拉高的时候，haddr valid



# Master侧发出的hwdata在Slave侧做decode

hwdata的data valid为什么要打拍？
Master侧，如果发出了AHB write transaction 请求，第一个address phase之后，wdata肯定会ready
IDLE，BUSY之类的只会在address阶段去插入，不会在data阶段去插入



# Slave侧发出的hrdata在Master侧做decode

hreadyout拉高的时候就认为，hrdata valid
