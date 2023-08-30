# kafka

## 安装

先是准备下包，编译安装。但，依赖：zk java scala ，连带着就是：jdk gradle 、wrapper，放弃了，直接docker吧

下包：

> docker pull wurstmeister/kafka

运行：

```
docker run  -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=172.17.147.50:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.17.147.50:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-t wurstmeister/kafka 
```

> 注意一下，要把宿主机的IP改过来

进入docker 测试一下

> docker exec \-it xxxx /bin/bash

创建topic

> $KAFKA\_HOME/bin/kafka\-topics.sh \-\-create \-\-zookeeper 172.17.147.50:2181 \-\-replication\-factor 1 \-\-partitions 1 \-\-topic mykafka

查看topic

> $KAFKA\_HOME/bin/kafka\-topics.sh \-\-list \-\-zookeeper 172.17.147.50:2181

创建生产者

> $KAFKA\_HOME/bin/kafka\-console\-producer.sh \-\-broker\-list 127.0.0.1:9092 \-\-topic mykafka

创建消费者

> $KAFKA\_HOME/bin/kafka\-console\-consumer.sh \-\-bootstrap\-server 127.0.0.1:9092 \-\-topic mykafka \-\-from\-beginning

查看配置

> cat /opt/kafka/config/server.properties
> 
> 
> //==============Log Basics 模块下的===========
> 
> 
> log.dirs=/kafka/kafka\-logs\-cfc5eb4cf9e3

保存消息内容的目录

刚刚创建的topic 就在这个目录下面

其中文件夹名是：主题名\+分区号

.index

.log

.timeindex

leader\-epoch\-checkpoint

也可以到zk中查看

zkCli.sh

查看所有主题

> ls /bokers/topics

查看主题下的分区

> ls /bokers/topics/主题名

查看详细数据

> get /bokers/topics/主题名

有类似如下的信息：

> {"version":2,"partitions":{"0":\[0\]},"adding\_replicas":{},"removing\_replicas":{}}
> 
> 
> {0:\[0\]}:0分区分配 了一个副本,在brokerId=0 上面

查看某个主题的详细信息

> $KAFKA\_HOME/bin/kafka\-topics.sh \-\-zookeeper 172.17.147.50:2181 \-\-describe \-\-topic mykafka

## PHP扩展安装

现在比较好的是用一个composer扩展：

> https://github.com/longyan/phpkafka/blob/master/doc/producer.md

也可以用:composer require "nmred/kafka\-php"

## KAFKA基础

解释：分布式消息队列。

|术语/名词|解释                                     |
|---------|-----------------------------------------|
|broker   |一个节点，一台服务器，是被zookeeper管理。|
|Topic    |一个分类                                 |
|partition|基于Topic，分区 。大文件，可以分开存放   |
|Producer |生产者，生产消息交投递到kafka中          |
|Consumer |消费者，从kafka消息队列中读取消息        |

Consumer 根据offset ，更新哪些 消息 被消费

每一个partition ，在服务器上都会有一个文件夹，下面包含：一组 消息 文件和 索引文件。实际上是把一个大的 消息 文件，按照一定hash规则，打散成若干个小文件segment。

读取的时候，先从索引文件中找出分段offset，最后再去 小文件中找到具体的消息值。

消息删除机制：kafka 是不删除消息的，通过移动offset决定哪些消息被消费了。但可以设置过期时间，过期策略，删除消息

Producer往broker的指定Topic的指定partition中写

Consumer从broker的指定Topic的指定partition中读

Broker Controller：实际上就是zookeeper的leader，不同的是zk只是选举出一个leader就结束了，kafka是用这个leader来做接下来的操作。也就是zk提供了基础功能，kafka扩展了一下。

选举出ctrl后：它是一个线程，它会再为每个partition再选举出leader

leader：负责读写，followe 只负责备份。

容灾：当某一个broker，挂了，zk那里有watch 会通知，Controller，会找到受影响的partition，找出

新leader。

同步机制

1：发过去即OK，不管成功与否

2：当写leader即返回，其它follower再从leader上去拉数据，也叫异步备份同步，主备切换可能丢数据。

3：要等到ISR里所有机器同步成功后，再返回，不丢数据，但是慢。

Replication：每个partition会有一个leader

生产者在发送消息的时候，如何保证幂等？

可能出现的不幂等的情况，如下：

1 KAFKA broker触发了OOM/FULL GC，导致ack timeout

2 KAFKA broker网线抖动

以上两种情况，如果使用的JAVA SDK，且producer正常，会触发重发机制

实际上broker已经正常收到了消息，并保存会，如果重发会产生如下 问题：

1 消息重复

2 消息乱序

即使producer正常发了，comsumer 也会有如下问题：

1 先commit，但执行失败，消息丢失

2 先执行成功，最后commit，但是commit失败，重复消息

3 先执行成功，最后commit，异步执行fail，消息丢失

解决：KAFKA里加入了 producerId 和 sequenceId，不过这两个值，使用者是碰不到的

producer 初始化的时候，会自动分配一个pid

sequenceId 基于producerId,生产者每次发送会生成一个，以0开始的自增ID

这两个值，主要是保证了单会话的幂等性。

多会话的幂等，需要事务，不过也是基于此，在向KAFKA SERVER申请pid的时候，会连带返回producerEpoch

