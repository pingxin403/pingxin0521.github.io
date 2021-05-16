---
title: MySQL SQL 表创建（二）
date: 2019-05-04 13:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

### Table

在创建数据库之后，接下来就要在数据库中创建数据表。所谓创建数据表，指的是在已经创建的数据库中建立新表。
创建数据表的过程是规定数据列的属性的过程，同时也是实施数据完整性（包括实体完整性、引用完整性和域完整性）约束的过程。接下来我们介绍一下创建数据表的语法形式。

<!--more-->

#### 创建表

在 MySQL 中，可以使用 CREATE TABLE 语句创建表。其语法格式为：

```
 CREATE TABLE <表名> ([表定义选项])[表选项][分区选项];
```

其中，表定义选项的格式为：

```
 <列名1> <类型1> [,…] <列名n> <类型n>
```

CREATE TABLE 命令语法比较多，其主要是由表创建定义（create-definition）、表选项（table-options）和分区选项（partition-options）所组成的。

 这里首先描述一个简单的新建表的例子，然后重点介绍 CREATE TABLE 命令中的一些主要的语法知识点。

 CREATE TABLE 语句的主要语法及使用说明如下： 

-  CREATE TABLE：用于创建给定名称的表，必须拥有表CREATE的权限。
-  <表名>：指定要创建表的名称，在 CREATE TABLE 之后给出，必须符合标识符命名规则。表名称被指定为  db_name.tbl_name，以便在特定的数据库中创建表。无论是否有当前数据库，都可以通过这种方式创建。在当前数据库中创建表时，可以省略  db-name。如果使用加引号的识别名，则应对数据库和表名称分别加引号。例如，'mydb'.'mytbl' 是合法的，但  'mydb.mytbl' 不合法。
-  <表定义选项>：表创建定义，由列名（col_name）、列的定义（column_definition）以及可能的空值说明、完整性约束或表索引组成。
-  默认的情况是，表被创建到当前的数据库中。若表已存在、没有当前数据库或者数据库不存在，则会出现错误。

>  提示：使用 CREATE TABLE 创建表时，必须指定以下信息：
>
> -  要创建的表的名称不区分大小写，不能使用SQL语言中的关键字，如DROP、ALTER、INSERT等。
> -  数据表中每个列（字段）的名称和数据类型，如果创建多个列，要用逗号隔开。

#####  在指定的数据库中创建表

数据表属于数据库，在创建数据表之前，应使用语句“USE<数据库>”指定操作在哪个数据库中进行，如果没有选择数据库，就会抛出 No database selected 的错误。

 【实例 1】创建员工表 tb_emp1，结构如下表所示。

| 字段名称 | 数据类型    | 备注         |
| -------- | ----------- | ------------ |
| id       | INT(ll)     | 员工编号     |
| name     | VARCHAR(25) | 员工名称     |
| deptld   | INT(ll)     | 所在部门编号 |
| salary   | FLOAT       | 工资         |

 选择创建表的数据库 test_db，创建 tb_emp1 数据表，输入的 SQL 语句和运行结果如下所示。 

```
mysql> USE test_db;
Database changed
mysql> CREATE TABLE tb_emp1
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
```

 语句执行后，便创建了一个名称为 tb_emp1 的数据表，使用 SHOW TABLES；语句查看数据表是否创建成功，如下所示。 

```
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp1            |
+--------------------+
1 rows in set (0.00 sec)
```

#####  查看表结构

在 MySQL 中，使用 SQL 语句创建好数据表之后，可以查看结构的定义，以确认表的定义是否正确。在 MySQL 中，查看表结构可以使用 DESCRIBE 和 SHOW CREATE TABLE 语句。

 DESCRIBE/DESC 语句可以查看表的字段信息，包括字段名、字段数据类型、是否为主键、是否有默认值等，语法规则如下：

 DESCRIBE <表名>;

 或简写成： 

 DESC <表名>;

 【实例 2】使用 DESCRIBE 查看表 tb_emp1 的结构，输入的 SQL 语句和运行结果如下所示。 

```
mysql> DESCRIBE tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```

 其中，各个字段的含义如下： 

-  Null：表示该列是否可以存储 NULL 值。
-  Key：表示该列是否已编制索引。PRI 表示该列是表主键的一部分，UNI 表示该列是 UNIQUE 索引的一部分，MUL 表示在列中某个给定值允许出现多次。
-  Default：表示该列是否有默认值，如果有，值是多少。
-  Extra：表示可以获取的与给定列有关的附加信息，如 AUTO_INCREMENT 等。

 SHOW CREATE TABLE语句可以用来显示创建表时的CREATE TABLE语句，语法格式如下： 

```
SHOW CREATE TABLE <表名>\G；
```

 【实例 3】使用 SHOW CREATE TABLE 查看表 tb_emp1 的详细信息，输入的 SQL 语句和运行结果如下所示。 

