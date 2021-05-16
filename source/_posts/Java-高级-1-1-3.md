---
title: Java Thread 安全--Synchroized和volatile（一）
date: 2019-05-05 13:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### 线程安全

当多个线程访问同一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替运行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获取正确的结果，那这个对象是线程安全的。

<!--more-->

关于定义的理解这是一个仁者见仁智者见智的事情。出现线程安全的问题一般是因为主内存和工作内存数据不一致性和重排序导致的，而解决线程安全的问题最重要的就是理解这两种问题是怎么来的，那么，理解它们的核心在于理解java内存模型（JMM）。

我们都知道java的内存模型中有主内存和线程的工作内存之分，主内存上存放的是线程共享的变量（实例字段，静态字段和构成数组的元素），线程的工作内存是线程私有的空间，存放的是线程私有的变量（方法参数与局部变量）。

![](https://i.loli.net/2019/05/05/5cceb6bb123fb.png)

线程在工作的时候如果要操作主内存上的共享变量，为了获得更好的执行性能并不是直接去修改主内存而是会在线程私有的工作内存中创建一份变量的拷贝（缓存），在工作内存上对变量的拷贝修改之后再把修改的值刷回到主内存的变量中去，JVM提供了8中原子操作来完成这一过程：lock, unlock, read, load, use, assign, store, write。

如果只有一个线程当然不会有什么问题，但是如果有多个线程同时在操作主内存中的变量，因为8种操作的非连续性和线程抢占cpu执行的机制就会带来冲突的问题，也就是多线程的安全问题。线程安全的定义就是：**如果线程执行过程中不会产生共享资源的冲突就是线程安全的。**



线程安全在三个方面体现:

1. 原子性：提供互斥访问，同一时刻只能有一个线程对数据进行操作，（atomic,synchronized）；

2. 可见性：一个线程对主内存的修改可以及时地被其他线程看到，（synchronized,volatile）；

3. 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序，该观察结果一般杂乱无序，（happens-before原则）。


**cpu术语的定义**

|    术语    |        英文单词        |                           术语描述                           |
| :--------: | :--------------------: | :----------------------------------------------------------: |
|  内存屏障  |    memory barriers     |         是一组处理器指令，用于实现内存操作的顺序限制         |
|   缓冲行   |       cache line       | 缓存中可以分配的最小存储单位。处理器填写缓存线时会加载整个缓存线，需要使用多个主内存读周期 |
|  原子操作  |   atomic operations    |                  不可中断的一个或一系列操作                  |
| 缓存行填充 |    cache line fill     | 当处理器识别到从内存中读取操作数是可缓存的，处理器读取整个缓存行到适当的缓存（L1,L2,L3或所有） |
|  缓存命中  |       cache hit        | 如果进行高速缓存行填充操作的内存位置仍然是下次处理器访问的地址时，处理器从缓存中读取操作数，而不是从内存读取 |
|   写命中   |       write hit        | 当处理器将操作数写回到一个内存缓存的区域时，它首先会检查这个缓存的内存地址是否在缓存行中，如果存在一个有效的缓存行，则处理器将这个操作数写回到缓存，而不是写回到内存，这个操作被称为写命中 |
|   写缺失   | write misses the cache |           一个有效的缓存行被写入到不存在的内存区域           |


### 互斥同步锁（悲观锁）

互斥同步锁也叫做阻塞同步锁，特征是会对没有获取锁的线程进行阻塞。

要理解互斥同步锁，首选要明白什么是互斥什么是同步。简单的说互斥就是非你即我，同步就是顺序访问。互斥同步锁就是以互斥的手段达到顺序访问的目的。操作系统提供了很多互斥机制比如信号量，互斥量，临界区资源等来控制在某一个时刻只能有一个或者一组线程访问同一个资源。

### Synchorized

synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：

![1.png](https://i.loli.net/2019/05/06/5ccfe6623580d.png)

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

如果锁的是类对象的话，尽管new多个实例对象，但他们仍然是属于同一个类依然会被锁住，即线程之间保证同步关系。

隐式规则：

![](https://i.loli.net/2019/05/05/5cced24690e99.png)

#### 修饰一个代码块

```java
/**
 * 同步线程
 */
class SyncThread implements Runnable {
   private static int count;

   public SyncThread() {
      count = 0;
   }

   public  void run() {
      synchronized(this) {
         for (int i = 0; i < 5; i++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }

   public int getCount() {
      return count;
   }
}
```
SyncThread的调用：
```
SyncThread syncThread = new SyncThread();
Thread thread1 = new Thread(syncThread, "SyncThread1");
Thread thread2 = new Thread(syncThread, "SyncThread2");
thread1.start();
thread2.start();
```
结果如下：
```
SyncThread1:0
SyncThread1:1
SyncThread1:2
SyncThread1:3
SyncThread1:4
SyncThread2:5
SyncThread2:6
SyncThread2:7
SyncThread2:8
SyncThread2:9
```

当两个并发线程(thread1和thread2)访问同一个对象(syncThread)中的synchronized代码块时，在同一时刻只能有一个线程得到执行，另一个线程受阻塞，必须等待当前线程执行完这个代码块以后才能执行该代码块。Thread1和thread2是互斥的，因为在执行synchronized代码块时会锁定当前的对象，只有执行完该代码块才能释放该对象锁，下一个线程才能执行并锁定该对象。

我们再把SyncThread的调用稍微改一下：
```
Thread thread1 = new Thread(new SyncThread(), "SyncThread1");
Thread thread2 = new Thread(new SyncThread(), "SyncThread2");
thread1.start();
thread2.start();
```
执行结果：
```
SyncThread2:0
SyncThread1:1
SyncThread2:2
SyncThread1:3
SyncThread2:4
SyncThread1:5
SyncThread2:6
SyncThread1:7
SyncThread2:8
SyncThread1:9
```
不是说一个线程执行synchronized代码块时其它的线程受阻塞吗？为什么上面的例子中thread1和thread2同时在执行。这是因为synchronized只锁定对象，每个对象只有一个锁（lock）与之相关联，而上面的代码等同于下面这段代码：
```
SyncThread syncThread1 = new SyncThread();
SyncThread syncThread2 = new SyncThread();
Thread thread1 = new Thread(syncThread1, "SyncThread1");
Thread thread2 = new Thread(syncThread2, "SyncThread2");
thread1.start();
thread2.start();
```
这时创建了两个SyncThread的对象syncThread1和syncThread2，线程thread1执行的是syncThread1对象中的synchronized代码(run)，而线程thread2执行的是syncThread2对象中的synchronized代码(run)；我们知道synchronized锁定的是对象，这时会有两把锁分别锁定syncThread1对象和syncThread2对象，而这两把锁是互不干扰的，不形成互斥，所以两个线程可以同时执行。

**当一个线程访问对象的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该对象中的非synchronized(this)同步代码块。**

```java
class Counter implements Runnable{
   private int count;

   public Counter() {
      count = 0;
   }

   public void countAdd() {
      synchronized(this) {
         for (int i = 0; i < 5; i ++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }

   //非synchronized代码块，未对count进行读写操作，所以可以不用synchronized
   public void printCount() {
      for (int i = 0; i < 5; i ++) {
         try {
            System.out.println(Thread.currentThread().getName() + " count:" + count);
            Thread.sleep(100);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
   }

   public void run() {
      String threadName = Thread.currentThread().getName();
      if (threadName.equals("A")) {
         countAdd();
      } else if (threadName.equals("B")) {
         printCount();
      }
   }
}
```
调用代码：
```
Counter counter = new Counter();
Thread thread1 = new Thread(counter, "A");
Thread thread2 = new Thread(counter, "B");
thread1.start();
thread2.start();
```
运行结果：
```
B count:0
A:0
B count:1
A:1
B count:2
A:2
B count:3
A:3
B count:4
A:4
```
上面代码中countAdd是一个synchronized的，printCount是非synchronized的。从上面的结果中可以看出一个线程访问一个对象的synchronized代码块时，别的线程可以访问该对象的非synchronized代码块而不受阻塞。

#### 指定要给某个对象加锁

```java
/**
 * 银行账户类
 */
class Account {
   String name;
   float amount;

   public Account(String name, float amount) {
      this.name = name;
      this.amount = amount;
   }
   //存钱
   public  void deposit(float amt) {
      amount += amt;
      try {
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }
   //取钱
   public  void withdraw(float amt) {
      amount -= amt;
      try {
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }

   public float getBalance() {
      return amount;
   }
}

/**
 * 账户操作类
 */
class AccountOperator implements Runnable{
   private Account account;
   public AccountOperator(Account account) {
      this.account = account;
   }

   public void run() {
      synchronized (account) {
         account.deposit(500);
         account.withdraw(500);
         System.out.println(Thread.currentThread().getName() + ":" + account.getBalance());
      }
   }
}
```

调用代码：
```
Account account = new Account("ping", 10000.0f);
AccountOperator accountOperator = new AccountOperator(account);

final int THREAD_NUM = 5;
Thread threads[] = new Thread[THREAD_NUM];
for (int i = 0; i < THREAD_NUM; i ++) {
    threads[i] = new Thread(accountOperator, "Thread" + i);
    threads[i].start();
}
```
结果：
```
Thread0:10000.0
Thread4:10000.0
Thread3:10000.0
Thread2:10000.0
Thread1:10000.0
```
在AccountOperator 类中的run方法里，我们用synchronized 给account对象加了锁。这时，当一个线程访问account对象时，其他试图访问account对象的线程将会阻塞，直到该线程访问account对象结束。也就是说谁拿到那个锁谁就可以运行它所控制的那段代码。

当有一个明确的对象作为锁时，就可以用类似下面这样的方式写程序。
```
public void method3(SomeObject obj)
{
   //obj 锁定的对象
   synchronized(obj)
   {
      // todo
   }
}
```
当没有明确的对象作为锁，只是想让一段代码同步时，可以创建一个特殊的对象来充当锁：

```
class Test implements Runnable
{
   private byte[] lock = new byte[0];  // 特殊的instance变量
   public void method()
   {
      synchronized(lock) {
         // todo 同步代码块
      }
   }

   public void run() {

   }
}
```
说明：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock = new Object()则需要7行操作码。

其效果和上面是一样的，synchronized作用于一个类T时，是给这个类T加锁，T的所有对象用的是同一把锁。

#### 修饰一个方法

Synchronized修饰一个方法很简单，就是在方法的前面加synchronized，`public synchronized void method(){//todo};` synchronized修饰方法和修饰一个代码块类似，只是作用范围不一样，修饰代码块是大括号括起来的范围，而修饰方法范围是整个函数。如将上例中的run方法改成如下的方式，实现的效果一样。

```
public synchronized void run() {
   for (int i = 0; i < 5; i ++) {
      try {
         System.out.println(Thread.currentThread().getName() + ":" + (count++));
         Thread.sleep(100);
      } catch (InterruptedException e) {
         e.printStackTrace();
      }
   }
}
```
Synchronized作用于整个方法的写法。
```
public synchronized void method()
{
   // todo
}
public void method()
{
   synchronized(this) {
      // todo
   }
}
```
写法一修饰的是一个方法，写法二修饰的是一个代码块，但写法一与写法二是等价的，都是锁定了整个方法时的内容。

在用synchronized修饰方法时要注意以下几点：

1. synchronized关键字不能继承。

   虽然可以使用synchronized来定义方法，但synchronized并不属于方法定义的一部分，因此，synchronized关键字不能被继承。

   如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized关键字才可以。当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。

  ```java
  // 在子类方法中加上synchronized关键字
  class Parent {
     public synchronized void method() { }
  }
  class Child extends Parent {
     public synchronized void method() { }
  }

  //在子类方法中调用父类的同步方法
  class Parent {
     public synchronized void method() {   }
  }
  class Child extends Parent {
     public void method() { super.method();   }
  }
  ```

2. 在定义接口方法时不能使用synchronized关键字。

3. 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。

#### 修饰一个静态的方法

Synchronized也可修饰一个静态方法，用法如下：
```java
public synchronized static void method() {
   // todo
}
```
我们知道静态方法是属于类的而不属于对象的。同样的，synchronized修饰的静态方法锁定的是这个类的所有对象。
```java
/**
 * 同步线程
 */
class SyncThread implements Runnable {
   private static int count;

   public SyncThread() {
      count = 0;
   }

   public synchronized static void method() {
      for (int i = 0; i < 5; i ++) {
         try {
            System.out.println(Thread.currentThread().getName() + ":" + (count++));
            Thread.sleep(100);
         } catch (InterruptedException e) {
            e.printStackTrace();
         }
      }
   }

   public synchronized void run() {
      method();
   }
}
```
调用代码
```
SyncThread syncThread1 = new SyncThread();
SyncThread syncThread2 = new SyncThread();
Thread thread1 = new Thread(syncThread1, "SyncThread1");
Thread thread2 = new Thread(syncThread2, "SyncThread2");
thread1.start();
thread2.start();
```
结果：
```
SyncThread1:0
SyncThread1:1
SyncThread1:2
SyncThread1:3
SyncThread1:4
SyncThread2:5
SyncThread2:6
SyncThread2:7
SyncThread2:8
SyncThread2:9
```
syncThread1和syncThread2是SyncThread的两个对象，但在thread1和thread2并发执行时却保持了线程同步。这是因为run中调用了静态方法method，而静态方法是属于类的，所以syncThread1和syncThread2相当于用了同一把锁。

#### 修饰一个类

Synchronized还可作用于一个类，用法如下：
```java
class ClassName {
   public void method() {
      synchronized(ClassName.class) {
         // todo
      }
   }
}
```
示例：
```java
/**
 * 同步线程
 */
class SyncThread implements Runnable {
   private static int count;

   public SyncThread() {
      count = 0;
   }

   public static void method() {
      synchronized(SyncThread.class) {
         for (int i = 0; i < 5; i ++) {
            try {
               System.out.println(Thread.currentThread().getName() + ":" + (count++));
               Thread.sleep(100);
            } catch (InterruptedException e) {
               e.printStackTrace();
            }
         }
      }
   }

   public synchronized void run() {
      method();
   }
}
```
#### 对象锁（monitor）机制

先写一个简单的demo:

```java
public class SynchronizedDemo {
    public static void main(String[] args) {
        synchronized (SynchronizedDemo.class) {
        }
        method();
    }

    private static void method() {
    }
}
```

编译之后，切换到SynchronizedDemo.class的同级目录之后，然后用**javap -v SynchronizedDemo.class查看字节码文件**

![lJrnzT.png](https://s2.ax1x.com/2020/01/01/lJrnzT.png)

如图，上面用黄色高亮的部分就是需要注意的部分了，这也是Synchronized关键字之后独有的。

执行同步代码块后首先要**先执行monitorenter指令，退出的时候monitorexit指令**。通过分析之后可以看出，使用Synchronized进行同步，其关键就是**必须要对对象的监视器monitor进行获取，当线程获取monitor后才能继续往下执行，否则就只能等待。**

而这个获取的过程是**互斥**的，即**同一时刻只有一个线程能够获取到monitor**。

 上面的demo中在执行完同步代码块之后紧接着再会去执行一个静态同步方法，而这个方法锁的对象依然就这个类对象，那么这个正在执行的线程还需要获取该锁吗？答案是**不必的**。

 从上图中就可以看出来，执行静态同步方法的时候就只有一条monitorexit指令，并没有monitorenter获取锁的指令。这就是**锁的重入性，即在同一锁程中，线程不需要再次获取同一把锁。**Synchronized先天具有**重入性**。

**每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一。**

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到BLOCKED状态

下图表现了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![1.png](https://i.loli.net/2019/05/06/5ccff634d78f7.png)



#### synchronized的happens-before关系

监视器锁规则：对同一个监视器的解锁，happens-before于对该监视器的加锁。继续来看代码：
```
public class MonitorDemo {
    private int a = 0;

    public synchronized void writer() {     // 1
        a++;                                // 2
    }                                       // 3

    public synchronized void reader() {    // 4
        int i = a;                         // 5
    }                                      // 6
}
```
该代码的happens-before关系如图所示：

![1.png](https://i.loli.net/2019/05/06/5ccff9f1cf42e.png)

在图中每一个箭头连接的两个节点就代表之间的happens-before关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程A释放锁happens-before线程B加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来happens-befor关系，通过传递性规则进一步推导的happens-before关系。现在我们来重点关注2 happens-before 5，通过这个关系我们可以得出什么？

根据happens-before的定义中的一条:如果A happens-before B，则A的执行结果对B可见，并且A的执行顺序先于B。线程A先对共享变量A进行加一，由2 happens-before 5关系可知线程A的执行结果对线程B可见即线程B所读取到的a的值为1。


#### 锁获取和锁释放的内存语义

我们接下来看看基于java内存抽象模型的Synchronized的内存语义。

![1.png](https://i.loli.net/2019/05/06/5ccffbbb7c284.png)

从上图可以看出，线程A会首先先从主内存中读取共享变量a=0的值然后将该变量拷贝到自己的本地内存，进行加一操作后，再将该值刷新到主内存，整个过程即为线程A 加锁-->执行临界区代码-->释放锁相对应的内存语义。

线程B获取锁的时候同样会从主内存中共享变量a的值，这个时候就是最新的值1,然后将该值拷贝到线程B的工作内存中去，释放锁的时候同样会重写到主内存中。

从整体上来看，线程A的执行结果（a=1）对线程B是可见的，实现原理为：释放锁的时候会将值刷新到主内存中，其他线程获取锁时会强制从主内存中获取最新的值。另外也验证了2 happens-before 5，2的执行结果对5是可见的。

从横向来看，这就像线程A通过主内存中的共享变量和线程B进行通信，A 告诉 B 我们俩的共享数据现在为1啦，这种线程间的通信机制正好吻合java的内存模型正好是共享内存的并发模型结构。

#### 总结

A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

**synchronized底层实现请参看jvm的java类结构**

[synchronized实现原理，以及JVM对锁性能的优化](https://www.jianshu.com/p/73b9a8466b9c)

### volatile关键字

volatile是java提供的一种同步手段，只不过它是轻量级的同步，为什么这么说，因为volatile只能保证多线程的内存可见性，不能保证多线程的执行有序性。而最彻底的同步要保证有序性和可见性，例如synchronized。任何被volatile修饰的变量，都不拷贝副本l到工作内存，任何修改都及时写在主存。因此对于Valatile修饰的变量的修改，所有线程马上就能看到，但是volatile不能保证对变量的修改是有序的。

volatile存在的意义是，任何线程对a的修改，都会马上被其他线程读取到，因为直接操作主存，没有线程对工作内存和主存的同步。所以，volatile的使用场景是有限的，在有限的一些情形下可以使用 volatile 变量替代锁。

```
public volatile boolean exit = false;
```
在定义exit时，使用了一个Java关键字volatile，这个关键字的目的是使exit同步，也就是说在同一时刻只能由一个线程来修改exit的值.

现在我们有了一个大概的印象就是：**被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。**


volatile关键字用于声明简单类型变量，如int、float、boolean等数据类型。如果这些简单数据类型声明为volatile，对它们的操作就会变成原子级别的。但这有一定的限制。例如，下面的例子中的n就不是原子级别的：
```java
public class ThreadVolatile extends Thread {
    public static volatile int n = 0;

    public void run() {
        for (int i = 0; i < 10; i++)
            try {
                n = n + 1;
                sleep(3); // 为了使运行结果更随机，延迟3毫秒

            } catch (Exception e) {
            }
    }

    public static void main(String[] args) throws Exception {

        Thread threads[] = new Thread[100];
        for (int i = 0; i < threads.length; i++)
            // 建立100个线程
            threads[i] = new ThreadVolatile();
        for (int i = 0; i < threads.length; i++)
            // 运行刚才建立的100个线程
            threads[i].start();
        for (int i = 0; i < threads.length; i++)
            // 100个线程都执行完后继续
            threads[i].join();
        System.out.println("n=" + ThreadVolatile.n);
    }
}
```
如果对n的操作是原子级别的，最后输出的结果应该为n=1000，而在执行上面代码时，很多时侯输出的n都小于1000，这说明n=n+1不是原子级别的操作。原因是声明为volatile的简单变量如果当前值由该变量以前的值相关，那么volatile关键字不起作用，也就是说如下的表达式都不是原子操作:
```
n = n + 1;
n++;
```
如果要想使这种情况变成原子操作，需要使用synchronized关键字，如上的代码可以改成如下的形式：
```java
public class ThreadVolatile2 extends Thread {
    public static volatile int n = 0;

    public static synchronized void inc() {
        n++;
    }

    public void run() {
        for (int i = 0; i < 10; i++)
            try {
                inc(); // n = n + 1 改成了 inc();
                sleep(3); // 为了使运行结果更随机，延迟3毫秒

            } catch (Exception e) {
            }
    }

    public static void main(String[] args) throws Exception {

        Thread threads[] = new Thread[100];
        for (int i = 0; i < threads.length; i++)
            // 建立100个线程
            threads[i] = new ThreadVolatile2();
        for (int i = 0; i < threads.length; i++)
            // 运行刚才建立的100个线程
            threads[i].start();
        for (int i = 0; i < threads.length; i++)
            // 100个线程都执行完后继续
            threads[i].join();
        System.out.println("n=" + ThreadVolatile2.n);
    }
}
```
上面的代码将n=n+1改成了inc()，其中inc方法使用了synchronized关键字进行方法同步。因此，在使用volatile关键字时要慎重，并不是只要简单类型变量使用volatile修饰，对这个变量的所有操作都是原来操作，当变量的值由自身的上一个决定时，如n=n+1、n++等，volatile关键字将失效，只有当变量的值和自身上一个值无关时对该变量的操作才是原子级别的，如n = m + 1，这个就是原级别的。所以在使用volatile关键时一定要谨慎，如果自己没有把握，可以使用synchronized来代替volatile。

####  volatile实现原理

volatile是怎样实现了？比如一个很简单的Java代码：

```
instance = new Instancce()  //instance是volatile变量
```

在生成汇编代码时会在volatile修饰的共享变量进行写操作的时候会多出Lock前缀的指令（具体的大家可以使用一些工具去看一下，这里我就只把结果说出来）。我们想这个Lock指令肯定有神奇的地方，那么Lock前缀的指令在多核处理器下会发现什么事情了？主要有这

两个方面的影响：

- 将当前处理器缓存行的数据写回系统内存；
- 这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现 **缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了**，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。因此，经过分析我们可以得出如下结论：

- Lock前缀的指令会引起处理器缓存写回内存；
- 一个处理器的缓存回写到内存会导致其他处理器的缓存失效；
- 当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

这样针对volatile变量通过这样的机制就使得每个线程都能获得该变量的最新值。

#### volatile的happens-before关系

经过上面的分析，我们已经知道了volatile变量可以通过缓存一致性协议保证每个线程都能获得最新值，即满足数据的“可见性”。将并发分析的切入点分为两个核心，三大性质。

两大核心：JMM内存模型（主内存和工作内存）以及happens-before；
三条性质：原子性，可见性，有序性


在六条happens-before规则中有一条是：
**volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。**

下面我们结合具体的代码，我们利用这条规则推导下：
```
public class VolatileExample {
    private int a = 0;
    private volatile boolean flag = false;
    public void writer(){
        a = 1;          //1
        flag = true;   //2
    }
    public void reader(){
        if(flag){      //3
            int i = a; //4
        }
    }
}
```
上面的实例代码对应的happens-before关系如下图所示：

![1.png](https://i.loli.net/2019/05/06/5cd00542843f2.png)

加锁线程A先执行writer方法，然后线程B执行reader方法图中每一个箭头两个节点就代码一个happens-before关系，黑色的代表根据程序顺序规则推导出来，红色的是根据volatile变量的写happens-before 于任意后续对volatile变量的读，而蓝色的就是根据传递性规则推导出来的。这里的2 happen-before 3，同样根据happens-before规则定义：如果A happens-before B,则A的执行结果对B可见，并且A的执行顺序先于B的执行顺序，我们可以知道操作2执行结果对操作3来说是可见的，也就是说当线程A将volatile变量 flag更改为true后线程B就能够迅速感知。

#### volatile的内存语义实现

我们都知道，为了性能优化，JMM在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序，那如果想阻止重排序要怎么办了？答案是可以添加内存屏障。

JMM内存屏障分为四类见下图

![1.png](https://i.loli.net/2019/05/06/5cd00b5507a28.png)

java编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。为了实现volatile的内存语义，JMM会限制特定类型的编译器和处理器重排序，JMM会针对编译器制定volatile重排序规则表：

![2.png](https://i.loli.net/2019/05/06/5cd00b5466db6.png)

"NO"表示禁止重排序。为了实现volatile内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM采取了保守策略：

- 在每个volatile写操作的前面插入一个StoreStore屏障；
- 在每个volatile写操作的后面插入一个StoreLoad屏障；
- 在每个volatile读操作的后面插入一个LoadLoad屏障；
- 在每个volatile读操作的后面插入一个LoadStore屏障。

需要注意的是：volatile写是在前面和后面分别插入内存屏障，而volatile读操作是在后面插入两个内存屏障

- StoreStore屏障：禁止上面的普通写和下面的volatile写重排序；
- StoreLoad屏障：防止上面的volatile写与下面可能有的volatile读/写重排序
- LoadLoad屏障：禁止下面所有的普通读操作和上面的volatile读重排序
- LoadStore屏障：禁止下面所有的普通写操作和上面的volatile读重排序

下面以两个示意图进行理解，图片摘自相当好的一本书《java并发编程的艺术》。

![3.png](https://i.loli.net/2019/05/06/5cd00b53e7bce.png)

![4.png](https://i.loli.net/2019/05/06/5cd00b538f156.png)


#### 示例
单例模式（重排序）
```
public class Singleton {
    public static volatile Singleton singleton;

    /**
     * 构造函数私有
     */
    private Singleton(){}

    /**
     * 单例实现
     * @author fuyuwei
     * 2017年5月14日 上午10:07:07
     * @return
     */
    public static Singleton getInstance(){
        if(singleton == null){
            synchronized (singleton) {
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
我们知道实例化一个对象经过分配内存空间、初始化对象、将内存空间的地址赋值给对应的引用，上面的单例模式可以解释为分配内存空间、将内存地址赋值给对应的应用、初始化对象。上面的代码如果我们不加volatile在并发环境下可能会出现Singleton的多次实例化，假如线程A进入getInstance方法，发现singleton==null，然后加锁通过new Singleton进行实例化，然后释放锁，我们知道new Singleton在JVM中其实是分为3步，假如线程啊在释放锁之后还没来得及通知其他线程，这时候线程B进入getInstance的时候发现singleton==null便会再次实例化。

对volatile变量的写操作与普通变量的主要区别有两点：
（1）修改volatile变量时会强制将修改后的值刷新的主内存中。
（2）修改volatile变量后会导致其他线程工作内存中对应的变量值失效。因此，再读取该变量值的时候就需要重新从读取主内存中的值。

### 参考
1. [Java中synchronized的用法](http://www.importnew.com/21866.html)
2. [java多线程-慎重使用volatile关键字](http://www.cnblogs.com/linjiqin/p/3212737.html)
3. [让你彻底理解volatile](https://www.jianshu.com/p/157279e6efdb)
