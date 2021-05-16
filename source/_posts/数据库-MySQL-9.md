---
title: MySQL SQL 数据操纵和视图（四）
date: 2019-05-06 19:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

### INSERT

数据库与表创建成功以后，需要向数据库的表中插入数据。在 MySQL 中可以使用 INSERT 语句向数据库已有的表中插入一行或者多行元组数据。 

<!--more-->

INSERT 语句有两种语法形式，分别是 INSERT…VALUES 语句和 INSERT…SET 语句。

1. INSERT…VALUES语句
   INSERT VALUES 的语法格式为：

```
INSERT INTO <表名> [ <列名1> [ , … <列名n>] ]
VALUES (值1) [… , (值n) ];
```


语法说明如下。

    <表名>：指定被操作的表名。
    <列名>：指定需要插入数据的列名。若向表中的所有列插入数据，则全部的列名均可以省略，直接采用 INSERT<表名>VALUES(…) 即可。
    VALUES 或 VALUE 子句：该子句包含要插入的数据清单。数据清单中数据的顺序要和列的顺序相对应。
2. INSERT…SET语句

 语法格式为： 

```
 INSERT INTO <表名>
 SET <列名1> = <值1>,
         <列名2> = <值2>,
         …
```

 此语句用于直接给表中的某些列指定对应的列值，即要插入的数据的列名在 SET 子句中指定，col_name 为指定的列名，等号后面为指定的数据，而对于未指定的列，列值会指定为该列的默认值。

 由 INSERT 语句的两种形式可以看出： 

-  使用 INSERT…VALUES 语句可以向表中插入一行数据，也可以插入多行数据；
-  使用 INSERT…SET 语句可以指定插入行中每列的值，也可以指定部分列的值；
-  INSERT…SELECT 语句向表中插入其他表的数据。
-  采用 INSERT…SET 语句可以向表中插入部分列的值，这种方式更为灵活；
-  INSERT…VALUES 语句可以一次插入多条数据。

在 MySQL 中，用单条 INSERT 语句处理多个插入要比使用多条 INSERT 语句更快。
当使用单条 INSERT 语句插入多行数据的时候，只需要将每行数据用圆括号括起来即可。

####  向表中的全部字段添加值

在 test_db 数据库中创建一个课程信息表 tb_courses，包含课程编号 course_id、课程名称 course_name、课程学分 course_grade 和课程备注 course_info，输入的 SQL 语句和执行结果如下所示。 

```
mysql> CREATE TABLE tb_courses
    -> (
    -> course_id INT NOT NULL AUTO_INCREMENT,
    -> course_name CHAR(40) NOT NULL,
    -> course_grade FLOAT NOT NULL,
    -> course_info CHAR(100) NULL,
    -> PRIMARY KEY(course_id)
    -> );
Query OK, 0 rows affected (0.00 sec)
```

 向表中所有字段插入值的方法有两种：一种是指定所有字段名；另一种是完全不指定字段名。

 【实例 1】在 tb_courses 表中插入一条新记录，course_id 值为 1，course_name 值为“Network”，course_grade 值为 3，info 值为“Computer Network”。

 在执行插入操作之前，查看 tb_courses 表的SQL语句和执行结果如下所示。 

```
mysql> SELECT * FROM tb_courses;
Empty set (0.00 sec)
```

 查询结果显示当前表内容为空，没有数据，接下来执行插入数据的操作，输入的 SQL 语句和执行过程如下所示。 

```
mysql> INSERT INTO tb_courses
    -> (course_id,course_name,course_grade,course_info)
    -> VALUES(1,'Network',3,'Computer Network');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
+-----------+-------------+--------------+------------------+
1 row in set (0.00 sec)
```

 可以看到插入记录成功。在插入数据时，指定了 tb_courses 表的所有字段，因此将为每一个字段插入新的值。

 INSERT 语句后面的列名称顺序可以不是 tb_courses 表定义时的顺序，即插入数据时，不需要按照表定义的顺序插入，只要保证值的顺序与列字段的顺序相同就可以。

 【实例 2】在 tb_courses 表中插入一条新记录，course_id 值为 2，course_name 值为“Database”，course_grade 值为 3，info值为“MySQL”。输入的 SQL 语句和执行结果如下所示。 

