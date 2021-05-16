---
title: Ubuntu 点滴笔记
date: 2019-05-22 11:55:22
tags:
 - Linux
 - Ubuntu
category:
 - Linux
 - Ubuntu
---

#### Linux 设置默认编辑器

visudo 等操作会打开默认编辑器，在linux中默认编辑器读取EDITOR环境变量，可通过一下命令设置

<!--more-->

```
export EDITOR=nano
```

可将其加入~/.bashrc文件，使得每次登录都可使用

```
nano ~/.bashrc

export EDITOR=nano

. ~/.bashrc
```

debian系统提供了一个管理工具来设置默认编辑器

```
sudo update-alternatives --config editor 
```