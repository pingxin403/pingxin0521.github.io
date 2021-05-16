---
title: ssL原理与实现
date: 2019-03-25 09:18:59
tags:
 - Linux
 - 安全
categories:
 - Linux
 - 安全
---

### 前言

跨主机网络通信实质上是不同主机上的进程之间的通信，现在网络通信协议是五层协议一种实现，但是不提供安全保护，所以需要对网络连接进行加密。SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密。

SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。SSL协议可分为两层： SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。 SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

提供服务：

1）认证用户和服务器，确保数据发送到正确的客户机和服务器；

2）加密数据以防止数据中途被窃取；

3）维护数据的完整性，确保数据在传输过程中不被改变。

<!--more-->

### 工作原理

SSL协议实现的安全机制包含：

- 传输数据的机密性：利用对称密钥算法对传输的数据进行加密。

- 身份验证机制：基于证书利用数字签名方法对server和client进行身份验证，当中client的身份验证是可选的。

- 消息完整性验证：消息传输过程中使用MAC算法来检验消息的完整性。

#### 传输数据的机密性

网络上传输的数据非常easy被非法用户窃取，SSL採用在通信两方之间建立加密通道的方法保证传输数据的机密性。

所谓加密通道，是指发送方在发送数据前，使用加密算法和加密密钥对数据进行加密，然后将数据发送给对方。接收方接收到数据后，利用解密算法和解密密钥从密文中获取明文。没有解密密钥的第三方，无法将密文恢复为明文，从而保证传输数据的机密性。

加解密算法分为两类：

-  对称密钥算法：数据加密和解密时使用同样的密钥。

-  非对称密钥算法：数据加密和解密时使用不同的密钥，一个是公开的公钥，一个是由用户秘密保存的私钥。

利用公钥（或私钥）加密的数据仅仅能用对应的私钥（或公钥）才干解密。

与非对称密钥算法相比。对称密钥算法具有计算速度快的长处，通经常使用于对大量信息进行加密（如对全部报文加密）；而非对称密钥算法，一般用于数字签名和对较少的信息进行加密。

SSL加密通道上的数据加解密使用对称密钥算法。眼下主要支持的算法有DES、3DES、AES等，这些算法都能够有效地防止交互数据被窃听。

对称密钥算法要求解密密钥和加密密钥全然一致。因此，利用对称密钥算法加密数据传输之前。须要在通信两端部署同样的密钥。

#### 身份验证机制

电子商务和网上银行等应用中必须保证要登录的Webserver是真实的，以免重要信息被非法窃取。SSL利用数字签名来验证通信对端的身份。

非对称密钥算法能够用来实现数字签名。因为通过私钥加密后的数据仅仅能利用相应的公钥进行解密，因此依据解密是否成功，就能够推断发送者的身份。如同发送者对数据进行了“签名”。比如。Alice使用自己的私钥对一段固定的信息加密后发给Bob，Bob利用Alice的公钥解密，假设解密结果与固定信息同样。那么就能够确认信息的发送者为Alice，这个过程就称为数字签名。

SSLclient必须验证SSLserver的身份，SSLserver是否验证SSLclient的身份。则由SSLserver决定。SSLclient和SSLserver的身份验证过程。

使用数字签名验证身份时。须要确保被验证者的公钥是真实的，否则。非法用户可能会冒充被验证者与验证者通信。如图1所看到的。Cindy冒充Bob，将自己的公钥发给Alice，并利用自己的私钥计算出签名发送给Alice，Alice利用“Bob”的公钥（实际上为Cindy的公钥）成功验证该签名，则Alice觉得Bob的身份验证成功，而实际上与Alice通信的是冒充Bob的Cindy。SSL利用PKI提供的机制保证公钥的真实性。

#### 消息完整性验证

为了避免网络中传输的数据被非法篡改，SSL利用基于MD5或SHA的MAC算法来保证消息的完整性。

MAC算法是在密钥參与下的数据摘要算法，能将密钥和随意长度的数据转换为固定长度的数据。利用MAC算法验证消息完整性的过程如图2所看到的。

