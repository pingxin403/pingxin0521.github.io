---
title: Java  NIO  Selector和FileLock（四）
date: 2019-04-24 14:18:59
tags:
 - Java
 - NIO
categories:
 - Java
 - 高级
---
>生活就是一系列猛然的醒悟。 ——R. Van Winkle

### Selector
选择器（Selector） 是 SelectableChannle 对象的多路复用器，Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector可使一个单独的线程管理多个 Channel，selector 是非阻塞 IO 的核心。

<!--more-->
![1.jpg](https://i.loli.net/2019/04/24/5cc044583907f.jpg)

#### 构件

**选择器(Selector)**

选择器类管理着一个被注册的通道集合的信息和它们的就绪状态。通道是和选择器一起被注册的,并且使用选择器来更新通道的就绪状态。当这么做的时候,可以选择将被激发的线程挂起,直到有就绪的的通道。

**可选择通道(SelectableChannel)**

这个抽象类提供了实现通道的可选择性所需要的公共方法。它是所有支持就绪检查的通道类的父类。FileChannel 对象不是可选择的,因为它们没有继承 SelectableChannel。所有 socket 通道都是可选择的,包括从管道(Pipe)对象的中获得的通道。SelectableChannel 可以被注册到 Selector 对象上,同时可以指定对那个选择器而言,那种操作是感兴趣的。一个通道可以被注册到多个选择器上,但对每个选择器而言只能被注册一次。

**选择键(SelectionKey)**

选择键封装了特定的通道与特定的选择器的注册关系。选择键对象被SelectableChannel.register( ) 返回并提供一个表示这种注册关系的标记。选择键包含了两个比特集(以整数的形式进行编码),指示了该注册关系所关心的通道操作,以及通道已经准备好的操作。

要使用 Selector,得向 Selector 注册 Channel,然后调用它的 select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回,线程就可以处理这些事件,事件的例子有如新连接进来,数据接收等。

让我们看看 SelectableChannel 的相关 API 方法：

```
public abstract class SelectableChannel
extends AbstractChannel
implements Channel
{
// This is a partial API listing
public abstract SelectionKey register (Selector sel, int ops)
throws ClosedChannelException;
public abstract SelectionKey register (Selector sel, int ops,
Object att)
throws ClosedChannelException;
public abstract boolean isRegistered( );

public abstract SelectionKey keyFor (Selector sel);
public abstract int validOps( );
public abstract void configureBlocking (boolean block)
throws IOException;
public abstract boolean isBlocking( );
public abstract Object blockingLock( );
}
```
用可选择通道的 register( )方法会将它注册到一个选择器上。如果您试图注册一个处于阻塞状态的通道,register( )将抛出未检查的 IllegalBlockingModeException 异常。此外,通道一旦被注册,就不能回到阻塞状态。试图这么做的话,将在调用 configureBlocking( )方法时将抛出IllegalBlockingModeException 异常。

并且,理所当然地,试图注册一个已经关闭的 SelectableChannel 实例的话,也将抛出ClosedChannelException 异常,就像方法原型指示的那样。

在我们进一步了解 register( )和 SelectableChannel 的其他方法之前,让我们先了解一下Selector 类的 API,以确保我们可以更好地理解这种关系:

```
public abstract class Selector
{
public static Selector open( ) throws IOException
public abstract boolean isOpen( );
public abstract void close( ) throws IOException;
public abstract SelectionProvider provider( );
public abstract int select( ) throws IOException;
public abstract int select (long timeout) throws IOException;
public abstract int selectNow( ) throws IOException;
public abstract void wakeup( );
public abstract Set keys( );
public abstract Set selectedKeys( );
}
```
尽管 SelectableChannel 类上定义了 register( )方法,还是应该将通道注册到选择器上,而不是另一种方式。选择器维护了一个需要监控的通道的集合。一个给定的通道可以被注册到多于一个的选择器上,而且不需要知道它被注册了那个Selector对象上 。将register( ) 放 在SelectableChannel 上而不是 Selector 上,这种做法看起来有点随意。它将返回一个封装了两个对象的关系的选择键对象。重要的是要记住选择器对象控制了被注册到它之上的通道的选择过程。
```
public abstract class SelectionKey
{
public static final int OP_READ
public static final int OP_WRITE
public static final int OP_CONNECT
public static final int OP_ACCEPT
public abstract SelectableChannel channel( );
public abstract Selector selector( );
public abstract void cancel( );
public abstract boolean isValid( );
public abstract int interestOps( );
public abstract void interestOps (int ops);
public abstract int readyOps( );
public final boolean isReadable( )
public final boolean isWritable( )
public final boolean isConnectable( )
public final boolean isAcceptable( )
public final Object attach (Object ob)
public final Object attachment( )
}
```
选择器才是提供管理功能的对象,而不是可选择通道对象。选择器对象对注册到它之上的通道执行就绪选择,并管理选择键。

对于键的 interest(感兴趣的操作)集合和 ready(已经准备好的操作)集合的解释是和特定的通道相关的。每个通道的实现,将定义它自己的选择键类。在 register( )方法中构造它并将它传递给所提供的选择器对象。

#### Selector

选择器提供选择执行已经就绪任务的能力，其实现依赖底层操作系统的select()和poll()这两个系统调用，使得Java也能像C或C++提供同时管理多个I/O通道；

- Selector：多个Channel的监听者，通过select()方法实时响应就绪好的通道，keys()方法其返回的是SelectionKey对象
- SelectorChannel：支持就绪检查的通道类的抽象；其可以被注册到Selector上，一个SelectorChannel可以被注册到多个Selector上，但对每个Selector而言每个SelectorChannel只能注册一次
- SelectionKey：代指了通道与选择器之间的注册关系；

通过调用 Selector.open()方法创建一个 Selector,如下:
```
Selector selector = Selector.open();
```
Selector 对象是通过调用静态工厂方法 open( )来实例化的。选择器不是像通道或流(stream)那样的基本 I/O 对象:数据从来没有通过它们进行传递。类方法 open( )向 SPI 发出请求,通过默认的 SelectorProvider 对象获取一个新的实例。通过调用一个自定义的 SelectorProvider对象的 openSelector( )方法来创建一个 Selector 实例也是可行的。您可以通过调用 provider( )方法来决定由哪个 SelectorProvider 对象来创建给定的 Selector 实例。大多数情况下,您不需要关心 SPI;只需要调用 open( )方法来创建新的 Selector 对象。

*向 Selector 注册通道*

为了将 Channel 和 Selector 配合使用,必须将 Channel 注册到 Selector 上。通过 SelectableChannel.register()方法来实现,如下:
```
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
```

与 Selector 一起使用时,Channel 必须处于非阻塞模式下。这意味着不能将FileChannel 与 Selector 一起使用,因为 FileChannel 不能切换到非阻塞模式。而套接字通道都可以。

注意 register()方法的第二个参数。这是一个― 兴趣 (interest) 集合,意思是在通过 Selector 监听 Channel 时对什么事件感兴趣。

可以监听四种不同类型的事件:
1. Connect
2. Accept
3. Read
4. Write

Channel 能够触发了一个事件,意思是该事件已经就绪。所以,某个 channel成功连接到另一个服务器称为―连接就绪。一个 server socket channel 准备好接收新进入的连接称为―接收就绪。一个有数据可读的通道可以说是―读就绪。等待写数据的通道可以说是―写就绪。

这四种事件用 SelectionKey 的四个常量来表示:
1. SelectionKey.OP_CONNECT
2. SelectionKey.OP_ACCEPT
3. SelectionKey.OP_READ
4. SelectionKey.OP_WRITE

如果你对不止一种事件感兴趣,那么可以用―位或‖操作符将常量连接起来,如下:
```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```
**SelectionKey**

当向 Selector 注册 Channel 时,register()方法会返回一个SelectionKey 对象。这个对象包含了一些你感兴趣的属性:
- interest 集合
- ready 集合
- Channel
- Selector
- 附加的对象(可选)

*interest 集合*

就像向 Selector 注册通道一节中所描述的,interest 集合是你所选择的感兴趣的事件集合。可以通过 SelectionKey 读写 interest 集合,像这样:
```
int interestSet = selectionKey.interestOps();
boolean isInterestedInAccept= (interestSet &SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet &SelectionKey.OP_CONNECT;
boolean isInterestedInRead= interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite= interestSet &SelectionKey.OP_WRITE;
```
可以看到,用―和操作 interest 集合和给定的 SelectionKey 常量,可以确定某个确定的事件是否在 interest 集合中。

*ready 集合*

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后,你会首先访问这个 ready set。Selection 将在下一小节进行解释。可以这样访问 ready 集合:
```
int readySet = selectionKey.readyOps();
```
可以用像检测 interest 集合那样的方法,来检测 channel 中什么事件或操作已经就绪。但是,也可以使用以下四个方法,它们都会返回一个布尔类型:
```
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```
*Channel + Selector*

从 SelectionKey 访问 Channel 和 Selector 很简单。如下:
```
Channel channel= selectionKey.channel();
Selector selector = selectionKey.selector();
```
我们可以通过SelectionKey对象的cancel()方法来取消特定的注册关系。该方法调用之后，该SelectionKey对象将会被”拷贝”至已取消键的集合中，该键此时已经失效，但是该注册关系并不会立刻终结。在下一次select()时，已取消键的集合中的元素会被清除，相应的注册关系也真正终结。

**为SelectionKey绑定附加对象**

可以将一个对象或者更多信息附着到 SelectionKey 上,这样就能方便的识别某个给定的通道。例如,可以附加 与通道一起使用的 Buffer,或是包含聚集数据的某个对象。使用方法如下:

```
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```
还可以在用 register()方法向 Selector 注册 Channel 的时候附加对象。如:
```
SelectionKey key = channel.register(selector,SelectionKey.OP_READ, theObject);
```
如果要取消该对象，则可以通过该种方式:
```
selectionKey.attach(null);
```

需要注意的是如果附加的对象不再使用，一定要人为清除，因为垃圾回收器不会回收该对象，若不清除的话会成内存泄漏。

一个单独的通道可被注册到多个选择器中，有些时候我们需要通过isRegistered（）方法来检查一个通道是否已经被注册到任何一个选择器上。 通常来说，我们并不会这么做。


**通过 Selector 选择通道**

选择器维护注册过的通道的集合，并且这种注册关系都被封装在SelectionKey当中.

Selector维护的三种类型SelectionKey集合：

- 已注册的键的集合(Registered key set)

所有与选择器关联的通道所生成的键的集合称为已经注册的键的集合。并不是所有注册过的键都仍然有效。这个集合通过 keys() 方法返回，并且可能是空的。这个已注册的键的集合不是可以直接修改的；试图这么做的话将引发java.lang.UnsupportedOperationException。

- 已选择的键的集合(Selected key set)

所有与选择器关联的通道所生成的键的集合称为已经注册的键的集合。并不是所有注册过的键都仍然有效。这个集合通过 keys() 方法返回，并且可能是空的。这个已注册的键的集合不是可以直接修改的；试图这么做的话将引发java.lang.UnsupportedOperationException。

- 已取消的键的集合(Cancelled key set)

已注册的键的集合的子集，这个集合包含了 cancel() 方法被调用过的键(这个键已经被无效化)，但它们还没有被注销。这个集合是选择器对象的私有成员，因而无法直接访问。

>注意： 当键被取消（ 可以通过isValid( ) 方法来判断）时，它将被放在相关的选择器的已取消的键的集合里。注册不会立即被取消，但键会立即失效。当再次调用 select( ) 方法时（或者一个正在进行的select()调用结束时），已取消的键的集合中的被取消的键将被清理掉，并且相应的注销也将完成。通道会被注销，而新的SelectionKey将被返回。当通道关闭时，所有相关的键会自动取消（记住，一个通道可以被注册到多个选择器上）。当选择器关闭时，所有被注册到该选择器的通道都将被注销，并且相关的键将立即被无效化（取消）。一旦键被无效化，调用它的与选择相关的方法就将抛出CancelledKeyException。




一旦向 Selector 注册了一或多个通道,就可以调用几个重载的 select()方法。这些方法返回你所感兴趣的事件(如连接、接受、读或写)已经准备就绪的那些通道。换句话说,如果你对―读就绪的通道感兴趣,select()方法会返回读事件已经就绪的那些通道。

下面是 select()方法:
```
select()阻塞到至少有一个通道在你注册的事件上就绪了。
select(long timeout)和 select()一样,除了最长会阻塞 timeout 毫秒(参数)。
selectNow()不会阻塞,不管什么通道就绪都立刻返回(译者注:此方法执行非阻塞的选择操作。如果自从前一次选择操作后,没有通道变成可选择的,则此方法直接返回零。)。
```
**select()方法返回的 int 值表示有多少通道已经就绪。亦即,自上次调用select()方法后有多少通道变成就绪状态。之前在select（）调用时进入就绪的通道不会在本次调用中被记入，而在前一次select（）调用进入就绪但现在已经不在处于就绪的通道也不会被记入。** 如果调用 select()方法,因为有一个通道变成就绪状态,返回了 1,若再次调用 select()方法,如果另一个通道就绪了,它会再次返回 1。如果对第一个就绪的 channel 没有做任何操作,现在就有两个就绪的通道,但在每次 select()方法调用之间,只有一个通道就绪了。

**selectedKeys()**

一旦调用了 select()方法,并且返回值表明有一个或更多个通道就绪了,然后可以通过调用 selector 的 selectedKeys()方法,访问―已选择键集(selectedkey set)中的就绪通道。如下所示:
```
Set selectedKeys = selector.selectedKeys();
```
当像 Selector 注册 Channel 时,Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该 Selector 的通道。可以通过SelectionKey 的 selectedKeySet()方法访问这些对象

可以遍历这个已选择的键集合来访问就绪的通道。如下:
```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
  SelectionKey key = keyIterator.next();
  if(key.isAcceptable()) {
  // 一个连接被 ServerSocketChannel 接受
  } else if (key.isConnectable()) {
  // 与远程服务器建立了连接
  } else if (key.isReadable()) {
  // 一个 channel 做好了读准备
  } else if (key.isWritable()) {
  // 一个 channel 做好了写准备
  }
keyIterator.remove();
}
```
这个循环遍历已选择键集中的每个键,并检测各个键所对应的通道的就绪事件。

注意每次迭代末尾的 keyIterator.remove()调用。Selector 不会自己从已选择键集中移除 SelectionKey 实例。必须在处理完通道时自己移除。下次该通道变成就绪时,Selector 会再次将其放入已选择键集中。SelectionKey.channel()方法返回的通道需要转型成你要处理的类型,如ServerSocketChannel 或 SocketChannel 等。

**停止选择的方法**

*wakeUp()*

某个线程调用 select()方法后阻塞了,即使没有通道已经就绪,也有办法让其从 select()方法返回。只要让其它线程在第一个线程调用 select()方法的那个对象上调用 Selector.wakeup()方法即可。阻塞在 select()方法上的线程会立马返回。如果有其它线程调用了 wakeup()方法,但当前没有线程阻塞在select()方法上,下个调用 select()方法的线程会立即―醒来(wake up)。

*close()*

用完 Selector 后调用其 close()方法会关闭该 Selector,且使注册到该Selector 上的所有 SelectionKey 实例无效。通道本身并不会关闭。


这里有一个完整的示例,打开一个 Selector,注册一个通道注册到这个Selector 上(通道的初始化过程略去),然后持续监控这个 Selector 的四种事件(接受,连接,读,写)是否就绪。

```
Selector selector = Selector.open();
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,
SelectionKey.OP_READ);
while(true) {
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
  Set<SelectionKey> selectedKeys = selector.selectedKeys();
  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();
  while(keyIterator.hasNext()) {
  SelectionKey key = keyIterator.next();
  if(key.isAcceptable()) {
  // 一个连接被 ServerSocketChannel 接受
  } else if (key.isConnectable()) {
  // 与远程服务器建立了连接
  } else if (key.isReadable()) {
  // 一个 channel 做好了读准备
  } else if (key.isWritable()) {
  // 一个 channel 做好了写准备
  }
  keyIterator.remove();
  }
}
```

### 非阻塞式服务器
即使你知道 Java NIO 非阻塞的工作特性(如 Selector,Channel,Buffer 等组件),但是想要设计一个非阻塞的服务器仍然是一件很困难的事。非阻塞式服务器相较于阻塞式来说要多上许多挑战。本节将会讨论非阻塞式服务器的主要几个难题,并针对这些难题给出一些可能的解决方案。

源码：https://github.com/hanyunpeng0521/java-nio-server

**非阻塞式 IO 管道(Pipelines)**

一个非阻塞式 IO 管道是由各个处理非阻塞式 IO 组件组成的链。其中包括读/写IO。下图就是一个简单的非阻塞式 IO 管道组成

![](https://i.loli.net/2019/04/24/5cc04e40d6623.png)


一个组件使用 Selector 监控 Channel 什么时候有可读数据。然后这个组件读取输入并且根据输入生成相应的输出。最后输出将会再次写入到一个 Channel 中。

一个非阻塞式 IO 管道不需要将读数据和写数据都包含,有一些管道可能只会读数据,另一些可能只会写数据。

上图仅显示了一个单一的组件。一个非阻塞式 IO 管道可能拥有超过一个以上的组件去处理输入数据。一个非阻塞式管道的长度是由他的所要完成的任务决定。

一个非阻塞 IO 管道可能同时读取多个 Channel 里的数据。举个例子:从多个SocketChannel 管道读取数据。

其实上图的控制流程还是太简单了。这里是组件从 Selector 开始从 Channel 中读取数据,而不是 Channel 将数据推送给 Selector 进入组件中,即便上图画的就是这样。


**非阻塞式 vs 阻塞式管道**

非阻塞和阻塞 IO 管道两者之间最大的区别在于他们如何从底层Channel(Socket 或者 file)读取数据。

IO 管道通常从流中读取数据(来自 socket 或者 file)并且将这些数据拆分为一系列连贯的消息。这和使用 tokenizer(这里估计是解析器之类的意思)将数据流解析为 token(这里应该是数据包的意思)类似。相反你只是将数据流分解为更大的消息体。我将拆分数据流成消息这一组件称为―消息读取器(Message Reader)下面是 Message Reader 拆分流为消息的示意图:

![](https://i.loli.net/2019/04/24/5cc04eabc1650.png)

一个阻塞 IO 管道可以使用类似 InputStream 的接口每次一个字节地从底层Channel 读取数据,并且这个接口阻塞直到有数据可以读取。这就是阻塞式Message Reader 的实现过程。

使用阻塞式 IO 接口简化了 Message Reader 的实现。阻塞式 Message Reader从不用处理在流没有数据可读的情况,或者它只读取流中的部分数据并且对于消息的恢复也要延迟处理的情况。

同样,阻塞式 Message Writer(一个将数据写入流中组件)也从不用处理只有部分数据被写入和写入消息要延迟恢复的情况。

**阻塞式 IO 管道的缺陷**

虽然阻塞式 Message Reader 容易实现,但是也有一个不幸的缺点:每一个要分解成消息的流都需要一个独立的线程。必须要这样做的理由是每一个流的 IO 接口会阻塞,直到它有数据读取。这就意味着一个单独的线程是无法尝试从一个没有数据的流中读取数据转去读另一个流。一旦一个线程尝试从一个流中读取数据,那么这个线程将会阻塞直到有数据可以读取。

如果 IO 管道是必须要处理大量并发链接服务器的一部分的话,那么服务器就需要为每一个链接维护一个线程。对于任何时间都只有几百条并发链接的服务器这确实不是什么问题。但是如果服务器拥有百万级别的并发链接量,这种设计方式就没有良好收放。每个线程都会占用栈 32bit-64bit 的内存。所以一百万个线程占用的内存将会达到 1TB!不过在此之前服务器将会把所有的内存用以处理传过来的消息(例如:分配给消息处理期间使用对象的内存)

为了将线程数量降下来,许多服务器使用了服务器维持线程池(例如:常用线程为 100)的设计,从而一次一个地从入站链接(inbound connections)地读取。

入站链接保存在一个队列中,线程按照进入队列的顺序处理入站链接。这一设计如下图所示:(注:Tomcat 就是这样的)

![](https://i.loli.net/2019/04/24/5cc04fbe58f1c.png)

然而,这一设计需要入站链接合理地发送数据。如果入站链接长时间不活跃,那么大量的不活跃链接实际上就造成了线程池中所有线程阻塞。这意味着服务器响应变慢甚至是没有反应。

一些服务器尝试通过弹性控制线程池的核心线程数量这一设计减轻这一问题。例如,如果线程池线程不足时,线程池可能开启更多的线程处理请求。这一方案意味着需要大量的长时链接才能使服务器不响应。但是记住,对于并发线程数任然是有一个上限的。因此,这一方案仍然无法很好地解决一百万个长时链接。

**基础非阻塞式 IO 管道设计**

一个非阻塞式 IO 管道可以使用一个单独的线程向多个流读取数据。这需要流可以被切换到非阻塞模式。在非阻塞模式下,当你读取流信息时可能会返回 0 个字节或更多字节的信息。如果流中没有数据可读就返回 0 字节,如果流中有数据可读就返回 1+字节。

为了避免检查没有可读数据的流我们可以使用 Java NIO Selector. 一个或多个SelectableChannel 实例可以同时被一个 Selector 注册。

当你调用 Selector的 select()或者 selectNow()方法它只会返回有数据读取的SelectableChannel 的实例. 下图是该设计的示意图:

![](https://i.loli.net/2019/04/24/5cc052c62ad5a.png)

**读取部分消息**

当我们从一个 SelectableChannel 读取一个数据包时,我们不知道这个数据包相比于源文件是否有丢失或者重复数据(原文是:When we read a block of datafrom a SelectableChannel we do not know if that data block contains less ormore than a message)。一个数据包可能的情况有:缺失数据(比原有消息的数据少)、与原有一致、比原来的消息的数据更多(例如:是原来的 1.5 或者 2.5倍)。数据包可能出现的情况如下图所示:

![](https://i.loli.net/2019/04/24/5cc0533f37956.png)

在处理类似上面这样部分信息时,有两个问题:
1. 判断你是否能在数据包中获取完整的消息。
2. 在其余消息到达之前如何处理已到达的部分消息。

判断消息的完整性需要消息读取器(Message Reader)在数据包中寻找是否存在至少一个完整消息体的数据。如果一个数据包包含一个或多个完整消息体,这些消息就能够被发送到管道进行处理。寻找完整消息体这一处理可能会重复多次,因此这一操作应该尽可能的快。

判断消息完整性和存储部分消息都是消息读取器(Message Reader)的责任。为了避免混合来自不同 Channel 的消息,我们将对每一个 Channel 使用一个 Message Reader。设计如下图所示:

![](https://i.loli.net/2019/04/24/5cc053974053e.png)

在从 Selector 得到可从中读取数据的 Channel 实例之后,与该 Channel 相关联的 Message Reader 读取数据并尝试将他们分解为消息。这样读出的任何完整消息可以被传到读取通道(read pipeline)任何需要处理这些消息的组件中。

一个 Message Reader 一定满足特定的协议。Message Reader 需要知道它尝试读取的消息的消息格式。如果我们的服务器可以通过协议来复用,那它需要有能够插入 Message Reader 实现的功能 – 可能通过接收一个 Message Reader 工厂作为配置参数。

**存储部分消息**

现在我们已经确定 Message Reader 有责任存储部分消息,直到收到完整的消息,我们需要弄清楚这些部分消息的存储应该如何实现。

有两个设计因素我们要考虑:
1. 我们想尽可能少地复制消息数据。复制越多,性能越低。
2. 我们希望将完整的消息存储在连续的字节序列中,使解析消息更容易。

**每个 Message Reader 的缓冲区**

很显然部分消息需要存储某些缓冲区中。简单的实现方式可以是每一个 Message Reader 内部简单地有一个缓冲区。但是这个缓冲区应该多大?它要大到足够储存最大允许储存消息。因此,如果最大允许储存消息是 1MB,那么 Message Reader 内部缓冲区将至少需要 1MB。

当我们的链接达到百万数量级,每个链接都使用 1MB 并没有什么作用。1,000,000* 1MB 仍然是 1TB 的内存!那如果最大的消息是 16MB 甚至是 128MB 呢?

**大小可调的缓冲区**

另一个选择是在 Message Reader 内部实现一个大小可调的缓冲区。大小可调的缓冲区开始的时候很小,如果它获取的消息过大,那缓冲区会扩大。这样每一条链接就不一定需要如 1MB 的缓冲区。每条链接的缓冲区只要需要足够储存下一条消息的内存就行了。

有几个可实现可调大小缓冲区的方法。它们都各自有自己的优缺点,所以接下来的部分我将逐个讨论。

*通过复制调整大小*

实现可调大小缓冲区的第一种方式是从一个大小(例如:4KB)的缓冲区开始。如果4KB 的缓冲区装不下一个消息,则会分配一个更大的缓冲区(如:8KB),并将大小为4KB 的缓冲区数据复制到这个更大的缓冲区中去。

通过复制实现大小可调缓冲区的优点在于消息的所有数据被保存在一个连续的字节数组中,这就使得消息的解析更加容易。它的缺点就是在复制更大消息的时候会导致大量的数据。

为了减少消息的复制,你可以分析流进你系统的消息的大小,并找出尽量减少复制量的缓冲区的大小。例如,你可能看到大多数消息都小于 4KB,这是因为它们都仅包含很小的 request/responses。这意味着缓冲区的初始值应该设为 4KB。

然后你可能有一个消息大于 4KB,这通常是因为它里面包含一个文件。你可能注意到大多数流进系统的文件都是小于 128KB 的。这样第二个缓冲区的大小设置为 128KB 就较为合理。

最后你可能会发现一旦消息超过 128KB 之后,消息的大小就没有什么固定的模式,因此缓冲区最终的大小可能就是最大消息的大小。

根据流经系统的消息大小,上面三种缓冲区大小可以减少数据的复制。小于 4KB的消息将不会复制。对于一百万个并发链接其结果是: 1,000,000 * 4KB = 4GB,对于目前大多数服务器还是有可能的。介于 4KB – 128KB 的消息将只会复制一次,并且只有 4KB 的数据复制进 128KB 的缓冲区中。介于 128KB 至最大消息大小的消息将会复制两次。第一次复制 4KB,第二次复制 128KB,所以最大的消息总共复制了 132KB。假设没有那么多超过 128KB 大小的消息那还是可以接受的。

一旦消息处理完毕,那么分配的内存将会被清空。这样在同一链接接收到的下一条消息将会再次从最小缓冲区大小开始算。这样做的必要性是确保了不同连接间内存的有效共享。所有的连接很有可能在同一时间并不需要打的缓冲区。

这有一些示例：https://github.com/hanyunpeng0521/java-resizable-array

*通过追加调整大小*

调整缓冲区大小的另一种方法是使缓冲区由多个数组组成。当你需要调整缓冲区大小时,你只需要另一个字节数组并将数据写进去就行了。

这里有两种方法扩张一个缓冲区。一个方法是分配单独的字节数组,并将这些数组保存在一个列表中。另一个方法是分配较大的共享字节数组的片段,然后保留分配给缓冲区的片段的列表。就个人而言,我觉得片段的方式会好些,但是差别不大。

通过追加单独的数组或片段来扩展缓冲区的优点在于写入过程中不需要复制数据。所有的数据可以直接从 socket (Channel)复制到一个数组或片段中。

以这种方式扩展缓冲区的缺点是在于数据不是存储在单独且连续的数组中。这将使得消息的解析更困难,因为解析器需要同时查找每个单独数组的结尾处和所有数组的结尾处。由于你需要在写入的数据中查找消息的结尾,所以该模型并不容易使用。

**TLV 编码消息**

一些协议消息格式是使用 TLV 格式(类型(Type)、长度(Length)、值(Value))编码。这意味着当消息到达时,消息的总长度被存储在消息的开头。这一方式你可以立即知道应该对整个消息分配多大的内存。

TLV 编码使得内存管理变得更加容易。你可以立即知道要分配多大的内存给这个消息。只有部分在结束时使用的缓冲区才会使得内存浪费。
TLV 编码的一个缺点是你要在消息的所有数据到达之前就分配好这个消息需要的所有内存。一些慢连接可能因此分配完你所有可用内存,从而使得你的服务器无法响应。

此问题的解决方法是使用包含多个 TLV 字段的消息格式。因此,服务器是为每个字段分配内存而不是为整个消息分配内存,并且是字段到达之后再分配内存。然而,一个大消息中的一个大字段在你的内存管理有同样的影响。

另外一个方案就是对于还未到达的信息设置超时时间,例如 10-15 秒。当恰好有许多大消息到达服务器时,这个方案能够使得你的服务器可以恢复,但是仍然会造成服务器一段时间无法响应。另外,恶意的 DoS (Denial of Service 拒绝服务)攻击仍然可以分配完你服务器的所有内存。

TLV 编码存在许多不同的形式。实际使用的字节数、自定字段的类型和长度都依赖于每一个 TLV 编码。TLV 编码首先放置字段的长度、然后是类型、然后是值(一个 LTV 编码)。 虽然字段的顺序不同,但它仍然是 TLV 的一种。

TLV 编码使内存管理更容易这一事实,其实是 HTTP 1.1 是如此可怕的协议的原因之一。 这是他们试图在 HTTP 2.0 中修复数据的问题之一,数据在 LTV 编码帧中传输。 这也是为什么我们使用 TLV 编码的 VStack.co project 设计了我们自己的网络协议。


**写部分数据**

在非阻塞 IO 管道中写数据仍然是一个挑战。当你调用一个处于非阻塞式 Channel对象的 write(ByteBuffer)方法时,ByteBuffer 写入多少数据是无法保证的。write(ByteBuffer)方法会返回写入的字节数,因此可以跟踪写入的字节数。这就是挑战:跟踪部分写入的消息,以便最终可以发送一条消息的所有字节。

为了管理部分消息写入 Channel,我们将创建一个消息写入器(Message Writer)。

就像 Message Reader 一样,每一个要写入消息的 Channel 我们都需要一个Message Writer。在每个 Message Writer 中,我们跟踪正在写入的消息的字节数。

如果达到的消息量超过 Message Writer 可直接写入 Channel 的消息量,消息就需要在 Message Writer 排队。然后 Message Writer 尽快地将消息写入到Channel 中。

![](https://i.loli.net/2019/04/24/5cc058a3a5760.png)

为了使 Message Writer 能够尽快发送数据,Message Writer 需要能够不时被调用,这样就能发送更多的消息。

如果你有大量的连接那你将需要大量的 Message Writer 实例。检查 Message Writer 实例(如:一百万个)看写任何数据时是否缓慢。 首先,许多 Message Writer 实例都没有任何消息要发送,我们并不想检查那些 Message Writer 实例。其次,并不是所有的 Channel 实例都可以准备好写入数据。 我们不想浪费时间尝试将数据写入无法接受任何数据的 Channel。

为了检查 Channel 是否准备好进行写入,您可以使用 Selector 注册 Channel。然而我们并不想将所有的 Channel 实例注册到 Selector 中去。想象一下,如果你有 1,000,000 个连接且其中大多是空闲的,并且所有的连接已经与 Selector注册。然后当你调用 select()时,这些 Channel 实例的大部分将被写入就绪(它们大都是空闲的,记得吗?)然后你必须检查所有这些连接的 Message Writer,以查看他们是否有任何数据要写入。

为了避免检查所有消息的 Message Writer 实例和所有不可能被写入任何信息的Channel 实例,我们使用这两步的方法:
1. 当一个消息被写入 Message Writer,Message Writer 向 Selector 注册其相关Channel(如果尚未注册)。

2. 当你的服务器有时间时,它检查 Selector 以查看哪些注册的 Channel 实例已准备好进行写入。 对于每个写就绪 Channel,请求其关联的 Message Writer将数据写入 Channel。 如果 Message Writer 将其所有消息写入其 Channel,则 Channel 将再次从 Selector 注册。

这两个小步骤确保了有消息写入的 Channel 实际上已经被 Selector 注册了。

**汇总**

正如你所见,一个非阻塞式服务器需要时不时检查输入的消息来判断是否有任何的新的完整的消息发送过来。服务器可能会在一个或多个完整消息发来之前就检查了多次。检查一次是不够的。

同样,一个非阻塞式服务器需要时不时检查是否有任何数据需要写入。如果有,服务器需要检查是否有任何相应的连接准备好将该数据写入它们。只有在第一次排队消息时才检查是不够的,因为消息可能被部分写入。

所有这些非阻塞服务器最终都需要定期执行的三个―管道(pipelines):

1. 读取管道(The read pipeline),用于检查是否有新数据从开放连接进来的。
2. 处理管道(The process pipeline),用于所有任何完整消息。
3. 写入管道(The write pipeline),用于检查是否可以将任何传出的消息写入任何打开的连接。

这三条管道在循环中重复执行。你可能可以稍微优化执行。例如,如果没有排队的消息可以跳过写入管道。 或者,如果我们没有收到新的,完整的消息,也许您可以跳过处理管道。

以下是说明完整服务器循环的图:

![](https://i.loli.net/2019/04/24/5cc05974c9123.png)

如果仍然发现这有点复杂,请记住查看 GitHub 资料库:https://github.com/hanyunpeng0521/java-nio-server

也许看到正在执行的代码可能会帮助你了解如何实现这一点。

**服务器线程模型**

GitHub 资源库里面的非阻塞式服务器实现使用了两个线程的线程模式。第一个线程用来接收来自 ServerSocketChannel 的传入连接。第二个线程处理接受的连接,意思是读取消息,处理消息并将响应写回连接。这两个线程模型的图解如下:

![](https://i.loli.net/2019/04/24/5cc05a875b818.png)



#### C/S通信示例

```
//client端
private static void client()
{
    try {
        //1. 获取socketChannel
        SocketChannel sChannel = SocketChannel.open();
        //2. 创建连接
        sChannel.connect(new InetSocketAddress("127.0.0.1", 9898));
        ByteBuffer buf = ByteBuffer.allocate(1024);
        //3. 设置通道为非阻塞
        sChannel.configureBlocking(false);

        @SuppressWarnings("resource")
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String msg = scanner.nextLine();

            buf.put((new Date() + "：" + msg).getBytes());
            buf.flip();
            //4. 向通道写数据
            sChannel.write(buf);
            buf.clear();
        }

    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

//server端
private static void server()
{
    try {
        //1. 获取服务端通道
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        ssChannel.bind(new InetSocketAddress(9898));
        //2. 设置为非阻塞模式
        ssChannel.configureBlocking(false);

        //3. 打开一个监听器
        Selector selector = Selector.open();
        //4. 向监听器注册接收事件
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (selector.select() > 0) {
            //5. 获取监听器上所有的监听事件值
            Iterator<SelectionKey> it = selector.selectedKeys().iterator();

            //6. 如果有值
            while (it.hasNext()) {
                //7. 取到SelectionKey
                SelectionKey key = it.next();

                //8. 根据key值判断对应的事件
                if (key.isAcceptable()) {
                    //9. 接入处理
                    SocketChannel socketChannel = ssChannel.accept();
                    socketChannel.configureBlocking(false);
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) {
                    //10. 可读事件处理
                    SocketChannel channel = (SocketChannel) key.channel();
                    readMsg(channel);
                }
                //11. 移除当前key
                it.remove();
            }
        }
    } catch (ClosedChannelException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

private static void readMsg(SocketChannel channel) throws IOException {
    ByteBuffer buf = ByteBuffer.allocate(1024);
    int len = 0;
    while ((len = channel.read(buf)) > 0) {
        buf.flip();
        byte[] bytes = new byte[1024];
        buf.get(bytes, 0, len);
        System.out.println(new String(bytes, 0, len));
    }
}
```






### 文件锁（FileLock）

文件锁定机制允许一个进程阻止其他进程存取某文件,或限制其存取方式。通常的用途是控制共享信息的更新方式,或用于事务隔离。在控制多个实体并行访问共同资源方面,文件锁定是必不可少的。数据库等复杂应用严重信赖于文件锁定。

文件锁的对象是文件而不是通道或线程；“文件锁定”从字面上看有锁定整个文件的意思(通常的确是那样),但锁定往往可以发生在更为细微的层面,锁定区域往往可以细致到单个字节。锁定与特定文件相关,开始于文件的某个特定字节地址,包含特定数量的连续字节。这对于协调多个进程互不影响地访问文件不同区域,是至关重要的。

文件锁定有两种方式:共享的和独占的。多个共享锁可同时对同一文件区域发生作用;独占锁则不同,它要求相关区域不能有其他锁定在起作用。获得独占锁的前提是对文件有写权限，获得共享锁只要对文件有读权限即可。


共享锁和独占锁的经典应用,是控制最初用于读取的共享文件的更新。某个进程要读取文件,会先取得该文件或该文件部分区域的共享锁。第二个希望读取相同文件区域的进程也会请求共享锁。两个进程可以并行读取,互不影响。但是,假如有第三个进程要更新该文件,它会请求独占锁。该进程会处于阻滞状态,直到既有锁定(共享的、独占的)全部解除。一旦给予独占锁,其他共享锁的读取进程会处于阻滞状态,直到独占锁解除。这样,更新进程可以更改文件,而其他读取进程不会因为文件的更改得到前后不一致的结果。

![共享锁阻断独占锁请求](https://i.loli.net/2019/04/25/5cc11bee2ac91.png)

![独占锁阻断共享锁请求](https://i.loli.net/2019/04/25/5cc11bee14ee4.png)


如果一个线程在一个文件上获得了独占锁，若运行在同一JVM上，则另一个线程请求文件的独占锁是允许的；若运行在不同的JVM上，则其线程请求文件的独占锁会被阻塞；因为锁最终是由操作系统或文件系统来判优并且几乎总是在进程级别而不是在线程级别上判优

独占锁和共享锁是由底层操作系统或文件系统决定，若底层不支持共享锁，则即使获取锁时的shared参数为true，调用FileLock的isShared()方法也将返回false。文件锁有建议使用和强制使用之分。建议型文件锁会向提出请求的进程提供当前锁定信息,但操作系统并不要求一定这样做,而是由相关进程进行协调并关注锁定信息。多数 Unix 和类 Unix 操作系统使用建议型锁,有些也使用强制型锁或兼而有之。

强制型锁由操作系统或文件系统强行实施,不管进程对锁的存在知道与否,都会阻止其对文件锁定区域的访问。微软的操作系统往往使用的是强制型锁。假定所有文件锁均为建议型,并在访问共同资源的各个应用程序间使用一致的文件锁定,是明智之举,也是唯一可行的跨平台策略。依赖于强制文件锁定的应用程序,从根子上讲就是不可移植的。

FileLock从获取到之后有效，失效条件：

- 调用FileLock中的release()方法
- 它所关联的通道被关闭
- JVM关闭

```
// 这里是FileChannel关于FileLock的API（抛出IOException，这里未列出）

// 获取文件的独占锁（获取操作是阻塞的），等价于调用lock(0,Long.MAX_VALUE , false)
public final FileLock lock()

//指定文件内部锁定区域的开始position以及锁定区域的size，shared标识待获取是共享锁(true)还是独占锁(false)
// 文件锁的范围可以大于文件的大小
public abstract FileLock lock (long position, long size, boolean shared)

// lock方法的非阻塞方式，若是待获取的区域是已经被锁定的，则此时会直接返回null
public final FileLock tryLock()
public abstract FileLock tryLock (long position, long size, boolean shared)
```

锁定区域的范围不一定要限制在文件的 size 值以内,锁可以扩展从而超出文件尾。因此,我们可以提前把待写入数据的区域锁定,我们也可以锁定一个不包含任何文件内容的区域,比如文件最后一个字节以外的区域。如果之后文件增长到达那块区域,那么您的文件锁就可以保护该区域的文件内容了。相反地,如果您锁定了文件的某一块区域,然后文件增长超出了那块区域,那么新增加的文件内容将不会受到您的文件锁的保护。

不带参数的简单形式的 lock( )方法是一种在整个文件上请求独占锁的便捷方法,锁定区域等于它能达到的最大范围。该方法等价于:
```
fileChannel.lock (0L, Long.MAX_VALUE, false);
```
如果您正请求的锁定范围是有效的,那么 lock( )方法会阻塞,它必须等待前面的锁被释放。假如您的线程在此情形下被暂停,该线程的行为受中断语义控制。如果通道被另外一个线程关闭,该暂停线程将恢复并产生一个 AsynchronousCloseException 异常。假如该暂停线程被直接中断(通过调用它的 interrupt( )方法),它将醒来并产生一个FileLockInterruptionException 异常。如果在调用 lock( )方法时线程的 interrupt status 已经被设置,也会产生 FileLockInterruptionException 异常

这两个tryLock( )和 lock( )方法起相同的作用,不过如果请求的锁不能立即获取到则会返回一个 null.

```
public abstract class FileLock
{
public final FileChannel channel( )
public final long position( )
public final long size( )
public final boolean isShared( )
public final boolean overlaps (long position, long size)
public abstract boolean isValid( );
public abstract void release( ) throws IOException;
}
```
FileLock 类封装一个锁定的文件区域。FileLock 对象由 FileChannel 创建并且总是关联到那个特定的通道实例。您可以通过调用 channel( )方法来查询一个 lock 对象以判断它是由哪个通道创建的。

一个 FileLock 对象创建之后即有效,直到它的 release( )方法被调用或它所关联的通道被关闭或Java 虚拟机关闭时才会失效。我们可以通过调用 isValid( )布尔方法来测试一个锁的有效性。一个锁的有效性可能会随着时间而改变,不过它的其他属性——位置(position)、范围大小(size)和独占性(exclusivity)——在创建时即被确定,不会随着时间而改变。

您可以通过调用 isShared( )方法来测试一个锁以判断它是共享的还是独占的。如果底层的操作系统或文件系统不支持共享锁,那么该方法将总是返回 false 值,即使您申请锁时传递的参数值是 true。假如您的程序依赖共享锁定行为,请测试返回的锁以确保您得到了您申请的锁类型。FileLock 对象是线程安全的,多个线程可以并发访问一个锁对象。

最后,您可以通过调用 overlaps( )方法来查询一个 FileLock 对象是否与一个指定的文件区域重叠。这将使您可以迅速判断您拥有的锁是否与一个感兴趣的区域(region of interest)有交叉。不过即使返回值是 false 也不能保证您就一定能在期望的区域上获得一个锁,因为 Java 虚拟机上的其他地方或者外部进程可能已经在该期望区域上有一个或多个锁了。您最好使用 tryLock( )方法确认一下。

尽管一个 FileLock 对象是与某个特定的 FileChannel 实例关联的,它所代表的锁却是与一个底层文件关联的,而不是与通道关联。因此,如果您在使用完一个锁后而不释放它的话,可能会导致冲突或者死锁。请小心管理文件锁以避免出现此问题。一旦您成功地获取了一个文件锁,如果随后在通道上出现错误的话,请务必释放这个锁。推荐使用类似下面的代码形式:

```
FileLock lock = fileChannel.lock( )
try {
<perform read/write/whatever on channel>
} catch (IOException) [
<handle unexpected exception>
} finally {
lock.release( )
}
```
示例：[LockTest.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/LockTest.java)

参考：
1. [Java NIO](https://pan.baidu.com/s/1uzTW4)
2. [示例代码](http://www.javanio.info/filearea/bookexamples/)
