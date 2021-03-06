# 事务

事务（Transaction）是并发控制的基本单位。**所谓的事务，它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。**例如，银行转账工作：从一个账号扣款并使另一个账号增款，这两个操作要么都执行，要么都不执行。所以，应该把它们看成一个事务。事务是数据库维护数据一致性的单位，在每个事务结束时，都能保持数据一致性。

我们以Msql数据库的操作为例，再进一步解释一下数据库事务： 首先我们用以下命令查看该Mysql会话的事务隔离级别，关于事务隔离级别及其作用，我们在后面的章节中会进行详细介绍，这里只要简单知道数据库可以设置不同的事务隔离级别，不同的隔离级别会对事务的操作产生不同的效果即可。使用以下命令可以查询当前Mysql会话的事务隔离级别，可以看到，Mysql默认的事务隔离级别是REPEATABLE-READ。

```bash
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

为了用实例来解释事务，我们创建了如下的bank数据表，并插入一条数据，

```bash
mysql> describe bank;
+---------+---------------+------+-----+---------+----------------+
| Field   | Type          | Null | Key | Default | Extra          |
+---------+---------------+------+-----+---------+----------------+
| id      | int(11)       | NO   | PRI | NULL    | auto_increment |
| name    | varchar(40)   | NO   |     | NULL    |                |
| balance | decimal(10,2) | YES  |     | NULL    |                |
+---------+---------------+------+-----+---------+----------------+

