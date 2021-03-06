---
title: Computer network--网络层part1
date: 2019-12-06 14:47:43
tags:  Computer Network
categories:  Computer Network
index_img: /img/image-20191119095439198.png
---

# 1. 网络层的设计问题

## 1.1 存储转发数据包交换

阴影椭圆中的是网络服务提供商(ISP)的设备

如果一台主机想要发送一个数据包，他就将数据包传输给最近的路由器。

在该数据包到达路由器，并且路由器的链路层完成了对它校验和的验证之后，它先被存储在路由器上；然后沿着路径被转发到下一个路由器，直至到达目标主机

<center><img src="image-20191119095439198.png" alt="image-20191119095439198" style="zoom: 25%;" /></center>


## 1.2 提供给传输层的服务

1. 向上提供的服务应该独立于路由器技术。
2. 应该向传输层屏蔽路由器的数量、类型和拓扑关系。 
3. 传输层可用的网络地址应该有一个统一编址方案，甚至可以跨越 LAN 和 WAN。



## 1.3 无连接服务的实现

> 所有的数据包都被独立地注入到网络中，并且每个数据包独立路由，不需要提前建立任何 设置。在这样的上下文中，数据包通常称为数据报[datagram]，它类似于电报[telegram], 对应的网络称为数据报网络[datagram network]

<center><img src="image-20191219154907937.png" alt="image-20191219154907937" style="zoom:40%;" /></center>

## 1.4 面向连接服务的实现

> 在发送数据包之前，必须首先建立起一条从源路由器到目标路由器之间的路径。这个连接称为虚电路[VC, virtual circuit]，它类似于电话系统中建立的物理电路，对应的网络称为虚电路网络[virtual-circuit network]

虚电路背后的思想是避免为要发送的数据包选择新路径。相反，当建立一个连接时，从源机器到目标机器之间的一条路径就被当作这个连接的一部分确定了下来，并且保存在这些中间路由器的表中。所有需要在这个连接上通过的流量，都使用这条路径。

如图，主机H1与主机H2之间建立了连接1。这条连接被记录在每个路由表中的第一项中。 

A 路由表的第一行说明如果一 个标示了连接标识符1的数据包来自于H1，那么它将被发送到路由器 C，并且赋予连接标识符 1 。类似地， C路由表中的第一项将该数据包路由到 E，也赋予连接标识符 1

现在我们来考虑如果 H3 也希望与 H2 建立连接则情形会怎么样。 H3 选择连接标识符 1 [因为是它发起连接，而且这是它唯一的连接]，并且告诉网络要建立虚电路。因此路由表增加了第二行。请注意，这里有一个冲突，因为，尽管 A 很容易区分出标识连接 1 的数据包是来自H1还是来自 H3 ，但是， C 无法区分它们。基于这个原因， A 给第二个连接的出境流量分配一个不同的连接标识符。这种避免冲突的做法说明了为什么路由器需要具备替换出境数据包中连接标识符的能力。

<center><img src="image-20191219154931266.png" alt="image-20191219154931266" style="zoom:50%;" /></center>



## 1.5 虚电路和数据报网络的比较

<center><img src="image-20191219222827287.png" alt="image-20191219222827287" style="zoom:40%;" /></center>

* 建立时间： 
  * 虚电路需要一个建立阶段，这个阶段花费时间也消耗资源。但是，付出这个代价后，处理一个数据包的方法却非常简单：路由器只要使用电路号作 为索引，在表中找到该数据包的去向即可。
  * 在数据报网络中，不需要建立电路，但路由器需要执行一个更为复杂的查找过程以便找到目标表项。
* 避免拥塞：
  * 虚电路在建立连接接时，资源可以提前预留（比如缓冲区空间、带宽和 CPU 周期〉。一旦数据包开始到来， 所需要的带宽和路由器容量都已经准备就绪。
  * 而对于数据报网络，避免拥塞更困难些。
* 脆弱性：
  * 虚电路也存在脆弱性问题。如果一台路由器崩溃并且内存中的数据全部丢失，那么即 使它一秒钟之后又重新启动，所有从它这里经过的虚电路都将不得不中断。
  * 如果一台数据报路由器岩机，只有那些当时尚留在路由器队列中的数据包用户会受到影响

# 2. 路由算法

