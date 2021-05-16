---
title: Java Thread 安全--锁原理（二）
date: 2019-05-05 18:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### Lock
<!--more-->

#### 简介

我们下来看concurent包下的lock子包。锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源。在Lock接口出现之前，java程序主要是靠synchronized关键字实现锁功能的，而java SE5之后，并发包中增加了lock接口，它提供了与synchronized一样的锁功能。虽然它失去了像synchronize关键字隐式加锁解锁的便捷性，但是却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。通常使用显示使用lock的形式如下：

```
Lock lock = new ReentrantLock();
lock.lock();
try{
    .......
}finally{
    lock.unlock();
}
```
需要注意的是synchronized同步块执行完成或者遇到异常是锁会自动释放，而lock必须调用unlock()方法释放锁，因此在finally块中释放锁。

#### Lock接口API
我们现在就来看看lock接口定义了哪些方法：
```
void lock(); //获取锁
void lockInterruptibly() throws InterruptedException；//获取锁的过程能够响应中断
boolean tryLock();//非阻塞式响应中断能立即返回，获取锁放回true反之返回fasle
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//超时获取锁，在超时内或者未中断的情况下能够获取锁
Condition newCondition();//获取与lock绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回
```
那么在locks包下有哪些类实现了该接口了？先从最熟悉的ReentrantLock说起。

```
public class ReentrantLock implements Lock, java.io.Serializable
```

很显然ReentrantLock实现了lock接口，接下来我们来仔细研究一下它是怎样实现的。当你查看源码时你会惊讶的发现ReentrantLock并没有多少代码，另外有一个很明显的特点是：基本上所有的方法的实现实际上都是调用了其静态内存类Sync中的方法，而Sync类继承了AbstractQueuedSynchronizer（AQS）。可以看出要想理解ReentrantLock关键核心在于对队列同步器AbstractQueuedSynchronizer（简称同步器）的理解。

#### AQS

同步器是用来构建锁和其他同步组件的基础框架，它的实现主要依赖一个int成员变量来表示同步状态以及通过一个FIFO队列构成等待队列。它的子类必须重写AQS的几个protected修饰的用来改变同步状态的方法，其他方法主要是实现了排队和阻塞机制。状态的更新使用getState,setState以及compareAndSetState这三个方法。

子类被推荐定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干同步状态的获取和释放方法来供自定义同步组件的使用，同步器既支持独占式获取同步状态，也可以支持共享式获取同步状态，这样就可以方便的实现不同类型的同步组件。

同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者的关系：锁是面向使用者，它定义了使用者与锁交互的接口，隐藏了实现细节；同步器是面向锁的实现者，它简化了锁的实现方式，屏蔽了同步状态的管理，线程的排队，等待和唤醒等底层操作。锁和同步器很好的隔离了使用者和实现者所需关注的领域。

在同步组件的实现中，AQS是核心部分，同步组件的实现者通过使用AQS提供的模板方法实现同步组件语义，AQS则实现了对同步状态的管理，以及对阻塞线程进行排队，等待通知等等一些底层的实现处理。AQS的核心也包括了这些方面:同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取这些特性的实现，而这些实际上则是AQS提供出来的模板方法，归纳整理如下：

**独占式锁：**

```
void acquire(int arg)：独占式获取同步状态，如果获取失败则插入同步队列进行等待；
void acquireInterruptibly(int arg)：与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
boolean tryAcquireNanos(int arg, long nanosTimeout)：在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
boolean release(int arg)：释放同步状态，该方法会唤醒在同步队列中的下一个节点
```

**共享式锁：**
```
void acquireShared(int arg)：共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；
void acquireSharedInterruptibly(int arg)：在acquireShared方法基础上增加了能响应中断的功能；
boolean tryAcquireSharedNanos(int arg, long nanosTimeout)：在acquireSharedInterruptibly基础上增加了超时等待的功能；
boolean releaseShared(int arg)：共享式释放同步状态
```


#### AQS的模板方法设计模式

AQS的设计是使用模板方法设计模式，它将一些方法开放给子类进行重写，而同步器给同步组件所提供模板方法又会重新调用被子类所重写的方法。举个例子，AQS中需要重写的方法tryAcquire：
```
protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
}
```
ReentrantLock中NonfairSync（继承AQS）会重写该方法为：
```
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```
而AQS中的模板方法acquire():
```
public final void acquire(int arg) {
       if (!tryAcquire(arg) &&
           acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
           selfInterrupt();
}
```

