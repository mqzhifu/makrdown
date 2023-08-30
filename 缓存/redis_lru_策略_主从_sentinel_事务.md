# redis lru 策略 主从 sentinel 事务

LRU

Least Recently Used ，最少 最近 被使用的。

redis的淘汰策略得结合3个参数

maxmemory:占用内存大小。如果为0，不做限制

maxmemory\_policy：LUR规则

maxmemory\_samples：采样概率

几种 LUR算法规则：

noeviction:返回错误，不淘汰

allkeys\-lru:所有的数据均可淘汰，采取LRU淘汰

allkeys\-random:回收所有的数据，采用随机算法

volatile\-lru:只有设置超时的数据才会淘汰,采用LUR算法

volatile\-ttl:只有设置超时的数据才会淘汰，但采用TTL 淘汰

volatile\-random：回收设置超时的数据，采用随机算法

实际上从这几种算法也能看出，既然是缓存数据就应该知道失效时间，任何程序员不设失效时间都是耍流氓。

volatile是非全量的，所以如果失效时间较多的实例，这种方式清理出的空间很少。而且还浪费计算时间。

WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.

WARNING overcommit\_memory is set to 0\! Background save may fail under low memory condition

//内存分配策略

/etc/sysctl.conf

vm.overcommit\_memory

0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。

1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。

2， 表示内核允许分配超过所有物理内存和交换空间总和的内存

启动redis 报了几个错误

WARNING you have Transparent Huge Pages \(THP\) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never \> /sys/kernel/mm/t

ransparent\_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.

主从

因为，换了端口，redis从进程，取不到主进程的信息

顺带，截取一段redis日志，看看他的机制：

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

开始同步\-》找配置文件中的IP：PORT ，然后连接失败，尝试N次后，放弃同步。但是进程还是正常起着，没被KILL

流程：

从库，检查到 slaveof 127.0.0.1 6370 ，开一个新的进程，连接主库，成功后

发送指令：sync

主库，收到从库请求指令，FULL or partial（增量） ，开启，新进程，把内存数据FLUSH 到DISK，原来让从库来取。

在从库摘取期间，主库还会把 新到的指令数据，添加到临时缓冲区，等从库同步完成后，再同步这个区域的数据

同步结束。

如果，主库接收到写指令，会同步给从库（增加同步）

//查看 主从复制信息

info replication

从库只能读，不能写

runid\(replication ID\)，主服务器运行id，Redis实例在启动时，随机生成一个长度40的唯一字符串来标识当前节点

replication backlog buffer，复制积压缓冲区。是一个固定长度的FIFO队列，大小由配置参数repl\-backlog\-size指定，默认大小1MB。

需要注意的是该缓冲区由master维护并且有且只有一个，所有slave共享此缓冲区，其作用在于备份最近主库发送给从库的数据

offset :主从各维护一份，主库发送给从库的数据，发多少这个值加多少，从库收到数据，一样累加这个值。

replication backlog buffer：这个值比较操蛋，生成在主库实例，是给从库同步使用，就是缓存主库的指令，增量同步给从库。

如果从库掉线，此时再发出连接请求且带着offset，主库会先判断buffer里的offset 能不能满足，如果不满足就得重新全同步。

另外，因为是缓存，主库的进程重启，缓存丢失，那无论 从库发什么offset，依然都是全同步

当slave连接到master，会执行PSYNC \<runid（replication ID）\> \<offset\>，这样master能够只发送slave所缺的增量部分。

但是如果master的复制积压缓存区没有足够的命令记录，或者slave传的runid 不对，就会进行完整重同步

从服务器，每秒会PING 一次主服务器

REPLCONF ACK \<replication\_offset\>

role

monitor

min\-slaves\-to\-write 3 :只有当3个从库连接是正常的情况下，主库才接收写操作。

sentinel

一个类似zookeeper的分布式，监听/容灾 ，工具

先看下配置文件 ：

port 26379

protected\-mode no

sentinel monitor \<masterName\> \<ip\> \<port\> \<quorum\>

sentinel down\-after\-milliseconds \<masterName\> \<timeout\>

sentinel deny\-scripts\-reconfig yes

\#parallel\-syncs 1

daemonize yes

dir "/soft/redis/data"

logfile "/soft/redis/log/sentinel\_26379.log"

配置好后，直接启动

redis\-sentinel /soft/redis/conf/xxx.conf

开启monitor

执行过程：

ping master\_redis 实例

subscribe : 订阅 "**sentinel**:hello" 频道 channel

publish : 发消息给 "**sentinel**:hello" 频道 "127.0.0.1,26379,74e75dffbeb4d414b30a937eb0f3591e089142a4,0,master\_6370,127.0.0.1,6370,0"

info : 获取该主库的 从库信息

和从库及其它sentinel 建立 连接，进入 循环状态

循环状态的3个定时器：

每秒对其它机器上的sentinel进行PING，每秒对其它机器上的redis 实例 ping

每2秒，每个sentinel，使用redis master channel 交换信息

sentinel 每10秒，会对 master和slave 执行info，获取slave信息，和主从关系

主观下线\(SDOWN subjectively down\)：单sentinel检测出master ping 不通，在配置的秒数范围内，没有影响了。此时，标注下线

客观下线\(ODOWN:objectively down \)：某个sentinel标注了master为下线状态，那我再PING 一次，如果也不行，那也标注为主观下线，当所有sentinel都检测出PING不通，那么此master被标注为客观下线。就是真的下线了

当一个sentinel对某一个台机器上的master,ping ,响应时间超过 down\-after\-milliseconds 值时，标识该机器的master主观下线

之后，该 sentinel 将会向其他sentinel间歇性\(一秒\)发送 is\-master\-down\-by\-addr \<ip\> \<port\>，通知该master 已经down了

等待获取响应信息

此时，其实机器，也开始以每秒一次的速度 ，ping 上面的发的IP:PORT，将响应结果返回给上面的sentinel

当其它机器也同时标记且过了半数\(配置值\)，那么，该机器就会被真的下线，注：只对 主库有效

此时，就要开启选举模式了：

先到先得，runid

架构连接体系：

A 主库 B从库 C sentinel1 D sentinel2

C：连接 A B D

D: 连接 A B C

事务

multi \#标记事务的开始

exec \#执行事务的commands队列

discard \#结束事务，并清除commands队列，

开启事务后，任何操作都是 返回：queue，告诉C端，该指令已接收到，等待最后批量执行，如果期间有脚本是错误的（语法 ），会提示

最后exec执行，但如果期间有脚本 出错，后面的搜集依然还是会执行。

discard，只是是结束了当前事务脚本

所以，redis事务，只是保证了一组指令执行，并不支持回滚~

加入watch，实现‘类似’真实的事务

session 1

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

因为返回的是nil证明事务失败

问题：幻读

session 1

get kkk

.....

doing

\(此时 session 提交了事务修改了kkk=2\)

get kkk

eval

在redis中，执行 指令，为了一致性。

可以加入一些逻辑判断 ，如：if

事务跟LUA 有点像，但没有回滚机制

过期策略

正常，一个KEY如果过期了，REDIS是不知道的，只有，有一个SESSION连接，并操作key，redis才会去检查

优点：没必要在key是否过期上浪费时间，减少CPU占用

缺点：浪费内存

LRU：会定期抽样，清理掉一些过期的KEY

