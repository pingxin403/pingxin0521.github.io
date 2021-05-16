---
title: Java 网络编程（三）
date: 2019-04-07 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

#### 多任务处理

基本的TCP相应服务器是一次只能处理一个客户端请求，无法处理同时多个客户端请求，Java中多线程技术解决这一问题。多线程有两种方式：一是一客户一线程；二是线程池；

<!--more-->

##### 服务器协议

既然我们将要介组的多任务服务器方法与特定的客户/服务器协议相互独立，我们希望能够实现一个同时满足两者的协议。EchoProtocol中给出了回显协议的代码，这个类的静态方法handleEchoCI1ent()中封装了对每个客户端的处理过程。除添加了写日志功能外，这段代码与TCPEchoServer.Java中的连接处理部分几乎完全一致。该方法的参数客户端Socket实例和Logger对象的引用，EchoProtocol类实现了Runnable接口（run()方法只根锯该实例的Socket和Logger引用，简单地调用harulleEchoCIfenU)方法）,因此我们可以戗建一个独立执行run()方法的线程。另外，服务器端的协议执行过程可以通过直接调用这个静态方法实现（为其传入Socket和Logger实例的引用）。

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;

public class EchoProtocol implements Runnable {
  private static final int BUFSIZE = 32; // Size (in bytes) of I/O buffer
  private Socket clntSock;               // Socket connect to client
  private Logger logger;                 // Server logger

  public EchoProtocol(Socket clntSock, Logger logger) {
    this.clntSock = clntSock;
    this.logger = logger;
  }

  public static void handleEchoClient(Socket clntSock, Logger logger) {
    try {
      // Get the input and output I/O streams from socket
      InputStream in = clntSock.getInputStream();
      OutputStream out = clntSock.getOutputStream();

      int recvMsgSize; // Size of received message
      int totalBytesEchoed = 0; // Bytes received from client
      byte[] echoBuffer = new byte[BUFSIZE]; // Receive Buffer
      // Receive until client closes connection, indicated by -1
      while ((recvMsgSize = in.read(echoBuffer)) != -1) {
        out.write(echoBuffer, 0, recvMsgSize);
        totalBytesEchoed += recvMsgSize;
      }

      logger.info("Client " + clntSock.getRemoteSocketAddress() + ", echoed "
          + totalBytesEchoed + " bytes.");

    } catch (IOException ex) {
      logger.log(Level.WARNING, "Exception in echo protocol", ex);
    } finally {
      try {
        clntSock.close();
      } catch (IOException e) {
      }
    }
  }

