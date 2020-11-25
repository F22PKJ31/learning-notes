> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6890926320297967623)

单条 SQL 语句执行时，会被当成一个事务提交吗？
-------------------------

以下内容摘自 《高性能 MySQL》(第 3 版)

> “
> 
> MySQL 默认采用自动提交（AUTOCOMMIT）模式。也就是说，如果不是显式地开始一个事务，则每个查询都被当作一个事务执行提交操作。在当前连接中，可以通过设置 AUTOCOMMIT 变量来启用或者禁用自动提交模式
> 
> ”

![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfebeccb02f34535bb8cee86558b4a9d~tplv-k3u1fbpfcp-zoom-1.image)

MySQL 是如何实现事务的 ACID 的？
----------------------

事务具有 ACID 四大特性，那么 MySQL 是如何实现事务的这四个属性的呢？

*   原子性 要么全部成功，要么全部失败。MySQL 是通过记录 undo_log 的方式来实现的原子性。undo_log 即**回滚日志**，在真正的 SQL 执行之前先将 undo_log 写入磁盘，然后再对数据库的数据进行操作。如果发生异常或回滚，就可以依据 undo_log 进行反向操作，恢复数据在事务执行之前的样子。
    
*   持久性 事务一旦被正常提交，它对数据库的影响就应该是永久的。此时即使系统崩溃，修改的数据也不会丢失。InnoDB 作为 MySQ L 的存储引擎，数据是存放在磁盘中的，但如果每次读写数据都需要磁盘 IO，效率会很低。为此，InnoDB 提供了缓存 (Buffer Pool)，作为访问数据库的缓冲：当从数据库读取数据时，会首先从 Buffer Pool 中读取，如果 Buffer Pool 中没有，则从磁盘读取后放入 Buffer Pool ；当向数据库写入数据时，会首先写入 Buffer Pool，Buffer Pool 中修改的数据会定期刷新到磁盘中。
    
    这样的设计也带来了相应的问题：如果数据提交了，这时数据还在缓冲池里（还没刷盘），此时 MySQL 宕机、断电了怎么办？数据会不会丢失？
    
    答案是不会，MySQL 通过 redo_log 的机制，保证了持久性。redo_log 即**重做日志**，简单说就是当数据修改时，除了修改 Buffer Pool 中的数据，还会在 redo_log 记录这次操作；当事务提交时，会调用 fsync 接口对 redo_log 进行刷盘。如果 MySQL 宕机，重启时可以读取 redo_log 中的数据，对数据库进行恢复。
    
*   隔离性
    
    隔离性是 ACID 里面最复杂的一个，这里面涉及到隔离级别的概念，一共有四个
    
    *   Read uncommitted
        
    *   Read committed
        
    *   Repeatable read
        
    *   Serializable
        
    
    **简单说隔离级别就是规定了：一个事务中数据的修改，哪些事务之间可见，哪些不可见。而隔离性就是要管理多个并发读写请求的访问顺序。**
    
    MySQL 对于隔离性的具体实现我们后面会展开说。
    
*   一致性
    
    通过回滚、恢复和在并发环境下的隔离做到一致性。
    

事务并发可能导致的问题
-----------

通过上个问题我知道单条 DDL 执行也会被当成一个事务自动提交，那么无论是多条 SQL 并发，还是多个自己手动组织的包含多条 SQL 的事务并发，都会导致事务并发问题。

具体来说有：

*   脏写 （一个事务提交的数据覆盖了另一个事务未提交的数据）
    
*   脏读 （一个事务读取到另一个事务未提交的数据）
    
*   不可重复读 （重点在于 update 和 delete 一个事务内多次读取的数据不一样）
    
*   幻读 （重点在于 insert 一个事务内多次读取的记录数不一样）
    

上面我们提到了事务的隔离级别，MySQL 的所有隔离级别都能保证不产生脏写，所以就剩下脏读、不可重复读和幻读的问题了。

下面具体看下各隔离级别是如何解决或未解决上面这些问题的：

![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48edff187dc74af29b68990b79a3e3a1~tplv-k3u1fbpfcp-zoom-1.image)

