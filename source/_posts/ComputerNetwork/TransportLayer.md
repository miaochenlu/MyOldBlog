---
title: Computer network--传输层
date: 2019-12-09 15:15:44
tags:  Computer Network
categories:  Computer Network
index_img: /img/image-20191203144306819.png
---


# 1. The Transport Service

* transport layer services
  * To provide efficient, ***reliable,*** and cost effective service to its users, normally processes in the application layer.
  * To make use of the services provided by the network layer.

* the transport entity

  * the hardware and/or software within the transport layer that does the work.

<center><img src="image-20191126095920516.png" alt="image-20191126095920516" style="zoom:35%;" /></center>

## 1.1 网络层和传输层的区别与联系

> 考虑有两个家庭，一家位于美国东海岸，一家位于美国西海岸，每家有12个孩子。东海岸的孩子们是西海岸家庭孩子们的堂兄弟姐妹。这两个家庭的孩子们喜欢彼此通信，每个人每周要给每个堂兄弟姐妹写一封信，每封信都用单独的信封通过传统的邮政服务发送。因此，每个家庭每周向另一家庭发送144封信。（如果他们有电子邮件的话，这些孩子可以省不少钱！）每个家庭有一个孩子负责收发邮件，西海岸家庭是Ann而东海岸家庭是Bill。每周Ann去她所有的兄弟姐妹那里收集邮件，并将这些邮件交到每天到家门口的邮政运输车上。当信件到达西海岸家庭时，Ann也负责将信件发到她的兄弟姐妹手上，东海岸家庭中Bill也负责类似工作。

我们可以做下面这样的类比:

```
应用层报文　　＝　信封上的字符
进程　　　　　＝　堂兄弟姐妹
主机(端系统)　＝　家庭
运输层协议　　＝　Ann和Bill
网络层协议　　＝　邮政服务
```

> 在这个例子中，网络层就像邮递员，而运输层就像Ann和Bill。现在我们再来理解上面的这句话:**网络层提供了主机之间的逻辑通信，而传输层为不同主机上的进程之间提供了逻辑通信。**邮政服务知识将信件送到指定的家庭，它不会将信件分发到家庭的具体成员手中。这个分发的工作则是由Ann和Bill提供的。值得注意的是Ann和Bill都是在各自的家里进行工作的，他们没有参与任何中间邮件中心对邮件进行分拣的工作，也没有将邮件从一个邮件中心送到另一个邮件中心。相应的，**运输层协议只工作在端系统中**。在端系统中，运输层协议将来自应用进程的的报文移动到网络边缘(即网络层)，但对有关这些报文在网络层中如何移动却不做任何规定。
> 由这个例子，我们可以做出如下概括:**网络层提供了不同端系统的数据交付服务，而传输层则将这种不同主机间的交付拓展为了运行在不同端系统上的两个进程之间的交付。**进程到进程之间的数据交付是传输层提供的最基本的服务之一。

{:.success}

传输层为应用进程之间提供端到端的逻辑通信（但网络层是为主机之间提供逻辑通信。传输层还要对收到的报文进行差错检测。

<center><img src="image-20191212232948465.png" alt="image-20191212232948465" style="zoom:40%;" /></center>
<center><img src="image-20191212233116016.png" alt="image-20191212233116016" style="zoom:50%;" /></center>

<br/>

## 1.2 Hypothetical primitives

<center><img src="image-20191126100224177.png" alt="image-20191126100224177" style="zoom:30%;" /></center>

<br/>

用segment表示传输实体间发送的消息[一些老协议使用了更加笨拙的名称一一传输协议数据单元。TPDU, Transport Protocol Data Unit]

<center><img src="image-20191209001317925.png" alt="image-20191209001317925" style="zoom: 30%;" /></center>

***How to use these primitives for an application?***

<center><img src="image-20191209000910981.png" alt="image-20191209000910981" style="zoom:35%;" /></center>

***Connection establishment and connection release***

<center><img src="image-20191209002015904.png" alt="image-20191209002015904" style="zoom:35%;" /></center>

<br/>

# 2. Elements of Transport Protocols

## 2.1 传输层协议与数据链路层协议

传输协议在有些方面类似数据链路协议。这两种协议都要处理<u>错误控制、 顺序性和流量控制以及其他一些问题。</u>

然而，两者之间也存在着重大的差别。这些差别是因为这两种协议的运行环境不同而造成的。

<center><table>
  <tr>
    <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191209134511204.png" alt="image-20191209134511204" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>数据链路层环境</h4>     
    </div>     
    <p>两台路由器通过一条有线或者无线物理信道直接进行通信</p>  
  </div>
</div></td>
  <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191209134550181.png" alt="image-20191209134550181" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>传输层</h4>     
    </div>     
    <p>该物理信道被整个网络所替代                             </p>  
  </div>
 </div></td>
  </tr>
</table></center>




* Addressing: 
  * 在点到点链路上，路由器不必指定它要与哪一台路由器进行通话，因为每条出境线路直接通向一台特定的路由器。
  * 而在传输层，必须显式地指定接收方的地址。
* Connection establishment:
  *  在一条线路上建立一个连接的过程非常简单，另一端总是在那里（除非它崩溃了才不在那里）。两边都不需要做很多事情。如果因发生错误而消息没有被确认，可以再次重发。
  *  而在传输层中，初始的连接建立过程非常复杂。
* Storage capacity:
  * 数据链路层和传输层之间的另一个（非常恼人的）差别是，网络存在着潜在的存储容量。当路由器发送一帧到一条链路上后，该帧可能到达对方也可能丢失，但是它不可能先蹦跄一会儿，再躲到远处一个角落中，然后在其他数据包发送出去很久后突然又冒了出来。
  * 如果网络使用数据报技术，即网络内部的路由是独立进行的，那么就存在一个不可忽略的概率：一个包可能选取了风景优美的路线，一路观光姗姗来迟到达目的地，这将扰乱预期的接收顺序，甚至它的重复数据包都已经到达目的地它还没到呢。网络具有的这种延迟和重复数据包的特性所产生的后果有时是灾难性的，因此要求使用特殊的协议，以便正确地传输信息。 
* Buffering:
  * 这两层都需要缓冲和流量控制，但是，由于传输层上存在着大量并且数量可变的连接，而且由于连接之间的相互竞争造成连接的可用带宽上下波动，因此需要一种不同于数据链路层使用的方法。



## 2.2 寻址

端口

{%note success%}

当一个应用进程希望与另一个远程应用进程建立连接时，它必须制定要连接到哪个应用进程上。通常使用的方法是为那些能够监听连接请求的进程定义相应的传输地址。将这些端点称为***<u>端口</u>***[port]

