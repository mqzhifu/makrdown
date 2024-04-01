# 概览

大家认知中的异常处理：try catch 

GO的异常错误机制：error defer panic recover 

>GO 点反人类，虽然也看了文章，大概理解这么设计，但真是苦啊....

# error


已知错误，比如：函数如果正常执行，返回的是 nil，如果有错误即不为nil

优点：程序员永远手动处理，手动解决，都是已知的
缺点：程序员大师充斥着类似的判断，很恶心人

## panic


程序运行中，发现错误，如：空指针。
程序直接停止，报错

捕捉方法：defer\+recover

这东西它只有在当前调用函数内使用，且还得是同一协程，且还得必须用defer配合用。
并不像 try catch ，在最上层监听一下，不管下面多少级的函数，只要出错，就直接能捕获。只需要写一次即可。

简单说就是：但凡你认为程序可能出错的地方就得有个 defer+recover 函数


所以 ，只要项目大一点，可能到处都是这种鬼东西。

## fatal

跟 panic 差不多，直接叫停程序，但不能捕获
>这个真是反人类中的反人类

1. all goroutines are asleep \- deadlock\!
2. concurrent map read and map write
3. concurrent map writes
4. runtime: out of memory
5. unexpected signal during runtime execution
6. stack overflow
7. found bad pointer in Go heap
8. signal SIGSEGV: segmentation violation

## defer\(\)

延迟函数/析构函数：函数执行完毕 ，会自动执行此函数
>有点析构函数的意思


recover 就得放在此函数内才生效，才能捕获panic

>这东西，看网上一堆面试题，感觉玩出花了~直接分析源码

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

> defer func()

defer 为关键词，最终被解释成一个函数：runtime.deferproc

```go
func deferproc(siz int32, fn *funcval) { // arguments of fn follow fn
    //用户goroutine才能使用defer
    if getg().m.curg != getg() {
        // go code on the system stack can't defer
        throw("defer on system stack")
    }
    //也就是调用deferproc之前的rsp寄存器的值
    sp := getcallersp()
    // argp指向defer函数的第一个参数
    argp := uintptr(unsafe.Pointer(&fn)) + unsafe.Sizeof(fn)
    // 存储的是 caller 中，call deferproc 的下一条指令的地址
    // deferproc函数的返回地址
    callerpc := getcallerpc()

    //创建defer
    d := newdefer(siz)
    if d._panic != nil {
        throw("deferproc: d.panic != nil after newdefer")
    }
    //需要延迟执行的函数
    d.fn = fn
    //记录deferproc函数的返回地址
    d.pc = callerpc
    //调用deferproc之前rsp寄存器的值
    d.sp = sp
    switch siz {
    case 0:
        // Do nothing.
    case sys.PtrSize:
        *(*uintptr)(deferArgs(d)) = *(*uintptr)(unsafe.Pointer(argp))
    default:
        //通过memmove拷贝defered函数的参数
        memmove(deferArgs(d), unsafe.Pointer(argp), uintptr(siz))
    }

    // deferproc通常都会返回0
    return0()
}
```

原理：保存返回地址等等，但它的核心是 newdefer


```go
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

创建：\_defer 结构体，获取当前协程信息，将新的 \_defer结构体中的link成员变量，指向当前协程中的\_defer

```go
type _defer struct {
	siz     int32 // 参数和返回值的内存大小
	started bool
	heap    bool    // 区分该结构是在栈上分配的，还是对上分配的
	sp        uintptr  // sp 计数器值，栈指针；
	pc        uintptr  // pc 计数器值，程序计数器；
	fn        *funcval // defer 传入的函数地址，也就是延后执行的函数;
	_panic    *_panic  // panic that is running defer
	link      *_defer   // 链表,指向下一个 _defer
}
```

当正常函数执行完，开始执行该 g 的 defer 列表中的 defer函数了，每个函数执行完后，就会走deferReturn

```go

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

获取当前协程信息，如果有 defer，则顺着当前协程信息，找到 \_defer连接队列，并判断是否属于此函数，然后从头部开始依次执行

源码分析：
1. 用户在某个函数中创建了一个 derfer func()
2. 编译器，遇到  derfer 关键字，开始解析
3. derfer 关键字会转换成  deferproc

执行分析过程：

1. G 结构体中，有一个成员变量：* \_defer
2. 当一个函数正常执行完毕，到 return 时
3. 先保存 return 的值，到一个临时变量中
4. 开始找 *\_defer  是否后面挂着值
5. 如果有，就开始执行
6. 执行完一个 defer 函数后，判断后来是否还有derver
7. 直到所有 derfer 函数执行完
8. 把第3步保存的临时值，返回



就是一个保存\<回调函数\>的链表（栈），该链表基于某个协程上。当函数执行return触发该链表的执行过程。return时defer执行顺序：后进先出

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
