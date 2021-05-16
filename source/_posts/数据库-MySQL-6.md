---
title: MySQL SQL 触发器和索引(七) 
date: 2019-05-14 13:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

MySQL 数据库中触发器是一个特殊的存储过程，不同的是执行存储过程要使用 CALL 语句来调用，而触发器的执行不需要使用 CALL 语句来调用，也不需要手工启动，只要一个预定义的事件发生就会被 MySQL自动调用。

<!--more-->

引发触发器执行的事件一般如下： 

-  增加一条学生记录时，会自动检查年龄是否符合范围要求。
-  每当删除一条学生信息时，自动删除其成绩表上的对应记录。
-  每当删除一条数据时，在数据库存档表中保留一个备份副本。

 触发程序的优点如下： 

-  触发程序的执行是自动的，当对触发程序相关表的数据做出相应的修改后立即执行。
-  触发程序可以通过数据库中相关的表层叠修改另外的表。
-  触发程序可以实施比 FOREIGN KEY 约束、CHECK 约束更为复杂的检查和操作。

 触发器与表关系密切，主要用于保护表中的数据。特别是当有多个表具有一定的相互联系的时候，触发器能够让不同的表保持数据的一致性。

 在 MySQL 中，只有执行 INSERT、UPDATE 和 DELETE 操作时才能激活触发器。

#####  1) INSERT 触发器

 在 INSERT 语句执行之前或之后响应的触发器。

 使用 INSERT 触发器需要注意以下几点： 

-  在 INSERT 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问被插入的行。
-  在 BEFORE INSERT 触发器中，NEW 中的值也可以被更新，即允许更改被插入的值（只要具有对应的操作权限）。
-  对于 AUTO_INCREMENT 列，NEW 在 INSERT 执行之前包含的值是 0，在 INSERT 执行之后将包含新的自动生成值。

#####  2) UPDATE 触发器

 在 UPDATE 语句执行之前或之后响应的触发器。

 使用 UPDATE 触发器需要注意以下几点： 

-  在 UPDATE 触发器代码内，可引用一个名为 NEW（不区分大小写）的虚拟表来访问更新的值。
-  在 UPDATE 触发器代码内，可引用一个名为 OLD（不区分大小写）的虚拟表来访问 UPDATE 语句执行前的值。
-  在 BEFORE UPDATE 触发器中，NEW 中的值可能也被更新，即允许更改将要用于 UPDATE 语句中的值（只要具有对应的操作权限）。
-  OLD 中的值全部是只读的，不能被更新。

>  注意：当触发器设计对触发表自身的更新操作时，只能使用 BEFORE 类型的触发器，AFTER 类型的触发器将不被允许。

#####  3) DELETE 触发器

 在 DELETE 语句执行之前或之后响应的触发器。

 使用 DELETE 触发器需要注意以下几点： 

-  在 DELETE 触发器代码内，可以引用一个名为 OLD（不区分大小写）的虚拟表来访问被删除的行。
-  OLD 中的值全部是只读的，不能被更新。

 总体来说，触发器使用的过程中，MySQL 会按照以下方式来处理错误。

 若对于事务性表，如果触发程序失败，以及由此导致的整个语句失败，那么该语句所执行的所有更改将回滚；对于非事务性表，则不能执行此类回滚，即使语句失败，失败之前所做的任何更改依然有效。

 若 BEFORE 触发程序失败，则 MySQL 将不执行相应行上的操作。

 若在 BEFORE 或 AFTER 触发程序的执行过程中出现错误，则将导致调用触发程序的整个语句失败。

 仅当 BEFORE 触发程序和行操作均已被成功执行，MySQL 才会执行AFTER触发程序。

#### CREATE TRIGGER

触发器是与 MySQL 数据表有关的数据库对象，在满足定义条件时触发，并执行触发器中定义的语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性。 

在 MySQL 5.7 中，可以使用 CREATE TRIGGER 语句创建触发器。

 语法格式如下： 

```
 CREATE <触发器名> < BEFORE | AFTER >
 <INSERT | UPDATE | DELETE >
 ON <表名> FOR EACH Row<触发器主体>
```

 语法说明如下。 

