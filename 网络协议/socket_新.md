# socket 新

## 前置条件

1. 熟悉 TCP/IP
2. 了解 UDP
3. 有计算机网络基础

这里不做讲解了，有兴趣百度

> 但还是得致敬一下那些科学家吧，提供论文。

不讲UDP，下面所有的描述大部分是基于TCP，原因：

1. 99%的场景都是TCP，它更通用
2. UDP 仅适用于特定的场景：
    1. 帧同步，对传输速度有非常高的要求
    2. 音视频流，既想追求但速度传输文件过大，不过允许丢帧
3. UDP太复杂，因为要自实现 丢包重传、阻塞、面向连接等

## 概览

socket：网络中不同主机上的应用进程之间进行双向通信的实现

> 翻译：你如果想使用计算机网络传输东西，用这个 socket 就得了，它会给你露出几个API接口，直接调用，简单明了。

这里注意下，最早的商业化服务器 OS 是 unix，socket也是基于unix产生的，致敬下unix

> WIN/MAC 也是后来的产物，但UNIX更倾向专业人士，且没有 UI窗口化。

为啥一定要用socket?

> 它帮助你做了很多软硬件的工作。

1. 它有现成的代码/类库
2. 网卡如何收发数据
3. 数据传输中是否丢包
4. 双方以什么格式传输数据
5. 如何找到对方的地址
6. ......

> 个人觉得它最大的贡献是统一了规范\(前提是它确实好用\)，全世界的人都用它这一套代码规范，省了很多不必要的兼容/维护的麻烦

简单说说进程，它是由 OS 衍生而来，我们所有的程序，最终在 OS 层面，都是生成一个进程（分配内存、CPU执行、硬盘IO、权限等）。然后每个进程执行你自己的代码，可以进行网络通信。所以，从这个角度上来看，socket 也是进程间的通信。

> 进程间的通信不一定得走网络，还有IPC，有兴趣的自查可以

端口号，这东西，完全就是给网络用，不用网络的话，端口号没啥意义。给每个进程再标一个端口号，方便OS进行网络/进程 识别。

## socket 与 TCP/IP 的 关系

TCP/IP是协议规范，更像是论文，没到代码层面。而 socket 是通过代码实现了这套协议规范。

> 注：socket 不仅仅实现了 TCP/UDP， 还实现了IP层的一些协议。也可以理解为： socket 不完全等于 TCP/UDP

从实际角度上来看：当你使用 socket 后，基本就不太关心 TCP/UDP/IP 了，具体使用 TCP or UDP 只需要调用 socket 接口时，换个参数即可。

那么，实际上 socket 也可以理解为 操作 TCP/UDP/IP 协议的一个类库。

像：TCP的连接过程，IP路由等等这些，全被 socket 统一覆盖了

另外，我们听过 socket 编程，但很少听过 TCP 编程，我觉得从这个角度上来看，也是 socket 实现了 tcp/ip 协议。并在此基础上，可以做些编程如：自定义协议开发

从分层的角度看，socket 更倾向于在传输层的一系列操作。应用层是：HTTP FTP 这些软件，物理层是网卡/网线在操作。

## 大概的通信流程

![socket-1.jpg](image/socket-1.jpg)

> 从这里我们也看出，用户态的做的事情非常少，主要还是socket帮助我们做了大部分的事情。
> 
> 
> 再换个角度看，其实更像是：两台机器OS内核间的通信

## socket 编程 函数概览

> 这里以C语言函数为标准

先看下大概流程图：

S端

|函数名|描述                               |备注|
|------|-----------------------------------|----|
|bind  |此进程想要绑定当前机器的哪个端口号 |    |
|listen|开始监听这个 端口号发过来的网络操作|    |
|accept|接收连接成功后的FD                 |    |

C端

|函数名 |描述         |备注       |
|-------|-------------|-----------|
|connect|与S端建议连接|完成3次握手|

C/S共用

|函数名 |描述                                    |备注        |
|-------|----------------------------------------|------------|
|socket |创建一个 socket 结构体，并初始化基础字段|            |
|close  |                                        |            |
|read   |读取数据                                |阻塞        |
|write  |发送数据                                |阻塞        |
|recv   |读取数据                                |可选择不阻塞|
|send   |发送数据                                |可选择不阻塞|
|readv  |读取数据，可读入多个缓冲区              |            |
|writev |发送数据，可写入多个缓冲区              |            |
|recvmsg|读取数据                                |            |
|sendmsg|发送数据                                |            |

![cc91c731ebcb47be87d574670635ce29.jpg](image/cc91c731ebcb47be87d574670635ce29.jpg)

所有函数大体被分成了两类：

1. 面向连接
2. 面向IO

最核心的函数是：socket ，不管C还是S，且必须在最开始的时候，调用这个函数，后续所有的操作都要基于这个 函数返回值

我们看到：最终，还是IO函数 最多，基本上也就确定了，程序员日常操作都集中在IO层面。

## socket 函数

主要是创建一个 socket FD\(根据用户态的参数信息\)，大致如下：

1. 创建一个 scoket 结构体 并填充数据
2. 创建一个 sock 结构体 并填充数据，与socket结构体做关联
    1. 填充该结构的基础信息，如：IP、端口、协议类型等
    2. 给该结构体设置状态，如：连接状态
    3. 找到协议栈的代码，继续填充，如：TCP 的 SYN 包发送的头信息等
3. 创建 inet\_sock：TTL,组播列表，IP地址，端口

> 两端的基础网络信息放在这里\(尽量与协议无关\)

1. 创建 inet\_connection\_sock：面向连接的一些属性
2. 创建 tcp\_sock，TCP的一些专有属性，如：滑窗、拥塞控制

> tcp\_sock 继承 inet\_connection\_sock 继承 inet\_sock 继承 sock

用户态最终拿到的是 socket FD ，而实际大部分真实的操作是基于 sock 结构体

这里主要分析下：socket 和 与 sock 结构体

```
type  socket struct {
	short         		type;  //套接字所用的流类型
	socket_state  		state; //套接字所处状态
	long          		flags; //标识字段，目前尚无明确作用
	struct proto_ops  	*ops;  //操作函数集指针
	struct wait_queue 	**wait;//等待队列
	struct inode      	*inode;//inode结构指针
    struct sock		    *sk;
    struct socket_wq __rcu *sk_wq; ///套接字的等待队列
};
```

> socket 结构看着挺简单，它主要是透给用户态的，而 sock 结构是给内核态的

proto\_ops 就是你选择了什么协议类型，就找到该协议的一组操作函数

sk\_wq： epoll就是挂载在这个上面，一但读写缓冲区就绪，就会回调

inode 主要是把 socket 结构体与硬盘绑定最终以FD形式返回

```
enum sock_type
{
   SOCK_STREAM  = 1, // 用于与TCP层中的tcp协议数据的struct socket 
   SOCK_DGRAM   = 2, //用于与TCP层中的udp协议数据的struct socket 
   SOCK_RAW     = 3, // raw struct socket
   SOCK_RDM     = 4, //可靠传输消息的struct socket
   SOCK_SEQPACKET = 5,// sequential packet socket
   SOCK_DCCP    = 6,
   SOCK_PACKET  = 10, //从dev level中获取数据包的socket
};
```

> 感觉更像是协议类型，如：TCP/UDP

```
typedef enum {  
    SS_FREE = 0,            //该socket还未分配  
    SS_UNCONNECTED,         //未连向任何socket  
    SS_CONNECTING,          //正在连接过程中  
    SS_CONNECTED,           //已连向一个socket  
    SS_DISCONNECTING        //正在断开连接的过程中  
}socket_state;
```

> socket状态。这里发现，并没有 ESTABLISHED CLOSEING FIN\-WAIT\-1 常见状态，因为这些状态是专属于 TCP 协议的。而实际上这个状态才更通用化的。

```
struct socket 中的 flags 字段取值：
  #define SOCK_ASYNC_NOSPACE  0
  #define SOCK_ASYNC_WAITDATA 1
  #define SOCK_NOSPACE        2
  #define SOCK_PASSCRED       3
  #define SOCK_PASSSEC        4
```

sock 结构体 因为太大，列出主要的吧：

```
struct sock_common	__sk_common;	 // 网络层套接字通用结构体

socket_lock_t sk_lock;// 套接字同步锁
  
struct sk_buff_head	sk_receive_queue;// 收到的数据包队列
struct sk_buff_head	sk_write_queue;  // 发送队列/重传队列
struct sk_buff_head back_log;//当套接字忙时，数据包暂存这里

union {
    struct socket_wq __rcu    *sk_wq;//进程等待队列
    struct socket_wq    *sk_wq_raw;
};

unsigned long   sk_lingertime;//连接关闭前未发送数据的总数
struct sk_buff_head sk_error_queue;//错误链表，存放详细的出错信息。
long            sk_rcvtimeo;//读超时
long            sk_sndtimeo;//写超时
struct timer_list    sk_timer;//定时器，像3次握手等使用

    
```

sock 结构体 才是整个 网络传输的 核心，这里主要功能：

1. TCP 稳定控制：重传、滑窗、阻塞、排序
2. 收发数据
3. TCP 连接控制：建立连接、关闭连接、连接状态控制

> 一般来说 内核控制 sock 结构体，用户态控制的是 socket 结构体

sk\_lock:用户态读数据得加锁，因为这期间可能内核收到网卡信息，要写入缓存队列。另外，状态发生变更时，内核也会加锁。

sk\_wq:进程等待连接、等待输出缓冲区、等待读数据时，都会将进程暂存到此队列中

sk\_lingertime:这个参数引发一个问题，连接如果关闭，而还有数据未发送完，如何处理？好像内核有个参数，你开启它就等待一段时间 你关闭他就直接把数据丢了，非优雅关闭。

sk\_rcvtimeo:这个参数有意思。socket 本身就是阻塞模式\(epool seelect recv阶外\)的，像：accept/connect/send/recv这些都是阻塞/半阻塞的。如果设置这个参数就可以非阻塞。但感觉可能还是有性能的牺牲或者得会用，知道怎么用，不然还是默认值吧。

sock\_common

```
union {
        __addrpair    skc_addrpair;
        struct {
            __be32    skc_daddr;//远端地址 IP4本地地址
            __be32    skc_rcv_saddr;//本地地址 IP4本地地址
        };
};

unsigned short      skc_family;         //地址族
volatile unsigned char  skc_state;      //连接状态
unsigned char       skc_reuse;          //SO_REUSEADDR设置
int         		skc_bound_dev_if;
struct hlist_node   skc_node;
struct hlist_node   skc_bind_node;      //哈希表相关
atomic_t        	skc_refcnt;         //引用计数
```

这里是些基础的信息，还有连接状态等。sock 结构体也算是继承了些结构体吧

#### socket sock 结构体 小结

其实，unix/linux 内核在实现 socket 中还有N多个结构体，还牵扯到 网卡、驱动、中断机制等等吧，我这里也只是列出2个关键的结构，太细节的我也不懂，也没时间看。

但是从 sock 这个结构体来分析看，它其实就是把网络进行又一次分层，封装，达到最终只露出几个公共接口给程序员使用。

其中的核心就是：通过几个发/发缓冲数据队列，内核再睡眠/唤醒进程，就实现了....

## 数据收发队列

它主要是包含了4个队列：

1. backlog：用户正在占用 socket ，可能正在读操作，数据暂时放到backlog，如果满了就直接丢弃数据。数据：可能乱序，还有TCP头信息
2. prequeue:
3. sk\_receive\_queue：最干净的数据，没有TCP协议头、顺序正确、校验过的，用户可直接使用
4. out\_of\_order：乱序报文

> 以上队列是有优先集之分的，级别越高，越先处理。也就是序号大的

正常情况

> 内核肯定是希望来的数据都是正常的，直接扔到 sk\_receive\_queue 中，用户直接可以取走的。

数据乱序情况

1. A给B~发送3段报文 1 2 3
2. 报文3先到的，用户态希望的是报文1，所以 不能直接读取。把3报文扔到 out\_of\_order 中
3. 报文1 来了，2也来了，都扔到 sk\_receive\_queue 中
4. 从 out\_of\_order 中把3取出来，扔到 receive\_queue 中。

socket 被用户态给锁住了

> 用户态正在操作 socket 读数据中，此时有消息过来了，直接扔到 backlog 中

读数据阻塞中

1. 用户态开始读数据，此时没有数据了，它就放弃锁，然后，进入睡眠状态，等待新数据过来 。
2. 此时，有新数据来了，因为用户没加锁，也没有正在读，新来的数据不会进入backlog。这条数据会插入到 prequeue 中，
3. prequeue 有新数据会唤醒该进程，唤醒后立刻加锁
4. 先读 sk\_receive\_queue 数据 ，再读 prequeue ，最后读 backlog

总结：

通过一个锁，用户态读取操\(可能会操作 sk\_receive\_queue prequeue \)作就会加锁，新来的数据就得进入 backlog。用户态没有加锁的时候，内核就可以直接操作 sk\_receive\_queue \+ out\_of\_order。而用户态没加锁但是阻塞模式下，就是操作 prequeue。

## bind 函数

把要监听 的IP 端口号，更新到 sock 结构体内，listen 函数执行之前必须得调用 bind 函数

## listen 函数

1. 找到 sock 结构 ，
2. 找到 inet\_connection\_sock 结构 ，找到 icsk\_accept\_queue 成员
3. 创建 request\_sock\_quue
    1. 创建半连接队列
    2. 创建全连接队列

request\_sock\_quue 队列

