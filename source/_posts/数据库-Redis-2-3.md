---
title: Redis 哨兵模式
date: 2019-10-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### Sentinel（哨兵模式）

Sentinel(哨兵)是用于监控redis集群中Master状态的工具，其已经被集成在redis2.4+的版本中

<!--more-->

#### 主从复制的缺点

首先回顾一下主从复制，主要有两个作用，第一为主节点提供数据备份，当主节点宕机时，从节点会有数据的完整副本。第二个就是为主节点提供读的分流，实现读写分离，可以减轻主节点的压力。

但是呢单使用主从复制模型会存在以下问题：

1. 手动故障转移
   就是一旦主节点出现故障，那么故障转移基本上是需要手工来完成的。
2. 写能力和存储能力受限
   写只能写在一个节点上，而且存储也是在一个节点上进行存储。

**主从复制-master宕掉**

一主两从模型，当master发生宕机时，那么复制也必然断掉了，而从节点与主节点的连接肯定也是失败的，这样数据的读取是正常的，但是数据的更新就无法保障了。

**主从复制-master宕掉故障处理**

首先要选择一个slave执行命令 `slave no one`，使其成为master节点。然后对其余的slave执行 `slaveof new master`，这样就完成了新的master节点以及其从节点的构建过程。同时需要让客户端的写操作在新的master上进行。

整个过程都是完全手工进行的，即使不手工而采用编写脚本的方式，让脚本不断监控master是否有问题，有问题之后选择一个新的master，然后让其他的slave都指向新的master节点，最后再去迁移所有的客户端。单独写这么一个完美的组件，也是非常困难的。

为了弥补主从模型在高可用方面的不足，Redis为我们提供了 `Redis Sentinel` 这样一个高可用的实现。

#### Redis Sentinel架构

Redis Sentinel 是一个分布式系统， 你可以在一个架构中运行多个 Sentinel 进程（progress）， 这些进程使用流言协议（gossip protocols)来接收关于主服务器是否下线的信息， 并使用投票协议（agreement protocols）来决定是否执行自动故障迁移， 以及选择哪个从服务器作为新的主服务器。

虽然 Redis Sentinel 释出为一个单独的可执行文件 redis-sentinel ， 但实际上它只是一个运行在特殊模式下的 Redis 服务器， 你可以在启动一个普通 Redis 服务器时通过给定 –sentinel 选项来启动 Redis Sentinel 。