会调用tryAcquire方法，而此时当继承AQS的NonfairSync调用模板方法acquire时就会调用已经被NonfairSync重写的tryAcquire方法。这就是使用AQS的方式，在弄懂这点后会lock的实现理解有很大的提升。可以归纳总结为这么几点：

- 同步组件（这里不仅仅值锁，还包括CountDownLatch等）的实现依赖于同步器AQS，在同步组件实现中，使用AQS的方式被推荐定义继承AQS的静态内存类；
- AQS采用模板方法进行设计，AQS的protected修饰的方法需要由继承AQS的子类进行重写实现，当调用AQS的子类的方法时就会调用被重写的方法；
- AQS负责同步状态的管理，线程的排队，等待和唤醒这些底层操作，而Lock等同步组件主要专注于实现同步语义；
- 在重写AQS的方式时，使用AQS提供的getState(),setState(),compareAndSetState()方法进行修改同步状态

AQS可重写的方法如下图（摘自《java并发编程的艺术》一书）：

![1.png](https://i.loli.net/2019/05/06/5cd0116e89651.png)

在实现同步组件时AQS提供的模板方法如下图：

![2.png](https://i.loli.net/2019/05/06/5cd0117e0ff7f.png)

AQS提供的模板方法可以分为3类：

1. 独占式获取与释放同步状态；
2. 共享式获取与释放同步状态；
3. 查询同步队列中等待线程情况；

同步组件通过AQS提供的模板方法实现自己的同步语义。

下面使用一个例子来进一步理解下AQS的使用。这个例子也是来源于AQS源码中的example。

```java
class Mutex implements Lock, java.io.Serializable {
    // Our internal helper class
    // 继承AQS的静态内存类
    // 重写方法
    private static class Sync extends AbstractQueuedSynchronizer {
        // Reports whether in locked state
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // Acquires the lock if state is zero
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // Otherwise unused
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // Releases the lock by setting state to zero
        protected boolean tryRelease(int releases) {
            assert releases == 1; // Otherwise unused
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // Provides a Condition
        Condition newCondition() {
            return new ConditionObject();
        }

        // Deserializes properly
        private void readObject(ObjectInputStream s)
                throws IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    // The sync object does all the hard work. We just forward to it.
    private final Sync sync = new Sync();
    //使用同步器的模板方法实现自己的同步语义
    public void lock() {
        sync.acquire(1);
    }

    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
}
```

MutexDemo：
```java
public class MutextDemo {
    private static Mutex mutex = new Mutex();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {
                mutex.lock();
                Thread t = Thread.currentThread();
                System.out.println(t.getId()+"获取锁");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println(t.getId()+"释放锁");
                    mutex.unlock();
                }
            });
            thread.start();
        }
    }
}
```
上面的这个例子实现了不可重入的独占锁的语义，在同一个时刻只允许一个线程占有锁。MutexDemo新建了10个线程，分别睡眠3s。从执行情况也可以看出来当前Thread-6正在执行占有锁而其他Thread-7,Thread-8等线程处于WAIT状态。按照推荐的方式，Mutex定义了一个继承AQS的静态内部类Sync,并且重写了AQS的tryAcquire等等方法，而对state的更新也是利用了setState(),getState()，compareAndSetState()这三个方法。在实现实现lock接口中的方法也只是调用了AQS提供的模板方法（因为Sync继承AQS）。

从这个例子就可以很清楚的看出来，在同步组件的实现上主要是利用了AQS，而AQS“屏蔽”了同步状态的修改，线程排队等底层实现，通过AQS的模板方法可以很方便的给同步组件的实现者进行调用。而针对用户来说，只需要调用同步组件提供的方法来实现并发编程即可。

同时在新建一个同步组件时需要把握的两个关键点是：

- 实现同步组件时推荐定义继承AQS的静态内存类，并重写需要的protected修饰的方法；
- 同步组件语义的实现依赖于AQS的模板方法，而AQS模板方法又依赖于被AQS的子类所重写的方法。

通俗点说，因为AQS整体设计思路采用模板方法设计模式，同步组件以及AQS的功能实际上别切分成各自的两部分：

**同步组件实现者的角度：**

通过可重写的方法：

- 独占式： tryAcquire()(独占式获取同步状态），tryRelease()（独占式释放同步状态）；

- 共享式 ：tryAcquireShared()(共享式获取同步状态)，tryReleaseShared()(共享式释放同步状态)；

告诉AQS怎样判断当前同步状态是否成功获取或者是否成功释放。同步组件专注于对当前同步状态的逻辑判断，从而实现自己的同步语义。

这句话比较抽象，举例来说，上面的Mutex例子中通过tryAcquire方法实现自己的同步语义，在该方法中如果当前同步状态为0（即该同步组件没被任何线程获取），当前线程可以获取同时将状态更改为1返回true，否则，该组件已经被线程占用返回false。很显然，该同步组件只能在同一时刻被线程占用，Mutex专注于获取释放的逻辑来实现自己想要表达的同步语义。

**AQS的角度**

而对AQS来说，只需要同步组件返回的true和false即可，因为AQS会对true和false会有不同的操作，true会认为当前线程获取同步组件成功直接返回，而false的话就AQS也会将当前线程插入同步队列等一系列的方法。

总的来说，同步组件通过重写AQS的方法实现自己想要表达的同步语义，而AQS只需要同步组件表达的true和false即可，AQS会针对true和false不同的情况做不同的处理

#### 同步队列

当共享资源被某个线程占有，其他请求该资源的线程将会阻塞，从而进入同步队列。就数据结构而言，队列的实现方式无外乎两者一是通过数组的形式，另外一种则是链表的形式。AQS中的同步队列则是通过链式方式进行实现。接下来，很显然我们至少会抱有这样的疑问：
1. 节点的数据结构是什么样的？
2. 是单向还是双向？
3. 是带头结点的还是不带头节点的？

我们依旧先是通过看源码的方式。

在AQS有一个静态内部类Node，其中有这样一些属性：
```java
volatile int waitStatus //节点状态
volatile Node prev //当前节点/线程的前驱节点
volatile Node next; //当前节点/线程的后继节点
volatile Thread thread;//加入同步队列的线程引用
Node nextWaiter;//等待队列中的下一个节点
```
节点的状态有以下这些：
```java
int CANCELLED =  1//节点从同步队列中取消
int SIGNAL    = -1//后继节点的线程处于等待状态，如果当前节点释放同步状态会通知后继节点，使得后继节点的线程能够运行；
int CONDITION = -2//当前节点进入等待队列中
int PROPAGATE = -3//表示下一次共享式同步状态获取将会无条件传播下去
int INITIAL = 0;//初始状态
```
现在我们知道了节点的数据结构类型，并且每个节点拥有其前驱和后继节点，很显然这是一个双向队列。
```java
public class LockDemo {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(() -> {
                lock.lock();
                Thread t = Thread.currentThread();
                System.out.println(t.getId()+"获取锁");
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println(t.getId()+"释放锁");
                    lock.unlock();
                }
            });
            thread.start();
        }
    }
}
```
实例代码中开启了5个线程，先获取锁之后再睡眠10S中，实际上这里让线程睡眠是想模拟出当线程无法获取锁时进入同步队列的情况。

Thread-0先获得锁后进行睡眠，其他线程（Thread-1,Thread-2,Thread-3,Thread-4）获取锁失败进入同步队列，同时也可以很清楚的看出来每个节点有两个域：prev(前驱)和next(后继)，并且每个节点用来保存获取同步状态失败的线程引用以及等待状态等信息。另外AQS中有两个重要的成员变量：

```java
private transient volatile Node head;
private transient volatile Node tail;
```
也就是说AQS实际上通过头尾指针来管理同步队列，同时实现包括获取锁失败的线程进行入队，释放锁时对同步队列中的线程进行通知等核心方法。其示意图如下：
![3.png](https://i.loli.net/2019/05/06/5cd01386492b0.png)

通过对源码的理解以及做实验的方式，现在我们可以清楚的知道这样几点：

- 节点的数据结构，即AQS的静态内部类Node,节点的等待状态等信息；
- 同步队列是一个双向队列，AQS通过持有头尾指针管理同步队列；

那么，节点如何进行入队和出队是怎样做的了？实际上这对应着锁的获取和释放两个操作：获取锁失败进行入队操作，获取锁成功进行出队操作。

### 独占锁

#### 独占锁的获取（acquire方法）
我们继续通过看源码和debug的方式来看，还是以上面的demo为例，调用lock()方法是获取独占式锁，获取失败就将当前线程加入同步队列，成功则线程执行。而lock()方法实际上会调用AQS的acquire()方法，源码如下
```java
public final void acquire(int arg) {
        //先看同步状态是否获取成功，如果成功则方法结束返回
        //若失败则先调用addWaiter()方法再调用acquireQueued()方法
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```
关键信息请看注释，acquire根据当前获得同步状态成功与否做了两件事情：
1. 成功，则方法结束返回
2. 失败，则先调用addWaiter()然后在调用acquireQueued()方法。

**获取同步状态失败，入队操作**

当线程获取独占式锁失败后就会将当前线程加入同步队列，那么加入队列的方式是怎样的了？我们接下来就应该去研究一下addWaiter()和acquireQueued()。addWaiter()源码如下：
```java
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
分析可以看上面的注释。程序的逻辑主要分为两个部分：
1. 当前同步队列的尾节点为null，调用方法enq()插入;
2. 当前队列的尾节点不为null，则采用尾插入（compareAndSetTail（）方法）的方式入队。

另外还会有另外一个问题：如果 if (compareAndSetTail(pred, node))为false怎么办？会继续执行到enq()方法，同时很明显compareAndSetTail是一个CAS操作，通常来说如果CAS操作失败会继续自旋（死循环）进行重试。

因此，经过我们这样的分析，enq()方法可能承担两个任务：
1. 处理当前同步队列尾节点为null时进行入队操作；
2. 如果CAS尾插入节点失败后负责自旋进行尝试。

那么是不是真的就像我们分析的一样了？只有源码会告诉我们答案,enq()源码如下：
```java
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                //1. 构造头结点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 2. 尾插入，CAS操作失败自旋尝试
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
}
```
在上面的分析中我们可以看出在第1步中会先创建头结点，说明同步队列是带头结点的链式存储结构。带头结点与不带头结点相比，会在入队和出队的操作中获得更大的便捷性，因此同步队列选择了带头结点的链式存储结构。那么带头节点的队列初始化时机是什么？自然而然是在tail为null时，即当前线程是第一次插入同步队列。compareAndSetTail(t, node)方法会利用CAS操作设置尾节点，如果CAS操作失败会在for (;;)for死循环中不断尝试，直至成功return返回为止。因此，对enq()方法可以做这样的总结：


- 在当前线程是第一个加入同步队列时，调用compareAndSetHead(new Node())方法，完成链式队列的头结点的初始化；
- 自旋不断尝试CAS尾插入节点直至成功为止。

现在我们已经很清楚获取独占式锁失败的线程包装成Node然后插入同步队列的过程了？那么紧接着会有下一个问题？在同步队列中的节点（线程）会做什么事情了来保证自己能够有机会获得独占式锁了？带着这样的问题我们就来看看acquireQueued()方法，从方法名就可以很清楚，这个方法的作用就是排队获取锁的过程，源码如下：

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                // 1. 获得当前节点的先驱节点
                final Node p = node.predecessor();
                // 2. 当前节点能否获取独占式锁                  
                // 2.1 如果当前节点的先驱节点是头结点并且成功获取同步状态，即可以获得独占式锁
                if (p == head && tryAcquire(arg)) {
                    //队列头指针用指向当前节点
                    setHead(node);
                    //释放前驱节点
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 2.2 获取锁失败，线程进入等待状态等待获取独占式锁
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
程序逻辑通过注释已经标出，整体来看这是一个这又是一个自旋的过程（for (;;)），代码首先获取当前节点的先驱节点，如果先驱节点是头结点的并且成功获得同步状态的时候（if (p == head && tryAcquire(arg))），当前节点所指向的线程能够获取锁。反之，获取锁失败进入等待状态。整体示意图为下图：

![1.png](https://i.loli.net/2019/05/06/5cd0151d31bc0.png)

**获取锁成功，出队操作**

获取锁的节点出队的逻辑是：
```
//队列头结点引用指向当前节点
setHead(node);
//释放前驱节点
p.next = null; // help GC
failed = false;
return interrupted;
```
setHead()方法为：
```
private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
}
```
将当前节点通过setHead()方法设置为队列的头结点，然后将之前的头结点的next域设置为null并且pre域也为null，即与队列断开，无任何引用方便GC时能够将内存进行回收。示意图如下：
![2.png](https://i.loli.net/2019/05/06/5cd015883b1cb.png)

那么当获取锁失败的时候会调用shouldParkAfterFailedAcquire()方法和parkAndCheckInterrupt()方法。

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    if (ws > 0) {
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
shouldParkAfterFailedAcquire()方法主要逻辑是使用`compareAndSetWaitStatus(pred, ws, Node.SIGNAL)`使用CAS将节点状态由INITIAL设置成SIGNAL，表示当前线程阻塞。当compareAndSetWaitStatus设置失败则说明shouldParkAfterFailedAcquire方法返回false，然后会在acquireQueued()方法中for (;;)死循环中会继续重试，直至compareAndSetWaitStatus设置节点状态位为SIGNAL时shouldParkAfterFailedAcquire返回true时才会执行方法parkAndCheckInterrupt()方法，该方法的源码为：

```
private final boolean parkAndCheckInterrupt() {
        //使得该线程阻塞
        LockSupport.park(this);
        return Thread.interrupted();
}
```
该方法的关键是会调用LookSupport.park()方法（关于LookSupport会在以后的文章进行讨论），该方法是用来阻塞当前线程的。因此到这里就应该清楚了，acquireQueued()在自旋过程中主要完成了两件事情：

- 如果当前节点的前驱节点是头节点，并且能够获得同步状态的话，当前线程能够获得锁该方法执行结束退出；

- 获取锁失败的话，先将节点状态设置成SIGNAL，然后调用LookSupport.park方法使得当前线程阻塞。

经过上面的分析，独占式锁的获取过程也就是acquire()方法的执行流程如下图所示：

![1.png](https://i.loli.net/2019/05/06/5cd0165b4cfad.png)

####  独占锁的释放（release()方法）

独占锁的释放就相对来说比较容易理解了，废话不多说先来看下源码：
```
public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
}
```
这段代码逻辑就比较容易理解了，如果同步状态释放成功（tryRelease返回true）则会执行if块中的代码，当head指向的头结点不为null，并且该节点的状态值不为0的话才会执行unparkSuccessor()方法。unparkSuccessor方法源码：
```
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */

    //头节点的后继节点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        //后继节点不为null时唤醒该线程
        LockSupport.unpark(s.thread);
}
```
源码的关键信息请看注释，首先获取头节点的后继节点，当后继节点的时候会调用LookSupport.unpark()方法，该方法会唤醒该节点的后继节点所包装的线程。因此，**每一次锁释放后就会唤醒队列中该节点的后继节点所引用的线程，从而进一步可以佐证获得锁的过程是一个FIFO（先进先出）的过程**。

到现在我们终于啃下了一块硬骨头了，通过学习源码的方式非常深刻的学习到了独占式锁的获取和释放的过程以及同步队列。可以做一下总结：

1. 线程获取锁失败，线程被封装成Node进行入队操作，核心方法在于addWaiter()和enq()，同时enq()完成对同步队列的头结点初始化工作以及CAS操作失败的重试;

2. 线程获取锁是一个自旋的过程，当且仅当 当前节点的前驱节点是头结点并且成功获得同步状态时，节点出队即该节点引用的线程获得锁，否则，当不满足条件时就会调用LookSupport.park()方法使得线程阻塞；

3. 释放锁的时候会唤醒后继节点；

总体来说：**在获取同步状态时，AQS维护一个同步队列，获取同步状态失败的线程会加入到队列中进行自旋；移除队列（或停止自旋）的条件是前驱节点是头结点并且成功获得了同步状态。在释放同步状态时，同步器会调用unparkSuccessor()方法唤醒后继节点。**


#### 可中断式获取锁（acquireInterruptibly方法）

我们知道lock相较于synchronized有一些更方便的特性，比如能响应中断以及超时等待等特性，现在我们依旧采用通过学习源码的方式来看看能够响应中断是怎么实现的。可响应中断式锁可调用方法lock.lockInterruptibly();而该方法其底层会调用AQS的acquireInterruptibly方法，源码为：
```
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        //线程获取锁失败
        doAcquireInterruptibly(arg);
}
```
在获取同步状态失败后就会调用doAcquireInterruptibly方法：
```
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    //将节点插入到同步队列中
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            //获取锁出队
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                //线程中断抛异常
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
关键信息请看注释，现在看这段代码就很轻松了吧:),与acquire方法逻辑几乎一致，唯一的区别是当parkAndCheckInterrupt返回true时即线程阻塞时该线程被中断，代码抛出被中断异常。

