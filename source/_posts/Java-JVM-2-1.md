---
title: java 四种引用
date: 2019-07-01 19:18:59
tags:
 - Java
categories:
 - Java
 - JVM
---

#### java四种引用

我们知道java相比C，C++中没有令人头痛的指针，但是却有和指针作用相似的引用对象（Reference），就是常说的引用，比如，Object obj = new Object()；这个obj就是引用，它指向的是真正的对象Object的地址，不过今天要说的是java中的四种引用。有人可能比较懵逼，四种引用？是的，从JDK1.2之后，java对引用这块的概念进行了扩充，按照引用的强度分为了四种引用：强引用，软引用，弱引用，虚引用。下面就让我们来看看这四种引用都具体的情况吧。

#### 强引用

我们平时代码中使用得最多的引用，对象的类是：StrongReference。就比如上面说的Object obj = new Object()；我们再熟悉不过了，作为最强的引用，只要引用还存在着，垃圾收集器就不会将该引用给回收，即使会出现OOM（内存溢出）。就是说这种引用只要引用还一直指向的对象，垃圾收集器是不会去管它的，所以它被称为强引用。不过如果

```
Object obj = new Object();
obj = null;
```

obj被赋值为了null，该引用就断了，垃圾收集器会在合适的时候回收改引用的内存。

还有一种情况就是obj是成员变量，方法执行完了，obj随着被栈帧被回收了，obj引用也是一起被回收了。强引用的使用就不介绍了，地球人都知道。

#### 软引用

软引用是用来描述一些有用但是非必须的对象。对应的类是SoftReference，它被回收的时机是系统内存不足的时候，如果内存足够，它不会被回收，内存不足了，可能会发生OOM了，软引用的对象就会被回收。这样的特性是不是就像缓存？是的，软引用可以用来存放缓存的数据，内存足够的时候一直可以访问，内存不足的时候，需要重新创建或者访问原对象。

其实不管是软引用，弱引用，还是虚引用，代码中使用方式都是像下面这样，使用对应的Reference将对象放入到构造函数当中，然后使用的地方reference.get()来调用具体对象。

```
Object obj = new Object();
SoftReference<Object> softReference = new SoftReference<>(obj);
softReference.get();
```

同时可以使用ReferenceQueue来把引用和引用队列给关联起来：

```
Object obj = new Object();
ReferenceQueue<Object> refQueue = new ReferenceQueue<>();
SoftReference<Object> softReference = new SoftReference<>(obj, refQueue);
```

**所谓关联起来，其实就是当引用被回收的时候，会被添加到ReferenceQueue中，使用ReferenceQueue.poll()方法可以返回当前可用的引用，并从队列冲删除。**简单来说就是引用和引用队列关联起来（引用的构造函数传入队列），然后引用被回收的时候会被添加到队列中，然后使用poll()方法可以返回引用。

#### 弱引用

弱引用比上面两个引用就更菜了，只要垃圾收集器扫描到了它，被弱引用关联的对象就会被回收。被弱引用关联对象的生命周期其实就是从对象创建到下一次垃圾回收。对应的类是WeakReference。

```java
public static void main(String[] args) throws InterruptedException {
      Object obj = new Object();
      ReferenceQueue<Object> refQueue = new ReferenceQueue<>();
      WeakReference<Object> weakRef = new WeakReference<>(obj, refQueue);
      System.out.println("引用：" + weakRef.get());
      System.out.println("队列中的东西：" + refQueue.poll());
      // 清除强引用, 触发GC
      obj = null;
      System.gc();
      Thread.sleep(200);
      System.out.println("引用：" + weakRef.get());
      System.out.println("引用加入队列了吗？ " + weakRef.isEnqueued());
      System.out.println("队列中的东西：" + refQueue.poll());
      /**
       * 输出结果
       * 引用：java.lang.Object@7bb11784
       * 队列中的东西：null
       * 引用：null
       * 引用加入队列了吗？ true
       * 队列中的东西：java.lang.ref.WeakReference@33a10788
       */
  }

```

可以看到当强引用被清除，手动触发GC后，弱引用回收，被加入到队列中了。

WeakHashMap跟hashMap很像，差别就在于，当WeakHashMap的key（弱引用），指向的对象被回收了，weakhashMap中的对象也就消失了。不会和HashMap一样一直持有该对象，导致无法回收。

#### 虚引用

