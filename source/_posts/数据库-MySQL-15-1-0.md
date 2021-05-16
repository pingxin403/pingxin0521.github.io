---
title: MySQL数据库 分库分表 shared-jdbc（一）
date: 2019-10-16 13:18:59
tags:
 - MySQL
 - 数据库
 - Java
categories:
 - 数据库
 - MySQL
---

在日常的工作中，关系型数据库本身比较容易成为系统的瓶颈点，虽然读写分离能分散数据库的读写压力，但并没有分散存储压力，当数据量达到千万甚至上亿时，单台数据库服务器的存储能力会成为系统的瓶颈，主要体现在以下几个方面：

- 数据量太大，读写的性能会下降，即使有索引，索引也会变得很大，性能同样会降下。
- 数据库文件会得很大，数据库备份和恢复需要耗时很长。
- 数据库文件越大，极端情况下丢失数据的风险越高。

因此，当流量越来越大时，且单机容量达到上限时，此时需要考虑对其进行切分，切分的目的就在于减少单机数据库的负担，将由多台数据库服务器一起来分担，缩短查询时间。

[shared-jdbc](https://shardingsphere.apache.org/)

<!--more-->

#### 分库分表是什么

小明是一家初创电商平台的开发人员,他负责卖家模块的功能开发,其中涉及了店铺、商品的相关业务,设计如下数据库:

![1sAOqU.png](https://s2.ax1x.com/2020/02/05/1sAOqU.png)

通过以下SQL能够获取到商品相关的店铺信息、地理区域信息:

```sql
SELECT p.*,r.[地理区域名称],s.[店铺名称],s.[信誉]
FROM [商品信息] p
LEFT JOIN [地理区域] r ON p.[产地] = r.[地理区域编码]
LEFT JOIN [店铺信息] s ON p.id = s.[所属店铺]
WHERE p.id = ?
```

形成类似以下列表展示:

![1sEpGR.png](https://s2.ax1x.com/2020/02/05/1sEpGR.png)

随着公司业务快速发展,数据库中的数据量猛增,访问性能也变慢了,优化迫在眉睫。分析一下问题出现在哪儿呢? 关系型数据库本身比较容易成为系统瓶颈,单机存储容量、连接数、处理能力都有限。当单表的数据量达到1000W或100G以后,由于查询维度较多,即使添加从库、优化索引,做很多操作时性能仍下降严重。

- 方案1:
  通过提升服务器硬件能力来提高数据处理能力,比如增加存储容量 、CPU等,这种方案成本很高,并且如果瓶颈在MySQL本身那么提高硬件也是有很的。
- 方案2:
  把数据分散在不同的数据库中,使得单一数据库的数据量变小来缓解单一数据库的性能问题,从而达到提升数据库性能的目的,如下图:将电商数据库拆分为若干独立的数据库,并且对于大表也拆分为若干小表,通过这种数据库拆分的方法来解决数据库的性能问题。

![1sEVde.png](https://s2.ax1x.com/2020/02/05/1sEVde.png)

分库分表就是为了解决由于数据量过大而导致数据库性能降低的问题,将原来独立的数据库拆分成若干数据库组成,将数据大表拆分成若干数据表组成,使得单一数据库、单一数据表的数据量变小,从而达到提升数据库性能的目的。

#### 分库分表的方式

分库分表包括分库和分表两个部分,在生产中通常包括:垂直分库、水平分库、垂直分表、水平分表四种方式。

1. 垂直分表

   下边通过一个商品查询的案例讲解垂直分表:

   通常在商品列表中是不显示商品详情信息的,如下图:

   ![1sEpGR.png](https://s2.ax1x.com/2020/02/05/1sEpGR.png)

   用户在浏览商品列表时,只有对某商品感兴趣时才会查看该商品的详细描述。因此,商品信息中商品描述字段访问频次较低,且该字段存储占用空间较大,访问单个数据IO时间较长;商品信息中商品名称、商品图片、商品价格等其他字段数据访问频次较高。

   由于这两种数据的特性不一样,因此他考虑将商品信息表拆分如下:

   将访问频次低的商品描述信息单独存放在一张表中,访问频次较高的商品基本信息单独放在一张表中。

   ![1sE1L8.png](https://s2.ax1x.com/2020/02/05/1sE1L8.png)

   商品列表可采用以下 sql:

   ```
   SELECT p.*,r.[地理区域名称],s.[店铺名称],s.[信誉]
   FROM [商品信息] p
   LEFT JOIN [地理区域] r ON p.[产地] = r.[地理区域编码]
   LEFT JOIN [店铺信息] s ON p.id = s.[所属店铺]
   WHERE...ORDER BY...LIMIT...
   ```

   需要获取商品描述时,再通过以下sql获取:

   ```
   SELECT *
   FROM [商品描述]
   WHERE [商品ID] = ?
   ```

   小明进行的这一步优化,就叫垂直分表。

   垂直分表定义:将一个表按照字段分成多表,每个表存储其中一部分字段。

   它带来的提升是:

   - 为了避免IO争抢并减少锁表的几率,查看详情的用户与商品信息浏览互不影响
   - 充分发挥热门数据的操作效率,商品信息的操作的高效率不会被商品描述的低效率所拖累。

   一般来说,某业务实体中的各个数据项的访问频次是不一样的,部分数据项可能是占用存储空间比较大的BLOB或是TEXT。例如上例中的商品描述。所以,当表数据量很大时,可以将表按字段切开,将热门字段、冷门字段分开放置在不同库中,这些库可以放在不同的存储设备上,避免IO争抢。垂直切分带来的性能提升主要集中在热门数据的操作效率上,而且磁盘争用情况减少。

   通常我们按以下原则进行垂直拆分:

   - 把不常用的字段单独放在一张表;
   - 把text,blob等大字段拆分出来放在附表中;
   - 经常组合查询的列放在一张表中;

2. 垂直分库

   通过垂直分表性能得到了一定程度的提升,但是还没有达到要求,并且磁盘空间也快不够了,因为数据还是始终限制在一台服务器,库内垂直分表只解决了单一表数据量过大的问题,但没有将表分布到不同的服务器上,因此每个表还是竞争同一个物理机的CPU、内存、网络IO、磁盘。

   经过思考,他把原有的SELLER_DB(卖家库),分为了PRODUCT_DB(商品库)和STORE_DB(店铺库),并把这两个库分散到不同服务器,如下图:

   ![1sVQh9.png](https://s2.ax1x.com/2020/02/05/1sVQh9.png)

   由于商品信息与商品描述业务耦合度较高,因此一起被存放在PRODUCT_DB(商品库);而店铺信息相对独立,因此单独被存放在STORE_DB(店铺库)。

   小明进行的这一步优化,就叫**垂直分库**。

   **垂直分库**是指按照业务将表进行分类,分布到不同的数据库上面,每个库可以放在不同的服务器上,它的核心理念是**专库专用**。

   它带来的提升是:

   - 解决业务层面的耦合,业务清晰
   - 能对不同业务的数据进行分级管理、维护、监控、扩展等
   - 高并发场景下,垂直分库一定程度的提升 IO、数据库连接数、降低单机硬件资源的瓶颈

   垂直分库通过将表按业务分类,然后分布在不同数据库,并且可以将这些数据库部署在不同服务器上,从而达到多个服务器共同分摊压力的效果,但是依然没有解决单表数据量过大的问题。

3. 水平分库

   经过垂直分库后,数据库性能问题得到一定程度的解决,但是随着业务量的增长,PRODUCT_DB(商品库)单库存储数据已经超出预估。粗略估计,目前有8w店铺,每个店铺平均150个不同规格的商品,再算上增长,那商品数量得往1500w+上预估,并且PRODUCT_DB(商品库)属于访问非常频繁的资源,单台服务器已经无法支撑。此时该如何优化?

   再次分库?但是从业务角度分析,目前情况已经无法再次垂直分库。

   尝试水平分库,将店铺ID为单数的和店铺ID为双数的商品信息分别放在两个库中。

   ![1sVB9A.png](https://s2.ax1x.com/2020/02/05/1sVB9A.png)

   也就是说,要操作某条数据,先分析这条数据所属的店铺ID。如果店铺ID为双数,将此操作映射至RRODUCT_DB1(商品库1);如果店铺ID为单数,将操作映射至RRODUCT_DB2(商品库2)。此操作要访问数据库名称的表达式为`RRODUCT_DB[店铺ID%2 + 1]` 。

   小明进行的这一步优化,就叫水平分库。

   水平分库是把同一个表的数据按一定规则拆到不同的数据库中,每个库可以放在不同的服务器上。它带来的提升是:

   - 解决了单库大数据,高并发的性能瓶颈。
   - 提高了系统的稳定性及可用性。

   当一个应用难以再细粒度的垂直切分,或切分后数据量行数巨大,存在单库读写、存储性能瓶颈,这时候就需要进行水平分库了,经过水平切分的优化,往往能解决单库存储量及性能瓶颈。但由于同一个表被分配在不同的数据库,需要额外进行数据操作的路由工作,因此大大提升了系统复杂度。

4. 水平分表

   按照水平分库的思路对他把PRODUCT_DB_X(商品库)内的表也可以进行水平拆分,其目的也是为解决单表数据量大的问题,如下图:

   ![1sBPp9.png](https://s2.ax1x.com/2020/02/05/1sBPp9.png)

   与水平分库的思路类似,不过这次操作的目标是表,商品信息及商品描述被分成了两套表。如果商品ID为双数,将此操作映射至商品信息1表;如果商品ID为单数,将操作映射至商品信息2表。此操作要访问表名称的表达式为`商品信息[商品ID%2 + 1]` 。

   小明进行的这一步优化,就叫水平分表。

   水平分表是在同一个数据库内,把同一个表的数据按一定规则拆到多个表中。

   它带来的提升是:

   - 优化单一表数据量过大而产生的性能问题
   - 避免 IO争抢并减少锁表的几率

   库内的水平分表,解决了单一表数据量过大的问题,分出来的小表中只包含一部分数据,从而使得单个表的数据量变小,提高检索性能。
   
5. 数据库分区

   数据库分区：将大表进行分区，不同分区可以放置在不同存储设备上，这些分区在逻辑上组成一个大表，对客户端透明

   1. 分区方式和水平切片是类似的，分区方式也和水平切片方式类似，如范围切片，取模切片等
   2. 数据库分区是数据库自身的特性，切片则是外部强制手段控制完成的
   3. 数据库分区无法将分区跨库，更不能跨数据库服务器，但能保存在不同数据文件从而放置在不同存储设备上
   4. 数据库分区是数据库的特性，数据完整性、一致性等实现起来很方便，这一切都是数据库自身保证的

   在数据库切片流行之前，对大表的处理方式就是划分分区表。数据库分区相比于切片，最大的缺点在于无法跨库、跨服务器，所以在某些方面的压力得到不缓解。

**小结**

- 垂直分表:可以把一个宽表的字段按访问频次、是否是大字段的原则拆分为多个表,这样既能使业务清晰,还能提升部分性能。拆分后,尽量从业务角度避免联查,否则性能方面将得不偿失。
- 垂直分库:可以把多个表按业务耦合松紧归类,分别存放在不同的库,这些库可以分布在不同服务器,从而使访问
  压力被多服务器负载,大大提升性能,同时能提高整体架构的业务清晰度,不同的业务库可根据自身情况定制优化方案。但是它需要解决跨库带来的所有复杂问题。
- 水平分库:可以把一个表的数据(按数据行)分到多个不同的库,每个库只有这个表的部分数据,这些库可以分布在不同服务器,从而使访问压力被多服务器负载,大大提升性能。它不仅需要解决跨库带来的所有复杂问题,还要解决数据路由的问题(数据路由问题后边介绍)。
- 水平分表:可以把一个表的数据(按数据行)分到多个同一个数据库的多张表中,每个表只有这个表的部分数据,这样做能小幅提升性能,它仅仅作为水平分库的一个补充优化。

一般来说,在系统设计阶段就应该根据业务耦合松紧来确定垂直分库,垂直分表方案,在数据量及访问压力不是特别大的情况,首先考虑缓存、读写分离、索引技术等方案。若数据量极大,且持续增长,再考虑水平分库水平分表方案。

#### 分库分表带来的问题

分库分表能有效的缓解了单机和单库带来的性能瓶颈和压力,突破网络 IO、硬件资源、连接数的瓶颈,同时也带来了一些问题。

1. 事务一致性问题
   由于分库分表把数据分布在不同库甚至不同服务器,不可避免会带来分布式事务问题。

2. 跨节点关联查询
   在没有分库前,我们检索商品时可以通过以下SQL对店铺信息进行关联查询:

   ```sql
   SELECT p.*,r.[地理区域名称],s.[店铺名称],s.[信誉]
   FROM [商品信息] p
   LEFT JOIN [地理区域] r ON p.[产地] = r.[地理区域编码]
   LEFT JOIN [店铺信息] s ON p.id = s.[所属店铺]
   WHERE...ORDER BY...LIMIT...
   ```

   但垂直分库后**[商品信息]**和**[店铺信息]**不在一个数据库,甚至不在一台服务器,无法进行关联查询。

   可将原关联查询分为两次查询,第一次查询的结果集中找出关联数据id,然后根据id发起第二次请求得到关联数据,最后将获得到的数据进行拼装。

3. 跨节点分页、排序函数

   跨节点多库进行查询时,limit分页、order by排序等问题,就变得比较复杂了。需要先在不同的分片节点中将数据进行排序并返回,然后将不同分片返回的结果集进行汇总和再次排序。

   如,进行水平分库后的商品库,按ID倒序排序分页,取第一页:

   ![1sxl2F.png](https://s2.ax1x.com/2020/02/05/1sxl2F.png)

   以上流程是取第一页的数据,性能影响不大,但由于商品信息的分布在各数据库的数据可能是随机的,如果是取第N页,需要将所有节点前N页数据都取出来合并,再进行整体的排序,操作效率可想而知。所以请求页数越大,系统的性能也会越差。

   在使用Max、Min、Sum、Count之类的函数进行计算的时候,与排序分页同理,也需要先在每个分片上执行相应的函数,然后将各个分片的结果集进行汇总和再次计算,最终将结果返回。

4. 主键避重

   在分库分表环境中,由于表中数据同时存在不同数据库中,主键值平时使用的自增长将无用武之地,某个分区数据库生成的ID无法保证全局唯一。因此需要单独设计全局主键,以避免跨库主键重复问题。

   ![1sxdPK.png](https://s2.ax1x.com/2020/02/05/1sxdPK.png)

5. 公共表

   实际的应用场景中,参数表、数据字典表等都是数据量较小,变动少,而且属于高频联合查询的依赖表。例子中
   理区域表也属于此类型。

   可以将这类表在每个数据库都保存一份,所有对公共表的更新操作都同时发送到所有分库执行。

由于分库分表之后,数据被分散在不同的数据库、服务器。因此,对数据的操作也就无法通过常规方式完成,并且它还带来了一系列的问题。好在,这些问题不是所有都需要我们在应用层面上解决,市面上有很多中间件可供我们选择,其中Sharding-JDBC使用流行度较高,我们来了解一下它。

#### Sharding-JDBC介绍

[Apache ShardingSphere](https://shardingsphere.apache.org/document/current/cn/quick-start/)(Incubator) 是一套开源的分布式数据库中间件解决方案组成的生态圈，它由Sharding-JDBC、Sharding-Proxy和Sharding-Sidecar（规划中）这3款相互独立，却又能够混合部署配合使用的产品组成。它们均提供标准化的数据分片、分布式事务和数据库治理功能，可适用于如Java同构、异构语言、云原生等各种多样化的应用场景。

ShardingSphere定位为关系型数据库中间件，旨在充分合理地在分布式的场景下利用关系型数据库的计算和存储能力，而并非实现一个全新的关系型数据库。它通过关注不变，进而抓住事物本质。关系型数据库当今依然占有巨大市场，是各个公司核心业务的基石，未来也难于撼动，我们目前阶段更加关注在原有基础上的增量，而非颠覆。

Apache官方发布从4.0.0版本开始。

![1ySq3T.png](https://s2.ax1x.com/2020/02/05/1ySq3T.png)

1. Sharding-JDBC

   定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。

   - 适用于任何基于JDBC的ORM框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
   - 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP等。
   - 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer，PostgreSQL以及任何遵循SQL92标准的数据库。

   ![1ySYh6.png](https://s2.ax1x.com/2020/02/05/1ySYh6.png)

2. Sharding-Proxy

   定位为透明化的数据库代理端，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。 目前先提供MySQL/PostgreSQL版本，它可以使用任何兼容MySQL/PostgreSQL协议的访问客户端(如：MySQL Command Client, MySQL Workbench, Navicat等)操作数据，对DBA更加友好。

   - 向应用程序完全透明，可直接当做MySQL/PostgreSQL使用。
   - 适用于任何兼容MySQL/PostgreSQL协议的的客户端。

   ![1ySdje.png](https://s2.ax1x.com/2020/02/05/1ySdje.png)

3. Sharding-Sidecar（TODO）

   定位为Kubernetes的云原生数据库代理，以Sidecar的形式代理所有对数据库的访问。 通过无中心、零侵入的方案提供与数据库交互的的啮合层，即Database Mesh，又可称数据网格。

   Database Mesh的关注重点在于如何将分布式的数据访问应用与数据库有机串联起来，它更加关注的是交互，是将杂乱无章的应用与数据库之间的交互有效的梳理。使用Database Mesh，访问数据库的应用和数据库终将形成一个巨大的网格体系，应用和数据库只需在网格体系中对号入座即可，它们都是被啮合层所治理的对象。

   ![1ySDHA.png](https://s2.ax1x.com/2020/02/05/1ySDHA.png)

|            | *Sharding-JDBC* | *Sharding-Proxy* | *Sharding-Sidecar* |
| :--------- | :-------------- | :--------------- | :----------------- |
| 数据库     | 任意            | MySQL            | MySQL              |
| 连接消耗数 | 高              | 低               | 高                 |
| 异构语言   | 仅Java          | 任意             | 任意               |
| 性能       | 损耗低          | 损耗略高         | 损耗低             |
| 无中心化   | 是              | 否               | 是                 |
| 静态入口   | 无              | 有               | 无                 |

**混合架构**

Sharding-JDBC采用无中心化架构，适用于Java开发的高性能的轻量级OLTP应用；Sharding-Proxy提供静态入口以及异构语言的支持，适用于OLAP应用以及对分片数据库进行管理和运维的场景。

ShardingSphere是多接入端共同组成的生态圈。 通过混合使用Sharding-JDBC和Sharding-Proxy，并采用同一注册中心统一配置分片策略，能够灵活的搭建适用于各种场景的应用系统，架构师可以更加自由的调整适合于当前业务的最佳系统架构。

![1yS64P.png](https://s2.ax1x.com/2020/02/05/1yS64P.png)

**功能列表**

1. 数据分片

   - 分库 & 分表

   - 读写分离

   - 分片策略定制化

   - 无中心化分布式主键

2. 分布式事务

   - 标准化事务接口

   - XA强一致事务

   - 柔性事务

3. 数据库治理

   - 配置动态化

   - 编排 & 治理

   - 数据脱敏

   - 可视化链路追踪

   - 弹性伸缩(规划中)

**项目状态**

![1yShuQ.png](https://s2.ax1x.com/2020/02/05/1yShuQ.png)

#### 快速入门

上述示例代码位置：<https://github.com/hanyunpeng0521/spring-boot-learn/tree/master/spring-boot-20-sharing-jdbc>

操作顺序：先看博文，熟悉步骤--》创建主从--》在主服务器初始化库表--根据步骤修改配置文件。

人工创建两张表,t_order_1和t_order_2,这两张表是订单表拆分后的表,通过Sharding-Jdbc向订单表插入数据,按照一定的分片规则,主键为偶数的进入t_order_1,另一部分数据进入t_order_2,通过Sharding-Jdbc 查询数据,根据 SQL语句的内容从t_order_1或t_order_2查询数据。

1. 创建数据库

   ```sql
   -- 创建订单库 order_db
   CREATE DATABASE `order_db` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
   
   -- 在order_db中创建t_order_1、t_order_2表
   
   DROP TABLE IF EXISTS `t_order_1`;
   CREATE TABLE `t_order_1`
   (
   `order_id` bigint(20) NOT NULL COMMENT '订单id',
   `price` decimal(10, 2) NOT NULL COMMENT '订单价格',
   `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
   `status` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态',
   PRIMARY KEY (`order_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `t_order_2`;
   CREATE TABLE `t_order_2`
   (
   `order_id` bigint(20) NOT NULL COMMENT '订单id',
   `price` decimal(10, 2) NOT NULL COMMENT '订单价格',
   `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
   `status` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态',
   PRIMARY KEY (`order_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   ```

2. 引入maven依赖

   ```xml
   <dependencies>
   
            <dependency>
                   <groupId>io.springfox</groupId>
                   <artifactId>springfox-swagger2</artifactId>
                   <version>2.9.2</version>
               </dependency>
   
               <dependency>
                   <groupId>io.springfox</groupId>
                   <artifactId>springfox-swagger-ui</artifactId>
                   <version>2.9.2</version>
               </dependency>
   
               <dependency>
                   <groupId>org.projectlombok</groupId>
                   <artifactId>lombok</artifactId>
                   <version>1.18.0</version>
               </dependency>
   
   
               <dependency>
                   <groupId>javax.interceptor</groupId>
                   <artifactId>javax.interceptor-api</artifactId>
                   <version>1.2</version>
               </dependency>
   
   
               <dependency>
                   <groupId>mysql</groupId>
                   <artifactId>mysql-connector-java</artifactId>
               </dependency>
   
               <dependency>
                   <groupId>org.mybatis.spring.boot</groupId>
                   <artifactId>mybatis-spring-boot-starter</artifactId>
                   <version>2.0.0</version>
               </dependency>
   
               <dependency>
                   <groupId>com.alibaba</groupId>
                   <artifactId>druid-spring-boot-starter</artifactId>
                   <version>1.1.16</version>
               </dependency>
   
   
               <dependency>
                   <groupId>org.apache.shardingsphere</groupId>
                   <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
                   <version>4.0.0-RC1</version>
               </dependency>
   
   
               <dependency>
                   <groupId>com.baomidou</groupId>
                   <artifactId>mybatis-plus-boot-starter</artifactId>
                   <version>3.1.0</version>
               </dependency>
   
               <dependency>
                   <groupId>com.baomidou</groupId>
                   <artifactId>mybatis-plus-generator</artifactId>
                   <version>3.1.0</version>
               </dependency>
   
               <dependency>
                   <groupId>org.mybatis</groupId>
                   <artifactId>mybatis-typehandlers-jsr310</artifactId>
                   <version>1.0.2</version>
               </dependency>
       
              <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
   
   ```
   
3. 配置：分片规则配置是sharding-jdbc进行对分库分表操作的重要依据,配置内容包括:数据源、主键生成策略、分片策略等。

   ```properties
   # 项目contextPath，一般在正式发布版本中，我们不配置
   server.servlet.context-path=/sharing
   # 错误页，指定发生错误时，跳转的URL。请查看BasicErrorController源码便知
   server.error.path=/error
   # 服务端口
   server.port=8080
   # session最大超时时间(分钟)，默认为30
   server.session-timeout=60
   # 该服务绑定IP地址，启动服务器时如本机不是该IP地址则抛出异常启动失败，只有特殊需求的情况下才配置
   # server.address=192.168.16.11
   #如果目录src/main/resources下同时存在banner.txt和banner.gif，项目会先将banner.gif每一个画面打印完毕之后，再打印banner.txt中的内容。
   spring.application.name=sharding-jdbc-simple-demo
   spring.http.encoding.enabled=true
   spring.http.encoding.charset=UTF-8
   spring.http.encoding.force=true
   spring.main.allow-bean-definition-overriding=true
   mybatis.configuration.map-underscore-to-camel-case=true
   #sharding-jdbc分片规则配置
   #数据源
   spring.shardingsphere.datasource.names=m1
   spring.shardingsphere.datasource.m1.type=com.alibaba.druid.pool.DruidDataSource
   spring.shardingsphere.datasource.m1.driver-class-name=com.mysql.cj.jdbc.Driver
   spring.shardingsphere.datasource.m1.url=jdbc:mysql://localhost:3306/order_db?useUnicode=true
   spring.shardingsphere.datasource.m1.username=root
   spring.shardingsphere.datasource.m1.password=mysql
   # 指定t_order表的数据分布情况，配置数据节点 m1.t_order_1,m1.t_order_2
   spring.shardingsphere.sharding.tables.t_order.actual-data-nodes=m1.t_order_$->{1..2}
   # 指定t_order表的主键生成策略为SNOWFLAKE
   spring.shardingsphere.sharding.tables.t_order.key-generator.column=order_id
   spring.shardingsphere.sharding.tables.t_order.key-generator.type=SNOWFLAKE
   # 指定t_order表的分片策略，分片策略包括分片键和分片算法
   spring.shardingsphere.sharding.tables.t_order.table-strategy.inline.sharding-column=order_id
   spring.shardingsphere.sharding.tables.t_order.table-strategy.inline.algorithm-expression=t_order_$->{order_id % 2 + 1}
   # 打开sql输出日志
   spring.shardingsphere.props.sql.show=true
   swagger.enable=true
   logging.level.root=info
   logging.level.org.springframework.web=info
   logging.level.com.itheima.dbsharding=debug
   logging.level.druid.sql=debug
   ```

   1. 首先定义数据源m1,并对m1进行实际的参数配置。
   2. 指定t_order表的数据分布情况,他分布在m1.t_order_1,m1.t_order_2
   3. 指定t_order表的主键生成策略为SNOWFLAKE,SNOWFLAKE是一种分布式自增算法,保证id全局唯一
   4. 定义t_order分片策略,order_id为偶数的数据落在t_order_1,为奇数的落在t_order_2,分表策略的表达式为`t_order_$->{order_id % 2 + 1}`

4. 接口类

   ```java
   @Mapper
   @Component
   public interface OrderDao {
   
       /**
        * 插入订单
        *
        * @param price
        * @param userId
        * @param status
        * @return
        */
       @Insert("insert into t_order(price,user_id,status)values(#{price},#{userId},#{status})")
       int insertOrder(@Param("price") BigDecimal price, @Param("userId") Long userId, @Param("status") String status);
   
       /**
        * 根据id列表查询订单
        *
        * @param orderIds
        * @return
        */
       @Select("<script>" +
               "select" +
               " * " +
               " from t_order t " +
               " where t.order_id in " +
               " <foreach collection='orderIds' open='(' separator=',' close=')' item='id'>" +
               " #{id} " +
               " </foreach>" +
               "</script>")
       List<Map> selectOrderbyIds(@Param("orderIds") List<Long> orderIds);
   }
   ```

5. 测试

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest(classes = {SharingApplication.class})
   public class OrderDaoTest {
   
   
       OrderDao orderDao;
   
       @Test
       public void testInsertOrder(){
           for(int i=1;i<20;i++){
               orderDao.insertOrder(new BigDecimal(i),1L,"SUCCESS");
           }
       }
   
       @Test
       public void testSelectOrderbyIds(){
           List<Long> ids = new ArrayList<>();
           ids.add(373897739357913088L);
           ids.add(373897037306920961L);
   
           List<Map> maps = orderDao.selectOrderbyIds(ids);
           System.out.println(maps);
       }
   }
   
   ```

   执行 testInsertOrder，通过日志可以发现order_id为奇数的被插入到t_order_2表,为偶数的被插入到t_order_1表,达到预期目标。

   执行testSelectOrderbyIds，通过日志可以发现,根据传入order_id的奇偶不同,sharding-jdbc分别去不同的表检索数据,达到预期目标。

**流程分析**

通过日志分析,Sharding-JDBC在拿到用户要执行的sql之后干了哪些事儿:

1. 解析sql,获取片键值,在本例中是order_id
2. Sharding-JDBC通过规则配置 t_order_$->{order_id % 2 + 1},知道了当order_id为偶数时,应该往t_order_1表插数据,为奇数时,往t_order_2插数据。
3. 于是Sharding-JDBC根据order_id的值改写sql语句,改写后的SQL语句是真实所要执行的SQL语句。
4. 执行改写后的真实sql语句
5. 将所有真正执行sql的结果进行汇总合并,返回。

#### 水平分库

前面已经介绍过,水平分库是把同一个表的数据按一定规则拆到不同的数据库中,每个库可以放在不同的服务器上。接下来看一下如何使用Sharding-JDBC实现水平分库,咱们继续对示例中的例子进行完善。

1. 将原有order_db库拆分为order_db_1、order_db_2

2. 分片规则修改
   由于数据库拆分了两个,这里需要配置两个数据源。

   分库需要配置分库的策略,和分表策略的意义类似,通过分库策略实现数据操作针对分库的数据库进行操作。

   ```properties
   # 定义多个数据源
   spring.shardingsphere.datasource.names = m1,m2
   spring.shardingsphere.datasource.m1.type = com.alibaba.druid.pool.DruidDataSource
   spring.shardingsphere.datasource.m1.driver‐class‐name = com.mysql.jdbc.Driver
   spring.shardingsphere.datasource.m1.url = jdbc:mysql://localhost:3306/order_db_1?useUnicode=true
   spring.shardingsphere.datasource.m1.username = root
   spring.shardingsphere.datasource.m1.password = root
   spring.shardingsphere.datasource.m2.type = com.alibaba.druid.pool.DruidDataSource
   spring.shardingsphere.datasource.m2.driver‐class‐name = com.mysql.jdbc.Driver
   spring.shardingsphere.datasource.m2.url = jdbc:mysql://localhost:3306/order_db_2?useUnicode=true
   spring.shardingsphere.datasource.m2.username = root
   spring.shardingsphere.datasource.m2.password = root
   ...
   # 分库策略,以user_id为分片键,分片策略为user_id % 2 + 1,user_id为偶数操作m1数据源,否则操作m2。
   spring.shardingsphere.sharding.tables.t_order.database‐strategy.inline.sharding‐column = user_id
   spring.shardingsphere.sharding.tables.t_order.database‐strategy.inline.algorithm‐expression =m$‐>{user_id % 2 + 1}
   ```

   分库策略定义方式如下:

   ```properties
   #分库策略,如何将一个逻辑表映射到多个数据源
   spring.shardingsphere.sharding.tables.<逻辑表名称>.database‐strategy.<分片策略>.<分片策略属性名>= #分片策略属性值#分表策略,如何将一个逻辑表映射为多个实际表
   spring.shardingsphere.sharding.tables.<逻辑表名称>.table‐strategy.<分片策略>.<分片策略属性名>= #分片策略属性值
   ```

   Sharding-JDBC支持以下几种分片策略:

   不管理分库还是分表,策略基本一样。

   - standard :标准分片策略,对应StandardShardingStrategy。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。StandardShardingStrategy只支持单分片键,提供PreciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法。PreciseShardingAlgorithm是必选的,用于处理=和IN的分片。RangeShardingAlgorithm是可选的,用于处理BETWEEN AND分片,如果不配置RangeShardingAlgorithm,SQL中的BETWEEN AND将按照全库路由处理。
   - complex :符合分片策略,对应ComplexShardingStrategy。复合分片策略。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。ComplexShardingStrategy支持多分片键,由于多分片键之间的关系复杂,因此并未进行过多的封装,而是直接将分片键值组合以及分片操作符透传至分片算法,完全由应用开发者实现,提供最大的灵活度。
   - inline :行表达式分片策略,对应InlineShardingStrategy。使用Groovy的表达式,提供对SQL语句中的=和IN的分片操作支持,只支持单分片键。对于简单的分片算法,可以通过简单的配置使用,从而避免繁琐Java代码开发,如: t_user_$ ->{u_id % 8} 表示t_user表根据u_id模8,而分成8张表,表名称为 t_user_0 到t_user_7 。
   - hint :Hint分片策略,对应HintShardingStrategy。通过Hint而非SQL解析的方式分片的策略。对于分片段非SQL决定,而由其他外置条件决定的场景,可使用SQL Hint灵活的注入分片字段。例:内部系统,按照员工登录主键分库,而数据库中并无此字段。SQL Hint支持通过Java API和SQL注释(待实现)两种方式使用。
   - none :不分片策略,对应NoneShardingStrategy。不分片的策略。

3. 测试

   ```java
       @Test
       public void testInsertOrder2(){
           for (int i = 0 ; i<10; i++){
               orderDao.insertOrder(new BigDecimal((i+1)*5),1L,"WAIT_PAY");
           }
           for (int i = 0 ; i<10; i++){
               orderDao.insertOrder(new BigDecimal((i+1)*10),2L,"WAIT_PAY");
           }
       }
   ```

   观察输出发现：操作user_id为偶数使用m2，为奇数使用m1

#### 垂直分库

垂直分库是指按照业务将表进行分类,分布到不同的数据库上面,每个库可以放在不同的服务器上,它的核心理念是专库专用。接下来看一下如何使用Sharding-JDBC实现垂直分库

1. 创建数据库

   ```sql
   -- 创建数据库user_db
   CREATE DATABASE `user_db` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
   -- 在user_db中创建t_user表
   DROP TABLE IF EXISTS `t_user`;
   CREATE TABLE `t_user`
   (
   `user_id` bigint(20) NOT NULL COMMENT '用户id',
   `fullname` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户姓名',
   `user_type` char(1) DEFAULT NULL COMMENT '用户类型',
   PRIMARY KEY (`user_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   ```

2. 在Sharding-JDBC规则中修改

   ```properties
   # 新增m0数据源,对应user_db
   spring.shardingsphere.datasource.names = m0,m1,m2
   ...
   spring.shardingsphere.datasource.m0.type = com.alibaba.druid.pool.DruidDataSource
   spring.shardingsphere.datasource.m0.driver‐class‐name = com.mysql.jdbc.Driver
   spring.shardingsphere.datasource.m0.url = jdbc:mysql://localhost:3306/user_db?useUnicode=true
   spring.shardingsphere.datasource.m0.username = root
   spring.shardingsphere.datasource.m0.password = root
   ....
   # t_user分表策略,固定分配至m0的t_user真实表
   spring.shardingsphere.sharding.tables.t_user.actual‐data‐nodes = m$‐>{0}.t_user
   spring.shardingsphere.sharding.tables.t_user.table‐strategy.inline.sharding‐column = user_id
   spring.shardingsphere.sharding.tables.t_user.table‐strategy.inline.algorithm‐expression = t_user
   ```

3. 新增UserDao

   ```java
   @Mapper
   @Component
   public interface UserDao {
   
       /**
        * 新增用户
        * @param userId 用户id
        * @param fullname 用户姓名
        * @return
        */
       @Insert("insert into t_user(user_id, fullname) value(#{userId},#{fullname})")
       int insertUser(@Param("userId")Long userId, @Param("fullname")String fullname);
   
       /**
        * 根据id列表查询多个用户
        * @param userIds 用户id列表
        * @return
        */
       @Select({"<script>",
               " select",
               " * ",
               " from t_user t ",
               " where t.user_id in",
               "<foreach collection='userIds' item='id' open='(' separator=',' close=')'>",
               "#{id}",
               "</foreach>",
               "</script>"
       })
       List<Map> selectUserbyIds(@Param("userIds") List<Long> userIds);
   
   }
   
   ```

4. 测试

   ```java
       @Test
       public void testInsertUser() {
           for (int i = 0; i < 10; i++) {
               Long id = i + 1L;
               userDao.insertUser(id, "姓名" + id);
           }
       }
   
       @Test
       public void testSelectUserbyIds() {
           List<Long> userIds = new ArrayList<>();
           userIds.add(1L);
           userIds.add(2L);
           List<Map> users = userDao.selectUserbyIds(userIds);
           System.out.println(users);
       }
   ```

   观察输出：插入使用m0，读取使用m0

#### 公共表

公共表属于系统中数据量较小,变动少,而且属于高频联合查询的依赖表。参数表、数据字典表等属于此类型。可以将这类表在每个数据库都保存一份,所有更新操作都同时发送到所有分库执行。接下来看一下如何使用Sharding-JDBC实现公共表。

1. 创建数据库

   分别在user_db、order_db_1、order_db_2中创建t_dict表:

   ```sql
   CREATE TABLE `t_dict`
   (
   `dict_id` bigint(20) NOT NULL COMMENT '字典id',
   `type` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '字典类型',
   `code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '字典编码',
   `value` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '字典值',
   PRIMARY KEY (`dict_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   ```

2. 在Sharding-JDBC规则中修改

   ```properties
   # 指定t_dict为公共表
   spring.shardingsphere.sharding.broadcast‐tables=t_dict
   ```

3. 数据操作

   ```java
   //新增DictDao:
   @Mapper
   @Component
   public interface DictDao {
   
       /**
        * 新增字典
        * @param type 字典类型
        * @param code 字典编码
        * @param value 字典值
        * @return
        */
       @Insert("insert into t_dict(dict_id,type,code,value) value(#{dictId},#{type},#{code},#{value})")
       int insertDict(@Param("dictId") Long dictId, @Param("type") String type, @Param("code")String code, @Param("value")String value);
   
       /**
        * 删除字典
        * @param dictId 字典id
        * @return
        */
       @Delete("delete from t_dict where dict_id = #{dictId}")
       int deleteDict(@Param("dictId") Long dictId);
   
   }
   
   
   ```

4. 字典操作测试

   ```java
       @Test
       public void testInsertDict() {
           dictDao.insertDict(1L, "user_type", "0", "管理员");
           dictDao.insertDict(2L, "user_type", "1", "操作员");
       }
   
       @Test
       public void testDeleteDict() {
           dictDao.deleteDict(1L);
           dictDao.deleteDict(2L);
       }
   ```

   输出

   ```
   //1
   2020-02-06 14:33:55.670  INFO 6486 --- [           main] ShardingSphere-SQL                       : Logic SQL: insert into t_dict(dict_id,type,code,value) value(?,?,?,?)
   2020-02-06 14:33:55.671  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m1 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [1, user_type, 0, 管理员]
   2020-02-06 14:33:55.671  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m2 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [1, user_type, 0, 管理员]
   2020-02-06 14:33:55.672  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m0 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [1, user_type, 0, 管理员]
   2020-02-06 14:33:55.786  INFO 6486 --- [           main] ShardingSphere-SQL                       : Rule Type: sharding
   2020-02-06 14:33:55.786  INFO 6486 --- [           main] ShardingSphere-SQL                       : Logic SQL: insert into t_dict(dict_id,type,code,value) value(?,?,?,?)
   2020-02-06 14:33:55.786  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m1 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [2, user_type, 1, 操作员]
   2020-02-06 14:33:55.787  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m2 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [2, user_type, 1, 操作员]
   2020-02-06 14:33:55.787  INFO 6486 --- [           main] ShardingSphere-SQL                       : Actual SQL: m0 ::: insert into t_dict (dict_id, type, code, value) VALUES (?, ?, ?, ?) ::: [2, user_type, 1, 操作员]
   
   //2
   
   2020-02-06 14:36:02.858  INFO 6684 --- [           main] ShardingSphere-SQL                       : Logic SQL: delete from t_dict where dict_id = ?
   2020-02-06 14:36:02.859  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m1 ::: delete from t_dict where dict_id = ? ::: [1]
   2020-02-06 14:36:02.859  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m2 ::: delete from t_dict where dict_id = ? ::: [1]
   2020-02-06 14:36:02.860  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m0 ::: delete from t_dict where dict_id = ? ::: [1]
   2020-02-06 14:36:02.970  INFO 6684 --- [           main] ShardingSphere-SQL                       : Rule Type: sharding
   2020-02-06 14:36:02.971  INFO 6684 --- [           main] ShardingSphere-SQL                       : Logic SQL: delete from t_dict where dict_id = ?
   2020-02-06 14:36:02.971  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m1 ::: delete from t_dict where dict_id = ? ::: [2]
   2020-02-06 14:36:02.971  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m2 ::: delete from t_dict where dict_id = ? ::: [2]
   2020-02-06 14:36:02.971  INFO 6684 --- [           main] ShardingSphere-SQL                       : Actual SQL: m0 ::: delete from t_dict where dict_id = ? ::: [2]
   ```

5. 字典关联查询测试:字典表已在各各分库存在,各业务表即可和字典表关联查询。
   定义用户关联查询dao:

   ```java
   //在UserDao中定义:
      /**
        * 根据id列表查询多个用户
        * @param userIds 用户id列表
        * @return
        */
       @Select({"<script>",
               " select",
               " * ",
               " from t_user t ,t_dict b",
               " where t.user_type = b.code and t.user_id in",
               "<foreach collection='userIds' item='id' open='(' separator=',' close=')'>",
               "#{id}",
               "</foreach>",
               "</script>"
       })
       List<Map> selectUserInfobyIds(@Param("userIds") List<Long> userIds);
   
   //定义测试方法:
   
       @Test
       public void testSelectUserInfobyIds() {
           List<Long> userIds = new ArrayList<>();
           userIds.add(1L);
           userIds.add(2L);
           List<Map> users = userDao.selectUserInfobyIds(userIds);
           JSONArray jsonUsers = new JSONArray(users);
           System.out.println(jsonUsers);
       }
   ```

   输出

   ```
   2020-02-06 14:37:16.635  INFO 6768 --- [           main] ShardingSphere-SQL                       : Logic SQL: select  *   from t_user t ,t_dict b  where t.user_type = b.code and t.user_id in  (   ?  ,  ?  )
   2020-02-06 14:37:16.637  INFO 6768 --- [           main] ShardingSphere-SQL                       : Actual SQL: s0 ::: select  *   from t_user t ,t_dict b  where t.user_type = b.code and t.user_id in  (   ?  ,  ?  ) ::: [1, 2]
   ```

#### 读写分离

读写分离的数据节点中的数据内容是一致的,而水平分片的每个数据节点的数据内容却并不相同。将水平分片和读写分离联合使用,能够更加有效的提升系统的性能。

Sharding-JDBC读写分离则是根据SQL语义的分析,将读操作和写操作分别路由至主库与从库。它提供透明化读写分离,让使用方尽量像使用一个数据库一样使用主从数据库集群

![1y4n0g.png](https://s2.ax1x.com/2020/02/06/1y4n0g.png)

Sharding-JDBC提供一主多从的读写分离配置,可独立使用,也可配合分库分表使用,同一线程且同一数据库连接内,如有写入操作,以后的读操作均从主库读取,用于保证数据一致性。Sharding-JDBC不提供主从数据库的数据同步功能,需要采用其他机制支持。

![1y4810.png](https://s2.ax1x.com/2020/02/06/1y4810.png)

接下来,咱们对上面例子中user_db进行读写分离实现。为了实现Sharding-JDBC的读写分离,首先,要进行mysql的主从同步配置。

配置方法参考：<https://hanyunpeng0521.github.io/2019/10/16/数据库-MySQL-12-1/>

1. 在Sharding-JDBC规则中修改

   ```properties
   # 增加数据源s0,使用上面主从同步配置的从库。
   spring.shardingsphere.datasource.names = m0,m1,m2,s0
   ...
   spring.shardingsphere.datasource.s0.type = com.alibaba.druid.pool.DruidDataSource
   spring.shardingsphere.datasource.s0.driver‐class‐name = com.mysql.jdbc.Driver
   spring.shardingsphere.datasource.s0.url = jdbc:mysql://localhost:3307/user_db?useUnicode=true
   spring.shardingsphere.datasource.s0.username = root
   spring.shardingsphere.datasource.s0.password = root
   ....
   # 主库从库逻辑数据源定义 ds0为user_db
   spring.shardingsphere.sharding.master‐slave‐rules.ds0.master‐data‐source‐name=m0
   spring.shardingsphere.sharding.master‐slave‐rules.ds0.slave‐data‐source‐names=s0
   # t_user分表策略,固定分配至ds0的t_user真实表
   spring.shardingsphere.sharding.tables.t_user.actual‐data‐nodes = ds0.t_user
   ....
   ```

2. 测试:执行testInsertUser单元测试,执行testSelectUserbyIds单元测试

   插入使用m0，读取使用s0

#### 总结示例

电商平台商品列表展示,每个列表项中除了包含商品基本信息、商品描述信息之外,还包括了商品所属的店铺信
息,如下：

![1sEpGR.png](https://s2.ax1x.com/2020/02/05/1sEpGR.png)

本案例实现功能如下:

1. 添加商品
2. 商品分页查询
3. 商品统计

**数据库设计**

数据库设计如下,其中商品与店铺信息之间进行了垂直分库,分为了PRODUCT_DB(商品库)和STORE_DB(店铺库);商品信息还进行了垂直分表,分为了商品基本信息(product_info)和商品描述信息(product_descript),地理区域信息(region)作为公共表,冗余在两库中:

![1yxNkt.png](https://s2.ax1x.com/2020/02/06/1yxNkt.png)

考虑到商品信息的数据增长性,对PRODUCT_DB(商品库)进行了水平分库,分片键使用店铺id,分片策略为店铺ID%2 + 1,因此商品描述信息对所属店铺ID进行了冗余;

对商品基本信息(product_info)和商品描述信息(product_descript)进行水平分表,分片键使用商品id,分片策略为商品ID%2 + 1,并将为这两个表设置为绑定表,避免笛卡尔积join;

为避免主键冲突,ID生成策略采用雪花算法来生成全局唯一ID,最终数据库设计为下图:

![1yxgkq.png](https://s2.ax1x.com/2020/02/06/1yxgkq.png)

要求使用读写分离来提升性能,可用性。

参考读写分离章节,对以下库进行主从同步配置:

```properties
# 设置需要同步的数据库
binlog‐do‐db=store_db
binlog‐do‐db=product_db_1
binlog‐do‐db=product_db_2
```

1. 初始化数据库

   ```sql
   -- 创建store_db数据库,并执行以下脚本创建表:
   CREATE DATABASE `store_db` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
   use `store_db`;
   DROP TABLE IF EXISTS `region`;
   CREATE TABLE `region`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '地理区域编码',
     `region_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '地理区域名称',
     `level` tinyint(1) NULL DEFAULT NULL COMMENT '地理区域级别(省、市、县)',
     `parent_region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '上级地理区域编码',
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   INSERT INTO `region` VALUES (1, '110000', '北京', 0, NULL);
   INSERT INTO `region` VALUES (2, '410000', '河南省', 0, NULL);
   INSERT INTO `region` VALUES (3, '110100', '北京市', 1, '110000');
   INSERT INTO `region` VALUES (4, '410100', '郑州市', 1, '410000');
   DROP TABLE IF EXISTS `store_info`;
   CREATE TABLE `store_info`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `store_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '店铺名称',
     `reputation` int(11) NULL DEFAULT NULL COMMENT '信誉等级',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '店铺所在地',
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   INSERT INTO `store_info` VALUES (1, 'XX零食店', 4, '110100');
   INSERT INTO `store_info` VALUES (2, 'XX饮品店', 3, '410100');
   
   -- 创建product_db_1、product_db_2数据库,并分别对两库执行以下脚本创建表:
   CREATE DATABASE `product_db_1` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
   use `product_db_1`;
   DROP TABLE IF EXISTS `product_descript_1`;
   CREATE TABLE `product_descript_1`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `product_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属商品id',
     `descript` longtext CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '商品描述',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `FK_Reference_2`(`product_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_descript_2`;
   CREATE TABLE `product_descript_2`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `product_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属商品id',
     `descript` longtext CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '商品描述',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `FK_Reference_2`(`product_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_info_1`;
   CREATE TABLE `product_info_1`
   (
     `product_info_id` bigint(20) NOT NULL COMMENT 'id',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     `product_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '商品名称',
     `spec` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '规
   格',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '产地',
     `price` decimal(10, 0) NULL DEFAULT NULL COMMENT '商品价格',
     `image_url` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '商品图片',
     PRIMARY KEY (`product_info_id`) USING BTREE,
     INDEX `FK_Reference_1`(`store_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_info_2`;
   CREATE TABLE `product_info_2`
   (
     `product_info_id` bigint(20) NOT NULL COMMENT 'id',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     `product_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '商品名称',
     `spec` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '规
   格',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '产地',
     `price` decimal(10, 0) NULL DEFAULT NULL COMMENT '商品价格',
     `image_url` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '商品图片',
     PRIMARY KEY (`product_info_id`) USING BTREE,
     INDEX `FK_Reference_1`(`store_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   
   DROP TABLE IF EXISTS `region`;
   CREATE TABLE `region`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '地理区域编码',
     `region_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '地理区域名称',
     `level` tinyint(1) NULL DEFAULT NULL COMMENT '地理区域级别(省、市、县)',
     `parent_region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '上级地理区域编码',
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   INSERT INTO `region` VALUES (1, '110000', '北京', 0, NULL);
   INSERT INTO `region` VALUES (2, '410000', '河南省', 0, NULL);
   INSERT INTO `region` VALUES (3, '110100', '北京市', 1, '110000');
   INSERT INTO `region` VALUES (4, '410100', '郑州市', 1, '410000');
   
   -- 创建product_db_1、product_db_2数据库,并分别对两库执行以下脚本创建表:
   CREATE DATABASE `product_db_2` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
   use `product_db_2`;
   DROP TABLE IF EXISTS `product_descript_1`;
   CREATE TABLE `product_descript_1`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `product_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属商品id',
     `descript` longtext CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '商品描述',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `FK_Reference_2`(`product_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_descript_2`;
   CREATE TABLE `product_descript_2`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `product_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属商品id',
     `descript` longtext CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '商品描述',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     PRIMARY KEY (`id`) USING BTREE,
     INDEX `FK_Reference_2`(`product_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_info_1`;
   CREATE TABLE `product_info_1`
   (
     `product_info_id` bigint(20) NOT NULL COMMENT 'id',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     `product_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '商品名称',
     `spec` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '规
   格',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '产地',
     `price` decimal(10, 0) NULL DEFAULT NULL COMMENT '商品价格',
     `image_url` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '商品图片',
     PRIMARY KEY (`product_info_id`) USING BTREE,
     INDEX `FK_Reference_1`(`store_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   DROP TABLE IF EXISTS `product_info_2`;
   CREATE TABLE `product_info_2`
   (
     `product_info_id` bigint(20) NOT NULL COMMENT 'id',
     `store_info_id` bigint(20) NULL DEFAULT NULL COMMENT '所属店铺id',
     `product_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '商品名称',
     `spec` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '规
   格',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '产地',
     `price` decimal(10, 0) NULL DEFAULT NULL COMMENT '商品价格',
     `image_url` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '商品图片',
     PRIMARY KEY (`product_info_id`) USING BTREE,
     INDEX `FK_Reference_1`(`store_info_id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   
   DROP TABLE IF EXISTS `region`;
   CREATE TABLE `region`
   (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT
       '地理区域编码',
     `region_name` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '地理区域名称',
     `level` tinyint(1) NULL DEFAULT NULL COMMENT '地理区域级别(省、市、县)',
     `parent_region_code` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
       COMMENT '上级地理区域编码',
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   INSERT INTO `region` VALUES (1, '110000', '北京', 0, NULL);
   INSERT INTO `region` VALUES (2, '410000', '河南省', 0, NULL);
   INSERT INTO `region` VALUES (3, '110100', '北京市', 1, '110000');
   INSERT INTO `region` VALUES (4, '410100', '郑州市', 1, '410000');
   ```

2. 项目：<https://github.com/hanyunpeng0521/mall/tree/master/shopping-sharing-jdbc>

### 参考

1. [数据库分库分表策略，如何分库，如何分表？](https://www.jianshu.com/p/f093ff9ace4b)
2. [面试官：“谈谈分库分表吧？”](https://baijiahao.baidu.com/s?id=1622441635115622194&wfr=spider&for=pc)

### 附 SQL支持说明

详细参考:https://shardingsphere.apache.org/document/current/cn/features/sharding/use-norms/sql/

