# 概览

Redis 支持三种集群模式：主从模式、哨兵模式和Cluster模式
主从模式：最简单
哨兵模式：单独再开一组进程，用来监控主从模式
- 监听主节点状态
- 发送报警通知
- master 自动转移
cluster模式：未知


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


#### 搭建


前置条件：redis 进程均已启动，且主从同步模式正常

创建一个配置文件：sentinel.conf
编辑配置文件：vim sentinel.conf

```
// 2台哨兵实例判断监视服务器为主观下线，则该监视服务器变为客观下线(可以故障转移)
sentinel monitor mymaster 127.0.0.1 6379 2  

// master 无效回复时间达到 30000ms，则该服务器主观下线 
sentinel down-after-milliseconds mymaster 30000 

//选项指定了在执行故障转移时，最多可以有多少个从服务器同时对新的主服务器进行同步
这个数字越小，完成故障转移所需的时间就越长。
sentinel parallel-syncs mymaster 1 

sentinel failover-timeout mymaster 180000

sentinel auth-pass manager1 123456
```

启动：
redis-server /path/to/redis-sentinel.conf --sentinel


#### 通信流程

sentinel 进程启动后：

1. 根据配置文件中的： master IP:PORT，连接主节点
	- 订阅连接，订阅 master Channel：\_sentinel\_:hello 
		1. 发送：hello:我的IP：PORT
		2. 接收新的 slave 注册 ip:port 信息
	- 命令连接
		1. 每秒定时心跳：向 master 和 所有 slave 发送 ping 
		2. 每10s一次，向被监控的主服务器发送 info 命令，获取 master/slave 状态
1. 向从节点的 也建立两个连接：订阅连接 和  命令连接
2. sentinel 之间也会建立连接，但只是命令连接，不用订阅。只需要知道对方是否挂了即可
3. 其它哨兵也是重复上面的过程
4. 这样：sentinel 与sentinel之间有通信，sentinel 与 主从 实例也有通信了

简单看：
- 通过 主从 的 redis 实例，订阅 channel 来确定其它 sentinel 的存在与状态
- 通过 主从 的 redis 实例，info 来确定 主从 相关详细信息
- 通过 sentinel 实例，确定 sentinel 健康状态

>sentinel 与 sentinel之间 有监控
>sentinel 与 主从实例 有监控

sentinel 切换主从：
1. 当节点A sentinel 向 master 发出PING消息，但是一定时间内没有收到 master 返回的 pong
2. 它会向其它 sentinel 节点发送消息：+switch-master 。这里叫：主观下线
3. 其它 sentinel 节点 收到 +switch-master，会向 master 发消息验证是否真的下线了
4. 如果真的 master 出现问题，就回复 A节点： master 确实出问题了
5. 节点A收到其它节点的回复，达到一定数量：确认 master 已经出问题了。主节点就真下线了(客观下线)
>数量：取决于配置文件中的设置
7. 下线后就开启选举新主节点过程

sentinel 投票（假设有3台机器）：
- 开始都投自己
- A 会收到 B C 的投票，但肯定有时间顺序
- 如果B先到，那C的投票直接拒绝。同时向AC  发送 B 为 leader
>这种可以叫：拉票，即：谁先到，就认为谁可能是新的 leader 然后给对方投票

这里用到了：定时器，一轮投票什么时候结束，除了接收到所有投票外，还要有个定时器，且：定时器是随机数，这样能更快的选举成功。



当 之前挂了的 master 重新启动后：它将不再是master，而是做为 slave 接收新的master的同步数据；





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

