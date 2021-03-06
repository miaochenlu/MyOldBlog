---
title: Computer network--网络层part2
date: 2019-12-06 15:00:03
tags:  Computer Network
categories:  Computer Network
index_img: /img/image-20191126085207750.png
---

# 3. 拥塞控制算法

### i. 什么是拥塞[congestion]

{%note warning%}

网络中存在太多的数据包导致数据包被延迟延迟和丢失，从而降低了传输性能，这种情况称为拥塞

{%endnote%}

网络层和传输层共同承担着处理拥塞的责任。

* 由于拥塞发生在网络内，正是网络层直接经历着拥塞，而且必须由它最终确定如何处理过载的数据包。

* 然而，控制拥塞的最有效方法是减少传输层注入网络的负载。这就需要网络层和传输层共同努力协同工作。

### ii. 拥塞的影响

<center><img src="image-20191202212045404.png" alt="image-20191202212045404" style="zoom:30%;" /></center>

* 当主机发送到网络的数据包数量在其承载能力范围之内时， 送达的数据包数与发送的数据包数成正比例增长
* 然而，随着负载接近承载能力，偶尔突发的流量填满了路由器内部的缓冲区，因而某些数据包会被丢失。这些丢失的数据包消耗了部分容量，因此，送达的数据包数量低于理想曲线。网络现在开始拥塞。


* 除非网络是精心设计的，否则它极有可能会遭遇拥塞崩溃 [Ccongestion collapse ]，表现为随着注入负载的增加到超出网络的容量，网络性能骤降。

<br/>

拥塞控制和流量控制

> * 拥塞控制的任务是确保网络能够承载所有到达的流量。这是一个全局性的问题，涉及各方面的行为，包括所有的主机和所有的路由器。
> * 与此相反，流量控制只与特定的发送方和特定的接收方之间的点到点流量有关。它的任务是确保一个快速的发送方不会持续地以超过接收方接收能力的速率传输数据。

<br/>

## 3.1 拥塞控制的途径

拥塞的出现意味着负载（暂时）大于资源（在网络的一部分〉可以处理的能力。

很自然能想到两个解决方案

* 增加资源
* 减少负载。

<center><img src="image-20191202215504805.png" alt="image-20191202215504805" style="zoom: 33%;" /></center>

## 3.2 流量感知路由[traffic-aware routing]

把链路权重设置成一个（固定）链路带宽、传输延迟、（可变）测量负载或平均排队延迟的函数。在所有其他条件都相同的情况下，最小权重的路径更青睐那些***轻负载***的路径。

<br/>

## 3.3 准入控制[admission control]

一种广泛应用于虚电路网络，防止出现拥塞的技术是准入控制（ admission control ）

思想：

{%note warning%}

除非网络可以携带额外的流量而不会变得拥塞，否则不再建立新的虚电路。因此，任何建立新的虚电路的尝试或许会失败。

{%endnote%}

常用：<u>leaky bucket</u> or <u>token bucket</u>



## 3.4 流量调节[traffic throttling]

在 Internet 和许多其他计算机网络中，发送方调整它们的传输速度以便发送网络能实际传送的流量。在这种设置下，网络的目标是在拥塞发生之前正常工作。

而当拥塞迫在眉睫，它必须告诉发送方紧急刹车放慢传输速度。这种反馈是一种常态而不是针对特殊情况的一种处理。

<br/>

### 限制流量的方法

要解决两个问题：

#### 1. 路由器必须确定何时快要接近拥塞，最好在拥塞发生之前能确定[排队时延]

> 解决方法：每个路由器连续监测它正在使用的资源。可以输出在路由器内缓冲的排队数据包。路由器内部的排队延迟直接捕获了数据包经历过的任何拥塞情况
>
> 为了维持良好的排队延迟估计d, 假设s表示瞬时队列长度的样值, d可以定期生成，按如下方式更新，其中$\alpha$是一个参数，加权移动平均
>
> $$d_{new}=\alpha d_{old}+(1-\alpha)s$$
>
> 如果d升高到某一个阈值之上，则路由器就要开始注意拥堵了

#### 2. 路由器必须及时把反馈消息传递给造成拥塞的发送方

**2.A 抑制包**

