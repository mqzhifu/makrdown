# 概览


hash：散列、打碎、哈希\(音译\)，因为是个外文，也没精准的翻译
大概的意思是：给定一个值(数字|字符组合，可以有多个)，根据一个算法，将此值转换成一个新值（不定长）


它的使用场景，大体看有2个大方向：

1. 数据分片|分布式，即把一个大的集合划分\(hash函数\)成若干小的子集合，当有请求过来根据hash函数，找到特定的那个小集合中
    数据均衡，一个值，能均衡的将此值分发到若干的子集合中去，跟上面类似
2. 将字符串，转换\(hash函数\)成一个新的字符串

由两个大方向，推导，细分领域：

|名称       |描述                                                               |算法|
|-----------|-------------------------------------------------------------------|----|
|加密/签名  |传输文件时，对文件内容进行hash后，得到一个，简短签名               |md5 |
|压缩       |对一堆字符串，使用hash函数后，得到一个定长简短的字符串             |md5 |
|负载均衡   |根据特定值\(ip\)，使用hash函数后，决定该请求指向哪台机器IP         |求余|
|数据分片/块|根据特定值\(如数值的个位数\) ，使用hash函数后，找到真实数据存储的块|求余|
|分布式     |跟上面类似                                                         |求余|

# 签名

1. A有个文件要传给B
2. 但害怕在传输过程中该文件内容被黑客截获篡改
3. A把文件里的所有字符用 hash 函数计算后，得到一个比较小的新的字符串
4. 在传输的时候：新的字符+文件一并传过去
5. B拿到文件和新字符串后，也重复A的算法，最后比对两个定长串是否一致。

>验证文件传输过程中，是否被篡改

# 加密

1. A的密码存在数据库中
2. 如果此时数据库被人盗取，所有的密码也都被盗取
3. 此时，如果给A的密码，通过 hash 函数生成新的字符串，黑客拿到手上也没用。
>因为，这个原密码只有用户只知道，hash 函数是固定的，只要生成密码就会被HASH转换成新的密码，连程序员都不知道此密码

# 压缩

1. 现有100W个URL地址，长度各异。
2. 要求尽快的统计出每个URL的次数
>如果一个一个URL的比较，就是一个字符一个字符的比较，效率有点低

3. 用 hash 函数 把这种长度各异的URL地址，统一转化成固定长度且字符量较小的串
4. 因为新的字符串长度统一、更短，统计起来效率更高

>毕竟是大长串变成小串，可能会有不同URL但新串一样的情况，但因为这种统计并不要求绝对准确

# 负载均衡

1. 现在有3台服务器，给每台服务器设置ID：1 2 3 
2. 根据请求的IP，最后一位数值，与3求余
3. 决定该请求最终真实的处理的服务器

# 大数据分块

1. 当前数据库里存储了100W用户
2. 每次查询时，要扫描整张表\(未建索引\)，
3. 这时将单张表，拆分成10张表，根据用户ID最后一位，与10求余。

# 分布式

跟上面类似

# DEMO，数据分片/均衡

> 给定一个集合，按照一定算法，将这集合里的元素值，分成若干个子集合，并把值映射进去。 
> 这里就拿数组做实现吧，略形象些

申请了一个大小为：100个元素的数组，

> arr = new array\(100\)

划分子集合：数组就是一个大集合，将其分成10等份

> 即：1\-10，11\-20....91\-100

假设现在过来100个数\(随意的数字\)，要如何平均分散插入到这个数组中呢？

> number % 10 = mod

要添加的数字，跟10求余，其结果肯定 是：0~9，根据这个余数去找对应的子集合，然后顺序存储即可

#### 这个hash函数有什么用？

这里的DEMO比较简单，如果把这10个子集合，看成若干台机器，把100个数再放大到无限大时，即可实现了：多台机器存储同一业务的功能，即：分布式，同时也有一定的负载均衡的功能。



#  一致性哈希


上面，数组大小是100，即：只能装下100个数字，如果此时要插入 1000 个元素咋办？



1. 给定的100个数值是递增的，那求余后，基本上是平均散落到这10个子集合中，这是理想状态，如果不是递增的话，比如：个位数全是1，那么，还是存不进去...

> 这个就是数据的散列度与hash函数的选择了

1. hash后，重复值如何处理？

> 这种情况叫：碰撞，使用拉链法