{%endnote%}

传输服务访问点[transport service access point]

{%note success%}

传输层的一个特殊端点

{%endnote%}

网络访问服务点[network service access point]

{%note success%}

网络层上的端点[即网络层地址]

{%endnote%}

接下来来看一下过程

应用进程（包括客户和服务器） 可以将自己关联到）1个本地 TSAP 上，以便与一个远程 TSAP 建立连接。这些连接运行在 每台主机的 NSAP 之上，如图所示。在有些网络中，每台计算机只有一个 NSAP ，但是可能有多个传输端点共享此 NSAP

<div class="item">   
  <div class="item__image"> 
    <img  src="image-20191209134703993.png" alt="image-20191209134703993" style="zoom:40%;" /> 
  </div>   
  <div class="item__content">     
  <div class="item__header">       
    <h4>Addressing过程</h4>     
  </div>     
  <div class="item__description">     
    <p>1. Host 2 上的邮件服务器进程将自己关联到 TSAP 1522 上[例如调用listen]，等待入境连接请求的到 来。</p>
    <p>2. Host 1 上的应用进程希望发送一个邮件消息，所以它把自己关联到 TSAP1208 上，并且发出一个 CONNECT 请求。该请求消息指定主机 1 上的 TSAP 1208 作为源，主机 2 上 的 TSAP 1522 作为目标。这个动作最终导致在应用进程和服务器之间建立了一个连接。</p>
    <p>3. 应用进程发送邮件消息。 </p>
    <p>4. 作为响应，Host 2上的邮件服务器表示它将传递该消息。</p>
    <p>5. 传输连接被释放。 请注意，在主机 2 上很可能还有其他的服务器被关联到其他的 TSAP 上，它们也在等 待经过同一个 NSAP 到达的入境连接请求。</p>
  </div>   
  </div> 
</div>




<br/>

### A. host 1上的用户进程如何知道邮件服务器被关联到了TSAP 1522 上

1. 固定TSAP[将某些服务器永久关联到一些端口上]

   http=TCP:80

   https=TCP:443

   RDP=TCP:3389

   ftp=TCP:21

   SMTP=TCP:25

   DNS=UDP:53

   <img src="image-20200105235129774.png" alt="image-20200105235129774" style="zoom:50%;" />

2. 端口映射。存在－个称为端口映射器（ portmapper）的特殊进程。

   * 为了找到一个给定服务名字相对应的 TSAP 地址，用户需要***<u>与端口映射器[server上？]建立一个连接</u>***
   * 然后，用户通过该连接发送一条消息指定它想要的服务名字[如"BitTorrent"]，端口映射器返回相应的 TSAP 地址。
   * 之后，用户释放它与端口映射器之间的连接， 再与所需的服务建立一个新的连接。

{%note info%}

当一个新的服务被创建时，它必须向端口映射器注册，把它的服务名字（通常是一个 ASCII 字符串〉和 TSAP 告诉端口映射器。端口映射器将该信息记录到它 的内部数据库中，所以，以后当用户查询时，它就知道答案了。端口映射器的功能类似于电话系统中的查号操作员, 提供的是从名字到电话号码之间的映射关系。

{%endnote%}





{%note success%}

用`netstat -an`命令查看应用监听的端口

`netstat -n`查看建立的会话

{%endnote%}

<center><img src="image-20191213095702465.png" alt="image-20191213095702465" style="zoom:50%;" /></center>

{%note success%}

`netstat -nb`查看建立会话的进程

{%endnote%}

{%note success%}

`telnet 192.168.80.100 3389`测试远程计算机某个端口是否打开

{%endnote%}

### B. Many server processes will be used rarely and it is wasteful to have them active

<br/>

<div class="item">   
  <div class="item__image"> 
    <img  src="image-20191209135340835.png" alt="image-20191209135340835" style="zoom:40%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        initial connection protocol
      </h4>
    </div>
  <div class="item__description">     
    <p>每台希望向远程用户提供服务的机器有一个特殊的进程服务器（ process server ）充当那些不那么频繁使用的服务器的代理。它在同一时间监听一组端口，等待连接请求的到来。一个服务的潜在用户发出一个连接请求，并指定他们所需服务的 TSAP 地址。如果该 TSAP 地址上没有服务器正等着，则他们得到一条与进程服务器的连接，如图a所示。</p>
    <p>在获取入境请求后，进程服务器派生出被请求的服务器，允许该服务器继承与用户的现有连接。新服务器完成要求的工作，而进程服务器可回去继续监听新的连接要求，如图b所示。这种方法只适用于服务器可按需创建的场合。</p>
  </div>   
  </div> 
</div>




## 2.3 连接建立

一个巨大的问题

{%note warning%}

***delayed duplicates problem***

{%endnote%}

> 用户建立了一个与银行的连接，并且发送消息告诉银行把一大笔钱转移到一个并不是完全值得信赖的人的账户。不幸的是，数据包决定采取一条到目的地的风景路径，去探索网络的某个偏僻角落。然后，发送端超时，并且再次发送这些消息。这一次，数据包采取了一条最短路径，并且很快被交付给接收端，所以发送端释放连接。不幸的是，最终原先的那批数据包终于从某个隐藏处冒了出来，并到达目的地，要求银行建立一个新的连接和汇款（再次）。银行没有办法得知这些是重复请求。它必须假设 这是第二个并且独立的交易请求，因此再次转钱。

{%note success%}

Desired behavior

{%endnote%}

<center><img src="image-20191210093758100.png" alt="image-20191210093758100" style="zoom:30%;" /></center>

### A. Solution 1

{%note info%}

use sequence numbers

{%endnote%}

```python
packet arrival:
  if(packet.seq see before)
  	discard packet
```

`problems`

* Possible wrap around of seqno

* Client/Host or Server may crash

<center><img src="image-20191210094719473.png" alt="image-20191210094719473" style="zoom:30%;" /></center>

Problem : How to differentiate ***<u>delay duplicate</u>*** and ***<u>new pkt with wrapped around seqno</u>***?

Idea: ***<u>use time</u>***

Two assumptionsthat simplifies the problem

1. Time for seqno wrap around (T1) is typically large if, e.g., seqno is 32 bits long.

2. If pkt delays a relatively short time (T2<T1), we can use time to differentiate

   the two.



### B. Solution 2

```python
if (packet.seq see before and time is short (<T))
	discard packet
if (packet.seq see before and time is large (>T))
	accept packet
```

