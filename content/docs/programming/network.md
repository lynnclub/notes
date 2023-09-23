---
title: "网络"
---

## 分层模型

![ISO网络模型](/notes/images/programming/network1.png)

![网络模型](/notes/images/programming/network.png)

## TCP三次握手、四次挥手

TCP三次握手是建立TCP连接的过程，它确保了通信双方之间的可靠性和同步性。下面是TCP三次握手的步骤：

1. 第一次握手（SYN）：客户端向服务器发送一个SYN（同步）报文段。这个报文段中包含了初始序列号（ISN）和设置SYN标志位，表示客户端请求建立连接。
2. 第二次握手（SYN + ACK）：服务器收到客户端的SYN报文段后，确认序列号，并发送一个带有SYN和ACK（确认）标志位的报文段作为响应。服务器端也会为自己的初始序列号生成一个随机的ISN。
3. 第三次握手（ACK）：客户端收到服务器的SYN + ACK报文段后，确认序列号并发送一个带有ACK标志位的报文段给服务器。这个报文段的序列号是服务器发送的SYN报文段的确认序列号加1，表示客户端已经接受到了服务器的确认，并告知服务器连接已建立。

完成了这个三次握手过程后，TCP连接就建立起来了，双方可以开始进行数据的传输。

这个三次握手的过程是为了确保客户端和服务器都同意建立连接，并且双方都知道对方的序列号，以便在数据传输过程中进行**可靠的顺序传送和数据确认**。如果在握手过程中任何一方没有收到确认，或者超时没有收到回复，就会触发重新发送握手报文的过程，直到建立连接成功或达到一定的重试次数。

![TCP](/notes/images/programming/network2.png)
![TCP](/notes/images/programming/network3.png)

## TIME_WAIT

问题：time_wait 连接占用端口，上限65535，当大量的连接处于 time_wait 时，新建立 TCP 连接会出错，address already in use : connect

原因：HTTP 请求中，如果 connection 头部被设置为 close 时，基本都由 服务端 发起主动关闭连接。主动发起关闭连接 的一端，会进入 time_wait 状态。TCP 四次挥手关闭连接机制中，为了保证 ACK 重发和丢弃延迟数据，设置 time_wait 为 2 倍的 MSL（报文最大存活时间，MSL 为 2 分钟）。

解决：

1. 客户端 HTTP 请求的头部，connection 设置为 keep-alive，保持存活一段时间：现在的浏览器，一般都这么进行了。 
2. 服务器端 允许 time_wait 状态的 socket 被重用，缩减 time_wait 时间，设置为 1 MSL。

## 顺序传输、延迟ACK

连接建立后，发送方将数据切割，并且在头部分配了序列号。接收方收到的数据包顺序可能乱，需要根据序列号排序，因此接收方并不会立即回复ACK，而是由定时器每隔200ms检查是否需要发送。

延迟同时还可以节省网络流量，连续收到多个相同包只需回复一次。如果接收方有数据发送，还可以带上ACK。

## 流量控制

如果发送方数据发送过快，接收方就可能来不及接收，造成数据丢失。流量控制（flow control）就是让发送方的发送速率不要太快，要让接收方来得及接收。

利用滑动窗口机制可以很方便地在 TCP 连接上实现发送方流量控制。通过接收方的确认报文中的窗口字段，发送方能够准确地控制发送字节数。

## 拥塞控制

计算机网络处在一个共享的环境中，可能会发生网络拥堵，导致延迟、丢包，这时 TCP 就会重传数据，但是重传会导致网络的负担更重，会加剧延迟、丢包问题。

拥塞控制是避免发送方的数据填满整个网络。为了在发送方调节所要发送数据的数据量，定义了⼀个叫做 拥塞窗口 的概念。拥塞窗口 cwnd 是发送方维护的⼀个的状态变量，它会根据网络的拥塞程度动态变化。

拥塞控制算法