  public void run() {
    handleEchoClient(clntSock, logger);
  }
}
```

##### 多线程

- 一客户一线程：即为每个连接创建一个线程来处理，服务器端会循环执行，监听指定端口的连接，反复接收来自客户端的连接请求，并为每个连接创建一个新线程来对其处理。

  缺点：一客户一线程的方式虽然处理可以多个客户端请求，但每个新线程都会消耗系统资源（如CPU）,并且每个线程独有自己的数据结构（如栈）也要消耗系统内存。另外一个线程阻塞是，JVM会保存其状态，选择另外一个线程运行，并在上下文转换时恢复阻塞线程的状态。随着线程数的增加，线程将消耗越来越多的资源。这会导致系统将花费更多时间来处理上下文的转换和线程管理，更少的时间来对连接服务，加入额外的线程实际上可能会增加客户端总服务时间。

  解决：通过限制总线程数并重复使用线程来避免一客户一线程的缺陷。

  TCPEchoServerThread.Java实现了这种一客户一线程的服务结构。它与迭代服务器非常相似，也用一个循环来接收和处理客户的请求。主要不同点在于这种服务器为每个连接创建了一个新的线程来处理，而不是直接处理。这是可行的，因为EchoProtocol类实现了Runnable接口）因此，当多个户端几乎同时连接服务器时，后请求的客户端不需要等服务器对前面的客户端处理结束后才获得服务，相反，它们看起来是同时接受的服务（虽然比对单一客户端进行服务要稍微慢一些)，

  ```java
  import java.io.IOException;
  import java.net.ServerSocket;
  import java.net.Socket;
  import java.util.logging.Logger;
  
  public class TCPEchoServerThread {
  
    public static void main(String[] args) throws IOException {
  
      if (args.length != 1) { // Test for correct # of args
        throw new IllegalArgumentException("Parameter(s): <Port>");
      }
  
      int echoServPort = Integer.parseInt(args[0]); // Server port
  
      // Create a server socket to accept client connection requests
      ServerSocket servSock = new ServerSocket(echoServPort);
  
      Logger logger = Logger.getLogger("practical");
  
      // Run forever, accepting and spawning a thread for each connection
      while (true) {
        Socket clntSock = servSock.accept(); // Block waiting for connection
        // Spawn thread to handle new connection
        Thread thread = new Thread(new EchoProtocol(clntSock, logger));
        thread.start();
        logger.info("Created and started Thread " + thread.getName());
      }
      /* NOT REACHED */
    }
  }
  
  ```

- 使用线程池：与为每个线程创建新的线程不同，服务器在启动时创建固定数量的线程组成的线程池。当有一个客户端请求过来时，线程池将分配一个线程处理，线程在处理完请求后将会返回线程池，为下一次请求处理做好准备。如果连接请求到达服务器端，线程池中所有的线程都已被占用，它们则在一个队列中等待，直到有空闲的线程可用。

  线程池服务端具体实现的步骤：

  1. 服务器端创建一个ServerSocket实例。

  2. 创建N个线程，每个线程都反复循环，从（共享的）ServerSocket实例中接收客户端连接。当多个线程同时调用同一个ServerSocket实例的accept()方法将会阻塞等待，直到一个新连接创建成功。

  3. 新建立连接对应Socket实例则只在选中的线程中返回。其他线程将阻塞，直到成功建立下一个连接和选中下一个幸运的线程。

  缺点：创建的线程池太少，客户端可能等待很长时间才能获取服务，线程池大小不能根据客户端请求数量进行调整。

  解决：Java中提供一个调度工具（系统管理调用接口Executor），可以在系统负载时扩展线程池的大小，负载较轻时缩减线程池的大小。

  ```java
  import java.io.IOException;
  import java.net.ServerSocket;
  import java.net.Socket;
  import java.util.logging.Level;
  import java.util.logging.Logger;
  
  public class TCPEchoServerPool {
  
    public static void main(String[] args) throws IOException {
  
      if (args.length != 2) { // Test for correct # of args
        throw new IllegalArgumentException("Parameter(s): <Port> <Threads>");
      }
  
      int echoServPort = Integer.parseInt(args[0]); // Server port
      int threadPoolSize = Integer.parseInt(args[1]);
  
      // Create a server socket to accept client connection requests
      final ServerSocket servSock = new ServerSocket(echoServPort);
  
      final Logger logger = Logger.getLogger("practical");
  
      // Spawn a fixed number of threads to service clients
      for (int i = 0; i < threadPoolSize; i++) {
        Thread thread = new Thread() {
          public void run() {
            while (true) {
              try {
                Socket clntSock = servSock.accept(); // Wait for a connection
                EchoProtocol.handleEchoClient(clntSock, logger); // Handle it
              } catch (IOException ex) {
                logger.log(Level.WARNING, "Client accept failed", ex);
              }
            }
          }
        };
        thread.start();
        logger.info("Created and started Thread = " + thread.getName());
      }
    }
  }
  
  ```

- 系统管理调度：接口Executor

  Executor接口代表了一个根据某种策略来执行Runnable实例的对象，其中包含了排队和调度的细节，或者如何选择要执行的任务。Executor接口只定义了一个方法：

  ```java
  public interface Executor {
    void execute(Runnable command);
  }
  ```

  Java中内置了大量的Executor接口实现，很简单使用，也可以进行扩展性配置。其中一些还提供了处理维护线程等繁琐细节的功能。ExecutorService接口继承于Executor接口，提供了一个更高级的工具来关闭服务器，包括正常关闭和突然关闭。ExecutorService还允许在完成任务后返回一个结果，这需要用到Callable接口，它和Runnable接口很像，只是多了一个返回值。

  ```java
  import java.io.IOException;
  import java.net.ServerSocket;
  import java.net.Socket;
  import java.util.concurrent.Executor;
  import java.util.concurrent.Executors;
  import java.util.logging.Logger;
  
  public class TCPEchoServerExecutor {
  
    public static void main(String[] args) throws IOException {
  
      if (args.length != 1) { // Test for correct # of args
        throw new IllegalArgumentException("Parameter(s): <Port>");
      }
  
      int echoServPort = Integer.parseInt(args[0]); // Server port
  
      // Create a server socket to accept client connection requests
      ServerSocket servSock = new ServerSocket(echoServPort);
  
      Logger logger = Logger.getLogger("practical");
  
      Executor service = Executors.newCachedThreadPool();  // Dispatch svc
  
      // Run forever, accepting and spawning threads to service each connection
      while (true) {
        Socket clntSock = servSock.accept(); // Block waiting for connection
        service.execute(new EchoProtocol(clntSock, logger));
      }
      /* NOT REACHED */
    }
  }
  
  ```

#### 阻塞和超时

socket的调用可能存多种原因而阻塞。

1. read()，receive()方法在没有数据可读时会阻塞。ServerSocket的accept()方法和Socket构造方法会阻塞等待，直到建立连接。

   解决:使用socket,ServerSocket,以及DatagramSocket类的setSoTimeout()方法，来设置其阻塞的最长时间。指定时间内方法没有返回将会抛出异常InterruptedIOException。对于Socket实例，可以在read()方法前，在套接字的InputStream上调用available()方法检测是否有可读数据。

2. 连接和写数据

   Socket类的构造函数会尝试根据参数中指定的主机和端口来建立连接，并阻塞等待，直到连接成功建立或发生了系统定义的超时。不幸的是，系统定义的超时时间很长，而Java又没有提供任何缩短它的方法。要改变这种情况，可以使用Socket类的无参数构造函数，它返回的是一个没有建立连接的Socket实例。需要建立连接时，调用该实例的connect()方法，并指定一个远程终端和超时时间。

   write()方法调用也会阻塞等待，直到最后一个字节成功写入到了TCP实现的本地缓存中。如果可用的缓存空间比要写入的数据小，在write()方法调用返回前，必须把一些数据成功传输到连接的另一端。

3. 限制客户端的时间

   通过服务期限和当前计算出截止时间，每次调用read()结束后，重新计算当前和截止时间的差值，即服务截止时间，并将套接字的超时时间设置为该剩余时间

   ```java
   import java.io.IOException;
   import java.io.InputStream;
   import java.io.OutputStream;
   import java.net.Socket;
   import java.util.logging.Level;
   import java.util.logging.Logger;
   
   class TimeLimitEchoProtocol implements Runnable {
     private static final int BUFSIZE = 32;  // Size (bytes) buffer
     private static final String TIMELIMIT = "10000";  // Default limit (ms)
     private static final String TIMELIMITPROP = "Timelimit";  // Thread property
   
     private static int timelimit;
     private Socket clntSock;
     private Logger logger;
   
     public TimeLimitEchoProtocol(Socket clntSock, Logger logger) {
       this.clntSock = clntSock;
       this.logger = logger;
       // Get the time limit from the System properties or take the default
       timelimit = Integer.parseInt(System.getProperty(TIMELIMITPROP,TIMELIMIT));
     }
   
     public static void handleEchoClient(Socket clntSock, Logger logger) {
   
       try {
         // Get the input and output I/O streams from socket
         InputStream in = clntSock.getInputStream();
         OutputStream out = clntSock.getOutputStream();
         int recvMsgSize;                        // Size of received message
         int totalBytesEchoed = 0;               // Bytes received from client
         byte[] echoBuffer = new byte[BUFSIZE];  // Receive buffer
         long endTime = System.currentTimeMillis() + timelimit;
         int timeBoundMillis = timelimit;
   
         clntSock.setSoTimeout(timeBoundMillis);
         // Receive until client closes connection, indicated by -1
         while ((timeBoundMillis > 0) &&     // catch zero values
                ((recvMsgSize = in.read(echoBuffer)) != -1)) {
           out.write(echoBuffer, 0, recvMsgSize);
           totalBytesEchoed += recvMsgSize;
           timeBoundMillis = (int) (endTime - System.currentTimeMillis()) ;
           clntSock.setSoTimeout(timeBoundMillis);
         }
         logger.info("Client " + clntSock.getRemoteSocketAddress() +
   		  ", echoed " + totalBytesEchoed + " bytes.");
       } catch (IOException ex) {
         logger.log(Level.WARNING, "Exception in echo protocol", ex);
       }
     }
     
     public void run() {
       handleEchoClient(this.clntSock, this.logger);
     }
   }
   ```

#### 多接受者

1. 广播和多播：TCP套接字中客户端只能接收和发送指定服务器端过来的数据，这种一对一的通信方式叫单播，而UDP套接字可以容许一个发送端和多个接收端情况，一对多的类型有：广播和多播。

   - 广播：本地网络中所有的主机都会接收到一份数据副本。IPv4广播地址（255.255.255.255）将消息发送到同一个广播网络中上的所有主机，本地广播信息不回被路由器转发。广播不能使用连接，有些操作系统不支持普通用户进行广播操作。

   - 多播：网络只分发数据给想要接收数据的多播地址的主机。一个多播地址指示了一组接收者，IP协议的设计者为多播分配了一定范围的地址空间，IPv4中多播地址是224.0.0.0到239.255.255.255，IPv6中多播地址是FF开头的地址。多播报文会初始化一个TTL值（Time To Live，生命周期），当存在路由器转发便会减1，TTL值为0时，丢弃该数据报文。

   示例：

   ```java
   import java.io.IOException;
   import java.net.DatagramPacket;
   import java.net.InetAddress;
   import java.net.MulticastSocket;
   
   public class VoteMulticastSender {
   
     public static final int CANDIDATEID = 475;
     
     public static void main(String args[]) throws IOException {
   
       if ((args.length < 2) || (args.length > 3)) { // Test # of args
         throw new IllegalArgumentException("Parameter(s): <Multicast Addr> <Port> [<TTL>]");
       }
   
       InetAddress destAddr = InetAddress.getByName(args[0]); // Destination
       if (!destAddr.isMulticastAddress()) { // Test if multicast address
         throw new IllegalArgumentException("Not a multicast address");
       }
   
       int destPort = Integer.parseInt(args[1]); // Destination port
       int TTL = (args.length == 3) ? Integer.parseInt(args[2]) : 1; // Set TTL
   
       MulticastSocket sock = new MulticastSocket();
       sock.setTimeToLive(TTL); // Set TTL for all datagrams
       
       VoteMsgCoder coder = new VoteMsgTextCoder();
       
       VoteMsg vote = new VoteMsg(true, true, CANDIDATEID, 1000001L);
   
       // Create and send a datagram
       byte[] msg = coder.toWire(vote);
       DatagramPacket message = new DatagramPacket(msg, msg.length, destAddr, destPort);
       System.out.println("Sending Text-Encoded Request (" + msg.length + " bytes): ");
       System.out.println(vote);
       sock.send(message);
   
       sock.close();
     }
   }
   
   import java.io.IOException;
   import java.net.DatagramPacket;
   import java.net.InetAddress;
   import java.net.MulticastSocket;
   import java.util.Arrays;
   
   public class VoteMulticastReceiver {
   
     public static void main(String[] args) throws IOException {
   
       if (args.length != 2) { // Test for correct # of args
         throw new IllegalArgumentException("Parameter(s): <Multicast Addr> <Port>");
       }
   
       InetAddress address = InetAddress.getByName(args[0]); // Multicast address
       if (!address.isMulticastAddress()) { // Test if multicast address
         throw new IllegalArgumentException("Not a multicast address");
       }
   
       int port = Integer.parseInt(args[1]); // Multicast port
       MulticastSocket sock = new MulticastSocket(port); // for receiving
       sock.joinGroup(address); // Join the multicast group
   
       VoteMsgTextCoder coder = new VoteMsgTextCoder();
   
       // Receive a datagram
       DatagramPacket packet = new DatagramPacket(new byte[VoteMsgTextCoder.MAX_WIRE_LENGTH],
           VoteMsgTextCoder.MAX_WIRE_LENGTH);
       sock.receive(packet);
   
       VoteMsg vote = coder.fromWire(Arrays.copyOfRange(packet.getData(), 0, packet
           .getLength()));
   
       System.out.println("Received Text-Encoded Request (" + packet.getLength()
           + " bytes): ");
       System.out.println(vote);
   
       sock.close();
     }
   }
   
   ```

2. Keep-Alive:TCP协议提供了一种Keep-alive机制，发送端和接收端一段时间内没有数据交换时，发送端会向终端发送探测消息，终端如果处于活跃状态会回复一个确认消息。几次尝试后依然没有收到终端消息，则会终止发送探测消息，关闭套接字，下次IO操作时会抛出异常。

3. 发送和接收缓存区的大小：当创建了Socket或者DatagramSocket实例的时候，操作系统就必须为其分配缓存区以存放接收的和要发送的数据。方法setReceiveBufferSize(int size)和setSendBufferSize(int size);

4. 超时：很多IO操作如果不能立即完成就会阻塞等待，读操作会阻塞等待直到至少一个字节可读；接收操作将阻塞等待直到成功建立连接。通过调用setSoTimeout()方法设置读，接收数据以及accept()方法的最长阻塞时间。

5. 地址重用：某些情况下，希望能将多个套接字绑定到同一个套接字地址，对于UDP多播情况，同一个主机上可能有多个应用程序加入相同的多播组。对于TCP来说，当一个连接关闭后，通信的一端必须在“Time-Wait”状态上等一段时间，以对传输途中丢失的数据包进行清理，但通信终端可能无法等待Time-Wait结束。这两种情况需要能够与正在使用的地址进行绑定的能力，实现地址的重用。

6. 消除缓冲延迟：TCP协议将数据缓存起来直到足够多时一次发送，以避免发送过小的数据包而浪费网络资源。虽然这个功能有利于网络，但是应用程序可能对所造成的缓冲延迟无法容忍，可以禁用缓存功能，调用方法setTcpNoDelay(true).

7. 紧急数据：TCP协议中包含了紧急数据的概念，如果需要发送一条紧急数据，但是前面已经有很多其他数据，要求能够绕过这些常规数据（频道外数据）。但Java中紧急数据几乎没有什么用，因为常规数据与紧急数据顺序混在一起，接收者无法区别。

8. 关闭后停留：调用套接字的close()方法后，即使套接字的缓冲区中还存在没有发送的数据，它也会立即返回。这样不发送完所有数可能导致的问题是主机将在后面的某个时刻发生故障。其实可以选择然close()方法“停留”或者阻塞一段时间，直到发送所有数据都已经发送并确认，或者发生了超时。调用方法setSoLinger(boolean on, int linger)。

9. 广播许可：一些操作系统要求显式地对广播许可进行请求，可以对广播许可进行控制。调用方法，setBroadcast（true）,true表示允许广播

10. 通信等级：有的网络对满足服务条件的数据报提供了增强的服务或者额外的保险。一个数据报的通信登记由数据包在网络中传输时其内部的一个值来指定。但是通信等级会收到网络提供者的限制，不能保证这项功能可用。（方法setTrafficClass(int tc)）

11. 基于性能的协议选择：TCP协议不是套接字唯一可选的协议，Java允许开发者根据不同的性能特征对于应用程序的重要程序，为具体实现给出建议，底层网络系统可能会根据这些建议，在一组能够提供同等的数据流服务，同时有具有不同的性能特征的不同协议中做出选择。方法setPerformancePreferences(int connectionTime,int latency,int bandwidth)设置连接时间，延迟和带宽，底层会根据这些参数设置选择合适的协议。

12. 关闭连接：网络协议通常明确指定了谁来关闭连接。但直接调用Socket的close()将会同时终止输出和输入两个方向的数据流，如客户端发送完数据后调用close()方法就会导致接收不到数据，需要一种方法来告诉连接的另一端我已经发送完所有数据，并且保持能够接收数据的能力。套接字里面就提供了这样的功能，Socket类的shutdonwInput()和shutdownOutput()方法能够将输入输出流单独关闭。

    - 调用shutdownInput()后，套接字的输入流将无法使用，任何没有发送的数据都将被丢弃，任何想从套接字的输入流读取数据的操作都将返回-1。

    - 调用shutdownOutput()方法时，套接字的输出流将无法再发送数据，任何尝试向输出流写数据的操作都将抛出异常IOException异常。

      ![](https://i.loli.net/2019/06/27/5d145d77082d971237.png)

下面考虑另一种协议：假设你需要一个压缩服务器，将接收到的字节流压缩后，发回给客户端。这种情况下应该由哪一端来关闭连接呢？

由于从客户端发来的字节流的长度是任意的，客户端需要关闭连接以通知服务器要压缩的字节流已经发送完毕，那么客户端应该什么时候调用close()方法呢？

如果客户端在其发送完最后一个字节后立即调用套接字的c1ose(),它将无法接收到压缩后数据的最后一些字节，或许客户端可以像回显协议那样，在接收完所有压缩后的数据才关闭连接，但不幸的是，这样一来服务器和客户端都不知道到底有多少数据要接收，因此这也不可行。我们需要一种方法来告诉连接的另一端“我已经发送完所有数据”同时还要保持接收数据的能力。

幸运的是套接字提供了一种实现这个功能的方法，Socket类的shutdownInput()和shutdownOutput()方法能够将綸入输出流相互独立地关闭。调用shutdownInput()后，套接字的输人流将无法使用。任何没有发送的数据都将毫无提示地被丢弃，任何想从套接字的输人流读取数据的搡作都将返回-1，当Socket调用shutdownOutputO方法后，套接字的输出流将无法再发送数据，任何尝试向输出流写数据的操作都将抛出一个IOExceptlon异常，在调用shutdownOutput()之前写出的敫据可能能够被远程套接字读取，之后，在远程套接字输入流上的读操作将返回应用程序调用shutdownOutputO后还能继续从套接字读取数据，类似的，在调用shutdownlnputO后也能够继续写数据。

在压缩协议中，客户端向服务器发送待压缩的字节，发送完成后调用shutdownOutput()关闭输出流，并从服务器读取压缩后的字节流，脤务器反复地获取未压缩的数据，并将压缩后的数据发回给客户端，直到客户端执行了停机操作，导致脤务器的read操作返回-1，这表示数据流的结束，然后脤务器关闭连接并退出。

![](https://i.loli.net/2019/06/27/5d145ec99b8f675751.png)

```java
import java.net.Socket;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;