```
mysql> INSERT INTO tb_courses
    -> (course_name,course_info,course_id,course_grade)
    -> VALUES('Database','MySQL',2,3);
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
+-----------+-------------+--------------+------------------+
2 rows in set (0.00 sec)
```

 使用 INSERT 插入数据时，允许列名称列表 column_list 为空，此时值列表中需要为表的每一个字段指定值，并且值的顺序必须和数据表中字段定义时的顺序相同。

 【实例 3】在 tb_courses 表中插入一条新记录，course_id 值为 3，course_name 值为“

Java

”，course_grade 值为 4，info 值为“Jave EE”。输入的 SQL 语句和执行结果如下所示。 

```
mysql> INSERT INTO tb_courses
    -> VLAUES(3,'Java',4,'Java EE');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
+-----------+-------------+--------------+------------------+
3 rows in set (0.00 sec)
```

 INSERT 语句中没有指定插入列表，只有一个值列表。在这种情况下，值列表为每一个字段列指定插入的值，并且这些值的顺序必须和 tb_courses 表中字段定义的顺序相同。 

>  注意：虽然使用 INSERT 插入数据时可以忽略插入数据的列名称，若值不包含列名称，则 VALUES  关键字后面的值不仅要求完整，而且顺序必须和表定义时列的顺序相同。如果表的结构被修改，对列进行增加、删除或者位置改变操作，这些操作将使得用这种方式插入数据时的顺序也同时改变。如果指定列名称，就不会受到表结构改变的影响。

####  向表中指定字段添加值

为表的指定字段插入数据，是在 INSERT 语句中只向部分字段中插入值，而其他字段的值为表定义时的默认值。

 【实例 4】在 tb_courses 表中插入一条新记录，course_name 值为“System”，course_grade 值为 3，course_info 值为“Operating System”，输入的 SQL 语句和执行结果如下所示。 

```
mysql> INSERT INTO tb_courses
    -> (course_name,course_grade,course_info)
    -> VALUES('System',3,'Operation System');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

 可以看到插入记录成功。如查询结果显示，这里的 course_id 字段自动添加了一个整数值 4。这时的 course_id  字段为表的主键，不能为空，系统自动为该字段插入自增的序列值。在插入记录时，如果某些字段没有指定插入值，MySQL 将插入该字段定义时的默认值。 

####  使用 INSERT INTO…FROM 语句复制表数据

INSERT INTO…SELECT…FROM 语句用于快速地从一个或多个表中取出数据，并将这些数据作为行数据插入另一个表中。

 SELECT 子句返回的是一个查询到的结果集，INSERT 语句将这个结果集插入指定表中，结果集中的每行数据的字段数、字段的数据类型都必须与被操作的表完全一致。

 在数据库 test_db 中创建一个与 tb_courses 表结构相同的数据表 tb_courses_new，创建表的 SQL 语句和执行过程如下所示。 

```
mysql> CREATE TABLE tb_courses_new
    -> (
    -> course_id INT NOT NULL AUTO_INCREMENT,
    -> course_name CHAR(40) NOT NULL,
    -> course_grade FLOAT NOT NULL,
    -> course_info CHAR(100) NULL,
    -> PRIMARY KEY(course_id)
    -> );
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM tb_courses_new;
Empty set (0.00 sec)
```

 【实例 5】从 tb_courses 表中查询所有的记录，并将其插入 tb_courses_new 表中。输入的 SQL 语句和执行结果如下所示。 

```
mysql> INSERT INTO tb_courses_new
    -> (course_id,course_name,course_grade,course_info)
    -> SELECT course_id,course_name,course_grade,course_info
    -> FROM tb_courses;
Query OK, 4 rows affected (0.17 sec)
Records: 4  Duplicates: 0  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

### UPDATE

使用 UPDATE 语句修改单个表，语法格式为： 

```
 UPDATE <表名> SET 字段 1=值 1 [,字段 2=值 2… ] [WHERE 子句 ]
 [ORDER BY 子句] [LIMIT 子句]
```

 语法说明如下： 

-  `<表名>`：用于指定要更新的表名称。
-  `SET` 子句：用于指定表中要修改的列名及其列值。其中，每个指定的列值可以是表达式，也可以是该列对应的默认值。如果指定的是默认值，可用关键字 DEFAULT 表示列值。
-  `WHERE` 子句：可选项。用于限定表中要修改的行。若不指定，则修改表中所有的行。
-  `ORDER BY` 子句：可选项。用于限定表中的行被修改的次序。
-  `LIMIT` 子句：可选项。用于限定被修改的行数。

