---
title: Java 线程阀--计数器CountDownLatch、Semaphore、CyclicBarrier
date: 2019-05-10 23:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### 同步计数器CountDownLatch

CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。在Java并发中，CountDownLatch的概念是一个常见的面试题，所以一定要确保你很好的理解了它.

<!--more-->

CountDownLatch是在java1.5被引入的，跟它一起被引入的并发工具类还有CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue，它们都存在于java.util.concurrent包下。CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

![8.png](https://i.loli.net/2019/05/13/5cd8d4bd2824f67294.png)

CountDownLatch的伪代码如下所示：

```
//Main thread start
//Create CountDownLatch for N threads
//Create and start N threads
//Main thread wait on latch
//N threads completes there tasks are returns
//Main thread resume execution
```

#### CountDownLatch如何工作

构造器中的**计数值（count）实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。

与CountDownLatch的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用**CountDownLatch.await()**方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他N 个线程必须引用闭锁对象，因为他们需要通知CountDownLatch对象，他们已经完成了各自的任务。这种通知机制是通过 **CountDownLatch.countDown()**方法来完成的；每调用一次这个方法，在构造函数中初始化的count值就减1。所以当N个线程都调 用了这个方法，count的值等于0，然后主线程就能通过await()方法，恢复执行自己的任务。

#### 在实时系统中的使用场景

让我们尝试罗列出在java实时系统中CountDownLatch都有哪些使用场景。我所罗列的都是我所能想到的。如果你有别的可能的使用方法，请在留言里列出来，这样会帮助到大家。

1. **实现最大的并行性**：有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为1的CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用  一次countDown()方法就可以让所有的等待线程同时恢复执行。
2. **开始执行前等待n个线程完成各自任务**：例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。
3. **死锁检测：**一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

#### CountDownLatch使用例子

在这个例子中，我模拟了一个应用程序启动类，它开始时启动了n个线程类，这些线程将检查外部系统并通知闭锁，并且启动类一直在闭锁上等待着。一旦验证和检查了所有外部服务，那么启动类恢复执行。

**BaseHealthChecker.java：**这个类是一个Runnable，负责所有特定的外部服务健康的检测。它删除了重复的代码和闭锁的中心控制代码。

```java
public abstract class BaseHealthChecker implements Runnable {
 
    private CountDownLatch _latch;
    private String _serviceName;
    private boolean _serviceUp;
 
    //Get latch object in constructor so that after completing the task, thread can countDown() the latch
    public BaseHealthChecker(String serviceName, CountDownLatch latch)
    {
        super();
        this._latch = latch;
        this._serviceName = serviceName;
        this._serviceUp = false;
    }
 
    @Override
    public void run() {
        try {
            verifyService();
            _serviceUp = true;
        } catch (Throwable t) {
            t.printStackTrace(System.err);
            _serviceUp = false;
        } finally {
            if(_latch != null) {
                _latch.countDown();
            }
        }
    }
 
    public String getServiceName() {
        return _serviceName;
    }
 
    public boolean isServiceUp() {
        return _serviceUp;
    }
    //This methos needs to be implemented by all specific service checker
    public abstract void verifyService();
}
```

**NetworkHealthChecker.java：**这个类继承了BaseHealthChecker，实现了verifyService()方法。**DatabaseHealthChecker.java**和**CacheHealthChecker.java**除了服务名和休眠时间外，与NetworkHealthChecker.java是一样的

```java
public class NetworkHealthChecker extends BaseHealthChecker
{
    public NetworkHealthChecker (CountDownLatch latch)  {
        super("Network Service", latch);
    }
 
    @Override
    public void verifyService()
    {
        System.out.println("Checking " + this.getServiceName());
        try
        {
            Thread.sleep(7000);
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
        System.out.println(this.getServiceName() + " is UP");
    }
}
```

**ApplicationStartupUtil.java：**这个类是一个主启动类，它负责初始化闭锁，然后等待，直到所有服务都被检测完。

```java
public class ApplicationStartupUtil
{
    //List of service checkers
    private static List<BaseHealthChecker> _services;
 
    //This latch will be used to wait on
    private static CountDownLatch _latch;
 
    private ApplicationStartupUtil()
    {
    }
 
    private final static ApplicationStartupUtil INSTANCE = new ApplicationStartupUtil();
 
    public static ApplicationStartupUtil getInstance()
    {
        return INSTANCE;
    }
 
    public static boolean checkExternalServices() throws Exception
    {
        //Initialize the latch with number of service checkers
        _latch = new CountDownLatch(3);
 
        //All add checker in lists
        _services = new ArrayList<BaseHealthChecker>();
        _services.add(new NetworkHealthChecker(_latch));
        _services.add(new CacheHealthChecker(_latch));
        _services.add(new DatabaseHealthChecker(_latch));
 
        //Start service checkers using executor framework
        Executor executor = Executors.newFixedThreadPool(_services.size());
 
        for(final BaseHealthChecker v : _services)
        {
            executor.execute(v);
        }
 
        //Now wait till all services are checked
        _latch.await();
 
        //Services are file and now proceed startup
        for(final BaseHealthChecker v : _services)
        {
            if( ! v.isServiceUp())
            {
                return false;
            }
        }
        return true;
    }
}
```

现在你可以写测试代码去检测一下闭锁的功能了。

```java
public class Main {
    public static void main(String[] args)
    {
        boolean result = false;
        try {
            result = ApplicationStartupUtil.checkExternalServices();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("External services validation completed !! Result was :: "+ result);
    }
}
```

#### 源码分析

##### 内部类Sync

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;
    
    // 传入初始次数
    Sync(int count) {
        setState(count);
    }
    // 获取还剩的次数
    int getCount() {
        return getState();
    }
    // 尝试获取共享锁
    protected int tryAcquireShared(int acquires) {
        // 注意，这里state等于0的时候返回的是1，也就是说count减为0的时候获取总是成功
        // state不等于0的时候返回的是-1，也就是count不为0的时候总是要排队
        return (getState() == 0) ? 1 : -1;
    }
    // 尝试释放锁
    protected boolean tryReleaseShared(int releases) {
        for (;;) {
            // state的值
            int c = getState();
            // 等于0了，则无法再释放了
            if (c == 0)
                return false;
            // 将count的值减1
            int nextc = c-1;
            // 原子更新state的值
            if (compareAndSetState(c, nextc))
                // 减为0的时候返回true，这时会唤醒后面排队的线程
                return nextc == 0;
        }
    }
}
```

Sync重写了tryAcquireShared()和tryReleaseShared()方法，并把count存到state变量中去。

这里要注意一下，上面两个方法的参数并没有什么卵用。

##### 构造方法

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

构造方法需要传入一个count，也就是初始次数。

##### await()方法

```java
// java.util.concurrent.CountDownLatch.await()
public void await() throws InterruptedException {
    // 调用AQS的acquireSharedInterruptibly()方法
    sync.acquireSharedInterruptibly(1);
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly()
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    // 尝试获取锁，如果失败则排队
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

await()方法是等待其它线程完成的方法，它会先尝试获取一下共享锁，如果失败则进入AQS的队列中排队等待被唤醒。

根据上面Sync的源码，我们知道，state不等于0的时候tryAcquireShared()返回的是-1，也就是说count未减到0的时候所有调用await()方法的线程都要排队。

##### countDown()方法

```java
// java.util.concurrent.CountDownLatch.countDown()
public void countDown() {
    // 调用AQS的释放共享锁方法
    sync.releaseShared(1);
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.releaseShared()
public final boolean releaseShared(int arg) {
    // 尝试释放共享锁，如果成功了，就唤醒排队的线程
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

countDown()方法，会释放一个共享锁，也就是count的次数会减1。

根据上面Sync的源码，我们知道，tryReleaseShared()每次会把count的次数减1，当其减为0的时候返回true，这时候才会唤醒等待的线程。

注意，doReleaseShared()是唤醒等待的线程

##### 总结

1. CountDownLatch表示允许一个或多个线程等待其它线程的操作执行完毕后再执行后续的操作；
2. CountDownLatch使用AQS的共享锁机制实现；
3. CountDownLatch初始化的时候需要传入次数count；
4. 每次调用countDown()方法count的次数减1
5. 每次调用await()方法的时候会尝试获取锁，这里的获取锁其实是检查AQS的state变量的值是否为0；
6. 当count的值（也就是state的值）减为0的时候会唤醒排队着的线程（这些线程调用await()进入队列）；

#### 常见面试题

可以为你的下次面试准备以下一些CountDownLatch相关的问题：

- 解释一下CountDownLatch概念?
- CountDownLatch 和CyclicBarrier的不同之处?
- 给出一些CountDownLatch使用的例子?
-  CountDownLatch 类中主要的方法?

### 信号量Semaphore

Semaphore又称信号量，是操作系统中的一个概念，在Java并发编程中，信号量控制的是线程并发的数量。

Semaphore管理一系列许可证。每个acquire方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个release方法增加一个许可证，这可能会释放一个阻塞的acquire方法。然而，其实并没有实际的许可证这个对象，Semaphore只是维持了一个可获得许可证的数量。 

Reentrantlock是锁住只有一个线程可以操作，而Semaphore可以设置有多少线程可以同时并发

Reentrantlock通过state状态是否0，以及阻塞队列队头不断获取锁，来保持始终一个线程获取锁执行，获取锁的动作也是CAS并且标记下是哪个线程获得了锁，更改下state；而Semaphore 通过state(可获得的信号量)，以及acquire需要的信号量进行计算，是否<0去决定能不能获得信号量，去执行compareAndSetState 就表示获得了信号量可以继续走了

#### 方法

- Semaphore(int permits)
  创建具有给定许可数目、非公平的Semphore对象

- Semaphore(int permits, boolean fair)
  创建具有给定许可数目、公平的Semaphore对象。所谓公平性就是先来先服务FIFO

- void acquire()
  从此信号量获取一个许可，在获取到许可之前线程将被阻塞

- int availablePermits()
  返回此信号量中的可用许可数目

- void release()

  释放当前许可

#### 阻塞队列和挂起线程

需要获取的信号量-可允许的信号量<0，线程就会被放入阻塞队列addWaiter(Node.SHARED)，并且被挂起parkAndCheckInterrupt

```java
 private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### 用法

Semaphore又称信号量，是操作系统中的一个概念，在Java并发编程中，信号量控制的是线程并发的数量。

```
public Semaphore(int permits)
```

其中参数*permits*就是允许同时运行的线程数目;

下面先看一个信号量实现单线程的例子，也就是*permits*=1:

```java
import java.util.concurrent.Semaphore;

class Driver {
    // 控制线程的数目为1，也就是单线程
    private Semaphore semaphore = new Semaphore(1);

    public void driveCar() {
        try {
            // 从信号量中获取一个允许机会
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + " start at " + System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " stop at " + System.currentTimeMillis());
            // 释放允许，将占有的信号量归还
            semaphore.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
 class Car extends Thread{
    private Driver driver;

    public Car(Driver driver) {
        super();
        this.driver = driver;
    }

    public void run() {
        driver.driveCar();
    }
}
public class Run {
    public static void main(String[] args) {
        Driver driver = new Driver();
        for (int i = 0; i < 5; i++) {
            (new Car(driver)).start();
        }
    }
}
```

从输出可以看出，改输出与单线程是一样的，执行完一个线程，再执行另一个线程。

如果信号量大于1呢，我们将信号量设为3。

从输出的前三行可以看出，有3个线程可以同时执行，三个线程同时运行的时候，第四个线程必须等待前面有一个要完成，才能执行第四个线程启动。

当然也可以用acquire动态地添加*permits*的数量，它表示的是一次性获取许可的数量。

我们可以用`public int availablePermits()`查看现在可用的信号量。

还有一个方法`public int drainPermits()`，这个方法返回即可所有的许可数目，并将许可置0。

#### 源码分析

##### 内部类Sync

```java
// java.util.concurrent.Semaphore.Sync
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 1192457210091910933L;
    // 构造方法，传入许可次数，放入state中
    Sync(int permits) {
        setState(permits);
    }
    // 获取许可次数
    final int getPermits() {
        return getState();
    }
    // 非公平模式尝试获取许可
    final int nonfairTryAcquireShared(int acquires) {
        for (;;) {
            // 看看还有几个许可
            int available = getState();
            // 减去这次需要获取的许可还剩下几个许可
            int remaining = available - acquires;
            // 如果剩余许可小于0了则直接返回
            // 如果剩余许可不小于0，则尝试原子更新state的值，成功了返回剩余许可
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
    // 释放许可
    protected final boolean tryReleaseShared(int releases) {
        for (;;) {
            // 看看还有几个许可
            int current = getState();
            // 加上这次释放的许可
            int next = current + releases;
            // 检测溢出
            if (next < current) // overflow
                throw new Error("Maximum permit count exceeded");
            // 如果原子更新state的值成功，就说明释放许可成功，则返回true
            if (compareAndSetState(current, next))
                return true;
        }
    }
    // 减少许可
    final void reducePermits(int reductions) {
        for (;;) {
            // 看看还有几个许可
            int current = getState();
            // 减去将要减少的许可
            int next = current - reductions;
            // 检测举出
            if (next > current) // underflow
                throw new Error("Permit count underflow");
            // 原子更新state的值，成功了返回true
            if (compareAndSetState(current, next))
                return;
        }
    }
    // 销毁许可
    final int drainPermits() {
        for (;;) {
            // 看看还有几个许可
            int current = getState();
            // 如果为0，直接返回
            // 如果不为0，把state原子更新为0
            if (current == 0 || compareAndSetState(current, 0))
                return current;
        }
    }
}
```

通过Sync的几个实现方法，我们获取到以下几点信息：

1. 许可是在构造方法时传入的；
2. 许可存放在状态变量state中；
3. 尝试获取一个许可的时候，则state的值减1；
4. 当state的值为0的时候，则无法再获取许可；
5. 释放一个许可的时候，则state的值加1；
6. 许可的个数可以动态改变；

##### 内部类NonfairSync

```java
// java.util.concurrent.Semaphore.NonfairSync
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;
    // 构造方法，调用父类的构造方法
    NonfairSync(int permits) {
        super(permits);
    }
    // 尝试获取许可，调用父类的nonfairTryAcquireShared()方法
    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}
```

非公平模式下，直接调用父类的nonfairTryAcquireShared()尝试获取许可。

##### 内部类FairSync

```java
// java.util.concurrent.Semaphore.FairSync
static final class FairSync extends Sync {
    private static final long serialVersionUID = 2014338818796000944L;
    // 构造方法，调用父类的构造方法
    FairSync(int permits) {
        super(permits);
    }
    // 尝试获取许可
    protected int tryAcquireShared(int acquires) {
        for (;;) {
            // 公平模式需要检测是否前面有排队的
            // 如果有排队的直接返回失败
            if (hasQueuedPredecessors())
                return -1;
            // 没有排队的再尝试更新state的值
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
}
```

公平模式下，先检测前面是否有排队的，如果有排队的则获取许可失败，进入队列排队，否则尝试原子更新state的值。

##### 构造方法

```java
// 构造方法，创建时要传入许可次数，默认使用非公平模式
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}
// 构造方法，需要传入许可次数，及是否公平模式
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

创建Semaphore时需要传入许可次数。

Semaphore默认也是非公平模式，但是你可以调用第二个构造方法声明其为公平模式。

以下的方法都是针对非公平模式来描述。

##### acquire()方法

```
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```

获取一个许可，默认使用的是可中断方式，如果尝试获取许可失败，会进入AQS的队列中排队。

##### acquireUninterruptibly()方法

```
public void acquireUninterruptibly() {
    sync.acquireShared(1);
}
```

获取一个许可，非中断方式，如果尝试获取许可失败，会进入AQS的队列中排队。

##### tryAcquire()方法

```
public boolean tryAcquire() {
    return sync.nonfairTryAcquireShared(1) >= 0;
}
```

尝试获取一个许可，使用Sync的非公平模式尝试获取许可方法，不论是否获取到许可都返回，只尝试一次，不会进入队列排队。

##### tryAcquire(long timeout, TimeUnit unit)方法

```
public boolean tryAcquire(long timeout, TimeUnit unit)
    throws InterruptedException {
    return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
}
```

尝试获取一个许可，先尝试一次获取许可，如果失败则会等待timeout时间，这段时间内都没有获取到许可，则返回false，否则返回true；

##### release()方法

```
public void release() {
    sync.releaseShared(1);
}
```

释放一个许可，释放一个许可时state的值会加1，并且会唤醒下一个等待获取许可的线程。

##### acquire(int permits)方法

```
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
```

一次获取多个许可，可中断方式。

##### acquireUninterruptibly(int permits)方法

```
public void acquireUninterruptibly(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireShared(permits);
}
```

一次获取多个许可，非中断方式。

##### tryAcquire(int permits)方法

```
public boolean tryAcquire(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.nonfairTryAcquireShared(permits) >= 0;
}
```

一次尝试获取多个许可，只尝试一次。

##### tryAcquire(int permits, long timeout, TimeUnit unit)方法

```
public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
}
```

尝试获取多个许可，并会等待timeout时间，这段时间没获取到许可则返回false，否则返回true。

##### release(int permits)方法

```
public void release(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    sync.releaseShared(permits);
}
```

一次释放多个许可，state的值会相应增加permits的数量。

##### availablePermits()方法

```
public int availablePermits() {
    return sync.getPermits();
}
```

获取可用的许可次数。

##### drainPermits()方法

```
public int drainPermits() {
    return sync.drainPermits();
}
```

销毁当前可用的许可次数，对于已经获取的许可没有影响，会把当前剩余的许可全部销毁。

##### reducePermits(int reduction)方法

```
protected void reducePermits(int reduction) {
    if (reduction < 0) throw new IllegalArgumentException();
    sync.reducePermits(reduction);
}
```

减少许可的次数。

##### 总结

1. Semaphore，也叫信号量，通常用于控制同一时刻对共享资源的访问上，也就是限流场景；

2. Semaphore的内部实现是基于AQS的共享锁来实现的；

3. Semaphore初始化的时候需要指定许可的次数，许可的次数是存储在state中；

4. 获取一个许可时，则state值减1；

5. 释放一个许可时，则state值加1

6. 可以动态减少n个许可；

7. 可以动态增加n个许可吗？

   调用release(int permits)即可。我们知道释放许可的时候state的值会相应增加，再回头看看释放许可的源码，发现与ReentrantLock的释放锁还是有点区别的，Semaphore释放许可的时候并不会检查当前线程有没有获取过许可，所以可以调用释放许可的方法动态增加一些许可。

### 同步计数器CyclicBarrier

循环屏障允许一组线程互相等待，直到到达某个公共屏障点，然后所有的这组线程再同步往后执行。因为该 barrier 在释放等待线程后可以重用，所以称它为循环的barrier。

主要方法

- CyclicBarrier(int parties)
  创建一个新的循环屏障，它将在给定数量的线程处于等待状态时启动，但不会在启动barrier时执行预定义的操作。
- CyClicBarrier(int parties, Runnable barrierAction)
  创建一个新的循环屏障，它将在给定数量的线程处于等待状态时启动，并在启动barrier时执行给定的屏障操作barrierAction，该操作由最后一个进入barrier的线程执行。

- int await()
  在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。

- int await(long timeout, TimeUnit unit)
  在所有参与者都已经在此屏障上调用 await 方法之前将一直等待,或者超出了指定的等待时间。

- int getNumberWaiting()
  返回当前在屏障处等待的线程数目

- int getParties()
  返回要求启动此barrier的线程数目

- void reset()
  将循环屏障重置为初始状态。

示例：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(3, new TotalTask());

        Worker worker1 = new Worker("worker1", barrier);
        Worker worker2 = new Worker("worker2", barrier);
        Worker worker3 = new Worker("worker3", barrier);

        worker1.start();
        worker2.start();
        worker3.start();

        System.out.println("main thread end");
    }

    // 启动barrier时执行该任务，即当最后一个线程进入barrier时执行该任务
    static class TotalTask extends Thread {
        @Override
        public void run() {
            System.out.println("所有线程到达barrier");
        }
    }

    // 任务线程
    static class Worker extends Thread {
        private String name;
        private CyclicBarrier barrier;

        public Worker(String name, CyclicBarrier barrier) {
            this.name = name;
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                System.out.println(name+"任务开始");
                Thread.sleep(1000L);
                System.out.println(name+"任务完成");

                barrier.await();    // 线程到达屏障
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### CountDownLatch与CyclicBarrier的区别

CountDownLatch：一个线程等待另外N个线程完成某个事情之后才能执行，重点是一个线程在等待 
CyclicBarrier：N个线程互相等待，任何一个线程完成之前，所有线程都必须等待。

#### 主要内部类

```
private static class Generation {
    boolean broken = false;
}
```

Generation，中文翻译为代，一代人的代，用于控制CyclicBarrier的循环使用。

比如，上面示例中的三个线程完成后进入下一代，继续等待三个线程达到栅栏处再一起执行，而CountDownLatch则做不到这一点，CountDownLatch是一次性的，无法重置其次数。

#### 主要属性

```java
// 重入锁
private final ReentrantLock lock = new ReentrantLock();
// 条件锁，名称为trip，绊倒的意思，可能是指线程来了先绊倒，等达到一定数量了再唤醒
private final Condition trip = lock.newCondition();
// 需要等待的线程数量
private final int parties;
// 当唤醒的时候执行的命令
private final Runnable barrierCommand;
// 代
private Generation generation = new Generation();
// 当前这一代还需要等待的线程数
private int count;
```

通过属性可以看到，CyclicBarrier内部是通过重入锁的条件锁来实现的，那么你可以脑补一下这个场景吗？

脑补一下：假如初始时`count = parties = 3`，当第一个线程到达栅栏处，count减1，然后把它加入到Condition的队列中，第二个线程到达栅栏处也是如此，第三个线程到达栅栏处，count减为0，调用Condition的signalAll()通知另外两个线程，然后把它们加入到AQS的队列中，等待当前线程运行完毕，调用lock.unlock()的时候依次从AQS的队列中唤醒一个线程继续运行，也就是说实际上三个线程先依次（排队）到达栅栏处，再依次往下运行。

#### 构造方法

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    // 初始化parties
    this.parties = parties;
    // 初始化count等于parties
    this.count = parties;
    // 初始化都到达栅栏处执行的命令
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

构造方法需要传入一个parties变量，也就是需要等待的线程数。

#### await()方法

每个需要在栅栏处等待的线程都需要显式地调用await()方法等待其它线程的到来。

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        // await 的逻辑封装在 dowait 中
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
    
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException, TimeoutException {
    
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        final Generation g = generation;

        // 如果 g.broken = true，表明屏障被破坏了，这里直接抛出异常
        if (g.broken)
            throw new BrokenBarrierException();

        // 如果线程中断，则调用 breakBarrier 破坏屏障
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        /*
         * index 表示线程到达屏障的顺序，index = parties - 1 表明当前线程是第一个
         * 到达屏障的。index = 0，表明当前线程是最有一个到达屏障的。
         */ 
        int index = --count;
        // 当 index = 0 时，唤醒所有处于等待状态的线程
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                // 如果回调对象不为 null，则执行回调
                if (command != null)
                    command.run();
                ranAction = true;
                // 重置屏障状态，使其进入新一轮的运行过程中
                nextGeneration();
                return 0;
            } finally {
                // 若执行回调的过程中发生异常，此时调用 breakBarrier 破坏屏障
                if (!ranAction)
                    breakBarrier();
            }
        }

        // 线程运行到此处的线程都会被屏障挡住，并进入等待状态。
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                /*
                 * 若下面的条件成立，则表明本轮运行还未结束。此时调用 breakBarrier 
                 * 破坏屏障，唤醒其他线程，并抛出异常
                 */ 
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    /*
                     * 若上面的条件不成立，则有两种可能：
                     * 1. g != generation
                     *     此种情况下，表明循环屏障的第 g 轮次的运行已经结束，屏障已经
                     *     进入了新的一轮运行轮次中。当前线程在稍后返回 到达屏障 的顺序即可
                     *     
                     * 2. g = generation 但 g.broken = true
                     *     此种情况下，表明已经有线程执行过 breakBarrier 方法了，当前
                     *     线程则会在稍后抛出 BrokenBarrierException
                     */
                    Thread.currentThread().interrupt();
                }
            }
            
            // 屏障被破坏，则抛出 BrokenBarrierException 异常
            if (g.broken)
                throw new BrokenBarrierException();

            // 屏障进入新的运行轮次，此时返回线程在上一轮次到达屏障的顺序
            if (g != generation)
                return index;

            // 超时判断
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
    
/** 开启新的一轮运行过程 */
private void nextGeneration() {
    // 唤醒所有处于等待状态中的线程
    trip.signalAll();
    // 重置 count
    count = parties;
    // 重新创建 Generation，表明进入循环屏障进入新的一轮运行轮次中
    generation = new Generation();
}

/** 破坏屏障 */
private void breakBarrier() {
    // 设置屏障是否被破坏标志
    generation.broken = true;
    // 重置 count
    count = parties;
    // 唤醒所有处于等待状态中的线程
    trip.signalAll();
}
```