1. 触发器名
   触发器的名称，触发器在当前数据库中必须具有唯一的名称。如果要在某个特定数据库中创建，名称前面应该加上数据库的名称。

2. INSERT | UPDATE | DELETE
   触发事件，用于指定激活触发器的语句的种类。

   注意：三种触发器的执行时间如下。

    INSERT：将新行插入表时激活触发器。例如，INSERT 的 BEFORE 触发器不仅能被 MySQL 的 INSERT 语句激活，也能被 LOAD DATA 语句激活。
    DELETE： 从表中删除某一行数据时激活触发器，例如 DELETE 和 REPLACE 语句。
    UPDATE：更改表中某一行数据时激活触发器，例如 UPDATE 语句。

3. BEFORE | AFTER
   BEFORE 和 AFTER，触发器被触发的时刻，表示触发器是在激活它的语句之前或之后触发。若希望验证新数据是否满足条件，则使用 BEFORE 选项；若希望在激活触发器的语句执行之后完成几个或更多的改变，则通常使用 AFTER 选项。
4. 表名
   与触发器相关联的表名，此表必须是永久性表，不能将触发器与临时表或视图关联起来。在该表上触发事件发生时才会激活触发器。同一个表不能拥有两个具有相同触发时刻和事件的触发器。例如，对于一张数据表，不能同时有两个 BEFORE UPDATE 触发器，但可以有一个 BEFORE UPDATE 触发器和一个 BEFORE INSERT 触发器，或一个 BEFORE UPDATE 触发器和一个 AFTER UPDATE 触发器。
5. 触发器主体
   触发器动作主体，包含触发器激活时将要执行的 MySQL 语句。如果要执行多个语句，可使用 BEGIN…END 复合语句结构。
6. FOR EACH ROW
   一般是指行级触发，对于受触发事件影响的每一行都要激活触发器的动作。例如，使用 INSERT 语句向某个表中插入多行数据时，触发器会对每一行数据的插入都执行相应的触发器动作。

> 注意：每个表都支持 INSERT、UPDATE 和 DELETE 的 BEFORE 与 AFTER，因此每个表最多支持 6 个触发器。每个表的每个事件每次只允许有一个触发器。单一触发器不能与多个事件或多个表关联。

另外，在 MySQL 中，若需要查看数据库中已有的触发器，则可以使用 SHOW TRIGGERS 语句。 

#####  创建 BEFORE 类型触发器

在 test_db 数据库中，数据表 tb_emp8 为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp8 的表结构如下所示。 

```
mysql> SELECT * FROM tb_emp8;
Empty set (0.07 sec)
mysql> DESC tb_emp8;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(22) | YES  | UNI | NULL    |       |
| deptId | int(11)     | NO   | MUL | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.05 sec)
```

 【实例 1】创建一个名为 SumOfSalary 的触发器，触发的条件是向数据表 tb_emp8 中插入数据之前，对新插入的 salary 字段值进行求和计算。输入的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE TRIGGER SumOfSalary
    -> BEFORE INSERT ON tb_emp8
    -> FOR EACH ROW
    -> SET @sum=@sum+NEW.salary;
Query OK, 0 rows affected (0.35 sec)
```

 触发器 SumOfSalary 创建完成之后，向表 tb_emp8 中插入记录时，定义的 sum 值由 0 变成了 1500，即插入值 1000 和 500 的和，如下所示。 

```
SET @sum=0;
Query OK, 0 rows affected (0.05 sec)
mysql> INSERT INTO tb_emp8
    -> VALUES(1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0
mysql> SELECT @sum;
+------+
| @sum |
+------+
| 1500 |
+------+
1 row in set (0.03 sec)
```

#####  创建 AFTER 类型触发器

在 test_db 数据库中，数据表 tb_emp6 和 tb_emp7 都为员工信息表，包含 id、name、deptId 和 salary 字段，数据表 tb_emp6 和 tb_emp7 的表结构如下所示。 

```
mysql> SELECT * FROM tb_emp6;
Empty set (0.07 sec)
mysql> SELECT * FROM tb_emp7;
Empty set (0.03 sec)
mysql> DESC tb_emp6;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  | MUL | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
mysql> DESC tb_emp7;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | 0       |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.04 sec)
```

 【实例 2】创建一个名为 double_salary 的触发器，触发的条件是向数据表 tb_emp6 中插入数据之后，再向数据表 tb_emp7  中插入相同的数据，并且 salary 为 tb_emp6 中新插入的 salary 字段值的 2 倍。输入的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE TRIGGER double_salary
    -> AFTER INSERT ON tb_emp6
    -> FOR EACH ROW
    -> INSERT INTO tb_emp7
    -> VALUES (NEW.id,NEW.name,deptId,2*NEW.salary);
Query OK, 0 rows affected (0.25 sec)
```

 触发器 double_salary 创建完成之后，向表 tb_emp6 中插入记录时，同时向表 tb_emp7 中插入相同的记录，并且 salary 字段为 tb_emp6 中 salary 字段值的 2 倍，如下所示。 

