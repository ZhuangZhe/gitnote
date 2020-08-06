# MVCC

**多版本并发控制**

**多版本控制**: 指的是一种提高并发的技术。最早的数据库系统，只有读读之间可以并发，读写，写读，写写都要阻塞。引入多版本之后，只有写写之间相互阻塞，其他三种操作都可以并行，这样大幅度提高了InnoDB的并发度。在内部实现中，与Postgres在数据行上实现多版本不同，InnoDB是在undolog中实现的，通过undolog可以找回数据的历史版本。找回的数据历史版本可以提供给用户读\(按照隔离级别的定义，有些读请求只能看到比较老的数据版本\)，也可以在回滚的时候覆盖数据页上的数据。在InnoDB内部中，会记录一个全局的活跃读写事务数组，其主要用来判断事务的可见性。

* MySQL的大多数事务型存储引擎实现的其实都不是简单的行级锁。**基于提升并发性能的考虑**, 它们一般都同时实现了多版本并发控制\(MVCC\)。不仅是MySQL, 包括Oracle,PostgreSQL等其他数据库系统也都实现了MVCC, 但各自的实现机制不尽相同, 因为MVCC没有一个统一的实现标准。
* 可以认为MVCC是行级锁的一个变种, 但是它在很多情况下避免了加锁操作, 因此开销更低。虽然实现机制有所不同, 但大都实现了非阻塞的读操作，写操作也只锁定必要的行。
* MVCC的实现方式有多种, 典型的有乐观\(optimistic\)并发控制和悲观\(pessimistic\)并发控制。
* MVCC只在`READ COMMITTED`和`REPEATABLE READ`两个隔离级别下工作。其他两个隔离级别够和MVCC不兼容, 因为 `READ UNCOMMITTED` 总是读取最新的数据行, 而不是符合当前事务版本的数据行。而 `SERIALIZABLE` 则会对所有读取的行都加锁。

## 相关概念

### `read view`**、**`快照snapshot`

此文虽然是以PostgreSQL进行的说明, 但并不影响理解, 在"事务快照的实现"该部分有细节需要注意:

* 事务快照是用来存储数据库的事务运行情况。
* 一个事务快照的创建过程可以概括为：查看当前所有的未提交并活跃的事务，存储在数组中选取未提交并活跃的事务中最小的XID，记录在快照的xmin中**选取所有已提交事务中最大的XID，加1后记录在xmax中。**

PostgreSQL中snapshot的概念, 对应MySQL中, 其实就是你在网上看到的`read view`**。**

`read view`主要是用来做可见性判断的, 比较普遍的解释便是"本事务不可见的当前其他活跃事务"。

对于read view快照的生成时机, 也非常关键, **正是因为生成时机的不同, 造成了RC,RR两种隔离级别的不同可见性：**

* 在innodb中\(默认repeatable read级别\), 事务在begin/start transaction之后的第一条select读操作后, 会创建一个快照\(read view\), 将当前系统中活跃的其他事务记录记录起来。
* 在innodb中\(默认repeatable committed级别\), 事务中每条select语句都会创建一个快照\(read view\)。

### undo-log

* Undo-log是InnoDB MVCC事务特性的重要组成部分。当我们对记录做了变更操作时就会产生undo记录，Undo记录默认被记录到系统表空间\(ibdata\)中，但从5.6开始，也可以使用独立的Undo 表空间。
* Undo记录中存储的是老版本数据，当一个旧的事务需要读取数据时，为了能读取到老版本的数据，需要顺着undo链找到满足其可见性的记录。当版本链很长时，通常可以认为这是个比较耗时的操作（例如bug\#69812）。
* 大多数对数据的变更操作包括INSERT/DELETE/UPDATE，其中INSERT操作在事务提交前只对当前事务可见，因此产生的Undo日志可以在事务提交后直接删除（谁会对刚插入的数据有可见性需求呢！！），而对于UPDATE/DELETE则需要维护多版本信息，在InnoDB里，UPDATE和DELETE操作产生的Undo日志被归成一类，即update\_undo
* 另外, 在回滚段中的undo logs分为: `insert undo log` 和 `update undo log`
  * insert undo log : 事务对insert新记录时产生的undolog, 只在事务回滚时需要, 并且在事务提交后就可以立即丢弃。
  * update undo log : 事务对记录进行delete和update操作时产生的undo log, 不仅在事务回滚时需要, 一致性读也需要，所以不能随便删除，只有当数据库所使用的快照中不涉及该日志记录，对应的回滚日志才会被purge线程删除。

**参考资料：**

* [MySQL-InnoDB-MVCC多版本并发**控制**](https://segmentfault.com/a/1190000012650596)\*\*\*\*
* [数据库内核月报 － 2017 / 12](http://mysql.taobao.org/monthly/2017/12)



