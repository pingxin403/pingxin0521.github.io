---
title: java 线程池 基础篇
date: 2019-04-07 13:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 基础
---
### 线程池

在前面的文章中，我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。
<!--more-->

**线程池的作用：**

线程池作用就是限制系统中执行线程的数量。
     根据系统的环境情况，可以自动或手动设置线程数量，达到运行的最佳效果；少了浪费了系统资源，多了造成系统拥挤效率不高。用线程池控制线程数量，其他线程排队等候。一个任务执行完毕，再从队列的中取最前面的任务开始执行。若队列中没有等待进程，线程池的这一资源处于等待。当一个新任务需要运行时，如果线程池中有等待的工作线程，就可以开始运行了；否则进入等待队列。

**为什么要用线程池:**

1.减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。

2.可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。

![1ikMIx.png](https://s2.ax1x.com/2020/01/20/1ikMIx.png)

如上图，最顶层的接口 Executor 仅声明了一个方法`execute`。ExecutorService 接口在其父类接口基础上，声明了包含但不限于`shutdown`、`submit`、`invokeAll`、`invokeAny` 等方法。至于 ScheduledExecutorService 接口，则是声明了一些和定时任务相关的方法，比如 `schedule`和`scheduleAtFixedRate`。线程池的核心实现是在 ThreadPoolExecutor 类中，我们使用 Executors 调用`newFixedThreadPool`、`newSingleThreadExecutor`和`newCachedThreadPool`等方法创建线程池均是 ThreadPoolExecutor 类型。

比较重要的几个类：

| ExecutorService             | 真正的线程池接口。                                           |
| --------------------------- | ------------------------------------------------------------ |
| ScheduledExecutorService    | 能和Timer/TimerTask类似，解决那些需要任务重复执行的问题。    |
| ThreadPoolExecutor          | ExecutorService的默认实现。                                  |
| ScheduledThreadPoolExecutor | 继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现。 |

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在Executors类里面提供了一些静态工厂，生成一些常用的线程池。

| 静态构造方法                             | 说明                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| newFixedThreadPool(int nThreads)         | 构建包含固定线程数的线程池，默认情况下，空闲线程不会被回收   |
| newCachedThreadPool()                    | 构建线程数不定的线程池，线程数量随任务量变动，空闲线程存活时间超过60秒后会被回收 |
| newSingleThreadExecutor()                | 构建线程数为1的线程池，等价于 newFixedThreadPool(1) 所构造出的线程池 |
| newScheduledThreadPool(int corePoolSize) | 构建核心线程数为 corePoolSize，可执行定时任务的线程池        |
| newSingleThreadScheduledExecutor()       | 等价于 newScheduledThreadPool(1)                             |

1. newSingleThreadExecutor

   创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

   ```java
   public static ExecutorService newSingleThreadExecutor() {   
               return new FinalizableDelegatedExecutorService   
                    (new ThreadPoolExecutor(1, 1,   
                                          0L, TimeUnit.MILLISECONDS,   
                                         new LinkedBlockingQueue<Runnable>()));   
          }
   ```

2. newFixedThreadPool

   创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

   可以看到，corePoolSize和maximumPoolSize的大小是一样的（实际上，后面会介绍，如果使用无界queue的话maximumPoolSize参数是没有意义的），keepAliveTime和unit的设值表名什么？-就是该实现不想keep
   alive！最后的BlockingQueue选择了LinkedBlockingQueue，该queue有一个特点，他是无界的。

   ```java
   public static ExecutorService newFixedThreadPool(int nThreads) {   
              return new ThreadPoolExecutor(nThreads, nThreads,   
                                           0L, TimeUnit.MILLISECONDS,   
                                           new LinkedBlockingQueue<Runnable>());   
         }
   ```

   

3. newCachedThreadPool

   创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，

   那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

   这个实现就有意思了。首先是无界的线程池，所以我们可以发现maximumPoolSize为big big。其次BlockingQueue的选择上使用SynchronousQueue。可能对于该BlockingQueue有些陌生，简单说：该QUEUE中，每个插入操作必须等待另一个线程的对应移除操作。

   ```java
    public static ExecutorService newCachedThreadPool() {   
                return new ThreadPoolExecutor(0, Integer.MAX_VALUE,   
                                            60L, TimeUnit.SECONDS,   
                                           new SynchronousQueue<Runnable>());   
       }
   ```

    先从BlockingQueue<Runnable\> workQueue这个入参开始说起。在JDK中，其实已经说得很清楚了，一共有三种类型的queue。

   所有BlockingQueue 都可用于传输和保持提交的任务。可以使用此队列与池大小进行交互：

   - 如果运行的线程少于 corePoolSize，则 Executor始终首选添加新的线程，而不进行排队。（如果当前运行的线程小于corePoolSize，则任务根本不会存放，添加到queue中，而是直接抄家伙（thread）开始运行）

   - 如果运行的线程等于或多于 corePoolSize，则 Executor始终首选将请求加入队列，而不添加新的线程。

   - 如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。

4. newScheduledThreadPool

   创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
   
5. newSingleThreadScheduledExecutor

   创建一个只有一个线程的线程池。此线程池支持定时以及周期性执行任务的需求。

6. newWorkStealingPool

   创建ForkJoin线程池

#### ThreadPoolExecutor类

java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻地了解Java中的线程池，必须先了解这个类。
```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);

    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
        BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler
                             );
    ...
}
```
从上面的代码可以得知，ThreadPoolExecutor继承了AbstractExecutorService类，并提供了四个构造器，事实上，通过观察每个构造器的源码具体实现，发现前面三个构造器都是调用的第四个构造器进行的初始化工作。

 下面解释下一下构造器中各个参数的含义：

**corePoolSize**：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。

默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；

**maximumPoolSize：** 线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；

| 序号 | 条件                                                        | 动作             |
| :--- | :---------------------------------------------------------- | :--------------- |
| 1    | 线程数 < corePoolSize                                       | 创建新线程       |
| 2    | 线程数 ≥ corePoolSize，且 workQueue 未满                    | 缓存新任务       |
| 3    | corePoolSize ≤ 线程数 ＜ maximumPoolSize，且 workQueue 已满 | 创建新线程       |
| 4    | 线程数 ≥ maximumPoolSize，且 workQueue 已满                 | 使用拒绝策略处理 |

**keepAliveTime：** 表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

**unit：** 参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

```
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒
```
**workQueue：** 一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：
```
ArrayBlockingQueue;
LinkedBlockingQueue;
SynchronousQueue;
```
ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和Synchronous。线程池的排队策略与BlockingQueue有关。

| 实现类                | 类型       | 说明                                                         |
| :-------------------- | :--------- | :----------------------------------------------------------- |
| SynchronousQueue      | 同步队列   | 该队列不存储元素，每个插入操作必须等待另一个线程调用移除操作，否则插入操作会一直阻塞 |
| ArrayBlockingQueue    | 有界队列   | 基于数组的阻塞队列，按照 FIFO 原则对元素进行排序             |
| LinkedBlockingQueue   | 无界队列   | 基于链表的阻塞队列，按照 FIFO 原则对元素进行排序             |
| PriorityBlockingQueue | 优先级队列 | 具有优先级的阻塞队列                                         |

**threadFactory：** 线程工厂，主要用来创建线程；

**handler：** 表示当拒绝处理任务时的策略，有以下四种取值：

```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```

从上面给出的ThreadPoolExecutor类的代码可以知道，ThreadPoolExecutor继承了AbstractExecutorService，我们来看一下AbstractExecutorService的实现：
```java
public abstract class AbstractExecutorService implements ExecutorService {


    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) { };
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) { };
    public Future<?> submit(Runnable task) {};
    public <T> Future<T> submit(Runnable task, T result) { };
    public <T> Future<T> submit(Callable<T> task) { };
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                            boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
    };
    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
    };
    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
    };
}
```
AbstractExecutorService是一个抽象类，它实现了ExecutorService接口。

```java
public interface ExecutorService extends Executor {

    void shutdown();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

而ExecutorService又是继承了Executor接口。
```java
public interface Executor {
    void execute(Runnable command);
}
```

Executor是一个顶层接口，在它里面只声明了一个方法execute(Runnable)，返回值为void，参数为Runnable类型，从字面意思可以理解，就是用来执行传进去的任务的；

然后ExecutorService接口继承了Executor接口，并声明了一些方法：submit、invokeAll、invokeAny以及shutDown等；

抽象类AbstractExecutorService实现了ExecutorService接口，基本实现了ExecutorService中声明的所有方法；

然后ThreadPoolExecutor继承了类AbstractExecutorService。

在ThreadPoolExecutor类中有几个非常重要的方法：
```java
execute()
submit()
shutdown()
shutdownNow()
```

execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果（Future相关内容将在下一篇讲述）。

shutdown()和shutdownNow()是用来关闭线程池的。

还有很多其他的方法，比如：getQueue() 、getPoolSize() 、getActiveCount()、getCompletedTaskCount()等获取与线程池相关属性的方法，有兴趣的朋友可以自行查阅API。

##### queue上的三种类型。 

排队有三种通用策略：

**直接提交。**工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

**无界队列。**使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

**有界队列。**当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。  

#### 深入剖析线程池实现原理

##### 线程池状态

在ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final变量表示线程池的各个状态：

```java
volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
```
runState表示当前线程池的状态，它是一个volatile变量用来保证线程之间的可见性；下面的几个static final变量表示runState可能的几个取值。

当创建线程池后，初始时，线程池处于RUNNING状态；如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

##### 任务的执行

在了解将任务提交给线程池到任务执行完毕整个过程之前，我们先来看一下ThreadPoolExecutor类中其他的一些比较重要成员变量：
```java
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小
                                                              //、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集

private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数

private volatile int   poolSize;       //线程池中当前的线程数

private volatile RejectedExecutionHandler handler; //任务拒绝策略

private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程

private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数

private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

在ThreadPoolExecutor类中，最核心的任务提交方法是execute()方法，虽然通过submit也可以提交任务，但是实际上submit方法里面最终调用的还是execute()方法，所以我们只需要研究execute()方法的实现原理即可：

```java
+---- AbstractExecutorService.java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    // 创建任务
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    // 提交任务
    execute(ftask);
    return ftask;
}

+---- ThreadPoolExecutor.java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();
    // 如果工作线程数量 < 核心线程数，则创建新线程
    if (workerCountOf(c) < corePoolSize) {
        // 添加工作者对象
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    
    // 缓存任务，如果队列已满，则 offer 方法返回 false。否则，offer 返回 true
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    
    // 添加工作者对象，并在 addWorker 方法中检测线程数是否小于最大线程数
    else if (!addWorker(command, false))
        // 线程数 >= 最大线程数，使用拒绝策略处理任务
        reject(command);
}

private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            // 检测工作线程数与核心线程数或最大线程数的关系
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 创建工作者对象，细节参考上一节所贴代码
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int rs = runStateOf(ctl.get());
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // 将 worker 对象添加到 workers 集合中
                    workers.add(w);
                    int s = workers.size();
                    // 更新 largestPoolSize 属性
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 开始执行任务
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
可以使用下图表示：

![](https://i.loli.net/2019/04/16/5cb59e27d0e3f.png)


1）首先，要清楚corePoolSize和maximumPoolSize的含义；

2）其次，要知道Worker是用来起到什么作用的；

它实际上实现了Runnable接口,既然Worker实现了Runnable接口，那么自然最核心的方法便是run()方法了。从run方法的实现可以看出，它首先执行的是通过构造器传进来的任务firstTask，在调用runTask()执行完firstTask之后，在while循环里面不断通过getTask()去取新的任务来执行，那么去哪里取呢？自然是从任务缓存队列里面去取，getTask是ThreadPoolExecutor类中的方法，并不是Worker类中的方法

3）要知道任务提交给线程池之后的处理策略，这里总结一下主要有4点：

- 如果当前线程池中的线程数目小于corePoolSize，则每来一个任务，就会**创建**一个线程去执行这个任务；

- 如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其**添加到任务缓存队列**当中，若添加成功，则该任务会等待空闲线程将其取出去执行；若添加失败（一般来说是任务缓存队列已满），则会尝试**创建新的线程**去执行这个任务；

- 如果当前线程池中的线程数目达到maximumPoolSize，则会采取**任务拒绝策略**进行处理；

- 如果线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被**终止**，直至线程池中的线程数目不大于corePoolSize；如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过keepAliveTime，线程也会被**终止**。

##### 线程池中的线程初始化

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。

在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：
```
prestartCoreThread()：初始化一个核心线程；
prestartAllCoreThreads()：初始化所有核心线程
```
下面是这2个方法的实现：
```java
public boolean prestartCoreThread() {
    return addIfUnderCorePoolSize(null); //注意传进去的参数是null
}

public int prestartAllCoreThreads() {
    int n = 0;
    while (addIfUnderCorePoolSize(null))//注意传进去的参数是null
        ++n;
    return n;
}

```
注意上面传进去的参数是null，根据第2小节的分析可知如果传进去的参数为null，则最后执行线程会阻塞在getTask方法中的
```java
r = workQueue.take();
```
即等待任务队列中有任务。

##### 任务缓存队列及排队策略

在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

workQueue的类型为BlockingQueue<Runnable\>，通常可以取下面三种类型：

1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

线程池运行如下：

![](https://i.loli.net/2019/04/16/5cb59e27ba38e.png)


##### 任务拒绝策略
当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：
```
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```
##### 线程池的关闭

ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：
```
shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务
```
##### 线程池容量的动态调整

ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，
```
setCorePoolSize：设置核心池大小
setMaximumPoolSize：设置线程池最大能创建的线程数目大小
```
当上述参数从小变大时，ThreadPoolExecutor进行线程赋值，还可能立即创建新的线程来执行任务。

#### 重要操作

##### 线程的创建与复用

在线程池的实现上，线程的创建是通过线程工厂接口`ThreadFactory`的实现类来完成的。默认情况下，线程池使用`Executors.defaultThreadFactory()`方法返回的线程工厂实现类。当然，我们也可以通过
`public void setThreadFactory(ThreadFactory)`方法进行动态修改。具体细节这里就不多说了，并不复杂，大家可以自己去看下源码。

在线程池中，线程的复用是线程池的关键所在。这就要求线程在执行完一个任务后，不能立即退出。对应到具体实现上，工作线程在执行完一个任务后，会再次到任务队列获取新的任务。如果任务队列中没有任务，且 keepAliveTime 也未被设置，工作线程则会被一致阻塞下去。通过这种方式即可实现线程复用。

说完原理，再来看看线程的创建和复用的相关代码（基于 JDK 1.8），如下：

```java
// Worker继承自AQS，自带锁的属性
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    // 真正工作的线程
    final Thread thread;
    // 第一个任务，从构造方法传进来
    Runnable firstTask;
    // 完成任务数
    volatile long completedTasks;

    // 构造方法 
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 使用线程工厂生成一个线程
        // 注意，这里把Worker本身作为Runnable传给线程
        this.thread = getThreadFactory().newThread(this);
    }

    // 实现Runnable的run()方法
    public void run() {
        // 调用ThreadPoolExecutor的runWorker()方法
        runWorker(this);
    }
    
    // 省略锁的部分
}

