---
title: MySQL SQL 备份--MySQL 工具(九)
date: 2019-05-17 13:08:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

#### 备份分类

按照是否能够继续提供服务，将数据库备份类型划分为：

- 热备份：在线备份，能读能写
- 温备份：能读不能写
- 冷备份：离线备份

按照备份数据库对象分类：

- 物理备份：直接复制数据文件
- 逻辑备份：将数据导出至文件中，必要时将其还原(也包括备份成sql语句的方式)

按照是否备份整个数据集分为：

- 完全备份：备份从开始到某段时间的全部数据
- 差异备份：备份自完全备份以来变化的数据
- 增量备份：备份自上次增量备份以来变化的数据

分类方式不同，不同分类的备份没有冲突的关系，它们可以任意组合。

<!--more-->

#### 备份内容和备份工具

需要备份的内容：文件、二进制日志、事务日志、配置文件、操作系统上和MySQL相关的配置（如sudo，定时任务）。

物理备份和逻辑备份的优缺点：

- 物理备份：直接复制数据文件，速度较快。
- 逻辑备份：将数据导出到文本文件中或其他格式的文件中。有MySQL服务进程参与，相比物理备份而言速度较慢；可能丢失浮点数精度；但可以使用文本工具二次处理；可以跨版本和跨数据库系统进行移植。

备份策略：要考虑安全，也要考虑还原时长

- 完全备份+增量
- 完全备份+差异

备份工具：

- mysqldump：逻辑备份工具。要求mysql服务在线。MyISAM(温备)，InnoDB（热备）
- mysqlhotcopy：物理备份工具，温备份，实际上是冷备。加锁、flush table并进行cp或scp。即将废弃的工具
- cp:冷备
- lvm快照：几乎热备。注意点是：先flush table、lock table、创建快照、释放锁、复制数据。因为要先flush table和lock table，这对于MyISAM来说很简单很容易实现。但**对于InnoDB来说，因为事务的原因，锁表后可能还有缓存中的数据在写入文件中，所以应该监控缓存中的数据是真的已经完全写入数据文件中**，之后才能进行复制数据。
- XtraBackup:开源。MyISAM（温备），InnoDB（热备），速度较快。

### mysqldump工具

官方手册：https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html

mysqldump默认会从配置文件中的mysqldump段读取选项，配置文件读取的顺序为：

```
/etc/my.cnf --> /etc/mysql/my.cnf --> /usr/local/mysql/etc/my.cnf --> ~/.my.cnf
```

**语法选项**

```
mysqldump [OPTIONS] database [tables]
mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
mysqldump [OPTIONS] --all-databases [OPTIONS]
```

1. 连接选项

   ```
   -u, --user=name        指定用户名
   -S, --socket=name      指定套接字路径
   -p, --password[=name]  指定密码
   -P, --port=#           指定端口
   -h, --host=name        指定主机名
   -r, --result-file=name  将导出结果保存到指定的文件中，在Linux中等同于覆盖重定向。在windows中因为回车符\r\n的原因，使用该选项比重定向更好
   ```

2. 筛选选项

   ```
   --all-databases, -A  
   指定dump所有数据库。等价于使用--databases选定所有库
   --databases, -B  
   指定需要dump的库。该选项后的所有内容都被当成数据库名；在输出文件中的每个数据库前会加上建库语句和use语句
   --ignore-table=db_name.tbl_name  
   导出时忽略指定数据库中的指定表，同样可用于忽略视图，要忽略多个则多次写该选项
   -d, --no-data       
   不导出表数据，可以用在仅导出表结构的情况。
   --events, -E  
   导出事件调度器
   --routines, -R   
   导出存储过程和函数。但不会导出它们的属性值，若要导出它们的属性，可以导出mysql.proc表然后reload
   --triggers  
   导出触发器，默认已开启
   --tables 
   覆盖--databases选项，导出指定的表。但这样只能导出一个库中的表。格式为--tables database_name tab_list
   --where='where_condition', -w 'where_condition'  
   指定筛选条件并导出表中符合筛选的数据，如--where="user='jim'"
   ```