发送者在密钥的參与下，利用MAC算法计算出消息的MAC值。并将其加在消息之后发送给接收者。接收者利用相同的密钥和MAC算法计算出消息的MAC值。并与接收到的MAC值比較。假设二者相同。则报文没有改变；否则，报文在传输过程中被改动，接收者将丢弃该报文。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g1euuztx61j30dm03zdfp.jpg)

MAC算法具有例如以下特征，使其可以用来验证消息的完整性：

- 消息的不论什么改变，都会引起输出的固定长度数据产生变化。通过比較MAC值，可以保证接收者可以发现消息的改变。

- MAC算法须要密钥的參与。因此没有密钥的非法用户在改变消息的内容后，无法加入正确的MAC值。从而保证非法用户无法任意改动消息内容。

MAC算法要求通信两方具有同样的密钥，否则MAC值验证将会失败。因此，利用MAC算法验证消息完整性之前，须要在通信两端部署同样的密钥。

#### 利用非对称密钥算法保证密钥本身的安全

对称密钥算法和MAC算法要求通信两方具有同样的密钥。否则解密或MAC值验证将失败。因此。要建立加密通道或验证消息完整性，必须先在通信两方部署一致的密钥。

SSL利用非对称密钥算法加密密钥的方法实现密钥交换，保证第三方无法获取该密钥。如图所看到的，SSLclient（如Web浏览器）利用SSLserver（如Webserver）的公钥加密密钥，将加密后的密钥发送给SSLserver。仅拥有相应私钥的SSLserver才干从密文中获取原始的密钥。SSL通常採用RSA算法加密传输密钥。