```
mysql> SHOW CREATE TABLE tb_emp1\G
*************************** 1. row ***************************
       Table: tb_emp1
Create Table: CREATE TABLE `tb_emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.03 sec)
```

>  提示：使用 SHOW CREATE TABLE 语句不仅可以查看创建表时的详细语句，而且可以查看存储引擎和字符编码。如果不加“\G”参数，显示的结果可能非常混乱，加上“\G”参数之后，可使显示的结果更加直观，易于查看。

#### 修改表

为实现数据库中表规范化设计的目的，有时候需要对之前已经创建的表进行结构修改或者调整。

在 MySQL 中可以使用 ALTER TABLE 语句来改变原有表的结构，例如增加或删减列、创建或取消索引、更改原有列类型、重新命名列或表等。 

修改表指的是修改数据库中已经存在的数据表的结构。MySQL 使用 ALTER TABLE 语句修改表。常用的修改表的操作有修改表名、修改字段数据类型或字段名、增加和删除字段、修改字段的排列位置、更改表的存储引擎、删除表的外键约束等。

 常用的语法格式如下： 

```
 ALTER TABLE <表名> [修改选项]
```

 修改选项的语法格式如下： 

```
 { ADD COLUMN <列名> <类型>
 | CHANGE COLUMN <旧列名> <新列名> <新列类型>
 | ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
 | MODIFY COLUMN <列名> <类型>
 | DROP COLUMN <列名>
 | RENAME TO <新表名> }
```

####  添加字段

随着业务的变化，可能需要在已经存在的表中添加新的字段，一个完整的字段包括字段名、数据类型、完整性约束。添加字段的语法格式如下：

```
ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] [FIRST|AFTER 已存在的字段名]；
```

新字段名为需要添加的字段的名称；FIRST 为可选参数，其作用是将新添加的字段设置为表的第一个字段；AFTER 为可选参数，其作用是将新添加的字段添加到指定的已存在的字段名的后面。

 【实例 1】使用 ALTER TABLE 修改表 tb_emp1 的结构，在表的第一列添加一个 int 类型的字段 col1，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> ADD COLUMN col1 INT FIRST;
Query OK, 0 rows affected (0.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

>  提示：“FIRST 或 AFTER 已存在的字段名”用于指定新增字段在表中的位置，如果 SQL 语句中没有这两个参数，则默认将新添加的字段设置为数据表的最后列。

 【实例 2】使用 ALTER TABLE 修改表 tb_emp1 的结构，在一列 name 后添加一个 int 类型的字段 col2，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> ADD COLUMN col2 INT AFTER name;
Query OK, 0 rows affected (0.50 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| col2   | int(11)     | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```

 可以看到，表 tb_emp1 中增加了一个名称为 col2 的字段，其位置在指定的 name 字段后面，添加字段成功。 

#####  修改字段数据类型

修改字段的数据类型就是把字段的数据类型转换成另一种数据类型。在 MySQL 中修改字段数据类型的语法规则如下：

```
ALTER TABLE <表名> MODIFY <字段名> <数据类型>
```

其中，表名指要修改数据类型的字段所在表的名称，字段名指需要修改的字段，数据类型指修改后字段的新数据类型。

 【实例 3】使用 ALTER TABLE 修改表 tb_emp1 的结构，将 name 字段的数据类型由 VARCHAR(22) 修改成 VARCHAR(30)，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> MODIFY name VARCHAR(30);
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| col2   | int(11)     | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```

 语句执行后，发现表 tb_emp1 中 name 字段的数据类型已经修改成 VARCHAR(30)，修改成功。 

#####  删除字段

删除字段是将数据表中的某个字段从表中移除，语法格式如下：

```
ALTER TABLE <表名> DROP <字段名>；
```

其中，字段名指需要从表中删除的字段的名称。

 【实例 4】使用 ALTER TABLE 修改表 tb_emp1 的结构，删除 col2 字段，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> DROP col2;
Query OK, 0 rows affected (0.53 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```

#####  修改字段名称

MySQL 中修改表字段名的语法规则如下：

```
ALTER TABLE <表名> CHANGE <旧字段名> <新字段名> <新数据类型>；
```

其中，旧字段名指修改前的字段名；新字段名指修改后的字段名；新数据类型指修改后的数据类型，如果不需要修改字段的数据类型，可以将新数据类型设置成与原来一样，但数据类型不能为空。

【实例 5】使用 ALTER TABLE 修改表 tb_emp1 的结构，将 col1 字段名称改为 col3，同时将数据类型变为 CHAR(30)，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> CHANGE col1 col3 CHAR(30);
Query OK, 0 rows affected (0.76 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col3   | char(30)    | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.01 sec)
```

 CHANGE 也可以只修改数据类型，实现和 MODIFY 同样的效果，方法是将 SQL 语句中的“新字段名”和“旧字段名”设置为相同的名称，只改变“数据类型”。 

>  提示：由于不同类型的数据在机器中的存储方式及长度并不相同，修改数据类型可能会影响数据表中已有的数据记录，因此，当数据表中已经有数据时，不要轻易修改数据类型。

#####  修改表名

MySQL 通过 ALTER TABLE 语句来实现表名的修改，语法规则如下：

```
ALTER TABLE <旧表名> RENAME [TO] <新表名>；
```

其中，TO 为可选参数，使用与否均不影响结果。

【实例 6】使用 ALTER TABLE 将数据表 tb_emp1 改名为 tb_emp2，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp1
    -> RENAME TO tb_emp2;
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp2            |
+--------------------+
1 rows in set (0.00 sec)
```

