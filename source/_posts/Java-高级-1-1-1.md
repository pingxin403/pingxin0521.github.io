---
title: Java  Thread 同步和ThreadLocal原理
date: 2019-05-05 08:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---
### 同步

**为何要使用同步？**

java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查），将会导致数据不准确，相互之间产生冲突，因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用，从而保证了该变量的唯一性和准确性。

<!--more-->

**同步方法**

即有synchronized关键字修饰的方法。由于java的每个对象都有一个内置锁，当用此关键字修饰方法时，内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。

代码如：
```
public synchronized void save(){}
```
>注： synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类


**同步代码块**

即有synchronized关键字修饰的语句块。被该关键字修饰的语句块会自动被加上内置锁，从而实现同步

代码如：
```
synchronized(object){
}
```

>注：同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。

代码实例：
```java
public class SynchronizedThread {

    class Bank {

        private int account = 100;

        public int getAccount() {
            return account;
        }

        /**
         * 用同步方法实现
         *
         * @param money
         */
        public synchronized void save(int money) {
            account += money;
        }

        /**
         * 用同步代码块实现
         *
         * @param money
         */
        public void save1(int money) {
            synchronized (this) {
                account += money;
            }
        }
    }

    class NewThread implements Runnable {
        private Bank bank;

        public NewThread(Bank bank) {
            this.bank = bank;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                // bank.save1(10);
                bank.save(10);
                System.out.println(i + "账户余额为：" + bank.getAccount());
            }
        }

    }

    /**
     * 建立线程，调用内部类
     */
    public void useThread() {
        Bank bank = new Bank();
        NewThread new_thread = new NewThread(bank);
        System.out.println("线程1");
        Thread thread1 = new Thread(new_thread);
        thread1.start();
        System.out.println("线程2");
        Thread thread2 = new Thread(new_thread);
        thread2.start();
    }

    public static void main(String[] args) {
        SynchronizedThread st = new SynchronizedThread();
        st.useThread();
    }

}
```

**使用特殊域变量(volatile)实现线程同步**

a.volatile关键字为域变量的访问提供了一种免锁机制，

b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，

c.因此每次使用该域就要重新计算，而不是使用寄存器中的值

d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量

例如：在上面的例子当中，只需在account前面加上volatile修饰，即可实现线程同步。

代码实例：
```java
//只给出要修改的代码，其余代码与上同
        class Bank {
            //需要同步的变量加上volatile
            private volatile int account = 100;

            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized
            public void save(int money) {
                account += money;
            }
        ｝
```

>注：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。用final域，有锁保护的域和volatile域可以避免非同步的问题。

**使用重入锁实现线程同步**

在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力

ReentrantLock类的常用方法有：
```
ReentrantLock() : 创建一个ReentrantLock实例
lock() : 获得锁
unlock() : 释放锁
```

>注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用

例如：在上面例子的基础上，改写后的代码为:

代码实例：
```java
//只给出要修改的代码，其余代码与上同
        class Bank {

            private int account = 100;
            //需要声明这个锁
            private Lock lock = new ReentrantLock();
            public int getAccount() {
                return account;
            }
            //这里不再需要synchronized
            public void save(int money) {
                lock.lock();
                try{
                    account += money;
                }finally{
                    lock.unlock();
                }

            }
        ｝
```

>注：关于Lock对象和synchronized关键字的选择：
>a.最好两个都不用，使用一种java.util.concurrent包提供的机制，能够帮助用户处理所有与锁相关的代码。
>b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码
>c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁

**使用局部变量实现线程同步**

如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。

ThreadLocal 类的常用方法

```
ThreadLocal() : 创建一个线程本地变量
get() : 返回此线程局部变量的当前线程副本中的值
initialValue() : 返回此线程局部变量的当前线程的"初始值"
set(T value) : 将此线程局部变量的当前线程副本中的值设置为value
```

例如：在上面例子基础上，修改后的代码为：

代码实例：