> * 路由器选择一个被拥塞的数据包，给该数据包的源主机返回一个抑制包（choke packet ）。
>
> 抑制包中的***<u>目标地址</u>***取自该拥塞数据包。同时，在原来的拥塞数据包上添加一个***<u>标记</u>***（设置头部中的一位），因而它在前行的路径上不会产生更多的抑制包。
>
> * 数据包的转发过程如同平常 一样。 
>
> * 当源主机收到了抑制包，按照要求它必须减少发送给指定目标的流量，比如说减少 50% 。
>
> 在数据报网络中，发生拥塞时路由器简单地随机选择一个数据包，很有可能就把抑 制包发给了快速发送方，因为发送方的速度越快，它的数据包排队在路由器队列中的数目就越多。

<center><img src="image-20191205230214522.png" alt="image-20191205230214522" style="zoom:50%;" /></center>

**2.B 逐跳后压[hop-by-hop choke packets]**

因为传播延迟的影响，拥塞信号发出到起作用之间已经有很多新的数据包注入了网络。

因此，可以让抑制包在沿途的每一跳都发挥作用



<center><img src="image-20191205232440838.png" alt="image-20191205232440838" style="zoom:50%;" /></center>

**2.C Explicit Congestion Notification [ECN]**

> * 路由器可以在它转发的任何数据包上打上标记[设置数据包头的某一个标志位]发出信号，表明它正在经历着拥塞。
>
> * 当网络传递数据包时， 接收方可以注意到有个拥塞己经发生，在它发送应答包时顺便告知发送方。
>
> * 然后发送方可以像以前那样紧急刹车降低传输速率。

<center><img src="image-20191205231809305.png" alt="image-20191205231809305" style="zoom:50%;" /></center>


<br/>

## 3.5 负载脱落[load shedding]

当路由器因为来不及处理数据包而面临被这些数据包淹没的危险时，就将它们丢弃。

关键问题：选择丢弃哪个数据包

* 旧的比新的好，丢新包[wine策略]
* 新的比旧的好[milk 策略]



### 随机早期检测[random early detection]

为什么早期：

> 在拥塞刚出现苗头时就处理它比等拥塞形成之后再设法解决它更加有效，即在所有的缓冲空间都精疲力竭之前丢弃数据包。

为什么随机:

> 随机选择丢弃的数据包使得快速发送方发现丢包的可能性更大。发送方没有收到期待的确认信息时，就会认为丢包了，然后传输协议将放慢速度。因此，丢失的数据包起到了传递抑制包的同样作用

<br/>

# 4. 服务质量[QUALITY OF SERVICE ]

A flow is a stream of packets from a source to a destination.

The needs of each flow can be characterized by fourprimary parameters:

* Reliability(可靠性)

* 延迟(Delay)

* 抖动(Jitter)

* 带宽(Bandwidth)

## 4.1 应用需求

<center><img src="image-20191205233904014.png" alt="image-20191205233904014" style="zoom:30%;" /></center>

## 4.2 流量整形[traffic shaping]

<center><img src="image-20191206000433623.png" alt="image-20191206000433623" style="zoom:50%;" /></center>

# 5. INTERNETWORKING [网络互连]

## 5.1 How networks differ

<center><img src="image-20191224232122964.png" alt="image-20191224232122964" style="zoom:40%;" /></center>

## 5.2 How networks can be connected

多协议路由器[multiprotocol router]

<center><img src="image-20191224232335024.png" alt="image-20191224232335024" style="zoom:40%;" /></center>

## 5.3 Tunneling

处理两个不同网络相互连接时的一般情况超级困难。然而，却存在一种最常见的甚至可管理不同网络协议的情形。

> 这种情形就是源主机和目标主机所在网络的类型完全相同， 但它们中间却隔着一个不同类型的网络。举例来说，请考虑一家跨国银行，它在巴黎有一 个IPv6 网，在伦敦也有一个 IPv6 网, 但是连接巴黎和伦敦办事处的却是 IPv4 Internet

可以用tunneling来解决这个问题。

* 为了给伦敦办事处的主机发送一个 IP 数据包，巴黎的主机构造一个包含伦敦 IPv6 地址的数据包
* 然后将该数据包发送到连接巴黎 IPv6 网络到 IPv4 Internet 上的多协议路由器
* 当该多协议路由器获得 IPv6数据包后，它把该数据包用一个 IPv4 头封装，封装后的 IPv4 数据包指向多协议路由器另 一边的 IPv4，该网络与伦敦的 IPv6网络相连：也就是说，路由器将一个IPv6数据包放 入到一个IPv4数据包中。
* 当这个包裹着的数据包到达伦敦路由器，原来的 IPv6 数据包 被提取出来，并被发送给最终的目标主机。