>  提示：用户可以在修改表名称时使用 DESC 命令查看修改后两个表的结构，修改表名并不修改表的结构，因此修改名称后的表和修改名称前的表的结构是相同的。

#### 删除表

在 MySQL 数据库中，对于不再需要的数据表，我们可以将其从数据库中删除。下面我们来了解一下 MySQL 数据库中数据表的删除方法。 

当需要删除一个表的时候，可以使用 

**DROP TABLE**

 语句来完成，语法格式如下： 

```
 DROP TABLE [IF EXISTS] <表名> [ , <表名1> , <表名2>] …
```

 语法说明如下： 

-  <表名>：被删除的表名。DROP TABLE 语句可以同时删除多个表，用户必须拥有该命令的权限。
-  表被删除时，所有的表数据和表定义会被取消，所以使用本语句要小心。
-  表被删除时，用户在该表上的权限并不会自动被删除。
-  参数`IF EXISTS`用于在删除前判断删除的表是否存在，加上该参数后，在删除表的时候，如果表不存在，SQL 语句可以顺利执行，但会发出警告（warning）。

选择创建表的数据库 test_db，创建 tb_emp3 数据表，输入的 SQL 语句和运行结果如下所示。 

```
mysql> USE test_db;
Database changed
mysql> CREATE TABLE tb_emp3
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.27 sec)
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp2            |
| tb_emp3            |
+--------------------+
2 rows in set (0.00 sec)
```

 【实例】删除数据表 tb_emp3，输入的 SQL 语句和运行结果如下所示。 

```
mysql> DROP TABLE tb_emp3;
Query OK, 0 rows affected (0.22 sec)
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp2            |
+--------------------+
1 rows in set (0.00 sec)
```

 执行结果可以看到，test_db 数据库的数据表列表中已经不存在名称为 tb_emp3 的表，删除操作成功。

### 主键key

“主键（PRIMARY KEY）”的完整称呼是“主键约束”。MySQL 主键约束是一个列或者列的组合，其值能唯一地标识表中的每一行。这样的一列或多列称为表的主键，通过它可以强制表的实体完整性。 

主键约束即在表中定义一个主键来唯一确定表中每一行数据的标识符。主键可以是表中的某一列或者多列的组合，其中由多列组合的主键称为复合主键。主键应该遵守下面的规则： 

-  每个表只能定义一个主键。
-  主键值必须唯一标识表中的每一行，且不能为 NULL，即表中不可能存在两行数据有相同的主键值。这是唯一性原则。
-  一个列名只能在复合主键列表中出现一次。
-  复合主键不能包含不必要的多余列。当把复合主键的某一列删除后，如果剩下的列构成的主键仍然满足唯一性原则，那么这个复合主键是不正确的。这是最小化原则。

在关系数据库中，一张表中的每一行数据被称为一条记录。一条记录就是由多个字段组成的。例如，`students`表的两行记录：

| id   | class_id | name | gender | score |
| ---- | -------- | ---- | ------ | ----- |
| 1    | 1        | 小明 | M      | 90    |
| 2    | 1        | 小红 | F      | 95    |

每一条记录都包含若干定义好的字段。同一个表的所有记录都有相同的字段定义。

对于关系表，有个很重要的约束，就是任意两条记录不能重复。不能重复不是指两条记录不完全相同，而是指能够通过某个字段唯一区分出不同的记录，这个字段被称为*主键*。

例如，假设我们把`name`字段作为主键，那么通过名字`小明`或`小红`就能唯一确定一条记录。但是，这么设定，就没法存储同名的同学了，因为插入相同主键的两条记录是不被允许的。

对主键的要求，最关键的一点是：记录一旦插入到表中，主键最好不要再修改，因为主键是用来唯一定位记录的，修改了主键，会造成一系列的影响。

由于主键的作用十分重要，如何选取主键会对业务开发产生重要影响。如果我们以学生的身份证号作为主键，似乎能唯一定位记录。然而，身份证号也是一种业务场景，如果身份证号升位了，或者需要变更，作为主键，不得不修改的时候，就会对业务产生严重影响。

所以，选取主键的一个基本原则是：**不使用任何业务相关的字段作为主键。**

因此，身份证号、手机号、邮箱地址这些看上去可以唯一的字段，均*不可*用作主键。

作为主键最好是完全业务无关的字段，我们一般把这个字段命名为`id`。常见的可作为`id`字段的类型有：

