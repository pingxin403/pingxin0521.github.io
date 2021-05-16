---
title: Java  Thread 安全--锁使用（三）
date: 2019-05-06 08:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### ReentrantLock

Java里面的互斥同步锁就是Synchorized和ReentrantLock，前者是由语言级别实现的互斥同步锁，理解和写法简单但是机制笨拙，在JDK6之后性能优化大幅提升，即使在竞争激烈的情况下也能保持一个和ReentrantLock相差不多的性能，所以JDK6之后的程序选择不应该再因为性能问题而放弃synchorized。

ReentrantLock是API层面的互斥同步锁，需要程序自己打开并在finally中关闭锁，和synchorized相比更加的灵活，体现在三个方面：等待可中断，公平锁以及绑定多个条件。但是如果程序猿对ReentrantLock理解不够深刻，或者忘记释放lock，那么不仅不会提升性能反而会带来额外的问题。

<!--more-->

另外synchorized是JVM实现的，可以通过监控工具来监控锁的状态，遇到异常JVM会自动释放掉锁。而ReentrantLock必须由程序主动的释放锁。

互斥同步锁都是可重入锁，好处是可以保证不会死锁。但是因为涉及到核心态和用户态的切换，因此比较消耗性能。JVM开发团队在JDK5-JDK6升级过程中采用了很多锁优化机制来优化同步无竞争情况下锁的性能。比如：自旋锁和适应性自旋锁，轻量级锁，偏向锁，锁粗化和锁消除。

顾名思义，ReentrantLock锁在同一个时间点只能被一个线程锁持有；而可重入的意思是，ReentrantLock锁，可以被单个线程多次获取。ReentrantLock分为“公平锁”和“非公平锁”。它们的区别体现在获取锁的机制上是否公平。“锁”是为了保护竞争资源，防止多个线程同时操作线程而出错，ReentrantLock在同一个时间点只能被一个线程获取(当某线程获取到“锁”时，其它线程就必须等待)；ReentraantLock是通过一个FIFO的等待队列来管理获取该锁所有线程的。在“公平锁”的机制下，线程依次排队获取锁；而“非公平锁”在锁是可获取状态时，不管自己是不是在队列的开头都会获取锁。

#### ReentrantLock函数列表

```java
// 创建一个 ReentrantLock ，默认是“非公平锁”。
ReentrantLock()
// 创建策略是fair的 ReentrantLock。fair为true表示是公平锁，fair为false表示是非公平锁。
ReentrantLock(boolean fair)

// 查询当前线程保持此锁的次数。
int getHoldCount()
// 返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正等待获取此锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正等待获取此锁的线程估计数。
int getQueueLength()
// 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回等待与此锁相关的给定条件的线程估计数。
int getWaitQueueLength(Condition condition)
// 查询给定线程是否正在等待获取此锁。
boolean hasQueuedThread(Thread thread)
// 查询是否有些线程正在等待获取此锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与此锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果是“公平锁”返回true，否则返回false。
boolean isFair()
// 查询当前线程是否保持此锁。
boolean isHeldByCurrentThread()
// 查询此锁是否由任意线程保持。
boolean isLocked()
// 获取锁。
void lock()
// 如果当前线程未被中断，则获取锁。
void lockInterruptibly()
// 返回用来与此 Lock 实例一起使用的 Condition 实例。
Condition newCondition()
// 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
boolean tryLock()
// 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。
boolean tryLock(long timeout, TimeUnit unit)
// 试图释放此锁。
void unlock()
```
可重入锁指在同一个线程中，可以重入的锁。当然，当这个线程获得锁后，其他线程将等待这个锁被释放后，才可以获得这个锁。

通常的使用方法：
```java
ReentrantLock lock = new ReentrantLock(); // not a fair lock
lock.lock();

try {

    // synchronized do something

} finally {
    lock.unlock();
}
```

#### 重入的实现

reentrant 锁意味着什么呢？简单来说，它有一个与锁相关的获取计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，然后锁需要被释放两次才能获得真正释放。这模仿了 synchronized 的语义；如果线程进入由线程已经拥有的监控器保护的 synchronized 块，就允许线程继续进行，当线程退出第二个（或者后续） synchronized 块的时候，不释放锁，只有线程退出它进入的监控器保护的第一个 synchronized 块时，才释放锁。

对于锁的重入，我们来想这样一个场景。当一个递归方法被sychronized关键字修饰时，在调用方法时显然没有发生问题，执行线程获取了锁之后仍能连续多次地获得该锁，也就是说sychronized关键字支持锁的重入。对于ReentrantLock，虽然没有像sychronized那样隐式地支持重入，但在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。

如果想要实现锁的重入，至少要解决一下两个问题：

- 线程再次获取锁：锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是，则再次成功获取。
- 锁的最终释放：线程重复n次获取了锁，随后在n次释放该锁后，其他线程能够获取该锁。锁的最终释放要求锁对于获取进行计数自增，计数表示当前锁被重复获取的次数，而锁被释放时，计数自减，当计数等于0时表示锁已经释放。

#### 公平锁与非公平锁