#### 超时等待式获取锁（tryAcquireNanos()方法）

通过调用lock.tryLock(timeout,TimeUnit)方式达到超时等待获取锁的效果，该方法会在三种情况下才会返回：

- 在超时时间内，当前线程成功获取了锁；
- 当前线程在超时时间内被中断；
- 超时时间结束，仍未获得锁返回false。

我们仍然通过采取阅读源码的方式来学习底层具体是怎么实现的，该方法会调用AQS的方法tryAcquireNanos(),源码为：
```
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    return tryAcquire(arg) ||
        //实现超时等待的效果
        doAcquireNanos(arg, nanosTimeout);
}
```
很显然这段源码最终是靠doAcquireNanos方法实现超时等待的效果，该方法源码如下：
```
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    //1. 根据超时时间和当前时间计算出截止时间
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            //2. 当前线程获得锁出队列
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            // 3.1 重新计算超时时间
            nanosTimeout = deadline - System.nanoTime();
            // 3.2 已经超时返回false
            if (nanosTimeout <= 0L)
                return false;
            // 3.3 线程阻塞等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            // 3.4 线程被中断抛出被中断异常
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
程序逻辑如图所示：

![2.png](https://i.loli.net/2019/05/06/5cd0172a464bc.png)

程序逻辑同独占锁可响应中断式获取基本一致，唯一的不同在于获取锁失败后，对超时时间的处理上，在第1步会先计算出按照现在时间和超时时间计算出理论上的截止时间，比如当前时间是8h10min,超时时间是10min，那么根据deadline = System.nanoTime() + nanosTimeout计算出刚好达到超时时间时的系统时间就是8h 10min+10min = 8h 20min。然后根据deadline - System.nanoTime()就可以判断是否已经超时了，比如，当前系统时间是8h 30min很明显已经超过了理论上的系统时间8h 20min，deadline - System.nanoTime()计算出来就是一个负数，自然而然会在3.2步中的If判断之间返回false。

如果还没有超时即3.2步中的if判断为true时就会继续执行3.3步通过LockSupport.parkNanos使得当前线程阻塞，同时在3.4步增加了对中断的检测，若检测出被中断直接抛出被中断异常。



### 共享锁

#### 共享锁的获取（acquireShared()方法）

在聊完AQS对独占锁的实现后，我们继续一鼓作气的来看看共享锁是怎样实现的？共享锁的获取方法为acquireShared，源码为：
```
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
这段源码的逻辑很容易理解，在该方法中会首先调用tryAcquireShared方法，tryAcquireShared返回值是一个int类型，当返回值为大于等于0的时候方法结束说明获得成功获取锁，否则，表明获取同步状态失败即所引用的线程获取锁失败，会执行doAcquireShared方法，该方法的源码为：
```
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 当该节点的前驱节点是头结点且成功获取同步状态
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
```
现在来看这段代码会不会很容易了？逻辑几乎和独占式锁的获取一模一样，这里的自旋过程中能够退出的条件是 **当前节点的前驱节点是头结点并且tryAcquireShared(arg)返回值大于等于0即能成功获得同步状态。**

#### 共享锁的释放（releaseShared()方法）
共享锁的释放在AQS中会调用方法releaseShared：
```
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
当成功释放同步状态之后即tryReleaseShared会继续执行doReleaseShared方法：
```
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
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
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```
这段方法跟独占式锁释放过程有点点不同，在共享式锁的释放过程中，对于能够支持多个线程同时访问的并发组件，必须保证多个线程能够安全的释放同步状态，这里采用的CAS保证，当CAS操作失败continue，在下一次循环中进行重试。

#### 可中断（acquireSharedInterruptibly()方法），超时等待（tryAcquireSharedNanos()方法）

关于可中断锁以及超时等待的特性其实现和独占式锁可中断获取锁以及超时等待的实现几乎一致，具体的就不再说了，如果理解了上面的内容对这部分的理解也是水到渠成的。



### 参考
1. [初识Lock与AbstractQueuedSynchronizer(AQS)](https://www.jianshu.com/p/7a65ab32de2a)
2. [深入理解AbstractQueuedSynchronizer(AQS)](https://www.jianshu.com/p/cc308d82cc71)