1. **自增整数类型**：数据库会在插入数据时自动为每一条记录分配一个自增整数，这样我们就完全不用担心主键重复，也不用自己预先生成主键；
2. **全局唯一GUID类型**：使用一种全局唯一的字符串作为主键，类似`8f55d96b-8acc-4636-8cb8-76bf8abc2f57`。GUID算法通过网卡MAC地址、时间戳和随机数保证任意计算机在任意时间生成的字符串都是不同的，大部分编程语言都内置了GUID算法，可以自己预算出主键。

对于大部分应用来说，通常自增类型的主键就能满足需求。我们在`students`表中定义的主键也是`BIGINT NOT NULL AUTO_INCREMENT`类型。

**如果使用INT自增类型，那么当一张表的记录数超过2147483647（约21亿）时，会达到上限而出错。使用BIGINT自增类型则可以最多约922亿亿条记录。使用Unsigned限定符，会使可用主键加倍。**

##### 联合主键（不推荐）

关系数据库实际上还允许通过多个字段唯一标识记录，即两个或更多的字段都设置为主键，这种主键被称为联合主键。

对于联合主键，允许一列有重复，只要不是所有主键列都重复即可：

| id_num | id_type | other columns... |
| ------ | ------- | ---------------- |
| 1      | A       | ...              |
| 2      | A       | ...              |
| 2      | B       | ...              |

如果我们把上述表的`id_num`和`id_type`这两列作为联合主键，那么上面的3条记录都是允许的，因为没有两列主键组合起来是相同的。

没有必要的情况下，我们尽量不使用联合主键，因为它给关系表带来了复杂度的上升。

主键是关系表中记录的唯一标识。主键的选取非常重要：主键不要带有业务含义，而应该使用BIGINT自增或者GUID类型。主键也不应该允许`NULL`。

可以使用多个列作为联合主键，但联合主键并不常用。

####  在创建表时设置主键约束

在 CREATE TABLE 语句中，主键是通过 PRIMARY KEY 关键字来指定的。

在定义列的同时指定主键，语法规则如下：

```
<字段名> <数据类型> PRIMARY KEY [默认值]
```

实例 1】在 test_db 数据库中创建 tb_emp 3 数据表，其主键为 id，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_emp3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp3;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```

 在定义完所有列之后，指定主键的语法格式为： 

```
 [CONSTRAINT <约束名>] PRIMARY KEY [字段名]
```

 【实例 2】在 test_db 数据库中创建 tb_emp 4 数据表，其主键为 id，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_emp4
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp4;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```

####  在创建表时设置复合主键

主键由多个字段联合组成，语法规则如下： 

```
 PRIMARY KEY [字段1，字段2，…,字段n]
```

