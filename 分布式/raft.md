# raft

## 官方演示地址

https://raft.github.io/

> 它这个DEMO挺好玩儿的，用一下，才能更好的理解：什么叫分布式

## 概览

raft：严格来说是一篇论文，它并没有具体的实现，更多的是偏向于理论设计。它解决了：强一致性、共识算法的问题，也就等于：去中心化、高可用的分布式协议。

#### 特点

1. 强领导者:Raft 推出一个 Leader 的角色，同时赋予该角色非常强的领导能力，所有数据的管理都交由该角色。
2. Leader 选举：Raft 使用了一个随机计时器来选举领导者。通过简单地在心跳机制的基础上加上一个随机计时器，避免了大部分同时选举的发生。
3. 成员关系调整：Raft 使用一种共同一致的方法来处理集群成员变换的问题，在这种方法下，处于调整过程中的两种不同的配置集群中大多数机器会有重叠，这就使得集群在成员变换的时候依然可以继续工作。

#### 结合paxos

1. 它的角色更少，更专注
2. 通过过程 更简单，通信次数较少

## 术语

#### 任期\(term\)：每个leader从竞选开始\-\>成功到最后放弃\(宕机\)，被称为一个term任期，也可以理解为一次leader的执政周期

|模块名         |说明                |
|---------------|--------------------|
|leader         |选举过程            |
|log replication|日志复制过程        |
|safety         |各种异常、安全等考虑|

|角色名   |说明                                                                     |
|---------|-------------------------------------------------------------------------|
|leader   |一台节点，通过选举产生，用于接收client的IO操作，同时分发到所有follwer节点|
|follower |一台节点，跟属leader领导                                                 |
|candidate|一台节点，当收不到leader心跳后，开启选举状态                             |

#### 通信RPC

1. RequestVote 消息体:各节点均可发起，由candidate选举期间发起

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

1. AppendEntries 消息体:只能由leader发起，日志追回请求，包括：执行日志复制、心跳、日志对齐

