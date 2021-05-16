---
title: Java  NIO Channel和Pipe（三）
date: 2019-04-23 14:18:59
tags:
 - Java
 - NIO
categories:
 - Java
 - 高级
---
### Channel

简单的说，Channel即通道，JDK1.4新引入的NIO概念，一种全新的、极好的Java I/O示例，提供与I/O服务的直接连接。Channel用于在字节缓存和位于Channel另一侧的实体(通常是File或Socket)之间有效的传输数据。

通道(Channel)是一种途径，借助该途径，可以用最小的总开销来访问操作系统本身的I/O服务。缓冲区(Buffer)则是通道内部用来发送和接受消息的端点。
<!--more-->

![继承](https://i.loli.net/2019/04/23/5cbebe5985995.png)


基本上,所有的 IO 在 NIO 中都从一个 Channel 开始。 Channel 有点像流。 数据可以从 Channel 读到 Buffer 中,也可以从 Buffer 写到 Channel 中。这里有个图示:

![](https://i.loli.net/2019/04/23/5cbebe0157cfc.png)

1. 方法
- close()：调用close方法会导致工作在该通道上的线程暂时阻塞；close关闭期间，任何其他调用close方法的线程将会阻塞；如果一个线程工作在该通道上被阻塞并且同时被中断，那么该通道将会关闭
- isOpen()：通道的开关状态

Channel 和 Buffer 有好几种类型。下面是 JAVA NIO 中的一些主要 Channel 的
实现:

- FileChannel 从文件中读写数据。
- DatagramChannel 能通过 UDP 读写网络中的数据。
- SocketChannel 能通过 TCP 读写网络中的数据。
- ServerSocketChannel 可以监听新进来的 TCP 连接,像 Web 服务器那样。对每一个新进来的连接都会创建一个 SocketChannel

正如你所看到的,这些通道涵盖了 UDP 和 TCP 网络 IO,以及文件 IO。

#### FileChannel

FileChannel是线程安全的，只能通过FileInputStream（只读Channel，不能使用write）,FileOutputStream（只写Channel，不能使用read）,RandomAccessFile的getChannel方法获取FileChannel通道，原理是获取到底层操作系统生成的fd(file descriptor)

FileChannel 无法设置为非阻塞模式,它总是运行在阻塞模式下。

FileChannel的position属于共享的，属于底层fd的position，当调用RandomAccessFile的seek()方法调整position，则生成的FileChannel对象的position为seek后的值

- truncate()：用于设置文件的长度size，若设置的size<当前size，则多出的部分会被删除

- force()：强制将全部待定的修改都写入磁盘中的文件上(所有的现代文件系统都会缓存数据和延迟磁盘文件更新以提高性能)【若是force作用于远程文件系统，则不能保证该操作一定能成功，需要验证当前使用的操作系统或文件系统在同步修改方面是否可以依赖】，有一个 boolean 类型的参数,指明是否同时将文件元数据(权限信息等)写到磁盘上。

- transferTo()和transferFrom()：允许将一个通道交叉连接到另一个通道传递数据；

- position() 获取当前位置，如果将位置设置在文件结束符之后,然后试图从文件通道中读取数据,读方法将返回-1 —— 文件结束标志。

- position(long pos),设置当前位置，如果将位置设置在文件结束符之后,然后向通道中写数据,文件将撑大到当前位置并写入数据。这可能导致―文件空洞,磁盘上物理文件中写入的数据间有空隙。

- size() 返回该实例所关联文件的大小

示例：

```(java)
private static void useNio1()
{
  RandomAccessFile aFile=null;
  try {
      aFile=new RandomAccessFile("test.sql","rw");
      FileChannel inChannel = aFile.getChannel();

      ByteBuffer byteBuffer = ByteBuffer.allocate(48);//分配Buffer
        //向buffer写数据
      int r=inChannel.read(byteBuffer); //从通道中读取数据
      //使用put方法写入buffer,put 方法有很多版本,允许你以不同的方式把数据写入到 Buffer 中。例如, 写到一个指定的位置,或者把一个字节数组写入到 Buffer。
      //byteBuffer.put((byte) 127);
      //byteBuffer.putDouble(123.05);
      while (-1!=r)
      {
	  System.out.println("Read:" + r);
	  //flip 方法将 Buffer 从写模式切换到读模式。调用 flip()方法会将 position设回 0,并将 limit 设置成之前 position 的值。
	  //换句话说,position 现在用于标记读的位置,limit 表示之前写进了多少个 byte、char 等 —— 现在能读取多少个 byte、char 等。
	  byteBuffer.flip();
	  while (byteBuffer.hasRemaining())
	  {
	    //读取数据
	    //从 Buffer 读取数据到 Channel;int bytesWritten = inChannel.write(buf);
	    //使用 get()方法从 Buffer 中读取数据。get 方法有很多版本,允许你以不同的方式从 Buffer 中读取数据。
	    //例如,从指定 position 读取,或者从 Buffer 中读取数据到字节数组。
	      System.out.println((char) byteBuffer.get());
	  }
	  //compact()方法将所有未读的数据拷贝到 Buffer 起始处。然后将 position 设到最后一个未读元素正后面。limit 属性依然像 clear()方法一样,设置成capacity。现在 Buffer 准备好写数据了,但是不会覆盖未读的数据。
	  //如果调用的是 clear()方法,position 将被设回 0,limit 被设置成 capacity的值。换句话说,Buffer 被清空了。Buffer 中的数据并未清除,只是这些标记告诉我们可以从哪里开始往 Buffer 里写数据
	  byteBuffer.clear();
	  r=inChannel.read(byteBuffer);
      }
      //用完 FileChannel 后必须将其关闭。
      inChannel.close();

  } catch (FileNotFoundException e) {
      e.printStackTrace();
  } catch (IOException e) {
      e.printStackTrace();
  }finally {
      try {
          aFile.close();
      } catch (IOException e) {
          e.printStackTrace();
      }
  }
}
```
注意 FileChannel.write()是在 while 循环中调用的。因为无法保证 write()方法一次能向 FileChannel 写入多少字节,因此需要重复调用 write()方法,直到 Buffer 中已经没有尚未写入通道的字节。


示例：[ChannelCopy.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/ChannelCopy.java)


**通道之间的数据传输**

在 Java NIO 中,如果两个通道中有一个是 FileChannel,那你可以直接将数据从一个 channel 传输到另外一个 channel。

*transferFrom()*

FileChannel 的 transferFrom()方法可以将数据从源通道传输到FileChannel 中(译者注:这个方法在 JDK 文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中)。下面是一个简单的例子:

```
fromFile=new RandomAccessFile("fromFile.txt","rw");
FileChannel fromChannel = fromFile.getChannel();

toFile=new RandomAccessFile("toFile.txt","rw");
FileChannel toChannel = toFile.getChannel();
long p=0;
long c=fromChannel.size();
toChannel.transferFrom(fromChannel,p, c);
```
方法的输入参数 position 表示从 position 处开始向目标文件写入数据,count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节,则所传输的字节数要小于请求的字节数。

此外要注意,在 SoketChannel 的实现中,SocketChannel 只会传输此刻准备好的数据(可能不足 count 字节)。因此,SocketChannel 可能不会将请求的所有数据(count 个字节)全部传输到 FileChannel 中。

*transferTo()*

transferTo()方法将数据从 FileChannel 传输到其他的 channel 中。下面是一个简单的例子:

```
fromFile=new RandomAccessFile("fromFile.txt","rw");
FileChannel fromChannel = fromFile.getChannel();

toFile=new RandomAccessFile("toFile.txt","rw");
FileChannel toChannel = toFile.getChannel();
long p=0;
long c=fromChannel.size();
fromChannel.transferTo(p, c,toChannel);
```
是不是发现这个例子和前面那个例子特别相似?除了调用方法的 FileChannel对象不一样外,其他的都一样

示例：[ChannelTransfer.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/ChannelTransfer.java)

#### SocketChannel

Java NIO 中的 SocketChannel 是一个连接到 TCP 网络套接字的通道。可以通过以下 2 种方式创建 SocketChannel:
1. 打开一个 SocketChannel 并连接到互联网上的某台服务器。
2. 一个新连接到达 ServerSocketChannel 时,会创建一个 SocketChannel。

```
//打开
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://www.baidu.com", 80));

//当用完 SocketChannel 之后调用 SocketChannel.close()关闭
socketChannel.close();

//要从 SocketChannel 中读取数据,调用一个 read()的方法之一
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = socketChannel.read(buf);

//写数据到 SocketChannel 用的是 SocketChannel.write()方法,该方法以一个 Buffer 作为参数。
String newData = "New String to write to file..." +
System.currentTimeMillis();
//生成 Buffer,并向 Buffer 中写数据
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
//切换 buffer 为读模式
buf.flip();
while(buf.hasRemaining()) {
  channel.write(buf);
}
```
**非阻塞模式**

可以设置 SocketChannel 为非阻塞模式(non-blocking mode).设置之后,就可以在异步模式下调用 connect(), read() 和 write()了。

*connect()*

如果 SocketChannel 在非阻塞模式下,此时调用 connect(),该方法可能在连接建立之前就返回了。为了确定连接是否建立,可以调用 finishConnect()的方法。像这样:

```
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://www.baidu.com", 80));
while(! socketChannel.finishConnect() ){
//wait, or do something else...
}
write()
```
非阻塞模式下,write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用 write()。

非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的 int 返回值,它会告诉你读取了多少字节。

非阻塞模式与选择器搭配会工作的更好,通过将一或多个 SocketChannel 注册到 Selector,可以询问选择器哪个通道已经准备好了读取,写入等。Selector与 SocketChannel 的搭配使用会在后面详讲。

示例：[ConnectAsync.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/ConnectAsync.java)

connect( )和 finishConnect( )方法是互相同步的,并且只要其中一个操作正在进行,任何读或写的方法调用都会阻塞,即使是在非阻塞模式下。如果此情形下您有疑问或不能承受一个读或写操作在某个通道上阻塞,请用 isConnected( )方法测试一下连接状态。

#### ServerSocketChannel

Java NIO 中的 ServerSocketChannel 是一个可以监听新进来的 TCP 连接的通道, 就像标准 IO 中的 ServerSocket 一样。ServerSocketChannel 类。在 java.nio.channels 包中。
```
ServerSocketChannel serverSocketChannel =ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
while(true){
//当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。
SocketChannel socketChannel =serverSocketChannel.accept();
//使用 socketChannel 做一些工作...
}
if(null!=serverSocketChannel){
serverSocketChannel.close();
}
```
**非阻塞模式**

ServerSocketChannel 可以设置成非阻塞模式。在非阻塞模式下, accept() 方法会立刻返回,如果还没有新进来的连接,返回的将是 null。 因此,需要检查返回的 SocketChannel 是否是 null。如:
```
ServerSocketChannel serverSocketChannel =ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);
while(true){
  SocketChannel socketChannel =
  serverSocketChannel.accept();
  if(socketChannel != null){
  //使用 socketChannel 做一些工作...
  }
}
```
示例：[ChannelAccept.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/ChannelAccept.java)

#### DatagramChannel

Java NIO 中的 DatagramChannel 是一个能收发 UDP 包的通道。因为 UDP 是无连接的网络协议,所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

```
//打开
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));

//通过 receive()方法从 DatagramChannel 接收数据,receive()方法会将接收到的数据包内容复制到指定的 Buffer
// 如果 Buffer 容不下收到的数据,多出的数据将被丢弃。
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
channel.receive(buf);

//通过 send()方法从 DatagramChannel 发送数据

String newData = "New String to write to file..."+ System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
int bytesSent = channel.send(buf, new InetSocketAddress("www.baidu.com", 80));
//因为服务端并没有监控这个端口,所以什么也不会发生。也不会通知你发出的数据包是否已收到,因为 UDP 在数据传送方面没有任何保证。

//可以将 DatagramChannel―连接‖到网络中的特定地址的。
//由于 UDP 是无连接的,连接到特定地址并不会像 TCP 通道那样创建一个真正的连接。
//而是锁住DatagramChannel ,让其只能从特定地址收发数据。
channel.connect(new InetSocketAddress("jenkov.com", 80));

//当连接后,也可以使用 read()和 write()方法,就像在用传统的通道一样。只是在数据传送方面没有任何保证。

int bytesRead = channel.read(buf);
int bytesWritten = channel.write(buf);
```
DatagramChannel 对数据报 socket 的连接语义不同于对流 socket 的连接语义。有时候,将数据报对话限制为两方是很可取的。将 DatagramChannel 置于已连接的状态可以使除了它所“连接”到的地址之外的任何其他源地址的数据报被忽略。这是很有帮助的,因为不想要的包都已经被网络层丢弃了,从而避免了使用代码来接收、检查然后丢弃包的麻烦。

当 DatagramChannel 已连接时,使用同样的令牌,您不可以发送包到除了指定给 connect( )方法的目的地址以外的任何其他地址。试图一定要这样做的话会导致一个 SecurityException 异常。我们可以通过调用带 SocketAddress 对象的 connect( )方法来连接一个 DatagramChannel,该SocketAddress 对象描述了 DatagramChannel 远程对等体的地址。如果已经安装了一个安全管理器,那么它会进行权限检查。之后,每次 send/receive 时就不会再有安全检查了,因为来自或去到任何其他地址的包都是不允许的。

已连接通道会发挥作用的使用场景之一是一个客户端/服务器模式、使用 UDP 通讯协议的实时游戏。每个客户端都只和同一台服务器进行会话而希望忽视任何其他来源地数据包。将客户端的DatagramChannel 实例置于已连接状态可以减少按包计算的总开销(因为不需要对每个包进行安全检查)和剔除来自欺骗玩家的假包。服务器可能也想要这样做,不过需要每个客户端都有一个DatagramChannel 对象。

不同于流 socket,数据报 socket 的无状态性质不需要同远程系统进行对话来建立连接状态。没有实际的连接,只有用来指定允许的远程地址的本地状态信息。由于此原因,DatagramChannel 上也就没有单独的 finishConnect( )方法。我们可以使用 isConnected( )方法来测试一个数据报通道的连接状态。

不同于 SocketChannel(必须连接了才有用并且只能连接一次),DatagramChannel 对象可以任意次数地进行连接或断开连接。每次连接都可以到一个不同的远程地址。调用 disconnect( )方法可以配置通道,以便它能再次接收来自安全管理器(如果已安装)所允许的任意远程地址的数据或发送数据到这些地址上。

当一个 DatagramChannel 处于已连接状态时,发送数据将不用提供目的地址而且接收时的源地址也是已知的。这意味着 DatagramChannel 已连接时可以使用常规的 read( )和 write( )方法,包括scatter/gather 形式的读写来组合或分拆包的数据。

read( )方法返回读取字节的数量,如果通道处于非阻塞模式的话这个返回值可能是“0”。write( )方法的返回值同 send( )方法一致:要么返回您的缓冲区中的字节数量,要么返回“0”(如果由于通道处于非阻塞模式而导致数据报不能被发送)。当通道不是已连接状态时调用 read( )或write( )方法,都将产生 NotYetConnectedException 异常。

下面列出了一些选择数据报 socket 而非流 socket 的理由:
```
您的程序可以承受数据丢失或无序的数据。
您希望“发射后不管”(fire and forget)而不需要知道您发送的包是否已接收。
数据吞吐量比可靠性更重要。
您需要同时发送数据给多个接受者(多播或者广播)。
包隐喻比流隐喻更适合手边的任务。
```
使用 DatagramChannel 发送请求到多个地址上的时间服务器

示例：[TimeClient.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/TimeClient.java)

[TimeServer.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/TimeServer.java)

#### AsynchronousFileChannel
在 Java 7 中,AsynchronousFileChannel 被添加到 Java NIO。AsynchronousFileChannel 使读取数据,并异步地将数据写入文件成为可能。

**创建**

```
//您可以通过它的静态方法 open()创建一个 AsynchronousFileChannel。
Path path = Paths.get("data/test.xml");
AsynchronousFileChannel fileChannel =AsynchronousFileChannel.open(path, StandardOpenOption.READ);
```

**读数据**

您可以通过两种方式从 AsynchronousFileChannel 读取数据。读取数据的每一种方法都调用 AsynchronousFileChannel 的 read()方法之一。
```
//从 AsynchronousFileChannel 读取数据的第一种方法是调用返回 Future 的read()方法。
//read()方法的这个版本将 ByteBuffer 作为第一个参数。
//从AsynchronousFileChannel 读取的数据被读入这个 ByteBuffer。
//第二个参数是文件中的字节位置,以便开始读取。
Future<Integer> operation = fileChannel.read(buffer, 0);

//read()方法会立即返回,即使读操作还没有完成。
//通过调用 read()方法返回的Future 实例的 isDone()方法,您可以检查读取操作是否完成。
AsynchronousFileChannel fileChannel =AsynchronousFileChannel.open(path, StandardOpenOption.READ);
ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
Future<Integer> operation = fileChannel.read(buffer, position);
while(!operation.isDone());
buffer.flip();
byte[] data = new byte[buffer.limit()];
buffer.get(data);
System.out.println(new String(data));
buffer.clear();

//从 AsynchronousFileChannel 读取数据的第二种方法是调用 read()方法版本,该方法将一个 CompletionHandler 作为参数。
fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
@Override
public void completed(Integer result, ByteBuffer attachment)
{
System.out.println("result = " + result);
attachment.flip();
byte[] data = new byte[attachment.limit()];
attachment.get(data);
System.out.println(new String(data));
attachment.clear();
}
@Override
public void failed(Throwable exc, ByteBuffer attachment) {
}
});
//一旦读取操作完成,将调用 CompletionHandler 的 completed()方法。
//对于completed()方法的参数传递一个整数,它告诉我们读取了多少字节,以及传递给 read()方法的―附件。
//附件是 read()方法的第三个参数。
//在本例中,它是ByteBuffer,数据也被读取。您可以自由选择要附加的对象。
//如果读取操作失败,则将调用 CompletionHandler 的 failed()方法。

```

**写数据**

就像阅读一样,您可以通过两种方式将数据写入一个AsynchronousFileChannel。写入数据的每一种方法都调用异步文件通道的write()方法之一。

```
//AsynchronousFileChannel 还允许您异步地写数据。
Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
Files.createFile(path);
}
AsynchronousFileChannel fileChannel =AsynchronousFileChannel.open(path,StandardOpenOption.WRITE);
ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
buffer.put("test data".getBytes());
buffer.flip();
Future<Integer> operation = fileChannel.write(buffer, position);
buffer.clear();
while(!operation.isDone());
System.out.println("Write done");

//注意,在此代码生效之前,文件必须已经存在。如果该文件不存在,那么 write()方法将抛出一个 java.nio.file.NoSuchFileException。


//您还可以使用一个 CompletionHandler 将数据写入到AsynchronousFileChannel 中,以告诉您何时完成写入,而不是 Future。

Path path = Paths.get("data/test-write.txt");
if(!Files.exists(path)){
Files.createFile(path);
}
AsynchronousFileChannel fileChannel =
AsynchronousFileChannel.open(path,
StandardOpenOption.WRITE);
ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;
buffer.put("test data".getBytes());
buffer.flip();
fileChannel.write(buffer, position, buffer, new
CompletionHandler<Integer, ByteBuffer>() {
@Override
public void completed(Integer result, ByteBuffer attachment)
{
System.out.println("bytes written: " + result);
}
@Override
public void failed(Throwable exc, ByteBuffer attachment) {
System.out.println("Write failed");
exc.printStackTrace();
}
});

```


#### Scatter/Gather
Java NIO 开始支持 scatter/gather, scatter/gather 用于描述从 Channel (注:Channel 在中文经常翻译为通道)中读取或者写入到 Channel 的操作。

**从 Channel 中分散(scatter)读取**,是指在读操作时将读取的数据写入多个buffer 中。因此,从 Channel 中读取的数据将―分散(scatter)到多个 Buffer中。

![](https://i.loli.net/2019/04/24/5cc0445811e35.png)


**聚集(gather)写入一个 Channel**,是指在写操作时将多个 buffer 的数据写入同一个 Channel,因此,多个 Buffer 中的数据将―聚集(gather)后写入到一个 Channel。

![](https://i.loli.net/2019/04/24/5cc0445839247.png)


scatter/gather 经常用于需要将传输的数据分开处理的场合,例如传输一个由消息头和消息体组成的消息,你可能会将消息体和消息头分散到不同的 buffer中,这样你可以方便的处理消息头和消息体。

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = { header, body };
channel.read(bufferArray);
```
![](https://i.loli.net/2019/04/25/5cc159ca8b751.png)

注意 buffer 首先被插入到数组,然后再将数组作为 channel.read()的输入参数。read()方法按照 buffer 在数组中的顺序将从 channel 中读取的数据写入到 buffer,当一个 buffer 被写满后,channel 紧接着向另一个 buffer 中写。

Scattering Reads 在移动下一个 buffer 前,必须填满当前的 buffer,这也意味着它不适用于动态消息(注: 消息大小不固定 )。换句话说,如果存在消息头和消息体,消息头必须完成填充(例如 128byte),Scattering Reads 才能正常工作。

Gathering Writes 是指数据从多个 buffer 写入到同一个 channel。如下图描述:

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = { header, body };
channel.write(bufferArray);
```
![](https://i.loli.net/2019/04/25/5cc159ca8bd5c.png)

buffers 数组是 write()方法的输入参数,write()方法会按照 buffer 在数组中的顺序,将数据写入到 channel,注意只有 position 和 limit 之间的数据才会被写入。因此,如果一个 buffer 的容量为 128byte,但是仅仅包含 58byte 的数据,那么这 58byte 的数据将被写入到 channel 中。因此与 Scattering Reads相反,Gathering Writes 能较好的处理动态消息。

带 offset 和 length 参数版本的 read( ) 和 write( )方法使得我们可以使用缓冲区阵列的子集缓冲区。这里的 offset 值指哪个缓冲区将开始被使用,而不是指数据的 offset。这里的 length 参数指示要使用的缓冲区数量。举个例子,假设我们有一个五元素的 fiveBuffers 阵列,它已经被初始化并引用了五个缓冲区,下面的代码将会写第二个、第三个和第四个缓冲区的内容:
```
int bytesRead = channel.write (fiveBuffers, 1, 3);
```


### 管道(Pipe)
java.nio.channels中含有一个名为Pipe(管道)的类，作用是使Java进程内部的两个通道(Channel)之间的数据传输;Pipe 有一个 source 通道和一个 sink 通道。数据会被写到 sink 通道,从 source 通道读取。管道是一对循环的通道。

核心知识：

- 调用Pipe.open()方法创建
- Pipe.SourceChannel：负责读，调用Pipe.source()获取
- Pipe.SlinkChannel：负责写，调用Pipe.slink()获取

![](https://i.loli.net/2019/04/24/5cc05c9b29e94.png)

```
//通过 Pipe.open()方法打开管道
Pipe pipe = Pipe.open();

//要向管道写数据,需要访问 sink 通道
Pipe.SinkChannel sinkChannel = pipe.sink();

//通过调用 SinkChannel 的 write()方法,将数据写入 SinkChannel
String newData = "New String to write to file..." +System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
while(buf.hasRemaining()) {
sinkChannel.write(buf);
}

//从管道读取数据,需要访问 source 通道
Pipe.SourceChannel sourceChannel = pipe.source();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
//read()方法返回的 int 值会告诉我们多少字节被读进了缓冲区。

```
管道可以被用来仅在同一个 Java 虚拟机内部传输数据。虽然有更加有效率的方式来在线程之间传输数据,但是使用管道的好处在于封装性。生产者线程和用户线程都能被写道通用的 ChannelAPI 中。根据给定的通道类型,相同的代码可以被用来写数据到一个文件、socket 或管道。选择器可以被用来检查管道上的数据可用性,如同在 socket 通道上使用那样地简单。这样就可以允许单个用户线程使用一个 Selector 来从多个通道有效地收集数据,并可任意结合网络连接或本地工作线程使用。因此,这些对于可伸缩性、冗余度以及可复用性来说无疑都是意义重大的。

Pipes 的另一个有用之处是可以用来辅助测试。一个单元测试框架可以将某个待测试的类连接到管道的“写”端并检查管道的“读”端出来的数据。它也可以将被测试的类置于通道的“读”端并将受控的测试数据写进其中。两种场景对于回归测试都是很有帮助的。

示例：[PipeTest.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/PipeTest.java)

### 通道工具类
一个工具类(java.nio.channels.Channels 的一个稍微重复的名称)定义了几种静态的工厂方法以使通道可以更加容易地同流和读写器互联。

![](https://i.loli.net/2019/04/25/5cc1b32ed93f2.png)
![](https://i.loli.net/2019/04/25/5cc1b32ec0386.png)

回忆一下,常规的流仅传输字节,readers 和 writers 则作用于字符数据。前四行描述了用于连接流、通道的方法。因为流和通道都是运行在字节流基础上的,所以这四个方法直接将流封装在通道上,反之亦然。

Readers 和 Writers 运行在字符的基础上,在 Java 的世界里字符同字节是完全不同的。将一个通道(仅了解字节)连接到一个 reader 或 writer 需要一个中间对话来处理字节/字符(byte/char)阻抗失配。为此,后半部分描述的工厂方法使用了字符集编码器和解码
器。

这些方法返回的包封 Channel 对象可能会也可能不会实现 InterruptibleChannel 接口,它们也可能不是从 SelectableChannel 引申而来。因此,可能无法将这些包封通道同 java.nio.channels包中定义的其他通道类型交换使用。细节是依赖实现的。如果您的程序依赖这些语义,那么请使用操作器实例测试一下返回的通道对象。



参考：
1. [Java NIO](https://pan.baidu.com/s/1uzTW4)
2. [示例代码](http://www.javanio.info/filearea/bookexamples/)