【实例 3】创建数据表 tb_emp5，假设表中没有主键 id，为了唯一确定一个员工，可以把 name、deptId 联合起来作为主键，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_emp5
    -> (
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(id,deptId)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp5;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| name   | varchar(25) | NO   | PRI | NULL    |       |
| deptId | int(11)     | NO   | PRI | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
3 rows in set (0.14 sec)
```

####  在修改表时添加主键约束

在修改数据表时添加主键约束的语法规则为：

```
ALTER TABLE <数据表名> ADD PRIMARY KEY(<列名>);
```

查看 tb_emp2 数据表的表结构，如下所示。 

```
mysql> DESC tb_emp2;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```

 【实例 4】修改数据表 tb_emp2，将字段 id 设置为主键，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp2
    -> ADD PRIMARY KEY(id);
Query OK, 0 rows affected (0.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp2;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.12 sec)
```

#### 外键

MySQL 外键约束（FOREIGN KEY）用来在两个表的数据之间建立链接，它可以是一列或者多列。一个表可以有一个或多个外键。

外键对应的是参照完整性，一个表的外键可以为空值，若不为空值，则每一个外键的值必须等于另一个表中主键的某个值。

外键是表的一个字段，不是本表的主键，但对应另一个表的主键。定义外键后，不允许删除另一个表中具有关联关系的行。

外键的主要作用是保持数据的一致性、完整性。例如，部门表 tb_dept 的主键是 id，在员工表 tb_emp5 中有一个键 deptId 与这个 id 关联。

-  主表（父表）：对于两个具有关联关系的表而言，相关联字段中主键所在的表就是主表。
-  从表（子表）：对于两个具有关联关系的表而言，相关联字段中外键所在的表就是从表。

 外键的使用条件：

1. 两个表必须是InnoDB表，MyISAM表暂时不支持外键（据说以后的版本有可能支持，但至少目前不支持）；
2. 外键列必须建立了索引，MySQL 4.1.2以后的版本在建立外键时会自动创建索引，但如果在较早的版本则需要显示建立； 
3. 外键关系的两个表的列必须是数据类型相似，也就是可以相互转换类型的列，比如int和tinyint可以，而int和char则不可以；

定义一个外键时，需要遵守下列规则： 

-  父表必须已经存在于数据库中，或者是当前正在创建的表。如果是后一种情况，则父表与子表是同一个表，这样的表称为自参照表，这种结构称为自参照完整性。
-  必须为父表定义主键。
-  主键不能包含空值，但允许在外键中出现空值。也就是说，只要外键的每个非空值出现在指定的主键中，这个外键的内容就是正确的。
-  在父表的表名后面指定列名或列名的组合。这个列或列的组合必须是父表的主键或候选键。
-  外键中列的数目必须和父表的主键中列的数目相同。
-  外键中列的数据类型必须和父表主键中对应列的数据类型相同。

外键的好处：可以使得两张表关联，保证数据的一致性和实现一些级联操作；

在创建表时添加：

```
[CONSTRAINT symbol] FOREIGN KEY [id] (index_col_name, ...)
    REFERENCES tbl_name (index_col_name, ...)
    [ON DELETE {RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT}]
    [ON UPDATE {RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT}]
该语法可以在 CREATE TABLE 和 ALTER TABLE 时使用，如果不指定CONSTRAINT symbol，MYSQL会自动生成一个名字。
ON DELETE、ON UPDATE表示事件触发限制，可设参数：
RESTRICT（限制外表中的外键改动）
CASCADE（跟随外键改动）
SET NULL（设空值）
SET DEFAULT（设默认值）
NO ACTION（无动作，默认的）

```

在修改数据表时添加外键约束的语法规则为：

```
ALTER TABLE <数据表名> ADD CONSTRAINT <索引名>
FOREIGN KEY(<列名>) REFERENCES <主表名> (<列名>);
```

对于数据库中定义的外键，如果不再需要，可以将其删除。外键一旦删除，就会解除主表和从表间的关联关系，MySQL 中删除外键的语法格式如下：

```
 ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;
```

##### 一对多

当我们用主键唯一标识记录时，我们就可以在`students`表中确定任意一个学生的记录：

| id   | name | other columns... |
| ---- | ---- | ---------------- |
| 1    | 小明 | ...              |
| 2    | 小红 | ...              |

我们还可以在`classes`表中确定任意一个班级记录：

| id   | name | other columns... |
| ---- | ---- | ---------------- |
| 1    | 一班 | ...              |
| 2    | 二班 | ...              |

但是我们如何确定`students`表的一条记录，例如，`id=1`的小明，属于哪个班级呢？由于一个班级可以有多个学生，在关系模型中，这两个表的关系可以称为“一对多”，即一个`classes`的记录可以对应多个`students`表的记录。

为了表达这种一对多的关系，我们需要在`students`表中加入一列`class_id`，让它的值与`classes`表的某条记录相对应：

| id   | class_id | name | other columns... |
| ---- | -------- | ---- | ---------------- |
| 1    | 1        | 小明 | ...              |
| 2    | 1        | 小红 | ...              |
| 5    | 2        | 小白 | ...              |

这样，我们就可以根据`class_id`这个列直接定位出一个`students`表的记录应该对应到`classes`的哪条记录。

例如：

- 小明的`class_id`是`1`，因此，对应的`classes`表的记录是`id=1`的一班；
- 小红的`class_id`是`1`，因此，对应的`classes`表的记录是`id=1`的一班；
- 小白的`class_id`是`2`，因此，对应的`classes`表的记录是`id=2`的二班。

在`students`表中，通过`class_id`的字段，可以把数据与另一张表关联起来，这种列称为`外键`。

外键并不是通过列名实现的，而是通过定义外键约束实现的：

```
ALTER TABLE students
ADD CONSTRAINT fk_class_id
FOREIGN KEY (class_id)
REFERENCES classes (id);
```

其中，外键约束的名称`fk_class_id`可以任意，`FOREIGN KEY (class_id)`指定了`class_id`作为外键，`REFERENCES classes (id)`指定了这个外键将关联到`classes`表的`id`列（即`classes`表的主键）。

通过定义外键约束，关系数据库可以保证无法插入无效的数据。即如果`classes`表不存在`id=99`的记录，`students`表就无法插入`class_id=99`的记录。

由于外键约束会降低数据库的性能，大部分互联网应用程序为了追求速度，并不设置外键约束，而是仅靠应用程序自身来保证逻辑的正确性。这种情况下，`class_id`仅仅是一个普通的列，只是它起到了外键的作用而已。

要删除一个外键约束，也是通过`ALTER TABLE`实现的：

```
ALTER TABLE students
DROP FOREIGN KEY fk_class_id;
```

注意：删除外键约束并没有删除外键这一列。删除列是通过`DROP COLUMN ...`实现的。

##### 多对多

通过一个表的外键关联到另一个表，我们可以定义出一对多关系。有些时候，还需要定义“多对多”关系。例如，一个老师可以对应多个班级，一个班级也可以对应多个老师，因此，班级表和老师表存在多对多关系。

多对多关系实际上是通过两个一对多关系实现的，即通过一个中间表，关联两个一对多关系，就形成了多对多关系：

`teachers`表：

| id   | name   |
| ---- | ------ |
| 1    | 张老师 |
| 2    | 王老师 |
| 3    | 李老师 |
| 4    | 赵老师 |

`classes`表：

| id   | name |
| ---- | ---- |
| 1    | 一班 |
| 2    | 二班 |

中间表`teacher_class`关联两个一对多关系：

| id   | teacher_id | class_id |
| ---- | ---------- | -------- |
| 1    | 1          | 1        |
| 2    | 1          | 2        |
| 3    | 2          | 1        |
| 4    | 2          | 2        |
| 5    | 3          | 1        |
| 6    | 4          | 2        |

通过中间表`teacher_class`可知`teachers`到`classes`的关系：

- `id=1`的张老师对应`id=1,2`的一班和二班；
- `id=2`的王老师对应`id=1,2`的一班和二班；
- `id=3`的李老师对应`id=1`的一班；
- `id=4`的赵老师对应`id=2`的二班。

同理可知`classes`到`teachers`的关系：

- `id=1`的一班对应`id=1,2,3`的张老师、王老师和李老师；
- `id=2`的二班对应`id=1,2,4`的张老师、王老师和赵老师；

因此，通过中间表，我们就定义了一个“多对多”关系。

##### 一对一

一对一关系是指，一个表的记录对应到另一个表的唯一一个记录。

例如，`students`表的每个学生可以有自己的联系方式，如果把联系方式存入另一个表`contacts`，我们就可以得到一个“一对一”关系：

| id   | student_id | mobile      |
| ---- | ---------- | ----------- |
| 1    | 1          | 135xxxx6300 |
| 2    | 2          | 138xxxx2209 |
| 3    | 5          | 139xxxx8086 |

有细心的童鞋会问，既然是一对一关系，那为啥不给`students`表增加一个`mobile`列，这样就能合二为一了？

如果业务允许，完全可以把两个表合为一个表。但是，有些时候，如果某个学生没有手机号，那么，`contacts`表就不存在对应的记录。实际上，一对一关系准确地说，是`contacts`表一对一对应`students`表。

还有一些应用会把一个大表拆成两个一对一的表，目的是把经常读取和不经常读取的字段分开，以获得更高的性能。例如，把一个大的用户表分拆为用户基本信息表`user_info`和用户详细信息表`user_profiles`，大部分时候，只需要查询`user_info`表，并不需要查询`user_profiles`表，这样就提高了查询速度。

关系数据库通过外键可以实现一对多、多对多和一对一的关系。外键既可以通过数据库来约束，也可以不设置约束，仅依靠应用程序的逻辑来保证。

####  删除外键约束

对于数据库中定义的外键，如果不再需要，可以将其删除。外键一旦删除，就会解除主表和从表间的关联关系，MySQL 中删除外键的语法格式如下：

```
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;
```

【实例 3】删除数据表 tb_emp2 中的外键约束 fk_tb_dept1，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp2
    -> DROP FOREIGN KEY fk_tb_dept1;
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE tb_emp2\G
*************************** 1. row ***************************
       Table: tb_emp2
Create Table: CREATE TABLE `tb_emp2` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_tb_dept1` (`deptId`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```

 可以看到，tb_emp2 中已经不存在 FOREIGN KEY，原有的名称为 fk_emp_dept 的外键约束删除成功。

### 索引

在关系数据库中，如果有上万甚至上亿条记录，在查找记录的时候，想要获得非常快的速度，就需要使用索引。

索引是关系数据库中对某一列或多个列的值进行预排序的数据结构。通过使用索引，可以让数据库系统不必扫描整个表，而是直接定位到符合条件的记录，这样就大大加快了查询速度。

例如，对于`students`表：

| id   | class_id | name | gender | score |
| ---- | -------- | ---- | ------ | ----- |
| 1    | 1        | 小明 | M      | 90    |
| 2    | 1        | 小红 | F      | 95    |
| 3    | 1        | 小军 | M      | 88    |

如果要经常根据`score`列进行查询，就可以对`score`列创建索引：

```
ALTER TABLE students
ADD INDEX idx_score (score);
```

使用`ADD INDEX idx_score (score)`就创建了一个名称为`idx_score`，使用列`score`的索引。索引名称是任意的，索引如果有多列，可以在括号里依次写上，例如：

```
ALTER TABLE students
ADD INDEX idx_name_score (name, score);
```

索引的效率取决于索引列的值是否散列，即该列的值如果越互不相同，那么索引效率越高。反过来，如果记录的列存在大量相同的值，例如`gender`列，大约一半的记录值是`M`，另一半是`F`，因此，对该列创建索引就没有意义。

可以对一张表创建多个索引。索引的优点是提高了查询效率，缺点是在插入、更新和删除记录时，需要同时修改索引，因此，索引越多，插入、更新和删除记录的速度就越慢。

对于主键，关系数据库会自动对其创建主键索引。使用主键索引的效率是最高的，因为主键会保证绝对唯一。

##### 唯一索引

在设计关系数据表的时候，看上去唯一的列，例如身份证号、邮箱地址等，因为他们具有业务含义，因此不宜作为主键。

但是，这些列根据业务要求，又具有唯一性约束：即不能出现两条记录存储了同一个身份证号。这个时候，就可以给该列添加一个唯一索引。例如，我们假设`students`表的`name`不能重复：

```
ALTER TABLE students
ADD UNIQUE INDEX uni_name (name);
```

通过`UNIQUE`关键字我们就添加了一个唯一索引。

也可以只对某一列添加一个唯一约束而不创建唯一索引：

```
ALTER TABLE students
ADD CONSTRAINT uni_name UNIQUE (name);
```

这种情况下，`name`列没有索引，但仍然具有唯一性保证。

无论是否创建索引，对于用户和应用程序来说，使用关系数据库不会有任何区别。这里的意思是说，当我们在数据库中查询时，如果有相应的索引可用，数据库系统就会自动使用索引来提高查询效率，如果没有索引，查询也能正常执行，只是速度会变慢。因此，索引可以在使用数据库的过程中逐步优化。

1. 索引的效率取决于索引列的值是否散列，即该列的值如果越互不相同，那么索引效率越高。
2. 索引的优点是提高了查询效率，缺点是在插入、更新和删除记录时，需要同时修改索引，因此，索引越多，插入、更新和删除记录的速度就越慢。
3. 通过创建唯一索引，可以保证某一列的值具有唯一性。

### UNIQUE KEY

MySQL唯一约束（Unique Key）要求该列唯一，允许为空，但只能出现一个空值。唯一约束可以确保一列或者几列不出现重复值。 

####  在创建表时设置唯一约束

在定义完列之后直接使用 UNIQUE 关键字指定唯一约束，语法规则如下：

```
<字段名> <数据类型> UNIQUE
```

【实例 1】创建数据表 tb_dept2，指定部门的名称唯一，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_dept2
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) UNIQUE,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_dept2;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(40) | YES  | UNI | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.08 sec)
```

提示：UNIQUE 和 PRIMARY KEY 的区别：一个表可以有多个字段声明为 UNIQUE，但只能有一个 PRIMARY KEY 声明；声明为 PRIMAY KEY 的列不允许有空值，但是声明为 UNIQUE 的字段允许空值的存在。

####  在修改表时添加唯一约束

在修改表时添加唯一约束的语法格式为： 

```
 ALTER TABLE <数据表名> ADD CONSTRAINT <唯一约束名> UNIQUE(<列名>);
