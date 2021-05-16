---
title: MySQL 面试
date: 2019-10-06 13:18:59
tags:
 - 数据库
 - MySQL
categories:
 - 数据库
 - MySQL
---

#### sql语句分类

![1xYcp8.png](https://s2.ax1x.com/2020/02/15/1xYcp8.png)

<!--more-->

SQL语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

1. DDL：数据定义语言（create drop）

   数据定义语言DDL用来创建数据库中的各种对象-----表、视图、
   索引、同义词、聚簇等如：
   CREATE TABLE/VIEW/INDEX/SYN/CLUSTER
   | | | | |
   表 视图 索引 同义词 簇

   DDL操作是隐性提交的！不能rollback

2. DML：数据操作语句（insert update delete）

   数据操纵语言DML主要有三种形式(没有特殊声明的话，Select也包括其内)：
   1) 插入：INSERT
   2) 更新：UPDATE
   3) 删除：DELETE

3. DQL：数据查询语句（select ）

   数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE
   子句组成的查询块：

   ```
   SELECT <字段名表>
   FROM <表或视图名>
   WHERE <查询条件>
   ```

4. DCL：数据控制语句，进行授权和权限回收（grant revoke）

   数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制
   数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
   1) GRANT：授权。

   
   2) ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。
   回滚---ROLLBACK
   回滚命令使数据库状态回到上次最后提交的状态。其格式为：
   SQL>ROLLBACK;

   
   3) COMMIT [WORK]：提交。

   
     在数据库的插入、删除和修改操作时，只有当事务在提交到数据
   库时才算完成。在事务提交前，只有操作数据库的这个人才能有权看
   到所做的事情，别人只有在最后提交完成后才可以看到。
   提交数据有三种类型：显式提交、隐式提交及自动提交。下面分
   别说明这三种类型。

   
   (1) 显式提交
   用COMMIT命令直接完成的提交为显式提交。其格式为：
   SQL>COMMIT；

   
   (2) 隐式提交
   用SQL命令间接完成的提交为隐式提交。这些命令是：
   ALTER，AUDIT，COMMENT，CONNECT，CREATE，DISCONNECT，DROP，
   EXIT，GRANT，NOAUDIT，QUIT，REVOKE，RENAME。


   (3) 自动提交
   若把AUTOCOMMIT设置为ON，则在插入、修改、删除语句执行后，
   系统将自动进行提交，这就是自动提交。其格式为：
   SQL>SET AUTOCOMMIT ON；

5. TPL：数据事务语句（commit collback savapoint）

#### 数据库三范式

- 第一范式：1NF是对属性的原子性约束，要求字段具有原子性，不可再分解；(只要是关系型数据库都满足1NF)
- 第二范式：2NF是在满足第一范式的前提下，非主键字段不能出现部分依赖主键；解决：消除复合主键就可避免出现部分以来，可增加单列关键字。
- 第三范式：3NF是在满足第二范式的前提下，非主键字段不能出现传递依赖，比如某个字段a依赖于主键，而一些字段依赖字段a，这就是传递依赖。解决：将一个实体信息的数据放在一个表内实现。

#### 事务隔离级别

- 未提交读(Read Uncommitted)：允许脏读，其他事务只要修改了数据，即使未提交，本事务也能看到修改后的数据值。也就是可能读取到其他会话中未提交事务修改的数据
- 提交读(Read Committed)：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别 (不重复读)。
- 可重复读(Repeated Read)：可重复读。无论其他事务是否修改并提交了数据，在这个事务中看到的数据值始终不受其他事务影响。
- 串行读(Serializable)：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞

MySQL数据库(InnoDB引擎)默认使用可重复读（ Repeatable read)