<center><img src="image-20191126102346530.png" alt="image-20191126102346530" style="zoom: 33%;" /></center>

#### How to realize

1. Restrict packet lifetime

   Packet lifetime can be restricted to a known maximum using one of the following techniques

   * Restricted subnet design.

   * Putting a hop counter in each packet. 将跳计数器 初始化为某个适当的值，然后每次数据包被转发的时候该跳计数器减一。网络协议简单丢弃掉那些跳计数器变成零的数据包

   * Timestamping each packet (router synchronization required) 每个数据包携带它的创建时间，路由 器负责丢弃那些年限超过某个预设值的数据包。后一种方法要求同步所有路由器的时钟.

   引入周期T, 他是数据报实际最大生存期的某个不太大的倍数：$$T=n\times (pkt\;lifetime)$$

   It is impossible to receive a delay duplicate after T

2. use time-of-day clock[日时钟] to limit sending rate

   核心是源端用序号作为段的标签，使得该段在 T 秒内不被重用。 T 的大小以及数据包速率（数据包／秒）确定了序号的大小。这样，在任何给定的时间内只能出现一个给定序号的数据包。

   为了解决一台机器崩溃之后丢失所有内存的问题，一种可能的解决方法是要求传输实体在机器崩溃之后空闲 T 秒。这段空闲期将确保所有老的段全部死掉，因而发送端可以启用任何一个序号值。然而，在复杂的互联网络中， T 可能会很大，因此这种策略不具吸引力。

   相反， Tomlinson 建议每台主机都配备一个日时钟（time-of-day clock ）。不同主机上的

   * 时钟不需要同步。假定每个时钟均采用了***<u>二进制计数器</u>***的形式。该计数器以统一的时间间隔递增自己，
   * 计数器的位数必须等于或者超过序号的位数。
   * 即使***<u>主机停机时钟也不停下</u>***，它将持续不停地运行。

   当建立一个连接时，时钟的低 k 位被用于同样是 k位的初始序号。每个连接都从一个完全不同的初始序号开始对它的段进行编号。序号空间应该足够大，以便当序号回绕时，原来那些具有老序号的段都己经消失。



***forbidden region***

<div class="item">   
  <div class="item__image"> 
   <img src="image-20191211220455615.png" alt="image-20191211220455615" style="zoom:50%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Forbidden region
      </h4>
    </div>
  <div class="item__description">     
    <p>Assume the same bits of seqno and time-of-day clock</p>
    <p>
      Two time instants, t and t', t << t'
    </p>
    <p>
      If we allow legitimate reuse of the same seqno at t and t', we should ensure that t'-t >T </p>
  </div>   
  </div> 
</div>


{%note warning%}

为了防止序号进入forbidden region，我们必须兼顾两个方面。

{%endnote%}

<center><img src="image-20191211222453374.png" alt="image-20191211222453374" style="zoom:40%;" /></center>

