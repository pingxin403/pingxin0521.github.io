---
title: Linux-邮件系统
date: 2019-07-25 21:55:22
tags: 
 - Linux
 - 运维
category: 
 - Linux
 - 运维
---

### 前言

**MUA (Mail User Agent)** 
MUA 既是"邮件使用者代理人"，因为除非你可以直接利用类似 telnet 之类的软件登入邮件主机来主动发出信件，否则您就得要透过 MUA 来帮你送信到邮件主机上头去。 最常见的 MUA 像是 Mozilla 推出的Thunderbird ( 雷鸟 ) 自由软件， 或者是 Linux 桌面 KDE 常见的 Kmail ，及Windows 内件的 Outlook Express (OE) 等 。MUA 主要的功能就是收受邮件主机的电子邮件，以及提供用户浏览与编写邮件的功能！

<!--more-->

**MTA (Mail Transfer Agent)**
MUA 帮用户传送邮件到邮件主机上，那这部邮件主机如果能够帮用户将这封信寄出去， 那它就是一部邮件传送主机 (MTA) 啦！这个 MTA 就是『邮件传送代理人』的意思。也来顾名思义一下，既然是『传送 代理人』， 那么使用者寄出的信，与使用者要收信时，就是找它 (MTA) 就对啦！基本上， MTA 的功能有这些：
1）收受信件：使用简单邮件传送协议 (SMTP)
MTA 主机最主要的功能就是将来自客户端或者是其它 MTA 的来信收下来，这个时候 MTA 使用的是 Simple Mail Transfer Protocol (SMTP) ，它使用的是25端口。
2） 转递信件
如果该封信件的目的地并不是本身用户，且该封信的相关数据符合使用 MTA 的权力， 那么MTA 就会将该封信再传送到下一部主机上。这即是所谓的转递 (Relay) 的功能。
3）响应使用者的收信要求
POP 或 IMAP 协定用户可以透过 MTA 主机提供的邮政服务协议 (Post Office Protocol, POP) 来收下自己的信件， 也可以透过IMAP (Internet Message Access Protocol) 协议将自己的信件保留在邮件主机上面， 并进一步建立邮件数据匣等进阶工作。

总之，一般提到的 Mail Server 就是 MTA ！而严格来说， MTA 其实仅是指 SMTP 这个协议而已。 而达成 MTA的 SMTP 功能的主要套件包括老牌的 sendmail ，后起之秀的 postfix ，还有qmail等等。

**MDA (Mail Delivery Agent)** 
字面上的意思是『邮件递送代理人』的意思。事实上，这个 MDA 是挂在 MTA 底下的一个小程序， 最主要的功能就是： 分析由 MTA 所收到的信件表头或内容等数据， 来决定这封邮件的去向。 所以说，上面提到的MTA 的信件转递功能，其实是由 MDA 达成的。 举例来说，如果 MTA 所收到的这封信目标是自己，那么MDA 会将这封信给它转到使用者的信箱 (Mailbox) 去， 如果不是呢？那就准备要转递出去了。此外， MDA 还有分析与过滤邮件等功能喔！如：过滤垃圾邮件，自动回复，自动转发等……。

各主要的 MTA 程序 (sendmail,postfix...) 都有自己的 MDA 功能，不过有些外挂的程序功能更强大， 举例来说 procmail就是一个过滤的好帮手，另外 Mailscanner + Spamassassion 也是可以使用的一些 MDA 喔。

**Mailbox** 
就是电子邮件信箱！简单的说，就是某个账号专用的信件收受档案。我们的 Linux 系统默认的信箱都是放在 /var/spool/mail/ 使用者账号 中！ 若 MTA 所收到的信件是本机的使用者， MDA 就会将信件送到该 mailbox 当中去！

**POP3**
(Post Office Protocol 3)即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的邮件,而POP3服务器则是遵循POP3协议的接收邮件服务器,用来接收电子邮件的。

**IMAP**
Interactive Mail Access Protocol(交互式邮件存取协议)是由美国华盛顿大学所研发的一种邮件获取协议。它的主要作用是邮件客户端（例如MS Outlook Express)可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。无论是POP3还是IMAP都是描述如何从邮箱取出邮件。

