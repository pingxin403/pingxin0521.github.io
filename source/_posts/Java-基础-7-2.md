---
title: Java 网络编程（二）
date: 2019-04-07 08:38:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

### 发送和接受数据

任何要交换信息的程序之间在信息的编码方式上必须达成共识（如将信息表示为位序列），以及哪个程序发信息，什么时候和怎样接受信息都将影响程序的行为。程序达成的这种包含了信息交换的形式和意义的共识称为**协议**，用来实现特定应用程序的协议叫做应用程序协议。在绝大部分实际应用中，客户端和服务端的行为都有依赖与他们所交换的信息，因此应用程序协议通常更加复杂。

<!--more-->

TCP/IP协议以字节的方式传输用户数据，并没有对其进行检査和修改，这个特点使 应用程序可以非常灵活地对其传输的信患进行编码，大部分应用程序协议是根据由字段 年 列 组 成的离散 信 息 定义的，其中毎个字段中都包含了一段以位序列编码的特定的信息，应用程序协议中明确定义了信息的发送者应该怎样排列和解释这些位序列，同时还要定义接收者应该怎样 解 析 ，这样才使信息的接收者能够抽取出每个字段的意义。TCP/IP协议的唯一约束是，信息必须在块（chunk)中发送和接收，而块的长度必须是8位的倍数，因此， 我 们可以认为在TCP/丨 P协议中传输的信息是字节序列。鉴于此， 我 们可以进一步把传输的信患看作数字序列或数组，每个数字的取值范围是0～255，这与8位编码的二进制数值范围是一致的：00000000代表0，00000001代表1，00000010代表2等，最多到11111111,即255。

如果你建立了一个程序使用套接字与其他程序交换信患，通常符合下面两种情况之 一：要么是你设计和编写了套接字的客户端和服务器端，这种情况下你能够随心所欲地 定义自己的应用程序协议；要么是你实现了一个已经存在的协议，或许是一个协议标准。

任何一种情况，在“线路上”的将不同类型的信息进行字节编码和解码的基本原理还是一样的，如果“线路”是文件，以下内容也是适用的。

#### 信息编码

首先，我们来考虑一下简单数据类型，如1nt、long、char、 String等，是如何通过套接字发送和接收的。从前面章节我们已经知道，传输倍息时可以通过套接字将字节信患写入一个**OutputStrean**实例中（该实例已经与一个Socket相关联），或将其封装进—个DataoramPacket实例中（该实例将由DatagranSocket发送然而，这些操作所能处理的唯一数据类型是字节和字节数组，作为一种》类型语言，Java需要把其他数据类型（int、String等）显式转換成字节数组，所幸的是Java的内置工具能够帮助我们完成这些转换，我们看到过String类的getBytes()方法，该方法就是将一个Srlnfl实例中的字符转換成字节的标准方式，在考虑数据类型转换的细节之前，我们先来看看大部分基本数据类型的表示方法。

**基本整性**

如我们所见，TCP和UDP套接字使我们能发送和接收字节序列（数组），即范围在 0~255之间的整数。使用这个功能，我们可以对值更大的基本整型数据进行编码，不过发送者和接收者必须先在一些方面达成共识.一是要传输的每个整数的字节大小。例如，Java程序中，int数据类型由32位表示，因此，我们可以使用4个字节来传输任意的int型变量或常置。short数据类型由16位表示，传输short类型的数据只需要两个字 节，同理，传输64位的long类型数据則需要8个字节。

下面我们考虑如何对一个包含了 4个整数的序列进行编码：一个byte型，一个short 型，一个int型，以及一个long型，按照这个顺序从发送者传输到接收者，我们总共需要15个字节：第一个宇节存放byte型数据，接下来两个字节存放short型数据，再后面4个字节存放int型数据，最后8个字节存放lonfl型数据，如下所示：

