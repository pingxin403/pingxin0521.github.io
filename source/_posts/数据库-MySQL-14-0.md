---
title: MySQL 日志 (一)
date: 2019-10-16 13:18:59
tags:
 - MySQL
 - 数据库
categories:
 - 数据库
 - MySQL
---

不管是哪个数据库产品，一定会有日志文件。在MariaDB/MySQL中，主要有5种日志文件：

1. 错误日志(error log)：记录mysql服务的启停时正确和错误的信息，还记录启动、停止、运行过程中的错误信息。
2. 查询日志(general log)：记录建立的客户端连接和执行的语句。
3. 二进制日志(bin log)：记录所有更改数据的语句，可用于数据复制。
4. 慢查询日志(slow log)：记录所有执行时间超过long_query_time的所有查询或不使用索引的查询。
5. 中继日志(relay log)：主从复制时使用的日志。

除了这5种日志，在需要的时候还会创建DDL日志。

<!--more-->

| 类型         | 介绍                                                         | 作用                                       | 参数              |
| ------------ | ------------------------------------------------------------ | ------------------------------------------ | ----------------- |
| 错误日志     | 记录对数据库的启动、运行、关闭过程进行记录信息               | 记录告警或正确信息                         | log_error         |
| 二进制日志   | 记录对数据库执行更改的所有操作，不包含select和show类型操作信息 | 用于恢复，复制，审计                       | log_bin           |
| 慢查询日志   | 记录对数据库执行长的SQL操作信息                              | 定位存在问题的SQL，从SQL语句层面上进行优化 | slow_query_log    |
| 通用查询日志 | 记录对数据库请求的所有操作信息                               | 用于恢复，审计                             | general_log       |
| 更新日志     | 记录从主服务器接收的从服务器的更新是否应该记录到从设备自己的二进制日志 | 用于复制                                   | log_slave_updates |

**日志刷新操作**

以下操作会刷新日志文件，刷新日志文件时会关闭旧的日志文件并重新打开日志文件。对于有些日志类型，如二进制日志，刷新日志会滚动日志文件，而不仅仅是关闭并重新打开。

```shell
mysql> FLUSH LOGS;
shell> mysqladmin flush-logs
shell> mysqladmin refresh
```

#### 错误日志

错误日志是最重要的日志之一，它记录了MariaDB/MySQL服务启动和停止正确和错误的信息，还记录了mysqld实例运行过程中发生的错误事件信息。

错误日志会记录如下信息：

- mysql执行过程中的错误信息
- mysql执行过程中的警告信息
- event scheduler运行时所产生的信息
- mysql启动和停止过程中的输出信息，未必是错误信息
- 主从复制结构中，从服务器IO复制线程的启动信息

mysql中，错误日志可以通过使用log_error以及log_warnings等参数进行定义。

可以使用`" --log-error=[file_name] "`来指定mysqld记录的错误日志文件，如果没有指定file_name，则默认的错误日志文件为datadir目录下的 `hostname`.err ，hostname表示当前的主机名。

也可以在MariaDB/MySQL配置文件中的mysqld配置部分，使用log-error指定错误日志的路径。

如果不知道错误日志的位置，可以查看变量log_error来查看。

```
mysql> show variables like '%log_error%';
+-----------------+---------------------+
| Variable_name   | Value               |
|-----------------+---------------------|
| log_error       | /var/log/mysqld.log |
+-----------------+---------------------+
```

在MySQL 5.5.7之前，刷新日志操作(如flush logs)会备份旧的错误日志(以_old结尾)，并创建一个新的错误日志文件并打开，在MySQL 5.5.7之后，执行刷新日志的操作时，错误日志会关闭并重新打开，如果错误日志不存在，则会先创建。

在MariaDB/MySQL正在运行状态下删除错误日志后，不会自动创建错误日志，只有在刷新日志的时候才会创建一个新的错误日志文件。

之前说过, log warnings.变量的值也与错误日志有关,那么 log warnings代表什么意思

log warnings用于标识警告信息是否一并记录到错误日志中。

- log Warnings的值为0,表示不记录警告信息
- log warnings的值为1,表示警告信息一并记录到错误日志中。
- log_ Warnings的值大于1,表示"失败的连接"的信息和创建新连接时"拒绝访问"类的错误信息也会被记录到错误日志中

mysql 5.5中 log warnings参数的默认值为1,表示警告信息一并记录到错误日志中

#### 一般查询日志

查询日志分为一般查询日志和慢查询日志，它们是通过查询是否超出变量 long_query_time 指定时间的值来判定的。在超时时间内完成的查询是一般查询，可以将其记录到一般查询日志中，**但是建议关闭这种日志（默认是关闭的）**，超出时间的查询是慢查询，可以将其记录到慢查询日志中。

