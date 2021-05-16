---
title: MySQL 复制--基于gtid（二）
date: 2019-10-16 18:18:59
tags:
 - MySQL
 - 数据库
categories:
 - 数据库
 - MySQL
---

相比传统的MySQL复制，gtid复制无论是配置还是维护都要轻松的多。

MySQL基于GTID复制官方手册：https://dev.mysql.com/doc/refman/5.7/en/replication-gtids.html

<!--more-->

#### 概念

传统的基于binlog position复制的方式有个严重的缺点：如果slave连接master时指定的binlog文件错误或者position错误，会造成遗漏或者重复，很多时候前后数据是有依赖性的，这样就会出错而导致数据不一致。

**什么是GTID**

1. 全局唯一，一个事务对应一个GTID
2. 替代传统的binlog+pos复制；使用master_auto_position=1自动匹配GTID断点进行复制
3. MySQL5.6开始支持
4. 在传统的主从复制中，slave端不用开启binlog；但是在GTID主从复制中，必须开启binlog
5. slave端在接受master的binlog时，会校验GTID值
6. 为了保证主从数据的一致性，多线程同时执行一个GTID

**组成**

```
Master_UUID:序列号
举例：ceb0ca3d-8366-11e8-ad2b-000c298b7c9a:1-5

ceb0ca3d-8366-11e8-ad2b-000c298b7c9a其实就是master的uuid值；
1-5是序列号，每次一个事务完成都会自增1，也就是说下一次为1-6。
```

从MYSQL5.6开始，mysql开始支持GTID复制。GTID的全称是global transaction id，表示的是全局事务ID。GTID的分配方式为`uuid:trans_id`，其中：

- `uuid`是每个mysql服务器都唯一的，记录在`$datadir/auto.cnf`中。如果复制结构中，任意两台服务器uuid重复的话(比如直接冷备份时，auto.conf中的内容是一致的)，在启动复制功能的时候会报错。这时可以删除auto.conf文件再重启mysqld。

```
show variables like "%uuid%";
+-----------------+--------------------------------------+
| Variable_name   | Value                                |
|-----------------+--------------------------------------|
| server_uuid     | 8980a824-f7cf-11e9-90a2-0242ac110002 |
+-----------------+--------------------------------------+
1 row in set
Time: 0.024s
```

- `trans_id`是事务ID，可以唯一标记某MySQL服务器上执行的某个事务。事务号从1开始，每提交一个事务，事务号加1。

**工作原理**

1. master更新数据时，会在事务前产生GTID，一同记录到binlog日志中。
2. slave端的i/o 线程将变更的binlog，写入到本地的relay log中。
3. sql线程从relay log中获取GTID，然后对比slave端的binlog是否有记录。
4. 如果有记录，说明该GTID的事务已经执行，slave会忽略。
5. 如果没有记录，slave就会从relay log中执行该GTID的事务，并记录到binlog。
6. 在解析过程中会判断是否有主键，如果没有就用二级索引，如果没有就用全部扫描

#### GTID主从配置

使用docker进行模拟

