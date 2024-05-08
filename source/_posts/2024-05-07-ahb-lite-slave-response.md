---
title: ahb-lite slave response
date: 2024-05-07 20:48:32
categories:
- AHB-Lite总线
tags:
- response
---

# Slave transfer responses

==》Master发起一次transfer之后就不能主动取消，接下来的传输过程由Slave控制；

==》Master需要知道transfer的传输状态（chenggong/不成功），因此Slave需要返回response

{% asset_img image-20240508090512509.png %}

==》<font color=blue>完整的transfer response是HRESP和HREADY的组合</font>

{% asset_img image-20240508090625149.png %}

## Transfer done

<font color=blue>HRESP OK+ HREADY HIGH</font>

## Transfer pending

Slave可以在data phase插入wait states；

在真正response前插入wait states时，<font color=blue>HRESP OK+ HREADY LOW</font>;



1. 每个从设备在释放总线`back off the bus`前可以插入一定数量的等待周期，（但是必须要定义一个`最大的插入数量`）：

   - 这是为了处理总线上的竞争和冲突


   - 而且也可以计算最大的laterncy


2. 建议每个从设备最多插入16个等待周期，以避免某个访问锁定总线导致大量时钟周期的延迟。然而，对于某些设备，比如串行启动ROM，这个建议可能不适用。因为这类设备通常只在系统启动期间访问，并且如果使用超过16个等待周期，对系统性能的影响可以忽略不计。

> 我的理解：
>
> 假设处理器先访问传感器和再访问存储器，
>
> 处理器访问传感器的时候，传感器数据可能没有ready，这个时候就需要插入等待周期；
>
> 在这期间，存储器不应该再占用总线资源，否则会出现竞争；

{% asset_img image-20240324164813564.png %}



## ERROR response

==》"ERROR" 响应在AHB-Lite协议中用于指示某种错误条件。通常，这表示一种保护错误，比如尝试向只读内存位置写入数据。



==》对于 "OKAY" 响应，从设备可以在单个时钟周期内发送。对于 "ERROR" 响应，需要两个时钟周期。

1. <font color=blue>第一个周期：HRESP HIGH + HREADY LOW</font>
2. <font color=blue>第二个周期：HRESP HIGH + HREADY HIGH</font>



==》因为总线的pipeline特性，才需要两个周期response error。

假设有这样一个情景：

1. 主设备向从设备发送一个读取请求（例如从存储器中读取数据）。
2. 在第一个时钟周期内，主设备已经向总线广播了读取请求的地址。
3. 然后，从设备（假设是一个只读存储器）检测到发生了错误，需要发送一个错误响应。

现在，`问题在于，在从设备开始发送错误响应之时，主设备已经广播了下一个传输的地址。这意味着，主设备已经准备好进行下一个传输，而且总线上已经开始了新的传输流程`。在这种情况下，`如果从设备只发送一个时钟周期的错误响应，主设备可能无法及时取消下一个传输的执行。因为主设备需要一定的时间来检测到错误响应并采取相应的措施。如果主设备无法及时取消下一个传输，可能会导致错误的传输继续进行，从而造成系统错误或数据错误。`

<font color=blue>核心就是：发生错误的时候，协议不希望Master再继续发送transfer，而是变为IDLE状态，防止错误传输的继续进行，造成系统错误</font>

如果Slave需要超过两个cycle来完成error response的传输，也可以在最开始插入Transfer pending，也就是wait状态。

{% asset_img image-20240508085623441.png %}

<font color=blue>（Master可以利用第一个周期：HRESP HIGH + HREADY LOW进行处理，取消之后的transfer；第二个周期：HRESP HIGH + HREADY HIGH的时候，传输类型就可以变为IDLE）</font>

>  注意：
>
> 如果master进行的是burst类型的传输，当slave error response的时候：
>
> - master可以选择取消接下来的burst传输，（这不是严格的限制）
> - master也可以选择继续接下来的burst传输，（从协议上看，也是可以接受的）

