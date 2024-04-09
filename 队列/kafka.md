# 安装

先是准备下包，编译安装。但，依赖：zk java scala ，连带着就是：jdk gradle 、wrapper，放弃了，直接 docker 吧

docker 下包：

>docker pull wurstmeister/zookeeper
>docker pull wurstmeister/kafka


启动zk:
>docker run -d --name zookeeper -p 2181:2181 wurstmeister/zookeeper

启动kafka:

```
docker run  -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=172.27.198.210:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.27.198.210:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-v /data/kafka:/kafka \
-t wurstmeister/kafka
```

> 注意一下，要把宿主机的 IP 改过来

进入 docker 测试一下

> docker exec \-it xxxx /bin/bash

# 基础操作

#### 创建 topic

> $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper 172.27.198.210:2181 --replication-factor 1 --partitions 2 --topic mykafka

#### 查看 topic

> $KAFKA_HOME/bin/kafka-topics.sh --list --zookeeper 172.27.198.210:2181

#### 发送/生产消息

> $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic mykafka --print-data-log

#### 创建消费者/消费消息

> $KAFKA_HOME/bin/kafka\-console\-consumer.sh \-\-bootstrap\-server 127.0.0.1:9092 \-\-topic mykafka \-\-from\-beginning

#### 查看配置

> cat /opt/kafka/config/server.properties
> log.dirs=/kafka/kafka\-logs\-cfc5eb4cf9e3

#### 查看文件内容

/opt/kafka/bin/kafka-run-class.sh kafka.tools.DumpLogSegments --files  00000000000000000000.log

#### 查看某个主题的详细信息

> $KAFKA_HOME/bin/kafka\-topics.sh \-\-zookeeper 172.17.147.50:2181 \-\-describe \-\-topic mykafka




# PHP 扩展安装

现在比较好的是用一个 composer 扩展：

> https://github.com/longyan/phpkafka/blob/master/doc/producer.md

也可以用:composer require "nmred/kafka\-php"

# KAFKA 基础


linkedin公司开发的，使用scala语言编写

解释：分布式消息队列。

| 术语/名词 | 解释 | 分类 |
| ---- | ---- | ---- |
| broker | 一个节点，一台服务器，是被 zookeeper 管理。 | 服务器 |
| Topic | 一个分类，下面可以有N个 partition | 文件管理 |
| partition | 具体存储消息的容器，一个Topic有N个 partition。可以存储在不同的broker上 | 容器 |
| replication | partition的副本，用于容灾 | 容器 |
| Consumer | 消费者，从 kafka 消息队列中读取消息 | 角色 |
| Consumer Group | 由多个 consumer 组成。 消费者组内每个消费者负责消费不同分区的数据 | 角色 |
| Producer | 生产者，生产消息交投递到 kafka 中 | 角色 |
| Offset | 队列的 offset (消息的offset) , Consumer 消费的 offset(消费到了第几条) | 文件处理 |
| Leader | 每个分区多个副本的“主” | 分区管理 |
| Follower | 每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步 | 分区管理 |

# Topic

更像是一个大的分类，或者一个大的索引，它并不存在消息，而是管理 partition
# partition

真正存储消息的容器

#### 物理结构

- 每一个 partition ，在服务器上都会有一个文件夹：一组 消息 文件和 索引文件。
- 可以把 partition 看成是一个大文件，文件里的内容就是消息/记录。
- 再按照一定  规则（生成时间，文件大小），把大文件打散成若干个小文件 ，也叫：segment
>offset: 0 CreateTime: 1660546405647 keysize: 1 valuesize: 6 sequence: -1 key: 1 payload: kafka
- 索引文件：一列是 offset，一列是 position
>position：这条数据在 Segment 文件中的物理偏移量
>offset：这条数据在这个 Segment 文件中的位置，是这个文件的第几条


其中文件夹名是：主题名\+分区号

.index
.log
.timeindex
leader\-epoch\-checkpoint

也可以到 zk 中查看
zkCli.sh

#### 分区选择

1. 如果指定的 partition，那么直接进入该 partition。
2. 如果没有指定 partition，但是指定了key，使用key的 hash一选择partition。
3. 如果既没有指定 partition，也没有指定key，使用轮询一的方式进入partition。



#### 分析

发消息可以看成是往 这些文件里：追加数据。而读消息先从索引文件中找到哪个 segment 文件，再具体到哪行，直接读取文件即可

Consumer 根据 offset ，更新哪些 消息 被消费

消息删除机制：kafka 是不删除消息的，通过移动 offset 决定哪些消息被消费了。但可以设置过期时间，过期策略，删除消息


#### 数量

kafka 里，一个 partition 最多对应一个 消费者。
如果 partition = 1 ，同时最多只能有一个消费者。如果partition = N  ，那么就有N个消费者，很显然并发会更好。

但，更多的 partition 意味着，维护的成本也更高。


 max( (期望的 吞叶量 / 生产者的 吞吐量)   (期望的 吞叶量/ 消费者的 吞吐量) ) = partition 数量


#### 小结

如果与DBMA做对比：Topic 像是一个数据库，partition 像是其中一张表
>partition 更像是被划分后的表



# Producer

往 broker 指定的 Topic 的指定 partition 中写
>这里的看，broker 是否有集群，broker的数据，leader + flower 等
# Consumer 

从 broker 的指定 Topic 的指定 partition 中读。
>这里的看，broker 是否有集群，broker的数据，leader + flower 等



# Consumer Group


由 consumer  组成一个组
>也可以看成是一个虚拟的组

Consumer Group 也可以通配符式监听队列

启动 consumer 没有指定 group name ，系统会默认给生成一个组名

