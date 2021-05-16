---
title: Java 网络编程（四）
date: 2019-04-07 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

#### 套接字底层数据结构

要熟悉掌握网络编程，就需要理解套接字的具体实现所关联的数据结构和底层协议的工作细节，TCP套接字更是如此（Socket实例），需要理解套接字（socket）和Java中类Socket的概念，套接字（socket）指的是底层抽象，这种抽象由操作系统提供或者JVM自己实现（如嵌入式系统中），而Java中类Socket上的操作则转换成了这种底层抽象上的操作。需要注意的是，运行在统一主机上其他程序可能也会通过底层套接字抽象来使用网络，因此会与Java Socket实例竞争系统资源，如端口等。

<!--more-->

“套接字结构”指底层实现（包含了JVM和TCP/IP，通常是后者）的数据结构集，这些数据结构包含了特定Socket实例锁关联的信息。套接字结构除其他信息外还包含了：

1. 该套接字所关联的本地和远程互联网地址和端口号。

2. 一个FIFO（先进先出，First In First Out）队列用于存放接收到的等待分配的数据，以及一个用于存放等待传输的数据的队列。

3. 对于TCP套接字，还包含了与打开和关闭TCP握手相关的额外协议状态信息。

![1.png](https://i.loli.net/2019/06/27/5d1461e6edb6021150.png)

TCP提供了一种可信赖的字节流服务，任何写入Socket的OutputStream的数据副本都必须保留，直到其连接的另一端成功接收。向输出流写数据并不意味着数据实际上已经被发送，它们只是被复制到本地缓冲区。就算在Socket的OUtputStream上调用flush()操作，也无法保证数据发送到信道中，此外字节流服务的自身属性决定了其无法保留输入流中消息的边界消息。

对于DatagramSocket来说，数据包并没有为重传进行缓存，任何时候调用send()方法返回后，数据就已经发送给了执行传输任务的网络子系统。如果网络子系统由于某种原因无法处理这些消息，该数据包将毫无提示地被丢弃（很少发生）。

#### TCP数据传输底层实现

使用TCP套接字需要注意的是：不能假设在连接的一端将数据写入输出流和另有单从输入流读取数据之间有任何一致性。

尤其是在发送端由单个输出流的write()方法传输的数据，可能会通过另一端的多个输入流read()方法来获取；而一个read()方法可能会返回多个write()方法传输的数据。我们可以认为TCP连接上发送的所以字节序列在某一瞬间被分成3个FIFO队列：

1. SendQ:在发送端底层实现中缓存的字节，这些字节已经写入输出流，但还没有在接收端主机上成功接收。

2. RecvQ:在接收端底层实现中缓存的字节，等待分配到接收程序，即从输入流中读取。

3. Delivered:接收者从输入流已经读取到的字节。

发送端：调用out.write()方法将向SendQ追加字节，TCP协议负责将字节按顺序从SendQ移动到RecvQ。这个转移过程无法由用户程序控制或者直接观察到，并且在块中发生，这些块的大小在一定程序上独立与传递给write()方法的缓冲区大小。

接收端：从Socket的InputStream读取数据时，字节就从RecvQ移动到Delivered中，而转移的块的大小依赖于RecvQ中的数据量和传递给read()方法缓冲区大小。

需要说明一点的是：在UNIX(Linux)和Windows平台上，可以用netstat命令查看：SendQ和RecvQ中的字节数，本地和远程IP地址和端口，以及连接状态等。

```java
byte[] buffer0 = new byte[1000];
byte[] buffer1 = new byte[2000];
byte[] buffer2 = new byte[5000];
...
Socket socket = new Socket(destAddr,destPort);
OutputStream out = socket.getOutputStream();
...//设置缓冲区的代码
out.write(buffer0);
...//设置缓冲区的代码
out.write(buffer1);
...//设置缓冲区的代码
out.write(buffer2);
..
socket.close();
```

说明：其中圆点代表了设置缓冲区数据的代码。以下讨论中in代表接收端Socket的InputStream，out代表了发送端socket的OutputStream.

1. 上面的程序中3次调用out.write()方法后，另一端调用in.read()方法前，SendQ，RecvQ,Delivered3个队列的可能状态。不同的阴影效果分别代表了上文中3次调用write()方法传输的不同数据。即第一次写入1000字节+第二次写入中前500字节在RecvQ队列中，而第二次写入中剩余1500字节+第三次写入5000字节在SendQ队列中。

   ![2.png](https://i.loli.net/2019/06/27/5d1462a41e95390267.png)

2. 现在假设接收端调用read()方法时使用的缓冲区数组大小为2000字节，read()方法调用则将把等待分配队列（RecvQ）中的1500字节全部移动到数组中，返回值为1500。注意，这些数据包含了第一次和第二次调用write()方法时传输的字节。再过一段时间，当TCP连接传完更多数据后，这三部分的状态如下：

   ![3.png](https://i.loli.net/2019/06/27/5d1462a5425c250759.png)

3. 如果接收端现在调用read()方法时使用4000自己的缓冲区数组，将很多字节从等待分配的队列（RecvQ）转移到已分配队列（Delivered）中。这包括第二次调用write()方法时剩下的1500字节加上第三次调用write()的前2500字节。此时队列状态如下

   ![5.png](https://i.loli.net/2019/06/27/5d1462a51dc9a26143.png)

下次调用read()方法返回的字节数，取决于缓冲区数组的大小，以及发送方套接字/TCP实现通过网络向接收方实现传输数据的时机。数据从SendQ到RecvQ缓冲区的移动过程对应用程序协议设计有重要的指导性。

#### TCP数据传输中的死锁和性能

**死锁问题**

在TCP数据传输底层实现中（详细参见https://blog.csdn.net/lili13897741554/article/details/83104539）可能会出现死锁的情况，因此程序协议必须设计得非常小心，避免死锁的发生。以下情况可能导致死锁：

1. 每个对等端都在阻塞等待其他端完成一些工作，例如：如果在连接建立后，客户端和服务器端都立即尝试接收数据，显然将导致死锁。

2. SendQ和RecvQ缓冲区的容量在具体实现时会收到一定的限制，如果与TCP的流向控制机制结合使用，有可能到产生死锁的情况。

1）详细说明：

 虽然SendQ和RecvQ缓冲区容量使用的实际大小会动态地增大和收缩，还是需要一个硬性限制，防止行为异常的程序所控制的单独一个TCP连接将系统的内存全部耗尽，由于这些缓冲区的容量有限，它们可能被填满。一旦RecvQ已满，TCP流控制机制就会产生作用。它将阻止传输发送端主机的SendQ中的任何数据，直到接收者调用输入流的read()方法后腾出空间。（使用流控制机制的目的就是为了保证发送者不会传输太多数据，而超出了接收系统的处理能力。）发送程序可以持续地写出数据，直到SendQ队列被填满，然而，如果SendQ队列已满时调用out.write()方法，则将阻塞等待，直到有新的空间为止，也就是说直到一些字节传输到了接收到套接字的RecvQ队列中。如果此时RecvQ队列也已经被填满，所有操作都将停止，直到接收程序调用in.read()方法将一些字节传输到Delivered队列中。

假设SendQ队列和RecvQ队列大小分别是SQS和RQS。将一个大小为n的字节数组传递给write()方法调用，其中n>SQS，直到有至少n-SQS字节传递到接收端主机的RecvQ对列后，该方法才会返回。如果n大小超过了（SQS+RQS）,write()方法则将在接收程序从输入流中读取了n-(SQS+RQS)字节后才会返回。如果接收程序没有调用read()方法，大数量的send()调用则无法成功。特别是当连接的两端同时分别调用它们输出流的write()方法，而它们的缓冲区大小又大于SQS+RQS时将发生死锁，两个write操作都不能完成，两个程序都将永远保持阻塞状态。

2）具体实例：

主机A上的程序和主机B上的程序之间有一个连接，假设A和B上的SQS和RQS都是500字节，两个程序试图同时发送1500字节时的情况。主机A上的程序中前500字节已经传输到了另一端，另外500字节已经复制到了主机A的SendQ队列红，余下的500字节则无法发送（因此out.write()方法将无法返回）直到主机B上的RecvQ队列有空间空出来，然后主机B上程序也遇到同样的情况，因此两个程序的write()方法调用都永远无法完成。

注意点：要仔细设计协议，以避免两个方向上传输大量数据时产生死锁。

![1.png](https://i.loli.net/2019/06/27/5d1463269dc5438402.png)

解决方法：

1）方案之一，是在不同的线程中执行客户端的write()循环和read()循环。一个线程在客户端写完数据后调用该套接字的shutdownOutput()方法，另外一个线程从连接到服务器的输入流中反复读取服务器端反馈给客户端的信息，直到到达输入流的结尾（即服务器端关闭套接字）。如果一个线程阻塞了，另外一个线程仍然可以独立运行。

2）不使用多线程，使用NIO来解决（非阻塞Channel和Selector）。

**数据传输性能**

在TCP实现中，将用户数据复制到SendQ队列中不仅是因为可能重传数据，这还与性能有关。尤其是SendQ和RecvQ缓冲队列的大小，会对TCP连接的数据吞吐量产生影响。吞吐量是指用户数据字节从发送端发送到接收程序的频率。在要传输大量数据的程序中，我们希望最大化这个频率。在没有网络容量或其他限制的情况下，越大的缓冲区通常能够实现越高吞吐量。

如果传输n字节的数据，使用大小为n的缓冲区调用一次write()方法，通常要比使用大小为1字节的缓冲区调用n次write()方法效率高很多。然而，如果调用writer()方法是使用了比SQS(SendQ队列大小)大很多的缓冲区，系统还需要将数据从用户地址转换为大小为SQS的块。即套接字底层实现先将SendQ队列缓冲区填满，等待TCP协议将数据转移出去，再重新填满SendQ队列缓冲区，再等待数据转移，反复进行。套接字底层实现每次都要等待数据从SendQ队列中移除，这就一系统消息的形式消耗形式（系统需要上下文切换）浪费一些时间。由此可知，调用write()方法时实际有效缓冲区大小受到SQS限制，同样read()方法也会受到RQS大小的限制。

需要注意的是：只有当程序一次发送比缓冲区容量大很多的数据，并且要求程序的吞吐量时，可以考虑通过Socket的setSendBufferSize()和sendReceiveBufferSize()方法来改变发送和接收缓冲区的大小。

案例实现：客户端读取文件，然后将文件发送到服务器端，服务器端接收到未压缩的数据，将数据进行简单地压缩并返回给客户端。

1. 客户端代码

   ```java
   import java.io.FileInputStream;
   import java.io.FileOutputStream;
   import java.io.IOException;
   import java.io.InputStream;
   import java.io.OutputStream;
   import java.net.Socket;
   
   public class CompressClient {
   
     public static final int BUFSIZE = 256;  // Size of read buffer
   
     public static void main(String[] args) throws IOException {
   
       if (args.length != 3)  // Test for correct #  of args
         throw new IllegalArgumentException("Parameter(s): <Server> <Port> <File>");
   
       String server = args[0];               // Server name or IP address
       int port = Integer.parseInt(args[1]);  // Server port
       String filename = args[2];             // File to read data from
   
       // Open input and output file (named input.gz)
       final FileInputStream fileIn = new FileInputStream(filename);
       FileOutputStream fileOut = new FileOutputStream(filename + ".gz");
     
       // Create socket connected to server on specified port
       final Socket sock = new Socket(server, port);
   
       // Send uncompressed byte stream to server
       Thread thread = new Thread() {
         public void run() {
           try {
             SendBytes(sock, fileIn);
           } catch (Exception ignored) {}
         }
       };
       thread.start();
   
       // Receive compressed byte stream from server
       InputStream sockIn = sock.getInputStream();
       int bytesRead;                      // Number of bytes read
       byte[] buffer = new byte[BUFSIZE];  // Byte buffer
       while ((bytesRead = sockIn.read(buffer)) != -1) {
         fileOut.write(buffer, 0, bytesRead);
         System.out.print("R");   // Reading progress indicator
       }
       System.out.println();      // End progress indicator line
   
       sock.close();     // Close the socket and its streams
       fileIn.close();   // Close file streams
       fileOut.close();
     }
   
     public static void SendBytes(Socket sock, InputStream fileIn)
         throws IOException {
   
       OutputStream sockOut = sock.getOutputStream();
       int bytesRead;                      // Number of bytes read
       byte[] buffer = new byte[BUFSIZE];  // Byte buffer
       while ((bytesRead = fileIn.read(buffer)) != -1) {
         sockOut.write(buffer, 0, bytesRead);
         System.out.print("W");   // Writing progress indicator
       }
       sock.shutdownOutput();     // Done sending
     }
   }
   
   ```

2. 服务器端的代码

   ```java
   public class CompressServerExecutor {
     public static void main(String[] args) throws IOException {
       ServerSocket serverSocket = new ServerSocket(1234);
       ExecutorService service = Executors.newCachedThreadPool();
       Logger logger = Logger.getLogger("pratical");
       while(true) {
         Socket clientSocket = serverSocket.accept();
         service.submit(new CompressProtocol(clientSocket,logger));
       }
     }
   }
   public class CompressProtocol implements Runnable {
     private static final int BUFSIZE= 1024;
     private Socket clientSocket;
     private Logger logger;
     public CompressProtocol(Socket socket,Logger logger) {
       this.clientSocket=socket;
       this.logger=logger;
     }
     
     private static void handleCompressClient(Socket clientSocket,Logger logger) {
       try {
         InputStream in = clientSocket.getInputStream();
         GZIPOutputStream out = new GZIPOutputStream(clientSocket.getOutputStream());
         byte[] buffer = new byte[BUFSIZE];
         int bytesRead;
         while((bytesRead = in.read(buffer))!=-1) {
           out.write(buffer, 0, bytesRead);
           out.finish(); //刷新流
         }
         logger.info("client "+clientSocket.getRemoteSocketAddress()+" finished");
         clientSocket.close();
       } catch (IOException e) {
         logger.log(Level.WARNING,"Exception in echo Protocol",e);
       }
     }
     
     @Override
     public void run() {
       handleCompressClient(clientSocket,logger);
     }
   }
   ```

分析可能出现的情况：

客户端和服务器端的SendQ队列和RecvQ队列中都有500字节的数据，而客户端发送了一个大小为10000字节（未压缩）的文件，同时假设对于这个文件，服务器读取1000字节并返回500字节，即压缩比2:1。当客户端发送了2000字节后，服务器端最终全部读取这些字节，并发回1000字节，此时客户端的RecvQ队列和服务器端的SendQ队列都将被填满。当客户端又发送了1000字节并被服务器全部读取后，服务器端后续的任何write操作尝试都将阻塞。当客户端又发送了另外1000字节后，客户端的SendQ队列和服务器端的RecvQ队列都将填满，后续客户端write操作将阻塞，从而形成死锁。

解决方案之一：在不同的线程里面执行客户端write()和read()循环。即将CompressClient.java中sendBytes()方法放到线程里面。

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

public class CompressClientNoDeadlock {

  public static final int BUFSIZE = 256;  // Size of read buffer

  public static void main(String[] args) throws IOException {

    if (args.length != 3)  // Test for correct #  of args
      throw new IllegalArgumentException("Parameter(s): <Server> <Port> <File>");

    String server = args[0];               // Server name or IP address
    int port = Integer.parseInt(args[1]);  // Server port
    String filename = args[2];             // File to read data from

    // Open input and output file (named input.gz)
    final FileInputStream fileIn = new FileInputStream(filename);
    FileOutputStream fileOut = new FileOutputStream(filename + ".gz");
  
    // Create socket connected to server on specified port
    final Socket sock = new Socket(server, port);

    // Send uncompressed byte stream to server
    Thread thread = new Thread() {
      public void run() {
        try {
          SendBytes(sock, fileIn);
        } catch (Exception ignored) {}
      }
    };
    thread.start();

    // Receive compressed byte stream from server
    InputStream sockIn = sock.getInputStream();
    int bytesRead;                      // Number of bytes read
    byte[] buffer = new byte[BUFSIZE];  // Byte buffer
    while ((bytesRead = sockIn.read(buffer)) != -1) {
      fileOut.write(buffer, 0, bytesRead);
      System.out.print("R");   // Reading progress indicator
    }
    System.out.println();      // End progress indicator line

    sock.close();     // Close the socket and its streams
    fileIn.close();   // Close file streams
    fileOut.close();
  }

  public static void SendBytes(Socket sock, InputStream fileIn)
      throws IOException {

    OutputStream sockOut = sock.getOutputStream();
    int bytesRead;                      // Number of bytes read
    byte[] buffer = new byte[BUFSIZE];  // Byte buffer
    while ((bytesRead = fileIn.read(buffer)) != -1) {
      sockOut.write(buffer, 0, bytesRead);
      System.out.print("W");   // Writing progress indicator
    }
    sock.shutdownOutput();     // Done sending
  }
}

```

#### TCP套接字生命周期

#####  建立TCP连接

新的Socket实例创建后，就立即能用于发送和接收数据。也就是说，当Socket实例返回时，它已经连接到了一个远程终端，并通过协议的底层实现完成了TCP消息或握手信息的交换。

**客户端连接的建立**

Socket构造函数的调用与客户端连接建立时所关联的协议事件之间的关系下图所示：

![1.png](https://i.loli.net/2019/07/09/5d23f5e6c38dd43145.png)

当客户端以服务器端的互联网地址W.X.Y.Z和端口号Q作为参数，调用Socket的构造函数时，底层实现将创建一个套接字实例，该实例的初始状态是关闭的。**TCP开放握手也称为3次握手，这通常包括3条消息：一条从客户端到服务端的连接请求，一条从服务端到客户端的确认消息，以及另一条从客户端到服务端的确认消息。对客户端而言，一旦它收到了服务端发来的确认消息，就立即认为连接已经建立**。通常这个过程发生的很快，但连接请求消息或服务端的回复消息都有可能在传输过程中丢失，因此TCP协议实现将以递增的时间间隔重复发送几次握手消息。如果TCP客户端在一段时间后还没有收到服务端的回复消息，则发生超时并放弃连接。如果服务端并没有接收连接，则服务端的TCP将发送一条拒绝消息而不是确认消息。

**服务端连接的建立**

当客户端的事件序列则有所不同。服务端首先创建一个ServerSocket实例，并将其与已知端口相关联（在此为Q），套接字实现为新的ServerSocket实例创建一个底层数据结构，并就Q赋给本地端口，并将特定的通配符（*）赋给本地IP地址（服务器可能有多个IP地址，不过通常不会指定该参数），如下图所示：

![2.png](https://i.loli.net/2019/07/09/5d23f5e6e308979494.png)

现在服务端可以调用ServerSocket的accept（）方法，来将阻塞等待客户端连接请求的到来。当客户端的连接请求到来时，将为连接创建一个新的套接字数据结构。该套接字的地址根据到来的分组报文设置：分组报文的目标互联网地址和端口号成为该套接字的本地互联网地址和端口号；而分组报文的源地址和端口号则成为改套接字的远程互联网地址和端口号。注意，新套接字的本地端口号总是与ServerSocket的端口号一致。除了要创建一个新的底层套接字数据结构外，服务端的TCP实现还要向客户端发送一个TCP握手确认消息。如下图所示：

![3.png](https://i.loli.net/2019/07/09/5d23f5e6e2ea932948.png)

但是，对于服务端来说，在接收到客户端发来的第3条消息之前，服务端TCP并不会认为握手消息已经完成。一旦收到客户端发来的第3条消息，则表示连接已建立，此时一个新的数据结构将从服务端所关联的列表中移除，并为创建一个Socket实例，作为accept（）方法的返回值。如下图所示：

![4.png](https://i.loli.net/2019/07/09/5d23f5e6db0a613374.png)

这里有非常重要的一点需要注意，在ServerSocket关联的列表中的每个数据结构，都代表了一个与另一端的客户端已经完成建立的TCP连接。实际上，客户只要收到了开放握手的第2条消息，就可以立即发送数据——这可能比服务端调用accept（）方法为其获取一个Socket实例要早很长时间。

##### 关闭TCP连接

TCP协议有一个优雅的关闭机制，以保证应用程序在关闭时不必担心正在传输的数据会丢失，这个机制还可以设计为允许两个方向的数据传输相互独立地终止。关闭机制的工作流程是：应用程序通过调用连接套接字的close（）方法或shutdownOutput（）方法表明数据已经发送完毕。底层TCP实现首先将留在SendQ队列中的数据传输出去（这还要依赖于另一端的RecvQ队列的剩余空间），然后向另一端发送一个关闭TCP连接的握手消息。该关闭握手消息可以看做流结束的标志：它告诉接收端TCP不会再有新的数据传入RecvQ队列了。注意：关闭握手消息本身并没有传递给接收端应用程序，而是通过read（）方法返回-1来指示其在字节流中的位置。而正在关闭的TCP将等待其关闭握手消息的确认消息，该确认消息表明在连接上传输的所有数据已经安全地传输到了RecvQ中。只要收到了确认消息，该连接变成了“半关闭”状态。直到连接的另一个方向上收到了对称的握手消息后，连接才完全关闭——也就是说，连接的两端都表明它们没有数据发送了。

 TCP连接的关闭事件序列可能以两种方式发生：一种方式是先由一个应用程序调用close（）方法或shutdownOutput方法，并在另一端调用close（）方法之前完成其关闭握手消息；另一种方式是两端同时调用close（）方法，他们的关闭握手消息在网络上交叉传输。下图展示了以第一种方式关闭连接时，发起关闭的一端底层实现中的事件序列：

![5.png](https://i.loli.net/2019/07/09/5d23f5e709b8630956.png)

**注意，如果连接处于半关闭状态时，远程终端已经离开，那么本地底层数据结构则无限期地保持在该状态。当另一端的关闭握手消息到达后，则发回一条确认消息并将状态改为“Time—Wait”。虽然应用程序中相应的Socket实例可能早已消失，与之关联的底层数据结构还将在底层实现中继续存留几分钟。**

对于没有首先发起关闭的一端，关闭握手消息达到后，它立即发回一个确认消息，并将连接状态改为“Close—Wait”。此时，只需要等待应用程序调用Socket的close（）方法。调用该方法后，将发起最终的关闭消息 ，并释放底层套接字数据结构。 下图展示了没有首先发起关闭的一端底层实现中的事件序列：

![6.png](https://i.loli.net/2019/07/09/5d23f5e705c6768198.png)

注意这样一个事实：close（）方法和shutdownOutput（）方法都没有等待关闭握手的完成，而是调用后立即返回，这样，当应用程序调用close（）方法或shutdownOutput（）方法并成功关闭连接时，有可能还有数据留在SendQ队列中。如果连接的任何一端在数据传输到RecvQ队列之前崩溃，数据将丢失，而发送端应用程序却不会知道。

最好的解决方案是设计一种应用程序协议，以使首先调用close（）方法的一方在接收到了应用程序的数据已接收保证后，才真正执行关闭操作。例如，在<http://blog.csdn.net/ns_code/article/details/14642873>这篇博客的分析示例中，客户端程序确认其接收到的字节数与其发送的字节数相等后，它就能够知道此时在连接的两个方向上都没有数据在传输，因此可以安全地关闭连接。

 关闭TCP连接的最后微妙之处在于对Time—Wait状态的需要。TCP规范要求在终止连接时，两端的关闭握手都完成后，至少要有一个套接字在Time—Wait状态保持一段时间。这个要求的提出是由于消息在网络中传输时可能延迟。如果在连接两端都完成了关闭握手后，它们都移除了其底层数据结构，而此时在同样一对套接字地址之间又建立了新的连接，那么前一个连接在网络上传输时延迟的消息就可能在新建立的连接后到达。由于包含了相同的源地址和目的地址，旧消息就会被错误地认为是属于新连接的，其包含的数据就可能被错误地分配到应用程序中。虽然这种情况很少发生，TCP还是使用了包括Time—Write状态在内的多种机制对其进行防范。

 Time—Wait状态最重要的作用是：只要底层套接字数据结构还存在，就不允许在相同的本地端口上关联其他套接字，尤其试图使用该端口创建新的Socket实例时，将抛出IOException异常。