[数据库隔离级别及实现原理](https://blog.csdn.net/chenyiminnanjing/article/details/82714341)

[MySQL的事务隔离级别是怎么实现的？](https://blog.csdn.net/A1028151949/article/details/88430895)

[MySQL事务隔离级别的实现原理](https://blog.csdn.net/weixin_34174422/article/details/85967692)

#### 脏读&不可重复读&幻读

- **脏读: 是指事务T1将某一值修改，然后事务T2读取该值，此后T1因为某种原因撤销对该值的修改，这就导致了T2所读取到的数据是无效的。**
- **不可重复读 ：是指在数据库访问时，一个事务范围内的两次相同查询却返回了不同数据。**在一个事务内多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么在第一个事务中的两次读数据之间，由于第二个事务的修改，第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
- **幻读:** 是指当事务不是独立执行时发生的一种现象，比如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么就会发生，操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

##### 不可重复读&幻读区别:

如果使用锁机制来实现这两种隔离级别，在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会发现莫名其妙多了一条之前没有的数据，这就是幻读，**不能通过行锁来避免**。需要Serializable隔离级别 ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。

**不可重复读重点在于update和delete，而幻读的重点在于insert。如何通过锁机制来解决他们产生的问题**

[MySQL锁详解](https://www.cnblogs.com/luyucheng/p/6297752.html)

#### MyISAM和InnoDB的区别

1. MySQL默认采用的是InnoDB。

2. MyISAM不支持事务，而InnoDB支持。InnoDB的AUTOCOMMIT默认是打开的，即每条SQL语句会默认被封装成一个事务，自动提交，这样会影响速度，所以最好是把多条SQL语句显示放在begin和commit之间，组成一个事务去提交。

3. InnoDB支持数据行锁定，MyISAM不支持行锁定，只支持锁定整个表。即MyISAM同一个表上的读锁和写锁是互斥的，MyISAM并发读写时如果等待队列中既有读请求又有写请求，默认写请求的优先级高，即使读请求先到，所以MyISAM不适合于有大量查询和修改并存的情况，那样查询进程会长时间阻塞。因为MyISAM是锁表，所以某项读操作比较耗时会使其他写进程饿死。

4. InnoDB支持外键，MyISAM不支持。

5. InnoDB的主键范围更大，最大是MyISAM的2倍。

6. InnoDB不支持全文索引，而MyISAM支持。全文索引是指对char、varchar和text中的每个词（停用词除外）建立倒排序索引。MyISAM的全文索引其实没啥用，因为它不支持中文分词，必须由使用者分词后加入空格再写到数据表里，而且少于4个汉字的词会和停用词一样被忽略掉。

7. MyISAM支持GIS数据，InnoDB不支持。即MyISAM支持以下空间数据对象：Point,Line,Polygon,Surface等。

8. 没有where的count(*)使用MyISAM要比InnoDB快得多。因为MyISAM内置了一个计数器，count(*)时它直接从计数器中读，而InnoDB必须扫描全表。所以在InnoDB上执行`count(*)`时一般要伴随where，且where中要包含主键以外的索引列。为什么这里特别强调“主键以外”？因为InnoDB中primary index是和raw data存放在一起的，而secondary index则是单独存放，然后有个指针指向primary key。所以只是`count(*)`的话使用secondary index扫描更快，而primary key则主要在扫描索引同时要返回raw data时的作用较大。

#### varchar与char的区别

CHAR和VARCHAR类型类似，但它们存储和检索的方式不同，它们的最大长度和是否尾部空格被保留等方面也不同。

CHAR定义的列的长度为固定的，长度取值可以为0～255之间，当保存CHAR值时，在它们的右边填充空格以达到指定的长度。当检索到CHAR值时，尾部的空格被删除掉。比如定义 char(10)，那么不论你存储的数据是否达到了10个字节，都要占去10个字节的空间，不足的自动用空格填充。

VARCHAR定义的列的长度为可变长字符串，长度取值可以为0~65535之间，VARCHAR的最大有效长度由最大行大小和使用 的字符集确定。VARCHAR值保存时只保存需要的字符数，另加一个字节来记录长度(如果列声明的长度超过255，则使用两个字节)。VARCHAR值存储时不进行填充。，而且在值存储和检索时尾部的空格仍保留。

通过下表来表明它们在存储时的差异：

| 值        | CHAR(4) | 需要存储 | VARCHAR(4) | 需要存储 |
| --------- | ------- | -------- | ---------- | -------- |
| 'ab'      | 'ab  '  | 4 bytes  | 'ab'       | 3 bytes  |
| 'abcd'    | 'abcd'  | 4 bytes  | 'abcd'     | 5 bytes  |
| 'abcdefg' | 'abcd'  | 4 bytes  | 'abcd'     | 5 bytes  |

通过一个例子表明它们关于尾部空格的差异，从其中可以看出：取char类型的数据的时候会把空格去掉，但是在取varchar类型的数据时，数据的尾部空格会保留。

```shell
mysql> CREATE TABLE vc (v VARCHAR(4), c CHAR(4));
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO vc VALUES ('ab  ', 'ab  ');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT CONCAT('(', v, ')'), CONCAT('(', c, ')') FROM vc;
+---------------------+---------------------+
| CONCAT('(', v, ')') | CONCAT('(', c, ')') |
+---------------------+---------------------+
| (ab  )              | (ab)                |
+---------------------+---------------------+
1 row in set (0.00 sec)
```

varchar存储变长数据，可以节省存储空间，但存储效率没有 CHAR高。从空间上考虑，用varchar合适；从效率上考虑，用char合适，关键是根据实际情况找到权衡点。

#### varchar(50)中50的涵义

最多存放50个字符，varchar(50)和(200)存储hello所占空间一样，但后者在排序时会消耗更多内存，因为order by col采用fixed_length计算col长度(memory引擎也一样)。在早期 MySQL 版本中， 50 代表字节数，现在代表字符数。

#### int（20）中20的涵义

是指显示字符的长度

不影响内部存储，只是影响带 zerofill 定义的 int 时，前面补多少个 0，易于报表展示

#### delete、drop、truncate区别

- truncate 和 delete只删除数据，不删除表结构 ,drop删除表结构，并且释放所占的空间。
- **删除数据的速度，**drop> truncate > delete
- delete属于DML语言，需要事务管理，commit之后才能生效。drop和truncate属于DDL语言，操作立刻生效，不可回滚。
- 触发器和事务:DELETE支持事务和trigger
- **使用场合：**
  - 当你不再需要该表时， 用 drop;
  - 当你仍要保留该表，但要删除所有记录时， 用 truncate;
  - 当你要删除部分记录时（always with a where clause), 用 delete.

**注意：** 对于**有主外键关系的表**，不能使用truncate而应该**使用不带where子句的delete语句**，由于truncate不记录在日志中，不能够激活触发器

#### 五类索引

- index  ----  普通索引,数据可以重复，没有任何限制。
- unique   ---- 唯一索引,要求索引列的值必须唯一，但允许有空值；如果是组合索引，那么列值的组合必须唯一。

- primary key ---- 主键索引,是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值，一般是在创建表的同时创建主键索引。

- 组合索引 ----  在多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。

- fulltext ---- 全文索引,是对于大表的文本域：char，varchar，text列才能创建全文索引，主要用于查找文本中的关键字，并不是直接与索引中的值进行比较。fulltext更像是一个搜索引擎，配合match against操作使用，而不是一般的where语句加like。

注:全文索引目前只有MyISAM存储引擎支持全文索引，InnoDB引擎5.6以下版本还不支持全文索引

所有存储引擎对每个表至少支持16个索引，总索引长度至少为256字节，索引有两种存储类型，包括B型树索引和哈希索引。

索引可以提高查询的速度，但是创建和维护索引需要耗费时间，同时也会影响插入的速度，如果需要插入大量的数据时，最好是先删除索引，插入数据后再建立索引。

#### 索引生效条件

假设index（a,b,c）

1. 最左前缀匹配：模糊查询时，使用%匹配时：’a%‘会使用索引，’%a‘不会使用索引
2. 条件中有or，索引不会生效
3. a and c，a生效，c不生效
4. b and c，都不生效
5. a and b > 5 and c,a和b生效，c不生效。

#### 索引最左匹配原则

```mysql
create table test(
a int ,
b int,
c int,
d int,
key index_abc(a,b,c)
)engine=InnoDB default charset=utf8;

#插入 10000 条数据
DROP PROCEDURE IF EXISTS proc_initData;
DELIMITER $
CREATE PROCEDURE proc_initData()
BEGIN
DECLARE i INT DEFAULT 1;
WHILE i<=10000 DO
    INSERT INTO test(a,b,c,d) VALUES(i,i,i,i);
    SET i = i+1;
END WHILE;
END $
CALL proc_initData();

```

建立了联合索引（a，b，c）

```
explain select * from test where a<10 ;
explain select * from test where a<10 and b <10;
explain select * from test where a<10 and b <10 and c<10;
```

能不能将 a，b出现顺序换一下，a，b，c出现顺序换一下

```
explain select * from test where b<10 and a <10;
explain select * from test where b<10 and a <10 and c<10;
```

**不是最左匹配原则吗？**

查了下资料发现：mysql查询优化器会判断纠正这条sql语句该以什么样的顺序执行效率最高，最后才生成真正的执行计划。所以，当然是我们能尽量的利用到索引时的查询顺序效率最高咯，所以mysql查询优化器会最终以这种顺序进行查询执行。

**重点来了**

```
explain select * from test where b<10 and c <10;、
explain select * from test where a<10 and c <10;
```

为什么 b<10 and c <10,没有用到索引？而 a<10 and c <10用到了？

当b+树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+数是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，b+树会优先比较name来确定下一步的所搜方向，如果name相同再依次比较age和sex，最后得到检索的数据；但当(20,F)这样的没有name的数据来的时候，b+树就不知道下一步该查哪个节点，因为建立搜索树的时候name就是第一个比较因子，必须要先根据name来搜索才能知道下一步去哪里查询。比如当(张三,F)这样的数据来检索时，b+树可以用name来指定搜索方向，但下一个字段age的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是F的数据了， 这个是非常重要的性质，即索引的最左匹配特性。

**参考**

https://blog.csdn.net/qq_24690761/article/details/52787897

https://blog.csdn.net/SkySuperWL/article/details/52583579

#### 聚簇索引与非聚簇索引

必须为主键字段创建一个索引，这个索引就是所谓的"主索引"。主索引与唯一索引的唯一区别是：前者在定义时使用的关键字是PRIMARY而不是UNIQUE。

首先明白两句话: 

- innodb的次索引指向对主键的引用  (聚簇索引)
- myisam的次索引和主索引   都指向物理行 (非聚簇索引)

聚簇索引是对磁盘上实际数据重新组织以按指定的一个或多个列的值排序的算法。特点是存储数据的顺序和索引顺序一致。一般情况下主键会默认创建聚簇索引，且一张表只允许存在一个聚簇索引（理由：数据一旦存储，顺序只能有一种）。

在《数据库原理》一书中是这么解释聚簇索引和非聚簇索引的区别的：

聚簇索引的叶子节点就是数据节点，而非聚簇索引的叶子节点仍然是索引节点，只不过有指向对应数据块的指针。

**INNODB和MYISAM的主键索引与二级索引的对比：**

![lO7O81.png](https://s2.ax1x.com/2020/01/15/lO7O81.png)

- 也就是InnoDB的主索引的节点与数据放在一起，次索引的节点存放的是主键的位置。

- myisam的主索引和次索引都指向该数据在磁盘的位置。

InnoDB的的二级索引的叶子节点存放的是KEY字段加主键值。因此，通过二级索引查询首先查到是主键值，然后InnoDB再根据查到的主键值通过主键索引找到相应的数据块。

而MyISAM的二级索引叶子节点存放的还是列值与行号的组合，叶子节点中保存的是数据的物理地址。所以可以看出MYISAM的主键索引和二级索引没有任何区别，主键索引仅仅只是一个叫做PRIMARY的唯一、非空的索引，且MYISAM引擎中可以不设主键

也可以用下面这幅图理解:

**首先是myisam的索引主次索引都指向物理行:**

![lOHer8.png](https://s2.ax1x.com/2020/01/15/lOHer8.png)

**InnoDB的主索引叶子节点是主键和数据，次索引指向主键**

![lOHlPs.png](https://s2.ax1x.com/2020/01/15/lOHlPs.png)

innodb的主索引文件上 直接存放该行数据,称为聚簇索引,次索引指向对主键的引用

myisam中, 主索引和次索引,都指向物理行(磁盘位置).

注意: innodb来说,

1. 主键索引 既存储索引值,又在叶子中存储行的数据
2. 如果没有主键, 则会Unique key做主键
3. 如果没有unique,则系统生成一个内部的rowid做主键.
4. 像innodb中,主键的索引结构中,既存储了主键值,又存储了行数据,这种结构称为”聚簇索引”

**理论**

1. 聚簇索引

   a) 一个索引项直接对应实际数据记录的存储页，可谓“直达”
   b) 主键缺省使用它
   c) 索引项的排序和数据行的存储排序完全一致，利用这一点，想修改数据的存储顺序，可以通过改变主键的方法（撤销原有主键，另找也能满足主键要求的一个字段或一组字段，重建主键）
   d) 一个表只能有一个聚簇索引（理由：数据一旦存储，顺序只能有一种）

2. 非聚簇索引

   a) 不能“直达”，可能链式地访问多级页表后，才能定位到数据页
   b) 一个表可以有多个非聚簇索引