请注意：POP3/IMAP和SMTP可以组建在不同的服务器上，经常使用MUA的用户肯定记得软件的设置中经常将POP3/IMAP和SMTP进行分开设置。

#### SMTP、POP3、IPMAP三者说明

简单来说：SMTP是邮件发送协议；POP3和IMAP是邮件接收协议。其中：

1）SMTP
全称是"Simple Mail Transfer Protocol"，目标是向用户提供高效、可靠的邮件传输。它是一组用于由源地址到目的地址传送邮件的规则，
通过它来控制邮件的中转方式。SMTP协议属于TCP/IP 协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。

SMTP服务器就是遵循SMTP协议的发送邮件服务器。 SMTP认证，简单地说就是要求必须在提供了账户名和密码之后才可以登录SMTP 服务器，这就使得那些垃圾邮件的散播者无可乘之机。

增加SMTP认证的目的是为了使用户避免受到垃圾邮件的侵扰。

2）POP3
POP3是Post Office Protocol 3的简称，即邮局协议的第3个版本,它规定怎样将个人计算机连接到Internet的邮件服务器和下载电子邮件的电子协议。
它是因特网电子邮件的第一个离线协议标准,POP3允许用户从服务器上把邮件存储到本地主机（即自己的计算机）上,同时删除保存在邮件服务器上的
邮件，而POP3服务器则是遵循POP3协议的接收邮件服务器，用来接收电子邮件的

3）IMAP
IMAP全称是Internet Mail Access Protocol，即交互式邮件存取协议，它是跟POP3类似邮件访问标准协议之一。不同的是，开启了IMAP后，您在电子
邮件客户端收取的邮件仍然保留在服务器上，同时在客户端上的操作都会反馈到服务器上，如：删除邮件，标记已读等，服务器上的邮件也会做相应
的动作。所以无论从浏览器登录邮箱或者客户端软件登录邮箱，看到的邮件以及状态都是一致的。

#### POP3和IMAP的区别

POP3协议允许电子邮件客户端下载服务器上的邮件，但是在客户端的操作（如移动邮件、标记已读等），不会反馈到服务器上，比如通过客户端收取了
邮箱中的3封邮件并移动到其他文件夹，邮箱服务器上的这些邮件是没有同时被移动的 。

而IMAP提供webmail 与电子邮件客户端之间的双向通信，客户端的操作都会反馈到服务器上，对邮件进行的操作，服务器上的邮件也会做相应的动作。
同时，IMAP像POP3那样提供了方便的邮件下载服务，让用户能进行离线阅读。IMAP提供的摘要浏览功能可以让你在阅读完所有的邮件到达时间、主题、
发件人、大小等信息后才作出是否下载的决定。此外，IMAP 更好地支持了从多个不同设备中随时访问新邮件。

