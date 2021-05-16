---
title: Java AQS原理
date: 2019-05-10 23:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
在针对并发编程中，Doug Lea大师为我们提供了大量实用，高性能的工具类，针对这些代码进行研究会让我们队并发编程的掌握更加透彻也会大大提升我们队并发编程技术的热爱。这些代码在java.util.concurrent包下。

<!--more-->

其中包含了两个子包：atomic以及lock，另外在concurrent下的阻塞队列以及executors,这些就是concurrent包中的精华，之后会一一进行学习。而这些类的实现主要是依赖于volatile以及CAS，从整体上来看concurrent包的整体实现图如下图所示：

![1.png](https://i.loli.net/2019/05/06/5cd00bcebb607.png)

### 抽象队列化同步器AbstractQueuedSynchroizer

尽管JVM在并发上已经做了很多优化工作，如偏向锁、轻量级锁、自旋锁等等。但是基于`Synchronized` `wait` `notify`实现的同步机制还是无法满足日常开发中。原生同步机制在时间和空间上的开销也一直备受诟病。为了提升Java程序在并发场景下的性能、扩展性和健壮性，java.util.concurrent的使用必不可少。java.util.concurrent 包含许多线程安全、测试良好、高性能的并发构建块。通过使用java.util.concurrent,开发人员可以提高并发类的线程安全、可伸缩性、性能、可读性和可靠性。

`java.util.concurrent`的功能很强大，要想完整了解其全部细节也是很不容易的，需要多年的学习和实践经验。不过，通过深入其核心部分，可以快速了解其骨架和底层实现机制。那么谁才是`java.util.concurrent`的核心组件呢？稍微看过一点`java.util.concurrent`源码的同学知道，concurrent包下很多组件如：`ReentrantLock` `Semaphore` `CountDownLatch`在其内部都有一个sync类，而这个`sync`有继承自`java.util.concurrent.locks.AbstractQueuedSynchronizer`,而这个`AbstractQueuedSynchronizer`就是concurrent包的核心。尽管`AbstractQueuedSynchronizer`只是一个类，但其实质上却提供了一个框架，通过提供基于FIFO的队列管理机制、线程阻塞机制和状态同步机制，用户可以快速基于`AbstractQueuedSynchonizer`完成一系列复杂的进程同步操作。如果第一次接触到`AbstractQueuedSynchronizer`,建议读一下其作者的论文：[The java.util.concurrent Synchronizer Framework](https://link.jianshu.com?t=http://gee.cs.oswego.edu/dl/papers/aqs.pdf)。

`AbstractQueuedSynchronizer`（以下简称`AQS`）从字面理解是一个抽象的基于队列的同步器，所以`AQS`至少要完成以下几部分工作：

- 同步状态的原子性管理
- 等待线程队列的维护
- 线程的阻塞和唤醒
- 仅定义核心操作，留出足够的扩展性给子类

`AQS`定义了两个核心操作:`acquire` `release`及其变种。前者用于进入同步块前获取同步块执行权，后者用于释放对于同步块的占有权。

`acquire`核心逻辑如下：

```
// 循环里不断尝试，典型的失败后重试
while (synchronization state does not allow acquire) {
     // 同步状态不允许获取，进入循环体，也就是失败后的处理
     enqueue current thread if not already queued;     // 如果当前线程不在等待队列里，则加入等待队列
     possibly block current thread;     // 可能的话，阻塞当前线程
}

// 执行到这里，说明已经成功获取，如果之前有加入队列，则出队列。
dequeue current thread if it was queued; 
```

`release`核心逻辑如下：

```
update synchronization state;    //  更新同步状态
if (state may permit a blocked thread to acquire) // 检查状态是否允许一个阻塞线程获取
      unblock one or more queued threads;     // 允许，则唤醒后继的一个或多个阻塞线程。
```

而要实现上述两个核心接口，就必须实现前文提到的`AQS`主要工作的前三项：

- 同步状态的原子性管理
- 阻塞线程队列的维护
- 线程的阻塞和唤醒

实际使用中，`AQS`提供了以下5个模板方法：

```
tryAcquire(int)      // 试图在独占模式下获取对象状态。此方法应该查询是否允许它在独占模式下获取对象状态，如果允许，则获取它。
tryRelease(int)       // 试图设置状态来反映独占模式下的一个释放。
tryAcquireShared(int)       // 试图在共享模式下获取对象状态。此方法应该查询是否允许它在共享模式下获取对象状态，如果允许，则获取它。
tryReleaseShared(int)       // 试图设置状态来反映共享模式下的一个释放。
isHeldExclusively()      // 如果对于当前（正调用的）线程，同步是以独占方式进行的，则返回 true。
//此方法是在每次调用非等待 AbstractQueuedSynchronizer.ConditionObject 方法时调用的。
//（等待方法则调用 release(int)。）

```

#### 同步状态的原子性管理

`AQS`内部维护一个32bit字段`state`用于描述当前状态，`state`字段有`volatile`修饰，保证了其可见性。同时`AQS`还提供了`getState`,`setState`, `compareAndSetState`等方法用于状态的读取和更新：

- getState：提供一个基于内存语义（memory semantics）的volatile变量（state）读取
- setState：提供一个基于内存予以（memory semantics）的volatile变量（state）更新
- compareAndSetState：提供一个基于CAS（compare and swap）的原子性状态更新操作

通过简单的原子读写就可以达到内存可视性，减少了同步的需求。子类可以获取和设置状态的值，通过定义状态的值来表示 AQS 对象是否被获取或被释放。

#### 线程的阻塞与唤醒

`AQS`基于`java.util.concurrent.locks.LockSupport` 支持创建锁和其他同步类需要的基本线程阻塞、解除阻塞原语。

这个类最主要的功能有两个：

- park：把线程阻塞
- unpark：让线程恢复执行

其实除了`LockSupport`，Java之初就有`Object`对象的wait和notify方法可以实现线程的阻塞和唤醒。那么它们的区别是什么呢？

主要的区别应该说是它们面向的对象不同。阻塞和唤醒是对于线程来说的，LockSupport的park/unpark更符合这个语义，以“线程”作为方法的参数， 语义更清晰，使用起来也更方便。而wait/notify的实现使得“线程”的阻塞/唤醒对线程本身来说是被动的，要准确的控制哪个线程、什么时候阻塞/唤醒很困难， 要不随机唤醒一个线程（notify）要不唤醒所有的（notifyAll）。

`LockSupport`并不需要获取对象的监视器。LockSupport机制是每次`unpark`给线程1个“许可”——最多只能是1，而`park`则相反，如果当前 线程有许可，那么park方法会消耗1个并返回，否则会阻塞线程直到线程重新获得许可，在线程启动之前调用`park/unpark`方法没有任何效果。

```
// 1次unpark给线程1个许可
LockSupport.unpark(Thread.currentThread());
// 如果线程非阻塞重复调用没有任何效果
LockSupport.unpark(Thread.currentThread());
// 消耗1个许可
LockSupport.park(Thread.currentThread());
// 阻塞
LockSupport.park(Thread.currentThread());
```

因为它们本身的实现机制不一样，所以它们之间没有交集，也就是说LockSupport阻塞的线程，notify/notifyAll没法唤醒。

#### 队列维护

队列管理是`AQS`的核心部分，作者采用了基于`CLH锁队列`来实现内部队列。CLH锁（可参考：[CLH锁](https://link.jianshu.com?t=http://coderbee.net/index.php/java/20131115/577)）通常用于自旋锁，我们反而用于阻塞同步器，但使用相同的基本策略：在（线程）它自己结点持有关于线程的一些控制信息。每个结点的 “status” 字段跟踪一个线程是否应该阻塞。一个结点在它的前驱释放时被通知。队列的每个结点作为一个特定通知风格（specific-notification-style）的监视器服务，持有单一等待线程。

”status” 字段不控制线程是否授予。一个线程可能尝试去获取如果它是第一个进入队列，但成为第一个不保证就成功；它只是获得权利去竞争，所以当前释放的竞争者线程可能需要再次等待（注：这是公平性的问题，子类的实现可以进行控制）。

为了进入CLH锁队列，你只需要原子地把它作为一个新的尾结点拼接；为了出队列，你只需要设置 “head” 字段。

#### 总结

作为一个维护内部竞争队列的同步器，`AQS`需要完成三部分工作：

- 共享状态的原子性维护
- 线程的阻塞与唤醒
- 竞争队列的维护

### 核心源码

#### 重要方法介绍

本节将介绍三组重要的方法，通过使用这三组方法即可实现一个同步组件。

第一组方法是用于访问/设置同步状态的，如下：

| 方法                                               | 说明                  |
| :------------------------------------------------- | :-------------------- |
| int getState()                                     | 获取同步状态          |
| void setState()                                    | 设置同步状态          |
| boolean compareAndSetState(int expect, int update) | 通过 CAS 设置同步状态 |

第二组方需要由同步组件覆写。如下：

| 方法                              | 说明                       |
| :-------------------------------- | :------------------------- |
| boolean tryAcquire(int arg)       | 独占式获取同步状态         |
| boolean tryRelease(int arg)       | 独占式释放同步状态         |
| int tryAcquireShared(int arg)     | 共享式获取同步状态         |
| boolean tryReleaseShared(int arg) | 共享式私房同步状态         |
| boolean isHeldExclusively()       | 检测当前线程是否获取独占锁 |

第三组方法是一组模板方法，同步组件可直接调用。如下：

| 方法                                              | 说明                                                         |
| :------------------------------------------------ | :----------------------------------------------------------- |
| void acquire(int arg)                             | 独占式获取同步状态，该方法将会调用 tryAcquire 尝试获取同步状态。获取成功则返回，获取失败，线程进入同步队列等待。 |
| void acquireInterruptibly(int arg)                | 响应中断版的 acquire                                         |
| boolean tryAcquireNanos(int arg,long nanos)       | 超时+响应中断版的 acquire                                    |
| void acquireShared(int arg)                       | 共享式获取同步状态，同一时刻可能会有多个线程获得同步状态。比如读写锁的读锁就是就是调用这个方法获取同步状态的。 |
| void acquireSharedInterruptibly(int arg)          | 响应中断版的 acquireShared                                   |
| boolean tryAcquireSharedNanos(int arg,long nanos) | 超时+响应中断版的 acquireShared                              |
| boolean release(int arg)                          | 独占式释放同步状态                                           |
| boolean releaseShared(int arg)                    | 共享式释放同步状态                                           |

上面列举了一堆方法，看似繁杂。但稍微理一下，就会发现上面诸多方法无非就两大类：一类是独占式获取和释放共享状态，另一类是共享式获取和释放同步状态。至于这两类方法的实现细节，我会在接下来的章节中讲到，继续往下看吧。

#### 主要内部类

在并发的情况下，AQS 会将未获取同步状态的线程将会封装成节点，并将其放入同步队列尾部。同步队列中的节点除了要保存线程，还要保存等待状态。不管是独占式还是共享式，在获取状态失败时都会用到节点类。所以这里我们要先看一下节点类的实现，为后面的源码分析进行简单铺垫。源码如下：

```java
static final class Node {

    /** 共享类型节点，标记节点在共享模式下等待 */
    static final Node SHARED = new Node();
    
    /** 独占类型节点，标记节点在独占模式下等待 */
    static final Node EXCLUSIVE = null;

    /** 等待状态 - 取消 */
    static final int CANCELLED =  1;
    
    /** 
     * 等待状态 - 通知。某个节点是处于该状态，当该节点释放同步状态后，
     * 会通知后继节点线程，使之可以恢复运行 
     */
    static final int SIGNAL    = -1;
    
    /** 等待状态 - 条件等待。表明节点等待在 Condition 上 */
    static final int CONDITION = -2;
    
    /**
     * 等待状态 - 传播。表示无条件向后传播唤醒动作，详细分析请看第五章
     */
    static final int PROPAGATE = -3;

    /**
     * 等待状态，取值如下：
     *   SIGNAL,
     *   CANCELLED,
     *   CONDITION,
     *   PROPAGATE,
     *   0
     * 
     * 初始情况下，waitStatus = 0
     */
    volatile int waitStatus;

    /**
     * 前驱节点
     */
    volatile Node prev;

    /**
     * 后继节点
     */
    volatile Node next;

    /**
     * 对应的线程
     */
    volatile Thread thread;

    /**
     * 下一个等待节点，用在 ConditionObject 中
     */
    Node nextWaiter;

    /**
     * 判断节点是否是共享节点
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
     * 获取前驱节点
     */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    /** addWaiter 方法会调用该构造方法 */
    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    /** Condition 中会用到此构造方法 */
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

典型的双链表结构，节点中保存着当前线程、前一个节点、后一个节点以及线程的状态等信息。

#### 主要属性

```java
// 队列的头节点
private transient volatile Node head;
// 队列的尾节点
private transient volatile Node tail;
// 控制加锁解锁的状态变量
private volatile int state;
```

定义了一个状态变量和一个队列，状态变量用来控制加锁解锁，队列用来放置等待的线程。

注意，这几个变量都要使用volatile关键字来修饰，因为是在多线程环境下操作，要保证它们的值修改之后对其它线程立即可见。

这几个变量的修改是直接使用的Unsafe这个类来操作的：

```java
// 获取Unsafe类的实例，注意这种方式仅限于jdk自己使用，普通用户是无法这样调用的
private static final Unsafe unsafe = Unsafe.getUnsafe();
// 状态变量state的偏移量
private static final long stateOffset;
// 头节点的偏移量
private static final long headOffset;
// 尾节点的偏移量
private static final long tailOffset;
// 等待状态的偏移量（Node的属性）
private static final long waitStatusOffset;
// 下一个节点的偏移量（Node的属性）
private static final long nextOffset;

static {
    try {
        // 获取state的偏移量
        stateOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
        // 获取head的偏移量
        headOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
        // 获取tail的偏移量
        tailOffset = unsafe.objectFieldOffset
            (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
        // 获取waitStatus的偏移量
        waitStatusOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("waitStatus"));
        // 获取next的偏移量
        nextOffset = unsafe.objectFieldOffset
            (Node.class.getDeclaredField("next"));

    } catch (Exception ex) { throw new Error(ex); }
}

// 调用Unsafe的方法原子更新state
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

#### 子类需要实现的主要方法

我们可以看到AQS的全称是AbstractQueuedSynchronizer，它本质上是一个抽象类，说明它本质上应该是需要子类来实现的，那么子类实现一个同步器需要实现哪些方法呢？

```java
// 互斥模式下使用：尝试获取锁
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
// 互斥模式下使用：尝试释放锁
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
// 共享模式下使用：尝试获取锁
protected int tryAcquireShared(int arg) {
    throw new UnsupportedOperationException();
}
// 共享模式下使用：尝试释放锁
protected boolean tryReleaseShared(int arg) {
    throw new UnsupportedOperationException();
}
// 如果当前线程独占着锁，返回true
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

问题：这几个方法为什么不直接定义成抽象方法呢？

因为子类只要实现这几个方法中的一部分就可以实现一个同步器了，所以不需要定义成抽象方法。

下面我们通过一个案例来介绍AQS中的部分方法。

#### 基于AQS自己动手写一个锁

直接上代码：

```java
public class MyLockBaseOnAqs {

    // 定义一个同步器，实现AQS类
    private static class Sync extends AbstractQueuedSynchronizer {
        // 实现tryAcquire(acquires)方法
        @Override
        public boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
        // 实现tryRelease(releases)方法
        @Override
        protected boolean tryRelease(int releases) {
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }
    }

    // 声明同步器
    private final Sync sync = new Sync();

    // 加锁
    public void lock() {
        sync.acquire(1);
    }

    // 解锁
    public void unlock() {
        sync.release(1);
    }


    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        MyLockBaseOnAqs lock = new MyLockBaseOnAqs();

        CountDownLatch countDownLatch = new CountDownLatch(1000);

        IntStream.range(0, 1000).forEach(i -> new Thread(() -> {
            lock.lock();

            try {
                IntStream.range(0, 10000).forEach(j -> {
                    count++;
                });
            } finally {
                lock.unlock();
            }
//            System.out.println(Thread.currentThread().getName());
            countDownLatch.countDown();
        }, "tt-" + i).start());

        countDownLatch.await();

        System.out.println(count);
    }
}
```

运行main()方法总是打印出10000000（一千万），说明这个锁也是可以直接使用的，当然这也是一个不可重入的锁。

#### 独占模式分析

##### 获取同步状态

独占式获取同步状态时通过 acquire 进行的，下面来分析一下该方法的源码。如下：

```java
/**
 * 该方法将会调用子类复写的 tryAcquire 方法获取同步状态，
 * - 获取成功：直接返回
 * - 获取失败：将线程封装在节点中，并将节点置于同步队列尾部，
 *     通过自旋尝试获取同步状态。如果在有限次内仍无法获取同步状态，
 *     该线程将会被 LockSupport.park 方法阻塞住，直到被前驱节点唤醒
 */
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

/** 向同步队列尾部添加一个节点 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // 尝试以快速方式将节点添加到队列尾部
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    
    // 快速插入节点失败，调用 enq 方法，不停的尝试插入节点
    enq(node);
    return node;
}

/**
 * 通过 CAS + 自旋的方式插入节点到队尾
 */
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            // 设置头结点，初始情况下，头结点是一个空节点
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            /*
             * 将节点插入队列尾部。这里是先将新节点的前驱设为尾节点，之后在尝试将新节点设为尾节
             * 点，最后再将原尾节点的后继节点指向新的尾节点。除了这种方式，我们还先设置尾节点，
             * 之后再设置前驱和后继，即：
             * 
             *    if (compareAndSetTail(t, node)) {
             *        node.prev = t;
             *        t.next = node;
             *    }
             *    
             * 但但如果是这样做，会导致一个问题，即短时内，队列结构会遭到破坏。考虑这种情况，
             * 某个线程在调用 compareAndSetTail(t, node)成功后，该线程被 CPU 切换了。此时
             * 设置前驱和后继的代码还没带的及执行，但尾节点指针却设置成功，导致队列结构短时内会
             * 出现如下情况：
             *
             *      +------+  prev +-----+       +-----+
             * head |      | <---- |     |       |     |  tail
             *      |      | ----> |     |       |     |
             *      +------+ next  +-----+       +-----+
             *
             * tail 节点完全脱离了队列，这样导致一些队列遍历代码出错。如果先设置
             * 前驱，在设置尾节点。及时线程被切换，队列结构短时可能如下：
             *
             *      +------+  prev +-----+ prev  +-----+
             * head |      | <---- |     | <---- |     |  tail
             *      |      | ----> |     |       |     |
             *      +------+ next  +-----+       +-----+
             *      
             * 这样并不会影响从后向前遍历，不会导致遍历逻辑出错。
             * 
             * 参考：
             *    https://www.cnblogs.com/micrari/p/6937995.html
             */
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

/**
 * 同步队列中的线程在此方法中以循环尝试获取同步状态，在有限次的尝试后，
 * 若仍未获取锁，线程将会被阻塞，直至被前驱节点的线程唤醒。
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 循环获取同步状态
        for (;;) {
            final Node p = node.predecessor();
            /*
             * 前驱节点如果是头结点，表明前驱节点已经获取了同步状态。前驱节点释放同步状态后，
             * 在不出异常的情况下， tryAcquire(arg) 应返回 true。此时节点就成功获取了同
             * 步状态，并将自己设为头节点，原头节点出队。
             */ 
            if (p == head && tryAcquire(arg)) {
                // 成功获取同步状态，设置自己为头节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            
            /*
             * 如果获取同步状态失败，则根据条件判断是否应该阻塞自己。
             * 如果不阻塞，CPU 就会处于忙等状态，这样会浪费 CPU 资源
             */
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        /*
         * 如果在获取同步状态中出现异常，failed = true，cancelAcquire 方法会被执行。
         * tryAcquire 需同步组件开发者覆写，难免不了会出现异常。
         */
        if (failed)
            cancelAcquire(node);
    }
}

/** 设置头节点 */
private void setHead(Node node) {
    // 仅有一个线程可以成功获取同步状态，所以这里不需要进行同步控制
    head = node;
    node.thread = null;
    node.prev = null;
}

/**
 * 该方法主要用途是，当线程在获取同步状态失败时，根据前驱节点的等待状态，决定后续的动作。比如前驱
 * 节点等待状态为 SIGNAL，表明当前节点线程应该被阻塞住了。不能老是尝试，避免 CPU 忙等。
 *    —————————————————————————————————————————————————————————————————
 *    | 前驱节点等待状态 |                   相应动作                     |
 *    —————————————————————————————————————————————————————————————————
 *    | SIGNAL         | 阻塞                                          |
 *    | CANCELLED      | 向前遍历, 移除前面所有为该状态的节点               |
 *    | waitStatus < 0 | 将前驱节点状态设为 SIGNAL, 并再次尝试获取同步状态   |
 *    —————————————————————————————————————————————————————————————————
 */
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    /* 
     * 前驱节点等待状态为 SIGNAL，表示当前线程应该被阻塞。
     * 线程阻塞后，会在前驱节点释放同步状态后被前驱节点线程唤醒
     */
    if (ws == Node.SIGNAL)
        return true;
        
    /*
     * 前驱节点等待状态为 CANCELLED，则以前驱节点为起点向前遍历，
     * 移除其他等待状态为 CANCELLED 的节点。
     */ 
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
         * 等待状态为 0 或 PROPAGATE，设置前驱节点等待状态为 SIGNAL，
         * 并再次尝试获取同步状态。
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}

