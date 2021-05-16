---
title: MySQL SQL 事务(九)
date: 2019-05-18 13:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

### TRANSACTION

MySQL 数据库中事务是用户一系列的数据库操作序列，这些操作要么全做要么全不做，是一个不可分割的工作单位。 

事务具有 4 个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持续性（Durability）。这 4 个特性简称为 ACID 特性。

1. 原子性

   事务必须是原子工作单元，事务中的操作要么全部执行，要么全都不执行，不能只完成部分操作。原子性在数据库系统中，由恢复机制来实现。

2. 一致性

   事务开始之前，数据库处于一致性的状态；事务结束后，数据库必须仍处于一致性状态。数据库一致性的定义是由用户负责的。例如，在银行转账中，用户可以定义转账前后两个账户金额之和保持不变。

3. 隔离性

   系统必须保证事务不受其他并发执行事务的影响，即当多个事务同时运行时，各事务之间相互隔离，不可互相干扰。事务查看数据时所处的状态，要么是另一个并发事务修改它之前的状态，要么是另一个并发事务修改它之后的状态，事务不会查看中间状态的数据。隔离性通过系统的并发控制机制实现。

4. 持久性

   一个已完成的事务对数据所做的任何变动在系统中是永久有效的，即使该事务产生的修改不正确，错误也将一直保持。持久性通过恢复机制实现，发生故障时，可以通过日志等手段恢复数据库信息。

事务的 ACID 原则保证了一个事务或者成功提交，或者失败回滚，二者必居其一。因此，它对事务的修改具有可恢复性。即当事务失败时，它对数据的修改都会恢复到该事务执行前的状态。

#### 事务提交的方式

在MariaDB/MySQL中有3种事务提交的方式。

1. 显式开启和提交。

   使用begin或者start transaction来显式开启一个事务，显式开启的事务必须使用commit或者rollback显式提交或回滚。几种特殊的情况除外：行版本隔离级别下的更新冲突和死锁会自动回滚。

   在存储过程中开启事务时必须使用start transaction，因为begin会被存储过程解析为begin...end结构块。

   另外，MariaDB/MySQL中的DDL语句会自动提交前面所有的事务（包括显示开启的事务），而在SQL Server中DDL语句还是需要显式提交的，也就是说在SQL Server中DDL语句也是可以回滚的。

2. 自动提交。(MySQL默认的提交方式)

   不需要显式begin或者start transaction来显式开启事务，也不需要显式提交或回滚事务，每次执行DML和DDL语句都会在执行语句前自动开启一个事务，执行语句结束后自动提交或回滚事务。

3. 隐式提交事务

   隐式提交事务是指执行某些语句会自动提交事务，包括已经显式开启的事务。

   会隐式提交事务的语句主要有：

   - DDL语句(其中有truncate table)。
   - 隐式修改mysql数据库架构的操作:create user,drop user,grant,rename user,revoke,set password。
   - 管理语句：analyze table、cache index、check table、load index into cache、optimize table、repair table。

   通过设置 auto_commit 变量值为1或0来设置是否自动提交，为1表示自动提交，0表示关闭自动提交，即必须显式提交。但是不管设置为0还是1，显式开启的事务必须显式提交，而且隐式提交的事务不受任何人为控制。

#### 事务分类

1. 扁平事务

   即最常见的事务。由begin开始，commit或rollback结束，中间的所有操作要么都回滚要么都提交。扁平事务在生产环境中占绝大多数使用情况。因此每一种数据库产品都支持扁平事务。

   扁平事务的缺点在于无法回滚或提交一部分，只能全部回滚或全部提交，所以就有了"带有保存点"的扁平事务。

2. 带保存点的扁平事务

   通过在事务内部的某个位置使用savepoint，将来可以在事务中回滚到此位置。

   MariaDB/MySQL中设置保存点的命令为:

   ```
   savepoint [savepoint_name]
   ```

   回滚到指定保存点的命令为：

   ```
   rollback to savepoint_name
   ```

   删除一个保存点的命令为：

   ```
   release savepoint savepoint_name
   ```

   实际上，**扁平事务也是有保存点的，只不过它只有一个隐式的保存点，且自动建立在事务开始的位置，**因此扁平事务只能回滚到事务开始处。

