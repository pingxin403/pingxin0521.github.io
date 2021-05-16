---
title: MySQL数据库 分库分表  MyCat（三）
date: 2019-11-16 13:18:59
tags:
 - MySQL
 - 数据库
 - Java
categories:
 - 数据库
 - MySQL
---

**MyCat的作用和特点是什么？**

<!--more-->

MyCat是目前最流行的**基于Java**语言编写的**数据库中间件**，是一个实现了**MySQL**协议的服务器，前端用户可以把它看作是一个**数据库代理**，用MySQL**客户端工具**和**命令行**访问，而其后端可以用MySQL**原生协议**与多个MySQL**服务器通信**，也可以用**JDBC协议**与大多数主流数据库服务器通信，其核心功能是**分库分表**。配合数据库的主从模式还可实现**读写分离**。

MyCat是基于**阿里开源的Cobar**产品而研发，Cobar的**稳定性**、**可靠性**、**优秀的架构**和**性能**以及众多成熟的使用案例使得MyCat变得非常的强大。

MyCat发展到目前的版本，已经不是一个单纯的**MySQL代理**了，它的后端可以支持**MySQL**、**SQL Server**、**Oracle**、**DB2**、**PostgreSQL**等主流数据库，也支持**MongoDB**这种新型NoSQL方式的存储，未来还会支持更多类型的存储。而在最终用户看来，无论是那种存储方式，在MyCat里，都是一个**传统的数据库表**，支持**标准的SQL语句**进行数据的操作，这样一来，对前端业务系统来说，可以大幅降低开发难度，提升开发速度。

