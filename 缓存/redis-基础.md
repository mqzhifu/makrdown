
# 概览

多线程，主线程 + 辅助线程

主线程（6.0以前）：网络IO 、指令IO。
>我猜：redis 的 瓶颈是在内存管理上，而不是CPU。在网络IO与指令IO上，单线程应该是够。如果加上多线程，肯定得上锁机制、线程安全等等，太复杂。

辅助线程：持久化、异步删除、主从复制等

网络模式：SELECT/Epoll

6.0之后，主线程 换成了多线程：
- 把网络IO性能提上来。
- 指令阻塞（删除某个大KEY的时候）


所有的客户端过来的请示，最终都会统一到一个地方去执行，因为它是单线程。

# redis 守护进程-定时器

定时器：在 select timeout 时间超时之前，去执行定时器
定时器会取一个 小根堆里 取里面的要执行的函数

更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等
清理数据库中的过期键值对
对不合理的数据库进行大小调整
关闭和清理连接失效的客户端
尝试进行 AOF 或 RDB 持久化操作
如果服务器是主节点的话，对附属节点进行定期同步
如果处于集群模式的话，对集群进行定期同步和连接测试



# 基础数据类型：

| 名称        | 分类 | 底层结构            |
| ----------- | ---- | ------------------- |
| String      | 基础 | SDS                 |
| List        | 基础 | zipList+quickList   |
| Set         | 基础 | intset hashTable    |
| Sorted Set  | 基础 | zipList + skipTable |
| Hash        | 基础 | zipList + hashTable | 



## string

### C 字符串问题
C语言中是使用 char */chat[]  来存储 string，这里有3个问题：

1. 如果计算 string 长度是  O(n)
2. 添加操作，可能会对内存多次重新分配
>字符串拼接可能有溢出风险，需要重新分配 。字符串缩减可能有泄露风险，，需要重新分配
3. 二进制安全？
>C没有真正的字符串，但有字符串概念，在字符串末尾加上：\0
>数据 + \0 = 字符串，这里有个问题，一但 数据中包含 \0，那 len 函数就会错误，也就是数据不安全
>字符串常量：数据不可更改。字符串数组：可任意更改
>不管是 指针 还是 字符数组，最终都会在末尾加上：\0
>总结：C语言的字符串并不单独计算长度，且数据与分隔符混用了

所以，redis 又封装了一个新的结构体 sds ，在日常操作字符串就都使用这个结构体，不直接操作 char *

### sds

Simple Dynamic String

```c
typedef char *sds;
struct sdshdr {
    // buf 已占用长度
    int len;
    // buf 剩余可用长度
    int free;
    // 实际保存字符串数据的地方
    char buf[];
};
```

解决的问题：
1. 加了个索引项，当需要计算长度时，直接读取该项值   O(1)
2. 加了个剩余长度，申请内存的时候，多申请一小块，追加时，避免内存重新分配(动态扩容)
3. 明文了已用未用空间大小，在字符串操作的时候，避免了 溢出
4. 明文了已使用长度，即使数据中包含 \0 依然可以正确使用
6. 真正存数据并不是以字符，而是字节为单位存储。这样就兼容了二进制

未使用：空间的动态分配：
value < 1mb ，申请  value * 2
value > 1mb , 申请  1m

这就是空间换时间，会有一定的内存浪费


3.5 之后 redis 对 sds 继续优化，上面的sds  结构体，变成了5个：
sdshdr5 sdshdr8 sdshdr16 sdshdr32 sdshdr64

```c
struct __attribute__ ((__packed__)) sdshdr64 {  
	uint64_t len; //已使用
	uint64_t alloc; // 总共可用的字符空间大小，应该是实际buf的大小减1 (因为c字符串末尾必须是 \0, 不计算在内)。
	unsigned char flags; // 标志位，主要是识别这是sdshdr几，目前只用了3位，还有5位空余
	char buf[];  // 实际存储字符串的地方 其实就是 C 原生字符串+部分空余空间
};
```



大体上：
- 整形单独存，基本上3~4个字节妥妥够
- 小字符串，单独存
- 大字符串，也单独存

Sdshdr5: 存储字符串长度区间为[0, 1<<5)之间, 没有Len和Alloc字段,只有一个Flags字段,整个sdshdr5占用1Byte
Sdshdr8: 存储字符串长度区间为[1<<5, 1<<8)之间, Len和Alloc字段都占用1Byte, 整个sdshdr8占用3Bytes
Sdshdr16:存储字符串长度区间为[1<<8, 1<<16)之间, Len和Alloc字段都占用2Bytes, 整个sdshdr16占用5Bytes
Sdshdr32:存储字符串长度区间为[1<<16, 1<<32)之间, Len和Alloc字段都占用4Bytes, 整个sdshdr32占用9Bytes
Sdshdr64:存储字符串长度区间为[1<<32, 1<<64]之间, Len和Alloc字段都占用8Bytes, 整个sdshdr64占用17Bytes