```

【实例 2】修改数据表 tb_dept1，指定部门的名称唯一，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept1
    -> ADD CONSTRAINT unique_name UNIQUE(name);
Query OK, 0 rows affected (0.63 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept1;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   | UNI | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

####  删除唯一约束

在 MySQL 中删除唯一约束的语法格式如下：

```
ALTER TABLE <表名> DROP INDEX <唯一约束名>;
```

【实例 3】删除数据表 tb_dept1 中的唯一约束 unique_name，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept1
    -> DROP INDEX unique_name;
Query OK, 0 rows affected (0.20 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept1;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   |     | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

### CHECK

MySQL 检查约束（CHECK）可以通过 CREATE TABLE 或 ALTER TABLE 语句实现，根据用户实际的完整性要求来定义。它可以分别对列或表实施 CHECK 约束。 

####  选取设置检查约束的字段

检查约束使用 CHECK 关键字，具体的语法格式如下：

```
CHECK <表达式>
```

其中：<表达式>指的就是 SQL 表达式，用于指定需要检查的限定条件。

 若将 CHECK 约束子句置于表中某个列的定义之后，则这种约束也称为基于列的 CHECK 约束。

 在更新表数据的时候，系统会检查更新后的数据行是否满足 CHECK 约束中的限定条件。MySQL 可以使用简单的表达式来实现 CHECK 约束，也允许使用复杂的表达式作为限定条件，例如在限定条件中加入子查询。 

>  注意：若将 CHECK 约束子句置于所有列的定义以及主键约束和外键定义之后，则这种约束也称为基于表的 CHECK 约束。该约束可以同时对表中多个列设置限定条件。

####  在创建表时设置检查约束

创建表时设置检查约束的语法规则如下： 

```
 CHECK(<检查约束>)
