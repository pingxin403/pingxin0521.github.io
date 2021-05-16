---
title: Java  NIO Buffer（二）
date: 2019-04-22 10:18:59
tags:
 - Java
 - NIO
categories:
 - Java
 - 高级
---

### Buffer

数据总是从通道读取到缓冲区中,或者从缓冲区写入到通道中。

缓冲区本质上是一块可以写入数据,然后可以从中读取数据的内存。这块内存被包装成 NIO Buffer 对象,并提供了一组方法,用来方便的访问该块内存。
<!--more-->

继承关系：

![继承](https://i.loli.net/2019/04/23/5cbebcb7475ab.png)


1. 属性
- capacity：容量
- limit：上界，缓冲区的临界区，即最多可读到哪个位置
- position：下标，当前读取到的位置(例如当前读出第5个元素，则读完后，position为6)
- mark：标记，备忘位置

大小关系：0 <= mark <= position <= limit <= capacity

2. 方法
- mark()：记录当前position的位置，使mark=position
- reset()：恢复到上次备忘的位置，即position=mark
- clear()：将缓存区置为待填充状态，即position=0;limit=capacity;mark=-1;
- compact()：丢弃已经读取的数据，保留未读取的数据，并使缓存中处于待填充状态
- flip()：将缓冲区的内容切换为待读取状态，即limit=position;position=0;mark=-1
- rewind()：恢复缓冲区为待读取状态，即position=0;mark=-1;,所以你可以重读 Buffer 中的所有数据。limit 保持不变,仍然表示能从 Buffer 中读取多少个元素(byte、char 等)
- remaining()：缓冲区剩余元素，即limit-position
- isDirect()：是否是直接操作内存的Buffer；若是，则此Buffer直接操作JVM堆外内存 ，使用Unsafe实现；否则操作JVM堆内存
- slice()：从当前buffer中生成一个该buffer尚未使用部分的新的缓冲区，例如当前buffer的position为3，limit为5，则新的缓冲区limit和capacity都为2，offset的3，数据区域两者共享；

以下是 Java NIO 里关键的 Buffer 实现:

- ByteBuffer
- CharBuffer
- DoubleBuffer
-  FloatBuffer
-  IntBuffer
-  LongBuffer
-  ShortBuffer
这些 Buffer 覆盖了你能通过 IO 发送的基本数据类型:`byte, short, int, long, float, double 和 char。`

Java NIO 还有个 MappedByteBuffer,用于表示内存映射文件。

使用 Buffer 读写数据一般遵循以下四个步骤:

1. 写入数据到 Buffer
2. 调用 flip()方法
3. 从 Buffer 中读取数据
4. 调用 clear()方法或者 compact()方法

当向 buffer 写入数据时,buffer 会记录下写了多少数据。一旦要读取数据,需要通过 flip()方法将 Buffer 从写模式切换到读模式。在读模式下,可以读取之前写入到 buffer 的所有数据。

一旦读完了所有的数据,就需要清空缓冲区,让它可以再次被写入。有两种方式能清空缓冲区:调用 clear()或 compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处,新写入的数据将放到缓冲区未读数据的后面。

```
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
position 和 limit 的含义取决于 Buffer 处在读模式还是写模式。不管 Buffer处在什么模式,capacity 的含义总是一样的。

这里有一个关于 capacity,position 和 limit 在读写模式中的说明,详细的解释在插图后面。

![](https://i.loli.net/2019/04/23/5cbec7fa94410.png)

**capacity**

作为一个内存块,Buffer 有一个固定的大小值,也叫―capacity‖.你只能往里写capacity 个 byte、long,char 等类型数据。一旦 Buffer 满了,需要将其清空(通过读数据或者清除数据)才能继续往里写数据。

**position**

当你写数据到 Buffer 中时, position 表示当前的位置。初始的 position 值为0。当一个 byte、long 等数据写到 Buffer 后, position 会向前移动到下一个可插入数据的 Buffer 单元。position 最大可为 capacity – 1.当读取数据时,也是从某个特定位置读。当将 Buffer 从写模式切换到读模式,position 会被重置为 0. 当从 Buffer 的 position 处读取数据时,position向前移动到下一个可读的位置。

**limit**

在写模式下,Buffer 的 limit 表示你最多能往 Buffer 里写多少数据。 写模式下,limit 等于 Buffer 的 capacity。当切换 Buffer 到读模式时, limit 表示你最多能读到多少数据。因此,当切换Buffer 到读模式时,limit 会被设置成写模式下的 position 值。换句话说,你能读到之前写入的所有数据(limit 被设置成已写数据的数量,这个值在写模式下就是 position)

**mark()与 reset()方法**
通过调用 Buffer.mark()方法,可以标记 Buffer 中的一个特定 position。之后可以通过调用 Buffer.reset()方法恢复到这个 position。例如:
```
buffer.mark();
//调用几次 buffer.get()方法。例如在解析过程中
buffer.reset();
//设置 position 恢复到标记的位置.
```

### ByteBuffer
ByteBuffer是NIO里用得最多的Buffer，它包含两个实现方式：HeapByteBuffer是基于Java堆的实现，而DirectByteBuffer则使用了unsafe的API进行了堆外的实现。这里只说HeapByteBuffer。



**使用**

ByteBuffer最核心的方法是put(byte)和get()。分别是往ByteBuffer里写一个字节，和读一个字节。值得注意的是，ByteBuffer的读写模式是分开的，正常的应用场景是：往ByteBuffer里写一些数据，然后flip()，然后再读出来。这里插两个Channel方面的对象，以便更好的理解Buffer。

ReadableByteChannel是一个从Channel中读取数据，并保存到ByteBuffer的接口，它包含一个方法：
```
public intread(ByteBuffer dst) throwsIOException;
```

WritableByteChannel则是从ByteBuffer中读取数据，并输出到Channel的接口：
```
public intwrite(ByteBuffer src) throwsIOException;
```

那么，一个ByteBuffer的使用过程是这样的：
```
1. byteBuffer = ByteBuffer.allocate(N);　　　　//创建
2. readableByteChannel.read(byteBuffer);　　 //读取数据，写入byteBuffer
3. byteBuffer.flip(); 　　　　　　　　　　　　　//变读为写
4. writableByteChannel.write(byteBuffer); 　　//读取byteBuffer，写入数据
```

1. put。写模式下，往buffer里写一个字节，并把postion移动一位。写模式下，一般limit与capacity相等
![1.png](https://i.loli.net/2019/04/25/5cc1294931668.png)


2. flip。写完数据，需要开始读的时候，将postion复位到0，并将limit设为当前postion。

3. get。从buffer里读一个字节，并把postion移动一位。上限是limit，即写入数据的最后位置。
![2.png](https://i.loli.net/2019/04/25/5cc129493085f.png)

4. clear。将position置为0，并不清除buffer内容。


**填充**

Get 和 put 可以是相对的或者是绝对的。在前面的程序列表中,相对方案是不带有索引参数的函数。当相对函数被调用时,位置在返回时前进一。如果位置前进过多,相对运算就会抛出异常。对于put() , 如果运算会导致位置超出上界, 就会抛出BufferOverflowException 异 常 。 对于get() , 如果位置不小于上界,就会抛出BufferUnderflowException 异常。绝对存取不会影响缓冲区的位置属性,但是如果您所提供的索引超出范围(负数或不小于上界),也将抛IndexOutOfBoundsException 异常。


我们将代表“Hello”字符串的 ASCII 码载入一个名为 buffer 的ByteBuffer 对象中。
```
buffer.put((byte)'H').put((byte)'e').put((byte)'l').put((byte)'l').put((byte)'o');
```
![](https://i.loli.net/2019/04/25/5cc139343ff95.png)

因为我们存放的是字节而不是字符。记住在 java 中,字符在内部以 Unicode 码表示,每个 Unicode 字符占 16 位。本章节的例子使用包含 ascii 字符集数值的字节。通过将char 强制转换为 byte,我们删除了前八位来建立一个八位字节值。这通常只适合于拉丁字符而不能适合所有可能的 Unicode 字符。为了让事情简化,我们暂时故意忽略字符集的映射问题。

既然我们已经在 buffer 中存放了一些数据,如果我们想在不丢失位置的情况下进行一些更改该怎么办呢?put()的绝对方案可以达到这样的目的。假设我们想将缓冲区中的内容从“Hello”的 ASCII 码更改为“Mellow”。我们可以这样实现:
```
buffer.put(0,(byte)'M').put((byte)'w');
```
![](https://i.loli.net/2019/04/25/5cc1393449b97.png)


**翻转**

我们已经写满了缓冲区,现在我们必须准备将其清空。我们想把这个缓冲区传递给一个通道,以使内容能被全部写出。但如果通道现在在缓冲区上执行 get(),那么它将从我们刚刚插入的有用数据之外取出未定义数据。如果我们将位置值重新设为 0,通道就会从正确位置开始获取,但是它是怎样知道何时到达我们所插入数据末端的呢?这就是上界属性被引入的目的。上界属性指明了缓冲区有效内容的末端。我们需要将上界属性设置为当前位置,然后将位置重置为 0。我们可以人工用下面的代码实现:
```
buffer.limit(buffer.position()).position(0);
```
可以直接使用该函数：
```
Buffer.flip();
```
![](https://i.loli.net/2019/04/25/5cc1393448182.png)


**释放**

布尔函数 hasRemaining()会在释放缓冲区时告诉您是否已经达到缓冲区的上界。
```
for (int i = 0; buffer.hasRemaining( ), i++) {
myByteArray [i] = buffer.get( );
}
```
作为选择,remaining()函数将告知您从当前位置到上界还剩余的元素数目
```
int count = buffer.remaining( );
for (int i = 0; i < count, i++) {
myByteArray [i] = buffer.get( );
}
```
一旦缓冲区对象完成填充并释放,它就可以被重新使用了。Clear()函数将缓冲区重置为空状态。它并不改变缓冲区中的任何数据元素,而是仅仅将上界设为容量的值,并把位置设回 0。
```
buffer.clear( );
```
示例：
```
char [] chars = new char [60];

CharBuffer cb = CharBuffer.wrap (chars, 12, 42);
//pos=12,limit=12+42=54,cap=60
System.out.println ("pos=" + cb.position() + ", limit=" + cb.limit() + ", cap=" + cb.capacity());
//pos=12+21=33,,limit=54,cap=60
cb.put ("This is a test String");
//pos=0,limit=33,cap=60
cb.flip();

System.out.println ("pos=" + cb.position() + ", limit=" + cb.limit() + ", cap=" + cb.capacity());
//pos=0,limit=60,cap=60
cb.clear();
//pos=6,limit=60,cap=60
cb.put ("Foobar");

System.out.println ("pos=" + cb.position() + ", limit=" + cb.limit() + ", cap=" + cb.capacity());

//clean只改变limit和position
for (int i = 0; i < 60; i++) {
    System.out.println ("[" + i + "] = " + chars [i]);
}
```


**压缩**

有时,您可能只想从缓冲区中释放一部分数据,而不是全部,然后重新填充。为了实现这一点,未读的数据元素需要下移以使第一个元素索引为 0。尽管重复这样做会效率低下,但这有时非常必要,而 API 对此为您提供了一个 compact()函数。这一缓冲区工具在复制数据时要比您使用 get()和 put()函数高效得多。所以当您需要时,请使用 compact()。
![压缩前](https://i.loli.net/2019/04/25/5cc13934304a3.png)

```
buffer.compact();
```

![压缩后](https://i.loli.net/2019/04/25/5cc139342e84a.png)

这里发生了几件事。您会看到数据元素 2-5 被复制到 0-3 位置。位置 4 和 5 不受影响,但现在正在或已经超出了当前位置,因此是“死的”。它们可以被之后的 put()调用重写。还要注意的是,位置已经被设为被复制的数据元素的数目。也就是说,缓冲区现在被定位在缓
冲区中最后一个“存活”元素后插入数据的位置。最后,上界属性被设置为容量的值,因此缓冲区可以被再次填满。调用 compact()的作用是丢弃已经释放的数据,保留未释放的数据,并使缓冲区对重新填充容量准备就绪。

您可以用这种类似于先入先出(FIFO)队列的方式使用缓冲区。当然也存在更高效的算法(缓冲区移位并不是一个处理队列的非常高效的方法)。但是压缩对于使缓冲区与您从端口中读入的数据(包)逻辑块流的同步来说也许是一种便利的方法。

如果您想在压缩后释放数据,缓冲区会像之前所讨论的那样需要被翻转。无论您之后是否要向缓冲区中添加新的数据,这一点都是必要的。

**标记**

标记,使缓冲区能够记住一个位置并在之后将其返回。缓冲区的标记在 mark( )函数被调用之前是未定义的,调用时标记被设为当前位置的值。reset( )函数将位置设为当前的标记值。如果标记值未定义,调用 reset( )将导致 InvalidMarkException 异常。一些缓冲区函数会抛弃已经设定的标记(rewind( ),clear( ),以及 flip( )总是抛弃标记)。如果新设定的值比当前的标记小,调用limit( )或 position( )带有索引参数的版本会抛弃标记。
```
buffer.position(2).mark().position(4);
```
![](https://i.loli.net/2019/04/25/5cc13bcb3013b.png)

如果这个缓冲区现在被传递给一个通道,两个字节(“ow”)将会被发送,而位置会前进到 6。如果我们此时调用 reset( ),位置将会被设为标记。再次将缓冲区传递给通道将导致四个字节(“llow”)被发送。

![](https://i.loli.net/2019/04/25/5cc13bcac055d.png)

**比较**


可以使用 equals()和 compareTo()方法比较两个 Buffer。

*equals()*

当满足下列条件时,表示两个 Buffer 相等:
1. 有相同的类型(byte、char、int 等)
2. Buffer 中剩余的 byte、char 等的个数相等。
3. Buffer 中所有剩余的 byte、char 等都相同。

如你所见, equals 只是比较 Buffer 的一部分,不是每一个在它里面的元素都比较。实际上,它只比较 Buffer 中的剩余元素。

![相等缓冲区](https://i.loli.net/2019/04/25/5cc13d13cc040.png)

![不相等缓冲区](https://i.loli.net/2019/04/25/5cc13d13b39ed.png)

*compareTo()方法*

compareTo()方法比较两个 Buffer 的剩余元素(byte、char 等), 如果满足下列条件,则认为一个 Buffer小于另一个 Buffer:
1. 第一个不相等的元素小于另一个 Buffer 中对应的元素 。
2. 所有元素都相等,但第一个 Buffer 比另一个先耗尽(第一个 Buffer 的元素个数比另一个少)。

(剩余元素是从 position 到 limit 之间的元素)

**批量移动**
缓冲区的涉及目的就是为了能够高效传输数据。一次移动一个数据元素并不高效。

有两种形式的 get( )可供从缓冲区到数组进行的数据复制使用。第一种形式只将一个数组作为参数,将一个缓冲区释放到给定的数组。第二种形式使用 offset 和 length 参数来指定目标数组的子区间。这些批量移动的合成效果与前文所讨论的循环是相同的,但是这些方法可能高效得多,因为这种缓冲区实现能够利用本地代码或其他的优化来移动数据。

下面是CharBuffer中的函数，其他Buffer中的函数使用与之类似：
```
public CharBuffer get (char [] dst)
public CharBuffer get (char [] dst, int offset, int length)
public final CharBuffer put (char[] src)
public CharBuffer put (char [] src, int offset, int length)
public CharBuffer put (CharBuffer src)
public final CharBuffer put (String src)
public CharBuffer put (String src, int start, int end)
```
如果您所要求的数量的数据不能被传送,那么不会有数据被传递,缓冲区的状态保持不变,同时抛出 BufferUnderflowException 异常。因此当您传入一个数组并且没有指定长度,您就相当于要求整个数组被填充。如果缓冲区中的数据不够完全填满数组,您会得到一个异常。这意味着如果您想将一个小型缓冲区传入一个大型数组,您需要明确地指定缓冲区中剩余的数据长度。
```
char [] bigArray = new char [1000];
// Get count of chars remaining in the buffer
int length = buffer.remaining( );
// Buffer is known to contain < 1,000 chars
buffer.get (bigArrray, 0, length);
// Do something useful with the data
processData (bigArray, length);
```
记住在调用 get( )之前必须查询缓冲区中的元素数量(因为我们需要告知 processData( )被放置在 bigArray 中的字符个数)。调用 get( )会向前移动缓冲区的位置属性,所以之后调用remaining( )会返回 0。

另一方面,如果缓冲区存有比数组能容纳的数量更多的数据。
```
char [] smallArray = new char [10];
while (buffer.hasRemaining( )) {
int length = Math.min (buffer.remaining( ), smallArray.length);
buffer.get (smallArray, 0, length);
processData (smallArray, length);
}
```
Put()的批量版本工作方式相似,但以相反的方向移动数据,从数组移动到缓冲区。


**复制缓冲区**

```
public abstract class CharBuffer
extends Buffer implements CharSequence, Comparable
{
// This is a partial API listing
public abstract CharBuffer duplicate( );
public abstract CharBuffer asReadOnlyBuffer( );
public abstract CharBuffer slice( );
}
```
Duplicate()函数创建了一个与原始缓冲区相似的新缓冲区。两个缓冲区共享数据元素,拥有同样的容量,但每个缓冲区拥有各自的位置,上界和标记属性。对一个缓冲区内的数据元素所做的改变会反映在另外一个缓冲区上。这一副本缓冲区具有与原始缓冲区同样的数据视图。如果原始的缓冲区为只读,或者为直接缓冲区,新的缓冲区将继承这些属性。

```
CharBuffer buffer = CharBuffer.allocate (8);
buffer.position (3).limit (6).mark( ).position (5);
CharBuffer dupeBuffer = buffer.duplicate( );
buffer.clear( );
```
![](https://i.loli.net/2019/04/25/5cc1482cb460c.png)

可 以 使 用 asReadOnlyBuffer() 函 数 来 生 成 一 个 只 读 的 缓 冲 区 视 图 。 这 与duplicate()相同,除了这个新的缓冲区不允许使用 put(),并且其 isReadOnly()函数将 会 返 回 true 。 对 这 一 只 读 缓 冲 区 的 put() 函 数 的 调 用 尝 试 会 导 致 抛 出ReadOnlyBufferException 异常。

分割缓冲区与复制相似,但 slice()创建一个从原始缓冲区的当前位置开始的新缓冲区,并且其容量是原始缓冲区的剩余元素数量(limit-position)。这个新缓冲区与原始缓冲区共享一段数据元素子序列。分割出来的缓冲区也会继承只读和直接属性。
```
CharBuffer buffer = CharBuffer.allocate (8);
buffer.position (3).limit (5);
CharBuffer sliceBuffer = buffer.slice( );
```
![](https://i.loli.net/2019/04/25/5cc1482c9dc88.png)

要创建一个映射到数组位置 12-20(9 个元素)的 buffer 对象,应使用下面的代码实现:
```
char [] myBuffer = new char [100];
CharBuffer cb = CharBuffer.wrap (myBuffer);
cb.position(12).limit(21);
CharBuffer sliced = cb.slice( );
```


#### 源码解析

我们上面介绍了mark、position、limit、capacity的使用和原理，其实buffer内部就是根据这四个索引来进行操作。除此之外，ByteBuffer还有其他的成员：
```
final byte[] hb;  // Non-null only for heap buffers
final int offset; //
boolean isReadOnly; //是否只读
```


在向ByteBuffer写入数据后，position为缓冲区中刚刚读入的数据的最后一个字节的位置，flip方法将limit值置为position值，position置0，这样在调用get*()方法从ByteBuffer中取数据时就可以取到ByteBuffer中的有效数据
```
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```
在调用four.write(buff)时，就将buff缓冲区中的数据写入到输出管道，此时调用ByteBuffer.clear()方法为下次从管道中读取数据做准备，但是调用clear方法并不将缓冲区的数据清空，而是设置position，mark，limit这三个变量的值，如果不清空的话，假设此次该ByteBuffer中的数据是满的，下次读取的数据不足以填满缓冲区，那么就会存在上一次已经处理的的数据。
```
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```
判断缓冲区中是否还有可用数据时，使用ByteBuffer.hasRemaining()方法，比较了position和limit的值，用以判断是否还有可用数据
```
public final boolean hasRemaining() {
    return position < limit;
}
```
在ByteBuffer类中，还有个方法是compact，对于ByteBuffer，其子类HeapByteBuffer的compact方法实现是这样的
```
public ByteBuffer compact() {
    System.arraycopy(hb, ix(position()), hb, ix(0), remaining());
    position(remaining());
    limit(capacity());
    return this;
}
```
如果position()方法返回当前缓冲区中的position值，remaining()方法返回limit与position这段区间的长度
```
public final int remaining() {
    return limit - position;
}
```
所以compact()方法中第一条语句作用是将数组hb当前position所指向的位置开始复制长度为limit-position的数据到hb数组的开始出，其中使用到了ix()函数，这个函数是将参数值加上一个offset值，offset即一个偏移值，在这样的比如一个这样的场景对于一个很大的缓冲区，将其分成两段，第一段的起始位置是p1，长度是q1,第二段起始位置是p2，长度是q2，那么可以分别将这两段包装成一个HeapByteBuffer对象，然后这两个HeapByteBuffer对象(ByteBuffer的子类，默认实现)的offset属性分别设置为p1和p2，这样就可以通过在内部使用ix()函数来简化ByteBuffer对外提供的接口，在使用者看来，与默认的ByteBuffer并没有区别。

在compact函数中，接着将当前的缓冲区的position索引置为limit-position，limit索引置为缓冲区的容量，这样调用compact方法中就可以将缓冲区的有效数据全部移到缓冲区的首部，而position指向下一个可写位置。

ByteBuffer还有一个名为mark的方法，该方法设置mark索引为position的值
```
public final Buffer mark() {
    mark = position;
    return this;
}
```
与其功能相反的方法为reset方法，即将position的值设置为mark

```
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```
外还有一个名为rewind的方法，这个方法将position索引置为0，mark索引置为-1。
```
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```
通过这些方法，就可以很方便的操作一个缓冲区，关键是要理解这些方法具体的作用，以及对三个索引值的影响(capacity是不变的)。

ByteBuffer继承自Buffer类，上面的方法四个索引值都定义在Buffer类中，操作索引值的方法也都定义在Buffer类中。

以字节数组形式返回整个缓冲区的数据/byte[] hb的数据
```
public final byte[] array() {
    if (hb == null)
 throw new UnsupportedOperationException();
     if (isReadOnly)
        throw new ReadOnlyBufferException();  
     return hb;
}
```

toString（）,输出[pos + offset, limit - start]之间的元素。
示例：
```
public static void main (String [] argv)
        throws Exception
{
    CharBuffer cb = CharBuffer.allocate (15);

    cb.put ("Hello World");
    println (cb);
    //    limit = position;
    //    position = 0;
    cb.flip();
    println (cb);

    cb.flip();
    println (cb);
}

private static void println (CharBuffer cb)
{
    System.out.println ("pos=" + cb.position() + ", limit="
            + cb.limit() + ": '" + cb + "'");
}
```
输出：
```
pos=11, limit=15: '    '
pos=0, limit=11: 'Hello World'
pos=0, limit=0: ''
```


**创建ByteBuffer对象的方式**

 1. allocate方式
```
public static ByteBuffer allocate(int capacity) {
if (capacity < 0)
  throw new IllegalArgumentException();
  return new HeapByteBuffer(capacity, capacity);
}

HeapByteBuffer(int cap, int lim) {  // package-private
 super(-1, 0, lim, cap, new byte[cap], 0);
 /*
 hb = new byte[cap];
 offset = 0;
 */
}


// Creates a new buffer with the given mark, position, limit, capacity,
// backing array, and array offset
//
ByteBuffer(int mark, int pos, int lim, int cap, // package-private
    byte[] hb, int offset)
{
super(mark, pos, lim, cap);
this.hb = hb;
this.offset = offset;
}


// Creates a new buffer with the given mark, position, limit, and capacity,
// after checking invariants.
//
Buffer(int mark, int pos, int lim, int cap) { // package-private
if (cap < 0)
  throw new IllegalArgumentException();
 this.capacity = cap;
 limit(lim);
 position(pos);
 if (mark >= 0) {
   if (mark > pos)
      throw new IllegalArgumentException();
   this.mark = mark;
 }
}
```
由此可见，allocate方式创建ByteBuffer对象的主要工作包括： 新建底层字节数组byte[] hb(长度为capacity)，mark置为-1，position置为0，limit置为capacity，capacity为用户指定的长度。

 2. wrap方式
```
public static ByteBuffer wrap(byte[] array) {
return wrap(array, 0, array.length);
}


public static ByteBuffer wrap(byte[] array,
			    int offset, int length)
{
try {
   return new HeapByteBuffer(array, offset, length);
} catch (IllegalArgumentException x) {
   throw new IndexOutOfBoundsException();
}
}

HeapByteBuffer(byte[] buf, int off, int len) { // package-private
 super(-1, off, off + len, buf.length, buf, 0);
/*
hb = buf;
offset = 0;
*/
}
```
wrap方式和allocate方式本质相同，不过因为由用户指定的参数不同，参数为byte[] array，所以不需要新建字节数组，byte[] hb置为byte[] array，mark置为-1,position置为0，limit置为array.length，capacity置为array.length。




3. array()

通过 allocate()或者 wrap()函数创建的缓冲区通常都是间接的。间接的缓冲区使用备份数组,像我们之前讨论的,您可以通过上面列出的API 函数获得对这些数组的存取权。Boolean 型函数 hasArray()告诉您这个缓冲区是否有一个可存取的备份数组。如果这个函数的返回 true,array()函数会返回这个缓冲区对象所使用的数组存储空间的引用。

arrayOffset(),返回缓冲区数据在数组中存储的开始位置的偏移量(从数组头 0 开始计算)。如果您使用了带有三个参数的版本的 wrap()函数来创建一个缓冲区,对于这个缓冲区,arrayOffset()会一直返回 0,像我们之前讨论的那样。然而,如果您切分了由一个数组提供存储的缓冲区,得到的缓冲区可能会有一个非 0 的数组偏移量。这个数组偏移量和缓冲区容量值会告诉您数组中哪些元素是被缓冲区使用的。

**直接缓冲区**

非直接缓冲区：通过 allocate() 方法分配缓冲区，将缓冲区建立在 JVM 的内存中

![非直接](https://i.loli.net/2019/04/25/5cc16c83c595e.png)


直接缓冲区：通过 allocateDirect() 方法分配直接缓冲区，将缓冲区建立在物理内存中。可以提高效率

![直接](https://i.loli.net/2019/04/25/5cc16c83b18e1.png)


直接字节缓冲区通常是 I/O 操作最好的选择。在设计方面,它们支持 JVM 可用的最高效I/O 机制。非直接字节缓冲区可以被传递给通道,但是这样可能导致性能损耗。通常非直接缓冲不可能成为一个本地 I/O 操作的目标。如果您向一个通道中传递一个非直接 ByteBuffer对象用于写入,通道可能会在每次调用中隐含地进行下面的操作:

1. 创建一个临时的直接 ByteBuffer 对象。
2. 将非直接缓冲区的内容复制到临时缓冲中。
3. 使用临时缓冲区执行低层次 I/O 操作。
4. 临时缓冲区对象离开作用域,并最终成为被回收的无用数据。

这可能导致缓冲区在每个 I/O 上复制并产生大量对象,而这种事都是我们极力避免的。不过,依靠工具,事情可以不这么糟糕。运行时间可能会缓存并重新使用直接缓冲区或者执行其他一些聪明的技巧来提高吞吐量。如果您仅仅为一次使用而创建了一个缓冲区,区别并不是很明显。另一方面,如果您将在一段高性能脚本中重复使用缓冲区,分配直接缓冲区并重新使用它们会使您游刃有余。

直接缓冲区是 I/O 的最佳选择,但可能比创建非直接缓冲区要花费更高的成本。直接缓冲区使用的内存是通过调用本地操作系统方面的代码分配的,绕过了标准 JVM 堆栈。建立和销毁直接缓冲区会明显比具有堆栈的缓冲区更加破费,这取决于主操作系统以及 JVM 实现。直接缓冲区的内存区域不受无用存储单元收集支配,因为它们位于标准 JVM 堆栈之外。

使用直接缓冲区或非直接缓冲区的性能权衡会因JVM,操作系统,以及代码设计而产生巨大差异。通过分配堆栈外的内存,您可以使您的应用程序依赖于JVM未涉及的其它力量。当加入其他的移动部分时,确定您正在达到想要的效果。我以一条旧的软件行业格言建议您:先使其工作,再加快其运行。不要一开始就过多担心优化问题;首先要注重正确性。JVM实现可能会执行缓冲区缓存或其他的优化, 5 这会在不需要您参与许多不必要工作的情况下为您提供所需的性能。

直接 ByteBuffer 是通过调用具有所需容量的 ByteBuffer.allocateDirect()函数产生的,就像我们之前所涉及的 allocate()函数一样。注意用一个 wrap()函数所创建的被包装的缓冲区总是非直接的。



1. 字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在机 此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后）
2. 虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）。
3. 直接字节缓冲区可以通过调用此类的 allocateDirect() 工厂方法 来创建。此方法返回的 缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区 。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。
4. 直接字节缓冲区还可以过 通过FileChannel 的 map() 方法 将文件区域直接映射到内存中来创建 。该方法返回MappedByteBuffer 。Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。
5. 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 isDirect() 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。



>“我们应该忘记小的效率,因为 97%的情况下:过早的优化是所有祸害的根源。”——唐纳德·克努特。

**视图缓冲区**

就像我们已经讨论的那样,I/O 基本上可以归结成组字节数据的四处传递。在进行大数据量的 I/O 操作时,很又可能您会使用各种 ByteBuffer 类去读取文件内容,接收来自网络连接的数据,等等。一旦数据到达了您的 ByteBuffer,您就需要查看它以决定怎么做或者在将它发送出去之前对它进行一些操作。ByteBuffer 类提供了丰富的 API 来创建视图缓冲区。

视图缓冲区通过已存在的缓冲区对象实例的工厂方法来创建。这种视图对象维护它自己的属性,容量,位置,上界和标记,但是和原来的缓冲区共享数据元素。但是 ByteBuffer 类允许创建视图来将 byte 型缓冲区字节数据映射为其它的原始数据类型。例如,asLongBuffer()函数创建一个将八个字节型数据当成一个 long 型数据来存取的视图缓冲区。

下面列出的每一个工厂方法都在原有的 ByteBuffer 对象上创建一个视图缓冲区。调用其中的任何一个方法都会创建对应的缓冲区类型,这个缓冲区是基础缓冲区的一个切分,由基础缓冲区的位置和上界决定。新的缓冲区的容量是字节缓冲区中存在的元素数量除以视图类型中组成一个数据类型的字节数。在切分中任一个超过上界的元素对于这个视图缓冲区都是不可见的。视图缓冲区的第一个元素从创建它的 ByteBuffer 对象的位置开始(positon()函数的返回值)。具有能被自然数整除的数据元素个数的视图缓冲区是一种较好的实现。

```
public abstract CharBuffer asCharBuffer()创建此字节缓冲区的视图，作为 char 缓冲区。
新缓冲区的内容将从此缓冲区的当前位置开始。此缓冲区内容的更改在新缓冲区中是可见的，反之亦然；这两个缓冲区的位置、界限和标记值是相互独立的。
新缓冲区的位置将为零，其容量和界限将为此缓冲区中所剩余的字节数的二分之一，其标记是不确定的。当且仅当此缓冲区为直接时，新缓冲区才是直接的，当且仅当此缓冲区为只读时，新缓冲区才是只读的。
返回：
新的 char 缓冲区

public abstract DoubleBuffer asDoubleBuffer()创建此字节缓冲区的视图，作为 double 缓冲区。
public abstract FloatBuffer asFloatBuffer()创建此字节缓冲区的视图，作为 float 缓冲区。
public abstract IntBuffer asIntBuffer()创建此字节缓冲区的视图，作为 int 缓冲区。
public abstract LongBuffer asLongBuffer()创建此字节缓冲区的视图，作为 long 缓冲区。
public abstract ShortBuffer asShortBuffer()创建此字节缓冲区的视图，作为 short 缓冲区。
```
下面的代码创建了一个 ByteBuffer 缓冲区的 CharBuffer 视图
```
ByteBuffer byteBuffer =ByteBuffer.allocate (7).order (ByteOrder.BIG_ENDIAN);
CharBuffer charBuffer = byteBuffer.asCharBuffer( );
```

![](https://i.loli.net/2019/04/25/5cc14de443a2d.png)

示例：
```
public class BufferCharView
{
public static void main (String [] argv)
throws Exception
{
ByteBuffer byteBuffer =
ByteBuffer.allocate (7).order (ByteOrder.BIG_ENDIAN);
CharBuffer charBuffer = byteBuffer.asCharBuffer( );
// Load the ByteBuffer with some bytes
byteBuffer.put (0, (byte)0);
byteBuffer.put (1, (byte)'H');
byteBuffer.put (2, (byte)0);
byteBuffer.put (3, (byte)'i');
byteBuffer.put (4, (byte)0);
byteBuffer.put (5, (byte)'!');
byteBuffer.put (6, (byte)0);
println (byteBuffer);
println (charBuffer);
}
// Print info about a buffer
private static void println (Buffer buffer)
{
System.out.println ("pos=" + buffer.position( )
+ ", limit=" + buffer.limit( )
+ ", capacity=" + buffer.capacity( )
+ ": '" + buffer.toString( ) + "'");
}
}
```
运行 BufferCharView 程序的输出是:
```
pos=0, limit=7, capacity=7: 'java.nio.HeapByteBuffer[pos=0 lim=7 cap=7]'
pos=0, limit=3, capacity=3: 'Hi!
```
一 旦 您 得 到 了 视 图 缓 冲 区 , 您 可 以 用 duplicate() , slice() 和asReadOnlyBuffer()函数创建进一步的子视图。

**数据元素视图**

ByteBuffer 类提供了一个不太重要的机制来以多字节数据类型的形式存取 byte 数据组。ByteBuffer 类为每一种原始数据类型提供了存取的和转化的方法

![](https://i.loli.net/2019/04/25/5cc154f838f39.png)

这些函数从当前位置开始存取 ByteBuffer 的字节数据,就好像一个数据元素被存储在那里一样。根据这个缓冲区的当前的有效的字节顺序,这些字节数据会被排列或打乱成需要的原始数据类型。比如说,如果 getInt()函数被调用,从当前的位置开始的四个字节会被包装成一个 int 类型的变量然后作为函数的返回值返回。

假设一个叫 buffer 的 ByteBuffer 对象.

![](https://i.loli.net/2019/04/25/5cc159ca6f32c.png)


这段代码:
```
int value = buffer.getInt( );
```
会返回一个由缓冲区中位置 1-4 的 byte 数据值组成的 int 型变量的值。实际的返回值取决于缓冲区的当前的比特排序(byte-order)设置。更具体的写法是:
```
int value = buffer.order (ByteOrder.BIG_ENDIAN).getInt( );
```
这将会返回值 0x3BC5315E,同时:
```
int value = buffer.order (ByteOrder.LITTLE_ENDIAN).getInt( );
```
返回值 0x5E31C53B。

如果您试图获取的原始类型需要比缓冲区中存在的字节数更多的字节,会抛出BufferUnderflowException。对于图 Figure2-17 中的缓冲区,这段代码会抛出异常,因为一个 long 型变量是 8 个字节的,但是缓冲区中只有 5 个字节:
```
long value = buffer.getLong( );
```
这些函数返回的元素不需要被任何特定模块界限所限制。数值将会从以缓冲区的当前位置开始的字节缓冲区中取出并组合,无论字组是否对齐。这样的做法是低效的,但是它允许对一个字节流中的数据进行随机的放置。对于从二进制文件数据或者包装数据成特定平台的格式或者导出到外部的系统,这将是非常有用的。

Put 函数提供与 get 相反的操作。原始数据的值会根据字节顺序被分拆成一个个 byte数据。如果存储这些字节数据的空间不够,会抛出 BufferOverflowException。
每一个函数都有重载的和无参数的形式。重载的函数对位置属性加上特定的字节数。然后无参数的形式则不改变位置属性。

**存取无符号数据**
Java 编程语言对无符号数值并没有提供直接的支持(除了 char 类型)。但是在许多情况下您需要将无符号的信息转化成数据流或者文件,或者包装数据来创建文件头或者其它带有无符号数据区域结构化的信息。ByteBuffer 类的 API 对此并没有提供直接的支持,但是要实现并不困难。您只需要小心精度的问题。

可以使用该工具类操作无符号数据：[Unsigned.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/buffers/Unsigned.java)

### MappedByteBuffer

内存映射文件能让你创建和修改那些因为太大而无法放入内存的文件。有了内存映射文件，你就可以认为文件已经全部读进了内存，然后把它当成一个非常大的数组来访问。这种解决办法能大大简化修改文件的代码。

fileChannel.map(FileChannel.MapMode mode, long position, long size)将此通道的文件区域直接映射到内存中。注意，你必须指明，它是从文件的哪个位置开始映射的，映射的范围又有多大；也就是说，它还可以映射一个大文件的某个小片断。

内存映射 I/O 使用文件系统建立从用户空间直到可用文件系统页的虚拟内存映射,好处：

1. 用户进程把文件数据当作内存,所以无需发布 read( )或 write( )系统调用。
2. 当用户进程碰触到映射内存空间,页错误会自动产生,从而将文件数据从磁盘读进内存。如果用户修改了映射内存空间,相关页会自动标记为脏,随后刷新到磁盘,文件得到更新
3. 操作系统的虚拟内存子系统会对页进行智能高速缓存,自动根据系统负载进行内存管理。
4. 数据总是按页对齐的,无需执行缓冲区拷贝。
5. 大型文件使用映射,无需耗费大量内存,即可进行数据拷贝。

虚拟内存和磁盘 I/O 是紧密关联的,从很多方面看来,它们只是同一件事物的两面。在处理大量数据时,尤其要记得这一点。如果数据缓冲区是按页对齐的,且大小是内建页大小的倍数,那么,对大多数操作系统而言,其处理效率会大幅提升。

内存文件映射，简单地说就是将文件映射到内存的某个地址上。

1. 普通方式读取文件流程。首先内存空间分为内核空间和用户空间，在应用程序读取文件时，底层会发起系统调用，由系统调用将数据先读入到内核空间，然后再将数据拷贝到应用程序的用户空间供应用程序使用。这个过程多了一个从内核空间到用户空间拷贝的过程。

2. 内存文件映射流程文件会被映射到物理内存的某个地址上（不是数据加载到内存），此时应用程序读取文件的地址就是一个内存地址，而这个内存地址会被映射到了前面说到的物理内存的地址上。应用程序发起读之后，如果数据没有加载，系统调用就会负责把数据从文件加载到这块物理地址。应用程序便可以读取到文件的数据。省去了数据从内核空间到用户空间的拷贝过程。所以速度上也会有所提高。

在Java中，具体内存文件映射的方法是通过FileChannel的map()方法来创建一个由磁盘文件支持的虚拟文件映射(virtual memory mapping)并在那块虚拟内存空间外部封装一个MappingByteBuffer对象

通过内存映射机制来访问一个文件效率比其他方式高，因为不需要做明确的系统调用；其直接操作的内存使位于JVM堆外的内存，且虚拟内存可以自动缓存内存页(memory page)

当为一个文件建立虚拟内存映射之后，文件数据通常不会因此被从磁盘读取到内存(这取决于操作系统)。该过程类似打开一个文件：文件先被定位，然后一个文件句柄会被创建，当您准备好之后就可以通过这个句柄来访问文件数据

```
// 第二、三个参数分别表示映射区域的开始(position)和映射的总大小(size)
// 若map的size>文件的大小，则文件会自动扩容至size

public abstract MappedByteBuffer map(MapMode mode,long position, long size)
```

MapMode的三种模式，受FileChannel的访问权限控制
1. READ_ONLY：只读权限
2. READ_WRITE：写权限
3. PRIVATE：写时拷贝(copy-on-write)映射；意味着通过`put()`方法所做的任何修改都会导致产生一个对原数据的私有的数据拷贝，之后将修改应用到拷贝后的数据，此份数据只能被当前MappedByteBuffer对象所见;（**Notes：PRIVATE的拷贝是按内存页拷贝的，若一次put操作只修改了前一个内存页的内容，则后一个内存页的被READ_WRITE的修改也会应用到PRIVATE的映射中**）

MappedByteBuffer的几个方法如下：


- load()：加载整个文件至内存；该操作将会产生大量的页调入(page-in)，具体数量取决于文件中被映射区域的大小；该方法的主要作用是提前加载文件至内存以方便后续的访问速度尽可能的快

- isLoaded()：判断一个映射文件是否已经完全加载至内存

- force()： 同FileChannel的force方法，强制将缓冲区的修改应用到永久磁盘驱动器

如果映射是以 MapMode.READ_ONLY 或 MAP_MODE.PRIVATE 模式建立的,那么调用 force( )方法将不起任何作用,因为永远不会有更改需要应用到磁盘上(但是这样做也是没有害处的)。

MappedByteBuffer的确快，但也存在一些问题，主要就是内存占用和文件关闭等不确定问题。被MappedByteBuffer打开的文件只有在垃圾收集时才会被关闭，而这个点是不确定的。

示例：[MappedHttp](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/MappedHttp.java)

[MapFile.java](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/channels/MapFile.java)

#### 原理

MappedByteBuffer实际上是其子类DirectByteBuffer实例的引用。也就是说，我们获得的MappedByteBuffer实际上是DirectBuffer类型的缓冲区。也就是说，使用MappedByteBuffer并不会消耗Java虚拟机内存堆。

- 使用了 Unsafe 对象完成直接内存的分配回收,并且回收需要主动调用 freeMemory 方法
- ByteBuffer 的实现类内部,使用了 Cleaner (虚引用)来监测 ByteBuffer 对象,一旦
- ByteBuffer 对象被垃圾回收,那么就会由 ReferenceHandler 线程通过 Cleaner 的 clean 方法调用 freeMemory 来释放直接内存

```java
DirectByteBuffer(int cap) {                   // package-private

        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;
    }

```

参考：
1. [Java NIO](https://pan.baidu.com/s/1uzTW4)
2. [示例代码](http://www.javanio.info/filearea/bookexamples/)
3. [深入理解MappedByteBuffer](https://www.jianshu.com/p/f90866dcbffc)