private final boolean parkAndCheckInterrupt() {
    // 调用 LockSupport.park 阻塞自己
    LockSupport.park(this);
    return Thread.interrupted();
}

/**
 * 取消获取同步状态
 */
private void cancelAcquire(Node node) {
    if (node == null)
        return;

    node.thread = null;

    // 前驱节点等待状态为 CANCELLED，则向前遍历并移除其他为该状态的节点
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;

    // 记录 pred 的后继节点，后面会用到
    Node predNext = pred.next;

    // 将当前节点等待状态设为 CANCELLED
    node.waitStatus = Node.CANCELLED;

    /*
     * 如果当前节点是尾节点，则通过 CAS 设置前驱节点 prev 为尾节点。设置成功后，再利用 CAS 将 
     * prev 的 next 引用置空，断开与后继节点的联系，完成清理工作。
     */ 
    if (node == tail && compareAndSetTail(node, pred)) {
        /* 
         * 执行到这里，表明 pred 节点被成功设为了尾节点，这里通过 CAS 将 pred 节点的后继节点
         * 设为 null。注意这里的 CAS 即使失败了，也没关系。失败了，表明 pred 的后继节点更新
         * 了。pred 此时已经是尾节点了，若后继节点被更新，则是有新节点入队了。这种情况下，CAS 
         * 会失败，但失败不会影响同步队列的结构。
         */
        compareAndSetNext(pred, predNext, null);
    } else {
        int ws;
        // 根据条件判断是唤醒后继节点，还是将前驱节点和后继节点连接到一起
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                /*
                 * 这里使用 CAS 设置 pred 的 next，表明多个线程同时在取消，这里存在竞争。
                 * 不过此处没针对 compareAndSetNext 方法失败后做一些处理，表明即使失败了也
                 * 没关系。实际上，多个线程同时设置 pred 的 next 引用时，只要有一个能设置成
                 * 功即可。
                 */
                compareAndSetNext(pred, predNext, next);
        } else {
            /*
             * 唤醒后继节点对应的线程。这里简单讲一下为什么要唤醒后继线程，考虑下面一种情况：
             *        head          node1         node2         tail
             *        ws=0          ws=1          ws=-1         ws=0
             *      +------+  prev +-----+  prev +-----+  prev +-----+
             *      |      | <---- |     | <---- |     | <---- |     |  
             *      |      | ----> |     | ----> |     | ----> |     |
             *      +------+  next +-----+  next +-----+  next +-----+
             *      
             * 头结点初始状态为 0，node1、node2 和 tail 节点依次入队。node1 自旋过程中调用 
             * tryAcquire 出现异常，进入 cancelAcquire。head 节点此时等待状态仍然是 0，它
             * 会认为后继节点还在运行中，所它在释放同步状态后，不会去唤醒后继等待状态为非取消的
             * 节点 node2。如果 node1 再不唤醒 node2 的线程，该线程面临无法被唤醒的情况。此
             * 时，整个同步队列就回全部阻塞住。
             */
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}

private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    /*
     * 通过 CAS 将等待状态设为 0，让后继节点线程多一次
     * 尝试获取同步状态的机会
     */
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
       /*
        * 这里如果 s == null 处理，是不是表明 node 是尾节点？答案是不一定。原因之前在分析 
        * enq 方法时说过。这里再啰嗦一遍，新节点入队时，队列瞬时结构可能如下：
        *                      node1         node2
        *      +------+  prev +-----+ prev  +-----+
        * head |      | <---- |     | <---- |     |  tail
        *      |      | ----> |     |       |     |
        *      +------+ next  +-----+       +-----+
        * 
        * node2 节点为新入队节点，此时 tail 已经指向了它，但 node1 后继引用还未设置。
        * 这里 node1 就是 node 参数，s = node1.next = null，但此时 node1 并不是尾
        * 节点。所以这里不能从前向后遍历同步队列，应该从后向前。
        */
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        // 唤醒 node 的后继节点线程
        LockSupport.unpark(s.thread);
}
```

到这里，独占式获取同步状态的分析就讲完了。如果仅分析获取同步状态的大致流程，那么这个流程并不难。但若深入到细节之中，还是需要思考思考。这里对独占式获取同步状态的大致流程做个总结，如下：

1. 调用 tryAcquire 方法尝试获取同步状态
2. 获取成功，直接返回
3. 获取失败，将线程封装到节点中，并将节点入队
4. 入队节点在 acquireQueued 方法中自旋获取同步状态
5. 若节点的前驱节点是头节点，则再次调用 tryAcquire 尝试获取同步状态
6. 获取成功，当前节点将自己设为头节点并返回
7. 获取失败，可能再次尝试，也可能会被阻塞。这里简单认为会被阻塞。

上面的步骤对应下面的流程图：



##### 释放同步状态

相对于获取同步状态，释放同步状态的过程则要简单的多，这里简单罗列一下步骤：

1. 调用 tryRelease(arg) 尝试释放同步状态
2. 根据条件判断是否应该唤醒后继线程

就两个步骤，下面看一下源码分析。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        /*
         * 这里简单列举条件分支的可能性，如下：
         * 1. head = null
         *     head 还未初始化。初始情况下，head = null，当第一个节点入队后，head 会被初始
         *     为一个虚拟（dummy）节点。这里，如果还没节点入队就调用 release 释放同步状态，
         *     就会出现 h = null 的情况。
         *     
         * 2. head != null && waitStatus = 0
         *     表明后继节点对应的线程仍在运行中，不需要唤醒
         * 
         * 3. head != null && waitStatus < 0
         *     后继节点对应的线程可能被阻塞了，需要唤醒 
         */
        if (h != null && h.waitStatus != 0)
            // 唤醒后继节点，上面分析过了，这里不再赘述
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

#### 共享模式分析

与独占模式不同，共享模式下，同一时刻会有多个线程获取共享同步状态。共享模式是实现读写锁中的读锁、CountDownLatch 和 Semaphore 等同步组件的基础，搞懂了，再去理解一些共享同步组件就不难了。

##### 获取同步状态

共享类型的节点获取共享同步状态后，如果后继节点也是共享类型节点，当前节点则会唤醒后继节点。这样，多个节点线程即可同时获取共享同步状态。

```java
public final void acquireShared(int arg) {
    // 尝试获取共享同步状态，tryAcquireShared 返回的是整型
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 这里和前面一样，也是通过有限次自旋的方式获取同步状态
        for (;;) {
            final Node p = node.predecessor();
            /*
             * 前驱是头结点，其类型可能是 EXCLUSIVE，也可能是 SHARED.
             * 如果是 EXCLUSIVE，线程无法获取共享同步状态。
             * 如果是 SHARED，线程则可获取共享同步状态。
             * 能不能获取共享同步状态要看 tryAcquireShared 具体的实现。比如多个线程竞争读写
             * 锁的中的读锁时，均能成功获取读锁。但多个线程同时竞争信号量时，可能就会有一部分线
             * 程因无法竞争到信号量资源而阻塞。
             */ 
            if (p == head) {
                // 尝试获取共享同步状态
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 设置头结点，如果后继节点是共享类型，唤醒后继节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
   
/**
 * 这个方法做了两件事情：
 * 1. 设置自身为头结点
 * 2. 根据条件判断是否要唤醒后继节点
 */ 
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    // 设置头结点
    setHead(node);
    
    /*
     * 这个条件分支由 propagate > 0 和 h.waitStatus < 0 两部分组成。
     * h.waitStatus < 0 时，waitStatus = SIGNAL 或 PROPAGATE。这里仅依赖
     * 条件 propagate > 0 判断是否唤醒后继节点是不充分的，至于原因请参考第五章
     */
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        /*
         * 节点 s 如果是共享类型节点，则应该唤醒该节点
         * 至于 s == null 的情况前面分析过，这里不在赘述。
         */ 
        if (s == null || s.isShared())
            doReleaseShared();
    }
}

/**
 * 该方法用于在 acquires/releases 存在竞争的情况下，确保唤醒动作向后传播。
 */ 
private void doReleaseShared() {
    /*
     * 下面的循环在 head 节点存在后继节点的情况下，做了两件事情：
     * 1. 如果 head 节点等待状态为 SIGNAL，则将 head 节点状态设为 0，并唤醒后继节点
     * 2. 如果 head 节点等待状态为 0，则将 head 节点状态设为 PROPAGATE，保证唤醒能够正
     *    常传播下去。关于 PROPAGATE 状态的细节分析，后面会讲到。
     */
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            /* 
             * ws = 0 的情况下，这里要尝试将状态从 0 设为 PROPAGATE，保证唤醒向后
             * 传播。setHeadAndPropagate 在读到 h.waitStatus < 0 时，可以继续唤醒
             * 后面的节点。
             */
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

到这里，共享模式下获取同步状态的逻辑就分析完了，不过我这里只做了简单分析。相对于独占式获取同步状态，共享式的情况更为复杂。独占模式下，只有一个节点线程可以成功获取同步状态，也只有获取已同步状态节点线程才可以释放同步状态。但在共享模式下，多个共享节点线程可以同时获得同步状态，在一些线程获取同步状态的同时，可能还会有另外一些线程正在释放同步状态。所以，共享模式更为复杂。这里我的脑力跟不上了，没法面面俱到的分析，见谅。

最后说一下共享模式下获取同步状态的大致流程，如下：

1. 获取共享同步状态
2. 若获取失败，则生成节点，并入队
3. 如果前驱为头结点，再次尝试获取共享同步状态
4. 获取成功则将自己设为头结点，如果后继节点是共享类型的，则唤醒
5. 若失败，将节点状态设为 SIGNAL，再次尝试。若再次失败，线程进入等待状态

##### 释放共享状态

释放共享状态主要逻辑在 doReleaseShared 中，doReleaseShared 上节已经分析过，这里就不赘述了。共享节点线程在获取同步状态和释放同步状态时都会调用 doReleaseShared，所以 doReleaseShared 是多线程竞争集中的地方。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

#### PROPAGATE 状态存在的意义

##### 利用 PROPAGATE 传播唤醒动作

PROPAGATE 状态是用来传播唤醒动作的，那么它是在哪里进行传播的呢？答案是在`setHeadAndPropagate`方法中，这里再来看看 setHeadAndPropagate 方法的实现：

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    setHead(node);
    
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

大家注意看 setHeadAndPropagate 方法中那个长长的判断语句，其中有一个条件是`h.waitStatus < 0`，当 h.waitStatus = SIGNAL(-1) 或 PROPAGATE(-3) 是，这个条件就会成立。那么 PROPAGATE 状态是在何时被设置的呢？答案是在`doReleaseShared`方法中，如下：

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {...}
            
            // 如果 ws = 0，则将 h 状态设为 PROPAGATE
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        ...
    }
}
```

再回到 setHeadAndPropagate 的实现，该方法既然引入了`h.waitStatus < 0`这个条件，就意味着仅靠条件`propagate > 0`判断是否唤醒后继节点线程的机制是不充分的。至于为啥不充分，请继续往看下看。

##### 引入 PROPAGATE 所解决的问题

PROPAGATE 的引入是为了解决一个 BUG – [JDK-6801020](https://bugs.java.com/view_bug.do?bug_id=6801020)，复现这个 BUG 的代码如下：

```java
import java.util.concurrent.Semaphore;