虚引用是最弱的一种引用，它不会影响对象的生命周期，对象被回收跟它没啥关系。它引用的对象可以在任何时候被回收，而且也无法根据虚引用来取得一个对象的实例。仅仅当它指向的对象被回收的时候，它会受到一个通知。对应的类是PhantomReference。

有人就要问既然对对象回收没影响，那它有啥用（其实用处很少），我查阅网上的资料说是，可以用来监控对象的回收，和记录日志。简单点说就是对象被回收的时候，和虚引用相关的队列知道了实例对象被回收了。这个时候我们可以记录下来，知道对象是什么时候被回收的。

从而起到监控的作用。

```java
public static void main(String[] args) throws Exception {
       Object abc = new Object();
       ReferenceQueue<Object> refQueue = new ReferenceQueue<Object>();
       PhantomReference<Object> abcRef = new PhantomReference<Object>(abc, refQueue);
       System.out.println("队列中的东西：" + refQueue.poll());
       abc = null;
       System.gc();
       Thread.sleep(1000);
       System.out.println("引用加入队列了吗？ " + abcRef.isEnqueued());
       System.out.println("队列中的东西：" + refQueue.poll());
       /**
        * 输出：
        * 队列中的东西：null
        * 引用加入队列了吗？ true
        * 队列中的东西：java.lang.ref.PhantomReference@7bb11784
        */
   }

```

发现队列中有引用了，就可以添加日志记录了。

#### 实验代码

好了，理论说完了，下面我们来实践一下：我们新建一个 Java 工程，在里面新建一个类 `Main` 并添加实践代码（请谨慎尝试）：

```java
public class Main {

	// 引用测试类
	static class ReferenceTest {
		static final int _1M = 1024;
        // 引用队列，当某个引用所指向的对象被回收时这个引用本身会被添加到其对应的引用队列中
        // 其泛型为其中存放的引用要指向的对象类型
		ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

		// 强引用测试
		void testStrongReference() {
			ArrayList<byte[]> strongReferences = new ArrayList<>();
			try {
				while (true) {
					strongReferences.add(new byte[_1M]);
				}
			} catch (OutOfMemoryError e) {
				e.printStackTrace();
			}
		}

		// 软引用测试
		void testSoftReference() {
			ArrayList<SoftReference> softReferences = new ArrayList<>();
			try {
				while (true) {
					softReferences.add(new SoftReference<>(new byte[_1M], referenceQueue));
				}
			} catch (OutOfMemoryError e) {
				e.printStackTrace();
			}
		}

		// 弱引用测试
		void testWeakReference() {
			ArrayList<WeakReference> weakReferences = new ArrayList<>();
			try {
				while (true) {
					weakReferences.add(new WeakReference<>(new byte[_1M], referenceQueue));
				}
			} catch (OutOfMemoryError e) {
				e.printStackTrace();
			}
		}

		// 虚引用测试
		void testPhantomReference() {
			ArrayList<PhantomReference<byte[]>> phantomReferences = new ArrayList<>();
			try {
				while (true) {
					phantomReferences.add(new PhantomReference<>(new byte[_1M], referenceQueue));
				}
			} catch (OutOfMemoryError e) {
				e.printStackTrace();
			}
		}
	}

	public static void main(String[] args) {
		ReferenceTest test = new ReferenceTest();
		test.testStrongReference();
	}
}

```

我们在 `Main` 类中创建了一个静态内部类 `ReferenceTest`，并在其中提供了 4 个方法，方法很简单，就是不断创建新的 `byte` 数组，并且用不同类型的引用对象指向它。直到发生 `OutOfMemoryError` 异常。

运行过程中你会看到电脑内存占用飙升，最后会抛出 `OutOfMemoryError` 异常，这个结果是显而易见的，现在来看看对软引用的测试，修改一下 `main` 方法中的代码：`test.testSoftReference();`

同样的你会看到电脑占用内存飙升，但是最终会稳定一个值：因为我们现在用的是软引用来指向一个个 byte 数组。在 JVM 抛出 OutOfMemoryError 异常之前会将只被软引用指向的对象回收掉（通过执行垃圾回收动作），因此不会抛出 OutOfMemoryError 异常。下面来看看对弱引用的测试，我们改一下 main 方法中的一行代码：test.testWeakReference();

同样的你会看到相同的结果，原因也正如上文所说：JVM 的垃圾回收动作会回收掉所有只被弱引用指向的对象。最后来看看虚引用的测试，同样的修改 `main` 方法中的一行代码：`test.testPhantomReference();`