>  注意：修改一行数据的多个列值时，SET 子句的每个值用逗号分开即可。

####  修改表中的数据

【实例 1】在 tb_courses_new 表中，更新所有行的 course_grade 字段值为 4，输入的 SQL 语句和执行结果如下所示。 

```
mysql> UPDATE tb_courses_new
    -> SET course_grade=4;
Query OK, 3 rows affected (0.11 sec)
Rows matched: 4  Changed: 3  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | Database    |            4 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

####  根据条件修改表中的数据

【实例 2】在 tb_courses 表中，更新 course_id 值为 2 的记录，将 course_grade 字段值改为 3.5，将 course_name 字段值改为“DB”，输入的 SQL 语句和执行结果如下所示。 

```
mysql> UPDATE tb_courses_new
    -> SET course_name='DB',course_grade=3.5
    -> WHERE course_id=2;
Query OK, 1 row affected (0.13 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | DB          |          3.5 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```

 注意：保证 UPDATE 以 WHERE 子句结束，通过 WHERE 子句指定被更新的记录所需要满足的条件，如果忽略 WHERE 子句，MySQL 将更新表中所有的行。

###   DELETE

使用 DELETE 语句从单个表中删除数据，语法格式为： 

```
 DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]
```

 语法说明如下： 

-  `<表名>`：指定要删除数据的表名。
-  `ORDER BY` 子句：可选项。表示删除时，表中各行将按照子句中指定的顺序进行删除。
-  `WHERE` 子句：可选项。表示为删除操作限定删除条件，若省略该子句，则代表删除该表中的所有行。
-  `LIMIT` 子句：可选项。用于告知服务器在控制命令被返回到客户端前被删除行的最大值。

注意：在不使用 WHERE 条件的时候，将删除所有数据。

#### 删除表中的全部数据

【实例 1】删除 tb_courses_new 表中的全部数据，输入的 SQL 语句和执行结果如下所示。 

```
mysql> DELETE FROM tb_courses_new;
Query OK, 3 rows affected (0.12 sec)
mysql> SELECT * FROM tb_courses_new;
Empty set (0.00 sec)
```

#### 根据条件删除表中的数据

【实例 2】在 tb_courses_new 表中，删除 course_id 为 4 的记录，输入的 SQL 语句和执行结果如下所示。 

```
mysql> DELETE FROM tb_courses
    -> WHERE course_id=4;
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
+-----------+-------------+--------------+------------------+
3 rows in set (0.00 sec)
```

 由运行结果可以看出，course_id 为 4 的记录已经被删除。

### 视图

视图是数据库系统中一种非常有用的数据库对象。MySQL 5.0 之后的版本添加了对视图的支持。 

视图是一个虚拟表，其内容由查询定义。同真实表一样，视图包含一系列带有名称的列和行数据，但视图并不是数据库真实存储的数据表。

 视图是从一个、多个表或者视图中导出的表，包含一系列带有名称的数据列和若干条数据行。

 视图并不同于数据表，它们的区别在于以下几点： 

-  视图不是数据库中真实的表，而是一张虚拟表，其结构和数据是建立在对数据中真实表的查询基础上的。
-  存储在数据库中的查询操作 SQL 语句定义了视图的内容，列数据和行数据来自于视图查询所引用的实际表，引用视图时动态生成这些数据。
-  视图没有实际的物理记录，不是以数据集的形式存储在数据库中的，它所对应的数据实际上是存储在视图所引用的真实表中的。
-  视图是数据的窗口，而表是内容。表是实际数据的存放单位，而视图只是以不同的显示方式展示数据，其数据来源还是实际表。
-  视图是查看数据表的一种方法，可以查询数据表中某些字段构成的数据，只是一些 SQL 语句的集合。从安全的角度来看，视图的数据安全性更高，使用视图的用户不接触数据表，不知道表结构。
-  视图的建立和删除只影响视图本身，不影响对应的基本表。

 视图与表在本质上虽然不相同，但视图经过定义以后，结构形式和表一样，可以进行查询、修改、更新和删除等操作。同时，视图具有如下优点： 

1.  定制用户数据，聚焦特定的数据

    在实际的应用过程中，不同的用户可能对不同的数据有不同的要求。例如，当数据库同时存在时，如学生基本信息表、课程表和教师信息表等多种表同时存在时，可以根据需求让不同的用户使用各自的数据。学生查看修改自己基本信息的视图，安排课程人员查看修改课程表和教师信息的视图，教师查看学生信息和课程信息表的视图。 

2. 简化数据操作

    在使用查询时，很多时候要使用聚合函数，同时还要显示其他字段的信息，可能还需要关联到其他表，语句可能会很长，如果这个动作频繁发生的话，可以创建视图来简化操作。 

3. 提高基表数据的安全性

    视图是虚拟的，物理上是不存在的。可以只授予用户视图的权限，而不具体指定使用表的权限，来保护基础数据的安全。 

4. 共享所需数据

    通过使用视图，每个用户不必都定义和存储自己所需的数据，可以共享数据库中的数据，同样的数据只需要存储一次。 

5. 更改数据格式

    通过使用视图，可以重新格式化检索出的数据，并组织输出到其他应用程序中。 

6. 重用 SQL 语句

    视图提供的是对查询操作的封装，本身不包含数据，所呈现的数据是根据视图定义从基础表中检索出来的，如果基础表的数据新增或删除，视图呈现的也是更新后的数据。视图定义后，编写完所需的查询，可以方便地重用该视图。 

>  注意：要区别视图和数据表的本质，即视图是基于真实表的一张虚拟的表，其数据来源均建立在真实表的基础上。

 使用视图的时候，还应该注意以下几点： 

-  创建视图需要足够的访问权限。
-  创建视图的数目没有限制。
-  视图可以嵌套，即从其他视图中检索数据的查询来创建视图。
-  视图不能索引，也不能有关联的触发器、默认值或规则。
-  视图可以和表一起使用。
-  视图不包含数据，所以每次使用视图时，都必须执行查询中所需的任何一个检索操作。如果用多个连接和过滤条件创建了复杂的视图或嵌套了视图，可能会发现系统运行性能下降得十分严重。因此，在部署大量视图应用时，应该进行系统测试。

 ORDER BY 子句可以用在视图中，但若该视图检索数据的 SELECT 语句中也含有 ORDER BY 子句，则该视图中的 ORDER BY 子句将被覆盖。

####  CREATE VIEW

创建视图是指在已经存在的 MySQL 数据库表上建立视图。视图可以建立在一张表中，也可以建立在多张表中。 

 可以使用 CREATE VIEW 语句来创建视图。

 语法格式如下： 

```
 CREATE VIEW <视图名> AS <SELECT语句>
