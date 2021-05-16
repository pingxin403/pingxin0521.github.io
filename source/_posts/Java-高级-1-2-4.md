---
title: Java Thread 安全--Condition等待/通知机制（四）
date: 2019-05-06 13:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### Condition

Condition是在java 1.5中才出现的，它用来替代传统的Object的wait()、notify()实现线程间的协作，相比使用Object的wait()、notify()，使用Condition的await()、signal()这种方式实现线程间协作更加安全和高效。因此通常来说比较推荐使用Condition。

<!--more-->

Condition类能实现synchronized和wait、notify搭配的功能，另外比后者更灵活，Condition可以实现多路通知功能，也就是在一个Lock对象里可以创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而可以有选择的进行线程通知，在调度线程上更加灵活。而synchronized就相当于整个Lock对象中只有一个单一的Condition对象，所有的线程都注册在这个对象上。线程开始notifyAll时，需要通知所有的WAITING线程，没有选择权，会有相当大的效率问题。

1. Condition是个接口，基本的方法就是await()和signal()方法。

2. Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition()。
```java
//实例化一个ReentrantLock对象
private ReentrantLock lock=new ReentrantLock();
//为线程A注册一个Condition
public Condition conditionA=lock.newCondition();
//为线程B注册一个Condition
public Condition conditionB=lock.newCondition();
```
3. 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用。

4. Conditon中的await()对应Object的wait()，Condition中的signal()对应Object的notify()，Condition中的signalAll()对应Object的notifyAll()。

从整体上来看Object的wait和notify/notify是与对象监视器配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性。两者除了在使用方式上不同外，在功能特性上还是有很多的不同：

- Condition能够支持不响应中断，而通过使用Object方式不支持；
- Condition能够支持多个等待队列（new 多个Condition对象），而Object方式只能支持一个；
- Condition能够支持超时时间的设置，而Object不支持


#### 方法

**针对Object的wait方法**

- void await() throws InterruptedException:当前线程进入等待状态，如果其他线程调用condition的signal或者signalAll方法并且当前线程获取Lock从await方法返回，如果在等待状态中被中断会抛出被中断异常；
- long awaitNanos(long nanosTimeout)：当前线程进入等待状态直到被通知，中断或者超时；
- boolean await(long time, TimeUnit unit)throws InterruptedException：同第二种，支持自定义时间单位
- boolean awaitUntil(Date deadline) throws InterruptedException：当前线程进入等待状态直到被通知，中断或者到了某个时间

**针对Object的notify/notifyAll方法**

- void signal()：唤醒一个等待在condition上的线程，将该线程从等待队列中转移到同步队列中，如果在同步队列中能够竞争到Lock则可以从等待方法中返回。
- void signalAll()：与1的区别在于能够唤醒所有等待在condition上的线程

通过使用condition提供的await和signal/signalAll方法就可以实现等待/通知机制，而这种机制能够解决最经典的问题就是“生产者与消费者问题”，关于“生产者消费者问题”之后会用单独的一篇文章进行讲解，这也是面试的高频考点。await和signal和signalAll方法就像一个开关控制着线程A（等待方）和线程B（通知方）。它们之间的关系可以用下面一个图来表现得更加贴切：