```
mysql> INSERT INTO tb_emp6
    -> VALUES (1,'A',1,1000),(2,'B',1,500);
Query OK, 2 rows affected (0.09 sec)
Records: 2  Duplicates: 0  Warnings: 0
mysql> SELECT * FROM tb_emp6;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   1000 |
|  2 | B    |      1 |    500 |
+----+------+--------+--------+
3 rows in set (0.04 sec)
mysql> SELECT * FROM tb_emp7;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   2000 |
|  2 | B    |      1 |   1000 |
+----+------+--------+--------+
2 rows in set (0.06 sec)
```

#### DROP TRIGGER

 与其他 MySQL 数据库对象一样，可以使用 DROP 语句将触发器从数据库中删除。

语法格式如下：

```
DROP TRIGGER [ IF EXISTS ] [数据库名] <触发器名>
```

语法说明如下：

1. 触发器名
   要删除的触发器名称。
2. 数据库名
   可选项。指定触发器所在的数据库的名称。若没有指定，则为当前默认的数据库。
3. 权限
   执行 DROP TRIGGER 语句需要 SUPER 权限。
4.  IF EXISTS
   可选项。避免在没有触发器的情况下删除触发器。

> 注意：删除一个表的同时，也会自动删除该表上的触发器。另外，触发器不能更新或覆盖，为了修改一个触发器，必须先删除它，再重新创建。

####  删除触发器

使用 DROP TRIGGER 语句可以删除 MySQL 中已经定义的触发器。

 【实例】删除 double_salary 触发器，输入的 SQL 语句和执行过程如下所示。 

```
mysql> DROP TRIGGER double_salary;
Query OK, 0 rows affected (0.03 sec)
```

 删除 double_salary 触发器后，再次向数据表 tb_emp6 中插入记录时，数据表 tb_emp7 的数据不再发生变化，如下所示。 

```
mysql> INSERT INTO tb_emp6
    -> VALUES (3,'C',1,200);
Query OK, 1 row affected (0.09 sec)
mysql> SELECT * FROM tb_emp6;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   1000 |
|  2 | B    |      1 |    500 |
|  3 | C    |      1 |    200 |
+----+------+--------+--------+
3 rows in set (0.00 sec)
mysql> SELECT * FROM tb_emp7;
+----+------+--------+--------+
| id | name | deptId | salary |
+----+------+--------+--------+
|  1 | A    |      1 |   2000 |
|  2 | B    |      1 |   1000 |
+----+------+--------+--------+
2 rows in set (0.00 sec)
```

### 索引

索引是 MySQL 中一种十分重要的数据库对象。它是数据库性能调优技术的基础，常用于实现数据的快速检索。

索引就是根据表中的一列或若干列按照一定顺序建立的列值与记录行之间的对应关系表，实质上是一张描述索引列的列值与原表中记录行之间一一对应关系的有序表。

在 MySQL 中，通常有以下两种方式访问数据库表的行数据：

1. 顺序访问

   顺序访问是在表中实行全表扫描，从头到尾逐行遍历，直到在无序的行数据中找到符合条件的目标数据。这种方式实现比较简单，但是当表中有大量数据的时候，效率非常低下。例如，在几千万条数据中查找少量的数据时，使用顺序访问方式将会遍历所有的数据，花费大量的时间，显然会影响数据库的处理性能。

