---
title: Git设置SSH key
date: 2019-03-21 09:18:59
tags:
 - 工具
 - Git
categories:
 - 工具
 - 版本控制
 - Git
---

### 前言

*这篇是将本地机做客户机，设置免密获取或推送到服务器。*

通过了解，我们知道Git是分布式版本控制系统，而GitHub是一个代码托管网站，相当于分布式中的一个节点，帮助用户进行代码的托管，具有和本地Git仓库相同的功能。那么，它也需要用户在上传代码到仓库时，进行认证，普通的做法就是输入邮箱/用户名和密码，如果上传频率过多的话，每次都需要输入，显得的繁琐。因此，Git引入了SSH  Key和GPG key，也就是公钥体系的一种来支持无秘登录。这里我们就来针对Git来进行SSH key的配置。

<!--more-->

#### github设置SSH key

在注册好github账号后，选择新建一个用户

```shell
sudo useradd git -d /home/git -m -s /bin/bash
su root
su - git
```

创建SSH key。

```shell
[git@localhost ~]$ ssh-keygen -t rsa -C "your email" 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/git/.ssh/id_rsa):
Enter passphrase (empty for no passphrase): #输入github的账号密码
Enter same passphrase again:
Your identification has been saved in /home/git/.ssh/id_rsa.
Your public key has been saved in /home/git/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:8KZP6JrZlzbkooR+miDanUCD2VinoK06e4FI0rruPmQ m13839441583@163.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|... . .          |
|oX.o   o         |
|Oo*     S        |
|+E.o   +.        |
|*.o.. oo..       |
|*=.=.*.o*        |
|BB*oBoo+..       |
+----[SHA256]-----+
```

然后,复制.ssh/id_rsa.pub里面的字符串，登录github，选择Setting--》SSH and GPG keys--》New SSH Key，然后随便输入title，把复制的字符串粘贴到Key里面。

接着进行测试是否可以使用。

```shell
[git@localhost ~]$ ssh -T git@github.com
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
RSA key fingerprint is MD5:16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Hi hanyunpeng0521! You've successfully authenticated, but GitHub does not provide shell access.
```

我们来进一步测试：

```shell
[git@localhost ~]$ git clone git@github.com:hanyunpeng0521/utils.git
正克隆到 'utils'...
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 112 (delta 0), reused 1 (delta 0), pack-reused 109
接收对象中: 100% (112/112), 115.71 KiB | 28.00 KiB/s, 完成.
处理 delta 中: 100% (23/23), 完成.
检查连接... 完成。
```

> 注意：我们设置的是SSH key无秘登录,所以在clone是使用SSH的链接，这样才会使用SSH key。如果使用https，则需要输入用户名、密码，因为没有设置https的无秘登录。

如果使用的是https来clone到本地，则需要在本地更新origin为ssh格式。
HTTPS的格式为：https://github.com/用户名/仓库名.git
SSH的格式为：git@github.com:用户名/仓库名.git

```
git remote remove origin
git remote add origin git@github.com:用户名/仓库名.git
```

再push的时候实际上是采用的SSH方式推送的代码。（打开仓库下.git/config文件可以看到[remote "origin"]的url为ssh格式）。

接下来测试提交到远程仓库。

```shell
[git@localhost utils]$ git push
Everything up-to-date
```

这样github上的SSH Key就设置完成了。



#### 更新被拒绝，因为远程仓库包含您本地尚不存在的提交。这通常是因为另外 提示：一个仓库已向该引用进行了推送。再次推送前，您可能需要先整合远程变更

当linux系统下git发生如上向远程仓库push出错时，解决办法

1.首先强制使用push：`$ git push -u origin +master` 
 如果仍然发生如下错误

```
error: src refspec master​ does not match any.
error: 无法推送一些引用到 ‘git@gitlab.xxx:xxx.git’
```

2.需先同步远程仓库文件到本地，再提交一次即可 
`$git pull` 
信息如下：

```
warning: no common commits
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
展开对象中: 100% (3/3), 完成.
来自 gitlab.corp.anjuke.com:yuqing02/TestGit
* [新分支] master -> origin/master
当前分支没有跟踪信息。
请指定您要合并哪一个分支。
详见 git-pull(1)。
git pull
如果您想要为此分支创建跟踪信息，您可以执行：
git branch –set-upstream-to=origin/ master

```

3.再一次push到远程仓库

```
$ git push origin master
```

