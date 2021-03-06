---
title: Computer network--数据链路层
date: 2019-10-16 15:31:45
tags:  Computer Network
categories:  Computer Network
index_img: /img/image-20191012225542485.png
---

计算机网络课程--数据链路层总结

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


<br/>



# 1. 数据链路层的设计问题

数据链路层使用物理层提供的服务在通信信道上发送和接收比特。要实现的功能包括

* 向网络层提供一个定义良好的服务接口
* 处理传输错误
* 调节数据流，确保慢速的接受方不会被快速的发送方淹没

数据包被封装成帧，每个帧包含一个Header,一个Payload field,一个Trailer

<center><img src="image-20191012225542485.png" alt="image-20191012225542485" style="zoom:50%;" /></center>

## 1.1 提供给网络层的服务

三种服务类型

* 无确认的无连接服务

> 特点
>
> * 事先不用建立物理连接，事后也不用释放逻辑连接
>
> * 源机器向目标机器发送独立的frame,目标机器不对这些frame进行确认
>
> 适合的场合
>
> * 实时通信[数据迟到比数据受损更难忍受]
>
> * 错误率比较低的场合[因为数据链路层可靠性不高，所以物理层reliable要求会高一点，适合有线网络

* 有确认的无连接服务

> 特点
>
> * 不用建立物理连接
> * 发送的每一帧都要单独确认，这样发送方可以知道一个帧是否已经正确到达目的地。如果一个帧在制定时间间隔内还没有到达，则发送方将再次发送该帧
> * 不能保证收到包的顺序和发送顺序一致
>
> 适用场合
>
> * 不可靠的信道：无线系统,WiFi

* 有确认的有连接服务

> 源机器和目标机器在传输任何数据之前要建立一个连接，连接发送每一帧都被编号，数据链路层确保发出的每个帧都会真正被接收方按顺序收到且只收到一次。相当于提供了一个可靠的比特流
>
> 适用场合
>
> * 长距离且不可靠的链路 



## 1.2 framing[成帧]

数据链路层要检测和纠正错误。

数据链路层通常的做法是将比特流拆分成多个离散的帧。

为每个帧计算一个称为校验和的短令牌，放在帧中一起传输。

帧到达目标机器时，重新计算校验和。如果新计算的校验和与传输过来的不同，说明产生了错误。

<br/>

拆分方法

### 1.2.1 Byte count

Method: to use a field in the header to specify the number of characters in the frame.

<center><img src="image-20191008100755672.png" alt="image-20191008100755672" style="zoom:35%;" /></center>

problem:

错了一个character count就会全错

<center><img src="image-20191012232424422.png" alt="image-20191012232424422" style="zoom:50%;" /></center>

### 1.2.2  flag bytes with byte stuffing字节填充的标志字节法

考虑到出错之后到重新同步问题，用一些标志字节(flag byte)作为一个帧开始和结束

<center><img src="image-20191012233630323.png" alt="image-20191012233630323" style="zoom:50%;" /></center>

如图所示的这种frame的结构

两个连续的flag标志了一帧的结束和下一帧的开始。

problem:

如果标志字节在数据中出现，会干扰到帧的分界

<br/>

所以提出的解决方法是字节填充(byte stuffing)，发送方的数据链路层在数据中偶尔出现的每个标志字节前面插入一个特殊的转义字节(ESC),如果转移字符也出现在数据中，再用一个转义字符填充

<center><img src="image-20191012234408243.png" alt="image-20191012234408243" style="zoom:35%;" /></center>

### 1.2.3 flag bit with bit stuffing

考虑到字节填充只能使用8bits的字节，这里bit填充可以使帧包含任意大小单元

flag bits 01111110

也要考虑数据中出现flag的问题，所以发送方的数据链路层在数据中每遇到连续的5个1，就添加1个0

接收方看到5个连续的1，后面紧跟1个0，就自动剔除这个0

<center><img src="image-20191013180557730.png" alt="image-20191013180557730" style="zoom:50%;" /> </center>

### 1.2.4 physical layer encoding violation

物理层有讲过4B/5B的编码，4个比特被映射成5个比特，说明有16种信号不会出现在数据中，可以用来作为flag

## 1.3 Error control

如何保证所有帧最终传递给目标机器的网络层，并且保持正确的顺序

– To provide the sender with some feedback

*  Positive acknowledgement (ACK) 收到信号回一个ack
*  Negative acknowledgement (NAK) 没收到信号回一个nack， 潜在的假设是我知道发送的包以及发送的顺序

– To provide timeout timers

* Resend as necessary

– To number frames

* To distinguish retransmissions from originals

## 1.4 Flow control

发送方发送帧的速度超过了接收方接收的速度，该如何处理

- To introduce flow control to throttle the sender into sending no faster than the receiver can handle the traffic.

- Flow control protocol contains well-defined rules about when a sender may transmit the next frame.

- Two approaches
  – **Feedback-based flow control** 由receiver决定发送的速度

  – Rate-based flow control



# 2. Error Detection and Correction

Error Types

> isolated errors单个的错误
>
> burst errors一连串的错误

approaches: 在数据中引入一些<u>冗余</u>来进行error detection和error correction



## 2.1 一些概念

### i.

一帧有

* m个数据位
* r个冗余位。r个校验位是由m个数据位的函数计算得到的

令数据块总长度为n(n=m+r)，我们称其为(n,m)码。

<u>码字</u>(codeword): 一个包含了数据位和校验位的n位单元

码率(code rate): codeword中数据部分占比m/n

### ii.

Hamming distance of 2 codewords

>  两个codeword中不相同的bit的个数

Hamming distance of complete code(all valid codewords)

> The minimum Hamming distance of two valid codewords in the code

<center><img src="image-20191013185144979.png" alt="image-20191013185144979" style="zoom:33%;" /></center>

To detect d errors, you need d+1 Hamming distance code.

> 因为，d+1的hamming distance,说明两个codeword之间最少有d+1位不相同。所以如果d个错误出现，这个codeword也不会错成一个valid codeword.



To correct d errors, you need 2d+1 Hamming distance code

> 因为,两个codeword之间最少有2d+1个bits不相同，如果有d errors,那么偏离原codeword的hamming distance 为d, 2d+1的距离保证了与这个错误的codeword距离最近的依然是原codeword

## 2.2 Error Correction codes

### 2.2.1 Hamming code for single error

参考以下两篇blog可以有一个基本了解，我结合他们做了一个总结

https://blog.csdn.net/Yonggie/article/details/83186280

https://blog.csdn.net/blue_starry_sky/article/details/53997548

(图片中应该是redundancy)

<table>
  <tr>
    <td><img src="IMG_5ACFEACB9F7D-1.png" alt="IMG_5ACFEACB9F7D-1" style="zoom: 30%;" /></td>
    <td>
    <img src="IMG_AA14DF432E13-1.png" alt="IMG_AA14DF432E13-1" style="zoom:40%;" /></td>
  </tr>
</table>


为什么上述纠错可行呢？

因为如果没有出现错误的话$hi\oplus(P_i组内异或结果)=0$，因为我们计算的时候$h_i=P_i组内异或结果$, 自己和自己异或为0。如果出现了一个错误的话，与这个错误相关的组$hi\oplus(P_i组内异或结果)=1$, 这些组别标识了错误的位置

<br/>

<br/>

-你知道吗，Hamming codes也可以用来correct burst error呢？

-哦？怎么做呢

-嘿嘿嘿，有一点tricky

-  one codeword per row. k consecutive codewords
-  –Transmit the matrix by one column at a time.

这样一行的连串错误，在列看来就是一列可以有一个错误

<center><img src="image-20191229155224546.png" alt="image-20191229155224546" style="zoom:50%;" /></center>

## 2.3 Error Detection codes

相比error correction, error detection代价更小。

error correction引入太多redundancy bit, 不如用error detection，如果检测到error,就retransmission这样代价小。

### 2.3.1 Parity

> Append a parity bit to detect single error.

奇校验：数据bit 1的个数为奇数就加一个0bit, 是偶数就加一个1bit。保持1的个数为奇数

偶校验：数据bit 1的个数为偶数就加一个0bit, 是奇数就加一个1bit。保持1点个数为偶数

single error显然是可以检测出来，其实burst error也是可以的，如下图，上文提到的按列发送

<center><img src="image-20191024153859898.png" alt="image-20191024153859898" style="zoom:50%;" /></center>

### 2.3.2 Checksum

> 将数据分成若干段，做加法，取模

一个internet checksum的例子

<center><img src="IMG_7FC95B48427A-1.png" alt="IMG_7FC95B48427A-1" style="zoom: 33%;" />                  </center>

性质

– Improved error detection over parity bits

– Vulnerable to systematic errors, 比如：在后面加了一连串0，检测不出错



### 2.3.3 CRC

<center><img src="IMG_BB50334654CE-1.png" alt="IMG_BB50334654CE-1" style="zoom:36%;" /></center>
<center><img src="IMG_B37D4802C2E6-1.png" alt="IMG_B37D4802C2E6-1" style="zoom:40%;" /></center>
<center><img src="IMG_CAA2C1958BE2-1.png" alt="IMG_CAA2C1958BE2-1" style="zoom:31%;" /></center>

注意一个$x^{k-1}+\cdots+1$是一个$k-1$阶多项式

选择余数的位数为除数的阶。比如$x^4+x+1$是一个4阶多项式，选择余数为4阶。

<br/>

# 3. 基本数据链路层协议

<center><img src="image-20191024230806384.png" alt="image-20191024230806384" style="zoom:25%;" /></center>

## 3.1 Elementary Data Link Protocol

### 3.1.2 Some Assumptions

* 物理层、数据链路层、网络层都是独立的进程，通过来回传递消息进行通信。
* 机器A希望用一个可靠的、面向连接的服务向机器B发送一个长数据流
* 机器不会崩溃

### 3.1.3 声明

`seq_nr`对帧进行编号，0~MAX_SEQ

一个帧由4个字段组成:kind, seq, ack, info 前三个包含控制信息，最后一个可能包含要传输的实际数据，这些字段合起来称为 **帧头**

> kind指出帧中是否有数据
>
> seq序号
>
> ack确认
>
> info数据包

信道传输，有时会丢失帧，发送方会启动计时器，在预设时间间隔没有收到应答，则时钟超时，链路层发出终端信号。实现

> 让过程 `wait for event `返回 `event= timeout` 
>
> 过程 `start_timer` 和 `stop_timer` 分别打开和关闭计时器。
>
> 只有当计时器在运行并且调用 `stop_timer` 之前，超时事件才有可能发生。在计时器运行的同时，允许显式地调用 start_timer: 这样的调用只是重置时钟，等到再经过一个完整的时钟间隔之后引发下一次超时事件（除 非它再次被重置，或者被关闭〉。

过程 `start_ack_timer` 和 `stop_ack_timer` 控制一个辅助计时器，该定时器被用于在特定条件下产生确认。

```cpp
#define MAX_PKT 1024
typedef enum {false, true} boolean;
typedef unsigned int seq_nr;
typedef struct {unsigned char data[MAX_PKT];} packet;
typedef enum {data, ack, nak} frame_kind;

typedef struct {
  frame_kind kind;
  seq_nr seq;
  seq_nr ack;
  packet info;
} frame;

void wait_for_event(event_type* event);
void from_network_layer(packet* p);
void to_network_layer(packet* p);
void from_physical_layer(frame* r);
void start_timer(seq_nr k);
void stop_timer(seq_nr k);
void start_ack_timer(void);
void stop_ack_timer(void);
void enable_network_layer(void);
void disable_network_layer(void);
//increment k circularly
#define inc(k) if(k < MAX_SEQ) k = k + 1; else k = 0
```

### 3.1.4 一个乌托邦式的单工协议

* Data are transmitted in <u>one direction only</u>.

* The communication channel <u>never damages or loses frames</u>.

* Both the transmitting and receiving network layers are <u>always ready</u>.

* Processing time can be ignored.

* Infinite buffer space is available.

```cpp
typedef enum {frame_arrival} event_type;
#include "protocol.h"

void sender1(void) {
  //buffer for an outbound frame
  frame s;
  //buffer for an outbound packet
  packet buffer;
  while(true) {
    //从网络层拿一个包
    from_network_layer(&buffer);
    //放到frame
    s.info = buffer;
    //传包
    to_physical_layer(&s);
  }
}
```



```cpp
void receiver1(void) {
  frame r;
  //filled in by wait, but not used here
  event_type event;
  
  while(true) {
    //only possibility is frame_arrival
    wait_for_event(&event);
    //go get the inbound frame
    from_physical_layer(&r);
    //pass the data to the network layer
    to_network_layer(&r.info);
  }
}
```



### 3.1.5 A simplex stop-and-wait protocol

* Data traffic is still <u>simplex protocol</u>

* The communication channel is assumed to be <u>error free</u>. 

* The sender is always ready. ***<u>The receiver is NOT always ready or the receiver has limited buffer space</u>***.这里放宽了protocol的限制
  * The sender simply inserts a delay into protocol 1 to slow it down sufficiently to keep from swamping the receiver. --> low utilization of bandwidth.
  * The receiver provides <u>feedback</u> to the sender, permitting the sender to transmit the next frame.

* Protocol(p2) ensures sender can't outspace receiver:
  * Receiver returns a dummy frame (ack) when ready
  * Only one frame out at a time ± called stop-and-wait
  * We added flow control!

<center><img src="image-20191025105854425.png" alt="image-20191025105854425" style="zoom:30%;" /></center>

```cpp
typedef enum {frame_arrival} event_type;
#include "protocol.h"

void sender2(void) {
  frame s;
  packet buffer;
  event_type event;
  
  while(true) {
    from_network_layer(&buffer);
    s.info = buffer;
    to_physical_layer(&s);
    wait_for_event(&event);
  }
}
```

```cpp
void receiver2(void) {
  frame r, s;
  event_type event;
  while(true) {
    wait_for_event(&event);
    from_physical_layer(&r);
    to_network_layer(&r.info);
    to_physical_layer(&s);
  }
}
```



### 3.1.6 A simplex protocol for a noisy channel

* Data traffic is <u>still simplex</u>
* The communication channel is ***<u>NOT free of errors</u>***.
* The receiver is ***<u>Not always ready</u>***.



possible solutions:

* Protocol2 + timer ---> duplicate packets

* Protocol2 + timer + to number the frame

如果在一个协议中，发送方在前移到下一个数据之前必须等待一个肯定确认，这样的协议称为自动重复请求(ARQ, Automatic Repeat Request)或者带有重传的肯定确认(PAR, Positive Acknowledgement with Retransmission)



<table>
  <tr>
    <td>
      <img src="image-20191025200128444.png" alt="image-20191025200128444" style="zoom:50%;" />
    </td>
    <td>
      <img src="image-20191025195618998.png" alt="image-20191025195618998" style="zoom:50%;" />
    </td> 
    <td>
      <img src="image-20191025200041993.png" alt="image-20191025200041993" style="zoom:50%;" />
    </td>
  </tr>
</table>




```cpp
#define MAZ_SEQ1
typedef enum {frame_arrival, cksum_err, timeout} event_type;
#include "protocol.h"
void sender3(void) {
  //seq number of next outgoing frame
  seq_nr next_frame_to_send;
  //scratch variable
  frame s;
  //buffer for an outbound packet
  packet buffer;
  event_type event;
  //initialize outbound sequence numbers
  next_frame_to_send = 0;
  //fetch first packet
  from_network_layer(&buffer);
  
  while(true) {
    //construct a frame for transmission
    s.info = buffer;
    //insert sequence number in frame
    s.seq = next_frame_to_send;
    //send it on its way
    to_physical_layer(&s);
    //if answer takes too long, timeout
    start_timer(s.seq);
    //frame_arrival, cksum_err, timeout
    wait_for_event(&event);
    if(event == frame_arrival){
      	//get the acknowledgement
        from_physical_layer(&s);
        if(s.ack == next_frame_to_send) {
          //turn the timer off
          stop_timer(s.ack);
          //get the next one to send
          from_network_layer(&buffer);
          //invert next_frame_to_send
          inc(next_frame_to_send);
      	}
    }
  }
}
```

```cpp
void receiver3(void) {
  frame r, s;
  event_type event;
  frame_expected = 0;
  while(true) {
    wait_for_event(&event);//frame_arrival, cksum_err
    if(event == frame_arrival) {
      from_physical_layer(&r);
      if(r.seq == frame_expected) {
        to_network_layer(&r.info);
        inc(frame_expected);
      }
    }
  }
  s.ack = 1 - frame_expected;
}
```

<table>
  <tr>
    <td>
      <img src="IMG_DA892339BCCD-1.png" alt="IMG_DA892339BCCD-1" style="zoom:60%;" />
    </td>
		<td>
  <img src="IMG_430578E193A6-1.png" alt="IMG_430578E193A6-1" style="zoom:70%;" />
  	</td> 
  </tr>
</table>


​     

### 3.1.7 Sliding window protocol

前面的协议中，数据帧单向传输，但是大多数情况，需要双向传输。

实现全双工的一种方法是运行前面协议的两个实例，每个实例使用一条独立的链路进行单工数据传输。但是这样每条链路由一个“前向”信道(用于数据)和一个"逆向"信道(用于确认)组成。两种情况下的逆向带宽几乎被完全浪费了。

一种更好的做法是使用同一条链路来传输两个方向上的数据。

机器A向机器B发送数据帧和确认帧[确认收到机器B上一次发送的帧]混合，接收方只需检查帧头部kind字段，就可以分辨数据帧和确认帧

<br/>

**i. 捎带确认[piggybacking]**：

> 暂时延缓确认以便将确认信息搭载在下一个出境数据帧上的技术。确认信息被附加在往外发送的数据帧上[使用帧头的ack字段]
>
> 优化了上述发送数据帧和确认帧的方法，减轻了接收方的处理负担

**ii. 滑动窗口[sliding window]**:

> 所有滑动窗口协议的本质是在任何时刻发送方总是维持一组序号，对应于允许它发送的帧。称这些帧落在<u>发送窗口</u>[sliding window]内。
>
> 接收方也维持一个<u>接受窗口</u>[receiving window]，对应于一组允许它接受的帧。
>
> 发送方的窗口和接收方的窗口不必有同样的上下界，甚至也不必有同样的大小
>
> 
>
> 任何一个出境帧都包含一个序号， 范围从 0 到某个最大值。序号的最大值通常是$2^n-1$，这样序号正好可以填入到一个n位的字段中。停-等式滑动窗口协议使用n=1，限制了序号只能是 0 和 1 ，但是更加复杂的协议版本可以使用任意的n

https://blog.csdn.net/wdscq1234/article/details/52444277

数据链路层协议将数据包递交给网络层的次序必须  和  发送机器上数据包从网络层传递给数据链路层的次序相同。

<center><img src="image-20191026220230537.png" alt="image-20191026220230537" style="zoom:50%;" /></center>

**<u>A. 一位滑动窗口协议</u>**

窗口尺寸为1，发送方发出一帧以后，必须等待前一帧的确认的到来才能发送下一帧，这个协议使用了停-等式办法



<table>
  <tr>
    <td>
      <img src="IMG_BE2535BE4B0D.png" alt="IMG_BE2535BE4B0D-1" style="zoom:55%;" />
    </td>
    <td>
      <img src="https://miaochenlu.github.io/picture/IMG_7FBAD2BAF798-1.png" alt="IMG_7FBAD2BAF798-1" style="zoom:50%;" />
     </td>
	</tr>
</table>



<center><img src="image-20191025202700163.png" alt="image-20191025202700163" style="zoom:50%;" /></center>

**<u>B. Go back N</u>**

<a href="http://www.ccs-labs.org/teaching/rn/animations/gbn_sr/">go back N动画</a>

<a href="https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/go-back-n-protocol/index.html">GBN animination</a>

https://blog.csdn.net/qq_34501940/article/details/51180268

<center><img src="image-20191229212229327.png" alt="image-20191229212229327" style="zoom:50%;" /></center>



> 在发送完一个帧后，不用停下来等待确认，而是可以连续发送多个数据帧。收到确认帧时，任可发送数据，这样就减少了等待时间，整个通信的通吞吐量提高。 
> 如果前一个帧在超时时间内未得到确认，就认为丢失或被破坏，需要重发出错帧及其后面的所有数据帧。这样有可能有把正确的数据帧重传一遍，降低了传送效率。 
> 线路很差时，使用退后N帧的协议会浪费大量的带宽重传帧。

<br/>

<br/>

**<u>C. Selective Repeat ARQ</u>**

<a href="https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html">ARQ</a>

如果错误很少发生，则回退 n 协议可以工作得很好：但是，如果线路质量很差，那么重传的帧要浪费大量带宽。另一种处理错误的策略是选择重传协议，允许接收方***接受并缓存坏帧***或者丢失帧后面的所有帧。



<center><img src="image-20191229210854474.png" alt="image-20191229210854474" style="zoom:50%;" /><</center>



> NAK：非确认帧，当在一定时间内没有收到某个数据帧的ACK时，回复一个NACK。 
> 在发送过程中，如果一个数据帧计时器超时，就认为该帧丢失或者被破坏，接收端只把出错的的帧丢弃，其后面的数据帧保存在缓存中，并向发送端回复NAK。发送端接收到NAK时，只发送出错的帧。 
> 如果落在窗口的帧从未接受过，那么存储起来，等比它序列号小的所有帧都按次序交给网络层，那么此帧才提交给网络层。 
> 接收端收到的数据包的顺序可能和发送的数据包顺序不一样。因此在数据包里必须含有顺序字符来帮助接受端来排序。 
> 选择重传协议可以避免重复传送那些正确到达接收端的数据帧。但是接收端要设置具有相当容量的缓存空间，这在许多情况下是不够经济的。



***Window size***

* Send Window $Size <= (MAX\_SEQ+1)/2$ 注意Max_seq是指最大序号，不是最大数量

* Receive Window Size = Send Window Size

> 这个问题的本质在于当接收方向前移动它的窗口后，新的有效序号范围与老的序号范围有重叠。因此，后续的一批帧可能是重复的帧（如果所有的确认都丢失了），也可能是新的帧（如果所有的确认都接收到了)。接收方根本无法区分这两种情形。