2. 索引访问

   索引访问是通过遍历索引来直接访问表中记录行的方式。使用这种方式的前提是对表建立一个索引，在列上创建了索引之后，查找数据时可以直接根据该列上的索引找到对应记录行的位置，从而快捷地查找到数据。索引存储了指定列数据值的指针，根据指定的排序顺序对这些指针排序。

例如，在学生基本信息表 students 中，如果基于 student_id 建立了索引，系统就建立了一张索引列到实际记录的映射表，当用户需要查找 student_id 为 12022 的数据的时候，系统先在 student_id 索引上找到该记录，然后通过映射表直接找到数据行，并且返回该行数据。因为扫描索引的速度一般远远大于扫描实际数据行的速度，所以采用索引的方式可以大大提高数据库的工作效率。 

####  索引的分类

索引的类型和存储引擎有关，每种存储引擎所支持的索引类型不一定完全相同。根据存储方式的不同，MySQL 中常用的索引在物理上分为以下两类。 

#####  1) B-树索引

 B-树索引又称为 BTREE 索引，目前大部分的索引都是采用 B-树索引来存储的。B-树索引是一个典型的数据结构，其包含的组件主要有以下几个： 

-  叶子节点：包含的条目直接指向表里的数据行。叶子节点之间彼此相连，一个叶子节点有一个指向下一个叶子节点的指针。
-  分支节点：包含的条目指向索引里其他的分支节点或者叶子节点。
-  根节点：一个 B-树索引只有一个根节点，实际上就是位于树的最顶端的分支节点。

 基于这种树形数据结构，表中的每一行都会在索引上有一个对应值。因此，在表中进行数据查询时，可以根据索引值一步一步定位到数据所在的行。

 B-树索引可以进行全键值、键值范围和键值前缀查询，也可以对查询结果进行 ORDER BY 排序。但 B-树索引必须遵循左边前缀原则，要考虑以下几点约束： 

-  查询必须从索引的最左边的列开始。
-  查询不能跳过某一索引列，必须按照从左到右的顺序进行匹配。
-  存储引擎不能使用索引中范围条件右边的列。

#####  2) 哈希索引

 哈希（Hash）一般翻译为“散列”，也有直接音译成“哈希”的，就是把任意长度的输入（又叫作预映射，pre-image）通过散列算法变换成固定长度的输出，该输出就是散列值。

 哈希索引也称为散列索引或 HASH 索引。MySQL 目前仅有 MEMORY 存储引擎和 HEAP 存储引擎支持这类索引。其中，MEMORY 存储引擎可以支持 B- 树索引和 HASH 索引，且将 HASH 当成默认索引。

 HASH 索引不是基于树形的数据结构查找数据，而是根据索引列对应的哈希值的方法获取表的记录行。哈希索引的最大特点是访问速度快，但也存在下面的一些缺点： 

-  MySQL 需要读取表中索引列的值来参与散列计算，散列计算是一个比较耗时的操作。也就是说，相对于 B- 树索引来说，建立哈希索引会耗费更多的时间。
-  不能使用 HASH 索引排序。
-  HASH 索引只支持等值比较，如“=”“IN()”或“<=>”。
-  HASH 索引不支持键的部分匹配，因为在计算 HASH 值的时候是通过整个索引值来计算的。

 

根据索引的具体用途，MySQL 中的索引在逻辑上分为以下 5 类： 

#####  1) 普通索引

 普通索引是最基本的索引类型，唯一任务是加快对数据的访问速度，没有任何限制。创建普通索引时，通常使用的关键字是 INDEX 或 KEY。 

#####  2) 唯一性索引

 唯一性索引是不允许索引列具有相同索引值的索引。如果能确定某个数据列只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该用关键字 UNIQUE 把它定义为一个唯一性索引。

 创建唯一性索引的目的往往不是为了提高访问速度，而是为了避免数据出现重复。 

#####  3) 主键索引

 主键索引是一种唯一性索引，即不允许值重复或者值为空，并且每个表只能有一个主键。主键可以在创建表的时候指定，也可以通过修改表的方式添加，必须指定关键字 PRIMARY KEY。 

