---
title: Hexo博客（CentOS）-无秘部署
date: 2019-03-16 14:50:22
tags:
 - 博客
 - Hexo
categories:
 - Hexo博客  
---

### 前言

开发环境：Ubuntu 18.04，网络正常情况下 （想使用Windows或者CentOS 7搭建的同学也可以参考）
前提：电脑装有Git和NodeJS
使用工具：NodeJS、Hexo

*本系列文章主旨:适合自己的才是最好的*

我们进行了之前的教程，现在发现每次部署都需要进行输入用户名、密码，如果有多个部署点，就需要输入多个，这样就变得很麻烦。幸运的是，我们一般可以使用SSH Key来使使用Git的站点进行无秘登录。

<!--more-->

### Git安装

```(shell)
//CentOS
sudo yum install git
//或
//Ubuntu
sudo apt install git
```

[git的使用](http://iissnan.com/progit/)

用于后续将博客上传到github上，（注：使用git前需要[配置](http://iissnan.com/progit/html/zh/ch1_5.html)）。

更换为上传的用户,当然不更换也行。

```
$ su - bloger
```


基本配置：

```(shell)
$ git config --global user.name "pingxin"
$ git config --global user.email "m13839441583@163.com"
```

###  生成秘钥

使用`ssh-keygen`生成公钥和私钥。

```(shell)
$ ssh-keygen -t rsa -C "your email"
```

### 修改配置文件

我们打开`_config.yml`,如果deploy下type为git的repo为https格式的话，需要改为ssh格式。如下

*HTTPS的格式为：https://github.com/用户名/仓库名.git*

*SSH的格式为：git@github.com:用户名/仓库名.git*

```(yaml)
deploy:
 - type: git
   repo: https://github.com/hanyunpeng0521/hanyunpeng0521.github.io.git
   branch: master
 - type: git
   repo: https://git.dev.tencent.com/pingxin0521/pingxin0521.coding.me.git
   branch: master
```

更改为：

```(yaml)
deploy:
 - type: git
   repo: git@github.com:hanyunpeng0521/hanyunpeng0521.github.io.git
   branch: master
 - type: git
   repo: git@git.dev.tencent.com:pingxin0521/pingxin0521.coding.me.git
   branch: master
```

### 配置站点

#### 配置github

复制.ssh/id_rsa.pub里面的字符串，登录github，选择Setting--》SSH and GPG keys--》New SSH Key，然后随便输入title，把复制的字符串粘贴到Key里面。

接着进行测试是否可以使用。

```(shell)
$ ssh -T git@github.com
```

#### 配置Coding

复制.ssh/id_rsa.pub里面的字符串，登录coding，选择个人设置--》SSH 公钥--》新增公钥，然后随便输入title，把复制的字符串粘贴到Key里面，有效期默认即可。

接着进行测试是否可以使用。

```(shell)
$ ssh -T git@git.coding.net
```

### 部署测试

现在测试是否可以进行无秘部署。

首先清除生成文件。

```(shell)
$ hexo clean
INFO  Deleted database.
INFO  Deleted public folder.
```

然后生成部署文件。

```(shell)
$ hexo g
```

然后部署。

```(shell)
$ hexo d
```

可以发现部署到github和coding是没有要求输入用户名和密码（第一次上传可能要求确认）。