/* WARNING: this code can deadlock if a large file (more than a few
 * 10's of thousands of bytes) is sent.
 */

public class CompressClient {

  public static final int BUFSIZE = 256;  // Size of read buffer

  public static void main(String[] args) throws IOException {

    if (args.length != 3) { // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Server> <Port> <File>");
    }

    String server = args[0];               // Server name or IP address
    int port = Integer.parseInt(args[1]);  // Server port
    String filename = args[2];             // File to read data from

    // Open input and output file (named input.gz)
    FileInputStream fileIn = new FileInputStream(filename);
    FileOutputStream fileOut = new FileOutputStream(filename + ".gz");
  
    // Create socket connected to server on specified port
    Socket sock = new Socket(server, port);

    // Send uncompressed byte stream to server
    sendBytes(sock, fileIn);

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

  private static void sendBytes(Socket sock, InputStream fileIn) 
      throws IOException {
    OutputStream sockOut = sock.getOutputStream();
    int bytesRead;                      // Number of bytes read
    byte[] buffer = new byte[BUFSIZE];  // Byte buffer
    while ((bytesRead = fileIn.read(buffer)) != -1) {
      sockOut.write(buffer, 0, bytesRead);
      System.out.print("W");   // Writing progress indicator
    }
    sock.shutdownOutput();     // Finished sending
  }
}

```

压缩协议

```java
import java.net.Socket;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.zip.GZIPOutputStream;
import java.util.logging.Logger;
import java.util.logging.Level;

public class CompressProtocol implements Runnable {