使用" --general_log={0|1} "来决定是否启用一般查询日志，使用" --general_log_file=file_name "来指定查询日志的路径。不给定路径时默认的文件名以 `hostname`.log 命名。

和查询日志有关的变量有：

```
long_query_time = 10 # 指定慢查询超时时长，超出此时长的属于慢查询，会记录到慢查询日志中
log_output={TABLE|FILE|NONE}  # 定义一般查询日志和慢查询日志的输出格式，不指定时默认为file
```

TABLE表示记录到名为mysql.general_log的表中，这表的默认引擎是CSV，FILE表示记录日志到文件中，NONE表示不记录日志。只要这里指定为NONE，即使开启了一般查询日志和慢查询日志，也都不会有任何记录。

```
mysql > show variables like 'log_output';
+-----------------+---------+
| Variable_name   | Value   |
|-----------------+---------|
| log_output      | FILE    |
+-----------------+---------+

```

和一般查询日志相关的变量有：

```
general_log=off # 是否启用一般查询日志，为全局变量，必须在global上修改。
sql_log_off=off # 在session级别控制是否启用一般查询日志，默认为off，即启用
general_log_file=/mydata/data/hostname.log  # 默认是库文件路径下主机名加上.log
```

MySQL中的参数general_log用来控制开启、关闭MySQL查询日志,参数general_log_file用来控制查询日志的位置。所以如果你要判断MySQL数据库是否开启了查询日志，可以使用下面命令。general_log为ON表示开启查询日志，OFF表示关闭查询日志。

```
mysql> show variables like '%general_log%';
+------------------+--------------------------------------------+
| Variable_name    | Value                                      |
|------------------+--------------------------------------------|
| general_log      | OFF                                        |
| general_log_file | /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ.log |
+------------------+--------------------------------------------+
```

在MySQL 5.6以前的版本还有一个"log"变量也是决定是否开启一般查询日志的。在5.6版本开始已经废弃了该选项。

默认没有开启一般查询日志，也不建议开启一般查询日志。此处打开该类型的日志，看看是如何记录一般查询日志的。

首先开启一般查询日志。

```
mysql> set global general_log = on;
mysql> show variables like '%general_log%';
+------------------+--------------------------------------------+
| Variable_name    | Value                                      |
|------------------+--------------------------------------------|
| general_log      | ON                                         |
| general_log_file | /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ.log |
+------------------+--------------------------------------------+
```

执行几个语句

```
mysql> select host,user from mysql.user;
mysql> show variables like "%error%";
mysql> insert into ttt values(233); 
mysql> create table tt(id int);  
mysql> set @a:=3;
```

可以看到日志文件中新增内容

```
2019-11-28T05:49:35.395658Z	   10 Query	select host,user from mysql.user
2019-11-28T05:49:40.024901Z	   10 Query	show variables like "%error%"
2019-11-28T05:49:41.024901Z	   10 insert into ttt values(233)
2019-11-28T05:49:43.947400Z	   13 Query	create table tt(id int)
2019-11-28T05:49:55.326133Z	   10 Query	set @a:=3
```

由此可知，一般查询日志查询的不止是select语句，几乎所有的语句都会记录。

**查询日志归档**

```
mysql> system mv /var/lib/mysql/DB-Server.log  /var/lib/mysql/DB-Server.log.20170706

mysql> system mysqladmin flush-logs -p

Enter password:
```

或者你在shell中执行下面命令

```shell
[root@DB-Server mysql]# mv /var/lib/mysql/DB-Server.log  /var/lib/mysql/DB-Server.log.20170706
[root@DB-Server mysql]# mysqladmin flush-logs -p
Enter password:
```

**修改查询日志名称或位置**

```
mysql> show variables like 'general_log%';
+------------------+------------------------------+
| Variable_name    | Value                        |
+------------------+------------------------------+
| general_log      | ON                           |
| general_log_file | /var/lib/mysql/DB-Server.log |
+------------------+------------------------------+
2 rows in set (0.00 sec)
 
mysql> set global general_log='OFF';
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global general_log_file='/u02/mysql_log.log';
Query OK, 0 rows affected (0.00 sec)
mysql> set global general_log='ON';
Query OK, 0 rows affected (0.02 sec)
```

如果你遇到下面类似问题，这个是因为权限问题导致。

```shell
mysql> set global general_log_file='/u02/mysql_log.log';

ERROR 1231 (42000): Variable 'general_log_file' can't be set to the value of '/u02/mysql_log.log'

#将对应目录的owner修改为mysql即可解决问题。如下所示：

[root@DB-Server u02]# chown -R mysql:mysql  /u02
```

