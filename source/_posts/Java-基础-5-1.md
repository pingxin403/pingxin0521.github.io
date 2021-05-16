---
title: Java IO
date: 2019-04-06 11:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 概述

在整个Java.io包中最重要的就是5个类和一个接口。5个类指的是File、OutputStream、InputStream、Writer、Reader；一个接口指的是Serializable.掌握了这些IO的核心操作那么对于Java中的IO体系也就有了一个初步的认识了

Java I/O主要包括如下几个层次，包含三个部分：

1. 流式部分――IO的主体部分；

2. 非流式部分――主要包含一些辅助流式部分的类，如：File类、RandomAccessFile类和FileDescriptor等类；

3. 其他类--文件读取部分的与安全相关的类，如：SerializablePermission类，以及与本地操作系统相关的文件系统的类，如：FileSystem类和Win32FileSystem类和WinNTFileSystem类。

<!--more-->

![IO体系.png](https://i.loli.net/2019/04/12/5cb014edcbbc7.png)

主要的类如下：

      1. File（文件特征与管理）：用于文件或者目录的描述信息，例如生成新目录，修改文件名，删除文件，判断文件所在路径等。
      2. InputStream（二进制格式操作）：抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征。
      3. OutputStream（二进制格式操作）：抽象类。基于字节的输出操作。是所有输出流的父类。定义了所有输出流都具有的共同特征。
      4. Reader（文件格式操作）：抽象类，基于字符的输入操作。
      5. Writer（文件格式操作）：抽象类，基于字符的输出操作。
   6. RandomAccessFile（随机文件操作）：一个独立的类，直接继承至Object.它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作。

> java中比较常用的除了IO还有NIO、AIO（NIO2）

**BIO、NIO、AIO的概述**

首先，传统的 java.io包，它基于流模型实现，提供了我们最熟知的一些 IO 功能，比如 File 抽象、输入输出流等。交互方式是同步、阻塞的方式，也就是说，在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。

java.io包的好处是代码比较简单、直观，缺点则是 IO 效率和扩展性存在局限性，容易成为应用性能的瓶颈。

很多时候，人们也把 java.net下面提供的部分网络 API，比如 Socket、ServerSocket、HttpURLConnection 也归类到同步阻塞 IO 类库，因为网络通信同样是 IO 行为。

第二，在 Java 1.4 中引入了 NIO 框架（java.nio 包），提供了 Channel、Selector、Buffer 等新的抽象，可以构建多路复用的、同步非阻塞 IO 程序，同时提供了更接近操作系统底层的高性能数据操作方式。

第三，在 Java 7 中，NIO 有了进一步的改进，也就是 NIO 2，引入了异步非阻塞 IO 方式，也有很多人叫它 AIO（Asynchronous IO）。异步 IO 操作基于事件和回调机制，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

这里只讲一下经典的BIO，NIO和AIO后续会讲到。


### IO流

Java流类的类结构图

![IO流.jpg](https://i.loli.net/2019/04/12/5cb0138e70ae3.jpg)

1、流的概念和作用

流：代表任何有能力产出数据的数据源对象或者是有能力接受数据的接收端对象

流的本质:数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。

流的作用：为数据源和目的地建立一个输送通道。

Java中将输入输出抽象称为流，就好像水管，将两个容器连接起来。流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流.


2、Java IO所采用的模型  


Java的IO模型设计非常优秀，它使用Decorator(装饰者)模式，按功能划分Stream，您可以动态装配这些Stream，以便获得您需要的功能。

例如，您需要一个具有缓冲的文件输入流，则应当组合使用FileInputStream和BufferedInputStream。

3、IO流的分类

- 根据处理数据类型的不同分为：字符流和字节流

-  根据数据流向不同分为：输入流和输出流

- 按数据来源（去向）分类：

```
1. File（文件）： FileInputStream, FileOutputStream, FileReader, FileWriter
2. byte[]：ByteArrayInputStream, ByteArrayOutputStream
3. Char[]: CharArrayReader,CharArrayWriter
4. String:StringBufferInputStream, StringReader, StringWriter
5. 网络数据流：InputStream,OutputStream, Reader, Writer
```

#### 字符流和字节流

流序列中的数据既可以是未经加工的原始二进制数据，也可以是经一定编码处理后符合某种格式规定的特定数据。因此Java中的流分为两种：

1. 字节流：数据流中最小的数据单元是字节

2. 字符流：数据流中最小的数据单元是字符， Java中的字符是Unicode编码，一个字符占用两个字节。

字符流的由来： Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。为此，JAVA中引入了处理字符的流。因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。

设备上的数据无论是图片或者视频，文字，它们都以二进制存储的。二进制的最终都是以一个8位为数据单元进行体现，所以计算机中的最小数据单元就是字节。意味着，字节流可以处理设备上的所有数据，所以字节流一样可以处理字符数据。

结论：只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。

#### 输入流和输出流

根据数据的输入、输出方向的不同对而将流分为输入流和输出流。

![IO输入输出.jpg](https://i.loli.net/2019/04/12/5cb0167e6a089.jpg)


1) 输入流

程序从输入流读取数据源。数据源包括外界(键盘、文件、网络…)，即是将数据源读入到程序的通信通道

```
    InputStream 是所有的输入字节流的父类，它是一个抽象类。
    ByteArrayInputStream、StringBufferInputStream、FileInputStream 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。
    PipedInputStream 是从与其它线程共用的管道中读取数据，与Piped 相关的知识后续单独介绍。
    ObjectInputStream 和所有FilterInputStream 的子类都是装饰流（装饰器模式的主角）。
```

2) 输出流

程序向输出流写入数据。将程序中的数据输出到外界（显示器、打印机、文件、网络…）的通信通道。

```
    OutputStream 是所有的输出字节流的父类，它是一个抽象类。
    ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。
    PipedOutputStream 是向与其它线程共用的管道中写入数据。
    ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流。
```

采用数据流的目的就是使得输出输入独立于设备。输入流( Input  Stream )不关心数据源来自何种设备（键盘，文件，网络）。输出流( Output Stream )不关心数据的目的是何种设备（键盘，文件，网络）。

相对于程序来说，输出流是往存储介质或数据通道写入数据，而输入流是从存储介质或数据通道中读取数据，一般来说关于流的特性有下面几点：

1. 先进先出，最先写入输出流的数据最先被输入流读取到。

2. 顺序存取，可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（RandomAccessFile可以从文件的任意位置进行存取（输入输出）操作）

3. 只读或只写，每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

#### 节点流与处理流

节点流：直接与数据源相连，读入或读出。
直接使用节点流，读写不方便，为了更快的读写文件，才有了处理流。

![1.png](https://i.loli.net/2019/04/12/5cb02a1d1aad1.png)

常用的节点流
```
父　类 ：InputStream 、OutputStream、 Reader、 Writer
文　件 ：FileInputStream 、 FileOutputStrean 、FileReader 、FileWriter 文件进行处理的节点流
数　组 ：ByteArrayInputStream、 ByteArrayOutputStream、 CharArrayReader 、CharArrayWriter 对数组进行处理的节点流（对应的不再是文件，而是内存中的一个数组）
字符串 ：StringReader、 StringWriter 对字符串进行处理的节点流
管　道 ：PipedInputStream 、PipedOutputStream 、PipedReader 、PipedWriter 对管道进行处理的节点流
```
处理流和节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

![2.png](https://i.loli.net/2019/04/12/5cb02a1cd7e05.png)

常用的处理流
```
    缓冲流：BufferedInputStrean 、BufferedOutputStream、 BufferedReader、 BufferedWriter 增加缓冲功能，避免频繁读写硬盘。
    转换流：InputStreamReader 、OutputStreamReader实现字节流和字符流之间的转换。
    数据流： DataInputStream 、DataOutputStream 等-提供将基础数据类型写入到文件中，或者读取出来。
```

#### 正确关闭流

先关闭外层流，再关闭内层流。一般情况下是：先打开的后关闭，后打开的先关闭；另一种情况：看依赖关系，如果流a依赖流b，应该先关闭流a，再关闭流b。例如处理流a依赖节点流b，应该先关闭处理流a，再关闭节点流b

使用以下格式

```
声明流=null;
try{

定义流;

}catch()
{
  //错误处理
}finally{

  if(流不为null)
  {
    关闭流;
  }
}

```

### IO类

#### 输入字节流InputStream

![1.png](https://i.loli.net/2019/06/25/5d12122a7527125372.png)

IO 中输入字节流的继承图可见上图，可以看出：

1.    InputStream是所有的输入字节流的父类，它是一个抽象类。

2.    ByteArrayInputStream、StringBufferInputStream(上图的StreamBufferInputStream)、FileInputStream是三种基本的介质流，它们分别从Byte数组、StringBuffer、和本地文件中读取数据。

3.    PipedInputStream是从与其它线程共用的管道中读取数据.

4.    ObjectInputStream和所有FilterInputStream的子类都是装饰流（装饰器模式的主角）。

![2.png](https://i.loli.net/2019/04/12/5cb01ea81d754.png)

InputStream中的三个基本的读方法
```
abstract int read() ：读取一个字节数据，并返回读到的数据，如果返回-1，表示读到了输入流的末尾。
int read(byte[] b) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾。
int read(byte[] b, int off, int len) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾。off指定在数组b中存放数据的起始偏移位置；len指定读取的最大字节数。
```

流结束的判断：方法read()的返回值为-1时；readLine()的返回值为null时。

其它方法:
```
long skip(long n)：在输入流中跳过n个字节，并返回实际跳过的字节数。
int available() ：返回在不发生阻塞的情况下，可读取的字节数。
void close() ：关闭输入流，释放和这个流相关的系统资源。
void mark(int readlimit) ：在输入流的当前位置放置一个标记，如果读取的字节数多于readlimit设置的值，则流忽略这个标记。
void reset() ：返回到上一个标记。
boolean markSupported() ：测试当前流是否支持mark和reset方法。如果支持，返回true，否则返回false。
```

#### 输出字节流OutputStream

![3.png](https://i.loli.net/2019/06/25/5d12122ab5f5b56396.png)

IO 中输出字节流的继承图可见上图，可以看出：

1.    OutputStream是所有的输出字节流的父类，它是一个抽象类。

2.    ByteArrayOutputStream、FileOutputStream是两种基本的介质流，它们分别向Byte数组、和本地文件中写入数据。PipedOutputStream是向与其它线程共用的管道中写入数据。

3.    ObjectOutputStream和所有FilterOutputStream的子类都是装饰流。

![4.png](https://i.loli.net/2019/04/12/5cb01ea81fbbb.png)

outputStream中的三个基本的写方法:
```
   abstract void write(int b)：往输出流中写入一个字节。
     void write(byte[] b) ：往输出流中写入数组b中的所有字节。
     void write(byte[] b, int off, int len) ：往输出流中写入数组b中从偏移量off开始的len个字节的数据。
```
其它方法
```
   void flush() ：刷新输出流，强制缓冲区中的输出字节被写出。
     void close() ：关闭输出流，释放和这个流相关的系统资源。
```

#### 字符输入流Reader

![2.png](https://i.loli.net/2019/06/25/5d12122a8a70956395.png)

在上面的继承关系图中可以看出：

1.    Reader是所有的输入字符流的父类，它是一个抽象类。

2.    CharReader、StringReader是两种基本的介质流，它们分别从Char数组、String中读取数据。PipedReader是从与其它线程共用的管道中读取数据。

3.    BufferedReader很明显就是一个装饰器，它和其子类负责装饰其它Reader对象。

4.    FilterReader是所有自定义具体装饰流的父类，其子类PushbackReader对Reader对象进行装饰，会增加一个行号。

5.    InputStreamReader是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream转变为Reader的方法。我们可以从这个类中得到一定的技巧。Reader中各个类的用途和使用方法基本和InputStream中的类使用一致。后面会有Reader与InputStream的对应关系。

![2.png](https://i.loli.net/2019/04/12/5cb01f766d621.png)

主要方法：
```
(1) public int read() throws IOException; //读取一个字符，返回值为读取的字符

(2) public int read(char cbuf[]) throws IOException; /*读取一系列字符到数组cbuf[]中，返回值为实际读取的字符的数量*/
(3) public abstract int read(char cbuf[],int off,int len) throws IOException;
/*读取len个字符，从数组cbuf[]的下标off处开始存放，返回值为实际读取的字符数量，该方法必须由子类实现*/
```

#### 字符输出流Writer

![4.png](https://i.loli.net/2019/06/25/5d12122aa06cf59025.png)

在上面的关系图中可以看出：

1.    Writer是所有的输出字符流的父类，它是一个抽象类。

2.    CharArrayWriter、StringWriter是两种基本的介质流，它们分别向Char数组、String中写入数据。PipedWriter是向与其它线程共用的管道中写入数据，

3.    BufferedWriter是一个装饰器为Writer提供缓冲功能。

4.    PrintWriter和PrintStream极其类似，功能和使用也非常相似。

5.    OutputStreamWriter是OutputStream到Writer转换的桥梁，它的子类FileWriter其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。功能和使用和OutputStream极其类似.

![4.png](https://i.loli.net/2019/04/12/5cb01f766ba21.png)

主要方法：
```
(1)  public void write(int c) throws IOException； //将整型值c的低16位写入输出流
(2)  public void write(char cbuf[]) throws IOException； //将字符数组cbuf[]写入输出流
(3)  public abstract void write(char cbuf[],int off,int len) throws IOException； //将字符数组cbuf[]中的从索引为off的位置处开始的len个字符写入输出流
(4)  public void write(String str) throws IOException； //将字符串str中的字符写入输出流
(5)  public void write(String str,int off,int len) throws IOException； //将字符串str 中从索引off开始处的len个字符写入输出流
```

#### 输入与输出的对应

**字节流的输入与输出的对应**

![1.png](https://i.loli.net/2019/04/12/5cb020ea16f61.png)

图中蓝色的为主要的对应部分，红色的部分就是不对应部分。从上面的图中可以看出JavaIO中的字节流是极其对称的。“存在及合理”我们看看这些字节流中不太对称的几个类吧！

1.    LineNumberInputStream主要完成从流中读取数据时，会得到相应的行号，至于什么时候分行、在哪里分行是由改类主动确定的，并不是在原始中有这样一个行号。在输出部分没有对应的部分，我们完全可以自己建立一个LineNumberOutputStream，在最初写入时会有一个基准的行号，以后每次遇到换行时会在下一行添加一个行号，看起来也是可以的。好像更不入流了。

2.    PushbackInputStream的功能是查看最后一个字节，不满意就放入缓冲区。主要用在编译器的语法、词法分析部分。输出部分的BufferedOutputStream几乎实现相近的功能。

3.    StringBufferInputStream已经被Deprecated，本身就不应该出现在InputStream部分，主要因为String应该属于字符流的范围。已经被废弃了，当然输出部分也没有必要需要它了！还允许它存在只是为了保持版本的向下兼容而已。

4.    SequenceInputStream可以认为是一个工具类，将两个或者多个输入流当成一个输入流依次读取。完全可以从IO包中去除，还完全不影响IO包的结构，却让其更“纯洁”――纯洁的Decorator模式。

5.    PrintStream也可以认为是一个辅助工具。主要可以向其他输出流，或者FileInputStream写入数据，本身内部实现还是带缓冲的。本质上是对其它流的综合运用的一个工具而已。一样可以踢出IO包！System.out和System.out就是PrintStream的实例！

**字符流的输入与输出的对应**

![2.png](https://i.loli.net/2019/04/12/5cb020e9e76aa.png)

**字符流与字节流转换**

转换流的特点：

1.    其是字符流和字节流之间的桥梁

2.    可对读取到的字节数据经过指定编码转换成字符

3.    可对读取到的字符数据经过指定编码转换成字节

何时使用转换流？

1.    当字节和字符之间有转换动作时；

2.    流操作的数据需要编码或解码时。

具体的对象体现：

转换流：在IO中还存在一类是转换流，将字节流转换为字符流，同时可以将字符流转化为字节流。

1.    InputStreamReader:字节到字符的桥梁
2.     OutputStreamWriter:字符到字节的桥梁

```
OutputStreamWriter(OutStreamout):将字节流以字符流输出。
InputStreamReader(InputStream in)：将字节流以字符流输入。
```

这两个流对象是字符体系中的成员，它们有转换作用，本身又是字符流，所以在构造的时候需要传入字节流对象进来。


**字节流和字符流的区别（重点）**

实际上字节流在操作时本身不会用到缓冲区（内存），是文件本身直接操作的，而字符流在操作时使用了缓冲区，通过缓冲区再操作文件

![3.jpg](https://i.loli.net/2019/04/12/5cb020ea095c6.jpg)

节流没有缓冲区，是直接输出的，而字符流是输出到缓冲区的。因此在输出时，字节流不调用colse()方法时，信息已经输出了，而字符流只有在调用close()方法关闭缓冲区时，信息才输出。要想字符流在未关闭时输出信息，则需要手动调用flush()方法。

 - **读写单位不同**：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。

 - **处理对象不同**：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。

结论：只要是处理纯文本数据，就优先考虑使用字符流。除此之外都使用字节流。
