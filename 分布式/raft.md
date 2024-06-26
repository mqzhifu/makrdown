# 概览

raft：严格来说是一篇论文，它并没有具体的实现，更多的是偏向于理论设计。它解决了：强一致性、共识算法的问题，也就等于：去中心化、高可用的分布式协议。

## 官方演示地址

https://raft.github.io/

它这个 DEMO 挺好玩儿的，用一下，才能更好的理解：什么叫分布式

## 特点

1. 强领导者:Raft 推出一个 Leader 的角色，同时赋予该角色非常强的领导能力，所有数据的管理都交由该角色。
2. Leader 选举：Raft 使用了一个随机计时器来选举领导者。通过简单地在心跳机制的基础上加上一个随机计时器，避免了大部分同时选举的发生。
3. 成员关系调整：Raft 使用一种共同一致的方法来处理集群成员变换的问题，在这种方法下，处于调整过程中的两种不同的配置集群中大多数机器会有重叠，这就使得集群在成员变换的时候依然可以继续工作。

## 结合 paxos

1. 它的角色更少，更专注
2. 通过过程 更简单，通信次数较少

## 术语

#### 任期(term)

每个 leader 从竞选开始->成功到最后放弃(宕机)，被称为一个 term 任期，也可以理解为一次 leader 的执政周期

| 模块名          | 说明                 |
| --------------- | -------------------- |
| leader          | 选举过程             |
| log replication | 日志复制过程         |
| safety          | 各种异常、安全等考虑 |

| 角色名    | 说明                                                                            |
| --------- | ------------------------------------------------------------------------------- |
| leader    | 一台节点，通过选举产生，用于接收 client 的 IO 操作，同时分发到所有 follwer 节点 |
| follower  | 一台节点，跟属 leader 领导                                                      |
| candidate | 一台节点，当收不到 leader 心跳后，开启选举状态                                  |

## 通信 RPC

1. RequestVote 消息体:各节点均可发起，由 candidate 选举期间发起

```
type RequestVoteReq struct{
    term int //自己当前任期号
    candidateld int //自己的ID
    last_log_index //自己最后一个日志ID
    last_log_term //自己最后一个日志的任期
}

type RequestVoteRes struct{
    term int //自己当前任期号
    voteGranted bool //自己会不会投票给这个candidate
}
```

1. AppendEntries 消息体:只能由 leader 发起，日志追回请求，包括：执行日志复制、心跳、日志对齐

```go
type AppendEntriesReq struct {
	Term         int    //当前任期号
	LeaderId     int    //自己的leaderId
	PrevLogIndex int    //前一个日志号
	PrevLogTerm  int    //前一个任期
	Entries      []byte //具体日志内容，如：指令集
	LeaderCommit int    //leader提交日志号
}

type AppendEntriesRes struct {
	Term    int  //自己当前任期号
	success bool //如果follower包含前一个日志号，返回true
}

```

# leader election 选举

先看下它的官方图:

> 定时器的超时时间：随机\(150~300ms\)，具体也可以配置
>
> 这个还得具体分析，一次广播到落盘大概是 5~20ms，它官方给的最大间隔是:5~500ms

1. 所有机器首次启动，均为 follower 状态
2. 创建心跳定时器\(时间为随时\)，如在 timer 周期内未收到任何\(leader \+ candidate \)消息，开启选举过程：
   1. 状态变更： follower =\> candidate
   2. 读取本地属性： currentTermNumber，然后： term+1
   3. 读取本地日志最大索引号：last_index_id
   4. 设置：自己的 ID 为被投票者（间接看：自己投自己一票 ）
   5. 将：currentTermNumber last_index_id term\+1 candidateId ，打包 RequestVoteReq，发送：RequestVote RPC 消息
   6. 创建定时器\(时间为随时\)，等待其它节点消息返回
