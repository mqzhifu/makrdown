# zookeeper

zookeeper

管理\<服务\>注册与发现：集群、分布式。服务去ZK中注册，客户端去ZK拿服务器信息。
划重点：获取/更改信息，那就得有 文件存储 这个功能。牵扯到文件存储，且是分布式的，那就得有分布式锁。
session：管理两端长连接，当发生变化时，通过两端。
znode：一个Server节点，准确点说，是一个微服务节点，一个服务节点下面会有N台服务器。
watcher：客户端可以注册一个事件，当S端任何变动时，S端会主动推送到C端，以事件驱动的模式，更及时，性能更好。
ACL：对信息进行权限控制。create read write delete admin
分布式集群
每个节点有如下角色：

leader
learner:follower、obServer
client

每个节点状态：looking、leading、following、observer

leader 选举：首先至少得大于1台服务器。通过上面4种状态切换。集群间通过：ZAB广播通信。

Zookeeper Atomic Broadcast 协议：当leader挂了，它便进入恢复模式，开始选择新的leader，选过多后，进入广播模式。开始同步数据。也就是说，所有数据在每台服务器中都有一份副本。

选出LEADER后，leader只负责消息同步和写操作，follower负责读，和把写转到leader，observer也可以接收client的读操作，写操作依然转发到leader。

如果集群，那就牵涉到事务唯一，它是通过zxid，全局唯一号，也就是事务号。

ISR：a set of in\-sync replicas

docker pull wurstmeister/zookeeper

//启动zk

docker run \-rm \-d \-\-name zookeeper \-p 2181:2181 \-v /etc/localtime:/etc/localtime wurstmeister/zookeeper