在Java的ReentrantLock构造函数中提供了两种锁：创建公平锁和非公平锁（默认）。代码如下：
```java
/**
 * 默认构造方法，非公平锁
 */
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * true公平锁，false非公平锁
 * @param fair
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```
**如果获取一个锁是按照请求的顺序得到的，那么就是公平锁，否则就是非公平锁。**

在没有深入了解内部机制及实现之前，先了解下为什么会存在公平锁和非公平锁。公平锁保证一个阻塞的线程最终能够获得锁，因为是有序的，所以总是可以按照请求的顺序获得锁。非公平锁意味着后请求锁的线程可能在其前面排列的休眠线程恢复前拿到锁，这样就有可能提高并发的性能。这是因为通常情况下挂起的线程重新开始与它真正开始运行，二者之间会产生严重的延时。因此非公平锁就可以利用这段时间完成操作。这是非公平锁在某些时候比公平锁性能要好的原因之一。

锁Lock分为“公平锁”和“非公平锁”，**公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。而非公平锁就是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样的就是先来的不一定先得到锁，这个方式可能造成某些线程一直拿不到锁，结果也就是不公平的了。**

#### ReentrantLock 扩展的功能

1. 实现可轮询的锁请求

   在内部锁中，死锁是致命的——唯一的恢复方法是重新启动程序，唯一的预防方法是在构建程序时不要出错。而可轮询的锁获取模式具有更完善的错误恢复机制，可以规避死锁的发生。

   如果你不能获得所有需要的锁，那么使用可轮询的获取方式使你能够重新拿到控制权，它会释放你已经获得的这些锁，然后再重新尝试。可轮询的锁获取模式，由tryLock()方法实现。此方法仅在调用时锁为空闲状态才获取该锁。如果锁可用，则获取锁，并立即返回值true。如果锁不可用，则此方法将立即返回值false。

2. 实现可定时的锁请求

   当使用内部锁时，一旦开始请求，锁就不能停止了，所以内部锁给实现具有时限的活动带来了风险。为了解决这一问题，可以使用定时锁。当具有时限的活
   动调用了阻塞方法，定时锁能够在时间预算内设定相应的超时。如果活动在期待的时间内没能获得结果，定时锁能使程序提前返回。可定时的锁获取模式，由tryLock(long, TimeUnit)方法实现。

3. 实现可中断的锁获取请求

   可中断的锁获取操作允许在可取消的活动中使用。lockInterruptibly()方法能够使你获得锁的时候响应中断。


**ReentrantLock 与 synchronized 的比较**

相同：ReentrantLock提供了synchronized类似的功能和内存语义。

不同：

1. 与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。
2. ReentrantLock还提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更加适合（下面会阐述Condition）。
3. ReentrantLock提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized则一旦进入锁请求要么成功，要么一直阻塞，所以相比synchronized而言，ReentrantLock会不容易产生死锁些。
4. ReentrantLock支持更加灵活的同步代码块，但是使用synchronized时，只能在同一个synchronized块结构中获取和释放。注：ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。
5. ReentrantLock支持中断处理，且性能较synchronized会好些。

**ReentrantLock 不好与需要注意的地方**

1. lock 必须在 finally 块中释放。否则，如果受保护的代码将抛出异常，锁就有可能永远得不到释放！这一点区别看起来可能没什么，但是实际上，它极为重要。忘记在 finally 块中释放锁，可能会在程序中留下一个定时炸弹，当有一天炸弹爆炸时，您要花费很大力气才有找到源头在哪。而使用同步，JVM 将确保锁会获得自动释放。
2. 当 JVM 用 synchronized 管理锁定请求和释放时，JVM 在生成线程转储时能够包括锁定信息。这些对调试非常有价值，因为它们能标识死锁或者其他异常行为的来源。 Lock 类只是普通的类，JVM 不知道具体哪个线程拥有 Lock 对象。

#### 示例

1. 用两个线程在控制台有序打出数字

   ```java
public class FirstReentrantLock {
       public static void main(String[] args) {
           Runnable runnable = new ReentrantLockThread(10);
           new Thread(runnable, "a").start();
           new Thread(runnable, "b").start();
       }    
   }
   class ReentrantLockThread implements Runnable{
       private int count;
       ReentrantLockThread(int count) {
           this.count = count;
       }
       @Override
       public void run() {
           for (int i = 0; i < count; i++) {
               System.out.println(Thread.currentThread().getName() + "输出了：  " + i);
           }
       }
   }
   ```
   
   可以看到，并没有顺序，杂乱无章。

   那使用ReentrantLock加入锁，代码如下：

   ```java
class ReentrantLockThread implements Runnable {
   
       ReentrantLock lock=new ReentrantLock();
       private int count;
   
       ReentrantLockThread(int count) {
           this.count = count;
       }
   
       @Override
       public void run() {
           try {
               lock.lock();
               for (int i = 0; i < count; i++) {
   
                   System.out.println(Thread.currentThread().getName() + "输出了：  " + i);
               }
           }finally {
               lock.unlock();
           }
   
       }
   }
   ```
   
   这就是锁的作用，它是互斥的，当一个线程持有锁的时候，其他线程只能等待，待该线程执行结束，再通过竞争得到锁。
