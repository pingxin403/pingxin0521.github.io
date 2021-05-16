---
title: java Thread （二）
date: 2019-04-06 19:38:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 线程组

线程组ThreadGroup表示一组线程的集合,一旦一个线程归属到一个线程组之中后，就不能再更换其所在的线程组。那么为什么要使用线程组呢？个人认为有以下的好处：方便统一管理，线程组可以进行复制，快速定位到一个线程，统一进行异常设置等。ThreadGroup它其实并不属于Java并发包中的内容，它是java.lang中的内容。但是掌握对其的于理解，在实际应用中有很大的帮助。
<!--more-->

在java中为了方便线程管理出现了线程组ThreadGroup的概念，每个ThreadGroup可以同时包含多个子线程和多个子线程组，在一个进程中线程组是以树形的方式存在，通常情况下根线程组是system。system线程组下是main线程组，默认情况下第一级应用自己的线程组是通过main线程组创建出来的。

```（java）
public class ThreadGroupTest {
    public static void main(String[] args) throws InterruptedException {
        //主线程对应的线程组
        printGroupInfo(Thread.currentThread());//线程组为main父线程组为system

        //新建线程，系统默认的线程组
        Thread appThread = new Thread(()->{},"appThread");
        printGroupInfo(appThread);//线程组为main父线程组为system

        //自定义线程组
        ThreadGroup factoryGroup=new ThreadGroup("factory");
        Thread workerThread=new Thread(factoryGroup,()->{},"worker");
        printGroupInfo(workerThread);//线程组为factory，父线程组为main

        //设置父线程组
        ThreadGroup deviceGroup=new ThreadGroup(factoryGroup,"device");
        Thread pcThread=new Thread(deviceGroup,()->{},"pc");
        printGroupInfo(pcThread);//线程组为device，父线程组为factory

    }

    static void printGroupInfo(Thread t) {
        ThreadGroup group = t.getThreadGroup();
        System.out.println("thread " + t.getName()
                + " group name is "+ group.getName()
                + " max priority is " + group.getMaxPriority()
                + " thread count is " + group.activeCount()
                + " parent group is "+ (group.getParent()==null?null:group.getParent().getName()));

        ThreadGroup parent=group;
        do {
            ThreadGroup current = parent;
            parent = parent.getParent();
            if (parent == null) {
                break;
            }
            System.out.println(current.getName() +" Group's  parent group name is "+parent.getName());

        } while (true);
        System.out.println("--------------------------");
    }
}
```

#### ThreadGroup线程组的操作

1. 线程组信息的获取
```
public int activeCount(); // 获得当前线程组中线程数目， 包括可运行和不可运行的
public int activeGroupCount()； //获得当前线程组中活动的子线程组的数目
public int enumerate（Thread list[]）; //列举当前线程组中的线程
public int enumerate（ThreadGroup list[]）； //列举当前线程组中的子线程组
public final int getMaxPriority（）; //获得当前线程组中最大优先级
public final String getName（）； //获得当前线程组的名字
public final ThreadGroup getParent（）; //获得当前线程组的父线程组
public boolean parentOf（ThreadGroup g）； //判断当前线程组是否为指定线程的父线程
public boolean isDaemon（）; //判断当前线程组中是否有监护线程
public void list（）; //列出当前线程组中所有线程和子线程名
```

2. 线程组的操作
```
public final void resume（）； //使被挂起的当前组内的线程恢复到可运行状态
public final void setDaemon (boolean daemon); //指定一个线程为当前线程组的监护线程
public final void setMaxPriority（int pri）； //设置当前线程组允许的最大优先级
public final void stop（）；//终止当前线程组中所有线程
public final void suspend（）; //挂起当前线程组中所有线程
public String toStrinng（）; //将当前线程组转换为String类的对象
```

示例：