  public static final int BUFSIZE = 1024;   // Size of receive buffer
  private Socket clntSock;
  private Logger logger;

  public CompressProtocol(Socket clntSock, Logger logger) {
    this.clntSock = clntSock;
    this.logger = logger;
  }

  public static void handleCompressClient(Socket clntSock, Logger logger) {
    try {
      // Get the input and output streams from socket
      InputStream in = clntSock.getInputStream();
      GZIPOutputStream out = new GZIPOutputStream(clntSock.getOutputStream());

      byte[] buffer = new byte[BUFSIZE];   // Allocate read/write buffer
      int bytesRead;                       // Number of bytes read
      // Receive until client closes connection, indicated by -1 return
      while ((bytesRead = in.read(buffer)) != -1)
        out.write(buffer, 0, bytesRead);
      out.finish();      // Flush bytes from GZIPOutputStream

      logger.info("Client " + clntSock.getRemoteSocketAddress() + " finished");
    } catch (IOException ex) {
      logger.log(Level.WARNING, "Exception in echo protocol", ex);
    }

    try {  // Close socket
      clntSock.close();
    } catch (IOException e) {
      logger.info("Exception = " +  e.getMessage());
    }
  }
  
  public void run() {
    handleCompressClient(this.clntSock, this.logger);
  }
}

```

### NIO

NIO使用请参看[NIO章节](https://hanyunpeng0521.github.io/2019/04/Java-%E9%AB%98%E7%BA%A7-3/)

示例：

协议接口：

```java
import java.nio.channels.SelectionKey;
import java.io.IOException;

public interface TCPProtocol {
  void handleAccept(SelectionKey key) throws IOException;
  void handleRead(SelectionKey key) throws IOException;
  void handleWrite(SelectionKey key) throws IOException;
}
```

EchoSelector协议实现

```java
import java.nio.channels.SelectionKey;
import java.nio.channels.SocketChannel;
import java.nio.channels.ServerSocketChannel;
import java.nio.ByteBuffer;
import java.io.IOException;

public class EchoSelectorProtocol implements TCPProtocol {

