---
title: MySQL 源码编译安装
date: 2019-05-01 13:08:59
tags:
 - 数据库
 - MySQL
categories:
 - 数据库
 - MySQL
---

### 前言
![mysql.jpeg](https://i.loli.net/2019/04/02/5ca3684e8dc0f.jpeg)
MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 是最流行的关系型数据库管理系统之一，在 WEB 应用方面，MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件。

<!--more-->

MySQL是一种关系数据库管理系统，关系数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了双授权政策，分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。
由于其社区版的性能卓越，搭配 PHP 和 Apache 可组成良好的开发环境。

和其他关系数据库有所不同的是，**MySQL本身实际上只是一个SQL接口**，它的内部还包含了多种数据引擎，常用的包括：

- InnoDB：由Innobase Oy公司开发的一款支持事务的数据库引擎，2006年被Oracle收购；
- MyISAM：MySQL早期集成的默认数据库引擎，不支持事务。

MySQL接口和数据库引擎的关系就好比某某浏览器和浏览器引擎（IE引擎或Webkit引擎）的关系。对用户而言，切换浏览器引擎不影响浏览器界面，切换MySQL引擎不影响自己写的应用程序使用MySQL的接口。

使用MySQL时，不同的表还可以使用不同的数据库引擎。如果你不知道应该采用哪种引擎，记住总是选择*InnoDB*就好了。

因为MySQL一开始就是开源的，所以基于MySQL的开源版本，又衍生出了各种版本：

**MariaDB**

由MySQL的创始人创建的一个开源分支版本，使用XtraDB引擎。

**Aurora**

由Amazon改进的一个MySQL版本，专门提供给在AWS托管MySQL用户，号称5倍的性能提升。

**PolarDB**

由Alibaba改进的一个MySQL版本，专门提供给在[阿里云](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=cz36baxa)托管的MySQL用户，号称6倍的性能提升。

而MySQL官方版本又分了好几个版本：

- Community Edition：社区开源版本，免费；
- Standard Edition：标准版；
- Enterprise Edition：企业版；
- Cluster Carrier Grade Edition：集群版。

以上版本的功能依次递增，价格也依次递增。不过，功能增加的主要是监控、集群等管理功能，对于基本的SQL功能是完全一样的。

所以使用MySQL就带来了一个巨大的好处：可以在自己的电脑上安装免费的Community Edition版本，进行学习、开发、测试，部署的时候，可以选择付费的高级版本，或者云服务商提供的兼容版本，而不需要对应用程序本身做改动。

高效查看mysql文档：<https://www.cnblogs.com/chenqionghe/p/4834565.html>

### 编译安装

> 环境：Ubuntu18.10+Mysql 8.0.16

0. 下载source源码略（最好下载带boost那个）
1. 安装工具：`sudo apt-get install gcc  g++ cpp  cmake zlib1g`
2. 安装依赖包：`sudo apt-get install openssl libssl-dev libncurses5-dev bison libxml2 libxml2-dev`  
3. 添加用户： `groupadd mysql； useradd -r -g mysql -D /data/mysql -s /bin/false mysql`
4. `cd mysql-VERSION; mkdir bld; cd bld`
5. `cmake .. -DCMAKE_BUILD_TYPE=Debug -DWITH_BOOST=../boost -DWITH_SSL=system -DCMAKE_INSTALL_PREFIX=/opt/db/mysql`

> (我需要打开debug调试功能，你也许不需要就去掉前面的Debug, ps：可以用`cmake .. -LAH`查看所有支持选项)
> 注意，有时候装了依赖库后还会莫名报错，只要 rm CMakeCache.txt后重新cmake就能过去

6. `make -j 4`
7. `make install`  (/opt/mysql要先解决好权限）

现在是安装完毕，下面是配置。
#### 配置

1. 简单配置文件(可以放在/etc/my.cnf或者$HOME/.my.cnf)如下：

```
[client]
socket = /tmp/mysql.sock
[mysqld]
user = mysql
socket = /tmp/mysql.sock
basedir = /opt/mysql
datadir = /data/mysql
log-error = error.log
server-id = 3306
```

2. 按以上建立目录

``` (shell)
$ sudo mkdir -p /data/mysql
$ sudo chown mysql:mysql  /data/mysql -R
$ sudo chmod 777 /data/mysql -R
```

3. 初始化并开启MySQL服务器

``` (shell)
$ /opt/mysql/bin/mysqld --initialize-insecure
$ /opt/mysql/bin/mysqld_safe &
```

4. 客户端连接（直接回车即可）： 

``` (shell)
$ /opt/mysql/bin/mysql -u root -h localhost -p
```
5. 设置root本地密码

``` (shell)
> use mysql;
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
```

6. 设置完密码关闭数据库（会要求输入刚设置的密码）：　

``` (shell)
$ /opt/mysql/bin/mysqladmin -u root -h localhost -p shutdown
```

7. 设置环境变量。现在命令使用需要路径，加入环境变量PATH中，使用时省略路径。

``` (shell)
$ vim /etc/profile
#在末尾加入以下
export MYSQL_HOME=/opt/mysql
export PATH=${PATH}:${MYSQL_HOME}/bin
```
现在配置完成。


#### CMake 参数
```
-DCMAKE_INSTALL_PREFIX= 指向mysql安装目录,prefix官方推荐设为/usr
-DINSTALL_SBINDIR=sbin 指向可执行文件目录（prefix/sbin）
-DMYSQL_DATADIR=/var/lib/mysql 指向mysql数据文件目录（/var/lib/mysql）
-DSYSCONFDIR=/etc/mysql 指向mysql配置文件目录（/etc/mysql）
-DINSTALL_PLUGINDIR=lib/mysql/plugin 指向插件目录（prefix/lib/mysql/plugin）
-DINSTALL_MANDIR=share/man 指向man文档目录（prefix/share/man）
-DINSTALL_SHAREDIR=share 指向aclocal/mysql.m4安装目录（prefix/share）
-DINSTALL_LIBDIR=lib/mysql 指向对象代码库目录（prefix/lib/mysql）
-DINSTALL_INCLUDEDIR=include/mysql 指向头文件目录（prefix/include/mysql）
-DINSTALL_INFODIR=share/info 指向info文档存放目录（prefix/share/info）
```

**Storage Engine相关**

类型csv,myisam,myisammrg,heap,innobase,archive,blackhole,若想启用某个引擎的支持：`-DWITH_<ENGINE>_STORAGE_ENGINE=1`,
如：

```
-DWITH_INNOBASE_STORAGE_ENGINE=1
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
```
若想禁用某个引擎的支持：`-DWITHOUT_<ENGINE>_STORAGE_ENGINE=1`
如：
```
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1
-DWITHOUT_PARTITION_STORAGE_ENGINE=1
```

**Library相关**

```
-DWITH_READLINE=1 启用readline库支持（提供可编辑的命令行）
-DWITH_SSL=system 启用ssl库支持（安全套接层）
-DWITH_ZLIB=system 启用libz库支持（zib、gzib相关）
-DWTIH_LIBWRAP=0 禁用libwrap库（实现了通用TCP包装的功能，为网络服务守护进程使用）
-DMYSQL_TCP_PORT=3306 指定TCP端口为3306
-DMYSQL_UNIX_ADDR=/tmp/mysqld.sock 指定mysql.sock路径
-DENABLED_LOCAL_INFILE=1 启用本地数据导入支持
-DEXTRA_CHARSETS=all 启用额外的字符集类型（默认为all）
-DDEFAULT_CHARSET=utf8 指定默认的字符集为utf8
-DDEFAULT_COLLATION=utf8_general_ci 设定默认排序规则（utf8_general_ci快速/utf8_unicode_ci准确）
-DWITH_EMBEDDED_SERVER=1 编译嵌入式服务器支持
-DMYSQL_USER=mysql 指定mysql用户(默认为mysql)
-DWITH_DEBUG=0 禁用debug（默认为禁用）
-DENABLE_PROFILING=0 禁用Profiling分析（默认为开启）
-DWITH_COMMENT='string' 一个关于编译环境的描述性注释
```

#### 安装二进制包

环境：Ubuntu 18.04 + mysql 8.0.17

在`Ubuntu`中，默认情况下，只有最新版本的`MySQL`包含在`APT`软件包存储库中,要安装它，只需更新服务器上的包索引并安装默认包`apt-get`。

但是，默认最高只是5.7，不支持8.0以上版本，因此选择从官网上下载对应的apt安装包，下载连接：<https://dev.mysql.com/downloads/repo/apt/>，下载后安装，然后根据弹出的对应版本进行选择，选择完成后确定OK，然后就可以直接安装你所选定的mysql-server版本或者类型了，还可以选择安装 MySQL Tools & Connectors（including connectors,MySQL Workbench, MySQL Utilities and MySQL router）、MySQL Preview Packages等。

```
$ sudo dpkg -i /PATH/version-specific-package-name.deb
$ sudo apt-get update
$ sudo apt-get install mysql-server
```

然后在安装的时候设置好root密码，记住，此次安装是将mysql-server作为系统服务，因此从mysqld_safe命令运行会引起错误，应当使用service或者systemctl来运行或者关闭，配置文件在/etc/mysql下根据不同模块有不同的配置文件，具体查看自己的文件。

```
$ sudo service mysql status
$ sudo service mysql stop
$ sudo service mysql start
```

如果想安装更多包，请参看[官方文档](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)

#### docker 安装 mysql 8 版本

```bash
# docker 中下载 mysql
$ docker pull mysql

#启动
$ docker run  --name mysql-hyp -e MYSQL_ROOT_PASSWORD=123456 -d -p 3306:3306 mysql:latest
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
bbf8e1aec6a0        mysql:latest        "docker-entrypoint.s…"   5 seconds ago       Up 3 seconds        0.0.0.0:3306->3306/tcp, 33060/tcp   mysql-hyp
$ docker cp bbf8e1aec6a0:/etc/mysql/conf.d/* ~/docker/mysql/conf

$ docker run --name mysql -p 3306:3306 -v ~/docker/mysql/conf:/etc/mysql/conf.d -v ~/docker/mysql/logs:/logs -v ~/docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

#进入容器
$ docker exec -it mysql bash

#登录mysql
$ mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

#添加远程登录用户
CREATE USER 'pingxin'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'pingxin'@'%';
```

### 存储引擎

#####  什么是存储引擎

 数据库存储引擎是数据库底层软件组件，数据库管理系统使用数据引擎进行创建、查询、更新和删除数据操作。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎还可以获得特定的功能。

 现在许多数据库管理系统都支持多种不同的存储引擎。MySQL 的核心就是存储引擎。

提示：InnoDB 事务型数据库的首选引擎，支持事务安全表（ACID），支持行锁定和外键。MySQL 5.5.5 之后，InnoDB 作为默认存储引擎。

 MyISAM 是基于 ISAM 的存储引擎，并对其进行扩展，是在 Web、数据仓储和其他应用环境下最常使用的存储引擎之一。MyISAM 拥有较高的插入、查询速度，但不支持事务。

 MEMORY 存储引擎将表中的数据存储到内存中，为查询和引用其他数据提供快速访问。

#####  MySQL 5.7 支持的存储引擎

 MySQL 支持多种类型的数据库引擎，可分别根据各个引擎的功能和特性为不同的数据库处理任务提供各自不同的适应性和灵活性。在 MySQL 中，可以利用 SHOW ENGINES 语句来显示可用的数据库引擎和默认引擎。

 MySQL 提供了多个不同的存储引擎，包括处理事务安全表的引擎和处理非事务安全表的引擎。在 MySQL 中，不需要在整个服务器中使用同一种存储引擎，针对具体的要求，可以对每一个表使用不同的存储引擎。

 MySQL 5.7 支持的存储引擎有 InnoDB、MyISAM、Memory、Merge、Archive、Federated、CSV、BLACKHOLE 等。可以使用`SHOW ENGINES`语句查看系统所支持的引擎类型

Support 列的值表示某种引擎是否能使用，YES表示可以使用，NO表示不能使用，DEFAULT表示该引擎为当前默认的存储引擎。 

#####  如何选择 MySQL 存储引擎

不同的存储引擎都有各自的特点，以适应不同的需求，如表所示。为了做出选择，首先要考虑每一个存储引擎提供了哪些不同的功能。

| 功能         | MylSAM | MEMORY | InnoDB | Archive |
| ------------ | ------ | ------ | ------ | ------- |
| 存储限制     | 256TB  | RAM    | 64TB   | None    |
| 支持事务     | No     | No     | Yes    | No      |
| 支持全文索引 | Yes    | No     | No     | No      |
| 支持树索引   | Yes    | Yes    | Yes    | No      |
| 支持哈希索引 | No     | Yes    | No     | No      |
| 支持数据缓存 | No     | N/A    | Yes    | No      |
| 支持外键     | No     | No     | Yes    | No      |

可以根据以下的原则来选择 MySQL 存储引擎： 

-  如果要提供提交、回滚和恢复的事务安全（ACID 兼容）能力，并要求实现并发控制，InnoDB 是一个很好的选择。
-  如果数据表主要用来插入和查询记录，则 MyISAM 引擎提供较高的处理效率。
-  如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 MEMORY 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。
-  如果只有 INSERT 和 SELECT 操作，可以选择Archive 引擎，Archive 存储引擎支持高并发的插入操作，但是本身并不是事务安全的。Archive 存储引擎非常适合存储归档数据，如记录日志信息可以使用 Archive 引擎。

提示：使用哪一种引擎要根据需要灵活选择，一个数据库中多个表可以使用不同的引擎以满足各种性能和实际需求。使用合适的存储引擎将会提高整个数据库的性能。

#####  MySQL 默认存储引擎

InnoDB 是系统的默认引擎，支持可靠的事务处理。

 使用下面的语句可以修改数据库临时的默认存储引擎 

```
 SET default_storage_engine=< 存储引擎名 >
```

但是当再次重启客户端时，默认存储引擎仍然是 InnoDB。可以从配置文件进行修改属性。

###  MySQL 服务器端实用工具

1. mysqld
   SQL 后台程序（即 MySQL 服务器进程）。该程序必须运行之后，客户端才能通过连接服务器来访问数据库。
2. mysqld_safe
   服务器启动脚本。在 UNIX 和 NewWare 中推荐使用 mysqld_safe 来启动 mysqld 服务器。mysqld_safe 增加了一些安全性，例如，当出现错误时，重启服务器并向错误日志文件中写入运行时间信息。
3.  mysql.server
   服务器启动脚本。该脚本用于使用包含为特定级别的、运行启动服务器脚本的、运行目录的系统。它调用 mysqld_safe 来启动 MySQL 服务器。
4. mysqld_multi
   服务器启动脚本，可以启动或停止系统上安装的多个服务器。
5.  mysamchk
   用来描述、检查、优化和维护 MyISAM 表的实用工具。
6. mysql.server
   服务器启动脚本。在 UNIX 中的 MySQL 分发版包括 mysql.server 脚本。
7. mysqlbug
   MySQL 缺陷报告脚本。它可以用来向 MySQL 邮件系统发送缺陷报告。
8. mysql_install_db
   该脚本用默认权限创建 MySQL 授予权表。通常只是在系统上首次安装 MySQL 时执行一次。 

###  MySQL 客户端实用工具

1. myisampack
   压缩 MyISAM 表以产生更小的只读表的一个工具。
2.  mysql
   交互式输入 SQL 语句或从文件经批处理模式执行它们的命令行工具。
3. mysqlacceess
   检查访问主机名、用户名和数据库组合的权限的脚本。
4. mysqladmin
   执行管理操作的客户程序，例如创建或删除数据库、重载授权表、将表刷新到硬盘上以及重新打开日志文件。Mysqladmin 还可以用来检索版本、进程以及服务器的状态信息。
5.  mysqlbinlog
   从二进制日志读取语句的工具。在二进制日志文件中包含执行过的语句，可用来帮助系统从崩溃中恢复。
6. mysqlcheck
   检查、修复、分析以及优化表的表维护客户程序。
7. mysqldump
   将 MySQL 数据库转储到一个文件（例如 SQL 语句或 Tab 分隔符文本文件）的客户程序。
8.  mysqlhotcopy
   当服务器在运行时，快速备份 MyISAM 或 ISAM 表的工具。
9. mysql import
   使用 LOAD DATA INFILE 将文本文件导入相应的客户程序。
10.  mysqlshow
    显示数据库、表、列以及索引相关信息的客户程序。
11. perror
    显示系统或 MySQL 错误代码含义的工具。

### my.cnf配置文件

加载顺序：/etc/my.cnf >  /etc/mysql/my.cnf > ~/.my.cnf

以下是my.cnf配置文件参数解释：

```
[client]
port = 3306 #端口号
socket = /tmp/mysql.sock #socket所在路径
[mysqld]
!include /home/mysql/etc/mysqld.cnf #包含的配置文件 ，把用户名，密码文件单独存放
port = 3306
socket = /tmp/mysql.sock
pid-file = /home/mysql/var/mysql.pid#进程pid
basedir = /home/mysql/#mysql的安装路径
datadir = /home/mysql/var/ #数据文件所在路径
tmpdir = /home/mysql/tmp/#临时文件保存路径
slave-load-tmpdir=/home/mysql/tmp#当slave执行load data infile时用
skip-name-resolve#grant时，必须使用ip不能使用主机名
skip-symbolic-links#不能使用连接文件
skip-external-locking#不指定系统锁定
back_log = 50 #接受队列，对于没建立 tcp 连接的请求队列放入缓存中，队列大小为 back_log，受限制与 OS 参数
max_connections = 1000 #最大并发连接数 ，增大该值需要相应增加允许打开的文件描述符数
max_connect_errors = 10000 #如果某个用户发起的连接 error 超过该数值，则该用户的下次连接将被阻塞
open_files_limit = 10240#打开文件限制
connect-timeout = 10 #连接超时之前的最大秒数
wait-timeout = 28800 #等待关闭连接的时间
interactive-timeout = 28800 #关闭连接之前，允许 interactive_timeout（取代了wait_timeout）秒的不活动时间。
slave-net-timeout = 600#从服务器超过slave_net_timeout 秒没有从主服务器收到数据才通知网络中断
net_read_timeout = 30 #从服务器读取信息的超时
net_write_timeout = 60 #从服务器写入信息的超时
net_retry_count = 10 #如果某个通信端口的读操作中断了，在放弃前重试多次
net_buffer_length = 16384 #包消息缓冲区初始化字节
table_cache = 512 #所有线程打开的表的数目
thread_stack = 192K #每个线程的堆栈大小
thread_cache_size = 20 #线程缓存
thread_concurrency = 8 #同时运行的线程的数据 此处最好为 CPU 个数两倍。
query_cache_size = 256M #查询缓存大小
query_cache_limit = 2M #不缓存查询大于该值的结果
query_cache_min_res_unit = 2K #查询缓存分配的最小块大小
default_table_type = INNODB#默认表存储引擎
default-time-zone = system #服务器时区
character-set-server = utf8 #server 级别字符集
default-storage-engine = InnoDB #默认存储
tmp_table_size = 512M #临时表大小
log-bin = mysql-bin #打开binlog
log-bin-index = mysql-bin.index
relay-log = relay-log
relay_log_index = relay-log.index
log-error = /home/mysql/log/mysql.err#错误文件路径
log_output = FILE #慢查询输出格式
slow_query_log = 1
long-query-time = 1 #慢查询时间 超过 1 秒则为慢查询
slow_query_log_file = /home/mysql/log/slow.log#慢查询存储路径
general_log = 1
general_log_file = /home/mysql/log/mysql.log#一般查询存储路径
max_binlog_size = 1G#最大binlog
max_relay_log_size = 1G#最大relaylog
relay-log-purge = 1 #当不用中继日志时，删除他们。这个操作有 SQL 线程完成
expire_logs_days = 30 #超过 30 天的 binlog 删除
binlog_cache_size = 1M #session 级别
replicate-wild-ignore-table = mysql.% #复制时忽略数据库及表
replicate-wild-ignore-table = test.% #复制时忽略数据库及表
key_buffer_size = 256M#查询排序时所能使用的缓冲区大小
sort_buffer_size = 2M #排序 buffer 大小
read_buffer_size = 2M #读查询操作所能使用的缓冲区大小
join_buffer_size = 8M # join buffer 大小
query_cache_size = 64M#指定 MySQL 查询缓冲区的大小
read_rnd_buffer_size = 8M#随机读缓存大小
innodb_file_per_table#独立表空间
innodb_additional_mem_pool_size = 100M#附加的内存池
innodb_buffer_pool_size = 2G #缓冲池
innodb_data_file_path = ibdata1:1G:autoextend#表空间，自动递增
innodb_file_io_threads = 4 #io 线程数
innodb_thread_concurrency = 16 #并发线程数
innodb_flush_log_at_trx_commit = 1#刷新事务日志到磁盘
innodb_log_buffer_size = 8M #事物日志缓存
innodb_log_file_size = 500M #事物日志大小
innodb_log_files_in_group = 2 #两组事物日志
innodb_log_group_home_dir = /home/mysql/var/#日志组
innodb_max_dirty_pages_pct = 90 #innodb 主线程刷新缓存池中的数据，使脏数据比例小于 90%
innodb_lock_wait_timeout = 50 #InnoDB 事务在被回滚之前可以等待一个锁定的超时秒数
innodb_flush_method = O_DSYNC  # InnoDB 用来刷新日志的方法
innodb_force_recovery=1#导出表空间损坏的表
innodb_fast_shutdown#加速innodb关闭
max_allowed_packet = 64M#最大允许的包大小
[mysql]
default-character-set = utf8
connect-timeout = 3
[mysqld_safe]
open-files-limit  = 8192#可打开文件数量
```

### information_schema

information_schema数据库是MySQL自带的，它提供了访问数据库元数据的方式。什么是元数据呢？元数据是关于数据的数据，如数据库名或表名，列的数据类型，或访问权限等。有些时候用于表述该信息的其他术语包括“数据词典”和“系统目录”。

在MySQL中，把  information_schema  看作是一个数据库，确切说是信息数据库。其中保存着关于MySQL服务器所维护的所有其他数据库的信息。如数据库名，数据库的表，表栏的数据类型与访问权限等。在INFORMATION_SCHEMA中，有数个只读表。它们实际上是视图，而不是基本表，因此，你将无法看到与之相关的任何文件。

#### 字符集

- CHARACTER_SETS（字符集）表：提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表。

- COLLATIONS表：提供了关于各字符集的对照信息。

- COLLATION_CHARACTER_SET_APPLICABILITY表：指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。

下面我们说一下character sets和collations的区别：

字符集（character sets）存储字符串，是指人类语言中最小的表义符号。例如’A'、’B'等；排序规则（collations）规则比较字符串，collations是指在同一字符集内字符之间的比较规则

每个字符序唯一对应一种字符集，但一个字符集可以对应多种字符序，其中有一个是默认字符序(Default Collation)

MySQL中的字符序名称遵从命名惯例：以字符序对应的字符集名称开头；以_ci(表示大小写不敏感)、_cs(表示大小写敏感)或_bin(表示按编码值比较)结尾。例如：在字符序“utf8_general_ci”下，字符“a”和“A”是等价的

看一下有关于字符集和校对相关的MySQL变量：

- character_set_server：默认的内部操作字符集
- character_set_client：客户端来源数据使用的字符集
- character_set_connection：连接层字符集
- character_set_results：查询结果字符集
- character_set_database：当前选中数据库的默认字符集
- character_set_system：系统元数据(字段名等)字符集

再看一下MySQL中的字符集转换过程：

1. MySQL Server收到请求时将请求数据从character_set_client转换为character_set_connection；

2. 进行内部操作前将请求数据从character_set_connection转换为内部操作字符集，其确定方法如下：

   使用每个数据字段的CHARACTER SET设定值；
   若上述值不存在，则使用对应数据表的DEFAULT CHARACTER SET设定值(MySQL扩展，非SQL标准)；
   若上述值不存在，则使用对应数据库的DEFAULT CHARACTER SET设定值；
   若上述值不存在，则使用character_set_server设定值。

3. 将操作结果从内部操作字符集转换为character_set_results。

#### 权限

- USER_PRIVILEGES（用户权限）表：给出了关于全程权限的信息。该信息源自mysql.user授权表。是非标准表。

- SCHEMA_PRIVILEGES（方案权限）表：给出了关于方案（数据库）权限的信息。该信息来自mysql.db授权表。是非标准表。

- TABLE_PRIVILEGES（表权限）表：给出了关于表权限的信息。该信息源自mysql.tables_priv授权表。是非标准表。

- COLUMN_PRIVILEGES（列权限）表：给出了关于列权限的信息。该信息源自mysql.columns_priv授权表。是非标准表。

MySQL授权的层次，SCHEMA，TABLE，COLUMN级别，当然这些都是基于用户来授予的。可以看得到MySQL的授权也是相当的细密的，可以具体到列，这在某一些应用场景下还是很有用的，比如审计等。

#### 存储数据库系统的实体对象

- COLUMNS：存储表的字段信息，所有的存储引擎
- INNODB_SYS_COLUMNS ：存放的是INNODB的元数据， 他是依赖于SYS_COLUMNS这个统计表而存在的。
- ENGINES ：引擎类型，是否支持这个引擎，描述，是否支持事物，是否支持分布式事务，是否能够支持事物的回滚点
- EVENTS ：记录MySQL中的事件，类似于定时作业
- FILES ：这张表提供了有关在MySQL的表空间中的数据存储的文件的信息，文件存储的位置，这个表的数据是从InnoDB in-memory中拉取出来的，所以说这张表本身也是一个内存表，每次重启重新进行拉取。也就是我们下面要说的INNODB_SYS_DATAFILES这张表。还要注意一点的是这张表包含有临时表的信息，所以说和SYS_DATAFILES 这张表是不能够对等的，还是要从INNODB_SYS_DATAFILES看。如果undo表空间也配置是InnoDB 的话，那么也是会被记录下来的。
- PARAMETERS ：参数表存储了一些存储过程和方法的参数，以及存储过程的返回值信息。存储和方法在ROUTINES里面存储。
- PLUGINS ：基本上是MySQL的插件信息，是否是活动状态等信息。其实SHOW PLUGINS本身就是通过这张表来拉取道德数据
- ROUTINES：关于存储过程和方法function的一些信息，不过这个信息是不包括用户自定义的，只是系统的一些信息。
- SCHEMATA：这个表提供了实例下有多少个数据库，而且还有数据库默认的字符集
- TRIGGERS :这个表记录的就是触发器的信息，包括所有的相关的信息。系统的和自己用户创建的触发器。
- VIEWS :视图的信息，也是系统的和用户的基本视图信息。

这些表存储的都是一些数据库的实体对象，方便我们进行查询和管理，对于一个DBA来说，这些表能够大大方便我们的工作，更快更方便的了结和查询数据库的相关信息。

#### 约束外键

- REFERENTIAL_CONSTRAINTS：这个表提供的外键相关的信息，而且只提供外键相关信息
- TABLE_CONSTRAINTS ：这个表提供的是 相关的约束信息
- INNODB_SYS_FOREIGN_COLS ：这个表也是存储的INNODB关于外键的元数据信息和SYS_FOREIGN_COLS 存储的信息是一致的
- INNODB_SYS_FOREIGN ：存储的INNODB关于外键的元数据信息和SYS_FOREIGN_COLS 存储的信息是一致的，只不过是单独对于INNODB来说的
- KEY_COLUMN_USAGE：数据库中所有有约束的列都会存下下来，也会记录下约束的名字和类别

为什么要把外键和约束单列出来呢，因为感觉这是一块独立的东西，虽然我们的生产环境大部分都不会使用外键，因为这会降低性能，但是合理的利用约束还是一个不错的选择，比如唯一约束。

#### 管理

- GLOBAL_STATUS ，GLOBAL_VARIABLES，SESSION_STATUS，SESSION_VARIABLES：这四张表分别记录了系统的变量，状态（全局和会话的信息），作为DBA相信大家也都比较熟悉了，而且这几张表也是在系统重启的时候回重新加载的。也就是内存表。
- PARTITIONS ：MySQL分区表相关的信息，通过这张表我们可以查询到分区的相关信息（数据库中已分区的表，以及分区表的分区和每个分区的数据信息）
- PROCESSLIST：show processlist其实就是从这个表拉取数据，PROCESSLIST的数据是他的基础。由于是一个内存表，所以我们相当于在内存中查询一样，这些操作都是很快的。
- INNODB_CMP_PER_INDEX，INNODB_CMP_PER_INDEX_RESET：这两个表存储的是关于压缩INNODB信息表的时候的相关信息,有关整个表和索引信息都有.我们知道对于一个INNODB压缩表来说,不管是数据还是二级索引都是会被压缩的,因为数据本身也可以看作是一个聚集索引。
- INNODB_CMPMEM ，INNODB_CMPMEM_RESET：这两个表是存放关于MySQL INNODB的压缩页的buffer pool信息，但是要注意一点的就是,用这两个表来收集所有信息的表的时候,是会对性能造成严重的影响的,所以说默认是关闭状态的。如果要打开这个功能的话我们要设置innodb_cmp_per_index_enabled参数为ON状态。
- INNODB_BUFFER_POOL_STATS ：表提供有关INNODB 的buffer pool相关信息，和show engine innodb status提供的信息是相同的。也是show engine innodb status的信息来源。
- INNODB_BUFFER_PAGE_LRU，INNODB_BUFFER_PAGE :维护了INNODB LRU LIST的相关信息
- INNODB_BUFFER_PAGE ：这个表就比较屌了，存的是buffer里面缓冲的页数据。查询这个表会对性能产生很严重的影响，千万不要再我们自己的生产库上面执行这个语句，除非你能接受服务短暂的停顿
- INNODB_SYS_DATAFILES ：这张表就是记录的表的文件存储的位置和表空间的一个对应关系(INNODB)
- INNODB_TEMP_TABLE_INFO ：这个表惠记录所有的INNODB的所有用户使用到的信息，但是只能记录在内存中和没有持久化的信息。
- INNODB_METRICS ：提供INNODB的各种的性能指数，是对INFORMATION_SCHEMA的补充，收集的是MySQL的系统统计信息。这些统计信息都是可以手动配置打开还是关闭的。有以下参数都是可以控制的：innodb_monitor_enable, innodb_monitor_disable, innodb_monitor_reset, innodb_monitor_reset_all。
- INNODB_SYS_VIRTUAL :表存储的是INNODB表的虚拟列的信息，当然这个还是比较简单的，在MySQL 5.7中，支持两种Generated Column，即Virtual Generated Column和Stored Generated Column，前者只将Generated Column保存在数据字典中（表的元数据），并不会将这一列数据持久化到磁盘上；后者会将Generated Column持久化到磁盘上，而不是每次读取的时候计算所得。很明显，后者存放了可以通过已有数据计算而得的数据，需要更多的磁盘空间，与实际存储一列数据相比并没有优势，因此，MySQL 5.7中，不指定Generated Column的类型，默认是Virtual Column。
- INNODB_CMP，INNODB_CMP_RESET：存储的是关于压缩INNODB信息表的时候的相关信息

为什么把这些表列为管理相关的表呢，因为我感觉像连接，分区，压缩表，innodb buffer pool等表，我们通过这些表都能很清晰的看到自己数据库的相关功能的状态，特别是我们通过一些变量更容易窥透MySQL的运行状态，方便我们进行管理。

#### 表信息和索引信息

TABLES，TABLESPACES，INNODB_SYS_TABLES ，INNODB_SYS_TABLESPACES ：

- TABLES这张表毫无疑问了，就是记录的数据库中表的信息，其中包括系统数据库和用户创建的数据库。show table status like 'test1'\G的来源就是这个表；
- TABLESPACES 却是标注的活跃表空间。 这个表是不提供关于innodb的表空间信息的，对于我们来说并没有太大作用，因为我们生产库是强制INNODB的；
- INNODB_SYS_TABLES 这张表依赖的是SYS_TABLES数据字典中拉取出来的。此表提供了有关表格的格式和存储特性，包括行格式，压缩页面大小位级别的信息（如适用）
  提供的是关于INNODB的表空间信息，其实和SYS_TABLESPACES 中的INNODB信息是一致的。
- STATISTICS：这个表提供的是关于表的索引信息，所有索引的相关信息。
- INNODB_SYS_INDEXES：提供相关INNODB表的索引的相关信息，和SYS_INDEXES 这个表存储的信息基本是一样的，只不过后者提供的是所有存储引擎的索引信息，后者只提供INNODB表的索引信息。
- INNODB_SYS_TABLESTATS：
  这个表就比较重要了，记录的是MySQL的INNODB表信息以及MySQL优化器会预估SQL选择合适的索引信息，其实就是MySQL数据库的统计信息
  这个表的记录是记录在内存当中的，是一个内存表，每次重启后就会重新记录，所以只能记录从上次重启后的数据库统计信息。有了这个表，我们对于索引的维护就更加方便了，我们可以查询索引的使用次数，方便清理删除不常用的索引，提高表的更新插入等效率，节省磁盘空间。
- INNODB_SYS_FIELDS ：这个表记录的是INNODB的表索引字段信息，以及字段的排名
- INNODB_FT_CONFIG :这张表存的是全文索引的信息
- INNODB_FT_DEFAULT_STOPWORD：这个表存放的是stopword 的信息,是和全文索引匹配起来使用的，和innodb的 INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD 是相同的，这个STOPWORD必须是在创建索引之前创建，而且必须指定字段为varchar。stopword 也就是我们所说的停止词，全文检索时，停止词列表将会被读取和检索，在不同的字符集和排序方式下，会造成命中失败或者找不到此数据，这取决于停止词的不同的排序方式。我们可以使用这个功能筛选不必要字段。
- INNODB_FT_INDEX_TABLE：这个表存储的是关于INNODB表有全文索引的索引使用信息的，同样这个表也是要设置innodb_ft_aux_table以后才能够使用的，一般情况下是空的
- INNODB_FT_INDEX_CACHE ：这张表存放的是插入前的记录信息，也是为了避免DML时候昂贵的索引重组

#### MySQL优化

- OPTIMIZER_TRACE ：提供的是优化跟踪功能产生的信息.
- PROFILING：SHOW PROFILE可以深入的查看服务器执行语句的工作情况。以及也能帮助你理解执行语句消耗时间的情况。一些限制是它没有实现的功能，不能查看和剖析其他连接的语句，以及剖析时所引起的消耗。
- SHOW PROFILES显示最近发给服务器的多条语句，条数根据会话变量profiling_history_size定义，默认是15，最大值为100。设为0等价于关闭分析功能。详细信息请见MySQL profile
- INNODB_FT_BEING_DELETED,INNODB_FT_DELETED: INNODB_FT_BEING_DELETED 这张表是
- INNODB_FT_DELETED的一个快照,只在OPTIMIZE TABLE 的时候才会使用。

#### MySQL事物和锁

- INNODB_LOCKS:现在获取的锁，但是不含没有获取的锁，而且只是针对INNODB的。
- INNODB_LOCK_WAITS：系统锁等待相关信息，包含了阻塞的一行或者多行的记录，而且还有锁请求和被阻塞改请求的锁信息等。
- INNODB_TRX：包含了所有正在执行的的事物相关信息（INNODB），而且包含了事物是否被阻塞或者请求锁。

我们通过这些表就能够很方便的查询出来未结束的事物和被阻塞的进程，这是不是更方便了