另外，MySQL的查询日志记录了所有MySQL数据库请求的信息。无论这些请求是否得到了正确的执行。这个就是即使我查询一个不存在的表的SQL，查询日志依然会记录。如下测试所示：

```
mysql> select * from MyDB.test1;
ERROR 1146 (42S02): Table 'MyDB.test1' doesn't exist
mysql> select * from MyDB.test2;
+-------+------+
| id    | sex  |
+-------+------+
| 10001 |      |
| 10002 |      |
| 10003 |     |
+-------+------+
3 rows in set (0.07 sec)
 
mysql> select * from MyDB.kkk;
ERROR 1146 (42S02): Table 'MyDB.kkk' doesn't exist
mysql> 
```

#### 慢查询

**慢查询是什么，如何操作？**

MySQL记录下查询超过指定时间的语句，超过指定时间的SQL语句查询称为“慢查询”

mysql记录慢查询日志是在查询执行完毕且已经完全释放锁之后才记录的，因此慢查询日志记录的顺序和执行的SQL查询语句顺序可能会不一致(例如语句1先执行，查询速度慢，语句2后执行，但查询速度快，则语句2先记录)。

注意，MySQL 5.1之后就支持微秒级的慢查询超时时长，对于DBA来说，一个查询运行0.5秒和运行0.05秒是非常不同的，前者可能索引使用错误或者走了表扫描，后者可能索引使用正确。

另外，指定的慢查询超时时长表示的是超出这个时间的才算是慢查询，等于这个时间的不会记录。

和慢查询有关的变量：

```m
long_query_time=10 # 指定慢查询超时时长(默认10秒)，超出此时长的属于慢查询
log_output={TABLE|FILE|NONE} # 定义一般查询日志和慢查询日志的输出格式，默认为file
log_slow_queries={yes|no}    # 是否启用慢查询日志，默认不启用
slow_query_log={1|ON|0|OFF}  # 也是是否启用慢查询日志，此变量和log_slow_queries修改一个另一个同时变化
slow_query_log_file=/mydata/data/hostname-slow.log  #默认路径为库文件目录下主机名加上-slow.log
log_queries_not_using_indexes=OFF # 查询没有使用索引的时候是否也记入慢查询日志
```

**开启慢查询**

1. `方法一`：用命令开启慢查询

   ```
   查看默认慢查询的时间（10秒）
       mysql> show variables like "%long%";  
       +-----------------+-----------+  
       | Variable_name   | Value     |  
       +-----------------+-----------+  
       | long_query_time | 10.000000 |  
       +-----------------+-----------+  
       1 row in set (0.00 sec)  
   
   设置成2秒，加上global,下次进mysql已然生效 
       mysql> set global long_query_time=2;           
       Query OK, 0 rows affected (0.00 sec)  
   
   查看一下慢查询是不是已经开启 
       mysql> show variables like "%slow%";           
       +---------------------+---------------------------------+  
       | Variable_name       | Value                           |  
       +---------------------+---------------------------------+  
       | log_slow_queries    | OFF                             |  
       | slow_launch_time    | 2                               |  
       | slow_query_log      | OFF                             |  
       | slow_query_log_file | /usr/local/mysql/mysql-slow.log |  
       +---------------------+---------------------------------+  
       4 rows in set (0.00 sec)  
   
   设置开启慢查询
       mysql> set slow_query_log='ON';                          
       ERROR 1229 (HY000): Variable 'slow_query_log' is a GLOBAL variable and should be set with SET GLOBAL  
       
   设置开启慢查询，加上global，不然会报错的
       mysql> set global slow_query_log='ON';         	
       Query OK, 0 rows affected (0.28 sec)  
   
   查看是否已经开启 
       mysql> show variables like "%slow%";               
   +---------------------------+-------------------------------------------------+
   | Variable_name             | Value                                           |
   |---------------------------+-------------------------------------------------|
   | log_slow_admin_statements | OFF                                             |
   | log_slow_extra            | OFF                                             |
   | log_slow_slave_statements | OFF                                             |
   | slow_launch_time          | 2                                               |
   | slow_query_log            | OFF                                             |
   | slow_query_log_file       | /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log |
   +---------------------------+-------------------------------------------------+
   
       4 rows in set (0.00 sec)  
      
   开启执行sql日志
   	set  global general_log='ON';  
   
   ```

2. `方法二`：修改mysql的配置文件my.cnfgeneral_log

   ```
   在[mysqld]里面加上以下内容
       long_query_time = 2  
       log-slow-queries = /usr/local/mysql/mysql-slow.log  
   
   重起一下
   	/usr/local/mysql/libexec/mysqld restart
   ```