<center><img src="image-20191229205721964.png" alt="image-20191229205721964" style="zoom:50%;" /></center>



<img src="image-20200104140132312.png" alt="image-20200104140132312" style="zoom:50%;" />

1. 从t0->t1, Node-B返回的最后一个包是R3,3

说明Node-B收到了0～2号包，所有下一个期待A发送3号包

因此，收到的有S0,0 & S1,0 & S2,0



2. 注意，题目中说，transmission sequence number and acknowledgment sequence number是3bit

所有一共有8个序号

在GBN协议中，序号个数≥发送窗口+1

所以，发送窗口为7

<img src="image-20200104140642889.png" alt="image-20200104140642889" style="zoom:50%;" />

A还可以发送5个数据包

当发送第一个序号为5的数据帧时，可以同时对乙方发来的且按序到达的1号数据帧进行捎带确认，确认序号为2，因此甲方发送的第一个数据帧为S5,2；

同理，当发送最后一个序号为1的数据帧时，可以同时对乙方发来的且按序到达的1号数据帧进行捎带确认，确认序号为2，因此甲方发送的最后一个数据帧时S1,2。需要注意的是，尽管甲方收到了R3,3，也就是乙方发来的序号为3的数据帧，但是该数据帧并未按序到达，因为甲方之前没有收到序号为2的数据帧，因此甲方不能对R3,3进行捎带确认。