>  注意：主键是数据库考察的重点。注意每个表只能有一个主键。

#####  4) 空间索引

 空间索引主要用于地理空间数据类型 GEOMETRY。 

#####  5) 全文索引

 全文索引只能在 VARCHAR 或 TEXT 类型的列上创建，并且只能在 MyISAM 表中创建。

 索引在逻辑上分为以上 5 类，但在实际使用中，索引通常被创建成单列索引和组合索引。 

-  单列索引就是索引只包含原表的一个列。
-  组合索引也称为复合索引或多列索引，相对于单列索引来说，组合索引是将原表的多个列共同组成一个索引。

>  提示：一个表可以有多个单列索引，但这些索引不是组合索引。一个组合索引实质上为表的查询提供了多个索引，以此来加快查询速度。比如，在一个表中创建了一个组合索引(c1，c2，c3)，在实际查询中，系统用来实际加速的索引有三个：单个索引(c1)、双列索引(c1，c2)和多列索引(c1，c2，c3)。

 为了提高索引的应用性能，MySQL中的索引可以根据具体应用采用不同的索引策略。这些索引策略所对应的索引类型有聚集索引、次要索引、覆盖索引、复合索引、前缀索引、唯一索引等。 

#### 使用原则和注意事项

虽然索引可以加快查询速度，提高 MySQL 的处理性能，但是过多地使用索引也会造成以下弊端： 

-  创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。
-  除了数据表占数据空间之外，每一个索引还要占一定的物理空间。如果要建立聚簇索引，那么需要的空间就会更大。
-  当对表中的数据进行增加、删除和修改的时候，索引也要动态地维护，这样就降低了数据的维护速度。

>  注意：索引可以在一些情况下加速查询，但是在某些情况下，会降低效率。

 索引只是提高效率的一个因素，因此在建立索引的时候应该遵循以下原则： 

-  在经常需要搜索的列上建立索引，可以加快搜索的速度。
-  在作为主键的列上创建索引，强制该列的唯一性，并组织表中数据的排列结构。
-  在经常使用表连接的列上创建索引，这些列主要是一些外键，可以加快表连接的速度。
-  在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，所以其指定的范围是连续的。
-  在经常需要排序的列上创建索引，因为索引已经排序，所以查询时可以利用索引的排序，加快排序查询。
-  在经常使用 WHERE 子句的列上创建索引，加快条件的判断速度。

 与此对应，在某些应用场合下建立索引不能提高 MySQL 的工作效率，甚至在一定程度上还带来负面效应，降低了数据库的工作效率，一般来说不适合创建索引的环境如下： 

-  对于那些在查询中很少使用或参考的列不应该创建索引。因为这些列很少使用到，所以有索引或者无索引并不能提高查询速度。相反，由于增加了索引，反而降低了系统的维护速度，并增大了空间要求。
-  对于那些只有很少数据值的列也不应该创建索引。因为这些列的取值很少，例如人事表的性别列。查询结果集的数据行占了表中数据行的很大比例，增加索引并不能明显加快检索速度。
-  对于那些定义为 TEXT、IMAGE 和 BIT 数据类型的列不应该创建索引。因为这些列的数据量要么相当大，要么取值很少。
-  当修改性能远远大于检索性能时，不应该创建索引。因为修改性能和检索性能是互相矛盾的。当创建索引时，会提高检索性能，降低修改性能。当减少索引时，会提高修改性能，降低检索性能。因此，当修改性能远远大于检索性能时，不应该创建索引。

#### CREATE INDEX

MySQL 提供了三种创建索引的方法： 

#####  1) 使用 CREATE INDEX 语句

 可以使用专门用于创建索引的 CREATE INDEX 语句在一个已有的表上创建索引，但该语句不能创建主键。

 语法格式： 

```
 CREATE <索引名> ON <表名> (<列名> [<长度>] [ ASC | DESC])
```

 语法说明如下： 

