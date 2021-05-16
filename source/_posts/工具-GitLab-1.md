---
title: GitLab 搭建git服务
date: 2019-08-06 11:18:59
tags:
 - 工具
 - GitLab
categories:
 - 工具
 - GitLab
---

GitLab：是一个基于Git实现的在线代码仓库托管软件，你可以用gitlab自己搭建一个类似于Github一样的系统，一般用于在企业、学校等内部网络搭建git私服。

功能：Gitlab 是一个提供代码托管、提交审核和问题跟踪的代码管理平台。对于软件工程质量管理非常重要。

版本：GitLab 分为社区版（CE） 和企业版（EE）。

配置：建议CPU2核，内存2G以上。
<!--more-->

**Gitlab的服务构成：**

- Nginx：静态web服务器。

- gitlab-shell：用于处理Git命令和修改authorized keys列表。（Ruby）

- gitlab-workhorse: 轻量级的反向代理服务器。（go）

  GitLab Workhorse是一个敏捷的反向代理。它会处理一些大的HTTP请求，比如文件上传、文件下载、Git push/pull和Git包下载。其它请求会反向代理到GitLab Rails应用，即反向代理给后端的unicorn。

- logrotate：日志文件管理工具。

- postgresql：数据库。

- redis：缓存数据库。

- sidekiq：用于在后台执行队列任务（异步执行）。（Ruby）

- unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。（Ruby Web Server,主要使用Ruby编写）

#### GitLab安装

注意: gitlab-ce 镜像仅支持 x86-64 架构

我的环境：ubuntu 18.04