```

 语法说明如下。 

-  `<视图名>`：指定视图的名称。该名称在数据库中必须是唯一的，不能与其他表或视图同名。
-  `<SELECT语句>`：指定创建视图的 SELECT 语句，可用于查询多个基础表或源视图。

 对于创建视图中的 SELECT 语句的指定存在以下限制：

-  用户除了拥有 CREATE VIEW 权限外，还具有操作中涉及的基础表和其他视图的相关权限。
-  SELECT 语句不能引用系统或用户变量。
-  SELECT 语句不能包含 FROM 子句中的子查询。
-  SELECT 语句不能引用预处理语句参数。

 视图定义中引用的表或视图必须存在。但是，创建完视图后，可以删除定义引用的表或视图。可使用 CHECK TABLE 语句检查视图定义是否存在这类问题。

 视图定义中允许使用 ORDER BY 语句，但是若从特定视图进行选择，而该视图使用了自己的 ORDER BY 语句，则视图定义中的 ORDER BY 将被忽略。

 视图定义中不能引用 TEMPORARY 表（临时表），不能创建 TEMPORARY 视图。

 WITH CHECK OPTION 的意思是，修改视图时，检查插入的数据是否符合 WHERE 设置的条件。 

 **创建基于单表的视图**

 MySQL 可以在单个数据表上创建视图。

 查看 test_db 数据库中的 tb_students_info 表的数据，如下所示。 

```
mysql> SELECT * FROM tb_students_info;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John   |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily   |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan  |       4 |   23 | F    |    170 | 2015-10-01 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
| 10 | Tom    |       4 |   23 | M    |    165 | 2016-08-05 |
+----+--------+---------+------+------+--------+------------+
10 rows in set (0.00 sec)
```

 【实例 1】在 tb_students_info 表上创建一个名为 view_students_info 的视图，输入的 SQL 语句和执行结果如下所示。 

```
mysql> CREATE VIEW view_students_info
    -> AS SELECT * FROM tb_students_info;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM view_students_info;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John   |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily   |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan  |       4 |   23 | F    |    170 | 2015-10-01 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