![Qk7vTI.png](https://s2.ax1x.com/2019/11/29/Qk7vTI.png)

**Sentinel作用**

- Master-Slave切换后，master_redis.conf、slave_redis.conf和sentinel.conf的内容都会发生改变，即master_redis.conf中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换
- **监控（Monitoring**）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

**Sentinel工作方式**

1. 每个Sentinel以每秒钟一次的频率向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令

2. 如果一个实例（instance）距离最后一次有效回复PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel 标记为主观下线

3. 如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。

4. 当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线

5. 在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令

6. 当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次

7. 若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除

   若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除

**故障转移处理**

1. 多个sentinel发现并确认master有问题
2. 选举出一个sentinel作为领导
3. 选出一个slave作为master
4. 通知其余slave成为新的master的slave
5. 通知客户端主从变化
6. 等待老的master复活成为新master的slave

**主观下线和客观下线**

主观下线：Subjectively Down，简称 SDOWN，指的是当前 Sentinel 实例对某个redis服务器做出的下线判断。

客观下线：Objectively Down， 简称 ODOWN，指的是多个 Sentinel 实例在对Master Server做出 SDOWN 判断，并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后，得出的Master Server下线判断，然后开启failover.

SDOWN适合于Master和Slave，只要一个 Sentinel 发现Master进入了ODOWN， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对下线的主服务器执行自动故障迁移操作。

ODOWN只适用于Master，对于Slave的 Redis 实例，Sentinel 在将它们判断为下线前不需要进行协商，所以Slave的 Sentinel 永远不会达到ODOWN。

如果一个服务器没有在 master-down-after-milliseconds 选项所指定的时间内， 对向它发送 PING 命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线。

服务器对 PING 命令的有效回复可以是以下三种回复的其中一种：

- 返回 +PONG 。
- 返回 -LOADING 错误。
- 返回 -MASTERDOWN 错误。

如果服务器返回除以上三种回复之外的其他回复， 又或者在指定时间内没有回复 PING 命令， 那么 Sentinel 认为服务器返回的回复无效（non-valid）。

**Sentinel的通信命令**

`Sentinel` 节点连接一个 `Redis` 实例的时候，会创建 `cmd` 和 `pub/sub` 两个 **连接**。`Sentinel` 通过 `cmd` 连接给 `Redis` 发送命令，通过 `pub/sub` 连接到 `Redis` 实例上的其他 `Sentinel` 实例。

`Sentinel` 与 `Redis` **主节点** 和 **从节点** 交互的命令，主要包括：

| 命令      | 作 用                                                        |
| --------- | ------------------------------------------------------------ |
| PING      | `Sentinel` 向 `Redis` 节点发送 `PING` 命令，检查节点的状态   |
| INFO      | `Sentinel` 向 `Redis` 节点发送 `INFO` 命令，获取它的 **从节点信息** |
| PUBLISH   | `Sentinel` 向其监控的 `Redis` 节点 `__sentinel__:hello` 这个 `channel` 发布 **自己的信息** 及 **主节点** 相关的配置 |
| SUBSCRIBE | `Sentinel` 通过订阅 `Redis` **主节点** 和 **从节点** 的 `__sentinel__:hello` 这个 `channnel`，获取正在监控相同服务的其他 `Sentinel` 节点 |

`Sentinel` 与 `Sentinel` 交互的命令，主要包括：

| 命令                            | 作 用                                                        |
| ------------------------------- | ------------------------------------------------------------ |
| PING                            | `Sentinel` 向其他 `Sentinel` 节点发送 `PING` 命令，检查节点的状态 |
| SENTINEL:is-master-down-by-addr | 和其他 `Sentinel` 协商 **主节点** 的状态，如果 **主节点** 处于 `SDOWN` 状态，则投票自动选出新的 **主节点** |

**Redis Sentinel的工作原理**

每个 `Sentinel` 节点都需要 **定期执行** 以下任务：

- 每个 `Sentinel` 以 **每秒钟** 一次的频率，向它所知的 **主服务器**、**从服务器** 以及其他 `Sentinel` **实例** 发送一个 `PING` 命令。

  ![QkHAmj.png](https://s2.ax1x.com/2019/11/29/QkHAmj.png)

- 如果一个 **实例**（`instance`）距离 **最后一次** 有效回复 `PING` 命令的时间超过 `down-after-milliseconds` 所指定的值，那么这个实例会被 `Sentinel` 标记为 **主观下线**。

  ![QkHQcF.png](https://s2.ax1x.com/2019/11/29/QkHQcF.png)

- 如果一个 **主服务器** 被标记为 **主观下线**，那么正在 **监视** 这个 **主服务器** 的所有 `Sentinel` 节点，要以 **每秒一次** 的频率确认 **主服务器** 的确进入了 **主观下线** 状态。

  ![QkHa9K.png](https://s2.ax1x.com/2019/11/29/QkHa9K.png)

- 如果一个 **主服务器** 被标记为 **主观下线**，并且有 **足够数量** 的 `Sentinel`（至少要达到 **配置文件** 指定的数量）在指定的 **时间范围** 内同意这一判断，那么这个 **主服务器** 被标记为 **客观下线**。

  ![QkH0je.png](https://s2.ax1x.com/2019/11/29/QkH0je.png)

- 在一般情况下， 每个 `Sentinel` 会以每 `10` 秒一次的频率，向它已知的所有 **主服务器** 和 **从服务器** 发送 `INFO` 命令。当一个 **主服务器** 被 `Sentinel` 标记为 **客观下线** 时，`Sentinel` 向 **下线主服务器** 的所有 **从服务器** 发送 `INFO` 命令的频率，会从 `10` 秒一次改为 **每秒一次**。

  ![QkHcNt.png](https://s2.ax1x.com/2019/11/29/QkHcNt.png)

- `Sentinel` 和其他 `Sentinel` 协商 **主节点** 的状态，如果 **主节点** 处于 `SDOWN` 状态，则投票自动选出新的 **主节点**。将剩余的 **从节点** 指向 **新的主节点** 进行 **数据复制**。

  ![QkH7En.png](https://s2.ax1x.com/2019/11/29/QkH7En.png)

- 当没有足够数量的 `Sentinel` 同意 **主服务器** 下线时， **主服务器** 的 **客观下线状态** 就会被移除。当 **主服务器** 重新向 `Sentinel` 的 `PING` 命令返回 **有效回复** 时，**主服务器** 的 **主观下线状态** 就会被移除。

  ![QkHLCV.png](https://s2.ax1x.com/2019/11/29/QkHLCV.png)

> 注意：一个有效的 `PING` 回复可以是：`+PONG`、`-LOADING` 或者 `-MASTERDOWN`。如果 **服务器** 返回除以上三种回复之外的其他回复，又或者在 **指定时间** 内没有回复 `PING` 命令， 那么 `Sentinel` 认为服务器返回的回复 **无效**（`non-valid`）。

#### 搭建

**Redis Sentinel的部署须知**

1. 一个稳健的 `Redis Sentinel` 集群，应该使用至少 **三个** `Sentinel` 实例，并且保证讲这些实例放到 **不同的机器** 上，甚至不同的 **物理区域**。
2. `Sentinel` 无法保证 **强一致性**。
3. 常见的 **客户端应用库** 都支持 `Sentinel`。
4. `Sentinel` 需要通过不断的 **测试** 和 **观察**，才能保证高可用。



![Qk7usH.png](https://s2.ax1x.com/2019/11/29/Qk7usH.png)

**步骤**

1. 配置开启主从节点
2. 配置开启sentinel监控主节点。(sentinel是特殊的redis)
3. 实际应该多机器
4. 详细配置节点

**配置文件解释**

```shell
# 哨兵sentinel实例运行的端口，默认26379  
port 26379
# 哨兵sentinel的工作目录
dir ./

# 哨兵sentinel监控的redis主节点的 
## ip：主机ip地址
## port：哨兵端口号
## master-name：可以自己命名的主节点名字（只能由字母A-z、数字0-9 、这三个字符".-_"组成。）
## quorum：当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了  
# sentinel monitor <master-name> <ip> <redis-port> <quorum>  
sentinel monitor mymaster 127.0.0.1 6379 2

# 当在Redis实例中开启了requirepass <foobared>，所有连接Redis实例的客户端都要提供密码。
# sentinel auth-pass <master-name> <password>  
sentinel auth-pass mymaster 123456  

# 指定主节点应答哨兵sentinel的最大时间间隔，超过这个时间，哨兵主观上认为主节点下线，默认30秒  
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000  

# 指定了在发生failover主备切换时，最多可以有多少个slave同时对新的master进行同步。这个数字越小，完成failover所需的时间就越长；反之，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1，来保证每次只有一个slave，处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1  

# 故障转移的超时时间failover-timeout，默认三分钟，可以用在以下这些方面：
## 1. 同一个sentinel对同一个master两次failover之间的间隔时间。  
## 2. 当一个slave从一个错误的master那里同步数据时开始，直到slave被纠正为从正确的master那里同步数据时结束。  
## 3. 当想要取消一个正在进行的failover时所需要的时间。
## 4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来同步数据了
# sentinel failover-timeout <master-name> <milliseconds>  
sentinel failover-timeout mymaster 180000

# 当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本。一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
# 对于脚本的运行结果有以下规则：  
## 1. 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10。
## 2. 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。  
## 3. 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
# sentinel notification-script <master-name> <script-path>  
sentinel notification-script mymaster /var/redis/notify.sh

# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```

**节点划分**

| 角色            | IP地址       | 端口号 |
| --------------- | ------------ | ------ |
| Redis Master    | 172.16.91.88 | 6379   |
| Redis Slave1    | 172.16.91.88 | 6380   |
| Redis Slave2    | 172.16.91.88 | 6381   |
| Redis Sentinel1 | 172.16.91.88 | 26379  |
| Redis Sentinel2 | 172.16.91.88 | 26381  |
| Redis Sentinel3 | 172.16.91.88 | 26382  |

主从的配置参考上一篇文章

**配置文件**

指定监听Master(三个节点),监视上一章的主从

```shell
# vi ./conf/sentinel_26379.conf
port 26379
daemonize yes
dir /tmp
logfile "redis_26379.log"
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 900000
 
# vi ./conf/sentinel_26381.conf
port 26381
daemonize yes
dir /tmp
logfile "redis_26381.log"
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 900000

# vi ./conf/sentinel_26382.conf
port 26382
daemonize yes
dir /tmp
logfile "redis_26382.log"
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 900000


#上面配置文件说明如下：
####
port 5000#sentinel
#监听的端口
sentinel monitor mymaster192.168.241.150 9400 2 
#192.168.241.150是master redis的IP地址和端口，2是代表2个sentinel检测到异常，才判断是real fail. Mymaster主机组的名称可以随便定义
sentinel down-after-millisecondsmymaster 30000      #指定sentinel监控到redis实例持续异常多长时间后，会判决其状态为down。若实际业务需要sentinel尽快判决出redis实例异常，则可适当配小,单位是毫秒
sentinel can-failover mymaster yes                  #在sentinel检测到O_DOWN后，是否对这台redis启动failover机制
sentinel parallel-syncs mymaster 1                  #执行故障转移时，最多可以有多少个从服务器同时对新的主服务器进行同步，这个数字越小，完成故障转移所需的时间就越长，但越大就意味着越多的从服务器因为复制而不可用。可以通过将这个值设为 1 来保证每次只有一个从服务器处于不能处理命令请求的状态。
sentinel failover-timeout mymaster900000            #若sentinel在该配置值内未能完成failover操作（即故障时master/slave自动切换），则认为本次failover失败
sentinel auth-pass master1rot@minunix              #当redis访问都需要密码的时候，即在redis.conf有配置requirepass项的时候，需要定义此项
sentinel notification-script master1/data/scripts/send_mail.sh
#当sentinel触发时，切换主从状态时，需要执行的脚本。当主down的时候可以通知当事人,
```

启动sentinel(三个节点)：

```shell
# ./bin/redis-sentinel ./conf/sentinel_26379.conf
# ./bin/redis-sentinel ./conf/sentinel_26381.conf
# ./bin/redis-sentinel ./conf/sentinel_26382.conf
```

注意，必须使用redis-sentinel命令启动sentinel_26379.conf，虽然redis-sentinel是redis-server的软连接

启动sentinel后，查看sentinel_26379.conf、sentinel_26380.conf、sentinel_26381.conf.可发现配置文件会记录已经加入的集群节点,并且配置文件内容改变了：

```shell
[root@localhost redis]# cat ./conf/sentinel-26379.conf
port 26379
daemonize yes
dir "/tmp"
logfile "redis_26382.log"
sentinel myid 50f35915257992b0439b1c9e0931dc5c3aec8619
sentinel deny-scripts-reconfig yes
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel failover-timeout mymaster 900000
# Generated by CONFIG REWRITE
protected-mode no
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel known-replica mymaster 172.16.91.88 6381
sentinel known-replica mymaster 172.16.91.88 6380
sentinel known-sentinel mymaster 172.16.91.88 26381 f61d75e58716c42a927d9455ab8a58101dca15da
sentinel known-sentinel mymaster 172.16.91.88 26382 c3d145dd8d1f2e677405c7fbeb08702928ac628c
sentinel current-epoch 0


[root@localhost redis]# cat ./conf/sentinel_26381.conf    
port 26381
daemonize yes
dir "/tmp"
logfile "redis_26381.log"
sentinel myid f61d75e58716c42a927d9455ab8a58101dca15da
sentinel deny-scripts-reconfig yes
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel failover-timeout mymaster 900000
# Generated by CONFIG REWRITE
protected-mode no
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel known-replica mymaster 172.16.91.88 6380
sentinel known-replica mymaster 172.16.91.88 6381
sentinel known-sentinel mymaster 172.16.91.88 26379 50f35915257992b0439b1c9e0931dc5c3aec8619
sentinel known-sentinel mymaster 172.16.91.88 26382 c3d145dd8d1f2e677405c7fbeb08702928ac628c
sentinel current-epoch 0


[root@localhost redis]# cat ./conf/sentinel_26382.conf
port 26382
daemonize yes
dir "/tmp"
logfile "redis_26382.log"
sentinel myid c3d145dd8d1f2e677405c7fbeb08702928ac628c
sentinel deny-scripts-reconfig yes
sentinel monitor mymaster 172.16.91.88 6379 2
sentinel failover-timeout mymaster 900000
# Generated by CONFIG REWRITE
protected-mode no
sentinel config-epoch mymaster 0
sentinel leader-epoch mymaster 0
sentinel known-replica mymaster 172.16.91.88 6381
sentinel known-replica mymaster 172.16.91.88 6380
sentinel known-sentinel mymaster 172.16.91.88 26379 50f35915257992b0439b1c9e0931dc5c3aec8619
sentinel known-sentinel mymaster 172.16.91.88 26381 f61d75e58716c42a927d9455ab8a58101dca15da
sentinel current-epoch 0
```

查看启动状态：

```shell
# ps -ef | grep redis
root     12978     1  0 13:57 ?        00:00:00 ./bin/redis-server 0.0.0.0:6379
root     12983     1  0 13:57 ?        00:00:00 ./bin/redis-server 0.0.0.0:6380
root     12989     1  0 13:57 ?        00:00:00 ./bin/redis-server 0.0.0.0:6381
root     13012     1  0 14:03 ?        00:00:00 ./bin/redis-sentinel *:26379 [sentinel]
root     13017     1  0 14:04 ?        00:00:00 ./bin/redis-sentinel *:26382 [sentinel]
root     13037     1  0 14:05 ?        00:00:00 ./bin/redis-sentinel *:26381 [sentinel]

```

查看集群状态:

```shell
$ ./bin/redis-cli info replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.16.91.88,port=6380,state=online,offset=62594,lag=1
slave1:ip=172.16.91.88,port=6381,state=online,offset=62594,lag=1
master_replid:aaa2b9039913ed499d0274dd8a2106d22b9e85df
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:62733
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:62733
```

查看sentinel状态：

```shell
$  ./bin/redis-cli -p 26379 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.16.91.88:6379,slaves=2,sentinels=3

$ ./bin/redis-cli -p 26381 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.16.91.88:6379,slaves=2,sentinels=3

$ ./bin/redis-cli -p 26382 info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.16.91.88:6379,slaves=2,sentinels=3

```

 **Sentinel时客户端命令**

```shell
#显示被监控的所有 主节点 以及它们的状态。
127.0.0.1:26379> SENTINEL masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "172.16.91.88"
    5) "port"
    6) "6379"
    7) "runid"
    8) "540c2d779bad31c469c885f797b334536c0cd9fd"
    9) "flags"
   10) "master"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "1052"
   19) "last-ping-reply"
   20) "1052"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "7228"
   25) "role-reported"
   26) "master"
   27) "role-reported-time"
   28) "1653625"
   29) "config-epoch"
   30) "0"
   31) "num-slaves"
   32) "2"
   33) "num-other-sentinels"
   34) "2"
   35) "quorum"
   36) "2"
   37) "failover-timeout"
   38) "900000"
   39) "parallel-syncs"
   40) "1"
#显示指定 主节点 的信息和状态。
#> SENTINEL master <master_name>
#显示指定 主节点 的所有 从节点 以及它们的状态。
#> SENTINEL slaves <master_name>

127.0.0.1:26379> SENTINEL slaves  mymaster
1)  1) "name"
    2) "172.16.91.88:6380"
    3) "ip"
    4) "172.16.91.88"
    5) "port"
    6) "6380"
    7) "runid"
    8) "1a7c6cdad51571419ba27626e365bdc1416996b3"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "548"
   19) "last-ping-reply"
   20) "548"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "4590"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "2042636"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "172.16.91.88"
   35) "master-port"
   36) "6379"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "559168"
2)  1) "name"
    2) "172.16.91.88:6381"
    3) "ip"
    4) "172.16.91.88"
    5) "port"
    6) "6381"
    7) "runid"
    8) "defdc5aa58e6c1944688e1c585dc46f97773d2d2"
    9) "flags"
   10) "slave"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "548"
   19) "last-ping-reply"
   20) "548"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "4590"
   25) "role-reported"
   26) "slave"
   27) "role-reported-time"
   28) "2042636"
   29) "master-link-down-time"
   30) "0"
   31) "master-link-status"
   32) "ok"
   33) "master-host"
   34) "172.16.91.88"
   35) "master-port"
   36) "6379"
   37) "slave-priority"
   38) "100"
   39) "slave-repl-offset"
   40) "559168"

#返回指定 主节点 的 IP 地址和 端口。如果正在进行 failover 或者 failover 已经完成，将会显示被提升为 主节点 的 从节点 的 IP 地址和 端口。
#> SENTINEL get-master-addr-by-name <master_name>

127.0.0.1:26379> SENTINEL get-master-addr-by-name mymaster
1) "172.16.91.88"
2) "6379"

#重置名字匹配该 正则表达式 的所有的 主节点 的状态信息，清除它之前的 状态信息，以及 从节点 的信息。
#> SENTINEL reset <pattern>

#强制当前 Sentinel 节点执行 failover，并且不需要得到其他 Sentinel 节点的同意。但是 failover 后会将 最新的配置 发送给其他 Sentinel 节点。 
#>SENTINEL failover <master_name>   
```

#### Redis Sentinel故障切换与恢复

 **Redis CLI客户端跟踪**

上面的日志显示，`redis-16379` 节点为 **主节点**，它的进程 `ID` 为 `12978`。为了模拟 `Redis` 主节点故障，强制杀掉这个进程。

```
# kill -9 12978
```

使用 `redis-cli` 客户端命令进入 `sentinel-26379` 节点，查看 `Redis` **节点** 的状态信息。

查看 `Redis` 主从集群的 **主节点** 信息。可以发现 `redis-6380` 晋升为 **新的主节点**。

```shell
# ./bin/redis-cli -p 26379
127.0.0.1:26379> SENTINEL masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "172.16.91.88"
    5) "port"
    6) "6380"
    7) "runid"
    8) ""
    9) "flags"
   10) "master,disconnected"
   11) "link-pending-commands"
   12) "3"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "59"
   17) "last-ok-ping-reply"
   18) "59"
   19) "last-ping-reply"
   20) "59"
   21) "down-after-milliseconds"
   22) "30000"
   23) "info-refresh"
   24) "36673"
   25) "role-reported"
   26) "master"
   27) "role-reported-time"
   28) "59"
   29) "config-epoch"
   30) "1"
   31) "num-slaves"
   32) "2"
   33) "num-other-sentinels"
   34) "2"
   35) "quorum"
   36) "2"
   37) "failover-timeout"
   38) "900000"
   39) "parallel-syncs"
   40) "1"

```

**Redis Sentinel日志跟踪**

查看任意 `Sentinel` 节点的日志如下：

```shell
# cat /tmp/redis_26382.log
...
13079:X 29 Nov 2019 15:00:49.483 # +sdown master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.535 # +odown master mymaster 172.16.91.88 6379 #quorum 2/2
13079:X 29 Nov 2019 15:00:49.535 # +new-epoch 1
13079:X 29 Nov 2019 15:00:49.535 # +try-failover master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.538 # +vote-for-leader c3d145dd8d1f2e677405c7fbeb08702928ac628c 1
13069:X 29 Nov 2019 15:00:49.540 # +new-epoch 1
13069:X 29 Nov 2019 15:00:49.542 # +vote-for-leader c3d145dd8d1f2e677405c7fbeb08702928ac628c 1
13079:X 29 Nov 2019 15:00:49.542 # 50f35915257992b0439b1c9e0931dc5c3aec8619 voted for c3d145dd8d1f2e677405c7fbeb08702928ac628c 1
13079:X 29 Nov 2019 15:00:49.543 # f61d75e58716c42a927d9455ab8a58101dca15da voted for c3d145dd8d1f2e677405c7fbeb08702928ac628c 1
13069:X 29 Nov 2019 15:00:49.553 # +sdown master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.600 # +elected-leader master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.600 # +failover-state-select-slave master mymaster 172.16.91.88 6379
13069:X 29 Nov 2019 15:00:49.653 # +odown master mymaster 172.16.91.88 6379 #quorum 3/2
13069:X 29 Nov 2019 15:00:49.653 # Next failover delay: I will not start a failover before Fri Nov 29 15:30:49 2019
13079:X 29 Nov 2019 15:00:49.701 # +selected-slave slave 172.16.91.88:6380 172.16.91.88 6380 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.701 * +failover-state-send-slaveof-noone slave 172.16.91.88:6380 172.16.91.88 6380 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:49.759 * +failover-state-wait-promotion slave 172.16.91.88:6380 172.16.91.88 6380 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:50.334 # +promoted-slave slave 172.16.91.88:6380 172.16.91.88 6380 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:50.334 # +failover-state-reconf-slaves master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:50.397 * +slave-reconf-sent slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6379
13069:X 29 Nov 2019 15:00:50.398 # +config-update-from sentinel c3d145dd8d1f2e677405c7fbeb08702928ac628c 172.16.91.88 26382 @ mymaster 172.16.91.88 6379
13069:X 29 Nov 2019 15:00:50.398 # +switch-master mymaster 172.16.91.88 6379 172.16.91.88 6380
13069:X 29 Nov 2019 15:00:50.398 * +slave slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6380
13069:X 29 Nov 2019 15:00:50.398 * +slave slave 172.16.91.88:6379 172.16.91.88 6379 @ mymaster 172.16.91.88 6380
13079:X 29 Nov 2019 15:00:50.608 # -odown master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:51.364 * +slave-reconf-inprog slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:51.364 * +slave-reconf-done slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:51.426 # +failover-end master mymaster 172.16.91.88 6379
13079:X 29 Nov 2019 15:00:51.427 # +switch-master mymaster 172.16.91.88 6379 172.16.91.88 6380
13079:X 29 Nov 2019 15:00:51.427 * +slave slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6380
13079:X 29 Nov 2019 15:00:51.427 * +slave slave 172.16.91.88:6379 172.16.91.88 6379 @ mymaster 172.16.91.88 6380
13069:X 29 Nov 2019 15:01:20.420 # +sdown slave 172.16.91.88:6379 172.16.91.88 6379 @ mymaster 172.16.91.88 6380
13079:X 29 Nov 2019 15:01:21.432 # +sdown slave 172.16.91.88:6379 172.16.91.88 6379 @ mymaster 172.16.91.88 6380
```

分析日志，可以发现 `redis-6379` 节点先进入 `sdown` **主观下线** 状态。

```
13079:X 29 Nov 2019 15:00:49.483 # +sdown master mymaster 172.16.91.88 6379
```

哨兵检测到 `redis-6379` 出现故障，`Sentinel` 进入一个 **新纪元**，从 `0` 变为 `1`。

```
13079:X 29 Nov 2019 15:00:49.535 # +new-epoch 1
```

三个 `Sentinel` 节点开始协商 **主节点** 的状态，判断其是否需要 **客观下线**。

```
13069:X 29 Nov 2019 15:00:49.542 # +vote-for-leader c3d145dd8d1f2e677405c7fbeb08702928ac628c 1
```

超过 `quorum` 个数的 `Sentinel` 节点认为 **主节点** 出现故障，`redis-6379` 节点进入 **客观下线** 状态。

```
13069:X 29 Nov 2019 15:00:49.653 # +odown master mymaster 172.16.91.88 6379 #quorum 3/2
```

`Sentinal` 进行 **自动故障切换**，协商选定 `redis-6380` 节点作为新的 **主节点**。

```
13079:X 29 Nov 2019 15:00:51.427 # +switch-master mymaster 172.16.91.88 6379 172.16.91.88 6380
```

`redis-36329` 节点和已经 **客观下线** 的 `redis-6379` 节点成为 `redis-6380` 的 **从节点**。

```
13079:X 29 Nov 2019 15:00:51.427 * +slave slave 172.16.91.88:6381 172.16.91.88 6381 @ mymaster 172.16.91.88 6380
13079:X 29 Nov 2019 15:00:51.427 * +slave slave 172.16.91.88:6379 172.16.91.88 6379 @ mymaster 172.16.91.88 6380
```

**Redis的配置文件**

分别查看三个 `redis` 节点的配置文件，发生 **主从切换** 时 `redis.conf` 的配置会自动发生刷新。

```shell
# cat ./conf/master-6739.conf 
bind 0.0.0.0
port 6379
dir /tmp
logfile "6379.log"
dbfilename "dump-6379.rdb"
daemonize yes
rdbcompression yes

# cat ./conf/slave-6380.conf 
bind 0.0.0.0
port 6380
dir "/tmp"
logfile "6380.log"
dbfilename "dump-6380.rdb"
daemonize yes
rdbcompression yes

# cat ./conf/slave-6381.conf 
bind 0.0.0.0
port 6381
dir "/tmp"
logfile "6381.log"
dbfilename "dump-6381.rdb"
daemonize yes
rdbcompression yes
replicaof 172.16.91.88 6380

```

**分析**：`redis-6380` 节点 `slaveof` 配置被移除，晋升为 **主节点**。`redis-6379` 节点处于 **宕机状态**。`redis-6381` 的 `slaveof` 配置更新为 `127.0.0.1 redis-6380`，成为 `redis-6380` 的 **从节点**。

重启节点 `redis-6379`。待正常启动后，再次查看它的 `redis.conf` 文件，配置如下：

```shell
# cat ./conf/master-6739.conf 
bind 0.0.0.0
port 6379
dir "/tmp"
logfile "6379.log"
dbfilename "dump-6379.rdb"
daemonize yes
rdbcompression yes
# Generated by CONFIG REWRITE
replicaof 172.16.91.88 6380

```

节点 `redis-6379` 的配置文件新增一行 `replicaof` 配置属性，指向 `redis-6380`，即成为 **新的主节点** 的 **从节点**。

### 参考

1. [Redis Sentinel](https://www.jianshu.com/p/ded0fd07f0b8)
2. [Redis集群redis主从自动切换Sentinel（哨兵模式）](https://www.duoluosb.com/665.html)
3. [Redis 的 Sentinel 文档](http://www.redis.cn/topics/sentinel.html)
4. [冰叔带你了解Redis-哨兵模式和高可用集群解析](https://www.cnblogs.com/bingshu/p/9776610.html)