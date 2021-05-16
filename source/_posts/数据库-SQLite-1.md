---
title: SQLite 入门
date: 2019-05-16 13:18:59
tags:
 - SQLite
 - 数据库
categories:
 - 数据库
 - SQLite
---

SQLite 是一个软件库，实现了自给自足的、无服务器的、零配置的、事务性的 SQL 数据库引擎。SQLite 是在世界上最广泛部署的 SQL 数据库引擎。SQLite 源代码不受版权限制。

就像其他数据库，SQLite 引擎不是一个独立的进程，可以按应用程序需求进行静态或动态连接。SQLite 直接访问其存储文件。

<!--more-->

SQLite，是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。它是D.RichardHipp建立的公有领域项目。它的设计目标是嵌入式的，而且目前已经在很多嵌入式产品中使用了它，它占用资源非常的低，在嵌入式设备中，可能只需要几百K的内存就够了。它能够支持Windows/Linux/Unix等等主流的操作系统，同时能够跟很多程序语言相结合，比如 Tcl、C#、PHP、Java等，还有ODBC接口，同样比起Mysql、PostgreSQL这两款开源的世界著名数据库管理系统来讲，它的处理速度比他们都快。

不像常见的客户-服务器范例，SQLite引擎不是个程序与之通信的独立进程，而是连接到程序中成为它的一个主要部分。所以主要的通信协议是在编程语言内的直接API调用。这在消耗总量、延迟时间和整体简单性上有积极的作用。整个数据库(定义、表、索引和数据本身)都在宿主主机上存储在一个单一的文件中。它的简单的设计是通过在开始一个事务的时候锁定整个数据文件而完成的。

#### 为什么要用 SQLite？

- 不需要一个单独的服务器进程或操作的系统（无服务器的）。
- SQLite 不需要配置，这意味着不需要安装或管理。
- 一个完整的 SQLite 数据库是存储在一个单一的跨平台的磁盘文件。
- SQLite 是非常小的，是轻量级的，完全配置时小于 400KiB，省略可选功能配置时小于250KiB。
- SQLite 是自给自足的，这意味着不需要任何外部的依赖。
- SQLite 事务是完全兼容 ACID 的，允许从多个进程或线程安全访问。
- SQLite 支持 SQL92（SQL2）标准的大多数查询语言的功能。
- SQLite 使用 ANSI-C 编写的，并提供了简单和易于使用的 API。
- SQLite 可在 UNIX（Linux, Mac OS-X, Android, iOS）和 Windows（Win32, WinCE, WinRT）中运行。

#### 特点

1. 轻量级
2. 独立性，没有依赖，无序安装
3. 隔离性 全部在一个文件夹系统
4. 跨平台 支持众多操作系统
5. 多语言接口 支持众多编程语言
6. 安全性 事物，通过独占性和共享锁来实现独立事务的处理，多个进程可以在同一个时间内从同一个数据库读取数据，但只有一个可以写入数据
7. 所支持的数据类型:NULL，INTEGER，Real，text，blob数据类型，依次代表，空值，整型值，浮点值，字符串类型，二进制对象；
8. 动态类型引用（弱引用）：当某个值插入到数据库是，SQlite将会检查他的类型，如果该类型与关联的列不匹配，SQlite则会尝试将改制转换成该列的类型，如果不能转换，则该值将作为本身的类型储存

9. 没有可用于SQlite的网络服务器，只能通过网络共享可能存在文件锁定或者性能问题。

10. 没有用户账户的概念，而是根据文件系统的共享设置

#### SQLite 局限性

在 SQLite 中，SQL92 不支持的特性如下所示：

| 特性             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| RIGHT OUTER JOIN | 只实现了 LEFT OUTER JOIN。                                   |
| FULL OUTER JOIN  | 只实现了 LEFT OUTER JOIN。                                   |
| ALTER TABLE      | 支持 RENAME TABLE 和 ALTER TABLE 的 ADD COLUMN variants 命令，不支持 DROP COLUMN、ALTER COLUMN、ADD CONSTRAINT。 |
| Trigger 支持     | 支持 FOR EACH ROW 触发器，但不支持 FOR EACH STATEMENT 触发器。 |
| VIEWs            | 在 SQLite 中，视图是只读的。您不可以在视图上执行 DELETE、INSERT 或 UPDATE 语句。 |
| GRANT 和 REVOKE  | 可以应用的唯一的访问权限是底层操作系统的正常文件访问权限。   |

#### SQLite 命令

与关系数据库进行交互的标准 SQLite 命令类似于 SQL。命令包括 CREATE、SELECT、INSERT、UPDATE、DELETE 和 DROP。这些命令基于它们的操作性质可分为以下几种：

**DDL - 数据定义语言**

| 命令   | 描述                                                   |
| ------ | ------------------------------------------------------ |
| CREATE | 创建一个新的表，一个表的视图，或者数据库中的其他对象。 |
| ALTER  | 修改数据库中的某个已有的数据库对象，比如一个表。       |
| DROP   | 删除整个表，或者表的视图，或者数据库中的其他对象。     |

**DML - 数据操作语言**

| 命令   | 描述           |
| ------ | -------------- |
| INSERT | 创建一条记录。 |
| UPDATE | 修改记录。     |
| DELETE | 删除记录。     |

**DQL - 数据查询语言**

| 命令   | 描述                           |
| ------ | ------------------------------ |
| SELECT | 从一个或多个表中检索某些记录。 |

#### SQLite 有用的网站