| 10 | Tom    |       4 |   23 | M    |    165 | 2016-08-05 |
+----+--------+---------+------+------+--------+------------+
10 rows in set (0.04 sec)
```

 默认情况下，创建的视图和基本表的字段是一样的，也可以通过指定视图字段的名称来创建视图。

 【实例 2】在 tb_students_info 表上创建一个名为 v_students_info 的视图，输入的 SQL 语句和执行结果如下所示。 

```
mysql> CREATE VIEW v_students_info
    -> (s_id,s_name,d_id,s_age,s_sex,s_height,s_date)
    -> AS SELECT id,name,dept_id,age,sex,height,login_date
    -> FROM tb_students_info;
Query OK, 0 rows affected (0.06 sec)
mysql> SELECT * FROM v_students_info;
+------+--------+------+-------+-------+----------+------------+
| s_id | s_name | d_id | s_age | s_sex | s_height | s_date     |
+------+--------+------+-------+-------+----------+------------+
|    1 | Dany   |    1 |    24 | F     |      160 | 2015-09-10 |
|    2 | Green  |    3 |    23 | F     |      158 | 2016-10-22 |
|    3 | Henry  |    2 |    23 | M     |      185 | 2015-05-31 |
|    4 | Jane   |    1 |    22 | F     |      162 | 2016-12-20 |
|    5 | Jim    |    1 |    24 | M     |      175 | 2016-01-15 |
|    6 | John   |    2 |    21 | M     |      172 | 2015-11-11 |
|    7 | Lily   |    6 |    22 | F     |      165 | 2016-02-26 |
|    8 | Susan  |    4 |    23 | F     |      170 | 2015-10-01 |
|    9 | Thomas |    3 |    22 | M     |      178 | 2016-06-07 |
|   10 | Tom    |    4 |    23 | M     |      165 | 2016-08-05 |
+------+--------+------+-------+-------+----------+------------+
10 rows in set (0.01 sec)
```

 可以看到，view_students_info 和 v_students_info 两个视图中的字段名称不同，但是数据却相同。因此，在使用视图时，可能用户不需要了解基本表的结构，更接触不到实际表中的数据，从而保证了数据库的安全。 

 **创建基于多表的视图**

 MySQL 中也可以在两个以上的表中创建视图，使用 CREATE VIEW 语句创建。

 【实例 3】在表 tb_student_info 和表 tb_departments 上创建视图 v_students_info，输入的 SQL 语句和执行结果如下所示。 

```
mysql> CREATE VIEW v_students_info
    -> (s_id,s_name,d_id,s_age,s_sex,s_height,s_date)
    -> AS SELECT id,name,dept_id,age,sex,height,login_date
    -> FROM tb_students_info;
Query OK, 0 rows affected (0.06 sec)
mysql> SELECT * FROM v_students_info;
+------+--------+------+-------+-------+----------+------------+
| s_id | s_name | d_id | s_age | s_sex | s_height | s_date     |
+------+--------+------+-------+-------+----------+------------+
|    1 | Dany   |    1 |    24 | F     |      160 | 2015-09-10 |
|    2 | Green  |    3 |    23 | F     |      158 | 2016-10-22 |
|    3 | Henry  |    2 |    23 | M     |      185 | 2015-05-31 |
|    4 | Jane   |    1 |    22 | F     |      162 | 2016-12-20 |
|    5 | Jim    |    1 |    24 | M     |      175 | 2016-01-15 |
|    6 | John   |    2 |    21 | M     |      172 | 2015-11-11 |
|    7 | Lily   |    6 |    22 | F     |      165 | 2016-02-26 |
|    8 | Susan  |    4 |    23 | F     |      170 | 2015-10-01 |
|    9 | Thomas |    3 |    22 | M     |      178 | 2016-06-07 |
|   10 | Tom    |    4 |    23 | M     |      165 | 2016-08-05 |
+------+--------+------+-------+-------+----------+------------+
10 rows in set (0.01 sec)
```

 通过这个视图可以很好地保护基本表中的数据。视图中包含 s_id、s_name 和 dept_name，s_id 字段对应  tb_students_info 表中的 id 字段，s_name 字段对应 tb_students_info 表中的 name  字段，dept_name 字段对应 tb_departments 表中的 dept_name 字段。 

####  查询视图

 视图一经定义之后，就可以如同查询数据表一样，使用 SELECT 语句查询视图中的数据，语法和查询基础表的数据一样。

 视图用于查询主要应用在以下几个方面： 

-  使用视图重新格式化检索出的数据。
-  使用视图简化复杂的表连接。
-  使用视图过滤数据。

 DESCRIBE 可以用来查看视图，语法如下： 

```
 DESCRIBE 视图名；