##### 优化方式

直接分析mysql慢查询日志 ,利用explain关键字可以模拟优化器执行SQL查询语句，来分析sql慢查询语句

```
mysql>SELECT sleep(10); #模拟

#log文件
# Time: 2019-11-28T06:07:29.556122Z
# User@Host: hyp[hyp] @  [117.84.72.27]  Id:    13
# Query_time: 10.000298  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use test;
SET timestamp=1574921239;
SELECT sleep(10);
```

随着时间的推移，慢查询日志文件中的记录可能会变得非常多，这对于分析查询来说是非常困难的。好在提供了一个专门归类慢查询日志的工具mysqldumpslow。

```
# mysqldumpslow --help
  -d           debug 
  -v           verbose：显示详细信息
  -t NUM       just show the top n queries：仅显示前n条查询
  -a           don't abstract all numbers to N and strings to 'S'：归类时不要使用N替换数字，S替换字符串
  -g PATTERN   grep: only consider stmts that include this string：通过grep来筛选select语句。

```

该工具归类的时候，默认会将**同文本但变量值不同的查询语句视为同一类，并使用N代替其中的数值变量，使用S代替其中的字符串变量**。可以使用-a来禁用这种替换

```shell
# mysqldumpslow  /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log

Reading mysql slow query log from /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log
Count: 1  Time=10.00s (10s)  Lock=0.00s (0s)  Rows=1.0 (1), hyp[hyp]@[117.84.72.27]
  SELECT sleep(N)
  
# mysqldumpslow -a  /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log

Reading mysql slow query log from /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log
Count: 1  Time=10.00s (10s)  Lock=0.00s (0s)  Rows=1.0 (1), hyp[hyp]@[117.84.72.27]
  SELECT sleep(10)
```

如果想要显示及其精确的秒数，则使用-d选项启用调试功能。

```shell
# mysqldumpslow -d  /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log

Reading mysql slow query log from /var/lib/mysql/iZbp125p95hhdb1kcx81ogZ-slow.log
[[/usr/sbin/mysqld, Version: 8.0.18 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 2019-11-28T06:07:29.556122Z
# User@Host: hyp[hyp] @  [117.84.72.27]  Id:    13
# Query_time: 10.000298  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use test;
SET timestamp=1574921239;
SELECT sleep(10);
]]
<<>>
<<# Time: 2019-11-28T06:07:29.556122Z
# User@Host: hyp[hyp] @  [117.84.72.27]  Id:    13
# Query_time: 10.000298  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use test;
SET timestamp=1574921239;
SELECT sleep(10);
>> at /usr/bin/mysqldumpslow line 98, <> chunk 1.
[[# Time: 2019-11-28T06:07:29.556122Z
# User@Host: hyp[hyp] @  [117.84.72.27]  Id:    13
# Query_time: 10.000298  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
use test;
SET timestamp=1574921239;
SELECT sleep(10);
]]
{{  SELECT sleep(N)}}

Count: 1  Time=10.00s (10s)  Lock=0.00s (0s)  Rows=1.0 (1), hyp[hyp]@[117.84.72.27]
  SELECT sleep(N)

```

慢查询在SQL语句调优的时候非常有用，应该将它启用起来，且应该让慢查询阈值尽量小，例如1秒甚至低于1秒。就像一天执行上千次的1秒语句，和一天执行几次的20秒语句，显然更值得去优化这个1秒的语句。

#### 二进制日志

##### 二进制日志文件

二进制日志包含了**引起或可能引起数据库改变**(如delete语句但没有匹配行)的事件信息，但绝不会包括select和show这样的查询语句。语句以"事件"的形式保存，所以包含了时间、事件开始和结束位置等信息。

二进制日志是**以事件形式记录的，不是事务日志(但可能是基于事务来记录二进制日志)**，不代表它只记录innodb日志，myisam表也一样有二进制日志。

对于事务表的操作，二进制日志**只在事务提交的时候一次性写入(基于事务的innodb二进制日志)，提交前的每个二进制日志记录都先cache，提交时写入**。对于非事务表的操作，每次执行完语句就直接写入。

MariaDB/MySQL默认没有启动二进制日志，要启用二进制日志使用 --log-bin=[on|off|file_name] 选项指定，如果没有给定file_name，则默认为datadir下的主机名加"-bin"，并在后面跟上一串数字表示日志序列号，如果给定的日志文件中包含了后缀(logname.suffix)将忽略后缀部分。

或者在配置文件中的[mysqld]部分设置log-bin也可以。注意：对于mysql 5.7，直接启动binlog可能会导致mysql服务启动失败，这时需要在配置文件中的mysqld为mysql实例分配server_id。

