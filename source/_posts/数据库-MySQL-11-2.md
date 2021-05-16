---
title: MySQL SQL 备份--xtrabackup (九)
date: 2019-05-17 13:58:59
tags:
 - 数据库
 - MySQL
 - SQL
categories:
 - 数据库
 - MySQL
---

### xtrabackup用法和原理

xtrabackup是percona团队研发的备份工具，比MySQL官方的ibbackup的功能还要多。支持myisam温全备、innodb热全备和温增备，还可以实现innodb的定时点恢复，而且备份和恢复的速度都较快。在目前MySQL的备份实现上，考虑价格、速度、安全、一致性等角度，xtrabackup是非常合适的工具。

<!--more-->

MariaDB也可以使用percona xtrabackup进行备份，不过MariaDB基于percona xtrabackup开发了它自己的备份工具：MariaDB Backup。它基于xtrabackup开发，所以所用方法基本和xtrabackup相同，只是有些自己的特性。详细内容见MariaDB Backup官方手册：https://mariadb.com/kb/en/library/mariadb-backup/

xtrabackup官方手册：https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html

#### 安装xtrabackup

**对应版本：**

```
mysql 5.1 – xtrabackup2.0
mysql5.6 --xtrabackup 2.2
mysql5.7 --xtrabackup2.4
mysql 8.0 -- xtrabackup 8.0
```

下载地址：https://www.percona.com/downloads/XtraBackup/LATEST/

rpm仓库(实际上是percona的仓库)：http://repo.percona.com/release/

清华大学percona源：https://mirrors.tuna.tsinghua.edu.cn/percona/

使用MySQL 5.7

因为只是一个备份工具，所以没必要编译安装，直接下载它的rpm包即可。但是该rpm包依赖于libev.so.4，该依赖包可以在epel源中找到。

```shell
#centos
#https://www.percona.com/doc/percona-xtrabackup/2.4/installation/yum_repo.html
$  sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm -y
$  sudo yum -y install percona-xtrabackup-24

#ubuntu
#https://www.percona.com/doc/percona-xtrabackup/2.4/installation/apt_repo.html
$ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
$ sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
$ sudo apt-get update
$ sudo apt-get install percona-xtrabackup-24
```

装完xtrabackup后，生成以下几个工具:

- xbcloud和xbcloud_osenv是xtrabackup新的高级特性：云备份；

- xbcrypt也是新的特性，加密备份集；

- xbstream是xtrabackup的流数据功能，通过流数据功能，可将备份内容打包并传给管道后的压缩工具进行压缩；

- xtrabackup是主程序；

  ```
  常用选项:  
     --host     指定主机
     --user     指定用户名
     --password    指定密码
     --port     指定端口
     --databases     指定数据库
     --incremental    创建增量备份
     --incremental-basedir   指定包含完全备份的目录
     --incremental-dir      指定包含增量备份的目录   
     --apply-log        对备份进行预处理操作             
       一般情况下，在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据文件仍处理不一致状态。“准备”的主要作用正是通过回滚未提交的事务及同步已经提交的事务至数据文件也使得数据文件处于一致性状态。
     --redo-only      不回滚未提交事务
     --copy-back     恢复备份目录
  ```

