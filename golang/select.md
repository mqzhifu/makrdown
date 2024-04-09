编译的时候，会转换 select 代码段，其中有下面几种情况：

1. 没有 case
2. 一个 case
3. 一个 case + default
4. 多个 case + defualt
5. 多个 case

没有 case 这种最简单，直接 gopark ，也可以理解为：死循环，并让出执行权
```go
func block() {
	gopark(nil, nil, waitReasonSelectNoCases, traceEvGoStop, 1)
}
```

一个 case 这种情况比较少，因为完全没必要用 select ，直接一行代码：   <- ch  ，自己阻塞不就得了
实际上编译 select 的时候，也是这么干的


一个 case + default:

```go
if selectnbsend(ch, i) {
    ...
} else {
    ...
}

func selectnbsend(c *hchan, elem unsafe.Pointer) (selected bool) {
	return chansend(c, elem, false, getcallerpc())
}

```


核心函数：selectnbsend-> chansend

chansend：是管道里面使用的，就是在 收发消息的时候，看是否有缓冲可IO数据，如果没有，不阻塞该协程，立刻返回



多个 case：找寻所有  chanel 中哪些可以IO的，有就执行，没有就，把自己注册成sudo ，发送给所有的chanel



多个 case + default : 不挂起，直接 走 default，重复循环