```
[mysqld]
# server_id=1234
log-bin=[on|filename]
```

mysqld还**创建一个二进制日志索引文件**，当二进制日志文件滚动的时候会向该文件中写入对应的信息。所以该文件包含所有使用的二进制日志文件的文件名。默认情况下该文件与二进制日志文件的文件名相同，扩展名为'.index'。要指定该文件的文件名使用 --log-bin-index[=file_name] 选项。当mysqld在运行时不应手动编辑该文件，免得mysqld变得混乱。

当重启mysql服务或刷新日志或者达到日志最大值时，将滚动二进制日志文件，滚动日志时只修改日志文件名的数字序列部分。

二进制日志文件的最大值通过变量 max_binlog_size 设置(默认值为1G)。但由于二进制日志可能是基于事务来记录的(如innodb表类型)，而事务是绝对不可能也不应该跨文件记录的，如果正好二进制日志文件达到了最大值但事务还没有提交则不会滚动日志，而是继续增大日志，所以 max_binlog_size 指定的值和实际的二进制日志大小不一定相等。

因为二进制日志文件增长迅速，但官方说明因此而损耗的性能小于1%，且二进制目的是为了恢复定点数据库和主从复制，所以出于安全和功能考虑，**极不建议将二进制日志和datadir放在同一磁盘上**。

##### 查看相关信息

查看binlog相关信息：

```mysql
mysql> SHOW VARIABLES LIKE '%log_bin%'
+---------------------------------+-----------------------------+
| Variable_name                   | Value                       |
|---------------------------------+-----------------------------|
| log_bin                         | ON                          |
| log_bin_basename                | /var/lib/mysql/binlog       |
| log_bin_index                   | /var/lib/mysql/binlog.index |
| log_bin_trust_function_creators | OFF                         |
| log_bin_use_v1_row_events       | OFF                         |
| sql_log_bin                     | ON                          |
+---------------------------------+-----------------------------+

#SHOW {BINARY | MASTER} LOGS      # 查看使用了哪些日志文件
#SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos]   # 查看日志中进行了哪些操作
#SHOW MASTER STATUS         # 显式主服务器中的二进制日志信息

mysql> SHOW BINARY LOGS;
+---------------+-------------+-------------+
| Log_name      |   File_size | Encrypted   |
|---------------+-------------+-------------|
| binlog.000001 |         996 | No          |
| binlog.000002 |         528 | No          |
+---------------+-------------+-------------+
mysql> SHOW MASTER LOGS;
+---------------+-------------+-------------+
| Log_name      |   File_size | Encrypted   |
|---------------+-------------+-------------|
| binlog.000001 |         996 | No          |
| binlog.000002 |         528 | No          |
+---------------+-------------+-------------+
mysql>SHOW MASTER STATUS;
+---------------+------------+----------------+--------------------+---------------------+
| File          |   Position | Binlog_Do_DB   | Binlog_Ignore_DB   | Executed_Gtid_Set   |
|---------------+------------+----------------+--------------------+---------------------|
| binlog.000002 |        528 |                |                    |                     |
+---------------+------------+----------------+--------------------+---------------------+

mysql> flush logs; #刷新日志
mysql>reset master; #删除所有的二进制日志文件
mysql>purge master logs to 'mysql-bin.000003';#编号小于 000003 的日志会被删除。
mysql>purge master logs before 'datatime'; # 删除 datatime 之前的日志，日期格式是 YYYY-MM-DD hh:mm:ss
#如果日志文件正在用于主从备份，它不会被删除。
```

**mysqlbinlog**

二进制日志可以使用mysqlbinlog命令查看。

```
mysqlbinlog [option] log-file1 log-file2...
```

以下是常用的几个选项：

```
-d,--database=name：只查看指定数据库的日志操作
-o,--offset=#：忽略掉日志中的前n个操作命令
-r,--result-file=name：将输出的日志信息输出到指定的文件中，使用重定向也一样可以。
-s,--short-form：显示简单格式的日志，只记录一些普通的语句，会省略掉一些额外的信息如位置信息和时间信息以及基于行的日志。可以用来调试，生产环境千万不可使用
--set-charset=char_name：在输出日志信息到文件中时，在文件第一行加上set names char_name
--start-datetime,--stop-datetime：指定输出开始时间和结束时间内的所有日志信息
--start-position=#,--stop-position=#：指定输出开始位置和结束位置内的所有日志信息
-v,-vv：显示更详细信息，基于row的日志默认不会显示出来，此时使用-v或-vv可以查看
```

查看二进制文件信息：