```java
//只改Bank类，其余代码与上同
        public class Bank{
            //使用ThreadLocal类管理共享变量account
            private static ThreadLocal<Integer> account = new ThreadLocal<Integer>(){
                @Override
                protected Integer initialValue(){
                    return 100;
                }
            };
            public void save(int money){
                account.set(account.get()+money);
            }
            public int getAccount(){
                return account.get();
            }
        }
```

>注：ThreadLocal与同步机制
>a.ThreadLocal与同步机制都是为了解决多线程中相同变量的访问冲突问题。
>b.前者采用以"空间换时间"的方法，后者采用以"时间换空间"的方式

**使用阻塞队列实现线程同步**

前面5种同步方式都是在底层实现的线程同步，但是我们在实际开发当中，应当尽量远离底层结构。 使用javaSE5.0版本中新增的java.util.concurrent包将有助于简化开发。

本小节主要是使用LinkedBlockingQueue<E\>来实现线程的同步 ,LinkedBlockingQueue<E\>是一个基于已连接节点的，范围任意的blocking queue。

LinkedBlockingQueue 类常用方法

```
LinkedBlockingQueue() : 创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue
put(E e) : 在队尾添加一个元素，如果队列满则阻塞
size() : 返回队列中的元素个数
take() : 移除并返回队头元素，如果队列空则阻塞
```

代码实例： 实现商家生产商品和买卖商品的同步

```java
import java.util.Random;
import java.util.concurrent.LinkedBlockingQueue;

public class BlockingSynchronizedThread {
    /**
     * 定义一个阻塞队列用来存储生产出来的商品
     */
    private LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer>();
    /**
     * 定义生产商品个数
     */
    private static final int size = 10;
    /**
     * 定义启动线程的标志，为0时，启动生产商品的线程；为1时，启动消费商品的线程
     */
    private int flag = 0;

    private class LinkBlockThread implements Runnable {
        @Override
        public void run() {
            int new_flag = flag++;
            System.out.println("启动线程 " + new_flag);
            if (new_flag == 0) {
                for (int i = 0; i < size; i++) {
                    int b = new Random().nextInt(255);
                    System.out.println("生产商品：" + b + "号");
                    try {
                        queue.put(b);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    System.out.println("仓库中还有商品：" + queue.size() + "个");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            } else {
                for (int i = 0; i < size / 2; i++) {
                    try {
                        int n = queue.take();
                        System.out.println("消费者买去了" + n + "号商品");
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    System.out.println("仓库中还有商品：" + queue.size() + "个");
                    try {
                        Thread.sleep(100);
                    } catch (Exception e) {
                        // TODO: handle exception
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        BlockingSynchronizedThread bst = new BlockingSynchronizedThread();
        LinkBlockThread lbt = bst.new LinkBlockThread();
        Thread thread1 = new Thread(lbt);
        Thread thread2 = new Thread(lbt);
        thread1.start();
        thread2.start();
    }
}
```
>注：BlockingQueue<E\>定义了阻塞队列的常用方法，尤其是三种添加元素的方法，我们要多加注意，当队列满时：add()方法会抛出异常;offer()方法返回false;put()方法会阻塞

**使用原子变量实现线程同步**

需要使用线程同步的根本原因在于对普通变量的操作不是原子的。那么什么是原子操作呢？

原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作。即-这几种行为要么同时完成，要么都不完成。

在java的util.concurrent.atomic包中提供了创建了原子类型变量的工具类，使用该类可以简化线程同步。其中AtomicInteger 表可以用原子方式更新int的值，可用在应用程序中(如以原子方式增加的计数器)，但不能用于替换Integer；可扩展Number，允许那些处理机遇数字类的工具和实用工具进行统一访问。

AtomicInteger类常用方法：

```
AtomicInteger(int initialValue) : 创建具有给定初始值的新的AtomicInteger
addAddGet(int dalta) : 以原子方式将给定值与当前值相加
get() : 获取当前值
```

代码实例：只改Bank类，其余代码与上面第一个例子相同

```
class Bank {
        private AtomicInteger account = new AtomicInteger(100);

        public AtomicInteger getAccount() {
            return account;
        }

        public void save(int money) {
            account.addAndGet(money);
        }
    }
```

>补充--原子操作主要有：对于引用变量和大多数原始变量(long和double除外)的读写操作；对于所有使用volatile修饰的变量(包括long和double)的读写操作。

