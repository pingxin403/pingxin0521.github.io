---
title: MySQL 架构和底层知识
date: 2019-10-06 13:18:59
tags:
 - 数据库
 - MySQL
categories:
 - 数据库
 - MySQL
---

#### MySQL逻辑架构

![MCP6Lq.png](https://s2.ax1x.com/2019/11/06/MCP6Lq.png)

<!--more-->

MySQL逻辑架构整体分为三层 :

1. 客户端 : 并非MySQL所独有，诸如 : 连接处理、授权认证、安全等功能均在这一层处理
2. 核心服务 : 包括查询解析、分析、优化、缓存、内置函数(比如 : 时间、数学、加密等函数)，所有的跨存储引擎的功能也在这一层实现 : 存储过程、触发器、视图等
3. 存储引擎 : 负责 MySQL 中的数据存储和提取，和 Linux 下的文件系统类似，每种存储引擎都有其优势和劣势，中间的服务层通过 API 与存储引擎通信，这些 API接口 屏蔽不同存储引擎间的差异

#### MySQL基架

![1zrInP.png](https://s2.ax1x.com/2020/02/15/1zrInP.png)

MySQL基架大致包括如下几大模块组件：

1. MySQL向外提供的交互接口（Connectors）

    Connectors组件，是MySQL向外提供的交互组件，如java,.net,php等语言可以通过该组件来操作SQL语句，实现与SQL的交互。

2. 管理服务组件和工具组件(Management Service & Utilities)

    提供对MySQL的集成管理，如备份(Backup),恢复(Recovery),安全管理(Security)等

3. 连接池组件(Connection Pool)

   负责监听对客户端向MySQL Server端的各种请求，接收请求，转发请求到目标模块。每个成功连接MySQL Server的客户请求都会被

   创建或分配一个线程，该线程负责客户端与MySQL Server端的通信，接收客户端发送的命令，传递服务端的结果信息等。

4. SQL接口组件(SQL Interface)

   接收用户SQL命令，如DML,DDL和存储过程等，并将最终结果返回给用户。

5. 查询分析器组件(Parser)

    首先分析SQL命令语法的合法性，并尝试将SQL命令分解成数据结构，若分解失败，则提示SQL语句不合理。

6. 优化器组件（Optimizer）

   对SQL命令按照标准流程进行优化分析。

7. 缓存主件（Caches & Buffers）

   缓存和缓冲组件

8. 插件式存储引擎（Pluggable Storage Engines）

9. 物理文件（File System）

   实际存储MySQL 数据库文件和一些日志文件等的系统，如Linux，Unix,Windows等。



#### MySQL查询过程

![MCPbex.png](https://s2.ax1x.com/2019/11/06/MCPbex.png)

MySQL 整个查询执行过程，总的来说分为 5 个步骤 :

1. 客户端向 MySQL 服务器发送一条查询请求
2. 服务器首先检查查询缓存，如果命中缓存，则立刻返回存储在缓存中的结果，否则进入下一阶段
3. 服务器进行 SQL解析、预处理、再由优化器生成对应的执行计划
4. MySQL 根据执行计划，调用存储引擎的 API来执行查询
5. 将结果返回给客户端，同时缓存查询结果

**客户端/服务端通信协议**

MySQL客户端/服务端通信协议 是 “半双工” 的，在任一时刻，要么是服务器向客户端发送数据，要么是客户端向服务器发送数据，这两个动作不能同时发生。一旦一端开始发送消息，另一端要接收完整个消息才能响应它，所以**无法也无须将一个消息切成小块独立发送，也没有办法进行流量控制**。客户端用一个单独的数据包将查询请求发送给服务器，所以当查询语句很长的时候，需要设置 max_allowed_packet参数，如果查询实在是太大，服务端会拒绝接收更多数据并抛出异常。与之相反的是，服务器响应给用户的数据通常会很多，由多个数据包组成。但是当服务器响应客户端请求时，客户端必须完整的接收整个返回结果，而不能简单的只取前面几条结果，然后让服务器停止发送。因而在实际开发中，尽量保持查询简单且只返回必需的数据，减小通信间数据包的大小和数量是一个非常好的习惯，这也是查询中尽量避免使用 SELECT * 以及加上 LIMIT 限制的原因之一

#### 查询缓存

mysql缓存机制就是缓存sql 文本及缓存结果，用KV形式保存再服务器内存中，如果运行相同的sql,服务器直接从缓存中去获取结果，不需要在再去解析、优化、执行sql。 如果这个表修改了，那么使用这个表中的所有缓存将不再有效，查询缓存值得相关条目将被清空。表中得任何改变是值表中任何数据或者是结构的改变，包括insert,update,delete,truncate,alter table,drop table或者是drop database 包括那些映射到改变了的表的使用merge表的查询，显然，者对于频繁更新的表，查询缓存不合适，对于一些不变的数据且有大量相同sql查询的表，查询缓存会节省很大的性能。

**命中条件**

缓存存在一个hash表中，通过**查询SQL，查询数据库，客户端协议等作为key**,在判断命中前，mysql不会解析SQL，而是使用SQL去查询缓存，**SQL上的任何字符的不同，如空格，注释，都会导致缓存不命中**。如果查询有不确定的数据like now(),current_date()，那么查询完成后结果者不会被缓存，包含不确定的数的是不会放置到缓存中。

**工作流程**

1. 服务器接收SQL，以SQL和一些其他条件为key查找缓存表
2. 如果找到了缓存，则直接返回缓存
3. 如果没有找到缓存，则执行SQL查询，包括原来的SQL解析，优化等。
4. 执行完SQL查询结果以后，将SQL查询结果缓存入缓存表

**缓存失败**

当某个表正在写入数据，则这个表的缓存（命中缓存，缓存写入等）将会处于失效状态，在Innodb中，如果某个事务修改了这张表，则这个表的缓存在事务提交前都会处于失效状态，在这个事务提交前，这个表的相关查询都无法被缓存。

**缓存的内存管理**

缓存会在内存中开辟一块内存（query_cache_size）来维护缓存数据，其中大概有40K的空间是用来维护缓存数据的元数据的，例如空间内存，例如空间内存，数据表和查询结果映射，SQL和查询结果映射的。

mysql将这个大内存块分为小内存块（query_cache_min_res_unit),每个小块中存储自身的类型、大小和查询结果数据，还有前后内存块的指针。

mysql需要设置单个小存储块大小，在SQL查询开始（还未得到结果）时就去申请一块内存空间，所以即使你的缓存数据没有达到这个大小也需要这个大小的数据块去保存（like linux filesystem’s block)。如果超出这个内存块的大小，则需要再申请一个内存块。当查询完成发现申请的内存有富余，则会将富余的内存空间是放点，这就会造成内存碎片的问题，见下图

![MCiXBq.png](https://s2.ax1x.com/2019/11/06/MCiXBq.png)

**缓存的使用时机**

衡量打开缓存是否对系统有性能提升是一个很难的话题

1. 通过缓存命中率判断, 缓存命中率 = 缓存命中次数 (Qcache_hits) / 查询次数 (Com_select)
2. 通过缓存写入率, 写入率 = 缓存写入次数 (Qcache_inserts) / 查询次数 (Qcache_inserts)
3. 通过 命中-写入率 判断, 比率 = 命中次数 (Qcache_hits) / 写入次数 (Qcache_inserts), 高性能MySQL中称之为比较能反映性能提升的指数,一般来说达到3:1则算是查询缓存有效,而最好能够达到10:1

**缓存参数配置**

1. query_cache_type: 是否打开缓存
   可选项
   1) OFF: 关闭
   2) ON: 总是打开
   3) DEMAND: 只有明确写了SQL_CACHE的查询才会吸入缓存
