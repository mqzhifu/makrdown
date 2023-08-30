# 基础

redis内核：

# string

申请一小块内存空间存字符串，实际上，要比你正常set一个值要大一些，可以动态扩容。主要是，一但这个值修改时，比如：A\+BCD ，不用再申请内存了

# list

链表结构 ziplist quicklist

# hash

数组\+链表

# set

也是一个hash结构，value 为空

# zset

跳表\-链表

跳表：就是在一个有序的链表基础之上，再建立 1级索引，以二分法的方式建立。这样搜索更快。

再具体点就是：单个 链表节点，原本可能只存VALUE，现在加个叫 L1指针

以上四种：共用如下策略

1、如果容器不存在，即创建。

2、如果容里没有值，即删除。

注：如果给一个key 设置了过期时间，然后你又set 这个key ,那么之前的过期时间就消失

所有key 的存储结构：字典

一维数组\+链表结构

单线程

网络模式：SELECT

定时器：在select timeout 时间超时之前，去执行定时器

定时器会取一个 小根堆里 取里面的要执行的函数

redis守护进程\-定时器

更新服务器的各类统计信息，比如时间、内存占用、数据库占用情况等

清理数据库中的过期键值对

对不合理的数据库进行大小调整

关闭和清理连接失效的客户端

尝试进行 AOF 或 RDB 持久化操作

如果服务器是主节点的话，对附属节点进行定期同步

如果处于集群模式的话，对集群进行定期同步和连接测试

dbsize

monitor

info

info replication

role

KEYS

exists：判断KEY是否存在

dump key：将 value 序列化，目前不确定有啥实际用处

EXPIRE key second：设置一个KEY 在多少秒后失效

EXPIREAT key timestamp：设置一个KEY在UNIX时间内失效

KEYS pattern ：正则查找 遍历所有 KEY

MOVE key db ：将一个KEY 从DB\-A移到DB\-B

PERSIST key ：删除KEY的失效时间，变成永远KEY

TTL key ：返回KEY 的剩余失效时间

RENAME key newkey ：重命名一个KEY

TYPE key ：返回当前KEY的数据类型

del key：删除KEY

5大结构：String、Hash、List、Set、Sorted Set

String

最基本的结构，跟MEMCACHE，有些像~

get key

set key

setex key second：设置一个XX秒失效的KEY

setnx：仅当KEY 没有被设置，才会生效

mget：获取多个KEY

GETRANGE key start end ：获取VALUE 指定位置的值

STRLEN key：返回一个KEY的长度

INCR key：累加1，KEYS

INCRBY key increment：累加X

DECR key：累减1

APPEND key value ：如果KEY是STRING，在末尾追加

hash

hset

hmset

HSETNX key field value

HDEL key field1 \[field2\]

Hget key field1 \[field2\]

HMGET key field1 \[field2\]

HEXISTS key field

hgetall key

HKEYS key

HLEN key

HVALS key

list 队列

lpop

rpop

rpush

lpush

lrange

set

集合，元素唯一，无序

sorted set

有序集合，元素唯一，有序

HyperLogLog

基数统计

发布订阅

持久化

RDB：将某一时刻的内存数据，PUSH到DISk，先PUSH到临时文件，都OK了，再替换上一次文件。同时，是新开一个进程做持久化。

AOF：英文是Append Only File，即只允许追加不允许改写的文件。AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍，就这么简单。AOF持久化策略是每秒钟fsync一次

性能测试

redis\-benchmark \[option\] \[option value\]