假设：partition 有3个，而 consumer 只有一个，那么 该  partition 会同时接收3个 partition 的消息
如果  consumer 有3个，那么 partition的3个分区会一对一指向  consumer

一个 consumer 订阅了 topc-1 ， 另外一个 consumer 也订阅了 topic-1 ，且两个consumer 的group  不同，那么给 topic-1 发消息， 两个 consumer 都会收到消息

#### rebalance

触发条件 :
- Consumer Group 添加了 一个 consumer
- Consumer Group 删除了 一个 consumer
- 新添加了 topic ，Consumer Group 是通配符监听

原本的消费体系就会被打破，需要重新组 织  partition 和  consumer 的关系


偏移量提交/消费消息时 ，提交 offset 类型：

- 自动：每N秒，自动提交一次 offset
>一次拉下来一批消息，定时再提交一下，如果提前之前出现错误，就会导致 offset 未更新，会重复消费
- 手动：commitSync 和 commitAsync
	- commitAsync：异步提交，肯定是快，但发出去时，出现错误，会重复消费
	- commitSync：阻塞，最慢，且还会重试。但最稳定



# Broker Controller

实际上就是 zookeeper 的 leader，不同的是 zk 只是选举出一个 leader 就结束了，kafka 是用这个 leader 来做接下来的操作。也就是 zk 提供了基础功能，kafka 扩展了一下。

选举出 ctrl 后：它是一个线程，它会再为每个 partition 再选举出 leader

leader：负责读写，followe 只负责备份。

容灾：当某一个 broker，挂了，zk 那里有 watch 会通知，Controller，会找到受影响的 partition，找出

新 leader。



# replication

partition 分区副本：冗余，容灾

可通过配置文件，设置副本因子

假设副本因子设置为2，topic 有3个分区，那就是 3+3=6 个分区
>副本因子 * 分区数 = 一个  topic 的总分区数  

每个副本的数据与源 partition 99.9%是一致的。
>这里牵扯到 raft   一致性计算的情况，极低的情况可能会丢数据

所有的副本，都不能在当前节点复制
> 备份肯定要存储于其它机器上，不然备份失去意义了呀

也就是说：每个 broker 上保存着另外 broker 的 每个 topic 的 若干  replication

broken 必须得大于1 ，不然副本这东西没作用

副本的数量不能超过broker的数量


具体，副本如何同步 partition 数据，得参考 leader 同步机制了


# 选举机制

- controller
- 分区 leader
- 消费者


### controller

管理 kafka 集群

每个 broker  启动后，会支 zk 中 set 一个临时值，如果能 set 成功就是 controller。
set 失败的  broker 会放弃竞选 controller，同时 watch 该值，一但该值没有了，继续竞选

主要功能：
- 创建/删除主题
- 增加分区并分配leader分区 
- 集群Broker管理（新增 Broker、Broker 主动关闭、Broker 故障)


### replication/同步机制/选举机制

一个分区又增加了 replication 分区：
- replication 之间同步
- 数据一致性
- 角色划分

AR（Assigned Replicas）：一个Partition的所有 Replica 集合统称为，已分配的副本
ISR In-Sync Replicas：同步副本。与Leader Replica 保持同步的Replica集合 
OSR（Out-of-Sync Replicas，失去同步的副本）：与Leader Replica保持失去同步的Replica集合

AR = ISR + OSR

Replication：每个 partition 会有一个 leader

Leader Replica：处理生产者和消费者的读写请求
Follower Replica：从领导者同步数据，并在领导者副本故障时接管领导者角色。
>虽然有了副本，也有同步，但是它不提供读操作


#### 同步：
1. leader 上的数据肯定是最多的，因为它是第一个写入的
2. 同上，follower 的数据 参差不齐
3. 数据在同步过程中，肯定有延迟
4. 取 follower 数据最短（HW）的那个做为一个临界值
5. 临界值以上的消息算是被确认了，可以提供C端消费
6. 临界值以下的就是需要各节点同步的消息

LEO（Log End Offset）：下一条等待写入消息的offset(最新的offset+1)。
HW（High Watermark）:ISR中最小的LEO。leader会将ISR中最小的LEO作为HW


重新选出一个 leader：
1. 当 ledaer 挂了
2. 添加了新 broker
3. 新加了一个分区


选出的过程由 ctrl 负责 


# ACK


- 发过去即 OK，不管成功与否
- 当写 leader 即返回，其它 follower 再从 leader 上去拉数据，也叫异步备份同步，主备切换可能丢数据。
- 要等到 ISR 里所有机器同步成功后，再返回，不丢数据，但是慢。



生产者在发送消息的时候，如何保证幂等？

可能出现的不幂等的情况，如下：

- KAFKA broker 触发了 OOM/FULL GC，导致 ack timeout
- KAFKA broker 网线抖动

以上两种情况，如果使用的 JAVA SDK，且 producer 正常，会触发重发机制

实际上 broker 已经正常收到了消息，并保存会，如果重发会产生如下 问题：

- 消息重复
- 消息乱序

即使 producer 正常发了，comsumer 也会有如下问题：

- 先 commit，但执行失败，消息丢失
- 先执行成功，最后 commit，但是 commit 失败，重复消息
- 先执行成功，最后 commit，异步执行 fail，消息丢失

解决：KAFKA 里加入了 producerId 和 sequenceId，不过这两个值，使用者是碰不到的

producer 初始化的时候，会自动分配一个 pid

sequenceId 基于 producerId,生产者每次发送会生成一个，以 0 开始的自增 ID

这两个值，主要是保证了单会话的幂等性。

多会话的幂等，需要事务，不过也是基于此，在向 KAFKA SERVER 申请 pid 的时候，会连带返回 producerEpoch




