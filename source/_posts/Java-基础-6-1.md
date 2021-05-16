---
title: Java Thread （一）
date: 2019-04-06 19:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 基础
---
### 前言

用多线程只有一个目的，那就是更好的利用cpu的资源，因为所有的多线程代码都可以用单线程来实现。说这个话其实只有一半对，因为反应“多角色”的程序代码，最起码每个角色要给他一个线程吧，否则连实际场景都无法模拟，当然也没法说能用单线程来实现：比如最常见的“生产者，消费者模型”。

<!--more-->
#### 进程和线程

进程是指一个内存中运行的应用程序，每个进程都有自己独立的一块内存空间，即**进程空间**或（虚空间）。进程不依赖于线程而独立存在，一个进程中可以启动多个线程。比如在Windows系统中，一个运行的exe就是一个进程。

线程是指进程中的一个执行流程，**一个进程中可以运行多个线程**。比如java.exe进程中可以运行很多线程。**线程总是属于某个进程，线程没有自己的虚拟地址空间，与进程内的其他线程一起共享分配给该进程的所有资源**。

“同时”执行是人的感觉，在线程之间实际上轮换执行。

进程在执行过程中拥有独立的内存单元，进程有独立的地址空间，而多个线程共享内存，从而极大地提高了程序的运行效率。

线程在执行过程中与进程还是有区别的。**每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。**

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动，进程是系统进行**资源分配和调度**的一个独立单位。

线程是进程的一个实体，是**CPU调度和分派的基本单位**，它是比进程更小的能独立运行的基本单位。线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源（如程序计数器,一组寄存器和栈），但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程包含以下内容：

- 一个指向当前被执行指令的指令指针；
- 一个栈；
- 一个寄存器值的集合，定义了一部分描述正在执行线程的处理器状态的值
- 一个私有的数据区。

我们使用Join()方法挂起当前线程，直到调用Join()方法的线程执行完毕。该方法还存在包含参数的重载版本，其中的参数用于指定等待线程结束的最长时间（即超时）所花费的毫秒数。如果线程中的工作在规定的超时时段内结束，该版本的Join()方法将返回一个布尔量True。

简而言之：

1. 一个程序至少有一个进程，一个进程至少有一个线程。
2. 线程的划分尺度小于进程，使得多进程程序的并发性高。
3. 另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
4. 线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。
5. 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

在Java中，每次程序运行至少启动2个线程：**一个是main线程，一个是垃圾收集线程**。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM实际上就是在操作系统中启动了一个进程。


#### 基本概念

很多人都对其中的一些概念不够明确，如同步、并发等等，让我们先建立一个数据字典，以免产生误会。

1. 多线程。指的是这个程序（一个进程）运行时产生了不止一个线程

2.  并行与并发。
       - 并行：多个cpu实例或者多台机器**同时执行一段处理逻辑**，是真正的同时。
       - 并发：通过cpu调度算法，让用户看上去同时执行，实际上**从cpu操作层面不是真正的同时**。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS或者QPS来反应这个系统的处理能力。

并发和并行的区别：