3.  DDL选项

   ```
   --add-drop-database 
   在输出中的create database语句前加上drop database语句先删除数据库
   --add-drop-table    
   在输出的create table语句前加上drop table语句先删除表，默认是已开启的
   --add-drop-trigger  
   在输出中的create trigger语句前加上drop trigger语句先删除触发器
   -n, --no-create-db   
   指定了--databases或者--all-databases选项时默认会加上数据库创建语句，该选项抑制建库语句的输出
   -t, --no-create-info 
   不在输出中包含建表语句
   --replace            
   使用replace代替insert语句
   ```

4. 字符集选项

   ```
   --default-character-set=charset_name 
   在导出数据的过程中，指定导出的字符集。很重要，客户端服务端字符集不同导出时可能乱码，默认使用utf8
   --set-charset 
   在导出结果中加上set names charset_name语句。默认启用。
   ```

5. 复制选项

   ```
   --apply-slave-statements
   --delete-master-logs
   --dump-slave[=value]
   --include-master-host-port
   --master-data[=value]  
   该选项主要用来建立一个replication，当值为1时，导出文件中记录change master语句；
   当值为2时，change master语句被写成注释语句，默认值为空。
   该选项自动忽略--lock-tables，当没有使用--single-transaction时自动启用--lock-all-tables。
   
   ```