```
struct request_sock_queue {
    struct request_sock	*rskq_accept_head;
    struct request_sock	*rskq_accept_tail;
    rwlock_t		syn_wait_lock;
    u8			rskq_defer_accept;
    /* 3 bytes hole, try to pack */
    struct listen_sock	*listen_opt;
};
```

request\_sock 全连接队列

listen\_sock 半连接队列，关键参数参数：backlog ，半连接队列最大长度

两个队列里存储的成员都是：requset\_sock 结构体

```
struct request_sock {
	struct sock_common		__req_common;
    ......
};
```

没用的我就没记，其中好像就一个是值得说的：sock\_common ，基本上它就是一个小型时 sock 结构体，也能理解，毕竟还到正式使用，甚至可能还是半连接状态 ，没必要创建一个完整的 sock 结构体。它只要能表示清楚：要连接方的基础信息5元组即可。

当半连接完成后，进入 全连接后，既：把这个成员从半连接队列转移到全连接状态后。 accept 函数就能拿到此socket fd ，但在这这前，内核会把这个 request\_sock 进行完整化，如：创建 socket sock 结构体，绑定FD等等。

## accept

从处于 established 状态的连接队列头部取出一个已经完成的连接，如果这个队列没有已经完成的连接，accept\(\)函数就会阻塞，直到取出队列中已完成的用户连接为止。另：此函数会返回一个新的FD

这里很有趣，我们S端正常要监听，得主动创建一个 socket FD，但是 accept 会源原不断的返回新的 socket FD ，有啥区别？

## 读写缓存区

内核给每一个TCP套接字都有一个发送缓冲区

SO\_SNDBUF控制大小。调用send发送数据时\(默认阻塞模式\)，如果缓冲区没满，调用直接返回。系统内核在IP层发送数据的时候，并不是按照我们调用send接口发送的数据包大小来进行发送，即使我们调用send发送的数据大小小于1460字节\(MTU\-TCP首部\-IP首部\)。因为我们调用send接口实际是将数据复制到缓冲区中，而内核基本上是按照最大MSS大小\(1460字节\)从缓冲区中取数据发送出去，当缓冲区中数据小于MSS，则将剩余数据全部发送出去。TCP的发送缓冲区必须为已发送的数据保留一个副本，直到它被对端确认为止，才能从缓冲区中删掉已确认的数据。

内核给每一个TCP套接字都有一个接收缓冲区，

可以通过SO\_RCVBUF套接字选项来更改。接收缓冲区被TCP用来保存接收到的数据，直到应用程序来读取。对于TCP来说，接收缓冲区中可用空间的大小限定了TCP通告对端的窗口大小。TCP套接字的接收缓冲区不能溢出，所以发送端不能发送超过接收端通知的窗口大小，否则在接收端将丢弃数据包。收端通知发端，接收窗口关闭（win=0），从而保证了TCP是可靠传输

以上两个都以队列维护

## 阻塞

先区分4个概念：

1. 阻塞\(blocking\)：程序执行到某个点~等待着结果返回不动了，进程被挂起。如：accept、recv等
2. 非阻塞\(non\-blocking\)：执行到某个点，立刻返回，不管其它的。
3. 同步\(synchronous\)：程序执行到某个点~等待着结果返回不动了，进程依然处于活动状态
4. 异步\(asynchronous\)：执行到某个点，立刻返回，不管其它的。

往底层次看的话，阻塞其实是：当进程的代码，被调度到 CPU上，开始执行。在执行的过程中，发生了睡眠，如：sleep http\-request 等，比较耗时的操作或主动睡眠，此时 OS 就把该进程扔到睡眠队列中，更新进程状态。让出 CPU 执行权限，给到其它进程继续执行。

所以阻塞这东西，更像是OS为了平衡进程执行时间而产生的。

> 当然也有主动的，如：SLEEP 这种不讨论

## epoll

sk\_wq

## 总结

为什么一定要内核介入？

因为 网络IO过于复杂，牵扯了大部分计算机资源，包括硬件，如网卡。需要的权限过多，内核直接来参与更方便。最终，通过通过 socket API 给到用户

从用户的角度看，socket 更像是一个文件的IO，这也是LINUX哲学的：万物皆文件。收消息 是读 ，发消息是写。

另外，从多协议/分层 来看，不管什么协议最终要实现的目标就是：收/发消息。

