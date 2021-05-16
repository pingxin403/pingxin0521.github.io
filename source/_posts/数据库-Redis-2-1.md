---
title: Redis 持久化、日志、慢查询、数据导入导出
date: 2019-10-28 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

#### Redis高可用概述

在 `Web` 服务器中，**高可用** 是指服务器可以 **正常访问** 的时间，衡量的标准是在 **多长时间** 内可以提供正常服务（`99.9%`、`99.99%`、`99.999%` 等等）。在 `Redis` 层面，**高可用** 的含义要宽泛一些，除了保证提供 **正常服务**（如 **主从分离**、**快速容灾技术** 等），还需要考虑 **数据容量扩展**、**数据安全** 等等。

在 `Redis` 中，实现 **高可用** 的技术主要包括 **持久化**、**复制**、**哨兵** 和 **集群**，下面简单说明它们的作用，以及解决了什么样的问题：

<!--more-->

- **持久化**：持久化是 **最简单的** 高可用方法。它的主要作用是 **数据备份**，即将数据存储在 **硬盘**，保证数据不会因进程退出而丢失。
- **复制**：复制是高可用 `Redis` 的基础，**哨兵** 和 **集群** 都是在 **复制基础** 上实现高可用的。复制主要实现了数据的多机备份以及对于读操作的负载均衡和简单的故障恢复。缺陷是故障恢复无法自动化、写操作无法负载均衡、存储能力受到单机的限制。
- **哨兵**：在复制的基础上，哨兵实现了 **自动化** 的 **故障恢复**。缺陷是 **写操作** 无法 **负载均衡**，**存储能力** 受到 **单机** 的限制。
- **集群**：通过集群，`Redis` 解决了 **写操作** 无法 **负载均衡** 以及 **存储能力** 受到 **单机限制** 的问题，实现了较为 **完善** 的 **高可用方案**。

### 持久化

Redis有两种持久化的方式：快照（RDB文件）和追加式文件（AOF文件）

1. RDB持久化方式是在一个特定的间隔保存某个时间点的一个数据快照。（默认模式）
2. 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作 。

注意：Redis的持久化是可以禁用的，两种方式的持久化是可以同时存在的，但是当Redis重启时，AOF文件会被优先用于重建数据。

#### RDB

##### what

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。

整个过程中，主进程是不进行任何IO操作的，这就确保了如果需要进行大规模数据的恢复时还能有极高的性能。