3. 

方在t0时刻到t1时刻期间共发送了序号为0~4的5个数据帧。在t1时刻甲方超时重传2号数据帧，这表明甲方没有收到乙方对2号数据帧的确认，这可能是由于2号数据帧未按序到达乙方或按序到达乙方但出现了误码。由于甲乙双方都使用GBN协议，因此甲方需要重传超时的数据帧及其后续数据帧，也就是甲方需要重传序号为2~4的3个数据帧。重传的第一个帧的序号为2，由于之前已经按序正确收到乙方发来的序号为2的数据帧，因此可以进行捎带确认，确认号为3，因此重传的第一个帧为S2,3。

4. 

$$\frac{7\times 1000\times 8/100Mbps}{0.96ms+ 1000\times 8/100Mbps}=50\%$$


# 4. 数据链路层协议实例

## 4.1 PPP

A standard protocol called PPP (Point to Point Protocol) is used to send packets over the links, including the SONET fiber optic links and ADSL links

<center><img src="image-20191229214521665.png" alt="image-20191229214521665" style="zoom:50%;" /></center>

PPP 功能包括处理错误检测链路的配置、支持多种协议、允许身份认证等。它是一个 早期简化协议的改进，那个协议称为串行线路 Internet 协议 （SLIP, Serial Line Internet Protocol ）。伴随着一组广泛选项， PPP 提供了 3 个主要特性：