6. 格式化选项

   ```
   --compact                   
   简化输出导出的内容，几乎所有注释都不会输出
   
   --complete-insert, -c       
   在insert语句中加上插入的列信息
   
   --create-options            
   在导出的建表语句中，加上所有的建表选项
   
   --tab=dir_name, -T dir_name 
   将每个表的结构定义和数据分别导出到指定目录下文件名同表名的.sql和txt文件中，其中.txt
   文件中的字段分隔符是制表符。要求mysqldump必须和MySQL Server在同一主机，且mysql用
   户对指定的目录有写权限，并且连接数据库的用户必须有file权限。且指定要dump的表，不能和
   --databases或--all-databases一起使用。它的实质是执行select into outfile。
   
   --fields-terminated-by=name            
   指定输出文件中的字段分隔符
   
   --fields-enclosed-by=name              
   指定输出文件中的字段值的包围符，如使用引号将字符串包围起来引用
   
   --fields-optionally-enclosed-by=name   
   指定输出文件中可选字段引用符
   
   --fields-escaped-by=name               
   指定输出文件中的转义符
   
   --lines-terminated-by=name            
   指定输出文件中的换行符   
   
   -Q, --quote-names                       
   引用表名和列名时使用的标识符，默认使用反引号"`" 
   ```

7. 性能选项

   ```
   --delayed-insert  
   对于非事务表，在insert时支持delayed功能，但在MySQL5.6.6开始该选项已经废弃
   --disable-keys, -K  
   在insert语句前后加上禁用和启用索引语句，大量数据插入时该选项很适合。默认开启
   --insert-ignore  
   使用insert ignore语句替代insert语句
   --quick, -q 
   快速导出数据，该选项对于导出大表非常好用。默认导出数据时会一次性检索表中所有数据并加入
   到内存中，而该选项是每次检索一行并导出一行
   ```

8. 加锁和事务相关选项

   ```
   --add-locks         
   在insert语句前后加上lock tables和unlock tables语句，默认已开启。
   
   --flush-logs, -F 
   在开始dump前先flush logs，如果同时使用了--all-databases则依次在每个数据库dump前flush，
   如果同时使用了--lock-all-tables,--master-data或者--single-transaction，则仅flush
   一次，等价于使用flush tables with read lock锁定所有表，这样可以让dump和flush在完全精
   确的同一时刻执行。
   
   --flush-privileges   
   在dump完所有数据库后在数据文件的结尾加上flush privileges语句，在导出的数据涉及mysql库或
   者依赖于mysql库时都应该使用该选项
   
   --lock-all-tables, -x  
   为所有表加上一个持续到dump结束的全局读锁。该选项在dump阶段仅加一次锁，一锁锁永久且锁所有。
   该选项自动禁用--lock-tables和--single-transaction选项
   
   --lock-tables, -l  
   在dump每个数据库前依次对该数据库中所有表加read local锁(多次加锁，lock tables...read local)，
   这样就允许对myisam表进行并发插入。对于innodb存储引擎，使用--single-transaction比
   --lock-tables更好，因为它不完全锁定表。因为该选项是分别对数据库加锁的，所以只能保证每个数
   据库的一致性而不能保证所有数据库之间的一致性。该选项主要用于myisam表，如果既有myisam又有
   innodb，则只能使用--lock-tables，或者分开dump更好
   
   --single-transaction 
   该选项在dump前将设置事务隔离级别为repeatable read并发送一个start transaction语句给
   服务端。该选项对于导出事务表如innodb表很有用，因为它在发出start transaction后能保证导
   出的数据库的一致性时而不阻塞任何的程序。该选项只能保证innodb表的一致性，无法保证myisam表
   的一致性。在使用该选项的时候，一定要保证没有任何其他连接在使用ALTER TABLE,CREATE TABLE,
   DROP TABLE,RENAME TABLE,TRUNCATE TABLE语句，因为一致性读无法隔离这些语句。
   --single-transaction选项和--lock-tables选项互斥，因为lock tables会隐式提交事务。
   要导出大的innodb表，该选项结合--quick选项更好
   
   --no-autocommit  
   在insert语句前后加上SET autocommit = 0，并在需要提交的地方加上COMMIT语句
   
   --order-by-primary  
   如果表中存在主键或者唯一索引，则排序后按序导出。对于myisam表迁移到innobd表时比较有用，但是
   这样会让事务变得很长很慢
   ```

**mysqldump导出示例**

1. 简单备份示例

   创建示例数据库和表。

   ```SQL
   # 创建第一个数据库
   CREATE DATABASE backuptest;
   USE backuptest;
   # 创建innodb表
   CREATE TABLE `student` (
       `studentid` INT (11) NOT NULL,
       `sname` CHAR (30) NOT NULL,
       `gender` enum ('male', 'female') DEFAULT NULL,
       `birth` date DEFAULT NULL,
       PRIMARY KEY (`studentid`)
   ) ENGINE = INNODB DEFAULT CHARSET = latin1;
   
   INSERT INTO student
   VALUES
       (1,'malongshuai','male',curdate()),
       (2,'gaoxiaofang','female',date_add(curdate(), INTERVAL - 2 YEAR)),
       (3,'longshuai','male',date_add(curdate(), INTERVAL - 5 YEAR)),
       (4,'meishaonv','female',date_add(curdate(), INTERVAL - 3 YEAR)),
       (5,'tun\'er','female',date_add(curdate(), INTERVAL - 4 YEAR));
   
   # 创建myisam表，并且字符集设置为UTF8
   CREATE TABLE teacher (
       tid INT NOT NULL PRIMARY KEY,
       tname VARCHAR (30),
       gender enum ('male', 'female'),
       classname CHAR (10)
   ) ENGINE = myisam DEFAULT charset = utf8;
   
   INSERT INTO teacher
   VALUES
       (1,'wugui','male','计算机网络'),
       (2,'woniu','female','C语言'),
       (3,'xiaowowo','female','oracle');
   
   # 创建第二个数据库
   CREATE DATABASE backuptest1;
   USE backuptest1;
   create table student1 as select * from backuptest.student;
   create table teacher1 charset=utf8 engine=myisam as select * from backuptest.teacher;
   ```

   备份单个库，此处备份mysql库。重定向符号可以在Linux中等价于mysqldump的-r选项，所以下面的语句是等价的。

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock mysql >/tmp/mysql1.bak
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock -r /tmp/mysql2.bak mysql
   ```

   查看备份文件，会发现**dump单个库的时候不会在输出文件中记录建库语句和use语句**。

   备份多个库:

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock --databases backuptest backuptest1 >/tmp/mutil_db.bak
   ```

   备份所有库:

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock --all-databases >/tmp/all_db.bak
   ```

   备份多个库或所有库时，会在dump文件中加入建库语句和use语句。实际上，只要使用--databases选项，即使只备份一个库也会加上建库语句和use语句。

   **所以使用mysqldump备份的时候，无论何时都建议--databases或者--all-databases选项二选一**，这样就免去了连进数据库的过程。