- innobackupex在以前是一个perl脚本，会调用xtrabackup这个二进制工具，从xtrabackup 2.3开始，该工具使用C语言进行了重写，当前它是xtabackup二进制工具的一个软连接，但是实际的使用方法却不同，并且在以后的版本中会删除该工具。

  使用innobackupex备份时，其会调用xtrabackup备份所有的InnoDB表，复制所有关于表结构定义的相关文件(.frm)、以及MyISAM、MERGE、CSV和ARCHIVE表的相关文件，同时还会备份触发器和数据库配置信息相关的文件，这些文件会被保存到一个以时间命名的目录当中。在备份的同时，innobackupex还会在备份目录中创建如下文件：

  ```
  (1)xtrabackup_checkpoints -- 备份类型(如完全或增量)、备份状态(如是否已经为prepared状态)和LSN(日志序列号)范围信息：
  
  每个InnoDB页(通常为16k大小)
  都会包含一个日志序列号，即LSN，LSN是整个数据库系统的系统版本号，每个页面相关的LSN能够表明此页面最近是如何发生改变的。
  
  (2)xtrabackup_binlog_info  --  mysql服务器当前正在使用的二进制日志文件及备份这一刻位置二进制日志时间的位置。
  
  (3)xtrabackup_binlog_pos_innodb  --  二进制日志文件及用于InnoDB或XtraDB表的二进制日志文件的当前position。
  
  (4)xtrabackup_binary  --  备份中用到的xtrabackup的可执行文件；
  
  (5)backup-my.cnf  --  备份命令用到的配置选项信息：
  
  在使用innobackupex进行备份时，还可以使用--no-timestamp选项来阻止命令自动创建一个以时间命名的目录：如此一来，innobackupex命令将会创建一个BACKUP-DIR目录来存储备份数据。
  ```

#### 备份锁

一篇不错的介绍xtrabackup锁的文章：https://www.percona.com/blog/2014/03/11/introducing-backup-locks-percona-server-2/。

percona Server 5.6+ **支持一种新锁——backup lock(备份锁)**，这种锁是percona对MySQL的补充，专门为备份而设计。这种锁在percona Server 5.6+ 有，MariaDB中也有，但是Oracle的MySQL中没有，至少MySQL 5.7中没有。

这种锁用在备份的时候替代 flush tables with read lock 获取全局锁，是一种轻量级的全局锁。它有两种类型的锁：备份表锁和二进制日志锁。为此新增了3种语法：

```
lock tables for backup   # 申请备份表锁
lock binlog for backup   # 申请二进制日志锁
unlock binlog            # 释放二进制日志锁
```

备份表锁在全局范围内只对非innodb表加锁，所以持有该锁后无法修改非innodb表，但却不影响innodb表的DML。当然，因为是全局锁，所以也会阻塞DDL操作。

二进制日志锁在全局范围内锁定二进制日志，所以会阻塞其他会话修改二进制日志。这样可以保证能够获取到二进制日志中一致性的位置坐标。 

#### xtrabackup备份原理说明

不管是使用innobackupex还是xtrabackup工具进行备份和恢复，都有3个步骤：备份(backup)、准备(prepare)、恢复(copy back)。

注意，xtrabackup备份过程中，先备份innodb表，再备份非innodb表

**备份过程(backup阶段)**

1. 在启动xtrabackup时记下LSN并将redo log拷贝到备份目标目录下的xtrabackup_logfile文件中。由于拷贝需要一定时间，如果在拷贝时间段内有日志写入，将导致拷贝的日志和MySQL的redo log不一致，所以xtrabackup还有一个后台进程监控着mysql的redo log，每秒监控一次，当MySQL的redo log有变化，该监控进程会立即将变化的内容写入到xtrabackup_logfile文件，这样就能保证拷贝走的redo log中记录了一切变化。但是这也是有风险的，因为redo是轮训式循环写入的，如果某一时刻有非常大量的日志写到redo log中，使得还没开始复制的日志就被新日志覆盖了，这样会日志丢失，并报错。

2. 拷贝完初始版的redo log后，xtrabackup开始拷贝innodb表的数据文件(即表空间文件.ibd文件和ibdata1)。注意，此时不拷贝innodb的frm文件。

3. 当innodb相关表的数据文件拷贝完成后，xtrabackup开始准备拷贝非innodb的文件。但在拷贝它们之前，要先对非innodb表进行加锁防止拷贝时有语句修改这些类型的表数据。

   对于不支持backup lock的版本，只能通过flush tables with read lock来获取全局读锁，但这样也同样会锁住innodb表，杀伤力太大。所以使用xtrabackup备份Oracle的MySQL，实质上只能实现innodb表的部分时间热备、部分时间温备。

   对于支持backup lock的版本，xtrabackup通过lock tables for backup获取轻量级的backup locks来替代flush tables with read lock，因为它只锁定非innodb表，所以由此实现了innodb表的真正热备。