2. 测试可重入锁的重入特性

   ```java
   import java.util.Calendar;
   import java.util.concurrent.locks.ReentrantLock;
   
   public class TestLock {
   
       private ReentrantLock lock = null;
   
       public TestLock() {
           // 创建一个自由竞争的可重入锁
           lock = new ReentrantLock();
       }
   
       public static void main(String[] args) {
   
           TestLock tester = new TestLock();
   
           try{
               // 测试可重入，方法testReentry() 在同一线程中,可重复获取锁,执行获取锁后，显示信息的功能
               tester.testReentry();
               // 能执行到这里而不阻塞，表示锁可重入
               tester.testReentry();
               // 再次重入
               tester.testReentry();
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               // 释放重入测试的锁，要按重入的数量解锁，否则其他线程无法获取该锁。
               tester.getLock().unlock();
               tester.getLock().unlock();
               tester.getLock().unlock();
   
           }
       }
   
       public ReentrantLock getLock() {
           return lock;
       }
   
       public void testReentry() {
           lock.lock();
   
           Calendar now = Calendar.getInstance();
   
           System.out.println(now.getTime() + " " + Thread.currentThread().getName()
                   + " get lock.");
       }
   
   }
   ```

3. 公平锁和非公平锁的差异

   公平锁

   ```java
   import java.util.concurrent.locks.ReentrantLock;
   
   public class LockDemo2 {
       public static void main(String[] args) throws InterruptedException {
           final Service service = new Service(true);  //改为false就为非公平锁了  
           Runnable runnable = new Runnable() {
               public void run() {
                   System.out.println("**线程： " + Thread.currentThread().getName()
                           +  " 运行了 " );
                   service.serviceMethod();
               }
           };
   
           Thread[] threadArray = new Thread[10];
   
           for (int i=0; i<10; i++) {
               threadArray[i] = new Thread(runnable);
           }
           for (int i=0; i<10; i++) {
               threadArray[i].start();
           }
       }
   }
   
   class Service {
   
       private ReentrantLock lock ;
   
       public Service(boolean isFair) {
           lock = new ReentrantLock(isFair);
       }
   
       public void serviceMethod() {
           try {
               lock.lock();
               System.out.println("ThreadName=" + Thread.currentThread().getName()
                       + " 获得锁定");
           } finally {
               lock.unlock();
           }
       }
   }
   ```

   打印的结果是按照线程加锁的顺序输出的，即线程运行了，则会先获得锁。

   非公平锁:将下面语句中的参数true改为false就为非公平锁了。

   ```
   final Service service = new Service(true);  //改为false就为非公平锁了  
   ```

   是乱序的，说明先start()启动的线程不代表先获得锁。

运行结果反映：

在公平的锁上，线程按照他们发出请求的顺序获取锁，但在非公平锁上，则允许“插队”：当一个线程请求非公平锁时，如果在发出请求的同时该锁变成可用状态，那么这个线程会跳过队列中所有的等待线程而获得锁。非公平的ReentrantLock 并不提倡 插队行为，但是无法防止某个线程在合适的时候进行插队。

在公平的锁中，如果有另一个线程持有锁或者有其他线程在等待队列中等待这个锁，那么新发出的请求的线程将被放入到队列中。而非公平锁上，只有当锁被某个线程持有时，新发出请求的线程才会被放入队列中。

非公平锁性能高于公平锁性能的原因：在恢复一个被挂起的线程与该线程真正运行之间存在着严重的延迟。假设线程A持有一个锁，并且线程B请求这个锁。由于锁被A持有，因此B将被挂起。当A释放锁时，B将被唤醒，因此B会再次尝试获取这个锁。与此同时，如果线程C也请求这个锁，那么C很可能会在B被完全唤醒之前获得、使用以及释放这个锁。这样就是一种双赢的局面：B获得锁的时刻并没有推迟，C更早的获得了锁，并且吞吐量也提高了。

当持有锁的时间相对较长或者请求锁的平均时间间隔较长，应该使用公平锁。在这些情况下，插队带来的吞吐量提升（当锁处于可用状态时，线程却还处于被唤醒的过程中）可能不会出现。

### 读写锁

ReentrantLock属于排他锁，这些锁在同一时刻只允许一个线程进行访问，而读写锁在同一时刻可以允许多个线程访问，但是在写线程访问时，所有的读和其他写线程都被阻塞。读写锁维护了一对锁，一个读锁和一个写锁，通过分离读锁和写锁，使得并发性相比一般的排他锁有了很大提升。

Java并发包提供读写锁的实现是ReentrantReadWriteLock

下面我们来看看读写锁ReentrantReadWriter特性

- 公平性选择：支持非公平（默认）和公平的锁获取模式，吞吐量还是非公平优于公平
- 重入性：该锁支持重入锁，以读写线程为例：读线程在获取读锁之后，能够再次读取读锁，而写线程在获取写锁之后可以同时再次获取读锁和写锁
- 锁降级：遵循获取写锁，获取读锁再释放写锁的次序，写锁能够降级为读锁
- 锁升级：读取锁不能直接升级为写入锁因为获取一个写入锁需要释放所以读取锁
- 锁捕获中断：读取锁和写入锁都支持在获取锁期间被中断
- 添加变量：写入锁提供Condition的支持，读取锁缺不允许，否则会抛出UnsupportedOperationExcetion异常
- 重入数：读取锁和写入锁的数量最大分别是65535