```
import java.util.Date;
import java.util.Random;
import java.util.concurrent.TimeUnit;

public class ThreadGroupDemo {

    public static void main(String[] args) throws InterruptedException {
        // 创建5个线程，并入group里面进行管理
        ThreadGroup threadGroup = new ThreadGroup("threadGroupTest1");
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(threadGroup,()->{
                System.out.println("Thread Start " + Thread.currentThread().getName());
                try {
                    int value = (int)new Random((new Date()).getTime()).nextDouble()*100;
                    System.out.printf("Thread %s doTask: %d\n", Thread.currentThread().getName(),value);
                    TimeUnit.SECONDS.sleep(value);
                } catch (InterruptedException e) {
                    System.out.printf("Thread %s: Interrupted\n", Thread.currentThread().getName());
                    return;
                }
                System.out.println("Thread end " + Thread.currentThread().getName());
            });
            thread.start();
            TimeUnit.SECONDS.sleep(1);
        }
        //group信息
        System.out.printf("Number of Threads: %d\n", threadGroup.activeCount());
        System.out.printf("Information about the Thread Group\n");
        threadGroup.list();

        //复制group的thread信息
        Thread[] threads = new Thread[threadGroup.activeCount()];
        threadGroup.enumerate(threads);
        for (int i = 0; i < threadGroup.activeCount(); i++) {
            System.out.printf("Thread %s: %s\n", threads[i].getName(),threads[i].getState());
        }

        //等待结束
        while (threadGroup.activeCount() > 9) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //中断group中的线程
        threadGroup.interrupt();
    }
}
```

**线程组和线程池的区别**

线程组和线程池是两个不同的概念，他们的作用完全不同，前者是为了方便线程的管理，后者是为了管理线程的生命周期，复用线程，减少创建销毁线程的开销。

### ThreadLocal

ThreadLocal一般称为线程本地变量，它是一种特殊的线程绑定机制，将变量与线程绑定在一起，为每一个线程维护一个独立的变量副本。通过ThreadLocal可以将对象的可见范围限制在同一个线程内。

#### 跳出误区

需要重点强调的的是，不要拿ThreadLocal和synchronized做类比，因为这种比较压根就是无意义的！sysnchronized是一种互斥同步机制，是为了保证在多线程环境下对于共享资源的正确访问。而ThreadLocal从本质上讲，无非是提供了一个“线程级”的变量作用域，它是一种线程封闭（每个线程独享变量）技术，更直白点讲，ThreadLocal可以理解为将对象的作用范围限制在一个线程上下文中，使得变量的作用域为“线程级”。

没有ThreadLocal的时候，一个线程在其声明周期内，可能穿过多个层级，多个方法，如果有个对象需要在此线程周期内多次调用，且是跨层级的（线程内共享），通常的做法是通过参数进行传递；而ThreadLocal将变量绑定在线程上，在一个线程周期内，无论“你身处何地”，只需通过其提供的get方法就可轻松获取到对象。极大地提高了对于“线程级变量”的访问便利性。

假设我们要为每个线程关联一个唯一的序号，在每个线程周期内，我们需要多次访问这个序号，这时我们就可以使用ThreadLocal了.（当然下面这个例子没有完全体现出跨层级跨方法的调用，理解就可以了）

```java
public class ThreadMain {
    // 1.通过匿名内部类覆盖ThreadLocal的initialValue()方法，指定初始值
    private static ThreadLocal<Integer> seqNum = new ThreadLocal<Integer>() {
        public Integer initialValue() {
            return 0;
        }
    };
    // 2.获取下一个序列值
    public int getNextNum() {
        seqNum.set(seqNum.get() + 1);
        return seqNum.get();
    }
    public static void main(String[] args) {
        ThreadMain sn = new ThreadMain();
        // 3. 3个线程共享sn，各自产生序列号
        TestClient t1 = new TestClient(sn);
        TestClient t2 = new TestClient(sn);
        TestClient t3 = new TestClient(sn);
        t1.start();
        t2.start();
        t3.start();
    }
    private static class TestClient extends Thread {
        private ThreadMain sn;
        public TestClient(ThreadMain sn) {
            this.sn = sn;
        }
        public void run() {
            for (int i = 0; i < 3; i++) {
                // 4. 每个线程打出3个序列值
                System.out.println("thread[" + Thread.currentThread().getName() + "] --> sn["
                        + sn.getNextNum() + "]");
            }
        }
    }
}  
```