聚簇索引优势劣势:

- 优势: 根据主键查询条目比较少时,不用回行(数据就在主键节点下)
- 劣势: 如果碰到不规则数据插入时,造成频繁的页分裂.

聚簇索引的页分裂过程

理解:  原来索引如下

![lObpQ0.png](https://s2.ax1x.com/2020/01/15/lObpQ0.png)

此时插入一个8，需要将13,16,17移动之后插入8

![lOb9yV.png](https://s2.ax1x.com/2020/01/15/lOb9yV.png)

对于myisam引擎：只需要存储数据之后移动索引节点，

对于innoDb的聚簇索引：插入数据之后需要移动13,16,17.但是因为这三个节点上面有数据，也就造成了额外的开销。相当于三个节点搬家的同时带着数据搬家。

也可以用下图理解:

![lOqd3R.png](https://s2.ax1x.com/2020/01/15/lOqd3R.png)

[聚簇索引与非聚簇索引的区别](https://www.cnblogs.com/qlqwjy/p/7770580.html)

#### sql优化

**explain出来的各种item的意义**

```
id:每个被独立执行的操作的标志，表示对象被操作的顺序。一般来说， id 值大，先被执行；如果 id 值相同，则顺序从上到下。
select_type：查询中每个 select 子句的类型。
table:名字，被操作的对象名称，通常的表名(或者别名)，但是也有其他格式。
partitions:匹配的分区信息。
type:join 类型。
possible_keys：列出可能会用到的索引。
key:实际用到的索引。
key_len:用到的索引键的平均长度，单位为字节。
ref:表示本行被操作的对象的参照对象，可能是一个常量用 const 表示，也可能是其他表的
key 指向的对象，比如说驱动表的连接列。
rows:估计每次需要扫描的行数。
filtered:rows*filtered/100 表示该步骤最后得到的行数(估计值)。
extra:重要的补充信息。
```

type类型：从最好到最差的连接类型为const、eq_reg、ref、range、index和ALL；

- all: full table scan ;MySQL将遍历全表以找到匹配的行；
- index ： index scan; index 和 all的区别在于index类型只遍历索引；
- range：索引范围扫描，对索引的扫描开始于某一点，返回匹配值的行，常见与between ，< ,>等查询；
- ref：非唯一性索引扫描，返回匹配某个单独值的所有行，常见于使用非唯一索引即唯一索引的非唯一前缀进行查找；
- eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常用于主键或者唯一索引扫描；
- const，system：当MySQL对某查询某部分进行优化，并转为一个常量时，使用这些访问类型；如果将主键置于where列表中，MySQL就能将该查询转化为一个常量；

**profile的意义以及使用场景**

Profile 用来分析 sql 性能的消耗分布情况。当用 explain 无法解决慢 SQL 的时候，需要用profile 来对 sql 进行更细致的分析，找出 sql 所花的时间大部分消耗在哪个部分，确认 sql的性能瓶颈。

**explain 中的索引问题**

Explain 结果中，一般来说，要看到尽量用 index(type 为 const、 ref 等， key 列有值)，避免使用全表扫描(type 显式为 ALL)。比如说有 where 条件且选择性不错的列，需要建立索引。

被驱动表的连接列，也需要建立索引。被驱动表的连接列也可能会跟 where 条件列一起建立联合索引。当有排序或者 group by 的需求时，也可以考虑建立索引来达到直接排序和汇总的需求。

#### MySQL中InnoDB引擎的行锁是通过加在什么上完成(或称实现)的？为什么是这样子的？

InnoDB是基于主索引来完成行锁
例: `select * from tab_with_index where id = 1 for update;`
for update 可以根据条件来完成行锁锁定,并且 id 是有索引键的列,如果 id 不是索引键那么InnoDB将完成表锁,,并发将无从谈起

#### 字段为什么要求定义为not null?

MySQL官网这样介绍:

> NULL columns require additional space in the rowto record whether their values are NULL. For MyISAM tables, each NULL columntakes one bit extra, rounded up to the nearest byte.

null值会占用更多的字节,且会在程序中造成很多与预期不符的情况.

#### 如果要存储用户的密码散列,应该使用什么字段进行存储?

[用户密码加密存储十问十答，一文说透密码安全存储](https://www.cnblogs.com/xinzhao/p/6035847.html)

#### 超大分页怎么处理?

超大的分页一般从两个方向上来解决.

- 数据库层面,这也是我们主要集中关注的(虽然收效没那么大),类似于select * from table where age > 20 limit 1000000,10这种查询其实也是有可以优化的余地的. 这条语句需要load1000000数据然后基本上全部丢弃,只取10条当然比较慢. 当时我们可以修改为select * from table where id in (select id from table where age > 20 limit 1000000,10).这样虽然也load了一百万的数据,但是由于索引覆盖,要查询的所有字段都在索引中,所以速度会很快. 同时如果ID连续的好,我们还可以select * from table where id > 1000000 limit 10,效率也是不错的,优化的可能性有许多种,但是核心思想都一样,就是减少load的数据.
- 从需求的角度减少这种请求….主要是不做类似的需求(直接跳转到几百万页之后的具体某一页.只允许逐页查看或者按照给定的路线走,这样可预测,可缓存)以及防止ID泄漏且连续被人恶意攻击.

解决超大分页,其实主要是靠缓存,可预测性的提前查到内容,缓存至redis等k-V数据库中,直接返回即可.

在阿里巴巴《Java开发手册》中,对超大分页的解决办法是类似于上面提到的第一种.

![1xqUmT.png](https://s2.ax1x.com/2020/02/15/1xqUmT.png)

#### 关心过业务系统里面的sql耗时吗?统计过慢查询吗?对慢查询都怎么优化过?

在业务系统中,除了使用主键进行的查询,其他的我都会在测试库上测试其耗时,慢查询的统计主要由运维在做,会定期将业务中的慢查询反馈给我们.

慢查询的优化首先要搞明白慢的原因是什么? 是查询条件没有命中索引?是load了不需要的数据列?还是数据量太大?

所以优化也是针对这三个方向来的,

- 首先分析语句,看看是否load了额外的数据,可能是查询了多余的行并且抛弃掉了,可能是加载了许多结果中并不需要的列,对语句进行分析以及重写.
- 分析语句的执行计划,然后获得其使用索引的情况,之后修改语句或者修改索引,使得语句可以尽可能的命中索引.
- 如果对语句的优化已经无法进行,可以考虑表中的数据量是否太大,如果是的话可以进行横向或者纵向的分表.

#### 上面提到横向分表和纵向分表,可以分别举一个适合他们的例子吗?

横向分表是按行分表.假设我们有一张用户表,主键是自增ID且同时是用户的ID.数据量较大,有1亿多条,那么此时放在一张表里的查询效果就不太理想.我们可以根据主键ID进行分表,无论是按尾号分,或者按ID的区间分都是可以的. 假设按照尾号0-99分为100个表,那么每张表中的数据就仅有100w.这时的查询效率无疑是可以满足要求的.

纵向分表是按列分表.假设我们现在有一张文章表.包含字段id-摘要-内容.而系统中的展示形式是刷新出一个列表,列表中仅包含标题和摘要,当用户点击某篇文章进入详情时才需要正文内容.此时,如果数据量大,将内容这个很大且不经常使用的列放在一起会拖慢原表的查询速度.我们可以将上面的表分为两张.id-摘要,id-内容.当用户点击详情,那主键再来取一次内容即可.而增加的存储量只是很小的主键字段.代价很小.

当然,分表其实和业务的关联度很高,在分表之前一定要做好调研以及benchmark.不要按照自己的猜想盲目操作.

#### MySQL支持的分区类型有哪些？

表分区，是指根据一定规则，将数据库中的一张表分解成多个更小的，容易管理的部分。从逻辑上看，只有一张表，但是底层却是由多个物理分区组成。

**表分区与分表的区别**

- **分表**：指的是通过一定规则，将一张表分解成多张不同的表。比如将用户订单记录根据时间成多个表。

- **分表与分区的区别在于**：分区从逻辑上来讲只有一张表，而分表则是将一张表分解成多张表。

**表分区有什么好处？**

1. **存储更多数据**。分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备。和单个磁盘或者文件系统相比，可以存储更多数据

2. **优化查询**。在where语句中包含分区条件时，可以只扫描一个或多个分区表来提高查询效率；涉及sum和count语句时，也可以在多个分区上并行处理，最后汇总结果。

3. **分区表更容易维护**。例如：想批量删除大量数据可以清除整个分区。

4. **避免某些特殊的瓶颈**，例如InnoDB的单个索引的互斥访问，ext3问价你系统的inode锁竞争等。

**分区表的限制因素**

1. 一个表最多只能有1024个分区
2. MySQL5.1中，分区表达式必须是整数，或者返回整数的表达式。在MySQL5.5中提供了非整数表达式分区的支持。
3. 如果分区字段中有主键或者唯一索引的列，那么多有主键列和唯一索引列都必须包含进来。即：分区字段要么不包含主键或者索引列，要么包含全部主键和索引列。
4. 分区表中无法使用外键约束
5. MySQL的分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区，也不能只对索引分区而不对表分区，也不能只对表的一部分数据分区。

**如何判断当前MySQL是否支持分区？**

命令：show variables like '%partition%' 运行结果:

have_partintioning 的值为YES，表示支持分区。

**MySQL支持的分区类型有哪些？**

1. **RANGE分区**： 这种模式允许将数据划分不同范围。例如可以将一个表通过年份划分成若干个分区
2. **LIST分区**： 这种模式允许系统通过预定义的列表的值来对数据进行分割。按照List中的值分区，与RANGE的区别是，range分区的区间范围值是连续的。
3. **HASH分区** ：这中模式允许通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区。例如可以建立一个对表主键进行分区的表。
4. **KEY分区** ：上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。

#### 优化

[MySQL性能优化](https://blog.csdn.net/vbirdbest/article/details/81061053)

#### 什么时候会用到临时表？

什么是临时表：MySQL用于存储一些中间结果集的表，临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。为什么会产生临时表：一般是由于复杂的SQL导致临时表被大量创建

临时表分为两种，一种是内存临时表，一种是磁盘临时表。内存临时表采用的是memory存储引擎，磁盘临时表采用的是myisam存储引擎（磁盘临时表也可以使用innodb存储引擎，通过internal_tmp_disk_storage_engine参数来控制使用哪种存储引擎，从mysql5.7.6之后默认为innodb存储引擎，之前版本默认为myisam存储引擎）。分别通过Created_tmp_disk_tables 和 Created_tmp_tables 两个参数来查看产生了多少磁盘临时表和所有产生的临时表（内存和磁盘）。

内存临时表空间的大小由两个参数控制：tmp_table_size 和 max_heap_table_size 。一般来说是通过两个参数中较小的数来控制内存临时表空间的最大值，而对于开始在内存中创建的临时表，后来由于数据太大转移到磁盘上的临时表，只由max_heap_table_size参数控制。针对直接在磁盘上产生的临时表，没有大小控制。

#### Mysql数据库是否发生死锁？死锁的场景

<https://blog.csdn.net/Hpsyche/article/details/102076870>

#### mysql 高并发环境的解决方案

1. MySQL 高并发环境解决方案  分库  分表  分布式  增加二级缓存
2. 现有解决方式：
  - 水平分库分表，由单点分布到多点数据库中，从而降低单点数据库压力。
  - 集群方案：解决DB宕机带来的单点DB不能访问问题。
  - 引入负载均衡策略（LoadBalancePolicy简称LB）
  - 读写分离策略：极大限度提高了应用中Read数据的速度和并发量。无法解决高写入压力

#### SELECT语句执行顺序

https://www.cnblogs.com/warehouse/p/9410599.html

### 参考

1. [MySQL经典面试题](https://www.cnblogs.com/panwenbin-logs/p/8366940.html)
2. [MySQL面试题](https://www.jianshu.com/p/61389bc1c15b)
3. **[【推荐】MySQL/数据库 知识点总结](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL)**
4. **[阿里巴巴开发手册数据库部分的一些最佳实践](https://snailclimb.gitee.io/javaguide/#/docs/database/阿里巴巴开发手册数据库部分的一些最佳实践)**
5. **[一千行MySQL学习笔记](https://snailclimb.gitee.io/javaguide/#/docs/database/一千行MySQL命令)**
6. [MySQL高性能优化规范建议](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL高性能优化规范建议)
7. [数据库索引总结](https://snailclimb.gitee.io/javaguide/#/docs/database/MySQL Index)
8. [事务隔离级别(图文详解)](https://snailclimb.gitee.io/javaguide/#/docs/database/事务隔离级别(图文详解))
9. [一条SQL语句在MySQL中如何执行的](https://snailclimb.gitee.io/javaguide/#/docs/database/一条sql语句在mysql中如何执行的)
10. [Mysql锁：灵魂七拷问](https://tech.youzan.com/seven-questions-about-the-lock-of-mysql/)
11. [随笔分类 - MySQL](https://www.cnblogs.com/luyucheng/category/920876.html)