读写锁机制：

- 读-读不互斥
- 读-写互斥
- 写-写互斥

#### 读写锁接口详解

ReentrantReadWriterLock是ReadWriterLock的接口实现类，但是ReadWriterLock接口仅有读锁、写锁两个方法
```java
public interface ReadWriteLock {
   Lock readLock();
    Lock writeLock();
}
```
ReentrantReadWriterLock自己提供了一些内部工作状态方法，例如
```java
/**
* 返回当前读锁被获取的次数，该次数不等于获取锁的线程数，因为同一个线程可以多次获取支持重入锁
*/
public int getReadLockCount() {
   return sync.getReadLockCount();
}
/**
* 返回当前线程获取读锁的次数，Java6之后使用ThreadLocal保存当前线程获取的次数
*/
final int getReadHoldCount() {
   if (getReadLockCount() == 0)
       return 0;
   Thread current = Thread.currentThread();
   if (firstReader == current)
       return firstReaderHoldCount;
   HoldCounter rh = cachedHoldCounter;
   if (rh != null && rh.tid == current.getId())
       return rh.count;
   int count = readHolds.get().count;
   if (count == 0) readHolds.remove();
   return count;
}
/**
* 判断写锁是否被获取
*/
final boolean isWriteLocked() {
   return exclusiveCount(getState()) != 0;
}
/**
* 判断当前写锁被获取的次数
*/
final int getWriteHoldCount() {
   return isHeldExclusively() ? exclusiveCount(getState()) : 0;
}
```
下面我们来看一个通过缓存示例说明读写锁的使用
```java
public class Cache {
    static Map<String,Object> map = new HashMap<String,Object>();
    static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    static Lock r = rwl.readLock();
    static Lock w = rwl.writeLock();

    public static final Object get(String key){
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }

    public static final Object put(String key,String value){
        w.lock();
        try{
            return map.put(key, value);
        }finally{
            w.unlock();
        }
    }

    public static final void clear(){
        w.lock();
        try{
            map.clear();
        }finally{
            w.unlock();
        }
    }

}
```
上面的HashMap虽然是非线程安全，但是我们在get和put时候分别使用读写锁，保证了线程安全，其中写锁支持并发访问。

#### 读写锁的实现原理分析

读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。回想ReentrantLock中自定义同步器的实现，同步状态表示锁被一个线程重复获取的次数，而读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态，使得该状态的设计成为读写锁实现的关键。如果在一个整型变量上维护多种状态，就一定需要“按位切割使用”这个变量，读写锁将变量切分成了两个部分，高16位表示读，低16位表示写

当前同步状态表示一个线程已经获取了写锁，且重进入了两次，同时也连续获取了两次读锁。读写锁是如何迅速确定读和写各自的状态呢？答案是通过位运算。假设当前同步状态值为S，写状态等于S&0x0000FFFF（将高16位全部抹去），读状态等于S>>>16（无符号补0右移16位）。当写状态增加1时，等于S+1，当读状态增加1时，等于S+(1<<16)，也就是S+0x00010000。根据状态的划分能得出一个推论：S不等于0时，当写状态（S&0x0000FFFF）等于0时，则读状态（S>>>16）大于0，即读锁已被获取。

#### 写锁的获取与释放

写锁是一个支持重进入的排它锁。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程不是已经获取写锁的线程，则当前线程进入等待状态，我们看下ReentrantReadWriteLock的tryAcquire方法

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        // 存在读锁或者当前获取线程不是已经获取锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
该方法除了重入条件（当前线程为获取了写锁的线程）之外，增加了一个读锁是否存在的判断。如果存在读锁，则写锁不能被获取，原因在于：读写锁要确保写锁的操作对读锁可见，如果允许读锁在已被获取的情况下对写锁的获取，那么正在运行的其他读线程就无法感知到当前写线程的操作。因此，只有等待其他读线程都释放了读锁，写锁才能被当前线程获取，而写锁一旦被获取，则其他读写线程的后续访问均被阻塞。写锁的释放与ReentrantLock的释放过程基本类似，每次释放均减少写状态，当写状态为0时表示写锁已被释放，从而等待的读写线程能够继续访问读写锁，同时前次写线程的修改对后续读写线程可见

#### 读锁的获取与释放
读锁是一个支持重进入的共享锁，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，而所做的也只是（线程安全的）增加读状态。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。获取读锁的实现从Java 5到Java 6变得复杂许多，主要原因是新增了一些功能，例如getReadHoldCount()方法，作用是返回当前线程获取读锁的次数。读状态是所有线程获取读锁次数的总和，而每个线程各自获取读锁的次数只能选择保存在ThreadLocal中，由线程自身维护，这使获取读锁的实现变得复杂