{%note success%}

可以把通过 IPv4 Internet 的路径看作是一根从一个多协议路由器延伸到另一个多协议路由器的大隧道。

{%endnote%}

<center><img src="image-20191224232619688.png" alt="image-20191224232619688" style="zoom:40%;" /></center>

## 5.4 Internetwork Routing

### AS[Autonamous System]

> each network is operated independently of all the others, it is often referred to as an AS (Autonomous System).

Use Two-level routing

* Internet(inter-AS) routing: Exterior gateway protocol, like BGP
* Intranet(intra-AS) routing: Interior gateway protocol, like LS, DV



## 5.5 Internetworking: Fragmentation

每个网络或链路都会限制其数据包的最大长度。主机一般倾向于传输大的数据包，因为这样可以降低开销，比如浪费在头字节上的带宽。当一个大数据包要穿过一个最大数据包尺寸太小的网络时，一个明显的网络互联问题就出现了。

MTU[path Maximum Transmission Unit]

> 到达接收方可以发送的最大数据包尺寸

可以对数据包进行分段来解决这个问题，有两种分段策略

### A. transparent

> 由“小数据 包”网络引起的分段过程对于沿途后续的网络都是透明的，也就是说，从该网络一直到最终的目标途中的每个网络都感觉不到曾经发生过分段

一些问题

* 出口路由器必须知道什么时候它己经接收到了全部的段，所以每个分段中必须提供一个计数字段或者一个“数据包结束”标志位。
* 由于所有的数据包必须经过同一个出口路由器才能进行重组，因此路由受到了限制。由于不允许有些段沿着一条路径到达最终目标，而另一些段沿着一条不相交路径到达最终目标，所以，可能会损失一些性能。
* 路由器可能不得不做大量的工作。如果不是所有的段都己到达，它还需要缓冲到达的段，并且决定何时丢弃这些段。

<center><img src="image-20191224233917684.png" alt="image-20191224233917684" style="zoom:40%;" /></center>

### B. nontransparent

`优点`

是路由器做的工作比较少

IP采用的是非透明方式。

> 给每个段一个数据包序号(所有的数据包都携带), 一个数据包内的绝对字节偏移量和一个指明是否到达数据包末尾的标志位

<center><img src="image-20191224234335972.png" alt="image-20191224234335972" style="zoom:40%;" /></center>



***path MTU discovery***

这样还是会存在问题。但真正的问题首先还是因为段的存在，因此开销可能比透明分段高。因为除了增加头开销，数据包的丢失概率也增加了。

我们要提供在网络中避免分段操作的方式。

> 每个 IP 数据包发出时在它的头设置一个比特，指示不允许对该数据包实施分段操作。
>
> 如果一个路由器接收的数据包太大，它就生成一个报错数据包并发送给源端，然后丢弃该数据包
>
> 当源端收到报错数据包，它就使用报错数据包携带的信息重新将出错数据包分段，每个段足够小到报错路由器能处理。
>
> 如果沿着路径前进又遇到一个MTU更小的路由器，那么重复上述过程。

<center><img src="image-20191224234604657.png" alt="image-20191224234604657" style="zoom:50%;" /></center>

# 6

## 6.1 IPv4协议

IP数据包包含：

* 一个头[20bytes的定长部分和可选的变长部分组成]
* 一个正文

<center><img src="image-20191126085207750.png" alt="image-20191126085207750" style="zoom:50%;" /></center>

* version: 数据报属于协议的哪个版本, IPv4该字段总是4

* IHL[internet header length]: 由于头的长度不固定，IHL字段指明头到底有多长，以32bits长度为单位。

  IHL的最小值为5，5words=160bits=20bytes, 这表明头没有options字段。

  IHL的最大值为15

* Differentiated services: 

  - **Type of service** (past): 3 for priority, 3 for Delay, Throughput, and Reliability, and 2 unused. 
  - **Differentiated services** (now): 6 for service class, 2 for congestion (e.g. ECN). 

* Total length: the length of header and data. 以bytes为单位 最大值为65535bytes

* Identification: 让目标主机确定一个新到达的分段***属于哪一个数据报***。同一个数据报的所有段包含同样的表示值

