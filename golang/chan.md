# chan


![[h_chan.png]]
先上源码


> go/src/runtime/chan.go

```go
type hchan struct {
	qcount   uint           // 队列中的元素总量
	dataqsiz uint           // 缓冲大小，=make(chan T, x)中的x
	buf      unsafe.Pointer // 缓冲区地址
	elemsize uint16         // 元素大小，单位为字节
	closed   uint32         //chan关闭标记
	elemtype *_type         // 元素类型
	sendx    uint   // 待发送元素在缓冲器中的索引
	recvx    uint   // 待接收元素在缓冲器中的索引
	recvq    waitq  // 接收等待队列，用于阻塞接收协程
	sendq    waitq  // 发送等待队列，用于阻塞发送协程
	
	lock mutex  //自旋锁
}
```


buf：存数据的队列（数组结构）
recvq：哪些G在等待接收(记录缓存中的数据读取到哪里了)
sendq：哪些G在等待发送(记录缓存中的数据发送到哪里了)
sendx：buf队列，可以发送数据的索引值
recvx ：buf队列，可以接收的数据的索引值

>感觉就这5个成员变量基本就说明了整个使用的过程

读写的时候，加个锁，有其它协程过来，放弃等待队列中，然后睡眠......


waitq：被阻塞的/等待的G，使用这个结构体

```go
 type waitq struct {
     first *sudog
     last *sudog
 }
```


当写入方，发现队列满了后，创建一个 sudog：
1. 自己的G地址
2. 写不进入的数据的地址
3. chann 的地址
加入到 sendx 中，让出当前线程执行权。

此时，接收方，开始读缓存数据，读出一条数据后，如果写等待队列有值，便通知调度器，让G改成可执行状态。等待线程执行。


还有4个函数，就不上源码了

1. makechan:创建一个chan
2. chansend1:给一个chan发送一条消息
3. chanrecv1:从一个chan接收消息
4. closechan:关闭一个管道



正常收发就不写了，需要注意的，如下：

closed ：先是打上一个标识，但管道里的值不做清空，释放所有recvq，并存入glist等待唤醒 ，再释放所有sendq，同样存入golist，等待唤醒 ，最后唤醒所有glist里的协程

这里有趣的是：如果是阻塞模式，一但被关闭就被告知不用阻塞了，唯一能做判定的是：读出的值如果是一个类型的默认值即为被关闭了，这个挺模糊的