![[redis_sds_struct.png]]


### BitMap

使用 sds 存储

有时候场景可以使用上，如：
1. 统计一个用户，一年的登陆次数
2. 统计用户的签到次数
3. ......

主要是日常统计中，非黑即白的情况，bit 正好就是两个数0 和 1 

redis 的sds 是数据安全，支持二进制的，间接就是原生 支持 bit 操作的。


## list

链表结构 ziplist quicklist

### ziplist

看名字叫压缩列表，但它是：对内存的优化，虽然外看是链表。但元素内并不存 next prev ，它实际是个 char 类型的数组

ziplist 内部结构：

```
area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
                                       ^                          ^        ^
address                                |                          |        |
                                ZIPLIST_ENTRY_HEAD                |   ZIPLIST_ENTRY_END
                                                                  |
                                                         ZIPLIST_ENTRY_TAIL
```

- zlbytes  压缩列表所占用的字节（包括本身占4个字节），当重新分配内存的时候使用，不需要遍历整个列表来计算内存大小。
- zltail ：表中最后一项（entry）在ziplist中的偏移字节数。可以很方便地找到最后一项（不用遍历整个ziplist），可以在ziplist尾端快速地执行push或pop操作。
- zllen：压缩列表包含的节点(entry)个数
- entry:表示真正存放数据的数据项，长度不定。一个数据项（entry）也有它自己的内部结构。
- zlend: ziplist最后1个字节，值固定等于255，其是一个结束标记。


entry 内部结构：

```
area        |<------------------- entry -------------------->|

            +------------------+----------+--------+---------+
component   | pre_entry_length | encoding | length | content |
            +------------------+----------+--------+---------+
```
prevlen：前一项的长度。方便快速找到前一个元素地址，如果当前元素地址是x，(x-prelen)则是前一个元素的地址
encoding：当前项长度信息的编码结果。比较复杂，稍后介绍
data：当前项的实际存储数据

它的核心就是：把数组改成链表来使用


数组的优点：
1. 连续的内存空间，不会有内存碎片。
2. 带有下标，可以快速检索

数组的缺点：
1. 表头 表尾 可以快速检查，但是表的中部数据检查起来很麻烦
2. 插入/更新，最快的情况，可能整个链表都要重新进行内存分配 

个人总结：ziplist 可以节省内存，提高一定的检查效率。不适合大数据，一但发现内存重新分配 或 查询中间数据的时候就吃力了。

###  quickList

具说3.2版本以前是 linkedlist ，比较典型的双向链表，其中内容存储项：value 指向 SDS。缺点就是：遍历慢，next prev 吃内存。


quickList 更像是对 ziplist 的索引（类似：2-3树/基排序/跳表的感觉）。 其内部的元素都是指向 一个  ziplist 结构

```c
typedef struct quicklist {
    //指向头部(最左边)quicklist节点的指针
    quicklistNode *head;
    //指向尾部(最右边)quicklist节点的指针
    quicklistNode *tail;
    //ziplist中的entry节点计数器
    unsigned long count;        /* total count of all entries in all ziplists */
    //quicklist的quicklistNode节点计数器
    unsigned int len;           /* number of quicklistNodes */
    //保存ziplist的大小，配置文件设定，占16bits
    int fill : 16;              /* fill factor for individual nodes */
    //保存压缩程度值，配置文件设定，占16bits，0表示不压缩
    unsigned int compress : 16; /* depth of end nodes not to compress;0=off */
} quicklist;
```

```c
typedef struct quicklistNode {
    struct quicklistNode *prev;     //前驱节点指针
    struct quicklistNode *next;     //后继节点指针

    //不设置压缩数据参数recompress时指向一个ziplist结构
    //设置压缩数据参数recompress指向quicklistLZF结构
    unsigned char *zl;
    //压缩列表ziplist的总长度
    unsigned int sz;                  /* ziplist size in bytes */
    //ziplist中包的节点数，占16 bits长度
    unsigned int count : 16;          /* count of items in ziplist */
    //表示是否采用了LZF压缩算法压缩quicklist节点，1表示压缩过，2表示没压缩，占2 bits长度
    unsigned int encoding : 2;        /* RAW==1 or LZF==2 */
    //表示一个quicklistNode节点是否采用ziplist结构保存数据，2表示压缩了，1表示没压缩，默认是2，占2bits长度
    unsigned int container : 2;       /* NONE==1 or ZIPLIST==2 */
    //标记quicklist节点的ziplist之前是否被解压缩过，占1bit长度
    //如果recompress为1，则等待被再次压缩
    unsigned int recompress : 1; /* was this node previous compressed? */
    //测试时使用
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    //额外扩展位，占10bits长度
    unsigned int extra : 10; /* more bits to steal for future usage */
} quicklistNode;
```