- [SQLite Home Page](http://www.sqlite.org/) - SQLite 官方网站提供了最新的 SQLite 安装版本，最新的 SQLite 资讯以及完整的 SQLite 教程。
- [PHP SQLite3](http://www.php.net/manual/en/book.sqlite3.php) - 网站提供了 SQLite 3 数据库的 PHP 支持的完整细节。
- [SQLite JDBC Driver:](https://bitbucket.org/xerial/sqlite-jdbc) - SQLite JDBC，由 Taro L. Saito 开发的，是一个用于 Java 中访问和创建 SQLite 数据库文件的库。 
- [DBD-SQLite-0.31](http://search.cpan.org/~msergeant/DBD-SQLite-0.31/) - SQLite Perl driver 驱动程序与 Perl DBI 模块一起使用。
- [DBI-1.625](http://search.cpan.org/~timb/DBI/) - Perl DBI 模块为包括 SQLite 在内的任何数据库提供了通用接口。
- [SQLite Python](http://docs.python.org/2/library/sqlite3.html) - sqlite3 python 模块由 Gerhard Haring 编写的。它提供了与 DB-API 2.0 规范兼容的 SQL 接口。

#### SQLite 安装

目前，几乎所有版本的 Linux 操作系统都附带 SQLite。所以，只要使用下面的命令来检查您的机器上是否已经安装了 SQLite。

```bash
$ sqlite3
SQLite version 3.7.15.2 2013-01-09 11:53:05
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite>
```

如果没有看到上面的结果，那么就意味着没有在 Linux 机器上安装 SQLite。因此，让我们按照下面的步骤安装 SQLite：

- 请访问 [SQLite 下载页面](http://www.sqlite.org/download.html)，从源代码区下载 **sqlite-autoconf-\*.tar.gz**。或者下载sqlite-tools-linux-xxx.zip,直接解压使用即可。
- 步骤如下：

```bash
$ tar xvzf sqlite-autoconf-3071502.tar.gz
$ cd sqlite-autoconf-3071502
$ ./configure --prefix=/usr/local
$ make
$ make install
```

上述步骤将在 Linux 机器上安装 SQLite，您可以按照上述讲解的进行验证。

#### SQLite 命令

让我们在命令提示符下键入一个简单的 **sqlite3** 命令，在 SQLite 命令提示符下，您可以使用各种 SQLite 命令。

```bash
$ sqlite3
SQLite version 3.28.0 2019-04-16 19:49:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
```

如需获取可用的点命令的清单，可以在任何时候输入 ".help"。例如：

```sqlite
sqlite>.help
```

上面的命令会显示各种重要的 SQLite 点命令的列表，如下所示：

| 命令                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| .backup ?DB? FILE     | 备份 DB 数据库（默认是 "main"）到 FILE 文件。                |
| .bail ON\|OFF         | 发生错误后停止。默认为 OFF。                                 |
| .databases            | 列出数据库的名称及其所依附的文件。                           |
| .dump ?TABLE?         | 以 SQL 文本格式转储数据库。如果指定了 TABLE 表，则只转储匹配 LIKE 模式的 TABLE 表。 |
| .echo ON\|OFF         | 开启或关闭 echo 命令。                                       |
| .exit                 | 退出 SQLite 提示符。                                         |
| .explain ON\|OFF      | 开启或关闭适合于 EXPLAIN 的输出模式。如果没有带参数，则为 EXPLAIN on，及开启 EXPLAIN。 |
| .header(s) ON\|OFF    | 开启或关闭头部显示。                                         |
| .help                 | 显示消息。                                                   |
| .import FILE TABLE    | 导入来自 FILE 文件的数据到 TABLE 表中。                      |
| .indices ?TABLE?      | 显示所有索引的名称。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表的索引。 |
| .load FILE ?ENTRY?    | 加载一个扩展库。                                             |
| .log FILE\|off        | 开启或关闭日志。FILE 文件可以是 stderr（标准错误）/stdout（标准输出）。 |
| .mode MODE            | 设置输出模式，MODE 可以是下列之一： 1. **csv** 逗号分隔的值 2.**column** 左对齐的列 3.**html** HTML 的 <table\> 代码 4.**insert** TABLE 表的 SQL 插入（insert）语句 5.**line** 每行一个值 6.**list** 由 .separator 字符串分隔的值 7.**tabs** 由 Tab 分隔的值 8.**tcl** TCL 列表元素 |
| .nullvalue STRING     | 在 NULL 值的地方输出 STRING 字符串。                         |
| .output FILENAME      | 发送输出到 FILENAME 文件。                                   |
| .output stdout        | 发送输出到屏幕。                                             |
| .print STRING...      | 逐字地输出 STRING 字符串。                                   |
| .prompt MAIN CONTINUE | 替换标准提示符。                                             |
| .quit                 | 退出 SQLite 提示符。                                         |
| .read FILENAME        | 执行 FILENAME 文件中的 SQL。                                 |
| .schema ?TABLE?       | 显示 CREATE 语句。如果指定了 TABLE 表，则只显示匹配 LIKE 模式的 TABLE 表。 |
| .separator STRING     | 改变输出模式和 .import 所使用的分隔符。                      |
| .show                 | 显示各种设置的当前值。                                       |
| .stats ON\|OFF        | 开启或关闭统计。                                             |
| .tables ?PATTERN?     | 列出匹配 LIKE 模式的表的名称。                               |
| .timeout MS           | 尝试打开锁定的表 MS 毫秒。                                   |
| .width NUM NUM        | 为 "column" 模式设置列宽度。                                 |
| .timer ON\|OFF        | 开启或关闭 CPU 定时器。                                      |

让我们尝试使用 **.show** 命令，来查看 SQLite 命令提示符的默认设置。

```sqlite
sqlite> .show
        echo: off
         eqp: off
     explain: auto
     headers: off
        mode: list
   nullvalue: ""
      output: stdout
colseparator: "|"
rowseparator: "\n"
       stats: off
       width: 
    filename: :memory:
```

> 确保 sqlite> 提示符与点命令之间没有空格，否则将无法正常工作。

**格式化输出**

您可以使用下列的点命令来格式化输出为本教程下面所列出的格式：

```sqlite
sqlite>.header on
sqlite>.mode column
sqlite>.timer on
sqlite>
```

上面设置将产生如下格式的输出：

```sqlite
ID          NAME        AGE         ADDRESS     SALARY
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0
2           Allen       25          Texas       15000.0
3           Teddy       23          Norway      20000.0
4           Mark        25          Rich-Mond   65000.0
5           David       27          Texas       85000.0
6           Kim         22          South-Hall  45000.0
7           James       24          Houston     10000.0
CPU Time: user 0.000000 sys 0.000000
```

**sqlite_master 表格**

主表中保存数据库表的关键信息，并把它命名为 **sqlite_master**。如要查看表概要，可按如下操作：

```sqlite
sqlite> .schema sqlite_master
CREATE TABLE sqlite_master (
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);

```

#### SQLite 语法

有个重要的点值得注意，SQLite 是**不区分大小写**的，但也有一些命令是大小写敏感的，比如 **GLOB** 和 **glob** 在 SQLite 的语句中有不同的含义。

**注释**

SQLite 注释是附加的注释，可以在 SQLite 代码中添加注释以增加其可读性，他们可以出现在任何空白处，包括在表达式内和其他 SQL 语句的中间，但它们不能嵌套。

SQL 注释以两个连续的 "-" 字符（ASCII 0x2d）开始，并扩展至下一个换行符（ASCII 0x0a）或直到输入结束，以先到者为准。

您也可以使用 C 风格的注释，以 "/*" 开始，并扩展至下一个 "*/" 字符对或直到输入结束，以先到者为准。SQLite的注释可以跨越多行。

```
sqlite>.help -- 这是一个简单的注释
```

**SQLite 语句**

所有的 SQLite 语句可以以任何关键字开始，如 SELECT、INSERT、UPDATE、DELETE、ALTER、DROP 等，所有的语句以分号（;）结束。

**SQLite ANALYZE 语句：**

```
ANALYZE;
or
ANALYZE database_name;
or
ANALYZE database_name.table_name;
```

**SQLite AND/OR 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION-1 {AND|OR} CONDITION-2;
```

**SQLite ALTER TABLE 语句：**

```
ALTER TABLE table_name ADD COLUMN column_def...;
```

**SQLite ALTER TABLE 语句（Rename）：**

```
ALTER TABLE table_name RENAME TO new_table_name;
```

**SQLite ATTACH DATABASE 语句：**

```
ATTACH DATABASE 'DatabaseName' As 'Alias-Name';
```

**SQLite BEGIN TRANSACTION 语句：**

```
BEGIN;
or
BEGIN EXCLUSIVE TRANSACTION;
```

**SQLite BETWEEN 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name BETWEEN val-1 AND val-2;
```

**SQLite COMMIT 语句：**

```
COMMIT;
```

**SQLite CREATE INDEX 语句：**

```
CREATE INDEX index_name
ON table_name ( column_name COLLATE NOCASE );
```

**SQLite CREATE UNIQUE INDEX 语句：**

```
CREATE UNIQUE INDEX index_name
ON table_name ( column1, column2,...columnN);
```

**SQLite CREATE TABLE 语句：**

```
CREATE TABLE table_name(
   column1 datatype,
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
   PRIMARY KEY( one or more columns )
);
```

**SQLite CREATE TRIGGER 语句：**

```
CREATE TRIGGER database_name.trigger_name 
BEFORE INSERT ON table_name FOR EACH ROW
BEGIN 
   stmt1; 
   stmt2;
   ....
END;
```

**SQLite CREATE VIEW 语句：**

```
CREATE VIEW database_name.view_name  AS
SELECT statement....;
```

**SQLite CREATE VIRTUAL TABLE 语句：**

```
CREATE VIRTUAL TABLE database_name.table_name USING weblog( access.log );
or
CREATE VIRTUAL TABLE database_name.table_name USING fts3( );
```

**SQLite COMMIT TRANSACTION 语句：**

```
COMMIT;
```

**SQLite COUNT 子句：**

```
SELECT COUNT(column_name)
FROM   table_name
WHERE  CONDITION;
```

**SQLite DELETE 语句：**

```
DELETE FROM table_name
WHERE  {CONDITION};
```

**SQLite DETACH DATABASE 语句：**

```
DETACH DATABASE 'Alias-Name';
```

**SQLite DISTINCT 子句：**

```
SELECT DISTINCT column1, column2....columnN
FROM   table_name;
```

**SQLite DROP INDEX 语句：**

```
DROP INDEX database_name.index_name;
```

**SQLite DROP TABLE 语句：**

```
DROP TABLE database_name.table_name;
```

**SQLite DROP VIEW 语句：**

```
DROP VIEW view_name;
```

**SQLite DROP TRIGGER 语句：**

```
DROP TRIGGER trigger_name
```

**SQLite EXISTS 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name EXISTS (SELECT * FROM   table_name );
```

**SQLite EXPLAIN 语句：**

```
EXPLAIN INSERT statement...;
or 
EXPLAIN QUERY PLAN SELECT statement...;
```

**SQLite GLOB 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name GLOB { PATTERN };
```

**SQLite GROUP BY 子句：**

```
SELECT SUM(column_name)
FROM   table_name
WHERE  CONDITION
GROUP BY column_name;
```

**SQLite HAVING 子句：**

```
SELECT SUM(column_name)
FROM   table_name
WHERE  CONDITION
GROUP BY column_name
HAVING (arithematic function condition);
```

**SQLite INSERT INTO 语句：**

```
INSERT INTO table_name( column1, column2....columnN)
VALUES ( value1, value2....valueN);
```

**SQLite IN 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name IN (val-1, val-2,...val-N);
```

**SQLite Like 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name LIKE { PATTERN };
```

**SQLite NOT IN 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name NOT IN (val-1, val-2,...val-N);
```

**SQLite ORDER BY 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION
ORDER BY column_name {ASC|DESC};
```

**SQLite PRAGMA 语句：**

```
PRAGMA pragma_name;

For example:

PRAGMA page_size;
PRAGMA cache_size = 1024;
PRAGMA table_info(table_name);
```

**SQLite RELEASE SAVEPOINT 语句：**

```
RELEASE savepoint_name;
```

**SQLite REINDEX 语句：**

```
REINDEX collation_name;
REINDEX database_name.index_name;
REINDEX database_name.table_name;
```

**SQLite ROLLBACK 语句：**

```
ROLLBACK;
or
ROLLBACK TO SAVEPOINT savepoint_name;
```

**SQLite SAVEPOINT 语句：**

```
SAVEPOINT savepoint_name;
```

**SQLite SELECT 语句：**

```
SELECT column1, column2....columnN
FROM   table_name;
```

**SQLite UPDATE 语句：**

```
UPDATE table_name
SET column1 = value1, column2 = value2....columnN=valueN
[ WHERE  CONDITION ];
```

**SQLite VACUUM 语句：**

```
VACUUM;
```

**SQLite WHERE 子句：**

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  CONDITION;
```

#### SQLite 数据类型

 SQLite 数据类型是一个用来指定任何对象的数据类型的属性。SQLite 中的每一列，每个变量和表达式都有相关的数据类型。

您可以在创建表的同时使用这些数据类型。SQLite 使用一个更普遍的动态类型系统。在 SQLite 中，值的数据类型与值本身是相关的，而不是与它的容器相关。

**SQLite 存储类**

每个存储在 SQLite 数据库中的值都具有以下存储类之一：

| 存储类  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| NULL    | 值是一个 NULL 值。                                           |
| INTEGER | 值是一个带符号的整数，根据值的大小存储在 1、2、3、4、6 或 8 字节中。 |
| REAL    | 值是一个浮点值，存储为 8 字节的 IEEE 浮点数字。              |
| TEXT    | 值是一个文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储。 |
| BLOB    | 值是一个 blob 数据，完全根据它的输入存储。                   |

SQLite 的存储类稍微比数据类型更普遍。INTEGER 存储类，例如，包含 6 种不同的不同长度的整数数据类型。

**SQLite 亲和(Affinity)类型**

SQLite支持列的亲和类型概念。任何列仍然可以存储任何类型的数据，当数据插入时，该字段的数据将会优先采用亲缘类型作为该值的存储方式。SQLite目前的版本支持以下五种亲缘类型：

| 亲和类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| TEXT     | 数值型数据在被插入之前，需要先被转换为文本格式，之后再插入到目标字段中。 |
| NUMERIC  | 当文本数据被插入到亲缘性为NUMERIC的字段中时，如果转换操作不会导致数据信息丢失以及完全可逆，那么SQLite就会将该文本数据转换为INTEGER或REAL类型的数据，如果转换失败，SQLite仍会以TEXT方式存储该数据。对于NULL或BLOB类型的新数据，SQLite将不做任何转换，直接以NULL或BLOB的方式存储该数据。需要额外说明的是，对于浮点格式的常量文本，如"30000.0"，如果该值可以转换为INTEGER同时又不会丢失数值信息，那么SQLite就会将其转换为INTEGER的存储方式。 |
| INTEGER  | 对于亲缘类型为INTEGER的字段，其规则等同于NUMERIC，唯一差别是在执行CAST表达式时。 |
| REAL     | 其规则基本等同于NUMERIC，唯一的差别是不会将"30000.0"这样的文本数据转换为INTEGER存储方式。 |
| NONE     | 不做任何的转换，直接以该数据所属的数据类型进行存储。         |

**SQLite 亲和类型(Affinity)及类型名称**

下表列出了当创建 SQLite3 表时可使用的各种数据类型名称，同时也显示了相应的亲和类型：

| 数据类型                                                     | 亲和类型 |
| ------------------------------------------------------------ | -------- |
| INT 、INTEGER、 TINYINT、 SMALLINT、 MEDIUMINT、 BIGINT、 UNSIGNED BIG INT、 INT2、 INT8 | INTEGER  |
| CHARACTER(20)、 VARCHAR(255)、 VARYING CHARACTER(255)、 NCHAR(55)、 NATIVE CHARACTER(70)、 NVARCHAR(100)、 TEXT、 CLOB | TEXT     |
| BLOB、 no datatype specified                                 | NONE     |
| REAL、 DOUBLE、 DOUBLE PRECISION、 FLOAT                     | REAL     |
| NUMERIC、 DECIMAL(10,5)、 BOOLEAN、 DATE、 DATETIME          | NUMERIC  |

**Boolean 数据类型**

SQLite 没有单独的 Boolean 存储类。相反，布尔值被存储为整数 0（false）和 1（true）。

**Date 与 Time 数据类型**

SQLite 没有一个单独的用于存储日期和/或时间的存储类，但 SQLite 能够把日期和时间存储为 TEXT、REAL 或 INTEGER 值。

| 存储类  | 日期格式                                                     |
| ------- | ------------------------------------------------------------ |
| TEXT    | 格式为 "YYYY-MM-DD HH:MM:SS.SSS" 的日期。                    |
| REAL    | 从公元前 4714 年 11 月 24 日格林尼治时间的正午开始算起的天数。 |
| INTEGER | 从 1970-01-01 00:00:00 UTC 算起的秒数。                      |

您可以以任何上述格式来存储日期和时间，并且可以使用内置的日期和时间函数来自由转换不同格式。

####  创建数据库

SQLite 的 **sqlite3** 命令被用来创建新的 SQLite 数据库。您不需要任何特殊的权限即可创建一个数据。

sqlite3 命令的基本语法如下：

```
$sqlite3 DatabaseName.db
```

通常情况下，数据库名称在 RDBMS 内应该是唯一的。您可以使用 SQLite **.quit** 命令退出 sqlite 提示符.

示例：

```sqlite
$ sqlite3 test.db
SQLite version 3.28.0 2019-04-16 19:49:53
Enter ".help" for usage hints.
sqlite> .databases
main: /home/hyp/Downloads/test.db
sqlite> .quit
```

**.dump 命令**

您可以在命令提示符中使用 SQLite **.dump** 点命令来导出完整的数据库在一个文本文件中，如下所示：

```
$sqlite3 testDB.db .dump > testDB.sql
```

上面的命令将转换整个 **testDB.db** 数据库的内容到 SQLite 的语句中，并将其转储到 ASCII 文本文件 **testDB.sql** 中。您可以通过简单的方式从生成的 testDB.sql 恢复，如下所示：

```
$sqlite3 testDB.db < testDB.sql
```

此时的数据库是空的，一旦数据库中有表和数据，您可以尝试上述两个程序

**SQLite 附加数据库**

假设这样一种情况，当在同一时间有多个数据库可用，您想使用其中的任何一个。SQLite 的 **ATTACH DATABASE** 语句是用来选择一个特定的数据库，使用该命令后，所有的 SQLite 语句将在附加的数据库下执行。

SQLite 的 ATTACH DATABASE 语句的基本语法如下：

```
ATTACH DATABASE 'DatabaseName' As 'Alias-Name';
```

如果数据库尚未被创建，上面的命令将创建一个数据库，如果数据库已存在，则把数据库文件名称与逻辑数据库 'Alias-Name' 绑定在一起。

如果想附加一个现有的数据库 **testDB.db**，则 ATTACH DATABASE 语句将如下所示：

```
sqlite> ATTACH DATABASE 'testDB.db' as 'TEST';
```

使用 SQLite **.database** 命令来显示附加的数据库。

```
sqlite> .databases
main: /home/hyp/Downloads/test.db
TEST: /home/hyp/Downloads/testDB.db
```

数据库名称 **main** 和 **temp** 被保留用于主数据库和存储临时表及其他临时数据对象的数据库。这两个数据库名称可用于每个数据库连接，且不应该被用于附加，否则将得到一个警告消息，如下所示：

```
sqlite>  ATTACH DATABASE 'testDB.db' as 'TEMP';
Error: database TEMP is already in use
sqlite>  ATTACH DATABASE 'testDB.db' as 'main';
Error: database main is already in use；
```

**SQLite 分离数据库**

SQLite的 **DETACH DTABASE** 语句是用来把命名数据库从一个数据库连接分离和游离出来，连接是之前使用 ATTACH 语句附加的。如果同一个数据库文件已经被附加上多个别名，DETACH 命令将只断开给定名称的连接，而其余的仍然有效。您无法分离 **main** 或 **temp** 数据库。

> 如果数据库是在内存中或者是临时数据库，则该数据库将被摧毁，且内容将会丢失。

SQLite 的 DETACH DATABASE 'Alias-Name' 语句的基本语法如下：

```
DETACH DATABASE 'Alias-Name';
```

在这里，'Alias-Name' 与之前使用 ATTACH 语句附加数据库时所用到的别名相同。

假设在前面的章节中您已经创建了一个数据库，并给它附加了 'test' 和 'currentDB'，使用 .database 命令，我们可以看到：

```
sqlite>.databases
seq  name             file
---  ---------------  ----------------------
0    main             /home/sqlite/testDB.db
2    test             /home/sqlite/testDB.db
3    currentDB        /home/sqlite/testDB.db
```

现在，让我们尝试把 'currentDB' 从 testDB.db 中分离出来，如下所示：

```
sqlite> DETACH DATABASE 'currentDB';
```

现在，如果检查当前附加的数据库，您会发现，testDB.db 仍与 'test' 和 'main' 保持连接。

#### 创建表

SQLite 的 **CREATE TABLE** 语句用于在任何给定的数据库创建一个新表。创建基本表，涉及到命名表、定义列及每一列的数据类型。

CREATE TABLE 语句的基本语法如下：

```
CREATE TABLE database_name.table_name(
   column1 datatype  PRIMARY KEY(one or more columns),
   column2 datatype,
   column3 datatype,
   .....
   columnN datatype,
);
```

CREATE TABLE 是告诉数据库系统创建一个新表的关键字。CREATE TABLE 语句后跟着表的唯一的名称或标识。您也可以选择指定带有 *table_name* 的  *database_name*。

下面是一个实例，它创建了一个 COMPANY 表，ID 作为主键，NOT NULL 的约束表示在表中创建纪录时这些字段不能为 NULL：

```
sqlite> CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);
```

让我们再创建一个表，我们将在随后章节的练习中使用：

```
sqlite> CREATE TABLE DEPARTMENT(
   ID INT PRIMARY KEY      NOT NULL,
   DEPT           CHAR(50) NOT NULL,
   EMP_ID         INT      NOT NULL
);
```

您可以使用 SQLIte 命令中的 **.tables** 命令来验证表是否已成功创建，该命令用于列出附加数据库中的所有表。

在这里，可以看到我们刚创建的两张表 COMPANY、 DEPARTMENT。

您可以使用 SQLite **.schema** 命令得到表的完整信息，如下所示：

```
sqlite>.schema COMPANY
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);
```

**SQLite 删除表**

SQLite 的 **DROP TABLE** 语句用来删除表定义及其所有相关数据、索引、触发器、约束和该表的权限规范。

> 使用此命令时要特别注意，因为一旦一个表被删除，表中所有信息也将永远丢失。

DROP TABLE 语句的基本语法如下。您可以选择指定带有表名的数据库名称，如下所示：

```
DROP TABLE database_name.table_name;
```

让我们先确认 COMPANY 表已经存在，然后我们将其从数据库中删除。

```
sqlite> .tables
COMPANY     DEPARTMENT
sqlite> DROP TABLE COMPANY;
sqlite> .tables
DEPARTMENT
```

#### Insert 语句

SQLite 的 **INSERT INTO** 语句用于向数据库的某个表中添加新的数据行。

INSERT INTO 语句有两种基本语法，如下所示：

```
INSERT INTO TABLE_NAME [(column1, column2, column3,...columnN)]  
VALUES (value1, value2, value3,...valueN);
```

在这里，column1, column2,...columnN 是要插入数据的表中的列的名称。

如果要为表中的所有列添加值，您也可以不需要在 SQLite 查询中指定列名称。但要确保值的顺序与列在表中的顺序一致。SQLite 的 INSERT INTO 语法如下：

```
INSERT INTO TABLE_NAME VALUES (value1,value2,value3,...valueN);
```

假设您已经在 testDB.db 中创建了 COMPANY表，如下所示：

```
sqlite> CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);
```

现在，下面的语句将在 COMPANY 表中创建六个记录：

```
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (1, 'Paul', 32, 'California', 20000.00 );

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (2, 'Allen', 25, 'Texas', 15000.00 );

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (3, 'Teddy', 23, 'Norway', 20000.00 );

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (4, 'Mark', 25, 'Rich-Mond ', 65000.00 );

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (5, 'David', 27, 'Texas', 85000.00 );

INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY)
VALUES (6, 'Kim', 22, 'South-Hall', 45000.00 );
```

您也可以使用第二种语法在 COMPANY 表中创建一个记录，如下所示：

```
INSERT INTO COMPANY VALUES (7, 'James', 24, 'Houston', 10000.00 );
```

**使用一个表来填充另一个表**

您可以通过在一个有一组字段的表上使用 select 语句，填充数据到另一个表中。下面是语法：

```
INSERT INTO first_table_name [(column1, column2, ... columnN)] 
   SELECT column1, column2, ...columnN 
   FROM second_table_name
   [WHERE condition];
```

#### Select 语句

SQLite 的 **SELECT** 语句用于从 SQLite 数据库表中获取数据，以结果表的形式返回数据。这些结果表也被称为结果集。

SQLite 的 SELECT 语句的基本语法如下：

```
SELECT column1, column2, columnN FROM table_name;
```

在这里，column1, column2...是表的字段，他们的值即是您要获取的。如果您想获取所有可用的字段，那么可以使用下面的语法：

```
SELECT * FROM table_name;
```

下面是一个实例，使用 SELECT 语句获取并显示所有这些记录。在这里，前俩个命令被用来设置正确格式化的输出。

```
sqlite> .header on 
sqlite> .mode column
sqlite> SELECT * FROM COMPANY;
ID          NAME        AGE         ADDRESS     SALARY    
----------  ----------  ----------  ----------  ----------
1           Paul        32          California  20000.0   
2           Allen       25          Texas       15000.0   
3           Teddy       23          Norway      20000.0   
4           Mark        25          Rich-Mond   65000.0   
5           David       27          Texas       85000.0   
6           Kim         22          South-Hall  45000.0   
7           James       24          Houston     10000.0  
```

如果只想获取 COMPANY 表中指定的字段，则使用下面的查询：

```
sqlite> SELECT ID, NAME, SALARY FROM COMPANY;
ID          NAME        SALARY    
----------  ----------  ----------
1           Paul        20000.0   
2           Allen       15000.0   
3           Teddy       20000.0   
4           Mark        65000.0   
5           David       85000.0   
6           Kim         45000.0   
7           James       10000.0  
```

**设置输出列的宽度**

有时，由于要显示的列的默认宽度导致 **.mode column**，这种情况下，输出被截断。此时，您可以使用 **.width num, num....** 命令设置显示列的宽度，如下所示：

```
sqlite> .width 10, 20, 10
sqlite> SELECT * FROM COMPANY;
ID          NAME                  AGE         ADDRESS     SALARY    
----------  --------------------  ----------  ----------  ----------
1           Paul                  32          California  20000.0   
2           Allen                 25          Texas       15000.0   
3           Teddy                 23          Norway      20000.0   
4           Mark                  25          Rich-Mond   65000.0   
5           David                 27          Texas       85000.0   
6           Kim                   22          South-Hall  45000.0   
7           James                 24          Houston     10000.0   
```

**Schema 信息**

因为所有的**点命令**只在 SQLite 提示符中可用，所以当您进行带有 SQLite 的编程时，您要使用下面的带有 **sqlite_master** 表的 SELECT 语句来列出所有在数据库中创建的表：

```
sqlite> SELECT tbl_name FROM sqlite_master WHERE type = 'table';
```

假设在 testDB.db 中已经存在唯一的 COMPANY 表，则将产生以下结果：

```
tbl_name
----------
COMPANY
```

您可以列出关于 COMPANY 表的完整信息，如下所示：

```
sqlite> SELECT sql FROM sqlite_master WHERE type = 'table' AND tbl_name = 'COMPANY';
```

假设在 testDB.db 中已经存在唯一的 COMPANY 表，则将产生以下结果：

```
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
)
```

#### 运算符

运算符是一个保留字或字符，主要用于 SQLite 语句的 WHERE 子句中执行操作，如比较和算术运算。

运算符用于指定 SQLite 语句中的条件，并在语句中连接多个条件。

- 算术运算符
- 比较运算符
- 逻辑运算符
- 位运算符

算术运算符

| 运算符 | 描述                                    | 实例              |
| ------ | --------------------------------------- | ----------------- |
| +      | 加法 - 把运算符两边的值相加             | a + b 将得到 30   |
| -      | 减法 - 左操作数减去右操作数             | a - b 将得到 -10  |
| *      | 乘法 - 把运算符两边的值相乘             | a * b 将得到 200  |
| /      | 除法 - 左操作数除以右操作数             | b / a 将得到 2    |
| %      | 取模 - 左操作数除以右操作数后得到的余数 | b % a will give 0 |

比较运算符

| 运算符 | 描述                                                         | 实例              |
| ------ | ------------------------------------------------------------ | ----------------- |
| ==     | 检查两个操作数的值是否相等，如果相等则条件为真。             | (a == b) 不为真。 |
| =      | 检查两个操作数的值是否相等，如果相等则条件为真。             | (a = b) 不为真。  |
| !=     | 检查两个操作数的值是否相等，如果不相等则条件为真。           | (a != b) 为真。   |
| <>     | 检查两个操作数的值是否相等，如果不相等则条件为真。           | (a <> b) 为真。   |
| >      | 检查左操作数的值是否大于右操作数的值，如果是则条件为真。     | (a > b) 不为真。  |
| <      | 检查左操作数的值是否小于右操作数的值，如果是则条件为真。     | (a < b) 为真。    |
| >=     | 检查左操作数的值是否大于等于右操作数的值，如果是则条件为真。 | (a >= b) 不为真。 |
| <=     | 检查左操作数的值是否小于等于右操作数的值，如果是则条件为真。 | (a <= b) 为真。   |
| !<     | 检查左操作数的值是否不小于右操作数的值，如果是则条件为真。   | (a !< b) 为假。   |
| !>     | 检查左操作数的值是否不大于右操作数的值，如果是则条件为真。   | (a !> b) 为真。   |

 逻辑运算符

| 运算符  | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| AND     | AND 运算符允许在一个 SQL 语句的 WHERE 子句中的多个条件的存在。 |
| BETWEEN | BETWEEN 运算符用于在给定最小值和最大值范围内的一系列值中搜索值。 |
| EXISTS  | EXISTS 运算符用于在满足一定条件的指定表中搜索行的存在。      |
| IN      | IN 运算符用于把某个值与一系列指定列表的值进行比较。          |
| NOT IN  | IN 运算符的对立面，用于把某个值与不在一系列指定列表的值进行比较。 |
| LIKE    | LIKE 运算符用于把某个值与使用通配符运算符的相似值进行比较。  |
| GLOB    | GLOB 运算符用于把某个值与使用通配符运算符的相似值进行比较。GLOB 与 LIKE 不同之处在于，它是大小写敏感的。 |
| NOT     | NOT 运算符是所用的逻辑运算符的对立面。比如 NOT EXISTS、NOT BETWEEN、NOT IN，等等。**它是否定运算符。** |
| OR      | OR 运算符用于结合一个 SQL 语句的 WHERE 子句中的多个条件。    |
| IS NULL | NULL 运算符用于把某个值与 NULL 值进行比较。                  |
| IS      | IS 运算符与 = 相似。                                         |
| IS NOT  | IS NOT 运算符与 != 相似。                                    |
| \|\|    | 连接两个不同的字符串，得到一个新的字符串。                   |
| UNIQUE  | UNIQUE 运算符搜索指定表中的每一行，确保唯一性（无重复）。    |

 位运算符

| 运算符 | 描述                                                         | 实例                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| &      | 如果同时存在于两个操作数中，二进制 AND 运算符复制一位到结果中。 | (A & B) 将得到 12，即为 0000 1100                            |
| \|     | 如果存在于任一操作数中，二进制 OR 运算符复制一位到结果中。   | (A \| B) 将得到 61，即为 0011 1101                           |
| ~      | 二进制补码运算符是一元运算符，具有"翻转"位效应，即0变成1，1变成0。 | (~A ) 将得到 -61，即为 1100 0011，一个有符号二进制数的补码形式。 |
| <<     | 二进制左移运算符。左操作数的值向左移动右操作数指定的位数。   | A << 2 将得到 240，即为 1111 0000                            |
| >>     | 二进制右移运算符。左操作数的值向右移动右操作数指定的位数。   | A >> 2 将得到 15，即为 0000 1111                             |

#### 表达式

表达式是一个或多个值、运算符和计算值的SQL函数的组合。

SQL 表达式与公式类似，都写在查询语言中。您还可以使用特定的数据集来查询数据库。

假设 SELECT 语句的基本语法如下：

```
SELECT column1, column2, columnN 
FROM table_name 
WHERE [CONDITION | EXPRESSION];
```

有不同类型的 SQLite 表达式，具体讲解如下：

**布尔表达式**

SQLite 的布尔表达式在匹配单个值的基础上获取数据。语法如下：

```
SELECT column1, column2, columnN 
FROM table_name 
WHERE SINGLE VALUE MATCHING EXPRESSION;
```

示例：

```
sqlite> SELECT * FROM COMPANY WHERE SALARY = 10000;
```

**数值表达式**

这些表达式用来执行查询中的任何数学运算。语法如下：

```
SELECT numerical_expression as  OPERATION_NAME
[FROM table_name WHERE CONDITION] ;
```

在这里，numerical_expression 用于数学表达式或任何公式。下面的实例演示了 SQLite 数值表达式的用法：

```
sqlite> SELECT (15 + 6) AS ADDITION
ADDITION = 21
```

有几个内置的函数，比如 avg()、sum()、count()，等等，执行被称为对一个表或一个特定的表列的汇总数据计算。

```
sqlite> SELECT COUNT(*) AS "RECORDS" FROM COMPANY; 
RECORDS = 7
```

**日期表达式**

日期表达式返回当前系统日期和时间值，这些表达式将被用于各种数据操作。

```
sqlite> SELECT CURRENT_TIMESTAMP; -- 当前系统日期和时间值
CURRENT_TIMESTAMP   
--------------------
2019-06-19 01:43:11
sqlite> select datetime('now','localtime') as time;  -- 本地时区时间
time                
--------------------
2019-06-19 09:43:32 
```

#### Where 子句

SQLite的 WHERE 子句用于指定从一个表或多个表中获取数据的条件。

如果满足给定的条件，即为真（true）时，则从表中返回特定的值。您可以使用 WHERE 子句来过滤记录，只获取需要的记录。

WHERE 子句不仅可用在 SELECT 语句中，它也可用在 UPDATE、DELETE 语句中，等等

SQLite 的带有 WHERE 子句的 SELECT 语句的基本语法如下：

```
SELECT column1, column2, columnN 
FROM table_name
WHERE [condition]
```

您还可以使用比较或逻辑运算符指定条件，比如 >、<、=、LIKE、NOT，等等。

```sqlite
-- 列出了 AGE 大于等于 25 且工资大于等于 65000.00 的所有记录
sqlite> SELECT * FROM COMPANY WHERE AGE >= 25 AND SALARY >= 65000;
-- 列出了 AGE 大于等于 25 或工资大于等于 65000.00 的所有记录
sqlite> SELECT * FROM COMPANY WHERE AGE >= 25 OR SALARY >= 65000;
-- 列出了 AGE 不为 NULL 的所有记录
sqlite>  SELECT * FROM COMPANY WHERE AGE IS NOT NULL;
-- 列出了 NAME 以 'Ki' 开始的所有记录，'Ki' 之后的字符不做限制
sqlite> SELECT * FROM COMPANY WHERE NAME LIKE 'Ki%';
-- 列出了 NAME 以 'Ki' 开始的所有记录，'Ki' 之后的字符不做限制
sqlite> SELECT * FROM COMPANY WHERE NAME GLOB 'Ki*';
-- 列出了 AGE 的值为 25 或 27 的所有记录
sqlite> SELECT * FROM COMPANY WHERE AGE IN ( 25, 27 );
-- 列出了 AGE 的值既不是 25 也不是 27 的所有记录
sqlite> SELECT * FROM COMPANY WHERE AGE NOT IN ( 25, 27 );
-- 列出了 AGE 的值在 25 与 27 之间的所有记录
sqlite> SELECT * FROM COMPANY WHERE AGE BETWEEN 25 AND 27;
-- 下面的 SELECT 语句使用 SQL 子查询，子查询查找 SALARY > 65000 的带有 AGE 字段的所有记录，
-- 后边的 WHERE 子句与 EXISTS 运算符一起使用，列出了外查询中的 AGE 存在于子查询返回的结果中的所有记录
sqlite> SELECT AGE FROM COMPANY 
        WHERE EXISTS (SELECT AGE FROM COMPANY WHERE SALARY > 65000);
-- 子查询查找 SALARY > 65000 的带有 AGE 字段的所有记录，后边的 WHERE 子句与 > 运算符一起使用，
-- 列出了外查询中的 AGE 大于子查询返回的结果中的年龄的所有记录
sqlite> SELECT * FROM COMPANY 
        WHERE AGE > (SELECT AGE FROM COMPANY WHERE SALARY > 65000);
```

 **Like 子句**

SQLite 的 **LIKE** 运算符是用来匹配通配符指定模式的文本值。如果搜索表达式与模式表达式匹配，LIKE 运算符将返回真（true），也就是 1。这里有两个通配符与 LIKE 运算符一起使用：

- 百分号 （%）
- 下划线 （_）

百分号（%）代表零个、一个或多个数字或字符。下划线（_）代表一个单一的数字或字符。这些符号可以被组合使用。

| 语句                      | 描述                                            |
| ------------------------- | ----------------------------------------------- |
| WHERE SALARY LIKE '200%'  | 查找以 200 开头的任意值                         |
| WHERE SALARY LIKE '%200%' | 查找任意位置包含 200 的任意值                   |
| WHERE SALARY LIKE '_00%'  | 查找第二位和第三位为 00 的任意值                |
| WHERE SALARY LIKE '2_%_%' | 查找以 2 开头，且长度至少为 3 个字符的任意值    |
| WHERE SALARY LIKE '%2'    | 查找以 2 结尾的任意值                           |
| WHERE SALARY LIKE '_2%3'  | 查找第二位为 2，且以 3 结尾的任意值             |
| WHERE SALARY LIKE '2___3' | 查找长度为 5 位数，且以 2 开头以 3 结尾的任意值 |

**Glob 子句**

SQLite 的 **GLOB** 运算符是用来匹配通配符指定模式的文本值。如果搜索表达式与模式表达式匹配，GLOB 运算符将返回真（true），也就是 1。与 LIKE 运算符不同的是，GLOB 是大小写敏感的，对于下面的通配符，它遵循 UNIX 的语法。

- 星号 （*）
- 问号 （?）

星号（*）代表零个、一个或多个数字或字符。问号（?）代表一个单一的数字或字符。这些符号可以被组合使用。

下面一些实例演示了 带有 '*' 和 '?' 运算符的 GLOB 子句不同的地方：

| 语句                      | 描述                                            |
| ------------------------- | ----------------------------------------------- |
| WHERE SALARY GLOB '200*'  | 查找以 200 开头的任意值                         |
| WHERE SALARY GLOB '*200*' | 查找任意位置包含 200 的任意值                   |
| WHERE SALARY GLOB '?00*'  | 查找第二位和第三位为 00 的任意值                |
| WHERE SALARY GLOB '2??'   | 查找以 2 开头，且长度至少为 3 个字符的任意值    |
| WHERE SALARY GLOB '*2'    | 查找以 2 结尾的任意值                           |
| WHERE SALARY GLOB '?2*3'  | 查找第二位为 2，且以 3 结尾的任意值             |
| WHERE SALARY GLOB '2???3' | 查找长度为 5 位数，且以 2 开头以 3 结尾的任意值 |

**Limit 子句**

SQLite 的 **LIMIT** 子句用于限制由 SELECT 语句返回的数据数量。

带有 LIMIT 子句的 SELECT 语句的基本语法如下：

```
SELECT column1, column2, columnN 
FROM table_name
LIMIT [no of rows]
```

下面是 LIMIT 子句与 OFFSET 子句一起使用时的语法：

```
SELECT column1, column2, columnN 
FROM table_name
LIMIT [no of rows] OFFSET [row num]
```

SQLite 引擎将返回从下一行开始直到给定的 OFFSET 为止的所有行，如下面的最后一个实例所示。

```
-- 限制了您想要从表中提取的行数
sqlite> SELECT * FROM COMPANY LIMIT 6;
-- 从第三位开始提取 3 个记录
sqlite> SELECT * FROM COMPANY LIMIT 3 OFFSET 2;
```

**Order By**

SQLite 的 **ORDER BY** 子句是用来基于一个或多个列按升序或降序顺序排列数据。

ORDER BY 子句的基本语法如下：

```
SELECT column-list 
FROM table_name 
[WHERE condition] 
[ORDER BY column1, column2, .. columnN] [ASC | DESC];
```

您可以在 ORDER BY 子句中使用多个列。确保您使用的排序列在列清单中。

```
-- 将结果按 SALARY 升序排序
sqlite> SELECT * FROM COMPANY ORDER BY SALARY ASC;
-- 将结果按 NAME 和 SALARY 升序排序
sqlite> SELECT * FROM COMPANY ORDER BY NAME, SALARY ASC;
```

**Group By**

SQLite 的 **GROUP BY**  子句用于与 SELECT 语句一起使用，来对相同的数据进行分组。

在 SELECT 语句中，GROUP BY 子句放在 WHERE 子句之后，放在 ORDER BY 子句之前。

下面给出了 GROUP BY 子句的基本语法。GROUP BY 子句必须放在 WHERE 子句中的条件之后，必须放在 ORDER BY 子句之前。

```
SELECT column-list
FROM table_name
WHERE [ conditions ]
GROUP BY column1, column2....columnN
ORDER BY column1, column2....columnN
```

您可以在 GROUP BY 子句中使用多个列。确保您使用的分组列在列清单中。

```
-- 了解每个客户的工资总额
sqlite> SELECT NAME, SUM(SALARY) FROM COMPANY GROUP BY NAME;
-- 把 ORDER BY 子句与 GROUP BY 子句一起使用
sqlite>  SELECT NAME, SUM(SALARY) 
         FROM COMPANY GROUP BY NAME ORDER BY NAME DESC;
```

**Having 子句**

HAVING 子句允许指定条件来过滤将出现在最终结果中的分组结果。

WHERE 子句在所选列上设置条件，而 HAVING 子句则在由 GROUP BY 子句创建的分组上设置条件。

下面是 HAVING 子句在 SELECT 查询中的位置：

```
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
```

在一个查询中，HAVING 子句必须放在 GROUP BY 子句之后，必须放在 ORDER BY 子句之前。下面是包含 HAVING 子句的 SELECT 语句的语法：

```
SELECT column1, column2
FROM table1, table2
WHERE [ conditions ]
GROUP BY column1, column2
HAVING [ conditions ]
ORDER BY column1, column2
```

示例

```
-- 显示名称计数小于 2 的所有记录
sqlite > SELECT * FROM COMPANY GROUP BY name HAVING count(name) < 2;
-- 显示名称计数大于 2 的所有记录
sqlite > SELECT * FROM COMPANY GROUP BY name HAVING count(name) > 2;
```

**Distinct 关键字**

SQLite 的 **DISTINCT** 关键字与 SELECT 语句一起使用，来消除所有重复的记录，并只获取唯一一次记录。

有可能出现一种情况，在一个表中有多个重复的记录。当提取这样的记录时，DISTINCT 关键字就显得特别有意义，它只获取唯一一次记录，而不是获取重复记录。

用于消除重复记录的 DISTINCT 关键字的基本语法如下：

```
SELECT DISTINCT column1, column2,.....columnN 
FROM table_name
WHERE [condition]
```

示例

```
-- 返回重复的工资记录
sqlite> SELECT name FROM COMPANY;
-- 使用 DISTINCT 关键字
sqlite> SELECT DISTINCT name FROM COMPANY;
```

#### Update 语句

SQLite 的 UPDATE 查询用于修改表中已有的记录。可以使用带有 WHERE 子句的 UPDATE 查询来更新选定行，否则所有的行都会被更新。

带有 WHERE 子句的 UPDATE 查询的基本语法如下：

```
UPDATE table_name
SET column1 = value1, column2 = value2...., columnN = valueN
WHERE [condition];
```

您可以使用 AND 或 OR 运算符来结合 N 个数量的条件。

```
-- 更新 ID 为 6 的客户地址
sqlite> UPDATE COMPANY SET ADDRESS = 'Texas' WHERE ID = 6;
-- 修改 COMPANY 表中 ADDRESS 和 SALARY 列的所有值，则不需要使用 WHERE 子句
sqlite> UPDATE COMPANY SET ADDRESS = 'Texas', SALARY = 20000.00;
-- 按 NAME 降序排序
sqlite> SELECT * FROM COMPANY ORDER BY NAME DESC;


```



#### Delete 语句

SQLite 的 **DELETE** 查询用于删除表中已有的记录。可以使用带有 WHERE 子句的 DELETE 查询来删除选定行，否则所有的记录都会被删除。

带有 WHERE 子句的 DELETE 查询的基本语法如下：

```
DELETE FROM table_name
WHERE [condition];
```

您可以使用 AND 或 OR 运算符来结合 N 个数量的条件。

```
-- 删除 ID 为 7 的客户
sqlite> DELETE FROM COMPANY WHERE ID = 7;
-- 从 COMPANY 表中删除所有记录，则不需要使用 WHERE 子句
sqlite> DELETE FROM COMPANY;
```

