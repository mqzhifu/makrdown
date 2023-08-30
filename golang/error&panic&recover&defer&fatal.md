# error&panic&recover&defer&fatal

先吐槽一下GO的异常错误机制，真是够另类~或者说有点反人类，虽然也看了文章，大概理解这么设计，但真是苦啊....

用惯了try catch模式

步入正题，GO里没有try catch，只有error defer panic recover ，且fatal error不能捕获。一切，全靠程序手写...

## error

已知错误，比如：add\(\)error，函数如果正常执行，返回的是nil，如果有错误即不为nil，简单理解就是：这种已知的错误，尽量多使用，多判断，这样就能减少panic fatal

## panic

直接叫停程序，前提是不使用defer\+recover。好消息：这个东西它能捕获，坏消息是：这东西它只有在当前调用函数内使用，且还得是同一协程，且还得必须用defer配合用。简单说就是：但凡你认为程序可能出错的地方就得有个defer\+recover函数。所以 ，只要项目大一点，可能到处都是这种鬼东西。

## fatal

这个真是反人类中的反人类，跟panic一样，直接叫停程序，并且不能捕获。

1. all goroutines are asleep \- deadlock\!
2. concurrent map read and map write
3. concurrent map writes
4. runtime: out of memory
5. unexpected signal during runtime execution
6. stack overflow
7. found bad pointer in Go heap
8. signal SIGSEGV: segmentation violation

## defer\(\)

延迟函数/析构函数:函数执行完毕|return|panic，会自动执行此函数

> recover就得放在此函数内才生效，才能捕获panic

这东西，看网上一堆面试题，感觉玩出花了~直接分析源码

主结构体：

```
type _defer struct {
  siz int32
  started bool
  heap bool
  openDefer bool
  sp uintptr // 栈指针
  pc uintptr // 调用方的程序计数器
  fn *funcval // 传入的函数
  _panic *_panic
  link *_defer // 指向下一个执行的defer函数
}
```

执行流程：创建

> defer aaaa\(\)

defer为关键词，最终被解释成一个函数：deferproc，原理：保存返回地址等等，但它的核心是newdefer

```
func newdefer(siz int32) *_defer {
  var d *_defer
  sc := deferclass(uintptr(siz))
  gp := getg() // 获得当前的goroutine
  ...
  d.link = gp._defer // 现在新的defer函数的link指向了当前的defer函数
  gp._defer = d // 新的defer函数现在是第一个被调用的函数了
  return d
}
```

创建：\_defer结构体，获取当前协程信息，将新的\_defer结构体中的link成员变量，指向当前协程中的\_defer

```

func deferreturn(arg0 uintptr) {
  gp := getg() // 获得当前的goroutine
  d := gp._defer
  if d == nil { // 如果没有defer函数，直接return
    return
  }
  ...
  fn := d.fn // 获得defer的func函数
  d.fn = nil // 重置
  gp._defer = d.link // 将前一个defer函数attach到当前goroutine
  freedefer(d) // 释放defer函数
  _ = fn.fn // 执行defer的func函数
  jmpdefer(fn, uintptr(unsafe.Pointer(&arg0)))
```

获取当前协程信息，如果有defer，则顺着当前协程信息，找到\_defer连接队列，并判断是否属于此函数，然后从头部开始依次执行

最终流程图如下：

```
graph LR
defer-->deferproc 
deferproc-->newdefer
newdefer-->new_defer_struct
user_func-->new_defer_struct
new_defer_struct-->当前协程_defer链表
deferreturn-->当前协程_defer链表
```

源码分析结果：就是一个保存\<回调函数\>的链表（栈），该链表基于某个协程上。当函数执行return触发该链表的执行过程。return时defer执行顺序：后进先出

单说refer的原理很简单，但是配合其它语法及使用场景，就麻烦了。

影响refer的X因素：

1. return
2. os.exit
3. panic
4. fatal

return:

原理：

1. 是否有返回值
2. 如果有返回值，判断是否函数尾部已给返回值定义了变量名
3. 如果未定义，新创建一个临时变量ret，再将返回值保存于ret中
4. 检查是否有defer，如果有就执行
5. 返回刚刚保存的临时变量

我们发现defer居然是插在return执行过程中的一步，也就是说：return是非原子操作。而如果我们定义了返回值的变量名，且有defer，且在defer中更改了返回值，那么就挺危险。

defer在一个函数中，可以定义N个。可以在任意位置定义。

os.Exit:这是个无敌的东西，触发即立刻停止忽略一切，包括defer

panic会触发defer，不然没法recover了

注意事项：

1. 变量作用域|影响了返回值
    地址|引用
    返回值为函数自带的变量名
    多defer函数同时修改父级变量
2. 受其它因素\(exit|fatalreturn\)影响，要知道是否能正确执行了defer
3. defer整体链是挂在一个协程中的
4. defer函数里：还可以再用协程，但还是尽量不用

应用场景：

1. lock/unlock
2. openFD/closeFD
3. 发生了运行时异常，容错处理

## log.fatal

1. 输出内容
2. 立刻停止
3. 不执行defer

## log.panic