quicklist -> quicklistNode ->ziplist

![[redis-QuickList.png]]

## hash

### hashTable

```c
typedef struct dictht {
    dictEntry **table;             // 哈希表数组，指向  dictEntry 
    unsigned long size;            // 哈希表数组的大小
    unsigned long sizemask;        // 用于映射位置的掩码，值永远等于(size-1)
    unsigned long used;            // 哈希表已有节点的数量
} dictht;

```

```c
typedef struct dictEntry {
    void *key;                  // 键
    union {                     // 值
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;     // 指向下一个哈希表节点，形成单向链表
} dictEntry;

```


比较简单，就是：数组+链表

数组：通过一个hash函数计算出 数组的下标，O(1)，满足hash的查询复杂度
链表：key 冲突的时候使用

如果数据量过多，数组过小，冲突就会加大，它还引入了另外一个结构：
```c
typedef struct dict { 
    dictType *type; 
    void *privdata; 
    dictht ht[2]; 
    int reshaidx; 
} dict; 
```

大体上的思路：就是再创建一个新的数组，间接算是扩容吧，再新创建一个 dict 结构体，管理这两个 数组

当  hashTable 过大时，会改成 ziplist 存储：
- 数据长度小于64
- 列表长度小于512


## set

数据量大且是字符类型： hashTable
数据量小且是数字类型： intSet

### inset
```c
typedef struct intset {  
	uint32_t encoding;  
	uint32_t length;  
	int8_t contents[];  
} intset;
```

比较简单，就是一个数组保存数据。不过，这只是 redis  其中的一个数据结构，小数量量还是不错的。


## zset

小数据且整形依然可以使用 setint

```c
typedef struct zset{
     //跳跃表
     zskiplist *zsl;
     //字典
     dict *dice;
} zset;
```

跳表：就是在一个有序的链表基础之上，再建立 1 级索引，以二分法的方式建立。这样搜索更快。

为什么不单独使用字典？
字典是无序的，查找快，但是排序不行。

为什么不单独使用跳表？
可以范围查找 ，但是具体 找某个值会很慢


```cpp
typedef struct zskiplist{
     //表头节点和表尾节点
     structz skiplistNode *header, *tail;
     //表中节点的数量
     unsigned long length;
     //表中层数最大的节点的层数
     int level;

}zskiplist;
```

```cpp
typedef struct zskiplistNode {
     //层
     struct zskiplistLevel{
    //前进指针
    struct zskiplistNode *forward;
    //跨度
	unsigned int span;
	}level[];
		//后退指针
		struct zskiplistNode *backward;
		//分值
		double score;
		//成员对象
		robj *obj;
} zskiplistNode
```

# 高级用法
## HyperLogLog

基数统计：一个集合内，把重复的多个元素计为1，非重复的也计为1，最终统计出总数

常规的统计算法肯定性能不行，得用：HyperLogLog 算法 

HyperLogLog：是一个算法 ，并不是 redis 独有

redis 随着使用的人数过多，也开始加入了一些新的算法。主要：日常使用 redis 除了缓存 也就是统计了，加点常用的算法 也挺好。

其实，如果自身语言支持 或 干脆下个开源库，redis 这个就有点多余。个人觉得，用处如下：
1. 有些程序员就不会写这个算法，也下不到这个算法，或者就是懒
2. 本机的计算量有限，交给3方机器来计算，省点资源
3. 它的算法对性能控制还是比较好的

具说，12KB内存能计算 2^64 个数字

### HyperLogLog 算法

从loglog 算法 派生而来，用于确定非常大的集合的基数，而不需要俱其所有值。
是个概率的，并非精准统计。误差率：0.81%。

如：统计多少个用户访问了网站，正常用个简单的set 即可，随着时间增长，UID越来越多，计算量就有点大了。

算法有几个公式，看不下去，看不懂，有兴趣的自己百度吧


## 发布订阅

publisher channel subscriber

发布一条消息:
>publish channel:1 hi

订阅一个通道，用于接收消息
>subscribe channel:1

 channel 支持 通配符：
 如：有2个管道叫    order   order_pay

如果监听的是: order* ，那这两个 channel 发来的消息，都会接收到

缺点：
1. 持久化
2. 没有UI管理
3. 没有 channel 管理
4. 安全  acl
5. 流控
6. ack 确认机制 
7. ......