  private int bufSize; // Size of I/O buffer

  public EchoSelectorProtocol(int bufSize) {
    this.bufSize = bufSize;
  }

  public void handleAccept(SelectionKey key) throws IOException {
    SocketChannel clntChan = ((ServerSocketChannel) key.channel()).accept();
    clntChan.configureBlocking(false); // Must be nonblocking to register
    // Register the selector with new channel for read and attach byte buffer
    clntChan.register(key.selector(), SelectionKey.OP_READ, ByteBuffer
        .allocate(bufSize));
  }

  public void handleRead(SelectionKey key) throws IOException {
    // Client socket channel has pending data
    SocketChannel clntChan = (SocketChannel) key.channel();
    ByteBuffer buf = (ByteBuffer) key.attachment();
    long bytesRead = clntChan.read(buf);
    if (bytesRead == -1) { // Did the other end close?
      clntChan.close();
    } else if (bytesRead > 0) {
      // Indicate via key that reading/writing are both of interest now.
      key.interestOps(SelectionKey.OP_READ | SelectionKey.OP_WRITE);
    }
  }

  public void handleWrite(SelectionKey key) throws IOException {
    /*
     * Channel is available for writing, and key is valid (i.e., client channel
     * not closed).
     */
    // Retrieve data read earlier
    ByteBuffer buf = (ByteBuffer) key.attachment();
    buf.flip(); // Prepare buffer for writing
    SocketChannel clntChan = (SocketChannel) key.channel();
    clntChan.write(buf);
    if (!buf.hasRemaining()) { // Buffer completely written?
      // Nothing left, so no longer interested in writes
      key.interestOps(SelectionKey.OP_READ);
    }
    buf.compact(); // Make room for more data to be read in
  }

}
```

TCP的非阻塞客户端实现：

```java
import java.net.InetSocketAddress;
import java.net.SocketException;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class TCPEchoClientNonblocking {

  public static void main(String args[]) throws Exception {

    if ((args.length < 2) || (args.length > 3)) // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Server> <Word> [<Port>]");

    String server = args[0]; // Server name or IP address
    // Convert input String to bytes using the default charset
    byte[] argument = args[1].getBytes();

    int servPort = (args.length == 3) ? Integer.parseInt(args[2]) : 7;

    // Create channel and set to nonblocking
    SocketChannel clntChan = SocketChannel.open();
    clntChan.configureBlocking(false);

    // Initiate connection to server and repeatedly poll until complete
    if (!clntChan.connect(new InetSocketAddress(server, servPort))) {
      while (!clntChan.finishConnect()) {
        System.out.print(".");  // Do something else
      }
    }
    ByteBuffer writeBuf = ByteBuffer.wrap(argument);
    ByteBuffer readBuf = ByteBuffer.allocate(argument.length);
    int totalBytesRcvd = 0; // Total bytes received so far
    int bytesRcvd; // Bytes received in last read
    while (totalBytesRcvd < argument.length) {
      if (writeBuf.hasRemaining()) {
        clntChan.write(writeBuf);
      }
      if ((bytesRcvd = clntChan.read(readBuf)) == -1) {
        throw new SocketException("Connection closed prematurely");
      }
      totalBytesRcvd += bytesRcvd;
      System.out.print(".");   // Do something else
    }

    System.out.println("Received: " +  // convert to String per default charset
         new String(readBuf.array(), 0, totalBytesRcvd));
    clntChan.close();
  }
}

```

TCP的Selector服务端实现：

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.util.Iterator;

public class TCPServerSelector {

  private static final int BUFSIZE = 256;  // Buffer size (bytes)
  private static final int TIMEOUT = 3000; // Wait timeout (milliseconds)

  public static void main(String[] args) throws IOException {

    if (args.length < 1) { // Test for correct # of args
      throw new IllegalArgumentException("Parameter(s): <Port> ...");
    }

    // Create a selector to multiplex listening sockets and connections
    Selector selector = Selector.open();

    // Create listening socket channel for each port and register selector
    for (String arg : args) {
      ServerSocketChannel listnChannel = ServerSocketChannel.open();
      listnChannel.socket().bind(new InetSocketAddress(Integer.parseInt(arg)));
      listnChannel.configureBlocking(false); // must be nonblocking to register
      // Register selector with channel. The returned key is ignored
      listnChannel.register(selector, SelectionKey.OP_ACCEPT);
    }

    // Create a handler that will implement the protocol
    TCPProtocol protocol = new EchoSelectorProtocol(BUFSIZE);

    while (true) { // Run forever, processing available I/O operations
      // Wait for some channel to be ready (or timeout)
      if (selector.select(TIMEOUT) == 0) { // returns # of ready chans
        System.out.print(".");
        continue;
      }

      // Get iterator on set of keys with I/O to process
      Iterator<SelectionKey> keyIter = selector.selectedKeys().iterator();
      while (keyIter.hasNext()) {
        SelectionKey key = keyIter.next(); // Key is bit mask
        // Server socket channel has pending connection requests?
        if (key.isAcceptable()) {
          protocol.handleAccept(key);
        }
        // Client socket channel has pending data?
        if (key.isReadable()) {
          protocol.handleRead(key);
        }
        // Client socket channel is available for writing and
        // key is valid (i.e., channel not closed)?
        if (key.isValid() && key.isWritable()) {
          protocol.handleWrite(key);
        }
        keyIter.remove(); // remove from set of selected keys
      }
    }
  }
}

```

UDP的服务端实现

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.util.Iterator;

public class UDPEchoServerSelector {