![1.png](https://i.loli.net/2019/05/06/5cd01ab9824e8.png)

如图，**线程awaitThread先通过lock.lock()方法获取锁成功后调用了condition.await方法进入等待队列，而另一个线程signalThread通过lock.lock()方法获取锁成功后调用了condition.signal或者signalAll方法，使得线程awaitThread能够有机会移入到同步队列中，当其他线程释放lock后使得线程awaitThread能够有机会获取lock，从而使得线程awaitThread能够从await方法中退出执行后续操作。如果awaitThread获取lock失败会直接进入到同步队列。**

#### 示例

接下来，使用Condition来实现等待/唤醒，并且能够唤醒制定线程。

```java
public class LockDemo3 {
    public static void main(String[] args) throws InterruptedException {
        MyService service = new MyService();
        Runnable runnable1 = new MyServiceThread1(service);
        Runnable runnable2 = new MyServiceThread2(service);

        new Thread(runnable1, "a").start();
        new Thread(runnable2, "b").start();

        // 线程sleep2秒钟
        Thread.sleep(2000);
        // 唤醒所有持有conditionA的线程
        service.signallA();

        Thread.sleep(2000);
        // 唤醒所有持有conditionB的线程
        service.signallB();
    }
}

//分别实例化了两个Condition对象，都是使用同一个lock注册。
// 注意conditionA对象的等待和唤醒只对使用了conditionA的线程有用，
// 同理conditionB对象的等待和唤醒只对使用了conditionB的线程有用。
class MyService {

    // 实例化一个ReentrantLock对象
    private ReentrantLock lock = new ReentrantLock();
    // 为线程A注册一个Condition
    public Condition conditionA = lock.newCondition();
    // 为线程B注册一个Condition
    public Condition conditionB = lock.newCondition();

    public void awaitA() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + "进入了awaitA方法");
            long timeBefore = System.currentTimeMillis();
            // 执行conditionA等待
            conditionA.await();
            long timeAfter = System.currentTimeMillis();
            System.out.println(Thread.currentThread().getName()+"被唤醒");
            System.out.println(Thread.currentThread().getName() + "等待了: " + (timeAfter - timeBefore)/1000+"s");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void awaitB() {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + "进入了awaitB方法");
            long timeBefore = System.currentTimeMillis();
            // 执行conditionB等待
            conditionB.await();
            long timeAfter = System.currentTimeMillis();
            System.out.println(Thread.currentThread().getName()+"被唤醒");
            System.out.println(Thread.currentThread().getName() + "等待了: " + (timeAfter - timeBefore)/1000+"s");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signallA() {
        try {
            lock.lock();
            System.out.println("启动唤醒程序");
            // 唤醒所有注册conditionA的线程
            conditionA.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void signallB() {
        try {
            lock.lock();
            System.out.println("启动唤醒程序");
            // 唤醒所有注册conditionB的线程
            conditionB.signalAll();
        } finally {
            lock.unlock();
        }
    }
}

//注意：MyServiceThread1 使用了awaitA()方法，持有的是conditionA！
class MyServiceThread1 implements Runnable{

    private MyService service;

    public MyServiceThread1(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
        service.awaitA();
    }
}
//注意：MyServiceThread2 使用了awaitB()方法，持有的是conditionB！
class MyServiceThread2 implements Runnable{

    private MyService service;

    public MyServiceThread2(MyService service) {
        this.service = service;
    }

    @Override
    public void run() {
        service.awaitB();
    }
}
```

使用conditionA的线程被唤醒，而后再唤醒使用conditionB的线程。学会使用Condition，那来用它实现生产者消费者模式。

#### 生产者和消费者

```java
public class LockDemo4 {
    public static void main(String[] args) {
        PCService service = new PCService();
        Runnable produce = new MyThreadProduce(service);
        Runnable consume = new MyThreadConsume(service);
        new Thread(produce, "生产者  ").start();
        new Thread(consume, "消费者  ").start();
    }

}

class PCService {

    private Lock lock = new ReentrantLock();
    private boolean flag = false;
    private Condition condition = lock.newCondition();
    // 以此为衡量标志
    private int number = 1;

    /**
     * 生产者生产
     */
    public void produce() {
        try {
            lock.lock();
            while (flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-----生产-----");
            number++;
            System.out.println("number: " + number);
            System.out.println();
            flag = true;
            // 提醒消费者消费
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    /**
     * 消费者消费生产的物品
     */
    public void consume() {
        try {
            lock.lock();
            while (!flag) {
                condition.await();
            }
            System.out.println(Thread.currentThread().getName() + "-----消费-----");
            number--;
            System.out.println("number: " + number);
            System.out.println();
            flag = false;
            // 提醒生产者生产
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

class MyThreadProduce implements Runnable{

    private PCService service;

    public MyThreadProduce(PCService service) {
        this.service = service;
    }

    @Override
    public void run() {
        for (;;) {
            service.produce();
        }
    }

}

class MyThreadConsume implements Runnable{

    private PCService service;

    public MyThreadConsume(PCService service) {
        this.service = service;
    }

    @Override
    public void run() {
        for (;;) {
            service.consume();
        }
    }
}
```
因为采用了无限循环，生产者线程和消费者线程会一直处于工作状态，可以看到，生产者线程执行完毕后，消费者线程就会执行，以这样的交替顺序，而且number也遵循着生产者生产+1，消费者消费-1的一个状态。这个就是使用ReentrantLock和Condition来实现的生产者消费者模式。

#### 顺序执行线程

充分发掘Condition的灵活性，可以用它来实现顺序执行线程。
```java
public class LockDemo5 {

    private static Runnable getThreadA(final ConditionService service) {
        return new Runnable() {
            @Override
            public void run() {
                for (int i=0;i<10;i++) {
                    service.excuteA();
                }
            }
        };
    }

    private static Runnable getThreadB(final ConditionService service) {
        return new Runnable() {
            @Override
            public void run() {
                for (int i=0;i<10;i++) {
                    service.excuteB();
                }
            }
        };
    }

    private static Runnable getThreadC(final ConditionService service) {
        return new Runnable() {
            @Override
            public void run() {
                for (int i=0;i<10;i++) {
                    service.excuteC();
                }
            }
        };
    }

    public static void main(String[] args) throws InterruptedException{
        ConditionService service = new ConditionService();
        Runnable A = getThreadA(service);
        Runnable B = getThreadB(service);
        Runnable C = getThreadC(service);

        new Thread(A, "A").start();
        new Thread(B, "B").start();
        new Thread(C, "C").start();
    }

}

class ConditionService {

    // 通过nextThread控制下一个执行的线程
    private static int nextThread = 1;
    private ReentrantLock lock = new ReentrantLock();
    // 有三个线程，所以注册三个Condition
    Condition conditionA = lock.newCondition();
    Condition conditionB = lock.newCondition();
    Condition conditionC = lock.newCondition();

    public void excuteA() {
        try {
            lock.lock();
            while (nextThread != 1) {
                conditionA.await();
            }
            System.out.println(Thread.currentThread().getName() + " 工作");
            nextThread = 2;
            conditionB.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void excuteB() {
        try {
            lock.lock();
            while (nextThread != 2) {
                conditionB.await();
            }
            System.out.println(Thread.currentThread().getName() + " 工作");
            nextThread = 3;
            conditionC.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void excuteC() {
        try {
            lock.lock();
            while (nextThread != 3) {
                conditionC.await();
            }
            System.out.println(Thread.currentThread().getName() + " 工作");
            nextThread = 1;\
            conditionA.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```
A,B,C三个线程一直按照顺序执行。

#### 实现原理

在java中，条件锁的实现都在AQS的ConditionObject类中，ConditionObject实现了Condition接口，下面我们通过一个例子来进入到条件锁的学习中。

`ConditionObject`是通过基于**单链表的条件队列**来管理等待线程的。线程在调用`await`方法进行等待时，会释放同步状态。同时线程将会被封装到一个等待节点中，并将节点置入条件队列尾部进行等待。当有线程在获取独占锁的情况下调用`signal`或`singalAll`方法时，队列中的等待线程将会被唤醒，重新竞争锁。另外，需要说明的是，一个锁对象可同时创建多个 ConditionObject 对象，这意味着多个竞争同一独占锁的线程可在不同的条件队列中进行等待。在唤醒时，可唤醒指定条件队列中的线程。其大致示意图如下：

![1An6SA.png](https://s2.ax1x.com/2020/01/22/1An6SA.png)

以上就是 ConditionObject 所实现的等待/通知机制的大致原理，并不是很难理解。当然，在具体的实现中，则考虑的更为细致一些。相关细节将会在接下来一章中进行说明，继续往下看吧。

#### 源码分析

##### ConditionObject的主要属性

```java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
}
```

可以看到条件锁中也维护了一个队列，为了和AQS的队列区分，我这里称为条件队列，firstWaiter是队列的头节点，lastWaiter是队列的尾节点，它们是干什么的呢？接着看。

##### lock.newCondition()方法

新建一个条件锁。

```java
// ReentrantLock.newCondition()
public Condition newCondition() {
    return sync.newCondition();
}
// ReentrantLock.Sync.newCondition()
final ConditionObject newCondition() {
    return new ConditionObject();
}
// AbstractQueuedSynchronizer.ConditionObject.ConditionObject()
public ConditionObject() { }
```

新建一个条件锁最后就是调用的AQS中的ConditionObject类来实例化条件锁。

##### condition.await()方法

ConditionObject 中实现了几种不同的等待方法，每种方法均有它自己的特点。比如`await()`会响应中断，而`awaitUninterruptibly()`则不响应中断。`await(long, TimeUnit)`则会在响应中断的基础上，新增了超时功能。除此之外，还有一些等待方法，这里就不一一列举了。

condition.await()方法，表明现在要等待条件的出现。

```java
/**
 * await 是一个响应中断的等待方法，主要逻辑流程如下：
 * 1. 如果线程中断了，抛出 InterruptedException 异常
 * 2. 将线程封装到节点对象里，并将节点添加到条件队列尾部
 * 3. 保存并完全释放同步状态，保存下来的同步状态在重新竞争锁时会用到
 * 4. 线程进入等待状态，直到被通知或中断才会恢复运行
 * 5. 使用第3步保存的同步状态去竞争独占锁
 */
// AbstractQueuedSynchronizer.ConditionObject.await()
public final void await() throws InterruptedException {
    // 如果线程中断了，抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    // 添加节点到Condition的队列中，并返回该节点
    Node node = addConditionWaiter();
    // 完全释放当前线程获取的锁
    // 因为锁是可重入的，所以这里要把获取的锁全部释放
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    // 是否在同步队列中
     /*
     * 判断节点是否在同步队列上，如果不在则阻塞线程。
     * 循环结束的条件：
     * 1. 其他线程调用 singal/singalAll，node 将会被转移到同步队列上。node 对应线程将
     *    会在获取同步状态的过程中被唤醒，并走出 while 循环。
     * 2. 线程在阻塞过程中产生中断
     */ 
    while (!isOnSyncQueue(node)) {
        // 阻塞当前线程
        LockSupport.park(this);
        
        // 上面部分是调用await()时释放自己占有的锁，并阻塞自己等待条件的出现
        // *************************分界线*************************  //
        // 下面部分是条件已经出现，尝试去获取锁
         /*
         * 检测中断模式，这里有两种中断模式，如下：
         * THROW_IE：
         *     中断在 node 转移到同步队列“前”发生，需要当前线程自行将 node 转移到同步队
         *     列中，并在随后抛出 InterruptedException 异常。
         *     
         * REINTERRUPT：
         *     中断在 node 转移到同步队列“期间”或“之后”发生，此时表明有线程正在调用 
         *     singal/singalAll 转移节点。在该种中断模式下，再次设置线程的中断状态。
         *     向后传递中断标志，由后续代码去处理中断。
         */
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    
    // 尝试获取锁，注意第二个参数，这是上一章分析过的方法
   /*
     * 被转移到同步队列的节点 node 将在 acquireQueued 方法中重新获取同步状态，注意这里
     * 的这里的 savedState 是上面调用 fullyRelease 所返回的值，与此对应，可以把这里的 
     * acquireQueued 作用理解为 fullyAcquire（并不存在这个方法）。
     * 
     * 如果上面的 while 循环没有产生中断，则 interruptMode = 0。但 acquireQueued 方法
     * 可能会产生中断，产生中断时返回 true。这里仍将 interruptMode 设为 REINTERRUPT，
     * 目的是继续向后传递中断，acquireQueued 不会处理中断。
     */
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    // 清除取消的节点
    /*
     * 正常通过 singal/singalAll 转移节点到同步队列时，nextWaiter 引用会被置空。
     * 若发生线程产生中断（THROW_IE）或 fullyRelease 方法出现错误等异常情况，
     * 该引用则不会被置空
     */ 
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    // 线程中断相关
    if (interruptMode != 0)
         /*
         * 根据 interruptMode 觉得中断的处理方式：
         *   THROW_IE：抛出 InterruptedException 异常
         *   REINTERRUPT：重新设置线程中断标志
         */ 
        reportInterruptAfterWait(interruptMode);
}
/** 将当先线程封装成节点，并将节点添加到条件队列尾部 */
// AbstractQueuedSynchronizer.ConditionObject.addConditionWaiter
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // 如果条件队列的尾节点已取消，从头节点开始清除所有已取消的节点
    /*
     * 清理等待状态为 CANCELLED 的节点。fullyRelease 内部调用 release 发生异常或释放同步状
     * 态失败时，节点的等待状态会被设置为 CANCELLED。所以这里要清理一下已取消的节点
     */
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        // 重新获取尾节点
        t = lastWaiter;
    }
    // 新建一个节点，它的等待状态是CONDITION
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    // 如果尾节点为空，则把新节点赋值给头节点（相当于初始化队列）
    // 否则把新节点赋值给尾节点的nextWaiter指针
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    // 尾节点指向新节点
    lastWaiter = node;
    // 返回新节点
    return node;
}

/** 清理等待状态为 CANCELLED 的节点 */ 
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    // 指向上一个等待状态为非 CANCELLED 的节点
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            /*
             * trail 为 null，表明 next 之前的节点等待状态均为 CANCELLED，此时更新 
             * firstWaiter 引用的指向。
             * trail 不为 null，表明 next 之前有节点的等待状态为 CONDITION，这时将 
             * trail.nextWaiter 指向 next 节点。
             */
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            // next 为 null，表明遍历到条件队列尾部了，此时将 lastWaiter 指向 trail
            if (next == null)
                lastWaiter = trail;
        }
        else
            // t.waitStatus = Node.CONDITION，则将 trail 指向 t
            trail = t;
        t = next;
    }
}
/**
 * 这个方法用于完全释放同步状态。这里解释一下完全释放的原因：为了避免死锁的产生，锁的实现上
 * 一般应该支持重入功能。对应的场景就是一个线程在不释放锁的情况下可以多次调用同一把锁的 
 * lock 方法进行加锁，且不会加锁失败，如失败必然导致导致死锁。锁的实现类可通过 AQS 中的整型成员
 * 变量 state 记录加锁次数，每次加锁，将 state++。每次 unlock 方法释放锁时，则将 state--，
 * 直至 state = 0，线程完全释放锁。用这种方式即可实现了锁的重入功能。
 */
// AbstractQueuedSynchronizer.fullyRelease
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        // 获取状态变量的值，重复获取锁，这个值会一直累加
        // 所以这个值也代表着获取锁的次数
        int savedState = getState();
        // 一次性释放所有获得的锁
        if (release(savedState)) {
            failed = false;
            // 返回获取锁的次数
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
/** 该方法用于判断节点 node 是否在同步队列上 */
// AbstractQueuedSynchronizer.isOnSyncQueue
final boolean isOnSyncQueue(Node node) {
    // 如果等待状态是CONDITION，或者前一个指针为空，返回false
    // 说明还没有移到AQS的队列中
     /*
     * 节点在同步队列上时，其状态可能为 0、SIGNAL、PROPAGATE 和 CANCELLED 其中之一，
     * 但不会为 CONDITION，所以可已通过节点的等待状态来判断节点所处的队列。
     * 
     * node.prev 仅会在节点获取同步状态后，调用 setHead 方法将自己设为头结点时被置为 
     * null，所以只要节点在同步队列上，node.prev 一定不会为 null
     */
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    // 如果next指针有值，说明已经移到AQS的队列中了
        /*
     * 如果节点后继被为 null，则表明节点在同步队列上。因为条件队列使用的是 nextWaiter 指
     * 向后继节点的，条件队列上节点的 next 指针均为 null。但仅以 node.next != null 条
     * 件断定节点在同步队列是不充分的。节点在入队过程中，是先设置 node.prev，后设置 
     * node.next。如果设置完 node.prev 后，线程被切换了，此时 node.next 仍然为 
     * null，但此时 node 确实已经在同步队列上了，所以这里还需要进行后续的判断。
     */
    if (node.next != null) // If has successor, it must be on queue
        return true;
    // 从AQS的尾节点开始往前寻找看是否可以找到当前节点，找到了也说明已经在AQS的队列中了
    return findNodeFromTail(node);
}

/** 由于同步队列上的的节点 prev 引用不会为空，所以这里从后向前查找 node 节点 */
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}

/** 检测线程在等待期间是否发生了中断 */
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
}

/** 
 * 判断中断发生的时机，分为两种：
 * 1. 中断在节点被转移到同步队列前发生，此时返回 true
 * 2. 中断在节点被转移到同步队列期间或之后发生，此时返回 false
 */
final boolean transferAfterCancelledWait(Node node) {

    // 中断在节点被转移到同步队列前发生，此时自行将节点转移到同步队列上，并返回 true
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
        // 调用 enq 将节点转移到同步队列中
        enq(node);
        return true;
    }
    
    /*
     * 如果上面的条件分支失败了，则表明已经有线程在调用 signal/signalAll 方法了，这两个
     * 方法会先将节点等待状态由 CONDITION 设置为 0 后，再调用 enq 方法转移节点。下面判断节
     * 点是否已经在同步队列上的原因是，signal/signalAll 方法可能仅设置了等待状态，还没
     * 来得及转移节点就被切换走了。所以这里用自旋的方式判断 signal/signalAll 是否已经完
     * 成了转移操作。这种情况表明了中断发生在节点被转移到同步队列期间。
     */
    while (!isOnSyncQueue(node))
        Thread.yield();
    }
    
    // 中断在节点被转移到同步队列期间或之后发生，返回 false
    return false;
}

/**
 * 根据中断类型做出相应的处理：
 * THROW_IE：抛出 InterruptedException 异常
 * REINTERRUPT：重新设置中断标志，向后传递中断
 */
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}

/** 中断线程 */   
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

这里有几个难理解的点：

1. Condition的队列和AQS的队列不完全一样；

   AQS的队列头节点是不存在任何值的，是一个虚节点；

   Condition的队列头节点是存储着实实在在的元素值的，是真实节点。

2. 各种等待状态（waitStatus）的变化；

   首先，在条件队列中，新建节点的初始等待状态是CONDITION（-2）；

   其次，移到AQS的队列中时等待状态会更改为0（AQS队列节点的初始等待状态为0）；

   然后，在AQS的队列中如果需要阻塞，会把它上一个节点的等待状态设置为SIGNAL（-1）；

   最后，不管在Condition队列还是AQS队列中，已取消的节点的等待状态都会设置为CANCELLED（1）；

   另外，后面我们在共享锁的时候还会讲到另外一种等待状态叫PROPAGATE（-3）。

3. 相似的名称；

   AQS中下一个节点是next，上一个节点是prev；

   Condition中下一个节点是nextWaiter，没有上一个节点。

如果弄明白了这几个点，看懂上面的代码还是轻松加愉快的，如果没弄明白，彤哥这里指出来了，希望您回头再看看上面的代码。

下面总结一下await()方法的大致流程：

1. 新建一个节点加入到条件队列中去；
2. 完全释放当前线程占有的锁；
3. 阻塞当前线程，并等待条件的出现；
4. 条件已出现（此时节点已经移到AQS的队列中），尝试获取锁；

也就是说await()方法内部其实是`先释放锁->等待条件->再次获取锁`的过程。

##### condition.signal()方法

condition.signal()方法通知条件已经出现。

```java
/** 将条件队列中的头结点转移到同步队列中 */
// AbstractQueuedSynchronizer.ConditionObject.signal
public final void signal() {
    // 如果不是当前线程占有着锁，调用这个方法抛出异常
    // 说明signal()也要在获取锁之后执行
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 条件队列的头节点
    Node first = firstWaiter;
    // 如果有等待条件的节点，则通知它条件已成立
    if (first != null)
        doSignal(first);
}
// AbstractQueuedSynchronizer.ConditionObject.doSignal
private void doSignal(Node first) {
    do {
        // 移到条件队列的头节点往后一位
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        // 相当于把头节点从队列中出队
        first.nextWaiter = null;
        // 转移节点到AQS队列中
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
public final void signalAll() {
    // 检查互斥锁持有情况。
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}

private void doSignalAll(Node first) {
    // 将firstWaiter和lastWaiter先清为null。
    lastWaiter = firstWaiter = null;
    // 从first开始一直遍历到第一个null节点。
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null);
}
/** 这个方法用于将条件队列中的节点转移到同步队列中 */
// AbstractQueuedSynchronizer.transferForSignal
final boolean transferForSignal(Node node) {
    // 把节点的状态更改为0，也就是说即将移到AQS队列中
    // 如果失败了，说明节点已经被改成取消状态了
    // 返回false，通过上面的循环可知会寻找下一个可用节点
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    // 调用AQS的入队方法把节点移到AQS的队列中
    // 注意，这里enq()的返回值是node的上一个节点，也就是旧尾节点
    Node p = enq(node);
    // 上一个节点的等待状态
    int ws = p.waitStatus;
    // 如果上一个节点已取消了，或者更新状态为SIGNAL失败（也是说明上一个节点已经取消了）
    // 则直接唤醒当前节点对应的线程
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    // 如果更新上一个节点的等待状态为SIGNAL成功了
    // 则返回true，这时上面的循环不成立了，退出循环，也就是只通知了一个节点
    // 此时当前节点还是阻塞状态
    // 也就是说调用signal()的时候并不会真正唤醒一个节点
    // 只是把节点从条件队列移到AQS队列中
    return true;
}
```

signal()方法的大致流程为：

1. 从条件队列的头节点开始寻找一个非取消状态的节点；
2. 把它从条件队列移到AQS队列；
3. 且只移动一个节点；

注意，这里调用signal()方法后并不会真正唤醒一个节点，那么，唤醒一个节点是在啥时候呢？

还记得开头例子吗？倒回去再好好看看，signal()方法后，最终会执行lock.unlock()方法，此时才会真正唤醒一个节点，唤醒的这个节点如果曾经是条件节点的话又会继续执行await()方法“分界线”下面的代码。

如果非要用一个图来表示的话，我想下面这个图可以大致表示一下（这里是用时序图画的，但是实际并不能算作一个真正的时序图哈，了解就好）：

![1kYftx.png](https://s2.ax1x.com/2020/01/21/1kYftx.png)

##### 总结

1. 重入锁是指可重复获取的锁，即一个线程获取锁之后再尝试获取锁时会自动获取锁；
2. 在ReentrantLock中重入锁是通过不断累加state变量的值实现的；
3. ReentrantLock的释放要跟获取匹配，即获取了几次也要释放几次；
4. ReentrantLock默认是非公平模式，因为非公平模式效率更高；
5. 条件锁是指为了等待某个条件出现而使用的一种锁；
6. 条件锁比较经典的使用场景就是队列为空时阻塞在条件notEmpty上；
7. ReentrantLock中的条件锁是通过AQS的ConditionObject内部类实现的；
8. await()和signal()方法都必须在获取锁之后释放锁之前使用；
9. await()方法会新建一个节点放到条件队列中，接着完全释放锁，然后阻塞当前线程并等待条件的出现
10. signal()方法会寻找条件队列中第一个可用节点移到AQS队列中；
11. 在调用signal()方法的线程调用unlock()方法才真正唤醒阻塞在条件上的节点（此时节点已经在AQS队列中）
12. 之后该节点会再次尝试获取锁，后面的逻辑与lock()的逻辑基本一致了。

### LockSupport

线程间等待/通知机制使用的Condition时都会调用LockSupport.park()方法和LockSupport.unpark()方法。而这个在同步组件的实现中被频繁使用的LockSupport到底是何方神圣，现在就来看看。LockSupport位于java.util.concurrent.locks包下，有兴趣的可以直接去看源码，该类的方法并不是很多。

LockSupprot是线程的阻塞原语，用来阻塞线程和唤醒线程。每个使用LockSupport的线程都会与一个许可关联，如果该许可可用，并且可在线程中使用，则调用park()将会立即返回，否则可能阻塞。如果许可尚不可用，则可以调用 unpark 使其可用。但是注意许可不可重入，也就是说只能调用一次park()方法，否则会一直阻塞。

#### LockSupport方法介绍

**阻塞线程**

```
void park()：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回
void park(Object blocker)：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
void parkNanos(long nanos)：阻塞当前线程，最长不超过nanos纳秒，增加了超时返回的特性；
void parkNanos(Object blocker, long nanos)：功能同方法3，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
void parkUntil(long deadline)：阻塞当前线程，知道deadline；
void parkUntil(Object blocker, long deadline)：功能同方法5，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象，方便进行问题排查；
```

**唤醒线程**

```
void unpark(Thread thread):唤醒处于阻塞状态的指定线程
```
实际上LockSupport阻塞和唤醒线程的功能是依赖于sun.misc.Unsafe，这是一个很底层的类，有兴趣的可以去查阅资料，比如park()方法的功能实现则是靠unsafe.park()方法。

另外在阻塞线程这一系列方法中还有一个很有意思的现象就是，每个方法都会新增一个带有Object的阻塞对象的重载方法

synchronzed致使线程阻塞，线程会进入到BLOCKED状态，而调用LockSupprt方法阻塞线程会致使线程进入到WAITING状态。


#### 示例

```java
public class LockSupportDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "被唤醒");
        });
        thread.start();
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LockSupport.unpark(thread);
    }
}
```
thread线程调用LockSupport.park()致使thread阻塞，当mian线程睡眠3秒结束后通过LockSupport.unpark(thread)方法唤醒thread线程,thread线程被唤醒执行后续操作。另外，还有一点值得关注的是，**LockSupport.unpark(thread)可以指定线程对象唤醒指定的线程。**


### 参考

1. [Java多线程之ReentrantLock与Condition](https://www.cnblogs.com/xiaoxi/p/7651360.html)
2. [详解Condition的await和signal等待/通知机制](https://www.jianshu.com/p/28387056eeb4)
3. [LockSupport工具](https://www.jianshu.com/p/9677a754cf60)
