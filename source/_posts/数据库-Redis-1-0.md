---
title: Redis入门和安装
date: 2019-06-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### 前言

[Redis](https://redis.io/) 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

<!--more-->

#### 特性

1. 速度快

   正常情况下，Redis执行命令的速度非常快，官方给出的数字是读写性能可以达到10万/秒，当然这也取决于机器的性能，但这里先不讨论机器性能上的差异，只分析一下是什么造就了Redis除此之快的速度，可以大致归纳为以下三点：

   - Redis的所有数据都是存放在内存中的，所以把数据放在内存中是Redis速度快的最主要原因。
   - Redis是用C语言实现的，一般来说C语言实现的程序“距离”操作系统更近，执行速度相对会更快。
   - Redis使用了单线程架构，预防了多线程可能产生的竞争问题。

2. 基于键值对的数据结构服务器

   几乎所有的编程语言都提供了类似字典的功能，例如Java里的map、Python里的dict，类似于这种组织数据的方式叫作基于键值的方式，与很多键值对数据库不同的是，Redis中的值不仅可以是字符串，而且还可以是具体的数据结构，这样不仅能便于在许多应用场景的开发，同时也能够提高开发效率。Redis的全称是REmote Dictionary Server，它主要提供了5种数据结构：字符串、哈希、列表、集合、有序集合。

3. 丰富的功能

   除了5种数据结构，Redis还提供了许多额外的功能：

   - 提供了键过期功能，可以用来实现缓存。
   - 提供了发布订阅功能，可以用来实现消息系统。
   - 支持Lua脚本功能，可以利用Lua创造出新的Redis命令。
   - 提供了简单的事务功能，能在一定程度上保证事务特性。
   - 提供了流水线（Pipeline）功能，这样客户端能将一批命令一次性传到Redis，减少了网络的开销。

4. 简单稳定

   Redis的简单主要表现在三个方面。

   - Redis的源码很少。
   - Redis使用单线程模型，这样不仅使得Redis服务端处理模型变得简单，而且也使得客户端开发变得简单。
   - Redis不需要依赖于操作系统中的类库（例如Memcache需要依赖libevent这样的系统类库），Redis自己实现了事件处理的相关功能。
     Redis虽然很简单，但是不代表它不稳定。维护的上千个Redis为例，没有出现过因为Redis自身bug而宕掉的情况。

5. 客户端语言多

   Redis提供了简单的TCP通信协议，很多编程语言可以很方便地接入到Redis，并且由于Redis受到社区和各大公司的广泛认可，所以支持Redis的客户端语言也非常多，几乎涵盖了主流的编程语言，例如Java、PHP、Python、C、C++、Nodejs等。

6. 持久化

   通常看，将数据放在内存中是不安全的，一旦发生断电或者机器故障，重要的数据可能就会丢失，因此Redis提供了两种持久化方式：RDB和AOF，即可以用两种策略将内存的数据保存到硬盘中（如图所示）这样就保证了数据的可持久性。

   ![UTOOLS1574342434921.png](http://yanxuan.nosdn.127.net/29673aaf8c55dd2eb4c2c1473c0a700b.png)

7. 主从复制

   Redis提供了复制功能，实现了多个相同数据的Redis副本（如图所示），复制功能是分布式Redis的基础。

   ![MopZes.png](https://s2.ax1x.com/2019/11/21/MopZes.png)

8. 高可用和分布式

   Redis从2.8版本正式提供了高可用实现Redis Sentinel，它能够保证Redis节点的故障发现和故障自动转移。Redis从3.0版本正式提供了分布式实现Redis Cluster，它是Redis真正的分布式实现，提供了高可用、读写和容量的扩展性。

#### 优势

- 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子 – Redis的所有操作都是原子性的，同时Redis还支持对几个操作合并后的原子性执行。（事务）
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

**Redis与其他key-value存储有什么不同**

- Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
- Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

更多参考：[Redis 和 Memcached 各有什么优缺点，主要的应用场景是什么样的？](https://www.zhihu.com/question/19829601)

[对比 Redis 和 Memcache](https://www.jianshu.com/p/35833aba2457)

**缺点**

1. 由于 Redis 是内存数据库，所以，单台机器，存储的数据量，跟机器本身的内存大小。虽然 Redis 本身有 Key 过期策略，但是还是需要提前预估和节约内存。如果内存增长过快，需要定期删除数据。
2. redis是单线程的，单台服务器无法充分利用多核服务器的CPU
3. 缓存问题
4. 如果进行完整重同步，由于需要生成rdb文件，并进行传输，会占用主机的CPU，并会消耗现网的带宽。不过redis2.8版本，已经有部分重同步的功能，但是还是有可能有完整重同步的。比如，新上线的备机。
5. 修改配置文件，进行重启，将硬盘中的数据加载进内存，时间比较久。在这个过程中，redis不能提供服务。

##### Redis的单线程和高性能

###### Redis 单线程为什么还能这么快？

因为它所有的数据都在内存中，所有的运算都是内存级别的运算（纳秒），而且单线程避免了多线程的切换（上下文切换）性能损耗问题。正因为 Redis 是单线程，所以要小心使用 Redis 指令，对于那些耗时的指令(比如keys)，一定要谨慎使用，一不小心就可能会导致 Redis 卡顿。

###### Redis 单线程如何处理那么多的并发客户端连接？

Redis的IO多路复用：redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中，依次放到文件事件分派器，事件分派器将事件分发给事件处理器。但windows不支持epoll模式，因此Windows上redis源码跟linux上有所不同。

Nginx也是采用IO多路复用原理解决C10K问题。c10k：https://www.jianshu.com/p/ba7fa25d3590

![UTOOLS1574599177138.png](https://i.loli.net/2019/11/24/w5efsBhAlu7SXDt.png)

### 安装

环境：Ubuntu 18.04 LTS

Redis借鉴了Linux 内核对于版本号的命名规则：

版本号第二位如果是**奇数**，则为**非稳定版本**（例如2.7、2.9、3.1），如果是**偶数**，则为**稳定版本**（例如2.6、2.8、3.0、3.2），

当前奇数版本就是下一个稳定版本的开发版本，例如2.9版本是3.0版本的开发版本，所以我们在生产环境通常选取偶数版本的Redis。

#### 源码编译安装

从[中文官网](http://www.redis.cn)下载最新的稳定版源码包，解压后进入目录下

```bash
$ make -j4
$ sudo make install PREFIX=/opt/db/redis
#启动
$ /opt/db/redis/bin/redis-server &
$ /opt/db/redis/bin/redis-benchmark  #基准测试
```
将/opt/db/redis/bin加入环境变量PATH

```bash
$ sudo vim /etc/profile

PATH=${PATH}:/opt/db/redis/bin
```

#### 安装二进制包

升级软件管理模块apt：

```
sudo apt-get update
```

安装Redis服务

```
sudo apt-get install redis-server
```

启动 Redis

```
redis-server &
```

查看 redis 是否还在运行

```bash
$ ps -ef | grep redis       #--> 查看进程
$ netstat -an|grep 6379     #--> redis的端口号是6379
$ redis-cli                #--> 查看redis
```

**注意：**Redis实际上也是通过socket根其他的程序共享数据的;Redis是将内容放入内存中的，效率高

#### Docker 安装 Redis

查找Docker Hub上的redis镜像

```
$ docker search  redis
```

这里我们拉取官方的镜像

```
$ docker pull  redis
```

等待下载完成后，我们就可以在本地镜像列表里查到REPOSITORY为redis,标签为latest的镜像。

```
$ docker images redis
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis               latest              598a6f110d01        2 weeks ago         118MB
```

运行容器

```shell
$ sudo mkdir /opt/db/redis/{conf,data} -p
$ sudo chmod 777 /opt/db/redis/
$ wget https://raw.githubusercontent.com/antirez/redis/4.0/redis.conf -O /opt/db/redis/conf/redis.conf
$ sed -i 's/logfile ""/logfile "access.log"/' /opt/db/redis/conf/redis.conf
$ sed -i 's/# requirepass foobared/requirepass 123456/' /opt/db/redis/conf/redis.conf
$ sed -i 's/appendonly no/appendonly yes/' /opt/db/redis/conf/redis.conf
$ sudo docker run -p 6379:6379 --name redis -v /opt/db/redis/data:/data -v /opt/db/redis/conf/redis.conf:/etc/redis/redis.conf:ro --privileged=true  -d redis redis-server /etc/redis/redis.conf 
```

命令说明：

**-p 6379:6379 :** 将容器的6379端口映射到主机的6379端口

**-v /data/redis:/data :** 将主机中/data/redis挂载到容器的/data

**redis-server --appendonly yes :** 在容器执行redis-server启动命令，并打开redis持久化配置

查看容器启动情况

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
63290ba5e80c        redis:latest        "docker-entrypoint.s…"   45 minutes ago      Up 45 minutes       0.0.0.0:6379->6379/tcp     condescending_hugl
```

连接、查看容器

使用redis镜像执行redis-cli命令连接到刚启动的容器,主机IP为172.17.0.1

```
$ docker exec -it 63290ba5e80c redis-cli -a 123456
```

**执行文件**

```shell
$ tree /opt/redis/
/opt/redis/
├── bin
│   ├── redis-benchmark #redis的性能测试工具
│   ├── redis-check-aof #Redis AOF 持久化文件检测和修复工具
│   ├── redis-check-rdb #Redis RDB 持久化文件检测和修复工具
│   ├── redis-cli #redis的客户端
│   ├── redis-sentinel -> redis-server #redis的sentinel(哨兵)服务器
│   └── redis-server #redis的服务器端
└── conf
    └── redis.conf
```

**redis服务端**

启动方式：

1. 默认启动

   ```
   $ ./redis-server 
   ```

2. 动态参数启动（变更端口号,所有参数都可覆盖）

   ```
   $ ./redis-server --port 6389
   ```

3. 配置文件启动

   ```
   $ ./redis-server  ../conf/redis.conf
   ```

常用命令：

```shell
$ redis-server -v
Redis server v=5.0.5 sha=00000000:0 malloc=jemalloc-5.1.0 bits=64 build=ed73b6e722d440f1

$ redis-server --help
Usage: ./redis-server [/path/to/redis.conf] [options]
       ./redis-server - (read config from stdin)
       ./redis-server -v or --version
       ./redis-server -h or --help
       ./redis-server --test-memory <megabytes>

Examples:
       ./redis-server (run the server with default conf)
       ./redis-server /etc/redis/6379.conf
       ./redis-server --port 7777
       ./redis-server --port 7777 --replicaof 127.0.0.1 8888
       ./redis-server /etc/myredis.conf --loglevel verbose

Sentinel mode:
       ./redis-server /etc/sentinel.conf --sentinel

$ ./redis-server --test-memory 1024 #测试内存
```

**redis客户端**

[redis命令](https://www.redis.net.cn/order/)

```shell
$ ./redis-cli -h 127.0.0.1 -p 6379 -a 123456 #连接
127.0.0.1:6379> info #相关信息
# Server
redis_version:5.0.5
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:ed73b6e722d440f1
redis_mode:standalone
os:Linux 3.10.0-1062.4.1.el7.x86_64 x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:4.8.5
process_id:19967
run_id:dbc20a116cd5f2ce80f03d8c3ba9e2759580daf4
tcp_port:6379
uptime_in_seconds:58
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:14066162
executable:/root/redis-5.0.5/redis-server
config_file:/opt/redis/conf/redis.conf

# Clients
connected_clients:1
client_recent_max_input_buffer:2
client_recent_max_output_buffer:0
blocked_clients:0
...


```

其他命令

1. auth  password    //验证密码 

2. echo  message    //打印文本 

3. ping   //测试连接，ping一下Redis服务器，如果连接正常（已连接到Redis服务器）返回PONG。 

   ```
   127.0.0.1:6379> ping
   PONG
   ```

4. select  dbIndex    //选择当前使用的数据库，默认使用数据库0，下标，从0开始。 

5. quit   //关闭当前连接，并退到上一级命令行 

6. time   //获取服务器上的当前时间。

   ```
   127.0.0.1:6379> time
   1) "1574347930"
   2) "298528"
   ```

   有2个返回值，第一个是当前时间的时间戳（s），第二个是当前这一秒已经逝去的微秒数。

   1秒=10^3毫秒=10^6微秒。

   两者组合可显示微秒级的时间。

注意：

有时候会有中文乱码。要在 redis-cli 后面加上 --raw

```
redis-cli --raw
```

就可以避免中文乱码了。

**服务端关闭方式**

通过redis-cli连接服务器后执行shutdown命令，则执行停止redis服务操作。

可以使用shutdown命令关闭redis服务器外，还可以使用kill+进程号的方式关闭redis服务。

不要使用Kill 9方式关闭redis进程，这样redis不会进行持久化操作，除此之外，还会造成缓冲区等资源不能优雅关闭，极端情况下会造成AOF和复制丢失数据的情况

```shell
./redis-cli -h 127.0.0.1 -p 6379 -a 123456 shutdown [save|nosave]
```

### 压力测试

> Redis 自带了一个叫 `redis-benchmark` 的工具来模拟 N 个客户端同时发出 M 个请求。 （类似于 Apache ab 程序）。你可以使用 redis-benchmark -h 来查看基准参数。

```tsx
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]
 
 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 2)
 -dbnum <db>        SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

常用参数

| 选项  | 描述                                    | 默认值    |
| ----- | --------------------------------------- | --------- |
| -h    | 指定服务器主机名                        | 127.0.0.1 |
| -p    | 指定服务器端口                          | 6379      |
| -s    | 指定服务器socket                        |           |
| -c    | 指定并发连接数                          | 50        |
| -n    | 指定请求数                              | 10000     |
| -d    | 以字节的形式指定 SET/GET 值的数据大小   | 2         |
| -k    | 1=keepalive ,0=reconnect                | 1         |
| -r    | SET/GET/incr 使用随机key,SADD使用随机值 |           |
| -P    | 通过管道传输 <numreq\>                  | 1         |
| -q    | 强制退出redis,仅显示query/sec 值        |           |
| --csv | 以CSV格式输出                           |           |
| -l    | 生成循环，永久执行测试                  |           |
| -t    | 仅运行以逗号分隔的测试命令列表          |           |
| -I    | Idle 模式，仅打开N个idle 连接并等待     |           |

压测命令

```shell
$ redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000 -t get
$ redis-benchmark -t set,lpush -n 100000 -q

SET: 83822.30 requests per second
LPUSH: 86505.19 requests per second

$ redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"

script load redis.call('set','foo','bar'): 79365.08 requests per second

$ redis-benchmark -r 1000000 -n 2000000 -t get,set,lpush,lpop -q -P 16
SET: 367647.06 requests per second
GET: 528680.94 requests per second
LPUSH: 388274.12 requests per second
LPOP: 389787.56 requests per second
```

**选择测试键的范围大小**
假设我们想设置 10 万随机 key 连续 SET 100 万次，我们可以使用下列的命令

```shell
$ redis-benchmark -h 127.0.0.1 -p 6379 -t set -r 100000 -n 1000000
====== SET ======
  1000000 requests completed in 16.68 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

94.40% <= 1 milliseconds
99.10% <= 2 milliseconds
99.65% <= 3 milliseconds
99.86% <= 4 milliseconds
99.93% <= 5 milliseconds
99.97% <= 6 milliseconds
99.99% <= 7 milliseconds
99.99% <= 8 milliseconds
99.99% <= 9 milliseconds
100.00% <= 21 milliseconds
100.00% <= 22 milliseconds
100.00% <= 22 milliseconds
59962.82 requests per second
```

**使用 pipelining**

默认情况下，每个客户端都是在一个请求完成之后才发送下一个请求 （benchmark 会模拟 50 个客户端除非使用 -c 指定特别的数量）， 这意味着服务器几乎是按顺序读取每个客户端的命令。Also RTT is payed as well.

 Redis pipelining 可以提高服务器的 TPS ，记得在多条命令需要处理时候使用 pipelining。

```shell
$ redis-benchmark -n 10000 -t set,get -P 16 -q

SET: 250000.00 requests per second
GET: 526315.81 requests per second
```

### 配置文件参数

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

    ```
    daemonize no
    ```

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

    ```
    pidfile /var/run/redis.pid
    ```

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

    ```
    port 6379
    ```

4. 绑定的主机地址,注释掉则开放远程访问

    ```
    bind 127.0.0.1
    ```

5. 当客户端闲置多长时间(s)后关闭连接，如果指定为0，表示关闭该功能

   ```
   timeout 300
   ```

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

    ```
    loglevel verbose
    ```

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

    ```
    logfile stdout
    ```

8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid\>命令在连接上指定数据库id

    ```
    databases 16
    ```

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

    ```
save <seconds> <changes>
    ```

    Redis默认配置文件中提供了三个条件：

    ```
save 900 1
save 300 10
save 60 10000
    ```
    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

    ```
    rdbcompression yes
    ```

11. 指定本地数据库文件名，默认值为dump.rdb

    ```
    dbfilename dump.rdb
    ```

12. 指定本地数据库存放目录

    ```
    dir ./
    ```

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

    ```
    slaveof <masterip> <masterport>
    ```

14. 当master服务设置了密码保护时，slav服务连接master的密码

    ```
    masterauth <master-password>
    ```

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password\>命令提供密码，默认关闭

    ```
    requirepass foobared
    ```

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

    ```
    maxclients 128
    ```

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

    ```
    maxmemory <bytes>
    ```

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

    ```
    appendonly no
    ```

19. 指定更新日志文件名，默认为appendonly.aof

    ```
     appendfilename appendonly.aof
    ```

20. 指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折衷，默认值）

    ```
    appendfsync everysec
    ```

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

    ```
    vm-enabled no
    ```

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

    ```
     vm-swap-file /tmp/redis.swap
    ```

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

    ```
     vm-max-memory 0
    ```

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

    ```
     vm-page-size 32
    ```

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

    ```
     vm-pages 134217728
    ```

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

    ```
     vm-max-threads 4
    ```

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

    ```
    glueoutputbuf yes
    ```

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

    ```
    hash-max-zipmap-entries 64
    
    hash-max-zipmap-value 512
    ```

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

    ```
    activerehashing yes
    ```

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

    ```
    include /path/to/local.conf
    ```



### 参考

1. [Redis详解（二）------ redis的配置文件介绍](https://www.cnblogs.com/ysocean/p/9074787.html)
2. [Redis学习笔记----Redis5.0.5配置文件详解](https://blog.csdn.net/weixin_42425970/article/details/94132652)