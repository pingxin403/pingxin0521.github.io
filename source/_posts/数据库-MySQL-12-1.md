---
title: MySQL 复制--传统方式（一）
date: 2019-10-16 13:18:59
tags:
 - MySQL
 - 数据库
categories:
 - 数据库
 - MySQL
---

1. MySQL Replication (MySQL 主从复制) 是什么？

2. 为什么要主从复制以及它的实现原理是什么？

<!--more-->

### 理论

#### MySQL 主从复制概念

MySQL 主从复制是指数据可以从一个MySQL数据库服务器主节点复制到一个或多个从节点。MySQL 默认采用异步复制方式，这样从节点不用一直访问主服务器来更新自己的数据，数据的更新可以在远程连接上进行，从节点可以复制主数据库中的所有数据库或者特定的数据库，或者特定的表。

以下几个知识点是掌握MySQL复制所必备的：

1. 复制的原理
2. 将master上已存在的数据恢复到slave上作为基准数据
3. 获取**正确的**binlog坐标
4. **深入理解**`show slave status`中的一些状态信息

mysql replication官方手册：https://dev.mysql.com/doc/refman/5.7/en/replication.html。

mysql复制是指从一个mysql服务器(MASTER)将数据**通过日志的方式经过网络传送**到另一台或多台mysql服务器(SLAVE)，然后在slave上重放(replay或redo)传送过来的日志，以达到和master数据同步的目的。

它的工作原理很简单。首先**确保master数据库上开启了二进制日志，这是复制的前提**。

- 在slave准备开始复制时，首先**要执行`change master to`语句设置连接到master服务器的连接参数**，在执行该语句的时候要提供一些信息，包括如何连接和要从哪复制binlog，这些信息在连接的时候会记录到slave的datadir下的master.info文件中，以后再连接master的时候将不用再提供这新信息而是直接读取该文件进行连接。

- 在slave上有两种线程，

  分别是IO线程和SQL线程

  - IO线程用于连接master，监控和接受master的binlog。当启动IO线程成功连接master时，**master会同时启动一个dump线程**，该线程将slave请求要复制的binlog给dump出来，之后IO线程负责监控并接收master上dump出来的二进制日志，当master上binlog有变化的时候，IO线程就将其复制过来并写入到自己的中继日志(relay log)文件中。
  - slave上的另一个线程SQL线程用于监控、读取并重放relay log中的日志，将数据写入到自己的数据库中。如下图所示。

站在slave的角度上看，过程如下：