```java
protected final int tryAcquireShared(int unused) {
      /*
       * Walkthrough:
       * 1. If write lock held by another thread, fail.
       * 2. Otherwise, this thread is eligible for
       *    lock wrt state, so ask if it should block
       *    because of queue policy. If not, try
       *    to grant by CASing state and updating count.
       *    Note that step does not check for reentrant
       *    acquires, which is postponed to full version
       *    to avoid having to check hold count in
       *    the more typical non-reentrant case.
       * 3. If step 2 fails either because thread
       *    apparently not eligible or CAS fails or count
       *    saturated, chain to version with full retry loop.
       */
      Thread current = Thread.currentThread();
      int c = getState();
      if (exclusiveCount(c) != 0 &&
          getExclusiveOwnerThread() != current)
          return -1;
      int r = sharedCount(c);
      if (!readerShouldBlock() &&
          r < MAX_COUNT &&
          compareAndSetState(c, c + SHARED_UNIT)) {
          if (r == 0) {
              firstReader = current;
              firstReaderHoldCount = 1;
          } else if (firstReader == current) {
              firstReaderHoldCount++;
          } else {
              HoldCounter rh = cachedHoldCounter;
              if (rh == null || rh.tid != current.getId())
                  cachedHoldCounter = rh = readHolds.get();
              else if (rh.count == 0)
                  readHolds.set(rh);
              rh.count++;
          }
          return 1;
      }
      return fullTryAcquireShared(current);
  }
```
在tryAcquireShared(int unused)方法中，如果其他线程已经获取了写锁，则当前线程获取读锁失败，进入等待状态。如果当前线程获取了写锁或者写锁未被获取，则当前线程（线程安全，依靠CAS保证）增加读状态，成功获取读锁。读锁的每次释放（线程安全的，可能有多个读线程同时释放读锁）均减少读状态，减少的值是（1<<16）


#### 锁降级

锁降级指的是写锁降级成为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程。
```java
public void processData() {
        readLock.lock();
        if (!update) {
            // 必须先释放读锁
            readLock.unlock();
            // 锁降级从写锁获取到开始
            writeLock.lock();
            try {
                if (!update) {
                    // 准备数据的流程（略）
                    update = true;
                }
                readLock.lock();
            } finally {
                writeLock.unlock();
            }// 锁降级完成，写锁降级为读锁
        }
        try {// 使用数据的流程（略）
        } finally {
            readLock.unlock();
        }
    }
```
上述示例中，当数据发生变更后，update变量（布尔类型且volatile修饰）被设置为false，此时所有访问processData()方法的线程都能够感知到变化，但只有一个线程能够获取到写锁，其他线程会被阻塞在读锁和写锁的lock()方法上。当前线程获取写锁完成数据准备之后，再获取读锁，随后释放写锁，完成锁降级。锁降级要先获取写锁为了保证线程安全，如果释放了写锁此时其他线程进行写操作我在在获取读锁读到的可能不是最新数据。

### AQS

AQS是AbstractQueuedSynchronizer的简称。AQS提供了一种实现**阻塞锁和一系列依赖FIFO等待队列的同步器**的框架，为一系列同步器依赖于一个单独的原子变量（state）的同步器提供了一个非常有用的基础。子类们必须定义改变state变量的protected方法，这些方法定义了state是如何被获取或释放的。鉴于此，本类中的其他方法执行所有的排队和阻塞机制。子类也可以维护其他的state变量，但是为了保证同步，必须原子地操作这些变量。

state的访问方式有三种:

> getState()
>  setState()
>  compareAndSetState()

**AQS定义两种资源共享方式：**
**Exclusive**（独占，**只有一个线程能执行**，如ReentrantLock）
**Share**（共享，**多个线程可同时执行**，如Semaphore/CountDownLatch）

#### AQS的设计

##### AQS核心

AQS则实现了**对同步状态的管理，以及对阻塞线程进行排队，等待通知**等等一些底层的实现处理。AQS的核心也包括了这些方面:**同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取**这些特性的实现。

从AQS提供的模板方法可以分为2类锁机制：

1. 独占式获取与释放同步状态

   ```
   void acquire(int arg)：独占式获取同步状态，如果获取失败则插入同步队列进行等待；
   void acquireInterruptibly(int arg)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
   boolean tryAcquireNanos(int arg, long nanosTimeout)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
   boolean release(int arg)：释放同步状态，该方法会唤醒在同步队列中的下一个节点
   
   ```

2. 共享式获取与释放同步状态

   ```
   void acquireShared(int arg)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；
   void acquireSharedInterruptibly(int arg)：在acquireShared方法基础上增加了能响应中断的功能；
   boolean tryAcquireSharedNanos(int arg, long nanosTimeout)：在acquireSharedInterruptibly基础上增加了超时等待的功能；
   boolean releaseShared(int arg)：共享式释放同步状态
   ```

##### 自定义同步器

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