```

【实例 1】在 test_db 数据库中创建 tb_emp7 数据表，要求 salary 字段值大于 0 且小于 10000，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_emp7
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CHECK(salary>0 AND salary<100),
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
```

####  在修改表时添加检查约束

修改表时设置检查约束的语法规则如下：

```
ALTER TABLE tb_emp7 ADD CONSTRAINT <检查约束名> CHECK(<检查约束>)
```

【实例 2】修改 tb_dept 数据表，要求 id 字段值大于 0，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_emp7
    -> ADD CONSTRAINT check_id
    -> CHECK(id>0);
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

#### 删除检查约束

修改表时删除检查约束的语法规则如下： 

```
 ALTER TABLE <数据表名> DROP CONSTRAINT <检查约束名>;
```

### DEFAULT

“默认值（Default）”的完整称呼是“默认值约束（Default Constraint）”。MySQL 默认值约束用来指定某列的默认值。

例如女性同学较多，性别就可以默认为“女”。如果插入一条新的记录时没有为这个字段赋值，那么系统会自动为这个字段赋值为“女”。 

####  在创建表时设置默认值约束

创建表时可以使用 DEFAULT 关键字设置默认值约束，具体的语法规则如下：

```
<字段名> <数据类型> DEFAULT <默认值>;
```