路由算法是网络层软件的一部分，负责确定一个入境数据包应该被发送到哪一条输出线路上。

**路由**和**转发**这两个功能的区别

* 路由

{% note warning%}

涉及一个网络的**所有路由器**，经由路由选择协议共同交互，来决定分组从源到目的地结点所采取的路 

{%endnote%}

* 转发

{%note warning%} 

分组在**单一的路由器**从一条入链路到一条出链路到传送 

{%endnote%}

<br/>

路由算法可以分成两类

> * 非自适应算法：不会根据当前测量或者估计的流量和拓扑结构，来调整它们的路由决策。
>
> * 自适应算法



## 2.1 优化原则

最优化原则：

{%note warning%}

如果路由器J在从路由器I到路由器K的最优路径上，那么从J到K到最优路径必定遵循同样的路由

{%endnote%}

<center><img src="image-20191129193218868.png" alt="image-20191129193218868" style="zoom:50%;" /></center>

直接推论：从所有的源到一个指定目标的最优路径的集合构成了一棵以目标节点为根的树。这样的树称为***汇集树[sink tree]***

路由算法的目标就是为所有的路由器找到汇集树，并根据汇集树来转发数据包

<center><img src="image-20191129193740688.png" alt="image-20191129193740688" style="zoom:50%;" /></center>

## 2.2 最短路径算法

可以选择距离、带宽、平均流量、通信成本、平均延迟等作为路径的权重

使用Dijkstra算法

<table>
  <tr>
    <td><img src="image-20191202182043293.png" alt="image-20191202182043293" style="zoom:50%;" /></td>
    <td><img src="image-20191202182130071.png" alt="image-20191202182130071" style="zoom:50%;" /></td>
  </tr>
  <tr>
    <td><img src="image-20191202182159979.png" alt="image-20191202182159979" style="zoom:50%;" /></td>
    <td><img src="image-20191202182231738.png" alt="image-20191202182231738" style="zoom:50%;" /></td>
  </tr>
  <tr>
    <td><img src="image-20191202182426228.png" alt="image-20191202182426228" style="zoom:50%;" /></td>
  </tr>
</table>




## 2.3 泛洪算法[flooding]

flooding将每一个入境数据包发送到除了该数据包到达的那条线路以外的每条出境线路。

泛洪法会产生大量的重复数据包。事实上，除非采取某些措施来抑制泛洪过程，否则将会产生无限多的数据包。

* 在每个数据包的header中设置一个hop counter

  > 每经过一跳该计数器减一，当计数器到达 0 时就丢弃该数据包。
  >
  > 理想情况下，跳计数器的初始值应该等于从源端到接收方之间路径的长度。如果发送方不知道该路径有多长， 它可以将计数器的初始值设置为最坏情形下的长度，即网络的直径。

* keep track of which packets have been flooded, to avoid sending them out a second time.

<br/>

## 2.4 距离矢量算法[Distance Vector]

> 每个路由器维护一张表（即 一个矢量〉，表中列出了当前己知的到每个目标的最佳距离，以及所使用的链路。
>
> 这些表通过邻居之间相互交换信息而不断被更新，最终每个路由器都了解到达每个目的地的最佳链路。

<a href="https://blog.csdn.net/u013007900/article/details/45565389">距离矢量算法详解</a>

### i. count-to-infinity problem

虽然DV算法总是能够收敛到正确的答案，但速度可能非常慢。尤其是它

* 对于好消息的反应非常迅速
* 而对于坏消息的反应异常迟缓。

#### i.A 来看一下对好消息是怎么反应的

<center><img src="image-20191202184133049.png" alt="image-20191202184133049" style="zoom: 40%;" /></center>

以跳数作为路径的权重

假设A最初处于停机状态，考虑A突然启动的情况

> 1. 一开始，B、C、D、E都不和A连通，所以都是“·”状态
>
> 2. 在第一次交换时， B 知道它左边的邻居到 A 的延迟为 0 。于是 B 在它的路由表中建立一个表项，说明 A 离它一跳远。所有其他的路由器仍然认为 A 是停机的。
> 3. 在接下去的交换中，C知道B有一条路径通向A，并且路径长度为1，所以它更新自己的路由表，指明它到 A 的路径长度为 2，但是 D 和 E 要到以后才能听到这个好消息。

从上面的例子可以看出

