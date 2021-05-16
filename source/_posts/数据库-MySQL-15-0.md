---
title: MySQL集群结构
date: 2019-10-15 13:18:59
tags:
 - 数据库
 - MySQL
categories:
 - 数据库
 - MySQL
---

在以前，数据库的集群配置一直很难，难点在于MySQL主从结构的高可用和读写分离。万幸的是，Galera/GR的出现，让整个集群的配置都极大程度地简化了。

<!--more-->

以下是一个简单的MySQL集群拓扑图：

![3Sxsc4.png](https://s2.ax1x.com/2020/02/16/3Sxsc4.png)

**1.MySQL中间件：对MySQL Server的读写操作进行路由(即读写分离)；分库分表(sharding)**

- (1).MySQL Router：MySQL官方提供的轻量级MySQL代理(路由)，只提供读写分离功能，前身为SQL Proxy。
- (2).ProxySQL：类似于MySQL Router，轻量级MySQL代理，提供读写分离功能，也支持一些sharding功能。有percona版和官方版两个版本。
- (3).MaxScale：MariaDB的中间件，和MySQL Router、ProxySQL类似。
  - 这三者类似，都是轻量级数据库中间件。
- (4).Amoeba、Cobar、MyCAT：提供很多功能，最主要的功能包括读写分离、sharding。
  - 这三者的渊源较深，都是开源的。Amoeba后继无人，于是Cobar出来，Cobar后继无人，加上2013年出现了一次较严重的问题，于是MyCAT站在Cobar的肩膀上出来了。

**2.MySQL主从复制的高可用：至少要实现主从切换或故障时选举新master节点**

- (1).MMM：淘汰了，在一致性和高并发稳定性等方面有些问题。
- (2).MHA：有些人还在用，但也有些问题，也是趋于淘汰的MySQL主从高可用方案。
- (3).Galera：引领时代的主从复制高可用技术。
- (4).MariaDB Galera Cluster：MariaDB对Galera的实现。
- (5).PXC：Percona XtraDB Cluster，是Percona对Galera的自我实现，用的人很多。
- (6).GR：Group Replication，MySQL官方提供的组复制技术(MySQL 5.7.17引入的技术)，基于Paxos算法。
  - MariaDB Galera Cluster、PXC、GR是类似的，都各有优点。但GR是革命性的，基于原生复制技术，据传很多方面都优于PXC。
  - MariaDB Galera Cluster、PXC、GR为了安全性和性能考虑，做出了很多强制性的限制。例如基于GTID复制、只能InnoDB表，每表都必须有主键等。要使用它们提供主从复制的高可用，必须要了解它们的各项限制。