【实例 1】创建数据表 tb_dept3，指定部门位置默认为 Beijing，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_dept3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22),
    -> location VARCHAR(50) DEFAULT 'Beijing'
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_dept3;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | YES  |     | NULL    |       |
| location | varchar(50) | YES  |     | Beijing |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.06 sec)
```

 以上语句执行成功之后，表 tb_dept3 上的字段 location 拥有了一个默认值 Beijing，新插入的记录如果没有指定部门位置，则默认都为 Beijing。 

####  在修改表时添加默认值约束

修改表时添加默认值约束的语法规则如下： 

```
 ALTER TABLE <数据表名>
 CHANGE COLUMN <字段名> <数据类型> DEFAULT <默认值>;
```

【实例 2】修改数据表 tb_dept3，将部门位置的默认值修改为 Shanghai，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT 'Shanghai';
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept3;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | YES  |     | NULL     |       |
| location | varchar(50) | YES  |     | Shanghai |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```

####  删除默认值约束

修改表时删除默认值约束的语法规则如下：

```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> DEFAULT NULL;
```

【实例 3】修改数据表 tb_dept3，将部门位置的默认值约束删除，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept3;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | YES  |     | NULL     |       |
| location | varchar(50) | YES  |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```

### NOT NULL

MySQL 非空约束（NOT NULL）可以通过 CREATE TABLE 或 ALTER TABLE 语句实现。在表中某个列的定义后加上关键字 NOT NULL 作为限定词，来约束该列的取值不能为空。

非空约束（Not Null Constraint）指字段的值不能为空。对于使用了非空约束的字段，如果用户在添加数据时没有指定值，数据库系统就会报错。 

####  在创建表时设置非空约束

创建表时可以使用 NOT NULL 关键字设置非空约束，具体的语法规则如下：

```
<字段名> <数据类型> NOT NULL;
```

【实例 1】创建数据表 tb_dept4，指定部门名称不能为空，输入的 SQL 语句和运行结果如下所示。 

```
mysql> CREATE TABLE tb_dept4
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_dept3;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   |     | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.06 sec)
```

####  在修改表时添加非空约束

修改表时设置非空约束的语法规则如下：

```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名>
<字段名> <数据类型> NOT NULL;
```

【实例 1】修改数据表 tb_dept4，指定部门位置不能为空，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept4
    -> CHANGE COLUMN location
    -> location VARCHAR(50) NOT NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept4;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | NO   |     | NULL     |       |
| location | varchar(50) | NO   |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```

####  删除非空约束

修改表时删除非空约束的语法规则如下：

```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> NULL;
```

【实例 2】修改数据表 tb_dept4，将部门位置的非空约束删除，输入的 SQL 语句和运行结果如下所示。 

```
mysql> ALTER TABLE tb_dept4
    -> CHANGE COLUMN location
    -> location VARCHAR(50) NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_dept4;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | NO   |     | NULL     |       |
| location | varchar(50) | YES  |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```

### 查看表中的约束

在 MySQL 中可以使用 SHOW CREATE TABLE 语句来查看表中的约束。

查看数据表中的约束语法格式如下： 

```
 SHOW CREATE TABLE <数据表名>;
```

【实例】创建数据表 tb_emp8 并指定 id 为主键约束，name 为唯一约束，deptId 为非空约束和外键约束，然后查看表中的约束，输入SQL语句运行结果如下。

```
mysql> CREATE TABLE tb_emp8
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) UNIQUE,
    -> deptId INT(11) NOT NULL,
    -> salary FLOAT DEFAULT 0,
    -> CHECK(salary>0),
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> SHOW CREATE TABLE tb_emp8 \G
*************************** 1. row ***************************
       Table: tb_emp8
Create Table: CREATE TABLE `tb_emp8` (
  `id` int(11) NOT NULL,
  `name` varchar(22) DEFAULT NULL,
  `deptId` int(11) NOT NULL,
  `salary` float DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`),
  KEY `deptId` (`deptId`),
  CONSTRAINT `tb_emp8_ibfk_1` FOREIGN KEY (`deptId`) REFERENCES `tb_dept1` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.19 sec)
```

