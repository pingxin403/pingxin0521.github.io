---
title: Redis 主从复制
date: 2019-10-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

Redis主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布 记录。同步对读取操作的可扩展性和数据冗余很有帮助。

<!--more-->

**单机有什么问题**？

单机即在一台机器上部署一个redis节点，主要会存在以下问题：

1. 机器故障

   如果发生机器故障，例如磁盘损坏，主板损坏等，未能在短时间内修复好，客户端将无法连接redis。
    当然如果仅仅是redis节点挂掉了，可以进行问题排查然后重启，姑且不考虑这段时间对外服务的可用性，那

   还是可以接受的。
    而发生机器故障，基本是无济于事。除非把redis迁移到另一台机器上，并且还要考虑数据同步的问题。

2. 容量瓶颈

   假如一台机器是16G内存，redis使用了12G内存，而其他应用还需要使用内存，假设我们总共需要60G内存要如何去做呢，是否有必要购买64G内存的机器？

3. QPS瓶颈

   redis官方数据显示可以达到10w的QPS，如果业务需要100w的QPS怎么去做呢？

   关于容量瓶颈和QPS瓶颈是redis分布式需要解决的问题，而机器故障就是高可用的问题了。

**Redis Replication的特点和优势**

下面的列表清楚的解释了Redis Replication的特点和优势。

1. 同一个Master可以同步多个Slaves。
2. Slave同样可以接受其它Slaves的连接和同步请求，这样可以有效的分载Master的同步压力。因此我们可以将Redis的Replication架构视为图结构。
3.  Master Server是以非阻塞的方式为Slaves提供服务。所以在Master-Slave同步期间，客户端仍然可以提交查询或修改请求。
4.  Slave Server同样是以非阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis则返回同步之前的数据。
5. 为了分载Master的读操作压力，Slave服务器可以为客户端提供只读操作的服务，写服务仍然必须由Master来完成。即便如此，系统的伸缩性还是得到了很大的提高。
6.  Master可以将数据保存操作交给Slaves完成，从而避免了在Master中要有独立的进程来完成此操作。

**主从复制的作用**

1. 数据副本(备份)
2. 扩展读性能(读写分离)

**简单总结**

1. 一个master可以有多个slave
2. 一个slave只能有一个master
3. 数据流向是单向的，master到slave

#### 主从架构

<img src="https://s2.ax1x.com/2019/11/28/QFaiPU.png" alt="QFaiPU.png" style="zoom:67%;" />

1. 读写分离

   在redis主从架构中，Master节点负责处理写请求，Slave节点只处理读请求。对于写请求少，读请求多的场景，例如电商详情页，通过这种读写分离的操作可以大幅提高并发量，通过增加redis从节点的数量可以使得redis的QPS达到10W+。