3. 定时器，等待期间可能会发生如下事情：
   1. 在 timer 时间内，收到其它节点返回的选票，其中选自己的票数：过半，那么证明自己就是 leader。
      1. 发消息:AppendEntries RPC, 给其它所有 clientNode ，告知我是 leader 了
      2. 将状态由 candidate =\> leader
   2. timer 超时，状态不变，再设定一个新 timer\(随机时间\)，给本地 term\+1，继续发送 AppendEntries RPC 给其它节点，开启新竞选
      1. 可能是有些节点的响应过慢没收着
      2. 所有节点都投票了，但是自己的票数不到一半
   3. 在 timer 时间内，收到其它机器竞选成功的：AppendEntries RPC，将 candidate =\> follower，删除旧 timer，创建新的 timer
4. 上面 3 步的前提：大家同时设定的心跳 timer，某一台机器先超时了，它就最先开启了选举，而其它机器的状态可能，如下：
   1. follower，因为别的机器先超时，肯定会先发竞选消息，自己被动接收，那么，刷新自己的 timer ，也就是 follower 尽量维持 follower，不参选
      1. 如果接收的消息的 term 比较自己小，直接丢掉
      2. 如果接收的消息的 term 比较自己大，且本轮 term 还未投过票，先更新本地 term，投票给它
      3. 如果接收的消息的 term 跟自己相同，先来先得，后来拒绝
   2. candidate，上面有一台觉醒，但同时还有一台机器也觉醒了，两个人同时发起竞选。两个人会同时收到 candidate 竞选的消息\(当确定已经参选，就证明肯定给自己投票了\)
      1. 如果接收的消息的 term
         1. 对方的 term 相同，直接回绝\(这个是两个随时时间差不多，就比网速吧，先到先得，也可能出现分票\)
         2. 对方的 term 更大，那么自己 candidate =\> follower\(自己在上一个任期整合都是挂了的状态，碰巧刚好，就参与了选票\)
         3. 对方的 term 更小，直接回绝\(这种是对面在整个任期都挂了，正好新任期它又好了，所以少了 N 个任期\)
      2. 分票：两人收到集群内所有的投票各自都是一半，那么选举失败，timer 超时，再定个 timer，重复投票，因为 timer 是随机的，最终肯定会有一个先到期
5. 特殊情况，假如 A B C D E 正常组成集群，A 为 leader，此时网络故障，A B 是原状态，C D E 组成新的集群，C 为新 leader，然后故障好了，此时
   1. C 会给 A 发送心跳，A 接收到心跳，发现对方的 term 更大，此时:leader =\> follower ，
   2. C 会给 B 发送心跳，B 更新本地 term 值，变更 leader 指向为 C
   3. C D E 同时也会收到 A 的心中，但发现 term 值过小，直接拒绝
6. 特殊情况，如果网络环境特别差：RTT \> 300MS ，那么就会无限选举。因为每个 candidate 最大超时是 300MS，它发出去一条到收一条消息的 RTT\>300ms，自己早就超时了，已经开启下一个任期遥选举了，不过是比较极端情况，不太可能发生，也可以根据自己机器的网络 RTT 时间，放大超时随时值
7. 特殊情况，假设一共 5 台机器，还剩下 B C D 是正常的，现在 D 是 leader，然后挂了，这时就只剩下 B C 两台机器，他们无论怎么选最多只能得到 2 票，并没有到过半的 3 票，所以会无限循环选举下去。
8. 特殊情况，假设一共 5 台机器，还剩下 B C D 是正常的，现在 D 是 leader，然后 B 或 C 挂了，此时集群还是能正常收消息，但是却无法确认，也就是消息在 D C 或 B 都存了，但收不到一半集群的'消息确认'

> 整个选举机制核心：两个随机时间的 timer 解决了一切，不会死锁，又能选出唯一的 leader,确实巧妙

## 分析一下 RequestVoteReq 消息体

有两个字段需要注意下：last_log_index 和 last_log_term

1. 假设当前 leader 为 A，在任期：1 期间，有一条日志已写入
2. 通知其它节点，进行复制操作
3. 收到半数节点复制成功的消息，开始本地提交，然后，通知所有节点进行提交
4. 假设，在这期间，某一台 follower\-B，也在任期：1，但因为网络不太好，复制进度较慢，未收到该条日志复制的请求，也未收日志提交的请求
5. 假设此时 leader\-A 挂了，B 是新的 leader，它的日志就是落后的，也没有上面 A 中的那条日志
6. 如果根据强 leader 覆盖法则，那么原 A 的日志多出的几条日志就被新 leader\-B 给覆盖掉