[安装文档](https://about.gitlab.com/install/)

##### Debian/Ubuntu 用户

1. 安装和配置必要的依赖项

   ```bash
   $ sudo apt-get update
   $ sudo apt-get install -y curl openssh-server ca-certificates
   ```

   接下来，安装Postfix发送通知电子邮件。如果您想使用其他解决方案发送电子邮件，请跳过此步骤，并在GitLab安装后配置一个外部SMTP服务器。

   ```
   $ sudo apt-get install -y postfix
   ```

   在Postfix安装期间，可能会出现配置屏幕。选择“Internet Site”并按回车键。使用服务器的外部DNS作为“邮件名称”，然后按enter。如果出现其他屏幕，则继续按enter接受默认值。

2. 添加GitLab包存储库并安装包

   导入公钥：

   ```bash
   $ curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
   ```

   添加GitLab包存储库。

   ```
   $ sudo vim /etc/apt/sources.list.d/gitlab_gitlab-ee.list
   #添加
   deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu bionic main
   ```

   更改 `https://gitlab.example.com`换成你想要访问GitLab实例的URL，安装将自动配置和启动GitLab在该URL。

   https：//GitLab将自动证书请求的url与我们进行加密，它需要入站HTTP访问和一个有效的主机名。您也可以使用您自己的证书或只是使用http：//。

   ```
   $ sudo apt-get update
   $ sudo EXTERNAL_URL="http://127.0.0.1" apt-get install gitlab-ce
   ```

##### RHEL/CentOS 用户

新建 `/etc/yum.repos.d/gitlab-ce.repo`，内容为

```
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```

再执行

```
sudo yum makecache
sudo yum install gitlab-ce
```

#### 配置

打开`/etc/gitlab/gitlab.rb`,将`external_url = 'http://git.example.com'`修改为自己的域名地址：`http://example.com`，默认为80端口，如要使用其他端口后面加上端口号，如：`http://127.0.0.1:8080`

然后执行：

```
sudo gitlab-ctl reconfigure
gitlab-ctl restart
```

启动完成后浏览器访问配置好的地址，应该出现重置管理员密码的界面。

**访问 GitLab页面**

如果没有域名，直接输入服务器ip和指定端口进行访问

初始账户: root 密码:5iveL!fe

第一次登录修改密码

这样就可以配置一个和github网址一样的网址了，而且你就是最强管理员，用有最高权限，可以用于公司或者团体内部进行使用。

当然使用的话会有更多配置，具体参考官方文档

#### Docker安装

查看[中文社区版github网址](https://github.com/twang2218/gitlab-ce-zh)

**使用 Docker Compose**

正常部署时，可以使用 Docker Compose 来配置启动。建立一个 `docker-compose.yml`，内容如下：

```
version: '2'
services:
    gitlab:
      image: 'twang2218/gitlab-ce-zh:11.1.4'
      restart: unless-stopped
      hostname: 'gitlab.example.com'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://gitlab.example.com'
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
          # 需要配置到 gitlab.rb 中的配置可以在这里配置，每个配置一行，注意缩进。
          # 比如下面的电子邮件的配置：
          # gitlab_rails['smtp_enable'] = true
          # gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
          # gitlab_rails['smtp_port'] = 465
          # gitlab_rails['smtp_user_name'] = "xxxx@xx.com"
          # gitlab_rails['smtp_password'] = "password"
          # gitlab_rails['smtp_authentication'] = "login"
          # gitlab_rails['smtp_enable_starttls_auto'] = true
          # gitlab_rails['smtp_tls'] = true
          # gitlab_rails['gitlab_email_from'] = 'xxxx@xx.com'
      ports:
        - '8084:8084'
        - '443:443'
        - '22:22'
      volumes:
        - config:/etc/gitlab
        - data:/var/opt/gitlab
        - logs:/var/log/gitlab
volumes:
    config:
    data:
    logs:
```

然后使用命令 `docker-compose up -d` 来启动，停止服务使用 `docker-compose down`。

**如果你的服务器有域名，将上面的 gitlab.example.com 替换为实际域名。**

实验时，也可以直接修改 `/etc/hosts` 方便测试。比如：

```
127.0.0.1   gitlab.example.com
```

**启动**

通常会将 GitLab 的配置 (etc) 、 日志 (log) 、数据 (data) 放到容器之外， 便于日后升级， 因此请先准备这三个目录:

```
docker volume create gitlab-config 
docker volume create gitlab-data 
docker volume create gitlab-logs 

```

准备好这三个目录之后， 就可以开始运行 Docker 镜像了。 我的建议是使用unless-stopped 作为重启策略， 
因为这样可以手工停止容器， 方便维护。

然后，我们需要创建自定义网络，从而让容器运行于独立的网络中，区别于默认网桥。

```
docker network create gitlab-net
```

准备好后，开始运行 Gitlab 容器,内部端口最好外部映射端口一样：

```
docker run -d \
    --hostname gitlab.example.com \
    -p 8084:8084 \
    -p 443:443 \
    -p 22:22 \
    --name gitlab \
    --restart unless-stopped \
    -v gitlab-config:/etc/gitlab \
    -v gitlab-logs:/var/log/gitlab \
    -v gitlab-data:/var/opt/gitlab \
    --network gitlab-net \
    twang2218/gitlab-ce-zh:11.1.4
```

如需卸载服务及相关内容，可以执行：

```
docker stop gitlab
docker rm gitlab
docker network rm gitlab-net
docker volume rm gitlab-config gitlab-datagitlab-logs
```

运行后访问127.0.0.1:8084，设置root用户的初始密码，修改后登录成功

##### 配置

- 修改external_url

  ```
$ docker exec -it gitlab bash
  $ vim /etc/gitlab/gitlab.rb
  external_url 'http://gitlab.example.com:8084'
  ```
  
  

#### 端口被占用

如果端口号被占用，改了默认的80端口，后面要接端口号

```
external_url 'http://192.168.182.129：8090' 注：这里容器内没改，只是映射到8090`
```

修改NGINX监听的端口为9999

```
nginx['listen_addresses'] = ['*']nginx['listen_port'] = 8090
```

如果8080端口被Tomcat占用，会出现502的页面

因此需要修改unicorn的配置，如下

```
### Advanced settings
# unicorn['listen'] = '127.0.0.1'
# unicorn['port'] = 8080
 
# 假设有Tomcat占用了8080，因此改为8082试一试
 unicorn['listen'] = '127.0.0.1'
 unicorn['port'] = 8082
```

#### 修改Gitlab数据存储路径

默认的Gitlab数据存储路径，在目录/var/opt/gitlab/git-data下，防止以后数据过大，所以可以修改路径存储为/data/gitlabData

```
vim /mnt/gitlab/etc/gitlab.rb
### For setting up different data storing directory
###! Docs: https://docs.gitlab.com/omnibus/settings/configuration.html#storing-git-data-in-an-alternative-directory
###! **If you want to use a single non-default directory to store git data use a
###!   path that doesn't contain symlinks.**
# git_data_dirs({ "default" => { "path" => "/var/opt/gitlab/git-data", 'gitaly_address' => 'unix:/var/opt/gitlab/gitaly/gitaly.socket' } })

#edited by ouyangpeng 2017-8-10  配置gitlab的数据存储位置为/data目录下，保证硬盘安全
git_data_dirs({ "default" => { "path" => "/data/gitlabData" } })
```

设置完后，过一段使用时间，可以看到该目录下的resposities

#### 配置并启动GitLab

像上面步骤修改了GitLab的ip地址一样,临时修改了GitLab的配置之后，得执行如下的命令，应用重新配好的配置并重启GitLab,然后查看GitLab的状态。

因为是容器，所以要进入到gitlab容器中执行命令

```
docker exec -ti gitlab /bin/bash
```

然后

```
gitlab-ctl reconfigure  #花时间比较多
gitlab-ctl restart    #改IP重启就可以了
gitlab-ctl status
```

#### 登陆

打开浏览器，输入本机的ip地址并登陆

```
http://192.168.182.129:8084
```

默认帐户的用户名是root，第一次访问时，将被重定向到密码重置屏幕,登录后，您可以更改用户名。

#### 常用的几个Gitlab命令

```
# 重新应用gitlab的配置
gitlab-ctl reconfigure
 
# 重启gitlab服务
gitlab-ctl restart
 
# 查看gitlab运行状态
gitlab-ctl status
 
#停止gitlab服务
gitlab-ctl stop
 
# 查看gitlab运行日志
gitlab-ctl tail
 
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sideki
```

#### 错误

1. docker启动报错信息如下

```
Error response from daemon: driver failed programming external connectivity on endpoint quirky_allen<strong>解决:</strong>
```

解决

检查docker端口映射是否冲突
重启docker服务后再启动容器

```
systemctl restart docker
docker start gitlab
```

#### 常用设置

1. 设置邮件服务器

   用途：

   - 有合并请求时，邮件通知
   - 账号注册时，邮件验证
   - 修改密码时，通过邮件修改

   1. 开启QQ邮箱的smtp服务(不建议使用163邮箱，发几次之后，就不能发送,当然公司最好使用自己的公司邮箱)
      设置–》账户–》smtp–》密保验证–》验证成功返回一串字符串保存返回的字符串

   2. 修改gitlab配置
      `vim /etc/gitlab/gitlab.rb`
      按/后输入smtp_enable，找到下面这一串文本，进行修改,记得去掉注释

      ```
      gitlab_rails['gitlab_email_enabled'] = true
      gitlab_rails['gitlab_email_from'] = '123456@qq.com'
      gitlab_rails['gitlab_email_display_name'] = 'pingxin'
      gitlab_rails['gitlab_email_reply_to'] = '123456@qq.com'
      gitlab_rails['gitlab_email_subject_suffix'] = ''
      
      
      gitlab_rails['smtp_enable'] = true
      gitlab_rails['smtp_address'] = "smtp.qq.com"
      gitlab_rails['smtp_port'] = 465
      gitlab_rails['smtp_user_name'] = "123456@qq.com"
      gitlab_rails['smtp_password'] = "开通smtp时返回的字符"
      gitlab_rails['smtp_domain'] = "qq.com"
      gitlab_rails['smtp_authentication'] = "login"
      gitlab_rails['smtp_enable_starttls_auto'] = true
      gitlab_rails['smtp_tls'] = true
      ```

      按/后输入git_user_email，找到下面这一文本，进行修改

      ```
      	user['git_user_email'] = "123456@qq.com"
      	gitlab_rails['gitlab_email_from'] = '123456@qq.com'
      ```

      按esc退出到命令行模式
      之后:wq 保存并退出

      重启配置

      ```
      gitlab-ctl reconfigure
      ```

   3. 测试邮件服务是否正常

      ```
      gitlab-rails console
      ```

      Notify.test_email(‘接收方邮件地址’,‘邮件标题’,‘邮件内容’).deliver_now

      ```
      Notify.test_email('xxx@163.com','hello','hello').deliver_now
      ```

      按回车，测试发送。

2. 设置用户

   管理区域--》新建用户--》填写姓名、用户名、电子邮箱等信息--》用户会收到邮件，设置密码

   ![YQxBjJ.png](https://s1.ax1x.com/2020/05/09/YQxBjJ.png)

3. 设置组

   管理员登陆gitlab服务器--》 点击管理区域—Group-Add group--》依次填入组路径，名称和描述等信息--》选择公开、私有、内部（一般私有）--》创建

   这里就是创建一个mall_dev群组,然后就可以添加项目或者子群组，也可以选择通知类型。

   ![YQWenH.png](https://s1.ax1x.com/2020/05/09/YQWenH.png)

   添加用户--》成员

4. 用户权限

   使用管理员打开要设置权限的项目--》点击【Settings】--【Members】--》添加完成

   下表完整的列出了Guest,Reporter,Developer,Master,Owner对应的权限。

   |                                       | Guest | Reporter | Developer | Master | Owner |
   | ------------------------------------- | ----- | -------- | --------- | ------ | ----- |
   | Create new issues                     | *     | *        | *         | *      | *     |
   | Leave comments                        | *     | *        | *         | *      | *     |
   | Pull the project code                 |       | *        | *         | *      | *     |
   | Download a project                    |       | *        | *         | *      | *     |
   | Create code snippets                  |       | *        | *         | *      | *     |
   | Create new merge requests             |       |          | *         | *      | *     |
   | Push changes to nonprotected branches |       |          | *         | *      | *     |
   | Remove nonprotected branches          |       |          | *         | *      | *     |
   | Add tags                              |       |          | *         | *      | *     |
   | Write a wiki                          |       |          | *         | *      | *     |
   | Manage the issue tracker              |       |          | *         | *      | *     |
   | Add new team members                  |       |          |           | *      | *     |
   | Push changes to protected branches    |       |          |           | *      | *     |
   | Manage the branch protection          |       |          |           | *      | *     |
   | Manage Git tags                       |       |          |           | *      | *     |
   | Edit the project                      |       |          |           | *      | *     |
   | Add deploy keys to the project        |       |          |           | *      | *     |
   | Configure the project hooks           |       |          |           | *      | *     |

#### 备份

##### Gitlab的备份目录路径设置

```shell
[root@code-server ~]# vim /etc/gitlab/gitlab.rb
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/data/gitlab/backups"    //gitlab备份目录
gitlab_rails['backup_archive_permissions'] = 0644       //生成的备份文件权限
gitlab_rails['backup_keep_time'] = 7776000              //备份保留天数为3个月（即90天，这里是7776000秒）
 
[root@code-server ~]# mkdir -p /data/gitlab/backups
[root@code-server ~]# chown -R git.git /data/gitlab/backups
[root@code-server ~]# chmod -R 777 /data/gitlab/backups
  
#如上设置了gitlab备份目录路径为/data/gitlab/backups，最后使用下面命令重载gitlab配置文件，是上述修改生效！
root@code-server ~]# gitlab-ctl reconfigure
```

Gitlab备份操作（使用备份命令"gitlab-rake gitlab:backup:create"）

```shell
#手动备份gitlab
[root@code-server backups]# gitlab-rake gitlab:backup:create
然后查看下备份文件（文件权限是设定好的644）
[root@code-server backups]# ll
total 244
-rw-r--r-- 1 git git 245760 Nov 12 15:33 1510472027_2017_11_12_9.4.5_gitlab_backup.tar
 
#编写备份脚本，结合crontab实施自动定时备份，比如每天0点、6点、12点、18点各备份一次
[root@code-server backups]# pwd
/data/gitlab/backups
[root@code-server backups]# vim gitlab_backup.sh
#!/bin/bash
/usr/bin/gitlab-rake gitlab:backup:create CRON=1
 
#注意：环境变量CRON=1的作用是如果没有任何错误发生时， 抑制备份脚本的所有进度输出
 
[root@code-server backups]# crontab -l
0 0,6,12,18 * * * /bin/bash -x /data/gitlab/backups/gitlab_backup.sh > /dev/null 2>&1
```

##### Gitlab恢复操作

```shell
#GItlab只能还原到与备份文件相同的gitlab版本。
#假设在上面gitlab备份之前创建了test项目，然后不小心误删了test项目，现在就进行gitlab恢复操作：
  
#1）停止相关数据连接服务
[root@code-server backups]# gitlab-ctl stop unicorn
ok: down: unicorn: 0s, normally up
[root@code-server backups]# gitlab-ctl stop sidekiq
ok: down: sidekiq: 1s, normally up
[root@code-server backups]# gitlab-ctl status

# 2）现在通过之前的备份文件进行恢复（必须要备份文件放到备份路径下，这里备份路径我自定义的/data/gitlab/backups，默认的是/var/opt/gitlab/backups）
[root@code-server backups]# pwd
/data/gitlab/backups
[root@code-server backups]# ll
total 244
-rw-r--r-- 1 git git 245760 Nov 12 15:33 1510472027_2017_11_12_9.4.5_gitlab_backup.tar
  
#Gitlab的恢复操作会先将当前所有的数据清空，然后再根据备份数据进行恢复
[root@code-server backups]# gitlab-rake gitlab:backup:restore BACKUP=1510472027_2017_11_12_9.4.5
#最后再次启动Gitlab
[root@code-server backups]# gitlab-ctl start
#恢复命令完成后，可以check检查一下恢复情况
[root@code-server backups]# gitlab-rake gitlab:check SANITIZE=true
```

#### 实现HTTPS服务

1. 下面假设你已有一个域名 gitlab.example.com，并且已经获得了该域名对应的证书：gitlab.example.com.key、gitlab.example.com.crt 。

2. 服务器的 "/opt/gitlab/conf" 目录（对应的是容器里的"/etc/gitlab"目录）下创建一个 ssl目录

   ```
   sudo mkdir -p /opt/gitlab/config/ssl
   ```

3. 将gitlab.example.com.key、gitlab.example.com.crt 拷贝到服务器的 "/srv/gitlab/config/ssl"目录中

4. 进入gitlab容器并打开配置文件

```
sudo docker exec -it gitlab vi /etc/gitlab/gitlab.rb
```

5. 将文件中的相关行修改如下：

```
external_url "https://gitlab.example.com"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
```

6. 重启容器

```
sudo docker restart gitlab
```

7. 访问 https://gitlab.example.com，就可以进入到相应的GitLab私有仓库页面了。

#### gitlab 实现 CI/CD

GitLab-CI就是一套配合GitLab使用的持续集成系统（当然，还有其它的持续集成系统，同样可以配合GitLab使用，比如Jenkins）。而且GitLab8.0以后的版本是默认集成了GitLab-CI并且默认启用的。

GitLab-Runner是配合GitLab-CI进行使用的。一般地，GitLab里面的每一个工程都会定义一个属于这个工程的软件集成脚本，用来自动化地完成一些软件集成工作。当这个工程的仓库代码发生变动时，比如有人push了代码，GitLab就会将这个变动通知GitLab-CI。这时GitLab-CI会找出与这个工程相关联的Runner，并通知这些Runner把代码更新到本地并执行预定义好的执行脚本。

所以，GitLab-Runner就是一个用来执行软件集成脚本的东西。你可以想象一下：Runner就像一个个的工人，而GitLab-CI就是这些工人的一个管理中心，所有工人都要在GitLab-CI里面登记注册，并且表明自己是为哪个工程服务的。当相应的工程发生变化时，GitLab-CI就会通知相应的工人执行软件集成脚本。如下图所示：

![YleTNF.png](https://s1.ax1x.com/2020/05/09/YleTNF.png)

Runner可以分布在不同的主机上，同一个主机上也可以有多个Runner。

**Runner类型**

GitLab-Runner可以分类两种类型：Shared Runner（共享型）和Specific Runner（指定型）。

- Shared Runner：这种Runner（工人）是所有工程都能够用的。只有系统管理员能够创建Shared Runner。

- Specific Runner：这种Runner（工人）只能为指定的工程服务。拥有该工程访问权限的人都能够为该工程创建Shared Runner。


安装：

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
yum install gitlab-ci-multi-runner
```



#### 参考

1. [gitlab的搭建（搭建、使用、备份、迁移恢复）](https://blog.51cto.com/11883699/2162900)

2. [手把手教你 GitLab 的安装及使用](https://www.jianshu.com/p/b04356e014fa)

3. [Gitlab备份和恢复操作记录](https://www.cnblogs.com/kevingrace/p/7821529.html)

4. [Docker Compose实例之nginx反向代理GitLab](https://www.centos.bz/2019/01/docker-compose实例之nginx反向代理gitlab/)

   