+----ThreadPoolExecutor.java
final void runWorker(Worker w) {
    // 工作线程
    Thread wt = Thread.currentThread();
    // 任务
    Runnable task = w.firstTask;
    w.firstTask = null;
    // 强制释放锁(shutdown()里面有加锁)
    // 这里相当于无视那边的中断标记
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 取任务，如果有第一个任务，这里先执行第一个任务
        // 只要能取到任务，这就是个死循环
        // 正常来说getTask()返回的任务是不可能为空的，因为前面execute()方法是有空判断的
        // 那么，getTask()什么时候才会返回空任务呢？
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // 检查线程池的状态
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            
            try {
                // 钩子方法，方便子类在任务执行前做一些处理
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 真正任务执行的地方
                    task.run();
                    // 异常处理
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 钩子方法，方便子类在任务执行后做一些处理
                    afterExecute(task, thrown);
                }
            } finally {
                // task置为空，重新从队列中取
                task = null;
                // 完成任务数加1
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 到这里肯定是上面的while循环退出了
        processWorkerExit(w, completedAbruptly);
    }
}
private Runnable getTask() {
    // 是否超时
    boolean timedOut = false;
    
    // 死循环
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 线程池状态是SHUTDOWN的时候会把队列中的任务执行完直到队列为空
        // 线程池状态是STOP时立即退出
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        
        // 工作线程数量//  
        int wc = workerCountOf(c);

        // 是否允许超时，有两种情况：
        // 1. 是允许核心线程数超时，这种就是说所有的线程都可能超时
        // 2. 是工作线程数大于了核心数量，这种肯定是允许超时的
        // 注意，非核心线程是一定允许超时的，这里的超时其实是指取任务超时
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // 超时判断（还包含一些容错判断）
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            // 超时了，减少工作线程数量，并返回null
            if (compareAndDecrementWorkerCount(c))
                return null;
            // 减少工作线程数量失败，则重试
            continue;
        }

        try {
            // 真正取任务的地方
            // 默认情况下，只有当工作线程数量大于核心线程数量时，才会调用poll()方法触发超时调用
            
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            
            // 取到任务了就正常返回
            if (r != null)
                return r;
            // 没取到任务表明超时了，回到continue那个if中返回null
            timedOut = true;
        } catch (InterruptedException retry) {
            // 捕获到了中断异常
            // 中断标记是在调用shutDown()或者shutDownNow()的时候设置进去的
            // 此时，会回到for循环的第一个if处判断状态是否要返回null
            timedOut = false;
        }
    }
}
```

1. execute()，提交任务的方法，根据核心数量、任务队列大小、最大数量，分成四种情况判断任务应该往哪去；
2. addWorker()，添加工作线程的方法，通过Worker内部类封装一个Thread实例维护工作线程的执行；
3. runWorker()，真正执行任务的地方，先执行第一个任务，再源源不断从任务队列中取任务来执行；
4. getTask()，真正从队列取任务的地方，默认情况下，根据工作线程数量与核心数量的关系判断使用队列的poll()还是take()方法，keepAliveTime参数也是在这里使用的

##### 关闭线程池

我们可以通过`shutdown`和`shutdownNow`两个方法关闭线程池。两个方法的区别在于，shutdown 会将线程池的状态设置为`SHUTDOWN`，同时该方法还会中断空闲线程。shutdownNow 则会将线程池状态设置为`STOP`，并尝试中断所有的线程。中断线程使用的是`Thread.interrupt`方法，未响应中断方法的任务是无法被中断的。最后，shutdownNow 方法会将未执行的任务全部返回。

调用 shutdown 和 shutdownNow 方法关闭线程池后，就不能再向线程池提交新任务了。对于处于关闭状态的线程池，会使用拒绝策略处理新提交的任务。

####  示例

我们创建一个线程池，它的核心数量为5，最大数量为10，空闲时间为1秒，队列长度为5，拒绝策略打印一句话。

如果使用它运行20个任务，会是什么结果呢？

```java
public class Test {
     public static void main(String[] args) {   
         ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 200, TimeUnit.MILLISECONDS,
                 new ArrayBlockingQueue<Runnable>(5));