> 上面的问题核心是：某个进度落后的 follower 虽然进度慢缺失了某几条消息，但是在整个任期来看，它没有全丢失，当 leader 挂了，它依然还能参选并成为新 leader，因为：他的 iterm 并没有落后 ，而如果只拿 item 判断是否可当 ledaer 就会有这样的总是，会把之前进度快的节点的消息给覆盖掉。

所以引入了上面两个字段值：

1. 进度较慢的 candidate，发出 RequestVoteReq 消息，其中包含了：candidate 的日志进度 last_log_index last_log_term
2. follower 在接收后，与本机的 last_log_index last_log_term 进行对比，如果 term 相同，再比较 last_log_index，发现自己的更新，直接回绝该 candidate
3. 如果某台 follower 进度很新，但苏醒较慢，可以用这个功能先概率性的过虑掉进度落后的 candidate，为自己争取时间参选

那么加两个字段的核心是：当原 leader 挂了后，尽量选出日志最长的那个 candidate

但这里还有个问题，两个值是一起判定还是有顺序的？

1. 先判断 last_log_term
2. 再判断 last_log_index
   所以，这两个值更倾向解决，在同一任期的前提下，看哪个日志更长，而如果这个时候，有的日志虽然较短，但是可能任期号更大，那它也完全可以成为 leader 的，那么引出另外一个问题：
   新 leader 的 term\+1 了，任期就更新了，但是本机上可能还有旧任期内未提交的日志，那新任期是否提前旧任期的日志呢？
   答案:不会
   因为有一个风险：如果提交旧任期的未提交的数据，那么此时它挂了，另外一个新机器当选新 leader，它的数据是缺失了，那么它可能会覆盖掉刚刚提交的旧任期的数据，这个挺可怕的，或者说，raft 不能接受覆盖已提交的数据

但如果有新的写入操作，会连带着旧的一起提交 commit_id

这个问题的核心是：集群\-提交确认，即：leader 提交后，要立刻发送给所有 follower，并阻塞，等待半数 follwer 回复：提交 OK 后，再回复 client 本次日志写入成功。不过 raft 没提，估计可能 raft 觉得没必要考虑这种场景吧。另外，就算加这个确认机制，但也会非常麻烦，还得考虑各种断网的情况，好像 google percolator 模型有解决。

每个节点必须得有 3 个属性：

currentTerm：节点当前所处的任期。

votedFor 　：节点当前任期所跟随的其他节点的 ID。如果是投票阶段，表示节点所投的候选人。如果已经竞选结束，就是当前的领导人。

currentLogIndex:本机日志索引号

> 基本上日常的操作流程就是这 3 个属性值\+状态值\+消息传输，基本上够用了。raft 的实现确实很简单。

## 复制状态机

Replicated state machines：保证每台机器上的状态是一致的，各节点之间通过复制 leader 的日志\(replicated log\)到本机上，再更新本机状态值

> 相同的初始状态 \+ 相同的输入 = 相同的结束状态

从定义上看，它可以分成两部分：

1. 日志的复制
2. 本机的状态变更

难实现的就是日志的复制/同步，再往深了看，它更像是：强一致性\(consistent\)与共识（consensus）算法

> 好像，如果说复制状态机的核心就是实现：强一致性与共识算法，貌似也可以

具体到服务器上：每个服务器存储一个包含一系列指令的日志，并且按顺序执行指令。由于日志都包含相同顺序的指令，状态机会按照相同的顺序执行指令，由于状态机是确定的（deterministic），因此状态机会产生相同的结果。

## 日志复制

一条日志包含 3 个值：指令\+日志 INDEX_ID\+termId

### 复制过程

1. client \(set a = 5\)=\> 集群中的某台机器
   1. 如果该机器就是 leader，就正常处理
   2. 如果该机器碰巧挂了，再重新找一台机器
   3. 如果该机器是 follower，follower 返回 leader 的 IP