### ThreadLocal

ThreadLocal是一个**本地线程副本变量**工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，**各个线程之间的变量互不干扰**，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不同的变量值完成操作的场景。

ThreadLocal提供了线程独有的局部变量，可以在整个线程存活的过程中随机取用，极大地方便了一些逻辑的实现。常见的ThreadLocal方法有：

- 存储单个线程上下文信息。比如存储id等

- 使变量线程安全。变量既然成为了每个线程的局部变量，自然就不会存在并发问题。

- 减少参数传递。比如做一个trace工具，能够输出工程从开始到结束的升格一次处理过程中的所有信息，从而方便debug。由于在工程各处随时取用，可放入ThreadLocal。

ThreadLocal每个线程维护一个 ThreadLocalMap 的映射表，映射表的 key 是 ThreadLocal 实例本身，value 是要存储的副本变量。ThreadLocal 实例本身并不存储值，它只是提供一个在当前线程中找到副本值的 key。

![lJrx0J.png](https://s2.ax1x.com/2020/01/01/lJrx0J.png)

我们从下面三个方面看下 ThreadLocal 的实现：

- 存储线程副本变量的数据结构
- 如何存取线程副本变量
- 如何对 ThreadLocal 的实例进行 Hash

#### 源码

**set方法**

```java
//ThreadLocal.set
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
//ThreadLocalMap.getMap
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
//ThreadLocalMap.createMap
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
//ThreadLocalMap.set
private void set(ThreadLocal <?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    // 根据 ThreadLocal 的散列值，查找对应元素在数组中的位置
    int i = key.threadLocalHashCode & (len - 1);

    // 使用线性探测法查找元素
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal <?> k = e.get();
        // ThreadLocal 对应的 key 存在，直接覆盖之前的值
        if (k == key) {
            e.value = value;
            return;
        }
        // key为 null，但是值不为 null，说明之前的 ThreadLocal 对象已经被回收了，当前数组中的 Entry 是一个陈旧（stale）的元素
        if (k == null) {
            // 用新元素替换陈旧的元素，这个方法进行了不少的垃圾清理动作，防止内存泄漏，具体可以看源代码，没看太懂
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // ThreadLocal 对应的 key 不存在并且没有找到陈旧的元素，则在空元素的位置创建一个新的 Entry。
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // cleanSomeSlot 清理陈旧的 Entry（key == null），具体的参考源码。如果没有清理陈旧的 Entry 并且数组中的元素大于了阈值，则进行 rehash。
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

set 方法大致流程如下：

1. 获取当前线程的成员变量map
2. map非空，则重新将ThreadLocal和新的value副本放入到map中
3. map空，则对线程的成员变量ThreadLocalMap进行初始化创建，并将ThreadLocal和value副本放入map中。

**注意：**

1. **int i = key.threadLocalHashCode & (len - 1);，这里实际上是对 len-1 进行了取余操作。**之所以能这样取余是因为 len 的值比较特殊，是 2 的 n 次方，减 1 之后低位变为全 1，高位变为全 0。例如 16，减 1 之后对应的二进制为: 00001111，这样其他数字中大于 16 的部分就会被 0 与掉，小于 16 的部分就会保留下来，就相当于取余了。
2. 在 **replaceStaleEntry 和 cleanSomeSlots** 方法中都会清理一些陈旧的 Entry，**防止内存泄漏**（关于内存泄漏，下面会讲）。
3. threshold 的值大小为**threshold = len \* 2 / 3;**
4. rehash 方法中首先会清理陈旧的 Entry，如果清理完之后元素数量仍然**大于 threshold 的 3/4**，则进行扩容操作（**数组大小变为原来的 2倍**）。

#### get方法

```java
//ThreadLocal.get 
public T get() {
    //1. 获取当前线程的实例对象
    Thread t = Thread.currentThread();
    //2. 获取当前线程的threadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //3. 获取map中当前threadLocal实例为key的值的entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //4. 当前entitiy不为null的话，就返回相应的值value
            T result = (T)e.value;
            return result;
        }
    }
    //5. 若map为null或者entry为null的话通过该方法初始化，并返回该方法返回的value
    return setInitialValue();
}
//ThreadLocal.setInitialValue 
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