### Read uncommitted

未提交读，这个级别在读的过程中不会加任何锁，只在写请求时加锁，所以写操作在读的过程中修改数据，就会造成脏读。也自然会产生不可重复读和幻读。

### Read committed

已提交读，与未提交读一样也是读不加锁，写加锁。不一样的是利用了 MVCC 机制避免了脏读的问题，同样会有不可重复读和幻读的问题。关于 MVCC 我们后面会详细说。

### Repeatable read

MySQL 默认的隔离级别，在这个级别 MySQL 利用两种方式解决问题

1.  读写锁 读读并行时加读锁，读读是共享锁的。 只要有写请求就加写锁，这样读写是串行的。 读取数据时加锁，其它事务无法修改这些数据。所以不会产生不可重复读。 修改删除数据时也要加锁，其它事务无法读取这些数据，所以不会产生脏读。 第一种方式就是我们常说的 **“悲观锁”**，数据在整个事务处理过程中处于锁定状态，比较保守，性能开销比较大。
    
2.  MVCC （后面讲）
    

此外还利用了 Next-Key 锁 在一定程度上解决了幻读的问题。关于这个我们后面再说。

### Serializable

在该隔离级别下事务都是串行顺序执行的。 如果禁用了自动提交，则 InnoDB 会将所有普通的 SELECT 语句隐式转换为 SELECT ... LOCK IN SHARE MODE。即给读操作隐式加一把读共享锁，从而避免了脏读、不可重读复读和幻读问题。

MVCC
----

> “
> 
> Multiversion concurrency control (MCC or MVCC), is a concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory
> 
> ”

翻译过来就是：多版本并发控制（MCC 或 MVCC）是一种并发控制方法，通常被数据库管理系统用来提供对数据库的并发访问，并以编程语言来实现事务存储。

**简单来说就是数据库用来控制并发的一种方法。每个数据库对于 MVCC 的实现可能不一样。**

以我们常用的 MySQL 来说，MySQL 的 InnoDB 引擎实现了 MVCC 。

### MVCC 能解决什么问题

从上面的定义我们能看出，MVCC 主要解决事务并发时数据一致性的问题

### InnoDB 是如何实现的 MVCC

下面这个图来自《高性能 MySQL》(第 3 版) ![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66d993429a804e5f844c787343159f2c~tplv-k3u1fbpfcp-zoom-1.image)

这本书写的很好，翻译的也不错，我对于 MySQL 最初的系统性认识也是因为读了这本书，然而在对于 MVCC 是如何实现的讲述上，个人认为是有些问题的。

**来看下哪里有问题**

*   首先看下 MySQL 的官方文档，我对比了 5.1、5.6、5.7 三个版本的 ，对 MVCC 这部分的描述，几乎是相同的。
    
    ![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e76cf70759445108167f56fa7c4cf96~tplv-k3u1fbpfcp-zoom-1.image)
    

根据文档很明显是在每条数据增加三个隐藏列：

*   6 字节的 DB_TRX_ID 字段，表示最近一次插入或者更新该记录的事务 ID。
    
*   7 字节的 DB_ROLL_PTR 字段，指向该记录的 rollback segment 的 undo log 记录。
    
*   6 字节的 DB_ROW_ID，当有新数据插入的时候会自动递增。当表上没有用户主键的时候，InnoDB 会自动产生聚集索引，包含 DB_ROW_ID 字段。
    

这里我补充一张包含 rollback segment 的 MySQL 内部结构图 ![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bc0e09f88de491eb469b3574b899c0f~tplv-k3u1fbpfcp-zoom-1.image)

**版本链**

之前我们讲过 undo_log 的概念，每条 undo 日志都有一个 roll_pointer 属性，那么所有的版本都会被 roll_pointer 属性连接成一个链表，我们把这个链表称之为版本链，版本链的头节点就是当前记录最新的值。

**ReadView**