2. leader 接收，并在保存在本地日志中，并标记状态为：uncommitted
3. 将该条日志内容，追加到一个心跳包\(AppendEntries RPC\)中，下次心跳：同步给所有 follower 节点
4. 为该次客户端请求，设定定时器，等待所有 follower 响应
   1. 超时\(收到 follower 成功数不过半\)，找到未响应的 follower，无限重试步骤 3，直接这个 follower 返回成功
   2. 未超时，收到了一半节点的响应成功
      1. 执行本条日志的对应的指令，更新状态机
      2. 将本条日志状态变更为：committed
      3. 更新本地变量：committed_id
      4. 将 committed_id\+term_id\+last_logindex 打包到 AppendEntriesRpc 中
      5. 在下一个心跳包\(AppendEntries RPC\)中，通知所有 follower 可以提交了。
   3. 未超时，但是收到的响应是拒绝的。可能在等待收确认包的过程中，自己的网络出现了问题，导致连不上其它机器，而其它机器间是正常的，它们也检测出 ledaer 失去了联系，于是又选出了新的 leader....然后你的网络正常，其它节点告诉新的 leader,你的 term 已经落后，此时旧的 leader 状态变更：leader =\> follower。同时本条未提交成功的日志会被新 leader 覆盖掉。

> 上面这个情况只是最理想且最简单，下面讲一下不一致的情况

> 从这个过程来看：复制状态机应该分成两个部分，一个是真的状态机，另一个达成共识。即：达成共识更像是一次操作的标识或一个锁，标识都没有问题后，再执行该标识对应的指令，去更新状态机里的值。

### 普通情况，机器挂了，日志不一致补齐：

1. follower 挂了后恢复：leader 发现该 follower 进度非常缓慢，可能遗漏了好几个任期。
   1. raft 一致性检查：leader 在发送每个 AppendEntries RPC 会追加两个字段：日志 INDEX_ID\+termId
   2. 此时因为 follower 在本地日志中找不到 leader 发送的 INDEX_ID\+termId,于是会拒绝
   3. leader 发现 follower 拒绝，就再把 INDEX_ID 向前取一个，依此类推
      > 感觉这种寻找日志 offset 的方法挺 2 的，用分段\(任期\)查找或二分查找多快，具说 raft 论文说没必要，失败的情况毕竟是少数，且不会日志落后太多，增加复杂度
2. leader 挂了：正常日志不冲突的时候，跟 1 差不多，逐步定位到日志 index_id 即可，特殊的情况：
   1. leader 接收到一个新请求，并写入本机一条日志，通知 follower 复制，但是此时挂了，但该日志已经写进本机日志了，而 follower 未收到该条指令
   2. 当重新选举出新的 follower 后，leader 再重新启动，就发现日志冲突了
   3. 此时也跟上面差不多，逐条检查到旧 leader 与新 leader 的日志 index_id，然后用新 leader 的数据覆盖掉冲突的日志
3. follower 缓慢\(响应也慢/未响应\)：leader 不断发送重度，让它追加日志，追上进度（此处与回复客户端请求并不冲突）

> 上面这种情况更接近现实，或者说出现的概率大一些，我用它官方的 demo 模拟下了，基本上出现挂机后，日志都是正常能补上的。而下面的情况，是非常极端的情况

### 特殊日志不一致，冲突的情况：

leader 未提交，就挂了，造成，本机日志与新 leader 冲突

假设初始机器 A B C D E ， A 为 leader，此时 A 挂了，B C D E 重新又选举，B 成为了新 leader。

这时 A 又好了，这里所有 follower（以及旧 leader）和新 leader 的日志对比，可能有几种情况：

1. 与新 leader 相同 ，什么都不用做，正常通信即可
2. 比新 leader 略少几个，但数据都相同，通过日志对齐，补上即可
3. 比新 leader 的日志还要多，非多出来的日志都是一样的。
   这种情况可能自己之前是 leader，所以日志是最长的，但是因为故障导致被新 leader 当选。多出来的几条日志：
   1. 可能是未提交状态，让新 leader 覆盖掉：自己多出来的几条日志
   2. 可能是已提交状态，然后挂了，这种情况好像论文 里选择了忽略 ，因为概率 较低 。那其实最后也会被覆盖掉