4. 当获取到非innodb表的锁以后，开始拷贝非innodb表的数据和.frm文件。当这些拷贝完成之后，继续拷贝其他存储引擎类型的文件。(实际上，拷贝非innodb表的数据是在获取backup locks(如果支持)后自动进行的，它们属于同一个过程)

5. 当拷贝阶段完成后，就到了备份的收尾阶段。包括获取二进制日志中一致性位置的坐标点、结束redo log的监控和拷贝、释放锁等。

   对于不支持backup lock的版本，收尾阶段的过程是这样的：获取二进制日志的一致性坐标点、结束redo log的监控和拷贝、释放锁。

   对于支持backup lock的版本，收尾阶段的过程是这样的：先通过lock binlog for bakcup来获取二进制日志锁，然后结束redo log的监控和拷贝，再unlock tables释放表锁，随后获取二进制日志的一致性位置坐标点，最后unlock binlog释放二进制日志锁。

6. 如果一切都OK，xtrabackup将以状态码0退出。

所以，对是否支持backup lock的版本，xtrabackup备份的时的行为是不一样的。

![UTOOLS1571841081495.png](https://i.loli.net/2019/10/23/TduRvhGc5EXp3qD.png)

backup阶段的过程具体如下图所示：

![UTOOLS1571841099790.png](https://i.loli.net/2019/10/23/fUHsvuWVeYgDzQ7.png)

FTWRL：flush table with read lock;

![UTOOLS1571841116034.png](https://i.loli.net/2019/10/23/Pwp3QasHAJETCvk.png)

**备份流程图**

![UTOOLS1571884296927.png](https://i.loli.net/2019/10/24/GAzchpri18FRML2.png)

1. innobackupex启动后，会先fork一个进程，用于启动xtrabackup，然后等待xtrabackup备份ibd数据文件；
2. xtrabackup在备份innoDB数据是，有2种线程：redo拷贝线程和ibd数据拷贝线程。xtrabackup进程开始执行后，会启动一个redo拷贝的线程，用于从最新的checkpoint点开始顺序拷贝redo.log；再启动ibd数据拷贝线程，进行拷贝ibd数据。这里是先启动redo拷贝线程的。在此阶段，innobackupex进行处于等待状态（等待文件被创建）
3. xtrabackup拷贝完成ibd数据文件后，会通知innobackupex（通过创建文件），同时xtrabackup进入等待状态（redo线程依旧在拷贝redo.log）
4. innobackupex收到xtrabackup通知后哦，执行FLUSH TABLES WITH READ LOCK（FTWRL），取得一致性位点，然后开始备份非InnoDB文件（如frm、MYD、MYI、CSV、opt、par等格式的文件），在拷贝非InnoDB文件的过程当中，数据库处于全局只读状态。
5. 当innobackup拷贝完所有的非InnoDB文件后，会通知xtrabackup，通知完成后，进入等待状态；
6. xtrabackup收到innobackupex备份完成的通知后，会停止redo拷贝线程，然后通知innobackupex，redo.log文件拷贝完成；
7. innobackupex收到redo.log备份完成后，就进行解锁操作，执行：UNLOCK TABLES；
8. 最后innbackupex和xtrabackup进程各自释放资源，写备份元数据信息等，innobackupex等xtrabackup子进程结束后退出。

**准备过程(prepare阶段)**

由于备份的时候拷贝走的数据文件可能是不一致的，比如监控着MySQL的redo log中在拷贝过程完成后又新的事务提交了，而拷贝走的数据是未提交状态的，那么就需要对该事务前滚；如果监控到的日志中有事务未提交，那么该事务就需要回滚。

但是如果只备份了myisam表或其他非事务表数据，因为备份阶段直接锁定了这些表，所以不会有不一致的状态。

xtrabackup有一个"准备"的阶段。这个阶段的实质就是对备份的innodb数据应用redo log，该回滚的回滚，该前滚的前滚，最终保证xtrabackup_logfile中记录的redo log已经全部应用到备份数据页上，并且实现了一致性。当应用结束后，会重写"xtrabackup_logfile"再次保证该redo log和备份的数据是对应的。

准备过程不需要连接数据库，该过程可以在任意装了xtrabackup软件的机器上进行，之所能实现是因为xtrabackup软件的内部嵌入了一个简化的innodb存储引擎，可以通过它完成日志的应用。

**恢复过程(copy back阶段)**

xtrabackup的恢复过程实质是将备份的数据文件和结构定义等文件拷贝回MySQL的datadir。同样可以拷贝到任意机器上。

要求恢复之前MySQL必须是停止运行状态，且datadir是空目录，除非恢复的操作是导入表的操作。具体见后文对应的内容。

#### 准备实验环境

创建测试数据库backuptest，并创建myisam表和innodb表，此处简单的使用数值辅助表并分别插入1亿条数据。使用navicat运行查询。

```mysql
DROP DATABASE IF EXISTS backuptest;
CREATE DATABASE backuptest;
USE backuptest;

# 创建myisam类型的数值辅助表和对应插入数据的存储过程
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

# 创建innodb类型的数值辅助表和对应插入数据的存储过程
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

# 分别向两个数值辅助表中插入1亿条数据，
CALL proc_num1(100000000);
CALL proc_num2(100000000);
```

#### innobackupex工具

**innobackupex实现全备份和恢复的过程**

1. 全备

   除了给定连接MySQL服务器的连接参数，只需再给定一个目录即可，该目录是备份的目标位置。默认xtrabackup连接数据库的时候从配置文件中去读取和备份相关的配置，可以使用选项--defaluts-file指定连接时的参数配置文件，但如果指定该选项，该选项只能放在第一个选项位置。

   ```
   innobackupex --user=root --password=123456 /bakdir/
   ```

   默认备份的路径是指定路径/bakdir下的一个以时间为时间戳的目录。

   查看该文件目录中文件和大小，可以看出xtrabackup的行为就是复制了目标数据库的相关文件，并新建了几个文件。

   其中：

   - backup-my.cnf是拷贝过来的配置文件。里面只包含[mysqld]配置片段和备份有关的选项。
   - xtrabackup_binlog_info中记录的是当前使用的二进制日志文件。
   - xtrabackup_checkpoints中记录了备份的类型是全备还是增备，还有备份的起始、终止LSN号。
   - xtrabackup_info中记录的是备份过程中的一些信息。
   - xtrabackup_logfile是复制和监控后写的redo日志。该日志是备份后下一个操作"准备"的关键。只有通过它才能实现数据一致性。

2. 全备的准备过程

   在全备份完成之后，备份的数据中如果有innodb数据，则还不能用来恢复。因为从xtrabackup开始备份的时候就监控着MySQL的redo log，在拷贝的innodb数据文件中很可能还有未提交的事务，并且拷贝完innodb数据之后还可能提交了事务或者开启了新的事务等等。总之，全备之后的状态不一定是一致的。但是如果只备份了myisam表或其他非事务表数据，因为备份阶段直接锁定了这些表，所以不会有不一致的状态。

   xtrabackup有一个"准备"的阶段。这个阶段的实质就是对备份的innodb数据应用redo log，该回滚的回滚，该前滚的前滚，最终保证xtrabackup_logfile中记录的redo log已经全部应用到备份数据页上，并且实现了一致性。当应用结束后，会重写"xtrabackup_logfile"再次保证该redo log和备份的数据是对应的。

   例如，备份的innodb数据文件中存在未提交的事务，但是在监控到的日志中进行了提交，那么就需要对该事务前滚；如果监控到的日志中有事务未提交，那么该事务就需要回滚。

   准备阶段使用的模式选项是"--apply-log"。准备阶段不会连接MySQL，所以不用指定连接选项如--user等。

   ```
   [root@xuexi bakdir]# innobackupex --apply-log /bakdir/2019-10-24_06-35-19/
   ```

   在准备成功时，会在频幕上输出如下提示内容：

   ```
   InnoDB: FTS optimize thread exiting.
   InnoDB: Starting shutdown...
   InnoDB: Shutdown completed; log sequence number 5104693288
   191024 06:37:51 completed OK!
   ```

   在准备阶段，有一个内存使用量选项"--use-memory"，该选项默认值为100M，值越大准备的过程越快。当然，将该值加大的前提是服务器内存够用。

3. 全备份的恢复过程

   恢复的阶段就是向MySQL的datadir拷贝。全备份的恢复要求MySQL必须处于stop状态，并且datadir必须为空哪怕是和MySQL无关的文件也不能存在，它不会去覆盖datadir中已存在的内容。否则会提示如下错误：

   ```
   innobackupex version 2.4.15 based on MySQL server 5.7.19 Linux (x86_64) (revision id: 544842a)
   Original data directory /var/lib/mysql is not empty!
   ```

   停止mysql并清空datadir。

   ```
   service mysqld stop
   rm /var/lib/mysql/* -rf
   ```

   恢复时使用的模式是"--copy-back"，选项后指定要恢复的源备份目录。恢复时因为不需要连接数据库，所以不用指定连接选项，如--user等。

   ```shell
   $ innobackupex --copy-back /bakdir/2019-10-24_06-35-19/
   191024 06:43:47 completed OK!
   ```

   **拷贝完成后，MySQL的datadir的文件的所有者和属组是innobackupex的调用者，所以需要改回mysql.mysql。**

   ```mysql
   $ ls -l /var/lib/mysql/
   total 647212
   drwxr-x--- 2 root root      4096 Oct 24 06:43 analysis
   drwxr-x--- 2 root root      4096 Oct 24 06:43 backuptest
   -rw-r----- 1 root root       454 Oct 24 06:42 ib_buffer_pool
   -rw-r----- 1 root root  50331648 Oct 24 06:42 ib_logfile0
   -rw-r----- 1 root root  50331648 Oct 24 06:42 ib_logfile1
   -rw-r----- 1 root root 549453824 Oct 24 06:42 ibdata1
   -rw-r----- 1 root root  12582912 Oct 24 06:42 ibtmp1
   drwxr-x--- 2 root root      4096 Oct 24 06:42 mysql
   drwxr-x--- 2 root root      4096 Oct 24 06:43 performance_schema
   drwxr-x--- 2 root root     12288 Oct 24 06:42 sys
   -rw-r----- 1 root root       422 Oct 24 06:43 xtrabackup_info
   -rw-r----- 1 root root         1 Oct 24 06:42 xtrabackup_master_key_id
   
   $ chown -R mysql.mysql /var/lib/mysql/*
   ```
   
   完成这些之后，就可以启动MySQL服务器了。可以进入mysql测试backuptest数据库中的数据是否完整。

**innobackupex实现增量备份和恢复的过程**

增量备份依赖于全备份。xtrabackup实现增量备份的原理是通过比较全备份的终点LSN和当前的LSN，增备时将从终点LSN开始一直备份到当前的LSN。在备份时也有redo log的监控线程，对于增备过程中导致LSN增长的操作也会写入到日志中。

增备的实现依赖于LSN，所以只对innodb有效，对myisam表使用增备时，背后进行的是全备。

**innobackupex实现导出和导入单张表的过程**

默认情况下，InnoDB表不能通过直接复制表文件的方式在mysql服务器之间进行移植，即便使用了innodb_file_per_table选项。而使用Xtrabackup工具可以实现此种功能，不过只能"导出"具有.ibd文件的表，也就是说导出表的mysql服务器启用了innodb_file_per_table选项，而且要导出的表还是在启用该选项之后才创建的。

导入表的是，要求导入表的服务器版本是MySQL 5.6+，且启用了innodb_file_per_table选项。

1. 导出表

   导出表是在"准备"的过程中进行的，不是在备份的时候导出。对于一个已经备份好的备份集，使用"--apply-log"和"--export"选项即可导出备份集中的表

   假如以全备份集/bakdir/2019-10-24_06-35-19/为例，要导出其中的表。

```
   innobackupex --apply-log --export /bakdir/2019-10-24_06-35-19/
```

   在导出过程中，会看到如下信息：

   ```
   xtrabackup: export metadata of table 'analysis/t' to file `./analysis/t.exp` (1 indexes)
   xtrabackup:     name=PRIMARY, id.low=41, page=3
   ```

   它说明了创建了一个.exp文件。

   查看备份集目录下的backuptest目录，会发现多出了2个文件：.cfg和.exp，再加上.ibd文件，这3个文件是后续导入表时所需的文件。

   ```
   -rw-r--r-- 1 root root  349 Apr  2 18:15 num_innodb.cfg
   -rw-r----- 1 root root  16K Apr  2 18:15 num_innodb.exp
   -rw-r----- 1 root root 8.4K Apr  2 17:41 num_innodb.frm
   -rw-r----- 1 root root  31M Apr  2 17:41 num_innodb.ibd
   ```

   其中.cfg文件是一种特殊的innodb数据字典文件，它和exp文件的作用是差不多的，只不过后者还支持在xtradb中导入，严格地讲，要将导出的表导入到MySQL5.6或者percona server 5.6中，".cfg"文件完全可以不需要，但是如果有该文件的话，会进行架构验证。

2. 导入表

   要在mysql服务器上导入来自于其它服务器的某innodb表，需要先在当前服务器上创建一个跟原表表结构一致的表，而后才能实现将表导入：

   ```
   mysql> CREATE TABLE tabletest (...)  ENGINE=InnoDB;
   ```

   然后将此表的表空间：

   ```
   mysql> ALTER TABLE mydatabase.tabletest  DISCARD TABLESPACE;
   ```

   接下来，将来自于"导出"表的的.ibd和.exp文件复制到当前服务器的数据目录，如果导入目标服务器是MySQL 5.6+，也可以复制.cfg文件。然后使用如下命令将其“导入”：

   ```
   mysql> ALTER TABLE mydatabase.tabletest IMPORT TABLESPACE;
   ```

**innobackupex实现部分备份和恢复的过程**

xtrabackup支持部分备份，意味着可以指定备份哪个数据库或者哪个表。

部分备份只有一点需要注意：在恢复的时候不要通过"--copy-back"的方式拷贝回datadir，而是应该使用导入表的方式。尽管使用拷贝的方式有时候是可行的，但是很多情况下会出现数据库不一致的状态。

1. 备份

   创建部分备份有三种方式：

   1.  通过"--include"选项可以指定正则来匹配要备份的表，这种方式要使用完整对象引用格式，即db_name.tab_name的方式。
   2. 将要备份的表分行枚举到一个文件中，通过"--tables-file"指定该文件。
   3. 或者使用"--databases"指定要备份的数据库或表，指定备份的表时要使用完整对象引用格式，多个元素使用空格分开。

   使用前两种部分备份方式，只能备份innodb表，不会备份任何myisam，即使指定了也不会备份。而且要备份的表必须有独立的表空间文件，也就是说必须开启了innodb_file_per_table，更精确的说，要备份的表是在开启了innodb_file_per_table选项之后才创建的。第三种备份方式可以备份myisam表。

   例如 `--include='^back.*[.]num_*'` ，将备份back字母开头的数据库中num开头的表，其中"[.]"的中括号不能少，因为正则中"."有特殊意义，所以使用中括号来枚举以实现对象的完整引用。

   ```
   innobackupex --user=root --password=123456 --include='^back*[.]num_*' /bakdir/
   ```

   使用"--include"和"--tables-file"备份后，会生成一个时间戳目录，目录中只有和要备份的表有关的文件。

   ```
   ls /bakdir/2017-04-02_17-35-46/
   backup-my.cnf  ibdata1  xtrabackup_binlog_info  xtrabackup_checkpoints  xtrabackup_info  xtrabackup_logfile
   ```

   如果使用的是--databases选项，则会生成一个时间戳目录，里面有备份的数据库代表的目录，如果只备份了某个表，则该数据库目录中只有该表相关的文件。

   ```
   innobackupex --user=root --password=123456 --databases='mysql.user backuptest' /bakdir/
   ```

   上面只备份mysql.user表和backuptest数据库，在生成的时间戳目录中将有两个mysql目录和backuptest目录。

2. 部分备份的准备和恢复过程

   部分备份的准备和恢复过程分别是导出表和导入表的过程

**innobackupex实现定时点恢复**

xtrabackup本身无法实现定时点恢复，只能通过恢复备份后通过二进制日志实现。实现方法和一般定时点恢复是一样的。

**流备份和远程备份**

xtrabackup支持备份流，当前可用的流类型只有tar和xtrabackup自带的xbstream，通过流可以将它们传递给其他程序进行相关的操作，如压缩。但是不建议在备份的同时进行压缩，因为压缩会占用极大的cpu资源，使得备份时间延长很多，温备的过程也就延长了。

另外，MySQL的数据文件压缩比非常大，所以建议备份后在空闲的时候进行压缩。

xtrabackup还支持远程备份，只需使用"--remote-host"指定远程的主机名即可，指定方式和ssh指定的方式一样。如--remote-host=root@192.168.100.18。

使用流备份的方法如下：

```shell
# 使用tar流
innobackupex --user=root --password=123456 --stream=tar /bakdir/ >/tmp/a.tar
# 使用tar流的同时交给gzip压缩
innobackupex --user=root --password=123456 --stream=tar /bakdir/ | gzip >/tmp/a.tar.gz
# 使用tar流备份到远程主机中并归档
innobackupex --user=root --password=123456 --stream=tar /bakdir/ | ssh root@192.168.100.10  "cat -  > /tmp/`date +%F_%H-%M-%S`.tar"
# 使用tar流备份到原远程主机中并解包
innobackupex --user=root --password=123456 --stream=tar /bakdir/ | ssh root@192.168.100.10  "cat -  | tar -x -C /tmp/"

# 使用xtrabackup自带的xbstream流
innobackupex --user=root --password=123456 --stream=xbstream /bakdir/ >/tmp/b.xbs
# 解压xbstream流
innobackupex --user=root --password=123456 --stream=xbstream /bakdir/ | ssh root@192.168.100.10  "cat -  | xbstream -x -C /tmp/"
# 使用xbstream流的同时进行压缩，使用"--compress"选项
innobackupex --user=root --password=123456 --stream=xbstream --compress /bakdir/ > /bakdir/backup.xbs
```

注意，如果在解压备份的.tar.gz时出错，可能在解压的时候需要使用-i选项。如tar -xif /tmp/b.tar/gz。

**加速备份**

当备份到本地的时候，可以使用"--rsync"选项，该选项用于在flush tables with read lock后调用rsync替代cp进程复制非Innodb数据和.frm文件，加快复制速度。

但要注意，因为支持备份锁的版本在获取到backup locks的时候会自动复制非Innodb数据和.frm文件，所以"--rsync"选项是无效的。

另外，该选项不能和"--stream"选项和"--remote-host"选项同时使用。

#### xtrabackup工具

xtrabackup工具是C语言编写的工具，在innobackupex使用C重写之后，innobackupex是该工具的一个软链接。但是它不能实现innobackupex的所有功能，例如xtrabackup工具没有恢复功能，而innobackupex有"--copy-back"选项来恢复。

xtrabackup工具有两种常用运行模式："--backup"和"--prepare"。还有两个比较少用的模式："--stats"和"--print-param"。

由于前文对innobackupex的介绍非常详细，xtrabackup在功能实现上和它是一样的。所以下面将简单介绍。

**xtrabackup实现全备** 

1. 备份过程

   和innobackupex备份过程不同的是，xtrabackup的备份路径是由"--target-dir"选项严格指定的，如果指定的目录不存在，它备份的时候不会在target-dir目录中再创建时间戳子目录。

   ```
   [root@xuexi data]# xtrabackup --backup --user=root --password=123456 --datadir=/var/lib/mysql/ --target-dir=/bakdir/fullback

   [root@xuexi data]# ls /bakdir/fullback
   backup-my.cnf  ibdata1  Performance         secure_dir  xtrabackup_binlog_info  xtrabackup_info
   backuptest     mysql    performance_schema  world       xtrabackup_checkpoints  xtrabackup_logfile
   ```

2. 准备过程

   ```
   xtrabackup --prepare --target-dir=/bakdir/fullback
   ```

3. 恢复过程

   xtrabackup自身不能恢复，只能通过拷贝备份集的方式来恢复。例如使用rsync或者cp等。

   另外，恢复时也一样要求MySQL是stop状态，datadir是空目录。并且拷贝完成后要修改datadir中文件的所有者和属组为mysql用户和组。

   ```
   service mysqld stop
   rm -rf /var/lib/mysql/*
   rsync -azP /bakdir/fullback/* /var/lib/mysql/
   chown -R mysql.mysql /var/lib/mysql/*
   ```

**xtrabackup实现增备**

1. 首先进行全备

   ```
   xtrabackup --backup --user=root --password=123456 --datadir=/var/lib/mysql/ --target-dir=/bakdir/base_full
   ```

2. 进行第一次增备

   ```
   xtrabackup --backup --user=root --password=123456 --target-dir=/bakdir/incr_bak1 --incremental-basedir=/bakdir/base_full --datadir=/var/lib/mysql/ 
   ```

   同样也可以在增备时使用"--incremental-lsn"来指定从哪个lsn开始增量备份，这和innobackupex是一样的。

3. 进行第二次增备

   ```
   xtrabackup --backup --user=root --password=123456 --target-dir=/bakdir/incr_bak2 --incremental-basedir=/bakdir/incr_bak1 --datadir=/var/lib/mysql/
   ```

4. 准备过程

   准备过程和innobackupex是一样的，使用"--apply-log-only"来直线向前地应用redo log，同样，在最后一个增备集的准备过程中不能使用"--apply-log-only"选项。

   ```
   xtrabackup --prepare --apply-log-only --target-dir=/bakdir/base_full
   xtrabackup --prepare --apply-log-only --target-dir=/bakdir/base_full --incremental-dir=/bakdir/incr_bak1
   xtrabackup --prepare --target-dir=/bakdir/base_full --incremental-dir=/bakdir/incr_bak2
   ```

5. 恢复阶段

   恢复阶段即拷贝阶段，和前面全备的恢复阶段是一样的，要求MySQL停止运行，datadir是空目录，拷贝全备目录到datadir，修改datadir的所有者和属组。

**xtrabackup实现部分备份**

xtrabackup部分备份和innobackupex不太一样，innobackupex的部分备份实质上是在已经备份好的备份集上导出导入表，而xtrabackup直接在备份过程中筛选要备份的目标，它不建立在已有的备份集上。

1. 备份过程

   - xtrabackup使用"--tables"选项对应innobackupex的"--include"选项，它们是一样的，都是正则匹配完整对象引用名称。
   - 使用"--tables-file"选项指定枚举要备份表的列表，每行一个表，表名需要使用完整对象引用名称。和innobackupex一样的。
   - 使用"--databases"和"--databases-file"指定要单独备份的数据库或表，后者可以枚举出要备份的列表。这两个选项不能使用通配符和正则匹配。

   ```
   xtrabackup --backup --user=root --password=123456 --target-dir=/bakdir/part_bak1 --datadir=/var/lib/mysql/ --tables="^back*[.]num_*"
   ```

2. 准备过程

   xtrabackup的部分备份的准备要比innobackupex方便的多，直接对备份集进行"--prepare"即可。

   ```
   xtrabackup --prepare --target-dir=/bakdir/part_bak1
   ```

### 参考

1. [备份和恢复(一)：mysqldump工具用法详述](http://www.cnblogs.com/f-ck-need-u/p/9013458.html)
2. [备份和恢复(二)：导入、导出表数据](http://www.cnblogs.com/f-ck-need-u/p/9013643.html)
3. [备份和恢复(三)：xtrabackup用法和原理详述](http://www.cnblogs.com/f-ck-need-u/p/9018716.html)
   
   ```
   
   ```