好消息扩散的速度是每交换一次往远处走一跳。如果一个网络中最长路径是 N 跳，那么经过 N 次交换之后，每个路由器都将知道新恢复的链路和路由器。



#### i.B 再来看一下对坏消息的反应

<center><img src="image-20191202185212842.png" alt="image-20191202185212842" style="zoom:40%;" /></center>

以跳数作为路径的权重

假设A最初处于启动状态，考虑A突然停机或者A和B之间链路断了的情况

> 1. 一开始，B、C、D、E都有通向A的链路
> 2. 在第一次信息交换时， B 没有听到来自 A 的任何信息。幸运的是， C 说“别担心，我有一条通向 A 的长度为 2 的路径”， B 并不怀疑 C 如何到达A。因此， B 认为它可以通过 C 到达 A，路径长度为 3 。在第一次交换之后， D 和 E 并不更新它们的 A 表项。
> 3. 在第二次信息交换时， C 注意到，它的每一个邻居都声称有一条通向 A 的长度为 3 的路径。它随机地挑选出一条，并且将它到A的距离更新为4。
> 4. 通过后续的交换，可以得到图中余下的记录历史。

逐渐地，所有的路由器都会趋向无穷大，但是所需交换的次数依赖于代表无穷大的数值。由于这样的原因，明智的做法是将无穷大设置为最长的路径+1

<a href="https://blog.csdn.net/tianlongtc/article/details/80261581">毒性逆转</a>

<br/>

## 2.5 链路状态路由[link state routing]

1. 发现它的邻居节点，并了解其网络地址。
2. 设置到每个邻居节点的距离或者成本度量值。
3. 构造一个包含所有刚刚获知的链路信息包。
4. 将这个包发送给所有其他的路由器，并接收来自所有其他路由器的信息包。
5. 计算出到每个其他路由器的最短路径。

<br/>

### i. 发现邻居

首先路由器要找到它的邻居。

路由器在每一条点到点线路上发送一个特殊的 HELLO 数据包，线路另一端的路由器应该返回一个应答说明自己是谁。

<br/>

注意，这里会将LAN看成一个pseudonode

<center><img src="image-20191224223921157.png" alt="image-20191224223921157" style="zoom:50%;" /></center>

### ii. 设置链路成本

需要每条链路以距离、成本、延迟度量。

测定延迟的方法是：

> 通过线路给另一边发送一个特殊的 ECHO 数据包，要求对方立即发回。通过测量往返时间再除以 2

<br/>

### iii. 构造链路状态包[link state packets]

<center><img src="image-20191203080937476.png" alt="image-20191203080937476" style="zoom: 33%;" /></center>

#### iii.A link state packets的结构

* 发送方的标识符
* 序号[Seq]
* 年龄[Age]
* 邻居列表，对于每个邻居，同时给出到这个邻居的延迟

#### iii.B 什么时候构造数据包

* 周期性地创建数据包
* 每当发生某些重要的事情时才创建数据包，比如一条线路断掉或者一个邻居节点停机...

<br/>

### iv. 分发链路状态包

链路状态路由算法最技巧的部分在于分发链路状态数据包

<br/>

思路：

泛洪将link state packets分发给所有路由器。为了控制flooding规模，每个数据包都有一个序号。

* Sequence number increments for each new pkt sent.

* Routers keeps track of all the (source router, sequence) pairs they see.

* If pkt is new, forward to lines except the incoming one

* Else, discard.

* If sequence number is lower than highest one seen so far, reject as obsolete.

* To include age of each packet and decrement it once per second. If age hits 0, discard the information.

<br/>

可能产生的问题与解决方法

* 如果序号绕回，可能会产生混淆。

  > 解决方案：使用一个 32 位的序号。

* 如果一个路由器崩溃了，那么它将丢失所有的序号记录表。如果它再从 0 开始， 那么，下一个数据包将被作为重复数据包而拒绝。

  再次，如果一个序号被破坏了，比如发送方发送的序号为 4，但是由于产生了 1 位错 误，所以接收方看到的序号是 65540 ，那么，序号从 5 到 65540 的数据包都将被当作过时数据包而拒绝接受，因为接收方认为当前的序号是 65540 。

  > 解决方案：在每个数据包的序号之后包含一个年龄[ age ]宇段， 并且每秒钟将年龄减 1 。当年龄字段的值被减到 0 时，来自路由器的该信息将被丢弃。通常情况下，每隔一段时间，比如说 10 秒，一个新的数据包就会到来：