         for(int i=0;i<20;i++){
             MyTask myTask = new MyTask(i);
             executor.execute(myTask);
             System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
             executor.getQueue().size()+"，已执行的任务数目："+executor.getCompletedTaskCount());
         }
         executor.shutdown();
     }
}


class MyTask implements Runnable {
    private int taskNum;

    public MyTask(int num) {
        this.taskNum = num;
    }

    @Override
    public void run() {
        System.out.println("正在执行task "+taskNum);
        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task "+taskNum+"执行完毕");
    }
}
```
执行结果：
```
正在执行task 0
线程池中线程数目：1，队列中等待执行的任务数目：0，已执行的任务数目：0
线程池中线程数目：2，队列中等待执行的任务数目：0，已执行的任务数目：0
正在执行task 1
线程池中线程数目：3，队列中等待执行的任务数目：0，已执行的任务数目：0
线程池中线程数目：4，队列中等待执行的任务数目：0，已执行的任务数目：0
正在执行task 3
线程池中线程数目：5，队列中等待执行的任务数目：0，已执行的任务数目：0
正在执行task 4
线程池中线程数目：5，队列中等待执行的任务数目：1，已执行的任务数目：0
线程池中线程数目：5，队列中等待执行的任务数目：2，已执行的任务数目：0
线程池中线程数目：5，队列中等待执行的任务数目：3，已执行的任务数目：0
线程池中线程数目：5，队列中等待执行的任务数目：4，已执行的任务数目：0
线程池中线程数目：5，队列中等待执行的任务数目：5，已执行的任务数目：0
线程池中线程数目：6，队列中等待执行的任务数目：5，已执行的任务数目：0
线程池中线程数目：7，队列中等待执行的任务数目：5，已执行的任务数目：0
正在执行task 11
正在执行task 10
线程池中线程数目：8，队列中等待执行的任务数目：5，已执行的任务数目：0
正在执行task 2
线程池中线程数目：9，队列中等待执行的任务数目：5，已执行的任务数目：0
正在执行task 12
正在执行task 13
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
正在执行task 14
main, discard task
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
main, discard task
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
main, discard task
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
main, discard task
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
main, discard task
线程池中线程数目：10，队列中等待执行的任务数目：5，已执行的任务数目：0
task 0执行完毕
正在执行task 5
task 1执行完毕
正在执行task 6
task 3执行完毕
正在执行task 7
task 4执行完毕
正在执行task 8
task 11执行完毕
正在执行task 9
task 10执行完毕
task 2执行完毕
task 13执行完毕
task 12执行完毕
task 14执行完毕
task 5执行完毕
task 6执行完毕
task 7执行完毕
task 8执行完毕
task 9执行完毕
```

注意，观察num值的打印信息，先是打印了0~4，再打印了10~14，最后打印了5~9，竟然不是按顺序打印的，为什么呢？

线程池的参数：核心数量5个，最大数量10个，任务队列5个。

答：执行前5个任务执行时，正好还不到核心数量，所以新建核心线程并执行了他们；

执行中间的5个任务时，已达到核心数量，所以他们先入队列；

执行后面5个任务时，已达核心数量且队列已满，所以新建非核心线程并执行了他们；

再执行最后5个任务时，线程池已达到满负荷状态，所以执行了拒绝策略。

不过在java doc中，并不提倡我们直接使用ThreadPoolExecutor，而是使用Executors类中提供的几个静态方法来创建线程池：

```java
Executors.newCachedThreadPool();        //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
Executors.newSingleThreadExecutor();   //创建容量为1的缓冲池
Executors.newFixedThreadPool(int);    //创建固定容量大小的缓冲池
```
下面是这三个静态方法的具体实现;
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
从它们的具体实现来看，它们实际上也是调用了ThreadPoolExecutor，只不过参数都已配置好了。
```
newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；