-  `<索引名>`：指定索引名。一个表可以创建多个索引，但每个索引在该表中的名称是唯一的。
-  `<表名>`：指定要创建索引的表名。
-  `<列名>`：指定要创建索引的列名。通常可以考虑将查询语句中在 JOIN 子句和 WHERE 子句里经常出现的列作为索引列。
-  `<长度>`：可选项。指定使用列前的 length  个字符来创建索引。使用列的一部分创建索引有利于减小索引文件的大小，节省索引列所占的空间。在某些情况下，只能对列的前缀进行索引。索引列的长度有一个最大上限  255 个字节（MyISAM 和 InnoDB 表的最大上限为 1000  个字节），如果索引列的长度超过了这个上限，就只能用列的前缀进行索引。另外，BLOB 或 TEXT 类型的列也必须使用前缀索引。
-  `ASC|DESC`：可选项。`ASC`指定索引按照升序来排列，`DESC`指定索引按照降序来排列，默认为`ASC`。

#####  2) 使用 CREATE TABLE 语句

 索引也可以在创建表（CREATE TABLE）的同时创建。在 CREATE TABLE 语句中添加以下语句。语法格式： 

```
CONSTRAINT PRIMARY KEY [索引类型] (<列名>,…)
```

 在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的主键。

 语法格式： 



```
KEY | INDEX [<索引名>] [<索引类型>] (<列名>,…)
```

 在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的索引。

 语法格式：  

```
UNIQUE [ INDEX | KEY] [<索引名>] [<索引类型>] (<列名>,…)
```

 在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的唯一性索引。

 语法格式： 

```
 FOREIGN KEY <索引名> <列名>
```

 在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的外键。

 在使用 CREATE TABLE 语句定义列选项的时候，可以通过直接在某个列定义后面添加 PRIMARY KEY  的方式创建主键。而当主键是由多个列组成的多列索引时，则不能使用这种方法，只能用在语句的最后加上一个 PRIMARY  KRY(<列名>，…) 子句的方式来实现。 

#####  3) 使用 ALTER TABLE 语句

 CREATE INDEX 语句可以在一个已有的表上创建索引，ALTER TABLE 语句也可以在一个已有的表上创建索引。在使用 ALTER  TABLE 语句修改表的同时，可以向已有的表添加索引。具体的做法是在 ALTER TABLE 语句中添加以下语法成分的某一项或几项。

 语法格式：  

```
ADD INDEX [<索引名>] [<索引类型>] (<列名>,…)
```

 在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加索引。

 语法格式： 

```
 ADD PRIMARY KEY [<索引类型>] (<列名>,…)
```

 在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加主键。

 语法格式： 

```
 ADD UNIQUE [ INDEX | KEY] [<索引名>] [<索引类型>] (<列名>,…)
```

 在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加唯一性索引。

 语法格式： 

```
 ADD FOREIGN KEY [<索引名>] (<列名>,…)
```

 在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加外键。 

####  创建一般索引

 【实例 1】创建一个表 tb_stu_info，在该表的 height 字段创建一般索引。输入的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE TABLE tb_stu_info
    -> (
    -> id INT NOT NULL,
    -> name CHAR(45) DEFAULT NULL,
    -> dept_id INT DEFAULT NULL,
    -> age INT DEFAULT NULL,
    -> height INT DEFAULT NULL,
    -> INDEX(height)
    -> );
Query OK，0 rows affected (0.40 sec)
mysql> SHOW CREATE TABLE tb_stu_info\G
*************************** 1. row ***************************
       Table: tb_stu_info
Create Table: CREATE TABLE `tb_stu_info` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL,
  KEY `height` (`height`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.01 sec)