你会看到和强引用测试一样的结果：JVM 最终会抛出一个 OutOfMemoryError 异常。可能有些小伙伴们会问了：虚引用不是引用强度最弱的的吗，怎么会因为它而抛出 OutOfMemoryError 异常呢？其实仔细一想：**虚引用确实是引用强度最弱的，但是还有一点是虚引用根本不会影响对象的声明周期，也就是说某个对象是否被 JVM 的垃圾回收动作回收和这个对象是否被虚引用所指向和被多少个虚引用所指向没有任何关系，既然其不会影响对象的生命周期，那么使用和不使用虚引用指向对象对这个对象是否被 JVM 回收是没有任何区别的，那么我们就可以将其看做没有使用虚引用时的代码，此时效果自然和直接使用强引用一样。关于这个，可以参考 PhantomReference 类的源码注释：

`Unlike soft and weak references, phantom references are not automatically cleared by the garbage collector as they are enqueued. An object that is reachable via phantom references will remain so until all such references are cleared or themselves become unreachable.`

#### 引用队列

在上节的代码中，我们新建了一个引用队列（ReferenceQueue）对象，并且在创建软引用、弱引用和虚引用对象时将其作为参数传入对应引用的构造方法中。在文章的开头提到过可以利用引用队列来检测某个引用指向的对象是否被垃圾回收器回收，那么具体应该怎么做呢。我们可以看一下 3 类引用的源码，这里以弱引用为例（剩余两种可以类比）：

```
public class WeakReference<T> extends Reference<T> {
    
    public WeakReference(T referent) {
        super(referent);
    }

    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}

```

可以看到 `WeakReference` 类继承了 `Reference` 类，可以猜到 `Reference` 类是 3 种引用的基类，我们看看这个类的源码 `Reference.java`：

```java
public abstract class Reference<T> {

    private T referent;         /* Treated specially by GC */

    volatile ReferenceQueue<? super T> queue;

    /* When active:   NULL
     *     pending:   this
     *    Enqueued:   next reference in queue (or this if last)
     *    Inactive:   this
     */
    @SuppressWarnings("rawtypes")
    Reference next; // 引用所处的状态不同时，该属性保存了不同的信息
    
	// ...
    
    /**
     * 获取当前引用所指向的对象的方法，如果所指向对象已经被 GC 回收，那么返回 null
     */
    public T get() {
        return this.referent;
    }

    /**
     * 清除该引用所指向的对象，该方法会在 GC 回收该引用指向的对象后被 GC 调用，
     * 之后，通过该引用对象的 get 方法得到的返回值为 null, 该方法不应该被程序员主动调用
     */
    public void clear() {
        this.referent = null;
    }

    /* -- Queue operations -- */

    /**
     * 判断当前引用是否已经进入对应的引用队列，
     * 如果构造该引用对象时没有指定对应的引用队列，那么该方法始终返回 false
     */
    public boolean isEnqueued() {
        return (this.queue == ReferenceQueue.ENQUEUED);
    }

    /**
     * 如果当前引用对象的引用队列属性（构造时由参数指定）不为 null, 
     * 那么当这个引用所指向的对象被 GC 回收之后会由 GC 调用这个方法，
     * 代表将该引用进入对应的引用队列（即该引用指向的对象被回收）
     */
    public boolean enqueue() {
        return this.queue.enqueue(this);
    }

    /* -- Constructors -- */
    Reference(T referent) {
        this(referent, null);
    }

    Reference(T referent, ReferenceQueue<? super T> queue) {
        this.referent = referent;
        this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
    }
}

```

我们在 `Reference` 类中的 `enqueue` 方法（这个方法本身会被 GC 线程调用）中发现其直接调用了对应引用队列（`ReferenceQueue`）的 `enqueue` 方法，我们来看看 `ReferenceQueue` 类的这个方法：