![UTOOLS1574523010635.png](https://i.loli.net/2019/11/23/uypnDFcsKt23QMl.png)

##### where

默认Redis会把快照文件存储为当前目录下一个名为**dump.rdb**的文件。要修改文件的存储路径和名称，可以通过修改配置文件redis.conf实现：

```
# RDB文件名，默认为dump.rdb。
dbfilename dump.rdb

# 文件存放的目录，AOF文件同样存放在此目录下。默认为当前工作目录。
dir ./
```

##### redis.conf如何配置

保存点可以设置多个，Redis的配置文件就默认设置了3个保存点：

```
# 以下配置表示的条件：
# 服务器在900秒之内被修改了1次
save 900 1
# 服务器在300秒之内被修改了10次
save 300 10
# 服务器在60秒之内被修改了10000次
save 60 10000

#如果想禁用快照保存的功能，可以通过注释掉所有"save"配置达到，或者在最后一条"save"配置后添加如下的配置：
save ""
```

##### 如何启动快照

有四种方式：

1. 在**配置文件**中你配置保存点，而不是save ""，当达到要求会进行快照。
2. **SAVE**命令：SAVE命令会使用同步的方式生成RDB快照文件，这意味着在这个过程中会阻塞所有其他客户端的请求。因此不建议在生产环境使用这个命令。
3. **BGSAVE**命令：BGSAVE命令使用后台的方式保存RDB文件，调用此命令后，会立刻返回OK返回码。Redis会产生一个子进程进行处理并立刻恢复对客户端的服务。在客户端我们可以使用LASTSAVE命令查看操作是否成功
4. **执行flushall**命令，也会产生dump.rdb文件，但里面是空的，无意义。

注意：配置文件里禁用了快照生成功能不影响SAVE和BGSAVE命令的效果。

#### AOF

##### what

以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作 。

注意：Aof保存的是**appendonly.aof**文件

![UTOOLS1574523165000.png](https://i.loli.net/2019/11/23/pPqHvXz5hSytU9o.png)

##### AOF启动、修复、恢复

1. 正常启动
   修改默认的`appendonly no`，改为yes（因为redis默认是RDB持久化，所以这里需要在redis.conf手动开启）

2. 如果启动失败，有可能是你的appendonly.aof有误（比如因为磁盘满了，命令只写了一半到日志文件里，我们也可以用redis-check-aof这个工具很简单的进行修复。）

   具体做法：

   ```
   redis-check-aof --fix进行修复
   ```

##### 同步策略

它有三种同步策略**：**appendfsync always(每修改同步), appendfsync everysec(每秒同步)， appendfsync no （不同步）

1. appendfsync always(每修改同步)： 同步持久化 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好。

2. appendfsync everysec(每秒同步)：异步操作，每秒记录. 如果发生灾难，您可能会丢失1秒的数据。

3. appendfsync no （不同步）：从不同步。

官方建议使用默认配置每秒同步，它既快速又安全。这个always策略在实践中非常缓慢， 没有办法做得fsync比现在更快。

##### AOF重写

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，

只保留可以恢复数据的最小指令集.

**重写原理**

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，

而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似

**触发机制**

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发（redis默认是64M进行重新一次，而实际生产环境这里至少要2G）

#### RDB和AOF优缺点

**RDB的优点**

1. 比起AOF，在数据量比较大的情况下，RDB的启动速度更快。
2. RDB文件是一个很简洁的单文件，它保存了某个时间点的Redis数据，很适合用于做备份。
3. RDB很适合用于灾备。单文件很方便就能传输到远程的服务器上。
4. RDB的性能很好，需要进行持久化时，主进程会fork一个子进程出来，然后把持久化的工作交给子进程，自己不会有相关的I/O操作。

**RDB缺点**

1. RDB容易造成数据的丢失。假设每5分钟保存一次快照，如果Redis因为某些原因不能正常工作，那么从上次产生快照到Redis出现问题这段时间的数据就会丢失了。
2. RDB使用fork()产生子进程进行数据的持久化，如果数据比较大的话可能就会花费点时间，造成Redis停止服务几毫秒。

**AOF优点**

1. 该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，如果发生灾难，您只可能会丢失1秒的数据。

2. AOF日志文件是一个纯追加的文件。就算服务器突然Crash，也不会出现日志的定位或者损坏问题。甚至如果因为某些原因（例如磁盘满了）命令只写了一半到日志文件里，我们也可以用redis-check-aof这个工具很简单的进行修复。

3. 当AOF文件太大时，Redis会自动在后台进行重写。重写很安全，因为重写是在一个新的文件上进行，同时Redis会继续往旧的文件追加数据。

**AOF缺点**

1. 在相同的数据集下，AOF文件的大小一般会比RDB文件大。

2. 在某些fsync策略下，AOF的速度会比RDB慢。通常fsync设置为每秒一次就能获得比较高的性能，而在禁止fsync的情况下速度可以达到RDB的水平。

#### RDB和AOF建议

官方建议：是同时开启两种持久化策略。因为有时需要RDB快照是进行数据库备份，更快重启以及发生AOF引擎错误的解决办法。（换句话就是通过RDB来多备份一份数据总是好的）

因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。

如果选择AOF，只要硬盘许可，应该尽量减少AOF rewrite的频率。因为一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。

AOF重写的基础大小默认值64M太小了，可以设到5G以上。

###### Redis 4.0 混合持久化

重启 Redis 时，我们很少使用 rdb 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 rdb 来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。 Redis 4.0 为了解决这个问题，带来了一个新的持久化选项——混合持久化。

AOF在重写(aof文件里可能有太多没用指令，所以aof会定期根据内存的最新数据生成aof文件)时将重写这一刻之前的内存rdb快照文件的内容和增量的 AOF修改内存数据的命令日志文件存在一起，都写入新的aof文件，新的文件一开始不叫appendonly.aof，等到重写完新的AOF文件才会进行改名，原子的覆盖原有的AOF文件，完成新旧两个AOF文件的替换；

aof 根据配置规则在后台自动重写，也可以人为执行命令bgrewriteaof重写AOF。 于是在 Redis 重启的时候，可以先加载 rdb 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，重启效率因此大幅得到提升。

###### 开启混合持久化：

```php
aof-use-rdb-preamble yes
```

###### 混合持久化aof文件结构

![UTOOLS1574598990040.png](https://i.loli.net/2019/11/24/EzGcY84UlyNegZ9.png)

###### 缓存淘汰策略(解决数据热点问题)：

当 Redis 内存超出物理内存限制时，内存的数据会开始和磁盘产生频繁的交换 (swap)。交换会让 Redis 的性能急剧下降，对于访问量比较频繁的 Redis 来说，这样龟速的存取效率基本上等于不可用。

在生产环境中我们是不允许 Redis 出现交换行为的，为了限制最大使用内存，Redis 提供了配置参数 maxmemory 来限制内存超出期望大小。

当实际内存超出 maxmemory 时，Redis 提供了几种可选策略 (maxmemory-policy) 来让用户自己决定该如何腾出新的空间以继续提供读写服务。

**noeviction：**

不会继续服务写请求 (DEL 请求可以继续服务)，读请求可以继续进行。这样可以保证不会丢失数据，但是会让线上的业务不能持续进行。这是默认的淘汰策略。

**volatile-lru：**
 尝试淘汰设置了过期时间的 key，最少使用的 key 优先被淘汰。没有设置过期时间的 key 不会被淘汰，这样可以保证需要持久化的数据不会突然丢失。

**volatile-ttl：**
 跟上面一样，除了淘汰的策略不是 LRU，而是 key 的剩余寿命 ttl 的值，ttl 越小越优先被淘汰。

**volatile-random：**
 跟上面一样，不过淘汰的 key 是过期 key 集合中随机的 key。

**allkeys-lru：**
 区别于 volatile-lru，这个策略要淘汰的 key 对象是全体的 key 集合，而不只是过期的 key 集合。这意味着没有设置过期时间的 key 也会被淘汰。
 allkeys-random跟上面一样，不过淘汰的策略是随机的 key。

**volatile-xxx ：**
 策略只会针对带过期时间的 key 进行淘汰，allkeys-xxx 策略会对所有的 key 进行淘汰。如果你只是拿 Redis 做缓存，那应该使用 allkeys-xxx，客户端写缓存时不必携带过期时间。如果你还想同时使用 Redis 的持久化功能，那就使用 volatile-xxx 策略，这样可以保留没有设置过期时间的 key，它们是永久的 key 不会被 LRU 算法淘汰。

### 日志

redis在默认情况下，是不会生成日志文件的，所以需要配置

1. 打开redis.conf文件
2. 找到 logfile 行，添加输入日志路径
3. 找到loglevel，Redis4默认的设置为notice，开发测试阶段可以用debug（日志内容较多一般不建议使用），生产模式一般选用notice
   -  debug：会打印出很多信息，适用于开发和测试阶段
   - verbose（冗长的）：包含很多不太有用的信息，但比debug要清爽一些
   - notice：适用于生产模式
   - warning : 警告信息

#### 慢查询日志

许多存储系统（例如MySQL）提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。

所谓**慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阈值，**

**就将这条命令的相关信息（例如：发生时间、耗时、命令的详细信息）记录下来**，Redis也提供了类似的功能

Redis客户端执行一条命令分为4个部分：

![UTOOLS1574582580052.png](https://i.loli.net/2019/11/24/sIbFmki3YgGWe5z.png)

1. 发送命令
2. 排队
3. 执行命令
4. 返回结果

注意：**慢查询只会记录执行命令的时间，没有慢查询并不代表客户端没有超时问题**。

##### 配置

首先我们需要知道redis的慢查询日志有什么用？日常在使用redis的时候为什么要用慢查询日志？

- 第一个问题：慢查询日志是为了记录执行时间超过给定时长的redis命令请求

- 第二个问题：让使用者更好地监视和找出在业务中一些慢redis操作，找到更好的优化方法

在Redis中，关于慢查询有两个设置--慢查询**最大超时时间**和**慢查询最大日志数**。

1. 可以通过修改配置文件或者直接在交互模式下输入以下命令来设置慢查询的时间限制，当超过这个时间，查询的记录就会加入到日志文件中。

   ```
   CONFIG  SET  slowlog-log-slower-than  num
   ```

   设置超过多少微妙的查询为慢查询，并且将这些慢查询加入到日志文件中，num的单位为**毫秒**，windows下redis的默认慢查询时10000微妙即10毫秒。

2. 可以通过设置最大数量限制日志中保存的慢查询日志的数量，此设置在交互模式下的命令如下

   ```
   CONFIG  SET  slowlog-max-len  num
   ```

   设置日志的最大数量，num无单位值，windows下redis默认慢查询日志的记录数量为128条。

   如果**slowlog-log-slower-than=0**，**那么系统会记录所有的命令**；如果**slowlog-log-slower-than<0**，**那么对任何命令都不会记录**。

在配置完成后执行

```
config rewrite
```

如果要Redis将配置持久化到本地配置文件，要执行config rewrite命令，它会重写配置文件。

CONFIG 命令会使redis客户端自行去寻找redis的.conf 配置文件，找到对应的配置项进行修改。

因此可以直接修改配置文件中的：`slowlog-log-slower-than  num`和`slowlog-max-len  num`

##### 操作

1. 那么接下来，如何查看慢查询呢？

   又是进入交互模式下，命令很简单。

   ```shell
   slowlog get [n]
   #（当然也可以用小写，redis客户端对大小写没有太严格的限制）
   
   
   127.0.0.1:6379[1]> SLOWLOG GET
   1  #表示日志唯一标识符uid
   1574517963 #命令执行时系统的时间戳
   12304 #命令执行的时长，以微妙来计算
   FLUSHALL #命令和命令的参数
   127.0.0.1:36390 #对应的客户端地址
   
   0
   1574492783
   312948
   FLUSHALL
   127.0.0.1:56634
   ```

   可以看到每个慢查询日志有4个属性组成，分别是慢查询日志的识别id、发生时间戳、命令耗时、执行命令和参数。

   ![UTOOLS1574582828291.png](https://i.loli.net/2019/11/24/wEvxBNDuQVjTp3c.png)

2. 获取慢查询日志列表当前的长度

   命令：`slowlog len`

   ```
   127.0.0.1:6379> slowlog len
   (integer) 2
   ```

3. 慢查询日志重置

   命令：`slowlog reset`

   实际是对慢查询日志列表做清理操作。

   ```
   127.0.0.1:6379> slowlog len
   (integer) 6
   127.0.0.1:6379> slowlog reset
   OK
   127.0.0.1:6379> slowlog len
   (integer) 1
   #为什么还有1个，因为阈值设的比较小，slowlog reset就属于慢查询。
   ```

##### 注意事项

慢查询功能可以有效的帮助我们找到Redis可能存在的瓶颈，但在实际使用过程中要注意以下几点：

1. slowlog-max-len配置建议：线上建议调大慢查询列表，记录慢查询时Redis会对长命令做截断操作，并不会占用大量内存。

   **增大慢查询列表可以减缓慢查询被剔除的可能**。

2. slowlog-log-slower-than配置建议：默认值超过10毫秒判定为慢查询，需要根据Redis并发量调整该值。

   由于Redis采用单线程响应命令，对于高流量的场景，如果命令执行时间在1毫秒以上，那么Redis最多可以支撑OPS不到1000，因此对于高OPS的场景的Redis建议设置1毫秒。

3. **慢查询只记录命令执行时间，并不包括命令排队和网络传输时间**。因此客户端执行命令的时间会大于命令实际执行的时间。

   因为命令执行排队机制，慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时，

   需要检查该时间点是否有对应的慢查询，从而分析出是否为慢查询导致的命令级联阻塞。

4. 由于慢查询日志是一个先进先出的队列，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，

   为了防止这种情况发生，可以定期执行slowlog get命令将慢查询日志持久化到其他存储中（例如，MySQL），

　　然后可以制作出可视化界面进行查询。

### 数据导入导出

#### redis-dump方式

1. 安装redis-dump工具

   ```shell
   $  sudo apt install ruby -y
   $ gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
   $ gem sources -l
   https://gems.ruby-china.com
   # 确保只有 gems.ruby-china.com
   $  sudo gem install redis-dump -V
   ```

2. 导出

   ```
   $ redis-dump -u :password@127.0.0.1:6379 > back.json
   ```

3. 导入

   ```
   cat back.json | redis-load -u :password@127.0.0.1:6379
   ```

#### aof导入方式

1. 源实例生成aof数据

   ```
   # 清空上文目标实例全部数据
   $ redis-cli  -a password flushall
   OK
   # 源实例开启aof功能，将在dir目录下生成appendonly.aof文件
   $ redis-cli -a password config set appendonly yes
   OK
   ```

2. 目标实例导入aof数据

   ```
   # 假设appendonly.aof就在当前路径下
   $ redis-cli - -a password --pipe < appendonly.aof
   All data transferred. Waiting for the last reply...
   Last reply received from server.
   errors: 0, replies: 5 # 源实例关闭aof功能
   
   $ redis-cli  -a password config set appendonly no
   OK
   ```

#### rdb文件迁移方式

1. 关闭要迁移到的服务器的redis的aof日志功能（我的要迁移到的是本机的redis6380.conf）

   vim redis.conf，将 `appendonly yes` 修改为 `appendonly no`

2. 我们先看一下当前redis的数据，并将数据用save命令固化到rdb文件中
3. 杀掉当前redis的进程，否则下一步的复制rdb文件，rdb处于打开的状态，复制的文件，会占用同样的句柄
4. 复制当前redis的rdb文件，名字为你要迁移的redis的rdb文件名,记住，一定要杀掉当前redis的进程，还有关闭要迁移的服务器的aof功能（如果不关闭aof，默认用aof文件来恢复数据）

以上就是在不同的redis之间进行rdb的数据迁移，思路就是，复制rdb文件，然后让要迁移的redis加载这个rdb文件就ok了

#### 源实例db0迁移至目标实例db1

```shell
$ cat redis_mv.sh
#!/bin/bash
redis-cli -h 127.0.0.1 -p 6379 -a password -n 0 keys "*" | while read key do redis-cli -h 127.0.0.1 -p 6379 -a password -n 0 --raw dump $key | perl -pe 'chomp if eof' | redis-cli -h 202.102.221.12 -p 6379 -a password -n 1 -x restore $key 0 echo "migrate key $key" done
```

#### 从MySQL导出数据到Redis

1. 当向Redis中一次性导入大数据时,可以将所有的插入命令写到一个txt文件中，如插入 key-value

   ```
   SET test0 abc
   SET test1 bcd
   SET test3 abcd
   ```

   或者

   ```
   sadd miaopai M44xschlTD19MkDL zoPKSNHdMiwz41tM sDSF-A6w6ewLuYLs BLsKbF6JmRjvMEf~ cWGJTDIa-DjfZClg17C6mA__ kHMb23MaqpohIVsM KZDTlh1aOWFjIbsW
   ```

   每个SET命令前要留一个空格，保存为data.txt,然后使用 redis的客户端 redis-cli的管道传输(redis的版本要大于2.6)
   linux下使用命令：

   ```
   cat data.txt | redis-cli --pipe
   cat sadd.txt | redis-cli --pipe -a password
   ```

   成功的话就会出现如下结果：

   ```
   All data transferred. Waiting for the last reply...
   Last reply received from server.
   errors: 0, replies: 3
   ```

2. 使用符合redis协议格式的数据,虽然第一种方法比较方便，不过存在的问题是，有时redis无法正确解释数据，所有推荐的第二种方式,此协议数据的格式如下：

   ```
   *3<cr><lf>
   $3<cr><lf>
   SET<cr><lf>
   $3<cr><lf>
   key<cr><lf>
   $5<cr><lf>
   value<cr><lf>
   ```

   意义如下：
    第一行： `*3` : 星号*是规定格式；3是参数的个数(如上：SET、key、value) ；`<cr>`是`'\r'`；` <lf>`是`'\n'`。`\r\n`代表回车换行。
    第二、三行： `$3<cr><lf>`  : $是规定格式；3是对应命令SET的长度(3个字母)；`<cr><lf>`同上
    所以上述格式又可写成：

   ```bash
   *3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n　
   ```

3. 使用mysql一次性导入大量数据的原理是一样的,将数据按上述协议的格式导出来，然后再通过redis-cli --pipe导入

   建表语句：

   ```objectivec
   CREATE TABLE events_all_time (  
     id int(11) unsigned NOT NULL AUTO_INCREMENT,  
     action varchar(255) NOT NULL,  
     count int(11) NOT NULL DEFAULT 0,  
     PRIMARY KEY (id),  
     UNIQUE KEY uniq_action (action)  
   ); 
   ```

   准备在每行数据中执行的redis命令如下：
    `HSET events_all_time [action] [count]` ,使用了redis哈希的数据类型。
    按照以上redis命令规则，创建一个events_to_redis.sql文件，内容是用来生成redis数据协议格式的SQL：

   ```mysql
   -- events_to_redis.sql  
   SELECT CONCAT(  
     "*4\r\n",  
     '$', LENGTH(redis_cmd), '\r\n',  
     redis_cmd, '\r\n',  
     '$', LENGTH(redis_key), '\r\n',  
     redis_key, '\r\n',  
     '$', LENGTH(hkey), '\r\n',  
     hkey, '\r\n',  
     '$', LENGTH(hval), '\r\n',  
     hval, '\r' 
   )  
   FROM (  
     SELECT 
     'HSET' as redis_cmd,  
     'events_all_time' AS redis_key,  
     action AS hkey,  
     count AS hval  
     FROM events_all_time  
   ) AS t 
   ```

   用下面的命令执行：

   ```shell
   mysql -h47.98.40.244 -uhyp -ppassword  test --skip-column-names --raw < events_to_redis.sql | redis-cli --pipe
   ```

   很重要的mysql参数说明：
    --raw: 使mysql不转换字段值中的换行符。
    --skip-column-names: 使mysql输出的每行中不包含列名。

4. 使用java读文本，写入redis

   ```java
   Pipeline p = redis.pipelined();
   
   private void loadDictionary(final URL dictionaryUrl) throws IOException {
       InputStreamReader inputStreamReader = new InputStreamReader(dictionaryUrl.openStream());
       BufferedReader reader = new BufferedReader(inputStreamReader);
       String word;
       while((word = reader.readLine()) != null) {
           word = word.trim();
           // Add the word if the word does not start with #
           if(!word.isEmpty() && !word.startsWith("#")) {
               addWord(word);
           }
       }
       reader.close();
       inputStreamReader.close();
   }
   
   private void addWord(final String word) {
       // Add all the possible prefixes of the given word and also the given
       // word with a * suffix.
       p.zadd(redisKey, 0, word + "*");
       for(int index = 1, total = word.length(); index < total; index++) {
           p.zadd(redisKey, 0, word.substring(0, index));
       }
   
   }
   ```

   