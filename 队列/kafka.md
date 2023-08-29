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

%E5%AE%89%E8%A3%85%0A\-\-\-\-%0A%E5%85%88%E6%98%AF%E5%87%86%E5%A4%87%E4%B8%8B%E5%8C%85%EF%BC%8C%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E3%80%82%E4%BD%86%EF%BC%8C%E4%BE%9D%E8%B5%96%EF%BC%9Azk%20java%20scala%20%EF%BC%8C%E8%BF%9E%E5%B8%A6%E7%9D%80%E5%B0%B1%E6%98%AF%EF%BC%9Ajdk%20gradle%20%E3%80%81wrapper%EF%BC%8C%E6%94%BE%E5%BC%83%E4%BA%86%EF%BC%8C%E7%9B%B4%E6%8E%A5docker%E5%90%A7%0A%0A%E4%B8%8B%E5%8C%85%EF%BC%9A%0A%3Edocker%20pull%20wurstmeister%2Fkafka%0A%0A%E8%BF%90%E8%A1%8C%EF%BC%9A%0A%60%60%60%0Adocker%20run%20%20\-d%20\-\-name%20kafka%20%5C%0A\-p%209092%3A9092%20%5C%0A\-e%20KAFKA\_BROKER\_ID%3D0%20%5C%0A\-e%20KAFKA\_ZOOKEEPER\_CONNECT%3D172.17.147.50%3A2181%20%5C%0A\-e%20KAFKA\_ADVERTISED\_LISTENERS%3DPLAINTEXT%3A%2F%2F172.17.147.50%3A9092%20%5C%0A\-e%20KAFKA\_LISTENERS%3DPLAINTEXT%3A%2F%2F0.0.0.0%3A9092%20%5C%0A\-t%20wurstmeister%2Fkafka%20%0A%60%60%60%0A%0A%3E%E6%B3%A8%E6%84%8F%E4%B8%80%E4%B8%8B%EF%BC%8C%E8%A6%81%E6%8A%8A%E5%AE%BF%E4%B8%BB%E6%9C%BA%E7%9A%84IP%E6%94%B9%E8%BF%87%E6%9D%A5%0A%0A%E8%BF%9B%E5%85%A5docker%20%E6%B5%8B%E8%AF%95%E4%B8%80%E4%B8%8B%0A%3Edocker%20exec%20\-it%20xxxx%20%20%2Fbin%2Fbash%0A%0A%E5%88%9B%E5%BB%BAtopic%0A%3E%24KAFKA\_HOME%2Fbin%2Fkafka\-topics.sh%20\-\-create%20\-\-zookeeper%20172.17.147.50%3A2181%20\-\-replication\-factor%201%20\-\-partitions%201%20\-\-topic%20mykafka%0A%0A%E6%9F%A5%E7%9C%8Btopic%0A%3E%24KAFKA\_HOME%2Fbin%2Fkafka\-topics.sh%20\-\-list%20\-\-zookeeper%20172.17.147.50%3A2181%0A%0A%E5%88%9B%E5%BB%BA%E7%94%9F%E4%BA%A7%E8%80%85%0A%3E%24KAFKA\_HOME%2Fbin%2Fkafka\-console\-producer.sh%20\-\-broker\-list%20127.0.0.1%3A9092%20\-\-topic%20mykafka%20%0A%0A%E5%88%9B%E5%BB%BA%E6%B6%88%E8%B4%B9%E8%80%85%0A%3E%24KAFKA\_HOME%2Fbin%2Fkafka\-console\-consumer.sh%20\-\-bootstrap\-server%20127.0.0.1%3A9092%20\-\-topic%20mykafka%20\-\-from\-beginning%0A%0A%0A%0A%E6%9F%A5%E7%9C%8B%E9%85%8D%E7%BD%AE%0A%3Ecat%20%2Fopt%2Fkafka%2Fconfig%2Fserver.properties%0A%2F%2F%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3DLog%20Basics%20%20%E6%A8%A1%E5%9D%97%E4%B8%8B%E7%9A%84%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%3D%0Alog.dirs%3D%2Fkafka%2Fkafka\-logs\-cfc5eb4cf9e3%0A%0A%E4%BF%9D%E5%AD%98%E6%B6%88%E6%81%AF%E5%86%85%E5%AE%B9%E7%9A%84%E7%9B%AE%E5%BD%95%20%0A%0A%E5%88%9A%E5%88%9A%E5%88%9B%E5%BB%BA%E7%9A%84topic%20%E5%B0%B1%E5%9C%A8%E8%BF%99%E4%B8%AA%E7%9B%AE%E5%BD%95%E4%B8%8B%E9%9D%A2%0A%E5%85%B6%E4%B8%AD%E6%96%87%E4%BB%B6%E5%A4%B9%E5%90%8D%E6%98%AF%EF%BC%9A%E4%B8%BB%E9%A2%98%E5%90%8D%2B%E5%88%86%E5%8C%BA%E5%8F%B7%0A.index%0A.log%0A.timeindex%0Aleader\-epoch\-checkpoint%0A%0A%E4%B9%9F%E5%8F%AF%E4%BB%A5%E5%88%B0zk%E4%B8%AD%E6%9F%A5%E7%9C%8B%0AzkCli.sh%0A%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E4%B8%BB%E9%A2%98%0A%3Els%20%2Fbokers%2Ftopics%0A%0A%E6%9F%A5%E7%9C%8B%E4%B8%BB%E9%A2%98%E4%B8%8B%E7%9A%84%E5%88%86%E5%8C%BA%0A%3Els%20%2Fbokers%2Ftopics%2F%E4%B8%BB%E9%A2%98%E5%90%8D%0A%0A%E6%9F%A5%E7%9C%8B%E8%AF%A6%E7%BB%86%E6%95%B0%E6%8D%AE%0A%3Eget%20%2Fbokers%2Ftopics%2F%E4%B8%BB%E9%A2%98%E5%90%8D%0A%0A%E6%9C%89%E7%B1%BB%E4%BC%BC%E5%A6%82%E4%B8%8B%E7%9A%84%E4%BF%A1%E6%81%AF%EF%BC%9A%0A%3E%7B%22version%22%3A2%2C%22partitions%22%3A%7B%220%22%3A%5B0%5D%7D%2C%22adding\_replicas%22%3A%7B%7D%2C%22removing\_replicas%22%3A%7B%7D%7D%0A%7B0%3A%5B0%5D%7D%3A0%E5%88%86%E5%8C%BA%E5%88%86%E9%85%8D%20%E4%BA%86%E4%B8%80%E4%B8%AA%E5%89%AF%E6%9C%AC%2C%E5%9C%A8brokerId%3D0%20%E4%B8%8A%E9%9D%A2%0A%0A%E6%9F%A5%E7%9C%8B%E6%9F%90%E4%B8%AA%E4%B8%BB%E9%A2%98%E7%9A%84%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF%0A%3E%24KAFKA\_HOME%2Fbin%2Fkafka\-topics.sh%20%20\-\-zookeeper%20172.17.147.50%3A2181%20\-\-describe%20\-\-topic%20mykafka%0A%0A%0A%0A%0APHP%E6%89%A9%E5%B1%95%E5%AE%89%E8%A3%85%0A\-\-\-\-\-%0A%0A%E7%8E%B0%E5%9C%A8%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9A%84%E6%98%AF%E7%94%A8%E4%B8%80%E4%B8%AAcomposer%E6%89%A9%E5%B1%95%EF%BC%9A%0A%3Ehttps%3A%2F%2Fgithub.com%2Flongyan%2Fphpkafka%2Fblob%2Fmaster%2Fdoc%2Fproducer.md%0A%0A%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%94%A8%3Acomposer%20require%20%22nmred%2Fkafka\-php%22%0A%0A%0A%0A%0A%0AKAFKA%E5%9F%BA%E7%A1%80%0A\-\-\-%0A%E8%A7%A3%E9%87%8A%EF%BC%9A%E5%88%86%E5%B8%83%E5%BC%8F%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E3%80%82%0A%0A%7C%20%E6%9C%AF%E8%AF%AD%2F%E5%90%8D%E8%AF%8D%20%7C%20%E8%A7%A3%E9%87%8A%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20broker%20%7C%20%E4%B8%80%E4%B8%AA%E8%8A%82%E7%82%B9%EF%BC%8C%E4%B8%80%E5%8F%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%EF%BC%8C%E6%98%AF%E8%A2%ABzookeeper%E7%AE%A1%E7%90%86%E3%80%82%20%7C%0A%7C%20Topic%20%7C%20%E4%B8%80%E4%B8%AA%E5%88%86%E7%B1%BB%20%7C%0A%7C%20partition%20%7C%20%E5%9F%BA%E4%BA%8ETopic%EF%BC%8C%E5%88%86%E5%8C%BA%20%E3%80%82%E5%A4%A7%E6%96%87%E4%BB%B6%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%88%86%E5%BC%80%E5%AD%98%E6%94%BE%20%7C%0A%7C%20Producer%20%7C%20%E7%94%9F%E4%BA%A7%E8%80%85%EF%BC%8C%E7%94%9F%E4%BA%A7%E6%B6%88%E6%81%AF%E4%BA%A4%E6%8A%95%E9%80%92%E5%88%B0kafka%E4%B8%AD%20%7C%0A%7C%20Consumer%20%7C%20%E6%B6%88%E8%B4%B9%E8%80%85%EF%BC%8C%E4%BB%8Ekafka%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E4%B8%AD%E8%AF%BB%E5%8F%96%E6%B6%88%E6%81%AF%20%7C%0A%0AConsumer%20%E6%A0%B9%E6%8D%AEoffset%20%EF%BC%8C%E6%9B%B4%E6%96%B0%E5%93%AA%E4%BA%9B%20%E6%B6%88%E6%81%AF%20%E8%A2%AB%E6%B6%88%E8%B4%B9%0A%0A%E6%AF%8F%E4%B8%80%E4%B8%AApartition%20%EF%BC%8C%E5%9C%A8%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E9%83%BD%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E5%A4%B9%EF%BC%8C%E4%B8%8B%E9%9D%A2%E5%8C%85%E5%90%AB%EF%BC%9A%E4%B8%80%E7%BB%84%20%E6%B6%88%E6%81%AF%20%E6%96%87%E4%BB%B6%E5%92%8C%20%E7%B4%A2%E5%BC%95%E6%96%87%E4%BB%B6%E3%80%82%E5%AE%9E%E9%99%85%E4%B8%8A%E6%98%AF%E6%8A%8A%E4%B8%80%E4%B8%AA%E5%A4%A7%E7%9A%84%20%E6%B6%88%E6%81%AF%20%E6%96%87%E4%BB%B6%EF%BC%8C%E6%8C%89%E7%85%A7%E4%B8%80%E5%AE%9Ahash%E8%A7%84%E5%88%99%EF%BC%8C%E6%89%93%E6%95%A3%E6%88%90%E8%8B%A5%E5%B9%B2%E4%B8%AA%E5%B0%8F%E6%96%87%E4%BB%B6segment%E3%80%82%0A%E8%AF%BB%E5%8F%96%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%85%88%E4%BB%8E%E7%B4%A2%E5%BC%95%E6%96%87%E4%BB%B6%E4%B8%AD%E6%89%BE%E5%87%BA%E5%88%86%E6%AE%B5offset%EF%BC%8C%E6%9C%80%E5%90%8E%E5%86%8D%E5%8E%BB%20%E5%B0%8F%E6%96%87%E4%BB%B6%E4%B8%AD%E6%89%BE%E5%88%B0%E5%85%B7%E4%BD%93%E7%9A%84%E6%B6%88%E6%81%AF%E5%80%BC%E3%80%82%0A%0A%E6%B6%88%E6%81%AF%E5%88%A0%E9%99%A4%E6%9C%BA%E5%88%B6%EF%BC%9Akafka%20%E6%98%AF%E4%B8%8D%E5%88%A0%E9%99%A4%E6%B6%88%E6%81%AF%E7%9A%84%EF%BC%8C%E9%80%9A%E8%BF%87%E7%A7%BB%E5%8A%A8offset%E5%86%B3%E5%AE%9A%E5%93%AA%E4%BA%9B%E6%B6%88%E6%81%AF%E8%A2%AB%E6%B6%88%E8%B4%B9%E4%BA%86%E3%80%82%E4%BD%86%E5%8F%AF%E4%BB%A5%E8%AE%BE%E7%BD%AE%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4%EF%BC%8C%E8%BF%87%E6%9C%9F%E7%AD%96%E7%95%A5%EF%BC%8C%E5%88%A0%E9%99%A4%E6%B6%88%E6%81%AF%0A%0AProducer%E5%BE%80broker%E7%9A%84%E6%8C%87%E5%AE%9ATopic%E7%9A%84%E6%8C%87%E5%AE%9Apartition%E4%B8%AD%E5%86%99%0AConsumer%E4%BB%8Ebroker%E7%9A%84%E6%8C%87%E5%AE%9ATopic%E7%9A%84%E6%8C%87%E5%AE%9Apartition%E4%B8%AD%E8%AF%BB%0A%0ABroker%20Controller%EF%BC%9A%E5%AE%9E%E9%99%85%E4%B8%8A%E5%B0%B1%E6%98%AFzookeeper%E7%9A%84leader%EF%BC%8C%E4%B8%8D%E5%90%8C%E7%9A%84%E6%98%AFzk%E5%8F%AA%E6%98%AF%E9%80%89%E4%B8%BE%E5%87%BA%E4%B8%80%E4%B8%AAleader%E5%B0%B1%E7%BB%93%E6%9D%9F%E4%BA%86%EF%BC%8Ckafka%E6%98%AF%E7%94%A8%E8%BF%99%E4%B8%AAleader%E6%9D%A5%E5%81%9A%E6%8E%A5%E4%B8%8B%E6%9D%A5%E7%9A%84%E6%93%8D%E4%BD%9C%E3%80%82%E4%B9%9F%E5%B0%B1%E6%98%AFzk%E6%8F%90%E4%BE%9B%E4%BA%86%E5%9F%BA%E7%A1%80%E5%8A%9F%E8%83%BD%EF%BC%8Ckafka%E6%89%A9%E5%B1%95%E4%BA%86%E4%B8%80%E4%B8%8B%E3%80%82%0A%E9%80%89%E4%B8%BE%E5%87%BActrl%E5%90%8E%EF%BC%9A%E5%AE%83%E6%98%AF%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%EF%BC%8C%E5%AE%83%E4%BC%9A%E5%86%8D%E4%B8%BA%E6%AF%8F%E4%B8%AApartition%E5%86%8D%E9%80%89%E4%B8%BE%E5%87%BAleader%0A%0A%0Aleader%EF%BC%9A%E8%B4%9F%E8%B4%A3%E8%AF%BB%E5%86%99%EF%BC%8Cfollowe%20%E5%8F%AA%E8%B4%9F%E8%B4%A3%E5%A4%87%E4%BB%BD%E3%80%82%0A%0A%E5%AE%B9%E7%81%BE%EF%BC%9A%E5%BD%93%E6%9F%90%E4%B8%80%E4%B8%AAbroker%EF%BC%8C%E6%8C%82%E4%BA%86%EF%BC%8Czk%E9%82%A3%E9%87%8C%E6%9C%89watch%20%E4%BC%9A%E9%80%9A%E7%9F%A5%EF%BC%8CController%EF%BC%8C%E4%BC%9A%E6%89%BE%E5%88%B0%E5%8F%97%E5%BD%B1%E5%93%8D%E7%9A%84partition%EF%BC%8C%E6%89%BE%E5%87%BA%0A%E6%96%B0leader%E3%80%82%0A%0A%E5%90%8C%E6%AD%A5%E6%9C%BA%E5%88%B6%0A1%EF%BC%9A%E5%8F%91%E8%BF%87%E5%8E%BB%E5%8D%B3OK%EF%BC%8C%E4%B8%8D%E7%AE%A1%E6%88%90%E5%8A%9F%E4%B8%8E%E5%90%A6%0A2%EF%BC%9A%E5%BD%93%E5%86%99leader%E5%8D%B3%E8%BF%94%E5%9B%9E%EF%BC%8C%E5%85%B6%E5%AE%83follower%E5%86%8D%E4%BB%8Eleader%E4%B8%8A%E5%8E%BB%E6%8B%89%E6%95%B0%E6%8D%AE%EF%BC%8C%E4%B9%9F%E5%8F%AB%E5%BC%82%E6%AD%A5%E5%A4%87%E4%BB%BD%E5%90%8C%E6%AD%A5%EF%BC%8C%E4%B8%BB%E5%A4%87%E5%88%87%E6%8D%A2%E5%8F%AF%E8%83%BD%E4%B8%A2%E6%95%B0%E6%8D%AE%E3%80%82%0A3%EF%BC%9A%E8%A6%81%E7%AD%89%E5%88%B0ISR%E9%87%8C%E6%89%80%E6%9C%89%E6%9C%BA%E5%99%A8%E5%90%8C%E6%AD%A5%E6%88%90%E5%8A%9F%E5%90%8E%EF%BC%8C%E5%86%8D%E8%BF%94%E5%9B%9E%EF%BC%8C%E4%B8%8D%E4%B8%A2%E6%95%B0%E6%8D%AE%EF%BC%8C%E4%BD%86%E6%98%AF%E6%85%A2%E3%80%82%0A%0A%0AReplication%EF%BC%9A%E6%AF%8F%E4%B8%AApartition%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AAleader%0A%0A%0A%0A%0A%0A%E7%94%9F%E4%BA%A7%E8%80%85%E5%9C%A8%E5%8F%91%E9%80%81%E6%B6%88%E6%81%AF%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E5%B9%82%E7%AD%89%EF%BC%9F%0A%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E7%9A%84%E4%B8%8D%E5%B9%82%E7%AD%89%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%A6%82%E4%B8%8B%EF%BC%9A%0A1%20KAFKA%20broker%E8%A7%A6%E5%8F%91%E4%BA%86OOM%2FFULL%20GC%EF%BC%8C%E5%AF%BC%E8%87%B4ack%20%20timeout%0A2%20KAFKA%20broker%E7%BD%91%E7%BA%BF%E6%8A%96%E5%8A%A8%0A%E4%BB%A5%E4%B8%8A%E4%B8%A4%E7%A7%8D%E6%83%85%E5%86%B5%EF%BC%8C%E5%A6%82%E6%9E%9C%E4%BD%BF%E7%94%A8%E7%9A%84JAVA%20SDK%EF%BC%8C%E4%B8%94producer%E6%AD%A3%E5%B8%B8%EF%BC%8C%E4%BC%9A%E8%A7%A6%E5%8F%91%E9%87%8D%E5%8F%91%E6%9C%BA%E5%88%B6%0A%E5%AE%9E%E9%99%85%E4%B8%8Abroker%E5%B7%B2%E7%BB%8F%E6%AD%A3%E5%B8%B8%E6%94%B6%E5%88%B0%E4%BA%86%E6%B6%88%E6%81%AF%EF%BC%8C%E5%B9%B6%E4%BF%9D%E5%AD%98%E4%BC%9A%EF%BC%8C%E5%A6%82%E6%9E%9C%E9%87%8D%E5%8F%91%E4%BC%9A%E4%BA%A7%E7%94%9F%E5%A6%82%E4%B8%8B%20%E9%97%AE%E9%A2%98%EF%BC%9A%20%0A1%20%E6%B6%88%E6%81%AF%E9%87%8D%E5%A4%8D%0A2%20%E6%B6%88%E6%81%AF%E4%B9%B1%E5%BA%8F%0A%0A%E5%8D%B3%E4%BD%BFproducer%E6%AD%A3%E5%B8%B8%E5%8F%91%E4%BA%86%EF%BC%8Ccomsumer%20%E4%B9%9F%E4%BC%9A%E6%9C%89%E5%A6%82%E4%B8%8B%E9%97%AE%E9%A2%98%EF%BC%9A%0A1%20%E5%85%88commit%EF%BC%8C%E4%BD%86%E6%89%A7%E8%A1%8C%E5%A4%B1%E8%B4%A5%EF%BC%8C%E6%B6%88%E6%81%AF%E4%B8%A2%E5%A4%B1%0A2%20%E5%85%88%E6%89%A7%E8%A1%8C%E6%88%90%E5%8A%9F%EF%BC%8C%E6%9C%80%E5%90%8Ecommit%EF%BC%8C%E4%BD%86%E6%98%AFcommit%E5%A4%B1%E8%B4%A5%EF%BC%8C%E9%87%8D%E5%A4%8D%E6%B6%88%E6%81%AF%0A3%20%E5%85%88%E6%89%A7%E8%A1%8C%E6%88%90%E5%8A%9F%EF%BC%8C%E6%9C%80%E5%90%8Ecommit%EF%BC%8C%E5%BC%82%E6%AD%A5%E6%89%A7%E8%A1%8Cfail%EF%BC%8C%E6%B6%88%E6%81%AF%E4%B8%A2%E5%A4%B1%20%0A%0A%0A%E8%A7%A3%E5%86%B3%EF%BC%9AKAFKA%E9%87%8C%E5%8A%A0%E5%85%A5%E4%BA%86%20producerId%20%E5%92%8C%20sequenceId%EF%BC%8C%E4%B8%8D%E8%BF%87%E8%BF%99%E4%B8%A4%E4%B8%AA%E5%80%BC%EF%BC%8C%E4%BD%BF%E7%94%A8%E8%80%85%E6%98%AF%E7%A2%B0%E4%B8%8D%E5%88%B0%E7%9A%84%0A%0Aproducer%20%E5%88%9D%E5%A7%8B%E5%8C%96%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E4%BC%9A%E8%87%AA%E5%8A%A8%E5%88%86%E9%85%8D%E4%B8%80%E4%B8%AApid%0AsequenceId%20%E5%9F%BA%E4%BA%8EproducerId%2C%E7%94%9F%E4%BA%A7%E8%80%85%E6%AF%8F%E6%AC%A1%E5%8F%91%E9%80%81%E4%BC%9A%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AA%EF%BC%8C%E4%BB%A50%E5%BC%80%E5%A7%8B%E7%9A%84%E8%87%AA%E5%A2%9EID%0A%0A%E8%BF%99%E4%B8%A4%E4%B8%AA%E5%80%BC%EF%BC%8C%E4%B8%BB%E8%A6%81%E6%98%AF%E4%BF%9D%E8%AF%81%E4%BA%86%E5%8D%95%E4%BC%9A%E8%AF%9D%E7%9A%84%E5%B9%82%E7%AD%89%E6%80%A7%E3%80%82%0A%E5%A4%9A%E4%BC%9A%E8%AF%9D%E7%9A%84%E5%B9%82%E7%AD%89%EF%BC%8C%E9%9C%80%E8%A6%81%E4%BA%8B%E5%8A%A1%EF%BC%8C%E4%B8%8D%E8%BF%87%E4%B9%9F%E6%98%AF%E5%9F%BA%E4%BA%8E%E6%AD%A4%EF%BC%8C%E5%9C%A8%E5%90%91KAFKA%20SERVER%E7%94%B3%E8%AF%B7pid%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E4%BC%9A%E8%BF%9E%E5%B8%A6%E8%BF%94%E5%9B%9EproducerEpoch%20%0A