2. 使用DDL选项备份示例

   DDL选项如下：

   ```
   --add-drop-database
   --add-drop-table   
   --add-drop-trigger 
   -n, --no-create-db  
   -t, --no-create-info
   --replace     
   ```

   --no-create-db将抑制建库语句，所以不建议使用。

   --no-create-info将抑制建表语句。使用和不使用的对比如下：

   ```shell
   $ mysqldump -uroot -p123456 -S /var/run/mysqld/mysqld.sock --databases backuptest >/tmp/backuptest.bak
   $ mysqldump -uroot -p123456 -S /var/run/mysqld/mysqld.sock --no-create-info --databases backuptest >/tmp/backuptest1.bak
   $ vimdiff /tmp/backuptest.bak /tmp/backuptest1.bak
   ```

   --replace将会把insert语句替换为replace语句。

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock --replace --databases backuptest >/tmp/backuptest2.bak
   $ grep -i 'replace' /tmp/backuptest2.bak
   
   REPLACE INTO `student` VALUES (1,'malongshuai','male','2017-03-31'),(2,'gaoxiaofang','female','2015-03-31'),(3,'longshuai','male','2012-03-31'),(4,'meishaonv','female','2014-03-31'),(5,'tun\'er','female','2013-03-31');
   REPLACE INTO `teacher` VALUES (1,'wugui','male','计算机网络'),(2,'woniu','female','C语言'),(3,'xiaowowo','female','oracle');
   ```

3. 使用字符集选项示例

   dump数据的时候，客户端和数据库的字符集不一致的话会进行字符集转换，转换的过程是不可逆的，所以有可能会导致乱码。

   例如，插入一个带有中文字符的记录到字符集为latin1的表student中。

   ```
   insert INTO backuptest.student VALUES (6,'马','male','2017-03-31');
   ```

   如果提示无法插入，则设置客户端字符集和连接字符集为latin1，character_set_client、character_set_connection、character_set_results，使用set names latin1即可，它会设置它们3个。

   插入成功之后，其他会话连接数据库查询将会是乱码的。dump的时候也是乱码的，因为dump默认会使用utf8字符集，在latin1转码为utf8的过程中出现了乱码。

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock --databases backuptest >/tmp/backuptest.bak
   $ grep -i 'insert' /tmp/backuptest.bak
   INSERT INTO `student` VALUES (1,'malongshuai','male','2017-03-31'),(2,'gaoxiaofang','female','2015-03-31'),(3,'longshuai','male','2012-03-31'),(4,'meishaonv','female','2014-03-31'),(5,'tun\'er','female','2013-03-31'),(6,'é©¬','male','2017-03-31');
   INSERT INTO `teacher` VALUES (1,'wugui','male','计算机网络'),(2,'woniu','female','C语言'),(3,'xiaowowo','female','oracle');
   ```

   再使用乱码的文件来恢复的话，肯定是乱码的结果。

   这时可以指定dump时的字符集为latin1来使得dump数据时无需转换字符集

   ```shell
   $ mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock --default-character-set=latin1 --databases backuptest >/tmp/backuptest.bak
   $ grep -i 'insert' /tmp/backuptest.bak
   INSERT INTO `student` VALUES (1,'malongshuai','male','2017-03-31'),(2,'gaoxiaofang','female','2015-03-31'),(3,'longshuai','male','2012-03-31'),(4,'meishaonv','female','2014-03-31'),(5,'tun\'er','female','2013-03-31'),(6,'马','male','2017-03-31');
   ```

   测试完成之后，将新插入的含有中文字符的记录删除。

   ```
   delete from backuptest.student where studentid=6;
   ```

#### mysqldump使用建议