#### 看看源码

set操作，为线程绑定变量
```
public void set(T value) {
    Thread t = Thread.currentThread();//1.首先获取当前线程对象
        ThreadLocalMap map = getMap(t);//2.获取该线程对象的ThreadLocalMap
        if (map != null)
            map.set(this, value);//如果map不为空，执行set操作，以当前threadLocal对象为key，实际存储对象为value进行set操作
        else
            createMap(t, value);//如果map为空，则为该线程创建ThreadLocalMap
}
```
可以看到，ThreadLocal不过是个入口，真正的变量是绑定在线程上的。
```
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;//线程对象持有ThreadLocalMap的引用
}
```
下面给是Thread类中的定义，每个线程对象都拥有一个ThreadLocalMap对象
```
    ThreadLocal.ThreadLocalMap threadLocals = null;
```

现在，我们能看出ThreadLocal的设计思想了：

1. ThreadLocal仅仅是个变量访问的入口；

2. 每一个Thread对象都有一个ThreadLocalMap对象，这个ThreadLocalMap持有对象的引用；

3. ThreadLocalMap以当前的threadlocal对象为key，以真正的存储对象为value。get时通过threadlocal实例就可以找到绑定在当前线程上的对象。

乍看上去，这种设计确实有些绕。我们完全可以在设计成Map<Thread,T>这种形式，一个线程对应一个存储对象。ThreadLocal这样设计的目的主要有两个：

一是可以保证当前线程结束时相关对象能尽快被回收；二是ThreadLocalMap中的元素会大大减少，我们都知道map过大更容易造成哈希冲突而导致性能变差。

我们再来看看get方法

```
public T get() {
     Thread t = Thread.currentThread();//1.首先获取当前线程
         ThreadLocalMap map = getMap(t);//2.获取线程的map对象
         if (map != null) {//3.如果map不为空，以threadlocal实例为key获取到对应Entry，然后从Entry中取出对象即可。
             ThreadLocalMap.Entry e = map.getEntry(this);
             if (e != null)
                 return (T)e.value;
         }
         return setInitialValue();//如果map为空，也就是第一次没有调用set直接get（或者调用过set，又调用了remove）时，为其设定初始值
     }
```

setInitialValue

```
private T setInitialValue() {
        T value = initialValue();//获取初始值
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```
initialValue方法，默认是null，访问权限是protected，即允许重写。谈到这儿，我们应该已经对ThreadLocal的设计目的及设计思想有一定的了解了。

使用ThreadLocal一般都是声明在静态变量，如果ThreadLocal而且没有调用其remove方法，会导致内存泄露。

#### 线程独享变量?

还有一个会引起疑惑的问题，我们说ThreadLocal为每一个线程维护一个独立的变量副本，那么是不是说各个线程之间真正的做到对于对象的“完全自治”而不对其他线程的对象产生影响呢？其实这已经不属于对于ThreadLocal的讨论，而是你出于何种目的去使用ThreadLocal。如果我们为一个线程关联的对象是“完全独享”的，也就是每个线程拥有一整套的新的 栈中的对象引用+堆中的对象，那么这种情况下是真正的彻底的“线程独享变量”，相当于一种深度拷贝，每个线程自己玩自己的，对该对象做任何的操作也不会对别的线程有任何影响。

另一种更普遍的情况，所谓的独享变量副本，其实也就是每个线程都拥有一个独立的对象引用，而堆中的对象还是线程间共享的，这种情况下，自然还是会涉及到对共享资源的访问操作，依然会有线程不安全的风险。所以说，ThreadLocal无法解决线程安全问题。

所以，需不需要完全独享变量，进行完全隔离，就取决于你的应用场景了。可以想象，对象过大的时候，如果每个线程都有这么一份“深拷贝”，并发又比较大，对于服务器的压力自然是很大的。像web开发中的servlet，servlet是线程不安全的，一请求一线程，多个线程共享一个servlet对象；而早期的CGI设计中，N个请求就对应N个对象，并发量大了之后性能自然就很差。

