
# LUA

2.6 之后  redis 开始支持的，算是对 redis 的补充吧。
redis 设计 之初就是：单条指令执行，既然事务也一样是单条处理。如果加入更多的复杂的处理，太复杂，性能、维护成本都是问题。另外，C语言是静态的，加入lua  可以做些动态的扩充。总之就是：LUA 对 redis 的扩展及功能是非常好的。


# 高级用法

## HyperLogLog

基数统计：一个集合内，把重复的多个元素计为 1，非重复的也计为 1，最终统计出总数

常规的统计算法肯定性能不行，得用：HyperLogLog 算法

HyperLogLog：是一个算法 ，并不是 redis 独有

redis 随着使用的人数过多，也开始加入了一些新的算法。主要：日常使用 redis 除了缓存 也就是统计了，加点常用的算法 也挺好。

其实，如果自身语言支持 或 干脆下个开源库，redis 这个就有点多余。个人觉得，用处如下：

1. 有些程序员就不会写这个算法，也下不到这个算法，或者就是懒
2. 本机的计算量有限，交给 3 方机器来计算，省点资源
3. 它的算法对性能控制还是比较好的

具说，12KB 内存能计算 2^64 个数字



### HyperLogLog 算法

从 loglog 算法 派生而来，用于确定非常大的集合的基数，而不需要俱其所有值。
是个概率的，并非精准统计。误差率：0.81%。

如：统计多少个用户访问了网站，正常用个简单的 set 即可，随着时间增长，UID 越来越多，计算量就有点大了。

算法有几个公式，看不下去，看不懂，有兴趣的自己百度吧

## 发布订阅

publisher channel subscriber

发布一条消息:

> publish channel:1 hi

订阅一个通道，用于接收消息

> subscribe channel:1

channel 支持 通配符：
如：有 2 个管道叫 order order_pay

如果监听的是: order\* ，那这两个 channel 发来的消息，都会接收到

缺点：

1. 持久化
2. 没有 UI 管理
3. 没有 channel 管理
4. 安全 acl
5. 流控
6. ack 确认机制
7. ......

优点：

1. 超级简单，没有任何配置，直接开干

总之，有点小儿科，小项目玩玩儿还行，大点的项目不可能用它来做 订阅发布，像：

1. 专门的队列系统 rabitmq rocketmq kafka
2. 专门的分布式系统 etcd/consul/zookeeper

## Geospatial

主要是计算经纬度的，其它文章有讲解，这里不做讲解了。

redis 原生支持 经纬度计算，省事儿了，避免自己写算法 了

# Stream

队列

redis 自带的 发布/订阅，缺点是：无法持久化，没有异处理等，且功能较少
>基于 list / sort set 也可以自己写队列，缺点是：得自己写代码，有些功能还实现不了

Stream 算是一个相对小型的队列功能。

Stream 内部使用的  Stream 结构，是一个链表，将所有的消息都串起来。
每条消息又是一个  hashTable ，也就是：一个消息，可以包含若干个K-V 值

1. 消费者/消费者组
2. 可消费历史消息，提供游标
3. 提供阻塞读/非阻塞读
4. ack 确认机制
5. 每个消息都有一个唯一的ID和对应的内容
6. 消息是持久化的

last_delivered_id：记录一个消费者组的，消息进度
pending_ids:记录每个消费者，未确认的消息数

基本上，小公司的队列需求都能满足，主要是比较简单，省事儿。
但是，大公司不行，redis 连个像样的 UI 管理都没有，更何谈安全稳定