3. 链事务

   链式事务是保存点扁平事务的变种。它在一个事务提交的时候自动隐式的将上下文传给下一个事务，也就是说一个事务的提交和下一个事务的开始是原子性的，下一个事务可以看到上一个事务的处理结果。通俗地说，就是事务的提交和事务的开始是链接式下去的。

   这样的事务类型，在提交事务的时候，会释放要提交事务内所有的锁和要提交事务内所有的保存点。因此链式事务只能回滚到当前所在事务的保存点，而不能回滚到已提交的事务中的保存点。

4. 嵌套事务

   嵌套事务由一个顶层事务控制所有的子事务。子事务的提交完成后不会真的提交，而是等到顶层事务提交才真正的提交。

   关于嵌套事务的机制，主要有以下3个结论：

   - 回滚内部事务的同时会回滚到外部事务的起始点。
   - 事务提交时从内向外依次提交。
   - 回滚外部事务的同时会回滚所有事务，包括已提交的内部事务。因为只提交内部事务时没有真的提交。

   不管怎么样，最好少用嵌套事务。且MariaDB/MySQL不原生态支持嵌套事务(SQL Server支持)。

5. 分布式事务

   将多个服务器上的事务(节点)组合形成一个遵循事务特性(acid)的分布式事务。

   例如在工行atm机转账到建行用户。工行atm机所在数据库是一个事务节点A，建行数据库是一个事务节点B，仅靠工行atm机是无法完成转账工作的，因为它控制不了建行的事务。所以它们组成一个分布式事务：

   - atm机发出转账口令。
   - atm机从工行用户减少N元。
   - 在建行用户增加N元。
   - 在atm机上返回转账成功或失败。

   上面涉及了两个事务节点，这些事务节点之间的事务必须同时具有acid属性，要么所有的事务都成功，要么所有的事务都失败，不能只成功atm机的事务，而建行的事务失败。

   MariaDB/MySQL的分布式事务使用两段式提交协议(2-phase commit,2PC)。最重要的是，MySQL 5.7.7之前，MySQL对分布式事务的支持一直都不完善(第一阶段提交后不会写binlog，导致宕机丢失日志)，这个问题持续时间长达数十年，直到MySQL 5.7.7，才完美支持分布式事务。

#### 事务控制语句

- `begin 和 start transaction`表示显式开启一个事务。它们之间并没有什么区别，但是在存储过程中，begin会被识别成begin...end的语句块，所以存储过程只能使用start transaction来显式开启一个事务。
- `commit 和 commit work`用于提交一个事务。
- `rollback 和 rollback work`用于回滚一个事务。
- `savepoint identifier`表示在事务中创建一个保存点。一个事务中允许存在多个保存点。
- `release savepoint identifier`表示删除一个保存点。当要删除的保存点不存在的时候会抛出异常。
- `rollback to savepoint`表示回滚到指定的保存点，回滚到保存点后，该保存点之后的所有操纵都被回滚。注意，rollback to不会结束事务，只是回到某一个保存点的状态。
- `set transaction`用来设置事务的隔离级别。可设置的隔离级别有read uncommitted/read committed/repeatable read/serializable。

commit与commit work以及rollback与rollback work作用是一样的。但是他们的作用却和变量completion_type的值有关。