  private static final int TIMEOUT = 3000; // Wait timeout (milliseconds)

  private static final int ECHOMAX = 255; // Maximum size of echo datagram

  public static void main(String[] args) throws IOException {

    if (args.length != 1) // Test for correct argument list
      throw new IllegalArgumentException("Parameter(s): <Port>");

    int servPort = Integer.parseInt(args[0]);

    // Create a selector to multiplex client connections.
    Selector selector = Selector.open();

    DatagramChannel channel = DatagramChannel.open();
    channel.configureBlocking(false);
    channel.socket().bind(new InetSocketAddress(servPort));
    channel.register(selector, SelectionKey.OP_READ, new ClientRecord());

    while (true) { // Run forever, receiving and echoing datagrams
      // Wait for task or until timeout expires
      if (selector.select(TIMEOUT) == 0) {
        System.out.print(".");
        continue;
      }

      // Get iterator on set of keys with I/O to process
      Iterator<SelectionKey> keyIter = selector.selectedKeys().iterator();
      while (keyIter.hasNext()) {
        SelectionKey key = keyIter.next(); // Key is bit mask

        // Client socket channel has pending data?
        if (key.isReadable())
          handleRead(key);

        // Client socket channel is available for writing and
        // key is valid (i.e., channel not closed).
        if (key.isValid() && key.isWritable())
          handleWrite(key);

        keyIter.remove();
      }
    }
  }

  public static void handleRead(SelectionKey key) throws IOException {
    DatagramChannel channel = (DatagramChannel) key.channel();
    ClientRecord clntRec = (ClientRecord) key.attachment();
    clntRec.buffer.clear();    // Prepare buffer for receiving
    clntRec.clientAddress = channel.receive(clntRec.buffer);
    if (clntRec.clientAddress != null) {  // Did we receive something?
      // Register write with the selector
      key.interestOps(SelectionKey.OP_WRITE);
    }
  }

  public static void handleWrite(SelectionKey key) throws IOException {
    DatagramChannel channel = (DatagramChannel) key.channel();
    ClientRecord clntRec = (ClientRecord) key.attachment();
    clntRec.buffer.flip(); // Prepare buffer for sending
    int bytesSent = channel.send(clntRec.buffer, clntRec.clientAddress);
    if (bytesSent != 0) { // Buffer completely written?
      // No longer interested in writes
      key.interestOps(SelectionKey.OP_READ);
    }
  }

  static class ClientRecord {
    public SocketAddress clientAddress;
    public ByteBuffer buffer = ByteBuffer.allocate(ECHOMAX);
  }
}
```