```

大致流程如下：

1. 获取当前线程的ThreadLocalMap对象threadLocals
2. 从map中获取线程存储的K-V Entry节点。
3. 从Entry节点获取存储的Value副本值返回。
4. map为空的话返回初始值null，即线程变量副本为null，在使用时需要注意判断NullPointerException。

#### ThreadLocalMap

线程使用 ThreadLocalMap 来存储每个线程副本变量，它是 ThreadLocal 里的一个静态内部类。ThreadLocalMap 也是采用的散列表（Hash）思想来实现的，但是实现方式和 HashMap **不太一样**。

**ThreadLocalMap** 中使用**开放地址法来处理散列冲突**，而 **HashMap** 中使用的**分离链表法**。之所以采用不同的方式主要是因为：在 ThreadLocalMap 中的散列值分散的十分均匀，很少会出现冲突，并且 ThreadLocalMap 经常需要清除无用的对象，使用纯数组更加方便。

解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。源码详情如下：

```java
/**
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * Decrement i modulo len.
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

**每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。**

我们知道 Map 是一种 key-value 形式的数据结构，所以在散列数组中存储的元素也是 key-value 的形式。ThreadLocalMap 使用 Entry 类来存储数据，下面是该类的定义：

```java
static class Entry extends WeakReference <ThreadLocal <?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal <?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Entry 将 ThreadLocal 实例作为 key，副本变量作为 value 存储起来。注意 **Entry 中对于 ThreadLocal 实例的引用是一个弱引用**，该引用定义在 Reference 类（WeakReference的父类）中

软引用参考：<https://hanyunpeng0521.github.io/2019/07/01/Java-JVM-2-1/#软引用>

下面是 super(k) 最终调用的代码：

```java
Reference(T referent) {
    this(referent, null);
}

Reference(T referent, ReferenceQueue <? super T> queue) {
    this.referent = referent;
    this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
}
```

#### 副本变量存取

存取的基本流程就是首先获得当前线程的 ThreadLocalMap，将 ThreadLocal 实例作为键值传入 Map，然后就是进行相关的变量存取工作了。线程中的 ThreadLocalMap 是懒加载的，只有真正的要存变量时才会调用 createMap 创建

#### ThreadLocal 散列值

当创建了一个 ThreadLocal 的实例后，它的散列值就已经确定了，下面是 ThreadLocal 中的实现：

```java
/**
 * ThreadLocals rely on per-thread linear-probe hash maps attached
 * to each thread (Thread.threadLocals and
 * inheritableThreadLocals).  The ThreadLocal objects act as keys,
 * searched via threadLocalHashCode.  This is a custom hash code
 * (useful only within ThreadLocalMaps) that eliminates collisions
 * in the common case where consecutively constructed ThreadLocals
 * are used by the same threads, while remaining well-behaved in
 * less common cases.
 */
private final int threadLocalHashCode = nextHashCode();

/**
 * The next hash code to be given out. Updated atomically. Starts at
 * zero.
 */
private static AtomicInteger nextHashCode =
    new AtomicInteger();

/**
 * The difference between successively generated hash codes - turns
 * implicit sequential thread-local IDs into near-optimally spread
 * multiplicative hash values for power-of-two-sized tables.
 */
private static final int HASH_INCREMENT = 0x61c88647;

/**
 * Returns the next hash code.
 */
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

我们看到 `threadLocalHashCode` 是一个常量，它通过 `nextHashCode()`函数产生。`nextHashCode()`函数其实就是在一个 AtomicInteger 变量（初始值为0）的基础上每次累加 `0x61c88647`，使用 AtomicInteger 为了保证每次的加法是原子操作。而 `0x61c88647` 这个就比较神奇了，它可以使 hashcode 均匀的分布在大小为 2 的 N 次方的数组里。其实`0x61c88647`就是 `Fibonacci Hashing`

#### ThreadLocalMap的问题

由于ThreadLocalMap的key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

