---
title: Docker 卷数据备份和恢复
date: 2019-08-11 10:18:59
tags:
 - 容器
 - Docker
categories:
 - 容器
 - Docker
---

数据备份一直是运维中最最重要的一件事，Docker中的卷数据备份也同等重要！下面就来研究下Docker中的卷数据备份。

<!--more-->

#### 备份

首先，我们运行一个mysql的数据库容器：

```shell
$ docker run -itd --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -v mysql_data:/var/lib/mysql mysql

```

下面进入mysql，由于容器启动时作了端口映射，所以宿主机安装了mysql客户端的话就可以直接连接mysql数据库了（当然了，也可以进入容器直接连接mysql操作）。

连接mysql数据库，创建一个docker_data的数据库，并在其中创建一个表table1。

```shell
$ docker exec -it  mysql mysql -u root -p

mysql> create database mysql_data;
mysql> use mysql_data
mysql> create table table1(name varchar(255));
mysql> insert into table1 values("hyp"),("pingxin");
```

我们在数据库中新建的库及表都存在

```shell
$ docker inspect mysql_data
[
    {
        "CreatedAt": "2019-11-15T14:26:37+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql_data/_data",
        "Name": "mysql_data",
        "Options": null,
        "Scope": "local"
    }
]
$ ls /var/lib/docker/volumes/mysql_data/_data/mysql_data
table1.ibd
```

下面开始备份mysql数据库数据：

```shell
$ docker run -it --rm --volumes-from mysql -v$(pwd):/backup alpine tar zcvf /backup/backup.tar.gz /var/lib/mysql
```

其中：

`--rm` 备份后自动删除容器

`--volumes-from` 加载上述mysql容器

`-v` 挂载当前目录到新容器的/backup目录下

`tar zcvf /backup/backup.tar.gz /var/lib/mysql` 在新容器内执行tar命令将mysql容器的/var/lib/mysql打包成backup.tar.gz放到当前目录下

当前目录下存在backup.tar.gz，且压缩包里存在新增的数据。

#### 恢复

```shell
$ docker run -it --rm --volumes-from mysql -v$(pwd):/backup alpine tar zxvf /backup/backup.tar.gz -C /
```

可见，docker_data数据库和table1表均恢复了。

**注意：执行恢复动作前，需要先将原来的mysql容器stop掉，不然可能会出现table1表数据报错**。

#### 迁移

创建新的容器mysql1

```
docker run -itd --name mysql1 -e MYSQL_ROOT_PASSWORD=123456 -p 3308:3306 -v mysql_data1:/var/lib/mysql mysql
```

将数据迁移到mysql1

```
docker run -it --rm --volumes-from mysql1-v $(pwd):/backup alpine tar zxvf /backup/backup.tar.gz -C /
```

新的mysql1容器也包含了备份包数据。

至此，Docker中卷数据的备份、恢复或迁移均完成。