public class TestSemaphore {

   private static Semaphore sem = new Semaphore(0);

   private static class Thread1 extends Thread {
       @Override
       public void run() {
           sem.acquireUninterruptibly();
       }
   }

   private static class Thread2 extends Thread {
       @Override
       public void run() {
           sem.release();
       }
   }

   public static void main(String[] args) throws InterruptedException {
       for (int i = 0; i < 10000000; i++) {
           Thread t1 = new Thread1();
           Thread t2 = new Thread1();
           Thread t3 = new Thread2();
           Thread t4 = new Thread2();
           t1.start();
           t2.start();
           t3.start();
           t4.start();
           t1.join();
           t2.join();
           t3.join();
           t4.join();
           System.out.println(i);
       }
   }
}
```

根据 BUG 的描述消息可知 JDK 6u11,6u17 两个版本受到影响。那么，接下来再来看看引起这个 BUG 的代码 – [JDK 6u17](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase6-419409.html) 中 setHeadAndPropagate 和 releaseShared 两个方法源码，如下：

```java
private void setHeadAndPropagate(Node node, int propagate) {
    setHead(node);
    if (propagate > 0 && node.waitStatus != 0) {
        /*
         * Don't bother fully figuring out successor.  If it
         * looks null, call unparkSuccessor anyway to be safe.
         */
        Node s = node.next;
        if (s == null || s.isShared())
            unparkSuccessor(node);
    }
}