![UTOOLS1571835685478.png](https://i.loli.net/2019/10/23/nt1BOgdHyk8zeMX.png)

站在master的角度上看，过程如下(默认的异步复制模式，前提是设置了`sync_binlog=1`，否则binlog刷盘时间由操作系统决定)：

![UTOOLS1571835714172.png](https://i.loli.net/2019/10/23/RNaIJ6LswyACFZE.png)

所以，可以认为复制大致有三个步骤：

1. 数据修改写入master数据库的binlog中。
2. slave的IO线程复制这些变动的binlog到自己的relay log中。
3. slave的SQL线程读取并重新应用relay log到自己的数据库上，让其和master数据库保持一致。

从复制的机制上可以知道，在复制进行前，slave上必须具有master上部分完整内容作为复制基准数据。例如，master上有数据库A，二进制日志已经写到了pos1位置，那么在复制进行前，slave上必须要有数据库A，且如果要从pos1位置开始复制的话，还必须有和master上pos1之前完全一致的数据。如果不满足这样的一致性条件，那么在replay中继日志的时候将不知道如何进行应用而导致数据混乱。**也就是说，复制是基于binlog的position进行的，复制之前必须保证position一致。**(注：这是传统的复制方式所要求的)

可以选择对哪些数据库甚至数据库中的哪些表进行复制。**默认情况下，MySQL的复制是异步的。slave可以不用一直连着master，即使中间断开了也能从断开的position处继续进行复制。**

MySQL 5.6对比MySQL 5.5在复制上进行了很大的改进，主要包括支持GTID(Global Transaction ID,全局事务ID)复制和多SQL线程并行重放。GTID的复制方式和传统的复制方式不一样，通过全局事务ID，它不要求复制前slave有基准数据，也不要求binlog的position一致。

MySQL 5.7.17则提出了组复制(MySQL Group Replication,MGR)的概念。像数据库这样的产品，必须要尽可能完美地设计一致性问题，特别是在集群、分布式环境下。Galera就是一个MySQL集群产品，它支持多主模型(多个master)，但是当MySQL 5.7.17引入了MGR功能后，Galera的优势不再明显，甚至MGR可以取而代之。MGR为MySQL集群中多主复制的很多问题提供了很好的方案，可谓是一项革命性的功能。

复制和二进制日志息息相关。

#### MySQL 主从复制主要用途

![UTOOLS1571835585634.png](https://i.loli.net/2019/10/23/eHXnrd1EAT2yzhW.png)

1. 提供了读写分离的能力。

   replication让所有的slave都和master保持数据一致，因此外界客户端可以从各个slave中读取数据，而写数据则从master上操作。也就是实现了读写分离。

   需要注意的是，为了保证数据一致性，**写操作必须在master上进行**。

   通常说到读写分离这个词，立刻就能意识到它会分散压力、提高性能。

2. 为MySQL服务器提供了良好的伸缩(scale-out)能力。

   由于各个slave服务器上只提供数据检索而没有写操作，因此"随意地"增加slave服务器数量来提升整个MySQL群的性能，而不会对当前业务产生任何影响。

   之所以"随意地"要加上双引号，是因为每个slave都要和master建立连接，传输数据。如果slave数量巨多，master的压力就会增大，网络带宽的压力也会增大。

3. 数据库备份时，对业务影响降到最低。

   由于MySQL服务器群中所有数据都是一致的(至少几乎是一致的)，所以在需要备份数据库的时候可以任意停止某一台slave的复制功能(甚至停止整个mysql服务)，然后从这台主机上进行备份，这样几乎不会影响整个业务(除非只有一台slave，但既然只有一台slave，说明业务压力并不大，短期内将这个压力分配给master也不会有什么影响)。

4. 能提升数据的安全性。

   这是显然的，任意一台mysql服务器断开，都不会丢失数据。即使是master宕机，也只是丢失了那部分还没有传送的数据(异步复制时才会丢失这部分数据)。

5. 数据分析不再影响业务。

   需要进行数据分析的时候，直接划分一台或多台slave出来专门用于数据分析。这样OLTP和OLAP可以共存，且几乎不会影响业务处理性能。

#### MySQL 主从形式

- 一主一从

- 一主多从，提高系统的读性能

  ![UTOOLS1571822738885.png](https://i.loli.net/2019/10/23/Qg32uEPKtZI9nLY.png)

  一主一从和一主多从是最常见的主从架构，实施起来简单并且有效，不仅可以实现HA，而且还能读写分离，进而提升集群的并发能力。

- 多主一从 （从5.7开始支持）

  ![UTOOLS1571822806910.png](https://i.loli.net/2019/10/23/spieUXkTINRDChl.png)

- 双主复制

  双主复制，也就是互做主从复制，每个master既是master，又是另外一台服务器的slave。这样任何一方所做的变更，都会通过复制应用到另外一方的数据库中。

- 级联复制

  ![UTOOLS1571822858116.png](https://i.loli.net/2019/10/23/gJorZTCU8ctEy9m.png)

  级联复制模式下，部分slave的数据同步不连接主节点，而是连接从节点。因为如果主节点有太多的从节点，就会损耗一部分性能用于replication，那么我们可以让3~5个从节点连接主节点，其它从节点作为二级或者三级与从节点连接，这样不仅可以缓解主节点的压力，并且对数据一致性没有负面影响。

#### MySQL 主从复制原理

MySQL主从复制涉及到三个线程，一个运行在主节点（log dump thread），其余两个(I/O thread, SQL thread)运行在从节点，如下图所示:

![UTOOLS1571822895104.png](https://i.loli.net/2019/10/23/W8flr4UqNtFed3g.png)

1. 主节点 binary log dump 线程

   当从节点连接主节点时，主节点会创建一个log dump 线程，用于发送bin-log的内容。在读取bin-log中的操作时，此线程会对主节点上的bin-log加锁，当读取完成，甚至在发动给从节点之前，锁会被释放。

2. 从节点I/O线程

   当从节点上执行`start slave`命令之后，从节点会创建一个I/O线程用来连接主节点，请求主库中更新的bin-log。I/O线程接收到主节点binlog dump 进程发来的更新之后，保存在本地relay-log中。

3. 从节点SQL线程

   SQL线程负责读取relay log中的内容，解析成具体的操作并执行，最终保证主从数据的一致性。

   对于每一个主从连接，都需要三个进程来完成。当主节点有多个从节点时，主节点会为每一个当前连接的从节点建一个binary log dump 进程，而每个从节点都有自己的I/O进程，SQL进程。从节点用两个线程将从主库拉取更新和执行分成独立的任务，这样在执行同步数据任务的时候，不会降低读操作的性能。比如，如果从节点没有运行，此时I/O进程可以很快从主节点获取更新，尽管SQL进程还没有执行。如果在SQL进程执行之前从节点服务停止，至少I/O进程已经从主节点拉取到了最新的变更并且保存在本地relay日志中，当服务再次起来之后，就可以完成数据的同步。

   要实施复制，首先必须**打开Master 端的binary log（bin-log）功能**，否则无法实现。

   因为整个复制过程实际上就是Slave 从Master 端获取该日志然后再在自己身上完全顺序的执行日志中所记录的各种操作。如下图所示：

   ![UTOOLS1571822974087.png](https://i.loli.net/2019/10/23/zaeOCKJXfF98rVo.png)

复制的基本过程如下：

- 从节点上的I/O 进程连接主节点，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容；
- 主节点接收到来自从节点的I/O请求后，通过负责复制的I/O进程根据请求信息读取指定日志指定位置之后的日志信息，返回给从节点。返回信息中除了日志所包含的信息之外，还包括本次返回的信息的bin-log file 的以及bin-log position；从节点的I/O进程接收到内容后，将接收到的日志内容更新到本机的relay log中，并将读取到的binary log文件名和位置保存到master-info 文件中，以便在下一次读取的时候能够清楚的告诉Master“我需要从某个bin-log 的哪个位置开始往后的日志内容，请发给我”；
- Slave 的 SQL线程检测到relay-log 中新增加了内容后，会将relay-log的内容解析成在祝节点上实际执行过的操作，并在本数据库中执行。

#### MySQL 主从复制模式

MySQL支持两种不同的复制方法：传统的复制方式和GTID复制。MySQL 5.7.17之后还支持组复制(MGR)。

- 传统的复制方法要求复制之前，slave上必须有基准数据，且binlog的position一致。

- GTID复制方法不要求基准数据和binlog的position一致性。GTID复制时，master上只要一提交，就会立即应用到slave上。这极大地简化了复制的复杂性，且更好地保证master上和各slave上的数据一致性。

从数据同步方式的角度考虑，MySQL支持4种不同的同步方式：同步(synchronous)、半同步(semisynchronous)、异步(asynchronous)、延迟(delayed)。所以对于复制来说，就分为同步复制、半同步复制、异步复制和延迟复制。

MySQL 主从复制默认是异步的模式。MySQL增删改操作会全部记录在binary log中，当slave节点连接master时，会主动从master处获取最新的bin log文件。并把bin log中的sql relay。

1. 异步模式（mysql async-mode）

   异步模式如下图所示，这种模式下，主节点不会主动push bin log到从节点，这样有可能导致failover的情况下，也许从节点没有即时地将最新的bin log同步到本地。

   ![UTOOLS1571823048643.png](https://i.loli.net/2019/10/23/n1iQOC4zxoM73Sr.png)

   客户端发送DDL/DML语句给master，**master执行完毕立即返回成功信息给客户端，而不管slave是否已经开始复制**。这样的复制方式导致的问题是，当master写完了binlog，而slave还没有开始复制或者复制还没完成时，**slave上和master上的数据暂时不一致，且此时master突然宕机，slave将会丢失一部分数据。如果此时把slave提升为新的master，那么整个数据库就永久丢失这部分数据。**

   ![img](https://images2018.cnblogs.com/blog/733013/201805/733013-20180524205215240-203795747.png)

2. 半同步模式(mysql semi-sync)

   客户端发送DDL/DML语句给master，master执行完毕后**还要等待一个slave写完relay log并返回确认信息给master，master才认为此次DDL/DML语句是成功的，然后才会发送成功信息给客户端**。半同步复制只需等待一个slave的回应，且等待的超时时间可以设置，超时后会自动降级为异步复制，所以在局域网内(网络延迟很小)使用半同步复制是可行的。

   ![UTOOLS1571835861550.png](https://i.loli.net/2019/10/23/OC1dT4gR6ZacXf7.png)

   例如上图中，只有第一个slave返回成功，master就判断此次DDL/DML成功，其他的slave无论复制进行到哪一个阶段都无关紧要。

   半同步模式不是mysql内置的，从mysql 5.5开始集成，需要master 和slave 安装插件开启半同步模式。

3. 全同步模式

   客户端发送DDL/DML语句给master，master执行完毕后还需要**等待所有的slave都写完了relay log才认为此次DDL/DML成功，然后才会返回成功信息给客户端**。同步复制的问题是master必须等待，所以延迟较大，在MySQL中不使用这种复制方式。

   ![UTOOLS1571835832317.png](https://i.loli.net/2019/10/23/gX7Det2yb9HFU4W.png)

   例如上图中描述的，只有3个slave全都写完relay log并返回ACK给master后，master才会判断此次DDL/DML成功。

4. 延迟复制

   顾名思义，延迟复制就是故意让slave延迟一段时间再从master上进行复制。

#### binlog记录格式

MySQL 主从复制有三种方式：基于SQL语句的复制（statement-based replication，SBR），基于行的复制（row-based replication，RBR)，混合模式复制（mixed-based replication,MBR)。对应的binlog文件的格式也有三种：STATEMENT,ROW,MIXED。

- Statement-base Replication (SBR)就是记录sql语句在bin log中，Mysql 5.1.4 及之前的版本都是使用的这种复制格式。优点是只需要记录会修改数据的sql语句到binlog中，减少了binlog日质量，节约I/O，提高性能。缺点是在某些情况下，会导致主从节点中数据不一致（比如sleep(),now()等）。
- Row-based Relication(RBR)是mysql master将SQL语句分解为基于Row更改的语句并记录在bin log中，也就是只记录哪条数据被修改了，修改成什么样。优点是不会出现某些特定情况下的存储过程、或者函数、或者trigger的调用或者触发无法被正确复制的问题。缺点是会产生大量的日志，尤其是修改table的时候会让日志暴增,同时增加bin log同步时间。也不能通过bin log解析获取执行过的sql语句，只能看到发生的data变更。
- Mixed-format Replication(MBR)，MySQL NDB cluster 7.3 和7.4 使用的MBR。是以上两种模式的混合，对于一般的复制使用STATEMENT模式保存到binlog，对于STATEMENT模式无法复制的操作则使用ROW模式来保存，MySQL会根据执行的SQL语句选择日志保存方式。

#### GTID复制模式

在传统的复制里面，当发生故障，需要主从切换，需要找到binlog和pos点，然后将主节点指向新的主节点，相对来说比较麻烦，也容易出错。在MySQL 5.6里面，不用再找binlog和pos点，我们只需要知道主节点的ip，端口，以及账号密码就行，因为复制是自动的，MySQL会通过内部机制GTID自动找点同步。

多线程复制（基于库），在MySQL 5.6以前的版本，slave的复制是单线程的。一个事件一个事件的读取应用。而master是并发写入的，所以延时是避免不了的。唯一有效的方法是把多个库放在多台slave，这样又有点浪费服务器。在MySQL 5.6里面，我们可以把多个表放在多个库，这样就可以使用多线程复制。

**基于GTID复制实现的工作原理**

- 主节点更新数据时，会在事务前产生GTID，一起记录到binlog日志中。
- 从节点的I/O线程将变更的bin log，写入到本地的relay log中。
- SQL线程从relay log中获取GTID，然后对比本地binlog是否有记录（所以MySQL从节点必须要开启binary log）。
- 如果有记录，说明该GTID的事务已经执行，从节点会忽略。
- 如果没有记录，从节点就会从relay log中执行该GTID的事务，并记录到bin log。
- 在解析过程中会判断是否有主键，如果没有就用二级索引，如果有就用全部扫描。

#### 小结

Mysql 主从复制是mysql 高可用，高性能的基础，有了这个基础，mysql 的部署会变得简单、灵活并且具有多样性，从而可以根据不同的业务场景做出灵活的调整。

下面开始实践课

### 实践

使用docker来模拟实验，也可以使用虚拟机

#### 主从复制

此处先配置默认的异步复制模式。由于复制和binlog息息相关，如果对binlog还不熟悉，请先了解binlog，见：[详细分析二进制日志](http://www.cnblogs.com/f-ck-need-u/p/9001061.html#blog5)。

mysql支持一主一从和一主多从。但是每个slave必须只能是一个master的从，否则从多个master接受二进制日志后重放将会导致数据混乱的问题。

以下是一主一从的结构图：

![UTOOLS1571836122080.png](https://i.loli.net/2019/10/23/c9MZjBlunOdwVNR.png)

在开始传统的复制(非GTID复制)前，需要完成以下几个关键点，**这几个关键点指导后续复制的所有步骤**。

1. 为master和slave设定不同的`server-id`，这是主从复制结构中非常关键的标识号。到了MySQL 5.7，似乎不设置server id就无法开启binlog。设置server id需要重启MySQL实例。
2. 开启master的binlog。刚安装并初始化的MySQL默认未开启binlog，建议手动设置binlog且为其设定文件名，否则默认以主机名为基名时修改主机名后会找不到日志文件。
3. 最好设置master上的变量`sync_binlog=1`(MySQL 5.7.7之后默认为1，之前的版本默认为0)，这样每写一次二进制日志都将其刷新到磁盘，让slave服务器可以尽快地复制。防止万一master的二进制日志还在缓存中就宕机时，slave无法复制这部分丢失的数据。
4. 最好设置master上的redo log的刷盘变量`innodb_flush_log_at_trx_commit=1`(默认值为1)，这样每次提交事务都会立即将事务刷盘保证持久性和一致性。
5. 在slave上开启中继日志relay log。这个是默认开启的，同样建议手动设置其文件名。
6. 建议在master上专门创建一个用于复制的用户，它只需要有复制权限`replication slave`用来读取binlog。
7. 确保slave上的数据和master上的数据在"复制的起始position之前"是完全一致的。如果master和slave上数据不一致，复制会失败。
8. 记下master开始复制前binlog的position，因为在slave连接master时需要指定从master的哪个position开始复制。
9. 考虑是否将slave设置为只读，也就是开启`read_only`选项。这种情况下，除了具有super权限(mysql 5.7.16还提供了`super_read_only`禁止super的写操作)和SQL线程能写数据库，其他用户都不能进行写操作。这种禁写对于slave来说，绝大多数场景都非常适合。



两个docker容器，每个容器一个MySQL8.0，一个为master，IP为172.17.0.2；另一个为slave，IP为172.17.0.3

```shell
$ docker pull mysql:laset
#master mysql,root密码root
$ docker run --privileged --name master -e MYSQL_ROOT_PASSWORD=root  -d mysql
$ docker exec -it master bash
#进入容器
$ mysql -u root -p
mysql > CREATE USER 'hyp'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
mysql > GRANT REPLICATION SLAVE  ON *.* TO 'hyp'@'%';
mysql > exit
$ echo "log-bin=mysql-bin" >> /etc/mysql/my.cnf
$ echo "server-id=1" >> /etc/mysql/my.cnf 
$ exit
#退出容器
$ docker restart master
$ docker exec -it master bash
$ mysql -u root -p
#记好File名和Position
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      155 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)


#slave mysql
$ docker run --privileged --name slave1 -e MYSQL_ROOT_PASSWORD=root  -d mysql
$ docker exec -it slave1 bash
#进入容器
$ echo "log-bin=mysql-bin" >> /etc/mysql/my.cnf
$ echo "server-id=1" >> /etc/mysql/my.cnf 
$ exit
#退出容器
$ docker restart slave1
$ docker exec -it slave1 bash
$ mysql -u root -p
mysql>CHANGE MASTER TO MASTER_HOST='172.17.0.2',MASTER_USER='hyp',MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000002',MASTER_LOG_POS=155;
#完成主从复制配置
#测试
mysql>START SLAVE;   #开启复制
mysql>SHOW SLAVE STATUS\G   #查看主从复制是否配置成功
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 172.17.0.2
                  Master_User: hyp
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 155
               Relay_Log_File: 4dd31d771ba3-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Connecting
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
          Exec_Master_Log_Pos: 155
              Relay_Log_Space: 155
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'hyp@172.17.0.2:3306' - retry-time: 60 retries: 1 message: Can't connect to MySQL server on '172.17.0.2' (111)
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 191023 11:39:55
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set (0.01 sec)
   #查看主从复制是否配置成功

```

当看到Slave_IO_Running: YES、Slave_SQL_Running: YES才表明状态正常

可以登录master MySQL，创建库、表、更改数据，然后再登录slave MySQL，可以查看到数据跟master MySQL相同。

#### 主主复制

主主复制即在两台MySQL主机内都可以变更数据，而且另外一台主机也会做出相应的变更。聪明的你也许已经想到该怎么实现了。对，就是将两个主从复制有机合并起来就好了。只不过在配置的时候我们需要注意一些问题，例如，主键重复，server-id不能重复等等。

两个docker容器，每个容器一个MySQL8.0，一个为master01，IP为172.17.0.2；另一个为master02，IP为172.17.0.3

**配置文件**

在配置文件`/etc/mysql/my.cnf`最后添加，方法参看下节

```
#172.17.0.2
server-id=11   #任意自然数n，只要保证两台MySQL主机不重复就可以了。
log-bin=mysql-bin   #开启二进制日志
auto_increment_increment=2   #步进值auto_imcrement。一般有n台主MySQL就填n
auto_increment_offset=1   #起始值。一般填第n台主MySQL。此时为第一台主MySQL
binlog-ignore=mysql   #忽略mysql库【我一般都不写】
binlog-ignore=information_schema   #忽略information_schema库【我一般都不写】
#replicate-do-db=aa   #要同步的数据库，默认所有库

#172.17.0.3
server-id=12
log-bin=mysql-bin
auto_increment_increment=2
auto_increment_offset=2
#replicate-do-db=aa
```

**构建步骤**

配置完成后两个MySQL会同步数据库aa，可以在之后进行测试

```shell
#master01 mysql,root密码root
$ docker run --privileged --name master01 -e MYSQL_ROOT_PASSWORD=root  -d mysql
$ docker exec -it master01 bash
#进入容器
$ mysql -u root -p
mysql > CREATE USER 'hyp'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
mysql > GRANT REPLICATION SLAVE  ON *.* TO 'hyp'@'%';
mysql > exit
$ cat >> /etc/mysql/my.cnf <<-EOF
#末尾添加如下内容
server-id=11
log-bin=mysql-bin
auto_increment_increment=2
auto_increment_offset=1
replicate-do-db=aa
EOF
$ exit
#退出容器
$ docker restart master01
$ docker exec -it master01 bash
$ mysql -u root -p
#记好File名和Position,用于master02的配置
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      155 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='172.17.0.3',MASTER_USER='hyp',MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=155;
mysql> exit
$ exit

#master02 mysql
$ docker run --privileged --name master02 -e MYSQL_ROOT_PASSWORD=root  -d mysql
$ docker exec -it master02 bash
#进入容器
$ mysql -u root -p
mysql > CREATE USER 'hyp'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
mysql > GRANT REPLICATION SLAVE  ON *.* TO 'hyp'@'%';
mysql > exit
$ cat >> /etc/mysql/my.cnf <<-EOF
#末尾添加如下内容
server-id=12
log-bin=mysql-bin
auto_increment_increment=2
auto_increment_offset=2
replicate-do-db=aa
EOF
$ exit
#退出容器
$ docker restart master02
$ docker exec -it master02 bash
$ mysql -u root -p
#记好File名和Position,用于master01的配置
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      155 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
mysql> CHANGE MASTER TO MASTER_HOST='172.17.0.2',MASTER_USER='hyp',MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=155;
mysql> exit
$ exit



#测试主主复制
#分别开启start slave；
#master01
mysql>START SLAVE;   #开启复制
mysql>SHOW SLAVE STATUS\G   #查看主从复制是否配置成功
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.3
                  Master_User: hyp
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 155
               Relay_Log_File: 74339ff05e94-relay-bin.000002
                Relay_Log_Pos: 322
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: aa
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 155
              Relay_Log_Space: 537
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
             Master_Server_Id: 11
                  Master_UUID: 5b81bd04-f591-11e9-a255-0242ac110003
             Master_Info_File: mysql.slave_master_info
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
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set (0.00 sec)

#master02
$ mysql -u root -p
mysql>START SLAVE;   #开启复制
mysql>SHOW SLAVE STATUS\G   #查看主从复制是否配置成功
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: 172.17.0.2
                  Master_User: hyp
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 155
               Relay_Log_File: 74339ff05e94-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Connecting
            Slave_SQL_Running: Yes
              Replicate_Do_DB: aa
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 155
              Relay_Log_Space: 155
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 2003
                Last_IO_Error: error connecting to master 'hyp@172.17.0.2:3306' - retry-time: 60 retries: 1 message: Can't connect to MySQL server on '172.17.0.2' (111)
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 191023 12:38:11
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
       Master_public_key_path: 
        Get_master_public_key: 0
            Network_Namespace: 
1 row in set (0.00 sec)

```

**注意事项**

1. 主主复制配置文件中auto_increment_increment和auto_increment_offset只能保证主键不重复，却不能保证主键有序。

2. 当配置完成Slave_IO_Running、Slave_SQL_Running不全为YES时，show slave status\G信息中有错误提示，可根据错误提示进行更正。

3. Slave_IO_Running、Slave_SQL_Running不全为YES时，大多数问题都是数据不统一导致。

常见出错点：

1. 两台数据库都存在db数据库，而第一台MySQL db中有tab1，第二台MySQL db中没有tab1，那肯定不能成功。

2. 已经获取了数据的二进制日志名和位置，又进行了数据操作，导致POS发生变更。在配置CHANGE MASTER时还是用到之前的POS。

3. stop slave后，数据变更，再start slave。出错。

 终极更正法：重新执行一遍CHANGE MASTER就好了。

#### 常见问题

1. 读写分离

   - 数据复制延迟；
   - 读到过期数据；
   - 从节点故障；

2. 主从配置不一样

   - maxmemory 不一致；
   - 数据结构优化参数，比如：`hash-max-ziplist-entries`，主节点优化了，从节点没优化，导致的内存不一致问题；

3. 规避全量复制

   - 第一次连接到 master 上，无法避免全量复制；

   - 设置小主节点；

   - 在低峰做全量复制；

   - 节点运行 ID 不匹配，比如主节点重启（运行 ID 变化）导致的全量复制，可用故障转移来解决，例如**哨兵**或**集群**；

   - 复制积压缓冲区（repl_back_buffer）不足，可以通过配置其大小 `rel_backlog_size` 来解决；

4. 规避复制风暴

   - 单主节点复制风暴：主节点重启，多从节点复制，可通过变更复制拓扑解决；

   - 单机器复制风暴：一台机器上有多个 master，机器重启后，所有的 master 都会进行全量复制，可以把 master 分布在多个机器上解决，更好的办法是高可用方案，master 挂掉，slave 晋升为 master；

#### 参考

1. [深度探索MySQL主从复制原理](https://baijiahao.baidu.com/s?id=1617888740370098866&wfr=spider&for=pc)
2. [MySQL复制(一)：复制理论和传统复制的配置](https://www.cnblogs.com/f-ck-need-u/p/9155003.html)
3. [MySQL复制(二)：基于GTID复制](https://www.cnblogs.com/f-ck-need-u/p/9164823.html)
4. [MySQL复制(三)：半同步复制](https://www.cnblogs.com/f-ck-need-u/p/9166452.html)