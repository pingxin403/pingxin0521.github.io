---
title: Java 网络编程(一)
date: 2019-04-07 08:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 前言

Java提供了非常易用的网络API，调用这些API我们可以很方便的通过建立TCP/IP或UDP套接字，在网络之间进行相互通信，其中TCP要比UDP更加常用。

<!--more-->

尽管Java网络API允许我们通过套接字（Socket）打开或关闭网络连接，但所有的网络通信均是基于Java IO类 InputStream和OutputStream实现的。

此外，我们还可以使用Java NIO API中相关的网络类，用法与Java网络API基本类似，Java NIO API可以以非阻塞模式工作，在某些特定的场景中使用非阻塞模式可以获得较大的性能提升。关于NIO，我们会在后面详细讲到。

**Java TCP网络基础**

通常情况下，客户端打开一个连接到服务器端的TCP/IP连接，然后客户端开始与服务器之间通信，当通信结束后客户端关闭连接，过程如下图所示：

![tcp.jpg](https://i.loli.net/2019/04/15/5cb3dad9bdf7b.jpg)

客户端通过一个已打开的连接可以发送不止一个请求。事实上在服务器处于接收状态下，客户端可以发送尽可能多的数据，服务器也可以主动关闭连接。

**Java中Socket类和ServerSocket类**

当客户端想要打开一个连接到服务器的TCP/IP连接时，就要使用到Java Socket类。socket类只需要被告知连接的IP地址和TCP端口，其余的都有Java实现。

假如我们想要打开一个监听服务，来监听客户端连接某些指定TCP端口的连接，那就需要使用Java ServerSocket类。当客户端通过Socket连接服务器端的ServerSocket监听时，服务器端会指定这个连接的一个Socket，此时客户端与服务器端间的通信就变成Socket与Socket之间的通信。

**Java UDP网络基础**

UDP的工作方式与TCP相比略有不同。使用UDP通信时，在客户端与服务器之间并没有建立连接的概念，客户端发送到服务器的数据，服务器可能（也可能并没有）收到这些数据，而且客户端也并不知道这些数据是否被服务器成功接收。当服务器向客户端发送数据时也是如此。

正因为是不可靠的数据传输，UDP相比与TCP来说少了很多的协议开销。

在某些场景中，使用无连接的UDP要优于TCP。

### URL


Java的网络类可以让你通过网络或者远程连接来实现应用。而且，这个平台现在已经可 以对国际互联网以及URL资源进行访问了。Java的URL类可以让访问网络资源就像是访问你本地的文件夹一样方便快捷。我们通过使用Java的URL类 就可以经由URL完成读取和修改数据的操作。

通过一个URL连接，我们就可以确定资源的位置，比如网络文件、网络页面以及网络应用程序等。其中包含了许多的语法元素。
从URL得到的数据可以是多种多样的，这些都需要一种统一的机制来完成对URL的读取与修改操作。Java语言在它的java.net软件包里就提供了这么一种机制。

URL class是从URL标示符中提取出来的。它允许Java程序设计人员打开某个特定URL连接，并对里边的数据进行读写操作以及对首部信息进行读写操作。而且，它还允许程序员完成其它的一些有关URL的操作。

#### what is URL?
url是统一资源定位符，对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址。互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。

![1.jpeg](https://i.loli.net/2019/06/26/5d1363fee4df538457.jpeg)

URL由三部分组成：资源类型、存放资源的主机域名、资源文件名。URL的一般语法格式为(带方括号[]的为可选项)：

```
protocol :// hostname[:port] / path / [;parameters][?query]#fragment
```

**protocol（协议）**

指定使用的传输协议，下表列出 protocol 属性的有效方案名称。 最常用的是HTTP协议，它也是目前WWW中应用最广的协议。

1. file 资源是本地计算机上的文件。格式file:///，注意后边应是三个斜杠。
2. ftp 通过 FTP访问资源。格式 FTP://
3. gopher 通过 Gopher 协议访问该资源。
4. http 通过 HTTP 访问该资源。 格式 HTTP://
5. https 通过安全的 HTTPS 访问该资源。 格式 HTTPS://
6. mailto 资源为电子邮件地址，通过 SMTP 访问。 格式 mailto:
7. MMS 通过 支持MMS（流媒体）协议的播放该资源。（代表软件：Windows Media Player）格式 MMS://
8. ed2k 通过 支持ed2k（专用下载链接）协议的P2P软件访问该资源。（代表软件：电驴） 格式 ed2k://
9. Flashget 通过 支持Flashget:（专用下载链接）协议的P2P软件访问该资源。（代表软件：快车） 格式 Flashget://
10. thunder 通过 支持thunder（专用下载链接）协议的P2P软件访问该资源。（代表软件：迅雷） 格式 thunder://
11. news 通过 NNTP 访问该资源。

**hostname（主机名）**

是指存放资源的服务器的域名系统(DNS) 主机名或 IP 地址。有时，在主机名前也可以包含连接到服务器所需的用户名和密码（格式：username:password@hostname）。

**port（端口号）**

整数，可选，省略时使用方案的默认端口，各种传输协议都有默认的端口号，如http的默认端口为80。如果输入时省略，则使用默认端口号。有时候出于安全或其他考虑，可以在服务器上对端口进行重定义，即采用非标准端口号，此时，URL中就不能省略端口号这一项。

**path（路径）**

由零或多个“/”符号隔开的字符串，一般用来表示主机上的一个目录或文件地址。

**parameters（参数）**

这是用于指定特殊参数的可选项。

**query(查询)**

可选，用于给动态网页（如使用CGI、ISAPI、PHP/JSP/ASP/ASP。NET等技术制作的网页）传递参数，可有多个参数，用“&”符号隔开，每个参数的名和值用“=”符号隔开。

**fragment（信息片断）**

字符串，用于指定网络资源中的片断。例如一个网页中有多个名词解释，可使用fragment直接定位到某一名词解释。


#### URL + URLConnection
在java.net包中包含两个有趣的类：URL类和URLConnection类。这两个类可以用来创建客户端到web服务器（HTTP服务器）的连接。下面是一个简单的代码例子：
```java
URL url = new URL("http://127.0.0.1:80");
URLConnection urlConnection = url.openConnection();
InputStream input = urlConnection.getInputStream();
int data = input.read();
while(data != -1){
System.out.print((char) data);
data = input.read();
}
input.close();
```

默认端口是80。

##### HTTP GET和POST

示例：

```java
//get
public  String doGet(String httpurl) {
		    String message="";
	        try {
	            // 创建远程url连接对象
	            URL url = new URL(httpurl);
	            // 通过远程url连接对象打开一个连接，强转成httpURLConnection类
	            HttpURLConnection connection = (HttpURLConnection)url.openConnection();
	            // 设置连接方式：get
	            connection.setRequestMethod("GET");
	            // 设置连接主机服务器的超时时间：15000毫秒
	            connection.setConnectTimeout(15000);
	            // 设置读取远程返回的数据时间：60000毫秒
	            connection.setReadTimeout(60000);
	            // 发送请求
	            connection.connect();
	            // 通过connection连接，获取输入流
	            if (connection.getResponseCode() == 200) {
	            	 InputStream inputStream=connection.getInputStream();
	                 byte[] data=new byte[1024];
	                 StringBuffer sb1=new StringBuffer();
	                 int length=0;
	                 while ((length=inputStream.read(data))!=-1){
	                     String s=new String(data, 0,length);
	                     sb1.append(s);
	                 }
	                 message=sb1.toString();
	                 inputStream.close();
	            }
	            //关闭连接
	            connection.disconnect();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	 
	        return message;
	    }
	   
//post
public  String getPost(String urls,Map<String, String> params){
        String message="";
        try {
        	//创建URL实例，打开URLConnection
            URL url=new URL(urls);
            // 通过远程url连接对象打开连接
            HttpURLConnection connection= (HttpURLConnection) url.openConnection();
            //设置连接参数
            connection.setRequestMethod("POST");//提交方式
            // 默认值为：false，当向远程服务器传送数据/写数据时，需要设置为true
            connection.setDoOutput(true);
            // 默认值为：true，当前向远程服务读取数据时，设置为true，该参数可有可无
            connection.setDoInput(true);
            connection.setUseCaches(false);
            // 设置连接主机服务器超时时间
            connection.setConnectTimeout(30000);
            // 设置读取主机服务器返回数据超时时间
            connection.setReadTimeout(30000);
            // 设置接收数据的格式:json格式数据
            connection.setRequestProperty("Accept", "application/json"); 
            //设置请求数据类型：json格式数据
            connection.setRequestProperty("Content-type","application/json");
            //链接并发送
            connection.connect();
            // 通过连接对象获取一个输出流
            OutputStream out = connection.getOutputStream(); 
            // 通过输出流对象将参数写出去/传输出去,它是通过字节数组写出的
			out.write(JSONObject.fromObject(params).toString().getBytes());
			out.flush();
			out.close();
			
            // 通过连接对象获取一个输入流，向远程读取
            InputStream inputStream=connection.getInputStream();
            byte[] data=new byte[1024];
            StringBuffer sb1=new StringBuffer();
            int length=0;
            while ((length=inputStream.read(data))!=-1){
                String s=new String(data, 0,length);
                sb1.append(s);
            }
            message=sb1.toString();
            inputStream.close();
            //关闭连接
            connection.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return message;
    }
```



##### 从URLs到本地文件

URL也被叫做统一资源定位符。如果你的代码不关心文件是来自网络还是来自本地文件系统，URL类是另外一种打开文件的方式。
下面是一个如何使用URL类打开一个本地文件系统文件的例子：
```java
URL url = new URL("file:///etc/fstab");
URLConnection urlConnection = url.openConnection();
InputStream input = urlConnection.getInputStream();
int data = input.read();
while(data != -1){
System.out.print((char) data);
data = input.read();
}
input.close();
```
注意：这和通过HTTP访问一个web服务器上的文件的唯一不同处就是URL：”file:///etc/fstab”。

**JarURLConnection**

Java的JarURLConnection类用来连接Java Jar文件。一旦连接上，你可以获取Jar文件的信息。一个简单的例子如下：
```java
String urlString = "http://www.riaway.com/uploads/soft/20150602/1433229921.jar";

URL jarUrl = new URL(urlString);
JarURLConnection connection = new JarURLConnection(jarUrl);

Manifest manifest = connection.getManifest();

JarFile jarFile = connection.getJarFile();
//do something with Jar file...
```

#### NetworkInterface

在使用 Java 开发网络程序时，有时候我们需要知道本机在局域网中的 IP 地址。很常见的一种做法是调用本地命令（比如 Windows 上的 ipconfig 命令和 Linux 上的 ifconfig 命令），接着解析本地命令的输出，最后得到本机在局域网内的 IP 地址。很明显，这种做法不够方便，也不够 Java。于是引出了 Java 在 JDK1.4 的时候添加的一个类： NetworkInterface 。

顾名思义，NetworkInterface  用于表示一个网络接口，这可以是一个物理的网络接口，也可以是一个虚拟的网络接口，而一个网络接口通常由一个 IP 地址来表示。既然  NetworkInterface  用来表示一个网络接口，那么如果可以获得当前机器所有的网络接口（包括物理的和虚拟的），然后筛选出表示局域网的那个网络接口，那就可以得到机器在局域网内的  IP 地址。

查看 NetworkInterface 类的所有静态方法：

```java
//获取当前机器上所有网络接口
public static Enumeration<NetworkInterface> getNetworkInterfaces()
//通过ip地址获取网络接口
public static NetworkInterface getByInetAddress(InetAddress addr) throws SocketException 
//通过接口名获取获取接口
public static NetworkInterface getByName(String name) throws SocketException
//通过下标获取获取接口
public static NetworkInterface getByIndex(int index) throws SocketException
```

NetworkInterface 类的所有方法：

```java
int getIndex()//网络接口的下标
String getName()// 获取网络接口的名称
String getDisplayName()// 获取网络接口的显示名称
int getMTU()//返回此接口的最大传输单元（Maximum Transmission Unit，MTU）
String getName()//获取此网络接口的名称
boolean isLoopback()//返回此网络接口是否是回送接口
boolean isPointToPoint()//返回此网络接口是否是点对点接口
boolean isUp()//返回此网络接口是否已经开启并运行
boolean isVirtual()//返回此接口是否是虚拟接口
boolean supportsMulticast()//是否支持多播
NetworkInterface getParent()//获取父网络接口
Enumeration<InetAddress> getInetAddresses()//返回绑定到该网卡的所有的 IP 地址
byte[] getHardwareAddress()//hardware address (usually MAC)
```

通过 API 文档可知，*getInetAddresses* 方法返回绑定到该网卡的所有的 IP 地址。（虽然一个网络接口确实可以绑定多个 IP 地址，然而通常情况下，一个网络接口都是只对应一个 IP 地址）

示例：获取本机网络接口信息

```java
import java.net.*;
import java.util.Arrays;
import java.util.Enumeration;

public class InetAddressExample {

    public static void main(String[] args) {

        // Get the network interfaces and associated addresses for this host
        try {
            Enumeration<NetworkInterface> interfaceList = NetworkInterface.getNetworkInterfaces();
            if (interfaceList == null) {
                System.out.println("--No interfaces found--");
            } else {
                while (interfaceList.hasMoreElements()) {
                    NetworkInterface iface = interfaceList.nextElement();
                    System.out.println("-------(" + iface.getIndex() + ")-------");
                    System.out.println("Interface " + iface.getName() + ":");
                    System.out.println(iface.toString());

                    Enumeration<InetAddress> addrList = iface.getInetAddresses();
                    if (!addrList.hasMoreElements()) {
                        System.out.println("\t(No addresses for this interface)");
                    }
                    while (addrList.hasMoreElements()) {
                        InetAddress address = addrList.nextElement();
                        System.out.print("\tAddress "
                                + ((address instanceof Inet4Address ? "(v4)"
                                : (address instanceof Inet6Address ? "(v6)" : "(?)"))));
                        System.out.println(": " + address.getHostAddress());
                    }
                    System.out.println("Hardware Address:" + Arrays.toString(iface.getHardwareAddress()));
                    System.out.println("MTU:" + iface.getMTU());
                    System.out.println("isLoopback:" + iface.isLoopback());
                    System.out.println("isPointToPoint:" + iface.isPointToPoint());
                    System.out.println("isUp:" + iface.isUp());
                    System.out.println("isVirtual:" + iface.isVirtual());
                    System.out.println("supportsMulticast:" + iface.supportsMulticast());
                    System.out.println("Parent:" + iface.getParent());
//                    System.out.println("InterfaceAddresses:");
//                    List<InterfaceAddress> interfaceAddresses = iface.getInterfaceAddresses();
//                    for (InterfaceAddress interfaceAddress : interfaceAddresses) {
//                        System.out.println(interfaceAddress.toString());
//                    }

                }
            }
        } catch (SocketException se) {
            System.out.println("Error getting network interfaces:" + se.getMessage());
        }

        // Get name(s)/address(es) of hosts given on command line
        for (String host : args) {
            try {
                System.out.println(host + ":");
                InetAddress[] addressList = InetAddress.getAllByName(host);
                for (InetAddress address : addressList) {
                    System.out.println("\t" + address.getHostName() + "/" + address.getHostAddress());
                }
            } catch (UnknownHostException e) {
                System.out.println("\tUnable to find address for " + host);
            }
        }
    }
}
```

#### InetAddress

InetAddress是java_socket编程一个必不可少的类，此类表示互联网协议 (IP) 地址。

Internet 上的主机有两种方式表示地址，分别为域名和 IP 地址。java.net 包中的 InetAddress 类对象包含一个 Internet 主机地址的域名和 IP 地址。

InetAddress 是 Java 对 IP 地址的封装。这个类的实例经常和 UDP DatagramSockets 和 Socket，ServerSocket 类一起使用。有两个子类：Inet4Address、Inet6Address，前者表示IPV4，后者表示IPV6

**主机名解析**

主机名到 IP 地址的 解析 通过使用本地机器配置信息和网络命名服务（如域名系统（Domain Name System，DNS）和网络信息服务（Network Information Service，NIS））来实现。要使用的特定命名服务默认情况下是本地机器配置的那个。对于任何主机名称，都返回其相应的 IP 地址。

反向名称解析 意味着对于任何 IP 地址，都返回与 IP 地址关联的主机。

InetAddress 类提供将主机名解析为其 IP 地址（或反之）的方法。

**InetAddress 缓存**

InetAddress 类具有一个缓存，用于存储成功及不成功的主机名解析。
默认情况下，当为了防止 DNS 哄骗攻击安装了安全管理器时，正主机名解析的结果会永远缓存。当未安装安全管理器时，默认行为将缓存一段有限（与实现相关）时间的条目。不成功主机名解析的结果缓存非常短的时间（10 秒）以提高性能。

如果不需要默认行为，则可以将 Java 安全属性设置为另外的 Time-to-live (TTL) 值来进行正缓存。类似地，系统管理员在需要时可以配置另外的负缓存 TTL 值。

**两个 Java 安全属性控制着用于正负主机名解析缓存的 TTL 值：**

- networkaddress.cache.ttl

  指示从名称服务进行成功名称查找的缓存策略。该值被指定为整数，指示缓存成功查找的秒数。默认设置将在某个特定于实现的时间内缓存。
  值 -1 指示“永远缓存”。

- networkaddress.cache.negative.ttl（默认值：10）

  指示从名称服务进行不成功名称查找的缓存策略。该值被指定为整数，指示缓存不成功查找故障的秒数。

  值 0 指示“永远不缓存”。值 -1 指示“永远缓存”

**InetAddress的方法**

![1.png](https://i.loli.net/2019/06/26/5d1362492015c67784.png)

##### 创建

InetAddress 没有公开的构造方法，因此你必须通过一系列静态方法中的某一个来获取它的实例。

InetAddress类的方法会抛出UnknownHostException异常，必须进行异常处理

下面是为一个域名实例化 InetAddres 类的例子：

```
InetAddress address = InetAddress.getByName("jenkov.com");
```

当然也会有为匹配某个 IP 地址来实例化一个 InetAddress:

```
InetAddress address = InetAddress.getByName("127.0.0.1");
```

另外，它还有通过获取本地 IP 地址的来获取 InetAddress 的方法（正在运行程序的那台机器）

```
InetAddress address = InetAddress.getLocalHost();
```

示例

```java
InetAddress addr = InetAddress.getByName("www.baidu.com");
System.out.println(addr.toString());

addr = InetAddress.getLocalHost();
System.out.println(addr.toString());
//www.baidu.com/39.156.66.18
//hyp-HP-Notebook/127.0.1.1
```

### TCP

TCP（Transmission Control Protocol 传输控制协议）是一种面向连接的、可靠的、基于字节流的传输层通信协议，由IETF的RFC 793定义。在简化的计算机网络OSI模型中，它完成第四层传输层所指定的功能，用户数据报协议（UDP）是同一层内另一个重要的传输协议。在因特网协议族（Internet protocol suite）中，TCP层是位于IP层之上，应用层之下的中间层。不同主机的应用层之间经常需要可靠的、像管道一样的连接，但是IP层不提供这样的流机制，而是提供不可靠的包交换。

应用层向TCP层发送用于网间传输的、用8位字节表示的数据流，然后TCP把数据流分区成适当长度的报文段（通常受该计算机连接的网络的数据链路层的最大传输单元（  MTU）的限制）。之后TCP把结果包传给IP层，由它来通过网络将包传送给接收端实体的TCP层。TCP为了保证不发生丢包，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接收端实体对已成功收到的包发回一个相应的确认（ACK）；如果发送端实体在合理的往返时延（RTT）内未收到确认，那么对应的数据包就被假设为已丢失将会被进行重传。TCP用一个校验和函数来检验数据是否有错误；在发送和接收时都要计算校验和。

**连接建立**

TCP是因特网中的传输层协议，使用三次握手协议建立连接。**当主动方发出SYN连接请求后，等待对方回答SYN+ACK，并最终对对方的 SYN 执行 ACK 确认。**这种建立连接的方法可以防止产生错误的连接，TCP使用的流量控制协议是可变大小的滑动窗口协议。

![MpJfJJ.png](https://s2.ax1x.com/2019/11/05/MpJfJJ.png)

TCP三次握手的过程如下：

1. 客户端主动打开，发送连接请求报文段，将SYN标识位置为1，Sequence Number置为x（TCP规定SYN=1时不能携带数据，x为随机产生的一个值），然后进入SYN_SEND状态
2. 服务器收到SYN报文段进行确认，将SYN标识位置为1，ACK置为1，Sequence Number置为y，Acknowledgment Number置为x+1，然后进入SYN_RECV状态，这个状态被称为半连接状态
3. 客户端再进行一次确认，将ACK置为1（此时不用SYN），Sequence Number置为x+1，Acknowledgment Number置为y+1发向服务器，最后客户端与服务器都进入ESTABLISHED状态

三次握手完成，TCP客户端和服务器端成功地建立连接，可以开始传输数据了。

**为什么在第3步中客户端还要再进行一次确认呢？**

这主要是**为了防止已经失效的连接请求报文段突然又传回到服务端而产生错误的场景：所谓"已失效的连接请求报文段"是这样产生的。**正常来说，客户端发出连接请求，但因为连接请求报文丢失而未收到确认。于是客户端再次发出一次连接请求，后来收到了确认，建立了连接。数据传输完毕后，释放了连接，客户端一共发送了两个连接请求报文段，其中第一个丢失，第二个到达了服务端，没有"已失效的连接请求报文段"。

现在假定一种异常情况，即客户端发出的第一个连接请求报文段并没有丢失，只是在某些网络节点长时间滞留了，以至于延误到连接释放以后的某个时间点才到达服务端。本来这个连接请求已经失效了，但是服务端收到此失效的连接请求报文段后，就误认为这是客户端又发出了一次新的连接请求。于是服务端又向客户端发出请求报文段，同意建立连接。假定不采用三次握手，那么只要服务端发出确认，连接就建立了。

由于现在客户端并没有发出连接建立的请求，因此不会理会服务端的确认，也不会向服务端发送数据，但是服务端却以为新的传输连接已经建立了，并一直等待客户端发来数据，这样服务端的许多资源就这样白白浪费了。

采用三次握手的办法可以防止上述现象的发生。比如在上述的场景下，客户端不向服务端的发出确认请求，服务端由于收不到确认，就知道客户端并没有要求建立连接。

**连接终止**

建立一个连接需要三次握手，而终止一个连接要经过**四次握手**，这是由TCP的半关闭（half-close）造成的。具体过程如下图所示。

![Mptn9H.png](https://s2.ax1x.com/2019/11/05/Mptn9H.png)

当客户端没有数据再需要发送给服务端时，就需要释放客户端的连接，这整个过程为：

1.  客户端发送一个报文给服务端（没有数据），其中FIN设置为1，Sequence Number置为u，客户端进入FIN_WAIT_1状态
2.  服务端收到来自客户端的请求，发送一个ACK给客户端，Acknowledge置为u+1，同时发送Sequence Number为v，服务端年进入CLOSE_WAIT状态
3.  服务端发送一个FIN给客户端，ACK置为1，Sequence置为w，Acknowledge置为u+1，用来关闭服务端到客户端的数据传送，服务端进入LAST_ACK状态
4.  客户端收到FIN后，进入TIME_WAIT状态，接着发送一个ACK给服务端，Acknowledge置为w+1，Sequence Number置为u+1，最后客户端和服务端都进入CLOSED状态


注意：

1. “通常”是指，某些情况下，步骤1的FIN随数据一起发送，另外，步骤2和步骤3发送的分节都出自执行被动关闭那一端，有可能被合并成一个分节。

2. 在步骤2与步骤3之间，从执行被动关闭一端到执行主动关闭一端流动数据是可能的，这称为“半关闭”（half-close）。

3. 当一个Unix进程无论自愿地（调用exit或从main函数返回）还是非自愿地（收到一个终止本进程的信号）终止时，所有打开的描述符都被关闭，这也导致仍然打开的任何TCP连接上也发出一个FIN。

无论是客户还是服务器，任何一端都可以执行主动关闭。通常情况是，客户执行主动关闭，但是某些协议，例如，HTTP/1.0却由服务器执行主动关闭。

**生命周期**

![MpNdiD.png](https://s2.ax1x.com/2019/11/05/MpNdiD.png)

为什么建链接要3次握手，断链接需要4次挥手？

- **对于建链接的3次握手，**主要是要初始化Sequence Number 的初始值。通信的双方要互相通知对方自己的初始化的Sequence Number（缩写为ISN：Inital Sequence Number）——所以叫SYN，全称Synchronize Sequence Numbers。也就上图中的 x 和 y。这个号要作为以后的数据通信的序号，以保证应用层接收到的数据不会因为网络上的传输的问题而乱序（TCP会用这个序号来拼接数据）。

- **对于4次挥手，**其实你仔细看是2次，因为TCP是全双工的，所以，发送方和接收方都需要Fin和Ack。只不过，有一方是被动的，所以看上去就成了所谓的4次挥手。如果两边同时断连接，那就会就进入到CLOSING状态，然后到达TIME_WAIT状态。下图是双方同时断连接的示意图（你同样可以对照着TCP状态机看）：

![MpayUf.png](https://s2.ax1x.com/2019/11/05/MpayUf.png)

#### Socket

当我们想要在Java中使用TCP/IP通过网络连接到服务器时，就需要创建java.net.Socket对象并连接到服务器。假如希望使用Java NIO，也可以创建Java NIO中的SocketChannel对象。socket属于传输层。

![socket(1).jpg](https://i.loli.net/2019/04/15/5cb3e45bc699f.jpg)

socket是基于应用服务与TCP/IP通信之间的一个抽象，他将TCP/IP协议里面复杂的通信逻辑进行分装，对用户来说，只要通过一组简单的API就可以实现网络的连接。借用网络上一组socket通信图给大家进行详细讲解：

![1.png](https://i.loli.net/2019/06/26/5d136b17bdf6442599.png)

首先，服务端初始化ServerSocket，然后对指定的端口进行绑定，接着对端口及进行监听，通过调用accept方法阻塞，此时，如果客户端有一个socket连接到服务端，那么服务端通过监听和accept方法可以与客户端进行连接。

##### 创建Socket

下面的示例代码是连接到IP地址为127.0.0.1 服务器上的80端口，这台服务器就是我们的Web服务器，而80端口就是Web服务端口。
```
Socket socket = new Socket("127.0.0.1", 80);
```
我们也可以像如下示例中使用域名代替IP地址：
```
Socket socket = new Socket("localhost", 80);
```
Socket 类有五个构造方法.

| **序号** | **方法描述**                                                 |
| -------- | ------------------------------------------------------------ |
| 1        | **public Socket(String host, int port) throws UnknownHostException, IOException.**  				创建一个流套接字并将其连接到指定主机上的指定端口号。 |
| 2        | **public Socket(InetAddress host, int port) throws IOException**  				创建一个流套接字并将其连接到指定 IP 地址的指定端口号。 |
| 3        | **public Socket(String host, int port, InetAddress localAddress, int localPort) throws IOException.**  				创建一个套接字并将其连接到指定远程主机上的指定远程端口。 |
| 4        | **public Socket(InetAddress host, int port, InetAddress localAddress, int localPort) throws IOException.**  				创建一个套接字并将其连接到指定远程地址上的指定远程端口。 |
| 5        | **public Socket()**  				通过系统默认类型的 SocketImpl 创建未连接套接字 |

当 Socket 构造方法返回，并没有简单的实例化了一个 Socket 对象，它实际上会尝试连接到指定的服务器和端口。

下面列出了一些感兴趣的方法，注意客户端和服务器端都有一个 Socket 对象，所以无论客户端还是服务端都能够调用这些方法。 

| **序号** | **方法描述**                                                 |
| -------- | ------------------------------------------------------------ |
| 1        | **public void connect(SocketAddress host, int timeout) throws IOException**  				将此套接字连接到服务器，并指定一个超时值。 |
| 2        | **public InetAddress getInetAddress()**  				 返回套接字连接的地址。 |
| 3        | **public int getPort()**  				返回此套接字连接到的远程端口。 |
| 4        | **public int getLocalPort()**  				返回此套接字绑定到的本地端口。 |
| 5        | **public SocketAddress getRemoteSocketAddress()**  				返回此套接字连接的端点的地址，如果未连接则返回 null。 |
| 6        | **public InputStream getInputStream() throws IOException**  				返回此套接字的输入流。 |
| 7        | **public OutputStream getOutputStream() throws IOException**  				返回此套接字的输出流。 |
| 8        | **public void close() throws IOException**  				关闭此套接字。 |

##### Socket发送数据

要通过Socket发送数据，我们需要获取Socket的输出流（OutputStream），示例代码如下：

```java
Socket socket = new Socket("127.0.0.1", 80);
OutputStream out = socket.getOutputStream();

out.write("some data".getBytes());
out.flush();
out.close();

socket.close();
```
代码非常简单，但是想要通过网络将数据发送到服务器端，一定不要忘记调用flush()方法。操作系统底层的TCP/IP实现会先将数据放入一个更大的数据缓存块中，而缓存块的大小是与TCP/IP的数据包大小相适应的。（注：调用flush()方法只是将数据写入操作系统缓存中，并不保证数据会立即发送）

##### Socket读取数据

从Socket中读取数据，我们就需要获取Socket的输入流（InputStream），代码如下：
```java
Socket socket = new Socket("127.0.0.1", 80);
InputStream in = socket.getInputStream();

int data = in.read();
//... read more data...

in.close();
socket.close();
```
代码也并不复杂，但需要注意的是，从Socket的输入流中读取数据并不能读取文件那样，一直调用read()方法直到返回-1为止，因为对Socket而言，只有当服务端关闭连接时，Socket的输入流才会返回-1，而是事实上服务器并不会不停地关闭连接。假设我们想要通过一个连接发送多个请求，那么在这种情况下关闭连接就显得非常愚蠢。

因此，从Socket的输入流中读取数据时我们必须要知道需要读取的字节数，这可以通过让服务器在数据中告知发送了多少字节来实现，也可以采用在数据末尾设置特殊字符标记的方式连实现。

##### 关闭Socket

当使用完Socket后我们必须将Socket关闭，断开与服务器之间的连接。关闭Socket只需要调用Socket.close()方法即可，代码如下：
```java
Socket socket = new Socket("127.0.0.1", 80);
socket.close();
```

注意：**在关闭输出流之后，会自动关闭链接**

示例：

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.net.SocketException;
//从服务器返回相同的字符串
//运行参数：服务器IP 发送的字符串 [端口地址]
public class TCPEchoClient {

    public static void main(String[] args) throws IOException {

        if ((args.length < 2) || (args.length > 3))  // Test for correct # of args
            throw new IllegalArgumentException("Parameter(s): <Server> <Word> [<Port>]");

        String server = args[0];       // Server name or IP address
        // Convert argument String to bytes using the default character encoding
        byte[] data = args[1].getBytes();

        int servPort = (args.length == 3) ? Integer.parseInt(args[2]) : 7;

        // Create socket that is connected to server on specified port
        Socket socket = new Socket(server, servPort);
        System.out.println("Connected to server...sending echo string");

        InputStream in = socket.getInputStream();
        OutputStream out = socket.getOutputStream();

        out.write(data);  // Send the encoded string to the server

        // Receive the same string back from the server
        int totalBytesRcvd = 0;  // Total bytes received so far
        int bytesRcvd;           // Bytes received in last read
        while (totalBytesRcvd < data.length) {
            if ((bytesRcvd = in.read(data, totalBytesRcvd,
                    data.length - totalBytesRcvd)) == -1)
                throw new SocketException("Connection closed prematurely");
            totalBytesRcvd += bytesRcvd;
        }  // data array is full

        System.out.println("Received: " + new String(data));

        socket.close();  // Close the socket and its streams
    }
}
```

有GUI的客户端

```java
import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.io.DataInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
//运行参数：服务器IP 端口号
public class TCPEchoClientGUI extends JFrame {

    public static void main(String[] args) {
        if ((args.length < 1) || (args.length > 2)) {
            throw new IllegalArgumentException("Parameter(s): <Server> [<Port>]");
        }

        String server = args[0]; // Server name or IP address
        int servPort = (args.length == 2) ? Integer.parseInt(args[1]) : 7;

        JFrame frame = new TCPEchoClientGUI(server, servPort);
        frame.setVisible(true);
    }

    public TCPEchoClientGUI(String server, int servPort) {
        super("TCP Echo Client"); // Set the window title
        setSize(500, 500); // Set the window size
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // Set echo send text field
        final JTextField echoSend = new JTextField(2);
        getContentPane().add(echoSend, "South");

        // Set echo replay text area
        final JTextArea echoReply = new JTextArea(8, 20);
        echoReply.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(echoReply);
        getContentPane().add(scrollPane, "Center");

        final Socket socket; // Client socket
        final DataInputStream in; // Socket input stream
        final OutputStream out; // Socket output stream
        try {
            // Create socket and fetch I/O streams
            socket = new Socket(server, servPort);

            in = new DataInputStream(socket.getInputStream());
            out = socket.getOutputStream();
            echoSend.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent event) {
                    if (event.getSource() == echoSend) {
                        byte[] byteBuffer = echoSend.getText().getBytes();
                        try {
                            out.write(byteBuffer);
                            in.readFully(byteBuffer);
                            echoReply.append(new String(byteBuffer) + "\n");
                            echoSend.setText("");
                        } catch (IOException e) {
                            echoReply.append("ERROR\n");
                        }
                    }
                }
            });

            addWindowListener(new WindowAdapter() {
                public void windowClosing(WindowEvent e) {
                    try {
                        socket.close();
                    } catch (Exception exception) {
                    }
                    System.exit(0);
                }
            });
        } catch (IOException exception) {
            echoReply.append(exception.toString() + "\n");
        }
    }
}
```

#### ServerSocket

用java.net.ServerSocket实现java服务通过TCP/IP监听客户端连接，你也可以用Java NIO 来代替java网络标准API，这时候需要用到 ServerSocketChannel。代表服务器套接字，主要功能是等待来自网络上的“请求”，服务器套接字一次可以与一个套接字链接，请求队列连接数默认最大为50。

ServerSocket 的构造方法如下所示。

- ServerSocket()：无参构造方法。
- ServerSocket(int port)：创建绑定到特定端口的服务器套接字。
- ServerSocket(int port,int backlog)：使用指定的 backlog 创建服务器套接字并将其绑定到指定的本地端口。
- ServerSocket(int port,int backlog,InetAddress bindAddr)：使用指定的端口、监听 backlog 和要绑定到本地的 IP 地址创建服务器。

常见方法：
![](https://i.loli.net/2019/04/15/5cb3fe068c5af.png)

##### 创建

以下是一个创建ServerSocket类来监听9000端口的一个简单的代码
```
ServerSocket serverSocket = new ServerSocket(9000);
```
  服务器应用程序通过使用 java.net.ServerSocket 类以获取一个端口,并且侦听客户端请求。

##### 监听

要获取请求的连接需要用ServerSocket.accept()方法。该方法返回一个Socket类，该类具有普通java Socket类的所有特性。代码如下：
```java
ServerSocket serverSocket = new ServerSocket(9000);
 boolean isStopped = false;
 while(!isStopped){   
   Socket clientSocket = serverSocket.accept();    //do something with clientSocket
 }
```
对每个调用了accept()方法的类都只获得一个请求的连接。

另外，请求的连接也只能在线程运行的server中调用了accept()方法之后才能够接受请求。线程运行在server中其它所有的方法上的时候都不能接受客户端的连接请求。所以”接受”请求的线程通常都会把Socket的请求连接放入一个工作线程池中，然后再和客户端连接。

##### 关闭客户端Socket
客户端请求执行完毕，并且不会再有该客户端的其它请求发送过来的时候，就需要关闭Socket连接，这和关闭一个普通的客户端Socket连接一样。如下代码来执行关闭：
```
socket.close();
```
##### 关闭服务端Sockets

要关闭服务的时候需要关掉 ServerSocket连接。通过执行如下代码：
```
serverSocket.close();
```

**socket 只要在 io流close的情况下 自动关闭，意思就是你想边发送边接受最正确的方式就是发送和 接受的操作都做完之后 再一起关闭IO流 完美解决。**

示例

```java

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketAddress;
//运行参数：端口号
public class TCPEchoServer {

    private static final int BUFSIZE = 32;   // Size of receive buffer

    public static void main(String[] args) throws IOException {

        if (args.length != 1)  // Test for correct # of args
            throw new IllegalArgumentException("Parameter(s): <Port>");

        int servPort = Integer.parseInt(args[0]);

        // Create a server socket to accept client connection requests
        ServerSocket servSock = new ServerSocket(servPort);

        int recvMsgSize;   // Size of received message
        byte[] receiveBuf = new byte[BUFSIZE];  // Receive buffer

        while (true) { // Run forever, accepting and servicing connections
            Socket clntSock = servSock.accept();     // Get client connection

            SocketAddress clientAddress = clntSock.getRemoteSocketAddress();
            System.out.println("Handling client at " + clientAddress);

            InputStream in = clntSock.getInputStream();
            OutputStream out = clntSock.getOutputStream();


            // Receive until client closes connection, indicated by -1 return
            while ((recvMsgSize = in.read(receiveBuf)) != -1) {
                out.write(receiveBuf, 0, recvMsgSize);
                System.out.print(new String(receiveBuf,0,recvMsgSize));
            }
            System.out.println();

            clntSock.close();  // Close the socket.  We are done with this client!
        }
        /* NOT REACHED */
    }
}

```

#### 简易聊天室

网址：<https://github.com/hanyunpeng0521/chat>


### UDP

在有些应用程序中，保持最快的速度比保证每一位数据都正确到达更重要。例如，在实时音频或视频中，丢失数据包只会作为干扰出现。干扰是可以容忍的，但当TCP请求重传或等待数据包到达而它却迟迟不到时，音频流中就会出现尴尬的停顿，这让人无法接受的。在其他应用中，可以在应用层实现可靠性传输。例如：如果客户端向服务器发送一个短的UDP请求，倘若制定时间内没有响应返回，它会认为这个包已丢失。

域名系统就是采用这样的工作方式。事实上，可以用UDP实现一个可靠的文件传输协议，而且很多人确实是这样做的：网络文件系统，简单FTP都使用了UDP协议。在这些协议中由应用程序来负责可靠性。

java中的UDP实现分为两个类：DatagramPacket和 DatagramSocket。DatagramPacket类将数据字节填充到UDP包中，这称为数据报。 DatagramSocket来发送这个包。要接受数据，可以从DatagramSocket中接受一个 DatagramPack对象，然后从该包中读取数据的内容。

![2.jpeg](https://i.loli.net/2019/06/26/5d136fbeae88d85978.jpeg)

这种职责的划分与TCP使用的Socket和ServerSocket有所不同。

首先，UDP没有两台主机间唯一连接的概念，它不需要知道对方是哪个远程主机。它可以从一个端口往多个主机发送信息，但是TCP是无法做到的。

其次，TCP socket把网络链接看作是流：通过从Socket得到的输入和输出流来收发数据。UDP不支持这一点，你处理总是单个数据包。填充在一个数据报中的所有数据会以包的形式进行发送，这些数据要么作为一个组要么全部接收，要么全部丢弃。一个包不一定与下一个包相关。给定两个包，没有办法知道哪个先发哪个后发。

对于流来说，必须提供数据的有序队列，与之不同，数据报会尽可能快的蜂拥到接收方。

#### DatagramSocket

Java使用DatagramSocket代表UDP协议的Socket，DatagramSocket本身只是码头，不维护状态，不能产生IO流，它的唯一作用就是**接收和发送数据报**，Java使用DatagramPacket来代表数据报，DatagramSocket接收和发送的数据都是通过DatagramPacket对象完成的。

先看一下DatagramSocket的构造器:

![1.png](https://i.loli.net/2019/06/26/5d1370875cc1936604.png)

通过上面几个构造器中的任意一个构造器即可创建一个DatagramSocket实例，通常在创建服务器时，创建指定端口的DatagramSocket实例--这样保证其他客户端可以将数据发送到该服务器。

一旦得到了DatagramSocket实例之后，就可以通过如下两个方法来接收和发送数据。

```
receive(DatagramPacket p)：从该DatagramSocket中接收数据报。
send(DatagramPacket p)：以该DatagramSocket对象向外发送数据报。
```
从上面两个方法可以看出，使用DatagramSocket发送数据报时，DatagramSocket并不知道将该数据报发送到哪里，而是由DatagramPacket自身决定数据报的目的地。就像码头并不知道每个集装箱的目的地，码头只是将这些集装箱发送出去，而集装箱本身包含了该集装箱的目的地

**常用方法**

![2.png](https://i.loli.net/2019/06/26/5d1370880e6c179688.png)

#### DatagramPacket

下面看一下DatagramPacket的构造器。
| 构造方法                                                     | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| DatagramPacket(byte[] buf,int length)                        | 构造 DatagramPacket，用来接收长度为 length 的数据包。        |
| DatagramPacket(byte[] buf,int offset,  int length)           | 构造 DatagramPacket，用来接收长度为 length 的包，在缓  冲区中指定了偏移量。 |
| DatagramPacket(byte[] buf,int length,  InetAddress address,int port) | 构造 DatagramPacket，用来将长度为 length 的包发送到指  定主机上的指定端口。 |
| DatagramPacket(byte[] buf,int length,  SocketAddress address) | 构造数据报包，用来将长度为 length 的包发送到指定主机上  的指定端口。 |
| DatagramPacket(byte[] buf,int offset,  int length,InetAddress address,int port) | 构造 DatagramPacket，用来将长度为 length 偏移量为 offset  的包发送到指定主机上的指定端口。 |
| DatagramPacket(byte[] buf,int offset,  int length,SocketAddress address) | 构造数据报包，用来将长度为 length、偏移量为 offset 的包发  送到指定主机上的指定端口。 |

当Client/Server程序使用UDP协议时，实际上并没有明显的服务器端和客户端，因为两方都需要先建立一个DatagramSocket对象，用来接收或发送数据报，然后使用DatagramPacket对象作为传输数据的载体。通常固定IP地址、固定端口的DatagramSocket对象所在的程序被称为服务器，因为该DatagramSocket可以主动接收客户端数据。

**常用方法**

| 方法                                             | 说明                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| InetAddress getAddress()                         | 返回某台机器的 IP 地址，此数据报将要发往该机器或者  从该机器接收。 |
| byte[] getData()                                 | 返回数据缓冲区。                                             |
| int getLength()                                  | 返回将要发送或者接收的数据的长度。                           |
| int getOffset()                                  | 返回将要发送或者接收的数据的偏移量。                         |
| int getPort()                                    | 返回某台远程主机的端口号，此数据报将要发往该主机或  者从该主机接收。 |
| getSocketAddress()                               | 获取要将此包发送或者发出此数据报的远程主机的  SocketAddress（通常为 IP地址+端口号）。 |
| void setAddress(InetAddress addr)                | 设置要将此数据报发往的目的机器的IP地址。                     |
| void setData(byte[] buf)                         | 为此包设置数据缓冲区。                                       |
| void setData(byte[] buf,int offset,  int length) | 为此包设置数据缓冲区。                                       |
| void setLength(int length)                       | 为此包设置长度。                                             |
| void setPort(int port)                           | 设置要将此数据报发往的远程主机的端口号。                     |
| void setSocketAddress(SocketAddress  address)    | 设置要将此数据报发往的远程主机的  SocketAddress（通常为 IP地址+端口号）。 |

在接收数据之前，应该采用上面的第一个或第三个构造器生成一个DatagramPacket对象，给出接收数据的字节数组及其长度。然后调用DatagramSocket 的receive()方法等待数据报的到来，receive()将一直等待（该方法会阻塞调用该方法的线程），直到收到一个数据报为止。如下代码所示：

```java
// 创建一个接收数据的DatagramPacket对象  
DatagramPacket packet=new DatagramPacket(buf, 256);  
// 接收数据报  
socket.receive(packet);  
```
在发送数据之前，调用第二个或第四个构造器创建DatagramPacket对象，此时的字节数组里存放了想发送的数据。除此之外，还要给出完整的目的地址，包括IP地址和端口号。发送数据是通过DatagramSocket的send()方法实现的，send()方法根据数据报的目的地址来寻径以传送数据报。如下代码所示：
```java
// 创建一个发送数据的DatagramPacket对象  
DatagramPacket packet = new DatagramPacket(buf, length, address, port);  
// 发送数据报  
socket.send(packet);
```

使用DatagramPacket接收数据时，会感觉DatagramPacket设计得过于烦琐。开发者只关心该DatagramPacket能放多少数据，而DatagramPacket是否采用字节数组来存储数据完全不想关心。

但Java要求创建接收数据用的DatagramPacket时，必须传入一个空的字节数组，该数组的长度决定了该DatagramPacket能放多少数据，这实际上暴露了DatagramPacket的实现细节。

接着DatagramPacket又提供了一个getData()方法，该方法又可以返回Datagram Packet对象里封装的字节数组，该方法更显得有些多余--如果程序需要获取DatagramPacket里封装的字节数组，直接访问传给 DatagramPacket构造器的字节数组实参即可，无须调用该方法。

当服务器端（也可以是客户端）接收到一个DatagramPacket对象后，如果想向该数据报的发送者"反馈"一些信息，但由于UDP协议是面向非连接的，所以接收者并不知道每个数据报由谁发送过来，但程序可以调用DatagramPacket的如下3个方法来获取发送者的IP地址和端口。
```
InetAddress getAddress()
//：当程序准备发送此数据报时，该方法返回此数据报的目标机器的IP地址；当程序刚接收到一个数据报时，该方法返回该数据报的发送主机的IP地址。

int getPort()
//：当程序准备发送此数据报时，该方法返回此数据报的目标机器的端口；当程序刚接收到一个数据报时，该方法返回该数据报的发送主机的端口。

SocketAddress getSocketAddress()
//：当程序准备发送此数据报时，该方法返回此数据报的目标SocketAddress；当程序刚接收到一个数据报时，该方法返回该数据报的发送主机的SocketAddress。
```
getSocketAddress()方法的返回值是一个SocketAddress对象，该对象实际上就是一个IP地址和一个端口号。也就是说，SocketAddress对象封装了一个InetAddress对象和一个代表端口的整数，所以使用SocketAddress对象可以同时代表IP地址和端口。

#### 示例

![1.jpeg](https://i.loli.net/2019/06/26/5d136f23aee5a23715.jpeg)

**客户端**

客户端五步走：

1. 初始化DatagramSocket

2. 创建用于发送消息的DatagramPacket
3. 向服务端发送消息
4. 创建用于接收消息的DatagramPacket
5. 接收服务端消息

```java

import java.io.IOException;
import java.io.InterruptedIOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
//运行参数：服务器IP 发送的字符串 [端口地址]
public class UDPEchoClientTimeout {

    private static final int TIMEOUT = 3000;   // Resend timeout (milliseconds)
    private static final int MAXTRIES = 5;     // Maximum retransmissions

    public static void main(String[] args) throws IOException {

        if ((args.length < 2) || (args.length > 3)) { // Test for correct # of args
            throw new IllegalArgumentException("Parameter(s): <Server> <Word> [<Port>]");
        }
        InetAddress serverAddress = InetAddress.getByName(args[0]);  // Server address
        // Convert the argument String to bytes using the default encoding
        byte[] bytesToSend = args[1].getBytes();

        int servPort = (args.length == 3) ? Integer.parseInt(args[2]) : 7;

        DatagramSocket socket = new DatagramSocket();

        socket.setSoTimeout(TIMEOUT);  // Maximum receive blocking time (milliseconds)

        DatagramPacket sendPacket = new DatagramPacket(bytesToSend,  // Sending packet
                bytesToSend.length, serverAddress, servPort);

        DatagramPacket receivePacket =                              // Receiving packet
                new DatagramPacket(new byte[bytesToSend.length], bytesToSend.length);

        int tries = 0;      // Packets may be lost, so we have to keep trying
        boolean receivedResponse = false;
        do {
            socket.send(sendPacket);          // Send the echo string
            try {
                socket.receive(receivePacket);  // Attempt echo reply reception

                if (!receivePacket.getAddress().equals(serverAddress)) {// Check source
                    throw new IOException("Received packet from an unknown source");
                }
                receivedResponse = true;
            } catch (InterruptedIOException e) {  // We did not get anything
                tries += 1;
                System.out.println("Timed out, " + (MAXTRIES - tries) + " more tries...");
            }
        } while ((!receivedResponse) && (tries < MAXTRIES));

        if (receivedResponse) {
            System.out.println("Received: " + new String(receivePacket.getData()));
        } else {
            System.out.println("No response -- giving up.");
        }
        socket.close();
    }
}

```
**服务器类**

服务端五步走：

1. 初始化DatagramSocket，指定端口号
2. 创建用于接收消息的DatagramPacket，指定接收数据大小
3. 接收客户端消息
4. 创建用于发送消息的DatagramPacket
5. 向客户端发送消息

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;

public class UDPEchoServer {

    private static final int ECHOMAX = 255; // Maximum size of echo datagram

    public static void main(String[] args) throws IOException {

        if (args.length != 1) { // Test for correct argument list
            throw new IllegalArgumentException("Parameter(s): <Port>");
        }

        int servPort = Integer.parseInt(args[0]);

        DatagramSocket socket = new DatagramSocket(servPort);
        DatagramPacket packet = new DatagramPacket(new byte[ECHOMAX], ECHOMAX);

        while (true) { // Run forever, receiving and echoing datagrams
            socket.receive(packet); // Receive packet from client
            System.out.println("Handling client at " + packet.getAddress().getHostAddress() + " on port " + packet.getPort());
            socket.send(packet); // Send the same packet back to client
            packet.setLength(ECHOMAX); // Reset length to avoid shrinking buffer
        }
        /* NOT REACHED */
    }
}

```


### Protocol Design

如果设计一个客户端到服务器的系统，那么同时也需要设计客户端和服务器之间的通信协议。当然，有时候协议已经为你决定好了，比如HTTP、XML_RPC（http response 的 body 使用xml）、或者SOAP(也是http response 的 body 使用xml)。设计客户端到服务端协议的时候，一旦协议决定开启一会儿，来看一些你必须考虑的地方：

1. 客户端到服务端的往返通讯

2. 区分请求结束和响应结束。

3. 防火墙穿透

#### 客户端-服务端往返

当客户端和服务端通信，执行操作时，他们在交换信息。比如，客户端执行一个服务请求，服务端尝试完成这个请求，发回响应告诉客户端结果。这种客户端和服务端的信息交换就叫做往返。示意图如下：

![](https://i.loli.net/2019/04/15/5cb3ef546fa07.png)

当一个计算机（客户端或者服务端）在网络中发送数据到另一个计算机时，从数据发送到另一端接收数据完会花费一定时间。这就是数据在网络间的传送的时间花费。这个时间叫做延迟。

协议中含有越多的往返，协议变得越慢，延迟特别高。HTTP协议只包含一个单独的响应来执行服务。换句话说就是一个单独的往返。另一方面，在一封邮件发送前，SMTP协议包含了几个客户端和服务端的往返。

在协议中有多个往返的原因是：有大量的数据从客户端发送到服务端。这种情况下你有2个选择：

1. 在分开往返中发送头信息；

2. 将消息分成更小的数据块。

如果服务端能完成头信息的一些初始验证 ，那么分开发送头信息是很明智的。如果头信息是空白的，发送大量数据本身就是浪费资源。

在传输大量数据时，如果网络连接失败了，得从头开始重新发送数据。数据分割发送时，只需要在网络连接失败处重新发送数据块。已经发送成功的数据块不需要重新发送。

#### 区分请求结束和响应结束

如果协议容许在同一个连接中发送多个请求，需要一个让服务端知道当前请求何时结束、下一个请求何时开始。客户端也需要知道一个响应何时结束了，下一个响应何时开始。

对于请求有2个方法区分结束：

1.在请求的开始处发送请求的字长

2.在请求数据的最后发送一个结束标记。

HTTP用第一个机制。在请求头中 发送了“Content-Length”。请求头会告诉服务端在头文件后有多少字节是属于请求的。

这个模型的优势在于没有请求结束标志的开销。为了避免数据看上去像请求结束标志，也不需要对数据体进行编码。

第一个方法的劣势：在数据传输前，发送者必须知道多少字节数将被传输。如果数据时动态生成的，在发送前，首先你得缓存所有的数据，这样才能计算出数据的字节数。

运用请求结束标志时，不需要知道发送了多少字节数。只需要知道请求结束标志在数据的末尾。当然，必须确认已发送的数据中不包含会导致请求结束标志错误的数据。可以这样做：

可以说请求结束标志是字节值255。当然数据可能包含值255。因此，对数据中包含值255的每一个字节添加一个额外的字节，还有值255。结束请求标志被从字节值255到255之后的值为0。如下编码：

255 in data –>255， 255

end-of-request –> 255, 0

这种255，0的序列永远不会出现在数据中，因为你把所有的值255变成了255,255。同时，255,255,0也不会被错认为255,0。255,255被理解成在一起的，0是单独的。

#### 防火墙穿透

比起HTTP协议，大多数防火墙会拦截所有的其他通信。因此把协议放在HTTP的上层是个好方法，像XML-RPC,SOAP和REST也可以这样做。

协议置于HTTP的上层，在客户端和服务端的HTTP请求和响应中可以来回发送数据。记住，HTTP请求和响应不止包含text或者HTML。也可以在里面发送二进制数据。

将请求放置在HTTP协议上，唯一有点奇怪的是：HTTP请求必须包含一个“主机”头字段。如果你在HTTP协议上设计P2P协议，同样的人最可能不会运行多个“主机”。在这种情况下需要头字段是不必要的开销（但是个小开销）。

### http协议

HTTP是无状态的，它的底层协议是由状态的TCP，但是HTTP的一次完整协议动作，里面是使用有状态的TCP协议来完成的。而每次协议动作之间没有任何关系。例如：第7次请求HTTP协议包，并不知道，这个包是为了什么？它或许是因为上次没有请求成功而重传，或许是上次的后续请求，或许是其他的，这些HTTP自身都不知道。

参考：https://blog.csdn.net/wu1991924/article/details/8548051

根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。

![](https://i.loli.net/2019/06/27/5d14a9053af7165904.png)

### 面试

1. 网络编程时的同步、异步、阻塞、非阻塞？

   - 同步：函数调用在没得到结果之前，没有调用结果，不返回任何结果。

   - 异步：函数调用在没得到结果之前，没有调用结果，返回状态信息。

   - 阻塞：函数调用在没得到结果之前，当前线程挂起。得到结果后才返回。

   - 非阻塞：函数调用在没得到结果之前，当前线程不会挂起，立即返回结果。

2. Java如何实现无阻塞方式的Socket编程？

   - NIO有效解决了多线程服务器存在的线程开销问题。

   - 在NIO中使用多线程主要目的不是为了应对每个客户端请求而分配独立的服务线程，

   - 而是通过多线程充分利用多个CPU的处理能力和处理中的等待时间，达到提高服务能力的目的。

3. 什么是java 的序列化(串行化)？

   简单说就是为了保存在内存中的各种对象的状态（也就是实例变量，不是方法），

   并且可以把保存的对象状态再读出来。虽然你可以用你自己的各种各样的方法来保存object states，但是Java给你提供一种应该比你自己好的保存对象状态的机制，那就是序列化。

4. 什么情况下需要序列化？序列化的注意事项，如何实现java 序列化(串行化)？

   - 当你想把的内存中的对象状态保存到一个文件中或者数据库中时候；

   - 当你想用套接字在网络上传送对象的时候；

   - 当你想通过RMI传输对象的时候；

   序列化注意事项

   - 如果子类实现Serializable接口而父类未实现时，父类不会被序列化，但此时父类必须有个无参构造方法，否则会抛InvalidClassException异常。

   - 静态变量不会被序列化，那是类的“菜”，不是对象的。串行化保存的是对象的状态，即非静态的属性，即实例变量。不能保存类变量。

   - transient关键字修饰变量可以限制序列化。对于不需要或不应该保存的属性，应加上transient修饰符。要串行化的对象的类必须是公开的（public）。

   - 虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID是否一致，就是 private static final long serialVersionUID = 1L。

   - Java 序列化机制为了节省磁盘空间，具有特定的存储规则，当写入文件的为同一对象时，并不会再将对象的内容进行存储，而只是再次存储一份引用。反序列化时，恢复引用关系。

   - 序列化到同一个文件时，如第二次修改了相同对象属性值再次保存时候，虚拟机根据引用关系知道已经有一个相同对象已经写入文件，因此只保存第二次写的引用，所以读取时，都是第一次保存的对象。

5. java中有几种类型的流？JDK为每种类型的流提供了一些抽象类以供继承，请说出他们分别是哪些类？

   JDK提供的流继承了四大类：

   InputStream(字节输入流)，OutputStream（字节输出流），Reader（字符输入流），Writer（字符输出流）。

   按流向分类：

   输入流: 程序可以从中读取数据的流。
   输出流: 程序能向其中写入数据的流。

   按数据传输单位分类：

   字节流：以字节（8位二进制）为单位进行处理。主要用于读写诸如图像或声音的二进制数据。
   字符流：以字符（16位二进制）为单位进行处理。
   都是通过字节流的方式实现的。字符流是对字节流进行了封装，方便操作。在最底层，所有的输入输出都是字节形式的。
   后缀是Stream是字节流，而后缀是Reader，Writer是字符流。

   按功能分类：

   节点流：从特定的地方读写的流类，如磁盘或者一块内存区域。
   过滤流：使用节点流作为输入或输出。过滤流是使用一个已经存在的输入流或者输出流连接创建的。

6. 用JAVA SOCKET 编程，读服务器几个 字符，再写入本地显示。

   客户端向服务器端发送连接请求后，就被动地等待服务器的响应。

   典型的TCP客户端要经过下面三步操作：

   - 创建一个Socket实例：构造函数向指定的远程主机和端口建立一个TCP连接；
   - 通过套接字的I/O流与服务端通信；
   - 使用Socket类的close方法关闭连接。


   服务端的工作是建立一个通信终端，并被动地等待客户端的连接。

   典型的TCP服务端执行如下两步操作：

   - 创建一个ServerSocket实例并指定本地端口，用来监听客户端在该端口发送的TCP连接请求；
   - 重复执行：
     - 调用ServerSocket的accept（）方法以获取客户端连接，并通过其返回值创建一个Socket实例；
     - 为返回的Socket实例开启新的线程，并使用返回的Socket实例的I/O流与客户端通信；
     - 通信完成后，使用Socket类的close（）方法关闭该客户端的套接字连接。

7. socket通信，以及长连接，分包，连接异常断开的处理。

   参考：http://www.jianshu.com/p/90348ef3f41e

   http://www.cnblogs.com/fuchongjundream/p/3914696.html

8.  socket通信模型的使用，AIO和NIO。

   参考：https://blog.csdn.net/u014401141/article/details/54406195

   https://blog.csdn.net/anxpp/article/details/51512200

9. http中，get post的区别

   - GET在浏览器回退时是无害的，而POST会再次提交请求。
   - GET产生的URL地址可以被Bookmark，而POST不可以。
   - GET请求会被浏览器主动cache，而POST不会，除非手动设置。
   - GET请求只能进行url编码，而POST支持多种编码方式。
   - GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
   - GET请求在URL中传送的参数是有长度限制的，而POST么有。
   - 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
   - GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
   - GET参数通过URL传递，POST放在Request body中。

   GET和POST本质上没有区别

   GET和POST是什么？HTTP协议中的两种发送请求的方法。

   HTTP是什么？HTTP是基于TCP/IP的关于数据如何在万维网中如何通信的协议。

   HTTP的底层是TCP/IP。所以GET和POST的底层也是TCP/IP，也就是说，GET/POST都是TCP链接。GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上是完全行的通的。 

   因此，GET和POST本质上就是TCP链接，并无差别。但是由于HTTP的规定和浏览器/服务器的限制，导致他们在应用过程中体现出一些不同。 

   GET和POST还有一个重大区别，简单的说：

   GET产生一个TCP数据包；POST产生两个TCP数据包。

   对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；

   而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

   因为POST需要两步，时间上消耗的要多一点，看起来GET比POST更有效。因此Yahoo团队有推荐用GET替换POST来优化网站性能。但这是一个坑！跳入需谨慎。为什么？

   1. GET与POST都有自己的语义，不能随便混用。
   2. 据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的TCP在验证数据包完整性上，有非常大的优点。
   3. 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次

10. 说说http,tcp,udp之间关系和区别。

    TCP/IP是个协议组，可分为四个层次：网络接口层、网络层、传输层和应用层。

    在网络层有IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议。

    在传输层中有TCP协议与UDP协议。

    在应用层有HTTP、FTP、TELNET、SMTP、DNS等协议。

    - TCP是传输层的一个协议，基于IP协议，用来传输类似HTTP的信息。如果把IP协议类比为一个“公路”的话，那TCP协议可以看成是在公路上行驶的“卡车”。TCP协议是面向连接的协议，通过三次握手机制，尽量保证连接的可靠性。tcp的链接需要进行三次握手，释放连接需要四次挥手。

    - UDP 用户数据报协议 （User Datagram Protocol） ：

      UDP也是传输层的一个协议。但是与TCP不同的是，UDP不是面向连接的，并不保证传输的可靠性，没有TCP的建立连接的三次握手机制，对于传输效率上面有了提升。

      个人理解：

      这个就比较简单粗暴了，A要给B传数据，然后就直接传了。

    - HTTP 超文本传输协议（HyperText Transfer Protocal）：

      HTTP是在应用层的一个协议，本身就是一个协议，是从Web服务器传输超文本到本地浏览器的传输协议。 

      HTTP协议基于请求\响应模型的，并且是基于TCP协议的。

      HTTP连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。

### 参考：

1. [Java网络教程](http://ifeve.com/java-network/)
2. <https://blog.csdn.net/qq_38950316/article/details/81087809>
