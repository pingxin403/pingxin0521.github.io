---
title: Java  CAS、原子类（五）
date: 2019-05-05 18:18:59
tags:
 - Java
 - 并发
categories:
 - Java
 - 高级
---

### CAS

在计算机科学中，比较和交换（Conmpare And Swap）是用于实现多线程同步的原子指令。 它将内存位置的内容与给定值进行比较，只有在相同的情况下，将该内存位置的内容修改为新的给定值。 这是作为单个原子操作完成的。 原子性保证新值基于最新信息计算; 如果该值在同一时间被另一个线程更新，则写入将失败。 操作结果必须说明是否进行替换; 这可以通过一个简单的布尔响应（这个变体通常称为比较和设置），或通过返回从内存位置读取的值来完成。

<!--more-->

**一个 CAS 涉及到以下操作：**

我们假设内存中的原数据V，旧的预期值A，需要修改的新值B。

> 比较 A 与 V 是否相等。（比较）
>  如果比较相等，将 B 写入 V。（交换）
>  返回操作是否成功。
>  当多个线程同时对某个资源进行CAS操作，只能有一个线程操作成功，但是并不会阻塞其他线程,其他线程只会收到操作失败的信号。可见 CAS 其实是一个乐观锁。

![lU7Pv8.png](https://s2.ax1x.com/2020/01/03/lU7Pv8.png)

如上图中，主存中保存V值，线程中要使用V值要先从主存中读取V值到线程的工作内存A中，然后计算后变成B值，最后再把B值写回到内存V值中。多个线程共用V值都是如此操作。CAS的核心是在将B值写入到V之前要比较A值和V值是否相同，如果不相同证明此时V值已经被其他线程改变，重新将V值赋给A，并重新计算得到B，如果相同，则将B值赋给V。

如果不使用CAS机制，看看存在什么问题，假如V=1，现在Thread1要对V进行加1，Thread2也要对V进行加1，首先Thread1读取V=1到自己工作内存A中此时A=1，假设Thread2此时也读取V=1到自己的工作内存A中，分别进行加1操作后，两个线程中B的值都为2，此时写回到V中时发现V的值为2，但是两个线程分别对V进行加处理结果却只加了1有问题。

#### CAS 实现

将对 java.util.concurrent.atomic 包下的原子类 AtomicInteger 中的 compareAndSet 方法进行分析，就能发现最终调用的是 sum.misc.Unsafe 这个类。看名称 Unsafe 就是一个不安全的类，这个类是利用了 Java 的类和包在可见性的的规则中的一个恰到好处处的漏洞。Unsafe 这个类为了速度，在Java的安全标准上做出了一定的妥协。
 再往下寻找我们发现 Unsafe的compareAndSwapInt 是 Native 的方法：

`public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);`

 也就是说，这几个 CAS 的方法应该是使用了本地的方法。所以这几个方法的具体实现需要我们自己去 jdk 的源码中搜索。

**最终到搜索 `cmpxchg` 函数**

```cpp
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest, jint compare_value) {  // 判断是否是多核 CPU
  int mp = os::is_MP();
  __asm {    // 将参数值放入寄存器中
    mov edx, dest    // 注意: dest 是指针类型，这里是把内存地址存入 edx 寄存器中
    mov ecx, exchange_value
    mov eax, compare_value    
    // LOCK_IF_MP
    cmp mp, 0
    /*
     * 如果 mp = 0，表明是线程运行在单核 CPU 环境下。此时 je 会跳转到 L0 标记处，
     * 也就是越过 _emit 0xF0 指令，直接执行 cmpxchg 指令。也就是不在下面的 cmpxchg 指令
     * 前加 lock 前缀。
     */
    je L0    /*
     * 0xF0 是 lock 前缀的机器码，这里没有使用 lock，而是直接使用了机器码的形式。至于这样做的
     * 原因可以参考知乎的一个回答：
     *     https://www.zhihu.com/question/50878124/answer/123099923
     */ 
    _emit 0xF0L0:    /*
     * 比较并交换。简单解释一下下面这条指令，熟悉汇编的朋友可以略过下面的解释:
     *   cmpxchg: 即“比较并交换”指令
     *   dword: 全称是 double word，在 x86/x64 体系中，一个 
     *          word = 2 byte，dword = 4 byte = 32 bit
     *   ptr: 全称是 pointer，与前面的 dword 连起来使用，表明访问的内存单元是一个双字单元
     *   [edx]: [...] 表示一个内存单元，edx 是寄存器，dest 指针值存放在 edx 中。
     *          那么 [edx] 表示内存地址为 dest 的内存单元
     *          
     * 这一条指令的意思就是，将 eax 寄存器中的值（compare_value）与 [edx] 双字内存单元中的值
     * 进行对比，如果相同，则将 ecx 寄存器中的值（exchange_value）存入 [edx] 内存单元中。
     */
    cmpxchg dword ptr [edx], ecx
  }
}
```

总结一下 JAVA 的 cas 是怎么实现的：

- java 的 cas 利用的的是 unsafe 这个类提供的 cas 操作。
- unsafe 的cas 依赖了的是 jvm 针对不同的操作系统实现的 Atomic::cmpxchg
- Atomic::cmpxchg 的实现使用了汇编的 cas 操作，并使用 cpu 硬件提供的 lock信号保证其原子性

#### ABA 问题

CAS 由三个步骤组成，分别是“读取->比较->写回”。考虑这样一种情况，线程1和线程2同时执行 CAS 逻辑，两个线程的执行顺序如下：

> 时刻1：线程1执行读取操作，获取原值 A，然后线程被切换走
>  时刻2：线程2执行完成 CAS 操作将原值由 A 修改为 B
>  时刻3：线程2再次执行 CAS 操作，并将原值由 B 修改为 A
>  时刻4：线程1恢复运行，将比较值（compareValue）与原值（oldValue）进行比较，发现两个值相等。
>  然后用新值（newValue）写入内存中，完成 CAS 操作

如上流程，线程1并不知道原值已经被修改过了，在它看来并没什么变化，所以它会继续往下执行流程。对于 ABA 问题，通常的处理措施是对每一次 CAS 操作设置版本号。java.util.concurrent.atomic 包下提供了一个可处理 ABA 问题的原子类 AtomicStampedReference

**ABA问题的解决办法**

1. 在变量前面追加版本号：每次变量更新就把版本号加1，则A-B-A就变成1A-2B-3A。
2. atomic包下的AtomicStampedReference类：其compareAndSet方法首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用的该标志的值设置为给定的更新值。

**其他问题**

**CAS除了ABA问题，仍然存在循环时间长开销大和只能保证一个共享变量的原子操作**

1. 循环时间长开销大

   自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

2. 只能保证一个共享变量的原子操作

   当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

#### CAS 的应用

1. Java的concurrent包下就有很多类似的实现类，如Atomic开头那些类。

   AtomicInteger 的 incrementAndGet()

   ```java
       public final int getAndAddInt(Object var1, long var2, int var4) {
           int var5;
           do {
               var5 = this.getIntVolatile(var1, var2);
           } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
   
           return var5;
       }
   ```

   与自旋锁有异曲同工之妙，就是一直while，直到操作成功为止。

2. 自旋锁

   ```java
   public class SpinLock {
   
     private AtomicReference<Thread> sign =new AtomicReference<>();
   
     public void lock(){
       Thread current = Thread.currentThread();
       while(!sign .compareAndSet(null, current)){
       }
     }
   
     public void unlock (){
       Thread current = Thread.currentThread();
       sign .compareAndSet(current, null);
     }
   }
   ```

   所谓自旋锁，我觉得这个名字相当的形象，在lock()的时候，一直while()循环，直到 cas 操作成功为止。

3. 令牌桶限流器
   就是系统以恒定的速度向桶内增加令牌。每次请求前从令牌桶里面获取令牌。如果获取到令牌就才可以进行访问。当令牌桶内没有令牌的时候，拒绝提供服务。我们来看看 eureka 的限流器是如何使用 CAS 来维护多线程环境下对 token 的增加和分发的。

   ```java
   public class RateLimiter {
   
       private final long rateToMsConversion;
   
       private final AtomicInteger consumedTokens = new AtomicInteger();
       private final AtomicLong lastRefillTime = new AtomicLong(0);
   
       @Deprecated
       public RateLimiter() {
           this(TimeUnit.SECONDS);
       }
   
       public RateLimiter(TimeUnit averageRateUnit) {
           switch (averageRateUnit) {
               case SECONDS:
                   rateToMsConversion = 1000;
                   break;
               case MINUTES:
                   rateToMsConversion = 60 * 1000;
                   break;
               default:
                   throw new IllegalArgumentException("TimeUnit of " + averageRateUnit + " is not supported");
           }
       }
   
       //提供给外界获取 token 的方法
       public boolean acquire(int burstSize, long averageRate) {
           return acquire(burstSize, averageRate, System.currentTimeMillis());
       }
   
       public boolean acquire(int burstSize, long averageRate, long currentTimeMillis) {
           if (burstSize <= 0 || averageRate <= 0) { // Instead of throwing exception, we just let all the traffic go
               return true;
           }
   
           //添加token
           refillToken(burstSize, averageRate, currentTimeMillis);
   
           //消费token
           return consumeToken(burstSize);
       }
   
       private void refillToken(int burstSize, long averageRate, long currentTimeMillis) {
           long refillTime = lastRefillTime.get();
           long timeDelta = currentTimeMillis - refillTime;
   
           //根据频率计算需要增加多少 token
           long newTokens = timeDelta * averageRate / rateToMsConversion;
           if (newTokens > 0) {
               long newRefillTime = refillTime == 0
                       ? currentTimeMillis
                       : refillTime + newTokens * rateToMsConversion / averageRate;
   
               // CAS 保证有且仅有一个线程进入填充
               if (lastRefillTime.compareAndSet(refillTime, newRefillTime)) {
                   while (true) {
                       int currentLevel = consumedTokens.get();
                       int adjustedLevel = Math.min(currentLevel, burstSize); // In case burstSize decreased
                       int newLevel = (int) Math.max(0, adjustedLevel - newTokens);
                       // while true 直到更新成功为止
                       if (consumedTokens.compareAndSet(currentLevel, newLevel)) {
                           return;
                       }
                   }
               }
           }
       }
   
       private boolean consumeToken(int burstSize) {
           while (true) {
               int currentLevel = consumedTokens.get();
               if (currentLevel >= burstSize) {
                   return false;
               }
   
               // while true 直到没有token 或者 获取到为止
               if (consumedTokens.compareAndSet(currentLevel, currentLevel + 1)) {
                   return true;
               }
           }
       }
   
       public void reset() {
           consumedTokens.set(0);
           lastRefillTime.set(0);
       }
   }
   ```

   所以梳理一下 CAS 在令牌桶限流器的作用。就是保证在多线程情况下，不阻塞线程的填充token 和消费token。

### 原子类

在并发编程中很容易出现并发安全的问题，有一个很简单的例子就是多线程更新变量i=1,比如多个线程执行i++操作，就有可能获取不到正确的值，而这个问题，最常用的方法是通过Synchronized进行控制来达到线程安全的目的。但是由于synchronized是采用的是悲观锁策略，并不是特别高效的一种解决方案。

实际上，在J.U.C下的atomic包提供了一系列的操作简单，性能高效，并能保证线程安全的类去更新基本类型变量，数组元素，引用类型以及更新对象中的字段类型。atomic包下的这些类都是采用的是乐观锁策略去原子更新数据，在java中则是使用CAS操作具体实现。

####  原子更新基本类型

atomic包提高原子更新基本类型的工具类，主要有这些：

1. AtomicBoolean：以原子更新的方式更新boolean；
2. AtomicInteger：以原子更新的方式更新Integer;
3. AtomicLong：以原子更新的方式更新Long；

这几个类的用法基本一致，这里以AtomicInteger为例总结常用的方法

1. addAndGet(int delta) ：以原子方式将输入的数值与实例中原本的值相加，并返回最后的结果；
2. incrementAndGet() ：以原子的方式将实例中的原值进行加1操作，并返回最终相加后的结果；
3. getAndSet(int newValue)：将实例中的值更新为新值，并返回旧值；
4. getAndIncrement()：以原子的方式将实例中的原值加1，返回的是自增前的旧值；

#### 原子更新数组类型

atomic包下提供能原子更新数组中元素的类有：

1. AtomicIntegerArray：原子更新整型数组中的元素；
2. AtomicLongArray：原子更新长整型数组中的元素；
3. AtomicReferenceArray：原子更新引用类型数组中的元素

这几个类的用法一致，就以AtomicIntegerArray来总结下常用的方法：

1. addAndGet(int i, int delta)：以原子更新的方式将数组中索引为i的元素与输入值相加；
2. getAndIncrement(int i)：以原子更新的方式将数组中索引为i的元素自增加1；
3. compareAndSet(int i, int expect, int update)：将数组中索引为i的位置的元素进行更新

可以看出，AtomicIntegerArray与AtomicInteger的方法基本一致，只不过在AtomicIntegerArray的方法中会多一个指定数组索引位i。下面举一个简单的例子：

```java
public class AtomicDemo {
    //    private static AtomicInteger atomicInteger = new AtomicInteger(1);
    private static int[] value = new int[]{1, 2, 3};
    private static AtomicIntegerArray integerArray = new AtomicIntegerArray(value);

    public static void main(String[] args) {
        //对数组中索引为1的位置的元素加5
        int result = integerArray.getAndAdd(1, 5);
        System.out.println(integerArray.get(1));
        System.out.println(result);
    }
}
输出结果：
7
2
```

通过getAndAdd方法将位置为1的元素加5，从结果可以看出索引为1的元素变成了7，该方法返回的也是相加之前的数为2。

#### 原子更新引用类型

如果需要原子更新引用类型变量的话，为了保证线程安全，atomic也提供了相关的类：

1. AtomicReference：原子更新引用类型；
2. AtomicReferenceFieldUpdater：原子更新引用类型里的字段；
3. AtomicMarkableReference：原子更新带有标记位的引用类型；

这几个类的使用方法也是基本一样的，以AtomicReference为例，来说明这些类的基本用法。下面是一个demo

```java
public class AtomicDemo {

    private static AtomicReference<User> reference = new AtomicReference<>();

    public static void main(String[] args) {
        User user1 = new User("a", 1);
        reference.set(user1);
        User user2 = new User("b",2);
        User user = reference.getAndSet(user2);
        System.out.println(user);
        System.out.println(reference.get());
    }

    static class User {
        private String userName;
        private int age;

        public User(String userName, int age) {
            this.userName = userName;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "userName='" + userName + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}

输出结果：
User{userName='a', age=1}
User{userName='b', age=2}
```

首先将对象User1用AtomicReference进行封装，然后调用getAndSet方法，从结果可以看出，该方法会原子更新引用的user对象，变为`User{userName='b', age=2}`，返回的是原来的user对象User`{userName='a', age=1}`。

#### 原子更新字段类型

如果需要更新对象的某个字段，并在多线程的情况下，能够保证线程安全，atomic同样也提供了相应的原子操作类：

1. AtomicIntegeFieldUpdater：原子更新整型字段类；
2. AtomicLongFieldUpdater：原子更新长整型字段类；
3. AtomicStampedReference：原子更新引用类型，这种更新方式会带有版本号。而为什么在更新的时候会带有版本号，是为了解决CAS的ABA问题；

要想使用原子更新字段需要两步操作：

1. 原子更新字段类都是抽象类，只能通过静态方法`newUpdater`来创建一个更新器，并且需要设置想要更新的类和属性；
2. 更新类的属性必须使用`public volatile`进行修饰；

这几个类提供的方法基本一致，以AtomicIntegerFieldUpdater为例来看看具体的使用：

```java
public class AtomicDemo {

    private static AtomicIntegerFieldUpdater updater = AtomicIntegerFieldUpdater.newUpdater(User.class,"age");
    public static void main(String[] args) {
        User user = new User("a", 1);
        int oldValue = updater.getAndAdd(user, 5);
        System.out.println(oldValue);
        System.out.println(updater.get(user));
    }

    static class User {
        private String userName;
        public volatile int age;

        public User(String userName, int age) {
            this.userName = userName;
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "userName='" + userName + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
} 

输出结果：
1
6
```

从示例中可以看出，创建`AtomicIntegerFieldUpdater`是通过它提供的静态方法进行创建，`getAndAdd`方法会将指定的字段加上输入的值，并且返回相加之前的值。user对象中age字段原值为1，加5之后，可以看出user对象中的age字段的值已经变成了6。

### AtomicInteger源码分析

原子操作是指不会被线程调度机制打断的操作，这种操作一旦开始，就一直运行到结束，中间不会有任何线程上下文切换。

原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序不可以被打乱，也不可以被切割而只执行其中的一部分，将整个操作视作一个整体是原子性的核心特征。

我们这里说的原子操作与数据库ACID中的原子性，笔者认为最大区别在于，数据库中的原子性主要运用在事务中，一个事务之内的所有更新操作要么都成功，要么都失败，事务是有回滚机制的，而我们这里说的原子操作是没有回滚的，这是最大的区别。

#### 主要属性

```java
// 获取Unsafe的实例
private static final Unsafe unsafe = Unsafe.getUnsafe();
// 标识value字段的偏移量
private static final long valueOffset;
// 静态代码块，通过unsafe获取value的偏移量
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
// 存储int类型值的地方，使用volatile修饰
private volatile int value;
```

1. 使用int类型的value存储值，且使用volatile修饰，volatile主要是保证可见性，即一个线程修改对另一个线程立即可见，主要的实现原理是内存屏障，这里不展开来讲，有兴趣的可以自行查阅相关资料。
2. 调用Unsafe的objectFieldOffset()方法获取value字段在类中的偏移量，用于后面CAS操作时使用。

#### compareAndSet()方法

```
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
// Unsafe中的方法
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

调用Unsafe.compareAndSwapInt()方法实现，这个方法有四个参数：

1. 操作的对象；
2. 对象中字段的偏移量；
3. 原来的值，即期望的值；
4. 要修改的值；

可以看到，这是一个native方法，底层是使用C/C++写的，主要是调用CPU的CAS指令来实现，它能够保证只有当对应偏移量处的字段值是期望值时才更新，即类似下面这样的两步操作：

```
if(value == expect) {
    value = newValue;
}
```

通过CPU的CAS指令可以保证这两步操作是一个整体，也就不会出现多线程环境中可能比较的时候value值是a，而到真正赋值的时候value值可能已经变成b了的问题。

#### getAndIncrement()方法

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

// Unsafe中的方法
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

getAndIncrement()方法底层是调用的Unsafe的getAndAddInt()方法，这个方法有三个参数：

（1）操作的对象；

（2）对象中字段的偏移量；

（3）要增加的值；

查看Unsafe的getAndAddInt()方法的源码，可以看到它是先获取当前的值，然后再调用compareAndSwapInt()尝试更新对应偏移量处的值，如果成功了就跳出循环，如果不成功就再重新尝试，直到成功为止，这可不就是（CAS+自旋）的乐观锁机制么^^

AtomicInteger中的其它方法几乎都是类似的，最终会调用到Unsafe的compareAndSwapInt()来保证对value值更新的原子性。

#### 总结

1. AtomicInteger中维护了一个使用volatile修饰的变量value，保证可见性；
2. AtomicInteger中的主要方法最终几乎都会调用到Unsafe的compareAndSwapInt()方法保证对变量修改的原子性。

### AtomicStampedReference

AtomicStampedReference是java并发包下提供的一个原子类，它能解决其它原子类无法解决的ABA问题。

ABA问题通常发生在无锁结构中，用代码来表示上面的过程大概就是这样：

```java
public class ABATest {

    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(1);

        new Thread(()->{
            int value = atomicInteger.get();
            System.out.println("thread 1 read value: " + value);

            // 阻塞1s
            LockSupport.parkNanos(1000000000L);

            if (atomicInteger.compareAndSet(value, 3)) {
                System.out.println("thread 1 update from " + value + " to 3");
            } else {
                System.out.println("thread 1 update fail!");
            }
        }).start();

        new Thread(()->{
            int value = atomicInteger.get();
            System.out.println("thread 2 read value: " + value);
            if (atomicInteger.compareAndSet(value, 2)) {
                System.out.println("thread 2 update from " + value + " to 2");

                // do sth

                value = atomicInteger.get();
                System.out.println("thread 2 read value: " + value);
                if (atomicInteger.compareAndSet(value, 1)) {
                    System.out.println("thread 2 update from " + value + " to 1");
                }
            }
        }).start();
    }
}
```

打印结果为：

```
thread 1 read value: 1
thread 2 read value: 1
thread 2 update from 1 to 2
thread 2 read value: 2
thread 2 update from 2 to 1
thread 1 update from 1 to 3
```

#### 内部类

```java
private static class Pair<T> {
    final T reference;
    final int stamp;
    private Pair(T reference, int stamp) {
        this.reference = reference;
        this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
        return new Pair<T>(reference, stamp);
    }
}
```

将元素值和版本号绑定在一起，存储在Pair的reference和stamp（邮票、戳的意思）中。

#### 属性

```java
private volatile Pair<V> pair;
private static final sun.misc.Unsafe UNSAFE = sun.misc.Unsafe.getUnsafe();
private static final long pairOffset =
    objectFieldOffset(UNSAFE, "pair", AtomicStampedReference.class);