```
docker pull mysql:5.7
docker run --name master -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker run --name slave1 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

环境：

|   主机IP   |  OS版本  |  MySQL版本   | 角色(master/slave) | 数据状态 |
| :--------: | :------: | :----------: | :----------------: | :------: |
| 172.17.0.3 | debian 9 | MySQL 5.7.28 |    master_gtid     | 全新实例 |
| 172.17.0.4 | debian 9 | MySQL 5.7.22 |    slave1_gtid     | 全新实例 |

因为是用作master和slave的mysql实例都是全新环境，所以这里简单配置一下即可。

1. 配置master

   ```shell
   # docker exec -it master bash
   # cat >> /etc/mysql/mysql.conf.d/mysqld.cnf<<-EOF
   
   # mysqld下添加
   server-id=1
   enforce_gtid_consistency=on   # gtid复制需要加上的必须项
   gtid_mode=on                  # gtid复制需要加上的必须项
   server-id=1
   binlog_format=row
   log-bin=mysql-bin
   EOF
   ```

   然后重启服务

   ```shell
   # exit
   # docker restart master
   ```

   

2. master授权配置

   ```shell
   # docker exec -it master mysql -uroot -p
   mysql> CREATE USER 'hyp'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
   mysql> GRANT ALL PRIVILEGES ON *.* TO 'hyp'@'%';
   mysql> flush privileges;
   ```

3. 配置slave

   ```shell
   # docker exec -it slave1 bash
   # cat >> /etc/mysql/mysql.conf.d/mysqld.cnf<<-EOF
   
   # mysqld下添加
   server-id=2
   enforce_gtid_consistency=on   # gtid复制需要加上的必须项
   gtid_mode=on                  # gtid复制需要加上的必须项
   binlog_format=ROW
   log-bin=mysql-bin
   log_slave_updates=ON
   skip-slave-start=1
   EOF
   # exit
   # docker restart slave1
   ```

4. slave配置同步

   ```shell
   # docker exec -it slave1 mysql -uroot -p
   mysql> change master to master_host='172.17.0.3',master_user='hyp',master_password='123456',master_port=3306,master_auto_position=1;
   mysql> start slave;
   ```

5. 查看slave的状态

   ```
   mysql> show slave status\G;
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 172.17.0.3
                     Master_User: hyp
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: mysql-bin.000001
             Read_Master_Log_Pos: 155
                  Relay_Log_File: 50cc3e1f9801-relay-bin.000002
                   Relay_Log_Pos: 369
           Relay_Master_Log_File: mysql-bin.000001
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 154
                 Relay_Log_Space: 581
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: e3dbcdbb-006d-11ea-ba7c-0242ac110003
                Master_Info_File: /var/lib/mysql/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: 
                   Auto_Position: 1
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
          Master_public_key_path: 
           Get_master_public_key: 0
               Network_Namespace: 
   1 row in set (0.00 sec)
   
   ERROR: 
   No query specified
   ```
   
   通过slave的状态信息，可以看到GTID的值、Matser_UUID等信息
   
   查看master状态，注意对比slave端，Executed_Gtid_Set的值应该是一样的。
   
   ```
   mysql> show master status;
   +------------------+----------+--------------+------------------+-------------------+
   | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +------------------+----------+--------------+------------------+-------------------+
   | mysql-bin.000001 |      154 |              |                  |                   |
   +------------------+----------+--------------+------------------+-------------------+
   1 row in set (0.00 sec)
   ```
   
6. 验证主从

   master上

   ```bash
   mysql> create database test01;
   Query OK, 1 row affected (0.00 sec)
   
   mysql> show master status \G
   *************************** 1. row ***************************
                File: mysql-bin.000001
            Position: 319
        Binlog_Do_DB: 
    Binlog_Ignore_DB: 
   Executed_Gtid_Set: e3dbcdbb-006d-11ea-ba7c-0242ac110003:1
   1 row in set (0.00 sec)
   ```

   slave上

   ```bash
   mysql> show databases;
   +--------------------+
   | Database           |
   +--------------------+
   | information_schema |
   | mysql              |
   | performance_schema |
   | sys                |
   | test01             |
   +--------------------+
   5 rows in set (0.07 sec)
   
   mysql> show slave status \G
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 172.17.0.3
                     Master_User: hyp
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: mysql-bin.000001
             Read_Master_Log_Pos: 319
                  Relay_Log_File: 50cc3e1f9801-relay-bin.000002
                   Relay_Log_Pos: 532
           Relay_Master_Log_File: mysql-bin.000001
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 319
                 Relay_Log_Space: 746
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: e3dbcdbb-006d-11ea-ba7c-0242ac110003
                Master_Info_File: /var/lib/mysql/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: e3dbcdbb-006d-11ea-ba7c-0242ac110003:1
               Executed_Gtid_Set: e3dbcdbb-006d-11ea-ba7c-0242ac110003:1
                   Auto_Position: 1
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)
   ```
   
   需要注意的是，GTID的值在完成一次事务后，变成了e3dbcdbb-006d-11ea-ba7c-0242ac110003:1（自增1）

遇到错误：<https://blog.csdn.net/fzdba/article/details/21697671>

**排障**

思路

a、确保master开放3306端口

b、最好关闭selinux

c、master上授权同步，slave上change master命令指定master的信息不要写错

d、UUID问题

![KRvddf.png](https://s2.ax1x.com/2019/10/29/KRvddf.png)

参考：https://blog.csdn.net/sunbocong/article/details/81634296

e、总结

排障过程中，注意需要停掉slave，做完修改之后在开启，否则你的修改可能是不会生效的。

#### 添加新的slave到gtid复制结构中

GTID复制是基于事务ID的，确切地说是binlog中的GTID，所以事务ID对GTID复制来说是命脉。

当master没有删除过任何binlog时，可以随意地向复制结构中添加新的slave，因为slave会复制所有的binlog到自己relay log中并replay。这样的操作尽管可能速度不佳，但胜在操作极其简便。

当master删除过一部分binlog后，在向复制结构中添加新的slave时，必须先获取到master binlog中当前已记录的第一个gtid之前的所有数据，然后恢复到slave上。只有slave上具有了这部分基准数据，才能保证和master的数据一致性。

而在实际环境中，往往会定期删除一部分binlog。所以，为了配置更通用的gtid复制环境，这里把前文的master的binlog给purge掉一部分。

master填充数据，`/root/backup.sql`文件：

```mysql

