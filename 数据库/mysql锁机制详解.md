# 按锁的粒度划分
- 表锁：意向锁(IS锁、IX锁)、自增锁；
- 行锁：记录锁、间隙锁、临键锁、插入意向锁；

# 共享/排它锁(Shared and Exclusive Locks)
- 共享锁（Share Locks，记为S锁），读取数据时加S锁 (共享锁之间不互斥，简记为：读读可以并行)
- 排他锁（eXclusive Locks，记为X锁），修改数据时加X锁 (排他锁与任何锁互斥，简记为：写读，写写不可以并行)
- 排他锁，一旦写数据的任务没有完成，数据是不能被其他任务读取的，这对并发度有较大的影响。对应到数据库，可以理解为，写事务没有提交，读相关数据的select也会被阻塞，这里的select是指加了锁的，普通的select仍然可以读到数据(快照读)。

# 意向锁(Intention Locks)
> InnoDB为了支持多粒度锁机制(multiple granularity locking)，即允许行级锁与表级锁共存，而引入了意向锁(intention locks)。意向锁是指，未来的某个时刻，事务可能要加共享/排它锁了，先提前声明一个意向。

1. 意向锁是一个表级别的锁(table-level locking)；
2. 意向锁又分为：
    - 意向共享锁(intention shared lock, IS)，它预示着，事务有意向对表中的某些行加共享S锁；
    - 意向排它锁(intention exclusive lock, IX)，它预示着，事务有意向对表中的某些行加排它X锁；

# 记录锁(Record Locks)
- 记录锁，它封锁索引记录

# 间隙锁(Gap Locks)
- 间隙锁的主要目的，就是为了防止其他事务在间隔中插入数据，以导致“不可重复读”。如果把事务的隔离级别降级为读提交(Read Committed, RC)，间隙锁则会自动失效。
- 间隙锁，它封锁索引记录中的间隔，或者第一条索引记录之前的范围，又或者最后一条索引记录之后的范围

# 临键锁(Next-key Locks)
- 临键锁，是记录锁与间隙锁的组合，它的封锁范围，既包含索引记录，又包含索引区间。
- 默认情况下，innodb使用next-key locks来锁定记录。但当查询的索引含有唯一属性的时候，Next-Key Lock 会进行优化，将其降级为Record Lock，即仅锁住索引本身，不是范围。

# 插入意向锁(Insert Intention Locks)
- 对已有数据行的修改与删除，必须加强互斥锁(X锁)，那么对于数据的插入，是否还需要加这么强的锁，来实施互斥呢？插入意向锁，孕育而生。
- 插入意向锁，是间隙锁(Gap Locks)的一种（所以，也是实施在索引上的），它是专门针对insert操作的。多个事务，在同一个索引，同一个范围区间插入记录时，如果插入的位置不冲突，不会阻塞彼此。

# 自增锁(Auto-inc Locks)
- 自增锁是一种特殊的表级别锁（table-level lock），专门针对事务插入AUTO_INCREMENT类型的列。最简单的情况，如果一个事务正在往表中插入记录，所有其他事务的插入必须等待，以便第一个事务插入的行，是连续的主键值。

#参考
- [mysql锁机制详解](https://www.cnblogs.com/volcano-liu/p/9890832.html)
- [MySQL锁机制与用法分析](https://www.jb51.net/article/139113.htm)
- [MySQL InnoDB锁机制全面解析分享](https://segmentfault.com/a/1190000014133576)
- [乐观锁与悲观锁——解决并发问题](https://www.cnblogs.com/0201zcr/p/4782283.html)
- [MySQL学习笔记（四）悲观锁与乐观锁](https://www.cnblogs.com/tinywan/p/9655664.html)