![并发并行.jpeg](https://i.loli.net/2019/04/14/5cb298c742af0.jpeg)

3. 线程安全。
经常用来描绘一段代码。**指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果**。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。反过来，**线程不安全就意味着线程的调度顺序会影响最终结果**，如不加事务的转账代码：
```
void transferMoney(User from, User to, float amount){
  to.setMoney(to.getBalance() + amount);
  from.setMoney(from.getBalance() - amount);
}
```

4. 同步。
Java中的同步指的是**通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确**。如上面的代码简单加入synchronized关键字。在保证结果准确的同时，提高性能，才是优秀的程序。线程安全的优先级高于性能。


### 线程

在Java中，“线程”指两件不同的事情：

1. java.lang.Thread类的一个实例；
2. 线程的执行。

在 Java程序中，有四种方法创建线程：

- 继承Thread类；
- 实现Runnable接口；
- 实现Callable接口通过FutureTask包装器来创建Thread线程；
- 使用ExecutorService、Callable、Future实现有返回结果的多线程。

其中前两种方式线程执行完后都没有返回值，后两种是带返回值的。

使用java.lang.Thread类或者java.lang.Runnable接口编写代码来定义、实例化和启动新线程。一个Thread类实例只是一个对象，像Java中的任何其他对象一样，具有变量和方法，生死于堆上。

Java中，每个线程都有一个调用栈，即使不在程序中创建任何新的线程，线程也在后台运行着。一个Java应用总是从main()方法开始运行，main()方法运行在一个线程内，他被称为主线程。一旦创建一个新的线程，就产生一个新的调用栈。

线程总体分两类：用户线程和守候线程。

**当所有用户线程执行完毕的时候，JVM自动关闭。但是守候线程却不独立于JVM，守候线程一般是由操作系统或者用户自己创建的**。

#### Thread方法

**构造方法：**

![](https://i.loli.net/2019/05/05/5cce8838960ec.png)

**常用方法：**

| 函数名 | 功能|
| --- | --- |
|static int activeCount() |返回当前线程的线程组中活动线程的数目。 |
|void checkAccess() |        判定当前运行的线程是否有权修改该线程。 |
| int countStackFrames() |已过时。 该调用的定义依赖于 suspend()，但它遭到了反对。此外，该调用的结果从来都不是意义明确的。 |
|static Thread currentThread() |返回对当前正在执行的线程对象的引用。|
|void destroy() |  已过时。 该方法最初用于破坏该线程，但不作任何清除。它所保持的任何监视器都会保持锁定状态。不过，该方法决不会被实现。即使要实现，它也极有可能以 suspend() 方式被死锁。如果目标线程被破坏时保持一个保护关键系统资源的锁，则任何线程在任何时候都无法再次访问该资源。如果另一个线程曾试图锁定该资源，则会出现死锁。这类死锁通常会证明它们自己是“冻结”的进程。有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。|
|static void dumpStack() |将当前线程的堆栈跟踪打印至标准错误流。 |
|static int enumerate(Thread[] tarray) |          将当前线程的线程组及其子组中的每一个活动线程复制到指定的数组中。 |
|static Map<Thread,StackTraceElement[]> getAllStackTraces() |          返回所有活动线程的堆栈跟踪的一个映射。 |
|ClassLoader getContextClassLoader() |          返回该线程的上下文 ClassLoader。 |
|static Thread.UncaughtExceptionHandler getDefaultUncaughtExceptionHandler() |          返回线程由于未捕获到异常而突然终止时调用的默认处理程序。 |
| long getId() |          返回该线程的标识符。 |
|String getName() |          返回该线程的名称。 |
| int getPriority() |          返回线程的优先级。|
| StackTraceElement[] getStackTrace() |          返回一个表示该线程堆栈转储的堆栈跟踪元素数组。 |
| Thread.State getState() |          返回该线程的状态。 |
| ThreadGroup getThreadGroup() |          返回该线程所属的线程组。 |
| Thread.UncaughtExceptionHandler getUncaughtExceptionHandler() |          返回该线程由于未捕获到异常而突然终止时调用的处理程序。 |
|static boolean holdsLock(Object obj) |          当且仅当当前线程在指定的对象上保持监视器锁时，才返回 true。 |
| void interrupt() |          中断线程。 |
|static boolean interrupted() |          测试当前线程是否已经中断。 |
| boolean isAlive() |        测试线程是否处于活动状态。 |
| boolean isDaemon() |          测试该线程是否为守护线程。 |
| boolean isInterrupted() |          测试线程是否已经中断。 |
|void join() |          等待该线程终止。 |
|void join(long millis) |          等待该线程终止的时间最长为 millis 毫秒。 |
| void join(long millis, int nanos) |          等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。 |
| void resume() |          已过时。 该方法只与 suspend() 一起使用，但 suspend() 已经遭到反对，因为它具有死锁倾向。有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。 |
| void run() |          如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，该方法不执行任何操作并返回。 |
| void setContextClassLoader(ClassLoader cl) |          设置该线程的上下文 ClassLoader。 |
| void setDaemon(boolean on) |          将该线程标记为守护线程或用户线程。 |
|static void setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh) |          设置当线程由于未捕获到异常而突然终止，并且没有为该线程定义其他处理程序时所调用的默认处理程序。 |
| void setName(String name)   |        改变线程名称，使之与参数 name 相同。 |
| void setPriority(int newPriority) |          更改线程的优先级。 |
| void setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler eh) |          设置该线程由于未捕获到异常而突然终止时调用的处理程序。 |
|static void sleep(long millis) |          在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。 |
|static void sleep(long millis, int nanos) |          在指定的毫秒数加指定的纳秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。 |
| void start() |          使该线程开始执行；Java 虚拟机调用该线程的 run 方法。 |
| void stop() |          已过时。 该方法具有固有的不安全性。用 Thread.stop 来终止线程将释放它已经锁定的所有监视器（作为沿堆栈向上传播的未检查 ThreadDeath 异常的一个自然后果）。如果以前受这些监视器保护的任何对象都处于一种不一致的状态，则损坏的对象将对其他线程可见，这有可能导致任意的行为。stop 的许多使用都应由只修改某些变量以指示目标线程应该停止运行的代码来取代。目标线程应定期检查该变量，并且如果该变量指示它要停止运行，则从其运行方法依次返回。如果目标线程等待很长时间（例如基于一个条件变量），则应使用 interrupt 方法来中断该等待。有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。|
|void stop(Throwable obj) |          已过时。 该方法具有固有的不安全性。有关详细信息，请参阅 stop()。该方法的附加危险是它可用于生成目标线程未准备处理的异常（包括若没有该方法该线程不太可能抛出的已检查的异常）。有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume。|
|void suspend() |已过时。 该方法已经遭到反对，因为它具有固有的死锁倾向。如果目标线程挂起时在保护关键系统资源的监视器上保持有锁，则在目标线程重新开始以前任何线程都不能访问该资源。如果重新开始目标线程的线程想在调用 resume 之前锁定该监视器，则会发生死锁。这类死锁通常会证明自己是“冻结”的进程。有关更多信息，请参阅为何不赞成使用 Thread.stop、Thread.suspend 和 Thread.resume|
| String toString() |          返回该线程的字符串表示形式，包括线程名称、优先级和线程组。 |
|static void yield() |          暂停当前正在执行的线程对象，并执行其他线程。 |


线程的优先级可以通过setPriority()和getPriority()来访问优先级，线程优先级界于1和10之间，缺省是5。