DROP DATABASE IF EXISTS backuptest;
CREATE DATABASE backuptest;
USE backuptest;

-- 创建myisam类型的数值辅助表和对应插入数据的存储过程
CREATE TABLE num_isam(n INT NOT NULL PRIMARY KEY)ENGINE=MYISAM;
DELIMITER $$
DROP PROCEDURE IF EXISTS proc_num1$$
CREATE PROCEDURE proc_num1(num INT) 
BEGIN
    DECLARE rn INT DEFAULT 1;
    TRUNCATE TABLE backuptest.num_isam;
    INSERT INTO backuptest.num_isam VALUES(1);
    dd: WHILE rn*2 < num DO
        BEGIN
            INSERT INTO backuptest.num_isam SELECT rn+n FROM backuptest.num_isam;
            SET rn = rn*2;
        END;
    END WHILE dd;
    INSERT INTO backuptest.num_isam SELECT n+rn FROM num_isam WHERE n+rn <=num;
END;$$
DELIMITER ;

-- 创建innodb类型的数值辅助表和对应插入数据的存储过程
CREATE TABLE num_innodb(n INT NOT NULL PRIMARY KEY)ENGINE=INNODB;
DELIMITER $$
DROP PROCEDURE IF EXISTS proc_num2$$
CREATE PROCEDURE proc_num2(num INT) 
BEGIN
    DECLARE rn INT DEFAULT 1;
    TRUNCATE TABLE backuptest.num_innodb;
    INSERT INTO backuptest.num_innodb VALUES(1);
    dd: WHILE rn*2 < num DO
        BEGIN
            INSERT INTO backuptest.num_innodb SELECT rn+n FROM backuptest.num_innodb;
            SET rn = rn*2;
        END;
    END WHILE dd;
    INSERT INTO backuptest.num_innodb SELECT n+rn FROM backuptest.num_innodb WHERE n+rn <=num;
END;$$
DELIMITER ;

-- 分别向两个数值辅助表中插入1亿条数据，
CALL proc_num1(100000000);
CALL proc_num2(100000000);
```

进入mysql

```
mysql> source /root/backup.sql

```

目前master上的binlog使用情况如下，不难发现绝大多数操作都集中在`mysql-bin.000001`这个binlog中。

```
# ls  /var/lib/mysql/mysql-bin.*
/var/lib/mysql/mysql-bin.000001  /var/lib/mysql/mysql-bin.index