%E5%89%8D%E7%BD%AE%E6%9D%A1%E4%BB%B6%0A\-\-\-%0A1.%20%E7%86%9F%E6%82%89%20TCP%2FIP%0A2.%20%E4%BA%86%E8%A7%A3%20UDP%0A3.%20%E6%9C%89%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80%0A%0A%E8%BF%99%E9%87%8C%E4%B8%8D%E5%81%9A%E8%AE%B2%E8%A7%A3%E4%BA%86%EF%BC%8C%E6%9C%89%E5%85%B4%E8%B6%A3%E7%99%BE%E5%BA%A6%0A%3E%E4%BD%86%E8%BF%98%E6%98%AF%E5%BE%97%E8%87%B4%E6%95%AC%E4%B8%80%E4%B8%8B%E9%82%A3%E4%BA%9B%E7%A7%91%E5%AD%A6%E5%AE%B6%E5%90%A7%EF%BC%8C%E6%8F%90%E4%BE%9B%E8%AE%BA%E6%96%87%E3%80%82%0A%0A%E4%B8%8D%E8%AE%B2UDP%EF%BC%8C%E4%B8%8B%E9%9D%A2%E6%89%80%E6%9C%89%E7%9A%84%E6%8F%8F%E8%BF%B0%E5%A4%A7%E9%83%A8%E5%88%86%E6%98%AF%E5%9F%BA%E4%BA%8ETCP%EF%BC%8C%E5%8E%9F%E5%9B%A0%EF%BC%9A%0A1.%2099%25%E7%9A%84%E5%9C%BA%E6%99%AF%E9%83%BD%E6%98%AFTCP%EF%BC%8C%E5%AE%83%E6%9B%B4%E9%80%9A%E7%94%A8%0A2.%20UDP%20%E4%BB%85%E9%80%82%E7%94%A8%E4%BA%8E%E7%89%B9%E5%AE%9A%E7%9A%84%E5%9C%BA%E6%99%AF%EF%BC%9A%0A%20%20%20%201.%20%E5%B8%A7%E5%90%8C%E6%AD%A5%EF%BC%8C%E5%AF%B9%E4%BC%A0%E8%BE%93%E9%80%9F%E5%BA%A6%E6%9C%89%E9%9D%9E%E5%B8%B8%E9%AB%98%E7%9A%84%E8%A6%81%E6%B1%82%0A%20%20%20%202.%20%E9%9F%B3%E8%A7%86%E9%A2%91%E6%B5%81%EF%BC%8C%E6%97%A2%E6%83%B3%E8%BF%BD%E6%B1%82%E4%BD%86%E9%80%9F%E5%BA%A6%E4%BC%A0%E8%BE%93%E6%96%87%E4%BB%B6%E8%BF%87%E5%A4%A7%EF%BC%8C%E4%B8%8D%E8%BF%87%E5%85%81%E8%AE%B8%E4%B8%A2%E5%B8%A7%0A3.%20UDP%E5%A4%AA%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%A6%81%E8%87%AA%E5%AE%9E%E7%8E%B0%20%E4%B8%A2%E5%8C%85%E9%87%8D%E4%BC%A0%E3%80%81%E9%98%BB%E5%A1%9E%E3%80%81%E9%9D%A2%E5%90%91%E8%BF%9E%E6%8E%A5%E7%AD%89%20%20%20%0A%20%20%20%0A%0A%E6%A6%82%E8%A7%88%0A\-\-\-\-%0Asocket%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%AD%E4%B8%8D%E5%90%8C%E4%B8%BB%E6%9C%BA%E4%B8%8A%E7%9A%84%E5%BA%94%E7%94%A8%E8%BF%9B%E7%A8%8B%E4%B9%8B%E9%97%B4%E8%BF%9B%E8%A1%8C%E5%8F%8C%E5%90%91%E9%80%9A%E4%BF%A1%E7%9A%84%E5%AE%9E%E7%8E%B0%0A%3E%E7%BF%BB%E8%AF%91%EF%BC%9A%E4%BD%A0%E5%A6%82%E6%9E%9C%E6%83%B3%E4%BD%BF%E7%94%A8%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E4%B8%9C%E8%A5%BF%EF%BC%8C%E7%94%A8%E8%BF%99%E4%B8%AA%20socket%20%E5%B0%B1%E5%BE%97%E4%BA%86%EF%BC%8C%E5%AE%83%E4%BC%9A%E7%BB%99%E4%BD%A0%E9%9C%B2%E5%87%BA%E5%87%A0%E4%B8%AAAPI%E6%8E%A5%E5%8F%A3%EF%BC%8C%E7%9B%B4%E6%8E%A5%E8%B0%83%E7%94%A8%EF%BC%8C%E7%AE%80%E5%8D%95%E6%98%8E%E4%BA%86%E3%80%82%0A%0A%E8%BF%99%E9%87%8C%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%8C%E6%9C%80%E6%97%A9%E7%9A%84%E5%95%86%E4%B8%9A%E5%8C%96%E6%9C%8D%E5%8A%A1%E5%99%A8%20OS%20%E6%98%AF%20unix%EF%BC%8Csocket%E4%B9%9F%E6%98%AF%E5%9F%BA%E4%BA%8Eunix%E4%BA%A7%E7%94%9F%E7%9A%84%EF%BC%8C%E8%87%B4%E6%95%AC%E4%B8%8Bunix%0A%3EWIN%2FMAC%20%E4%B9%9F%E6%98%AF%E5%90%8E%E6%9D%A5%E7%9A%84%E4%BA%A7%E7%89%A9%EF%BC%8C%E4%BD%86UNIX%E6%9B%B4%E5%80%BE%E5%90%91%E4%B8%93%E4%B8%9A%E4%BA%BA%E5%A3%AB%EF%BC%8C%E4%B8%94%E6%B2%A1%E6%9C%89%20UI%E7%AA%97%E5%8F%A3%E5%8C%96%E3%80%82%0A%0A%E4%B8%BA%E5%95%A5%E4%B8%80%E5%AE%9A%E8%A6%81%E7%94%A8socket%3F%0A%3E%E5%AE%83%E5%B8%AE%E5%8A%A9%E4%BD%A0%E5%81%9A%E4%BA%86%E5%BE%88%E5%A4%9A%E8%BD%AF%E7%A1%AC%E4%BB%B6%E7%9A%84%E5%B7%A5%E4%BD%9C%E3%80%82%0A1.%20%E5%AE%83%E6%9C%89%E7%8E%B0%E6%88%90%E7%9A%84%E4%BB%A3%E7%A0%81%2F%E7%B1%BB%E5%BA%93%0A2.%20%E7%BD%91%E5%8D%A1%E5%A6%82%E4%BD%95%E6%94%B6%E5%8F%91%E6%95%B0%E6%8D%AE%0A3.%20%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E4%B8%AD%E6%98%AF%E5%90%A6%E4%B8%A2%E5%8C%85%0A3.%20%E5%8F%8C%E6%96%B9%E4%BB%A5%E4%BB%80%E4%B9%88%E6%A0%BC%E5%BC%8F%E4%BC%A0%E8%BE%93%E6%95%B0%E6%8D%AE%0A4.%20%E5%A6%82%E4%BD%95%E6%89%BE%E5%88%B0%E5%AF%B9%E6%96%B9%E7%9A%84%E5%9C%B0%E5%9D%80%0A6.%20......%0A%0A%3E%E4%B8%AA%E4%BA%BA%E8%A7%89%E5%BE%97%E5%AE%83%E6%9C%80%E5%A4%A7%E7%9A%84%E8%B4%A1%E7%8C%AE%E6%98%AF%E7%BB%9F%E4%B8%80%E4%BA%86%E8%A7%84%E8%8C%83\(%E5%89%8D%E6%8F%90%E6%98%AF%E5%AE%83%E7%A1%AE%E5%AE%9E%E5%A5%BD%E7%94%A8\)%EF%BC%8C%E5%85%A8%E4%B8%96%E7%95%8C%E7%9A%84%E4%BA%BA%E9%83%BD%E7%94%A8%E5%AE%83%E8%BF%99%E4%B8%80%E5%A5%97%E4%BB%A3%E7%A0%81%E8%A7%84%E8%8C%83%EF%BC%8C%E7%9C%81%E4%BA%86%E5%BE%88%E5%A4%9A%E4%B8%8D%E5%BF%85%E8%A6%81%E7%9A%84%E5%85%BC%E5%AE%B9%2F%E7%BB%B4%E6%8A%A4%E7%9A%84%E9%BA%BB%E7%83%A6%0A%0A%E7%AE%80%E5%8D%95%E8%AF%B4%E8%AF%B4%E8%BF%9B%E7%A8%8B%EF%BC%8C%E5%AE%83%E6%98%AF%E7%94%B1%20OS%20%E8%A1%8D%E7%94%9F%E8%80%8C%E6%9D%A5%EF%BC%8C%E6%88%91%E4%BB%AC%E6%89%80%E6%9C%89%E7%9A%84%E7%A8%8B%E5%BA%8F%EF%BC%8C%E6%9C%80%E7%BB%88%E5%9C%A8%20OS%20%E5%B1%82%E9%9D%A2%EF%BC%8C%E9%83%BD%E6%98%AF%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AA%E8%BF%9B%E7%A8%8B%EF%BC%88%E5%88%86%E9%85%8D%E5%86%85%E5%AD%98%E3%80%81CPU%E6%89%A7%E8%A1%8C%E3%80%81%E7%A1%AC%E7%9B%98IO%E3%80%81%E6%9D%83%E9%99%90%E7%AD%89%EF%BC%89%E3%80%82%E7%84%B6%E5%90%8E%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E6%89%A7%E8%A1%8C%E4%BD%A0%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%8F%AF%E4%BB%A5%E8%BF%9B%E8%A1%8C%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E3%80%82%E6%89%80%E4%BB%A5%EF%BC%8C%E4%BB%8E%E8%BF%99%E4%B8%AA%E8%A7%92%E5%BA%A6%E4%B8%8A%E6%9D%A5%E7%9C%8B%EF%BC%8Csocket%20%E4%B9%9F%E6%98%AF%E8%BF%9B%E7%A8%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1%E3%80%82%0A%3E%E8%BF%9B%E7%A8%8B%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1%E4%B8%8D%E4%B8%80%E5%AE%9A%E5%BE%97%E8%B5%B0%E7%BD%91%E7%BB%9C%EF%BC%8C%E8%BF%98%E6%9C%89IPC%EF%BC%8C%E6%9C%89%E5%85%B4%E8%B6%A3%E7%9A%84%E8%87%AA%E6%9F%A5%E5%8F%AF%E4%BB%A5%0A%0A%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%8C%E8%BF%99%E4%B8%9C%E8%A5%BF%EF%BC%8C%E5%AE%8C%E5%85%A8%E5%B0%B1%E6%98%AF%E7%BB%99%E7%BD%91%E7%BB%9C%E7%94%A8%EF%BC%8C%E4%B8%8D%E7%94%A8%E7%BD%91%E7%BB%9C%E7%9A%84%E8%AF%9D%EF%BC%8C%E7%AB%AF%E5%8F%A3%E5%8F%B7%E6%B2%A1%E5%95%A5%E6%84%8F%E4%B9%89%E3%80%82%E7%BB%99%E6%AF%8F%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%86%8D%E6%A0%87%E4%B8%80%E4%B8%AA%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%8C%E6%96%B9%E4%BE%BFOS%E8%BF%9B%E8%A1%8C%E7%BD%91%E7%BB%9C%2F%E8%BF%9B%E7%A8%8B%20%E8%AF%86%E5%88%AB%E3%80%82%0A%0A%0Asocket%20%E4%B8%8E%20TCP%2FIP%20%E7%9A%84%20%E5%85%B3%E7%B3%BB%0A\-\-\-\-%0ATCP%2FIP%E6%98%AF%E5%8D%8F%E8%AE%AE%E8%A7%84%E8%8C%83%EF%BC%8C%E6%9B%B4%E5%83%8F%E6%98%AF%E8%AE%BA%E6%96%87%EF%BC%8C%E6%B2%A1%E5%88%B0%E4%BB%A3%E7%A0%81%E5%B1%82%E9%9D%A2%E3%80%82%E8%80%8C%20socket%20%E6%98%AF%E9%80%9A%E8%BF%87%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0%E4%BA%86%E8%BF%99%E5%A5%97%E5%8D%8F%E8%AE%AE%E8%A7%84%E8%8C%83%E3%80%82%0A%0A%3E%E6%B3%A8%EF%BC%9Asocket%20%E4%B8%8D%E4%BB%85%E4%BB%85%E5%AE%9E%E7%8E%B0%E4%BA%86%20TCP%2FUDP%EF%BC%8C%20%E8%BF%98%E5%AE%9E%E7%8E%B0%E4%BA%86IP%E5%B1%82%E7%9A%84%E4%B8%80%E4%BA%9B%E5%8D%8F%E8%AE%AE%E3%80%82%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%EF%BC%9A%20socket%20%E4%B8%8D%E5%AE%8C%E5%85%A8%E7%AD%89%E4%BA%8E%20TCP%2FUDP%20%0A%0A%0A%E4%BB%8E%E5%AE%9E%E9%99%85%E8%A7%92%E5%BA%A6%E4%B8%8A%E6%9D%A5%E7%9C%8B%EF%BC%9A%E5%BD%93%E4%BD%A0%E4%BD%BF%E7%94%A8%20socket%20%E5%90%8E%EF%BC%8C%E5%9F%BA%E6%9C%AC%E5%B0%B1%E4%B8%8D%E5%A4%AA%E5%85%B3%E5%BF%83%20TCP%2FUDP%2FIP%20%E4%BA%86%EF%BC%8C%E5%85%B7%E4%BD%93%E4%BD%BF%E7%94%A8%20TCP%20or%20UDP%20%E5%8F%AA%E9%9C%80%E8%A6%81%E8%B0%83%E7%94%A8%20socket%20%E6%8E%A5%E5%8F%A3%E6%97%B6%EF%BC%8C%E6%8D%A2%E4%B8%AA%E5%8F%82%E6%95%B0%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%E9%82%A3%E4%B9%88%EF%BC%8C%E5%AE%9E%E9%99%85%E4%B8%8A%20socket%20%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%20%E6%93%8D%E4%BD%9C%20TCP%2FUDP%2FIP%20%E5%8D%8F%E8%AE%AE%E7%9A%84%E4%B8%80%E4%B8%AA%E7%B1%BB%E5%BA%93%E3%80%82%0A%E5%83%8F%EF%BC%9ATCP%E7%9A%84%E8%BF%9E%E6%8E%A5%E8%BF%87%E7%A8%8B%EF%BC%8CIP%E8%B7%AF%E7%94%B1%E7%AD%89%E7%AD%89%E8%BF%99%E4%BA%9B%EF%BC%8C%E5%85%A8%E8%A2%AB%20socket%20%E7%BB%9F%E4%B8%80%E8%A6%86%E7%9B%96%E4%BA%86%0A%0A%0A%E5%8F%A6%E5%A4%96%EF%BC%8C%E6%88%91%E4%BB%AC%E5%90%AC%E8%BF%87%20socket%20%E7%BC%96%E7%A8%8B%EF%BC%8C%E4%BD%86%E5%BE%88%E5%B0%91%E5%90%AC%E8%BF%87%20TCP%20%E7%BC%96%E7%A8%8B%EF%BC%8C%E6%88%91%E8%A7%89%E5%BE%97%E4%BB%8E%E8%BF%99%E4%B8%AA%E8%A7%92%E5%BA%A6%E4%B8%8A%E6%9D%A5%E7%9C%8B%EF%BC%8C%E4%B9%9F%E6%98%AF%20socket%20%E5%AE%9E%E7%8E%B0%E4%BA%86%20tcp%2Fip%20%E5%8D%8F%E8%AE%AE%E3%80%82%E5%B9%B6%E5%9C%A8%E6%AD%A4%E5%9F%BA%E7%A1%80%E4%B8%8A%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%81%9A%E4%BA%9B%E7%BC%96%E7%A8%8B%E5%A6%82%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E5%8D%8F%E8%AE%AE%E5%BC%80%E5%8F%91%0A%0A%E4%BB%8E%E5%88%86%E5%B1%82%E7%9A%84%E8%A7%92%E5%BA%A6%E7%9C%8B%EF%BC%8Csocket%20%E6%9B%B4%E5%80%BE%E5%90%91%E4%BA%8E%E5%9C%A8%E4%BC%A0%E8%BE%93%E5%B1%82%E7%9A%84%E4%B8%80%E7%B3%BB%E5%88%97%E6%93%8D%E4%BD%9C%E3%80%82%E5%BA%94%E7%94%A8%E5%B1%82%E6%98%AF%EF%BC%9AHTTP%20FTP%20%E8%BF%99%E4%BA%9B%E8%BD%AF%E4%BB%B6%EF%BC%8C%E7%89%A9%E7%90%86%E5%B1%82%E6%98%AF%E7%BD%91%E5%8D%A1%2F%E7%BD%91%E7%BA%BF%E5%9C%A8%E6%93%8D%E4%BD%9C%E3%80%82%0A%0A%E5%A4%A7%E6%A6%82%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B%0A\-\-\-%0A\!%5B509d38472e3745c3527d26296033c044.jpeg%5D\(evernotecid%3A%2F%2F10DE096E\-2A4C\-4D0F\-BF3F\-D635A56EE3B6%2Fappyinxiangcom%2F35868219%2FENResource%2Fp105\)%0A%0A%3E%E4%BB%8E%E8%BF%99%E9%87%8C%E6%88%91%E4%BB%AC%E4%B9%9F%E7%9C%8B%E5%87%BA%EF%BC%8C%E7%94%A8%E6%88%B7%E6%80%81%E7%9A%84%E5%81%9A%E7%9A%84%E4%BA%8B%E6%83%85%E9%9D%9E%E5%B8%B8%E5%B0%91%EF%BC%8C%E4%B8%BB%E8%A6%81%E8%BF%98%E6%98%AFsocket%E5%B8%AE%E5%8A%A9%E6%88%91%E4%BB%AC%E5%81%9A%E4%BA%86%E5%A4%A7%E9%83%A8%E5%88%86%E7%9A%84%E4%BA%8B%E6%83%85%E3%80%82%0A%3E%E5%86%8D%E6%8D%A2%E4%B8%AA%E8%A7%92%E5%BA%A6%E7%9C%8B%EF%BC%8C%E5%85%B6%E5%AE%9E%E6%9B%B4%E5%83%8F%E6%98%AF%EF%BC%9A%E4%B8%A4%E5%8F%B0%E6%9C%BA%E5%99%A8OS%E5%86%85%E6%A0%B8%E9%97%B4%E7%9A%84%E9%80%9A%E4%BF%A1%0A%0A%0A%0Asocket%20%E7%BC%96%E7%A8%8B%20%E5%87%BD%E6%95%B0%E6%A6%82%E8%A7%88%0A\-\-\-\-\-%0A%3E%E8%BF%99%E9%87%8C%E4%BB%A5C%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0%E4%B8%BA%E6%A0%87%E5%87%86%0A%0A%E5%85%88%E7%9C%8B%E4%B8%8B%E5%A4%A7%E6%A6%82%E6%B5%81%E7%A8%8B%E5%9B%BE%EF%BC%9A%0A%0AS%E7%AB%AF%0A%0A%7C%20%E5%87%BD%E6%95%B0%E5%90%8D%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%20%E5%A4%87%E6%B3%A8%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20bind%20%7C%E6%AD%A4%E8%BF%9B%E7%A8%8B%E6%83%B3%E8%A6%81%E7%BB%91%E5%AE%9A%E5%BD%93%E5%89%8D%E6%9C%BA%E5%99%A8%E7%9A%84%E5%93%AA%E4%B8%AA%E7%AB%AF%E5%8F%A3%E5%8F%B7%20%20%7C%20%20%7C%0A%7C%20listen%20%7C%20%E5%BC%80%E5%A7%8B%E7%9B%91%E5%90%AC%E8%BF%99%E4%B8%AA%20%E7%AB%AF%E5%8F%A3%E5%8F%B7%E5%8F%91%E8%BF%87%E6%9D%A5%E7%9A%84%E7%BD%91%E7%BB%9C%E6%93%8D%E4%BD%9C%20%7C%20%20%7C%0A%7C%20accept%20%7C%20%20%E6%8E%A5%E6%94%B6%E8%BF%9E%E6%8E%A5%E6%88%90%E5%8A%9F%E5%90%8E%E7%9A%84FD%7C%20%20%7C%0A%0A%0AC%E7%AB%AF%0A%0A%7C%20%E5%87%BD%E6%95%B0%E5%90%8D%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%20%E5%A4%87%E6%B3%A8%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20connect%20%7C%20%E4%B8%8ES%E7%AB%AF%E5%BB%BA%E8%AE%AE%E8%BF%9E%E6%8E%A5%20%7C%20%E5%AE%8C%E6%88%903%E6%AC%A1%E6%8F%A1%E6%89%8B%20%7C%0A%0A%0AC%2FS%E5%85%B1%E7%94%A8%0A%0A%7C%20%E5%87%BD%E6%95%B0%E5%90%8D%20%7C%20%E6%8F%8F%E8%BF%B0%20%7C%20%E5%A4%87%E6%B3%A8%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20socket%20%7C%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20socket%20%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E5%B9%B6%E5%88%9D%E5%A7%8B%E5%8C%96%E5%9F%BA%E7%A1%80%E5%AD%97%E6%AE%B5%20%7C%20%20%7C%0A%7C%20close%20%7C%20%20%7C%20%20%7C%0A%7C%20read%20%20%7C%20%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE%20%7C%20%E9%98%BB%E5%A1%9E%20%7C%0A%7C%20write%20%7C%20%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%20%7C%E9%98%BB%E5%A1%9E%20%20%7C%0A%7C%20recv%20%7C%20%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE%20%7C%20%E5%8F%AF%E9%80%89%E6%8B%A9%E4%B8%8D%E9%98%BB%E5%A1%9E%20%7C%0A%7C%20send%20%7C%20%20%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%7C%20%E5%8F%AF%E9%80%89%E6%8B%A9%E4%B8%8D%E9%98%BB%E5%A1%9E%20%7C%0A%7Creadv%7C%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%8F%AF%E8%AF%BB%E5%85%A5%E5%A4%9A%E4%B8%AA%E7%BC%93%E5%86%B2%E5%8C%BA%7C%0A%7C%20writev%20%7C%20%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%8F%AF%E5%86%99%E5%85%A5%E5%A4%9A%E4%B8%AA%E7%BC%93%E5%86%B2%E5%8C%BA%20%7C%20%20%7C%0A%7Crecvmsg%7C%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE%7C%0A%7Csendmsg%7C%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%7C%0A%0A\!%5Bcc91c731ebcb47be87d574670635ce29.jpeg%5D\(evernotecid%3A%2F%2F10DE096E\-2A4C\-4D0F\-BF3F\-D635A56EE3B6%2Fappyinxiangcom%2F35868219%2FENResource%2Fp106\)%0A%0A%0A%E6%89%80%E6%9C%89%E5%87%BD%E6%95%B0%E5%A4%A7%E4%BD%93%E8%A2%AB%E5%88%86%E6%88%90%E4%BA%86%E4%B8%A4%E7%B1%BB%EF%BC%9A%0A1.%20%E9%9D%A2%E5%90%91%E8%BF%9E%E6%8E%A5%0A2.%20%E9%9D%A2%E5%90%91IO%0A%0A%E6%9C%80%E6%A0%B8%E5%BF%83%E7%9A%84%E5%87%BD%E6%95%B0%E6%98%AF%EF%BC%9Asocket%20%EF%BC%8C%E4%B8%8D%E7%AE%A1C%E8%BF%98%E6%98%AFS%EF%BC%8C%E4%B8%94%E5%BF%85%E9%A1%BB%E5%9C%A8%E6%9C%80%E5%BC%80%E5%A7%8B%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E8%B0%83%E7%94%A8%E8%BF%99%E4%B8%AA%E5%87%BD%E6%95%B0%EF%BC%8C%E5%90%8E%E7%BB%AD%E6%89%80%E6%9C%89%E7%9A%84%E6%93%8D%E4%BD%9C%E9%83%BD%E8%A6%81%E5%9F%BA%E4%BA%8E%E8%BF%99%E4%B8%AA%20%E5%87%BD%E6%95%B0%E8%BF%94%E5%9B%9E%E5%80%BC%0A%0A%E6%88%91%E4%BB%AC%E7%9C%8B%E5%88%B0%EF%BC%9A%E6%9C%80%E7%BB%88%EF%BC%8C%E8%BF%98%E6%98%AFIO%E5%87%BD%E6%95%B0%20%E6%9C%80%E5%A4%9A%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E4%B9%9F%E5%B0%B1%E7%A1%AE%E5%AE%9A%E4%BA%86%EF%BC%8C%E7%A8%8B%E5%BA%8F%E5%91%98%E6%97%A5%E5%B8%B8%E6%93%8D%E4%BD%9C%E9%83%BD%E9%9B%86%E4%B8%AD%E5%9C%A8IO%E5%B1%82%E9%9D%A2%E3%80%82%0A%0A%0Asocket%20%E5%87%BD%E6%95%B0%0A\-\-\-\-\-%0A%E4%B8%BB%E8%A6%81%E6%98%AF%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20socket%20FD\(%E6%A0%B9%E6%8D%AE%E7%94%A8%E6%88%B7%E6%80%81%E7%9A%84%E5%8F%82%E6%95%B0%E4%BF%A1%E6%81%AF\)%EF%BC%8C%E5%A4%A7%E8%87%B4%E5%A6%82%E4%B8%8B%EF%BC%9A%0A1.%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20scoket%20%E7%BB%93%E6%9E%84%E4%BD%93%20%E5%B9%B6%E5%A1%AB%E5%85%85%E6%95%B0%E6%8D%AE%0A2.%20%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%20%E5%B9%B6%E5%A1%AB%E5%85%85%E6%95%B0%E6%8D%AE%EF%BC%8C%E4%B8%8Esocket%E7%BB%93%E6%9E%84%E4%BD%93%E5%81%9A%E5%85%B3%E8%81%94%0A%20%20%20%201.%20%E5%A1%AB%E5%85%85%E8%AF%A5%E7%BB%93%E6%9E%84%E7%9A%84%E5%9F%BA%E7%A1%80%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%A6%82%EF%BC%9AIP%E3%80%81%E7%AB%AF%E5%8F%A3%E3%80%81%E5%8D%8F%E8%AE%AE%E7%B1%BB%E5%9E%8B%E7%AD%89%0A%20%20%20%202.%20%E7%BB%99%E8%AF%A5%E7%BB%93%E6%9E%84%E4%BD%93%E8%AE%BE%E7%BD%AE%E7%8A%B6%E6%80%81%EF%BC%8C%E5%A6%82%EF%BC%9A%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%0A%20%20%20%203.%20%E6%89%BE%E5%88%B0%E5%8D%8F%E8%AE%AE%E6%A0%88%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E7%BB%A7%E7%BB%AD%E5%A1%AB%E5%85%85%EF%BC%8C%E5%A6%82%EF%BC%9ATCP%20%E7%9A%84%20SYN%20%E5%8C%85%E5%8F%91%E9%80%81%E7%9A%84%E5%A4%B4%E4%BF%A1%E6%81%AF%E7%AD%89%0A3.%20%E5%88%9B%E5%BB%BA%20inet\_sock%EF%BC%9ATTL%2C%E7%BB%84%E6%92%AD%E5%88%97%E8%A1%A8%EF%BC%8CIP%E5%9C%B0%E5%9D%80%EF%BC%8C%E7%AB%AF%E5%8F%A3%0A%3E%E4%B8%A4%E7%AB%AF%E7%9A%84%E5%9F%BA%E7%A1%80%E7%BD%91%E7%BB%9C%E4%BF%A1%E6%81%AF%E6%94%BE%E5%9C%A8%E8%BF%99%E9%87%8C\(%E5%B0%BD%E9%87%8F%E4%B8%8E%E5%8D%8F%E8%AE%AE%E6%97%A0%E5%85%B3\)%0A4.%20%E5%88%9B%E5%BB%BA%20inet\_connection\_sock%EF%BC%9A%E9%9D%A2%E5%90%91%E8%BF%9E%E6%8E%A5%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B1%9E%E6%80%A7%0A5.%20%E5%88%9B%E5%BB%BA%20tcp\_sock%EF%BC%8CTCP%E7%9A%84%E4%B8%80%E4%BA%9B%E4%B8%93%E6%9C%89%E5%B1%9E%E6%80%A7%EF%BC%8C%E5%A6%82%EF%BC%9A%E6%BB%91%E7%AA%97%E3%80%81%E6%8B%A5%E5%A1%9E%E6%8E%A7%E5%88%B6%0A%0A%3Etcp\_sock%20%E7%BB%A7%E6%89%BF%20inet\_connection\_sock%20%E7%BB%A7%E6%89%BF%20%20inet\_sock%20%20%E7%BB%A7%E6%89%BF%20sock%0A%0A%E7%94%A8%E6%88%B7%E6%80%81%E6%9C%80%E7%BB%88%E6%8B%BF%E5%88%B0%E7%9A%84%E6%98%AF%20socket%20FD%20%EF%BC%8C%E8%80%8C%E5%AE%9E%E9%99%85%E5%A4%A7%E9%83%A8%E5%88%86%E7%9C%9F%E5%AE%9E%E7%9A%84%E6%93%8D%E4%BD%9C%E6%98%AF%E5%9F%BA%E4%BA%8E%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%0A%0A%E8%BF%99%E9%87%8C%E4%B8%BB%E8%A6%81%E5%88%86%E6%9E%90%E4%B8%8B%EF%BC%9Asocket%20%E5%92%8C%20%E4%B8%8E%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%0A%0A%60%60%60%0Atype%20%20socket%20struct%20%7B%0A%09short%20%20%20%20%20%20%20%20%20%09%09type%3B%20%20%2F%2F%E5%A5%97%E6%8E%A5%E5%AD%97%E6%89%80%E7%94%A8%E7%9A%84%E6%B5%81%E7%B1%BB%E5%9E%8B%0A%09socket\_state%20%20%09%09state%3B%20%2F%2F%E5%A5%97%E6%8E%A5%E5%AD%97%E6%89%80%E5%A4%84%E7%8A%B6%E6%80%81%0A%09long%20%20%20%20%20%20%20%20%20%20%09%09flags%3B%20%2F%2F%E6%A0%87%E8%AF%86%E5%AD%97%E6%AE%B5%EF%BC%8C%E7%9B%AE%E5%89%8D%E5%B0%9A%E6%97%A0%E6%98%8E%E7%A1%AE%E4%BD%9C%E7%94%A8%0A%09struct%20proto\_ops%20%20%09\*ops%3B%20%20%2F%2F%E6%93%8D%E4%BD%9C%E5%87%BD%E6%95%B0%E9%9B%86%E6%8C%87%E9%92%88%0A%09struct%20wait\_queue%20%09\*\*wait%3B%2F%2F%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97%0A%09struct%20inode%20%20%20%20%20%20%09\*inode%3B%2F%2Finode%E7%BB%93%E6%9E%84%E6%8C%87%E9%92%88%0A%20%20%20%20struct%20sock%09%09%20%20%20%20\*sk%3B%0A%20%20%20%20struct%20socket\_wq%20\_\_rcu%20\*sk\_wq%3B%20%2F%2F%2F%E5%A5%97%E6%8E%A5%E5%AD%97%E7%9A%84%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97%0A%7D%3B%0A%60%60%60%0A%3E%20socket%20%E7%BB%93%E6%9E%84%E7%9C%8B%E7%9D%80%E6%8C%BA%E7%AE%80%E5%8D%95%EF%BC%8C%E5%AE%83%E4%B8%BB%E8%A6%81%E6%98%AF%E9%80%8F%E7%BB%99%E7%94%A8%E6%88%B7%E6%80%81%E7%9A%84%EF%BC%8C%E8%80%8C%20sock%20%E7%BB%93%E6%9E%84%E6%98%AF%E7%BB%99%E5%86%85%E6%A0%B8%E6%80%81%E7%9A%84%0A%0Aproto\_ops%20%E5%B0%B1%E6%98%AF%E4%BD%A0%E9%80%89%E6%8B%A9%E4%BA%86%E4%BB%80%E4%B9%88%E5%8D%8F%E8%AE%AE%E7%B1%BB%E5%9E%8B%EF%BC%8C%E5%B0%B1%E6%89%BE%E5%88%B0%E8%AF%A5%E5%8D%8F%E8%AE%AE%E7%9A%84%E4%B8%80%E7%BB%84%E6%93%8D%E4%BD%9C%E5%87%BD%E6%95%B0%0Ask\_wq%EF%BC%9A%20epoll%E5%B0%B1%E6%98%AF%E6%8C%82%E8%BD%BD%E5%9C%A8%E8%BF%99%E4%B8%AA%E4%B8%8A%E9%9D%A2%EF%BC%8C%E4%B8%80%E4%BD%86%E8%AF%BB%E5%86%99%E7%BC%93%E5%86%B2%E5%8C%BA%E5%B0%B1%E7%BB%AA%EF%BC%8C%E5%B0%B1%E4%BC%9A%E5%9B%9E%E8%B0%83%0Ainode%20%E4%B8%BB%E8%A6%81%E6%98%AF%E6%8A%8A%20socket%20%E7%BB%93%E6%9E%84%E4%BD%93%E4%B8%8E%E7%A1%AC%E7%9B%98%E7%BB%91%E5%AE%9A%E6%9C%80%E7%BB%88%E4%BB%A5FD%E5%BD%A2%E5%BC%8F%E8%BF%94%E5%9B%9E%0A%60%60%60%0Aenum%20sock\_type%0A%7B%0A%20%20%20SOCK\_STREAM%20%20%3D%201%2C%20%2F%2F%20%E7%94%A8%E4%BA%8E%E4%B8%8ETCP%E5%B1%82%E4%B8%AD%E7%9A%84tcp%E5%8D%8F%E8%AE%AE%E6%95%B0%E6%8D%AE%E7%9A%84struct%20socket%20%0A%20%20%20SOCK\_DGRAM%20%20%20%3D%202%2C%20%2F%2F%E7%94%A8%E4%BA%8E%E4%B8%8ETCP%E5%B1%82%E4%B8%AD%E7%9A%84udp%E5%8D%8F%E8%AE%AE%E6%95%B0%E6%8D%AE%E7%9A%84struct%20socket%20%0A%20%20%20SOCK\_RAW%20%20%20%20%20%3D%203%2C%20%2F%2F%20raw%20struct%20socket%0A%20%20%20SOCK\_RDM%20%20%20%20%20%3D%204%2C%20%2F%2F%E5%8F%AF%E9%9D%A0%E4%BC%A0%E8%BE%93%E6%B6%88%E6%81%AF%E7%9A%84struct%20socket%0A%20%20%20SOCK\_SEQPACKET%20%3D%205%2C%2F%2F%20sequential%20packet%20socket%0A%20%20%20SOCK\_DCCP%20%20%20%20%3D%206%2C%0A%20%20%20SOCK\_PACKET%20%20%3D%2010%2C%20%2F%2F%E4%BB%8Edev%20level%E4%B8%AD%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE%E5%8C%85%E7%9A%84socket%0A%7D%3B%0A%60%60%60%0A%3E%E6%84%9F%E8%A7%89%E6%9B%B4%E5%83%8F%E6%98%AF%E5%8D%8F%E8%AE%AE%E7%B1%BB%E5%9E%8B%EF%BC%8C%E5%A6%82%EF%BC%9ATCP%2FUDP%0A%60%60%60%0Atypedef%20enum%20%7B%20%20%0A%20%20%20%20SS\_FREE%20%3D%200%2C%20%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E8%AF%A5socket%E8%BF%98%E6%9C%AA%E5%88%86%E9%85%8D%20%20%0A%20%20%20%20SS\_UNCONNECTED%2C%20%20%20%20%20%20%20%20%20%2F%2F%E6%9C%AA%E8%BF%9E%E5%90%91%E4%BB%BB%E4%BD%95socket%20%20%0A%20%20%20%20SS\_CONNECTING%2C%20%20%20%20%20%20%20%20%20%20%2F%2F%E6%AD%A3%E5%9C%A8%E8%BF%9E%E6%8E%A5%E8%BF%87%E7%A8%8B%E4%B8%AD%20%20%0A%20%20%20%20SS\_CONNECTED%2C%20%20%20%20%20%20%20%20%20%20%20%2F%2F%E5%B7%B2%E8%BF%9E%E5%90%91%E4%B8%80%E4%B8%AAsocket%20%20%0A%20%20%20%20SS\_DISCONNECTING%20%20%20%20%20%20%20%20%2F%2F%E6%AD%A3%E5%9C%A8%E6%96%AD%E5%BC%80%E8%BF%9E%E6%8E%A5%E7%9A%84%E8%BF%87%E7%A8%8B%E4%B8%AD%20%20%0A%7Dsocket\_state%3B%0A%60%60%60%0A%3Esocket%E7%8A%B6%E6%80%81%E3%80%82%E8%BF%99%E9%87%8C%E5%8F%91%E7%8E%B0%EF%BC%8C%E5%B9%B6%E6%B2%A1%E6%9C%89%20ESTABLISHED%20CLOSEING%20FIN\-WAIT\-1%20%E5%B8%B8%E8%A7%81%E7%8A%B6%E6%80%81%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%BF%99%E4%BA%9B%E7%8A%B6%E6%80%81%E6%98%AF%E4%B8%93%E5%B1%9E%E4%BA%8E%20TCP%20%E5%8D%8F%E8%AE%AE%E7%9A%84%E3%80%82%E8%80%8C%E5%AE%9E%E9%99%85%E4%B8%8A%E8%BF%99%E4%B8%AA%E7%8A%B6%E6%80%81%E6%89%8D%E6%9B%B4%E9%80%9A%E7%94%A8%E5%8C%96%E7%9A%84%E3%80%82%0A%60%60%60%0Astruct%20socket%20%E4%B8%AD%E7%9A%84%20flags%20%E5%AD%97%E6%AE%B5%E5%8F%96%E5%80%BC%EF%BC%9A%0A%20%20%23define%20SOCK\_ASYNC\_NOSPACE%20%200%0A%20%20%23define%20SOCK\_ASYNC\_WAITDATA%201%0A%20%20%23define%20SOCK\_NOSPACE%20%20%20%20%20%20%20%202%0A%20%20%23define%20SOCK\_PASSCRED%20%20%20%20%20%20%203%0A%20%20%23define%20SOCK\_PASSSEC%20%20%20%20%20%20%20%204%0A%60%60%60%0Asock%20%E7%BB%93%E6%9E%84%E4%BD%93%20%20%E5%9B%A0%E4%B8%BA%E5%A4%AA%E5%A4%A7%EF%BC%8C%E5%88%97%E5%87%BA%E4%B8%BB%E8%A6%81%E7%9A%84%E5%90%A7%EF%BC%9A%0A%60%60%60%0Astruct%20sock\_common%09\_\_sk\_common%3B%09%20%2F%2F%20%E7%BD%91%E7%BB%9C%E5%B1%82%E5%A5%97%E6%8E%A5%E5%AD%97%E9%80%9A%E7%94%A8%E7%BB%93%E6%9E%84%E4%BD%93%0A%0Asocket\_lock\_t%20sk\_lock%3B%2F%2F%20%E5%A5%97%E6%8E%A5%E5%AD%97%E5%90%8C%E6%AD%A5%E9%94%81%0A%20%20%0Astruct%20sk\_buff\_head%09sk\_receive\_queue%3B%2F%2F%20%E6%94%B6%E5%88%B0%E7%9A%84%E6%95%B0%E6%8D%AE%E5%8C%85%E9%98%9F%E5%88%97%0Astruct%20sk\_buff\_head%09sk\_write\_queue%3B%20%20%2F%2F%20%E5%8F%91%E9%80%81%E9%98%9F%E5%88%97%2F%E9%87%8D%E4%BC%A0%E9%98%9F%E5%88%97%0Astruct%20sk\_buff\_head%20back\_log%3B%2F%2F%E5%BD%93%E5%A5%97%E6%8E%A5%E5%AD%97%E5%BF%99%E6%97%B6%EF%BC%8C%E6%95%B0%E6%8D%AE%E5%8C%85%E6%9A%82%E5%AD%98%E8%BF%99%E9%87%8C%0A%0Aunion%20%7B%0A%20%20%20%20struct%20socket\_wq%20\_\_rcu%20%20%20%20\*sk\_wq%3B%2F%2F%E8%BF%9B%E7%A8%8B%E7%AD%89%E5%BE%85%E9%98%9F%E5%88%97%0A%20%20%20%20struct%20socket\_wq%20%20%20%20\*sk\_wq\_raw%3B%0A%7D%3B%0A%0Aunsigned%20long%20%20%20sk\_lingertime%3B%2F%2F%E8%BF%9E%E6%8E%A5%E5%85%B3%E9%97%AD%E5%89%8D%E6%9C%AA%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%E7%9A%84%E6%80%BB%E6%95%B0%0Astruct%20sk\_buff\_head%20sk\_error\_queue%3B%2F%2F%E9%94%99%E8%AF%AF%E9%93%BE%E8%A1%A8%EF%BC%8C%E5%AD%98%E6%94%BE%E8%AF%A6%E7%BB%86%E7%9A%84%E5%87%BA%E9%94%99%E4%BF%A1%E6%81%AF%E3%80%82%0Along%20%20%20%20%20%20%20%20%20%20%20%20sk\_rcvtimeo%3B%2F%2F%E8%AF%BB%E8%B6%85%E6%97%B6%0Along%20%20%20%20%20%20%20%20%20%20%20%20sk\_sndtimeo%3B%2F%2F%E5%86%99%E8%B6%85%E6%97%B6%0Astruct%20timer\_list%20%20%20%20sk\_timer%3B%2F%2F%E5%AE%9A%E6%97%B6%E5%99%A8%EF%BC%8C%E5%83%8F3%E6%AC%A1%E6%8F%A1%E6%89%8B%E7%AD%89%E4%BD%BF%E7%94%A8%0A%0A%0A%20%20%20%20%0A%60%60%60%0Asock%20%E7%BB%93%E6%9E%84%E4%BD%93%20%E6%89%8D%E6%98%AF%E6%95%B4%E4%B8%AA%20%E7%BD%91%E7%BB%9C%E4%BC%A0%E8%BE%93%E7%9A%84%20%E6%A0%B8%E5%BF%83%EF%BC%8C%E8%BF%99%E9%87%8C%E4%B8%BB%E8%A6%81%E5%8A%9F%E8%83%BD%EF%BC%9A%0A1.%20TCP%20%E7%A8%B3%E5%AE%9A%E6%8E%A7%E5%88%B6%EF%BC%9A%E9%87%8D%E4%BC%A0%E3%80%81%E6%BB%91%E7%AA%97%E3%80%81%E9%98%BB%E5%A1%9E%E3%80%81%E6%8E%92%E5%BA%8F%0A3.%20%E6%94%B6%E5%8F%91%E6%95%B0%E6%8D%AE%0A4.%20TCP%20%E8%BF%9E%E6%8E%A5%E6%8E%A7%E5%88%B6%EF%BC%9A%E5%BB%BA%E7%AB%8B%E8%BF%9E%E6%8E%A5%E3%80%81%E5%85%B3%E9%97%AD%E8%BF%9E%E6%8E%A5%E3%80%81%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%E6%8E%A7%E5%88%B6%0A%0A%3E%E4%B8%80%E8%88%AC%E6%9D%A5%E8%AF%B4%20%E5%86%85%E6%A0%B8%E6%8E%A7%E5%88%B6%20%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E7%94%A8%E6%88%B7%E6%80%81%E6%8E%A7%E5%88%B6%E7%9A%84%E6%98%AF%20socket%20%E7%BB%93%E6%9E%84%E4%BD%93%0A%0Ask\_lock%3A%E7%94%A8%E6%88%B7%E6%80%81%E8%AF%BB%E6%95%B0%E6%8D%AE%E5%BE%97%E5%8A%A0%E9%94%81%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%BF%99%E6%9C%9F%E9%97%B4%E5%8F%AF%E8%83%BD%E5%86%85%E6%A0%B8%E6%94%B6%E5%88%B0%E7%BD%91%E5%8D%A1%E4%BF%A1%E6%81%AF%EF%BC%8C%E8%A6%81%E5%86%99%E5%85%A5%E7%BC%93%E5%AD%98%E9%98%9F%E5%88%97%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8C%E7%8A%B6%E6%80%81%E5%8F%91%E7%94%9F%E5%8F%98%E6%9B%B4%E6%97%B6%EF%BC%8C%E5%86%85%E6%A0%B8%E4%B9%9F%E4%BC%9A%E5%8A%A0%E9%94%81%E3%80%82%0Ask\_wq%3A%E8%BF%9B%E7%A8%8B%E7%AD%89%E5%BE%85%E8%BF%9E%E6%8E%A5%E3%80%81%E7%AD%89%E5%BE%85%E8%BE%93%E5%87%BA%E7%BC%93%E5%86%B2%E5%8C%BA%E3%80%81%E7%AD%89%E5%BE%85%E8%AF%BB%E6%95%B0%E6%8D%AE%E6%97%B6%EF%BC%8C%E9%83%BD%E4%BC%9A%E5%B0%86%E8%BF%9B%E7%A8%8B%E6%9A%82%E5%AD%98%E5%88%B0%E6%AD%A4%E9%98%9F%E5%88%97%E4%B8%AD%0Ask\_lingertime%3A%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%E5%BC%95%E5%8F%91%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%8C%E8%BF%9E%E6%8E%A5%E5%A6%82%E6%9E%9C%E5%85%B3%E9%97%AD%EF%BC%8C%E8%80%8C%E8%BF%98%E6%9C%89%E6%95%B0%E6%8D%AE%E6%9C%AA%E5%8F%91%E9%80%81%E5%AE%8C%EF%BC%8C%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%EF%BC%9F%E5%A5%BD%E5%83%8F%E5%86%85%E6%A0%B8%E6%9C%89%E4%B8%AA%E5%8F%82%E6%95%B0%EF%BC%8C%E4%BD%A0%E5%BC%80%E5%90%AF%E5%AE%83%E5%B0%B1%E7%AD%89%E5%BE%85%E4%B8%80%E6%AE%B5%E6%97%B6%E9%97%B4%20%E4%BD%A0%E5%85%B3%E9%97%AD%E4%BB%96%E5%B0%B1%E7%9B%B4%E6%8E%A5%E6%8A%8A%E6%95%B0%E6%8D%AE%E4%B8%A2%E4%BA%86%EF%BC%8C%E9%9D%9E%E4%BC%98%E9%9B%85%E5%85%B3%E9%97%AD%E3%80%82%0Ask\_rcvtimeo%3A%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%E6%9C%89%E6%84%8F%E6%80%9D%E3%80%82socket%20%E6%9C%AC%E8%BA%AB%E5%B0%B1%E6%98%AF%E9%98%BB%E5%A1%9E%E6%A8%A1%E5%BC%8F\(epool%20seelect%20recv%E9%98%B6%E5%A4%96\)%E7%9A%84%EF%BC%8C%E5%83%8F%EF%BC%9Aaccept%2Fconnect%2Fsend%2Frecv%E8%BF%99%E4%BA%9B%E9%83%BD%E6%98%AF%E9%98%BB%E5%A1%9E%2F%E5%8D%8A%E9%98%BB%E5%A1%9E%E7%9A%84%E3%80%82%E5%A6%82%E6%9E%9C%E8%AE%BE%E7%BD%AE%E8%BF%99%E4%B8%AA%E5%8F%82%E6%95%B0%E5%B0%B1%E5%8F%AF%E4%BB%A5%E9%9D%9E%E9%98%BB%E5%A1%9E%E3%80%82%E4%BD%86%E6%84%9F%E8%A7%89%E5%8F%AF%E8%83%BD%E8%BF%98%E6%98%AF%E6%9C%89%E6%80%A7%E8%83%BD%E7%9A%84%E7%89%BA%E7%89%B2%E6%88%96%E8%80%85%E5%BE%97%E4%BC%9A%E7%94%A8%EF%BC%8C%E7%9F%A5%E9%81%93%E6%80%8E%E4%B9%88%E7%94%A8%EF%BC%8C%E4%B8%8D%E7%84%B6%E8%BF%98%E6%98%AF%E9%BB%98%E8%AE%A4%E5%80%BC%E5%90%A7%E3%80%82%0A%0A%0Asock\_common%0A%60%60%60%0Aunion%20%7B%0A%20%20%20%20%20%20%20%20\_\_addrpair%20%20%20%20skc\_addrpair%3B%0A%20%20%20%20%20%20%20%20struct%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20\_\_be32%20%20%20%20skc\_daddr%3B%2F%2F%E8%BF%9C%E7%AB%AF%E5%9C%B0%E5%9D%80%20IP4%E6%9C%AC%E5%9C%B0%E5%9C%B0%E5%9D%80%0A%20%20%20%20%20%20%20%20%20%20%20%20\_\_be32%20%20%20%20skc\_rcv\_saddr%3B%2F%2F%E6%9C%AC%E5%9C%B0%E5%9C%B0%E5%9D%80%20IP4%E6%9C%AC%E5%9C%B0%E5%9C%B0%E5%9D%80%0A%20%20%20%20%20%20%20%20%7D%3B%0A%7D%3B%0A%0Aunsigned%20short%20%20%20%20%20%20skc\_family%3B%20%20%20%20%20%20%20%20%20%2F%2F%E5%9C%B0%E5%9D%80%E6%97%8F%0Avolatile%20unsigned%20char%20%20skc\_state%3B%20%20%20%20%20%20%2F%2F%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%0Aunsigned%20char%20%20%20%20%20%20%20skc\_reuse%3B%20%20%20%20%20%20%20%20%20%20%2F%2FSO\_REUSEADDR%E8%AE%BE%E7%BD%AE%0Aint%20%20%20%20%20%20%20%20%20%09%09skc\_bound\_dev\_if%3B%0Astruct%20hlist\_node%20%20%20skc\_node%3B%0Astruct%20hlist\_node%20%20%20skc\_bind\_node%3B%20%20%20%20%20%20%2F%2F%E5%93%88%E5%B8%8C%E8%A1%A8%E7%9B%B8%E5%85%B3%0Aatomic\_t%20%20%20%20%20%20%20%20%09skc\_refcnt%3B%20%20%20%20%20%20%20%20%20%2F%2F%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0%0A%60%60%60%0A%0A%E8%BF%99%E9%87%8C%E6%98%AF%E4%BA%9B%E5%9F%BA%E7%A1%80%E7%9A%84%E4%BF%A1%E6%81%AF%EF%BC%8C%E8%BF%98%E6%9C%89%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%E7%AD%89%E3%80%82sock%20%E7%BB%93%E6%9E%84%E4%BD%93%E4%B9%9F%E7%AE%97%E6%98%AF%E7%BB%A7%E6%89%BF%E4%BA%86%E4%BA%9B%E7%BB%93%E6%9E%84%E4%BD%93%E5%90%A7%0A%0A%0A%23%23%23%23%20socket%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%20%E5%B0%8F%E7%BB%93%0A%E5%85%B6%E5%AE%9E%EF%BC%8Cunix%2Flinux%20%E5%86%85%E6%A0%B8%E5%9C%A8%E5%AE%9E%E7%8E%B0%20socket%20%E4%B8%AD%E8%BF%98%E6%9C%89N%E5%A4%9A%E4%B8%AA%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E8%BF%98%E7%89%B5%E6%89%AF%E5%88%B0%20%E7%BD%91%E5%8D%A1%E3%80%81%E9%A9%B1%E5%8A%A8%E3%80%81%E4%B8%AD%E6%96%AD%E6%9C%BA%E5%88%B6%E7%AD%89%E7%AD%89%E5%90%A7%EF%BC%8C%E6%88%91%E8%BF%99%E9%87%8C%E4%B9%9F%E5%8F%AA%E6%98%AF%E5%88%97%E5%87%BA2%E4%B8%AA%E5%85%B3%E9%94%AE%E7%9A%84%E7%BB%93%E6%9E%84%EF%BC%8C%E5%A4%AA%E7%BB%86%E8%8A%82%E7%9A%84%E6%88%91%E4%B9%9F%E4%B8%8D%E6%87%82%EF%BC%8C%E4%B9%9F%E6%B2%A1%E6%97%B6%E9%97%B4%E7%9C%8B%E3%80%82%0A%0A%E4%BD%86%E6%98%AF%E4%BB%8E%20sock%20%E8%BF%99%E4%B8%AA%E7%BB%93%E6%9E%84%E4%BD%93%E6%9D%A5%E5%88%86%E6%9E%90%E7%9C%8B%EF%BC%8C%E5%AE%83%E5%85%B6%E5%AE%9E%E5%B0%B1%E6%98%AF%E6%8A%8A%E7%BD%91%E7%BB%9C%E8%BF%9B%E8%A1%8C%E5%8F%88%E4%B8%80%E6%AC%A1%E5%88%86%E5%B1%82%EF%BC%8C%E5%B0%81%E8%A3%85%EF%BC%8C%E8%BE%BE%E5%88%B0%E6%9C%80%E7%BB%88%E5%8F%AA%E9%9C%B2%E5%87%BA%E5%87%A0%E4%B8%AA%E5%85%AC%E5%85%B1%E6%8E%A5%E5%8F%A3%E7%BB%99%E7%A8%8B%E5%BA%8F%E5%91%98%E4%BD%BF%E7%94%A8%E3%80%82%0A%0A%E5%85%B6%E4%B8%AD%E7%9A%84%E6%A0%B8%E5%BF%83%E5%B0%B1%E6%98%AF%EF%BC%9A%E9%80%9A%E8%BF%87%E5%87%A0%E4%B8%AA%E5%8F%91%2F%E5%8F%91%E7%BC%93%E5%86%B2%E6%95%B0%E6%8D%AE%E9%98%9F%E5%88%97%EF%BC%8C%E5%86%85%E6%A0%B8%E5%86%8D%E7%9D%A1%E7%9C%A0%2F%E5%94%A4%E9%86%92%E8%BF%9B%E7%A8%8B%EF%BC%8C%E5%B0%B1%E5%AE%9E%E7%8E%B0%E4%BA%86....%0A%0A%0A%E6%95%B0%E6%8D%AE%E6%94%B6%E5%8F%91%E9%98%9F%E5%88%97%0A\-\-\-\-%0A%E5%AE%83%E4%B8%BB%E8%A6%81%E6%98%AF%E5%8C%85%E5%90%AB%E4%BA%864%E4%B8%AA%E9%98%9F%E5%88%97%EF%BC%9A%0A1.%20backlog%EF%BC%9A%E7%94%A8%E6%88%B7%E6%AD%A3%E5%9C%A8%E5%8D%A0%E7%94%A8%20socket%20%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%AD%A3%E5%9C%A8%E8%AF%BB%E6%93%8D%E4%BD%9C%EF%BC%8C%E6%95%B0%E6%8D%AE%E6%9A%82%E6%97%B6%E6%94%BE%E5%88%B0backlog%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%BB%A1%E4%BA%86%E5%B0%B1%E7%9B%B4%E6%8E%A5%E4%B8%A2%E5%BC%83%E6%95%B0%E6%8D%AE%E3%80%82%E6%95%B0%E6%8D%AE%EF%BC%9A%E5%8F%AF%E8%83%BD%E4%B9%B1%E5%BA%8F%EF%BC%8C%E8%BF%98%E6%9C%89TCP%E5%A4%B4%E4%BF%A1%E6%81%AF%0A2.%20prequeue%3A%0A3.%20sk\_receive\_queue%EF%BC%9A%E6%9C%80%E5%B9%B2%E5%87%80%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%B2%A1%E6%9C%89TCP%E5%8D%8F%E8%AE%AE%E5%A4%B4%E3%80%81%E9%A1%BA%E5%BA%8F%E6%AD%A3%E7%A1%AE%E3%80%81%E6%A0%A1%E9%AA%8C%E8%BF%87%E7%9A%84%EF%BC%8C%E7%94%A8%E6%88%B7%E5%8F%AF%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8%0A4.%20out\_of\_order%EF%BC%9A%E4%B9%B1%E5%BA%8F%E6%8A%A5%E6%96%87%0A%0A%3E%E4%BB%A5%E4%B8%8A%E9%98%9F%E5%88%97%E6%98%AF%E6%9C%89%E4%BC%98%E5%85%88%E9%9B%86%E4%B9%8B%E5%88%86%E7%9A%84%EF%BC%8C%E7%BA%A7%E5%88%AB%E8%B6%8A%E9%AB%98%EF%BC%8C%E8%B6%8A%E5%85%88%E5%A4%84%E7%90%86%E3%80%82%E4%B9%9F%E5%B0%B1%E6%98%AF%E5%BA%8F%E5%8F%B7%E5%A4%A7%E7%9A%84%20%0A%0A%E6%AD%A3%E5%B8%B8%E6%83%85%E5%86%B5%0A%3E%E5%86%85%E6%A0%B8%E8%82%AF%E5%AE%9A%E6%98%AF%E5%B8%8C%E6%9C%9B%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E9%83%BD%E6%98%AF%E6%AD%A3%E5%B8%B8%E7%9A%84%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%89%94%E5%88%B0%20sk\_receive\_queue%20%E4%B8%AD%EF%BC%8C%E7%94%A8%E6%88%B7%E7%9B%B4%E6%8E%A5%E5%8F%AF%E4%BB%A5%E5%8F%96%E8%B5%B0%E7%9A%84%E3%80%82%0A%0A%E6%95%B0%E6%8D%AE%E4%B9%B1%E5%BA%8F%E6%83%85%E5%86%B5%0A1.%20A%E7%BB%99B~%E5%8F%91%E9%80%813%E6%AE%B5%E6%8A%A5%E6%96%87%201%202%203%20%0A2.%20%E6%8A%A5%E6%96%873%E5%85%88%E5%88%B0%E7%9A%84%EF%BC%8C%E7%94%A8%E6%88%B7%E6%80%81%E5%B8%8C%E6%9C%9B%E7%9A%84%E6%98%AF%E6%8A%A5%E6%96%871%EF%BC%8C%E6%89%80%E4%BB%A5%20%E4%B8%8D%E8%83%BD%E7%9B%B4%E6%8E%A5%E8%AF%BB%E5%8F%96%E3%80%82%E6%8A%8A3%E6%8A%A5%E6%96%87%E6%89%94%E5%88%B0%20out\_of\_order%20%E4%B8%AD%0A3.%20%E6%8A%A5%E6%96%871%20%E6%9D%A5%E4%BA%86%EF%BC%8C2%E4%B9%9F%E6%9D%A5%E4%BA%86%EF%BC%8C%E9%83%BD%E6%89%94%E5%88%B0%20sk\_receive\_queue%20%20%E4%B8%AD%0A4.%20%E4%BB%8E%20out\_of\_order%20%E4%B8%AD%E6%8A%8A3%E5%8F%96%E5%87%BA%E6%9D%A5%EF%BC%8C%E6%89%94%E5%88%B0%20receive\_queue%20%E4%B8%AD%E3%80%82%0A%0Asocket%20%E8%A2%AB%E7%94%A8%E6%88%B7%E6%80%81%E7%BB%99%E9%94%81%E4%BD%8F%E4%BA%86%0A%3E%E7%94%A8%E6%88%B7%E6%80%81%E6%AD%A3%E5%9C%A8%E6%93%8D%E4%BD%9C%20socket%20%E8%AF%BB%E6%95%B0%E6%8D%AE%E4%B8%AD%EF%BC%8C%E6%AD%A4%E6%97%B6%E6%9C%89%E6%B6%88%E6%81%AF%E8%BF%87%E6%9D%A5%E4%BA%86%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%89%94%E5%88%B0%20backlog%20%E4%B8%AD%0A%0A%E8%AF%BB%E6%95%B0%E6%8D%AE%E9%98%BB%E5%A1%9E%E4%B8%AD%0A1.%20%E7%94%A8%E6%88%B7%E6%80%81%E5%BC%80%E5%A7%8B%E8%AF%BB%E6%95%B0%E6%8D%AE%EF%BC%8C%E6%AD%A4%E6%97%B6%E6%B2%A1%E6%9C%89%E6%95%B0%E6%8D%AE%E4%BA%86%EF%BC%8C%E5%AE%83%E5%B0%B1%E6%94%BE%E5%BC%83%E9%94%81%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E8%BF%9B%E5%85%A5%E7%9D%A1%E7%9C%A0%E7%8A%B6%E6%80%81%EF%BC%8C%E7%AD%89%E5%BE%85%E6%96%B0%E6%95%B0%E6%8D%AE%E8%BF%87%E6%9D%A5%20%E3%80%82%20%0A2.%20%E6%AD%A4%E6%97%B6%EF%BC%8C%E6%9C%89%E6%96%B0%E6%95%B0%E6%8D%AE%E6%9D%A5%E4%BA%86%EF%BC%8C%E5%9B%A0%E4%B8%BA%E7%94%A8%E6%88%B7%E6%B2%A1%E5%8A%A0%E9%94%81%EF%BC%8C%E4%B9%9F%E6%B2%A1%E6%9C%89%E6%AD%A3%E5%9C%A8%E8%AF%BB%EF%BC%8C%E6%96%B0%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E4%B8%8D%E4%BC%9A%E8%BF%9B%E5%85%A5backlog%E3%80%82%E8%BF%99%E6%9D%A1%E6%95%B0%E6%8D%AE%E4%BC%9A%E6%8F%92%E5%85%A5%E5%88%B0%20prequeue%20%E4%B8%AD%EF%BC%8C%0A4.%20prequeue%20%E6%9C%89%E6%96%B0%E6%95%B0%E6%8D%AE%E4%BC%9A%E5%94%A4%E9%86%92%E8%AF%A5%E8%BF%9B%E7%A8%8B%EF%BC%8C%E5%94%A4%E9%86%92%E5%90%8E%E7%AB%8B%E5%88%BB%E5%8A%A0%E9%94%81%0A5.%20%E5%85%88%E8%AF%BB%20sk\_receive\_queue%20%E6%95%B0%E6%8D%AE%20%EF%BC%8C%E5%86%8D%E8%AF%BB%20prequeue%20%EF%BC%8C%E6%9C%80%E5%90%8E%E8%AF%BB%20backlog%0A%0A%E6%80%BB%E7%BB%93%EF%BC%9A%0A%E9%80%9A%E8%BF%87%E4%B8%80%E4%B8%AA%E9%94%81%EF%BC%8C%E7%94%A8%E6%88%B7%E6%80%81%E8%AF%BB%E5%8F%96%E6%93%8D\(%E5%8F%AF%E8%83%BD%E4%BC%9A%E6%93%8D%E4%BD%9C%20sk\_receive\_queue%20prequeue%20\)%E4%BD%9C%E5%B0%B1%E4%BC%9A%E5%8A%A0%E9%94%81%EF%BC%8C%E6%96%B0%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E5%B0%B1%E5%BE%97%E8%BF%9B%E5%85%A5%20backlog%E3%80%82%E7%94%A8%E6%88%B7%E6%80%81%E6%B2%A1%E6%9C%89%E5%8A%A0%E9%94%81%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%86%85%E6%A0%B8%E5%B0%B1%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E6%93%8D%E4%BD%9C%20sk\_receive\_queue%20%2B%20out\_of\_order%E3%80%82%E8%80%8C%E7%94%A8%E6%88%B7%E6%80%81%E6%B2%A1%E5%8A%A0%E9%94%81%E4%BD%86%E6%98%AF%E9%98%BB%E5%A1%9E%E6%A8%A1%E5%BC%8F%E4%B8%8B%EF%BC%8C%E5%B0%B1%E6%98%AF%E6%93%8D%E4%BD%9C%20prequeue%E3%80%82%0A%0Abind%20%E5%87%BD%E6%95%B0%0A\-\-\-%0A%E6%8A%8A%E8%A6%81%E7%9B%91%E5%90%AC%20%E7%9A%84IP%20%E7%AB%AF%E5%8F%A3%E5%8F%B7%EF%BC%8C%E6%9B%B4%E6%96%B0%E5%88%B0%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%E5%86%85%EF%BC%8Clisten%20%E5%87%BD%E6%95%B0%E6%89%A7%E8%A1%8C%E4%B9%8B%E5%89%8D%E5%BF%85%E9%A1%BB%E5%BE%97%E8%B0%83%E7%94%A8%20bind%20%E5%87%BD%E6%95%B0%0A%0Alisten%20%E5%87%BD%E6%95%B0%0A\-\-\-\-%0A1.%20%E6%89%BE%E5%88%B0%20sock%20%E7%BB%93%E6%9E%84%20%EF%BC%8C%0A2.%20%E6%89%BE%E5%88%B0%20inet\_connection\_sock%20%E7%BB%93%E6%9E%84%20%EF%BC%8C%E6%89%BE%E5%88%B0%20icsk\_accept\_queue%20%E6%88%90%E5%91%98%0A3.%20%E5%88%9B%E5%BB%BA%20request\_sock\_quue%0A%20%20%20%201.%20%E5%88%9B%E5%BB%BA%E5%8D%8A%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%0A%20%20%20%202.%20%E5%88%9B%E5%BB%BA%E5%85%A8%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%0A%0Arequest\_sock\_quue%20%20%E9%98%9F%E5%88%97%0A%60%60%60%0Astruct%20request\_sock\_queue%20%7B%0A%20%20%20%20struct%20request\_sock%09\*rskq\_accept\_head%3B%0A%20%20%20%20struct%20request\_sock%09\*rskq\_accept\_tail%3B%0A%20%20%20%20rwlock\_t%09%09syn\_wait\_lock%3B%0A%20%20%20%20u8%09%09%09rskq\_defer\_accept%3B%0A%20%20%20%20%2F\*%203%20bytes%20hole%2C%20try%20to%20pack%20\*%2F%0A%20%20%20%20struct%20listen\_sock%09\*listen\_opt%3B%0A%7D%3B%0A%60%60%60%0Arequest\_sock%20%E5%85%A8%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%0Alisten\_sock%20%E5%8D%8A%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%EF%BC%8C%E5%85%B3%E9%94%AE%E5%8F%82%E6%95%B0%E5%8F%82%E6%95%B0%EF%BC%9Abacklog%20%EF%BC%8C%E5%8D%8A%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%E6%9C%80%E5%A4%A7%E9%95%BF%E5%BA%A6%0A%0A%E4%B8%A4%E4%B8%AA%E9%98%9F%E5%88%97%E9%87%8C%E5%AD%98%E5%82%A8%E7%9A%84%E6%88%90%E5%91%98%E9%83%BD%E6%98%AF%EF%BC%9Arequset\_sock%20%E7%BB%93%E6%9E%84%E4%BD%93%0A%0A%60%60%60%0Astruct%20request\_sock%20%7B%0A%09struct%20sock\_common%09%09\_\_req\_common%3B%0A%20%20%20%20......%0A%7D%3B%0A%60%60%60%0A%E6%B2%A1%E7%94%A8%E7%9A%84%E6%88%91%E5%B0%B1%E6%B2%A1%E8%AE%B0%EF%BC%8C%E5%85%B6%E4%B8%AD%E5%A5%BD%E5%83%8F%E5%B0%B1%E4%B8%80%E4%B8%AA%E6%98%AF%E5%80%BC%E5%BE%97%E8%AF%B4%E7%9A%84%EF%BC%9Asock\_common%20%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%AE%83%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%9E%8B%E6%97%B6%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E4%B9%9F%E8%83%BD%E7%90%86%E8%A7%A3%EF%BC%8C%E6%AF%95%E7%AB%9F%E8%BF%98%E5%88%B0%E6%AD%A3%E5%BC%8F%E4%BD%BF%E7%94%A8%EF%BC%8C%E7%94%9A%E8%87%B3%E5%8F%AF%E8%83%BD%E8%BF%98%E6%98%AF%E5%8D%8A%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%20%EF%BC%8C%E6%B2%A1%E5%BF%85%E8%A6%81%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%AE%8C%E6%95%B4%E7%9A%84%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%E3%80%82%E5%AE%83%E5%8F%AA%E8%A6%81%E8%83%BD%E8%A1%A8%E7%A4%BA%E6%B8%85%E6%A5%9A%EF%BC%9A%E8%A6%81%E8%BF%9E%E6%8E%A5%E6%96%B9%E7%9A%84%E5%9F%BA%E7%A1%80%E4%BF%A1%E6%81%AF5%E5%85%83%E7%BB%84%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%E5%BD%93%E5%8D%8A%E8%BF%9E%E6%8E%A5%E5%AE%8C%E6%88%90%E5%90%8E%EF%BC%8C%E8%BF%9B%E5%85%A5%20%E5%85%A8%E8%BF%9E%E6%8E%A5%E5%90%8E%EF%BC%8C%E6%97%A2%EF%BC%9A%E6%8A%8A%E8%BF%99%E4%B8%AA%E6%88%90%E5%91%98%E4%BB%8E%E5%8D%8A%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%E8%BD%AC%E7%A7%BB%E5%88%B0%E5%85%A8%E8%BF%9E%E6%8E%A5%E7%8A%B6%E6%80%81%E5%90%8E%E3%80%82%20accept%20%E5%87%BD%E6%95%B0%E5%B0%B1%E8%83%BD%E6%8B%BF%E5%88%B0%E6%AD%A4socket%20fd%20%EF%BC%8C%E4%BD%86%E5%9C%A8%E8%BF%99%E8%BF%99%E5%89%8D%EF%BC%8C%E5%86%85%E6%A0%B8%E4%BC%9A%E6%8A%8A%E8%BF%99%E4%B8%AA%20request\_sock%20%E8%BF%9B%E8%A1%8C%E5%AE%8C%E6%95%B4%E5%8C%96%EF%BC%8C%E5%A6%82%EF%BC%9A%E5%88%9B%E5%BB%BA%20socket%20sock%20%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E7%BB%91%E5%AE%9AFD%E7%AD%89%E7%AD%89%E3%80%82%0A%0A%0Aaccept%0A\-\-\-\-\-%0A%E4%BB%8E%E5%A4%84%E4%BA%8E%20established%20%E7%8A%B6%E6%80%81%E7%9A%84%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%E5%A4%B4%E9%83%A8%E5%8F%96%E5%87%BA%E4%B8%80%E4%B8%AA%E5%B7%B2%E7%BB%8F%E5%AE%8C%E6%88%90%E7%9A%84%E8%BF%9E%E6%8E%A5%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%BF%99%E4%B8%AA%E9%98%9F%E5%88%97%E6%B2%A1%E6%9C%89%E5%B7%B2%E7%BB%8F%E5%AE%8C%E6%88%90%E7%9A%84%E8%BF%9E%E6%8E%A5%EF%BC%8Caccept\(\)%E5%87%BD%E6%95%B0%E5%B0%B1%E4%BC%9A%E9%98%BB%E5%A1%9E%EF%BC%8C%E7%9B%B4%E5%88%B0%E5%8F%96%E5%87%BA%E9%98%9F%E5%88%97%E4%B8%AD%E5%B7%B2%E5%AE%8C%E6%88%90%E7%9A%84%E7%94%A8%E6%88%B7%E8%BF%9E%E6%8E%A5%E4%B8%BA%E6%AD%A2%E3%80%82%E5%8F%A6%EF%BC%9A%E6%AD%A4%E5%87%BD%E6%95%B0%E4%BC%9A%E8%BF%94%E5%9B%9E%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84FD%0A%0A%E8%BF%99%E9%87%8C%E5%BE%88%E6%9C%89%E8%B6%A3%EF%BC%8C%E6%88%91%E4%BB%ACS%E7%AB%AF%E6%AD%A3%E5%B8%B8%E8%A6%81%E7%9B%91%E5%90%AC%EF%BC%8C%E5%BE%97%E4%B8%BB%E5%8A%A8%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%20socket%20FD%EF%BC%8C%E4%BD%86%E6%98%AF%20accept%20%E4%BC%9A%E6%BA%90%E5%8E%9F%E4%B8%8D%E6%96%AD%E7%9A%84%E8%BF%94%E5%9B%9E%E6%96%B0%E7%9A%84%20socket%20FD%20%EF%BC%8C%E6%9C%89%E5%95%A5%E5%8C%BA%E5%88%AB%EF%BC%9F%0A%0A%0A%E8%AF%BB%E5%86%99%E7%BC%93%E5%AD%98%E5%8C%BA%0A\-\-\-\-\-\-\-%0A%E5%86%85%E6%A0%B8%E7%BB%99%E6%AF%8F%E4%B8%80%E4%B8%AATCP%E5%A5%97%E6%8E%A5%E5%AD%97%E9%83%BD%E6%9C%89%E4%B8%80%E4%B8%AA%E5%8F%91%E9%80%81%E7%BC%93%E5%86%B2%E5%8C%BA%0ASO\_SNDBUF%E6%8E%A7%E5%88%B6%E5%A4%A7%E5%B0%8F%E3%80%82%E8%B0%83%E7%94%A8send%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%E6%97%B6\(%E9%BB%98%E8%AE%A4%E9%98%BB%E5%A1%9E%E6%A8%A1%E5%BC%8F\)%EF%BC%8C%E5%A6%82%E6%9E%9C%E7%BC%93%E5%86%B2%E5%8C%BA%E6%B2%A1%E6%BB%A1%EF%BC%8C%E8%B0%83%E7%94%A8%E7%9B%B4%E6%8E%A5%E8%BF%94%E5%9B%9E%E3%80%82%E7%B3%BB%E7%BB%9F%E5%86%85%E6%A0%B8%E5%9C%A8IP%E5%B1%82%E5%8F%91%E9%80%81%E6%95%B0%E6%8D%AE%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%B9%B6%E4%B8%8D%E6%98%AF%E6%8C%89%E7%85%A7%E6%88%91%E4%BB%AC%E8%B0%83%E7%94%A8send%E6%8E%A5%E5%8F%A3%E5%8F%91%E9%80%81%E7%9A%84%E6%95%B0%E6%8D%AE%E5%8C%85%E5%A4%A7%E5%B0%8F%E6%9D%A5%E8%BF%9B%E8%A1%8C%E5%8F%91%E9%80%81%EF%BC%8C%E5%8D%B3%E4%BD%BF%E6%88%91%E4%BB%AC%E8%B0%83%E7%94%A8send%E5%8F%91%E9%80%81%E7%9A%84%E6%95%B0%E6%8D%AE%E5%A4%A7%E5%B0%8F%E5%B0%8F%E4%BA%8E1460%E5%AD%97%E8%8A%82\(MTU\-TCP%E9%A6%96%E9%83%A8\-IP%E9%A6%96%E9%83%A8\)%E3%80%82%E5%9B%A0%E4%B8%BA%E6%88%91%E4%BB%AC%E8%B0%83%E7%94%A8send%E6%8E%A5%E5%8F%A3%E5%AE%9E%E9%99%85%E6%98%AF%E5%B0%86%E6%95%B0%E6%8D%AE%E5%A4%8D%E5%88%B6%E5%88%B0%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%AD%EF%BC%8C%E8%80%8C%E5%86%85%E6%A0%B8%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%98%AF%E6%8C%89%E7%85%A7%E6%9C%80%E5%A4%A7MSS%E5%A4%A7%E5%B0%8F\(1460%E5%AD%97%E8%8A%82\)%E4%BB%8E%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%AD%E5%8F%96%E6%95%B0%E6%8D%AE%E5%8F%91%E9%80%81%E5%87%BA%E5%8E%BB%EF%BC%8C%E5%BD%93%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%AD%E6%95%B0%E6%8D%AE%E5%B0%8F%E4%BA%8EMSS%EF%BC%8C%E5%88%99%E5%B0%86%E5%89%A9%E4%BD%99%E6%95%B0%E6%8D%AE%E5%85%A8%E9%83%A8%E5%8F%91%E9%80%81%E5%87%BA%E5%8E%BB%E3%80%82TCP%E7%9A%84%E5%8F%91%E9%80%81%E7%BC%93%E5%86%B2%E5%8C%BA%E5%BF%85%E9%A1%BB%E4%B8%BA%E5%B7%B2%E5%8F%91%E9%80%81%E7%9A%84%E6%95%B0%E6%8D%AE%E4%BF%9D%E7%95%99%E4%B8%80%E4%B8%AA%E5%89%AF%E6%9C%AC%EF%BC%8C%E7%9B%B4%E5%88%B0%E5%AE%83%E8%A2%AB%E5%AF%B9%E7%AB%AF%E7%A1%AE%E8%AE%A4%E4%B8%BA%E6%AD%A2%EF%BC%8C%E6%89%8D%E8%83%BD%E4%BB%8E%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%AD%E5%88%A0%E6%8E%89%E5%B7%B2%E7%A1%AE%E8%AE%A4%E7%9A%84%E6%95%B0%E6%8D%AE%E3%80%82%0A%0A%E5%86%85%E6%A0%B8%E7%BB%99%E6%AF%8F%E4%B8%80%E4%B8%AATCP%E5%A5%97%E6%8E%A5%E5%AD%97%E9%83%BD%E6%9C%89%E4%B8%80%E4%B8%AA%E6%8E%A5%E6%94%B6%E7%BC%93%E5%86%B2%E5%8C%BA%EF%BC%8C%0A%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87SO\_RCVBUF%E5%A5%97%E6%8E%A5%E5%AD%97%E9%80%89%E9%A1%B9%E6%9D%A5%E6%9B%B4%E6%94%B9%E3%80%82%E6%8E%A5%E6%94%B6%E7%BC%93%E5%86%B2%E5%8C%BA%E8%A2%ABTCP%E7%94%A8%E6%9D%A5%E4%BF%9D%E5%AD%98%E6%8E%A5%E6%94%B6%E5%88%B0%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E7%9B%B4%E5%88%B0%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E6%9D%A5%E8%AF%BB%E5%8F%96%E3%80%82%E5%AF%B9%E4%BA%8ETCP%E6%9D%A5%E8%AF%B4%EF%BC%8C%E6%8E%A5%E6%94%B6%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%AD%E5%8F%AF%E7%94%A8%E7%A9%BA%E9%97%B4%E7%9A%84%E5%A4%A7%E5%B0%8F%E9%99%90%E5%AE%9A%E4%BA%86TCP%E9%80%9A%E5%91%8A%E5%AF%B9%E7%AB%AF%E7%9A%84%E7%AA%97%E5%8F%A3%E5%A4%A7%E5%B0%8F%E3%80%82TCP%E5%A5%97%E6%8E%A5%E5%AD%97%E7%9A%84%E6%8E%A5%E6%94%B6%E7%BC%93%E5%86%B2%E5%8C%BA%E4%B8%8D%E8%83%BD%E6%BA%A2%E5%87%BA%EF%BC%8C%E6%89%80%E4%BB%A5%E5%8F%91%E9%80%81%E7%AB%AF%E4%B8%8D%E8%83%BD%E5%8F%91%E9%80%81%E8%B6%85%E8%BF%87%E6%8E%A5%E6%94%B6%E7%AB%AF%E9%80%9A%E7%9F%A5%E7%9A%84%E7%AA%97%E5%8F%A3%E5%A4%A7%E5%B0%8F%EF%BC%8C%E5%90%A6%E5%88%99%E5%9C%A8%E6%8E%A5%E6%94%B6%E7%AB%AF%E5%B0%86%E4%B8%A2%E5%BC%83%E6%95%B0%E6%8D%AE%E5%8C%85%E3%80%82%E6%94%B6%E7%AB%AF%E9%80%9A%E7%9F%A5%E5%8F%91%E7%AB%AF%EF%BC%8C%E6%8E%A5%E6%94%B6%E7%AA%97%E5%8F%A3%E5%85%B3%E9%97%AD%EF%BC%88win%3D0%EF%BC%89%EF%BC%8C%E4%BB%8E%E8%80%8C%E4%BF%9D%E8%AF%81%E4%BA%86TCP%E6%98%AF%E5%8F%AF%E9%9D%A0%E4%BC%A0%E8%BE%93%0A%0A%E4%BB%A5%E4%B8%8A%E4%B8%A4%E4%B8%AA%E9%83%BD%E4%BB%A5%E9%98%9F%E5%88%97%E7%BB%B4%E6%8A%A4%0A%0A%0A%E9%98%BB%E5%A1%9E%0A\-\-\-\-%0A%E5%85%88%E5%8C%BA%E5%88%864%E4%B8%AA%E6%A6%82%E5%BF%B5%EF%BC%9A%0A1.%20%E9%98%BB%E5%A1%9E\(blocking\)%EF%BC%9A%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C%E5%88%B0%E6%9F%90%E4%B8%AA%E7%82%B9~%E7%AD%89%E5%BE%85%E7%9D%80%E7%BB%93%E6%9E%9C%E8%BF%94%E5%9B%9E%E4%B8%8D%E5%8A%A8%E4%BA%86%EF%BC%8C%E8%BF%9B%E7%A8%8B%E8%A2%AB%E6%8C%82%E8%B5%B7%E3%80%82%E5%A6%82%EF%BC%9Aaccept%E3%80%81recv%E7%AD%89%0A2.%20%E9%9D%9E%E9%98%BB%E5%A1%9E\(non\-blocking\)%EF%BC%9A%E6%89%A7%E8%A1%8C%E5%88%B0%E6%9F%90%E4%B8%AA%E7%82%B9%EF%BC%8C%E7%AB%8B%E5%88%BB%E8%BF%94%E5%9B%9E%EF%BC%8C%E4%B8%8D%E7%AE%A1%E5%85%B6%E5%AE%83%E7%9A%84%E3%80%82%0A3.%20%E5%90%8C%E6%AD%A5\(synchronous\)%EF%BC%9A%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C%E5%88%B0%E6%9F%90%E4%B8%AA%E7%82%B9~%E7%AD%89%E5%BE%85%E7%9D%80%E7%BB%93%E6%9E%9C%E8%BF%94%E5%9B%9E%E4%B8%8D%E5%8A%A8%E4%BA%86%EF%BC%8C%E8%BF%9B%E7%A8%8B%E4%BE%9D%E7%84%B6%E5%A4%84%E4%BA%8E%E6%B4%BB%E5%8A%A8%E7%8A%B6%E6%80%81%0A4.%20%E5%BC%82%E6%AD%A5\(asynchronous\)%EF%BC%9A%E6%89%A7%E8%A1%8C%E5%88%B0%E6%9F%90%E4%B8%AA%E7%82%B9%EF%BC%8C%E7%AB%8B%E5%88%BB%E8%BF%94%E5%9B%9E%EF%BC%8C%E4%B8%8D%E7%AE%A1%E5%85%B6%E5%AE%83%E7%9A%84%E3%80%82%0A%0A%0A%E5%BE%80%E5%BA%95%E5%B1%82%E6%AC%A1%E7%9C%8B%E7%9A%84%E8%AF%9D%EF%BC%8C%E9%98%BB%E5%A1%9E%E5%85%B6%E5%AE%9E%E6%98%AF%EF%BC%9A%E5%BD%93%E8%BF%9B%E7%A8%8B%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%8C%E8%A2%AB%E8%B0%83%E5%BA%A6%E5%88%B0%20CPU%E4%B8%8A%EF%BC%8C%E5%BC%80%E5%A7%8B%E6%89%A7%E8%A1%8C%E3%80%82%E5%9C%A8%E6%89%A7%E8%A1%8C%E7%9A%84%E8%BF%87%E7%A8%8B%E4%B8%AD%EF%BC%8C%E5%8F%91%E7%94%9F%E4%BA%86%E7%9D%A1%E7%9C%A0%EF%BC%8C%E5%A6%82%EF%BC%9Asleep%20http\-request%20%E7%AD%89%EF%BC%8C%E6%AF%94%E8%BE%83%E8%80%97%E6%97%B6%E7%9A%84%E6%93%8D%E4%BD%9C%E6%88%96%E4%B8%BB%E5%8A%A8%E7%9D%A1%E7%9C%A0%EF%BC%8C%E6%AD%A4%E6%97%B6%20OS%20%E5%B0%B1%E6%8A%8A%E8%AF%A5%E8%BF%9B%E7%A8%8B%E6%89%94%E5%88%B0%E7%9D%A1%E7%9C%A0%E9%98%9F%E5%88%97%E4%B8%AD%EF%BC%8C%E6%9B%B4%E6%96%B0%E8%BF%9B%E7%A8%8B%E7%8A%B6%E6%80%81%E3%80%82%E8%AE%A9%E5%87%BA%20CPU%20%E6%89%A7%E8%A1%8C%E6%9D%83%E9%99%90%EF%BC%8C%E7%BB%99%E5%88%B0%E5%85%B6%E5%AE%83%E8%BF%9B%E7%A8%8B%E7%BB%A7%E7%BB%AD%E6%89%A7%E8%A1%8C%E3%80%82%20%0A%0A%E6%89%80%E4%BB%A5%E9%98%BB%E5%A1%9E%E8%BF%99%E4%B8%9C%E8%A5%BF%EF%BC%8C%E6%9B%B4%E5%83%8F%E6%98%AFOS%E4%B8%BA%E4%BA%86%E5%B9%B3%E8%A1%A1%E8%BF%9B%E7%A8%8B%E6%89%A7%E8%A1%8C%E6%97%B6%E9%97%B4%E8%80%8C%E4%BA%A7%E7%94%9F%E7%9A%84%E3%80%82%0A%3E%E5%BD%93%E7%84%B6%E4%B9%9F%E6%9C%89%E4%B8%BB%E5%8A%A8%E7%9A%84%EF%BC%8C%E5%A6%82%EF%BC%9ASLEEP%20%E8%BF%99%E7%A7%8D%E4%B8%8D%E8%AE%A8%E8%AE%BA%0A%0A%0Aepoll%0A\-\-\-%0Ask\_wq%0A%0A%E6%80%BB%E7%BB%93%0A\-\-\-%0A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%80%E5%AE%9A%E8%A6%81%E5%86%85%E6%A0%B8%E4%BB%8B%E5%85%A5%EF%BC%9F%0A%E5%9B%A0%E4%B8%BA%20%E7%BD%91%E7%BB%9CIO%E8%BF%87%E4%BA%8E%E5%A4%8D%E6%9D%82%EF%BC%8C%E7%89%B5%E6%89%AF%E4%BA%86%E5%A4%A7%E9%83%A8%E5%88%86%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%B5%84%E6%BA%90%EF%BC%8C%E5%8C%85%E6%8B%AC%E7%A1%AC%E4%BB%B6%EF%BC%8C%E5%A6%82%E7%BD%91%E5%8D%A1%E3%80%82%E9%9C%80%E8%A6%81%E7%9A%84%E6%9D%83%E9%99%90%E8%BF%87%E5%A4%9A%EF%BC%8C%E5%86%85%E6%A0%B8%E7%9B%B4%E6%8E%A5%E6%9D%A5%E5%8F%82%E4%B8%8E%E6%9B%B4%E6%96%B9%E4%BE%BF%E3%80%82%E6%9C%80%E7%BB%88%EF%BC%8C%E9%80%9A%E8%BF%87%E9%80%9A%E8%BF%87%20%20socket%20API%20%E7%BB%99%E5%88%B0%E7%94%A8%E6%88%B7%0A%0A%E4%BB%8E%E7%94%A8%E6%88%B7%E7%9A%84%E8%A7%92%E5%BA%A6%E7%9C%8B%EF%BC%8Csocket%20%E6%9B%B4%E5%83%8F%E6%98%AF%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84IO%EF%BC%8C%E8%BF%99%E4%B9%9F%E6%98%AFLINUX%E5%93%B2%E5%AD%A6%E7%9A%84%EF%BC%9A%E4%B8%87%E7%89%A9%E7%9A%86%E6%96%87%E4%BB%B6%E3%80%82%E6%94%B6%E6%B6%88%E6%81%AF%20%E6%98%AF%E8%AF%BB%20%EF%BC%8C%E5%8F%91%E6%B6%88%E6%81%AF%E6%98%AF%E5%86%99%E3%80%82%0A%0A%E5%8F%A6%E5%A4%96%EF%BC%8C%E4%BB%8E%E5%A4%9A%E5%8D%8F%E8%AE%AE%2F%E5%88%86%E5%B1%82%20%E6%9D%A5%E7%9C%8B%EF%BC%8C%E4%B8%8D%E7%AE%A1%E4%BB%80%E4%B9%88%E5%8D%8F%E8%AE%AE%E6%9C%80%E7%BB%88%E8%A6%81%E5%AE%9E%E7%8E%B0%E7%9A%84%E7%9B%AE%E6%A0%87%E5%B0%B1%E6%98%AF%EF%BC%9A%E6%94%B6%2F%E5%8F%91%E6%B6%88%E6%81%AF%E3%80%82%0A%0A