示例：
```java
import java.util.Random;
public class ThreadDemo2 {
    public static Random random=new Random();
    public static void print(){
        Thread curThread = Thread.currentThread() ;
        String curThreadName = curThread.getName();
        System.out.println("---------------"+curThread.getName()+"---------------");
        System.out.println("返回当前线程"+curThreadName+"的线程组中活动线程的数目："+Thread.activeCount());
        System.out.println("返回该线程"+curThreadName+"的标识符："+curThread.getId());
        System.out.println("返回线程"+curThreadName+"的优先级:"+curThread.getPriority());
        System.out.println("返回该线程"+curThreadName+"的状态:"+curThread.getState());
        System.out.println("返回该线程"+curThreadName+"所属的线程组:"+curThread.getThreadGroup());
        System.out.println("测试线程"+curThreadName+"是否处于活动状态:"+curThread.isAlive());
        System.out.println("测试线程"+curThreadName+"是否测试该线程是否为守护线程:"+curThread.isDaemon());

    }
     class ThreadA implements Runnable {
        @Override
        public void run() {
            try {
                //模拟做事情执行了1秒；//以便一会咱们的监控工具监控到啊！
                Thread.sleep((random.nextLong()%1000)+1000);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            ThreadDemo2.print();
        }
    }

    public static void main(String[] args) {
        ThreadDemo2 demo2=new ThreadDemo2();
        ThreadA threadA=demo2.new ThreadA();
        for (int i=0;i<5;i++)
        {
            new Thread(threadA,"Thread"+i).start();
        }

        ThreadDemo2.print();

        try {
            Thread.sleep((random.nextLong()%1000)+1000);//休息10s以便我们的监控工具能监控到
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
#### 线程状态

java线程状态有：NEW(新建), RUNNABLE(就绪), BLOCKED(阻塞), WAITING(等待), TIMED_WAITING(定时等待), TERMINATED(终止)。

![生命周期](https://i.loli.net/2019/05/05/5cce8482bc5c6.png)

分析wait,sleep,yield,join以及notify分别对应哪个状态?

CPU在一个时刻只能运行一个线程，当在运行一个线程的过程中转去运行另外一个线程，这个叫做**线程上下文切换**.

下次恢复时需要知道在这之前当前线程已经执行到哪条指令了，所以需要**记录程序计数器的值**，另外比如说线程正在进行某个计算的时候被挂起了，那么下次继续执行的时候需要知道之前挂起时变量的值时多少，因此需要**记录CPU寄存器的状态**。所以一般来说，线程上下文切换过程中会记录程序计数器、CPU寄存器状态等数据。

程序计数器:记录线程执行到哪条指令了；
CPU寄存器：记录线程挂起时变量值

1. start():
当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。由**新建**进入**就绪**

2. run():
当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。由**就绪**进入**执行**。

3. sleep：

   sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。

   但是有一点要非常注意，sleep方法**不会释放锁**，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。由**运行**进入**定时等待**状态。

4. yield：

   调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样**不会释放锁**。但是yield  **不能控制具体的交出CPU的时间**，另外，yield方法**只能让拥有相同优先级的线程有获取CPU执行时间的机会**。注意，调用yield方法并不会让线程进入阻塞状态，而是让线程 **重回就绪状态** ，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。由**运行**进入**就绪**状态。

5. join：
    假如在main线程中，**调用thread.join方法**，则main方法会等待**thread线程**执行完毕或者等待一定的时间。如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的事件。内部使用wait方法来实现，由**执行**进入**等待/定时等待**。
    
    join()方法是等待这个线程结束，完成其执行。它的主要起同步作用，使线程之间的执行从“并行”变成“串行”。
    
    也就是说，当我们在线程A中调用了线程B的join()方法时，线程执行过程发生改变：线程A，必须等待线程B执行完毕后，才可以继续执行下去。
    
    join() 和 sleep() 一样，都可以被中断（被中断时，会抛出 InterrupptedException 异常）；不同的是，join() 内部调用了 wait()，会出让锁，而 sleep() 会一直保持锁。

6. interrupt：
    单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；interrupt方法不能中断正在运行中的线程，但是可以通过interrupt方法和isInterrupted()方法来停止正在运行的线程(不推荐，使用标志位替代)。由**执行**进入**终止**。

7. wait

    wait()方法的作用是让当前线程进入等待状态，**会把当前的锁释放**，wait()会与notify()和notifyAll()方法一起使用。wait运行线程由**执行**进入**等待/定时等待**。

      **wait()方法和join()方法的相似处**

      - wait()和join()方法都用于暂停Java中的当前线程，进入等待状态。

      - 在Java中都可以调用interrupt()方法中断wait()和join()的线程状态。

      - wait()和join()都是非静态方法。

      - wait()和join()都在Java中重载。wait()和join（）没有超时，但接受超时参数。

      尽管wait()方法和join()方法有相似之处，但wait()方法和join()方法还**是存在差异的**。

      - 存在不同的java包中（最明显的区别）

        wait()方法需要在java.lang.Object类中声明；而，join()方法是在java.lang.Thread类中声明。

      - 使用目的不同

        wait()方法用于线程间通信；而join()方法用于在多个线程之间添加排序，第二个线程需要在第一个线程执行完成后才能开始执行。

      - 唤醒线程方面的区别

        我们可以通过使用notify()和notifyAll()方法启动一个通过wait()方法进入等待状态的线程。但是我们不能打破join()方法所施加的等待，除非或者中断调用了连接的线程已执行完了。

      - 同步上下文（最重要的区别）

        wait()方法必须从同步（synchronized）的上下文调用，即同步块或方法，否则会抛出IllegalMonitorStateException异常。但，在Java中有或没有同步的上下文，我们都可以调用join()方法。

8. notify、notifyAll

   notify()和notifyAll()方法的作用是唤醒等待中的线程，notify()方法：唤醒单个线程，notifyAll()方法：唤醒所有线程。

   notify/notifyAll()执行后，并不立即释放锁，而是要等到执行完临界区中代码后，再释放。故，在实际编程中，我们应该尽量在线程调用notify/notifyAll()后，立即退出临界区。即不要在notify/notifyAll()后面再写一些耗时的代码

9. setDaemon和isDaemon：
    用来**设置线程是否成为守护线程和判断线程是否是守护线程**。
    
    守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像**垃圾收集器线程就是守护线程**。


#### 守护线程

Java分为两种线程：用户线程和守护线程

所谓守护线程是指在程序运行的时候在后台提供一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者，并且这种线程并不属于程序中不可或缺的部分。因此，当所有的非守护线程结束时，程序也就终止了，同时会杀死进程中的所有守护线程。反过来说，只要任何非守护线程还在运行，程序就不会终止。

守护线程和用户线程的没啥本质的区别，唯一的不同之处就在于虚拟机的离开：如果用户线程已经全部退出运行了，只剩下守护线程存在了，虚拟机也就退出了。 因为没有了被守护者，守护线程也就没有工作可做了，也就没有继续运行程序的必要了。

将线程转换为守护线程可以通过调用Thread对象的setDaemon(true)方法来实现。在使用守护线程时需要注意一下几点：

1. thread.setDaemon(true)**必须在thread.start()之前设置**，否则会跑出一个IllegalThreadStateException异常。你不能把正在运行的常规线程设置为守护线程。

2. 在Daemon线程中产生的新线程也是Daemon的。

3. 守护线程应该**永远不去访问固有资源**，如文件、数据库，因为它会在任何时候甚至在一个操作的中间发生中断。

JRE判断程序是否执行结束的标准是所有的前台执行线程结束而不管后台线程的状态；

如果进程中所有非守护线程已结束或者退出时，即使仍有守护线程在运行，进程仍将结束，即当程序结束时，守护线程可能依然未退出。

示例：
```java
public class ThreadDemo3 {