```shell
mysql >reset master;
mysql >exit

$ mysqlbinlog  /var/lib/mysql/binlog.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#191128 16:44:12 server id 1  end_log_pos 124 CRC32 0x96763101 	Start: binlog v 4, server v 8.0.18 created 191128 16:44:12 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
3IjfXQ8BAAAAeAAAAHwAAAABAAQAOC4wLjE4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAADciN9dEwANAAgAAAAABAAEAAAAYAAEGggAAAAICAgCAAAACgoKKioAEjQA
CgEBMXaW
'/*!*/;
# at 124
#191128 16:44:12 server id 1  end_log_pos 155 CRC32 0xe46eb2ee 	Previous-GTIDs
# [empty]
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

现在在数据库中执行下面的操作：

```mysql
use test;
create table student(studentid int not null primary key,name varchar(30) not null,gender enum('female','mail'));
alter table student change gender gender enum('female','male');
insert into student values(1,'malongshuai','male'),(2,'gaoxiaofang','female');
```

再查看二进制日志信息。

```shell
$ mysqlbinlog /var/lib/mysql/binlog.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#191128 16:44:12 server id 1  end_log_pos 124 CRC32 0x96763101 	Start: binlog v 4, server v 8.0.18 created 191128 16:44:12 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
3IjfXQ8BAAAAeAAAAHwAAAABAAQAOC4wLjE4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAADciN9dEwANAAgAAAAABAAEAAAAYAAEGggAAAAICAgCAAAACgoKKioAEjQA
CgEBMXaW
'/*!*/;
# at 124
#191128 16:44:12 server id 1  end_log_pos 155 CRC32 0xe46eb2ee 	Previous-GTIDs
# [empty]
# at 155
#191128 16:44:52 server id 1  end_log_pos 234 CRC32 0x659171a7 	Anonymous_GTID	last_committed=0	sequence_number=1	rbr_only=no	original_committed_timestamp=1574930692892828	immediate_commit_timestamp=1574930692892828	transaction_length=278
# original_commit_timestamp=1574930692892828 (2019-11-28 16:44:52.892828 CST)
# immediate_commit_timestamp=1574930692892828 (2019-11-28 16:44:52.892828 CST)
/*!80001 SET @@session.original_commit_timestamp=1574930692892828*//*!*/;
/*!80014 SET @@session.original_server_version=80018*//*!*/;
/*!80014 SET @@session.immediate_server_version=80018*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 234
#191128 16:44:52 server id 1  end_log_pos 433 CRC32 0xc42bdd20 	Query	thread_id=34	exec_time=0	error_code=0	Xid = 152
use `test`/*!*/;
SET TIMESTAMP=1574930692/*!*/;
SET @@session.pseudo_thread_id=34/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1168113696/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=255/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
/*!80011 SET @@session.default_collation_for_utf8mb4=255*//*!*/;
/*!80013 SET @@session.sql_require_primary_key=0*//*!*/;
create table student(studentid int not null primary key,name varchar(30) not null,gender enum('female','mail'))
/*!*/;
# at 433
#191128 16:44:52 server id 1  end_log_pos 510 CRC32 0x522e7bd9 	Anonymous_GTID	last_committed=1	sequence_number=2	rbr_only=no	original_committed_timestamp=1574930692957930	immediate_commit_timestamp=1574930692957930	transaction_length=227
# original_commit_timestamp=1574930692957930 (2019-11-28 16:44:52.957930 CST)
# immediate_commit_timestamp=1574930692957930 (2019-11-28 16:44:52.957930 CST)
/*!80001 SET @@session.original_commit_timestamp=1574930692957930*//*!*/;
/*!80014 SET @@session.original_server_version=80018*//*!*/;
/*!80014 SET @@session.immediate_server_version=80018*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 510
#191128 16:44:52 server id 1  end_log_pos 660 CRC32 0xc120d309 	Query	thread_id=34	exec_time=0	error_code=0	Xid = 153
SET TIMESTAMP=1574930692/*!*/;
/*!80013 SET @@session.sql_require_primary_key=0*//*!*/;
alter table student change gender gender enum('female','male')
/*!*/;
# at 660
#191128 16:44:52 server id 1  end_log_pos 739 CRC32 0xcc290557 	Anonymous_GTID	last_committed=2	sequence_number=3	rbr_only=yes	original_committed_timestamp=1574930692982602	immediate_commit_timestamp=1574930692982602	transaction_length=320
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
# original_commit_timestamp=1574930692982602 (2019-11-28 16:44:52.982602 CST)
# immediate_commit_timestamp=1574930692982602 (2019-11-28 16:44:52.982602 CST)
/*!80001 SET @@session.original_commit_timestamp=1574930692982602*//*!*/;
/*!80014 SET @@session.original_server_version=80018*//*!*/;
/*!80014 SET @@session.immediate_server_version=80018*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 739
#191128 16:44:52 server id 1  end_log_pos 814 CRC32 0xacb3825b 	Query	thread_id=34	exec_time=0	error_code=0
SET TIMESTAMP=1574930692/*!*/;
BEGIN
/*!*/;
# at 814
#191128 16:44:52 server id 1  end_log_pos 878 CRC32 0x85a68c50 	Table_map: `test`.`student` mapped to number 161
# at 878
#191128 16:44:52 server id 1  end_log_pos 949 CRC32 0xfe8505c0 	Write_rows: table id 161 flags: STMT_END_F