4. 与新 leader 的日志前半部分差不多，可能后半部分全是错的，也就是冲突。
   这种情况可能自己之前是 leader，然后跟整个网络断了，它依然觉得自己还是 leader，
   但多出来的数据其实都是未提交状态 ，因为它得不到一半以上的响应，等新 leader 进行日志对齐时，把冲突的数据覆盖掉即可。

### 极其特殊日志不一致，冲突的情况：

假设初始机器 A B C D E ， A 为 leader

1. A 挂了，B 成为了新 leader
2. B 短期内又挂了，C 成为了新 leader
3. C 短期又挂了，A 又成为了 leader

这种短期挂了，短期又成为新 leader 的情况，相互成为 leader，如果期间又有 client 不停的请求，肯定会造成，各台机器上的日志完全参差不齐

因为它是强 leader 模式，肯定会造成互相覆盖的情况，具体我也没太懂原理，因为比较极端，不写了

### 日志补齐

从覆盖方式看，它是强 leader 模式，日志的流向只能 ledaer =\> follower , leader 不会调整自己的日志

比如出现日志不一致的情况：就算 leader 是落后的，但依然还会强制让其它 follower 用自己的日志。

### 复制总结

挺神奇的，居然是基于心跳包传输指令

优点：

1. 减少一次 follower 传输\(commit ack\)，
2. 减少了客户端的等待时间
3. 心跳是非实时的，一次可以缓存多条指令

缺点是：

1. 非实时的，可能会出现日志不一致的情况
2. 如果某些 follower 一直不响应，它就一直发，阻塞了 client 的请求

上面遗留个问题：如果说 leader 本地提交 成功，在下个心中包，告知所 follower 也一并提交 ，如果这个时候 挂了，好像官方没给出具体的处理方法

我的理解 ：这种异常本身 概率 小，另外 ，只要它能把消息发出去，大概率 会被一半执行成功。

另外，从冲突来看，是有一定概率造成最后一次心跳丢失最后几条指令的情况，感觉比较量不太大，也能接受。

# 安全

安全：对边界值的处理与优化

网络分隔：

假设现在有 5 台机器：A B C D E

开始的时候：A 为 leader, b c d e 为 follower

这个时候 a b 与 c d e 出现网络分隔的情况，因为这两组不能通信，A 依然认为自己是 leader，B 也依然能收到 A 的消息，证明一切正常。C D E 因为收到 leader 心跳，重新发起选举，假设 C 成为了新的 leader

那么，现在：C D E 是一组新的集群，且是最新，最正确的。A B 是旧的 term 集群，现在考虑：

1. 如果客户端要写入操作，请求 A ，那么 A 收不到另外 C D E 的确认消息，那么写入肯定是失败
2. 如果客户端要读入操作，请求 A 或 B，那么就有问题

# 集群成员变更

配置热更新/动态维护集群节点，无需停止整个集群

有个前提：不太可能同一时间变更集群所有机器的配置

脑裂：同一时间有两个 leader，假设有 A B C 3 台机器，此时增加 D E ，一共 5 台机器了。假设现在更新是不同步的，A B 还是旧配置\(以为是共计 3 个节点\)，C D E 已经切换成为新的配置了\(知道是 5 个节点\)，这个时候如果 leader 挂了，A B 以为还是 3 个节点，它能选出 A 当 leader，C D E 虽然知道是 5 个节点，但是它们的选票过半，会再选出一个 leader，那么就是两个 leader 平行存在了。

解决方法：联合一致状态

还是使用 AppendEntries 消息体

过程：

假设原有 A B C ，A 为 leader ， 增加 D E
raft 会先把 d e 设置为只读状态，然后，开始日志追补，直到追上 leader\-A
假设 D E 追补完成，现任 leader\-A，发起：Cold new
Cold:旧配置
new:新配置
此时 D E 必须满足 Cold new 两个配置信息，做选举都成功了，才算成功

# 时序图

[[分布式-raft.png]]