```
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

## leader election 选举

先看下它的官方图:

> 定时器的超时时间：随机\(150~300ms\)，具体也可以配置
> 
> 
> 这个还得具体分析，一次广播到落盘大概是5~20ms，它官方给的最大间隔是:5~500ms

1. 所有机器首次启动，均为follower状态
2. 创建心跳定时器\(时间为随时\)，如在 timer 周期内未收到任何\(leader \+ candidate \)消息，开启选举过程：
    1. 状态变更： follower =\> candidate
    2. 读取本地属性： currentTermNumber，然后： term\+1
    3. 读取本地日志最大索引号：last\_index\_id
    4. 设置：自己的ID为被投票者（间接看：自己投自己一票 ）
    5. 将：currentTermNumber last\_index\_id term\+1 candidateId ，打包RequestVoteReq，发送：RequestVote RPC 消息
    6. 创建定时器\(时间为随时\)，等待其它节点消息返回
3. 定时器，等待期间可能会发生如下事情：
    1. 在timer时间内，收到其它节点返回的选票，其中选自己的票数：过半，那么证明自己就是leader。
        1. 发消息:AppendEntries RPC, 给其它所有 clientNode ，告知我是leader了
        2. 将状态由 candidate =\> leader
    2. timer超时，状态不变，再设定一个新timer\(随机时间\)，给本地term\+1，继续发送AppendEntries RPC给其它节点，开启新竞选
        1. 可能是有些节点的响应过慢没收着
        2. 所有节点都投票了，但是自己的票数不到一半
    3. 在timer时间内，收到其它机器竞选成功的：AppendEntries RPC，将candidate =\> follower，删除旧timer，创建新的timer
4. 上面3步的前提：大家同时设定的心跳timer，某一台机器先超时了，它就最先开启了选举，而其它机器的状态可能，如下：
    1. follower，因为别的机器先超时，肯定会先发竞选消息，自己被动接收，那么，刷新自己的timer ，也就是follower尽量维持follower，不参选
        1. 如果接收的消息的term 比较自己小，直接丢掉
        2. 如果接收的消息的term 比较自己大，且本轮term还未投过票，先更新本地term，投票给它
        3. 如果接收的消息的term 跟自己相同，先来先得，后来拒绝
    2. candidate，上面有一台觉醒，但同时还有一台机器也觉醒了，两个人同时发起竞选。两个人会同时收到candidate竞选的消息\(当确定已经参选，就证明肯定给自己投票了\)
        1. 如果接收的消息的term
            1. 对方的term相同，直接回绝\(这个是两个随时时间差不多，就比网速吧，先到先得，也可能出现分票\)
            2. 对方的term更大，那么自己 candidate =\> follower\(自己在上一个任期整合都是挂了的状态，碰巧刚好，就参与了选票\)
            3. 对方的term更小，直接回绝\(这种是对面在整个任期都挂了，正好新任期它又好了，所以少了N个任期\)
        2. 分票：两人收到集群内所有的投票各自都是一半，那么选举失败，timer超时，再定个timer，重复投票，因为timer是随机的，最终肯定会有一个先到期
5. 特殊情况，假如 A B C D E 正常组成集群，A 为leader，此时网络故障，A B 是原状态，C D E 组成新的集群，C 为新leader，然后故障好了，此时
    1. C 会给A 发送心跳，A接收到心跳，发现对方的term更大，此时:leader =\> follower ，
    2. C 会给B 发送心跳，B更新本地term值，变更leader指向为C
    3. C D E 同时也会收到A的心中，但发现term值过小，直接拒绝
6. 特殊情况，如果网络环境特别差：RTT \> 300MS ，那么就会无限选举。因为每个candidate最大超时是300MS，它发出去一条到收一条消息的RTT\>300ms，自己早就超时了，已经开启下一个任期遥选举了，不过是比较极端情况，不太可能发生，也可以根据自己机器的网络RTT时间，放大超时随时值
7. 特殊情况，假设一共5台机器，还剩下B C D 是正常的，现在D是leader，然后挂了，这时就只剩下B C 两台机器，他们无论怎么选最多只能得到2票，并没有到过半的3票，所以会无限循环选举下去。
8. 特殊情况，假设一共5台机器，还剩下B C D 是正常的，现在D是leader，然后B或C挂了，此时集群还是能正常收消息，但是却无法确认，也就是消息在D C或B 都存了，但收不到一半集群的'消息确认'

> 整个选举机制核心：两个随机时间的timer解决了一切，不会死锁，又能选出唯一的leader,确实巧妙

#### 分析一下RequestVoteReq消息体

有两个字段需要注意下：last\_log\_index 和 last\_log\_term

1. 假设当前leader为A，在任期：1 期间，有一条日志已写入
2. 通知其它节点，进行复制操作
3. 收到半数节点复制成功的消息，开始本地提交，然后，通知所有节点进行提交
4. 假设，在这期间，某一台follower\-B，也在任期：1，但因为网络不太好，复制进度较慢，未收到该条日志复制的请求，也未收日志提交的请求
5. 假设此时 leader\-A挂了，B是新的leader，它的日志就是落后的，也没有上面A中的那条日志
6. 如果根据强leader覆盖法则，那么原A的日志多出的几条日志就被新leader\-B给覆盖掉

> 上面的问题核心是：某个进度落后的follower虽然进度慢缺失了某几条消息，但是在整个任期来看，它没有全丢失，当leader挂了，它依然还能参选并成为新leader，因为：他的iterm并没有落后 ，而如果只拿item判断是否可当ledaer就会有这样的总是，会把之前进度快的节点的消息给覆盖掉。

所以引入了上面两个字段值：

1. 进度较慢的 candidate，发出RequestVoteReq消息，其中包含了：candidate的日志进度 last\_log\_index last\_log\_term
2. follower在接收后，与本机的 last\_log\_index last\_log\_term 进行对比，如果term相同，再比较last\_log\_index，发现自己的更新，直接回绝该candidate
3. 如果某台follower进度很新，但苏醒较慢，可以用这个功能先概率性的过虑掉进度落后的candidate，为自己争取时间参选

那么加两个字段的核心是：当原leader挂了后，尽量选出日志最长的那个 candidate

但这里还有个问题，两个值是一起判定还是有顺序的？

1. 先判断 last\_log\_term
2. 再判断 last\_log\_index
    所以，这两个值更倾向解决，在同一任期的前提下，看哪个日志更长，而如果这个时候，有的日志虽然较短，但是可能任期号更大，那它也完全可以成为leader的，那么引出另外一个问题：
    新leader的term\+1了，任期就更新了，但是本机上可能还有旧任期内未提交的日志，那新任期是否提前旧任期的日志呢？
    答案:不会
    因为有一个风险：如果提交旧任期的未提交的数据，那么此时它挂了，另外一个新机器当选新leader，它的数据是缺失了，那么它可能会覆盖掉刚刚提交的旧任期的数据，这个挺可怕的，或者说，raft不能接受覆盖已提交的数据

但如果有新的写入操作，会连带着旧的一起提交 commit\_id

这个问题的核心是：集群\-提交确认，即：leader提交后，要立刻发送给所有follower，并阻塞，等待半数follwer回复：提交OK后，再回复client本次日志写入成功。不过raft没提，估计可能raft觉得没必要考虑这种场景吧。另外，就算加这个确认机制，但也会非常麻烦，还得考虑各种断网的情况，好像 google percolator模型有解决。

每个节点必须得有3个属性：

currentTerm：节点当前所处的任期。

votedFor 　：节点当前任期所跟随的其他节点的 ID。如果是投票阶段，表示节点所投的候选人。如果已经竞选结束，就是当前的领导人。

currentLogIndex:本机日志索引号

> 基本上日常的操作流程就是这3个属性值\+状态值\+消息传输，基本上够用了。raft的实现确实很简单。

## 复制状态机

Replicated state machines：保证每台机器上的状态是一致的，各节点之间通过复制leader的日志\(replicated log\)到本机上，再更新本机状态值

> 相同的初始状态 \+ 相同的输入 = 相同的结束状态

从定义上看，它可以分成两部分：

1. 日志的复制
2. 本机的状态变更

难实现的就是日志的复制/同步，再往深了看，它更像是：强一致性\(consistent\)与共识（consensus）算法

> 好像，如果说复制状态机的核心就是实现：强一致性与共识算法，貌似也可以

具体到服务器上：每个服务器存储一个包含一系列指令的日志，并且按顺序执行指令。由于日志都包含相同顺序的指令，状态机会按照相同的顺序执行指令，由于状态机是确定的（deterministic），因此状态机会产生相同的结果。

## 日志复制

一条日志包含3个值：指令\+日志INDEX\_ID\+termId

#### 复制过程

1. client \(set a = 5\)=\> 集群中的某台机器
    1. 如果该机器就是leader，就正常处理
    2. 如果该机器碰巧挂了，再重新找一台机器
    3. 如果该机器是follower，follower返回leader的IP
2. leader接收，并在保存在本地日志中，并标记状态为：uncommitted
3. 将该条日志内容，追加到一个心跳包\(AppendEntries RPC\)中，下次心跳：同步给所有follower节点
4. 为该次客户端请求，设定定时器，等待所有follower响应
    1. 超时\(收到follower成功数不过半\)，找到未响应的follower，无限重试步骤3，直接这个follower返回成功
    2. 未超时，收到了一半节点的响应成功
        1. 执行本条日志的对应的指令，更新状态机
        2. 将本条日志状态变更为：committed
        3. 更新本地变量：committed\_id
        4. 将committed\_id\+term\_id\+last\_logindex打包到AppendEntriesRpc中
        5. 在下一个心跳包\(AppendEntries RPC\)中，通知所有follower可以提交了。
    3. 未超时，但是收到的响应是拒绝的。可能在等待收确认包的过程中，自己的网络出现了问题，导致连不上其它机器，而其它机器间是正常的，它们也检测出ledaer失去了联系，于是又选出了新的leader....然后你的网络正常，其它节点告诉新的leader,你的term已经落后，此时旧的leader 状态变更：leader =\> follower。同时本条未提交成功的日志会被新leader覆盖掉。

> 上面这个情况只是最理想且最简单，下面讲一下不一致的情况

> 从这个过程来看：复制状态机应该分成两个部分，一个是真的状态机，另一个达成共识。即：达成共识更像是一次操作的标识或一个锁，标识都没有问题后，再执行该标识对应的指令，去更新状态机里的值。

#### 普通情况，机器挂了，日志不一致补齐：

1. follower 挂了后恢复：leader发现该follower进度非常缓慢，可能遗漏了好几个任期。
    1. raft一致性检查：leader在发送每个AppendEntries RPC 会追加两个字段：日志INDEX\_ID\+termId
    2. 此时因为follower在本地日志中找不到leader发送的INDEX\_ID\+termId,于是会拒绝
    3. leader发现follower拒绝，就再把INDEX\_ID向前取一个，依此类推
    > 感觉这种寻找日志offset的方法挺2的，用分段\(任期\)查找或二分查找多快，具说raft论文说没必要，失败的情况毕竟是少数，且不会日志落后太多，增加复杂度
2. leader 挂了：正常日志不冲突的时候，跟1差不多，逐步定位到日志index\_id即可，特殊的情况：
    1. leader 接收到一个新请求，并写入本机一条日志，通知follower复制，但是此时挂了，但该日志已经写进本机日志了，而follower未收到该条指令
    2. 当重新选举出新的follower后，leader再重新启动，就发现日志冲突了
    3. 此时也跟上面差不多，逐条检查到旧leader与新leader的日志index\_id，然后用新leader的数据覆盖掉冲突的日志
3. follower 缓慢\(响应也慢/未响应\)：leader 不断发送重度，让它追加日志，追上进度（此处与回复客户端请求并不冲突）

> 上面这种情况更接近现实，或者说出现的概率大一些，我用它官方的demo模拟下了，基本上出现挂机后，日志都是正常能补上的。而下面的情况，是非常极端的情况

#### 特殊日志不一致，冲突的情况：

leader 未提交，就挂了，造成，本机日志与新leader冲突

假设初始机器 A B C D E ， A为leader，此时A挂了，B C D E 重新又选举，B成为了新leader。

这时A又好了，这里所有follower（以及旧leader）和新leader的日志对比，可能有几种情况：

1. 与新leader相同 ，什么都不用做，正常通信即可
2. 比新leader略少几个，但数据都相同，通过日志对齐，补上即可
3. 比新leader的日志还要多，非多出来的日志都是一样的。
    这种情况可能自己之前是leader，所以日志是最长的，但是因为故障导致被新leader当选。多出来的几条日志：
    1. 可能是未提交状态，让新leader覆盖掉：自己多出来的几条日志
    2. 可能是已提交状态，然后挂了，这种情况好像论文 里选择了忽略 ，因为概率 较低 。那其实最后也会被覆盖掉
4. 与新leader的日志前半部分差不多，可能后半部分全是错的，也就是冲突。
    这种情况可能自己之前是leader，然后跟整个网络断了，它依然觉得自己还是leader，
    但多出来的数据其实都是未提交状态 ，因为它得不到一半以上的响应，等新leader进行日志对齐时，把冲突的数据覆盖掉即可。

#### 极其特殊日志不一致，冲突的情况：

假设初始机器 A B C D E ， A为leader

1. A挂了，B成为了新leader
2. B短期内又挂了，C成为了新leader
3. C短期又挂了，A又成为了leader

这种短期挂了，短期又成为新leader的情况，相互成为leader，如果期间又有client不停的请求，肯定会造成，各台机器上的日志完全参差不齐

因为它是强leader模式，肯定会造成互相覆盖的情况，具体我也没太懂原理，因为比较极端，不写了

#### 日志补齐

从覆盖方式看，它是强leader模式，日志的流向只能 ledaer =\> follower , leader 不会调整自己的日志

比如出现日志不一致的情况：就算leader是落后的，但依然还会强制让其它follower用自己的日志。

#### 复制总结

挺神奇的，居然是基于心跳包传输指令

优点：

1. 减少一次follower传输\(commit ack\)，
2. 减少了客户端的等待时间
3. 心跳是非实时的，一次可以缓存多条指令

缺点是：

1. 非实时的，可能会出现日志不一致的情况
2. 如果某些follower一直不响应，它就一直发，阻塞了client的请求

上面遗留个问题：如果说leader本地提交 成功，在下个心中包，告知所follower也一并提交 ，如果这个时候 挂了，好像官方没给出具体的处理方法

我的理解 ：这种异常本身 概率 小，另外 ，只要它能把消息发出去，大概率 会被一半执行成功。

另外，从冲突来看，是有一定概率造成最后一次心跳丢失最后几条指令的情况，感觉比较量不太大，也能接受。

## 安全

安全：对边界值的处理与优化

网络分隔：

假设现在有5台机器：A B C D E

开始的时候：A为leader, b c d e 为follower

这个时候a b 与 c d e 出现网络分隔的情况，因为这两组不能通信，A依然认为自己是leader，B也依然能收到A的消息，证明一切正常。C D E 因为收到leader心跳，重新发起选举，假设C成为了新的leader

那么，现在：C D E 是一组新的集群，且是最新，最正确的。A B 是旧的term集群，现在考虑：

1. 如果客户端要写入操作，请求A ，那么A 收不到另外C D E 的确认消息，那么写入肯定是失败
2. 如果客户端要读入操作，请求A 或 B，那么就有问题

## 集群成员变更

配置热更新/动态维护集群节点，无需停止整个集群

有个前提：不太可能同一时间变更集群所有机器的配置

脑裂：同一时间有两个leader，假设有A B C 3台机器，此时增加 D E ，一共5台机器了。假设现在更新是不同步的，A B 还是旧配置\(以为是共计3个节点\)，C D E 已经切换成为新的配置了\(知道是5个节点\)，这个时候如果leader挂了，A B 以为还是3个节点，它能选出A当leader，C D E 虽然知道是5个节点，但是它们的选票过半，会再选出一个leader，那么就是两个leader平行存在了。

解决方法：联合一致状态

还是使用 AppendEntries 消息体

过程：

假设原有 A B C ，A 为leader ， 增加 D E

raft 会先把d e 设置为只读状态，然后，开始日志追补，直到追上leader\-A

假设D E 追补完成，现任leader\-A，发起：Cold new

Cold:旧配置

new:新配置

此时D E 必须满足Cold new 两个配置信息，做选举都成功了，才算成功

%E5%AE%98%E6%96%B9%E6%BC%94%E7%A4%BA%E5%9C%B0%E5%9D%80%0A\-\-\-\-%0Ahttps%3A%2F%2Fraft.github.io%2F%0A%3E%E5%AE%83%E8%BF%99%E4%B8%AADEMO%E6%8C%BA%E5%A5%BD%E7%8E%A9%E5%84%BF%E7%9A%84%EF%BC%8C%E7%94%A8%E4%B8%80%E4%B8%8B%EF%BC%8C%E6%89%8D%E8%83%BD%E6%9B%B4%E5%A5%BD%E7%9A%84%E7%90%86%E8%A7%A3%EF%BC%9A%E4%BB%80%E4%B9%88%E5%8F%AB%E5%88%86%E5%B8%83%E5%BC%8F%0A%0A%E6%A6%82%E8%A7%88%0A\-\-\-%0Araft%EF%BC%9A%E4%B8%A5%E6%A0%BC%E6%9D%A5%E8%AF%B4%E6%98%AF%E4%B8%80%E7%AF%87%E8%AE%BA%E6%96%87%EF%BC%8C%E5%AE%83%E5%B9%B6%E6%B2%A1%E6%9C%89%E5%85%B7%E4%BD%93%E7%9A%84%E5%AE%9E%E7%8E%B0%EF%BC%8C%E6%9B%B4%E5%A4%9A%E7%9A%84%E6%98%AF%E5%81%8F%E5%90%91%E4%BA%8E%E7%90%86%E8%AE%BA%E8%AE%BE%E8%AE%A1%E3%80%82%E5%AE%83%E8%A7%A3%E5%86%B3%E4%BA%86%EF%BC%9A%E5%BC%BA%E4%B8%80%E8%87%B4%E6%80%A7%E3%80%81%E5%85%B1%E8%AF%86%E7%AE%97%E6%B3%95%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%8C%E4%B9%9F%E5%B0%B1%E7%AD%89%E4%BA%8E%EF%BC%9A%E5%8E%BB%E4%B8%AD%E5%BF%83%E5%8C%96%E3%80%81%E9%AB%98%E5%8F%AF%E7%94%A8%E7%9A%84%E5%88%86%E5%B8%83%E5%BC%8F%E5%8D%8F%E8%AE%AE%E3%80%82%0A%0A%23%23%23%23%20%E7%89%B9%E7%82%B9%0A1.%20%E5%BC%BA%E9%A2%86%E5%AF%BC%E8%80%85%3ARaft%20%E6%8E%A8%E5%87%BA%E4%B8%80%E4%B8%AA%20Leader%20%E7%9A%84%E8%A7%92%E8%89%B2%EF%BC%8C%E5%90%8C%E6%97%B6%E8%B5%8B%E4%BA%88%E8%AF%A5%E8%A7%92%E8%89%B2%E9%9D%9E%E5%B8%B8%E5%BC%BA%E7%9A%84%E9%A2%86%E5%AF%BC%E8%83%BD%E5%8A%9B%EF%BC%8C%E6%89%80%E6%9C%89%E6%95%B0%E6%8D%AE%E7%9A%84%E7%AE%A1%E7%90%86%E9%83%BD%E4%BA%A4%E7%94%B1%E8%AF%A5%E8%A7%92%E8%89%B2%E3%80%82%0A2.%20Leader%20%E9%80%89%E4%B8%BE%EF%BC%9ARaft%20%E4%BD%BF%E7%94%A8%E4%BA%86%E4%B8%80%E4%B8%AA%E9%9A%8F%E6%9C%BA%E8%AE%A1%E6%97%B6%E5%99%A8%E6%9D%A5%E9%80%89%E4%B8%BE%E9%A2%86%E5%AF%BC%E8%80%85%E3%80%82%E9%80%9A%E8%BF%87%E7%AE%80%E5%8D%95%E5%9C%B0%E5%9C%A8%E5%BF%83%E8%B7%B3%E6%9C%BA%E5%88%B6%E7%9A%84%E5%9F%BA%E7%A1%80%E4%B8%8A%E5%8A%A0%E4%B8%8A%E4%B8%80%E4%B8%AA%E9%9A%8F%E6%9C%BA%E8%AE%A1%E6%97%B6%E5%99%A8%EF%BC%8C%E9%81%BF%E5%85%8D%E4%BA%86%E5%A4%A7%E9%83%A8%E5%88%86%E5%90%8C%E6%97%B6%E9%80%89%E4%B8%BE%E7%9A%84%E5%8F%91%E7%94%9F%E3%80%82%0A3.%20%E6%88%90%E5%91%98%E5%85%B3%E7%B3%BB%E8%B0%83%E6%95%B4%EF%BC%9ARaft%20%E4%BD%BF%E7%94%A8%E4%B8%80%E7%A7%8D%E5%85%B1%E5%90%8C%E4%B8%80%E8%87%B4%E7%9A%84%E6%96%B9%E6%B3%95%E6%9D%A5%E5%A4%84%E7%90%86%E9%9B%86%E7%BE%A4%E6%88%90%E5%91%98%E5%8F%98%E6%8D%A2%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%8C%E5%9C%A8%E8%BF%99%E7%A7%8D%E6%96%B9%E6%B3%95%E4%B8%8B%EF%BC%8C%E5%A4%84%E4%BA%8E%E8%B0%83%E6%95%B4%E8%BF%87%E7%A8%8B%E4%B8%AD%E7%9A%84%E4%B8%A4%E7%A7%8D%E4%B8%8D%E5%90%8C%E7%9A%84%E9%85%8D%E7%BD%AE%E9%9B%86%E7%BE%A4%E4%B8%AD%E5%A4%A7%E5%A4%9A%E6%95%B0%E6%9C%BA%E5%99%A8%E4%BC%9A%E6%9C%89%E9%87%8D%E5%8F%A0%EF%BC%8C%E8%BF%99%E5%B0%B1%E4%BD%BF%E5%BE%97%E9%9B%86%E7%BE%A4%E5%9C%A8%E6%88%90%E5%91%98%E5%8F%98%E6%8D%A2%E7%9A%84%E6%97%B6%E5%80%99%E4%BE%9D%E7%84%B6%E5%8F%AF%E4%BB%A5%E7%BB%A7%E7%BB%AD%E5%B7%A5%E4%BD%9C%E3%80%82%0A%0A%0A%23%23%23%23%20%E7%BB%93%E5%90%88paxos%0A1.%20%E5%AE%83%E7%9A%84%E8%A7%92%E8%89%B2%E6%9B%B4%E5%B0%91%EF%BC%8C%E6%9B%B4%E4%B8%93%E6%B3%A8%0A2.%20%E9%80%9A%E8%BF%87%E8%BF%87%E7%A8%8B%20%E6%9B%B4%E7%AE%80%E5%8D%95%EF%BC%8C%E9%80%9A%E4%BF%A1%E6%AC%A1%E6%95%B0%E8%BE%83%E5%B0%91%0A%0A%E6%9C%AF%E8%AF%AD%0A\-\-\-\-\-%0A%23%23%23%23%20%E4%BB%BB%E6%9C%9F\(term\)%EF%BC%9A%E6%AF%8F%E4%B8%AAleader%E4%BB%8E%E7%AB%9E%E9%80%89%E5%BC%80%E5%A7%8B\-%3E%E6%88%90%E5%8A%9F\-%3E%E6%9C%80%E5%90%8E%E6%94%BE%E5%BC%83\(%E5%AE%95%E6%9C%BA\)%EF%BC%8C%E8%A2%AB%E7%A7%B0%E4%B8%BA%E4%B8%80%E4%B8%AAterm%E4%BB%BB%E6%9C%9F%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%90%86%E8%A7%A3%E4%B8%BA%E4%B8%80%E6%AC%A1leader%E7%9A%84%E6%89%A7%E6%94%BF%E5%91%A8%E6%9C%9F%0A%0A%7C%20%E6%A8%A1%E5%9D%97%E5%90%8D%20%7C%E8%AF%B4%E6%98%8E%20%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20leader%20%7C%20%E9%80%89%E4%B8%BE%E8%BF%87%E7%A8%8B%20%7C%0A%7C%20log%20replication%20%7C%20%E6%97%A5%E5%BF%97%E5%A4%8D%E5%88%B6%E8%BF%87%E7%A8%8B%20%7C%0A%7C%20safety%20%7C%20%E5%90%84%E7%A7%8D%E5%BC%82%E5%B8%B8%E3%80%81%E5%AE%89%E5%85%A8%E7%AD%89%E8%80%83%E8%99%91%20%7C%0A%0A%0A%7C%20%E8%A7%92%E8%89%B2%E5%90%8D%20%7C%20%E8%AF%B4%E6%98%8E%20%20%7C%0A%7C%20\-\-\-%20%7C%20\-\-\-%20%7C%0A%7C%20leader%20%7C%20%E4%B8%80%E5%8F%B0%E8%8A%82%E7%82%B9%EF%BC%8C%E9%80%9A%E8%BF%87%E9%80%89%E4%B8%BE%E4%BA%A7%E7%94%9F%EF%BC%8C%E7%94%A8%E4%BA%8E%E6%8E%A5%E6%94%B6client%E7%9A%84IO%E6%93%8D%E4%BD%9C%EF%BC%8C%E5%90%8C%E6%97%B6%E5%88%86%E5%8F%91%E5%88%B0%E6%89%80%E6%9C%89follwer%E8%8A%82%E7%82%B9%20%7C%0A%7C%20follower%7C%20%E4%B8%80%E5%8F%B0%E8%8A%82%E7%82%B9%EF%BC%8C%E8%B7%9F%E5%B1%9Eleader%E9%A2%86%E5%AF%BC%20%7C%0A%7C%20candidate%20%7C%20%E4%B8%80%E5%8F%B0%E8%8A%82%E7%82%B9%EF%BC%8C%E5%BD%93%E6%94%B6%E4%B8%8D%E5%88%B0leader%E5%BF%83%E8%B7%B3%E5%90%8E%EF%BC%8C%E5%BC%80%E5%90%AF%E9%80%89%E4%B8%BE%E7%8A%B6%E6%80%81%20%7C%0A%0A%23%23%23%23%20%E9%80%9A%E4%BF%A1RPC%0A1.%20RequestVote%20%E6%B6%88%E6%81%AF%E4%BD%93%3A%E5%90%84%E8%8A%82%E7%82%B9%E5%9D%87%E5%8F%AF%E5%8F%91%E8%B5%B7%EF%BC%8C%E7%94%B1candidate%E9%80%89%E4%B8%BE%E6%9C%9F%E9%97%B4%E5%8F%91%E8%B5%B7%0A%60%60%60%0Atype%20RequestVoteReq%20struct%7B%0A%20%20%20%20term%20int%20%2F%2F%E8%87%AA%E5%B7%B1%E5%BD%93%E5%89%8D%E4%BB%BB%E6%9C%9F%E5%8F%B7%0A%20%20%20%20candidateld%20int%20%2F%2F%E8%87%AA%E5%B7%B1%E7%9A%84ID%0A%20%20%20%20last\_log\_index%20%2F%2F%E8%87%AA%E5%B7%B1%E6%9C%80%E5%90%8E%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97ID%0A%20%20%20%20last\_log\_term%20%2F%2F%E8%87%AA%E5%B7%B1%E6%9C%80%E5%90%8E%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97%E7%9A%84%E4%BB%BB%E6%9C%9F%0A%7D%0A%0Atype%20RequestVoteRes%20struct%7B%0A%20%20%20%20term%20int%20%2F%2F%E8%87%AA%E5%B7%B1%E5%BD%93%E5%89%8D%E4%BB%BB%E6%9C%9F%E5%8F%B7%0A%20%20%20%20voteGranted%20bool%20%2F%2F%E8%87%AA%E5%B7%B1%E4%BC%9A%E4%B8%8D%E4%BC%9A%E6%8A%95%E7%A5%A8%E7%BB%99%E8%BF%99%E4%B8%AAcandidate%0A%7D%0A%60%60%60%0A2.%20AppendEntries%20%E6%B6%88%E6%81%AF%E4%BD%93%3A%E5%8F%AA%E8%83%BD%E7%94%B1leader%E5%8F%91%E8%B5%B7%EF%BC%8C%E6%97%A5%E5%BF%97%E8%BF%BD%E5%9B%9E%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%8C%85%E6%8B%AC%EF%BC%9A%E6%89%A7%E8%A1%8C%E6%97%A5%E5%BF%97%E5%A4%8D%E5%88%B6%E3%80%81%E5%BF%83%E8%B7%B3%E3%80%81%E6%97%A5%E5%BF%97%E5%AF%B9%E9%BD%90%0A%60%60%60%0Atype%20AppendEntriesReq%20struct%20%7B%0A%09Term%20%20%20%20%20%20%20%20%20int%20%20%20%20%2F%2F%E5%BD%93%E5%89%8D%E4%BB%BB%E6%9C%9F%E5%8F%B7%0A%09LeaderId%20%20%20%20%20int%20%20%20%20%2F%2F%E8%87%AA%E5%B7%B1%E7%9A%84leaderId%0A%09PrevLogIndex%20int%20%20%20%20%2F%2F%E5%89%8D%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97%E5%8F%B7%0A%09PrevLogTerm%20%20int%20%20%20%20%2F%2F%E5%89%8D%E4%B8%80%E4%B8%AA%E4%BB%BB%E6%9C%9F%0A%09Entries%20%20%20%20%20%20%5B%5Dbyte%20%2F%2F%E5%85%B7%E4%BD%93%E6%97%A5%E5%BF%97%E5%86%85%E5%AE%B9%EF%BC%8C%E5%A6%82%EF%BC%9A%E6%8C%87%E4%BB%A4%E9%9B%86%0A%09LeaderCommit%20int%20%20%20%20%2F%2Fleader%E6%8F%90%E4%BA%A4%E6%97%A5%E5%BF%97%E5%8F%B7%0A%7D%0A%0Atype%20AppendEntriesRes%20struct%20%7B%0A%09Term%20%20%20%20int%20%20%2F%2F%E8%87%AA%E5%B7%B1%E5%BD%93%E5%89%8D%E4%BB%BB%E6%9C%9F%E5%8F%B7%0A%09success%20bool%20%2F%2F%E5%A6%82%E6%9E%9Cfollower%E5%8C%85%E5%90%AB%E5%89%8D%E4%B8%80%E4%B8%AA%E6%97%A5%E5%BF%97%E5%8F%B7%EF%BC%8C%E8%BF%94%E5%9B%9Etrue%0A%7D%0A%0A%60%60%60%0A%0A%0A%0A%0Aleader%20election%20%E9%80%89%E4%B8%BE%0A\-\-\-\-%0A%E5%85%88%E7%9C%8B%E4%B8%8B%E5%AE%83%E7%9A%84%E5%AE%98%E6%96%B9%E5%9B%BE%3A%0A%0A%3E%E5%AE%9A%E6%97%B6%E5%99%A8%E7%9A%84%E8%B6%85%E6%97%B6%E6%97%B6%E9%97%B4%EF%BC%9A%E9%9A%8F%E6%9C%BA\(150~300ms\)%EF%BC%8C%E5%85%B7%E4%BD%93%E4%B9%9F%E5%8F%AF%E4%BB%A5%E9%85%8D%E7%BD%AE%0A%3E%E8%BF%99%E4%B8%AA%E8%BF%98%E5%BE%97%E5%85%B7%E4%BD%93%E5%88%86%E6%9E%90%EF%BC%8C%E4%B8%80%E6%AC%A1%E5%B9%BF%E6%92%AD%E5%88%B0%E8%90%BD%E7%9B%98%E5%A4%A7%E6%A6%82%E6%98%AF5~20ms%EF%BC%8C%E5%AE%83%E5%AE%98%E6%96%B9%E7%BB%99%E7%9A%84%E6%9C%80%E5%A4%A7%E9%97%B4%E9%9A%94%E6%98%AF%3A5~500ms%0A%0A%0A1.%20%E6%89%80%E6%9C%89%E6%9C%BA%E5%99%A8%E9%A6%96%E6%AC%A1%E5%90%AF%E5%8A%A8%EF%BC%8C%E5%9D%87%E4%B8%BAfollower%E7%8A%B6%E6%80%81%0A2.%20%E5%88%9B%E5%BB%BA%E5%BF%83%E8%B7%B3%E5%AE%9A%E6%97%B6%E5%99%A8\(%E6%97%B6%E9%97%B4%E4%B8%BA%E9%9A%8F%E6%97%B6\)%EF%BC%8C%E5%A6%82%E5%9C%A8%20timer%20%E5%91%A8%E6%9C%9F%E5%86%85%E6%9C%AA%E6%94%B6%E5%88%B0%E4%BB%BB%E4%BD%95\(leader%20%2B%20candidate%20\)%E6%B6%88%E6%81%AF%EF%BC%8C%E5%BC%80%E5%90%AF%E9%80%89%E4%B8%BE%E8%BF%87%E7%A8%8B%EF%BC%9A%0A%20%20%20%201.%20%E7%8A%B6%E6%80%81%E5%8F%98%E6%9B%B4%EF%BC%9A%20follower%20%3D%3E%20candidate%0A%20%20%20%202.%20%E8%AF%BB%E5%8F%96%E6%9C%AC%E5%9C%B0%E5%B1%9E%E6%80%A7%EF%BC%9A%20currentTermNumber%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%9A%20term%2B1%0A%20%20%20%203.%20%E8%AF%BB%E5%8F%96%E6%9C%AC%E5%9C%B0%E6%97%A5%E5%BF%97%E6%9C%80%E5%A4%A7%E7%B4%A2%E5%BC%95%E5%8F%B7%EF%BC%9Alast\_index\_id%0A%20%20%20%204.%20%E8%AE%BE%E7%BD%AE%EF%BC%9A%E8%87%AA%E5%B7%B1%E7%9A%84ID%E4%B8%BA%E8%A2%AB%E6%8A%95%E7%A5%A8%E8%80%85%EF%BC%88%E9%97%B4%E6%8E%A5%E7%9C%8B%EF%BC%9A%E8%87%AA%E5%B7%B1%E6%8A%95%E8%87%AA%E5%B7%B1%E4%B8%80%E7%A5%A8%20%EF%BC%89%0A%20%20%20%203.%20%E5%B0%86%EF%BC%9AcurrentTermNumber%20last\_index\_id%20term%2B1%20candidateId%20%EF%BC%8C%E6%89%93%E5%8C%85RequestVoteReq%EF%BC%8C%E5%8F%91%E9%80%81%EF%BC%9ARequestVote%20RPC%20%E6%B6%88%E6%81%AF%0A%20%20%20%204.%20%E5%88%9B%E5%BB%BA%E5%AE%9A%E6%97%B6%E5%99%A8\(%E6%97%B6%E9%97%B4%E4%B8%BA%E9%9A%8F%E6%97%B6\)%EF%BC%8C%E7%AD%89%E5%BE%85%E5%85%B6%E5%AE%83%E8%8A%82%E7%82%B9%E6%B6%88%E6%81%AF%E8%BF%94%E5%9B%9E%0A3.%20%E5%AE%9A%E6%97%B6%E5%99%A8%EF%BC%8C%E7%AD%89%E5%BE%85%E6%9C%9F%E9%97%B4%E5%8F%AF%E8%83%BD%E4%BC%9A%E5%8F%91%E7%94%9F%E5%A6%82%E4%B8%8B%E4%BA%8B%E6%83%85%EF%BC%9A%0A%20%20%20%201.%20%E5%9C%A8timer%E6%97%B6%E9%97%B4%E5%86%85%EF%BC%8C%E6%94%B6%E5%88%B0%E5%85%B6%E5%AE%83%E8%8A%82%E7%82%B9%E8%BF%94%E5%9B%9E%E7%9A%84%E9%80%89%E7%A5%A8%EF%BC%8C%E5%85%B6%E4%B8%AD%E9%80%89%E8%87%AA%E5%B7%B1%E7%9A%84%E7%A5%A8%E6%95%B0%EF%BC%9A%E8%BF%87%E5%8D%8A%EF%BC%8C%E9%82%A3%E4%B9%88%E8%AF%81%E6%98%8E%E8%87%AA%E5%B7%B1%E5%B0%B1%E6%98%AFleader%E3%80%82%0A%20%20%20%20%20%20%20%201.%20%E5%8F%91%E6%B6%88%E6%81%AF%3AAppendEntries%20RPC%2C%20%E7%BB%99%E5%85%B6%E5%AE%83%E6%89%80%E6%9C%89%20clientNode%20%EF%BC%8C%E5%91%8A%E7%9F%A5%E6%88%91%E6%98%AFleader%E4%BA%86%0A%20%20%20%20%20%20%20%202.%20%E5%B0%86%E7%8A%B6%E6%80%81%E7%94%B1%20candidate%20%3D%3E%20leader%0A%20%20%20%202.%20timer%E8%B6%85%E6%97%B6%EF%BC%8C%E7%8A%B6%E6%80%81%E4%B8%8D%E5%8F%98%EF%BC%8C%E5%86%8D%E8%AE%BE%E5%AE%9A%E4%B8%80%E4%B8%AA%E6%96%B0timer\(%E9%9A%8F%E6%9C%BA%E6%97%B6%E9%97%B4\)%EF%BC%8C%E7%BB%99%E6%9C%AC%E5%9C%B0term%2B1%EF%BC%8C%E7%BB%A7%E7%BB%AD%E5%8F%91%E9%80%81AppendEntries%20RPC%E7%BB%99%E5%85%B6%E5%AE%83%E8%8A%82%E7%82%B9%EF%BC%8C%E5%BC%80%E5%90%AF%E6%96%B0%E7%AB%9E%E9%80%89%0A%20%20%20%20%20%20%20%201.%20%E5%8F%AF%E8%83%BD%E6%98%AF%E6%9C%89%E4%BA%9B%E8%8A%82%E7%82%B9%E7%9A%84%E5%93%8D%E5%BA%94%E8%BF%87%E6%85%A2%E6%B2%A1%E6%94%B6%E7%9D%80%0A%20%20%20%20%20%20%20%202.%20%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E9%83%BD%E6%8A%95%E7%A5%A8%E4%BA%86%EF%BC%8C%E4%BD%86%E6%98%AF%E8%87%AA%E5%B7%B1%E7%9A%84%E7%A5%A8%E6%95%B0%E4%B8%8D%E5%88%B0%E4%B8%80%E5%8D%8A%0A%20%20%20%203.%20%E5%9C%A8timer%E6%97%B6%E9%97%B4%E5%86%85%EF%BC%8C%E6%94%B6%E5%88%B0%E5%85%B6%E5%AE%83%E6%9C%BA%E5%99%A8%E7%AB%9E%E9%80%89%E6%88%90%E5%8A%9F%E7%9A%84%EF%BC%9AAppendEntries%20RPC%EF%BC%8C%E5%B0%86candidate%20%3D%3E%20follower%EF%BC%8C%E5%88%A0%E9%99%A4%E6%97%A7timer%EF%BC%8C%E5%88%9B%E5%BB%BA%E6%96%B0%E7%9A%84timer%0A4.%20%E4%B8%8A%E9%9D%A23%E6%AD%A5%E7%9A%84%E5%89%8D%E6%8F%90%EF%BC%9A%E5%A4%A7%E5%AE%B6%E5%90%8C%E6%97%B6%E8%AE%BE%E5%AE%9A%E7%9A%84%E5%BF%83%E8%B7%B3timer%EF%BC%8C%E6%9F%90%E4%B8%80%E5%8F%B0%E6%9C%BA%E5%99%A8%E5%85%88%E8%B6%85%E6%97%B6%E4%BA%86%EF%BC%8C%E5%AE%83%E5%B0%B1%E6%9C%80%E5%85%88%E5%BC%80%E5%90%AF%E4%BA%86%E9%80%89%E4%B8%BE%EF%BC%8C%E8%80%8C%E5%85%B6%E5%AE%83%E6%9C%BA%E5%99%A8%E7%9A%84%E7%8A%B6%E6%80%81%E5%8F%AF%E8%83%BD%EF%BC%8C%E5%A6%82%E4%B8%8B%EF%BC%9A%0A%20%20%20%201.%20follower%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%88%AB%E7%9A%84%E6%9C%BA%E5%99%A8%E5%85%88%E8%B6%85%E6%97%B6%EF%BC%8C%E8%82%AF%E5%AE%9A%E4%BC%9A%E5%85%88%E5%8F%91%E7%AB%9E%E9%80%89%E6%B6%88%E6%81%AF%EF%BC%8C%E8%87%AA%E5%B7%B1%E8%A2%AB%E5%8A%A8%E6%8E%A5%E6%94%B6%EF%BC%8C%E9%82%A3%E4%B9%88%EF%BC%8C%E5%88%B7%E6%96%B0%E8%87%AA%E5%B7%B1%E7%9A%84timer%20%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AFfollower%E5%B0%BD%E9%87%8F%E7%BB%B4%E6%8C%81follower%EF%BC%8C%E4%B8%8D%E5%8F%82%E9%80%89%20%0A%20%20%20%20%20%20%20%201.%20%E5%A6%82%E6%9E%9C%E6%8E%A5%E6%94%B6%E7%9A%84%E6%B6%88%E6%81%AF%E7%9A%84term%20%E6%AF%94%E8%BE%83%E8%87%AA%E5%B7%B1%E5%B0%8F%EF%BC%8C%E7%9B%B4%E6%8E%A5%E4%B8%A2%E6%8E%89%0A%20%20%20%20%20%20%20%202.%20%E5%A6%82%E6%9E%9C%E6%8E%A5%E6%94%B6%E7%9A%84%E6%B6%88%E6%81%AF%E7%9A%84term%20%E6%AF%94%E8%BE%83%E8%87%AA%E5%B7%B1%E5%A4%A7%EF%BC%8C%E4%B8%94%E6%9C%AC%E8%BD%AEterm%E8%BF%98%E6%9C%AA%E6%8A%95%E8%BF%87%E7%A5%A8%EF%BC%8C%E5%85%88%E6%9B%B4%E6%96%B0%E6%9C%AC%E5%9C%B0term%EF%BC%8C%E6%8A%95%E7%A5%A8%E7%BB%99%E5%AE%83%0A%20%20%20%20%20%20%20%203.%20%E5%A6%82%E6%9E%9C%E6%8E%A5%E6%94%B6%E7%9A%84%E6%B6%88%E6%81%AF%E7%9A%84term%20%E8%B7%9F%E8%87%AA%E5%B7%B1%E7%9B%B8%E5%90%8C%EF%BC%8C%E5%85%88%E6%9D%A5%E5%85%88%E5%BE%97%EF%BC%8C%E5%90%8E%E6%9D%A5%E6%8B%92%E7%BB%9D%0A%20%20%20%202.%20candidate%EF%BC%8C%E4%B8%8A%E9%9D%A2%E6%9C%89%E4%B8%80%E5%8F%B0%E8%A7%89%E9%86%92%EF%BC%8C%E4%BD%86%E5%90%8C%E6%97%B6%E8%BF%98%E6%9C%89%E4%B8%80%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%B9%9F%E8%A7%89%E9%86%92%E4%BA%86%EF%BC%8C%E4%B8%A4%E4%B8%AA%E4%BA%BA%E5%90%8C%E6%97%B6%E5%8F%91%E8%B5%B7%E7%AB%9E%E9%80%89%E3%80%82%E4%B8%A4%E4%B8%AA%E4%BA%BA%E4%BC%9A%E5%90%8C%E6%97%B6%E6%94%B6%E5%88%B0candidate%E7%AB%9E%E9%80%89%E7%9A%84%E6%B6%88%E6%81%AF\(%E5%BD%93%E7%A1%AE%E5%AE%9A%E5%B7%B2%E7%BB%8F%E5%8F%82%E9%80%89%EF%BC%8C%E5%B0%B1%E8%AF%81%E6%98%8E%E8%82%AF%E5%AE%9A%E7%BB%99%E8%87%AA%E5%B7%B1%E6%8A%95%E7%A5%A8%E4%BA%86\)%0A%20%20%20%20%20%20%20%201.%20%E5%A6%82%E6%9E%9C%E6%8E%A5%E6%94%B6%E7%9A%84%E6%B6%88%E6%81%AF%E7%9A%84term%0A%20%20%20%20%20%20%20%20%20%20%20%201.%20%E5%AF%B9%E6%96%B9%E7%9A%84term%E7%9B%B8%E5%90%8C%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%9B%9E%E7%BB%9D\(%E8%BF%99%E4%B8%AA%E6%98%AF%E4%B8%A4%E4%B8%AA%E9%9A%8F%E6%97%B6%E6%97%B6%E9%97%B4%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E5%B0%B1%E6%AF%94%E7%BD%91%E9%80%9F%E5%90%A7%EF%BC%8C%E5%85%88%E5%88%B0%E5%85%88%E5%BE%97%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E5%87%BA%E7%8E%B0%E5%88%86%E7%A5%A8\)%0A%20%20%20%20%20%20%20%20%20%20%20%202.%20%E5%AF%B9%E6%96%B9%E7%9A%84term%E6%9B%B4%E5%A4%A7%EF%BC%8C%E9%82%A3%E4%B9%88%E8%87%AA%E5%B7%B1%20candidate%20%3D%3E%20follower\(%E8%87%AA%E5%B7%B1%E5%9C%A8%E4%B8%8A%E4%B8%80%E4%B8%AA%E4%BB%BB%E6%9C%9F%E6%95%B4%E5%90%88%E9%83%BD%E6%98%AF%E6%8C%82%E4%BA%86%E7%9A%84%E7%8A%B6%E6%80%81%EF%BC%8C%E7%A2%B0%E5%B7%A7%E5%88%9A%E5%A5%BD%EF%BC%8C%E5%B0%B1%E5%8F%82%E4%B8%8E%E4%BA%86%E9%80%89%E7%A5%A8\)%0A%20%20%20%20%20%20%20%20%20%20%20%203.%20%E5%AF%B9%E6%96%B9%E7%9A%84term%E6%9B%B4%E5%B0%8F%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%9B%9E%E7%BB%9D\(%E8%BF%99%E7%A7%8D%E6%98%AF%E5%AF%B9%E9%9D%A2%E5%9C%A8%E6%95%B4%E4%B8%AA%E4%BB%BB%E6%9C%9F%E9%83%BD%E6%8C%82%E4%BA%86%EF%BC%8C%E6%AD%A3%E5%A5%BD%E6%96%B0%E4%BB%BB%E6%9C%9F%E5%AE%83%E5%8F%88%E5%A5%BD%E4%BA%86%EF%BC%8C%E6%89%80%E4%BB%A5%E5%B0%91%E4%BA%86N%E4%B8%AA%E4%BB%BB%E6%9C%9F\)%0A%20%20%20%20%20%20%20%202.%20%20%E5%88%86%E7%A5%A8%EF%BC%9A%E4%B8%A4%E4%BA%BA%E6%94%B6%E5%88%B0%E9%9B%86%E7%BE%A4%E5%86%85%E6%89%80%E6%9C%89%E7%9A%84%E6%8A%95%E7%A5%A8%E5%90%84%E8%87%AA%E9%83%BD%E6%98%AF%E4%B8%80%E5%8D%8A%EF%BC%8C%E9%82%A3%E4%B9%88%E9%80%89%E4%B8%BE%E5%A4%B1%E8%B4%A5%EF%BC%8Ctimer%E8%B6%85%E6%97%B6%EF%BC%8C%E5%86%8D%E5%AE%9A%E4%B8%AAtimer%EF%BC%8C%E9%87%8D%E5%A4%8D%E6%8A%95%E7%A5%A8%EF%BC%8C%E5%9B%A0%E4%B8%BAtimer%E6%98%AF%E9%9A%8F%E6%9C%BA%E7%9A%84%EF%BC%8C%E6%9C%80%E7%BB%88%E8%82%AF%E5%AE%9A%E4%BC%9A%E6%9C%89%E4%B8%80%E4%B8%AA%E5%85%88%E5%88%B0%E6%9C%9F%0A5.%20%E7%89%B9%E6%AE%8A%E6%83%85%E5%86%B5%EF%BC%8C%E5%81%87%E5%A6%82%20A%20B%20C%20D%20E%20%E6%AD%A3%E5%B8%B8%E7%BB%84%E6%88%90%E9%9B%86%E7%BE%A4%EF%BC%8CA%20%E4%B8%BAleader%EF%BC%8C%E6%AD%A4%E6%97%B6%E7%BD%91%E7%BB%9C%E6%95%85%E9%9A%9C%EF%BC%8CA%20B%20%E6%98%AF%E5%8E%9F%E7%8A%B6%E6%80%81%EF%BC%8CC%20D%20E%20%E7%BB%84%E6%88%90%E6%96%B0%E7%9A%84%E9%9B%86%E7%BE%A4%EF%BC%8CC%20%E4%B8%BA%E6%96%B0leader%EF%BC%8C%E7%84%B6%E5%90%8E%E6%95%85%E9%9A%9C%E5%A5%BD%E4%BA%86%EF%BC%8C%E6%AD%A4%E6%97%B6%0A%20%20%20%201.%20C%20%E4%BC%9A%E7%BB%99A%20%E5%8F%91%E9%80%81%E5%BF%83%E8%B7%B3%EF%BC%8CA%E6%8E%A5%E6%94%B6%E5%88%B0%E5%BF%83%E8%B7%B3%EF%BC%8C%E5%8F%91%E7%8E%B0%E5%AF%B9%E6%96%B9%E7%9A%84term%E6%9B%B4%E5%A4%A7%EF%BC%8C%E6%AD%A4%E6%97%B6%3Aleader%20%3D%3E%20follower%20%EF%BC%8C%0A%20%20%20%202.%20C%20%E4%BC%9A%E7%BB%99B%20%E5%8F%91%E9%80%81%E5%BF%83%E8%B7%B3%EF%BC%8CB%E6%9B%B4%E6%96%B0%E6%9C%AC%E5%9C%B0term%E5%80%BC%EF%BC%8C%E5%8F%98%E6%9B%B4leader%E6%8C%87%E5%90%91%E4%B8%BAC%0A%20%20%20%203.%20C%20D%20E%20%E5%90%8C%E6%97%B6%E4%B9%9F%E4%BC%9A%E6%94%B6%E5%88%B0A%E7%9A%84%E5%BF%83%E4%B8%AD%EF%BC%8C%E4%BD%86%E5%8F%91%E7%8E%B0term%E5%80%BC%E8%BF%87%E5%B0%8F%EF%BC%8C%E7%9B%B4%E6%8E%A5%E6%8B%92%E7%BB%9D%0A6.%20%E7%89%B9%E6%AE%8A%E6%83%85%E5%86%B5%EF%BC%8C%E5%A6%82%E6%9E%9C%E7%BD%91%E7%BB%9C%E7%8E%AF%E5%A2%83%E7%89%B9%E5%88%AB%E5%B7%AE%EF%BC%9ARTT%20%3E%20300MS%20%EF%BC%8C%E9%82%A3%E4%B9%88%E5%B0%B1%E4%BC%9A%E6%97%A0%E9%99%90%E9%80%89%E4%B8%BE%E3%80%82%E5%9B%A0%E4%B8%BA%E6%AF%8F%E4%B8%AAcandidate%E6%9C%80%E5%A4%A7%E8%B6%85%E6%97%B6%E6%98%AF300MS%EF%BC%8C%E5%AE%83%E5%8F%91%E5%87%BA%E5%8E%BB%E4%B8%80%E6%9D%A1%E5%88%B0%E6%94%B6%E4%B8%80%E6%9D%A1%E6%B6%88%E6%81%AF%E7%9A%84RTT%3E300ms%EF%BC%8C%E8%87%AA%E5%B7%B1%E6%97%A9%E5%B0%B1%E8%B6%85%E6%97%B6%E4%BA%86%EF%BC%8C%E5%B7%B2%E7%BB%8F%E5%BC%80%E5%90%AF%E4%B8%8B%E4%B8%80%E4%B8%AA%E4%BB%BB%E6%9C%9F%E9%81%A5%E9%80%89%E4%B8%BE%E4%BA%86%EF%BC%8C%E4%B8%8D%E8%BF%87%E6%98%AF%E6%AF%94%E8%BE%83%E6%9E%81%E7%AB%AF%E6%83%85%E5%86%B5%EF%BC%8C%E4%B8%8D%E5%A4%AA%E5%8F%AF%E8%83%BD%E5%8F%91%E7%94%9F%EF%BC%8C%E4%B9%9F%E5%8F%AF%E4%BB%A5%E6%A0%B9%E6%8D%AE%E8%87%AA%E5%B7%B1%E6%9C%BA%E5%99%A8%E7%9A%84%E7%BD%91%E7%BB%9CRTT%E6%97%B6%E9%97%B4%EF%BC%8C%E6%94%BE%E5%A4%A7%E8%B6%85%E6%97%B6%E9%9A%8F%E6%97%B6%E5%80%BC%0A7.%20%E7%89%B9%E6%AE%8A%E6%83%85%E5%86%B5%EF%BC%8C%E5%81%87%E8%AE%BE%E4%B8%80%E5%85%B15%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%8C%E8%BF%98%E5%89%A9%E4%B8%8BB%20C%20D%20%E6%98%AF%E6%AD%A3%E5%B8%B8%E7%9A%84%EF%BC%8C%E7%8E%B0%E5%9C%A8D%E6%98%AFleader%EF%BC%8C%E7%84%B6%E5%90%8E%E6%8C%82%E4%BA%86%EF%BC%8C%E8%BF%99%E6%97%B6%E5%B0%B1%E5%8F%AA%E5%89%A9%E4%B8%8BB%20C%20%E4%B8%A4%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%8C%E4%BB%96%E4%BB%AC%E6%97%A0%E8%AE%BA%E6%80%8E%E4%B9%88%E9%80%89%E6%9C%80%E5%A4%9A%E5%8F%AA%E8%83%BD%E5%BE%97%E5%88%B02%E7%A5%A8%EF%BC%8C%E5%B9%B6%E6%B2%A1%E6%9C%89%E5%88%B0%E8%BF%87%E5%8D%8A%E7%9A%843%E7%A5%A8%EF%BC%8C%E6%89%80%E4%BB%A5%E4%BC%9A%E6%97%A0%E9%99%90%E5%BE%AA%E7%8E%AF%E9%80%89%E4%B8%BE%E4%B8%8B%E5%8E%BB%E3%80%82%0A8.%20%E7%89%B9%E6%AE%8A%E6%83%85%E5%86%B5%EF%BC%8C%E5%81%87%E8%AE%BE%E4%B8%80%E5%85%B15%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%8C%E8%BF%98%E5%89%A9%E4%B8%8BB%20C%20D%20%E6%98%AF%E6%AD%A3%E5%B8%B8%E7%9A%84%EF%BC%8C%E7%8E%B0%E5%9C%A8D%E6%98%AFleader%EF%BC%8C%E7%84%B6%E5%90%8EB%E6%88%96C%E6%8C%82%E4%BA%86%EF%BC%8C%E6%AD%A4%E6%97%B6%E9%9B%86%E7%BE%A4%E8%BF%98%E6%98%AF%E8%83%BD%E6%AD%A3%E5%B8%B8%E6%94%B6%E6%B6%88%E6%81%AF%EF%BC%8C%E4%BD%86%E6%98%AF%E5%8D%B4%E6%97%A0%E6%B3%95%E7%A1%AE%E8%AE%A4%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E6%B6%88%E6%81%AF%E5%9C%A8D%20C%E6%88%96B%20%E9%83%BD%E5%AD%98%E4%BA%86%EF%BC%8C%E4%BD%86%E6%94%B6%E4%B8%8D%E5%88%B0%E4%B8%80%E5%8D%8A%E9%9B%86%E7%BE%A4%E7%9A%84'%E6%B6%88%E6%81%AF%E7%A1%AE%E8%AE%A4'%0A%0A%3E%E6%95%B4%E4%B8%AA%E9%80%89%E4%B8%BE%E6%9C%BA%E5%88%B6%E6%A0%B8%E5%BF%83%EF%BC%9A%E4%B8%A4%E4%B8%AA%E9%9A%8F%E6%9C%BA%E6%97%B6%E9%97%B4%E7%9A%84timer%E8%A7%A3%E5%86%B3%E4%BA%86%E4%B8%80%E5%88%87%EF%BC%8C%E4%B8%8D%E4%BC%9A%E6%AD%BB%E9%94%81%EF%BC%8C%E5%8F%88%E8%83%BD%E9%80%89%E5%87%BA%E5%94%AF%E4%B8%80%E7%9A%84leader%2C%E7%A1%AE%E5%AE%9E%E5%B7%A7%E5%A6%99%0A%0A%23%23%23%23%20%E5%88%86%E6%9E%90%E4%B8%80%E4%B8%8BRequestVoteReq%E6%B6%88%E6%81%AF%E4%BD%93%0A%E6%9C%89%E4%B8%A4%E4%B8%AA%E5%AD%97%E6%AE%B5%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E4%B8%8B%EF%BC%9Alast\_log\_index%20%E5%92%8C%20last\_log\_term%20%0A%0A1.%20%E5%81%87%E8%AE%BE%E5%BD%93%E5%89%8Dleader%E4%B8%BAA%EF%BC%8C%E5%9C%A8%E4%BB%BB%E6%9C%9F%EF%BC%9A1%20%E6%9C%9F%E9%97%B4%EF%BC%8C%E6%9C%89%E4%B8%80%E6%9D%A1%E6%97%A5%E5%BF%97%E5%B7%B2%E5%86%99%E5%85%A5%0A2.%20%E9%80%9A%E7%9F%A5%E5%85%B6%E5%AE%83%E8%8A%82%E7%82%B9%EF%BC%8C%E8%BF%9B%E8%A1%8C%E5%A4%8D%E5%88%B6%E6%93%8D%E4%BD%9C%0A3.%20%E6%94%B6%E5%88%B0%E5%8D%8A%E6%95%B0%E8%8A%82%E7%82%B9%E5%A4%8D%E5%88%B6%E6%88%90%E5%8A%9F%E7%9A%84%E6%B6%88%E6%81%AF%EF%BC%8C%E5%BC%80%E5%A7%8B%E6%9C%AC%E5%9C%B0%E6%8F%90%E4%BA%A4%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E9%80%9A%E7%9F%A5%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E8%BF%9B%E8%A1%8C%E6%8F%90%E4%BA%A4%0A2.%20%E5%81%87%E8%AE%BE%EF%BC%8C%E5%9C%A8%E8%BF%99%E6%9C%9F%E9%97%B4%EF%BC%8C%E6%9F%90%E4%B8%80%E5%8F%B0follower\-B%EF%BC%8C%E4%B9%9F%E5%9C%A8%E4%BB%BB%E6%9C%9F%EF%BC%9A1%EF%BC%8C%E4%BD%86%E5%9B%A0%E4%B8%BA%E7%BD%91%E7%BB%9C%E4%B8%8D%E5%A4%AA%E5%A5%BD%EF%BC%8C%E5%A4%8D%E5%88%B6%E8%BF%9B%E5%BA%A6%E8%BE%83%E6%85%A2%EF%BC%8C%E6%9C%AA%E6%94%B6%E5%88%B0%E8%AF%A5%E6%9D%A1%E6%97%A5%E5%BF%97%E5%A4%8D%E5%88%B6%E7%9A%84%E8%AF%B7%E6%B1%82%EF%BC%8C%E4%B9%9F%E6%9C%AA%E6%94%B6%E6%97%A5%E5%BF%97%E6%8F%90%E4%BA%A4%E7%9A%84%E8%AF%B7%E6%B1%82%0A3.%20%E5%81%87%E8%AE%BE%E6%AD%A4%E6%97%B6%20leader\-A%E6%8C%82%E4%BA%86%EF%BC%8CB%E6%98%AF%E6%96%B0%E7%9A%84leader%EF%BC%8C%E5%AE%83%E7%9A%84%E6%97%A5%E5%BF%97%E5%B0%B1%E6%98%AF%E8%90%BD%E5%90%8E%E7%9A%84%EF%BC%8C%E4%B9%9F%E6%B2%A1%E6%9C%89%E4%B8%8A%E9%9D%A2A%E4%B8%AD%E7%9A%84%E9%82%A3%E6%9D%A1%E6%97%A5%E5%BF%97%0A4.%20%E5%A6%82%E6%9E%9C%E6%A0%B9%E6%8D%AE%E5%BC%BAleader%E8%A6%86%E7%9B%96%E6%B3%95%E5%88%99%EF%BC%8C%E9%82%A3%E4%B9%88%E5%8E%9FA%E7%9A%84%E6%97%A5%E5%BF%97%E5%A4%9A%E5%87%BA%E7%9A%84%E5%87%A0%E6%9D%A1%E6%97%A5%E5%BF%97%E5%B0%B1%E8%A2%AB%E6%96%B0leader\-B%E7%BB%99%E8%A6%86%E7%9B%96%E6%8E%89%0A%0A%3E%E4%B8%8A%E9%9D%A2%E7%9A%84%E9%97%AE%E9%A2%98%E6%A0%B8%E5%BF%83%E6%98%AF%EF%BC%9A%E6%9F%90%E4%B8%AA%E8%BF%9B%E5%BA%A6%E8%90%BD%E5%90%8E%E7%9A%84follower%E8%99%BD%E7%84%B6%E8%BF%9B%E5%BA%A6%E6%85%A2%E7%BC%BA%E5%A4%B1%E4%BA%86%E6%9F%90%E5%87%A0%E6%9D%A1%E6%B6%88%E6%81%AF%EF%BC%8C%E4%BD%86%E6%98%AF%E5%9C%A8%E6%95%B4%E4%B8%AA%E4%BB%BB%E6%9C%9F%E6%9D%A5%E7%9C%8B%EF%BC%8C%E5%AE%83%E6%B2%A1%E6%9C%89%E5%85%A8%E4%B8%A2%E5%A4%B1%EF%BC%8C%E5%BD%93leader%E6%8C%82%E4%BA%86%EF%BC%8C%E5%AE%83%E4%BE%9D%E7%84%B6%E8%BF%98%E8%83%BD%E5%8F%82%E9%80%89%E5%B9%B6%E6%88%90%E4%B8%BA%E6%96%B0leader%EF%BC%8C%E5%9B%A0%E4%B8%BA%EF%BC%9A%E4%BB%96%E7%9A%84iterm%E5%B9%B6%E6%B2%A1%E6%9C%89%E8%90%BD%E5%90%8E%20%EF%BC%8C%E8%80%8C%E5%A6%82%E6%9E%9C%E5%8F%AA%E6%8B%BFitem%E5%88%A4%E6%96%AD%E6%98%AF%E5%90%A6%E5%8F%AF%E5%BD%93ledaer%E5%B0%B1%E4%BC%9A%E6%9C%89%E8%BF%99%E6%A0%B7%E7%9A%84%E6%80%BB%E6%98%AF%EF%BC%8C%E4%BC%9A%E6%8A%8A%E4%B9%8B%E5%89%8D%E8%BF%9B%E5%BA%A6%E5%BF%AB%E7%9A%84%E8%8A%82%E7%82%B9%E7%9A%84%E6%B6%88%E6%81%AF%E7%BB%99%E8%A6%86%E7%9B%96%E6%8E%89%E3%80%82%0A%0A%E6%89%80%E4%BB%A5%E5%BC%95%E5%85%A5%E4%BA%86%E4%B8%8A%E9%9D%A2%E4%B8%A4%E4%B8%AA%E5%AD%97%E6%AE%B5%E5%80%BC%EF%BC%9A%0A1.%20%E8%BF%9B%E5%BA%A6%E8%BE%83%E6%85%A2%E7%9A%84%20candidate%EF%BC%8C%E5%8F%91%E5%87%BARequestVoteReq%E6%B6%88%E6%81%AF%EF%BC%8C%E5%85%B6%E4%B8%AD%E5%8C%85%E5%90%AB%E4%BA%86%EF%BC%9Acandidate%E7%9A%84%E6%97%A5%E5%BF%97%E8%BF%9B%E5%BA%A6%20last\_log\_index%20last\_log\_term%0A2.%20follower%E5%9C%A8%E6%8E%A5%E6%94%B6%E5%90%8E%EF%BC%8C%E4%B8%8E%E6%9C%AC%E6%9C%BA%E7%9A%84%20last\_log\_index%20last\_log\_term%20%E8%BF%9B%E8%A1%8C%E5%AF%B9%E6%AF%94%EF%BC%8C%E5%A6%82%E6%9E%9Cterm%E7%9B%B8%E5%90%8C%EF%BC%8C%E5%86%8D%E6%AF%94%E8%BE%83last\_log\_index%EF%BC%8C%E5%8F%91%E7%8E%B0%E8%87%AA%E5%B7%B1%E7%9A%84%E6%9B%B4%E6%96%B0%EF%BC%8C%E7%9B%B4%E6%8E%A5%E5%9B%9E%E7%BB%9D%E8%AF%A5candidate%0A2.%20%E5%A6%82%E6%9E%9C%E6%9F%90%E5%8F%B0follower%E8%BF%9B%E5%BA%A6%E5%BE%88%E6%96%B0%EF%BC%8C%E4%BD%86%E8%8B%8F%E9%86%92%E8%BE%83%E6%85%A2%EF%BC%8C%E5%8F%AF%E4%BB%A5%E7%94%A8%E8%BF%99%E4%B8%AA%E5%8A%9F%E8%83%BD%E5%85%88%E6%A6%82%E7%8E%87%E6%80%A7%E7%9A%84%E8%BF%87%E8%99%91%E6%8E%89%E8%BF%9B%E5%BA%A6%E8%90%BD%E5%90%8E%E7%9A%84candidate%EF%BC%8C%E4%B8%BA%E8%87%AA%E5%B7%B1%E4%BA%89%E5%8F%96%E6%97%B6%E9%97%B4%E5%8F%82%E9%80%89%20%0A%0A%E9%82%A3%E4%B9%88%E5%8A%A0%E4%B8%A4%E4%B8%AA%E5%AD%97%E6%AE%B5%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AF%EF%BC%9A%E5%BD%93%E5%8E%9Fleader%E6%8C%82%E4%BA%86%E5%90%8E%EF%BC%8C%E5%B0%BD%E9%87%8F%E9%80%89%E5%87%BA%E6%97%A5%E5%BF%97%E6%9C%80%E9%95%BF%E7%9A%84%E9%82%A3%E4%B8%AA%20candidate%20%0A%0A%E4%BD%86%E8%BF%99%E9%87%8C%E8%BF%98%E6%9C%89%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%8C%E4%B8%A4%E4%B8%AA%E5%80%BC%E6%98%AF%E4%B8%80%E8%B5%B7%E5%88%A4%E5%AE%9A%E8%BF%98%E6%98%AF%E6%9C%89%E9%A1%BA%E5%BA%8F%E7%9A%84%EF%BC%9F%0A1.%20%E5%85%88%E5%88%A4%E6%96%AD%20last\_log\_term%0A2.%20%E5%86%8D%E5%88%A4%E6%96%AD%20last\_log\_index%0A%E6%89%80%E4%BB%A5%EF%BC%8C%E8%BF%99%E4%B8%A4%E4%B8%AA%E5%80%BC%E6%9B%B4%E5%80%BE%E5%90%91%E8%A7%A3%E5%86%B3%EF%BC%8C%E5%9C%A8%E5%90%8C%E4%B8%80%E4%BB%BB%E6%9C%9F%E7%9A%84%E5%89%8D%E6%8F%90%E4%B8%8B%EF%BC%8C%E7%9C%8B%E5%93%AA%E4%B8%AA%E6%97%A5%E5%BF%97%E6%9B%B4%E9%95%BF%EF%BC%8C%E8%80%8C%E5%A6%82%E6%9E%9C%E8%BF%99%E4%B8%AA%E6%97%B6%E5%80%99%EF%BC%8C%E6%9C%89%E7%9A%84%E6%97%A5%E5%BF%97%E8%99%BD%E7%84%B6%E8%BE%83%E7%9F%AD%EF%BC%8C%E4%BD%86%E6%98%AF%E5%8F%AF%E8%83%BD%E4%BB%BB%E6%9C%9F%E5%8F%B7%E6%9B%B4%E5%A4%A7%EF%BC%8C%E9%82%A3%E5%AE%83%E4%B9%9F%E5%AE%8C%E5%85%A8%E5%8F%AF%E4%BB%A5%E6%88%90%E4%B8%BAleader%E7%9A%84%EF%BC%8C%E9%82%A3%E4%B9%88%E5%BC%95%E5%87%BA%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%9A%0A%E6%96%B0leader%E7%9A%84term%2B1%E4%BA%86%EF%BC%8C%E4%BB%BB%E6%9C%9F%E5%B0%B1%E6%9B%B4%E6%96%B0%E4%BA%86%EF%BC%8C%E4%BD%86%E6%98%AF%E6%9C%AC%E6%9C%BA%E4%B8%8A%E5%8F%AF%E8%83%BD%E8%BF%98%E6%9C%89%E6%97%A7%E4%BB%BB%E6%9C%9F%E5%86%85%E6%9C%AA%E6%8F%90%E4%BA%A4%E7%9A%84%E6%97%A5%E5%BF%97%EF%BC%8C%E9%82%A3%E6%96%B0%E4%BB%BB%E6%9C%9F%E6%98%AF%E5%90%A6%E6%8F%90%E5%89%8D%E6%97%A7%E4%BB%BB%E6%9C%9F%E7%9A%84%E6%97%A5%E5%BF%97%E5%91%A2%EF%BC%9F%0A%E7%AD%94%E6%A1%88%3A%E4%B8%8D%E4%BC%9A%0A%E5%9B%A0%E4%B8%BA%E6%9C%89%E4%B8%80%E4%B8%AA%E9%A3%8E%E9%99%A9%EF%BC%9A%E5%A6%82%E6%9E%9C%E6%8F%90%E4%BA%A4%E6%97%A7%E4%BB%BB%E6%9C%9F%E7%9A%84%E6%9C%AA%E6%8F%90%E4%BA%A4%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E9%82%A3%E4%B9%88%E6%AD%A4%E6%97%B6%E5%AE%83%E6%8C%82%E4%BA%86%EF%BC%8C%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%E6%96%B0%E6%9C%BA%E5%99%A8%E5%BD%93%E9%80%89%E6%96%B0leader%EF%BC%8C%E5%AE%83%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E7%BC%BA%E5%A4%B1%E4%BA%86%EF%BC%8C%E9%82%A3%E4%B9%88%E5%AE%83%E5%8F%AF%E8%83%BD%E4%BC%9A%E8%A6%86%E7%9B%96%E6%8E%89%E5%88%9A%E5%88%9A%E6%8F%90%E4%BA%A4%E7%9A%84%E6%97%A7%E4%BB%BB%E6%9C%9F%E7%9A%84%E6%95%B0%E6%8D%AE%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%8C%BA%E5%8F%AF%E6%80%95%E7%9A%84%EF%BC%8C%E6%88%96%E8%80%85%E8%AF%B4%EF%BC%8Craft%E4%B8%8D%E8%83%BD%E6%8E%A5%E5%8F%97%E8%A6%86%E7%9B%96%E5%B7%B2%E6%8F%90%E4%BA%A4%E7%9A%84%E6%95%B0%E6%8D%AE%0A%0A%E4%BD%86%E5%A6%82%E6%9E%9C%E6%9C%89%E6%96%B0%E7%9A%84%E5%86%99%E5%85%A5%E6%93%8D%E4%BD%9C%EF%BC%8C%E4%BC%9A%E8%BF%9E%E5%B8%A6%E7%9D%80%E6%97%A7%E7%9A%84%E4%B8%80%E8%B5%B7%E6%8F%90%E4%BA%A4%20commit\_id%0A%0A%E8%BF%99%E4%B8%AA%E9%97%AE%E9%A2%98%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AF%EF%BC%9A%E9%9B%86%E7%BE%A4\-%E6%8F%90%E4%BA%A4%E7%A1%AE%E8%AE%A4%EF%BC%8C%E5%8D%B3%EF%BC%9Aleader%E6%8F%90%E4%BA%A4%E5%90%8E%EF%BC%8C%E8%A6%81%E7%AB%8B%E5%88%BB%E5%8F%91%E9%80%81%E7%BB%99%E6%89%80%E6%9C%89follower%EF%BC%8C%E5%B9%B6%E9%98%BB%E5%A1%9E%EF%BC%8C%E7%AD%89%E5%BE%85%E5%8D%8A%E6%95%B0follwer%E5%9B%9E%E5%A4%8D%EF%BC%9A%E6%8F%90%E4%BA%A4OK%E5%90%8E%EF%BC%8C%E5%86%8D%E5%9B%9E%E5%A4%8Dclient%E6%9C%AC%E6%AC%A1%E6%97%A5%E5%BF%97%E5%86%99%E5%85%A5%E6%88%90%E5%8A%9F%E3%80%82%E4%B8%8D%E8%BF%87raft%E6%B2%A1%E6%8F%90%EF%BC%8C%E4%BC%B0%E8%AE%A1%E5%8F%AF%E8%83%BDraft%E8%A7%89%E5%BE%97%E6%B2%A1%E5%BF%85%E8%A6%81%E8%80%83%E8%99%91%E8%BF%99%E7%A7%8D%E5%9C%BA%E6%99%AF%E5%90%A7%E3%80%82%E5%8F%A6%E5%A4%96%EF%BC%8C%E5%B0%B1%E7%AE%97%E5%8A%A0%E8%BF%99%E4%B8%AA%E7%A1%AE%E8%AE%A4%E6%9C%BA%E5%88%B6%EF%BC%8C%E4%BD%86%E4%B9%9F%E4%BC%9A%E9%9D%9E%E5%B8%B8%E9%BA%BB%E7%83%A6%EF%BC%8C%E8%BF%98%E5%BE%97%E8%80%83%E8%99%91%E5%90%84%E7%A7%8D%E6%96%AD%E7%BD%91%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%A5%BD%E5%83%8F%20google%20percolator%E6%A8%A1%E5%9E%8B%E6%9C%89%E8%A7%A3%E5%86%B3%E3%80%82%0A%0A%0A%0A%E6%AF%8F%E4%B8%AA%E8%8A%82%E7%82%B9%E5%BF%85%E9%A1%BB%E5%BE%97%E6%9C%893%E4%B8%AA%E5%B1%9E%E6%80%A7%EF%BC%9A%0AcurrentTerm%EF%BC%9A%E8%8A%82%E7%82%B9%E5%BD%93%E5%89%8D%E6%89%80%E5%A4%84%E7%9A%84%E4%BB%BB%E6%9C%9F%E3%80%82%0AvotedFor%20%E3%80%80%EF%BC%9A%E8%8A%82%E7%82%B9%E5%BD%93%E5%89%8D%E4%BB%BB%E6%9C%9F%E6%89%80%E8%B7%9F%E9%9A%8F%E7%9A%84%E5%85%B6%E4%BB%96%E8%8A%82%E7%82%B9%E7%9A%84%20ID%E3%80%82%E5%A6%82%E6%9E%9C%E6%98%AF%E6%8A%95%E7%A5%A8%E9%98%B6%E6%AE%B5%EF%BC%8C%E8%A1%A8%E7%A4%BA%E8%8A%82%E7%82%B9%E6%89%80%E6%8A%95%E7%9A%84%E5%80%99%E9%80%89%E4%BA%BA%E3%80%82%E5%A6%82%E6%9E%9C%E5%B7%B2%E7%BB%8F%E7%AB%9E%E9%80%89%E7%BB%93%E6%9D%9F%EF%BC%8C%E5%B0%B1%E6%98%AF%E5%BD%93%E5%89%8D%E7%9A%84%E9%A2%86%E5%AF%BC%E4%BA%BA%E3%80%82%0AcurrentLogIndex%3A%E6%9C%AC%E6%9C%BA%E6%97%A5%E5%BF%97%E7%B4%A2%E5%BC%95%E5%8F%B7%0A%0A%3E%E5%9F%BA%E6%9C%AC%E4%B8%8A%E6%97%A5%E5%B8%B8%E7%9A%84%E6%93%8D%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%B0%B1%E6%98%AF%E8%BF%993%E4%B8%AA%E5%B1%9E%E6%80%A7%E5%80%BC%2B%E7%8A%B6%E6%80%81%E5%80%BC%2B%E6%B6%88%E6%81%AF%E4%BC%A0%E8%BE%93%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%A4%9F%E7%94%A8%E4%BA%86%E3%80%82raft%E7%9A%84%E5%AE%9E%E7%8E%B0%E7%A1%AE%E5%AE%9E%E5%BE%88%E7%AE%80%E5%8D%95%E3%80%82%0A%0A%E5%A4%8D%E5%88%B6%E7%8A%B6%E6%80%81%E6%9C%BA%0A\-\-\-\-\-%0AReplicated%20state%20machines%EF%BC%9A%E4%BF%9D%E8%AF%81%E6%AF%8F%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%B8%8A%E7%9A%84%E7%8A%B6%E6%80%81%E6%98%AF%E4%B8%80%E8%87%B4%E7%9A%84%EF%BC%8C%E5%90%84%E8%8A%82%E7%82%B9%E4%B9%8B%E9%97%B4%E9%80%9A%E8%BF%87%E5%A4%8D%E5%88%B6leader%E7%9A%84%E6%97%A5%E5%BF%97\(replicated%20log\)%E5%88%B0%E6%9C%AC%E6%9C%BA%E4%B8%8A%EF%BC%8C%E5%86%8D%E6%9B%B4%E6%96%B0%E6%9C%AC%E6%9C%BA%E7%8A%B6%E6%80%81%E5%80%BC%0A%3E%E7%9B%B8%E5%90%8C%E7%9A%84%E5%88%9D%E5%A7%8B%E7%8A%B6%E6%80%81%20%2B%20%E7%9B%B8%E5%90%8C%E7%9A%84%E8%BE%93%E5%85%A5%20%3D%20%E7%9B%B8%E5%90%8C%E7%9A%84%E7%BB%93%E6%9D%9F%E7%8A%B6%E6%80%81%0A%0A%E4%BB%8E%E5%AE%9A%E4%B9%89%E4%B8%8A%E7%9C%8B%EF%BC%8C%E5%AE%83%E5%8F%AF%E4%BB%A5%E5%88%86%E6%88%90%E4%B8%A4%E9%83%A8%E5%88%86%EF%BC%9A%0A1.%20%E6%97%A5%E5%BF%97%E7%9A%84%E5%A4%8D%E5%88%B6%0A2.%20%E6%9C%AC%E6%9C%BA%E7%9A%84%E7%8A%B6%E6%80%81%E5%8F%98%E6%9B%B4%0A%0A%E9%9A%BE%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%B0%B1%E6%98%AF%E6%97%A5%E5%BF%97%E7%9A%84%E5%A4%8D%E5%88%B6%2F%E5%90%8C%E6%AD%A5%EF%BC%8C%E5%86%8D%E5%BE%80%E6%B7%B1%E4%BA%86%E7%9C%8B%EF%BC%8C%E5%AE%83%E6%9B%B4%E5%83%8F%E6%98%AF%EF%BC%9A%E5%BC%BA%E4%B8%80%E8%87%B4%E6%80%A7\(consistent\)%E4%B8%8E%E5%85%B1%E8%AF%86%EF%BC%88consensus%EF%BC%89%E7%AE%97%E6%B3%95%0A%3E%E5%A5%BD%E5%83%8F%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%AF%B4%E5%A4%8D%E5%88%B6%E7%8A%B6%E6%80%81%E6%9C%BA%E7%9A%84%E6%A0%B8%E5%BF%83%E5%B0%B1%E6%98%AF%E5%AE%9E%E7%8E%B0%EF%BC%9A%E5%BC%BA%E4%B8%80%E8%87%B4%E6%80%A7%E4%B8%8E%E5%85%B1%E8%AF%86%E7%AE%97%E6%B3%95%EF%BC%8C%E8%B2%8C%E4%BC%BC%E4%B9%9F%E5%8F%AF%E4%BB%A5%20%0A%0A%E5%85%B7%E4%BD%93%E5%88%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%EF%BC%9A%E6%AF%8F%E4%B8%AA%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AD%98%E5%82%A8%E4%B8%80%E4%B8%AA%E5%8C%85%E5%90%AB%E4%B8%80%E7%B3%BB%E5%88%97%E6%8C%87%E4%BB%A4%E7%9A%84%E6%97%A5%E5%BF%97%EF%BC%8C%E5%B9%B6%E4%B8%94%E6%8C%89%E9%A1%BA%E5%BA%8F%E6%89%A7%E8%A1%8C%E6%8C%87%E4%BB%A4%E3%80%82%E7%94%B1%E4%BA%8E%E6%97%A5%E5%BF%97%E9%83%BD%E5%8C%85%E5%90%AB%E7%9B%B8%E5%90%8C%E9%A1%BA%E5%BA%8F%E7%9A%84%E6%8C%87%E4%BB%A4%EF%BC%8C%E7%8A%B6%E6%80%81%E6%9C%BA%E4%BC%9A%E6%8C%89%E7%85%A7%E7%9B%B8%E5%90%8C%E7%9A%84%E9%A1%BA%E5%BA%8F%E6%89%A7%E8%A1%8C%E6%8C%87%E4%BB%A4%EF%BC%8C%E7%94%B1%E4%BA%8E%E7%8A%B6%E6%80%81%E6%9C%BA%E6%98%AF%E7%A1%AE%E5%AE%9A%E7%9A%84%EF%BC%88deterministic%EF%BC%89%EF%BC%8C%E5%9B%A0%E6%AD%A4%E7%8A%B6%E6%80%81%E6%9C%BA%E4%BC%9A%E4%BA%A7%E7%94%9F%E7%9B%B8%E5%90%8C%E7%9A%84%E7%BB%93%E6%9E%9C%E3%80%82%0A%0A%0A%0A%E6%97%A5%E5%BF%97%E5%A4%8D%E5%88%B6%0A\-\-\-\-%0A%0A%E4%B8%80%E6%9D%A1%E6%97%A5%E5%BF%97%E5%8C%85%E5%90%AB3%E4%B8%AA%E5%80%BC%EF%BC%9A%E6%8C%87%E4%BB%A4%2B%E6%97%A5%E5%BF%97INDEX\_ID%2BtermId%0A%0A%23%23%23%23%20%E5%A4%8D%E5%88%B6%E8%BF%87%E7%A8%8B%0A1.%20client%20\(set%20a%20%3D%205\)%3D%3E%20%E9%9B%86%E7%BE%A4%E4%B8%AD%E7%9A%84%E6%9F%90%E5%8F%B0%E6%9C%BA%E5%99%A8%0A%20%20%20%201.%20%E5%A6%82%E6%9E%9C%E8%AF%A5%E6%9C%BA%E5%99%A8%E5%B0%B1%E6%98%AFleader%EF%BC%8C%E5%B0%B1%E6%AD%A3%E5%B8%B8%E5%A4%84%E7%90%86%0A%20%20%20%202.%20%E5%A6%82%E6%9E%9C%E8%AF%A5%E6%9C%BA%E5%99%A8%E7%A2%B0%E5%B7%A7%E6%8C%82%E4%BA%86%EF%BC%8C%E5%86%8D%E9%87%8D%E6%96%B0%E6%89%BE%E4%B8%80%E5%8F%B0%E6%9C%BA%E5%99%A8%0A%20%20%20%203.%20%E5%A6%82%E6%9E%9C%E8%AF%A5%E6%9C%BA%E5%99%A8%E6%98%AFfollower%EF%BC%8Cfollower%E8%BF%94%E5%9B%9Eleader%E7%9A%84IP%0A2.%20leader%E6%8E%A5%E6%94%B6%EF%BC%8C%E5%B9%B6%E5%9C%A8%E4%BF%9D%E5%AD%98%E5%9C%A8%E6%9C%AC%E5%9C%B0%E6%97%A5%E5%BF%97%E4%B8%AD%EF%BC%8C%E5%B9%B6%E6%A0%87%E8%AE%B0%E7%8A%B6%E6%80%81%E4%B8%BA%EF%BC%9Auncommitted%0A3.%20%E5%B0%86%E8%AF%A5%E6%9D%A1%E6%97%A5%E5%BF%97%E5%86%85%E5%AE%B9%EF%BC%8C%E8%BF%BD%E5%8A%A0%E5%88%B0%E4%B8%80%E4%B8%AA%E5%BF%83%E8%B7%B3%E5%8C%85\(AppendEntries%20RPC\)%E4%B8%AD%EF%BC%8C%E4%B8%8B%E6%AC%A1%E5%BF%83%E8%B7%B3%EF%BC%9A%E5%90%8C%E6%AD%A5%E7%BB%99%E6%89%80%E6%9C%89follower%E8%8A%82%E7%82%B9%0A3.%20%E4%B8%BA%E8%AF%A5%E6%AC%A1%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%B7%E6%B1%82%EF%BC%8C%E8%AE%BE%E5%AE%9A%E5%AE%9A%E6%97%B6%E5%99%A8%EF%BC%8C%E7%AD%89%E5%BE%85%E6%89%80%E6%9C%89follower%E5%93%8D%E5%BA%94%0A%20%20%20%201.%20%E8%B6%85%E6%97%B6\(%E6%94%B6%E5%88%B0follower%E6%88%90%E5%8A%9F%E6%95%B0%E4%B8%8D%E8%BF%87%E5%8D%8A\)%EF%BC%8C%E6%89%BE%E5%88%B0%E6%9C%AA%E5%93%8D%E5%BA%94%E7%9A%84follower%EF%BC%8C%E6%97%A0%E9%99%90%E9%87%8D%E8%AF%95%E6%AD%A5%E9%AA%A43%EF%BC%8C%E7%9B%B4%E6%8E%A5%E8%BF%99%E4%B8%AAfollower%E8%BF%94%E5%9B%9E%E6%88%90%E5%8A%9F%0A%20%20%20%202.%20%E6%9C%AA%E8%B6%85%E6%97%B6%EF%BC%8C%E6%94%B6%E5%88%B0%E4%BA%86%E4%B8%80%E5%8D%8A%E8%8A%82%E7%82%B9%E7%9A%84%E5%93%8D%E5%BA%94%E6%88%90%E5%8A%9F%0A%20%20%20%20%20%20%20%201.%20%E6%89%A7%E8%A1%8C%E6%9C%AC%E6%9D%A1%E6%97%A5%E5%BF%97%E7%9A%84%E5%AF%B9%E5%BA%94%E7%9A%84%E6%8C%87%E4%BB%A4%EF%BC%8C%E6%9B%B4%E6%96%B0%E7%8A%B6%E6%80%81%E6%9C%BA%0A%20%20%20%20%20%20%20%202.%20%E5%B0%86%E6%9C%AC%E6%9D%A1%E6%97%A5%E5%BF%97%E7%8A%B6%E6%80%81%E5%8F%98%E6%9B%B4%E4%B8%BA%EF%BC%9Acommitted%0A%20%20%20%20%20%20%20%203.%20%E6%9B%B4%E6%96%B0%E6%9C%AC%E5%9C%B0%E5%8F%98%E9%87%8F%EF%BC%9Acommitted\_id%0A%20%20%20%20%20%20%20%203.%20%E5%B0%86committed\_id%2Bterm\_id%2Blast\_logindex%E6%89%93%E5%8C%85%E5%88%B0AppendEntriesRpc%E4%B8%AD%0A%20%20%20%20%20%20%20%204.%20%E5%9C%A8%E4%B8%8B%E4%B8%80%E4%B8%AA%E5%BF%83%E8%B7%B3%E5%8C%85\(AppendEntries%20RPC\)%E4%B8%AD%EF%BC%8C%E9%80%9A%E7%9F%A5%E6%89%80%E6%9C%89follower%E5%8F%AF%E4%BB%A5%E6%8F%90%E4%BA%A4%E4%BA%86%E3%80%82%0A%20%20%20%203.%20%E6%9C%AA%E8%B6%85%E6%97%B6%EF%BC%8C%E4%BD%86%E6%98%AF%E6%94%B6%E5%88%B0%E7%9A%84%E5%93%8D%E5%BA%94%E6%98%AF%E6%8B%92%E7%BB%9D%E7%9A%84%E3%80%82%E5%8F%AF%E8%83%BD%E5%9C%A8%E7%AD%89%E5%BE%85%E6%94%B6%E7%A1%AE%E8%AE%A4%E5%8C%85%E7%9A%84%E8%BF%87%E7%A8%8B%E4%B8%AD%EF%BC%8C%E8%87%AA%E5%B7%B1%E7%9A%84%E7%BD%91%E7%BB%9C%E5%87%BA%E7%8E%B0%E4%BA%86%E9%97%AE%E9%A2%98%EF%BC%8C%E5%AF%BC%E8%87%B4%E8%BF%9E%E4%B8%8D%E4%B8%8A%E5%85%B6%E5%AE%83%E6%9C%BA%E5%99%A8%EF%BC%8C%E8%80%8C%E5%85%B6%E5%AE%83%E6%9C%BA%E5%99%A8%E9%97%B4%E6%98%AF%E6%AD%A3%E5%B8%B8%E7%9A%84%EF%BC%8C%E5%AE%83%E4%BB%AC%E4%B9%9F%E6%A3%80%E6%B5%8B%E5%87%BAledaer%E5%A4%B1%E5%8E%BB%E4%BA%86%E8%81%94%E7%B3%BB%EF%BC%8C%E4%BA%8E%E6%98%AF%E5%8F%88%E9%80%89%E5%87%BA%E4%BA%86%E6%96%B0%E7%9A%84leader....%E7%84%B6%E5%90%8E%E4%BD%A0%E7%9A%84%E7%BD%91%E7%BB%9C%E6%AD%A3%E5%B8%B8%EF%BC%8C%E5%85%B6%E5%AE%83%E8%8A%82%E7%82%B9%E5%91%8A%E8%AF%89%E6%96%B0%E7%9A%84leader%2C%E4%BD%A0%E7%9A%84term%E5%B7%B2%E7%BB%8F%E8%90%BD%E5%90%8E%EF%BC%8C%E6%AD%A4%E6%97%B6%E6%97%A7%E7%9A%84leader%20%E7%8A%B6%E6%80%81%E5%8F%98%E6%9B%B4%EF%BC%9Aleader%20%3D%3E%20follower%E3%80%82%E5%90%8C%E6%97%B6%E6%9C%AC%E6%9D%A1%E6%9C%AA%E6%8F%90%E4%BA%A4%E6%88%90%E5%8A%9F%E7%9A%84%E6%97%A5%E5%BF%97%E4%BC%9A%E8%A2%AB%E6%96%B0leader%E8%A6%86%E7%9B%96%E6%8E%89%E3%80%82%0A%0A%3E%E4%B8%8A%E9%9D%A2%E8%BF%99%E4%B8%AA%E6%83%85%E5%86%B5%E5%8F%AA%E6%98%AF%E6%9C%80%E7%90%86%E6%83%B3%E4%B8%94%E6%9C%80%E7%AE%80%E5%8D%95%EF%BC%8C%E4%B8%8B%E9%9D%A2%E8%AE%B2%E4%B8%80%E4%B8%8B%E4%B8%8D%E4%B8%80%E8%87%B4%E7%9A%84%E6%83%85%E5%86%B5%0A%0A%3E%E4%BB%8E%E8%BF%99%E4%B8%AA%E8%BF%87%E7%A8%8B%E6%9D%A5%E7%9C%8B%EF%BC%9A%E5%A4%8D%E5%88%B6%E7%8A%B6%E6%80%81%E6%9C%BA%E5%BA%94%E8%AF%A5%E5%88%86%E6%88%90%E4%B8%A4%E4%B8%AA%E9%83%A8%E5%88%86%EF%BC%8C%E4%B8%80%E4%B8%AA%E6%98%AF%E7%9C%9F%E7%9A%84%E7%8A%B6%E6%80%81%E6%9C%BA%EF%BC%8C%E5%8F%A6%E4%B8%80%E4%B8%AA%E8%BE%BE%E6%88%90%E5%85%B1%E8%AF%86%E3%80%82%E5%8D%B3%EF%BC%9A%E8%BE%BE%E6%88%90%E5%85%B1%E8%AF%86%E6%9B%B4%E5%83%8F%E6%98%AF%E4%B8%80%E6%AC%A1%E6%93%8D%E4%BD%9C%E7%9A%84%E6%A0%87%E8%AF%86%E6%88%96%E4%B8%80%E4%B8%AA%E9%94%81%EF%BC%8C%E6%A0%87%E8%AF%86%E9%83%BD%E6%B2%A1%E6%9C%89%E9%97%AE%E9%A2%98%E5%90%8E%EF%BC%8C%E5%86%8D%E6%89%A7%E8%A1%8C%E8%AF%A5%E6%A0%87%E8%AF%86%E5%AF%B9%E5%BA%94%E7%9A%84%E6%8C%87%E4%BB%A4%EF%BC%8C%E5%8E%BB%E6%9B%B4%E6%96%B0%E7%8A%B6%E6%80%81%E6%9C%BA%E9%87%8C%E7%9A%84%E5%80%BC%E3%80%82%0A%0A%0A%23%23%23%23%20%E6%99%AE%E9%80%9A%E6%83%85%E5%86%B5%EF%BC%8C%E6%9C%BA%E5%99%A8%E6%8C%82%E4%BA%86%EF%BC%8C%E6%97%A5%E5%BF%97%E4%B8%8D%E4%B8%80%E8%87%B4%E8%A1%A5%E9%BD%90%EF%BC%9A%0A1.%20follower%20%E6%8C%82%E4%BA%86%E5%90%8E%E6%81%A2%E5%A4%8D%EF%BC%9Aleader%E5%8F%91%E7%8E%B0%E8%AF%A5follower%E8%BF%9B%E5%BA%A6%E9%9D%9E%E5%B8%B8%E7%BC%93%E6%85%A2%EF%BC%8C%E5%8F%AF%E8%83%BD%E9%81%97%E6%BC%8F%E4%BA%86%E5%A5%BD%E5%87%A0%E4%B8%AA%E4%BB%BB%E6%9C%9F%E3%80%82%0A%20%20%20%201.%20raft%E4%B8%80%E8%87%B4%E6%80%A7%E6%A3%80%E6%9F%A5%EF%BC%9Aleader%E5%9C%A8%E5%8F%91%E9%80%81%E6%AF%8F%E4%B8%AAAppendEntries%20RPC%20%E4%BC%9A%E8%BF%BD%E5%8A%A0%E4%B8%A4%E4%B8%AA%E5%AD%97%E6%AE%B5%EF%BC%9A%E6%97%A5%E5%BF%97INDEX\_ID%2BtermId%0A%20%20%20%202.%20%E6%AD%A4%E6%97%B6%E5%9B%A0%E4%B8%BAfollower%E5%9C%A8%E6%9C%AC%E5%9C%B0%E6%97%A5%E5%BF%97%E4%B8%AD%E6%89%BE%E4%B8%8D%E5%88%B0leader%E5%8F%91%E9%80%81%E7%9A%84INDEX\_ID%2BtermId%2C%E4%BA%8E%E6%98%AF%E4%BC%9A%E6%8B%92%E7%BB%9D%0A%20%20%20%203.%20leader%E5%8F%91%E7%8E%B0follower%E6%8B%92%E7%BB%9D%EF%BC%8C%E5%B0%B1%E5%86%8D%E6%8A%8AINDEX\_ID%E5%90%91%E5%89%8D%E5%8F%96%E4%B8%80%E4%B8%AA%EF%BC%8C%E4%BE%9D%E6%AD%A4%E7%B1%BB%E6%8E%A8%0A%20%20%20%20%3E%E6%84%9F%E8%A7%89%E8%BF%99%E7%A7%8D%E5%AF%BB%E6%89%BE%E6%97%A5%E5%BF%97offset%E7%9A%84%E6%96%B9%E6%B3%95%E6%8C%BA2%E7%9A%84%EF%BC%8C%E7%94%A8%E5%88%86%E6%AE%B5\(%E4%BB%BB%E6%9C%9F\)%E6%9F%A5%E6%89%BE%E6%88%96%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E5%A4%9A%E5%BF%AB%EF%BC%8C%E5%85%B7%E8%AF%B4raft%E8%AE%BA%E6%96%87%E8%AF%B4%E6%B2%A1%E5%BF%85%E8%A6%81%EF%BC%8C%E5%A4%B1%E8%B4%A5%E7%9A%84%E6%83%85%E5%86%B5%E6%AF%95%E7%AB%9F%E6%98%AF%E5%B0%91%E6%95%B0%EF%BC%8C%E4%B8%94%E4%B8%8D%E4%BC%9A%E6%97%A5%E5%BF%97%E8%90%BD%E5%90%8E%E5%A4%AA%E5%A4%9A%EF%BC%8C%E5%A2%9E%E5%8A%A0%E5%A4%8D%E6%9D%82%E5%BA%A6%0A2.%20leader%20%E6%8C%82%E4%BA%86%EF%BC%9A%E6%AD%A3%E5%B8%B8%E6%97%A5%E5%BF%97%E4%B8%8D%E5%86%B2%E7%AA%81%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%E8%B7%9F1%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E9%80%90%E6%AD%A5%E5%AE%9A%E4%BD%8D%E5%88%B0%E6%97%A5%E5%BF%97index\_id%E5%8D%B3%E5%8F%AF%EF%BC%8C%E7%89%B9%E6%AE%8A%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%9A%0A%20%20%20%201.%20leader%20%E6%8E%A5%E6%94%B6%E5%88%B0%E4%B8%80%E4%B8%AA%E6%96%B0%E8%AF%B7%E6%B1%82%EF%BC%8C%E5%B9%B6%E5%86%99%E5%85%A5%E6%9C%AC%E6%9C%BA%E4%B8%80%E6%9D%A1%E6%97%A5%E5%BF%97%EF%BC%8C%E9%80%9A%E7%9F%A5follower%E5%A4%8D%E5%88%B6%EF%BC%8C%E4%BD%86%E6%98%AF%E6%AD%A4%E6%97%B6%E6%8C%82%E4%BA%86%EF%BC%8C%E4%BD%86%E8%AF%A5%E6%97%A5%E5%BF%97%E5%B7%B2%E7%BB%8F%E5%86%99%E8%BF%9B%E6%9C%AC%E6%9C%BA%E6%97%A5%E5%BF%97%E4%BA%86%EF%BC%8C%E8%80%8Cfollower%E6%9C%AA%E6%94%B6%E5%88%B0%E8%AF%A5%E6%9D%A1%E6%8C%87%E4%BB%A4%0A%20%20%20%202.%20%E5%BD%93%E9%87%8D%E6%96%B0%E9%80%89%E4%B8%BE%E5%87%BA%E6%96%B0%E7%9A%84follower%E5%90%8E%EF%BC%8Cleader%E5%86%8D%E9%87%8D%E6%96%B0%E5%90%AF%E5%8A%A8%EF%BC%8C%E5%B0%B1%E5%8F%91%E7%8E%B0%E6%97%A5%E5%BF%97%E5%86%B2%E7%AA%81%E4%BA%86%0A%20%20%20%203.%20%E6%AD%A4%E6%97%B6%E4%B9%9F%E8%B7%9F%E4%B8%8A%E9%9D%A2%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E9%80%90%E6%9D%A1%E6%A3%80%E6%9F%A5%E5%88%B0%E6%97%A7leader%E4%B8%8E%E6%96%B0leader%E7%9A%84%E6%97%A5%E5%BF%97index\_id%EF%BC%8C%E7%84%B6%E5%90%8E%E7%94%A8%E6%96%B0leader%E7%9A%84%E6%95%B0%E6%8D%AE%E8%A6%86%E7%9B%96%E6%8E%89%E5%86%B2%E7%AA%81%E7%9A%84%E6%97%A5%E5%BF%97%0A3.%20follower%20%E7%BC%93%E6%85%A2\(%E5%93%8D%E5%BA%94%E4%B9%9F%E6%85%A2%2F%E6%9C%AA%E5%93%8D%E5%BA%94\)%EF%BC%9Aleader%20%E4%B8%8D%E6%96%AD%E5%8F%91%E9%80%81%E9%87%8D%E5%BA%A6%EF%BC%8C%E8%AE%A9%E5%AE%83%E8%BF%BD%E5%8A%A0%E6%97%A5%E5%BF%97%EF%BC%8C%E8%BF%BD%E4%B8%8A%E8%BF%9B%E5%BA%A6%EF%BC%88%E6%AD%A4%E5%A4%84%E4%B8%8E%E5%9B%9E%E5%A4%8D%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AF%B7%E6%B1%82%E5%B9%B6%E4%B8%8D%E5%86%B2%E7%AA%81%EF%BC%89%0A%0A%3E%E4%B8%8A%E9%9D%A2%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E6%9B%B4%E6%8E%A5%E8%BF%91%E7%8E%B0%E5%AE%9E%EF%BC%8C%E6%88%96%E8%80%85%E8%AF%B4%E5%87%BA%E7%8E%B0%E7%9A%84%E6%A6%82%E7%8E%87%E5%A4%A7%E4%B8%80%E4%BA%9B%EF%BC%8C%E6%88%91%E7%94%A8%E5%AE%83%E5%AE%98%E6%96%B9%E7%9A%84demo%E6%A8%A1%E6%8B%9F%E4%B8%8B%E4%BA%86%EF%BC%8C%E5%9F%BA%E6%9C%AC%E4%B8%8A%E5%87%BA%E7%8E%B0%E6%8C%82%E6%9C%BA%E5%90%8E%EF%BC%8C%E6%97%A5%E5%BF%97%E9%83%BD%E6%98%AF%E6%AD%A3%E5%B8%B8%E8%83%BD%E8%A1%A5%E4%B8%8A%E7%9A%84%E3%80%82%E8%80%8C%E4%B8%8B%E9%9D%A2%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E6%98%AF%E9%9D%9E%E5%B8%B8%E6%9E%81%E7%AB%AF%E7%9A%84%E6%83%85%E5%86%B5%0A%0A%23%23%23%23%20%E7%89%B9%E6%AE%8A%E6%97%A5%E5%BF%97%E4%B8%8D%E4%B8%80%E8%87%B4%EF%BC%8C%E5%86%B2%E7%AA%81%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%9A%0Aleader%20%E6%9C%AA%E6%8F%90%E4%BA%A4%EF%BC%8C%E5%B0%B1%E6%8C%82%E4%BA%86%EF%BC%8C%E9%80%A0%E6%88%90%EF%BC%8C%E6%9C%AC%E6%9C%BA%E6%97%A5%E5%BF%97%E4%B8%8E%E6%96%B0leader%E5%86%B2%E7%AA%81%0A%E5%81%87%E8%AE%BE%E5%88%9D%E5%A7%8B%E6%9C%BA%E5%99%A8%20A%20B%20C%20D%20E%20%EF%BC%8C%20A%E4%B8%BAleader%EF%BC%8C%E6%AD%A4%E6%97%B6A%E6%8C%82%E4%BA%86%EF%BC%8CB%20C%20D%20E%20%E9%87%8D%E6%96%B0%E5%8F%88%E9%80%89%E4%B8%BE%EF%BC%8CB%E6%88%90%E4%B8%BA%E4%BA%86%E6%96%B0leader%E3%80%82%0A%E8%BF%99%E6%97%B6A%E5%8F%88%E5%A5%BD%E4%BA%86%EF%BC%8C%E8%BF%99%E9%87%8C%E6%89%80%E6%9C%89follower%EF%BC%88%E4%BB%A5%E5%8F%8A%E6%97%A7leader%EF%BC%89%E5%92%8C%E6%96%B0leader%E7%9A%84%E6%97%A5%E5%BF%97%E5%AF%B9%E6%AF%94%EF%BC%8C%E5%8F%AF%E8%83%BD%E6%9C%89%E5%87%A0%E7%A7%8D%E6%83%85%E5%86%B5%EF%BC%9A%0A1.%20%E4%B8%8E%E6%96%B0leader%E7%9B%B8%E5%90%8C%20%EF%BC%8C%E4%BB%80%E4%B9%88%E9%83%BD%E4%B8%8D%E7%94%A8%E5%81%9A%EF%BC%8C%E6%AD%A3%E5%B8%B8%E9%80%9A%E4%BF%A1%E5%8D%B3%E5%8F%AF%0A2.%20%E6%AF%94%E6%96%B0leader%E7%95%A5%E5%B0%91%E5%87%A0%E4%B8%AA%EF%BC%8C%E4%BD%86%E6%95%B0%E6%8D%AE%E9%83%BD%E7%9B%B8%E5%90%8C%EF%BC%8C%E9%80%9A%E8%BF%87%E6%97%A5%E5%BF%97%E5%AF%B9%E9%BD%90%EF%BC%8C%E8%A1%A5%E4%B8%8A%E5%8D%B3%E5%8F%AF%0A3.%20%E6%AF%94%E6%96%B0leader%E7%9A%84%E6%97%A5%E5%BF%97%E8%BF%98%E8%A6%81%E5%A4%9A%EF%BC%8C%E9%9D%9E%E5%A4%9A%E5%87%BA%E6%9D%A5%E7%9A%84%E6%97%A5%E5%BF%97%E9%83%BD%E6%98%AF%E4%B8%80%E6%A0%B7%E7%9A%84%E3%80%82%0A%20%20%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E5%8F%AF%E8%83%BD%E8%87%AA%E5%B7%B1%E4%B9%8B%E5%89%8D%E6%98%AFleader%EF%BC%8C%E6%89%80%E4%BB%A5%E6%97%A5%E5%BF%97%E6%98%AF%E6%9C%80%E9%95%BF%E7%9A%84%EF%BC%8C%E4%BD%86%E6%98%AF%E5%9B%A0%E4%B8%BA%E6%95%85%E9%9A%9C%E5%AF%BC%E8%87%B4%E8%A2%AB%E6%96%B0leader%E5%BD%93%E9%80%89%E3%80%82%E5%A4%9A%E5%87%BA%E6%9D%A5%E7%9A%84%E5%87%A0%E6%9D%A1%E6%97%A5%E5%BF%97%EF%BC%9A%0A%20%20%20%201.%20%E5%8F%AF%E8%83%BD%E6%98%AF%E6%9C%AA%E6%8F%90%E4%BA%A4%E7%8A%B6%E6%80%81%EF%BC%8C%E8%AE%A9%E6%96%B0leader%E8%A6%86%E7%9B%96%E6%8E%89%EF%BC%9A%E8%87%AA%E5%B7%B1%E5%A4%9A%E5%87%BA%E6%9D%A5%E7%9A%84%E5%87%A0%E6%9D%A1%E6%97%A5%E5%BF%97%0A%20%20%20%202.%20%E5%8F%AF%E8%83%BD%E6%98%AF%E5%B7%B2%E6%8F%90%E4%BA%A4%E7%8A%B6%E6%80%81%EF%BC%8C%E7%84%B6%E5%90%8E%E6%8C%82%E4%BA%86%EF%BC%8C%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E5%A5%BD%E5%83%8F%E8%AE%BA%E6%96%87%20%E9%87%8C%E9%80%89%E6%8B%A9%E4%BA%86%E5%BF%BD%E7%95%A5%20%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%A6%82%E7%8E%87%20%E8%BE%83%E4%BD%8E%20%E3%80%82%E9%82%A3%E5%85%B6%E5%AE%9E%E6%9C%80%E5%90%8E%E4%B9%9F%E4%BC%9A%E8%A2%AB%E8%A6%86%E7%9B%96%E6%8E%89%0A2.%20%E4%B8%8E%E6%96%B0leader%E7%9A%84%E6%97%A5%E5%BF%97%E5%89%8D%E5%8D%8A%E9%83%A8%E5%88%86%E5%B7%AE%E4%B8%8D%E5%A4%9A%EF%BC%8C%E5%8F%AF%E8%83%BD%E5%90%8E%E5%8D%8A%E9%83%A8%E5%88%86%E5%85%A8%E6%98%AF%E9%94%99%E7%9A%84%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF%E5%86%B2%E7%AA%81%E3%80%82%0A%20%20%E8%BF%99%E7%A7%8D%E6%83%85%E5%86%B5%E5%8F%AF%E8%83%BD%E8%87%AA%E5%B7%B1%E4%B9%8B%E5%89%8D%E6%98%AFleader%EF%BC%8C%E7%84%B6%E5%90%8E%E8%B7%9F%E6%95%B4%E4%B8%AA%E7%BD%91%E7%BB%9C%E6%96%AD%E4%BA%86%EF%BC%8C%E5%AE%83%E4%BE%9D%E7%84%B6%E8%A7%89%E5%BE%97%E8%87%AA%E5%B7%B1%E8%BF%98%E6%98%AFleader%EF%BC%8C%0A%20%20%E4%BD%86%E5%A4%9A%E5%87%BA%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E5%85%B6%E5%AE%9E%E9%83%BD%E6%98%AF%E6%9C%AA%E6%8F%90%E4%BA%A4%E7%8A%B6%E6%80%81%20%EF%BC%8C%E5%9B%A0%E4%B8%BA%E5%AE%83%E5%BE%97%E4%B8%8D%E5%88%B0%E4%B8%80%E5%8D%8A%E4%BB%A5%E4%B8%8A%E7%9A%84%E5%93%8D%E5%BA%94%EF%BC%8C%E7%AD%89%E6%96%B0leader%E8%BF%9B%E8%A1%8C%E6%97%A5%E5%BF%97%E5%AF%B9%E9%BD%90%E6%97%B6%EF%BC%8C%E6%8A%8A%E5%86%B2%E7%AA%81%E7%9A%84%E6%95%B0%E6%8D%AE%E8%A6%86%E7%9B%96%E6%8E%89%E5%8D%B3%E5%8F%AF%E3%80%82%0A%0A%23%23%23%23%20%E6%9E%81%E5%85%B6%E7%89%B9%E6%AE%8A%E6%97%A5%E5%BF%97%E4%B8%8D%E4%B8%80%E8%87%B4%EF%BC%8C%E5%86%B2%E7%AA%81%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%9A%0A%E5%81%87%E8%AE%BE%E5%88%9D%E5%A7%8B%E6%9C%BA%E5%99%A8%20A%20B%20C%20D%20E%20%EF%BC%8C%20A%E4%B8%BAleader%0A1.%20A%E6%8C%82%E4%BA%86%EF%BC%8CB%E6%88%90%E4%B8%BA%E4%BA%86%E6%96%B0leader%0A2.%20B%E7%9F%AD%E6%9C%9F%E5%86%85%E5%8F%88%E6%8C%82%E4%BA%86%EF%BC%8CC%E6%88%90%E4%B8%BA%E4%BA%86%E6%96%B0leader%0A3.%20C%E7%9F%AD%E6%9C%9F%E5%8F%88%E6%8C%82%E4%BA%86%EF%BC%8CA%E5%8F%88%E6%88%90%E4%B8%BA%E4%BA%86leader%0A%0A%E8%BF%99%E7%A7%8D%E7%9F%AD%E6%9C%9F%E6%8C%82%E4%BA%86%EF%BC%8C%E7%9F%AD%E6%9C%9F%E5%8F%88%E6%88%90%E4%B8%BA%E6%96%B0leader%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E7%9B%B8%E4%BA%92%E6%88%90%E4%B8%BAleader%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%9C%9F%E9%97%B4%E5%8F%88%E6%9C%89client%E4%B8%8D%E5%81%9C%E7%9A%84%E8%AF%B7%E6%B1%82%EF%BC%8C%E8%82%AF%E5%AE%9A%E4%BC%9A%E9%80%A0%E6%88%90%EF%BC%8C%E5%90%84%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%B8%8A%E7%9A%84%E6%97%A5%E5%BF%97%E5%AE%8C%E5%85%A8%E5%8F%82%E5%B7%AE%E4%B8%8D%E9%BD%90%0A%E5%9B%A0%E4%B8%BA%E5%AE%83%E6%98%AF%E5%BC%BAleader%E6%A8%A1%E5%BC%8F%EF%BC%8C%E8%82%AF%E5%AE%9A%E4%BC%9A%E9%80%A0%E6%88%90%E4%BA%92%E7%9B%B8%E8%A6%86%E7%9B%96%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%85%B7%E4%BD%93%E6%88%91%E4%B9%9F%E6%B2%A1%E5%A4%AA%E6%87%82%E5%8E%9F%E7%90%86%EF%BC%8C%E5%9B%A0%E4%B8%BA%E6%AF%94%E8%BE%83%E6%9E%81%E7%AB%AF%EF%BC%8C%E4%B8%8D%E5%86%99%E4%BA%86%0A%0A%23%23%23%23%20%E6%97%A5%E5%BF%97%E8%A1%A5%E9%BD%90%0A%E4%BB%8E%E8%A6%86%E7%9B%96%E6%96%B9%E5%BC%8F%E7%9C%8B%EF%BC%8C%E5%AE%83%E6%98%AF%E5%BC%BAleader%E6%A8%A1%E5%BC%8F%EF%BC%8C%E6%97%A5%E5%BF%97%E7%9A%84%E6%B5%81%E5%90%91%E5%8F%AA%E8%83%BD%20ledaer%20%3D%3E%20follower%20%2C%20leader%20%E4%B8%8D%E4%BC%9A%E8%B0%83%E6%95%B4%E8%87%AA%E5%B7%B1%E7%9A%84%E6%97%A5%E5%BF%97%0A%E6%AF%94%E5%A6%82%E5%87%BA%E7%8E%B0%E6%97%A5%E5%BF%97%E4%B8%8D%E4%B8%80%E8%87%B4%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%9A%E5%B0%B1%E7%AE%97leader%E6%98%AF%E8%90%BD%E5%90%8E%E7%9A%84%EF%BC%8C%E4%BD%86%E4%BE%9D%E7%84%B6%E8%BF%98%E4%BC%9A%E5%BC%BA%E5%88%B6%E8%AE%A9%E5%85%B6%E5%AE%83follower%E7%94%A8%E8%87%AA%E5%B7%B1%E7%9A%84%E6%97%A5%E5%BF%97%E3%80%82%0A%0A%0A%23%23%23%23%20%E5%A4%8D%E5%88%B6%E6%80%BB%E7%BB%93%0A%20%20%20%20%20%20%20%20%0A%E6%8C%BA%E7%A5%9E%E5%A5%87%E7%9A%84%EF%BC%8C%E5%B1%85%E7%84%B6%E6%98%AF%E5%9F%BA%E4%BA%8E%E5%BF%83%E8%B7%B3%E5%8C%85%E4%BC%A0%E8%BE%93%E6%8C%87%E4%BB%A4%0A%E4%BC%98%E7%82%B9%EF%BC%9A%0A1.%20%E5%87%8F%E5%B0%91%E4%B8%80%E6%AC%A1follower%E4%BC%A0%E8%BE%93\(commit%20ack\)%EF%BC%8C%0A2.%20%E5%87%8F%E5%B0%91%E4%BA%86%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E7%AD%89%E5%BE%85%E6%97%B6%E9%97%B4%20%0A3.%20%E5%BF%83%E8%B7%B3%E6%98%AF%E9%9D%9E%E5%AE%9E%E6%97%B6%E7%9A%84%EF%BC%8C%E4%B8%80%E6%AC%A1%E5%8F%AF%E4%BB%A5%E7%BC%93%E5%AD%98%E5%A4%9A%E6%9D%A1%E6%8C%87%E4%BB%A4%0A%0A%E7%BC%BA%E7%82%B9%E6%98%AF%EF%BC%9A%0A1.%20%E9%9D%9E%E5%AE%9E%E6%97%B6%E7%9A%84%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%BC%9A%E5%87%BA%E7%8E%B0%E6%97%A5%E5%BF%97%E4%B8%8D%E4%B8%80%E8%87%B4%E7%9A%84%E6%83%85%E5%86%B5%0A2.%20%E5%A6%82%E6%9E%9C%E6%9F%90%E4%BA%9Bfollower%E4%B8%80%E7%9B%B4%E4%B8%8D%E5%93%8D%E5%BA%94%EF%BC%8C%E5%AE%83%E5%B0%B1%E4%B8%80%E7%9B%B4%E5%8F%91%EF%BC%8C%E9%98%BB%E5%A1%9E%E4%BA%86client%E7%9A%84%E8%AF%B7%E6%B1%82%0A%0A%E4%B8%8A%E9%9D%A2%E9%81%97%E7%95%99%E4%B8%AA%E9%97%AE%E9%A2%98%EF%BC%9A%E5%A6%82%E6%9E%9C%E8%AF%B4leader%E6%9C%AC%E5%9C%B0%E6%8F%90%E4%BA%A4%20%E6%88%90%E5%8A%9F%EF%BC%8C%E5%9C%A8%E4%B8%8B%E4%B8%AA%E5%BF%83%E4%B8%AD%E5%8C%85%EF%BC%8C%E5%91%8A%E7%9F%A5%E6%89%80follower%E4%B9%9F%E4%B8%80%E5%B9%B6%E6%8F%90%E4%BA%A4%20%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%BF%99%E4%B8%AA%E6%97%B6%E5%80%99%20%E6%8C%82%E4%BA%86%EF%BC%8C%E5%A5%BD%E5%83%8F%E5%AE%98%E6%96%B9%E6%B2%A1%E7%BB%99%E5%87%BA%E5%85%B7%E4%BD%93%E7%9A%84%E5%A4%84%E7%90%86%E6%96%B9%E6%B3%95%0A%E6%88%91%E7%9A%84%E7%90%86%E8%A7%A3%20%EF%BC%9A%E8%BF%99%E7%A7%8D%E5%BC%82%E5%B8%B8%E6%9C%AC%E8%BA%AB%20%E6%A6%82%E7%8E%87%20%E5%B0%8F%EF%BC%8C%E5%8F%A6%E5%A4%96%20%EF%BC%8C%E5%8F%AA%E8%A6%81%E5%AE%83%E8%83%BD%E6%8A%8A%E6%B6%88%E6%81%AF%E5%8F%91%E5%87%BA%E5%8E%BB%EF%BC%8C%E5%A4%A7%E6%A6%82%E7%8E%87%20%E4%BC%9A%E8%A2%AB%E4%B8%80%E5%8D%8A%E6%89%A7%E8%A1%8C%E6%88%90%E5%8A%9F%E3%80%82%0A%0A%E5%8F%A6%E5%A4%96%EF%BC%8C%E4%BB%8E%E5%86%B2%E7%AA%81%E6%9D%A5%E7%9C%8B%EF%BC%8C%E6%98%AF%E6%9C%89%E4%B8%80%E5%AE%9A%E6%A6%82%E7%8E%87%E9%80%A0%E6%88%90%E6%9C%80%E5%90%8E%E4%B8%80%E6%AC%A1%E5%BF%83%E8%B7%B3%E4%B8%A2%E5%A4%B1%E6%9C%80%E5%90%8E%E5%87%A0%E6%9D%A1%E6%8C%87%E4%BB%A4%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E6%84%9F%E8%A7%89%E6%AF%94%E8%BE%83%E9%87%8F%E4%B8%8D%E5%A4%AA%E5%A4%A7%EF%BC%8C%E4%B9%9F%E8%83%BD%E6%8E%A5%E5%8F%97%E3%80%82%0A%0A%E5%AE%89%E5%85%A8%0A\-\-\-\-%0A%E5%AE%89%E5%85%A8%EF%BC%9A%E5%AF%B9%E8%BE%B9%E7%95%8C%E5%80%BC%E7%9A%84%E5%A4%84%E7%90%86%E4%B8%8E%E4%BC%98%E5%8C%96%0A%0A%E7%BD%91%E7%BB%9C%E5%88%86%E9%9A%94%EF%BC%9A%0A%E5%81%87%E8%AE%BE%E7%8E%B0%E5%9C%A8%E6%9C%895%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%9AA%20B%20C%20D%20E%20%0A%E5%BC%80%E5%A7%8B%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%9AA%E4%B8%BAleader%2C%20b%20c%20d%20e%20%E4%B8%BAfollower%0A%E8%BF%99%E4%B8%AA%E6%97%B6%E5%80%99a%20b%20%E4%B8%8E%20c%20d%20e%20%E5%87%BA%E7%8E%B0%E7%BD%91%E7%BB%9C%E5%88%86%E9%9A%94%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%9B%A0%E4%B8%BA%E8%BF%99%E4%B8%A4%E7%BB%84%E4%B8%8D%E8%83%BD%E9%80%9A%E4%BF%A1%EF%BC%8CA%E4%BE%9D%E7%84%B6%E8%AE%A4%E4%B8%BA%E8%87%AA%E5%B7%B1%E6%98%AFleader%EF%BC%8CB%E4%B9%9F%E4%BE%9D%E7%84%B6%E8%83%BD%E6%94%B6%E5%88%B0A%E7%9A%84%E6%B6%88%E6%81%AF%EF%BC%8C%E8%AF%81%E6%98%8E%E4%B8%80%E5%88%87%E6%AD%A3%E5%B8%B8%E3%80%82C%20D%20E%20%E5%9B%A0%E4%B8%BA%E6%94%B6%E5%88%B0leader%E5%BF%83%E8%B7%B3%EF%BC%8C%E9%87%8D%E6%96%B0%E5%8F%91%E8%B5%B7%E9%80%89%E4%B8%BE%EF%BC%8C%E5%81%87%E8%AE%BEC%E6%88%90%E4%B8%BA%E4%BA%86%E6%96%B0%E7%9A%84leader%0A%0A%E9%82%A3%E4%B9%88%EF%BC%8C%E7%8E%B0%E5%9C%A8%EF%BC%9AC%20D%20E%20%E6%98%AF%E4%B8%80%E7%BB%84%E6%96%B0%E7%9A%84%E9%9B%86%E7%BE%A4%EF%BC%8C%E4%B8%94%E6%98%AF%E6%9C%80%E6%96%B0%EF%BC%8C%E6%9C%80%E6%AD%A3%E7%A1%AE%E7%9A%84%E3%80%82A%20B%20%E6%98%AF%E6%97%A7%E7%9A%84term%E9%9B%86%E7%BE%A4%EF%BC%8C%E7%8E%B0%E5%9C%A8%E8%80%83%E8%99%91%EF%BC%9A%0A1.%20%E5%A6%82%E6%9E%9C%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%A6%81%E5%86%99%E5%85%A5%E6%93%8D%E4%BD%9C%EF%BC%8C%E8%AF%B7%E6%B1%82A%20%EF%BC%8C%E9%82%A3%E4%B9%88A%20%E6%94%B6%E4%B8%8D%E5%88%B0%E5%8F%A6%E5%A4%96C%20D%20E%20%E7%9A%84%E7%A1%AE%E8%AE%A4%E6%B6%88%E6%81%AF%EF%BC%8C%E9%82%A3%E4%B9%88%E5%86%99%E5%85%A5%E8%82%AF%E5%AE%9A%E6%98%AF%E5%A4%B1%E8%B4%A5%0A2.%20%E5%A6%82%E6%9E%9C%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%A6%81%E8%AF%BB%E5%85%A5%E6%93%8D%E4%BD%9C%EF%BC%8C%E8%AF%B7%E6%B1%82A%20%E6%88%96%20B%EF%BC%8C%E9%82%A3%E4%B9%88%E5%B0%B1%E6%9C%89%E9%97%AE%E9%A2%98%20%0A%0A%0A%E9%9B%86%E7%BE%A4%E6%88%90%E5%91%98%E5%8F%98%E6%9B%B4%0A\-\-\-\-%0A%E9%85%8D%E7%BD%AE%E7%83%AD%E6%9B%B4%E6%96%B0%2F%E5%8A%A8%E6%80%81%E7%BB%B4%E6%8A%A4%E9%9B%86%E7%BE%A4%E8%8A%82%E7%82%B9%EF%BC%8C%E6%97%A0%E9%9C%80%E5%81%9C%E6%AD%A2%E6%95%B4%E4%B8%AA%E9%9B%86%E7%BE%A4%0A%0A%E6%9C%89%E4%B8%AA%E5%89%8D%E6%8F%90%EF%BC%9A%E4%B8%8D%E5%A4%AA%E5%8F%AF%E8%83%BD%E5%90%8C%E4%B8%80%E6%97%B6%E9%97%B4%E5%8F%98%E6%9B%B4%E9%9B%86%E7%BE%A4%E6%89%80%E6%9C%89%E6%9C%BA%E5%99%A8%E7%9A%84%E9%85%8D%E7%BD%AE%0A%0A%E8%84%91%E8%A3%82%EF%BC%9A%E5%90%8C%E4%B8%80%E6%97%B6%E9%97%B4%E6%9C%89%E4%B8%A4%E4%B8%AAleader%EF%BC%8C%E5%81%87%E8%AE%BE%E6%9C%89A%20B%20C%203%E5%8F%B0%E6%9C%BA%E5%99%A8%EF%BC%8C%E6%AD%A4%E6%97%B6%E5%A2%9E%E5%8A%A0%20D%20E%20%EF%BC%8C%E4%B8%80%E5%85%B15%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%BA%86%E3%80%82%E5%81%87%E8%AE%BE%E7%8E%B0%E5%9C%A8%E6%9B%B4%E6%96%B0%E6%98%AF%E4%B8%8D%E5%90%8C%E6%AD%A5%E7%9A%84%EF%BC%8CA%20B%20%E8%BF%98%E6%98%AF%E6%97%A7%E9%85%8D%E7%BD%AE\(%E4%BB%A5%E4%B8%BA%E6%98%AF%E5%85%B1%E8%AE%A13%E4%B8%AA%E8%8A%82%E7%82%B9\)%EF%BC%8CC%20D%20E%20%E5%B7%B2%E7%BB%8F%E5%88%87%E6%8D%A2%E6%88%90%E4%B8%BA%E6%96%B0%E7%9A%84%E9%85%8D%E7%BD%AE%E4%BA%86\(%E7%9F%A5%E9%81%93%E6%98%AF5%E4%B8%AA%E8%8A%82%E7%82%B9\)%EF%BC%8C%E8%BF%99%E4%B8%AA%E6%97%B6%E5%80%99%E5%A6%82%E6%9E%9Cleader%E6%8C%82%E4%BA%86%EF%BC%8CA%20B%20%E4%BB%A5%E4%B8%BA%E8%BF%98%E6%98%AF3%E4%B8%AA%E8%8A%82%E7%82%B9%EF%BC%8C%E5%AE%83%E8%83%BD%E9%80%89%E5%87%BAA%E5%BD%93leader%EF%BC%8CC%20D%20E%20%E8%99%BD%E7%84%B6%E7%9F%A5%E9%81%93%E6%98%AF5%E4%B8%AA%E8%8A%82%E7%82%B9%EF%BC%8C%E4%BD%86%E6%98%AF%E5%AE%83%E4%BB%AC%E7%9A%84%E9%80%89%E7%A5%A8%E8%BF%87%E5%8D%8A%EF%BC%8C%E4%BC%9A%E5%86%8D%E9%80%89%E5%87%BA%E4%B8%80%E4%B8%AAleader%EF%BC%8C%E9%82%A3%E4%B9%88%E5%B0%B1%E6%98%AF%E4%B8%A4%E4%B8%AAleader%E5%B9%B3%E8%A1%8C%E5%AD%98%E5%9C%A8%E4%BA%86%E3%80%82%0A%0A%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95%EF%BC%9A%E8%81%94%E5%90%88%E4%B8%80%E8%87%B4%E7%8A%B6%E6%80%81%0A%0A%0A%E8%BF%98%E6%98%AF%E4%BD%BF%E7%94%A8%20AppendEntries%20%E6%B6%88%E6%81%AF%E4%BD%93%0A%0A%0A%E8%BF%87%E7%A8%8B%EF%BC%9A%0A%E5%81%87%E8%AE%BE%E5%8E%9F%E6%9C%89%20%20A%20B%20C%20%EF%BC%8CA%20%E4%B8%BAleader%20%EF%BC%8C%20%E5%A2%9E%E5%8A%A0%20D%20E%0Araft%20%E4%BC%9A%E5%85%88%E6%8A%8Ad%20e%20%E8%AE%BE%E7%BD%AE%E4%B8%BA%E5%8F%AA%E8%AF%BB%E7%8A%B6%E6%80%81%EF%BC%8C%E7%84%B6%E5%90%8E%EF%BC%8C%E5%BC%80%E5%A7%8B%E6%97%A5%E5%BF%97%E8%BF%BD%E8%A1%A5%EF%BC%8C%E7%9B%B4%E5%88%B0%E8%BF%BD%E4%B8%8Aleader\-A%0A%E5%81%87%E8%AE%BED%20E%20%E8%BF%BD%E8%A1%A5%E5%AE%8C%E6%88%90%EF%BC%8C%E7%8E%B0%E4%BB%BBleader\-A%EF%BC%8C%E5%8F%91%E8%B5%B7%EF%BC%9ACold%20new%0ACold%3A%E6%97%A7%E9%85%8D%E7%BD%AE%0Anew%3A%E6%96%B0%E9%85%8D%E7%BD%AE%0A%E6%AD%A4%E6%97%B6D%20E%20%20%E5%BF%85%E9%A1%BB%E6%BB%A1%E8%B6%B3Cold%20new%20%E4%B8%A4%E4%B8%AA%E9%85%8D%E7%BD%AE%E4%BF%A1%E6%81%AF%EF%BC%8C%E5%81%9A%E9%80%89%E4%B8%BE%E9%83%BD%E6%88%90%E5%8A%9F%E4%BA%86%EF%BC%8C%E6%89%8D%E7%AE%97%E6%88%90%E5%8A%9F%0A%0A%0A%0A%0A%0A%0A