通过隐藏列和版本链，MySQL 可以将数据恢复到指定版本；但是具体要恢复到哪个版本，则需要根据 ReadView 来确定。所谓 ReadView，是指事务（记做事务 A）在某一时刻给整个事务系统（trx_sys）打快照，之后再进行读操作时，会将读取到的数据中的事务 id 与 trx_sys 快照比较，从而判断数据对该 ReadView 是否可见，即对事务 A 是否可见。（）

**至此我们发现 MVCC 就是基于隐藏字段、undo_log 链和 ReadView 来实现的。**

### Read committed 中的 MVCC

前面我们讲过 Read committed 隔离级别中使用 MVCC 解决脏读问题。 这里我参考了两篇文章：

*   https://cloud.tencent.com/developer/article/1150633
    
*   https://cloud.tencent.com/developer/article/1150630
    

InnoDB 只会查找版本早于当前事务版本的数据行（也就是，行的版本号小于或是等于事务的系统版本 号），这样可以确保数据读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或修改过的。**因此不会产生脏读**。

Read committed 隔离级别下出现不可重复读是由于 read view 的生成机制造成的。在 Read committed 级别下，只要当前语句执行前已经提交的数据都是可见的。在每次语句执行的过程中，都关闭 read view, 重新创建当前的一份 read view。这样就可以根据当前的全局事务链表创建 read view 的事务区间。简单说就是在 Read committed 隔离级别下，MVCC 在每次 select 时生成一个快照版本，所以每次 select 都会读到不同的版本数据，**所以会产生不可重复读**。

### Repeatable read 中的 MVCC

Repeatable read 隔离级别解决了不可重复读的问题，一个事务中多次读取不会出现不同的结果，保证了可重复读。前文中我们说 Repeatable read 有两种实现方式，一种是悲观锁的方式，相对的 MVCC 就是乐观锁的方式。

Repeatable read 隔离级别能解决不可重复读根本原因其实就是 read view 的生成机制和 Read committed 不同。

*   Read committed ：只要是当前语句执行前已经提交的数据都是可见的。
    
*   Repeatable read ：只要是当前事务执行前已经提交的数据都是可见的。
    

不像 Read committed，在 Repeatable read 的隔离级别下，创建事务的时候，就生成了当前的 global read view, 一直维持到事务结束。这样就能实现可重复读。

幻读与 Next-Key 锁
--------------

### 当前读与快照读

通过 MVCC 机制，虽然让数据变得可重复读，但我们读到的数据可能是历史数据，是不及时的数据，不是数据库当前的数据！对于这种读取历史数据的方式，我们叫它**快照读** (snapshot read)，而读取数据库当前版本数据的方式，叫**当前读** (current read)

*   快照读：就是 select
    
    *   select * from table ….;
        
    
*   当前读：特殊的读操作，插入 / 更新 / 删除操作，属于当前读，处理的都是当前的数据，需要加锁。
    
    *   select * from table where ? lock in share mode;
        
    *   select * from table where ? for update;
        
    *   insert;
        
    *   update ;
        
    *   delete;
        
    

### 解决幻读

为了解决**当前读**中的幻读问题，MySQL 事务使用了 next-key lock 。

![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/208f0fb4a62545d2817e7b0f235fc03f~tplv-k3u1fbpfcp-zoom-1.image)

Repeatable read 通过 next-key lock 机制避免了幻读现象。

InnoDB 存储引擎有 3 种行锁的算法，分别是：

*   Record Lock: 单个记录上的锁
    
*   Gap Lock: 间隙锁，锁定一个范围，但不包括记录本上
    
*   Next-Key Lock: Gap Lock + Record Lock
    

next-key lock 是行锁的一种，实现相当于 record lock(记录锁) + gap lock(间隙锁)；其特点是不仅会锁住记录本身 ( record lock 的功能)，还会锁定一个范围 ( gap lock 的功能)。

当 InnoDB 扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。

当查询的索引含有唯一属性的时候，Next-Key Lock 会进行优化，将其降级为 Record Lock，即仅锁住索引本身，不是范围。

下图引用自 ![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77c69df7e1d24707943560205f896b0f~tplv-k3u1fbpfcp-zoom-1.image)