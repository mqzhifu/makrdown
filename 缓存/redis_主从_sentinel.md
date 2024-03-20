# 概览

Redis 支持三种集群模式，分别为主从模式、哨兵模式和Cluster模式。


# 主从模式


PSYNC

主节点：写操作
从节点：读操作


同步方式 ：
1. 全同步
>主节点把 RDB 完整发给从节点
3. 增量同步

全同步：
1. 从节点初次访启动时
2. 从节点落后主节点太多

增量同步：
从节点，同步时，带上 offset

同步指令：PSYNC

问题：
1. 从节点延迟
2. 主节点挂了，需要手动调整从节点切换成主节点
3. 全同步的时候，太影响主节点的主线程
4. 增量同步，也会影响主节点的性能
5. 如果从节点过多，同步过于频繁还是影响主节点主线程




# sentinel/哨兵模式


redis-server /path/to/redis-sentinel.conf --sentinel

单独一个进程，用来监控：redis 实例，收集数据，分析数据
1.  发送消息，做健康检查 
2.  如果，主节点出现故障
	- 把从节点切换成主节点
	- 通知客户端，主IP挂了，使用新IP
	>get-master-addr-by-name


sentinel 进程启动后：
1. 根据配置文件，连接主节点
2. 获取主节点的相关信息
3. 定时心跳
4. 订阅：__sentinel__:hello ，并发送：hello:我的IP：PORT
5. 其它哨兵也是重复上面的过程，sentinel 之间就正常通信了
6. 当某个节点的 sentinel 发送主节点故障，它就频道里发送：+switch-master 。这里叫：主观下线
7. 其它节点也会重复上面过程，过半后，该主节点要真的下线了。这里叫：客观下线
8. 下线后就开启选举新主节点过程
9. 过程跟 zab raft 类似吧，投票，第一轮选自己。
10. 同时接收其它 sentinel 的选票，根据每个节点的 同步进度
11. 一但过半，就找个新的出来

















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