<center><img src="image-20191203083626941.png" alt="image-20191203083626941" style="zoom:40%;" /></center>

### v. Computing the new routes

Once a router has accumulated a full set of link state packets, it can construct the entire subnet graph because every link is represented.

在路由器本地运行Dijkstra算法，来计算路由



<br/>

### LS-DV的比较

<center><img src="image-20191203084340533.png" alt="image-20191203084340533" style="zoom: 40%;" /></center>

## 2.6 层次路由

为什么要分层：

{%note info%}

随着网络规模的增长，路由器的路由表也成比例地增长．不断增长的路由表不仅消耗路由器内存，而且还需要更多的 CPU 时间来扫描路由表以及更多的带宽来发送有关的状态 报告。当网络增长到一定时可能会达到某种程度，此时每个路由器不太可能再为其他每一个路由器维护一个表项。所以 ， 路由不得不分层次进行

{%endnote%}

<br/>

Two level routing:

>* Every router knows all the details about how to route packets to destinations within its own region
>
>* but knows nothing about the internal structure of other regions.

<br/>

Multiple-level routing

> regions->clusters->zones->groups->...

<br/>

下面是一个two-level routing的例子，其中有5个区域

路由器1A的完整路由表[图b]有17个表项。

如果采用分级路由，则路由器如图c所示，路由表有7项

<center><img src="image-20191203084908185.png" alt="image-20191203084908185" style="zoom:35%;" /></center>

应该分成多少层

> 对于一个包含 N 个路由器的网络，最优的层数是 lnN，每个路由器所需的路由器表项是 elnN 个。

例如，考虑一个具有 720 个路由器的子网。如果没有分层，每个路由器需要 720 个路由表项

如果子网被分 成 24 个区域，每个区域 30 个路由器，那么每个路由器只需要 30 个本地表项，加上 23 个 远程表项，总共 53 个表项

如果采用三级层次结构，总共 8 个簇，每个簇包含 9 个区域，每个区域 10 个路由器，那么，每个路由器需要 10 个表项用于记录本地路由器， 8 个表项用于到同一簇内其他区域的路由， 7 个表项用于远程的簇，总共 25 个表项。



## 2.7 广播路由

广播：to send a packet to all destinations simultaneously.

广播的方法

* 让源机器给每一个目标单独发送一个数据包

  > 不仅浪费带宽，而且要求源机器拥有所有目标机器的完整地址

* 多目标路由[multidestination routing]

  > 每个数据包包含一组目标地址，或者一个位图，由该位图指定所期望到达的目标。
  >
  > 当一个数据包到达一个路由器时， 路由器检查数据包携带的所有目标，确定哪些输出线路是必要的[只要一条输出线路是到达至少一个目标的最佳路径，那么它就是必要的]。
  >
  > 路由器为每一条需要用到的输出线路生成一份该数据包新的副本，在这份副本中只包含那些使用这条线路的目标地址。实际上，原来的目标集合被分散到这些输出线路上。
  >
  > 在经过了足够多的跳数之后，每个数据包将只包含一个目标地址，因此可以被当作普通的数据包来对待。
  >
  > 
  >
  > 优点：当多个数据包必须遵循同样的路径时，其中一个数据包承担了全部的费用，而其他的数据包则是免费搭载。因此，网络带宽的利用率更高。
  >
  > 缺点：依然要求源端知道全部的目标地址，对于路由器来说，要确定从哪些线路转发多目标数 据包的工作量太大，尤其是处理多个不同的数据包时。

* flooding

* 逆向路径转发[reverse path forwarding]

  当一个广播数据包到达一个路由器时，路由器检查它到来的那条线路是否正是通常用来给广播源端发送数据包用的那条线路。

  * 如果是，则该广播数据包是沿着最佳路径被转发过来的，因而是到达当前路由器的第一份副本。该路由器将该数据包转发到除了到来的那条线路之外的所有 其他线路上。
  * 如果广播数据包是从其他任何一条并非首选的到达广播源的线路入境的话，该数据包被当作一个可能的重复数据包而丢弃。

  <center><img src="image-20191205155744495.png" alt="image-20191205155744495" style="zoom:40%;" /></center>

  <br/>

