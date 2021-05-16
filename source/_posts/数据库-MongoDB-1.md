---
title: MongoDB 安装和基本概念
date: 2019-05-16 13:18:59
tags:
 - NoSQL
 - MongoDB
 - 数据库
categories:
 - 数据库
 - MongoDB
---

### NoSQL

NoSQL(NoSQL = Not Only SQL )，意即"不仅仅是SQL"。

在现代的计算系统上每天网络上都会产生庞大的数据量。

这些数据有很大一部分是由关系数据库管理系统（RDBMS）来处理。  1970年 E.F.Codd's提出的关系模型的论文 "A relational model of data for large shared  data banks"，这使得数据建模和应用程序编程更加简单。

<!--more-->

通过应用实践证明，关系模型是非常适合于客户服务器编程，远远超出预期的利益，今天它是结构化数据存储在网络和商务应用的主导技术。 

NoSQL 是一项全新的数据库革命性运动，早期就有人提出，发展至2009年趋势越发高涨。NoSQL的拥护者们提倡运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。

#### 什么是NoSQL?

NoSQL，指的是非关系型的数据库。NoSQL有时也称作Not Only SQL的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。

NoSQL用于超大规模数据的存储。（例如谷歌或Facebook每天为他们的用户收集万亿比特的数据）。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

#### 为什么使用NoSQL ?

今天我们可以通过第三方平台（如：Google,Facebook等）可以很容易的访问和抓取数据。用户的个人信息，社交网络，地理位置，用户生成的数据和用户操作日志已经成倍的增加。我们如果要对这些用户数据进行挖掘，那SQL数据库已经不适合这些应用了,  NoSQL 数据库的发展却能很好的处理这些大的数据。

#### NoSQL的优点/缺点

优点:

-  \- 高可扩展性
- \- 分布式计算
- \- 低成本
- \- 架构的灵活性，半结构化数据
- \- 没有复杂的关系 

缺点:

- \- 没有标准化
- \- 有限的查询功能（到目前为止）
- \- 最终一致是不直观的程序



#### RDBMS vs NoSQL

RDBMS
- 高度组织化结构化数据
- 结构化查询语言（SQL） (SQL)
- 数据和关系都存储在单独的表中。
- 数据操纵语言，数据定义语言
- 严格的一致性
- 基础事务

NoSQL
- 代表着不仅仅是SQL
- 没有声明性查询语言
- 没有预定义的模式
- 键 - 值对存储，列存储，文档存储，图形数据库
- 最终一致性，而非ACID属性
- 非结构化和不可预知的数据
- CAP定理
- 高性能，高可用性和可伸缩性

####  CAP定理（CAP theorem）

在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点:

- **一致性(Consistency)** (所有节点在同一时间具有相同的数据)
- **可用性(Availability)** (保证每个请求不管成功或者失败都有响应)
- **分隔容忍(Partition tolerance)** (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

- CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
- CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。
- AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

#### BASE

BASE：Basically Available, Soft-state, Eventually Consistent。 由  Eric Brewer 定义。

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

BASE是NoSQL数据库通常对可用性及一致性的弱要求原则:

- Basically Availble --基本可用
- Soft-state --软状态/柔性事务。 "Soft state" 可以理解为"无连接"的, 而 "Hard state" 是"面向连接"的
- Eventual Consistency -- 最终一致性， 也是是 ACID 的最终目的。

#### ACID vs BASE

| ACID                    | BASE                                  |
| ----------------------- | ------------------------------------- |
| 原子性(**A**tomicity)   | 基本可用(**B**asically **A**vailable) |
| 一致性(**C**onsistency) | 软状态/柔性事务(**S**oft state)       |
| 隔离性(**I**solation)   | 最终一致性 (**E**ventual consistency) |
| 持久性 (**D**urable)    |                                       |

#### NoSQL 数据库分类

| 类型          | 部分代表                                                     | 特点                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 列存储        | Hbase  				 				 					Cassandra  				 				 					Hypertable | 顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。 |
| 文档存储      | MongoDB  				 				 					CouchDB | 文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。 |
| key-value存储 | Tokyo Cabinet / Tyrant  				 				 					Berkeley DB  				 				 					MemcacheDB  				 				 					Redis | 可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。（Redis包含了其他功能） |
| 图存储        | Neo4J  				 				 					FlockDB | 图形关系的最佳存储。使用传统关系数据库来解决的话性能低下，而且设计使用不方便。 |
| 对象存储      | db4o  				 				 					Versant | 通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据。 |
| xml数据库     | Berkeley DB XML  				 				 					BaseX | 高效的存储XML数据，并支持XML的内部查询语法，比如XQuery,Xpath。 |

### MongoDB

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key:value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

![1.png](https://i.loli.net/2019/05/24/5ce766e70821440351.png)

#### 特点

- MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。
- 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。
- 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
- 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
- Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。 
- MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
- Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
- Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
- Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
- GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
- MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。
- MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
- MongoDB安装简单。

### 安装

环境：Ubuntu 18.04 LTS

**二进制包**

从[官网下载](https://www.mongodb.com/download-center/community)最新的Release版安装包，选择Mongodb Community Server->合适的Version->适合自己的OS版本->Package选择TGZ，下载完成后，解压后。使用下面命令安装mongodb-compass-community
```
cd <你的包>/bin
sudo ./install_compass
```
把该包移动到`/opt/db/mongodb/`,并将`/opt/db/mongodb/bin`加入环境变量PATH中。

```
export PATH=<mongodb-install-directory>/bin:$PATH
```


**安装包**

从[官网下载](https://www.mongodb.com/download-center/community)最新的Release版安装包，选择Mongodb Community Server->合适的Version->适合自己的OS版本->Package选择Server、tools、mongos，下载完成后，安装即可。

java连接mongodb：

```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.10.2</version>
</dependency>
```

#### 创建数据库目录

MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。

以下实例中我们将data目录创建于根目录下(/)。

注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。

```
sudo mkdir -p /data/db
sudo chmod 777  /data/db
```

#### 命令行中运行 MongoDB 服务

你可以再命令行中执行mongo安装目录中的bin目录执行mongod命令来启动mongdb服务。

> 注意：如果你的数据库目录不是/data/db，可以通过 --dbpath 来指定。

```
$ mongod &
$ mongod --shutdown #关闭
```

####  MongoDB后台管理 Shell

如果你需要进入MongoDB后台管理，你需要先打开mongodb装目录的下的bin目录，然后执行mongo命令文件。

MongoDB Shell是MongoDB自带的交互式Javascript shell,用来对MongoDB进行操作和管理的交互式环境。

当你进入mongoDB后台后，它默认会链接到 test 文档（数据库）：

```
$ mongo
```

由于它是一个JavaScript shell，您可以运行一些简单的算术运算:

```
> 39-5
34
```

现在让我们插入一些简单的数据，并对插入的数据进行检索：

```
> db.learn.insert({x:100})
2019-05-24T11:50:32.283+0800 I STORAGE  [conn1] createCollection: test.learn with generated UUID: 3a3428a3-1b50-425b-81b2-7de80d07ee8d
WriteResult({ "nInserted" : 1 })
> db.learn.find()
{ "_id" : ObjectId("5ce76a08f4a0770652b0c800"), "x" : 100 }
```

第一个命令将数字 100 插入到 learn 集合的 x 字段中。

#### MongoDb web 用户界面

 MongoDB 提供了简单的 HTTP 用户界面。 如果你想启用该功能，需要在启动的时候指定参数 --rest 。

> **注意**：该功能只适用于 MongoDB 3.2 及之前的早期版本。

```
$ ./mongod --dbpath=/data/db --rest
```

MongoDB 的 Web 界面访问端口比服务的端口多1000。

如果你的MongoDB运行端口使用默认的27017，你可以在端口号为28017访问web用户界面，即地址为：http://localhost:28017。

#### docker 安装mongoDB

```bash
$ docker pull mongo
$ docker volume create mongodb_data
$ docker images mongo
#运行
$ docker run --name mongo  -p 27017:27017 -v mongodb_data:/data/db -d mongo
$ docker exec -it mongo mongo admin
# 创建一个名为 admin，密码为 123456 的用户。
>  db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'}]});
# 尝试使用上面创建的用户信息进行连接。
> db.auth('admin', '123456')
```

命令说明：

**-p 27017:27017 :**将容器的27017(第一个) 端口映射到主机的27017 端口

**-v mongodb_data:/data/db :**将主机中mongodb_data(第一个)挂载到容器的/data/db，作为mongo数据存储目录

查看容器启动情况

```
$ docker ps
```

使用mongo镜像执行mongo 命令连接到刚启动的容器,主机IP为172.17.0.1，查看docker0网卡的IP地址

```
$ docker run -it mongo mongo --host 172.17.0.1
```

### 工具

MongoDB 在 bin 目录下提供了一系列有用的工具，这些工具提供了 MongoDB 在运维管理上 的方便。

| 工具                                                       | 描述                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| [mongosniff](https://www.mongodb.org.cn/manual/201.html)   | mongodb监测工具，作用类似于 tcpdump                          |
| [mongodump](https://www.mongodb.org.cn/manual/193.html)    | MongoDB数据备份工具                                          |
| [mongoimport](https://www.mongodb.org.cn/manual/197.html)  | Mongodb数据导入工具                                          |
| [mongoexport](https://www.mongodb.org.cn/manual/198.html)  | Mongodb数据导出工具                                          |
| [bsondump](https://www.mongodb.org.cn/manual/195.html)     | 将 bson 格式的文件转储为 json 格式的数据                  |
| [mongoperf](https://www.mongodb.org.cn/manual/202.html)    |                                                              |
| [mongorestore](https://www.mongodb.org.cn/manual/194.html) | MongoDB数据恢复工具                                          |
| [mongod.exe](https://www.mongodb.org.cn/manual/188.html)   | MongoDB服务启动工具                                          |
| [mongostat](https://www.mongodb.org.cn/manual/199.html)    | mongodb自带的状态检测工具                                    |
| [mongofiles](https://www.mongodb.org.cn/manual/203.html)   | GridFS 管理工具，可实现二制文件的存取                        |
| [mongooplog](https://www.mongodb.org.cn/manual/196.html)   |                                                              |
| [mongotop](https://www.mongodb.org.cn/manual/200.html)     | 跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据 |
| [mongos](https://www.mongodb.org.cn/manual/189.html)       | 分片路由，如果使用了 sharding 功能，则应用程序连接的是 mongos 而不是 mongod |
| [mongo](https://www.mongodb.org.cn/manual/190.html)        | 客户端命令行工具，其实也是一个 js 解释器，支持 js 语法       |

### 概念

不管我们学习什么数据库都应该学习其中的基础概念，在mongodb中基本的概念是文档、集合、数据库，下面我们挨个介绍。

下表将帮助您更容易理解Mongo中的一些概念：

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |

通过下图实例，我们也可以更直观的了解Mongo中的一些概念：

![2.png](https://i.loli.net/2019/05/24/5ce76c3ce975459046.png)

#### 数据库

一个mongodb中可以建立多个数据库。

MongoDB的默认数据库为"db"，该数据库存储在data目录中。

MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限，不同的数据库也放置在不同的文件中。

**"show dbs"** 命令可以显示所有数据的列表。

```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```

执行 **"db"** 命令可以显示当前数据库对象或集合。

```
> db
test
```

运行"use"命令，可以连接到一个指定的数据库。

```
> use local
switched to db local
> db
local
```

以上实例命令中，"local" 是你要链接的数据库。

数据库也通过名字来标识。数据库名可以是满足以下条件的任意UTF-8字符串。

- 不能是空字符串（"")。
- 不得含有' '（空格)、.、$、/、\和\0 (空字符)。
- 应全部小写。
- 最多64字节。

有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。

- **admin**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
- **local:** 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合 
- **config**: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

#### 集合

集合就是 MongoDB 文档组，类似于 RDBMS （关系数据库管理系统：Relational Database Management System)中的表格。

 集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

集合相当于关系型数据库中的表。集合就是一组MongoDB文档。集合存在于数据库中，集合没有固定的结构，这意味着可以对集合插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

比如，我们可以将以下不同数据结构的文档插入到集合中：

```
{"site":"www.baidu.com"}
{"site":"www.google.com","name":"Google"}
{"site":"www.163.com","name":"网易","num":5}
```

当第一个文档插入时，集合就会被创建。

**合法的集合名**

- 集合名不能是空字符串""。
- 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。
- 集合名不能以"system."开头，这是为系统集合保留的前缀。
- 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。　

如下实例：

```
db.col.findOne()
```

**capped collections**

Capped collections 就是固定大小的collection。

它有很高的性能以及队列过期的特性(过期按照插入的顺序). 有点和 "RRD" 概念类似。

Capped collections 是高性能自动的维护对象的插入顺序。它非常适合类似记录日志的功能和标准的 collection  不同，你必须要显式的创建一个capped collection，指定一个 collection 的大小，单位是字节。collection  的数据存储空间值提前分配的。

Capped collections  可以按照文档的插入顺序保存到集合中，而且这些文档在磁盘上存放位置也是按照插入顺序来保存的，所以当我们更新Capped collections  中文档的时候，更新后的文档不可以超过之前文档的大小，这样话就可以确保所有文档在磁盘上的位置一直保持不变。

由于 Capped collection 是按照文档的插入顺序而不是使用索引确定插入位置，这样的话可以提高增添数据的效率。MongoDB 的操作日志文件 oplog.rs 就是利用 Capped Collection 来实现的。

要注意的是指定的存储大小包含了数据库的头信息。

```
db.createCollection("mycoll", {capped:true, size:100000})
```

- 在 capped collection 中，你能添加新的对象。
- 能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。
- 使用 Capped Collection 不能删除一个文档，可以使用 drop() 方法删除 collection 所有的行。
- 删除之后，你必须显式的重新创建这个 collection。
- 在32bit机器中，capped collection 最大存储为 1e9( 1X109)个字节。

#### 文档(Document)

 文档是一组键值(key-value)对(即 BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

文档相当于关系型数据库中的行。它一组键值对，具有动态的模式，所以文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这一点，跟关系型数据库有很大的不同。

**注：**

1. 文档中的键/值对是有序的。
2. MongoDB区分类型和大小写。
3. MongoDB的文档不能有重复的键。
4. 键不能含有\0 (空字符)。
5. .和$有特别的意义，只有在特定环境下才能使用。

一个简单的文档例子如下：

```
{"site":"www.baidu.com", "name":"百度"}
```

| RDBMS  | MongoDB                            |
| ------ | ---------------------------------- |
| 数据库 | 数据库                             |
| 表格   | 集合                               |
| 行     | 文档                               |
| 列     | 字段                               |
| 表联合 | 嵌入文档                           |
| 主键   | 主键 (MongoDB 提供了 key  为 _id ) |

命令行比较：

| 数据库服务和客户端 |        |
| :----------------: | ------ |
|   Mysqld/Oracle    | mongod |
|   mysql/sqlplus    | mongo  |

需要注意的是：

1. 文档中的键/值对是有序的。
2. 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
3. MongoDB区分类型和大小写。
4. MongoDB的文档不能有重复的键。
5. 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。 

文档键命名规范：

- 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
- .和$有特别的意义，只有在特定环境下才能使用。
- 以下划线"_"开头的键是保留的(不是严格要求的)。



#### 元数据

数据库的信息是存储在集合中。它们使用了系统的命名空间：

```
dbname.system.*
```

在MongoDB数据库中名字空间 <dbname\>.system.* 是包含多种系统信息的特殊集合(Collection)，如下:

| 集合命名空间             | 描述                                      |
| ------------------------ | ----------------------------------------- |
| dbname.system.namespaces | 列出所有名字空间。                        |
| dbname.system.indexes    | 列出所有索引。                            |
| dbname.system.profile    | 包含数据库概要(profile)信息。             |
| dbname.system.users      | 列出所有可访问数据库的用户。              |
| dbname.local.sources     | 包含复制对端（slave）的服务器信息和状态。 |

对于修改系统集合中的对象有如下限制。

在{{system.indexes}}插入数据，可以创建索引。但除此之外该表信息是不可变的(特殊的drop index命令将自动更新相关信息)。 

{{system.users}}是可修改的。 {{system.profile}}是可删除的。

#### MongoDB 数据类型

下表为MongoDB中常用的几种数据类型。

| 数据类型           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。 |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。 |
| Boolean            | 布尔值。用于存储布尔值（真/假）。                            |
| Double             | 双精度浮点值。用于存储浮点值。                               |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。 |
| Array              | 用于将数组或列表或多个值存储为一个键。                       |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间。                       |
| Object             | 用于内嵌文档。                                               |
| Null               | 用于创建空值。                                               |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。 |
| Object ID          | 对象 ID。用于创建文档的 ID。                                 |
| Binary Data        | 二进制数据。用于存储二进制数据。                             |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码。                 |
| Regular expression | 正则表达式类型。用于存储正则表达式。                         |

下面说明下几种重要的数据类型。

##### ObjectId

ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，含义是：

- 前 4 个字节表示创建 **unix** 时间戳,格林尼治时间 **UTC** 时间，比北京时间晚了 8 个小时
- 接下来的 3 个字节是机器标识码
- 紧接的两个字节由进程 id 组成 PID
- 最后三个字节是随机数

![3.jpeg](https://i.loli.net/2019/05/24/5ce76e38134d441304.jpeg)

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象

由于 ObjectId 中保存了创建的时间戳，所以你不需要为你的文档保存时间戳字段，你可以通过 getTimestamp 函数来获取文档的创建时间:

```
> var newObject = ObjectId()
> newObject.getTimestamp()
ISODate("2019-05-24T04:09:04Z")
```

ObjectId 转为字符串

```
> newObject.str
5ce76e60254d144d4c80b60e
```

##### 字符串

BSON 字符串都是 UTF-8 编码。

##### 时间戳

BSON 有一个特殊的时间戳类型用于 MongoDB 内部使用，与普通的 日期 类型不相关。 时间戳值是一个 64 位的值。其中：

- 前32位是一个 time_t 值（与Unix新纪元相差的秒数）
- 后32位是在某秒中操作的一个递增的`序数` 

在单个 mongod 实例中，时间戳值通常是唯一的。

在复制集中， oplog 有一个 ts 字段。这个字段中的值使用BSON时间戳表示了操作时间。

> BSON 时间戳类型主要用于 MongoDB 内部使用。在大多数情况下的应用开发中，你可以使用 BSON 日期类型。

##### 日期

表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期。

```
> var mydate1 = new Date()     //格林尼治时间
> var mydate2 = ISODate() //格林尼治时间
> mydate1
ISODate("2019-05-24T04:13:27.899Z")
> typeof mydate1
object
```

这样创建的时间是日期类型，可以使用 JS 中的 Date 类型的方法。

返回一个时间类型的字符串：

```
> var mydate1str=mydate1.toString()
> mydate1str
Fri May 24 2019 12:13:27 GMT+0800 (CST)
> typeof mydate1str
string

```

### 连接

执行启动操作后，mongodb 在输出一些必要信息后不会输出任何信息，之后就等待连接的建立，当连接被建立后，就会开始打印日志信息。

你可以使用 MongoDB shell 来连接 MongoDB 服务器。你也可以使用 PHP 来连接 MongoDB。使用  MongoDB shell 来连接 Mongodb 服务，

标准 URI 连接语法：

```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

- **mongodb://** 这是固定的格式，必须要指定。
- **username:password@** 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
-  **host1** 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
- **portX** 可选的指定端口，如果不填，默认为27017
-  **/database** 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。 
-  **?options** 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开 

标准的连接格式包含了多个选项(options)，如下所示：

| 选项                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| replicaSet=name     | 验证replica set的名称。 Impliesconnect=replicaSet.           |
| slaveOk=true\|false | true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。 	false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
| safe=true\|false    | true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).   false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。 |
| w=n                 | 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。     |
| wtimeoutMS=ms       | 驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true. |
| fsync=true\|false   | true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true. 	false: 驱动不会添加到getLastError命令中。 |
| journal=true\|false | 如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true |
| connectTimeoutMS=ms | 可以打开连接的时间。                                         |
| socketTimeoutMS=ms  | 发送和接受sockets的时间。                                    |

使用默认端口来连接 MongoDB 的服务。

```
$ mongo
```

#### 实例

连接本地数据库服务器，端口是默认的。

```
mongodb://localhost
```

使用用户名fred，密码foobar登录localhost的admin数据库。

```
mongodb://fred:foobar@localhost
```

使用用户名fred，密码foobar登录localhost的baz数据库。

```
mongodb://fred:foobar@localhost/baz
```

连接 replica pair, 服务器1为example1.com服务器2为example2。

```
mongodb://example1.com:27017,example2.com:27017
```

连接 replica set 三台服务器 (端口 27017, 27018, 和27019):

```
mongodb://localhost,localhost:27018,localhost:27019
```

连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。

```
mongodb://host1,host2,host3/?slaveOk=true
```

直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。

```
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```

当你的连接服务器有优先级，还需要列出所有服务器，你可以使用上述连接方式。

安全模式连接到localhost:

```
mongodb://localhost/?safe=true
```

以安全模式连接到replica set，并且等待至少两个复制服务器成功写入，超时时间设置为2秒。

```
mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000
```