> * 如果一台主机在一个新打开的连接上发送得太快，则实际序号与时间的曲线可能比初始序号与时间的曲线还要陡，导致序号进入forbidden region。
>   * 为了防止这种情况出现，每个连接上的最大数据速率是每个时钟滴答一次发送一段。这同时也意味着在机器崩溃并重启动之后， 传输实体必须等待时钟滴答，才能打开一个新的连接，以免同样的序号被使用两次。这两 点都倾向于使用短的时钟滴答 (1 微秒甚至更短）。但是相对序号来说时钟不能滴答得太快。 假设时钟速率为 C，序号空间大小为 S，则有 S/C>T，才能使得序号不至于回绕得太快。
> * 如果以任何低于时钟速率的数据率发送，实际使用的序号与时间之间的曲线最终会从左边进入到禁止区域中。实际序号曲线的斜度越大，则这种事件发生得越晚。
>   * 为了避免这种情况发生，应该限制连接序号递增的速度不能太慢（或连接可能会持续多久）。



<center><img src="image-20191126102810011.png" alt="image-20191126102810011" style="zoom: 33%;" /></center>

>看一道例题
>
>In a network whose max segment is 128 bytes, max segment lifetime is 30 sec, and has 8 bit sequence numbers, what is the maximum data rate per connection?
>
>**Answer**
>
>这个包是0号，则在30sec里面可以从0号发到255号，一共256个包
>
>number of bits=256 × 128 × 8 = 262,144 bits
>The data rate =262,144 bits /30 sec=8738 bps. 





以上内容解决了delay duplicate packets的问题。接下来将其投入应用

### Three-way handshake

<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211230251099.png" alt="image-20191211230251099" style="zoom:45%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
       Connection Establishment(1)
      </h4>
    </div>
  <div class="item__description">  
    <p> a. 主机1选择一个序号x，并且发送一个包含x的CONNECTION REQUEST段给主机2。</p>
    <p>
      b. 主机2回应一个ACK段作为对x的确认，并且宣告它自己的初始序号y。
    </p>
    <p>
      c. 最后，主机1在它发送的第一个数据段中，对主机 2 选择的初始序号进行确认。</p>
  </div>   
  </div> 
</div>



<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211230545824.png" alt="image-20191211230545824" style="zoom:35%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Connection Establishment(2)
      </h4>
    </div>
  <div class="item__description">  
    <p> a. 第一个段是一个旧连接上被延迟了的重复 CONNECTION REQUEST。该段到达主机 2, 而主机 1 对此并不知情。</p>
    <p>
      b. 主机 2 对这个段的回应是给主机 1 发送一个 ACK 段，其效果相 当于验证主机 1 是否真的请求建立一个新的连接。
    </p>
    <p>
      c. 当主机 1 拒绝了主机 2 的连接建立请求后，主机 2 就意识到被一个延迟的重复段所欺骗了，于是放弃连接。这样 p 一个延迟的重 复段没有造成任何伤害。</p>
  </div>   
  </div> 
</div>

<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211230935460.png" alt="image-20191211230935460" style="zoom:35%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Connection Establishment(3)
      </h4>
    </div>
  <div class="item__description">  
    <p> a. 第一个段是一个旧连接上被延迟了的重复 CONNECTION REQUEST。该段到达主机 2, 而主机 1 对此并不知情。</p>
    <p>
      b. 主机 2 对这个段的回应是给主机 1 发送一个 ACK 段，其效果相当于验证主机 1 是否真的请求建立一个新的连接。
    </p>
    <p>
      c. 当第二个延迟的段到达主机 2 时，主机 2 注意到确认的是 z 而不是 y，这个事实告诉主机 2 这也是一个老的重复数据包。</p>
  </div>   
  </div> 
</div>




## 2.4 连接释放

<center><table>
  <tr>
    <td><div class="card">    
  <div class="card__content">     
    <div class="card__header">       
      <h4>Asymmetric release</h4>     
    </div>     
    <p>当一方挂机后，连接就被中断了</p> 
    <p>
      很冒失，可能导致数据丢失
    </p>
  </div>
</div></td>
  <td><div class="card">   
  <div class="card__content">     
    <div class="card__header">       
      <h4>Symmetric release</h4>     
    </div>     
    <p>把连接看成两个独立的单向连接，要求单独释放每一个单向连接．                           </p>  
  </div>
 </div></td>
  </tr>
</table></center>


Two army problem, 完美的释放是不可能的

也采用三次握手协议

<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211234832859.png" alt="image-20191211234832859" style="zoom:50%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Normal case of three-way handshake
      </h4>
    </div>
  <div class="item__description">  
    <p> a. 一个用户发送一个 DR (DISCONNECTION REQUEST ）段来启动释放连接过程</p>
    <p>
      b. 当该段到达对方，接收端也发回一个DR段，并启动一个计时器，设置计时器的目的是为了防止它的DR丢失。
    </p>
    <p>
      c. 当这个 DR 到达时，最初的发 送端发回一个 ACK 段，并且释放连接</p>
    <p>
      d. 当 ACK 返回后，接收端也释放连接。释放 一个连接意味着传输实体将有关该连接的信息从它的内部表（记录了所有当前己打开的连 接）中删除，并且通知该连接的所有者（传输用户）
    </p>
  </div>   
  </div> 
</div>



<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211235148092.png" alt="image-20191211235148092" style="zoom:50%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Final ACK lost
      </h4>
    </div>
  <div class="item__description">  
    <p> 如果最后的 ACK 段被丢失, 可以通过一个计时器来补救。当计时器超时，无论如何连接都要被释放。</p>
  </div>   
  </div> 
</div>

<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211235320953.png" alt="image-20191211235320953" style="zoom:50%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Response lost
      </h4>
    </div>
  <div class="item__description">  
    <p> 现在考虑第二个 DR 被丢失的情形。发起释放连接的用户接收不到期望的响应，所以 它将超时，于是再次尝试释放连接。</p>
  </div>   
  </div> 
</div>

<div class="item">   
  <div class="item__image"> 
  <img src="image-20191211235440547.png" alt="image-20191211235440547" style="zoom:50%;" />
  </div>   
  <div class="item__content"> 
    <div class = "item__header">
      <h4>
        Response lost and subsequent DRs Lost
      </h4>
    </div>
  <div class="item__description">  
    <p> 但由于丢失段的原因，所有重传 DR 的尝试都失败。经过 N 次重试之后，发送端放弃了，并且释放连接。同时，接收端超时，于是退出连接。</p>
  </div>   
  </div> 
</div>


{%note success%}

Automatic disconnect rule

{%endnote%}

* If no segments have arrived for a certain number of seconds, the connection is then automatically disconnect.

* Thus, if one side ever disconnects, the other side will detect the lack of activity and also disconnect.

<br/>

## 2.5 error control and flow control

* Error detection:

  * The link layer checksum protects a frame while it crosses a single link.

  * The transport layer checksum protects a segment while it crosses an entire network path. It is an end-to-end check, which is not the same as having a check on every link.

End-to-end argument: transport check is essential for correctness while link layer check is valuable for performance.

<br/>

## 2.6 多路复用

* ***多路复用***（或多个会话共享连接、虚电路和物理链路）在网络体系结构的不同层次发挥着作用。在传输层，有几种方式需要多路复用。
  * 如果主机只有一个网络地址可用， 则该机器上的所有传输连接都必须使用这个地址。当到达了一个段，必须有某种方式告知 把它交给哪个进程处理。图中， 4 个独立的传输连接都使用了相同的网络连接（即 IP 地址）到达远程主机。

* ***逆向多路复用***，假设一台主机有多条网络路径可用。如果用户需要的带宽和可靠性比其中任何一条路径所能提供的还要多，那么一种解决办法是以轮询的方式把一个连接上的流量分摊到多条网络路径。



<center><img src="image-20191212000741805.png" alt="image-20191212000741805" style="zoom:50%;" /></center>

<br/>



# 3. Congestion Control

# 4. The Internet Transport Protocols：UDP

TCP 与 UCP的应用场景

* TCP: 需要将传输的文件分段 建立会话，可靠传输，流量控制  QQ传文件
* UDP： 一个数据包就能完成数据通信 不分段 不需要建立会话 不需要流量控制 不可靠传输 QQ聊天

<br/>

## 4.1 UDP overview

<center><img src="image-20191213103709184.png" alt="image-20191213103709184" style="zoom:50%;" /></center>

UDP length是header和payload两部分的总长度，以byte为单位

<center><img src="image-20191213103006241.png" alt="image-20191213103006241" style="zoom:50%;" /></center>

在计算检验和时，临时把“伪首部”和 UDP 用户数据报连接在一起。伪首部仅仅是为了计算检验和。

<h4>UDP checksum计算过程</h4>    

<img src="image-20191213103042682.png" alt="image-20191213103042682" style="zoom:40%;" /><img src="image-20191213110622335.png" alt="image-20191213110622335" style="zoom:40%;" />




## 4.2 RPC[Remote Procedure Call]

客户-服务器 RPC 是 UDP 被广泛应用的一个领域，

{%note success%}

RPC (Remote Procedure Call) allows programs to call procedures located on remote hosts.

{%endnote%}

> RPC 背后的思想是尽可能地使一个远程过程调用看起来像本地过程调用一样。
>
> 在最简单的形式中，为了调用一个远程过程，客户程序必须绑定（链接）到一个小的库过程，这 个库过程称为客户存根[client stub]，它代表了客户地址空间中的服务器过程。类似地，服务器需要绑定到一个称为服务器存根[server stub]的过程。正是这些过程，隐藏了客户机不在本地调用服务器的事实。

<center><img src="image-20191213122858021.png" alt="image-20191213122858021" style="zoom:50%;" /></center>

### A. 执行RPC的实际步骤

1. client调用client stub，参数入栈
2. client stub将参数封装到一个消息中，然后通过系统调用发送消息。参数封装的过程称为列集[marshalling]
3. kernel将消息从client machine发到server machine
4. kernel将入境数据包传递给server stub
5. server stub利用散集[unmarshaled]调用的参数调用服务器过程
6. 调用的结果沿着相反的方向按照同样的路径传递。



### B. RPC陷阱

* The use of pointer parameters. 

* Some problems for weakly typed languages (The length of an array).

* It is not always possible to deduce the types of the parameters, not even from a formal specification or the code itself. [printf]

* The use of global variables.



## 4.3 RTP[Real-time transport protocol]

UDP 的另一个应用领域是实时多媒体应用

{%note success%}

RTP is a transport protocol realized in the application layer.

{%endnote%}

### RTP的基本功能：

{%note info%}

RTP is to multiplex several real-time data streams onto a single stream of UDP packets and unicast or multicast the UDP packets.

{%endnote%}

### RTP格式包含了几种有助于接收端处于流媒体信息的特性

* Each RTP packet is given a <u>number</u> one higher than its predecessor. RTP has no flow control, no error control, no acknowledgements, and no retransmission support.
* Each RTP payload may contain multiple samples and they can <u>be coded any way that the application wants</u>. For example a single audio stream may be encoded as 8-bit PCM samples at 8kHz, delta encoding, predictive encoding  GSM encoding, MP3, and so on.
* RTP allows timestamping.

<center><img src="image-20191213124846490.png" alt="image-20191213124846490" style="zoom:50%;" /></center>

### RTCP[Realtime Transport Control Protocol]

#### A. RTCP一些特性

* Does not transport any data

* To handle feedback, synchronization, and the user interface

* To handle interstream synchronization.

* To name the various sources.

#### B. 带有缓冲和抖动控制的播放

**Smoothing the output stream by buffering packets**

一旦媒体信息到达接收端，它必须在合适的时间播放出来。 一般情况下，这个时间不是 RTP 包到达接收端的时间，因为数据包通过网络传输所需要的时间略有不同。即使在发送端，数据包以正确的时间间隔被依次注入网络，它们到达接收端的相对时间也不同。这种延迟的变化称为***<u>抖动[jitter</u>]***。即使是少量的数据包抖动，如果简单地按它到达的时间播放出来，也可能导致媒体成品发散，如抖动的画面帧和难以辨认的音频。

这个问题的解决办法是在接收端播放媒体之前对其进行缓冲，以此来减少抖动。

<center><img src="image-20191213130200893.png" alt="image-20191213130200893" style="zoom:50%;" /></center>

数据包 8 被延迟得太多，当播放到它时还不可用。这种情况下可以有两种选择。

* 第一种是跳过数据包 8 继续播放后续的数据包。
* 另一种方法是停止播放，直到数据包 8 到达，此时在播放的音乐或电影中就会出现一个恼人的间隙。

<center><img src="image-20191213130334229.png" alt="image-20191213130334229" style="zoom:50%;" /></center>

# 5. TCP

## 5.1 Overview

TCP[Transmission Control Protocol] provides a ***<u>reliable end-to-end byte stream</u>*** over an unreliable internetwork.

TCP must furnish (提供) the reliability that most users want and that IP does not provide.

* TCP 是面向连接的传输层协议。[三次握手]

* 每一条 TCP 连接只能有两个端点(endpoint)，每一条 TCP 连接只能是点对点的[一对一]。 

* 提供可靠交付的服务。

* TCP 提供全双工通信。[同时收发]
* 面向字节流。 



## 5.2 TCP: The Service Model

* TCP 服务由发送端和接收端创建一种称为套接字（ socket ）的端点来获得

{%note error%}

Socket = IP地址 + port

{%endnote%}

* TCP连接是***<u>全双工</u>***的[必须全双工，一方给另一方发送消息，另一方应该回复“嗯嗯嗯，我在听”]
* TCP连接是一个字节流，而不是消息流

> 如果发送进程将 4 个 512 字节的数据块写到一个 TCP 流中，那么这些数据有可能按 4 个 512 字节块、2 个 1024 字节块、 1 个 2048 字节块或者其他的方式被递交给接收进程。 接收端不管多么努力尝试，都无法获知这些数据被写入字节流时的单元大小。

<div class="card">   
  <div class="card__image">     
  <img class="image__lg" src="image-20191212233707007.png" alt="image-20191212233707007" style="zoom:67%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>TCP面向byte stream的概念</h4>     
    </div>     
  </div>
</div>


## 5.3 TCP header

每个TCP segment的起始部分是一个固定格式的 ***<u>20 字节</u>***头。固定的头部之后可能有头的选项。如果该数据段有数据部分的话，那么在选项之后是最多可达 65535-20-20=65495 个字节的数据，这里的第一个 20 是指 IP 头，第二个 20 指 TCP 头。 没有任何数据的 TCP 段也是合法的，通常被用作确认和控制消息。



<center><img src="image-20191215161710416.png" alt="image-20191215161710416" style="zoom:50%;" /></center>

* sequence number

> 是TCP数据部分的第一个字节在整个文件中所处第几个字节

<center><img src="image-20191215161855868.png" alt="image-20191215161855868" style="zoom:50%;" /></center>

* acknowledge number

> 期待发送者下一次发送的数据

<center><img src="image-20191213151314239.png" alt="image-20191213151314239" style="zoom:50%;" /></center>

* TCP header length

> TCP header的长度，以word[4个bytes]为单位

<div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215163422275.png" alt="image-20191215163422275" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>CWE&ECE</h4>     
    </div>     
    <p>CWR和ECE用作拥塞控制的信号。</p>
    <p>当receiver收到了来自网络的拥塞指示后，就设置ECE以便给sender发ECN-Echo[Explicit Congestion Notification-Echo]信号，告诉发送端放慢发送速率。</p>
    <p>TCP sender设置CWR，给 TCP 接收端发 CWR 信号</p>
    <p>receiver知道sender己经放慢速率，不必再给sender发 ECN-Echo 信号</p> 
  </div>
 </div>

<center><table>
  <tr>
    <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215162158221.png" alt="image-20191215162158221" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>ACK</h4>     
    </div>     
    <p>对上一次发过来的数据进行确认, 如果设置为1表示acknowledge number字段有效。如果为0，说明acknowledge number字段无效，没有确认信息</p>
		<p>因此发送建立会话请求时,ACK=0, 因为之前没有要确认的数据</p>  
  </div>
</div></td>
  <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215162258403.png" alt="image-20191215162258403" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>SYN</h4>     
    </div>     
    <p>表示要建立会话[to used to establish connections].比如我的电脑想要和某个网站建立连接 </p>
		<p><u>SYN for CONNECTION REQUEST, SYN+ACK for CONNECTION ACCEPTED.</u></p>  
  </div>
 </div></td>
  <td><div class="card">   
  <div class="card__image">     
  <img class="image"  src="image-20191215162320667.png" alt="image-20191215162320667" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>FIN</h4>     
    </div>     
    <p>释放连接</p>
  </div>
    </div></td>
  </tr>
</table></center>



<table>
  <tr>
    <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215162053100.png" alt="image-20191215162053100" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>URG</h4>     
    </div>     
    <p>在sender,紧急，这个数据包不要等待前面缓存发送了，插到第一个，马上给我发 bjl</p> 
  </div>
 </div></td>
    <td><div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215162216454.png" alt="image-20191215162216454" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>PSH</h4>     
    </div>     
    <p>在receiver,紧急，别处理缓存里的其他数据了，先处理这个包，bjl</p> 
  </div>
 </div></td>
  </tr>
</table>

<div class="card">   
  <div class="card__image">     
  <img class="image" src="image-20191215162237847.png" alt="image-20191215162237847" style="zoom:50%;" />
  </div>   
  <div class="card__content">     
    <div class="card__header">       
      <h4>RST</h4>     
    </div>     
    <p></p> 
  </div>
 </div>




* Window size

> TCP发送和接收都存在一个缓存区域。TCP 中的流量控制是通过一个在缓存中的可变大小的滑动窗口来处理的。窗口大小（ Window size) 字段指定了从被确认的字节算起可以发送多少个字节。

<center><img src="image-20191215163057718.png" alt="image-20191215163057718" style="zoom:50%;" /></center>

<img src="image-20191215165833678.png" alt="image-20191215165833678" style="zoom:50%;" />

> 首先计算机B要跟A说明自己的接受缓存是多少，然后A就会设置自己的发送缓存，不能超过B接收缓存的大小
>
> 然后A跟B说明自己接收缓存的大小，B设置其发送缓存



* urgent

  配合URG位使用，指明TCP数据中哪些是紧急的。比如urgent pointer设置为50，说明TCP数据中1-50是紧急的。

<center><img src="image-20191215164702576.png" alt="image-20191215164702576" style="zoom:50%;" /></center>

## 5.4 TCP连接建立

通过三次握手来建立连接

Host 1执行CONNECT, 与Host 2建立连接

Host 1发送连接请求

> `SYN`表示要建立connection, `SEQ=x`说明host 1发送的TCP数据的起始段是x

Host 2接收连接请求，回复一个确认[如果拒绝请求的话，发送一个设置了 RST 的应答报文]

> `SYN`表示建立connection, `SEQ=y`说明host 2发送的TCP数据的起始段是y, `ACK=x+1`说明确认了host 1发送过来的数据并且期待host1发送的下一个数据的起始段为x+1

host 1再进行确认

> host 1已经开始发送数据x+1，用piggybacking的方式对host 2对起始段y进行确认

<center><img src="image-20191212111637936.png" alt="image-20191212111637936" style="zoom:50%;" /></center>

## 5.5 TCP连接释放

* 为了释放一个连接，任何一方都可以发送一个设置了 FIN 标志位的 TCP 段，这表示它己经没有数据要发送了。
* 当 FIN 段被另 一方确认后，这个方向上的连接就被关闭，不再发送任何数据。然而，另一个方向上或许还在继续着无限的数据流。
* 当两个方向都关闭后，连接才算被彻底释放。

通常情况下，释 放一个连接需要 4 个 TCP 段：每个方向上一个 FIN 和一个 ACK。然而，第一个 ACK 和第二个 FIN 有可能被组合在同一个段中，从而将所需段总数降低到 3 个。

<center><img src="image-20191215173851930.png" alt="image-20191215173851930" style="zoom:50%;" /></center>

## 5.6 TCP连接管理模型

每条线都标记成一对“事件／动作“ [event / action]



<center><img src="image-20191215173944466.png" alt="image-20191215173944466" style="zoom:50%;" /></center>

沿着client的路径[粗实线]

> 当client上的一个应用程序发出`CONNECT`请求，本地的 TCP 实体创建一条连接记录，并将它标记为`SYN SENT`状态，然后发送一个`SYN`段。当 `SYN+ACK `到达的时候， TCP 发出三次握手过程的最后一个 `ACK` 段，然后切 换到 `ESTABLISHED` 状态。现在可以发送和接收数据了。
>
> 当一个应用结束时，它执行 `CLOSE` 原语，从而使本地的 TCP 实体发送一个 `FIN` 段， 并等待对应的 `ACK` 。当 `ACK` 到达时，状态迁移到 `FIN WAIT 2`，而且连接的一个方向被关闭。当另一方也关闭时，会到达一个 `FIN` 段，然后它被`ACK`。 现在，双方都已经关闭了连接，但是， TCP 要等待一段长度为最大数据包生存期两倍的时间，才能确保该连接上的所有数据包都己寿终正寝，以防万一发生确认被丢失的情形。当 计时器超时后， TCP 删除该连接记录。

注意这里的close是fin, ack, fin, ack总共四次

沿着server的路径[虚线]

> 服务器执行 LISTEN，并等待入境连接请求。当收到一个 `SYN` 时，服务器就确认该段并且进入到 `SYN RCVD` 状态。当服务器本身的 `SYN` 被确认后，就标志着三次握手过程的结束，服务器进入到 `ESTABLISHED` 状态。从现在开始双方可以传输数据了。
>
> 当客户完成了自己的数据传输，它就执行 `CLOSE`，从而导致 TCP 实体发送一个 `FIN 到服务器`。然后，服务器接到信号：当它也执行了` CLOSE `时， TCP 实体给客户发送一个 `FIN` 段。当该段的来自客户的确认返回后，服务器释放该连接，并且删除相应的连接记录。



## 5.7 TCP滑动窗口

<center><img src="image-20191215185528057.png" alt="image-20191215185528057" style="zoom:50%;" /></center>
<center><img src="image-20191215183419888.png" alt="image-20191215183419888" style="zoom:50%;" /></center>

<h4>sender buffer</h4><img class="image" src="image-20191215184124008.png" alt="image-20191215184124008" style="zoom:50%;" />
<h4>receiver buffer</h4>  

<img class="image" src="image-20191215184312475.png" alt="image-20191215184312475" style="zoom:50%;" />


delayed acknowledgement

**nagle's algorithm**---解决发送端应用每次向 TCP 传递一个字节而引起的问题

因为发送端发送多个小数据包的工作方式效率很低[比如41字节的数据包只包含1个字节的数据]

> 当数据每次以很少量方式进入到发送端时，发送端只是发送第一次到达的数据字节，然后将其余后面到达的字节缓冲起来，直到发送出去的那个数据包被确认
>
> 然后将所有缓冲的字节放在一个 TCP 段 中发送出去，并且继续开始缓冲字节，直到下一个段被确认。
>
> 这就是说，任何时候只有第 一个发送的数据包是小数据包。如果在一个来回时间内应用程序发送了许多数据，那么 Nagle 算法可以将这些数据放置在一个段中发送，由此大大地减少所需的带宽。另外，如 果应用传递来的数据足够多，多到可以填满一个最大数据段，则该算法也允许发送一个新的段。

***Clark's algorithm***---解决由于接收端应用每次从 TCP 流中读取一个字节而引起的问题

降低TCP性能的另一个问题是***silly window syndrome***。

> 当数据以大块形式传递,但是接收端的交互式应用每次只读取一个字节数据
>
> 初始时，接收端的 TCP 缓冲区为满，发送端知道这一点[即它有一个大小为 0 的窗口]。然后，交互式应用从TCP 流中读取一个字符，这个动作使得接收端的 TCP 欣喜若狂，它立刻发送一个窗口更新段给发送端，告诉它现在可以发送 1 个字节过来。发送端很感激，立即发送 1 个字节。现 在缓冲区又满了，所以，接收端对这 1 字节的数据段进行确认，同时设置窗口大小为 0。

Clark 的解决方案是禁止接收端发送只有 1 个字节的窗口更新段。相反，它强制接收端必须等待一段时间，直到有了一定数量的可用空间之后再通告给对方。特别是，只有当接收端能够处理它在建立连接时宣告的最大数据段，或者它的缓冲区一半为空时（相当于两者之中取较小的值〉，它才发送窗口更新段。而且，发送端不发送太小的段也会有所帮助。 相反，它应该等待一段时间，直到可以发送一个满的段，或者至少包含接收端缓冲区一半大小的段。

<center><img src="image-20191215192956539.png" alt="image-20191215192956539" style="zoom:50%;" /></center>

## 5.8 TCP timer management

### A. 重传计时器(RTO, Retransmission Timeout)

当 TCP 实体发出一个段时，它同时启动一个重传计时器。

* 如果在该计时器超时前该段被确认，则计时器被停止。
* 如果在确认到来之前计时器超时，则段被重传（并且该计时器被重新启动）

<br/>

#### 如何设置超时间隔

动态算法，根据网络性能的连续测量’情况，不断地调整超时间隔

***Jacobson算法***

对于每一个连接，TCP 维护一个变量SRTT[Smoothed Round-Trip Time，平滑的往返时间]，它代表到达接收方往返时间的当前最佳估计值。

当一个段被发送出去时， TCP 启动一个计时器，该计时器有两个作用:

* 一是看该段被确认需要多长时间
* 二是若确认时间太长，则触发重传动作。



***A. SRTT***

如果在计时器超时前确认返回，则 TCP 测量这次确认所花的时间RTT。然后 它根据下面的公式更新 SRTT: 

$$SRTT ＝\alpha SRTT + (1-\alpha） RTT$$

这里 $\alpha$ 是一个平滑因子，它决定了老的 RTT 值所占的权重。典型情况下 $\alpha＝\frac{7}{8}$ 。

<br/>

***B. RTTVAR***

即使有了一个好的 SRTT 值，要选择一个合适的重传超时间隔仍然不是一件容易的事情。Jacobson 提议让超时值对往返时间的变化以及平滑的往返时间要变得敏感。这种改变要求跟踪另一个平滑变量RTTVAR[ Round-Trip Time VARiation]

$$RTTVAR=\beta RTTVAR+(1-\beta)\vert SRTT-R\vert$$

通常$$\beta=\frac{3}{4}$$

***C. RTO***

重传超时值RTO

$$RTO=SRTT+4\times RTTVAR$$



但是采集往返时间样值R过程中还有一个问题是

> 当一个段超时并重新发送以后该怎么办？确认到达时，无法判断该确认是针对第一次传输，还是针对后来的重传。
>
> 若猜测错误，则会严重影响重传超时值RTT

***Karn算法***

> 不更新任何重传段的估算值。
>
> 此外，每次连续重传的超时间隔值加倍，直到段能一次通过为止。

### B. 持续计时器 persistence timer

重传计时器并不是 TCP 使用的唯一计时器。第二个计时器是持续计时器。它的设计意图是为了避免出现以下所述的死锁情况:

> 接收端发送一个窗口大小为 0 的确认，让发送端等一等。稍后，接收端更新了窗口，但是，携带更新消息的数据包丢失了。
>
> 现在，发送端和接收端都在等待对方的进一步动作。
>
> 当持续计时器超时后，发送端给接收端发送一个探询消息。接收端对探询消息的响应是将窗口大小告诉发送端。如果它仍 然为 0，则重置持续计时器，并开始下一轮循环。如果它非 0，则现在可以发送数据了。

### C. 保活计时器[keepalive timer]

to check whether the other side is still there

当一个连接空闲了较长一段时间以后，保活计时器可能超时，从而促使某一端查看另一端是否仍然还在。 如果另一端没有响应，则终止连接。



## 5.9 TCP congestion control

<center><img src="image-20191203131449883.png" alt="image-20191203131449883" style="zoom:40%;" /></center>

当提供给任何网络的负载超过它的处理能力时，拥塞便会产生。当路由器上的队列增长到很大时，网络层检测到拥塞，并试图通过丢弃数据包来管理拥塞。传输层接收到从网络层反馈来的拥塞信息，并减慢它发送到网络的流量速率。



* TCP维持一个***<u>拥塞窗口</u>*** (congestion window ），***窗口大小是任何时候发送端可以往网络发送的<u>字节数</u>***。相应的速率则是窗口大小除以连接的***往返时间***。 TCP 根据 AIMD 规则来调整该窗口的大小。

* 一个流量控制窗口，该窗口指出了***接收端可以缓冲的字节数***。

要并发跟踪这两个窗口，可能发送的字节数是两个窗口中较小的那个。 因此，有效窗口是发送端认为的应该大小和接收端认为的应该大小两者中的***较小者***。

> 如果拥塞窗口或流量控制窗口暂时己满，则 TCP 将停止发送数据。
>
> 如果接收端说“发送 64 KB数据”，但发送端知道超过 32KB 的突发将阻塞网络，它就只发送32KB
>
> 如果接收端说“发送 64 KB数据”，发送端知道高达 128 KB的突发通过网络都毫不费力，它会发送要求的全部 64 KB。

<br/>

Congestion Control Algorithm by Van Jacobson (1988) 

* To approximate an **AIMD** congestion window

* To represent congestion signal by packet loss
* To measure packet loss by a retransmission timer 
* To split data into segments (Ack Clock) 
* To use the optimal congestion window 

<center><img src="image-20191203140815575.png" alt="image-20191203140815575" style="zoom:50%;" /></center>

<br/>

### i. slow start

从初始值1启动，呈指数增长

<center><img src="image-20191212234251930.png" alt="image-20191212234251930" style="zoom:40%;" /></center>

> 在第一次往返时间，发送端把一个数据包注入网络（并且接收端接收到一个数据包）。在接下来的一个往返时间，发送端发出两个数据包，然后在第三个往返时间发送四个。
>
> 当发送端得到一个确认，他就把拥塞窗口的大小+1，并立即将两个数据包发送到网络中

由于慢速启动导致拥塞窗口按指数增长，网络很快拥塞。为了保持对慢速启动的控制，发送端为每个连接维持一个***<u>slow start threshold</u>***。

> 最初，这个值被设置得任意高。 TCP 以慢速启动方式不断增加拥塞窗口，直到发生超时，或者拥塞窗口超过该阔值。
>
> 每当检测到丢包，比如超时了，slow start threshold就被设置为当前拥塞窗口的一半，整个过程再重新启动。



### ii. Congestion Control

一旦slow start超过了threshold, TCP就切换到线性增加，每个往返时间，拥塞窗口只增加一段

<center><img src="image-20191203143540623.png" alt="image-20191203143540623" style="zoom:40%;" /></center>

拥塞窗口cwnd和最大段长MSS，一个常见的近似做法

针对cwnd/MSS中可能被确认的每个数据包，将cwnd增加$(MSS \times MSS)/cwnd$



### iii.  TCP Tahoe

<center><img src="image-20191203144306819.png" alt="image-20191203144306819" style="zoom:50%;" /></center>
<center><img src="image-20191215202752438.png" alt="image-20191215202752438" style="zoom:50%;" /></center>

- When **cwnd** is below **Threshold**, sender in <u>slow-start</u> phase, window grows <u>exponentially</u>. 

<center><img src="image-20191215205611729.png" alt="image-20191215205611729" style="zoom: 33%;" /></center>
<center><img src="image-20191215205648912.png" alt="image-20191215205648912" style="zoom:33%;" /></center>
<center><img src="image-20191215205720002.png" alt="image-20191215205720002" style="zoom: 33%;" /></center>
<center><img src="image-20191215205754864.png" alt="image-20191215205754864" style="zoom:33%;" /></center>

- When **cwnd** is above **Threshold**, sender is in <u>congestion-avoidanc</u>e phase, window grows <u>linearly</u>. 

<center><img src="image-20191215205825740.png" alt="image-20191215205825740" style="zoom:33%;" /></center>

- When packet loss occurs

  <center><img src="image-20191215205912344.png" alt="image-20191215205912344" style="zoom:33%;" /></center>

  - **cwnd** instead set to 1 MSS; 
  - window then grows exponentially to a **threshold**, then grows linearly
  - Threshold =1/2 of **cwnd** before loss event 

<center><img src="image-20191215205948869.png" alt="image-20191215205948869" style="zoom:33%;" /></center>

<br/>

### iv. TCP Reno

<center><img src="image-20191203144359731.png" alt="image-20191203144359731" style="zoom:40%;" /></center>

{%note info%}

发送端有一个快速方法来识别它的包己经被丢失。当丢失数据包的后续数据包到达接收端时，它们触发给发送端返回确认。这些确认段携带着相同的确认号，称为重复确认。发送端每次收到重复确认时，很可能另一个包己经到达接收端，而丢失的那个包仍然没有出现。TCP认为三个重复确认意味着这个数据包已经丢失

{%endnote%}

<center><img src="image-20191203144834670.png" alt="image-20191203144834670" style="zoom:35%;" /></center>

- When **cwnd** is below **Threshold**, sender in <u>slow-start</u> phase, window grows <u>exponentially</u>. 
- When **cwnd** is above **Threshold**, sender is in <u>congestion-avoidanc</u>e phase, window grows <u>linearly</u>. 
- When a triple duplicate ACK occurs 
  * **cwnd** is cut in half
  * window then grows linearly 
  * Threshold =1/2 of **cwnd** before loss event 
- When timeout occurs 
  - **cwnd** instead set to 1 MSS; 
  - window then grows exponentially to a **threshold**, then grows linearly 
  - Threshold =1/2 of **cwnd** before loss event 



总结如下：

<center><img src="image-20191203150750984.png" alt="image-20191203150750984" style="zoom:40%;" /></center>

### v. TCP throughput

What’s the average throughput of TCP as a function of window size and RTT?[Ignore slow start]

> Let W be the window size when loss occurs. 
>
> When window is W, throughput is $\frac{W}{RTT }$
>
> Just after loss, window drops to $\frac{W}{2}$, throughput to $\frac{W}{2RTT}$. 
>
> Average throughout: $\frac{0.75 W}{RTT}$



 

### vi. 一道例题

<center><img src="image-20191203151238340.png" alt="image-20191203151238340" style="zoom:40%;" /></center>

- **(1).** Identify the intervals of time when TCP slow start is operating. 

  看各阶段持续的时间。slow start有两段。答案是9

- **(2).** Identify the intervals of time when TCP congestion avoidance is 

  operating. 

  15

- **(3).** After the 16th transmission round, is segment loss detected by a triple duplicate ACK or by a timeout? 

  triple duplicate ACK 因为cwnd只降了一半

- **(4).** After the 22nd transmission round, is segment loss detected by a triple duplicate ACK or by a timeout? 

  timeout, 因为 cwnd降为1

- **(5).** What is the initial value of **Threshold** at the first transmission round? 

  32 因为32的时候进入了congestion avoidance阶段

- **(6).** What is the value of **Threshold** at the 18th transmission round? 

  21 threshold变成16th cwnd的一半

- **(7).** What is the value of **Threshold** at the 24th transmission round? 

  13 threshold变成16th cwnd的一半

- **(8).** Assuming a packet loss is detected after the 26th round by the receipt of a triple duplicate ACK, what will be the values of the **cwnd** and **Threshold**? 

  cwnd:4	 threshold:4

- **(9).** Suppose **TCP Tahoe** is used (instead of TCP Reno), and assume that triple duplicate ACKs are received at the 16th round. What are the **Threshold** and **cwnd** at the 19th round? 

  cwnd:4 	threshold:21



<br/>

reference:

[1] https://andrewpqc.github.io/2018/07/16/transport-layer-udp-and-tcp/