mysql> select * from bank;
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  3 | fufu | 2000.00 |
+----+------+---------+
```

使用start transaction命令开启数据库事务，

```bash
mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)
```

更新id为3的行的balance值为3000.00，

```bash
mysql> update bank set balance = 3000 where id = 3;
Query OK, 1 row affected (0.09 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from bank;
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  3 | fufu | 3000.00 |
+----+------+---------+
1 row in set (0.00 sec)
```

此时我们可以看到，select语句查询到的id为3的行的balance值已经修改为3000.00，接下来我们再尝试插入一条新数据，

```bash
mysql> insert into bank (name, balance) values ('melo', 1000);
Query OK, 1 row affected (0.06 sec)

mysql> select * from bank;
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  3 | fufu | 3000.00 |
|  4 | melo | 1000.00 |
+----+------+---------+
2 rows in set (0.00 sec)
```

由于以上的update和insert操作都是**在start transaction命令开启事务之后，所以直到事务结束，这些操作都属于同一事务**，假设我们在insert操作时产生了错误，可以根据事务的定义得知，这些属于同一事务的所有操作要么都执行要么都不执行，我们可以验证一下，使用rollback命令，模拟事务失败回滚，

```bash
mysql> rollback;
Query OK, 0 rows affected (0.01 sec)
```

此时我们在查询数据库中的所有数据，发现数据恢复到了update命令执行前的状态，id为3的行的balance值等于2000没有变化。

```bash
mysql> select * from bank;
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  3 | fufu | 2000.00 |
+----+------+---------+
1 row in set (0.00 sec)
```

到此，我们阐述了数据库事务的定义并用简单的Mysql操作说明了事务的操作方式，我们可以总结出数据库事务的生命周期如下：  
![](https://user-gold-cdn.xitu.io/2018/9/6/165ad9f0d6d6b87b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

可以看出事务的边界包括：

* 事务的开始边界
* 事务的正常结束边界（COMMIT），提交事务，永久保存被事务更新后的数据库状态。
* 事务的异常结束边界（ROLLBACK）：撤销事务，使数据库退回到执行事务前的初始状态。

现在我们回过头来思考一下上述示例，示例中的所有操作都是在一个Mysql会话中进行的，也就是没有其他用户在同时连接数据库进行操作，在这种没有并发会话的使用场景中，无论事务是正常结束还是异常结束，对于该单独用户读取数据不会造成任何影响，因为他的所有操作都是串行的。但是在实际应用场景中，数据库每时每刻都服务于很多会话，假设用户A的事务A开始后更新了数据库数据，此时用户B开始读取该数据，用户B将会读取到了新的值。但是如果紧接着事务A在下一条SQL语句操作时产生了错误，将事务A回滚了，那么用户B读取到的数据就是错误的无效数据了。这只是数据库事务在并发环境下会产生的一个简单的问题，所以接下来详细阐述并发事务会产生的问题。

## 并发事务会产生的问题

###  **丢失更新**

#### **第一类丢失更新**

定义：A事务撤销时，把已经提交的B事务的更新数据覆盖了。

| 时间点 | 事务A | 事务B |
| :--- | :--- | :--- |
| **T1** | **开始事务** | \*\*\*\* |
| **T2** | \*\*\*\* | **开始事务** |
| **T3** | 查询账户余额为1000元 | \*\*\*\* |
| **T4** | \*\*\*\* | 查询账户余额为1000元 |
| **T5** | \*\*\*\* | 存入100元把余额改为1100元 |
| **T6** | \*\*\*\* | **提交事务** |
| **T7** | 取出100元把余额改为900元 | \*\*\*\* |
| **T8** | **撤销事务** | \*\*\*\* |
| **T9** | 余额恢复为1000元（丢失更新） | \*\*\*\* |

以上的示例演示了第一类丢失更新问题，事务B虽然成功了，但是它所做的更新没有被永久存储，这种并发问题是由于完全没有隔离事务造成的。当两个事务更新相同的数据时，如果一个事务被提交，另一个事务却撤销，那么会连同第一个事务所做的更新也被撤销了。（这是绝对避免出现的事情） 事务A的开始时间和结束时间包含事务B的开始和结束时间，事务A回滚事务的同时，把B的已经提交的事务也回滚的，这是避免的，这就是第一类丢失更新。

#### **第二类丢失更新**

定义：A事务提交时，把已经提交的B事务的更新数据覆盖了。

| 时间点 | 事务A | 事务B |
| :--- | :--- | :--- |
| **T1** |  | **开始事务** |
| **T2** | **开始事务** |  |
| **T3** |  | 查询账户余额为1000元 |
| **T4** | 查询账户余额为1000元 |  |
| **T5** | \*\*\*\* | 取出100元把余额改为900元 |
| **T6** | \*\*\*\* | **提交事务** |
| **T7** | 存入100元把余额改为1100 |  |
| **T8** | **提交事务** |  |
| **T9** | 余额恢复为1100元（丢失更新） |  |

第二类丢失更新和第一类的区别实际上是对数据的影响是由A事务的撤销还是提交造成的，它和不可重复读\(下面介绍\)本质上是同一类并发问题，通常把它看做是不可重复读的一个特例。两个或多个事务查询同一数据。然后都基于自己的查询结果更新数据，这时会造成最后一个提交的更新事务，将覆盖其它已经提交的更新事务。

### **脏读**

定义：读到未提交更新的数据

| 时间点 | 事务A | 事务B |
| :--- | :--- | :--- |
| **T1** |  | **开始事务** |
| **T2** | **开始事务** |  |
| **T3** |  | 查询账户余额为1000元 |
| **T4** |  | 取出500元把余额改为500元 |
| **T5** | 查询账户余额为500元（脏读） |  |
| **T6** | \*\*\*\* | **撤销事务**，余额恢复为1000元 |
| **T7** | 存入100元把余额改为600元 |  |
| **T8** | **提交事务** |  |

A事务查询到了B事务未提交的更新数据，A事务依据这个查询结果继续执行相关操作。但是接着B事务撤销了所做的更新，这会导致A事务操作的是脏数据，以上的示例中T5时刻产生了脏读，最终导致A事务提交时账户余额的不正确，可能有人会有疑问，B事务还没有提交或撤销，T5时刻A事务为什么能读到已经改变的数据，这里要说的是，数据表中的数据是实时改变的，事务只是控制数据的最终状态，也就是说如果没有正确的隔离级别，在更新操作语句结束后，即使事务未完成，其他事务就已经可以读取到改变的数据值了。  
 **现在为止:所有的数据库都避免脏读操**，可以用两个Mysql会话试验一下以上的操作，在默认的隔离级别下（REPEATABLE-READ），A事务在T5时刻读取到的余额为1000元，不会是500元。

### **不可重复读**

定义：读到已经提交更新的数据，但一个事务范围内两个相同的查询却返回了不同数据。

| 时间点 | 事务A | 事务B |
| :--- | :--- | :--- |
| **T1** |  | **开始事务** |
| **T2** | **开始事务** |  |
| **T3** |  | 查询账户余额为1000元 |
| **T4** | 查询账户余额为1000元 |  |
| **T5** |  | 取出100元把余额改为900元 |
| **T6** | \*\*\*\* | **提交事务** |
| **T7** | 查询账户余额为900元（与T4读取的一不一致，不可重复读） |  |

### **幻读**

定义：读到已提交插入数据，幻读与不可重复读类似，幻读是查询到了另一个事务已提交的新插入数据，而不可重复读是查询到了另一个事务已提交的更新数据。

| 时间点 | 事务A | 事务B |
| :--- | :--- | :--- |
| **T1** |  | **开始事务** |
| **T2** | **开始事务** |  |
| **T3** | 统计用户Z总存款数为1000元 |  |
| **T4** |  | 新增Z的一个存款账号，存款100元 |
| **T5** |  | **提交事务** |
| **T6** | 再次统计用户Z总存款数为1100元（与T4读取的一不一致，幻读） |  |

A事务第一次查询时，没有问题，第二次查询时查到了B事务已提交的新插入数据，这导致两次查询结果不同。

**不可重复读和幻读的区别：** 简单来说，不可重复读是由于数据修改引起的，幻读是由数据插入或者删除引起的。

不可重复读，是指在数据库访问中，一个事务范围内两个相同的查询却返回了不同数据。这是由于查询时系统中其他事务修改的提交而引起的。比如事务T1读取某一数据，事务T2读取并修改了该数据，T1为了对读取值进行检验而再次读取该数据，便得到了不同的结果。

一种更易理解的说法是：在一个事务内，多次读同一个数据。在这个事务还没有结束时，另一个事务也访问该同一数据。那么，在第一个事务的两次读数据之间。由于第二个事务的修改，那么第一个事务读到的数据可能不一样，这样就发生了在一个事务内两次读到的数据是不一样的，因此称为不可重复读，即原始读取不可重复。

所谓幻读，是指事务A读取与搜索条件相匹配的若干行。事务B以插入或删除行等方式来修改事务A的结果集，然后再提交。

幻读是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，比如这种修改涉及到表中的“全部数据行”。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入“一行新数据”。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样.一般解决幻读的方法是增加范围锁RangeS，锁定检锁范围为只读，这样就避免了幻读。



**参考资料：**

* [数据库事务隔离级别](https://juejin.im/post/6844903670916579336)