BINLOG '
BInfXRMBAAAAQAAAAG4DAAAAAKEAAAAAAAEABHRlc3QAB3N0dWRlbnQAAwMP/gR4APcBBAEBAAID
/P8AUIymhQ==
BInfXR4BAAAARwAAALUDAAAAAKEAAAAAAAEAAgAD/wABAAAAC21hbG9uZ3NodWFpAgACAAAAC2dh
b3hpYW9mYW5nAcAFhf4=
'/*!*/;
# at 949
#191128 16:44:52 server id 1  end_log_pos 980 CRC32 0x24c0afc0 	Xid = 154
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

- 位置4-124记录的是二进制日志的一些固定信息。
- 位置124-433记录的是use和create table语句，语句的记录时间为16:44:12。但注意，这里的use不是执行的use语句，而是MySQL发现要操作的数据库为test，而自动进行的操作并记录下来。人为的use语句是不会记录的。
- 位置433-660记录的是alter table语句，语句的记录时间为5:20:21。
- 位置660-980记录的是insert操作，因为该操作是DML语句，因此记录了事务的开始BEGIN和提交COMMIT。
  - begin的起止位置为660-814；
  - insert into语句的起止位置为814-949，记录的时间和自动开启事务的begin时间是一样的；
  - commit的起止位置为949-980。

二进制日志文件Binlog的格式主要有三种：

- Statement：基于SQL语句级别的Binlog，每条修改数据的SQL都会保存到Binlog里面。
- ROW：基于行级别，每一行数据的变化都会记录到Binlog里面，但是并不记住原始SQL语句，因此它会记录的非常详细，日志量也比statement格式记录的多得多。在主从复制中，这样的Binlog格式不会因存储过程或触发器原因造成主从数据不一致的问题。
- Mixed：混合Statement和Row模式。

在mysql5.7中，默认是ROW模式记录，

```
mysql> show variables like 'binlog_format%';
+-----------------+---------+
| Variable_name   | Value   |
|-----------------+---------|
| binlog_format   | ROW     |
+-----------------+---------+
```

上面的三种复制模式也对应了MySQL的三种复制技术：

- binlog_format=Statement：基于SQL语句的复制，在MySQL5.1.4之前的版本都是基于SQL语句的复制

- binlog_format=ROW：基于行的复制

- binlog_Mixed：混合复制模式，基于行的复制和基于SQL语句的复制。这种模式下默认会采用statement的方式记录，只有以下几种情况会采用row的形式来记录日志。
  - 表的存储引擎为NDB，这时对表的DML操作都会以row的格式记录。
  - 使用了uuid()、user()、current_user()、found_rows()、row_count()等不确定函数。但测试发现对now()函数仍会以statement格式记录，而sysdate()函数会以row格式记录。
  - 使用了insert delay语句。
  - 使用了临时表。

在MySQL的主从复制中，主库的Binlog_format设置为ROW格式比Statement格式更能保证从库数据的一致性，只是ROW格式下的Binlog日志可能会增大非常多，在设置时需要考虑磁盘空间问题。

##### 删除二进制日志

删除二进制日志有几种方法。不管哪种方法，都会将删除后的信息同步到二进制index文件中。

1. reset master将会删除所有日志，并让日志文件重新从000001开始。

   ```
   mysql> reset master;
   ```

2. `PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr }`

    purge master logs to "binlog_name.00000X" 将会清空00000X之前的所有日志文件。例如删除000006之前的日志文件。

   ```
   mysql> purge master logs to "mysql-bin.000006";
   mysql> purge binary logs to "mysql-bin.000006";
   ```

   master和binary是同义词

    purge master logs before 'yyyy-mm-dd hh:mi:ss' 将会删除指定日期之前的所有日志。但是若指定的时间处在正在使用中的日志文件中，将无法进行purge。

3. 使用--expire_logs_days=N选项指定过了多少天日志自动过期清空。

##### 二进制日志相关的变量

注意：在配置binlog相关变量的时候，相关变量名总是搞混，因为有的是binlog，有的是log_bin，当他们分开的时候，log在前，当它们一起的时候，bin在前。在配置文件中也同样如此。

