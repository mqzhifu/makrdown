
# 分布式锁

分布式锁，我另外一遍文章写过，可参考。这里，直接上 redis 如何实现：

1. SETNX + EXPIRE(value=过期时间+随时值)
>非原子操作
2. set local_resource_id value NE EX 10（SET的扩展命令）
3. 使用Lua脚本(包含SETNX + EXPIRE两条指令)
	- 长时间占用锁，会阻塞其它连接
	- 长时间占用锁，超出key的失效时间，再执行删除操作，很可能是其它连接的锁
1. 开源框架:Redisson，接上面，当获取锁后，进行检查： expireTime / 3 = 每次时间
>好像是基于 java 的
3. 多机实现的分布式锁Redlock
>redis作者给的方案，主要是解决主从模式/多点出现故障，有点像是etcd cosule 的模式了，选举/应答制度。太复杂了。不推荐  



# 启动 redis 报了几个错误

WARNING you have Transparent Huge Pages \(THP\) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never \> /sys/kernel/mm/t

ransparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.

# 主从

因为，换了端口，redis 从进程，取不到主进程的信息

顺带，截取一段 redis 日志，看看他的机制：

```

8212:S 16 Apr 2020 20:20:29.108 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:20:29.108 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:20:29.108 \* Non blocking connect for SYNC fired the event.
8212:S 16 Apr 2020 20:20:29.108 \* Master replied to PING, replication can continue...
8212:S 16 Apr 2020 20:20:29.109 \* Partial resynchronization not possible \(no cached master\)
8212:S 16 Apr 2020 20:20:29.109 \* Full resync from master: 25a5b45fafeaff32dbfb3dc405aa4afc530ac0fc:0
8212:S 16 Apr 2020 20:20:29.114 \* MASTER \<\-\> REPLICA sync: receiving 175 bytes from master
8212:S 16 Apr 2020 20:20:29.114 \* MASTER \<\-\> REPLICA sync: Flushing old data
8212:S 16 Apr 2020 20:20:29.114 \* MASTER \<\-\> REPLICA sync: Loading DB in memory
8212:S 16 Apr 2020 20:20:29.114 \* MASTER \<\-\> REPLICA sync: Finished with success
8212:S 16 Apr 2020 20:33:27.185 \# Connection with master lost.
8212:S 16 Apr 2020 20:33:27.185 \* Caching the disconnected master state.
8212:S 16 Apr 2020 20:33:27.785 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:27.785 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:27.785 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:28.787 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:28.787 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:28.788 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:29.789 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:29.789 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:29.789 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:30.791 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:30.791 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:30.791 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:31.793 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:31.793 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:31.793 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:32.795 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:32.796 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:32.796 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:33.798 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:33.798 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:33.798 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:34.800 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:34.800 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:34.800 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:35.802 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:35.802 \* MASTER \<\-\> REPLICA sync started
...........................

8212:S 16 Apr 2020 20:33:40.813 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:41.815 \* Connecting to MASTER 127.0.0.1:6380
8212:S 16 Apr 2020 20:33:41.815 \* MASTER \<\-\> REPLICA sync started
8212:S 16 Apr 2020 20:33:41.815 \# Error condition on socket for SYNC: Connection refused
8212:S 16 Apr 2020 20:33:42.422 \# User requested shutdown...
8212:S 16 Apr 2020 20:33:42.422 \* Removing the pid file.
```

> 开始同步-> 找配置文件中的 IP：PORT ，然后连接失败，尝试 N 次后，放弃同步。但是进程还是正常起着，没被 KILL

流程：

- 从库，检查到 slaveof 127.0.0.1 6370 ，开一个新的进程，连接主库，成功后
- 发送指令：sync
- 主库，收到从库请求指令，FULL or partial（增量） ，开启，新进程，把内存数据 FLUSH 到 DISK，原来让从库来取。
- 在从库摘取期间，主库还会把 新到的指令数据，添加到临时缓冲区，等从库同步完成后，再同步这个区域的数据
- 同步结束。

如果，主库接收到写指令，会同步给从库（增加同步）

## 查看 主从复制信息

info replication