![QFlXvV.png](https://s2.ax1x.com/2019/11/28/QFlXvV.png)

**显式事务的次数统计**

通过全局状态变量`com_commit`和`com_rollback`可以查看当前已经显式提交和显式回滚事务的次数。还可以看到回滚到保存点的次数。

```
mysql> show global status like "%com_commit%";
+-----------------+---------+
| Variable_name   |   Value |
|-----------------+---------|
| Com_commit      |  110005 |
+-----------------+---------+
mysql> show global status like "%com_rollback%";
+---------------------------+---------+
| Variable_name             |   Value |
|---------------------------+---------|
| Com_rollback              |      13 |
| Com_rollback_to_savepoint |       0 |
+---------------------------+---------+
```

#### 一致性非锁定读(快照查询)

在innodb存储引擎中，存在一种数据查询方式：快照查询。因为查询的是快照数据，所以查询时不申请共享锁。

当进行一致性非锁定读查询的时候，查询操作不会去等待记录上的独占锁释放，而是直接去读取快照数据。快照数据是通过undo段来实现的，因此它基本不会产生开销。显然，通过这种方式，可以极大的提高读并发性。

![QF1YqS.png](https://s2.ax1x.com/2019/11/28/QF1YqS.png)

快照数据其实是**行版本数据**，一个行记录可能会存在多个行版本，并发时这种读取行版本的方式称为多版本并发控制(MVCC)。在隔离级别为read committed和repeatable read时，采取的查询方式就是一致性非锁定读方式。但是，不同的隔离级别下，读取行版本的方式是不一样的。在后面介绍对应的隔离级别时会作出说明。

下面是在innodb默认的隔离级别是repeatable read下的实验，该隔离级别下，事务总是在开启的时候获取最新的行版本，并一直持有该版本直到事务结束。更多的"一致性非锁定读"见后文说明read committed和repeatable read部分。

当前示例表ttt的记录如下：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

在会话1执行：

```
mysql> begin;
mysql> update ttt set id=100 where id=1
```

在会话2中执行：

```
mysql> begin;
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

查询的结果和预期的一样，来自开启事务前最新提交的行版本数据。

回到会话1提交事务：

```
mysql> commit;
```

再回到会话2中查询：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

再次去会话1更新该记录：

```
mysql> begin;
mysql> update ttt set id=1000 where id=100;
mysql> commit;
```

再回到会话2执行查询：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

这就是repeatable read隔离级别下的一致性非锁定读的特性。

当然，MySQL也支持一致性锁定读的方式。

#### 一致性锁定读

在隔离级别为read committed和repeatable read时，采取的查询方式就是一致性非锁定读方式。但是在某些情况下，需要人为的对读操作进行加锁。MySQL中对这种方式的支持是通过在select语句后加上`lock in share mode`或者`for update`。

- `select ... from ... where ... lock in share mode;`
- `select ...from ... where ... for update;`

使用lock in share mode会对select语句要查询的记录加上一个共享锁(S)，使用for update语句会对select语句要查询的记录加上独占锁(X)。

另外，对于一致性非锁定读操作，即使要查询的记录已经被for update加上了独占锁，也一样可以读取，就和纯粹的update加的锁一样，只不过此时读取的是快照数据而已。

#### 事务

事务特性(ACID)中的隔离性(I,isolation)就是隔离级别，它通过锁来实现。也就是说，**设置不同的隔离级别，其本质只是控制不同的锁行为**。例如操作是否申请锁，什么时候申请锁，申请的锁是立刻释放还是持久持有直到事务结束才释放等。

- 读未提交：read uncommitted
- 读已提交：read committed
- 可重复读：repeatable read
- 串行化：serializable

|                  | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| Read uncommitted | √    | √          | √    |
| Read committed   | ×    | √          | √    |
| Repeatable read  | ×    | ×          | √    |
| Serializable     | ×    | ×          | ×    |

隔离级别是基于会话设置的，当然也可以基于全局进行设置，设置为全局时，不会影响当前会话的级别。设置的方法是：

```
set [global | session] transaction isolation level {type}
type:
    read uncommitted | read committed | repeatable read | serializable
```

或者直接修改变量值也可以：

```
set @@global.tx_isolation = 'read-uncommitted' | 'read-committed' | 'repeatable-read' | 'serializable'
set @@session.tx_isolation = 'read-uncommitted' | 'read-committed' | 'repeatable-read' | 'serializable'
```

查看当前会话的隔离级别方法如下：

```
mysql> select @@tx_isolation;
mysql> select @@global.tx_isolation;
mysql> select @@tx_isolation;select @@global.tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
```

注意，事务隔离级别的设置只需在需要的一端设置，不用在两边会话都设置。例如想要让会话2的查询加锁，则只需在会话2上设置serializable，在会话1设置的serializable对会话2是没有影响的，这和SQL Server中一样。但是，MariaDB/MySQL除了serializable隔离级别，其他的隔离级别都默认会读取旧的行版本，所以查询永远不会造成阻塞。而SQL Server中只有基于快照的两种隔离级别才会读取行版本，所以在4种标准的隔离级别下，如果查询加的S锁被阻塞，查询会进入锁等待。

在MariaDB/MySQL中不会出现更新丢失的问题，因为独占锁一直持有直到事务结束。当1个会话开启事务A修改某记录，另一个会话也开启事务B修改该记录，该修改被阻塞，当事务A提交后，事务B中的更新立刻执行成功，但是执行成功后查询却发现数据并没有随着事务B的想法而改变，因为这时候事务B更新的那条记录已经不是原来的记录了。但是事务A回滚的话，事务B是可以正常更新的，但这没有丢失更新。

#####  read uncommitted

该级别称为未提交读，即允许读取未提交的数据。

在该隔离级别下，读数据的时候不会申请读锁，所以也不会出现查询被阻塞的情况。

在会话1执行：

```
create table ttt(id int);
insert into ttt select 1;
insert into ttt select 2;
begin;
update ttt set id=10 where id=1;
```

如果会话1的隔离级别不是默认的，那么在执行update的过程中，可能会遇到以下错误：

```
ERROR 1665 (HY000): Cannot execute statement: impossible to write to binary log since BINLOG_FORMAT = STATEMENT and at least one table uses a storage engine limited to row-based logging. InnoDB is limited to row-logging when transaction isolation level is READ COMMITTED or READ UNCOMMITTED.
```

这是read committed和read uncommitted两个隔离级别只允许row格式的二进制日志记录格式。而当前的二进制日志格式记录方式为statement时就会报错。要解决这个问题，只要将格式设置为row或者mixed即可。

```
set @@session.binlog_format=row;
```

在会话2执行：

```
set transaction isolation level read uncommitted;
select * from ttt;
+------+
| id   |
+------+
|   10 |
|    2 |
+------+
```

发现查询的结果是update后的数据，但是这个数据是会话1未提交的数据。这是**脏读**的问题，即读取了未提交的脏数据。

如果此时会话1进行了回滚操作，那么会话2上查询的结果又变成了id=1。

在会话1上执行：

```
rollback;
```

在会话2上查询：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

这是**读不一致**问题。即同一个会话中对同一条记录的读取结果不一致。

read uncommitted一般不会在生产环境中使用，因为问题太多，会导致**脏读、丢失的更新、幻影读、读不一致**的问题。但由于不申请读锁，从理论上来说，它的并发性是最佳的。所以在某些特殊情况下还是会考虑使用该级别。

要解决脏读、读不一致问题，只需在查询记录的时候加上共享锁即可。这样在其他事务更新数据的时候就无法查询到更新前的记录。这就是read commmitted隔离级别。

##### read committed

对于熟悉SQL Server的人来说，在说明这个隔离级别之前，必须先给个提醒：MariaDB/MySQL中的提交读和SQL Server中的提交读完全不一样，**MariaDB/MySQL中该级别基本类似于SQL Server中基于快照的提交读**。

在SQL Server中，提交读的查询会申请共享锁，并且在查询结束的一刻立即释放共享锁，如果要查询的记录正好被独占锁锁住，则会进入锁等待，而没有被独占锁锁住的记录则可以正常查询。SQL Server中基于快照的提交读实现的是语句级的事务一致性，每执行一次操作事务序列号加1，并且每次查询的结果都是最新提交的行版本快照。

也就是说，MariaDB/MySQL中read committed级别总是会读取最新提交的行版本。这在MySQL的innodb中算是一个术语:"**一致性非锁定读**"，即只读取快照数据，不加共享锁。这在前文已经说明过。

MariaDB/MySQL中的read committed隔离级别下，除非是要检查外键约束或者唯一性约束需要用到**gap lock算法**，其他时候都不会用到。也就是说在此隔离级别下，一般来说只会对行进行锁定，不会锁定范围，所以会导致**幻影读**问题。

这里要演示的就是在该级别下，会不断的读取最新提交的行版本数据。

当前示例表ttt的记录如下：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

在会话1中执行：

```
begin;update ttt set id=100 where id=1;
```

在会话2中执行：

```
set @@session.tx_isolation='read-committed';
begin;
select * from ttt;
```

会话2中查询得到的结果为id=1，因为查询的是最新提交的快照数据，而最新提交的快照数据就是id=1。

```
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

现在将会话1中的事务提交。

在会话1中执行：

```
commit;
```

在会话2中查询记录：

```
select * from ttt;
+------+
| id   |
+------+
|  100 |
|    2 |
+------+
```

结果为id=100，因为这个值是最新提交的。

再次在会话1中修改该值并提交事务。

在会话1中执行：

```
begin;update ttt set id=1000 where id=100;commit;
```

在会话2中执行：

```
select * from ttt;
+------+
| id   |
+------+
| 1000 |
|    2 |
+------+
```

发现结果变成了1000，因为1000是最新提交的数据。

read committed隔离级别的行版本读取特性，在和repeatable read隔离级别比较后就很容易理解。

##### repeatable read

同样是和上面一样的废话，对于熟悉SQL Server的人来说，在说明这个隔离级别之前，必须先给个提醒：MariaDB/MySQL中的重复读和SQL Server中的重复读完全不一样，**MariaDB/MySQL中该级别基本类似于SQL Server中快照隔离级别**。

在SQL Server中，重复读的查询会申请共享锁，并且在查询结束的一刻不释放共享锁，而是持有到事务结束。所以会造成比较严重的读写并发问题。SQL Server中快照隔离级别实现的是事务级的事务一致性，每次事务开启的时候获取最新的已提交行版本，只要事务不结束，读取的记录将一直是该行版本中的数据，不管其他事务是否已经提交过对应的数据了。但是SQL Server中的快照隔离会有更新冲突：当检测到两边都想要更新同一记录时，会检测出更新冲突，这样会提前结束事务(进行的是回滚操作)而不用再显式地commit或者rollback。

也就是说，MariaDB/MySQL中repeatable read级别**总是会在事务开启的时候读取最新提交的行版本，并将该行版本一直持有到事务结束。**但是MySQL中的repeatable read级别下不会像SQL Server一样出现更新冲突的问题。

前文说过read committed隔离级别下，读取数据时总是会去获取最新已提交的行版本。这是这两个隔离级别在"一致性非锁定读"上的区别。

另外，MariaDB/MySQL中的repeatable read的加锁方式是**next-key lock算法**，它会进行范围锁定。这就避免了幻影读的问题(官方手册上说无法避免)。在标准SQL中定义的隔离级别中，需要达到serializable级别才能避免幻影读问题，也就是说MariaDB/MySQL中的repeatable read隔离级别已经达到了其他数据库产品(如SQL Server)的serializable级别，而且SQL Server中的serializable加范围锁时，在有索引的时候式锁范围比较不可控(你不知道范围锁锁住哪些具体的范围)，而在MySQL中是可以判断锁定范围的(见[innodb锁算法](http://www.cnblogs.com/f-ck-need-u/p/8995475.html#blog4.3))。

这里要演示的就是在该级别下，读取的行版本数据是不随提交而改变的。

当前示例表ttt的记录如下：

```
mysql> select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

在会话1执行：

```
begin;update ttt set id=100 where id=1
```

在会话2中执行：

```
set @@session.tx_isolation='repeatable-read';
begin;select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

查询的结果和预期的一样，来自开启事务前最新提交的行版本数据。

回到会话1提交事务：

```
commit;
```

再回到会话2中查询：

```
select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

再次去会话1更新该记录：

```
begin;update ttt set id=1000 where id=100;commit;
```

再回到会话2执行查询：

```
select * from ttt;
+------+
| id   |
+------+
|    1 |
|    2 |
+------+
```

发现结果根本就不会改变，因为会话2开启事务时获取的行版本的id=1，所以之后读取的一直都是id=1所在的行版本。

##### serializable

在SQL Server中，serializable隔离级别会将查询申请的共享锁持有到事务结束，且申请的锁是范围锁，范围锁的情况根据表有无索引而不同：无索引时锁定整个表，有索引时锁定某些范围，至于锁定哪些具体的范围我发现是不可控的(至少我无法推测和计算)。这样就避免了幻影读的问题。

这种问题在MariaDB/MySQL中的repeatable read级别就已经实现了，MariaDB/MySQL中的next-key锁算法在加范围锁时也分有无索引：无索引时加锁整个表(实际上不是表而是无穷大区间的行记录)，有索引时加锁部分可控的范围。

MariaDB/MySQL中的serializable其实类似于repeatable read，只不过所有的select语句会自动在后面加上`lock in share mode`。也就是说会对所有的读进行加锁，而不是读取行版本的快照数据，也就不再支持"一致性非锁定读"。这样就实现了串行化的事务隔离：每一个事务必须等待前一个事务(哪怕是只有查询的事务)结束后才能进行哪怕只是查询的操作。

这个隔离级别对并发性来说，显然是有点太严格了。

#### 事务流程

**开始事务**

 事务以 BEGIN TRANSACTION 开始。

 语法格式如下： 

```
BEGIN TRANSACTION <事务名称> |@<事务变量名称>
```

 语法说明如下： 

-  `@<事务变量名称>`是由用户定义的变量，必须用 char、varchar、nchar 或 nvarchar数据类型来声明该变量。
-  `BEGIN TRANSACTION` 语句的执行使全局变量 @@TRANCOUNT 的值加 1。

**提交事务**

 COMMIT 表示提交事务，即提交事务的所有操作。具体地说，就是将事务中所有对数据库的更新写回到磁盘上的物理数据库中，事务正常结束。

 提交事务，意味着将事务开始以来所执行的所有数据修改成为数据库的永久部分，因此也标志着一个事务的结束。一旦执行了该命令，将不能回滚事务。只有在所有修改都准备好提交给数据库时，才执行这一操作。

 语法格式如下： 

```
 COMMIT TRANSACTION <事务名称> |@<事务变量名称>
```

 其中：

```
COMMIT TRANSACTION
```

语句的执行使全局变量 @@TRANCOUNT 的值减 1。 

**撤销事务**

 ROLLBACK 表示撤销事务，即在事务运行的过程中发生了某种故障，事务不能继续执行，系统将事务中对数据库的所有已完成的操作全部撤销，回滚到事务开始时的状态。这里的操作指对数据库的更新操作。

 当事务执行过程中遇到错误时，使用 ROLLBACK TRANSACTION 语句使事务回滚到起点或指定的保持点处。同时，系统将清除自事务起点或到某个保存点所做的所有的数据修改，并且释放由事务控制的资源。因此，这条语句也标志着事务的结束。

 语法格式如下： 

```
ROLLBACK [TRANSACTION]
 [<事务名称>| @<事务变量名称> | <存储点名称>| @ <含有存储点名称的变量名>
```

 语法说明如下： 

-  当条件回滚只影响事务的一部分时，事务不需要全部撤销已执行的操作。可以让事务回滚到指定位置，此时，需要在事务中设定保存点（SAVEPOINT）。保存点所在位置之前的事务语句不用回滚，即保存点之前的操作被视为有效的。保存点的创建通过“SAVING  TRANSACTION<保存点名称>”语句来实现，再执行“ROLLBACK  TRANSACTION<保存点名称>”语句回滚到该保存点。
-  若事务回滚到起点，则全局变量 @@TRANCOUNT 的值减 1；若事务回滚到指定的保存点，则全局变量 @@TRANCOUNT 的值不变。