// 和 release 方法的源码基本一样
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

下面来简单说明 TestSemaphore 这个类的逻辑。这个类持有一个数值为 0 的信号量对象，并创建了4个线程，线程 t1 和 t2 用于获取信号量，t3 和 t4 则是调用 release() 方法释放信号量。在一般情况下，TestSemaphore 这个类的代码都可以正常执行。但当有极端情况出现时，可能会导致同步队列挂掉。这里演绎一下这个极端情况，考虑某次循环时，队列结构如下：

![1Am7qK.png](https://s2.ax1x.com/2020/01/22/1Am7qK.png)

1. 时刻1：线程 t3 调用 unparkSuccessor 方法，head 节点状态由 SIGNAL(-1) 变为`0`，并唤醒线程 t1。此时信号量数值为1。
2. 时刻2：线程 t1 恢复运行，t1 调用 Semaphore.NonfairSync 的 tryAcquireShared，返回`0`。然后线程 t1 被切换，暂停运行。
3. 时刻3：线程 t4 调用 releaseShared 方法，因 head 的状态为`0`，所以 t4 不会调用 unparkSuccessor 方法。
4. 时刻4：线程 t1 恢复运行，t1 成功获取信号量，调用 setHeadAndPropagate。但因为 propagate = 0，线程 t1 无法调用 unparkSuccessor 唤醒线程 t2，t2 面临无线程唤醒的情况。因为 t2 无法退出等待状态，所以 t2.join 会阻塞主线程，导致程序挂住。

下面再来看一下修复 BUG 后的代码，根据 BUG 详情页显示，该 BUG 在 JDK 1.7 中被修复。这里找一个 JDK 7 较早版本（JDK 7u10）的代码看一下，如下：

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
    
    if (propagate > 0 || h == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();
    }
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

在按照上面的代码演绎一下逻辑，如下：

1. 时刻1：线程 t3 调用 unparkSuccessor 方法，head 节点状态由 SIGNAL(-1) 变为`0`，并唤醒线程t1。此时信号量数值为1。
2. 时刻2：线程 t1 恢复运行，t1 调用 Semaphore.NonfairSync 的 tryAcquireShared，返回`0`。然后线程 t1 被切换，暂停运行。
3. 时刻3：线程 t4 调用 releaseShared 方法，检测到`h.waitStatus = 0`，t4 将头节点等待状态由`0`设为`PROPAGATE(-3)`。
4. 时刻4：线程 t1 恢复运行，t1 成功获取信号量，调用 setHeadAndPropagate。因 propagate = 0，`propagate > 0` 条件不满足。而 `h.waitStatus = PROPAGATE(-3)`，所以条件`h.waitStatus < 0`成立。进而，线程 t1 可以唤醒线程 t2，完成唤醒动作的传播。

到这里关于状态 PROPAGATE 的内容就讲完了。最后，简单总结一下本章开头提的两个问题。

**问题一：PROPAGATE 状态用在哪里，以及怎样向后传播唤醒动作的？**
答：PROPAGATE 状态用在 setHeadAndPropagate。当头节点状态被设为 PROPAGATE 后，后继节点成为新的头结点后。若 `propagate > 0` 条件不成立，则根据条件`h.waitStatus < 0`成立与否，来决定是否唤醒后继节点，即向后传播唤醒动作。

**问题二：引入 PROPAGATE 状态是为了解决什么问题？**
答：引入 PROPAGATE 状态是为了解决**并发释放信号量**所导致部分请求信号量的线程无法被唤醒的问题。

### 参考

1. [AbstractQueuedSynchronizer源码解读 - 活在夢裡](https://www.cnblogs.com/micrari/p/6937995.html)
2. [《Java并发编程的艺术》 - 方腾飞 / 魏鹏 / 程晓明](https://book.douban.com/subject/26591326/)