dowait()方法里的整个逻辑分成两部分：

1. 最后一个线程走上面的逻辑，当count减为0的时候，打破栅栏，它调用nextGeneration()方法通知条件队列中的等待线程转移到AQS的队列中等待被唤醒，并进入下一代。
2. 非最后一个线程走下面的for循环逻辑，这些线程会阻塞在condition的await()方法处，它们会加入到条件队列中，等待被通知，当它们唤醒的时候已经更新换“代”了，这时候返回。

在上面的源代码中，我们可能需要注意Generation 对象，在上述代码中我们总是可以看到抛出BrokenBarrierException异常，那么什么时候抛出异常呢？如果一个线程处于等待状态时，如果其他线程调用reset()，或者调用的barrier原本就是被损坏的，则抛出BrokenBarrierException异常。同时，任何线程在等待时被中断了，则其他所有线程都将抛出BrokenBarrierException异常，并将barrier置于损坏状态。

同时，Generation描述着CyclicBarrier的更新换代。在CyclicBarrier中，同一批线程属于同一代。当有parties个线程到达barrier之后，generation就会被更新换代。其中broken标识该当前CyclicBarrier是否已经处于中断状态。
除了上面讲到的栅栏更新换代以及损坏状态，我们在使用CyclicBarrier时还要要注意以下几点：