```

【实例 4】通过 DESCRIBE 语句查看视图 v_students_info 的定义，输入的 SQL 语句和执行结果如下所示。 

```
mysql> DESCRIBE v_students_info;
+----------+---------------+------+-----+------------+-------+
| Field    | Type          | Null | Key | Default    | Extra |
+----------+---------------+------+-----+------------+-------+
| s_id     | int(11)       | NO   |     | 0          |       |
| s_name   | varchar(45)   | YES  |     | NULL       |       |
| d_id     | int(11)       | YES  |     | NULL       |       |
| s_age    | int(11)       | YES  |     | NULL       |       |
| s_sex    | enum('M','F') | YES  |     | NULL       |       |
| s_height | int(11)       | YES  |     | NULL       |       |
| s_date   | date          | YES  |     | 2016-10-22 |       |
+----------+---------------+------+-----+------------+-------+
7 rows in set (0.04 sec)
```

>  注意：DESCRIBE 一般情况下可以简写成 DESC，输入这个命令的执行结果和输入 DESCRIBE 是一样的。

#### ALTER VIEW

修改视图是指修改 MySQL 数据库中存在的视图，当基本表的某些字段发生变化时，可以通过修改视图来保持与基本表的一致性。 

可以使用 ALTER VIEW 语句来对已有的视图进行修改。

 语法格式如下： 

```
 ALTER VIEW <视图名> AS <SELECT语句>
```

 语法说明如下： 

-  `<视图名>`：指定视图的名称。该名称在数据库中必须是唯一的，不能与其他表或视图同名。
-  `<SELECT 语句>`：指定创建视图的 SELECT 语句，可用于查询多个基础表或源视图。

 需要注意的是，对于 ALTER VIEW 语句的使用，需要用户具有针对视图的 CREATE VIEW 和 DROP 权限，以及由 SELECT 语句选择的每一列上的某些权限。

 修改视图的定义，除了可以通过 ALTER VIEW 外，也可以使用 DROP VIEW 语句先删除视图，再使用 CREATE VIEW 语句来实现。 

 **修改视图内容**

视图是一个虚拟表，实际的数据来自于基本表，所以通过插入、修改和删除操作更新视图中的数据，实质上是在更新视图所引用的基本表的数据。 

>  注意：对视图的修改就是对基本表的修改，因此在修改时，要满足基本表的数据定义。

 某些视图是可更新的。也就是说，可以使用 UPDATE、DELETE 或 INSERT 等语句更新基本表的内容。对于可更新的视图，视图中的行和基本表的行之间必须具有一对一的关系。

 还有一些特定的其他结构，这些结构会使得视图不可更新。更具体地讲，如果视图包含以下结构中的任何一种，它就是不可更新的： 

-  聚合函数 SUM()、MIN()、MAX()、COUNT() 等。
-  DISTINCT 关键字。
-  GROUP BY 子句。
-  HAVING 子句。
-  UNION 或 UNION ALL 运算符。
-  位于选择列表中的子查询。
-  FROM 子句中的不可更新视图或包含多个表。
-  WHERE 子句中的子查询，引用 FROM 子句中的表。
-  ALGORITHM 选项为 TEMPTABLE（使用临时表总会使视图成为不可更新的）的时候。

【实例 1】使用 ALTER 语句修改视图 view_students_info，输入的 SQL 语句和执行结果如下所示。 

```
mysql> ALTER VIEW view_students_info
    -> AS SELECT id,name,age
    -> FROM tb_students_info;