![](https://i.loli.net/2019/06/27/5d142ca83743851561.png)

当字节数大于1的时候，我们必须提前知道字节的发送顺序：little-endian(从整数右边开始、由低位到高位发送)、big-endian(从左边开始，由高位到低位发送)。考虑长整型数123456787654321L，其64位表示为0X0000704885F926B1，如果以big-endian，则字节的十进制数值序列：

![](https://i.loli.net/2019/06/27/5d142e59e75ba14880.png)

如果以little-endian，则字节的十进制数值序列：

![](https://i.loli.net/2019/06/27/5d142e5a0fb8919372.png)

java默认使用little-endian。

发 送 者 和 接 收 者 需 要 达 成 共 识 的 最 后 一 个 细 节 是 ： 所 传 输 的 数 值 是 有 符 号 的 (signed)还是无符 号 的 （unsigned）。Java中的4 种 基 本 整 型 都是有符号的，它们的值以二 进 制 补 码 的 方 式 存 储 ， 这 是 有符号数值的常用表示方式。在处理有K位的有符号数时，用二进制补码的形式表示负整数-n(l<n<2^(k-1)),則补码的二进制值就为2^k-n。而对于非负整数p(0<p<2^(k-1)-1>，只是简单地用k位二进制数来表示p的值 。因此 ，对于 给 定 的k位 ，我 们 可 以 通 过 二 进 制 补 码 来 表 示-2^(k-1)到2^(k-1)范围的值。注意，最髙位(msb)标识了该数是正数(msb = 0)还是负数(msb=l)。另外，如果使用无 符 号编码，位可以直接表示0到2^k-1之间的数值。例如，32位数值Oxffffffff (所有位全为1）,将其解析为有符号数时，二进制补码整数表示-1,将其解析为无符号数时，他表示4294967296，由于Java并不支持无符号整型，如果要在Java中编码和解码无符号数，則需要做一点额外的工作.在此假设我们处理的都是有符号整数数据。

那么我们怎样才能将消患的正确值存入字节数坦呢？为了淸楚地展示需要微的步骤， 我们将对如何使用"位操作（bit-diddlingr）" (移位和屏蔽）来显式编码进行介绍。示例程序BruteForceCoding. java中有一个特殊的方法encodelntBigEndian()能够对任何值的基本类型数据进行编码，它的参数包括用来存放数值的字节数组，要进行编码的数值(表示为long型，它是最长的整型，能够保存其他整型的值），数值在字节数组中开始位置的偏移量，以及该数值写到数组中的字节数。如果我们在发送端进行了编码，那么必须能够在接收端进行解码。

BruteForceCodlno类同时还提供了 decodeIntBigEndian()方法，用来将字节数组的子集解码到一个Java的long型整数中。

```java
public class BruteForceCoding {
    private static byte byteVal = 101; // one hundred and one
    private static short shortVal = 10001; // ten thousand and one
    private static int intVal = 100000001; // one hundred million and one
    private static long longVal = 1000000000001L;// one trillion and one

    private final static int BSIZE = Byte.SIZE / Byte.SIZE;
    private final static int SSIZE = Short.SIZE / Byte.SIZE;
    private final static int ISIZE = Integer.SIZE / Byte.SIZE;
    private final static int LSIZE = Long.SIZE / Byte.SIZE;

    private final static int BYTEMASK = 0xFF; // 8 bits

    //把给定字节数组，按照无符号十进制，BYTEMASK防止在转换时发生符号扩展，即转换成无符号整型
    public static String byteArrayToDecimalString(byte[] bArray) {
        StringBuilder rtn = new StringBuilder();
        for (byte b : bArray) {
            rtn.append(b & BYTEMASK).append(" ");
        }
        return rtn.toString();
    }

    //右移八位，将移位后的数转换成byte，并存入字节数组的适当位置，除了低八位，其他位都丢弃。
    // Warning:  Untested preconditions (e.g., 0 <= size <= 8)
    public static int encodeIntBigEndian(byte[] dst, long val, int offset, int size) {
        for (int i = 0; i < size; i++) {
            dst[offset++] = (byte) (val >> ((size - i - 1) * Byte.SIZE));
        }
        return offset;
    }

    //左移八位。 Warning:  Untested preconditions (e.g., 0 <= size <= 8)
    public static long decodeIntBigEndian(byte[] val, int offset, int size) {
        long rtn = 0;
        for (int i = 0; i < size; i++) {
            rtn = (rtn << Byte.SIZE) | ((long) val[offset + i] & BYTEMASK);
        }
        return rtn;
    }

    public static void main(String[] args) {
        byte[] message = new byte[BSIZE + SSIZE + ISIZE + LSIZE];
        // Encode the fields in the target byte array
        int offset = encodeIntBigEndian(message, byteVal, 0, BSIZE);
        offset = encodeIntBigEndian(message, shortVal, offset, SSIZE);
        offset = encodeIntBigEndian(message, intVal, offset, ISIZE);
        encodeIntBigEndian(message, longVal, offset, LSIZE);
        System.out.println("Encoded message: " + byteArrayToDecimalString(message));

        // Decode several fields
        long value = decodeIntBigEndian(message, BSIZE, SSIZE);
        System.out.println("Decoded short = " + value);
        value = decodeIntBigEndian(message, BSIZE + SSIZE + ISIZE, LSIZE);
        System.out.println("Decoded long = " + value);

        // Demonstrate dangers of conversion
        offset = 4;
        value = decodeIntBigEndian(message, offset, BSIZE);
        System.out.println("Decoded value (offset " + offset + ", size " + BSIZE + ") = "
                + value);
        byte bVal = (byte) decodeIntBigEndian(message, offset, BSIZE);
        System.out.println("Same value as byte = " + bVal);
    }

}
```

本节内容也适用BigInteger类，但是BigInteger可以是任意大小，一种解决方法是使用基于长度的帧。

**字符串和文本**

可以将其他数据类型转换成字符串，再对其进行编码，这样就几乎能发送任何数据类型。通过getBytes()来将字符串转换成字节数组。

实际上，每个String对象内部都是以char []（字符数组），每个char在java内部表示为一个整数，例如，在”ASCII“编码中，字符”a“与整数97对应。但是”ASCII“编码只包含127个符号，不能在国际化的现在大规模使用。

因此，Java使用了一种称为Unicode的国际标准编码字符集来表示char型和String 型值。Unicode字符集将”世界上大部分的语言和符号” 映射到整数0-65 535之间，能更好地适用于国际化程序。例如，日文平假名中代表音节“o1•的符号映射成了整数12362, Unicode包含了ASCII码：毎个ASCII码中定义的符号在Unicode中所映射整数与其在ASCII码中映射的整数相同。这就为ASCII与Unicode之间提供了一定程度的向后兼容性。

发送者与接收者必须在符号与整数的映射方式上达成共识，才能使用文本信惠进行 通信，这就是他们所要达成一致的所有内容吗？还得根据情况而定，对于毎个整数值都比255小的一小组字符，則不需要其他倌惠，因为其每个字符都能够作为一个单独的字节进行编码.对于可能使用超过一个字节的大整数的编码方式，就有多种方式在线路上对其进行编码，因此，发送者和接收者还需要对这些整数如何表示成字节序列统一意见，即編 码 方案。编码 字 符 集 和 字 符 的 编 码 方 案 结 合 起 来 称 为 字 符 集。你也可以定义自己的字符集，但没有理由这样做，世界上已经 有 大 量 不 同 的 标准字符集在使用。Java提供了对任意字符集的支持，而且每种实现都必须支持以下至少一种字符集：US-ASCII (ASCII的另一个名字）,ISO-8859-1, UTF-8, UTF-I6BE,UTF-16LE,UTF-16.

调 用 String实 例 的 getBytes( ) 方 法 ， 将 返 回 一 个 字 节 数 组 ， 该 数 组 根 据 平 台默认字 符 集 对 String实 例 进 行 了 编 码。很 多 平 台 的 默 认 字 符 集 都 是 UTF-8, 然而在一些经常使用ASCII字符集以外的字符的地区，情况有所不同。要保证一个字符串 按 照 特 定 字 符 集 编 码 ， 只 番 要 将 该 字 符 集 的 名 字 作 为 参 数 （ string类型）传递给fletBytes()方法，其返回的字节数组躭包含了由指定字符集表示的字符串。 

下 面 举 例 来 对 fletBytes()方 法 进 行 说 明 • 如 果 在 本 书 所 讲 述 的 平 台 上 调 用  “Test!” .getBytes(),你将获得按照UTF-8字符集编码的字节数组

然而如果你调用"Test!" .getBytes( "UTF-16BE").

![](https://i.loli.net/2019/06/27/5d14363a883fc24863.png)

因此，发送者和接收者必须在文本字符串的表示方式上达成共识。

#### 组合输入输出流

Java中与流相关的类可以组合起来从而提供强大的功能。例如，我们可以将一个Socket实例的OutputStream包装在一个BufferedOutputStream实例中，这样可以先将字节暂时缓存在一起，然后再一次全部发送到底层的通信信道中，以提高程序的性能。我们还能再将这个BufferedOutputStream实例包裹在一个DataOutputStream实例中，以实现发送基本数据类型的功能。

![1.png](https://i.loli.net/2019/06/27/5d1436ef19f6457640.png)

在这个例子中，我们先将基本数据的值，一个一个写入Data OutputStream中，DataOutputStream再将这些数据以二进制的形式写入BufferedOutput-Stream并将三次写入的数据缓存起来，然后再由BufferedOutputStream一次性地将这些数据写入套接字的OutputStream，最后由OutputStream将数据发送到网络。在另一个终端，我们创建了相应的组合InputStream，以有效地接收基本数据类型。

![2.png](https://i.loli.net/2019/06/27/5d1436ef2347e96825.png)

#### 成帧与解析

这部分主要讲的是流传输中对数据开始和结束边界的处理，这也是为什么我们使用read()方法读取-1，进行判定读到流结束的原因。（SOGa！）

成帧（framing）技术则解决了接收端如何定位消息的首尾位置的问题。无论信息是编码成了文本、多字节二进制数、或是两者的结合，应用程序协议必须指定消息的接收者如何确定何时消息已完整接收。
由于UDP套接字保留了消息的边界信息，因此不需要进行成帧处理（实际上，主要是DatagramPacket负载的数据有一个确定的长度，接收者能够准确地知道消息的结束位置），而TCP协议中没有消息边界的概念，因此，在使用TCP套接字时，成帧就是一个非常重要的考虑因素（在TCP连接中，接收者读取完最后一条消息的最后一个字节后，将受到一个流结束标记，即read（）返回-1，该标记指示出已经读取到了消息的末尾，非严格意义上来讲，这也算是基于定界符方法的一种特殊情况）。

**主要有两种技术使接收者能够准确地找到消息的结束位置：**

- 基于定界符：消息的结束由一个唯一的标记指出，即发送者在传输完数据后显式添加的一个特定字节序列，这个特殊标记不能在传输的数据中出现（这也不是绝对的，应用填充技术能够对消息中出现的定界符进行修改，从而使接收者不将其识别为定界符）。该方法通常用在以文本方式编码的消息中。

- 显式长度：在变长字段或消息前附加一个固定大小的字段，用来指示该字段或消息中包含了多少字节。该方法主要用在以二进制字节方式编码的消息中。

 **基于定界符的方法**

通常用在以文本方式编码的消息中：定义一个特殊的字符或字符串来标识消息的结束。接收者只需要简单地扫描输入信息（以字节的方式）来查找定界序列，并将定界符前面的字符串返回。这种方法的缺点是消息本身不能包含有定界字符，否则接收者将提前认为消息已经结束。在基于定界符的成帧方法中，发送者要保证满足这个先决条件。幸运的是，填充（stuffing）技术能够对消息中出现的定界符进行修改，从而使接收者不将其识别为定界符。在接收者扫描定界符时，还能识别出修改过的数据，并在输出消息中对其进行还原，从而使其与原始消息一致。这个技术的缺点是发送者和接收者双方都必须扫描消息。

 **基于长度的方法**

更简单一些，不过要使用这种方法必须知道消息长度的上限。发送者先要确定消息的长度，将长度信息存入一个整数，作为消息的前缀。消息的长度上限定义了用来编码消息长度所需要的字节数：如果消息的长度小于256字节，则需要1个字节；如果消息的长度小于65 536字节，则需要2个字节等。

**代码实现：**

**Frame.java**

定义的Framer接口。它有两个方法：frameMsg（）方法用来添加成帧信息并将指定消息输出到指定流，nextMsg（）方法则扫描指定的流，从中抽取出下一条消息。

```java
import java.io.IOException;
import java.io.OutputStream;
 
public interface Framer {
  void frameMsg(byte[] message, OutputStream out) throws IOException;
  byte[] nextMsg() throws IOException;
}
```

**DelimFramer.java**

DelimFramer.java类实现了基于定界符的成帧方法，其定界符为“换行”符（“\n”，字节值为10）。frameMethod（）方法并没有实现填充，当成帧的字节序列中包含有定界符时，它只是简单地抛出异常。nextMsg（）方法扫描流，直到读取到了定界符，并返回定界符前面的所有字符，如果流为空则返回null。如果累积了一个消息的不少字符，但直到流结束也没有找到定界符，程序将抛出一个异常来指示成帧错误。

```java
import java.io.ByteArrayOutputStream;
import java.io.EOFException;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
 
public class DelimFramer implements Framer {
 
  private InputStream in;        // 数据来源
  private static final byte DELIMITER = '\n'; // 定界符
 
  public DelimFramer(InputStream in) {
    this.in = in;
  }
 
  public void frameMsg(byte[] message, OutputStream out) throws IOException {
    for (byte b : message) {
      if (b == DELIMITER) {
        //如果在消息中检查到界定符，则抛出异常
        throw new IOException("Message contains delimiter");
      }
    }
    out.write(message);
    out.write(DELIMITER);
    out.flush();
  }
 
  public byte[] nextMsg() throws IOException {
    ByteArrayOutputStream messageBuffer = new ByteArrayOutputStream();
    int nextByte;
 
    while ((nextByte = in.read()) != DELIMITER) {
      //如果流已经结束还没有读取到定界符
      if (nextByte == -1) { 
        //如果读取到的流为空，则返回null
        if (messageBuffer.size() == 0) { 
          return null;
        } else { 
          //如果读取到的流不为空，则抛出异常
          throw new EOFException("Non-empty message without delimiter");
        }
      }
      messageBuffer.write(nextByte); 
    }
 
    return messageBuffer.toByteArray();
  }
}
```

**LengthFramer.java**

LengthFramer.java类实现了基于长度的成帧方法，适用于长度小于65 535（216-1）字节的消息。发送者首先给出指定消息的长度，并将长度信息以big-endian顺序存入两个字节的整数中，再将这两个字节放在完整的消息内容前，连同消息一起写入输出流。在接收端，我们使用DataInputStream以读取整型的长度信息；readFully（）方法将阻塞等待，直到给定的数组完全填满，这正是我们需要的。值得注意的是，使用这种成帧方法，发送者**不需要检查要成帧的消息内容，而只需要检查消息的长度是否超出了限制**。

```java
import java.io.DataInputStream;
import java.io.EOFException;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
 
public class LengthFramer implements Framer {
  public static final int MAXMESSAGELENGTH = 65535;
  public static final int BYTEMASK = 0xff;
  public static final int SHORTMASK = 0xffff;
  public static final int BYTESHIFT = 8;
 
  private DataInputStream in;
 
  public LengthFramer(InputStream in) throws IOException {
    this.in = new DataInputStream(in);    //数据来源
  }
 
  //对字节流message添加成帧信息，并输出到指定流 
  public void frameMsg(byte[] message, OutputStream out) throws IOException {
    //消息的长度不能超过65535
    if (message.length > MAXMESSAGELENGTH) {
      throw new IOException("message too long");
    }
    out.write((message.length >> BYTESHIFT) & BYTEMASK);
    out.write(message.length & BYTEMASK);
    out.write(message);
    out.flush();
  }
 
  public byte[] nextMsg() throws IOException {
    int length;
    try { 
      //该方法读取2个字节，将它们作为big-endian整数进行解释，并以int型整数返回它们的值
      length = in.readUnsignedShort(); 
    } catch (EOFException e) { // no (or 1 byte) message
      return null;
    }
    // 0 <= length <= 65535
    byte[] msg = new byte[length];
    //该方法处阻塞等待，直到接收到足够的字节来填满指定的数组
    in.readFully(msg); //
    return msg;
  }
}
```

#### 构建和解析协议信息

简单的投票协议：一个客户端向服务器发送一个请求消息，消息中包含了一个候选人的ID，范围在0~1000。

支持两种请求：一种是查询请求，即向服务器询问候选人当前获得的投票总数，服务器发回一个响应消息，包含了原来的候选人ID和该候选人当前获得的选票总数；另一种是投票请求，即向指定候选人投一票，服务器对这种请求也发回响应消息，包含了候选人ID和获得的选票数（包含了刚刚投的一票）。

在实现一个协议时，一般会定义一个专门的类来存放消息中所包含的的信息。在我们的例子中，客户端和服务端发送的消息都很简单，唯一的区别是服务端发送的消息还包含了选票总数和一个表示相应消息的标志。因此，可以用一个类来表示客户端和服务端的两种消息。下面的VoteMsg.java类展示了每条消息中的基本信息：

- 布尔值isInquiry，true表示该消息是查询请求，false表示该消息是投票请求；
- 布尔值isResponse，true表示该消息是服务器发送的相应消息，false表示该消息为客户端发送的请求消息；
- 整型变量candidateID,指示了候选人的ID；
- 长整型变量voteCount，指示出所查询的候选人获得的总选票数。

另外，注意一下几点：

- candidateID的范围在0~1000；
- voteCount在请求消息中必须为0；
- voteCount不能为负数

VoteMsg代码如下：

```java
public class VoteMsg {
  private boolean isInquiry; // true if inquiry; false if vote
  private boolean isResponse;// true if response from server
  private int candidateID;   // in [0,1000]
  private long voteCount;    // nonzero only in response
 
  public static final int MAX_CANDIDATE_ID = 1000;
 
  public VoteMsg(boolean isResponse, boolean isInquiry, int candidateID, long voteCount)
      throws IllegalArgumentException {
    // check invariants
    if (voteCount != 0 && !isResponse) {
      throw new IllegalArgumentException("Request vote count must be zero");
    }
    if (candidateID < 0 || candidateID > MAX_CANDIDATE_ID) {
      throw new IllegalArgumentException("Bad Candidate ID: " + candidateID);
    }
    if (voteCount < 0) {
      throw new IllegalArgumentException("Total must be >= zero");
    }
    this.candidateID = candidateID;
    this.isResponse = isResponse;
    this.isInquiry = isInquiry;
    this.voteCount = voteCount;
  }
 
  public void setInquiry(boolean isInquiry) {
    this.isInquiry = isInquiry;
  }
 
  public void setResponse(boolean isResponse) {
    this.isResponse = isResponse;
  }
 
  public boolean isInquiry() {
    return isInquiry;
  }
 
  public boolean isResponse() {
    return isResponse;
  }
 
  public void setCandidateID(int candidateID) throws IllegalArgumentException {
    if (candidateID < 0 || candidateID > MAX_CANDIDATE_ID) {
      throw new IllegalArgumentException("Bad Candidate ID: " + candidateID);
    }
    this.candidateID = candidateID;
  }
 
  public int getCandidateID() {
    return candidateID;
  }
 
  public void setVoteCount(long count) {
    if ((count != 0 && !isResponse) || count < 0) {
      throw new IllegalArgumentException("Bad vote count");
    }
    voteCount = count;
  }
 
  public long getVoteCount() {
    return voteCount;
  }
 
  public String toString() {
    String res = (isInquiry ? "inquiry" : "vote") + " for candidate " + candidateID;
    if (isResponse) {
      res = "response to " + res + " who now has " + voteCount + " vote(s)";
    }
    return res;
  }
}
```

根据一定的协议来对其进行编解码，定义一个VoteMsgCoder接口，它提供了对投票消息进行序列化和反序列化的方法。toWrie（）方法用于根据一个特定的协议，将投票消息转换成一个字节序列，fromWire（）方法则根据相同的协议，对给定的字节序列进行解析，并根据信息的内容返回一个该消息类的实例。

```java
import java.io.IOException;
 
public interface VoteMsgCoder {
  byte[] toWire(VoteMsg msg) throws IOException;
  VoteMsg fromWire(byte[] input) throws IOException;
}
```

两个实现了VoteMsgCoder接口的类，一个实现的是基于文本的编码方式 ，一个实现的是基于二进制的编码方式。

首先是用文本方式对消息进行编码的程序。该协议指定使用ASCII字符集对文本进行编码。消息的开头是一个所谓的”魔术字符串“，即一个字符序列，用于快速将投票协议的消息和网络中随机到来的垃圾消息区分开，投票/查询布尔值被编码为字符形似，‘v’代表投票消息，‘i’代表查询消息。是否为服务器发送的响应消息，由字符‘R’指示，状态标记后面是候选人ID，其后跟的是选票总数，它们都编码成十进制字符串。

```java
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Scanner;
 
public class VoteMsgTextCoder implements VoteMsgCoder {
  /*
   * Wire Format "VOTEPROTO" <"v" | "i"> [<RESPFLAG>] <CANDIDATE> [<VOTECNT>]
   * Charset is fixed by the wire format.
   */
 
  // Manifest constants for encoding
  public static final String MAGIC = "Voting";
  public static final String VOTESTR = "v";
  public static final String INQSTR = "i";
  public static final String RESPONSESTR = "R";
 
  public static final String CHARSETNAME = "US-ASCII";
  public static final String DELIMSTR = " ";
  public static final int MAX_WIRE_LENGTH = 2000;
 
  public byte[] toWire(VoteMsg msg) throws IOException {
    String msgString = MAGIC + DELIMSTR + (msg.isInquiry() ? INQSTR : VOTESTR)
        + DELIMSTR + (msg.isResponse() ? RESPONSESTR + DELIMSTR : "")
        + Integer.toString(msg.getCandidateID()) + DELIMSTR
        + Long.toString(msg.getVoteCount());
    byte data[] = msgString.getBytes(CHARSETNAME);
    return data;
  }
 
  public VoteMsg fromWire(byte[] message) throws IOException {
    ByteArrayInputStream msgStream = new ByteArrayInputStream(message);
    Scanner s = new Scanner(new InputStreamReader(msgStream, CHARSETNAME));
    boolean isInquiry;
    boolean isResponse;
    int candidateID;
    long voteCount;
    String token;
 
    try {
      token = s.next();
      if (!token.equals(MAGIC)) {
        throw new IOException("Bad magic string: " + token);
      }
      token = s.next();
      if (token.equals(VOTESTR)) {
        isInquiry = false;
      } else if (!token.equals(INQSTR)) {
        throw new IOException("Bad vote/inq indicator: " + token);
      } else {
        isInquiry = true;
      }
 
      token = s.next();
      if (token.equals(RESPONSESTR)) {
        isResponse = true;
        token = s.next();
      } else {
        isResponse = false;
      }
      // Current token is candidateID
      // Note: isResponse now valid
      candidateID = Integer.parseInt(token);
      if (isResponse) {
        token = s.next();
        voteCount = Long.parseLong(token);
      } else {
        voteCount = 0;
      }
    } catch (IOException ioe) {
      throw new IOException("Parse error...");
    }
    return new VoteMsg(isResponse, isInquiry, candidateID, voteCount);
  }
}
```

下面将展示基于二进制格式对消息进行编码的程序。与基于文本的格式相反，二进制格式使用固定大小的消息，每条消息由一个特殊字节开始，该字节的最高六位为一个”魔术值“010101，该字节的最低两位对两个布尔值进行了编码，消息的第二个字节总是0，第三、四个字节包含了candidateID值，只有响应消息的最后8个字节才包含了选票总数信息。

```java

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;

/* Wire Format
 *                                1  1  1  1  1  1
 *  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
 * +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
 * |     Magic       |Flags|       ZERO            |
 * +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
 * |                  Candidate ID                 |
 * +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
 * |                                               |
 * |         Vote Count (only in response)         |
 * |                                               |
 * |                                               |
 * +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
 */
public class VoteMsgBinCoder implements VoteMsgCoder {

  // manifest constants for encoding
  public static final int MIN_WIRE_LENGTH = 4;
  public static final int MAX_WIRE_LENGTH = 16;
  public static final int MAGIC = 0x5400;
  public static final int MAGIC_MASK = 0xfc00;
  public static final int MAGIC_SHIFT = 8;
  public static final int RESPONSE_FLAG = 0x0200;
  public static final int INQUIRE_FLAG =  0x0100;

  public byte[] toWire(VoteMsg msg) throws IOException {
    ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
    DataOutputStream out = new DataOutputStream(byteStream); // converts ints

    short magicAndFlags = MAGIC;
    if (msg.isInquiry()) {
      magicAndFlags |= INQUIRE_FLAG;
    }
    if (msg.isResponse()) {
      magicAndFlags |= RESPONSE_FLAG;
    }
    out.writeShort(magicAndFlags);
    // We know the candidate ID will fit in a short: it's > 0 && < 1000 
    out.writeShort((short) msg.getCandidateID());
    if (msg.isResponse()) {
      out.writeLong(msg.getVoteCount());
    }
    out.flush();
    byte[] data = byteStream.toByteArray();
    return data;
  }

  public VoteMsg fromWire(byte[] input) throws IOException {
    // sanity checks
    if (input.length < MIN_WIRE_LENGTH) {
      throw new IOException("Runt message");
    }
    ByteArrayInputStream bs = new ByteArrayInputStream(input);
    DataInputStream in = new DataInputStream(bs);
    int magic = in.readShort();
    if ((magic & MAGIC_MASK) != MAGIC) {
      throw new IOException("Bad Magic #: " +
			    ((magic & MAGIC_MASK) >> MAGIC_SHIFT));
    }
    boolean resp = ((magic & RESPONSE_FLAG) != 0);
    boolean inq = ((magic & INQUIRE_FLAG) != 0);
    int candidateID = in.readShort();
    if (candidateID < 0 || candidateID > 1000) {
      throw new IOException("Bad candidate ID: " + candidateID);
    }
    long count = 0;
    if (resp) {
      count = in.readLong();
      if (count < 0) {
        throw new IOException("Bad vote count: " + count);
      }
    }
    // Ignore any extra bytes
    return new VoteMsg(resp, inq, candidateID, count);
  }
}
```

**发送和接受**

通过流发送消患非常简单，只需要创建消息，调用toWire()方法，添加适当的成帧信息，再写入流。当然，接收消息就要按照相反的順序执行。这个过程适用于TCP协议，而对于UDP协议，不需要显式地成帧，因为UDP协议中保留了消息的边界信息，为了对发送与接收过程进行展示，我们考虑投票脹务的如下几点：1）维护一个候选人ID与其获得选栗数的映射，2）记录提交的投票，3）根据其获得的投票数，对査询指定的候选人和为其投票的消息做出响应。首先，我们实现一个投票服务器所用到的服务，当接收到投票消息时，投票服务器将调用VoteServlce类的handleRequest(）方法对请求进行处理。

```java
import java.util.HashMap;
import java.util.Map;

public class VoteService {

  // Map of candidates to number of votes
  private Map<Integer, Long> results = new HashMap<Integer, Long>();

  public VoteMsg handleRequest(VoteMsg msg) {
    if (msg.isResponse()) { // If response, just send it back
      return msg;
    }
    msg.setResponse(true); // Make message a response
    // Get candidate ID and vote count
    int candidate = msg.getCandidateID();
    Long count = results.get(candidate);
    if (count == null) {
      count = 0L; // Candidate does not exist
    }
    if (!msg.isInquiry()) {
      results.put(candidate, ++count); // If vote, increment count
    }
    msg.setVoteCount(count);
    return msg;
  }
}
```

基于TCP套接字发送投票信息和接收信息，其中消息是使用的二进制方式进行编码。

```java
import java.io.OutputStream;
import java.net.Socket;

public class VoteClientTCP {

  public static final int CANDIDATEID = 888;

  public static void main(String args[]) throws Exception {

    if (args.length != 2) { // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Server> <Port>");
    }

    String destAddr = args[0]; // Destination address
    int destPort = Integer.parseInt(args[1]); // Destination port

    Socket sock = new Socket(destAddr, destPort);
    OutputStream out = sock.getOutputStream();

    // Change Bin to Text for a different framing strategy
    VoteMsgCoder coder = new VoteMsgBinCoder();
    // Change Length to Delim for a different encoding strategy
    Framer framer = new LengthFramer(sock.getInputStream());

    // Create an inquiry request (2nd arg = true)
    VoteMsg msg = new VoteMsg(false, true, CANDIDATEID, 0);
    byte[] encodedMsg = coder.toWire(msg);

    // Send request
    System.out.println("Sending Inquiry (" + encodedMsg.length + " bytes): ");
    System.out.println(msg);
    framer.frameMsg(encodedMsg, out);

    // Now send a vote
    msg.setInquiry(false);
    encodedMsg = coder.toWire(msg);
    System.out.println("Sending Vote (" + encodedMsg.length + " bytes): ");
    framer.frameMsg(encodedMsg, out);
    
    // Receive inquiry response
    encodedMsg = framer.nextMsg();
    msg = coder.fromWire(encodedMsg);
    System.out.println("Received Response (" + encodedMsg.length
               + " bytes): ");
    System.out.println(msg);

    // Receive vote response
    msg = coder.fromWire(framer.nextMsg());
    System.out.println("Received Response (" + encodedMsg.length
           + " bytes): ");
    System.out.println(msg);
    
    sock.close();
  }
}

public class VoteServerTCP {

  public static void main(String args[]) throws Exception {

    if (args.length != 1) { // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Port>");
    }

    int port = Integer.parseInt(args[0]); // Receiving Port

    ServerSocket servSock = new ServerSocket(port);
    // Change Bin to Text on both client and server for different encoding
    VoteMsgCoder coder = new VoteMsgBinCoder();
    VoteService service = new VoteService();

    while (true) {
      Socket clntSock = servSock.accept();
      System.out.println("Handling client at " + clntSock.getRemoteSocketAddress());
      // Change Length to Delim for a different framing strategy
      Framer framer = new LengthFramer(clntSock.getInputStream());
      try {
        byte[] req;
        while ((req = framer.nextMsg()) != null) {
          System.out.println("Received message (" + req.length + " bytes)");
          VoteMsg responseMsg = service.handleRequest(coder.fromWire(req));
          framer.frameMsg(coder.toWire(responseMsg), clntSock.getOutputStream());
        }
      } catch (IOException ioe) {
        System.err.println("Error handling client: " + ioe.getMessage());
      } finally {
        System.out.println("Closing connection");
        clntSock.close();
      }
    }
  }
}
```

基于UDP套接字发送投票信息和接收信息，其中消息是使用文本形式进行的编码。

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.util.Arrays;

public class VoteClientUDP {

  public static void main(String args[]) throws IOException {

    if (args.length != 3) { // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Destination>" +
                                          " <Port> <Candidate#>");
    }

    InetAddress destAddr = InetAddress.getByName(args[0]); // Destination addr
    int destPort = Integer.parseInt(args[1]); // Destination port
    int candidate = Integer.parseInt(args[2]); // 0 <= candidate <= 1000 req'd

    DatagramSocket sock = new DatagramSocket(); // UDP socket for sending
    sock.connect(destAddr, destPort);

    // Create a voting message (2nd param false = vote)
    VoteMsg vote = new VoteMsg(false, false, candidate, 0);

    // Change Text to Bin here for a different coding strategy
    VoteMsgCoder coder = new VoteMsgTextCoder();

    // Send request
    byte[] encodedVote = coder.toWire(vote);
    System.out.println("Sending Text-Encoded Request (" + encodedVote.length
        + " bytes): ");
    System.out.println(vote);
    DatagramPacket message = new DatagramPacket(encodedVote, encodedVote.length);
    sock.send(message);

    // Receive response
    message = new DatagramPacket(new byte[VoteMsgTextCoder.MAX_WIRE_LENGTH],
        VoteMsgTextCoder.MAX_WIRE_LENGTH);
    sock.receive(message);
    encodedVote = Arrays.copyOfRange(message.getData(), 0, message.getLength());

    System.out.println("Received Text-Encoded Response (" + encodedVote.length
        + " bytes): ");
    vote = coder.fromWire(encodedVote);
    System.out.println(vote);
  }
}

public class VoteServerUDP {

    public static void main(String[] args) throws IOException {

        if (args.length != 1) { // Test for correct # of args
            throw new IllegalArgumentException("Parameter(s): <Port>");
        }

        int port = Integer.parseInt(args[0]); // Receiving Port

        DatagramSocket sock = new DatagramSocket(port); // Receive socket

        byte[] inBuffer = new byte[VoteMsgTextCoder.MAX_WIRE_LENGTH];
        // Change Bin to Text for a different coding approach
        VoteMsgCoder coder = new VoteMsgTextCoder();
        VoteService service = new VoteService();

        while (true) {
            DatagramPacket packet = new DatagramPacket(inBuffer, inBuffer.length);
            sock.receive(packet);
            byte[] encodedMsg = Arrays.copyOfRange(packet.getData(), 0, packet.getLength());
            System.out.println("Handling request from " + packet.getSocketAddress() + " ("
                    + encodedMsg.length + " bytes)");

            try {
                VoteMsg msg = coder.fromWire(encodedMsg);
                msg = service.handleRequest(msg);
                packet.setData(coder.toWire(msg));
                System.out.println("Sending response (" + packet.getLength() + " bytes):");
                System.out.println(msg);
                sock.send(packet);
            } catch (IOException ioe) {
                System.err.println("Parse error in message: " + ioe.getMessage());
            }
        }
    }
}

```