优点：
1. 超级简单，没有任何配置，直接开干


总之，有点小儿科，小项目玩玩儿还行，大点的项目不可能用它来做 订阅发布，像：
1. 专门的队列系统  rabitmq rocketmq kafka 
2. 专门的分布式系统   etcd/consul/zookeeper



## Geospatial

# Stream

# LUA



# 共用策略

1、如果容器不存在，即创建。
2、如果容里没有值，即删除。

注：如果给一个 key 设置了过期时间，然后你又 set 这个 key ,那么之前的过期时间就消失

# key 的存储结构：字典

一维数组+链表结构

# 公共的一些指令

dbsize
monitor
info
info replication
role
KEYS

setex key second：设置一个 XX 秒失效的 KEY
exists：判断 KEY 是否存在
dump key：将 value 序列化，目前不确定有啥实际用处
EXPIRE key second：设置一个 KEY 在多少秒后失效
EXPIREAT key timestamp：设置一个 KEY 在 UNIX 时间内失效
KEYS pattern ：正则查找 遍历所有 KEY
MOVE key db ：将一个 KEY 从 DB\-A 移到 DB\-B
PERSIST key ：删除 KEY 的失效时间，变成永远 KEY
TTL key ：返回 KEY 的剩余失效时间
RENAME key newkey ：重命名一个 KEY
TYPE key ：返回当前 KEY 的数据类型
del key：删除 KEY

setnx：仅当 KEY 没有被设置，才会生效
mget：获取多个 KEY

GETRANGE key start end ：获取 VALUE 指定位置的值
STRLEN key：返回一个 KEY 的长度
INCR key：累加 1，KEYS
INCRBY key increment：累加 X
DECR key：累减 1
APPEND key value ：如果 KEY 是 STRING，在末尾追加



# 持久化

RDB：将某一时刻的内存数据，PUSH 到 DISk，先 PUSH 到临时文件，都 OK 了，再替换上一次文件。同时，是新开一个进程做持久化。

AOF：英文是 Append Only File，即只允许追加不允许改写的文件。AOF 方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍，就这么简单。AOF 持久化策略是每秒钟 fsync 一次

性能测试

redis\-benchmark \[option\] \[option value\]


# 事务

multi：开启事务
exec ：执行事务的指令队列
discard ：结束事务，并清除队列指令
watch：监控某个值是否发生了改变（只能在事务之前使用），它会影响后面的事务，在执行事务期间发现值变了，则自动失败
unwatch：取消监控

执行过程：
1. 开启事务
2. 发送若干指令
3. redis 把指令暂存队列中
4. 如果是 multi exec discard watch 会立刻执行指令队列
5. 如果是 之前有 watch ，判断值是否改变，如果改变了，终止事务
6. 即使指令错误，跳过错误指令 继续向下执行正确的指令（没有圆润）

事务不可以嵌套
multi 之后再发送 multi 会出错，但不会影响一个事务失败
multi 之后再发送  watch 会出错，但不会影响一个事务失败


开启事务后，任何操作都是 返回：queue，告诉 C 端，该指令已接收到，等待最后批量执行
如果期间有脚本是错误的（语法 ），会提示
最后 exec 执行，但如果期间有脚本 出错，后面的指令依然还是会执行。

###  watch 原理

有一个链表：watched_keys。存储被监视的key ，key 对应客户端ID，一个KEY同时可能被多信 client 监控。
在执行:set del lpush 这些指令时，执行结束后。会检查该 key 是否有监听的客户端，如果有：将该  client conn 标识出，此连接有监控的值被修改了。此时如果 client 再发送事务，直接拒绝，也算是间接实现了 回滚吧。


discard，只是是结束了当前事务脚本
所以，redis 事务，只是保证了一组指令执行，并不支持回滚~

加入 watch，实现‘类似’真实的事务：

session 1：
set kkk 1

session 2
watch kkk

multi
get kkk
set kkk 2

此时

session 1

set kkk 3

session 2

exec

nil

因为返回的是 nil 证明事务失败

问题：幻读

session 1

get kkk

.....

doing

\(此时 session 提交了事务修改了 kkk=2\)

get kkk

eval

在 redis 中，执行 指令，为了一致性。

可以加入一些逻辑判断 ，如：if

事务跟 LUA 有点像，但没有回滚机制

过期策略

正常，一个 KEY 如果过期了，REDIS 是不知道的，只有，有一个 SESSION 连接，并操作 key，redis 才会去检查

优点：没必要在 key 是否过期上浪费时间，减少 CPU 占用

缺点：浪费内存

LRU：会定期抽样，清理掉一些过期的 KEY