    class ThreadA extends Thread {
        private int times;


        public ThreadA(String name, int times) {
            super(name);
            if (times>0) {
                this.times = times;
            }
        }


        @Override
        public void run() {
            for (long i = 0; i < times; i++) {
                System.out.println("后台线程"+getName()+"第" + i + "次执行！");
                try {
                    Thread.sleep(7);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        ThreadDemo3 demo3=new ThreadDemo3();
        Thread tA = demo3.new ThreadA("A",20);
        Thread tB = demo3.new ThreadA("B",10);
        tA.setDaemon(true); // 设置为守护线程,注意一定要在开始之前调用

        tB.start();
        tA.start();
        Thread mainThread = Thread.currentThread();
        System.out.println("线程A是不是守护线程:"+tA.isDaemon());
        System.out.println("线程b是不是守护线程:"+tB.isDaemon());
        System.out.println("线程main是不是守护线程:"+mainThread.isDaemon());
    }
}
//可以看到B线程并没有结束运行
```

#### 线程中断机制

必须记住，中断是一种协作机制。当一个线程中断另一个线程时，被中断的线程不一定要立即停止正在做的事情。相反，中断是礼貌地请求另一个线程在它愿意并且方便的时候停止它正在做的事情。有些方法，例如 Thread.sleep()，很认真地对待这样的请求，但并不是每个方法一定要对中断作出响应。 您可以随意忽略中断请求，但是这样做的话会影响响应。   

**中断状态**

每个Java线程都有一个与之相关联的 Boolean 属性，该属性用于表示线程的中断状态，Thread类提供了三个方法用于操作中断状态，这些方法包括：

1. public static boolean interrupted():测试当前线程是否已经中断。线程的**中断状态 由该方法清除**。换句话说，如果连续两次调用该方法，则第二次调用将返回 false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。

2. public boolean isInterrupted() :测试线程是否已经中断。线程的中断状态不受该方法的影响。

3. public void interrupt():中断线程。

三个方法中，interrupt()方法是唯一能将中断状态设置为true的方法，另外两个方法都是用于检测当前中断状态的方法。

**中断的处理**

作为一种协作机制，中断机制不会强求被中断线程一定要在某个点进行处理。实际上，被中断线程只需在合适的时候处理中断即可，如果没有合适的时间点，甚至可以不处理，这时候在任务处理层面，就跟没有调用中断方法一样。

在JDK中，有很多阻塞方法的声明中有抛出InterruptedException异常，这暗示该方法是可中断的，这些方法会检测当前线程是否被中断，如果是，则立刻结束阻塞方法，并抛出InterruptedException异常。如果程序捕获到这些可中断的阻塞方法抛出的InterruptedException或检测到中断后，这些中断信息该如何处理？一般有以下两个通用原则：

- 如果遇到的是可中断的阻塞方法抛出InterruptedException，可以继续向方法调用栈的上层抛出该异常，如果是检测到中断，则可清除中断状态并抛出InterruptedException，使当前方法也成为一个可中断的方法。

- 若有时候不太方便在方法上抛出InterruptedException，比如要实现的某个接口中的方法签名上没有throws InterruptedException，这时就可以捕获可中断方法的InterruptedException并通过Thread.currentThread.interrupt()来重新设置中断状态。如果是检测并清除了中断状态，亦是如此。    

一般的代码中，尤其是作为一个基础类库时，绝不应当吞掉中断，即捕获到InterruptedException后在catch里什么也不做，清除中断状态后又不重设中断状态也不抛出InterruptedException等。因为吞掉中断状态会导致方法调用栈的上层得不到这些信息。

当然，凡事总有例外的时候，当你完全清楚自己的方法会被谁调用，而调用者也不会因为中断被吞掉了而遇到麻烦，就可以这么做。

**不可取消的任务**

有些任务拒绝被中断，这使得它们是不可取消的。但是，即使是不可取消的任务也应该尝试保留中断状态，以防在不可取消的任务结束之后，调用栈上更高层的代码需要对中断进行处理。以下代码展示了一个方法，该方法等待一个阻塞队列，直到队列中出现一个可用项目，而不管它是否被中断。为了方便他人，它在结束后在一个 finally 块中恢复中断状态，以免剥夺中断请求的调用者的权利。

示例：

```java
public class ThreadInterruptDemo implements Runnable{
    public static void main(String[] args) throws Exception {
        Thread thread = new Thread(new ThreadInterruptDemo(), "InterruptDemo Thread");
        System.out.println("Starting thread...");
        thread.start();
        Thread.sleep(3000);
        System.out.println("Interrupting thread...");
        thread.interrupt();
        System.out.println("线程是否中断：" + thread.isInterrupted());
        Thread.sleep(3000);
        System.out.println("Stopping application...");
    }
    @Override
    public void run() {
        boolean stop = false;
        while (!stop) {
            System.out.println("My Thread is running...");

            long time = System.currentTimeMillis();
            while ((System.currentTimeMillis() - time < 1000)) {
                // 让该循环持续一段时间，使上面的话打印次数少点
            }
             if (Thread.currentThread().isInterrupted()) {
             // 需要线程本身去处理一下它的终止状态
             break;
             }
//            try {
//                Thread.sleep(3L);
//                if (Thread.currentThread().isInterrupted()) {
//                    System.out.println("tttttt");
//                }
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//                break;
//            }
        }
        System.out.println("My Thread exiting under request...");
    }
}
```

### 线程实现

Java多线程实现的方式有四种

1. 继承Thread类，重写run方法
2. 实现Runnable接口，重写run方法，实现Runnable接口的实现类的实例对象作为Thread构造函数的target
3. 通过Callable和FutureTask创建线程
4. 通过线程池创建线程

前面两种可以归结为一类：无返回值，原因很简单，通过重写run方法，run方式的返回值是void，所以没有办法返回结果

后面两种可以归结成一类：有返回值，通过Callable接口，就要实现call方法，这个方法的返回值是Object，所以返回的结果可以放在Object对象中

#### 继承Thread类创建线程

Thread类本质上是实现了Runnable接口的一个实例，代表一个线程的实例。启动线程的唯一方法就是通过Thread类的start()实例方法。start()方法是一个native方法，它将启动一个新线程，并执行run()方法。这种方式实现多线程很简单，通过自己的类直接extend Thread，并复写run()方法，就可以启动新线程并执行自己定义的run()方法。

```java
public class CreateThreadDemo extends Thread {

    private int times;


    public CreateThreadDemo(String name, int times) {
        super(name);
        if (times > 0) {
            this.times = times;
        }
    }

    @Override
    public void run() {
        // 每隔1s中输出一次当前线程的名字
        for (int i = 0; i < times; i++) {
            // 输出线程的名字，与主线程名称相区分
            printThreadInfo();
            try {
                // 线程休眠一秒
                Thread.sleep(1000);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }


    }

    public static void main(String[] args) throws Exception {
        // 注意这里，要调用start方法才能启动线程，不能调用run方法
        new CreateThreadDemo("myThread", 5).start();
        // 演示主线程继续向下执行

        for (int i = 0; i < 6; i++) {
            printThreadInfo();
            Thread.sleep(1000);
        }


    }

    /**
     * 输出当前线程的信息
     */
    private synchronized static void printThreadInfo() {
        Thread curThread = Thread.currentThread();
        String curThreadName = curThread.getName();
        System.out.println("---------------" + curThread.getName() + "---------------");
        System.out.println("返回当前线程" + curThreadName + "的线程组中活动线程的数目：" + Thread.activeCount());
        System.out.println("返回该线程" + curThreadName + "的标识符：" + curThread.getId());
        System.out.println("返回线程" + curThreadName + "的优先级:" + curThread.getPriority());
        System.out.println("返回该线程" + curThreadName + "的状态:" + curThread.getState());
        System.out.println("返回该线程" + curThreadName + "所属的线程组:" + curThread.getThreadGroup());
        System.out.println("测试线程" + curThreadName + "是否处于活动状态:" + curThread.isAlive());
        System.out.println("测试线程" + curThreadName + "是否测试该线程是否为守护线程:" + curThread.isDaemon());
    }
}

```

这里要注意，在启动线程的时候，并不是调用线程类的run方法，而是调用了线程类的start方法。那么我们能不能调用run方法呢？

答案是肯定的，因为run方法是一个public声明的方法，因此我们是可以调用的，但是如果我们调用了run方法，那么这个方法将会作为一个普通的方法被调用，并不会开启线程。这里实际上是采用了设计模式中的**模板方法**模式，Thread类作为模板，而run方法是在变化的，因此放到子类来实现。


#### 实现Runnable接口创建线程

如果自己的类已经extends另一个类，就无法直接extends Thread，此时，可以实现一个Runnable接口，为了启动MyThread，需要首先实例化一个Thread，并传入自己的MyThread实例。

使用Runnable实现上面的例子步骤如下：

-  定义一个类实现Runnable接口，作为线程任务类
-  重写run方法，并实现方法体，方法体的代码就是线程所执行的代码
-  定义一个可以运行的类，并在main方法中创建线程任务类
-  创建Thread类，并将线程任务类做为Thread类的构造方法传入
-  启动线程


```java
public class CreateThreadDemo_Task implements Runnable {

    private int times;

    public CreateThreadDemo_Task(int times) {
        if (times > 0) {
            this.times = times;
        }
    }

    @Override
    public void run() {
        // 每隔1s中输出一次当前线程的名字
        for (int i = 0; i < times; i++) {
            // 输出线程的名字，与主线程名称相区分
            printThreadInfo();
            try {
                // 线程休眠一秒
                Thread.sleep(1000);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    }

    public static void main(String[] args) throws Exception {
        // 实例化线程任务类
        CreateThreadDemo_Task task = new CreateThreadDemo_Task(5);

        // 创建线程对象,并将线程任务类作为构造方法参数传入
        Thread thread = new Thread(task, "test01");

        thread.start();
        for (int i = 0; i < 6; i++) {
            printThreadInfo();
            Thread.sleep(1000);
        }
    }

    /**
     * 输出当前线程的信息
     */
    private synchronized static void printThreadInfo() {
        Thread curThread = Thread.currentThread();
        String curThreadName = curThread.getName();
        System.out.println("---------------" + curThread.getName() + "---------------");
        System.out.println("返回当前线程" + curThreadName + "的线程组中活动线程的数目：" + Thread.activeCount());
        System.out.println("返回该线程" + curThreadName + "的标识符：" + curThread.getId());
        System.out.println("返回线程" + curThreadName + "的优先级:" + curThread.getPriority());
        System.out.println("返回该线程" + curThreadName + "的状态:" + curThread.getState());
        System.out.println("返回该线程" + curThreadName + "所属的线程组:" + curThread.getThreadGroup());
        System.out.println("测试线程" + curThreadName + "是否处于活动状态:" + curThread.isAlive());
        System.out.println("测试线程" + curThreadName + "是否测试该线程是否为守护线程:" + curThread.isDaemon());
    }
}
```
事实上，当传入一个Runnable target参数给Thread后，Thread的run()方法就会调用target.run()，参考JDK源代码：
```java
public void run() {  
　　if (target != null) {  
　　 target.run();  
　　}  
}  
```
其实Runnable就是一个线程任务，线程任务和线程的控制分离，这也就是上面所说的解耦。**我们要实现一个线程，可以借助Thread类，Thread类要执行的任务就可以由实现了Runnable接口的类来处理。** 这就是Runnable的精髓之所在！

Runnable 是一个@FunctionalInterface 函数式接口，也就意味了可以利用JDK8提供的lambda的方式来创建线程任务

线程任务和线程的控制分离，那么一个线程任务可以提交给多个线程来执行。

这是很有用的，比如车站的售票窗口，每个窗口可以看做是一个线程，他们每个窗口做的事情都是一样的，也就是售票。这样我们程序在模拟现实的时候就可以定义一个售票任务，让多个窗口同时执行这一个任务。那么如果要改动任务执行计划，只要修改线程任务类，所有的线程就都会按照修改后的来执行。相比较继承Thread类的方式来创建线程的方式，实现Runnable接口是更为常用的。

**lambda方式创建线程任务**

这里就是为了简化内部类的编写，简化了大量的模板代码，显得更加简洁。如果读者看不明白，可以读完内部类方式之后，回过来再看这段代码。

```java
public class CreateThreadDemo {

    public static void main(String[] args) throws Exception {
        // 使用lambda的形式实例化线程任务类
        Runnable task = () -> {
            while (true) {
                // 输出线程的名字
                printThreadInfo();
            }
        };

        // 创建线程对象,并将线程任务类作为构造方法参数传入
        new Thread(task).start();

        // 主线程的任务，为了演示多个线程一起执行
        while (true) {
            printThreadInfo();
            Thread.sleep(1000);
        }
    }

    /**
     * 输出当前线程的信息
     */
    private static void printThreadInfo() {
        System.out.println("当前运行的线程名为： " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### 使用内部类的方式
这并不是一种新的实现线程的方式，只是另外的一种写法。比如有些情况我们的线程就想执行一次，以后就用不到了。那么像上面两种方式（继承Thread类和实现Runnable接口）都还要再定义一个类，显得比较麻烦，我们就可以通过匿名内部类的方式来实现。使用内部类实现依然有两种，分别是继承Thread类和实现Runnable接口。

```java
public class CreateThreadDemo {

    public static void main(String[] args) {
        // 基于子类的方式
        new Thread() {
            @Override
            public void run() {
                while (true) {
                    printThreadInfo();
                }
            }
        }.start();

        // 基于接口的实现
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    printThreadInfo();
                }
            }
        }).start();
    }

    /**
     * 输出当前线程的信息
     */
    private static void printThreadInfo() {
        System.out.println("当前运行的线程名为： " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```
如果通过上面的构造方法传入target，那么就会执行target中的run方法。可能有朋友就会问了，我们同时继承Thread类和实现Runnable接口，target不为空，那么为何不执行target的run呢。不要忘记了，我们在子类中已经重写了Thread类的run方法，因此run方法已经不在是我们看到的这样了。那当然也就不回执行target的run方法。


**lambda 方式改造**

刚才使用匿名内部类，会发现代码还是比较冗余的，lambda可以大大简化代码的编写。用lambda来改写上面的基于接口的形式的代码
```java
// 使用lambda的形式
new Thread(() -> {
    while (true) {
        printThreadInfo();
    }
}).start();


// 对比不使用lambda的形式
new Thread(new Runnable() {
    @Override
    public void run() {
        while (true) {
            printThreadInfo();
        }
    }
}).start();

```

#### 定时器

定时器可以说是一种基于线程的一个工具类，可以定时的来执行某个任务。在应用中经常需要定期执行一些操作，比如要在凌晨的时候汇总一些数据，比如要每隔10分钟抓取一次某个网站上的数据等等，总之计时器无处不在。

在Java中实现定时任务有很多种方式，JDK提供了Timer类来帮助开发者创建定时任务，另外也有很多的第三方框架提供了对定时任务的支持，比如Spring的schedule以及著名的quartz等等。因为Spring和quartz实现都比较重，依赖其他的包，上手稍微有些难度，不在本篇博客的讨论范围之内，这里就看一下JDK所给我们提供的API来实现定时任务。

**指定时间点执行**

```java

public class CreateThreadDemo_Timer {

    private static final SimpleDateFormat format =
            new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public static void main(String[] args) throws Exception {

        // 创建定时器
        Timer timer = new Timer();

        // 提交计划任务
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("定时任务执行了...");
            }
        }, format.parse("2017-10-11 22:00:00"));
    }
}
```

**间隔时间重复执行**

```java

public class CreateThreadDemo_Timer {

    public static void main(String[] args){

        // 创建定时器
        Timer timer = new Timer();

        // 提交计划任务
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("定时任务执行了...");
            }
        },new Date(), 1000);
    }
}
```

#### 带返回值的线程实现方式

实现Callable接口通过FutureTask包装器来创建Thread线程。

我们发现上面提到的不管是继承Thread类还是实现Runnable接口，发现有两个问题，第一个是无法抛出更多的异常，第二个是线程执行完毕之后并无法获得线程的返回值。那么下面的这种实现方式就可以完成我们的需求。这种方式的实现就是我们后面要详细介绍的Future模式，只是在jdk5的时候，官方给我们提供了可用的API，我们可以直接使用。但是使用这种方式创建线程比上面两种方式要复杂一些，步骤如下。


1. 创建Callable接口的实现类 ，并实现Call方法

2. 创建Callable实现类的实现，使用FutureTask类包装Callable对象，该FutureTask对象封装了Callable对象的Call方法的返回值

3. 使用FutureTask对象作为Thread对象的target创建并启动线程

4. 调用FutureTask对象的get()来获取子线程执行结束的返回值


Callable接口（也只有一个方法）定义如下：   

```java
public interface Callable<V>   {
  V call（） throws Exception;   
}
```
使用方法：

```java

public class CreateThreadDemo_Callable {

    public static void main(String[] args) throws Exception {

        // 创建线程任务
        Callable<Integer> call = () -> {
            System.out.println("线程任务开始执行了....");
            Thread.sleep(2000);
            return 1;
        };

        // 将任务封装为FutureTask
        FutureTask<Integer> task = new FutureTask<>(call);

        // 开启线程，执行线程任务
        new Thread(task).start();

        // ====================
        // 这里是在线程启动之后，线程结果返回之前
        System.out.println("这里可以为所欲为....");
        // ====================

        // 为所欲为完毕之后，拿到线程的执行结果
        Integer result = task.get();
        System.out.println("主线程中拿到异步任务执行的结果为：" + result);
    }
}
```

Callable中可以通过范型参数来指定线程的返回值类型。通过FutureTask的get方法拿到线程的返回值。

#### 基于线程池的方式

**使用ExecutorService、Callable、Future实现有返回结果的线程**

ExecutorService、Callable、Future三个接口实际上都是属于Executor框架。返回结果的线程是在JDK1.5中引入的新特征，有了这种特征就不需要再为了得到返回值而大费周折了。而且自己实现了也可能漏洞百出。

可返回值的任务必须实现Callable接口。类似的，无返回值的任务必须实现Runnable接口。

执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了。

>注意：get方法是阻塞的，即：线程无返回结果，get方法会一直等待。

再结合线程池接口ExecutorService就可以实现传说中有返回结果的多线程了。

下面提供了一个完整的有返回结果的多线程测试例子，在JDK1.5下验证过没问题可以直接使用。代码如下：

```java
public class CreateThreadDemo_ThreadPool {

    public static void main(String[] args) throws Exception {

        // 创建固定大小的线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);

        while (true) {
            // 提交多个线程任务，并执行
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    printThreadInfo();
                }
            });
        }
    }

    /**
     * 输出当前线程的信息
     */
    private static void printThreadInfo() {
        System.out.println("当前运行的线程名为： " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}
```

ExecutorService、Callable都是属于Executor框架。返回结果的线程是在JDK1.5中引入的新特征，还有Future接口也是属于这个框架，有了这种特征得到返回值就很方便了。
通过分析可以知道，他同样也是实现了Callable接口，实现了Call方法，所以有返回值。这也就是正好符合了前面所说的两种分类

执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了。get方法是阻塞的，即：线程无返回结果，get方法会一直等待。

上述代码中Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。

```
public static ExecutorService newFixedThreadPool(int nThreads)
创建固定数目线程的线程池。
public static ExecutorService newCachedThreadPool()
创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。
public static ExecutorService newSingleThreadExecutor()
创建一个单线程化的Executor。
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。
```
ExecutoreService提供了submit()方法，传递一个Callable，或Runnable，返回Future。如果Executor后台线程池还没有完成Callable的计算，这调用返回Future对象的get()方法，会阻塞直到计算完成。


#### 常见问题

1. 线程的名字，一个运行中的线程总是有名字的，名字有两个来源，一个是虚拟机自己给的名字，一个是你自己的定的名字。在没有指定线程名字的情况下，虚拟机总会为线程指定名字，并且主线程的名字总是mian，非主线程的名字不确定。
2. 线程都可以设置名字，也可以获取线程的名字，连主线程也不例外。
3. 获取当前线程的对象的方法是：Thread.currentThread()；
4. 在上面的代码中，只能保证：每个线程都将启动，每个线程都将运行直到完成。一系列线程以某种顺序启动并不意味着将按该顺序执行。对于任何一组启动的线程来说，调度程序不能保证其执行次序，持续时间也无法保证。
5. 当线程目标run()方法结束时该线程完成。
6. 一旦线程启动，它就永远不能再重新启动。只有一个新的线程可以被启动，并且只能一次。一个可运行的线程或死线程可以被重新启动。
7. 线程的调度是JVM的一部分，在一个CPU的机器上上，实际上一次只能运行一个线程。一次只有一个线程栈执行。JVM线程调度程序决定实际运行哪个处于可运行状态的线程。
众多可运行线程中的某一个会被选中做为当前线程。可运行线程被选择运行的顺序是没有保障的。
8. 尽管通常采用队列形式，但这是没有保障的。队列形式是指当一个线程完成“一轮”时，它移到可运行队列的尾部等待，直到它最终排队到该队列的前端为止，它才能被再次选中。事实上，我们把它称为可运行池而不是一个可运行队列，目的是帮助认识线程并不都是以某种有保障的顺序排列而成一个一个队列的事实。
9. 尽管我们没有无法控制线程调度程序，但可以通过别的方式来影响线程调度的方式。

### 面试题

1. Thread类的sleep()方法和对象的wait()方法都可以让线程暂停执行，它们有什么区别?

   答：sleep()方法（休眠）是线程类（Thread）的静态方法，调用此方法会让当前线程暂停执行指定的时间，将执行机会（CPU）让给其他线程，但是**对象的锁依然保持**，因此休眠时间结束后会自动恢复（线程回到就绪状态）。wait()是Object类的方法，调用对象的wait()方法导致**当前线程放弃对象的锁**（线程暂停执行），进入对象的等待池（wait pool），只有调用对象的notify()方法（或notifyAll()方法）时才能唤醒等待池中的线程进入等锁池（lock pool），如果线程重新获得对象的锁就可以进入就绪状态。

2. 线程的sleep()方法和yield()方法有什么区别？

   答：

   - sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；yield()方法只会给相同优先级或更高优先级的线程以运行的机会；
   -  线程执行sleep()方法后转入阻塞（blocked）状态，而执行yield()方法后转入就绪（ready）状态；
   - sleep()方法声明抛出InterruptedException，而yield()方法没有声明任何异常；
   - sleep()方法比yield()方法（跟操作系统CPU调度相关）具有更好的可移植性。

3. 当一个线程进入一个对象的synchronized方法A之后，其它线程是否可进入此对象的synchronized方法B？

   答：不能。其它线程只能访问该对象的非同步方法，同步方法则不能进入。因为非静态方法上的synchronized修饰符要求执行方法时要获得对象的锁，如果已经进入A方法说明对象锁已经被取走，那么试图进入B方法的线程就只能在等锁池（注意不是等待池哦）中等待对象的锁。

4. 请说出与线程同步以及线程调度相关的方法。

   答：

   - wait()：使一个线程处于等待（阻塞）状态，并且释放所持有的对象的锁；
   - sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要处理InterruptedException异常；
   - notify()：唤醒一个处于等待状态的线程，当然在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且与优先级无关；
   - notityAll()：唤醒所有处于等待状态的线程，该方法并不是将对象的锁给所有线程，而是让它们竞争，只有获得锁的线程才能进入就绪状态；

   补充：Java 5通过Lock接口提供了显式的锁机制（explicit lock），增强了灵活性以及对线程的协调。Lock接口中定义了加锁（lock()）和解锁（unlock()）的方法，同时还提供了newCondition()方法来产生用于线程之间通信的Condition对象；此外，Java 5还提供了信号量机制（semaphore），信号量可以用来限制对某个共享资源进行访问的线程的数量。在对资源进行访问之前，线程必须得到信号量的许可（调用Semaphore对象的acquire()方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用Semaphore对象的release()方法）。


### 参考

1. [Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)
2. [Java线程详解](https://blog.csdn.net/kwame211/article/details/78963044)
3. [JAVA多线程实现的四种方式](https://blog.csdn.net/king_kgh/article/details/78213576)