在threadlocal的生命周期中，都存在这些引用。看下图：实线代表强引用，虚线代表弱引用

![lJrx0J.png](https://s2.ax1x.com/2020/01/01/lJrx0J.png)

每个thread中都存在一个map，map的类型的ThreadLocal，ThreadLocalMap，Map中的key为一个threadlocal的实例，这个map的确使用了弱引用，不过弱引用只是针对key，每个key都弱引用指向threadlocal。当把threadlocal实例置为null以后，没有任何强引用指向threadlocal实例，所以threadlocal会被gc回收。但是，我们的value却不能被回收，因为存在一条从current thread连接过来的强引用。只有当前thread结束以后，current thread就不会存在栈中，强引用断开。Current Thread，Map，value将全部被gc回收。

所以得出一个结论就是只要这个线程对象被gc回收，就不会出现内存泄漏，但是在ThreadLocal设为null和线程结束这段时间不会被回收的，就发生我们认为的内存泄漏。其实这是一个对概念理解的不一致，也没什么好争论的。最要命的是线程对象不被回收的情况，这就发生了真正意义上的内存泄漏。比如使用线程池的时候，线程结束是不会销毁的，会再次使用的。就可能出现内存泄漏。

Java为了最小化减少内存泄漏的可能性和影响，在ThreadLocal的get，set的时候都会清除线程Map里所有key为null的value。所以最怕的情况就是threadlocal对象设null了，开始发生内存泄漏，然后使用线程池。这个线程任务结束，线程放回线程池中不销毁，这个线程一直不被使用，或者分配使用了又不再调用get，set方法，那么这个期间就会发生真正的内存泄漏。

**如何避免泄漏**

既然Key是**弱引用**，那么我们要做的事，就是在调用ThreadLocal的get()、set()方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

如果使用ThreadLocal的set方法之后，**没有显示的调用remove方法，就有可能发生内存泄露**，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得调用remove方法。

```java
ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
try {
    threadLocal.set(new Session(1, "Misout的博客"));
    // 其它业务逻辑
} finally {
    threadLocal.remove();
}
```

**应用场景**

Hibernate的session获取场景

```java
private static final ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();

//获取Session
public static Session getCurrentSession(){
    Session session =  threadLocal.get();
    //判断Session是否为空，如果为空，将创建一个session，并设置到本地线程变量中
    try {
        if(session ==null&&!session.isOpen()){
            if(sessionFactory==null){
                rbuildSessionFactory();// 创建Hibernate的SessionFactory
            }else{
                session = sessionFactory.openSession();
            }
        }
        threadLocal.set(session);
    } catch (Exception e) {
        // TODO: handle exception
    }

    return session;
}
```

为什么？每个线程访问数据库都应当是一个独立的Session会话，如果多个线程共享同一个Session会话，有可能其他线程关闭连接了，当前线程再执行提交时就会出现会话已关闭的异常，导致系统异常。**此方式能避免线程争抢Session，提高并发下的安全性。**

使用ThreadLocal的典型场景正如上面的数据库连接管理，线程会话管理等场景，只适用于独立变量副本的情况，**如果变量为全局共享的，则不适用在高并发下使用。**

- JVM利用设置ThreadLocalMap的Key为弱引用，来避免内存泄露

- JVM利用调用remove、get、set方法的时候，回收弱引用。
- 当ThreadLocal存储很多Key为null的Entry的时候，而不再去调用remove、get、set方法，那么将导致内存泄漏
- 当使用static ThreadLocal的时候，延长ThreadLocal的生命周期，那也可能导致内存泄漏。因为，static变量在类未加载的时候，它就已经加载，当线程结束的时候，static变量不一定会回收。那么，比起普通成员变量使用的时候才加载，static的生命周期加长将更容易导致内存泄漏危机

#### 总结

1. 每个ThreadLocal只能保存一个变量副本，如果想要上线一个线程能够保存多个副本以上，就需要创建多个ThreadLocal。
2. ThreadLocal内部的ThreadLocalMap键为弱引用，会有内存泄漏的风险。
3. 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。如果如果业务逻辑强依赖于副本变量，则不适合用ThreadLocal解决，需要另寻解决方案