1. 慢启动：⼀点⼀点的提高发送数据包的数量。规则：发送方每收到⼀个 ACK，拥塞窗⼝ cwnd 的大小就会加 1。
2. 拥塞避免：当拥塞窗口 cwnd 超过慢启动门限 ssthresh 就会进⼊拥塞避免算法。规则：每收到⼀个 ACK 时，cwnd 增加 1/cwnd。
3. 快重传：当接收方发现丢了⼀个中间包的时候，发送三次前⼀个包的ACK，于是发送端就会快速地重传，不必等待超时再重传。
4. 快恢复：快重传和快恢复⼀般同时使用，快速恢复算法认为，还能收到 3 个重复的 ACK 说明网络也不那么糟糕，所以没有必要像 RTO 超时那么强烈。进⼊快速恢复之前， cwnd 和 ssthresh 已被更新了：cwnd = cwnd/2 ，也就是设置为原来的⼀半，ssthresh = cwnd 。

## 五种IO模型

应用程序不能直接操作底层硬件，必须和系统内核交互，从缓冲区收发网络数据。

![缓冲区](/notes/images/programming/network4.png)

1. blockingIO - 阻塞IO
2. nonblockingIO - 非阻塞IO
3. signaldrivenIO - 信号驱动IO
4. asynchronousIO - 异步IO
5. IOmultiplexing - IO多路复用

### 阻塞IO

![阻塞IO](/notes/images/programming/network5.png)

### 非阻塞IO

![非阻塞IO](/notes/images/programming/network6.png)

### 信号驱动IO

![信号驱动IO](/notes/images/programming/network7.png)

### 异步IO

![异步IO](/notes/images/programming/network8.png)

### IO多路复用

![IO多路复用](/notes/images/programming/network9.png)

说明：

1. 阻塞IO、非阻塞IO：由进程直接与内核交互，开销大，并发低。
2. 信号驱动IO：基于信号回调，不适合处理高并发。
3. 异步IO、IO多路复用：需要系统底层支持，linux内核版本2.5以上，适合高并发。

IO多路复用的系统调用有select、poll、epoll，最大优势是系统开销小，系统不必创建进程/线程，也不必维护这些进程/线程，从而大大减小了系统的开销。

异步IO的实现会把数据从内核拷贝到用户空间，IO多路复用本质上是同步IO，需要在读写事件就绪后自己负责进行读写。

### select

<br>
<video width="640" height="360" controls>
  <source src="/notes/images/programming/select.mp4" type="video/mp4">
  您的浏览器不支持视频播放。
</video>

### epoll

<br>
<video width="640" height="360" controls>
  <source src="/notes/images/programming/epoll.mp4" type="video/mp4">
  您的浏览器不支持视频播放。
</video>

epoll的两种触发方式

水平触发（LT，Level Triggered）：当事件被触发时，epoll会立即通知应用程序，并在事件被处理完后返回。如果应用程序没有及时处理事件，epoll会持续通知应用程序，直到事件被处理完毕。

边沿触发（ET，Edge Triggered）：当事件被触发时，epoll只通知应用程序一次，并在事件被处理完后返回。如果应用程序没有及时处理事件，epoll不会再次通知应用程序。

需要注意的是，水平触发和边沿触发的选择是在创建epoll实例时指定的，可以通过epoll_create函数的中的flags参数来指定触发方式。如果未指定触发方式参数，则默认使用水平触发方式。

|       | select                             | poll                               | epoll                               |
|-------|------------------------------------|------------------------------------|-------------------------------------|
| 性能    | 随着连接数的增加，性能急剧下降，处理成千上万的并发连接数时，性能很差 | 随着连接数的增加，性能急剧下降，处理成千上万的并发连接数时，性能很差 | 随着连接数的增加，性能基本没有变化                   |
| 连接数   | 一般1024                             | 无限制                                | 无限制                                 |
| 内存拷贝  | 每次调用select拷贝                       | 每次调用poll拷贝                         | fd首次调用epoll_ctl拷贝，每次调用epoll_wait不拷贝 |
| 数据结构  | bitmap                             | 数组                                 | 红黑树                                 |
| 处理机制  | 线性轮询                               | 线性轮询                               | FD挂在红黑树，通过事件回调callback              |
| 时间复杂度 | O(n)                               | O(n)                               | O(1)                                |

## HTTP

![请求结构](/notes/images/programming/network10.png)

![响应结构](/notes/images/programming/network11.png)

问：怎么知道http处理完了？  
答：通过Content-Length包大小

## HTTPS

SSL/TLS

HTTPS证书是一种数字签名证书，包含颁发者的数字签名、颁发者标识符信息、证书的有效期、主题的公钥值、主题的标识符信息等。公钥是HTTPS证书的一部分，用于确保通信过程中的安全性。