Query OK, 0 rows affected (0.07 sec)
mysql> DESC view_students_info;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(11)     | NO   |     | 0       |       |
| name  | varchar(45) | YES  |     | NULL    |       |
| age   | int(11)     | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.03 sec)
```

 用户可以通过视图来插入、更新、删除表中的数据，因为视图是一个虚拟的表，没有数据。通过视图更新时转到基本表上进行更新，如果对视图增加或删除记录，实际上是对基本表增加或删除记录。

 查看视图 view_students_info 的数据内容，如下所示。 

```
mysql> SELECT * FROM view_students_info;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | Dany   |   24 |
|  2 | Green  |   23 |
|  3 | Henry  |   23 |
|  4 | Jane   |   22 |
|  5 | Jim    |   24 |
|  6 | John   |   21 |
|  7 | Lily   |   22 |
|  8 | Susan  |   23 |
|  9 | Thomas |   22 |
| 10 | Tom    |   23 |
+----+--------+------+
10 rows in set (0.00 sec)
```

 【实例 2】使用 UPDATE 语句更新视图 view_students_info，输入的 SQL 语句和执行结果如下所示。 

```
mysql> UPDATE view_students_info
    -> SET age=25 WHERE id=1;
Query OK, 0 rows affected (0.24 sec)
Rows matched: 1  Changed: 0  Warnings: 0
mysql> SELECT * FROM view_students_info;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | Dany   |   25 |
|  2 | Green  |   23 |
|  3 | Henry  |   23 |
|  4 | Jane   |   22 |
|  5 | Jim    |   24 |
|  6 | John   |   21 |
|  7 | Lily   |   22 |
|  8 | Susan  |   23 |
|  9 | Thomas |   22 |
| 10 | Tom    |   23 |
+----+--------+------+
10 rows in set (0.00 sec)
```

 查看基本表 tb_students_info 和视图 v_students_info 的内容，如下所示。 

```
mysql> SELECT * FROM tb_students_info;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John   |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily   |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan  |       4 |   23 | F    |    170 | 2015-10-01 |
|  9 | Thomas |       3 |   22 | M    |    178 | 2016-06-07 |
| 10 | Tom    |       4 |   23 | M    |    165 | 2016-08-05 |
+----+--------+---------+------+------+--------+------------+
10 rows in set (0.00 sec)

mysql> SELECT * FROM v_students_info;
+------+--------+------+-------+-------+----------+------------+
| s_id | s_name | d_id | s_age | s_sex | s_height | s_date     |
+------+--------+------+-------+-------+----------+------------+
|    1 | Dany   |    1 |    25 | F     |      160 | 2015-09-10 |
|    2 | Green  |    3 |    23 | F     |      158 | 2016-10-22 |
|    3 | Henry  |    2 |    23 | M     |      185 | 2015-05-31 |
|    4 | Jane   |    1 |    22 | F     |      162 | 2016-12-20 |
|    5 | Jim    |    1 |    24 | M     |      175 | 2016-01-15 |
|    6 | John   |    2 |    21 | M     |      172 | 2015-11-11 |
|    7 | Lily   |    6 |    22 | F     |      165 | 2016-02-26 |
|    8 | Susan  |    4 |    23 | F     |      170 | 2015-10-01 |
|    9 | Thomas |    3 |    22 | M     |      178 | 2016-06-07 |
|   10 | Tom    |    4 |    23 | M     |      165 | 2016-08-05 |
+------+--------+------+-------+-------+----------+------------+
10 rows in set (0.00 sec)
```

 **修改视图名称**

 修改视图的名称可以先将视图删除，然后按照相同的定义语句进行视图的创建，并命名为新的视图名称。

#### DORP VIEW

删除视图是指删除 MySQL 数据库中已存在的视图。删除视图时，只能删除视图的定义，不会删除数据。 

可以使用 DROP VIEW 语句来删除视图。

 语法格式如下： 

```
 DROP VIEW <视图名1> [ , <视图名2> …]
```

 其中：

```
<视图名> 
```

指定要删除的视图名。DROP VIEW 语句可以一次删除多个视图，但是必须在每个视图上拥有 DROP 权限。 

【实例】删除 v_students_info 视图，输入的 SQL 语句和执行过程如下所示。 

```
mysql> DROP VIEW IF EXISTS v_students_info;
Query OK, 0 rows affected (0.00 sec)
mysql> SHOW CREATE VIEW v_students_info;
ERROR 1146 (42S02): Table 'test_db.v_students_info' doesn't exist
```

 可以看到，v_students_info 视图已不存在，将其成功删除。