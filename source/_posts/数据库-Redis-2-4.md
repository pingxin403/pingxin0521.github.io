---
title: Redis 高可用
date: 2019-10-20 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### 高可用技术

Redis的几种常见使用方式包括：

- Redis单副本；
- Redis多副本（主从）；
- Redis Sentinel（哨兵）；
- Redis Cluster；

- Redis自研。

<!--more-->

#### 各种使用方式的优缺点

##### Redis单副本

Redis单副本，采用单个Redis节点部署架构，没有备用节点实时同步数据，不提供数据持久化和备份策略，适用于数据可靠性要求不高的纯缓存业务场景。

![UTOOLS1575358208054.png](https://i.loli.net/2019/12/03/djNo2OaM8wlTq4e.png)

**优点：**

- 架构简单，部署方便；
- 高性价比：缓存使用时无需备用节点（单实例可用性可以用supervisor或crontab保证），当然为了满足业务的高可用性，也可以牺牲一个备用节点，但同时刻只有一个实例对外提供服务；
- 高性能。

**缺点：**

- 不保证数据的可靠性；
- 在缓存使用，进程重启后，数据丢失，即使有备用的节点解决高可用性，但是仍然不能解决缓存预热问题，因此不适用于数据可靠性要求高的业务；
- 高性能受限于单核CPU的处理能力（Redis是单线程机制），CPU为主要瓶颈，所以适合操作命令简单，排序、计算较少的场景。也可以考虑用Memcached替代。

##### Redis多副本（主从）

Redis多副本，采用主从（replication）部署结构，相较于单副本而言最大的特点就是主从实例间数据实时同步，并且提供数据持久化和备份策略。主从实例部署在不同的物理服务器上，根据公司的基础环境配置，可以实现同时对外提供服务和读写分离策略。

![UTOOLS1575358645071.png](https://i.loli.net/2019/12/03/nUJEL3qWs7Imxiz.png)

**优点：**

- 高可靠性：一方面，采用双机主备架构，能够在主库出现故障时自动进行主备切换，从库提升为主库提供服务，保证服务平稳运行；另一方面，开启数据持久化功能和配置合理的备份策略，能有效的解决数据误操作和数据异常丢失的问题；
- 读写分离策略：从节点可以扩展主库节点的读能力，有效应对大并发量的读操作。

**缺点：**

- 故障恢复复杂，如果没有RedisHA系统（需要开发），当主库节点出现故障时，需要手动将一个从节点晋升为主节点，同时需要通知业务方变更配置，并且需要让其它从库节点去复制新主库节点，整个过程需要人为干预，比较繁琐；
- 主库的写能力受到单机的限制，可以考虑分片；
- 主库的存储能力受到单机的限制，可以考虑Pika；
- 原生复制的弊端在早期的版本中也会比较突出，如：Redis复制中断后，Slave会发起psync，此时如果同步不成功，则会进行全量同步，主库执行全量备份的同时可能会造成毫秒或秒级的卡顿；又由于COW机制，导致极端情况下的主库内存溢出，程序异常退出或宕机；主库节点生成备份文件导致服务器磁盘IO和CPU（压缩）资源消耗；发送数GB大小的备份文件导致服务器出口带宽暴增，阻塞请求，建议升级到最新版本。

##### Redis Sentinel（哨兵）

Redis Sentinel是社区版本推出的原生高可用解决方案，其部署架构主要包括两部分：Redis Sentinel集群和Redis数据集群。

其中Redis Sentinel集群是由若干Sentinel节点组成的分布式集群，可以实现故障发现、故障自动转移、配置中心和客户端通知。Redis Sentinel的节点数量要满足2n+1（n>=1）的奇数个。

![UTOOLS1575358602666.png](https://i.loli.net/2019/12/03/cY7p8sMUWNoea4b.png)

![UTOOLS1575358661383.png](https://i.loli.net/2019/12/03/yYxFdSRGJD5oZwu.png)

**优点**：

- Redis Sentinel 集群部署简单；
- 能够解决 Redis 主从模式下的高可用切换问题；
- 很方便实现 Redis 数据节点的线形扩展，轻松突破 Redis 自身单线程瓶颈，可极大满足 Redis 大容量或高性能的业务需求；
- 可以实现一套 Sentinel 监控一组 Redis 数据节点或多组数据节点。

**缺点**：

- 部署相对 Redis 主从模式要复杂一些，原理理解更繁琐；
- 资源浪费，Redis 数据节点中 slave 节点作为备份节点不提供服务；
- Redis Sentinel 主要是针对 Redis 数据节点中的主节点的高可用切换，对 Redis 的数据节点做失败判定分为主观下线和客观下线两种，对于 Redis 的从节点有对节点做主观下线操作，并不执行故障转移。
- 不能解决读写分离问题，实现起来相对复杂。

**建议**：

- 如果监控同一业务，可以选择一套 Sentinel 集群监控多组 Redis 数据节点的方案，反之选择一套 Sentinel 监控一组 Redis 数据节点的方案。

- sentinel monitor配置中的建议设置成 Sentinel 节点的一半加 1，当 Sentinel 部署在多个 IDC 的时候，单个 IDC 部署的 Sentinel 数量不建议超过（Sentinel 数量 – quorum）。

- 合理设置参数，防止误切，控制切换灵敏度控制：

  ```
  a. quorum
  b. down-after-milliseconds 30000
  c. failover-timeout 180000
  d. maxclient
  e. timeout
  ```

- 部署的各个节点服务器时间尽量要同步，否则日志的时序性会混乱。

- Redis 建议使用 pipeline 和 multi-keys 操作，减少 RTT 次数，提高请求效率。

- 自行搞定配置中心（zookeeper），方便客户端对实例的链接访问。

##### Redis Cluster

Redis Cluster 是社区版推出的 Redis 分布式集群解决方案，主要解决 Redis 分布式方面的需求，比如，当遇到单机内存，并发和流量等瓶颈的时候，Redis Cluster 能起到很好的负载均衡的目的。

Redis Cluster 集群节点最小配置 6 个节点以上（3 主 3 从），其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用。

Redis Cluster 采用虚拟槽分区，所有的键根据哈希函数映射到 0～16383 个整数槽内，每个节点负责维护一部分槽以及槽所印映射的键值数据。

![UTOOLS1575358759884.png](https://i.loli.net/2019/12/03/4qbxVyYSDwvQAIi.png)

**优点**：

- 无中心架构；
- 数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布；
- 可扩展性：可线性扩展到 1000 多个节点，节点可动态添加或删除；
- 高可用性：部分节点不可用时，集群仍可用。通过增加 Slave 做 standby 数据副本，能够实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave 到 Master 的角色提升；
- 降低运维成本，提高系统的扩展性和可用性。

**缺点**：

- Client 实现复杂，驱动要求实现 Smart Client，缓存 slots mapping 信息并及时更新，提高了开发难度，客户端的不成熟影响业务的稳定性。目前仅 JedisCluster 相对成熟，异常处理部分还不完善，比如常见的“max redirect exception”。
- 节点会因为某些原因发生阻塞（阻塞时间大于 clutser-node-timeout），被判断下线，这种 failover 是没有必要的。
- 数据通过异步复制，不保证数据的强一致性。
- 多个业务使用同一套集群时，无法根据统计区分冷热数据，资源隔离性较差，容易出现相互影响的情况。
- Slave 在集群中充当“冷备”，不能缓解读压力，当然可以通过 SDK 的合理设计来提高 Slave 资源的利用率。
- Key 批量操作限制，如使用 mset、mget 目前只支持具有相同 slot 值的 Key 执行批量操作。对于映射为不同 slot 值的 Key 由于 Keys 不支持跨 slot 查询，所以执行 mset、mget、sunion 等操作支持不友好。
- Key 事务操作支持有限，只支持多 key 在同一节点上的事务操作，当多个 Key 分布于不同的节点上时无法使用事务功能。
- Key 作为数据分区的最小粒度，不能将一个很大的键值对象如 hash、list 等映射到不同的节点。
- 不支持多数据库空间，单机下的 redis 可以支持到 16 个数据库，集群模式下只能使用 1 个数据库空间，即 db 0。
- 复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。
- 避免产生 hot-key，导致主库节点成为系统的短板。
- 避免产生 big-key，导致网卡撑爆、慢查询等。
- 重试时间应该大于 cluster-node-time 时间。
- Redis Cluster 不建议使用 pipeline 和 multi-keys 操作，减少 max redirect 产生的场景。

##### Redis 自研

Redis 自研的高可用解决方案，主要体现在配置中心、故障探测和 failover 的处理机制上，通常需要根据企业业务的实际线上环境来定制化。

![UTOOLS1575358834108.png](https://i.loli.net/2019/12/03/s4wiHUMfYOryu2L.png)

![UTOOLS1575358847433.png](https://i.loli.net/2019/12/03/lRYtsOdfSiXPFbw.png)

**优点**：

- 高可靠性、高可用性；
- 自主可控性高；
- 贴切业务实际需求，可缩性好，兼容性好。

**缺点**：

- 实现复杂，开发成本高；
- 需要建立配套的周边设施，如监控，域名服务，存储元数据信息的数据库等；
- 维护成本高。

### Redis集群模式

Redis Cluster 集群模式通常具有 **高可用**、**可扩展性**、**分布式**、**容错** 等特性。Redis 分布式方案一般有两种：

1. 客户端分区方案

   **客户端** 就已经决定数据会被 **存储** 到哪个 redis 节点或者从哪个 redis 节点 **读取数据**。其主要思想是采用 **哈希算法** 将 Redis 数据的 key 进行散列，通过 hash 函数，特定的 key会 **映射** 到特定的 Redis 节点上。

   ![UTOOLS1575358941014.png](https://i.loli.net/2019/12/03/uQbc53YUK7JZF8r.png)

   **客户端分区方案** 的代表为 Redis Sharding，Redis Sharding 是 Redis Cluster 出来之前，业界普遍使用的 Redis **多实例集群** 方法。Java 的 Redis 客户端驱动库 Jedis，支持 Redis Sharding 功能，即 ShardedJedis 以及 **结合缓存池** 的 ShardedJedisPool。

   **优点**

   不使用 **第三方中间件**，**分区逻辑** 可控，**配置** 简单，节点之间无关联，容易 **线性扩展**，灵活性强。

   **缺点**

   **客户端** 无法 **动态增删** 服务节点，客户端需要自行维护 **分发逻辑**，客户端之间 **无连接共享**，会造成 **连接浪费**。

2. 代理分区方案

   **客户端** 发送请求到一个 **代理组件**，**代理** 解析 **客户端** 的数据，并将请求转发至正确的节点，最后将结果回复给客户端。

   **优点**：简化 **客户端** 的分布式逻辑，**客户端** 透明接入，切换成本低，代理的 **转发** 和 **存储** 分离。

   **缺点**：多了一层 **代理层**，加重了 **架构部署复杂度** 和 **性能损耗**。

   ![UTOOLS1575359012485.png](https://i.loli.net/2019/12/03/k9ouL3VFEHNG8YD.png)

**代理分区** 主流实现的有方案有 Twemproxy 和 Codis。

1. Twemproxy

   Twemproxy 也叫 nutcraker，是 twitter 开源的一个 redis 和 memcache 的 **中间代理服务器** 程序。Twemproxy 作为 **代理**，可接受来自多个程序的访问，按照 **路由规则**，转发给后台的各个 Redis 服务器，再原路返回。Twemproxy 存在 **单点故障** 问题，需要结合 Lvs 和 Keepalived 做 **高可用方案**。

   **优点**：应用范围广，稳定性较高，中间代理层 **高可用**。

   **缺点**：无法平滑地 **水平扩容/缩容**，无 **可视化管理界面**，运维不友好，出现故障，不能 **自动转移**。

2. [Codis](https://github.com/CodisLabs/codis)

   Codis 是一个 **分布式** Redis 解决方案，对于上层应用来说，连接 Codis-Proxy 和直接连接 **原生的** Redis-Server 没有的区别。Codis 底层会 **处理请求的转发**，不停机的进行 **数据迁移** 等工作。Codis 采用了无状态的 **代理层**，对于 **客户端** 来说，一切都是透明的。

   **优点**

   实现了上层 Proxy 和底层 Redis 的 **高可用**，**数据分片** 和 **自动平衡**，提供 **命令行接口** 和 RESTful API，提供 **监控** 和 **管理** 界面，可以动态 **添加** 和 **删除** Redis 节点。

   **缺点**

   **部署架构** 和 **配置** 复杂，不支持 **跨机房** 和 **多租户**，不支持 **鉴权管理**。

##### 查询路由方案

**客户端随机地** 请求任意一个 Redis 实例，然后由 Redis 将请求 **转发** 给 **正确** 的 Redis 节点。Redis Cluster 实现了一种 **混合形式** 的 **查询路由**，但并不是 **直接** 将请求从一个 Redis 节点 **转发** 到另一个 Redis 节点，而是在 **客户端** 的帮助下直接 **重定向**（ redirected）到正确的 Redis 节点。



**优点**

**无中心节点**，数据按照 **槽** 存储分布在多个 Redis 实例上，可以平滑的进行节点 **扩容/缩容**，支持 **高可用** 和 **自动故障转移**，运维成本低。

**缺点**

严重依赖 Redis-trib 工具，缺乏 **监控管理**，需要依赖 Smart Client (**维护连接**，**缓存路由表**，MultiOp 和 Pipeline 支持)。Failover 节点的 **检测过慢**，不如 **中心节点** ZooKeeper 及时。Gossip 消息具有一定开销。无法根据统计区分 **冷热数据**。

#### 数据分布

##### 数据分布理论

**分布式数据库** 首先要解决把 **整个数据集** 按照 **分区规则** 映射到 **多个节点** 的问题，即把 **数据集** 划分到 **多个节点** 上，每个节点负责 **整体数据** 的一个 **子集**。



数据分布通常有 **哈希分区** 和 **顺序分区** 两种方式，对比如下：

分区方式 特点 相关产品 哈希分区 离散程度好，数据分布与业务无关，无法顺序访问 Redis Cluster，Cassandra，Dynamo 顺序分区 离散程度易倾斜，数据分布与业务相关，可以顺序访问 BigTable，HBase，Hypertable 由于 Redis Cluster 采用 **哈希分区规则**，这里重点讨论 **哈希分区**。常见的 **哈希分区** 规则有几种，下面分别介绍：

1. 节点取余分区

   使用特定的数据，如 Redis 的 **键** 或 **用户** ID，再根据 **节点数量** N 使用公式：hash（key）% N 计算出 **哈希值**，用来决定数据 **映射** 到哪一个节点上。

   

   **优点**

   这种方式的突出优点是 **简单性**，常用于 **数据库** 的 **分库分表规则**。一般采用 **预分区** 的方式，提前根据 **数据量** 规划好 **分区数**，比如划分为 512 或 1024 张表，保证可支撑未来一段时间的 **数据容量**，再根据 **负载情况** 将 **表** 迁移到其他 **数据库** 中。扩容时通常采用 **翻倍扩容**，避免 **数据映射** 全部被 **打乱**，导致 **全量迁移** 的情况。

   **缺点**

   当 **节点数量** 变化时，如 **扩容** 或 **收缩** 节点，数据节点 **映射关系** 需要重新计算，会导致数据的 **重新迁移**。

2. 一致性哈希分区

   **一致性哈希** 可以很好的解决 **稳定性问题**，可以将所有的 **存储节点** 排列在 **收尾相接** 的 Hash 环上，每个 key 在计算 Hash 后会 **顺时针** 找到 **临接** 的 **存储节点** 存放。而当有节点 **加入** 或 **退出** 时，仅影响该节点在 Hash 环上 **顺时针相邻** 的 **后续节点**。

   

   **优点**

   **加入** 和 **删除** 节点只影响 **哈希环** 中 **顺时针方向** 的 **相邻的节点**，对其他节点无影响。

   **缺点**

   **加减节点** 会造成 **哈希环** 中部分数据 **无法命中**。当使用 **少量节点** 时，**节点变化** 将大范围影响 **哈希环** 中 **数据映射**，不适合 **少量数据节点** 的分布式方案。**普通** 的 **一致性哈希分区** 在增减节点时需要 **增加一倍** 或 **减去一半** 节点才能保证 **数据** 和 **负载的均衡**。

   **注意**：因为 **一致性哈希分区** 的这些缺点，一些分布式系统采用 **虚拟槽** 对 **一致性哈希** 进行改进，比如 Dynamo 系统。

3. 虚拟槽分区

   **虚拟槽分区** 巧妙地使用了 **哈希空间**，使用 **分散度良好** 的 **哈希函数** 把所有数据 **映射** 到一个 **固定范围** 的 **整数集合** 中，整数定义为 **槽**（slot）。这个范围一般 **远远大于** 节点数，比如 Redis Cluster 槽范围是 0 ~ 16383。**槽** 是集群内 **数据管理** 和 **迁移** 的 **基本单位**。采用 **大范围槽** 的主要目的是为了方便 **数据拆分** 和 **集群扩展**。每个节点会负责 **一定数量的槽**，如图所示：

   

   当前集群有 5 个节点，每个节点平均大约负责 3276 个 **槽**。由于采用 **高质量** 的 **哈希算法**，每个槽所映射的数据通常比较 **均匀**，将数据平均划分到 5 个节点进行 **数据分区**。Redis Cluster 就是采用 **虚拟槽分区**。

   **节点1**： 包含 0 到 3276 号哈希槽。

   **节点2**：包含 3277 到 6553 号哈希槽。

   **节点3**：包含 6554 到 9830 号哈希槽。

   **节点4**：包含 9831 到 13107 号哈希槽。

   **节点5**：包含 13108 到 16383 号哈希槽。

   这种结构很容易 **添加** 或者 **删除** 节点。如果 **增加** 一个节点 6，就需要从节点 1 ~ 5 获得部分 **槽** 分配到节点 6 上。如果想 **移除** 节点 1，需要将节点 1 中的 **槽** 移到节点 2 ~ 5 上，然后将 **没有任何槽** 的节点 1 从集群中 **移除** 即可。

   由于从一个节点将 **哈希槽** 移动到另一个节点并不会 **停止服务**，所以无论 **添加删除**或者 **改变** 某个节点的 **哈希槽的数量** 都不会造成 **集群不可用** 的状态.

##### Redis的数据分区

Redis Cluster 采用 **虚拟槽分区**，所有的 **键** 根据 **哈希函数** 映射到 0~16383 整数槽内，计算公式：slot = CRC16（key）& 16383。每个节点负责维护一部分槽以及槽所映射的 **键值数据**

**Redis虚拟槽分区的特点**

解耦 **数据** 和 **节点** 之间的关系，简化了节点 **扩容** 和 **收缩** 难度。

**节点自身** 维护槽的 **映射关系**，不需要 **客户端** 或者 **代理服务** 维护 **槽分区元数据**。

支持 **节点**、**槽**、**键** 之间的 **映射查询**，用于 **数据路由**、**在线伸缩** 等场景。

**Redis集群的功能限制**

Redis 集群相对 **单机** 在功能上存在一些限制，需要 **开发人员** 提前了解，在使用时做好规避。

1. key **批量操作** 支持有限。

   类似 mset、mget 操作，目前只支持对具有相同 slot 值的 key 执行 **批量操作**。对于 **映射为不同** slot 值的 key 由于执行 mget、mget 等操作可能存在于多个节点上，因此不被支持。

2. key **事务操作** 支持有限。

   只支持 **多** key 在 **同一节点上** 的 **事务操作**，当多个 key 分布在 **不同** 的节点上时 **无法** 使用事务功能。

3. key 作为 **数据分区** 的最小粒度

   不能将一个 **大的键值** 对象如 hash、list 等映射到 **不同的节点**。

4. 不支持 **多数据库空间**

   **单机** 下的 Redis 可以支持 16 个数据库（db0 ~ db15），**集群模式** 下只能使用 **一个** 数据库空间，即 db0。

5. **复制结构** 只支持一层

   **从节点** 只能复制 **主节点**，不支持 **嵌套树状复制** 结构。

#### Redis集群搭建

Redis-Cluster 是 Redis 官方的一个 **高可用** 解决方案，Cluster 中的 Redis 共有 2^14（16384） 个 slot **槽**。创建 Cluster 后，**槽** 会 **平均分配** 到每个 Redis 节点上。

下面使用docker来部署

**基本概念**

每个Redis集群中的节点都需要打开两个TCP连接。一个连接用于正常的给Client提供服务，比如6379，还有一个额外的端口（通过在这个端口号上加10000）作为数据端口，比如16379。第二个端口（本例中就是16379）用于集群总线，这是一个用二进制协议的点对点通信信道。这个集群总线（Cluster bus）用于节点的失败侦测、配置更新、故障转移授权，等等。客户端从来都不应该尝试和这些集群总线端口通信，它们只应该和正常的Redis命令端口进行通信。注意，确保在你的防火墙中开放着两个端口，否则，Redis集群节点之间将无法通信。命令端口和集群总线端口的偏移量总是10000。

Redis集群不同一致性哈希，它用一种不同的分片形式，在这种形式中，每个key都是一个概念性（**hash slot**）的一部分。Redis集群中的每个节点负责一部分hash slots，并允许添加和删除集群节点。比如，如果你想增加一个新的节点D，那么久需要从A、B、C节点上删除一些hash slot给到D。同样地，如果你想从集群中删除节点A，那么会将A上面的hash slots移动到B和C，当节点A上是空的时候就可以将其从集群中完全删除。因为将hash slots从一个节点移动到另一个节点并不需要停止其它的操作，添加、删除节点以及更改节点所维护的hash slots的百分比都不需要任何停机时间。也就是说，移动hash slots是并行的，移动hash slots不会影响其它操作。

 为了保持可用，Redis集群用一个master-slave模式，这样的话每个hash slot就有1到N个副本。而**redis-cluster规定，至少需要3个master和3个slave**，即3个master-slave对。当我们给每个master节点添加一个slave节点以后，我们的集群最终会变成由A、B、C三个master节点和A1、B1、C1三个slave节点组成，这个时候如果B失败了，系统仍然可用。节点B1是B的副本，如果B失败了，集群会将B1提升为新的master，从而继续提供服务。然而，如果B和B1同时失败了，那么整个集群将不可用。

Redis集群不能保证强一致性。换句话说，Redis集群可能会丢失一些写操作，原因是因为它用异步复制。为了使用redis-cluster，需要配置以下几个参数：

- **cluster-enabled :** 如果是yes，表示启用集群，否则以单例模式启动
- **cluster-config-file :** 可选，这不是一个用户可编辑的配置文件，这个文件是Redis集群节点自动持久化每次配置的改变，为了在启动的时候重新读取它。
- **cluster-node-timeout :** 超时时间，集群节点不可用的最大时间。如果一个master节点不可到达超过了指定时间，则认为它失败了。注意，每一个在指定时间内不能到达大多数master节点的节点将停止接受查询请求。
- **cluster-slave-validity-factor :** 如果设置为0，则一个slave将总是尝试故障转移一个master。如果设置为一个正数，那么最大失去连接的时间是node timeout乘以这个factor。
- **cluster-migration-barrier :** 一个master和slave保持连接的最小数量（即：最少与多少个slave保持连接），也就是说至少与其它多少slave保持连接的slave才有资格成为master。
- **cluster-require-full-coverage :** 如果设置为yes，这也是默认值，如果key space没有达到百分之多少时停止接受写请求。如果设置为no，将仍然接受查询请求，即使它只是请求部分key。

**准备工具**

redis 5之前的配置方法：https://my.oschina.net/dslcode/blog/1936656

下面采用5.0之后的方法

1. 安装docker

2. 在docker库获取镜像：redis；

3. 找到一份原始的redis.conf文件，将其重命名为：redis-cluster.tmpl，并配置如下几个参数，此文件的目的是生成每一个redis实例的redis.conf:

   ```properties
   # bind 127.0.0.1
   protected-mode no
   port ${PORT}
   daemonize no
   dir /data/redis
   appendonly yes
   cluster-enabled yes
   cluster-config-file nodes.conf
   cluster-node-timeout 15000
   ```

**搭建**

这里我准备的是2套环境：所有redis实例运行在同一台宿主机上；redis实例运行在不同的宿主机上。相信大家在生产环境中都应该是部署在不同的机器上，下面我将分别讲述：

1. 所有redis实例运行在同一台宿主机上

   - 由于此处所有redis实例运行在同一台宿主机，而redis-cluster之间需要通信，所以需要创建docker network

     ```shell
     # 创建docker内部网络
     docker network create redis-cluster-net
     ```

   - 创建 master 和 slave 文件夹并生成配置文件，用于存放配置文件redis.conf以及redis数据

   - 运行docker redis 的 master 和 slave 实例

   - 组装masters : slaves 节点参数

   - 创建docker-cluster

     总的命令可以用以下脚本组成：

     ```bash
     docker network create redis-cluster-net
     # 创建 master 和 slave 文件夹
     matches=""
     for port in `seq 7000 7005`; do
         ms="master"
         if [ $port -ge 7003 ]; then
             ms="slave"
         fi
         
         # 创建 master 和 slave 文件夹
         mkdir -p ./$ms/$port/ && mkdir -p ./$ms/$port/data \
         && PORT=$port envsubst < ./redis-cluster.tmpl > ./$ms/$port/redis.conf;
         
         # 运行docker redis 的 master 和 slave 实例
        docker run -d -p $port:$port -p 1$port:1$port \
         -v $PWD/$ms/$port/redis.conf:/data/redis.conf \
         -v $PWD/$ms/$port/data:/data/redis \
         --restart always --name redis-$ms-$port --net redis-cluster-net \
         redis redis-server /data/redis.conf;
        
        # 组装masters : slaves 节点参数
         matches=$matches$(docker inspect --format '{{(index .NetworkSettings.Networks "redis-cluster-net").IPAddress}}' "redis-$ms-${port}"):${port}" ";
     done
     echo $matches
     # 创建docker-cluster
     docker run -it --rm --net redis-cluster-net redis redis-cli --cluster create $matches --cluster-replicas 1
     ```

     运行结果如下：

     ```bash
     $ bash cluster.sh
     >>> Performing hash slots allocation on 6 nodes...
     Master[0] -> Slots 0 - 5460
     Master[1] -> Slots 5461 - 10922
     Master[2] -> Slots 10923 - 16383
     Adding replica 172.18.0.6:7004 to 172.18.0.2:7000
     Adding replica 172.18.0.7:7005 to 172.18.0.3:7001
     Adding replica 172.18.0.5:7003 to 172.18.0.4:7002
     M: 5005a384c9db8c3213afa630328e5331036d581b 172.18.0.2:7000
        slots:[0-5460] (5461 slots) master
     M: a66b9b4f4fdfdb29d2397fdb618f5d426b1b4c58 172.18.0.3:7001
        slots:[5461-10922] (5462 slots) master
     M: 22595186092965a48ea71d3624d320ad24bf6d64 172.18.0.4:7002
        slots:[10923-16383] (5461 slots) master
     S: fc1f933e71161e8f3942c358854e0c3d97c857b4 172.18.0.5:7003
        replicates 22595186092965a48ea71d3624d320ad24bf6d64
     S: fb6135920cba4525163b050b73407ac8440ca667 172.18.0.6:7004
        replicates 5005a384c9db8c3213afa630328e5331036d581b
     S: d923750c72bfc7dbfddc8ec9f0bef19bc5dcd4d7 172.18.0.7:7005
        replicates a66b9b4f4fdfdb29d2397fdb618f5d426b1b4c58
     Can I set the above configuration? (type 'yes' to accept): yes
     >>> Nodes configuration updated
     >>> Assign a different config epoch to each node
     >>> Sending CLUSTER MEET messages to join the cluster
     Waiting for the cluster to join
     ........
     >>> Performing Cluster Check (using node 172.18.0.2:7000)
     M: 5005a384c9db8c3213afa630328e5331036d581b 172.18.0.2:7000
        slots:[0-5460] (5461 slots) master
        1 additional replica(s)
     S: fc1f933e71161e8f3942c358854e0c3d97c857b4 172.18.0.5:7003
        slots: (0 slots) slave
        replicates 22595186092965a48ea71d3624d320ad24bf6d64
     M: a66b9b4f4fdfdb29d2397fdb618f5d426b1b4c58 172.18.0.3:7001
        slots:[5461-10922] (5462 slots) master
        1 additional replica(s)
     S: d923750c72bfc7dbfddc8ec9f0bef19bc5dcd4d7 172.18.0.7:7005
        slots: (0 slots) slave
        replicates a66b9b4f4fdfdb29d2397fdb618f5d426b1b4c58
     M: 22595186092965a48ea71d3624d320ad24bf6d64 172.18.0.4:7002
        slots:[10923-16383] (5461 slots) master
        1 additional replica(s)
     S: fb6135920cba4525163b050b73407ac8440ca667 172.18.0.6:7004
        slots: (0 slots) slave
        replicates 5005a384c9db8c3213afa630328e5331036d581b
     [OK] All nodes agree about slots configuration.
     >>> Check for open slots...
     >>> Check slots coverage...
     [OK] All 16384 slots covered.
     ```

2. redis实例运行在不同的宿主机上

   这里我将3个master实例运行在一台机(10.82.12.95)上，3个slave实例运行在另一台机器(10.82.12.98)上

   - 在两台机器上分别创建文件夹

     ```bash
     # 创建文件夹
     for port in `seq 7000 7002`; do
         mkdir -p ./$port/ && mkdir -p ./$port/data \
         && PORT=$port envsubst < ./redis-cluster.tmpl > ./$port/redis.conf;
     done
     ```

   - 在两台机器上分别运行docker redis 实例，注意这里就没有使用docker network了，直接使用的是宿主机的host，原因是要在不同机器的docker容器中通信是很麻烦的

     ```bash
     # 运行docker redis 实例
     for port in `seq 7000 7002`; do
         docker run -d \
         -v $PWD/$port/redis.conf:/data/redis.conf \
         -v $PWD/$port/data:/data/redis \
         --restart always --name redis-$port --net host \
         redis redis-server /data/redis.conf;
     done
     ```

   - 在任意一台机器上创建docker-cluster

     ```shell
     # 创建docker-cluster
     docker run -it --rm redis redis-cli --cluster create 10.82.12.95:7000 10.82.12.95:7001 10.82.12.95:7002 10.82.12.98:7000 10.82.12.98:7001 10.82.12.98:7002 --cluster-replicas 1 
     ```

     需要在接下来的console中输入“yes”，即可完成docker-cluster的搭建

**测试**

执行命令：docker exec -it redis-master-7000 redis-cli -p 7000 ，就进入了redi-cli界面，可以进行任何骚操作。

#### java连接redis cluster

```java
public static void main(String[] args)
{
    Set<HostAndPort> nodes = new HashSet<>();
    nodes.add(new HostAndPort("192.168.0.107", 6380));
    nodes.add(new HostAndPort("192.168.0.107", 6381));
    nodes.add(new HostAndPort("192.168.0.107", 6382));
    nodes.add(new HostAndPort("192.168.0.107", 6383));
    nodes.add(new HostAndPort("192.168.0.107", 6384));
    nodes.add(new HostAndPort("192.168.0.107", 6385));

    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
    jedisPoolConfig.setMaxTotal(10); //最大连接数
    jedisPoolConfig.setMaxIdle(1);//最大空闲数
    jedisPoolConfig.setMaxWaitMillis(10*1000);//最大等待时间，超过该时间还未获取到连接，会抛出异常

    JedisCluster cluster = new JedisCluster(nodes, jedisPoolConfig);

    cluster.set("name", "l4");
    System.out.println(cluster.get("name"));
}

```

#### Redis常见数据丢失情况分析及解决

情况分析

1. 异步复制导致的数据丢失

   ![lQCkPP.png](https://s2.ax1x.com/2019/12/30/lQCkPP.png)

   因为master->slave的数据同步是异步的，所以可能存在部分数据还没有同步到slave，master就宕机了，此时这部分数据就丢失了。

2. 脑裂导致的数据丢失

   ![lQCYMF.png](https://s2.ax1x.com/2019/12/30/lQCYMF.png)

   当master所在的机器突然脱离的正常的网络，与其他slave、sentinel失去了连接，但是master还在运行着。此时sentinel就会认为master宕机了，会开始选举把slave提升为新的master，这个时候集群中就会出现两个master，也就是所谓的脑裂。

   此时虽然产生了新的master节点，但是客户端可能还没来得及切换到新的master，会继续向旧的master写入数据。

   当网络恢复正常时，旧的master会变成新的master的从节点，自己的数据会清空，重新从新的master上复制数据。

**解决方案**

Redis提供了这两个配置用来降低数据丢失的可能性



```swift
min-slaves-to-write 1 
min-slaves-max-lag 10
```

上面两行配置的意思是，要求至少有1个slave，数据复制和同步的延迟不能超过10秒，如果不符合这个条件，那么master将不会接收任何请求

1. 减少异步复制的数据丢失

   有了min-slaves-max-lag这个配置，就可以确保，一旦slave复制数据和ack延时太长，就认为master宕机后损失的数据太多了，那么就拒绝写请求，这样可以把master宕机时由于部分数据未同步到slave导致的数据丢失降低到可控范围内。

2. 减少脑裂的数据丢失

   如果一个master出现了脑裂，跟其他slave丢了连接，那么上面两个配置可以确保，如果不能继续给指定数量的slave发送数据，而且slave超过10秒没有给自己ack消息，那么就直接拒绝客户端的写请求

   这样脑裂后的旧master就不会接受client的新数据，也就避免了数据丢失。

   Redis并不能保证数据的强一致性，看官方文档的说明

   ![lQCcse.png](https://s2.ax1x.com/2019/12/30/lQCcse.png)

参考文档：

[http://www.redis.cn/topics/cluster-tutorial.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.redis.cn%2Ftopics%2Fcluster-tutorial.html)

### 参考

1. [redis的集群方案对比（Codis对比Twemproxy）](https://www.jianshu.com/p/83ac498e1b2c)
2. [docker redis 集群（cluster）搭建](https://my.oschina.net/dslcode/blog/1936656)