* 灰色的部分是一个未使用的位

* DF： Don't fragment，不分段标志位 。只有当 DF = 0 时才允许分片  

* MF：more fragment。除了最后一个段，其他段都必须设置这一位。MF = 1 表示后面“还有分片”。MF = 0 表示最后一个分片

* Fragment: 指明该段在数据报中的位置。除了数据包的最后一个段，其他所有段的长度必须是8bytes的倍数。这个值是该段第一个比特在未分段的数据中所处的位置/8

* Time to live: 限制数据报生存期的计数器。计数单位被设置为秒。在每一跳上该计数器必须被递减。实际上是***跳计数器***，当他递减到0时，数据报就被丢弃。

* Protocol: 指明该将它交给哪个传输进程[TCP UDP ...]

* Header checksum: 

* Source address: 源IP地址

* Destination address： 目标IP地址

* options

<center><img src="image-20191126090914574.png" alt="image-20191126090914574" style="zoom:50%;" /></center>

<a href="https://blog.csdn.net/ztguang/article/details/70949781">OSPF protocol</a>



### 数据包分段

注意除了数据包的最后一个段，其他所有段的长度必须是8bytes的倍数

<center><img src="image-20191225002643293.png" alt="image-20191225002643293" style="zoom:50%;" /></center>

* 先找total length字段，知道数据包header+data的byte数

* 再找IHL, 知道header的word数

* total-length得到data.

* 一个数据包最多发送max byte, 用max - IHL就是一个数据包可以发送的data大小
  * 如果不是最后一个fragment的话，需要减去一些大小，使得发送的数据是8bytes的倍数
  * 如果不是最后一个分段，无所谓
* 设置DF=0, MF=1[如果不是最后一个fragment], offset用上述方法计算

## 6.2 IP地址

一个IP地址并不真正指向一台主机，而是指向一个网络接口，所以如果一台主机在两个网络上，它必须有两个IP地址。然而大多数主机都连在一个网络，因而只有一个IP地址。

路由器有多个接口，从而有多个IP地址



### A.前缀

IP地址具有层次性，可以将IP地址分成两部分

* 高位可变长网络
* 低位的主机

<center><img src="image-20191126091437663.png" alt="image-20191126091437663" style="zoom:50%;" /></center>

**子网掩码**可以与一个IP地址进行AND操作，提取出IP地址的network部分

如上图，子网掩码为255.255.255.0



### B.子网

<center><img src="image-20191126091938551.png" alt="image-20191126091938551" style="zoom:50%;" /></center>

### C. CIDR 无类域间路由

<center><img src="image-20191126113616524.png" alt="image-20191126113616524" style="zoom: 33%;" /></center>

Question:

<center><img src="image-20191126113649618.png" alt="image-20191126113649618" style="zoom: 33%;" /></center>

***对于A:***

<center><img src="image-20191126114956080.png" alt="image-20191126114956080" style="zoom:50%;" /></center>

因为需要1024个地址，所以host有10位，子网掩码是22位

202.101.0.0/22

***对于B:***

<center><img src="image-20191126115309939.png" alt="image-20191126115309939" style="zoom:50%;" /></center>

host有11位，子网掩码是21位

202.101.8.0/21

***对于C:***

<center><img src="image-20191126115323260.png" alt="image-20191126115323260" style="zoom:50%;" /></center>

host有11位，子网掩码是21位

202.101.16.0/21

### E. NAT

处理IPv4地址短缺的问题。