1. 从性能考虑：在需要导出大量数据的时候，使用--quick选项可以加速导出，但导入速度不变。如果是innodb表，则可以同时加上--no-autocommit选项，这样大量数据量导入时将极大提升性能。

2. 一致性考虑：对于innodb表，几乎没有理由不用--single-transaction选项。对于myisam表，使用--lock-all-tables选项要好于--lock-tables。既有innodb又有myisam表时，可以分开导出，又能保证一致性，还能保证效率。

3. 方便管理和维护性考虑：在导出时flush log很有必要。加上--flush-logs选项即可。而且一般要配合--lock-all-tables选项或者--single-transaction选项一起使用，因为同时使用时，只需刷新一次日志即可，并且也能保证一致性。同时，还可以配合--master-data=2，这样就可以方便地知道二进制日志中备份结束点的位置。

4. 字符集考虑：如果有表涉及到了中文数据，在dump时，一定要将dump的字符集设置的和该表的字符集一样。

5. 杂项考虑：备份过程中会产生二进制日志，但是这是没有必要的。所以在备份前可以关掉，备份完后开启。set sql_log_bin=0关闭，set sql_log_bin=1开启。

以下是备份不同存储引擎类型表的示例：

备份myisam表：需要时加上--default-character-set

```
set sql_log_bin=0

mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock -q --lock-all-tables --flush-logs --master-data=2 --tables backuptest teacher >/tmp/myisam.sql ;

set sql_log_bin=1
```

备份innodb表：需要时加上--default-character-set

```
set sql_log_bin=0

mysqldump -uroot -proot -S /var/run/mysqld/mysqld.sock -q --no-autocommit --flush-logs --single-transaction --master-data=2 --tables backuptest student >/tmp/innodb.sql;

set sql_log_bin=1
```

#### mysqldump + 二进制日志备份

**mysqldump可以实现全备份，在mysqldump之后再二进制日志备份就相当于增量备份，这样就可以实现全备份之后的定时点还原。**

假设要备份的是一张innodb表。使用下面的语句：

```
mysqldump  -uroot -proot -S /var/run/mysqld/mysqld.sock -q --no-autocommit --flush-logs --single-transaction --master-data=2 --tables backuptest student >/tmp/innodb.sql;
```

因为dump前会flush二进制日志，所以之后对该表的操作会记录到新的滚动日志中。然后只需备份新的二进制日志即可。

然后在该表中插入一行记录。

```
insert into student select 10,'xiaolonglong','male','2015-01-02';
```

备份新的二进制日志。

```
mysqlbinlog mysql-bin.000002 >/tmp/new_binlog.sql
```

设计刚才备份的表误操作，如删除该表。

```
drop table student;
```

使用该表的完全备份和二进制日志恢复。因为备份时使用的是--tables选项，所以要恢复需要进入数据库指定数据库，然后使用source来加载sql文件。

```
use backuptest;
source /tmp/innodb.sql;
source /tmp/new_binlog.sql;
```

#### mysqldump工具的评价

mysqldump备份的文件是逻辑SQL语句，总体来说，简单，便捷，在有些时候迁移数据的时候比较有用。另外，它的功能也很多，例如导出数据，导出表结构等。

但是缺点是恢复速度太慢，因为恢复数据时是通过insert不断插入记录的，它的恢复速度远不及load data infile导入数据。

在备份方式上，mysqldump备份myisam表时因为要加--lock-all-tables，这时要备份的数据库全部被上锁，可读不可写，所以实现的是温备。mysqldump备份innodb表时因为要加--single-transaction，会自动将隔离级别设置为repeatable read并开启一个事务，这时mysqldump将获取dump执行前一刻的行版本，并处于一个长事务中直到dump结束。所以不影响目标数据库的使用，可读也可写，即实现的是热备。

### 导入、导出表数据

*load data infile*和*select into outfile*语句是配套的。*select into outfile*语句是将检索出来的数据按格式导出到文件中，数据迁移跨数据库系统时，该选项很有用，因为它可以指定分隔符。*load data infile*是将带有格式的数据文件导入到表中。