newSingleThreadExecutor将corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；

newCachedThreadPool将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。
```
实际中，如果Executors提供的三个静态方法能满足要求，就尽量使用它提供的三个方法，因为自己去手动配置ThreadPoolExecutor的参数有点麻烦，要根据实际任务的类型和数量来进行配置。

另外，如果ThreadPoolExecutor达不到要求，可以自己继承ThreadPoolExecutor类进行重写。

**如何合理配置线程池的大小**

一般需要根据任务的类型来配置线程池大小：

如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1

如果是IO密集型任务，参考值可以设置为2*NCPU

当然，这只是一个参考值，具体的设置还需要根据实际情况进行调整，比如可以先将线程池大小设置为参考值，再观察任务运行情况和系统负载、资源利用率来进行适当调整。

### 生命周期

其实，在我们讲线程池体系结构的时候，讲了一些方法，比如shutDown()/shutDownNow()，它们都是与线程池的生命周期相关联的。

我们先来看一下线程池ThreadPoolExecutor中定义的生命周期中的状态及相关方法：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3; // =29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1; // =000 11111...

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS; // 111 00000...
private static final int SHUTDOWN   =  0 << COUNT_BITS; // 000 00000...
private static final int STOP       =  1 << COUNT_BITS; // 001 00000...
private static final int TIDYING    =  2 << COUNT_BITS; // 010 00000...
private static final int TERMINATED =  3 << COUNT_BITS; // 011 00000...

// 线程池的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
// 线程池中工作线程的数量
private static int workerCountOf(int c)  { return c & CAPACITY; }
// 计算ctl的值，等于运行状态“加上”线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

从上面这段代码，我们可以得出：

1. 线程池的状态和工作线程的数量共同保存在控制变量ctl中，类似于AQS中的state变量，不过这里是直接使用的AtomicInteger，这里换成unsafe+volatile也是可以的；
2. ctl的高三位保存运行状态，低29位保存工作线程的数量，也就是说线程的数量最多只能有(2^29-1)个，也就是上面的CAPACITY；
3. 线程池的状态一共有五种，分别是RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED；
4. RUNNING，表示可接受新任务，且可执行队列中的任务；
5. SHUTDOWN，表示不接受新任务，但可执行队列中的任务；
6. STOP，表示不接受新任务，且不再执行队列中的任务，且中断正在执行的任务；
7. TIDYING，所有任务已经中止，且工作线程数量为0，最后变迁到这个状态的线程将要执行terminated()钩子方法，只会有一个线程执行这个方法；
8. TERMINATED，中止状态，已经执行完terminated()钩子方法；

#### 流程图

下面我们再来看看这些状态之间是怎么流转的：

![1i4ObR.png](https://s2.ax1x.com/2020/01/20/1i4ObR.png)

1. 新建线程池时，它的初始状态为RUNNING，这个在上面定义ctl的时候可以看到；
2. RUNNING->SHUTDOWN，执行shutdown()方法时；
3. RUNNING->STOP，执行shutdownNow()方法时；
4. SHUTDOWN->STOP，执行shutdownNow()方法时
5. STOP->TIDYING，执行了shutdown()或者shutdownNow()后，所有任务已中止，且工作线程数量为0时，此时会执行terminated()方法；
6. TIDYING->TERMINATED，执行完terminated()方法后；

#### 源码分析

1. RUNNING

   RUNNING，比较简单，创建线程池的时候就会初始化ctl，而ctl初始化为RUNNING状态，所以线程池的初始状态就为RUNNING状态。

   ```java
   // 初始状态为RUNNING
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   ```

2. SHUTDOWN

   执行shutdown()方法时把状态修改为SHUTDOWN，这里肯定会成功，因为advanceRunState()方法中是个自旋，不成功不会退出。

   ```java
   public void shutdown() {
       final ReentrantLock mainLock = this.mainLock;
       mainLock.lock();
       try {
           checkShutdownAccess();
           // 修改状态为SHUTDOWN
           advanceRunState(SHUTDOWN);
           // 标记空闲线程为中断状态
           interruptIdleWorkers();
           onShutdown();
       } finally {
           mainLock.unlock();
       }
       tryTerminate();
   }
   private void advanceRunState(int targetState) {
       for (;;) {
           int c = ctl.get();
           // 如果状态大于SHUTDOWN，或者修改为SHUTDOWN成功了，才会break跳出自旋
           if (runStateAtLeast(c, targetState) ||
               ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
               break;
       }
   }
   ```

3. STOP

   执行shutdownNow()方法时，会把线程池状态修改为STOP状态，同时标记所有线程为中断状态。

   ```java
   public List<Runnable> shutdownNow() {
       List<Runnable> tasks;
       final ReentrantLock mainLock = this.mainLock;
       mainLock.lock();
       try {
           checkShutdownAccess();
           // 修改为STOP状态
           advanceRunState(STOP);
           // 标记所有线程为中断状态
           interruptWorkers();
           tasks = drainQueue();
       } finally {
           mainLock.unlock();
       }
       tryTerminate();
       return tasks;
   }
   ```

   至于线程是否响应中断其实是在队列的take()或poll()方法中响应的，最后会到AQS中，它们检测到线程中断了会抛出一个InterruptedException异常，然后getTask()中捕获这个异常，并且在下一次的自旋时退出当前线程并减少工作线程的数量。

   ```java
   private Runnable getTask() {
       boolean timedOut = false; // Did the last poll() time out?
   
       for (;;) {
           int c = ctl.get();
           int rs = runStateOf(c);
   
           // 如果状态为STOP了，这里会直接退出循环，且减少工作线程数量
           // 退出循环了也就相当于这个线程的生命周期结束了
           if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
               decrementWorkerCount();
               return null;
           }
   
           int wc = workerCountOf(c);
   
           // Are workers subject to culling?
           boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
   
           if ((wc > maximumPoolSize || (timed && timedOut))
               && (wc > 1 || workQueue.isEmpty())) {
               if (compareAndDecrementWorkerCount(c))
                   return null;
               continue;
           }
   
           try {
               // 真正响应中断是在poll()方法或者take()方法中
               Runnable r = timed ?
                   workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                   workQueue.take();
               if (r != null)
                   return r;
               timedOut = true;
           } catch (InterruptedException retry) {
               // 这里捕获中断异常
               timedOut = false;
           }
       }
   }
   ```

   这里有一个问题，就是已经通过getTask()取出来且返回的任务怎么办？

   实际上它们会正常执行完毕，有兴趣的同学可以自己看看runWorker()这个方法

4. TIDYING

   当执行shutdown()或shutdownNow()之后，如果所有任务已中止，且工作线程数量为0，就会进入这个状态。

   ```java
   final void tryTerminate() {
       for (;;) {
           int c = ctl.get();
           // 下面几种情况不会执行后续代码
           // 1. 运行中
           // 2. 状态的值比TIDYING还大，也就是TERMINATED
           // 3. SHUTDOWN状态且任务队列不为空
           if (isRunning(c) ||
               runStateAtLeast(c, TIDYING) ||
               (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
               return;
           // 工作线程数量不为0，也不会执行后续代码
           if (workerCountOf(c) != 0) {
               // 尝试中断空闲的线程
               interruptIdleWorkers(ONLY_ONE);
               return;
           }
   
           final ReentrantLock mainLock = this.mainLock;
           mainLock.lock();
           try {
               // CAS修改状态为TIDYING状态
               if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                   try {
                       // 更新成功，执行terminated钩子方法
                       terminated();
                   } finally {
                       // 强制更新状态为TERMINATED，这里不需要CAS了
                       ctl.set(ctlOf(TERMINATED, 0));
                       termination.signalAll();
                   }
                   return;
               }
           } finally {
               mainLock.unlock();
           }
           // else retry on failed CAS
       }
   }
   ```

   实际更新状态为TIDYING和TERMINATED状态的代码都在tryTerminate()方法中，实际上tryTerminated()方法在很多地方都有调用，比如shutdown()、shutdownNow()、线程退出时，所以说几乎每个线程最后消亡的时候都会调用tryTerminate()方法，但最后只会有一个线程真正执行到修改状态为TIDYING的地方。

   修改状态为TIDYING后执行terminated()方法，最后修改状态为TERMINATED，标志着线程池真正消亡了。

5. TERMINATED

   见TIDYING中分析。

### 参考：

1. [Java并发编程：线程池的使用](https://www.cnblogs.com/dolphin0520/p/3932921.html)
2. [必须要理清的Java线程池（原创）](https://www.jianshu.com/p/50fffbf21b39)
3. [死磕 java线程系列之自己动手写一个线程池](https://www.cnblogs.com/tong-yuan/p/11639269.html)