MyCat官网：[http://www.mycat.io/](https://link.zhihu.com/?target=http%3A//www.mycat.io/)

MyCAT的目标是：**低成本**的将现有的**单机数据库**和**应用**平滑**迁移到“云”端**，解决**数据存储**和**业务规模**迅速增长情况下的数据瓶颈问题。从这一点介绍上来看，能满足数据库数据大量存储，提高了查询性能。MyCat在大数据方面的运用不容小觑啊。

#### MyCAT特性

- 支持 SQL 92标准
- 支持Mysql集群，可以作为Proxy使用
- 支持JDBC连接ORACLE、DB2、SQL Server，将其模拟为MySQL Server使用
- 支持galera for mysql集群，percona-cluster或者mariadb cluster，提供高可用性数据分片集群
- 自动故障切换，高可用性
- 支持读写分离，支持Mysql双主多从，以及一主多从的模式
- 支持全局表，数据自动分片到多个节点，用于高效表关联查询
- 支持独有的基于E-R 关系的分片策略，实现了高效的表关联查询
- 多平台支持，部署和实施简单

#### 功能

mycat的三大功能：数据分片、读写分离、主从切换

1. 数据分片
   垂直拆分(分库)、水平拆分(分表)、垂直+水平拆分(分库分表)

   ![12Uy7V.png](https://s2.ax1x.com/2020/02/07/12Uy7V.png)

2. 读写分离
   经过统计发现，对数据库的大量操作是读操作，一般占到所有操作70%以上。所以做读写分离还是很有必要的，如果不做读写分离，那么从库也是一种很大的浪费。

   ![12U1OI.png](https://s2.ax1x.com/2020/02/07/12U1OI.png)

3. 多数据源整合

   ![12Uf1J.png](https://s2.ax1x.com/2020/02/07/12Uf1J.png)

#### 原理

Mycat 的原理中最重要的一个动词是“拦截”,它拦截了用户发送过来的 SQL 语句,首先对 SQL语句做了一些特定的分析:如分片分析、路由分析、读写分离分析、缓存分析等,然后将此 SQL 发往后端的真实数据库,并将返回的结果做适当的处理,最终再返回给用户。

![12UIn1.png](https://s2.ax1x.com/2020/02/07/12UIn1.png)

这种方式把数据库的分布式从代码中解耦出来,程序员察觉不出来后台使用 Mycat 还是MySQL。

#### MyCat 基本元素

```
1.逻辑库，mycat中存在，对应用来说相当于mysql数据库，后端可能对应了多个物理数据库，逻辑库中不保存数据
2.逻辑表，逻辑库中的表，对应用来说相当于mysql的数据表，后端可能对应多个物理数据库中的表，也不保存数据
逻辑表分类
1.分片表，进行了水平切分的表，具有相同表结构但存储在不同数据库中的表，所有分片表的集合才是一张完整的表
2.非分片表，垂直切分的表，一个数据库中就保存了一张完整的表
3.全局表，所有分片数据库中都存在的表，如字典表，数量少，由mycat来进行维护更新
4.ER关系表，mycat独有，子表依赖父表，保证在同一个数据库中
```

#### 数据库中间件对比

![12NqQs.png](https://s2.ax1x.com/2020/02/07/12NqQs.png)

1. Cobar属于阿里B2B事业群,始于2008年,在阿里服役3年多,接管3000+个MySQL数据库的schema,集群日处理在线SQL请求50亿次以上。由于Cobar发起人的离职,Cobar停止维护。
2. Mycat是开源社区在阿里cobar基础上进行二次开发,解决了cobar存在的问题,并且加入了许多新的功能在其中。青出于蓝而胜于蓝。
3. OneProxy基于MySQL官方的proxy思想利用c进行开发的,OneProxy是一款商业收费的中间件。舍弃了一些功能,专注在性能和稳定性上。
4. kingshard由小团队用go语言开发,还需要发展,需要不断完善。
5. Vitess是Youtube生产在使用,架构很复杂。不支持MySQL原生协议,使用需要大量改造成本。
6. Atlas是360团队基于mysql proxy改写,功能还需完善,高并发下不稳定。
7. MaxScale是mariadb(MySQL原作者维护的一个版本) 研发的中间件
8. MySQL Route是MySQL官方Oracle公司发布的中间件

#### MyCat 实现读写分离

1. 部署环境

    MyCAT 是使用 JAVA 语言进行编写开发，使用前需要先安装 JAVA 运行环境(JRE),由于 MyCAT 中使用了 JDK7 中的一些特性，所以要求必须在 JDK7 以上的版本上运行。

2. 部署MyCat

   ```shell
   wget http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz -O Mycat-server.tar.gz
   tar -xf Mycat-server.tar.gz 
   
   ```

**文件**

- bin : 二进制文件，用于管理mycat，如启动start、重启restart、停止stop、查看状态status等
- conf : 配置文件
  - server.xml 用于配置Mycat的用户名、密码、逻辑数据库名、服务端口、读写权限等
  - schema.xml 最常用的配置文件，用于配置物理数据库信息(ip、port、username、password、database)、表的配置(表所在的数据节点、表的分片规则、主键是否自增等)、读写分离配置等
  - rule.xml 分片规则配置，mycat提供了十多种分片规则，也可以自定义分片规则
  - log4j2.xml 配置mycat的日志信息，开发的时候建议设置成debug级别

- lib : Mycat是Java语言开发的，这是引用的jar包
- logs: Mycat打印的日志文件
  - console.log mycat启动时的日志，启动成功一般会有有日志记录，如果没有日志记录可以通过mycat status命令来查看是否启动成功，如果启动失败可以去mycat.log中查看错误日志
  - mycat.log mycat执行sql对应的日志，可以通过该日志知道mycat是怎么样执行sql的，一些错误日志会输出到该文件中，是一个很重要的日志文件
  - wrapper.log 错误日志也可能在这个日志文件中
  - switch.log

##### 配置MyCat

MyCAT 目前主要通过配置文件的方式来定义逻辑库和相关配置:

1. /usr/local/mycat/conf/server.xml定义用户以及系统相关变量，如端口等。其中用户信息是前端应用程序连接 mycat 的用户信息。

2. /usr/local/mycat/conf/schema.xml定义逻辑库，表、分片节点等内容。

3. /usr/local/mycat/conf/rule.xml中定义分片规则。

###### 配置 `server.xml`

以下为代码片段

下面的用户和密码是应用程序连接到 MyCat 使用的，可以自定义配置

而其中的`schemas` 配置项所对应的值是逻辑数据库的名字，也可以自定义，但是这个名字需要和后面 `schema.xml` 文件中配置的一致。

```xml
<mycat:server>
<user name="root">
		<property name="password">123456</property>
		<property name="schemas">TESTDB</property>
		
		<!-- 表级 DML 权限设置 -->
		<!-- 		
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>		
		 -->
	</user>

	<user name="user">
		<property name="password">user</property>
		<property name="schemas">TESTDB</property>
		<property name="readOnly">true</property>
	</user>

</mycat:server>
```

==上面的配置中，假如配置了用户访问的逻辑库，那么必须在 `schema.xml` 文件中也配置这个逻辑库，否则报错，启动 mycat 失败==

###### 配置 `schema.xml`

以下是配置文件中的每个部分的配置块儿

**逻辑库和分表设置**

```jsx
<schema name="TESTDB"   // 逻辑库名称
        checkSQLschema="false"   // 不检查
        sqlMaxLimit="100"        // 最大连接数
        dataNode='dn1'>          // 数据节点名称
<!--这里定义的是分表的信息-->
</schema>
```

**数据节点**

```jsx
<dataNode name="dn1"              // 此数据节点的名称
          dataHost="localhost1"   // 主机组
          database="test">    // 真实的数据库名称
</dataNode>
```

**主机组**

```jsx
<dataHost name="localhost1"
            maxCon="1000" minCon="10"   // 连接
            balance="0"                 // 负载均衡
            writeType="0"               // 写模式配置
            dbType="mysql" dbDriver="native" // 数据库配置
            switchType="1" slaveThreshold="100">
    <!--这里可以配置关于这个主机组的成员信息，和针对这些主机的健康检查语句-->
</dataHost>
```

![12NSxA.png](https://s2.ax1x.com/2020/02/07/12NSxA.png)

**健康检查**

```xml
<heartbeat>select user()</heartbeat>
```

**读写配置**

```xml
    <!-- can have multi write hosts -->
    <writeHost host="hostM1" url="127.0.0.1:3306"
               user="hyp" password="123456">
      <!-- can have multi read hosts -->
      <readHost host="hostS2" url="127.0.0.1:3307"
               user="hyp" password="123456">
    </writeHost>
```

以下是组合为完整的配置文件，适用于一主一从的架构

```xml
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

  <schema name="TESTDB" 
          checkSQLschema="false"
          sqlMaxLimit="100" 
          dataNode='dn1'>
    <!--这里定义的是分库分表的信息-->
  </schema>

  <!--下面是配置读写分离的信息-->
  <dataNode name="dn1"
            dataHost="localhost1" database="test">
  </dataNode>

  <dataHost name="localhost1"
            maxCon="1000" minCon="10"
            balance="0" writeType="0"
            dbType="mysql" dbDriver="native"
            switchType="1" slaveThreshold="100">

    <heartbeat>select user()</heartbeat>
    <!-- can have multi write hosts -->
    <writeHost host="hostM1" url="127.0.0.1:3306"
               user="hyp" password="123456">
      <!-- can have multi read hosts -->
      <readHost host="hostS2" url="127.0.0.1:3307"
               user="hyp" password="123456">
    </writeHost>
  </dataHost>
</mycat:schema>
```

###### 配置 `log4j2.xml`

```xml
<!--设置日志级别为 debug , 默认是 info-->
<asyncRoot level="debug" includeLocation="true">
```

##### 启动 mycat

```
./mycat start 启动

./mycat stop 停止

./mycat console 前台运行

./mycat install 添加到系统自动启动（暂未实现）

./mycat remove 取消随系统自动启动（暂未实现）

./mycat restart 重启服务

./mycat pause 暂停

./mycat status 查看启动状态
```

启动后测试

```
mysql -uroot -p123456 -P8066 -h127.0.0.1
```

### 参考

1. <https://github.com/MyCATApache/Mycat-Server>
2. https://www.bilibili.com/video/av80475096
3. [MyCat 之路 | 配置 Mysql 读写分离+强制走写节点+根据主从延时的读写分离](https://blog.csdn.net/liupeifeng3514/article/details/79092477)