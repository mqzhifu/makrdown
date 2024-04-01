# 数组/切片

#### 数组

一个容器，在内存是一段连续的空间
>跟其它语言都差不多

创建一个数组：
```go
var a [3]int
a := [3]int{1,2,3}
```

数组都是值传递：函数的参数为数组类型，函数内部改动数组里的值，外部的数组值不变

```go
func main() {
	arrayA := [2]int{100, 200}
	var arrayB [2]int

	arrayB = arrayA

	fmt.Printf("arrayA : %p , %v\n", &arrayA, arrayA)
	fmt.Printf("arrayB : %p , %v\n", &arrayB, arrayB)

	testArray(arrayA)
}

func testArray(x [2]int) {
	fmt.Printf("func Array : %p , %v\n", &x, x)
}
```

>结果是3个不同的地址

从上面看，如果调用一个函数的参数是数组类型，且数组较大，那值复制操作也很大

可以把函数的参数类型改成，地址引用：  
```go
*[2]int
```


[10]int  和   [5]int 

这两在GO里是不同类型的数组....  


#### 切片 Slices

是引用，可任意更改数组大小，换个角度理解就是：可动态添加/删除数组元素，且是地址引用

```go
type slice struct {
	array unsafe.Pointer //指针。指向一个地址(数组首部)
	len   int //数组的长度
	cap   int //数组可容纳的元素
}
```

```go
var []int a      //注：不能设定切片大小，否则就跟数组一样了
a := []int{1,2,3}//注：不能设定切片大小，否则就跟数组一样了
//make( type ,  len ,cap )
a := make(int,3,6)
a := make(int,4)
//从数组中创建切片
arar := [4]{1,2,3,4}
a := [1:3] // 结果：2 3，从第1个(数组从0开始)位置开始截取，到第3个位置结束   [1:3)       
```


正常一个数组肯定是定长的~如果想要变长，就得使用 slices
slices 更像是在传统 数组上又封装了一层

区别是他可以在使用过程中，自动将原数组容量进行-扩容

扩容：
1. 如果新申请的容量，是原容量的2倍，直接扩容
2. 如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍、
3. 如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的 1/4
4. 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量

什么情况会产生扩容：
>append


扩容过程中，可能会产生bug?
 ```
 a := [4]int{1,2,3,4}
 b := a[1,3]
 c = append(1,b)
```

这里的问题是出在 append ，返回的是一个新的数组，还是旧的




#### 总结

感觉切片的模式，是弥补了数组的扩展性，避免了因为定长，而使开发者频繁 的需要手动扩容数组。还可以动态切割数组元素。那么使用的结论就是：数组\+切片配置使用。

# MAP

```go
type hmap struct {
    count     int // 元素的个数
    flags     uint8 // 标记读写状态，主要是做竞态检测，避免并发读写
    B         uint8  // 可以容纳 2 ^ N 个bucket
    noverflow uint16 // 溢出的bucket个数
    hash0     uint32 // hash 因子

    buckets    unsafe.Pointer // 指向数组buckets的指针
    oldbuckets unsafe.Pointer // growing 时保存原buckets的指针
    nevacuate  uintptr        // growing 时已迁移的个数

    extra *mapextra
}

type mapextra struct {
    overflow    *[]*bmap
    oldoverflow *[]*bmap

    nextOverflow *bmap
}

type bmap struct {
     tophash [bucketCnt]uint8
 }
 ​
 //在编译期间会产生新的结构体
 type bmap struct {
     tophash [8]uint8 //存储哈希值的高8位
     data    byte[1]  //key value数据:key/key/key/.../value/value/value...
     overflow *bmap   //溢出bucket的地址
 }
```

hmap 是一个汇总体，
buckets：指针。一个数组，存储 bmap（bucket ）
bmap： 具体数据的存储是，一个 bmap 里存8个KV。

B 是 buckets 数组的长度的对数，也就是说 buckets 数组的长度就是 2^B

flags：防止并发写的，他类似于全局锁。 出现就报pnaic了
同时有2个协程并发写：concurrent map read
读写并发：concurrent map read and map write
>操蛋的点也在这儿

允许并发读，不允许并发写、并发写读

## 大体转换过程

将 KEY 转化成一个数字，映射到数组中。

这个数字，包含两部分内容：

1. 低8位指向 bmap
2. 高8位指向 bmap中的真实的key

翻译下：低8位 hmap中的数组的键值，高8位为数组的bmap中的数组的键值

感觉就是个二级hash结构，一级是汇总数组的结构体，然后指向一个数组，这个数组里的值是hmap\-bucket结构体，而这个bucket结构体里又是一个汇总结构，此结构体里又包含一个数组.

## 流程

1. 创建一个MAP，即会有一个hmap结构体
    hmap结构体进行初始化结构体的元素值
    1. B值，这个是确定有几个bucket = 0
    2. buckets:指针，指向一个数组，这里是新创建一个数组
    3. old\-buckets:指针，指向一个数组，扩容时使用
    4. hash因子
    5. count = 0
    6. flags //处理并发写
    7. 溢出桶的一些信息
2. 此时，有一个元素添加
3. 将该元素的KEY，HASH FUNC \+ has因子 = 转换成一个数字
4. 低8位再转换一个数字，数字范围:0~数组最大长度\-1，即得到数组的key
    准确的说是：低8位与 hmap中的B值做\<与\>运算
5. 此时，创建一个bucket，同时再创建3个数组
    keys values tophash
6. 将真实的KV值，存到这个3个新的数组中
7. 最后，将该bucket放置到第4步中得出K值的数组中
8. 将hmap中的count值\+1

## 查找

1. 将该元素的KEY，HASH FUNC \+ has因子 = 转换成一个数字
2. 根据低8位，与B做与去处，找到该bucket所在数组的索引位置
3. 用高8位循环比对：从该 bucket 结构的元素 tophash 数组中，查找
4. 如果找不到，再去 overflow中找
5. 如果还是找不到，即该KEY不存在

这里的tophash有点意思，不是直接对真的key值比对，而是先找tophash，然后如果找到了，根据索引值，直接去另外两个数组里取值。也算是个取巧吧，对KEY值的搜索优化

## 扩容/链表

触发扩容条件：

1. map中实际存放元素总个数 / bucket \> 6.5
2. overflow bucket 过多
    1. B \<= 15 ，已使用溢出桶个数\>= 2^B
    2. b \> 15 ,已使用溢出桶个数\>= 2^15

溢出扩容：是等量扩容

总个数不够：翻倍扩容

当触发扩容后，会新创建一个数组，且旧数组依然在保存，也就是新/旧两个桶会并行一段时间。如果此时有新的数据过来了，肯定是往新桶里添加。