# chan

先上源码

> go/src/runtime/chan.go

```
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

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex  //自旋锁
}
```

看这个头文件，感觉挺轻巧的，就是一个简单的结构体外挂一个queue指针

还有4个函数，就不上源码了

1. makechan:创建一个chan
2. chansend1:给一个chan发送一条消息
3. chanrecv1:从一个chan接收消息
4. closechan:关闭一个管道

可参考图

正常收发就不写了，需要注意的，如下：

closed ：先是打上一个标识，但管道里的值不做清空，释放所有recvq，并存入glist等待唤醒 ，再释放所有sendq，同样存入golist，等待唤醒 ，最后唤醒所有glist里的协程

这里有趣的是：如果是阻塞模式，一但被关闭就被告知不用阻塞了，唯一能做判定的是：读出的值如果是一个类型的默认值即为被关闭了，这个挺模糊的

![image.png](image/image.png)