- CyclicBarrier使用独占锁来执行await方法，并发性可能不是很高
- 如果在等待过程中，线程被中断了，就抛出异常。但如果中断的线程所对应的CyclicBarrier不是这代的，比如，在最后一次线程执行signalAll后，并且更新了这个“代”对象。在这个区间，这个线程被中断了，那么，JDK认为任务已经完成了，就不必在乎中断了，只需要打个标记。该部分源码已在dowait(boolean, long)方法中进行了注释。
- 如果线程被其他的CyclicBarrier唤醒了，那么g肯定等于generation，这个事件就不能return了，而是继续循环阻塞。反之，如果是当前CyclicBarrier唤醒的，就返回线程在CyclicBarrier的下标。完成了一次冲过栅栏的过程。该部分源码已在dowait(boolean, long)方法中进行了注释。
  

#### 图解

![1kw0hR.png](https://s2.ax1x.com/2020/01/21/1kw0hR.png)

#### reset

reset 方法用于强制重置屏障，使屏障进入新一轮的运行过程中。代码如下：

```java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 破坏屏障
        breakBarrier();   // break the current generation
        // 开启新一轮的运行过程
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

#### 总结

1. CyclicBarrier会使一组线程阻塞在await()处，当最后一个线程到达时唤醒（只是从条件队列转移到AQS队列中）前面的线程大家再继续往下走；
2. CyclicBarrier不是直接使用AQS实现的一个同步器；
3. CyclicBarrier基于ReentrantLock及其Condition实现整个同步逻辑；

### 参考

1. [什么时候使用CountDownLatch](http://www.importnew.com/15731.html)
2. [java.util.concurrent解析——AbstractQueuedSynchronizer综述](https://www.jianshu.com/p/c84650df924c)
3. [同步计数器·Semaphore](https://blog.csdn.net/u014203449/article/details/83957958)
4. [同步计数器](https://blog.csdn.net/u010255818/article/details/72085717)