```

purge已有的binlog。

```
mysql> flush logs;
mysql> purge master logs to 'mysql-bin.000002';

# cat /var/lib/mysql/mysql-bin.index 
./mysql-bin.000002
```

但无论master是否purge过binlog，配置基于gtid的复制都极其方便，而且方法众多(只要理解了GTID的生命周期，可以随意折腾，基本上都能很轻松地维护好)，这是它"迷人"的优点。

现在的测试环境是这样的：

|   主机IP   |  OS版本  |  MySQL版本   | 角色(master/slave) |    数据状态     |
| :--------: | :------: | :----------: | :----------------: | :-------------: |
| 172.17.0.3 | debian 9 | MySQL 5.7.28 |    master_gtid     | 已purge过binlog |
| 172.17.0.4 | debian 9 | MySQL 5.7.28 |    slave1_gtid     |     已同步      |
| 172.17.0.5 | debian 9 | MySQL 5.7.28 |    slave2_gtid     |    全新实例     |

```
docker run --name slave2 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

其中slave2的配置文件和slave1的配置文件完全相同：

```
# docker exec -it slave2 bash
# cat >> /etc/mysql/mysql.conf.d/mysqld.cnf<<-EOF

# mysqld下添加
server-id=3
enforce_gtid_consistency=on   # gtid复制需要加上的必须项
gtid_mode=on                  # gtid复制需要加上的必须项
binlog_format=ROW
log-bin=mysql-bin
log_slave_updates=ON
skip-slave-start=1
EOF
# exit
# docker restart slave2
```

1. 备份master。

   ```shell
   # master上执行，备份所有数据：
   # docker exec -it master bash
   # mkdir /backdir   # 备份目录
   # wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.4/binary/debian/jessie/x86_64/percona-xtrabackup-24_2.4.4-1.jessie_amd64.deb
   # dpkg -i percona-xtrabackup-24_2.4.4-1.jessie_amd64.deb
   # apt install -f
   # xtrabackup --backup --user=root --password=123456 --datadir=/var/lib/mysql/ --target-dir=/backdir/fullback
   # exit
   
   # docker cp master:/backdir/fullback ./
   # docker cp ./fullback slave2:/tmp/
   ```

2. 将备份恢复到slave2。

   在slave2上执行：

   ```
    # docker exec -it slave2 bash
    # rm -rf /var/lib/mysql/*    # 恢复前必须先清空数据目录
    # xtrabackup --prepare --target-dir=/tmp/fullback/  # 恢复备份数据
    # rsync -azP /tmp/fullback/* /var/lib/mysql/
    # chown -R mysql.mysql /var/lib/mysql/  
    # exit
    # docker restart slave2
   ```