![](https://ws1.sinaimg.cn/large/006KyevZgy1g1euzrrys1j30by038745.jpg)

实际上，SSLclient发送给SSLserver的密钥不能直接用来加密数据或计算MAC值。该密钥是用来计算对称密钥和MAC密钥的信息，称为premaster secret。SSLclient和SSLserver利用premaster secret计算出同样的主密钥（master secret）。再利用master secret生成用于对称密钥算法、MAC算法等的密钥。premaster secret是计算对称密钥、MAC算法密钥的关键。

 用来实现密钥交换的算法称为密钥交换算法。非对称密钥算法RSA用于密钥交换时，也能够称之为密钥交换算法。

利用非对称密钥算法加密密钥之前，发送者须要获取接收者的公钥，并保证该公钥确实属于接收者。否则。密钥可能会被非法用户窃取。Cindy冒充Bob，将自己的公钥发给Alice。Alice利用Cindy的公钥加密发送给Bob的数据。Bob因为没有相应的私钥无法解密该数据，而Cindy截取数据后，能够利用自己的私钥解密该数据。SSL利用PKI提供的机制保证公钥的真实性，

#### 利用PKI保证公钥的真实性

PKI通过数字证书来公布用户的公钥，并提供了验证公钥真实性的机制。数字证书（简称证书）是一个包括用户的公钥及其身份信息的文件，证明了用户与公钥的关联。

数字证书由权威机构——CA签发，并由CA保证数字证书的真实性。

SSLclient把密钥加密传递给SSLserver之前，SSLserver须要将从CA获取的证书发送给SSLclient，SSLclient通过PKI推断该证书的真实性。假设该证书确实属于SSLserver，则利用该证书中的公钥加密密钥，发送给SSLserver。

验证SSLserver/SSLclient的身份之前，SSLserver/SSLclient须要将从CA获取的证书发送给对端。对端通过PKI推断该证书的真实性。

假设该证书确实属于SSLserver/SSLclient，则对端利用该证书中的公钥验证SSLserver/SSLclient的身份。

# 协议工作过程

## 3.1  SSL的分层结构

![img](http://www.h3c.com.cn/res/200812/12/20081212_705431_image005_622834_30003_0.png)

图4 SSL协议分层

如图4所看到的，SSL位于应用层和传输层之间，它能够为不论什么基于TCP等可靠连接的应用层协议提供安全性保证。SSL协议本身分为两层：

l              上层为SSL握手协议（SSL handshake protocol）、SSLpassword变化协议（SSL change cipher spec protocol）和SSL警告协议（SSL alert protocol）。

l              底层为SSL记录协议（SSL record protocol）。

当中：

l              SSL握手协议：是SSL协议很重要的组成部分。用来协商通信过程中使用的加密套件（加密算法、密钥交换算法和MAC算法等）、在server和client之间安全地交换密钥、实现server和client的身份验证。



l              SSLpassword变化协议：client和server端通过password变化协议通知对端。随后的报文都将使用新协商的加密套件和密钥进行保护和传输。



l              SSL警告协议：用来向通信对端报告告警信息，消息中包括告警的严重级别和描写叙述。

l              SSL记录协议：主要负责对上层的数据（SSL握手协议、SSLpassword变化协议、SSL警告协议和应用层协议报文）进行分块、计算并加入MAC值、加密。并把处理后的记录块传输给对端。

### SSL握手过程

SSL通过握手过程在client和server之间协商会话參数，并建立会话。会话包括的主要參数有会话ID、对方的证书、加密套件（密钥交换算法、数据加密算法和MAC算法等）以及主密钥（master secret）。通过SSL会话传输的数据，都将採用该会话的主密钥和加密套件进行加密、计算MAC等处理。

不同情况下，SSL握手过程存在差异。

以下将分别描写叙述以下三种情况下的握手过程：

- 仅仅验证server的SSL握手过程

- 验证server和client的SSL握手过程

-  恢复原有会话的SSL握手过程

#### 验证server的SSL握手过程

![](https://ws1.sinaimg.cn/large/006KyevZgy1g1ev9tve3cj308m091aa3.jpg)



如上图所看到的，仅仅须要验证SSLserver身份，不须要验证SSLclient身份时，SSL的握手过程为：

1. SSLclient通过Client Hello消息将它支持的SSL版本号、加密算法、密钥交换算法、MAC算法等信息发送给SSLserver。

2. SSLserver确定本次通信採用的SSL版本号和加密套件，并通过Server Hello消息通知给SSLclient。 设SSLserver同意SSLclient在以后的通信中重用本次会话，则SSLserver会为本次会话分配会话ID。并通过Server Hello消息发送给SSLclient。

3. SSLserver将携带自己公钥信息的数字证书通过Certificate消息发送给SSLclient。

4. SSLserver发送Server Hello Done消息。通知SSLclient版本号和加密套件协商结束。開始进行密钥交换。

5. SSLclient验证SSLserver的证书合法后，利用证书中的公钥加密SSLclient随机生成的premaster secret，并通过Client Key Exchange消息发送给SSLserver。

6. SSLclient发送Change Cipher Spec消息，通知SSLserver兴许报文将採用协商好的密钥和加密套件进行加密和MAC计算。

7. SSLclient计算已交互的握手消息（除Change Cipher Spec消息外全部已交互的消息）的Hash值，利用协商好的密钥和加密套件处理Hash值（计算并加入MAC值、加密等），并通过Finished消息发送给SSLserver。SSLserver利用相同的方法计算已交互的握手消息的Hash值，并与Finished消息的解密结果比較，假设二者相同，且MAC值验证成功，则证明密钥和加密套件协商成功。

8. 相同地。SSLserver发送Change Cipher Spec消息，通知SSLclient兴许报文将採用协商好的密钥和加密套件进行加密和MAC计算。

9. SSLserver计算已交互的握手消息的Hash值，利用协商好的密钥和加密套件处理Hash值（计算并加入MAC值、加密等），并通过Finished消息发送给SSLclient。SSLclient利用相同的方法计算已交互的握手消息的Hash值，并与Finished消息的解密结果比較，假设二者相同。且MAC值验证成功。则证明密钥和加密套件协商成功。

SSLclient接收到SSLserver发送的Finished消息后。假设解密成功，则能够推断SSLserver是数字证书的拥有者，即SSLserver身份验证成功，由于仅仅有拥有私钥的SSLserver才干从Client Key Exchange消息中解密得到premaster secret，从而间接地实现了SSLclient对SSLserver的身份验证。

> Change Cipher Spec消息属于SSLpassword变化协议，其它握手过程交互的消息均属于SSL握手协议，统称为SSL握手消息。
>
> 计算Hash值。指的是利用Hash算法（MD5或SHA）将随意长度的数据转换为固定长度的数据。





#### 验证server和client的SSL握手过程

![](https://ws1.sinaimg.cn/large/006KyevZgy1g1ev9xx9hrj308m0bkwel.jpg)

SSLclient的身份验证是可选的，由SSLserver决定是否验证SSLclient的身份。

图中蓝色部分标识的内容所看到的，假设SSLserver验证SSLclient身份。则SSLserver和SSLclient除了交互中的消息协商密钥和加密套件外，还须要进行下面操作：

1. SSLserver发送Certificate Request消息。请求SSLclient将其证书发送给SSLserver。

2. SSLclient通过Certificate消息将携带自己公钥的证书发送给SSLserver。SSLserver验证该证书的合法性。

3. SSLclient计算已交互的握手消息、主密钥的Hash值。利用自己的私钥对其进行加密，并通过Certificate Verify消息发送给SSLserver。

4.  SSLserver计算已交互的握手消息、主密钥的Hash值。利用SSLclient证书中的公钥解密Certificate Verify消息，并将解密结果与计算出的Hash值比較。假设二者同样，则SSLclient身份验证成功。



#### 恢复原有会话的SSL握手过程

![](https://ws1.sinaimg.cn/large/006KyevZgy1g1eva04881j308m06jt8o.jpg)

协商会话參数、建立会话的过程中。须要使用非对称密钥算法来加密密钥、验证通信对端的身份。计算量较大，占用了大量的系统资源。

为了简化SSL握手过程。SSL同意重用已经协商过的会话，详细过程为：

1. SSLclient发送Client Hello消息，消息中的会话ID设置为计划重用的会话的ID。

2. SSLserver假设同意重用该会话，则通过在Server Hello消息中设置同样的会话ID来应答。这样，SSLclient和SSLserver就能够利用原有会话的密钥和加密套件。不必又一次协商。

3.  SSLclient发送Change Cipher Spec消息，通知SSLserver兴许报文将採用原有会话的密钥和加密套件进行加密和MAC计算。

4. SSLclient计算已交互的握手消息的Hash值，利用原有会话的密钥和加密套件处理Hash值，并通过Finished消息发送给SSLserver，以便SSLserver推断密钥和加密套件是否正确。

5. 相同地。SSLserver发送Change Cipher Spec消息，通知SSLclient兴许报文将採用原有会话的密钥和加密套件进行加密和MAC计算。

6. SSLserver计算已交互的握手消息的Hash值，利用原有会话的密钥和加密套件处理Hash值，并通过Finished消息发送给SSLclient。以便SSLclient推断密钥和加密套件是否正确。

### OpenSSL使用

在计算机网络上，OpenSSL是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连接者的身份。这个包广泛被应用在互联网的网页服务器上。

OpenSSL整个软件包大概可以分成三个主要的功能部分：SSL协议库、应用程序以及密码算法库。OpenSSL的目录结构自然也是围绕这三个功能部分进行规划的。作为一个基于密码学的安全开发包，OpenSSL提供的功能相当强大和全面，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。

其组成主要包括一下三个组件：

- openssl：多用途的命令行工具
- libcrypto：加密算法库
- libssl：加密模块应用库，实现了ssl及tls

#### 对称加密

对称加密需要使用的标准命令为 enc ，用法如下：

```shell
openssl enc -ciphername [-in filename] [-out filename] [-pass arg] \
[-e] [-d] [-a/-base64] [-A] [-k password] [-kfile filename] [-K key] \
[-iv IV] [-S salt] [-salt] [-nosalt] [-z] [-md] [-p] [-P] \
[-bufsize number] [-nopad] [-debug] [-none] [-engine id]
```

常用选项有：

-in filename：指定要加密的文件存放路径

-out filename：指定加密后的文件存放路径

-salt：自动插入一个随机数作为文件内容加密，默认选项

-e：可以指明一种加密算法，若不指的话将使用默认加密算法

-d：解密，解密时也可以指定算法，若不指定则使用默认算法，但一定要与加密时的算法一致

-a/-base64：使用-base64位编码格式

示例：

```
加密：]# openssl enc -e -des3 -a -salt -in fstab -out jiami
解密：]# openssl enc -d -des3 -a -salt -in fstab -out jiami
```

#### 单向加密

单向加密需要使用的标准命令为 dgst ，用法如下：

```
openssl dgst [-md5|-md4|-md2|-sha1|-sha|-mdc2|-ripemd160|-dss1] \
[-c] [-d] [-hex] [-binary] [-out filename] [-sign filename] \
[-keyform arg] [-passin arg] [-verify filename] [-prverify filename] \
[-signature filename] [-hmac key] [file...]
```

常用选项有：

[-md5|-md4|-md2|-sha1|-sha|-mdc2|-ripemd160|-dss1] ：指定一种加密算法

-out filename：将加密的内容保存到指定文件中

示例：

```shell
[hyp@localhost ~]$ openssl dgst -md5 /etc/fstab
MD5(/etc/fstab)= 90bdb6bf9d7e644cea6e7602913afb75
[hyp@localhost ~]$ openssl dgst -md5 /etc/mtab
MD5(/etc/mtab)= 1b963956716d73980eaec0f0f324180b
```

单向加密除了 openssl dgst 工具还有：

md5sum，sha1sum，sha224sum，sha256sum ，sha384sum，sha512sum

示例：

```shell
[hyp@localhost ~]$ md5sum /etc/fstab
90bdb6bf9d7e644cea6e7602913afb75  /etc/fstab
[hyp@localhost ~]$ sha512sum /etc/fstab
f21432b91d805e877fff94a2b7ed91dde4f7dd008e7629a1ba97d47e07f3c25debc38934854bb4aeb8966952bb4f1165a684f319fb75fe5412e2ad246ec5bb54  /etc/fstab
```

#### 生成密码

生成密码需要使用的标准命令为 passwd ，用法如下：

```
openssl passwd [-crypt] [-1] [-apr1] [-salt string] [-in file] [-stdin] [-noverify] [-quiet] [-table] {password}
```

常用选项有：

-1：使用md5加密算法

-salt string：加入随机数，最多8位随机数

-in file：对输入的文件内容进行加密

-stdion：对标准输入的内容进行加密

示例如下：

```shell
[hyp@localhost ~]$ openssl passwd  970603
.F1Eg5Xr2f9Ak
[hyp@localhost ~]$ openssl passwd  -salt 16  970603
16K52.tSElFgg
[hyp@localhost ~]$ openssl passwd  -salt 16  970603
16K52.tSElFgg
```

#### 生成随机数

生成随机数需要用到的标准命令为 rand ，用法如下：

```
openssl rand [-out file] [-rand file(s)] [-base64] [-hex] num
```

常用选项有：

-out file：将生成的随机数保存至指定文件中

-base64：使用base64 编码格式

-hex：使用16进制编码格式

示例如下：

```shell
[hyp@localhost ~]$ openssl rand -hex 2
14d4
[hyp@localhost ~]$ openssl rand -base64 2
41c=
```

#### 生成秘钥对

首先需要先使用 genrsa 标准命令生成私钥，然后再使用 rsa 标准命令从私钥中提取公钥。

genrsa 的用法如下：

```
openssl genrsa [-out filename] [-passout arg] [-des] [-des3] [-idea] [-f4] [-3] [-rand file(s)] [-engine id] [numbits]
```

常用选项有：

-out filename：将生成的私钥保存至指定的文件中

-des|-des3|-idea：不同的加密算法

numbits：指定生成私钥的大小，默认是2048

一般情况下秘钥文件的权限一定要控制好，只能自己读写，因此可以使用 umask 命令设置生成的私钥权限，示例如下：

```shell
[hyp@localhost ~]$ (umask 077; openssl genrsa -out rsa_pri.key 2048)
Generating RSA private key, 2048 bit long modulus
...................+++
......................................................+++
e is 65537 (0x10001)
```

ras 的用法如下：

```
openssl rsa [-inform PEM|NET|DER] [-outform PEM|NET|DER] [-in filename] [-passin arg] [-out filename] [-passout arg]
       [-sgckey] [-des] [-des3] [-idea] [-text] [-noout] [-modulus] [-check] [-pubin] [-pubout] [-engine id]
```

常用选项：

-in filename：指明私钥文件

-out filename：指明将提取出的公钥保存至指定文件中

-pubout：根据私钥提取出公钥

示例如下：

```shell
[hyp@localhost ~]$ openssl rsa -in rsa_pri.key  -out rsa_pub.key -pubout
writing RSA key
[hyp@localhost ~]$ cat rsa_pub.key
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA6atn/OguE905kVkGsQLi
fsNMw501ylNBG4f7ZfyJ88Iw8+lW7WC1/5bOZOXVt4z9AA+CfZlHDE8n4qUbPEOk
k1k0sBhzkESwcEWDaykzcRNg4XjmUerDOkffLct6VxavYFMm4bni58E63NmjrM7e
Wkz0T1lbDOo28b2KYS/tqxRZ7r8JrBxMPxJCbtpeAcZitfm/EqADwzdzPWpsZDmq
r9gH19POhnOgvc180JetQM+MiA2+IlLSD9M5QCcGj+94gy847rnvqwtLm2XWP0ai
PCeS3dUWW7ungDHymbAuCCR4qanyC+vQXQoCqamuDteFvsyikahdJoCWAMkoA8DJ
LwIDAQAB
-----END PUBLIC KEY-----
```

#### 创建CA和申请证书

使用openssl工具创建CA证书和申请证书时，需要先查看配置文件，因为配置文件中对证书的名称和存放位置等相关信息都做了定义，具体可参考 /etc/pki/tls/openssl.cnf 文件。

```
1. PKC：Public-Key Certificate，公钥证书，简称证书。
2. CA：Certification Authority，认证机构。对证书进行管理，负责 1.生成密钥对、2. 注册公钥时对身份进行认证、3. 颁发证书、4. 作废证书。其中负责注册公钥和身份认证的，称为 RA（Registration Authority 注册机构）
3. PKI：Public-Key Infrastructure，公钥基础设施，是为了更高效地运用公钥而制定的一系列规范和规格的总称。比较著名的有PKCS（Public-Key Cryptography Standards，公钥密码标准，由 RSA 公司制定）、X.509 等。PKI 是由使用者、认证机构 CA、仓库（保存证书的数据库）组成。
CRL：Certificate Revocation List 证书作废清单，是 CA 宣布作废的证书一览表，会带有 CA 的数字签名。一般由处理证书的软件更新 CRL 表，并查询证书是否有效。
4.证书的编码格式:pem,der
```

  在要生成证书的目录下建立几个文件和文件夹，有./demoCA/ ./demoCA/newcerts/  ./demoCA/index.txt ./demoCA/serial，在serial文件中写入第一个序列号“01”

1.生成X509格式的CA自签名证书

```
$openssl req -new -x509 -keyout ca.key -out ca.crt
```

2.生成服务端的私钥(key文件)及csr文件

```
$openssl genrsa -des3 -out server.key 1024
$openssl req -new -key server.key -out server.csr
```

3.生成客户端的私钥(key文件)及csr文件

```
$openssl genrsa -des3 -out client.key 1024
$openssl req -new -key client.key -out client.csr
```

4.用生成的CA的证书为刚才生成的server.csr,client.csr文件签名

```
$openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key
$openssl ca -in client.csr -out client.crt -cert ca.crt -keyfile ca.key
```

5.生成p12格式证书

```
$openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx
$openssl pkcs12 -export -inkey server.key -in server.crt -out server.pfx
```

6.生成pem格式证书
有时需要用到pem格式的证书，可以用以下方式合并证书文件（crt）和私钥文件（key）来生成

```
$cat client.crt client.key> client.pem
$cat server.crt server.key > server.pem
```

7.PFX文件转换为X509证书文件和RSA密钥文件

```
$openssl pkcs12 -in server.pfx -nodes -out server.pem
$openssl rsa -in server.pem -out server2.key
$openssl x509 -in server.pem -out server2.crt
```

这样生成服务端证书：ca.crt, server.key, server.crt, server.pem, server.pfx，客户端证书：ca.crt, client.key, client.crt, client.pem, client.pfx  

#### 吊销证书

吊销证书的步骤也是在CA服务器上执行的，以刚才新建的 httpd.crt 证书为例，吊销步骤如下：

第一步：在客户机上获取要吊销证书的 serial 和 subject 信息

第二步：根据客户机提交的 serial 和 subject 信息，对比其余本机数据库 index.txt 中存储的是否一致

第三步：执行吊销操作

```shell
[hyp@localhost CA]$ sudo openssl ca -revoke /etc/pki/CA/newcerts/01.pem
Using configuration from /etc/pki/tls/openssl.cnf
Revoking Certificate 01.
Data Base Updated
```

第四步：生成吊销证书的吊销编号 （第一次吊销证书时执行）

]# echo 01 > /etc/pki/CA/crlnumber

第五步：更新证书吊销列表

]# openssl ca -gencrl -out /etc/pki/CA/crl/ca.crl

查看 crl 文件命令：

]# openssl crl -in /etc/pki/CA/crl/ca.crl -noout -text



参考：[openssl命令使用](https://www.e-learn.cn/content/linux/1448064)



### OpenSSH使用

OpenSSH 是 SSH （Secure SHell） 协议的免费开源实现。SSH协议族可以用来进行远程控制， 或在计算机之间传送文件。而实现此功能的传统方式，如telnet(终端仿真协议)、 rcp ftp、 rlogin、rsh都是极为不安全的，并且会使用明文传送密码。OpenSSH提供了服务端后台程序和客户端工具，用来加密远程控制和文件传输过程中的数据，并由此来代替原来的类似服务。

OpenSSH服务，sshd，是一个典型的独立守护进程(standalone daemon)，但也可以根据需要通过网络守护进程(Internet Daemon)-inetd或Ineternet Daemon's more modern-xinted加载。OpenSSH服务可以通过/etc/ssh/sshd_config文件进行配置。

程序主要包括了几个部分：

```
ssh
	rlogin与Telnet的替代方案。
scp、sftp
	rcp的替代方案，将文件复制到其他电脑上。
sshd
	SSH服务器。
ssh-keygen
	产生RSA或DSA密钥，用来认证用。
ssh-agent、ssh-add
	帮助用户不需要每次都要输入密钥密码的工具。
ssh-keyscan
	扫描一群机器，并记录其公钥。
```

**服务器端安装**

需要安装的软件:
openssh-server：服务器端
openssh：服务器端与客户端核心文件

**客户端安装**

需要安装的软件：
openssh：服务器端与客户端核心文件
openssh-clients : 客户端

**注意：** 如果一台机器既要做客户端又要做服务器端，就需要在该系统中安装上述三个软件包；CentOS 7中默认已安装好上述三个软件包。

#### 免密远程登陆及免密远程拷贝设置

为什么设置免密登录及远程拷贝？

方便操作，处理快速；
计算机集群中机器之间有频繁的数据交换需求。
设置方法：（假设A、B计算机要进行加密通信）

A计算机用户的命令行输入ssh-keygen –t rsa，生成密钥对；
若B计算机授权给A免密钥登录B，则将A计算机的公钥放入B计算机的authorized_keys文件中。

通俗理解设置：将计算机的信任关系与人之间的信任关系作类比。张三若信任李四，则表示李四在张三的受信任名单的列表中（类比A计算机的公钥放到B计算机的authorized_keys文件中）。

A计算机上操作：

```
]# ssh-keygen -t rsa #默认就好
#生成后复制id_rsa.pub里面的内容，使用PUTTY或其他远程连接工具，连接B计算机，
#创建~/.ssh/authorized_keys，
#使用vi ~/.ssh/authorized_keys，将复制公钥粘贴到该文件内,保存退出。
```

B计算机上配置：

更改/etc/ssh/sshd_config文件中配置

```
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication yes
```

更改~/.ssh的权限

```
[hyp@localhost ~]$ sudo chmod 700 ~/.ssh
[hyp@localhost ~]$ chmod 600 ~/.ssh/authorized_keys
```

这样A就可以无秘登录到B。