```java
public class ReferenceQueue<T> {

    /**
     * Constructs a new reference-object queue.
     */
    public ReferenceQueue() { }

    // ...

    static private class Lock { };
    private Lock lock = new Lock();
    private volatile Reference<? extends T> head = null;
    private long queueLength = 0;

    // 引用对象本身入队列的过程就是一个向单向链表中插入节点的过程
    boolean enqueue(Reference<? extends T> r) { /* Called only by Reference class */
        synchronized (lock) { // 保证线程安全
            // Check that since getting the lock this reference hasn't already been
            // enqueued (and even then removed)
            ReferenceQueue<?> queue = r.queue;
            if ((queue == NULL) || (queue == ENQUEUED)) {
                return false;
            }
            assert queue == this;
            r.queue = ENQUEUED; // 更新引用入队状态
           	// 前插法插入链表节点
            r.next = (head == null) ? r : head;
            head = r;
            queueLength++;
            if (r instanceof FinalReference) {
                sun.misc.VM.addFinalRefCount(1);
            }
            lock.notifyAll();
            return true;
        }
    }

    /**
     * 返回当前引用队列中的第一个引用对象，如果不存在则返回 null
     * 该方法不会阻塞线程
     */
    public Reference<? extends T> poll() {
        if (head == null)
            return null;
        synchronized (lock) {
            return reallyPoll();
        }
    }

    /**
     * 返回当前引用队列中第一个可用的引用对象，如果没有，则阻塞线程一定时间（参数指定）
     * 阻塞时间过后，如果当前队列中仍然没有可用的引用对象，那么抛出中断异常（InterruptedException）
     */
    public Reference<? extends T> remove(long timeout)
        throws IllegalArgumentException, InterruptedException
    {
        if (timeout < 0) {
            throw new IllegalArgumentException("Negative timeout value");
        }
        synchronized (lock) {
            Reference<? extends T> r = reallyPoll();
            if (r != null) return r;
            long start = (timeout == 0) ? 0 : System.nanoTime();
            for (;;) {
                lock.wait(timeout);
                r = reallyPoll();
                if (r != null) return r;
                if (timeout != 0) {
                    long end = System.nanoTime();
                    timeout -= (end - start) / 1000_000;
                    if (timeout <= 0) return null;
                    start = end;
                }
            }
        }
    }

    /**
     * 阻塞调用线程，直到当前引用队列中存在可用的引用对象，将该引用对象从引用队列中移除并返回该引用对象
     */
    public Reference<? extends T> remove() throws InterruptedException {
        return remove(0);
    }
}

```

利用注释和源代码，我们就可以将整个过程的逻辑串起来了：

**GC 线程回收对象 -> 将相关指向这个对象的引用加入到其引用队列（如果有）-> 更新引用入队状态（isEnqueued 方法返回 true）-> 在 Java 代码中可以得到引用队列中的已经入队的引用（即得到要回收对象的对应引用对象，作为对象回收的一个通知）。**

下面看一个小例子，利用引用队列来得知回收的对象，我们在上一节的代码中新建一个静态内部类 ReferenceQueueTest：

```java
// 引用队列测试类
static class ReferenceQueueTest {

	ReferenceQueue<byte[]> referenceQueue = new ReferenceQueue<>();

	// 对象回收时的引用通知测试
	void testReferenceNotify() {
		WeakReference<byte[]> weakReference = new WeakReference<>(new byte[1024], referenceQueue);
            // 后面的 ReferenceQueue.remove 方法会阻塞调用线程，因此开子线程进行操作
		Thread thread = new Thread(() -> {
			try {
				for (Reference pr; (pr = referenceQueue.remove()) != null; ) {
					System.out.println(pr + " 引用所指向的对象被回收!");
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		});
		/* 
		 因为 ReferenceQueue 对象的 remove 方法是阻塞线程的，因此子线程需设置守护线程，
		 否则如果 ReferenceQueue 中没有可取出的引用对象会导致线程一直阻塞，程序不能退出
		*/
		thread.setDaemon(true);
		thread.start();
		// 启动垃圾回收动作，将弱引用指向的对象回收
		System.gc();
	}
}

```

在 `main` 方法中新建该内部类对象并且调用 `testReferenceNotify` 方法

可以看到，当弱引用指向的对象被回收之后，我们成功的从该弱引用对象中的引用对象中得到了该弱引用对象，即完成了对象回收的监视过程。

#### 总结

将人比作垃圾收集器，引用比作食物，我们来总结下四种引用：

- 强引用是毒药，即使你很饿了你也不会去吃它；
- 软引用是零食，不饿的时候不吃，饿了饥不择食，零食也能填饱肚子；
- 弱引用是饭菜，到了吃饭时间（垃圾回收），就吃饭菜；
- 虚引用是剩菜，当你吃完东西（回收完对象），就回剩下剩菜，别人就知道你吃过饭了。

| 引用 | 回收时机         | 使用场景                  |
| ---- | ---------------- | ------------------------- |
| 强   | 不会被回收       | 正常编码使用              |
| 软   | 内存不够了，被GC | 可作为缓存                |
| 弱   | GC发生时         | 可作为缓存（WeakHashMap） |
| 虚   | 任何时候         | 监控对象回收，记录日志    |