3. 设置gtid_purged，连接master，开启复制功能。

   由于xtrabackup备份数据集却不备份binlog，所以必须先获取此次备份结束时的最后一个事务ID，并在slave上明确指定跳过这些事务，否则slave会再次从master上复制这些binlog并执行，导致数据重复执行。

   可以从slave2数据目录中的`xtrabackup_info`文件中获取。如果不是xtrabackup备份的，那么可以直接从master的`show global variables like "gtid_executed";`中获取，它表示master中已执行过的事务。

   ```
   [root@xuexi ~]# cat /var/lib/mysql/xtrabackup_info 
   uuid = f295f16f-00ae-11ea-a34c-0242ac110002
   name = 
   tool_name = xtrabackup
   tool_command = --backup --user=root --password=... --datadir=/var/lib/mysql/ --target-dir=/backdir/fullback
   tool_version = 2.4.4
   ibbackup_version = 2.4.4
   server_version = 5.7.28-log
   start_time = 2019-11-06 16:02:30
   end_time = 2019-11-06 16:03:18
   lock_time = 0
   binlog_pos = filename 'mysql-bin.000003', position '194', GTID of the last change 'e3dbcdbb-006d-11ea-ba7c-0242ac110003:1-67'
   innodb_from_lsn = 0
   innodb_to_lsn = 5104670069
   partial = N
   incremental = N
   format = file
   compact = N
   compressed = N
   encrypted = N
   ```

   其中`binlog_pos`中的GTID对应的就是已备份的数据对应的事务。换句话说，这里的gtid集合1-67表示这67个事务不需要进行复制。

   或者在master上直接查看executed的值，注意不是gtid_purged的值，master上的gtid_purged表示的是曾经删除掉的binlog。

   ```
   mysql> show global variables like '%gtid%';
   +----------------------------------+-------------------------------------------+
   | Variable_name                    | Value                                     |
   +----------------------------------+-------------------------------------------+
   | binlog_gtid_simple_recovery      | ON                                        |
   | enforce_gtid_consistency         | ON                                        |
   | gtid_executed                    | e3dbcdbb-006d-11ea-ba7c-0242ac110003:1-67 |
   | gtid_executed_compression_period | 1000                                      |
   | gtid_mode                        | ON                                        |
   | gtid_owned                       |                                           |
   | gtid_purged                      | e3dbcdbb-006d-11ea-ba7c-0242ac110003:1-67 |
   | session_track_gtids              | OFF                                       |
   +----------------------------------+-------------------------------------------+
   8 rows in set (0.01 sec)
   
   ```

   可以**在启动slave线程之前使用`gtid_purged`变量来指定需要跳过的gtid集合。**但因为要设置gtid_purged必须保证全局变量gtid_executed为空，所以先在slave上执行`reset master`(注意，不是reset slave)，再设置gtid_purged。

   ```
   # slave2上执行：
   mysql> reset master;
   mysql> set @@global.gtid_purged='e3dbcdbb-006d-11ea-ba7c-0242ac110003:1-67';
   ```

   设置好gtid_purged之后，就可以开启复制线程了。

   ```
   mysql> change master to master_host='172.17.0.3',master_user='hyp',master_password='123456',master_port=3306,master_auto_position=1;
   mysql> start slave;
   ```

   查看slave的状态，看是否正确启动了复制功能。如果没错，再在master上修改一部分数据，检查是否同步到slave1和slave2。

   ```
   mysql> show slave status \G
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 172.17.0.2
                     Master_User: hyp
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: mysql-bin.000003
             Read_Master_Log_Pos: 194
                  Relay_Log_File: 4870a09cff9b-relay-bin.000002
                   Relay_Log_Pos: 367
           Relay_Master_Log_File: mysql-bin.000003
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 194
                 Relay_Log_Space: 581
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: e3dbcdbb-006d-11ea-ba7c-0242ac110003
                Master_Info_File: /var/lib/mysql/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: e3dbcdbb-006d-11ea-ba7c-0242ac110003:1-67
                   Auto_Position: 1
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)
   
   ```

4. 回到master，purge掉已同步的binlog。

   当slave指定gtid_purged并实现了同步之后，为了下次重启mysqld实例不用再次设置gtid_purged(甚至可能会在启动的时候自动开启复制线程)，所以应该去master上将已经同步的binlog给purged掉。

   ```
   # master上执行：
   mysql> flush logs;    # flush之后滚动到新的日志master-bin.000006
   
   # 在确保所有slave都复制完000006之前的所有事务后，purge掉旧日志
   mysql> purge master logs to "mysql-bin.000006";
   ```

### 参考

1. [深度探索MySQL主从复制原理](https://baijiahao.baidu.com/s?id=1617888740370098866&wfr=spider&for=pc)
2. [MySQL复制(一)：复制理论和传统复制的配置](https://www.cnblogs.com/f-ck-need-u/p/9155003.html)
3. [MySQL复制(二)：基于GTID复制](https://www.cnblogs.com/f-ck-need-u/p/9164823.html)
4. [MySQL复制(三)：半同步复制](https://www.cnblogs.com/f-ck-need-u/p/9166452.html)