![1.png](https://i.loli.net/2019/05/05/5ccea0fc640bb.png)


ThreadLocal在spring的事务管理，包括Hibernate的session管理等都有出现，在web开发中，有时会用来管理用户会话 HttpSession，web交互中这种典型的一请求一线程的场景似乎比较适合使用ThreadLocal，但是需要特别注意的是，由于此时session与线程关联，而tomcat这些web服务器多会采用线程池机制，也就是说线程是可复用的，所以在每一次进入的时候都需要重新进行set，或者在结束时及时remove。

### 异常的处理策略

由于java thread本身牵涉到并发、锁等相关的问题已经够复杂了。再加上异常处理这些东西，使得它更加特殊。 概括起来，不外乎是三个主要的问题。
1. 在java启动的线程里可以抛出异常吗？

2. 在启动的线程里可以捕捉异常吗？

3. 如果可以捕捉异常，对于checked exception和unchecked exception，他们分别有什么的处理方式呢？

#### 线程里抛出异常

我们可以尝试一下在线程里抛异常。按照我们的理解，假定我们要在某个方法里抛异常，需要在该定义的方法头也加上声明。那么一个最简单的方式可能如下：
```
public class Task implements Runnable {  

    @Override  
    public void run() throws Exception {  
        int number0 = Integer.parseInt("1");  
        throw new Exception("Just for test");  
    }  
}  
```
发现这种方式行不通。也就是说，在线程里直接抛异常是不行的。可是，这又会引出一个问题，如果我们在线程代码里头确实是产生了异常，那该怎么办呢？比如说，我们通过一个线程访问一些文件或者对网络进行IO操作，结果产生了异常。或者说访问某些资源的时候系统崩溃了。这样的场景是确实可能会发生的，我们就需要针对这些情况进行进一步的讨论。

#### 异常处理的几种方式

在前面提到的几种在线程访问资源产生了异常的情况。我们可以看，比如说我们访问文件系统的时候，会抛出IOException, FileNotFoundException等异常。我们在访问的代码里实际上是需要采用两种方式来处理的。一种是在使用改资源的方法头增加throws IOException, FileNotFoundException等异常的修饰。还有一种是直接在这部分的代码块增加try/catch部分。由前面我们的讨论已经发现，在方法声明加throws Exception的方式是行不通的。那么就只有使用try/catch这么一种方式了。

另外，我们也知道，在异常的处理上，一般异常可以分为checked exception和unchecked exception。作为unchecked exception，他们通常是指一些比较严重的系统错误或者系统设计错误，比如Error, OutOfMemoryError或者系统直接就崩溃了。对于这种异常发生的时候，我们一般是无能为力也没法恢复的。那么这种情况的发生，我们会怎么来处理呢？

**checked exception**

在线程里面处理checked exception，按照我们以前的理解，我们是可以直接捕捉它来处理的。在一些thread的示例里我们也见过。比如说下面的一部分代码：
```
import java.util.Date;  
import java.util.concurrent.TimeUnit;  

public class FileLock implements Runnable {  
  @Override  
  public void run() {  
      for(int i = 0; i < 10; i++) {  
          System.out.printf("%s\n", new Date());  
          try {  
              TimeUnit.SECONDS.sleep(1);  
          } catch(InterruptedException e) {  
              System.out.printf("The FileClock has been interrupted");  
          }  
      }  
  }  
}
```
我们定义了一个线程执行代码，并且在这里因为调用TimeUnit.SECONDS.sleep()方法而需要捕捉异常。因为这个方法本身就会抛出InterruptedException，我们必须要用try/catch块来处理。

```
import java.util.concurrent.TimeUnit;  

public class Main {  
  public static void main(String[] args) {  
      // Creates a FileClock runnable object and a Thread  
      // to run it  
      FileClock clock=new FileClock();  
      Thread thread=new Thread(clock);  

      // Starts the Thread  
      thread.start();  
      try {  
          // Waits five seconds  
          TimeUnit.SECONDS.sleep(5);  
      } catch (InterruptedException e) {  
          e.printStackTrace();  
      };  
      // Interrupts the Thread  
      thread.interrupt();  
  }  
}  
```
这部分的代码是启动FileLock线程并尝试去中断它。我们可以发现在运行的时候FileLock里面执行的代码能够正常的处理异常。

因此，在thread里面，如果要处理checked exception，简单的一个try/catch块就可以了。

**unchecked exception**

对于这种unchecked exception，相对来说就会不一样一点。实际上，在Thread的定义里有一个实例方法：setUncaughtExceptionHandler(UncaughtExceptionHandler). 这个方法可以用来处理一些unchecked exception。那么，这种情况的场景是如何的呢？

setUncaughtExceptionHandler()方法相当于一个事件注册的入口。在jdk里面，该方法的定义如下：

```
public void setUncaughtExceptionHandler(UncaughtExceptionHandler eh) {  
    checkAccess();  
    uncaughtExceptionHandler = eh;  
}
```
而UncaughtExceptionHandler则是一个接口，它的声明如下：
```
public interface UncaughtExceptionHandler {  
  void uncaughtException(Thread t, Throwable e);  
}  
```
在异常发生的时候，我们传入的UncaughtExceptionHandler参数的uncaughtException方法会被调用。综合前面的讨论，我们这边要实现handle unchecked exception的方法的具体步骤可以总结如下：

1. 定义一个类实现UncaughtExceptionHandler接口。在实现的方法里包含对异常处理的逻辑和步骤。

2. 定义线程执行结构和逻辑。这一步和普通线程定义一样。

3. 在创建和执行改子线程的方法里在thread.start()语句前增加一个thread.setUncaughtExceptionHandler语句来实现处理逻辑的注册。

下面，我们就按照这里定义的步骤来实现一个示例：
```
public class ThreadExce {
  class ExceptionHandler implements Thread.UncaughtExceptionHandler {
      public void uncaughtException(Thread t, Throwable e) {
          System.out.printf("An exception has been captured\n");
          System.out.printf("Thread: %s\n", t.getId());
          System.out.printf("Exception: %s: %s\n",
                  e.getClass().getName(), e.getMessage());
          System.out.printf("Stack Trace: \n");
          e.printStackTrace(System.out);
          System.out.printf("Thread status: %s\n", t.getState());
      }
  }

  class Task implements Runnable {

      @Override
      public void run() {
          int number0 = Integer.parseInt("TTT");
      }
  }

  public static void main(String[] args) {
      ThreadExce exce=new ThreadExce();
      Task task = exce.new Task();
      Thread thread = new Thread(task);
      thread.setUncaughtExceptionHandler(exce.new ExceptionHandler());
      thread.start();
  }
}
```
现在我们去执行整个程序，会发现有如下的结果：
```
An exception has been captured
Thread: 12
Exception: java.lang.NumberFormatException: For input string: "TTT"
Stack Trace:
java.lang.NumberFormatException: For input string: "TTT"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.base/java.lang.Integer.parseInt(Integer.java:652)
	at java.base/java.lang.Integer.parseInt(Integer.java:770)
	at com.hyp.learn.thread.ThreadExce$Task.run(ThreadExce.java:26)
	at java.base/java.lang.Thread.run(Thread.java:834)
Thread status: RUNNABLE
```
这部分的输出正好就是我们前面实现UncaughtExceptionHandler接口的定义。

因此，对于unchecked exception，我们也可以采用类似事件注册的机制做一定程度的处理。

Java thread里面关于异常的部分比较奇特。你不能直接在一个线程里去抛出异常。一般在线程里碰到checked exception，推荐的做法是采用try/catch块来处理。而对于unchecked exception，比较合理的方式是注册一个实现UncaughtExceptionHandler接口的对象实例来处理。这些细节的东西如果没有碰到过确实很难回答。

### 参考

1. [Java中的ThreadGroup线程组](https://my.oschina.net/cqqcqqok/blog/1941629)
2. [谈谈Java中的ThreadLocal](http://www.cnblogs.com/chengxiao/p/6152824.html)
3. [Java thread中对异常的处理策略](https://www.cnblogs.com/googlemeoften/p/5769216.html)