```

####  创建唯一索引

 【实例 2】创建一个表 tb_stu_info2，在该表的 id 字段上使用 UNIQUE 关键字创建唯一索引。输入的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE TABLE tb_stu_info2
    -> (
    -> id INT NOT NULL,
    -> name CHAR(45) DEFAULT NULL,
    -> dept_id INT DEFAULT NULL,
    -> age INT DEFAULT NULL,
    -> height INT DEFAULT NULL,
    -> UNIQUE INDEX(height)
    -> );
Query OK，0 rows affected (0.40 sec)
mysql> SHOW CREATE TABLE tb_stu_info2\G
*************************** 1. row ***************************
       Table: tb_stu_info2
Create Table: CREATE TABLE `tb_stu_info2` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL,
  UNIQUE KEY `height` (`height`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

####  查看索引

 在 MySQL 中，如果要查看已创建的索引的情况，可以使用 SHOW INDEX 语句查看表中创建的索引。

 语法格式： 

 SHOW INDEX FROM <表名> [ FROM <数据库名>]

 语法说明如下： 

-  `<表名>`：要显示索引的表。
-  `<数据库名>`：要显示的表所在的数据库。

 显示数据库 mytest 的表 course 的索引情况。 

 mysql> SHOW INDEX FROM course FROM mytest;

 该语句会返回一张结果表，该表有如下几个字段，每个字段所显示的内容说明如下。 

-  Table：表的名称。
-  Non_unique：用于显示该索引是否是唯一索引。若不是唯一索引，则该列的值显示为 1；若是唯一索引，则该列的值显示为 0。
-  Key_name：索引的名称。
-  Seq_in_index：索引中的列序列号，从 1 开始计数。
-  Column_name：列名称。
-  Collation：显示列以何种顺序存储在索引中。在 MySQL 中，升序显示值“A”（升序），若显示为 NULL，则表示无分类。
-  Cardinality：显示索引中唯一值数目的估计值。基数根据被存储为整数的统计数据计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL 使用该索引的机会就越大。
-  Sub_part：若列只是被部分编入索引，则为被编入索引的字符的数目。若整列被编入索引，则为 NULL。
-  Packed：指示关键字如何被压缩。若没有被压缩，则为 NULL。
-  Null：用于显示索引列中是否包含 NULL。若列含有 NULL，则显示为 YES。若没有，则该列显示为 NO。
-  Index_type：显示索引使用的类型和方法（BTREE、FULLTEXT、HASH、RTREE）。
-  Comment：显示评注。

 【实例 3】使用 SHOW INDEX 语句查看表 tb_stu_info2 的索引信息，输入的 SQL 语句和执行结果如下所示。 

```
mysql> SHOW INDEX FROM tb_stu_info2\G
*************************** 1. row ***************************
        Table: tb_stu_info2
   Non_unique: 0
     Key_name: height
Seq_in_index: 1
  Column_name: height
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
1 row in set (0.03 sec)
```

#### DROP INDEX

在 MySQL 中修改索引可以通过删除原索引，再根据需要创建一个同名的索引，从而实现修改索引的操作。 

当不再需要索引时，可以使用 DROP INDEX 语句或 ALTER TABLE 语句来对索引进行删除。
1) 使用 DROP INDEX 语句

语法格式：

```
DROP INDEX <索引名> ON <表名>
```


语法说明如下：

- <索引名>：要删除的索引名。
- <表名>：指定该索引所在的表名。

2) 使用 ALTER TABLE 语句

根据 ALTER TABLE 语句的语法可知，该语句也可以用于删除索引。具体使用方法是将 ALTER TABLE 语句的语法中部分指定为以下子句中的某一项。

- DROP PRIMARY KEY：表示删除表中的主键。一个表只有一个主键，主键也是一个索引。
- DROP INDEX index_name：表示删除名称为 index_name 的索引。
- DROP FOREIGN KEY fk_symbol：表示删除外键。

注意：如果删除的列是索引的组成部分，那么在删除该列时，也会将该列从索引中删除；如果组成索引的所有列都被删除，那么整个索引将被删除。

【实例 1】删除表 tb_stu_info 中的索引，输入的 SQL 语句和执行结果如下所示。 

```
mysql> DROP INDEX height
    -> ON tb_stu_info;
Query OK, 0 rows affected (0.27 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE tb_stu_info\G
*************************** 1. row ***************************
       Table: tb_stu_info
Create Table: CREATE TABLE `tb_stu_info` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

 【实例 2】删除表 tb_stu_info2 中名称为 id 的索引，输入的 SQL 语句和执行结果如下所示。 

```
mysql> ALTER TABLE tb_stu_info2
    -> DROP INDEX height;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE tb_stu_info2\G
*************************** 1. row ***************************
       Table: tb_stu_info2
Create Table: CREATE TABLE `tb_stu_info2` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

