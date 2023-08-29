事务是必须满足4个条件:（ACID）

原子性（Autmic）：要么不做，要么全做

一致性（Consistency）：事务的操作应该使数据库从一个一致状态转变倒另一个一致得状态！

例：网上购物，你只有即让商品出库，又让商品进入顾客得购物篮才能构成

事务！

隔离性（Isolation）：如果多个事务并发执行，应象各个事务独立执行一样！

持久性（Durability）：一个成功执行得事务对数据库得作用是持久得，即使数据库应故障出错，也应该能够恢复！

SET AUTOCOMMIT=0//设置为不自动提交，因为MYSQL默认立即执行

BEGIN

ROOLBACK

COMMIT

查看当前会话隔离级别:select @@tx_isolation;

查看系统当前隔离级别:select @@global.tx_isolation;

设置当前会话隔离级别:set session transaction isolation level repeatable read;

设置系统当前隔离级别:set global transaction isolation level repeatable read;

低级别的隔离级一般支持更高的并发处理，并拥有更低的系统开销。

Read Uncommitted（可能出现脏读）：最低的隔离级别。一个事务可以读取另一个事务并未提交的更新结果。

脏读：事务A读取了被另一个事务B修改，但是还未提交的数据。假如B回退，则事务A读取的是无效的数据。这跟不可重复读类似，但是第二个事务不需要执行提交。

Read Committed（不可重复读）：大部分数据库采用的默认隔离级别。一个事务的更新操作结果只有在该事务提交之后，另一个事务才可以的读取到同一笔数据更新后的结果。

可避免脏读

不可重复读：在一个事务中,两次查询的结果不一致(针对的update操作)

select时不添加读锁，就会发生不可重复读问题。

Repeatable Read（可重读），mySQL的默认事务隔离级别，理论上，重读会看到同样的数据行，不过，这会导致另一个棘手的问题：幻读 （Phantom Read）。所以，得加琐，就是支持重读。

可避免脏读，可避免不可重复读，但会出现幻读

幻读：在一个事务中,两次查询的结果不一致(针对的insert操作)

Serializable：这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

MVCC

Multi-Version Concurrency Control：多版本并发控制，相对的就是基于锁的并发控制

读操作：快照读、当前读

快照读：读取的是可见的版本，加的共享锁。如select xxx (for update 除外)

当前读：读取最新版本，会加排它锁。 update delete insert

当前读 例子，update：

1 MYSQL 根据 where -> innodb -> 找到满足条件的数据记录 ->返回 mysql 记录，并加锁

2 MYSQL 发送更新数据 -> innodb- > 更新完成 解锁 -> 返回mysql

update 、delete差不多，insert操作可能会触发Unique Key的冲突检查，也会进行一个当前读

所以，一个MYSQL用户你禁了select操作，update 一样没权限