## 数据散列度

上面问题2中：发现，一但数据大部分以1结尾，那么，只能就一个数组有值，其它子集均为空，且无法实现分片功能。又或者，所有数据都是以1和2结尾的，就变成了只有2个子集合有值，而预期是希望：10个数组最好都有值

以上就引出了两个问题：

1. 数据集合中的值，是否适合用hash结构？
2. hash函数是否适合 数据集合中的值？

解答1：数值，什么样的集合适合？

1. 顺序递增，如：数据库的自增ID
2. 随机数
3. 可预知集合，如：集合中的数字是固定的，且轻易不会变的

解答2：hash函数的选择，要先查看下集合中的数值情况，再决定使用什么样的hash，并且有些数据值可能根本就不适合用hash结构

# 解决碰撞/拉链法

假设，当前有两个值：aa bb ，正常经过hash函数后，得出 1 和 2 ，这两个数代表的是数组的下标，存储的时候，即：arr\[1\] = aa , arr\[2\] = bb ，这时加了个C，经过hash函数后：2，出现了碰撞，那么该如何存储？这个时候给arr\[2\] 不存真正的值，而是存一个地址，地址指向一个链表\(也可以用数组\)，所有重复的值都在这外链表中。这种方法的缺点是：要增加额外的空间

## 开放地址法

线性探测

还是上面的情况，计算出哈希值=2，但是arr\[2\]，已经有值了，这个时候探测，去查找arr\[3\]，有没有值，如果没有，就存在这个位置，查找 的时候，肯定是先拿arr\[2\]里的值，发现不对，线性探测，也就是找arr\[2\]下面的值。那如果这时候 arr\[3\]要存值咋办？顺延，依次递增吧~ 一但发生这种情况，就等于所有元素再添加的时候都要被顺延，挺可怕的，这种情况叫聚集。另外，原本可以存100个不同元素值的数组，现在变成了只能存99个了....

```
二次探测
    跟上面差不多，只是：当发生碰撞后，向下递增长空位，改成：更大范围查找，如：X+1、X+n^2
    ，只是加大了个步长，还是没解决核心问题

再 hash 法
    如果出现碰撞，就换种hash法，肯定能得到不同的值~
```

> 感觉，以上3种方法，看着一个比一个秀，但都躲不过一个核心问题：数组的容量有限，只要多一个值，就得少存一个，并且会改变原来的 下标

问题2：这个得分成两类来看，如果是上面URL的例子，那就死局没办法，只能找个更好的hash函数，将碰撞降到最低，另外，URL的统计本就是个统计学的东西，就是个大概值就够了，有个1%的误差，其实无所谓的。但如果不是URL，像加密这种，1%的错误率也是挺可怕的，依然无解没太好办法。

如下我找了几个hash函数算法：

|算法名称|描述                                  |适用                                   |
|--------|--------------------------------------|---------------------------------------|
|md5     |太复杂了，不写了...                   |密码、名称、统计URL个数时用的多        |
|time33  |一个固定值 \* 33 \+ 一个字符串\(asci\)|字符串转成int，具说效率最高，散列最好  |
|求余    |X MOD N                               |最简单的函数，但当N发生变化时，问题较大|
| CRC32       |                                      |                                       |
|        |                                      |                                       |

> 算法的具体实现就别深入研究了，看了看MD5太复杂了...知道每种算法适合的领域吧...

优点：插入是O\(1\)，删除O（1），查找O（1）

缺点：

1. 会有碰撞值，一但KEY取不好，就完蛋。另外，还必须得平均分散
2. 它是基于数组的，初始的大小一但建立，后期产生变化有点麻烦
3. 不太好遍历
4. hash函数的选择

这些时候都是最基础的，正常面试会问：

1. hash分片后，如果遍历整个集合？

> 额外建立二级索引

1. 具体某些hash函数的算法实现

> time33 md5

1. hash重复值的处理

> 拉链法、开放寻址

1. hash的散列度预估及hash函数选择

> 这两其实是考核下自己对整体数据的预估，及哪些数据类型适合HASH结构，还有每种hash算法的核心

1. hash如何实现hashTableMap

> 这个应该是最复杂的，最难讲的，而且，不同语言的实现也不太相同，回头我统一找个地方好好写写吧.

1. 如果数据发生变更，原hash函数的散列平衡被打破，如果处理？

> 一致性hash算法