*  一种成帧方法。它可以毫无歧义地区分出一帧的结束和下一帧的开始．

*  一个链路控制协议．它可用于启动线路、测试线路、协商参数， 以及当线路不再需要时温和地关闭线路。该协议称为***<u>链路控制协议 （LCP, Link Control Protocol）</u>***。

*  一种协商网络层选项的方式。协商方式独立于网络层协议．所选择的方法是针对每一种支持的网络层都有一个不同的***<u>网络控制协议 （NCP, Network Control Protocol)</u>***。

因为没有必要重新发明轮子，所以 PPP 帧格式的选择酷似 HDLC 帧格式。 HDLC 是***<u>高级数据链路控制协议 （ High-level Data Link Control ）</u>***，是一个早期被广泛使用的家庭协议实例。

PPP 和 HDLC 之间的主要区别在于：

* PPP 是面向字节而不是面向比特的．特别是 PPP 使用字节填充技术，所有帧的长度均是字节的整数倍。 HDLC 协议则使用比特填充技术， 允许帧的长度不是字节的倍数，例如 30.25 字节。

* HDLC 协议提供了可靠的数据传输，所采用的方式正是我们已熟悉的滑动窗口、确认和超时机制等。 PPP 也可以在诸如无线网络等嘈杂的环境里提供可靠传输，具体细节由盯Cl663 定义。然而，实际上很少这样做．相反， Internet几乎都是采用一种“无编号模式”来提供无连接无确认的服务．

<center><img src="image-20191229214119334.png" alt="image-20191229214119334" style="zoom:45%;" /></center>

## 4.2 ADSL 

<center><img src="image-20191229214707273.png" alt="image-20191229214707273" style="zoom:40%;" /></center>

<center><img src="image-20191229215124692.png" alt="image-20191229215124692" style="zoom:50%;" /></center>