```

声明一个Pair类型的变量并使用Unsfae获取其偏移量，存储到pairOffset中。

#### 构造方法

```
public AtomicStampedReference(V initialRef, int initialStamp) {
    pair = Pair.of(initialRef, initialStamp);
}
```

构造方法需要传入初始值及初始版本号。

#### compareAndSet()方法

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    // 获取当前的（元素值，版本号）对
    Pair<V> current = pair;
    return
        // 引用没变
        expectedReference == current.reference &&
        // 版本号没变
        expectedStamp == current.stamp &&
        // 新引用等于旧引用
        ((newReference == current.reference &&
        // 新版本号等于旧版本号
          newStamp == current.stamp) ||
          // 构造新的Pair对象并CAS更新
         casPair(current, Pair.of(newReference, newStamp)));
}

private boolean casPair(Pair<V> cmp, Pair<V> val) {
    // 调用Unsafe的compareAndSwapObject()方法CAS更新pair的引用为新引用
    return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
}
```

1. 如果元素值和版本号都没有变化，并且和新的也相同，返回true；
2. 如果元素值和版本号都没有变化，并且和新的不完全相同，就构造一个新的Pair对象并执行CAS更新pair。

可以看到，java中的实现跟我们上面讲的ABA的解决方法是一致的。

首先，使用版本号控制；