- `log_bin = {on | off | base_name}` #指定是否启用记录二进制日志或者指定一个日志路径(路径不能加.否则.后的被忽略)
- `sql_log_bin ={ on | off }` #指定是否启用记录二进制日志，只有在log_bin开启的时候才有效
- `expire_logs_days =` #指定自动删除二进制日志的时间，即日志过期时间
- `binlog_do_db =` #明确指定要记录日志的数据库
- `binlog_ignore_db =` #指定不记录二进制日志的数据库
- `log_bin_index =` #指定mysql-bin.index文件的路径
- `binlog_format = { mixed | row | statement }` #指定二进制日志基于什么模式记录
- `binlog_rows_query_log_events = { 1|0 }` # MySQL5.6.2添加了该变量，当binlog format为row时，默认不会记录row对应的SQL语句，设置为1或其他true布尔值时会记录，但需要使用mysqlbinlog -v查看，这些语句是被注释的，恢复时不会被执行。
- `max_binlog_size =` #指定二进制日志文件最大值，超出指定值将自动滚动。但由于事务不会跨文件，所以并不一定总是精确。
- `binlog_cache_size = 32768` #**基于事务类型的日志会先记录在缓冲区**，当达到该缓冲大小时这些日志会写入磁盘
- `max_binlog_cache_size =` #指定二进制日志缓存最大大小，硬限制。默认4G，够大了，建议不要改
- `binlog_cache_use`：使用缓存写二进制日志的次数(这是一个实时变化的统计值)
- `binlog_cache_disk_use`:使用临时文件写二进制日志的次数，当日志超过了binlog_cache_size的时候会使用临时文件写日志，如果该变量值不为0，则考虑增大binlog_cache_size的值
- `binlog_stmt_cache_size = 32768` #一般等同于且决定binlog_cache_size大小，所以修改缓存大小时只需修改这个而不用修改binlog_cache_size
- `binlog_stmt_cache_use`：使用缓存写二进制日志的次数
- `binlog_stmt_cache_disk_use`: 使用临时文件写二进制日志的次数，当日志超过了binlog_cache_size的时候会使用临时文件写日志，如果该变量值不为0，则考虑增大binlog_cache_size的值
- `sync_binlog = { 0 | n }` #这个参数直接影响mysql的性能和完整性
  - `sync_binlog=0`:不同步，日志何时刷到磁盘由FileSystem决定，这个性能最好。
  - `sync_binlog=n`:每写n次二进制日志事件(不是事务)，MySQL将执行一次磁盘同步指令fdatasync()将缓存日志刷新到磁盘日志文件中。Mysql中默认的设置是sync_binlog=0，即不同步，这时性能最好，但风险最大。一旦系统奔溃，缓存中的日志都会丢失。

**在innodb的主从复制结构中，如果启用了二进制日志(几乎都会启用)，要保证事务的一致性和持久性的时候，必须将sync_binlog的值设置为1，因为每次事务提交都会写入二进制日志，设置为1就保证了每次事务提交时二进制日志都会写入到磁盘中，从而立即被从服务器复制过去。**

##### 二进制日志定点还原数据库

在使用 mysql 的全量备份恢复数据库之后，可以再使用二进制日志恢复到指定时间点。比如当前时间是11点多，我不小心把一个重要的库删除了。幸好每天凌晨两点做了数据库全量备份，这时我就可以先恢复数据库到两点，然后再使用二进制日志恢复到删除之前的数据库。

直接使用日志文件恢复数据库：

```shell
shell> mysqlbinlog binlog_files | mysql -u root -p
```

可以把日志文件导出文本，然后编辑文本，删除一些不需要执行的语句，然后再恢复数据库：

```shell
shell> mysqlbinlog binlog_files > tmpfile
shell> ... edit tmpfile ...
shell> mysql -u root -p < tmpfile
```

如果有多个日志文件，在一个连接中使用它们：

```shell
shell> mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p
```

恢复到指定时间点：

```shell
shell> mysqlbinlog --stop-datetime="2018-10-25 11:00:00" \
         /var/log/mysql/bin.123456 | mysql -u root -p
```

从指定时间点恢复：

```shell
shell> mysqlbinlog --start-datetime="2018-10-25 10:01:00" \
         /var/log/mysql/bin.123456 | mysql -u root -p
```

根据位置恢复：

```shell
shell> mysqlbinlog --stop-position=368312 /var/log/mysql/bin.123456 \
         | mysql -u root -p

shell> mysqlbinlog --start-position=368315 /var/log/mysql/bin.123456 \
         | mysql -u root -p
```

位置信息可以从日志文件的 `log_pos` 获取。