![image.png](https://i.loli.net/2019/09/16/R7w9BobrmKQtpsj.png)

总之：
IMAP 整体上为用户带来更为便捷和可靠的体验。POP3 更易丢失邮件或多次下载相同的邮件，但 IMAP 通过邮件客户端与webmail 之间的双向同步功能很好地避免了这些问题。

注意：
若在web邮箱中设置了“保存到已发送”，使用客户端POP服务发信时，已发邮件也会自动同步到网页端“已发送”文件夹内。

网易163免费邮箱相关服务器信息：

![image.png](https://i.loli.net/2019/09/16/PlFfTWzbQyI4V9u.png)

**Maildirs**
Maildirs是使用非常广泛的e-mail邮件存储格式。也可以说是一种基于目录的邮件存储格式。它在添加，移动或删除时并不依赖于应用程序级的文件锁定来维护消息的完成性。每一个消息(每一封邮件)被保存在一个独立的且名称唯一的文件中。所有的更改均使用基于文件系统的原子操作(atomic filesystem operations )因此文件系统来控制文件锁定从而避免一致性问题。通常Maildir为一个目录(名称为Maildir)其下包含三个子目录，分别为tmp，new和cur。

**Courier IMAP**
Courier IMAP server 是使用Maildir存储格式的高速，可扩展，企业级 IMAP 服务器。许多E-mail提供商使用Courier IMAP server来处理几十万的邮件用户，使用它建立IMAP和POP3集合代理，可以说Courier IMAP server 简直具有无限的水平扩展能力。在代理配置环境中，一些Courier 服务器提供IMAP和POP3服务，它们等待客户端登陆请求，查找并操作邮件用户的mailbox，与服务器建立代理连接，所有的这些操作都在一个单独的，无缝连接的进程中。

Courier-IMAP主要特点：
- 小巧而高效；
- 提供多种用户认证模块和方式；
- 支持虚拟邮箱；
- 可限制IMAP同时登录的总数目及同一个IP地址同时登录的数目，能有效保护系统在受到拒绝服务（Denial-of-service）攻击时不致因超载而瘫痪；

**maildrop**
具有过滤功能的邮件投递代理（MDA）。

**Courier-Authlib**
Courier authentication library 为其他 Courier 应用程序提供验证服务。

**SASL**
SASL的英文全称是Simple Authentication and Security Layer，即简单验证和安全层。SMTP 协议并没有提供用户验证功能，很容易匿名中转邮件。即使限制了可以转发的网段，也不安全。他的定义是： a method for adding authentication support to connection-based protocols，为基于连接的协议提供认证功能。SASL是一个胶合(glue)库，通过这个库把应用层 与 形式多样的认证系统整合在一起。这有点类似于PAM，但是后者是认证方式，决定什么人可以访问什么服务，而SASL是认证过程，侧重于信任建立过程，这个过程可以调用PAM来建立信任关系。

**Open-Relay是什么？**
Open-Relay（开放转发或匿名转发）是指由于邮件服务器不理会邮件发送者或邮件接受者的是否为系统所设定的用户，而对所有的入站邮件一律进行转发（RELAY）的功能。通常，若邮件服务器的此功能开放，则我们一般称此邮件服务器是Open-Relay的。

由于Internet E-mail采用开放式标准，所以MTA、MDA、MUA等不同角色，可分别由许多不同的软件包来扮演。实现相同协议的不同包，可以彼此互相交流，而不管它们是在什么系统上运行。如果将一个完整的E-mail邮件系统集中在一起，可以发现的是处理SMTP的是一套软件，处理POP/IMAP的是另一套软件。但邮件系统中的每一种角色，都有许多不同的软件可以选择。

#### docker部署

如果一步步部署的话会很复杂，参考：https://www.cnblogs.com/kevingrace/p/9383993.html

我们使用docker-compose启动，从官方的GitHub上拉取配置文件（https://github.com/EwoMail/ewomail-docker/blob/master/docker-compose.yml），下载的时候直接使用wget即可

```
version: '2.3'

services:
  portainer:
    image: "ewomail/ewomail"
    container_name: "ewomail"
    hostname: "ewomail"
    restart: always 
    ports:
      - "0.0.0.0:2210:22"
      - "0.0.0.0:25:25"
      - "0.0.0.0:109:109"
      - "0.0.0.0:110:110"
      - "0.0.0.0:143:143"
      - "0.0.0.0:465:465"
      - "0.0.0.0:587:587"
      - "0.0.0.0:993:993"
      - "0.0.0.0:995:995"
      - "127.0.0.1:8010:8000"
      - "127.0.0.1:8011:8010"
      - "127.0.0.1:8012:8020"
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
    privileged: true
    tty: true
    stdin_open: true
```

在配置文件需要注意几点：
1、根据具体情况修改配置文件中的hostname（改了也没用，还是要到后管中修改）；
2、注意宿主机开放必要的端口；
3、注意宿主机端口的占用情况；
4、根据docker-compose配置文件中的配置，WebMail端口被映射为8010，管理后台端口被映射为8011，8012端口映射为phpMyAdmin，请注意合理的网络安全策略；
5、EwoMail默认的后台管理系统后台用户名/密码为admin/ewomail123，Rainloop管理端地址为http://localhost:8010/?admin；

然后使用命令：

```
docker-compose -f docker-compose.yml up -d
```

先登录：http://localhost:8011/

此时可以根据http://doc.ewomail.com/docs/ewomail/ybyx中的步骤，登陆docker容器：