从库只能读，不能写

runid(replication ID)，主服务器运行 id，Redis 实例在启动时，随机生成一个长度 40 的唯一字符串来标识当前节点

replication backlog buffer，复制积压缓冲区。是一个固定长度的 FIFO 队列，大小由配置参数 repl\-backlog\-size 指定，默认大小 1MB。

需要注意的是该缓冲区由 master 维护并且有且只有一个，所有 slave 共享此缓冲区，其作用在于备份最近主库发送给从库的数据

offset :主从各维护一份，主库发送给从库的数据，发多少这个值加多少，从库收到数据，一样累加这个值。

replication backlog buffer：这个值比较操蛋，生成在主库实例，是给从库同步使用，就是缓存主库的指令，增量同步给从库。

如果从库掉线，此时再发出连接请求且带着 offset，主库会先判断 buffer 里的 offset 能不能满足，如果不满足就得重新全同步。

另外，因为是缓存，主库的进程重启，缓存丢失，那无论 从库发什么 offset，依然都是全同步

当 slave 连接到 master，会执行 PSYNC \<runid（replication ID）\> \<offset\>，这样 master 能够只发送 slave 所缺的增量部分。

但是如果 master 的复制积压缓存区没有足够的命令记录，或者 slave 传的 runid 不对，就会进行完整重同步

从服务器，每秒会 PING 一次主服务器

REPLCONF ACK \<replication_offset\>

# role

monitor

min-slaves-to-write 3 :只有当 3 个从库连接是正常的情况下，主库才接收写操作。

# sentinel

一个类似 zookeeper 的分布式，监听/容灾 ，工具

先看下配置文件 ：

```

port 26379
protected\-mode no
sentinel monitor \<masterName\> \<ip\> \<port\> \<quorum\>
sentinel down\-after\-milliseconds \<masterName\> \<timeout\>
sentinel deny\-scripts\-reconfig yes
\#parallel\-syncs 1
daemonize yes
dir "/soft/redis/data"
logfile "/soft/redis/log/sentinel\_26379.log"
```

配置好后，直接启动

redis\-sentinel /soft/redis/conf/xxx.conf

开启 monitor

执行过程：

ping master_redis 实例

subscribe : 订阅 "**sentinel**:hello" 频道 channel

publish : 发消息给 "**sentinel**:hello" 频道 "127.0.0.1,26379,74e75dffbeb4d414b30a937eb0f3591e089142a4,0,master_6370,127.0.0.1,6370,0"

info : 获取该主库的 从库信息

和从库及其它 sentinel 建立 连接，进入 循环状态

循环状态的 3 个定时器：

每秒对其它机器上的 sentinel 进行 PING，每秒对其它机器上的 redis 实例 ping

每 2 秒，每个 sentinel，使用 redis master channel 交换信息

sentinel 每 10 秒，会对 master 和 slave 执行 info，获取 slave 信息，和主从关系

主观下线\(SDOWN subjectively down\)：单 sentinel 检测出 master ping 不通，在配置的秒数范围内，没有影响了。此时，标注下线

客观下线\(ODOWN:objectively down \)：某个 sentinel 标注了 master 为下线状态，那我再 PING 一次，如果也不行，那也标注为主观下线，当所有 sentinel 都检测出 PING 不通，那么此 master 被标注为客观下线。就是真的下线了

当一个 sentinel 对某一个台机器上的 master,ping ,响应时间超过 down\-after\-milliseconds 值时，标识该机器的 master 主观下线

之后，该 sentinel 将会向其他 sentinel 间歇性\(一秒\)发送 is\-master\-down\-by\-addr \<ip\> \<port\>，通知该 master 已经 down 了

等待获取响应信息

此时，其实机器，也开始以每秒一次的速度 ，ping 上面的发的 IP:PORT，将响应结果返回给上面的 sentinel

当其它机器也同时标记且过了半数\(配置值\)，那么，该机器就会被真的下线，注：只对 主库有效

此时，就要开启选举模式了：

先到先得，runid

架构连接体系：

A 主库 B 从库 C sentinel1 D sentinel2

C：连接 A B D

D: 连接 A B C