<center><img src="image-20191205154558002.png" alt="image-20191205154558002" style="zoom:50%;" /></center>

  >(a)部分显示了一个网络， (b )部分显示了该网络中路由器 ***I*** 的一棵汇集树，图(c部分显示了逆向路径算法是如何工作的。
  >
  >1. 在第一跳， I 发送数据包给 F、 H、 J 和 N，如树中第二行所示。这些数据包中的每一个都是在通向 I 的首选路径（假定首选的路径都沿着汇集树)到来的， 这点用字母外面加一个圆圈来表示。
  >2. 在第二跳，共产生了 8 个数据包，其中，在第一跳接收到数据包的路由器各产生 2 个数据包。结果，所有这 8 个数据包都到达了以前没有访问过的路由器，其中 5 个是沿着首选线路到来的。
  >3. 在第三跳所产生的 6 个数据包中，只有 3 个是沿首选路径（在 C、 E 和 K）到来的，其他的都是重复数据包。在经过五跳和 24 个数据包以 后，广播过程终止。相比之下，如果完全沿着汇集树的话，只需要 4 跳和 14 个数据包。

* spanning tree

改进了逆向路径转发。

每个路由器都知道它的哪些线路属于生成树，它就可以将一个人境广播数据包复制到除了该数据包到来的那条线路之外的所有***<u>生成树线路</u>***上。

> 和逆向路径转发相比，逆向路径转发并不只转发到生成树线路，还会转发到其他线路，这些数据包会被认为是可能的重复数据包而被丢弃；而生成树算法没有丢弃问题

<br/>

## 2.8 组播路由[multicast]

to send messages to well-defined groups that are numerically large in size but small compared to the network as whole.

<center><img src="image-20191205155836717.png" alt="image-20191205155836717" style="zoom: 40%;" /></center>

通过修建广播生成树把不通往组成员的链路从树中删掉。修建结果得到的是一颗有效的组播生成树。

<br/>

生成树的修建方法：

* link state: 

* DV

* core-based trees

  > 所有路由器都同意某个路由器作为根，这个根称为核心（ core ）或会聚点（rendezvous point )，然后每个成员通过给根发送一个数据包来建立这棵树。
  >
  > 
  >
  > 为了把数据包发送到这个组，发送者把数据包发给核心：当数据包到达核心后，它再被沿着树往下转发

## 2.9 选播路由[Anycast routing]

在选播方式下，数据包被传 递给最近的一个组成员。

### 为什么要选播？

> 有时，对客户最重要的是获得正确的信息而不是与哪个节点取得联系，任何节点都可以， 只要它能提供所需的服务

LS, DV算法可以用来生产选播路由。

假设我们要选播数据包到组1的成员，该组成员都将被赋予 一个组地址1而不是一个个独立的地址。距离矢量路由将像往常一样分发向量，并且节点将只选择到目的地1的最短路径。这将导致数据包被发送到目的地1的最近实例。

此过程之所以能正常工作是因为路由协议认为节点1的实例都是同一个节点，

<center><img src="image-20191224225002633.png" alt="image-20191224225002633" style="zoom:50%;" /></center>

总结

* unicast：源给单个目标节点发送

* broadcast: 源给所有目标节点发送

* multicast: 源给一组目标节点发送

* Anycast: a packet is delivered to the nearest member of a group.



## 2.10 Routing for the mobile hosts

<center><img src="image-20191224231029464.png" alt="image-20191224231029464" style="zoom:50%;" /></center>

## 2.11 Routing in Ad Hoc Networks

In ad-hoc network, the network topology may be changing all the time.

***AODV***: 

> Ad hoc On-demand Distance Vector. The most popular routing algorithm for Ad Hoc Networks.

Consider the problem: A wants to send to I

> (a) A broadcasts, attempting to reach I.
>
> (b) B and D received, establish reverse paths to A.
>
> (c) B and D broadcast. C, F, and G received, establish reverse paths to B and D.
>
> (d) C,F,G broadcast. E, H, and I received (I reached!).
>
> I sends back a REPLY along the reverse path. G and D establish to path to I when they process REPLY.

<center><img src="image-20191224231601828.png" alt="image-20191224231601828" style="zoom:50%;" /></center>