![lUHlWt.png](https://s2.ax1x.com/2020/01/03/lUHlWt.png)

##### Java的AQS已实现的实例

以**ReentrantLock（独占方式）**为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是**可重入**的概念。但要注意，获取多少次就要释放多么次，这样才能**保证state是能回到零态**的。

以**CountDownLatch（共享方式）**以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方式，要么是共享方式，但AQS也支持自定义同步器**同时实现独占和共享两种方式**，如**ReentrantReadWriteLock**

#### AQS的源码分析

##### 同步队列

上面所述的FIFO等待队列，那么我们看看它到底是个什么东西。

当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列。就数据结构而言，队列的实现方式无外乎两者一是通过数组的形式，另外一种则是链表的形式。**AQS中的同步队列则是通过链式方式进行实现。**

```java
/**
   * <p>To enqueue into a CLH lock, you atomically splice it in as new
     * tail. To dequeue, you just set the head field.
     * <pre>
     *      +------+  prev +-----+       +-----+
     * head |      | <---- |     | <---- |     |  tail
     *      +------+       +-----+       +-----+
     * </pre>
**/
 static final class Node {
        //指示节点在共享模式下等待的标记
        static final Node SHARED = new Node();
        //标记，指示节点在独占模式下等待
        static final Node EXCLUSIVE = null;
        //表示线程已被取消的waitStatus值
        static final int CANCELLED =  1;
       //后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行；
        static final int SIGNAL    = -1;
        //waitStatus值，当前节点进入等待队列中
        static final int CONDITION = -2;
        //表示下一个默认值的waitStatus值表示下一次共享式同步状态获取将会无条件传播下去
        static final int PROPAGATE = -3;

        volatile int waitStatus;//当前节点状态，上面所列的静态变量
        volatile Node prev; //当前节点/线程的前驱节点 
        volatile Node next;//当前节点/线程的后继节点 
        volatile Thread thread;//加入同步队列的线程引用
        Node nextWaiter;//等待队列中的下一个节点

        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

上面就是节点的数据结构类型，并且每个节点拥有其前驱和后继节点，很显然这是一个**双向队列**，可以总结为下图：

![lUHteg.png](https://s2.ax1x.com/2020/01/03/lUHteg.png)

通过对源码的理解现在我们可以清楚的知道这样几点：

1. 节点的数据结构，即AQS的静态内部类Node,节点的等待状态等信息；
2. 同步队列是一个双向队列，AQS通过持有头尾指针管理同步队列；
3. 节点的入队和出队实际上这对应着锁的获取和释放两个操作：获取锁失败进行入队操作，获取锁成功进行出队操作。

##### 独占锁

###### 独占锁的获取（acquire方法）

```java
public final void acquire(int arg) {
        //先看同步状态是否获取成功，如果成功则方法结束返回
        //若失败则先调用addWaiter()方法再调用acquireQueued()方法
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

acquire函数的流程如下：

> tryAcquire()尝试直接去获取资源，如果成功则直接返回；
>  addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
>  acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
>  如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

1. tryAcquire(int)

   此方法尝试去获取独占资源。如果获取成功，则直接返回true，否则直接返回false。这也正是tryLock()的语义，还是那句话，当然不仅仅只限于tryLock()。如下是tryAcquire()的源码：

   ```java
   protected boolean tryAcquire(int arg) {
         throw new UnsupportedOperationException();
   }
   ```

   什么？直接throw异常？说好的功能呢？好吧，还记得概述里讲的AQS只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现吗？就是这里了！！！AQS这里只定义了一个接口，**具体资源的获取交由自定义同步器去实现了（通过state的get/set/CAS）！！！**至于能不能重入，能不能加塞，那就看具体的自定义同步器怎么去设计了！！！**当然，自定义同步器在进行资源访问时要考虑线程安全的影响。**

   **这里之所以没有定义成abstract**，是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。说到底，Doug Lea还是站在咱们开发者的角度，尽量减少不必要的工作量。

2. addWaiter

   ```csharp
   private Node addWaiter(Node mode) {
           // 1. 将当前线程构建成Node类型
           Node node = new Node(Thread.currentThread(), mode);
           // Try the fast path of enq; backup to full enq on failure
           // 2. 当前尾节点是否为null？
           Node pred = tail;
           if (pred != null) {
               // 2.2 将当前节点尾插入的方式插入同步队列中
               node.prev = pred;
               if (compareAndSetTail(pred, node)) {
                   pred.next = node;
                   return node;
               }
           }
           // 2.1. 当前同步队列尾节点为null，说明当前线程是第一个加入同步队列进行等待的线程
           enq(node);
           return node;
   }
   ```

   程序的逻辑主要分为两个部分：

    **1. 当前同步队列的尾节点为null，调用方法enq()插入;**
    **2. 当前队列的尾节点不为null，则采用尾插入（compareAndSetTail（）方法）的方式入队。**

   如果 if (compareAndSetTail(pred, node))为false会继续执行到**enq()方法**，同时很明显compareAndSetTail是一个CAS操作，通常来说如果CAS操作失败会继续**自旋（死循环）**进行重试

   ```java
   private Node enq(final Node node) {
       //CAS"自旋"，直到成功加入队尾
       for (;;) {
           Node t = tail;
           if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
               if (compareAndSetHead(new Node()))
                   tail = head;
           } else {//正常流程，放入队尾
               node.prev = t;
               if (compareAndSetTail(t, node)) {
                   t.next = node;
                   return t;
               }
           }
       }
   }
   ```

   **对enq()方法可以做这样的总结：**

   在当前线程是第一个加入同步队列时，调用compareAndSetHead(new Node())方法，完成链式队列的头结点的初始化；

   **自旋不断尝试CAS尾插入节点直至成功为止。**

3. acquireQueued

   通过tryAcquire()和addWaiter()，该线程获取资源失败，已经被放入等待队列尾部了。**该线程下一步该干什么了呢：**进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了。没错，就是这样！是不是跟医院排队拿号有点相似.

   ```java
   final boolean acquireQueued(final Node node, int arg) {
       boolean failed = true;//标记是否成功拿到资源
       try {
           boolean interrupted = false;//标记等待过程中是否被中断过
           
           //又是一个“自旋”！
           for (;;) {
               final Node p = node.predecessor();//拿到前驱
               //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
               if (p == head && tryAcquire(arg)) {
                   setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                   p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                   failed = false;
                   return interrupted;//返回等待过程中是否被中断过
               }
               
               //如果自己可以休息了，就进入waiting状态，直到被unpark()
               if (shouldParkAfterFailedAcquire(p, node) &&
                   parkAndCheckInterrupt())
                   interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
           }
       } finally {
           if (failed)
               cancelAcquire(node);
       }
   }
   ```

   其中有两个方法需要了解的shouldParkAfterFailedAcquire和parkAndCheckInterrupt

   **shouldParkAfterFailedAcquire()**

    此方法主要用于检查状态，看看自己是否真的可以去休息了。

   ```java
   private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
       int ws = pred.waitStatus;//拿到前驱的状态
       if (ws == Node.SIGNAL)
           //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
           return true;
       if (ws > 0) {
           /*
            * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
            * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
            */
           do {
               node.prev = pred = pred.prev;
           } while (pred.waitStatus > 0);
           pred.next = node;
       } else {
            //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
           compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
       }
       return false;
   }
   ```

   整个流程中，如果前驱结点的状态不是SIGNAL，那么自己就不能安心去休息，需要去找个安心的休息点，同时可以再尝试下看有没有机会轮到自己拿号。

   **parkAndCheckInterrupt()**

    如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。

   ```java
   private final boolean parkAndCheckInterrupt() {
           LockSupport.park(this);
           return Thread.interrupted();
       }
   ```

   ##### 小结：

   acquireQueued的主要流程：

   > 1.调用自定义同步器的tryAcquire()尝试直接去获取资源，如果成功则直接返回；
   >  2.没成功，则addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
   >  3.acquireQueued()使线程在等待队列中休息，有机会时（轮到自己，会被unpark()）会去尝试获取资源。获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
   >  4.如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

   ![lUHxpt.png](https://s2.ax1x.com/2020/01/03/lUHxpt.png)

###### 独占锁的释放（release方法）

它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

1. tryRelease

   它的返回值用来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计tryRelease()的时候要明确这一点！！

   ```java
    protected boolean tryRelease(int arg) {
           throw new UnsupportedOperationException();
    }
   ```

   跟tryAcquire()一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！**所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。**

2. unparkSuccessor

   此方法用于唤醒等待队列中下一个线程。

   ```csharp
   private void unparkSuccessor(Node node) {
       //这里，node一般为当前线程所在的结点。
       int ws = node.waitStatus;
       if (ws < 0)//置零当前线程所在的结点状态，允许失败。
           compareAndSetWaitStatus(node, ws, 0);
   
       Node s = node.next;//找到下一个需要唤醒的结点s
       if (s == null || s.waitStatus > 0) {//如果为空或已取消
           s = null;
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                   s = t;
       }
       if (s != null)
           LockSupport.unpark(s.thread);//唤醒
   }
   ```

   一句话概括：用unpark()唤醒等待队列中最前边的那个未放弃线程

##### 共享锁

###### 共享锁的获取acquireShared(int)

此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。下面是acquireShared()的源码：

```java
public final void acquireShared(int arg) {
      if (tryAcquireShared(arg) < 0)
          doAcquireShared(arg);
}
```

这里tryAcquireShared()依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里acquireShared()的流程就是：

> tryAcquireShared()尝试获取资源，成功则直接返回；
>  失败则通过doAcquireShared()进入等待队列，直到获取到资源为止才返回。

1. doAcquireShared

   ```java
   private void doAcquireShared(int arg) {
       final Node node = addWaiter(Node.SHARED);//加入队列尾部
       boolean failed = true;//是否成功标志
       try {
           boolean interrupted = false;//等待过程中是否被中断过的标志
           for (;;) {
               final Node p = node.predecessor();//前驱
               if (p == head) {//如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
                   int r = tryAcquireShared(arg);//尝试获取资源
                   if (r >= 0) {//成功
                       setHeadAndPropagate(node, r);//将head指向自己，还有剩余资源可以再唤醒之后的线程
                       p.next = null; // help GC
                       if (interrupted)//如果等待过程中被打断过，此时将中断补上。
                           selfInterrupt();
                       failed = false;
                       return;
                   }
               }
               
               //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
               if (shouldParkAfterFailedAcquire(p, node) &&
                   parkAndCheckInterrupt())
                   interrupted = true;
           }
       } finally {
           if (failed)
               cancelAcquire(node);
       }
   }
   ```

   有木有觉得跟acquireQueued()很相似？对，其实流程并没有太大区别。只不过这里将补中断的selfInterrupt()放到doAcquireShared()里了，而独占模式是放到acquireQueued()之外，其实都一样

   **注意：**

    跟独占模式比，还有一点需要注意的是，这里只有线程是head.next时（“老二”），才会去尝试获取资源，有剩余的话还会唤醒之后的队友。那么问题就来了，假如老大用完后释放了5个资源，而老二需要6个，老三需要1个，老四需要2个。老大先唤醒老二，老二一看资源不够，他是把资源让给老三呢，还是不让？**答案是否定的！**老二会继续park()等待其他线程释放资源，也更不会去唤醒老三和老四了。独**占模式，同一时刻只有一个线程去执行，这样做未尝不可；**但共享模式下，多个线程是可以同时执行的，现在因为老二的资源需求量大，**而把后面量小的老三和老四也都卡住了。**当然，这并不是问题，只是AQS保证严格按照入队顺序唤醒罢了（保证公平，但降低了并发）。

2. setHeadAndPropagate(Node, int)

   此方法在setHead()的基础上多了一步，就是自己苏醒的同时，如果条件符合（比如还有剩余资源），还会去唤醒后继结点，毕竟是共享模式！

   ```csharp
   private void setHeadAndPropagate(Node node, int propagate) {
       Node h = head; 
       setHead(node);//head指向自己
        //如果还有剩余量，继续唤醒下一个邻居线程
       if (propagate > 0 || h == null || h.waitStatus < 0) {
           Node s = node.next;
           if (s == null || s.isShared())
               doReleaseShared();
       }
   }
   ```

###### 共享锁的释放releaseShared

此方法是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。下面是releaseShared()的源码：

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放资源
        doReleaseShared();//唤醒后继结点
        return true;
    }
    return false;
}
```

此方法的流程也比较简单，一句话：释放掉资源后，唤醒后继。跟独占模式下的release()相似，但有一点稍微需要注意：独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。

例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后tryReleaseShared(2)返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，tryReleaseShared(2)返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定tryReleaseShared()的返回值。

```csharp
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);//唤醒后继
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)// head发生变化
            break;
    }
}
```

#### 简单应用

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
- tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

OK，下面我们就以AQS源码里的Mutex为例，讲一下AQS的简单应用。

##### Mutex（互斥锁）

Mutex是一个不可重入的互斥锁实现。锁资源（AQS里的state）只有两种状态：0表示未锁定，1表示锁定。下边是Mutex的核心源码：

```java
class Mutex implements Lock, java.io.Serializable {
    // 自定义同步器
    private static class Sync extends AbstractQueuedSynchronizer {
        // 判断是否锁定状态
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 尝试获取资源，立即返回。成功则返回true，否则false。
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // 这里限定只能为1个量
            if (compareAndSetState(0, 1)) {//state为0才设置为1，不可重入！
                setExclusiveOwnerThread(Thread.currentThread());//设置为当前线程独占资源
                return true;
            }
            return false;
        }