其次，不重复使用节点（Pair）的引用，每次都新建一个新的Pair来作为CAS比较的对象，而不是复用旧的；

最后，外部传入元素值及版本号，而不是节点（Pair）的引用。

#### 案例

让我们来使用AtomicStampedReference解决开篇那个AtomicInteger带来的ABA问题。

```java
public class ABATest {

    public static void main(String[] args) {
        testStamp();
    }

    private static void testStamp() {
        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1, 1);

        new Thread(()->{
            int[] stampHolder = new int[1];
            int value = atomicStampedReference.get(stampHolder);
            int stamp = stampHolder[0];
            System.out.println("thread 1 read value: " + value + ", stamp: " + stamp);

            // 阻塞1s
            LockSupport.parkNanos(1000000000L);

            if (atomicStampedReference.compareAndSet(value, 3, stamp, stamp + 1)) {
                System.out.println("thread 1 update from " + value + " to 3");
            } else {
                System.out.println("thread 1 update fail!");
            }
        }).start();

        new Thread(()->{
            int[] stampHolder = new int[1];
            int value = atomicStampedReference.get(stampHolder);
            int stamp = stampHolder[0];
            System.out.println("thread 2 read value: " + value + ", stamp: " + stamp);
            if (atomicStampedReference.compareAndSet(value, 2, stamp, stamp + 1)) {
                System.out.println("thread 2 update from " + value + " to 2");

                // do sth

                value = atomicStampedReference.get(stampHolder);
                stamp = stampHolder[0];
                System.out.println("thread 2 read value: " + value + ", stamp: " + stamp);
                if (atomicStampedReference.compareAndSet(value, 1, stamp, stamp + 1)) {
                    System.out.println("thread 2 update from " + value + " to 1");
                }
            }
        }).start();
    }
}
```

运行结果为：

```
thread 1 read value: 1, stamp: 1
thread 2 read value: 1, stamp: 1
thread 2 update from 1 to 2
thread 2 read value: 2, stamp: 2
thread 2 update from 2 to 1
thread 1 update fail!
```

可以看到线程1最后更新1到3时失败了，因为这时版本号也变了，成功解决了ABA的问题。

java中还有哪些类可以解决ABA的问题？

AtomicMarkableReference，它不是维护一个版本号，而是维护一个boolean类型的标记，标记值有修改，了解一下。



### 参考：

1. [Java中atomic包中的原子操作类总结](https://www.jianshu.com/p/84c75074fa03)

2. [多线程并发下的线程安全的集合类的使用](https://www.jianshu.com/p/b8544e2c5fdb?utm_campaign)