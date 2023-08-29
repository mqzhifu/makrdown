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

%E5%85%88%E5%90%90%E6%A7%BD%E4%B8%80%E4%B8%8BGO%E7%9A%84%E5%BC%82%E5%B8%B8%E9%94%99%E8%AF%AF%E6%9C%BA%E5%88%B6%EF%BC%8C%E7%9C%9F%E6%98%AF%E5%A4%9F%E5%8F%A6%E7%B1%BB~%E6%88%96%E8%80%85%E8%AF%B4%E6%9C%89%E7%82%B9%E5%8F%8D%E4%BA%BA%E7%B1%BB%EF%BC%8C%E8%99%BD%E7%84%B6%E4%B9%9F%E7%9C%8B%E4%BA%86%E6%96%87%E7%AB%A0%EF%BC%8C%E5%A4%A7%E6%A6%82%E7%90%86%E8%A7%A3%E8%BF%99%E4%B9%88%E8%AE%BE%E8%AE%A1%EF%BC%8C%E4%BD%86%E7%9C%9F%E6%98%AF%E8%8B%A6%E5%95%8A....%0A%0A%E7%94%A8%E6%83%AF%E4%BA%86try%20catch%E6%A8%A1%E5%BC%8F%0A%0A%E6%AD%A5%E5%85%A5%E6%AD%A3%E9%A2%98%EF%BC%8CGO%E9%87%8C%E6%B2%A1%E6%9C%89try%20catch%EF%BC%8C%E5%8F%AA%E6%9C%89error%20defer%20panic%20recover%20%EF%BC%8C%E4%B8%94fatal%20error%E4%B8%8D%E8%83%BD%E6%8D%95%E8%8E%B7%E3%80%82%E4%B8%80%E5%88%87%EF%BC%8C%E5%85%A8%E9%9D%A0%E7%A8%8B%E5%BA%8F%E6%89%8B%E5%86%99...%0A%0Aerror%0A\-\-\-\-\-\-\-\-\-%0A%E5%B7%B2%E7%9F%A5%E9%94%99%E8%AF%AF%EF%BC%8C%E6%AF%94%E5%A6%82%EF%BC%9Aadd\(\)error%EF%BC%8C%E5%87%BD%E6%95%B0%E5%A6%82%E6%9E%9C%E6%AD%A3%E5%B8%B8%E6%89%A7%E8%A1%8C%EF%BC%8C%E8%BF%94%E5%9B%9E%E7%9A%84%E6%98%AFnil%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89%E9%94%99%E8%AF%AF%E5%8D%B3%E4%B8%8D%E4%B8%BAnil%EF%BC%8C%E7%AE%80%E5%8D%95%E7%90%86%E8%A7%A3%E5%B0%B1%E6%98%AF%EF%BC%9A%E8%BF%99%E7%A7%8D%E5%B7%B2%E7%9F%A5%E7%9A%84%E9%94%99%E8%AF%AF%EF%BC%8C%E5%B0%BD%E9%87%8F%E5%A4%9A%E4%BD%BF%E7%94%A8%EF%BC%8C%E5%A4%9A%E5%88%A4%E6%96%AD%EF%BC%8C%E8%BF%99%E6%A0%B7%E5%B0%B1%E8%83%BD%E5%87%8F%E5%B0%91panic%20fatal%0A%0Apanic%0A\-\-\-\-\-\-\-\-\-%0A%E7%9B%B4%E6%8E%A5%E5%8F%AB%E5%81%9C%E7%A8%8B%E5%BA%8F%EF%BC%8C%E5%89%8D%E6%8F%90%E6%98%AF%E4%B8%8D%E4%BD%BF%E7%94%A8defer%2Brecover%E3%80%82%E5%A5%BD%E6%B6%88%E6%81%AF%EF%BC%9A%E8%BF%99%E4%B8%AA%E4%B8%9C%E8%A5%BF%E5%AE%83%E8%83%BD%E6%8D%95%E8%8E%B7%EF%BC%8C%E5%9D%8F%E6%B6%88%E6%81%AF%E6%98%AF%EF%BC%9A%E8%BF%99%E4%B8%9C%E8%A5%BF%E5%AE%83%E5%8F%AA%E6%9C%89%E5%9C%A8%E5%BD%93%E5%89%8D%E8%B0%83%E7%94%A8%E5%87%BD%E6%95%B0%E5%86%85%E4%BD%BF%E7%94%A8%EF%BC%8C%E4%B8%94%E8%BF%98%E5%BE%97%E6%98%AF%E5%90%8C%E4%B8%80%E5%8D%8F%E7%A8%8B%EF%BC%8C%E4%B8%94%E8%BF%98%E5%BE%97%E5%BF%85%E9%A1%BB%E7%94%A8defer%E9%85%8D%E5%90%88%E7%94%A8%E3%80%82%E7%AE%80%E5%8D%95%E8%AF%B4%E5%B0%B1%E6%98%AF%EF%BC%9A%E4%BD%86%E5%87%A1%E4%BD%A0%E8%AE%A4%E4%B8%BA%E7%A8%8B%E5%BA%8F%E5%8F%AF%E8%83%BD%E5%87%BA%E9%94%99%E7%9A%84%E5%9C%B0%E6%96%B9%E5%B0%B1%E5%BE%97%E6%9C%89%E4%B8%AAdefer%2Brecover%E5%87%BD%E6%95%B0%E3%80%82%E6%89%80%E4%BB%A5%20%EF%BC%8C%E5%8F%AA%E8%A6%81%E9%A1%B9%E7%9B%AE%E5%A4%A7%E4%B8%80%E7%82%B9%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%88%B0%E5%A4%84%E9%83%BD%E6%98%AF%E8%BF%99%E7%A7%8D%E9%AC%BC%E4%B8%9C%E8%A5%BF%E3%80%82%0A%0Afatal%0A\-\-\-\-\-\-\-\-\-\-\-\-\-%0A%E8%BF%99%E4%B8%AA%E7%9C%9F%E6%98%AF%E5%8F%8D%E4%BA%BA%E7%B1%BB%E4%B8%AD%E7%9A%84%E5%8F%8D%E4%BA%BA%E7%B1%BB%EF%BC%8C%E8%B7%9Fpanic%E4%B8%80%E6%A0%B7%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%8F%AB%E5%81%9C%E7%A8%8B%E5%BA%8F%EF%BC%8C%E5%B9%B6%E4%B8%94%E4%B8%8D%E8%83%BD%E6%8D%95%E8%8E%B7%E3%80%82%0A%0A1.%20all%20goroutines%20are%20asleep%20\-%20deadlock\!%0A2.%20concurrent%20map%20read%20and%20map%20write%0A3.%20concurrent%20map%20writes%0A3.%20runtime%3A%20out%20of%20memory%0A4.%20unexpected%20signal%20during%20runtime%20execution%0A5.%20stack%20overflow%0A6.%20found%20bad%20pointer%20in%20Go%20heap%0A7.%20signal%20SIGSEGV%3A%20segmentation%20violation%0A8.%20%0A%0A%0Adefer\(\)%0A\-\-\-\-\-\-\-\-\-\-\-%0A%E5%BB%B6%E8%BF%9F%E5%87%BD%E6%95%B0%2F%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0%3A%E5%87%BD%E6%95%B0%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%AF%95%7Creturn%7Cpanic%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%8A%A8%E6%89%A7%E8%A1%8C%E6%AD%A4%E5%87%BD%E6%95%B0%0A%0A%3Erecover%E5%B0%B1%E5%BE%97%E6%94%BE%E5%9C%A8%E6%AD%A4%E5%87%BD%E6%95%B0%E5%86%85%E6%89%8D%E7%94%9F%E6%95%88%EF%BC%8C%E6%89%8D%E8%83%BD%E6%8D%95%E8%8E%B7panic%0A%0A%E8%BF%99%E4%B8%9C%E8%A5%BF%EF%BC%8C%E7%9C%8B%E7%BD%91%E4%B8%8A%E4%B8%80%E5%A0%86%E9%9D%A2%E8%AF%95%E9%A2%98%EF%BC%8C%E6%84%9F%E8%A7%89%E7%8E%A9%E5%87%BA%E8%8A%B1%E4%BA%86~%E7%9B%B4%E6%8E%A5%E5%88%86%E6%9E%90%E6%BA%90%E7%A0%81%0A%0A%E4%B8%BB%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%9A%0A%60%60%60%0Atype%20\_defer%20struct%20%7B%0A%20%20siz%20int32%0A%20%20started%20bool%0A%20%20heap%20bool%0A%20%20openDefer%20bool%0A%20%20sp%20uintptr%20%2F%2F%20%E6%A0%88%E6%8C%87%E9%92%88%0A%20%20pc%20uintptr%20%2F%2F%20%E8%B0%83%E7%94%A8%E6%96%B9%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%A1%E6%95%B0%E5%99%A8%0A%20%20fn%20\*funcval%20%2F%2F%20%E4%BC%A0%E5%85%A5%E7%9A%84%E5%87%BD%E6%95%B0%0A%20%20\_panic%20\*\_panic%0A%20%20link%20\*\_defer%20%2F%2F%20%E6%8C%87%E5%90%91%E4%B8%8B%E4%B8%80%E4%B8%AA%E6%89%A7%E8%A1%8C%E7%9A%84defer%E5%87%BD%E6%95%B0%0A%7D%0A%60%60%60%0A%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%EF%BC%9A%E5%88%9B%E5%BB%BA%0A%0A%3Edefer%20aaaa\(\)%20%20%20%0A%0Adefer%E4%B8%BA%E5%85%B3%E9%94%AE%E8%AF%8D%EF%BC%8C%E6%9C%80%E7%BB%88%E8%A2%AB%E8%A7%A3%E9%87%8A%E6%88%90%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%EF%BC%9Adeferproc%EF%BC%8C%E5%8E%9F%E7%90%86%EF%BC%9A%E4%BF%9D%E5%AD%98%E8%BF%94%E5%9B%9E%E5%9C%B0%E5%9D%80%E7%AD%89%E7%AD%89%EF%BC%8C%E4%BD%86%E5%AE%83%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AFnewdefer%0A%0A%60%60%60%0Afunc%20newdefer\(siz%20int32\)%20\*\_defer%20%7B%0A%20%20var%20d%20\*\_defer%0A%20%20sc%20%3A%3D%20deferclass\(uintptr\(siz\)\)%0A%20%20gp%20%3A%3D%20getg\(\)%20%2F%2F%20%E8%8E%B7%E5%BE%97%E5%BD%93%E5%89%8D%E7%9A%84goroutine%0A%20%20...%0A%20%20d.link%20%3D%20gp.\_defer%20%2F%2F%20%E7%8E%B0%E5%9C%A8%E6%96%B0%E7%9A%84defer%E5%87%BD%E6%95%B0%E7%9A%84link%E6%8C%87%E5%90%91%E4%BA%86%E5%BD%93%E5%89%8D%E7%9A%84defer%E5%87%BD%E6%95%B0%0A%20%20gp.\_defer%20%3D%20d%20%2F%2F%20%E6%96%B0%E7%9A%84defer%E5%87%BD%E6%95%B0%E7%8E%B0%E5%9C%A8%E6%98%AF%E7%AC%AC%E4%B8%80%E4%B8%AA%E8%A2%AB%E8%B0%83%E7%94%A8%E7%9A%84%E5%87%BD%E6%95%B0%E4%BA%86%0A%20%20return%20d%0A%7D%0A%60%60%60%0A%E5%88%9B%E5%BB%BA%EF%BC%9A\_defer%E7%BB%93%E6%9E%84%E4%BD%93%EF%BC%8C%E8%8E%B7%E5%8F%96%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%B0%86%E6%96%B0%E7%9A%84\_defer%E7%BB%93%E6%9E%84%E4%BD%93%E4%B8%AD%E7%9A%84link%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%EF%BC%8C%E6%8C%87%E5%90%91%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B%E4%B8%AD%E7%9A%84\_defer%0A%0A%0A%60%60%60%0A%0Afunc%20deferreturn\(arg0%20uintptr\)%20%7B%0A%20%20gp%20%3A%3D%20getg\(\)%20%2F%2F%20%E8%8E%B7%E5%BE%97%E5%BD%93%E5%89%8D%E7%9A%84goroutine%0A%20%20d%20%3A%3D%20gp.\_defer%0A%20%20if%20d%20%3D%3D%20nil%20%7B%20%2F%2F%20%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89defer%E5%87%BD%E6%95%B0%EF%BC%8C%E7%9B%B4%E6%8E%A5return%0A%20%20%20%20return%0A%20%20%7D%0A%20%20...%0A%20%20fn%20%3A%3D%20d.fn%20%2F%2F%20%E8%8E%B7%E5%BE%97defer%E7%9A%84func%E5%87%BD%E6%95%B0%0A%20%20d.fn%20%3D%20nil%20%2F%2F%20%E9%87%8D%E7%BD%AE%0A%20%20gp.\_defer%20%3D%20d.link%20%2F%2F%20%E5%B0%86%E5%89%8D%E4%B8%80%E4%B8%AAdefer%E5%87%BD%E6%95%B0attach%E5%88%B0%E5%BD%93%E5%89%8Dgoroutine%0A%20%20freedefer\(d\)%20%2F%2F%20%E9%87%8A%E6%94%BEdefer%E5%87%BD%E6%95%B0%0A%20%20\_%20%3D%20fn.fn%20%2F%2F%20%E6%89%A7%E8%A1%8Cdefer%E7%9A%84func%E5%87%BD%E6%95%B0%0A%20%20jmpdefer\(fn%2C%20uintptr\(unsafe.Pointer\(%26arg0\)\)\)%0A%60%60%60%0A%0A%E8%8E%B7%E5%8F%96%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89defer%EF%BC%8C%E5%88%99%E9%A1%BA%E7%9D%80%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B%E4%BF%A1%E6%81%AF%EF%BC%8C%E6%89%BE%E5%88%B0\_defer%E8%BF%9E%E6%8E%A5%E9%98%9F%E5%88%97%EF%BC%8C%E5%B9%B6%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E5%B1%9E%E4%BA%8E%E6%AD%A4%E5%87%BD%E6%95%B0%EF%BC%8C%E7%84%B6%E5%90%8E%E4%BB%8E%E5%A4%B4%E9%83%A8%E5%BC%80%E5%A7%8B%E4%BE%9D%E6%AC%A1%E6%89%A7%E8%A1%8C%0A%0A%0A%E6%9C%80%E7%BB%88%E6%B5%81%E7%A8%8B%E5%9B%BE%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%0A%60%60%60%0Agraph%20LR%0Adefer\-\-%3Edeferproc%20%0Adeferproc\-\-%3Enewdefer%0Anewdefer\-\-%3Enew\_defer\_struct%0Auser\_func\-\-%3Enew\_defer\_struct%0Anew\_defer\_struct\-\-%3E%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B\_defer%E9%93%BE%E8%A1%A8%0Adeferreturn\-\-%3E%E5%BD%93%E5%89%8D%E5%8D%8F%E7%A8%8B\_defer%E9%93%BE%E8%A1%A8%0A%60%60%60%0A%0A%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%BB%93%E6%9E%9C%EF%BC%9A%E5%B0%B1%E6%98%AF%E4%B8%80%E4%B8%AA%E4%BF%9D%E5%AD%98%3C%E5%9B%9E%E8%B0%83%E5%87%BD%E6%95%B0%3E%E7%9A%84%E9%93%BE%E8%A1%A8%EF%BC%88%E6%A0%88%EF%BC%89%EF%BC%8C%E8%AF%A5%E9%93%BE%E8%A1%A8%E5%9F%BA%E4%BA%8E%E6%9F%90%E4%B8%AA%E5%8D%8F%E7%A8%8B%E4%B8%8A%E3%80%82%E5%BD%93%E5%87%BD%E6%95%B0%E6%89%A7%E8%A1%8Creturn%E8%A7%A6%E5%8F%91%E8%AF%A5%E9%93%BE%E8%A1%A8%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E3%80%82return%E6%97%B6defer%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F%EF%BC%9A%E5%90%8E%E8%BF%9B%E5%85%88%E5%87%BA%0A%0A%E5%8D%95%E8%AF%B4refer%E7%9A%84%E5%8E%9F%E7%90%86%E5%BE%88%E7%AE%80%E5%8D%95%EF%BC%8C%E4%BD%86%E6%98%AF%E9%85%8D%E5%90%88%E5%85%B6%E5%AE%83%E8%AF%AD%E6%B3%95%E5%8F%8A%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%8C%E5%B0%B1%E9%BA%BB%E7%83%A6%E4%BA%86%E3%80%82%0A%0A%E5%BD%B1%E5%93%8Drefer%E7%9A%84X%E5%9B%A0%E7%B4%A0%EF%BC%9A%0A1.%20return%20%0A2.%20os.exit%20%0A3.%20panic%20%0A4.%20fatal%0A%0Areturn%3A%0A%E5%8E%9F%E7%90%86%EF%BC%9A%0A1.%20%E6%98%AF%E5%90%A6%E6%9C%89%E8%BF%94%E5%9B%9E%E5%80%BC%0A2.%20%E5%A6%82%E6%9E%9C%E6%9C%89%E8%BF%94%E5%9B%9E%E5%80%BC%EF%BC%8C%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E5%87%BD%E6%95%B0%E5%B0%BE%E9%83%A8%E5%B7%B2%E7%BB%99%E8%BF%94%E5%9B%9E%E5%80%BC%E5%AE%9A%E4%B9%89%E4%BA%86%E5%8F%98%E9%87%8F%E5%90%8D%0A3.%20%E5%A6%82%E6%9E%9C%E6%9C%AA%E5%AE%9A%E4%B9%89%EF%BC%8C%E6%96%B0%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E4%B8%B4%E6%97%B6%E5%8F%98%E9%87%8Fret%EF%BC%8C%E5%86%8D%E5%B0%86%E8%BF%94%E5%9B%9E%E5%80%BC%E4%BF%9D%E5%AD%98%E4%BA%8Eret%E4%B8%AD%0A4.%20%E6%A3%80%E6%9F%A5%E6%98%AF%E5%90%A6%E6%9C%89defer%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%89%E5%B0%B1%E6%89%A7%E8%A1%8C%0A5.%20%E8%BF%94%E5%9B%9E%E5%88%9A%E5%88%9A%E4%BF%9D%E5%AD%98%E7%9A%84%E4%B8%B4%E6%97%B6%E5%8F%98%E9%87%8F%0A%0A%E6%88%91%E4%BB%AC%E5%8F%91%E7%8E%B0defer%E5%B1%85%E7%84%B6%E6%98%AF%E6%8F%92%E5%9C%A8return%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E4%B8%AD%E7%9A%84%E4%B8%80%E6%AD%A5%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E8%AF%B4%EF%BC%9Areturn%E6%98%AF%E9%9D%9E%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C%E3%80%82%E8%80%8C%E5%A6%82%E6%9E%9C%E6%88%91%E4%BB%AC%E5%AE%9A%E4%B9%89%E4%BA%86%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9A%84%E5%8F%98%E9%87%8F%E5%90%8D%EF%BC%8C%E4%B8%94%E6%9C%89defer%EF%BC%8C%E4%B8%94%E5%9C%A8defer%E4%B8%AD%E6%9B%B4%E6%94%B9%E4%BA%86%E8%BF%94%E5%9B%9E%E5%80%BC%EF%BC%8C%E9%82%A3%E4%B9%88%E5%B0%B1%E6%8C%BA%E5%8D%B1%E9%99%A9%E3%80%82%0A%0Adefer%E5%9C%A8%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%E4%B8%AD%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%AE%9A%E4%B9%89N%E4%B8%AA%E3%80%82%E5%8F%AF%E4%BB%A5%E5%9C%A8%E4%BB%BB%E6%84%8F%E4%BD%8D%E7%BD%AE%E5%AE%9A%E4%B9%89%E3%80%82%0A%0A%0A%0Aos.Exit%3A%E8%BF%99%E6%98%AF%E4%B8%AA%E6%97%A0%E6%95%8C%E7%9A%84%E4%B8%9C%E8%A5%BF%EF%BC%8C%E8%A7%A6%E5%8F%91%E5%8D%B3%E7%AB%8B%E5%88%BB%E5%81%9C%E6%AD%A2%E5%BF%BD%E7%95%A5%E4%B8%80%E5%88%87%EF%BC%8C%E5%8C%85%E6%8B%ACdefer%0Apanic%E4%BC%9A%E8%A7%A6%E5%8F%91defer%EF%BC%8C%E4%B8%8D%E7%84%B6%E6%B2%A1%E6%B3%95recover%E4%BA%86%0A%0A%0A%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9%EF%BC%9A%0A1.%20%E5%8F%98%E9%87%8F%E4%BD%9C%E7%94%A8%E5%9F%9F%7C%E5%BD%B1%E5%93%8D%E4%BA%86%E8%BF%94%E5%9B%9E%E5%80%BC%20%20%0A%20%20%20%20%E5%9C%B0%E5%9D%80%7C%E5%BC%95%E7%94%A8%0A%20%20%20%20%E8%BF%94%E5%9B%9E%E5%80%BC%E4%B8%BA%E5%87%BD%E6%95%B0%E8%87%AA%E5%B8%A6%E7%9A%84%E5%8F%98%E9%87%8F%E5%90%8D%0A%20%20%20%20%E5%A4%9Adefer%E5%87%BD%E6%95%B0%E5%90%8C%E6%97%B6%E4%BF%AE%E6%94%B9%E7%88%B6%E7%BA%A7%E5%8F%98%E9%87%8F%0A2.%20%E5%8F%97%E5%85%B6%E5%AE%83%E5%9B%A0%E7%B4%A0\(exit%7Cfatalreturn\)%E5%BD%B1%E5%93%8D%EF%BC%8C%E8%A6%81%E7%9F%A5%E9%81%93%E6%98%AF%E5%90%A6%E8%83%BD%E6%AD%A3%E7%A1%AE%E6%89%A7%E8%A1%8C%E4%BA%86defer%0A3.%20defer%E6%95%B4%E4%BD%93%E9%93%BE%E6%98%AF%E6%8C%82%E5%9C%A8%E4%B8%80%E4%B8%AA%E5%8D%8F%E7%A8%8B%E4%B8%AD%E7%9A%84%0A4.%20defer%E5%87%BD%E6%95%B0%E9%87%8C%EF%BC%9A%E8%BF%98%E5%8F%AF%E4%BB%A5%E5%86%8D%E7%94%A8%E5%8D%8F%E7%A8%8B%EF%BC%8C%E4%BD%86%E8%BF%98%E6%98%AF%E5%B0%BD%E9%87%8F%E4%B8%8D%E7%94%A8%0A%0A%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF%EF%BC%9A%0A1.%20lock%2Funlock%0A2.%20openFD%2FcloseFD%0A3.%20%E5%8F%91%E7%94%9F%E4%BA%86%E8%BF%90%E8%A1%8C%E6%97%B6%E5%BC%82%E5%B8%B8%EF%BC%8C%E5%AE%B9%E9%94%99%E5%A4%84%E7%90%86%0A%0A%0A%0Alog.fatal%0A\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-%0A1.%20%E8%BE%93%E5%87%BA%E5%86%85%E5%AE%B9%0A2.%20%E7%AB%8B%E5%88%BB%E5%81%9C%E6%AD%A2%0A3.%20%E4%B8%8D%E6%89%A7%E8%A1%8Cdefer%0A%0A%0Alog.panic%20%0A\-\-\-\-\-\-\-\-\-\-\-\-%0A%0A%0A