2. query_cache_size: 缓存使用的总内存空间大小,单位是字节,这个值必须是1024的整数倍,否则MySQL实际分配可能跟这个数值不同(感觉这个应该跟文件系统的blcok大小有关)
3. query_cache_min_res_unit: 分配内存块时的最小单位大小
4. query_cache_limit: MySQL能够缓存的最大结果,如果超出,则增加 Qcache_not_cached的值,并删除查询结果
5. query_cache_wlock_invalidate: 如果某个数据表被锁住,是否仍然从缓存中返回数据,默认是OFF,表示仍然可以返回

GLOBAL STAUS 中 关于 缓存的参数解释:

```
Qcache_free_blocks: 缓存池中空闲块的个数
Qcache_free_memory: 缓存中空闲内存量
Qcache_hits: 缓存命中次数
Qcache_inserts: 缓存写入次数
Qcache_lowmen_prunes: 因内存不足删除缓存次数
Qcache_not_cached: 查询未被缓存次数,例如查询结果超出缓存块大小,查询中包含可变函数等
Qcache_queries_in_cache: 当前缓存中缓存的SQL数量
Qcache_total_blocks: 缓存总block数
```

**减少碎片策略**

1. 选择合适的block大小

2. 使用 FLUSH QUERY CACHE 命令整理碎片.这个命令在整理缓存期间,会导致其他连接无法使用查询缓存
   PS: 清空缓存的命令式 RESET QUERY CACHE

   ![MCFQvd.png](https://s2.ax1x.com/2019/11/06/MCFQvd.png)

#### mysqlslap 性能测试工具

**常用参数**

```
–concurrency #代表并发数量，多个可以用逗号隔开。例如：–concurrency=50,200,500
–engines #代表要测试的引擎，可以有多个，用分隔符隔开。例如：–engines=myisam,innodb,memory
–iterations #代表要在不同并发环境下，各自运行测试多少次。
–auto-generate-sql #代表用mysqlslap工具自己生成的SQL脚本来测试并发压力。
–auto-generate-sql-add-auto-increment #代表对生成的表自动添加auto_increment列，从5.1.18版本开始，
–auto-generate-sql-load-type #代表要测试的环境是读操作还是写操作还是两者混合的（read,write,update,mixed）
–number-of-queries #代表总共要运行多少条查询。
–debug-info #代表要额外输出CPU以及内存的相关信息。
–number-int-cols #代表示例表中的INTEGER类型的属性有几个。
–number-char-cols #代表示例表中的vachar类型的属性有几个。
–create-schema #代表自定义的测试库名称。
–query #代表自定义的测试SQL脚本。
```

测试同时不同的存储引擎的性能进行对比：并发50-100，1000次查询

- default

  ```shell
  mysqlslap -a --concurrency=50,100 --number-of-queries 1000 --iterations=5 --engine=myisam,innodb --debug-info
  mysqlslap -a --concurrency=50,100 --number-of-queries 3000 --iterations=5 --auto-generate-sql --auto-generate-sql-add-auto-increment --engine=ndbcluster --debug-info
  ```

- mixed

  ```shell
  mysqlslap --defaults-file=/etc/my.cnf --concurrency=100,200,400 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=mixed --engine=ndbcluster --number-of-queries=3000000 --debug-info
  mysqlslap --defaults-file=/etc/my.cnf --concurrency=500 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=mixed --engine=ndbcluster --number-of-queries=3000000 --debug-info
  ```

- write

  ```shell
  mysqlslap --defaults-file=/etc/my.cnf --concurrency=500 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=write --engine=ndbcluster --number-of-queries=3000000 --debug-info
  mysqlslap --defaults-file=/etc/my.cnf --concurrency=500,600,700,800,900 --iterations=1 --number-int-cols=4 --number-char-cols=35 --auto-generate-sql --auto-generate-sql-add-autoincrement --auto-generate-sql-load-type=write --engine=ndbcluster --number-of-queries=3000000 --debug-info
  ```

  