        // 尝试释放资源，立即返回。成功则为true，否则false。
        protected boolean tryRelease(int releases) {
            assert releases == 1; // 限定为1个量
            if (getState() == 0)//既然来释放，那肯定就是已占有状态了。只是为了保险，多层判断！
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);//释放资源，放弃占有状态
            return true;
        }
    }

    // 真正同步类的实现都依赖继承于AQS的自定义同步器！
    private final Sync sync = new Sync();

    //lock<-->acquire。两者语义一样：获取资源，即便等待，直到成功才返回。
    public void lock() {
        sync.acquire(1);
    }

    //tryLock<-->tryAcquire。两者语义一样：尝试获取资源，要求立即返回。成功则为true，失败则为false。
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    //unlock<-->release。两者语文一样：释放资源。
    public void unlock() {
        sync.release(1);
    }

    //锁是否占有状态
    public boolean isLocked() {
        return sync.isHeldExclusively();
    }
}
```

同步类在实现时一般都将自定义同步器（sync）定义为内部类，供自己使用；而同步类自己（Mutex）则实现某个接口，对外服务。当然，接口的实现要直接依赖sync，它们在语义上也存在某种对应关系！！而sync只用实现资源state的获取-释放方式tryAcquire-tryRelelase，至于线程的排队、等待、唤醒等，上层的AQS都已经实现好了，我们不用关心。

除了Mutex，ReentrantLock/CountDownLatch/Semphore这些同步类的实现方式都差不多，不同的地方就在获取-释放资源的方式tryAcquire-tryRelelase。掌握了这点，AQS的核心便被攻破了！

### 参考
1. [Java多线程之ReentrantLock与Condition](https://www.cnblogs.com/xiaoxi/p/7651360.html)
2. [Java多线程读写锁ReentrantReadWriteLock原理详解](https://blog.csdn.net/fuyuwei2015/article/details/72597192)
3. [深入理解读写锁ReentrantReadWriteLock](https://www.jianshu.com/p/4a624281235e)
4. [Java的AQS](https://www.jianshu.com/p/9d87e3111640)