导出、导入数据时需要指定格式(如不指定，则使用默认)。格式涉及几个方面：字段分隔符、行分隔符、引用符号、转义符号。

还需注意一点，默认情况下(MySQL 5.6.34之后)这两个语句无法执行成功，因为全局变量secure_file_priv的默认值为null，它表示禁用这两种语句的导入导出。

为空会报这样的错误

```
(1290, 'The MySQL server is running with the --secure-file-priv option so it cannot execute this statement')
```

![UTOOLS1571838933846.png](https://i.loli.net/2019/10/23/7yutBMan6WEYOTD.png)

所以应该将其设置为空(不指定任何值)或者指定一个目录，将来该目录中的所有文件都可以进行mysql file类的交互。当然，变量指定的目录必须已经存在，且mysql系统用户和组必须对该目录有读写权限。

```
mkdir /data
chown -R mysql.mysql /data
```

这个变量是全局静态变量，只能在mysqld实例未启动的时候才能修改。所以将其写入配置文件。

```
[mysqld]
secure-file-priv=/data
# 或者
# secure-file-priv=
```

查看变量。

```
select @@global.secure_file_priv;
+---------------------------+
| @@global.secure_file_priv |
+---------------------------+
| /data/                    |
+---------------------------+
```

再看这两个语句的语法：

```SQL
SELECT ... INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        [export_options]
 
LOAD DATA [LOW_PRIORITY | CONCURRENT] [LOCAL] INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [CHARACTER SET charset_name]
        [export_options]
    [IGNORE number {LINES|ROWS}]
    [(col_name_or_user_var,...)]
    [SET col_name = expr,...]
 
 
export_options:
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
```

其中'char'表示只能使用一个字符，'string'表示可以指定多个字符。

*fields terminated by 'string'*指定字段分隔符；*enclosed by 'char*'指定所有字段都使用char符号包围，如果指定了*optionally*则只用在字符串和日期数据类型等字段上，默认未指定；*escaped by 'char'*指定转义符。

*lines starting by 'string'*指定行开始符，如每行开始记录前空一个制表符；*lines terminated by 'string'*为行分隔符。

要注意，在几种情况下需要使用转义符：数据中含有转义符本身或者字段分隔符。当指定了字段引用符*enclosed by*时，如果数据中含有字段引用符，则也需要转义，若未指定*enclosed by*，则默认不使用字段引用符，所以无需转义。

以下为它们的默认值：

```
fileds terminated by '\t' enclosed by '' escaped by '\\'
lines terminated by '\n' starting by ''
```

看上去语法还挺复杂的，使用示例来说明就很清晰易懂了。

给定如下表结构和数据。

```mysql
create database test;
use test;
create  table t(id int primary key,sex char(3),name char(20),ins_day date);

insert into t values(1,'nan','longshuai1','2010-04-19'),
                    (2,'nan','longshuai2','2011-04-19'),
                    (3,'nv','xiaofang1','2012-04-19'),
                    (4,'nv','xiaofang2','2013-04-19'),
                    (5,'nv','xiaofang3','2014-04-19'),
                    (6,'nv','xiaofang4','2015-04-19'),
                    (7,'nv','tun\'er','2016-04-19'),
                    (8,'nan','longshuai3','2017-04-19');
```

#### select into outfile导出数据

使用默认设置：

```mysql
select * from t into outfile '/data/t_data.sql';

$ cat /data/t_data.sql
1       nan     longshuai1      2010-04-19
2       nan     longshuai2      2011-04-19
3       nv      xiaofang1       2012-04-19
4       nv      xiaofang2       2013-04-19
5       nv      xiaofang3       2014-04-19
6       nv      xiaofang4       2015-04-19
7       nv      tun'er  2016-04-19
8       nan     longshuai3      2017-04-19
```

指定字段分隔符","，使用单引号包围各字段，每行前加上制表符。

```mysql
select * from t into outfile '/data/t_data1.sql' fields terminated by ',' enclosed by '\'' lines starting by '\t' terminated by '\n';

$ cat /data/t_data1.sql
        '1','nan','longshuai1','2010-04-19'
        '2','nan','longshuai2','2011-04-19'
        '3','nv','xiaofang1','2012-04-19'
        '4','nv','xiaofang2','2013-04-19'
        '5','nv','xiaofang3','2014-04-19'
        '6','nv','xiaofang4','2015-04-19'
        '7','nv','tun\'er','2016-04-19'
        '8','nan','longshuai3','2017-04-19'
```

#### load data infile导入数据

要导入格式化后的纯数据，可以使用*load data infile*，加载纯数据的插入方式比直接执行insert插入至少快20多倍。但在内部，它们其实是等价行为，load data infile也会触发insert相关触发器。

其中可以使用local关键字表示从客户端主机读取文件，如果没有指定local则表示从服务端主机读取文件。

fields和lines的相关选项和*select ... into outfile*是一样的，只不过*load data infile*多了几个选项。其中*ignore N lines|rows*表示忽略前N行数据不导入，*col_name_or_user_var*表示按此处给定的字段和顺序来导入数据，*set col_name=expr*表示对列进行一些表达式运算，如给某数值字段加5，给某字符串列尾部加上@qq.com字符等。

例如要加载如下文件到test.t表中。

```shell
cat /data/t_data.txt
1       nan     longshuai1      2010-04-19
2       nan     longshuai2      2011-04-19
3       nv      xiaofang1       2012-04-19
4       nv      xiaofang2       2013-04-19
5       nv      xiaofang3       2014-04-19
6       nv      xiaofang4       2015-04-19
7       nv      tun'er  2016-04-19
8       nan     longshuai3      2017-04-19
```

首先删除表中数据，再导入。

```mysql
truncate test.t;
load data infile '/data/t_data.sql' into table test.t fields terminated by '\t';
```

将如下包含字段分隔符","，字段引用符"'"，转义符"\"，行前缀"\t"的文件加载到test.t表中。

```shell
$ cat /data/t_data1.sql
        '1','nan','longshuai1','2010-04-19'
        '2','nan','longshuai2','2011-04-19'
        '3','nv','xiaofang1','2012-04-19'
        '4','nv','xiaofang2','2013-04-19'
        '5','nv','xiaofang3','2014-04-19'
        '6','nv','xiaofang4','2015-04-19'
        '7','nv','tun\'er','2016-04-19'
        '8','nan','longshuai3','2017-04-19'
```

首先删除表中数据，然后加载。

```mysql
truncate test.t;
load data infile '/data/t_data1.sql' into table test.t fields terminated by ',' enclosed by '\'' escaped by '\\' lines starting by '\t' terminated by '\n';
```

若要忽略前两行，则：

```mysql
truncate test.t;
load data infile '/data/t_data1.sql' into table test.t fields terminated by ',' enclosed by '\'' escaped by '\\' lines starting by '\t' terminated by '\n' ignore 2 rows;
```

如果想在id列值加上5，则：

```mysql
truncate test.t;
load data infile '/data/t_data1.sql' into table test.t fields terminated by ',' enclosed by '\'' escaped by '\\' lines starting by '\t' terminated by '\n' set id=id+5;
```

如果想name列后加上"@qq.com"字符串，则：

```mysql
truncate test.t;
load data infile '/data/t_data1.sql' into table test.t fields terminated by ',' enclosed by '\'' escaped by '\\' lines starting by '\t' terminated by '\n' set name=concat(name,'@qq.com');
```

如果想同时执行上面两个set，则：

```mysql
truncate test.t;
load data infile '/data/t_data1.sql' into table test.t fields terminated by ',' enclosed by '\'' escaped by '\\' lines starting by '\t' terminated by '\n' set name=concat(name,'@qq.com'), id=id+5;
```

#### mysqldump导出数据

和*select into outfile*功能类似的语句还有：此方法导出的数据中还包含了列名。

```mysql
mysql -uroot -p123456 -e "select * from test.t">/tmp/t_data2.sql

cat /tmp/t_data2.sql
id      sex     name    ins_day
1       nan     longshuai1      2010-04-19
2       nan     longshuai2      2011-04-19
3       nv      xiaofang1       2012-04-19
4       nv      xiaofang2       2013-04-19
5       nv      xiaofang3       2014-04-19
6       nv      xiaofang4       2015-04-19
7       nv      tun'er  2016-04-19
8       nan     longshuai3      2017-04-19
```

虽说*select ... into outfile*导出数据后可修改性和加载性非常强，但是毕竟没有导出结构。要导出结构，可以使用mysqldump的"--tab"选项，它既会导出表的结构定义语句到同表名的.sql文件中，还会导出数据到同表名的.txt文件中。

```mysql
mysqldump -uroot -p123456 --tab /data test t;

ls -l /data/t.*
-rw-r--r-- 1 root  root  1408 Apr 19 14:46 /data/t.sql   # test.t表定义语句
-rw-rw-rw- 1 mysql mysql  211 Apr 19 14:46 /data/t.txt   # test.t表内数据
```

mysqldump的"--tab"选项同样可以指定各种分隔符。如"*--fields-terminated-by=...,--fields-enclosed-by=...,--fields-optionally-enclosed-by=...,--fields-escaped-by=...*"。以下是指定字段分隔符为","。

```mysql
mysqldump -uroot -p123456 --tab /data --fields-terminated-by=',' test t;

cat /data/t.txt
1,nan,longshuai1,2010-04-19
2,nan,longshuai2,2011-04-19
3,nv,xiaofang1,2012-04-19
4,nv,xiaofang2,2013-04-19
5,nv,xiaofang3,2014-04-19
6,nv,xiaofang4,2015-04-19
7,nv,tun'er,2016-04-19
8,nan,longshuai3,2017-04-19
```

#### mysqlimport导入数据

mysqlimport和*load data infile*的本质是一样的。mysqlimport在执行时会像服务端发送*load data infile*来加载数据，并且mysqlimport支持多进程并行导入多张表的数据。

mysqlimport的语法和*load data infile*基本一致。不同的是它在MySQL/MariaDB的外部执行，且可以一次性并行多线程导入多张表(并非并行导入一张表)，所以能更快地导入所有数据。

```
mysqlimport [OPTIONS] database textfile...
```

注意：mysqlimport只能指定数据库名来导入，所以导入的文件名必须和数据库中的表名相对应(文件名后缀无所谓)。例如文件名为stu2.sql，而表名为student则无法导入，它会找stu2这个表。

例如，将以下格式的文件t.txt使用mysqlimport导入到test.t表中：

```shell
$ cat /data/t.txt
1,nan,longshuai1,2010-04-19
2,nan,longshuai2,2011-04-19
3,nv,xiaofang1,2012-04-19
4,nv,xiaofang2,2013-04-19
5,nv,xiaofang3,2014-04-19
6,nv,xiaofang4,2015-04-19
7,nv,tun'er,2016-04-19
8,nan,longshuai3,2017-04-19

$ mysqlimport -uroot -p123456 --fields-terminated-by=',' test '/data/t.txt'
```

使用"--use-threads"选项可以指定导入线程数。

例如，下面指定两个线程，导入两张表到数据库test库中的t1和t2表中。

```mysql
mysqlimport -uroot -p123456 --use-threads=2 --fields-terminated-by=',' test '/data/t1.txt' '/data/t2.txt'
```

### 参考

1. [备份和恢复(一)：mysqldump工具用法详述](http://www.cnblogs.com/f-ck-need-u/p/9013458.html)
2. [备份和恢复(二)：导入、导出表数据](http://www.cnblogs.com/f-ck-need-u/p/9013643.html)
3. [备份和恢复(三)：xtrabackup用法和原理详述](http://www.cnblogs.com/f-ck-need-u/p/9018716.html)
4. [Mysql DBA 高级运维学习之路-MySQL备份与恢复实战案例及生产方案](https://blog.51cto.com/10642812/2071532)
5. [原创｜高逼格企业级MySQL数据库备份方案，原来是这样....](https://www.cnblogs.com/youkanyouxiao/p/10893765.html)