2. 主从同步

   Master节点接收到写请求并处理后，需要告知Slave节点数据发生了改变，保持主从节点数据一致的行为称为主从同步，所有的Slave都和Master通信去同步数据也会加大Master节点的负担，实际上，除了主从同步，redis也可以从从同步，我们在这里统一描述为主从同步。

   ![QFalGD.png](https://s2.ax1x.com/2019/11/28/QFalGD.png)

#### 工作原理：

Redis的主从结构可以采用一主多从或者级联结构，Redis主从复制可以根据是否是全量分为全量同步和增量同步。

1. 全量同步

   全量复制主节点会将RDB文件也就是当前状态去同步给slave，在此期间主新写入的命令会单独记录起来，然后当RDB文件加载完毕之后，会通过偏移量对比将这个期间产生的写入值同步给slave，这样就能达到数据完全同步的效果

   Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份。具体步骤如下： 

   - 在其内部有一条命令`psync`，是做同步的命令，它可以完成全量复制和部分复制的功能，当启动slave节点时，它会发送`psync`命令给主节点，需要传递两个参数，`runid`和`offset`(偏移量)，也就是从向主传递主节点的runid以及自己的偏移量，对于第一次复制而言，就直接传递？和 -1，当然这个参数是由slave内部传的。

   - master接收到命令后知道从希望做全量复制，主就会将自己的runid和offset传递给从

   - slave节点保存master的基本信息

   - master执行`bgsave`生成RDB文件，并且在此期间新产生的写入命令会被记录到`repl_back_buffer`(复制缓冲区)

   - 主向从传输RDB文件

   - 主向从发送复制缓冲区内容

   - 清空从节点旧的数据

   - 从节点加载RDB文件到内存中，同时加载缓冲区数据

   完成上面几个步骤后就完成了从服务器数据初始化的所有操作，从服务器此时可以接收来自用户的读请求。

   实际上全量复制的开销是非常大的，主要体现在如下方面

   - bgsave时间(对cpu、 内存、硬盘都会有一定的开销)
   - RDB文件网络传输时间(网络带宽)
   - 从节点清空数据时间(根据从节点的数据规模)
   - 从节点加载RDB的时间
   - 可能的AOF重写时间(在最后从加载完RDB之后如果开启了AOF，会做AOF重写)

   全量复制除了上述开销之外，还会有个问题：

   假如master和slave网络发生了抖动，那一段时间内这些数据就会丢失，对于slave来说这段时间master更新的数据是不知道的。最简单的方式就是再做一次全量复制，从而获取到最新的数据，在redis2.8之前是这么做的。

   ![QF2XkQ.png](https://s2.ax1x.com/2019/11/28/QF2XkQ.png)

2. 增量同步

   Redis增量复制是指Slave初始化后开始正常工作时主服务器发生的写操作同步到从服务器的过程。 

   增量复制的过程主要是主服务器每执行一个写命令就会向从服务器发送相同的写命令，从服务器接收并执行收到的写命令。

   部分复制，redis2.8之后提供。如果发生类似抖动时候，可以有一种机制将这种损失降低到最低，如何实现的？

   - 如果发生了抖动，相当于连接断开了
   - 主会将写命令记录到缓冲区，repl_back_buffer
   - 当slave再次去连接master时候，就是说网络抖动结束之后，会触发增量复制
   - 从会执行pysnc命令，将当前自己的offset和主的runid传递给master
   - 如果发现传输的offset偏移量是在buffer内的，不在期间内就证明你已经错过了很多数据，buffer也是有限的，默认是1M，会将offset开始到队列结束的数据同步给从。这样master和slave就达到了一致。

   通过部分复制(增量复制)有效的降低了全量复制的开销。

**Redis主从同步策略**

 主从刚刚连接的时候，进行全量同步；全同步结束后，进行增量同步。当然，如果有需要，slave 在任何时候都可以发起全量同步。redis 策略是，无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步

**注意点**

如果多个Slave断线了，需要重启的时候，因为只要Slave启动，就会发送sync请求和主机全量同步，当多个同时出现的时候，可能会导致Master IO剧增宕机。

#### 主从同步的详细流程

1. 在从节点的配置文件中的`slaveof`配置项中配置了主节点的IP和port后，从节点就知道自己要和那个主节点进行连接了。

2. 从节点内部有个定时任务，会每秒检查自己要连接的主节点是否上线，如果发现了主节点上线，就跟主节点进行网络连接。注意，此时仅仅是取得连接，还没有进行主从数据同步。

3. 从节点发送ping命令给主节点进行连接，如果设置了口令认证（主节点设置了requirepass），那么从节点必须发送正确的口令（masterauth）进行认证。

4. 主从节点连接成功后，主从节点进行一次快照同步。事实上，是否进行快照同步需要判断主节点的`run id`，当从节点发现已经连接过某个`run id`的主节点，那么视此次连接为重新连接，就不会进行快照同步。相同IP和port的主节点每次重启服务都会生成一个新的`run id`，所以每次主节点重启服务都会进行一次快照同步，如果想重启主节点服务而不改变`run id`，使用`redis-cli debug reload`命令。

5. 当开始进行快照同步后，主节点在本地生成一份rdb快照文件，并将这个rdb文件发送给从节点，如果复制时间超过60秒（配置项：repl-timeout），那么就会认为复制失败，如果数据量比较大，要适当调大这个参数的值。主从节点进行快照同步的时候，主节点会把接收到的新请求命令写在缓存 buffer 中，当快照同步完成后，再把 buffer 中的指令增量同步到从节点。如果在快照同步期间，内存缓冲区大小超过256MB，或者超过64MB的状态持续时间超过60s（配置项：`client-output-buffer-limit slave 256MB 64MB 60`），那么也会认为快照同步失败。

6. 从节点接收到RDB文件之后，清空自己的旧数据，然后重新加载RDB到自己的内存中，在这个过程中基于旧的数据对外提供服务。如果主节点开启了AOF，那么在快照同步结束后会立即执行BGREWRITEAOF，重写AOF文件。

7. 主节点维护了一个backlog文件，默认是1MB大小，主节点向从节点发送全量数据（RDB文件）时，也会同步往backlog中写，这样当发送全量数据这个过程意外中断后，从backlog文件中可以得知数据有哪些是发送成功了，哪些还没有发送，然后当主从节点再次连接后，从失败的地方开始增量同步。这里需要注意的是，当快照同步连接中断后，主从节点再次连接并非是第一次连接，所以进行增量同步，而不是继续进行快照同步。

8. 快照同步完成后，主节点后续接收到写请求导致数据变化后，将和从节点进行增量同步，遇到 buffer 溢出则再触发快照同步。

9. 主从节点都会维护一个offset，随着主节点的数据变化以及主从同步的进行，主从节点会不断累加自己维护的offset，从节点每秒都会上报自己的offset给主节点，主节点也会保存每个从节点的offset，这样主从节点就能知道互相之间的数据一致性情况。从节点发送`psync runid offset`命令给主节点从而开始主从同步，主节点会根据自身的情况返回响应信息，可能是FULLRESYNC runid offset触发全量复制，也可能是CONTINUE触发增量复制。

10. 主从节点因为网络原因导致断开，当网络接通后，不需要手工干预，可以自动重新连接。

11. 主节点如果发现有多个从节点连接，在快照同步过程中仅仅会生成一个RDB文件，用一份数据服务所有从节点进行快照同步。

12. 从节点不会处理过期key，当主节点处理了一个过期key，会模拟一条del命令发送给从节点。

13. 主从节点会保持心跳来检测对方是否在线，主节点默认每隔10秒发送一次heartbeat，从节点默认每隔1秒发送一个heartbeat。

14. 建议在主节点使用AOF+RDB的持久化方式，并且在主节点定期备份RDB文件，而从节点不要开启AOF机制，原因有两个，一是从节点AOF会降低性能，二是如果主节点数据丢失，主节点数据同步给从节点后，从节点收到了空的数据，如果开启了AOF，会生成空的AOF文件，基于AOF恢复数据后，全部数据就都丢失了，而如果不开启AOF机制，从节点启动后，基于自身的RDB文件恢复数据，这样不至于丢失全部数据。

#### redis 主从模式配置

本示例：一个master节点有两个slave节点

在同一主机上，使用端口不同

```shell
$ vim ./conf/master-6739.conf

bind 0.0.0.0
port 6379
dir /tmp
logfile "6379.log"
dbfilename "dump-6379.rdb"
daemonize yes
rdbcompression yes
# 开启无磁盘化复制
repl-diskless-sync yes
# 当收到第一个复制请求时，等待 5s 后再开始复制，因为要等更多 slave 一起连接过来
repl-diskless-sync-delay 5
#如果在复制期间，rdb复制时间超过60秒，内存缓冲区持续消耗超过64MB，或者一次性超过256MB，那么将会停止复制(失败)
client-output-buffer-limit slave 256MB 64MB 60

$ vi ./conf/slave-6380.conf

bind 0.0.0.0
port 6380
dir /tmp
logfile "6380.log"
dbfilename "dump-6380.rdb"
daemonize yes
rdbcompression yes
# 配置主节点的IP和端口号
slaveof 172.16.91.88 6379
 # 从节点只做读的操作，保证主从数据的一致性
slave-read-only yes 

$ vi ./conf/slave-6381.conf

bind 0.0.0.0
port 6381
dir /tmp
logfile "6381.log"
dbfilename "dump-6381.rdb"
daemonize yes
rdbcompression yes
# 配置主节点的IP和端口号
slaveof 172.16.91.88 6379
 # 从节点只做读的操作，保证主从数据的一致性
slave-read-only yes 
```

master-6739.conf，为主节点配置文件，slave-6380.conf，slave-6381.conf为从节点配置文件
在从节点的配置文件中使用：slaveof 指定master节点

也可以在redis-server运行时，添加`--slaveof=<server ip> <server port>`

**注意**

> 起始版本：5.0.0，REPLICAOF替换saveof，但saveof依然可以用

命令`REPLICAOF` 可以在线修改当前服务器的复制设置

如果当前服务器已经是副本服务器，命令`REPLIACOF` NO ONE 会关闭当前服务器的复制并转变为主服务器。执行 `REPLIACOF` hostname port 会将当前服务器转变为某一服务器的副本服务器

如果当前服务器已经是某个主服务器(master server)的副本服务器，那么执行 REPLICAOF hostname port 将使当前服务器停止对原主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步

对一个副本服务器执行命令 REPLICAOF NO ONE 将使得这个副本服务器关闭复制，并从副本服务器转变回主服务器，原来同步所得的数据集不会被丢弃。因此，当原主服务器停止服务，可以将该副本服务器切换为主服务器，应用可以使用新主服务器进行读写。原主服务器修复后，可将其设置为新主服务器的副本服务器。

**启动三台reids服务**

```shell
$ ./bin/redis-server ./conf/master-6739.conf &
$ ./bin/redis-server ./conf/slave-6381.conf &
$ ./bin/redis-server ./conf/slave-6380.conf &
$ ps -ef | grep redis
root     11980     1  0 22:32 ?        00:00:00 ./bin/redis-server 0.0.0.0:6380
root     11986     1  0 22:33 ?        00:00:00 ./bin/redis-server 0.0.0.0:6381
root     12000     1  0 22:35 ?        00:00:00 ./bin/redis-server 0.0.0.0:6379
```

**测试主从模式**：

先分别连上三台Redis服务，获取key为name的值，通过-p 指定连接那个端口的redis服务

```
127.0.0.1:6379> get name
(nil)

127.0.0.1:6380> get name
(nil)


127.0.0.1:6381> get name
(nil)

```

给master节点set一个key

```
127.0.0.1:6379> set name pingxin
OK
127.0.0.1:6379> get name
"pingxin"
```

slave节点直接读取key为name的值

```
127.0.0.1:6380> get name
"pingxin"

127.0.0.1:6381> get name
"pingxin"

```

slave节点只提供读服务，不能进行写入操作

```
127.0.0.1:6381> set name hyp
(error) READONLY You can't write against a read only replica.
```

**常用命令**

```shell
#查看状态
#主
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.16.91.88,port=6381,state=online,offset=1151,lag=0
slave1:ip=172.16.91.88,port=6380,state=online,offset=1137,lag=1
master_replid:493a371e34f2e47eeeb99561be8b1903b4e7ed67
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1151
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1151
#从
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:172.16.91.88
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:1039
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:493a371e34f2e47eeeb99561be8b1903b4e7ed67
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1039
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1039

#断开主从，在salve节点
127.0.0.1:6380> slaveof no one
OK
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:0
master_replid:6bce06802066e7bd7738f98680615b5104155ffc
master_replid2:493a371e34f2e47eeeb99561be8b1903b4e7ed67
master_repl_offset:1235
second_repl_offset:1236
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1235

#断开后从重连
127.0.0.1:6380> slaveof 172.16.91.88 6379
OK
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:172.16.91.88
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_repl_offset:1347
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:493a371e34f2e47eeeb99561be8b1903b4e7ed67
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1347
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1348
repl_backlog_histlen:0

#主从复制时使用密码验证
requirepass
#从节点建议使用只读模式slave-read-only=yes,弱从节点修改数据，主从数据不一致
```

##### 注意事项

1. 传输延迟：主从节点如果不是部署在同一台机器上，那么复制时就会产生网络延迟问题。Redis 提供了 repl-disable-tcp-nodelay 参数，用于控制与 Linux 的配置选项 TCP_NODELAY 。

   disable 本身的含义是“使……失去能力；禁用”，那么这个参数的含义就是是否禁用 TCP_NODELAY ，参数值为 yes 和 no。

   > 禁用 TCP_NODELAY：no ，不禁用，双重否定等于肯定，即启用 ，TCP_NODELAY=on；
   > 禁用 TCP_NODELAY：yes，就是禁用, TCP_NODELAY=off。

   启用 TCP_NODELAY 时，无网络延迟，主服务器的命令无论大小都会及时发送给从节点，此时会增加网络带宽的消耗。适合主从网络环境良好的场景。

   禁用 TCP_NODELAY 时，主从节点的通信会受到网络拥塞控制，Linux 内核会做出一定的优化，如合并较小的 TCP 数据包从而节省带宽。默认发送时间间隔取决 Linux 内核，一般默认为 40 毫秒。适合跨机房部署的情况。

2. 使用主从模式，在master上关掉所有持久化，在slave上使用AOF持久化

   ```properties
   ######Master config
   ###General 配置
   daemonize yes    #使用daemon 方式运行程序，默认为非daemon方式运行
   pidfile /tmp/redis.pid  #pid文件位置
   port 6379  #使用默认端口
   timeout 30  # client 端空闲断开连接的时间
   loglevel warning  #日志记录级别，默认是notice，我这边使用warning,是为了监控日志方便。使用warning后，只有发生告警才会产生日志，这对于通过判断日志文件是否为空来监控报警非常方便。logfile /opt/logs/redis/redis.log  #日志产生的位置
   databases 16  #默认是0，也就是只用1 个db,我这边设置成16，方便多个应用使用同一个redis server。使用select n 命令可以确认使用的redis db ,这样不同的应用即使使用相同的key也不会有问题。
   
   ###下面是SNAPSHOTTING持久化方式的策略。为了保证数据相对安全，在下面的设置中，更改越频繁，SNAPSHOTTING越频繁，也就是说，压力越大，反而花在持久化上的资源会越多。所以我选择了master-slave模式，并在master关掉了SNAPSHOTTING。
   #save 900 1    #在900秒之内，redis至少发生1次修改则redis抓快照到磁盘
   #save 300 100  #在300秒之内，redis至少发生100次修改则redis抓快照到磁盘
   #save 60 10000  #在60秒之内，redis至少发生10000次修改则redis抓快照到磁盘
   rdbcompression yes  #使用压缩
   dbfilename dump.rdb  #SNAPSHOTTING的文件名
   dir /opt/data/redis/ #SNAPSHOTTING文件的路径
   
   ###REPLICATION 设置，
   #slaveof#如果这台机器是台redis slave，可以打开这个设置。如果使用master-slave模式，我就会在master上把SNAPSHOTTING关了，这样可以不用在master上做持久化，而是在slave上做，这样可以大大提高master 内存使用率和系统性能。
   #slave-serve-stale-data yes  #如果slave 无法与master 同步，是否还可以读
   
   ### SECURITY 设置 
   #requirepass aaaaaaaaa  #redis性能太好，用个passwd 意义不大
   #rename-command FLUSHALL ""  #可以用这种方式关掉非常危险的命令，如FLUSHALL这个命令，它清空整个 Redis 服务器的数据，而且不用确认且从不会失败###LIMIT 设置
   maxclients 0 #无client连接数量限制
   maxmemory 14gb #redis最大可使用的内存量，我的服务器内存是16G，如果使用redis SNAPSHOTTING的copy-on-write的持久会写方式，会额外的使用内存，为了使持久会操作不会使用系统VM，使redis服务器性能下降，建议保留redis最大使用内存的一半8G来留给持久化使用，我个人觉得非常浪费。我没有在master上不做持久化，使用主从方式maxmemory-policy volatile-lru  #使用LRU算法删除设置了过期时间的key,但如果程序写的时间没有写key的过期时间，建议使用allkeys-lru，这样至少保证redis不会不可写入。###APPEND ONLY MODE 设置
   appendonly no  #不使用AOF，AOF是另一种持久化方式，我没有使用的原因是这种方式并不能在服务器或磁盘损坏的情况下，保证数据可用性。
   appendfsync everysec 
   no-appendfsync-on-rewrite no
   auto-aof-rewrite-percentage 100
   auto-aof-rewrite-min-size 64mb
   
   ###SLOW LOG 设置
   slowlog-log-slower-than 10000  #如果操作时间大于0.001秒，记录slow log,这个log是记录在内存中的，可以用redis-cli slowlog get 命令查看slowlog-max-len 1024  #slow log 的最大长度
   
   ###VIRTUAL MEMORY 设置
   vm-enabled no  #不使用虚拟内存，在redis 2.4版本，作者已经非常不建议使用VM。
   vm-swap-file /tmp/redis.swap
   vm-max-memory 0
   vm-page-size 32
   vm-pages 134217728
   vm-max-threads 4
   
   ###ADVANCED CONFIG 设置，下面的设置主要是用来节省内存的，我没有对它们做修改
   hash-max-zipmap-entries 512 
   hash-max-zipmap-value 64
   list-max-ziplist-entries 512
   list-max-ziplist-value 64
   set-max-intset-entries 512
   zset-max-ziplist-entries 128
   zset-max-ziplist-value 64
   activerehashing yes
   
   ###INCLUDES 设置 ，使用下面的配置，可以配置一些个另其它的设置，如slave的配置
   #include /path/to/local.conf
   #include /path/to/other.conf
   #include /opt/redis/etc/slave.conf  如果是slave server,把这个注释打开slave 配置：
   $cat /opt/redis/etc/slave.conf
   
   ######slave config
   ###REPLICATION 设置，
   slaveof redis01 6397  #如果这台机器是台redis slave，可以打开这个设置。如果使用master-slave模式，我就会在master上把SNAPSHOTTING关了，这样可以不用在master上做持久化，而是在slave上做，这样可以大大提高master 内存使用率和系统性能。
   slave-serve-stale-data no  #如果slave 无法与master 同步，设置成slave不可读，方便监控脚本发现问题。
   ###APPEND ONLY MODE 设置
   appendonly yes  #在slave上使用了AOF,以保证数据可用性。其它后继数据备份工作
   ```

   - 用redis-cli bgsave 命令每天凌晨一次持久化一次master redis上的数据，并CP到其它备份服务器上。
   - 用redis-cli bgrewriteaof 命令每半小时持久化一次 slave redis上的数据，并CP到其它备份服务器上。
   - 写个脚本 ，定期get master和slave上的key,看两个是否同步，如果没有同步，及时报警。

### 参考

1. [Redis 主从配置实例、注意事项、及备份方式](https://www.duoluosb.com/666.html?btwaf=79487282)
2. [Redis主从复制](https://www.jianshu.com/p/ba3cc187da9c)
3. [Linux下 Redis主从架构持久化操作详述](https://www.duoluosb.com/668.html?btwaf=29904111)