> NAT 的基本思想是 ISP 为每个家庭或每个公司分配一个IP地址，用这个 IP 地址来传输Internet 流量。在客户网络内部，每台计算机有唯一的IP 地址，该地址主要用来路由内部流量。然而，当一个数据包需要离开客户网络，发向其他 ISP 时，它必须执行一个地址转换，把唯一的内部 IP 地址转换成那个共享的公共 IP 地址。这种地址转换使用了 IP 地址的三个范围，这些地址己经被声明为私有化。任何网络可以在内部随意地使用这些地址。仅有的规则是不允许包含这些地址的数据包出现在 Internet 上。这 3 个保留的地址范围为：
>
> 10.0.0.0~ 10.255.255.255/8 (16 777216 个主机）
>
> 172.16.0.0 ~ 172.31.255.255/12 ( 1048576 个主机〉
>
> 192.168.0.0~ 192.168.255.255/16 (65536 个主机〉

<center><img src="image-20191227230836370.png" alt="image-20191227230836370" style="zoom:35%;" /></center>







## 6.4 IPv6 Protocol

IPv6协议采用128位地址，基本在未来任何时间都不可能出现地址短缺问题



### 主要的IPv6头[40bytes]

<center><img src="image-20191126120754968.png" alt="image-20191126120754968" style="zoom: 40%;" /></center>

* version: 对于IPv6,总是6

* Differentiated services: 区分数据包的服务类别

* Flow label: 为源端和接收方提供了一种建立伪连接的方式，即源端和接收方把一组具有同样需求并希望得到网络同等对待的数据报打上标记。
* payload length: 数据的字节数，这里是不包括头的，和IPv4不一样
* Next header: 指明当前头后还有哪种扩展头。如果当前的头是最后一个IP头，那么下一个头字段指定了该数据报将被传递给哪个传输协议处理(TCP, UDP)
* Hop limit: 跳数限制，相当于IPv4的TTL

* entension

IPv6地址的书写

**8** groups of **4** hexadecimal digits with colons between the groups

e.g. 8000:0000:0000:0000:0123:4567:89AB:CDEF

优化

1. leading zeros within a group can be omitted

   0123可以写作123

2. one or more groups of 16 zero bits can be replaced by a pair of colons

   8000::123:4567:89AB:CDEF

3. IPv4 addresses can be written as a pair of colons and an old dotted decimal number

   ::192.31.20.46





## 6.4 Internet控制协议

### ICMP---Internet Control Message Protocol

当路由器在处理一个数据包的过程中发生了意外，可以用过ICMP向数据包的源端报告有关事件

ICMP还可以用来测试Internet



<center><img src="image-20191129174803743.png" alt="image-20191129174803743" style="zoom:50%;" /></center>

### ARP---The address resolution protocol

<center><img src="image-20191227233627543.png" alt="image-20191227233627543" style="zoom:50%;" /></center>

作用是：将IP地址解析为MAC地址

为什么同时需要IP地址和MAC地址呢？

<a href="https://www.jianshu.com/p/0ce15c07b294">IP/MAC address</a>



ip地址负责标记发送方和接收方，MAC地址负责传输过程中的分段传送，比如下一跳的MAC地址。

<a href="https://blog.csdn.net/u010164190/article/details/79446575">ARP详解</a>

<a href="https://blog.csdn.net/lm409/article/details/80299823">ARP详解2</a>



### DHCP---Dynamic Host Configuration Protocol

动态获取IP地址

<a href="https://blog.csdn.net/wuruixn/article/details/8282554">DHCP协议的工作过程</a>

<center><img src="image-20191227233813861.png" alt="image-20191227233813861" style="zoom:50%;" /></center>

> 采用 DHCP 时，每个网络必须有一个 DHCP 服务器负责地址配置。
>
> * 当计算机启动时， 它有一个嵌入在 NIC 中的内置以太网地址或其他链路层地址，但没有 IP 地址。像 ARP 一 样，该计算机在自己的网络上广播一个报文，请求 IP 地址。这个请求报文就是 DHCP DISCOVER 包
> * 这个包必须到达 DHCP 服务器。如果 DHCP 服务器没有直接连在本地网络， 那么必须将路由器配置成能接收 DHCP 广播并将该请求报文中继给 DHCP 服务器，由 DHCP 服务器来处理 DHCP 报文。
> * 当 DHCP 服务器收到请求，它就为该主机分配一个空闲的 IP 地址，并通过 DHCP OFFER 包返回给主机（这个报文或许也要通过路由器中继〉。为了在主机没有 IP 地址的情况下完成此项工作，服务器用主机的以太网地址来标识这台主机（主机的以太网地址由 DHCP DISCOVER 包携带过来）。



为了确定一个主机拥有IP地址的时间，采用了leasing技术。在leasing期满前，主机必须和DHCP续订。



### Label Switching and MPLS

MPLS(multiprotocol label switching)

MPLS 在每个数据包前面增加一个标签，路由器根据***<u>数据包标签</u>***而不是数据包目标地址实施转发。用标签作为一个内部表的索引，快速查找该表找出正确的输出线路，因而这只是一个表查操作。使用这种技术，路由器的转发速度非常快。

<center><img src="image-20191227234537219.png" alt="image-20191227234537219" style="zoom:50%;" /></center>





