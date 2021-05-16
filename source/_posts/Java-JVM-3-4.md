---
title: Java中的内存泄露和GC
date: 2019-06-09 19:18:59
tags:
 - Java
categories:
 - Java
 - JVM
---
虽然Java有垃圾收集器帮助实现内存自动管理，虽然GC有效的处理了大部分内存，但是并不能完全保证内存的不泄露。

<!--more-->

#### 内存泄露

内存泄露就是堆内存中不再使用的对象，但是垃圾回收期无法从内存中删除他们的情况，因此他们会被不必要的一直存在。，这种情况会耗尽内存资源并降低系统性能，最终以OOM终止。

垃圾回收器会定期删除未引用的对象，但它永远不会收集那些仍在引用的对象。

```css
两种内存溢出异常[注意内存溢出是error级别的]
1.StackOverFlowError:当请求的栈深度大于虚拟机所允许的最大深度
2.OutOfMemoryError:虚拟机在扩展栈时无法申请到足够的内存空间[一般都能设置扩大]
```

内存泄露的症状：

-   应用程序长时间连续运行时性能严重下降；
-   应用程序中的OutOfMemoryError堆错误；
-   自发且奇怪的应用程序崩溃；
-   应用程序偶尔会耗尽连接对象。

##### Java中内存泄露类型

1. static字段引起的内存泄露

   大量使用static字段会潜在的导致内存泄露，在Java中，静态字段通常拥有与整个应用程序相匹配的生命周期。

   解决办法：最大限度的减少静态变量的使用；单例模式时，依赖于延迟加载对象而不是立即加载方式。

2. 未关闭的资源导致内存泄露

   每当创建连接或者打开流时，JVM都会为这些资源分配内存。如果没有关闭连接，会导致持续占有内存。在任意情况下，资源留下的开放连接都会消耗内存，如果我们不处理，就会降低性能，甚至OOM。

   解决办法：使用finally块关闭资源；关闭资源的代码，不应该有异常；jdk1.7后，可以使用try-with-resource块。

3. 不正确的equals()和hashCode()

   在HashMap和HashSet这种集合中，常常用到equal()和hashCode()来比较对象，如果重写不合理，将会成为潜在的内存泄露问题。

   解决办法：用最佳的方式重写equals()和hashCode。

4. 引用了外部类的内部类

   非静态内部内的初始化，总是需要外部类的实例；默认情况下，每个非静态内部类都包含对其包含内的隐式引用，如果我们在应用程序中使用这个内部类对象，那么即使在我们的包含类对象超出范围后，它也不会被垃圾收集。

   解决办法：如果内部类不需要访问包含的类成员，考虑转换为静态类。

5. finalize()方法造成的内存泄露

   重写finalize()方法时，该类的对象不会立即被垃圾收集器收集，如果finalize()方法的代码有问题，那么会潜在的引发OOM；

   解决办法：避免重写finalize()。

6. 常量字符串造成的内存泄露

   如果我们读取一个很大的String对象，并调用了inter(），那么它将放到字符串池中，位于PermGen中，只要应用程序运行，该字符串就会保留，这就会占用内存，可能造成OOM。

   解决办法：增加PermGen的大小，-XX:MaxPermSize=512m；升级Java版本，JDK1.7后字符串池转移到了堆中。

7. 使用ThreadLocal造成内存泄露

   使用ThreadLocal时，每个线程只要处于存货状态就可保留对其ThreadLocal变量副本的隐式调用，且将保留其自己的副本。使用不当，就会引起内存泄露。

   一旦线程不在存在，ThreadLocals就应该被垃圾收集，而现在线程的创建都是使用线程池，线程池有线程重用的功能，因此线程就不会被垃圾回收器回收。所以使用到ThreadLocals来保留线程池中线程的变量副本时，ThreadLocals没有显示的删除时，就会一直保留在内存中，不会被垃圾回收。

   解决办法：不在使用ThreadLocal时，调用remove()方法，该方法删除了此变量的当前线程值。不要使用ThreadLocal.set(null)，它只是查找与当前线程关联的Map并将键值对设置为当前线程为null。

例子：

```java
public class MemErrorTest {
    public static void main(String[] args) {
        try {
            List<Object> list = new ArrayList<Object>();
            for(;;) {
                list.add(new Object()); //创建对象速度可能高于jvm回收速度
            }
        } catch (OutOfMemoryError e) {
            e.printStackTrace();
        }

        try {
            hi();//递归造成StackOverflowError 这边因为每运行一个方法将创建一个栈帧，栈帧创建太多无法继续申请到内存扩展
        } catch (StackOverflowError e) {
            e.printStackTrace();
        }

    }

    public static void hi() {
        hi();
    }
}
```

更多请参看：

1. <https://www.javatang.com/archives/tag/内存泄漏>
2. [定位Java内存泄漏](https://www.jianshu.com/p/3be49723d2f8)

### GC

> GC(Garbage Collection)：即垃圾回收器，诞生于1960年MIT的Lisp语言，主要是用来回收，释放垃圾占用的空间。

java GC泛指java的垃圾回收机制，该机制是java与C/C++的主要区别之一，我们在日常写java代码的时候，一般都不需要编写内存回收或者垃圾清理的代码，也不需要像C/C++那样做类似delete/free的操作。

对象的内存分配在java虚拟机的自动内存分配机制下，一般不容易出现内存泄漏问题。但是写代码难免会遇到一些特殊情况，比如OOM神马的。。尽管虚拟机内存的动态分配与内存回收技术很成熟，可万一出现了这样那样的内存溢出问题，那么将难以定位错误的原因所在。

从三个角度切入来学习GC

1. 哪些内存要回收

2. 什么时候回收

3. 怎么回收

**哪些内存要回收**

java内存模型中分为五大区域已经有所了解。我们知道`程序计数器`、`虚拟机栈`、`本地方法栈`，由线程而生，随线程而灭，其中栈中的栈帧随着方法的进入顺序的执行的入栈和出栈的操作，一个栈帧需要分配多少内存取决于具体的虚拟机实现并且在编译期间即确定下来【忽略JIT编译器做的优化，基本当成编译期间可知】，当方法或线程执行完毕后，内存就随着回收，因此无需关心。

而`Java堆`、`方法区`则不一样。方法区存放着类加载信息，但是一个接口中多个实现类需要的内存可能不太一样，一个方法中多个分支需要的内存也可能不一样【只有在运行期间才可知道这个方法创建了哪些对象没需要多少内存】，这部分内存的分配和回收都是动态的，gc关注的也正是这部分的内存。

Java堆是GC回收的“重点区域”。堆中基本存放着所有对象实例，gc进行回收前，第一件事就是确认哪些对象存活，哪些死去[即不可能再被引用]

**堆的回收区域**

为了高效的回收，jvm将堆分为三个区域

1. 新生代（Young Generation）NewSize和MaxNewSize分别可以控制年轻代的初始大小和最大的大小
2. 老年代（Old Generation）
3. 永久代（Permanent Generation）【1.8以后采用元空间，就不在堆中了】

[GC为什么要分代-R大的回答](https://www.zhihu.com/question/53613423/answer/135743258)

[关于元空间](http://lovestblog.cn/blog/2016/10/29/metaspace/)

**判断对象是否存活算法**

1. 引用计数算法
   早期判断对象是否存活大多都是以这种算法，这种算法判断很简单，简单来说就是给对象添加一个引用计数器，每当对象被引用一次就加1，引用失效时就减1。当为0的时候就判断对象不会再被引用。
   优点:实现简单效率高，被广泛使用与如python何游戏脚本语言上。
   缺点:难以解决循环引用的问题，就是假如两个对象互相引用已经不会再被其它其它引用，导致一直不会为0就无法进行回收。

2. 可达性分析算法
   目前主流的商用语言[如java、c#]采用的是可达性分析算法判断对象是否存活。这个算法有效**解决了循环利用的弊端**。
   它的基本思路是通过一个称为“GC Roots”的对象为起始点，搜索所经过的路径称为引用链，当一个对象到GC Roots没有任何引用跟它连接则证明对象是不可用的。

![MF5BEn.png](https://s2.ax1x.com/2019/11/07/MF5BEn.png)

可作为GC Roots的对象有四种

- 虚拟机栈(栈桢中的本地变量表)中的引用的对象，就是平时所指的java对象，存放在堆中。
- 方法区中的类静态属性引用的对象，一般指被static修饰引用的对象，加载类的时候就加载到内存中。
- 方法区中的常量引用的对象,
- 本地方法栈中JNI（native方法)引用的对象

即使可达性算法中不可达的对象，也不是一定要马上被回收，还有可能被抢救一下。网上例子很多，基本上和深入理解JVM一书讲的一样

要真正宣告对象死亡需经过两个过程。

1. 可达性分析后没有发现引用链
2. 查看对象是否有finalize方法，如果有重写且在方法内完成自救[比如再建立引用]，还是可以抢救一下，注意这边一个类的finalize只执行一次，这就会出现一样的代码第一次自救成功第二次失败的情况。[如果类重写finalize且还没调用过，会将这个对象放到一个叫做F-Queue的序列里，这边finalize不承诺一定会执行，这么做是因为如果里面死循环的话可能会时F-Queue队列处于等待，严重会导致内存崩溃，这是我们不希望看到的。]

**对象的生存还是死亡**

即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程:如果对象在进行可达性分析后发现没有与GC Roots 相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize() 方法。当对象没有覆盖finalize() 方法，或者finalize() 方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

 **如果这个对象被判定为有必要执行finalize()方法，那么这个对象将会放置在一个叫做F-Queuc的队列之中，并在稍后由一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它**。这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，这样做的原因是如果一个对象在finalizeO 方法中执行缓慢，或者发生了死循环(更极端的情况)，将很可能会导致F-Queue队列中其他对象永久处于等待，甚至导致整个内存回收系统崩溃。finalize() 方法是对象逃脱死亡命运的最后一次机公稍后GC将对F-QUCUC中的对象进行第二次小规模的标记，如果对象要在finalize()中成功扬救自己一只要重新与引用链上的任何- 一个对象建立关联即可，譬如把自己(this 关键字) 赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移除出“即将回收”的集合: 如果对象这时候还没有逃脱，那基本上它就真的被回收了。从代码清单3-2 中我们可以看到一个对象的finalize()被执行，但是它仍然可以存活。

```java
/*此代码演示了两点
 * 对象可以在GC时自我拯救
 * 这种自救只会有一次，因为一个对象的finalize方法只会被自动调用一次
 * */
public class FinalizeEscapeGC {
	public static FinalizeEscapeGC SAVE_HOOK=null;
	public void isAlive(){
		System.out.println("yes我还活着");
	}
	public void finalize() throws Throwable{
		super.finalize();
		System.out.println("执行finalize方法");
		FinalizeEscapeGC.SAVE_HOOK=this;//自救
	}
	public static void main(String[] args) throws InterruptedException{
		SAVE_HOOK=new FinalizeEscapeGC();
		//对象的第一次回收
		SAVE_HOOK=null;
		System.gc();
		//因为finalize方法的优先级很低所以暂停0.5秒等它
		Thread.sleep(500);
		if(SAVE_HOOK!=null){
			SAVE_HOOK.isAlive();
		}else{
			System.out.println("no我死了");
		}
		//下面的代码和上面的一样，但是这次自救却失败了
		//对象的第一次回收
		SAVE_HOOK=null;
		System.gc();
		Thread.sleep(500);
		if(SAVE_HOOK!=null){
			SAVE_HOOK.isAlive();
		}else{
			System.out.println("no我死了");
		}
	}
}
//结果：
//执行finalize方法
//yes我还活着
//no我死了
```

**HotSpot虚拟机如何实现可达性算法**

1. 枚举根节点

   问题：

   - 在从gc root向下查找引用链时，可作为GC ROOT的节点主要在全局性引用（常量、静态变量）和执行上下文（栈帧中的本地变量表），通常方法区就有数百兆，逐个检查消耗会很大
   - 在查找引用链过程中，需要保证引用链的一致性，即在分析过程中对象的引用关系不能再变化，否则分析准确性则无法得到保证。因此通常GC执行时会stop the world，停止所有执行线程，即使几乎不发生停顿的CMS收集器中，枚举根节点也是需要停顿的。

   OopMap：在HotSpot中，使用的是一种称为OopMap的数据结构来存储对象内什么偏移量存储的是什么类型的数据的映射关系，在JIT编译过程中，也会在特定位置记录下栈和寄存器中的那些位置和引用的，这样GC在扫描时就可以直接获得这些信息。

2. 安全点(safe point)

   实际上，JVM并非为所有指令都生成OopMap，只是在特定位置记录这些信息，这些位置就成为安全点。即，程序并非在所有地方都能停顿下来执行GC，只有在到达安全点时，才可以将线程停顿下来GC。

   选定：安全点的选定通常是以是否能让程序长时间执行的特征选定的。例如方法调用、执行跳转、异常跳转等处

   暂停线程方案：在GC发生时需要让线程停顿下来，让线程停顿下来的方案有两种，抢先式中断和主动式中断

   - 抢先式中断：在GC发生时先中断所有线程，如果线程不在安全点上，则启动该线程使其执行到安全点后挂起。**几乎已没有**虚拟机只用此种方式
   - 主动式中断：不需要直接对线程进行操作，在线程执行时主动轮询这个标识，若中断标识为真，在线程自己中断挂起。这个标识和安全点是重合的　　　

3. 安全区域（safe region）

   上面的安全点检查仿佛完全解决了如何进入GC的问题，但只有安全点还是不够的，安全点只解决了那些在运行的程序，保证了他们可以运行到安全点并挂起，但如果有些线程此时并未执行，例如处于sleep或blocked状态的线程，就无法响应JVM的中断请求，这是就用到了安全区域

   定义：安全区域是指在此区域内，对象的引用关系不会发生变化（即不会影响枚举根节点）

   原理：当线程运行到安全区域时会将自己标识，在JVM准备进行GC时将视这些线程为安全的，不影响GC，当线程运行完毕要离开安全区域时，线程会检查JVM是否在枚举根节点，若是，则等待完成后再离开安全区域继续执行。

#### 垃圾收集算法

1. 标记/清除算法【最基础】
2. 复制算法
3. 标记/整理算法

jvm采用`分代收集算法`对不同区域采用不同的回收算法。

[Java虚拟机：GC算法深度解析](https://www.cnblogs.com/fangfuhai/p/7203468.html?utm_source=itdadao&utm_medium=referral)

**新生代采用复制算法**

新生代中因为对象都是"朝生夕死的"，【深入理解JVM虚拟机上说98%的对象,不知道是不是这么多，总之就是存活率很低】，适用于复制算法【复制算法比较适合用于存活率低的内存区域】。它优化了标记/清除算法的效率和内存碎片问题，且JVM不以5:5分配内存【由于存活率低，不需要复制保留那么大的区域造成空间上的浪费，因此不需要按1:1【原有区域:保留空间】划分内存区域，而是将内存分为一块Eden空间和From Survivor、To Survivor【保留空间】，三者默认比例为8:1:1，优先使用Eden区，若Eden区满，则将对象复制到第二块内存区上。但是不能保证每次回收都只有不多于10%的对象存货，所以Survivor区不够的话，则会依赖老年代年存进行分配】。

GC开始时，对象只会存于Eden和From Survivor区域，To Survivor【保留空间】为空。

GC进行时，Eden区所有存活的对象都被复制到To Survivor区，而From Survivor区中，仍存活的对象会根据它们的年龄值决定去向，年龄值达到年龄阈值(默认15是因为对象头中年龄战4bit，新生代每熬过一次垃圾回收，年龄+1)，则移到老年代，没有达到则复制到To Survivor。

**老年代采用`标记/清除算法`或`标记/整理算法`**

由于老年代存活率高，没有额外空间给他做担保，必须使用这两种算法。

##### 枚举根节点算法

`GC Roots` 被虚拟机用来判断对象是否存活

> 可作为GC Roos的节点主要是在一些全局引用【如常量或静态属性】、执行上下文【如栈帧中本地变量表】中。那么如何在这么多全局变量和本地变量表找到【枚举】根节点将是个问题。

可达性分析算法需考虑

1. 如果方法区几百兆，一个个检查里面的引用，将耗费大量资源。

2. 在分析时，需保证这个对象引用关系不再变化，否则结果将不准确。【因此GC进行时需停掉其它所有java执行线程(Sun把这种行为称为‘Stop the World’)，即使是号称几乎不会停顿的CMS收集器，枚举根节点时也需停掉线程】

解决办法:实际上当系统停下来后JVM不需要一个个检查引用，而是通过OopMap数据结构【HotSpot的叫法】来标记对象引用。

虚拟机先得知哪些地方存放对象的引用，在类加载完时。HotSpot把对象内什么偏移量什么类型的数据算出来，在jit编译过程中，也会在特定位置记录下栈和寄存器哪些位置是引用，这样GC在扫描时就可以知道这些信息。【目前主流JVM使用准确式GC】

OopMap可以帮助HotSpot快速且准确完成GC Roots枚举以及确定相关信息。但是也存在一个问题，可能导致引用关系变化。

这个时候有个safepoint(安全点)的概念。

HotSpot中GC不是在任意位置都可以进入，而只能在safepoint处进入。 GC时对一个Java线程来说，它要么处在safepoint,要么不在safepoint。

safepoint不能太少，否则GC等待的时间会很久

safepoint不能太多，否则将增加运行GC的负担

安全点主要存放的位置

1. 循环的末尾 
2. 方法临返回前/调用方法的call指令后 
3. 可能抛异常的位置

#### 垃圾收集器

如果说垃圾回收算法是内存回收的方法论，那么垃圾收集器就是具体实现。jvm会结合针对不同的场景及用户的配置使用不同的收集器。

```css
年轻代收集器
Serial、ParNew、Parallel Scavenge
老年代收集器
Serial Old、Parallel Old、CMS收集器
特殊收集器
G1收集器[新型，不在年轻、老年代范畴内]
```

![MFIbiq.png](https://s2.ax1x.com/2019/11/07/MFIbiq.png)

##### 新生代收集器

1.  Serial

   最基本、发展最久的收集器，在jdk3以前是gc收集器的唯一选择，Serial是单线程收集器，Serial收集器只能使用一条线程进行收集工作，在收集的时候必须得停掉其它线程，等待收集工作完成其它线程才可以继续工作。

   ```css
   虽然Serial看起来很坑，需停掉别的线程以完成自己的gc工作，但是也不是完全没用的，比如说Serial在运行在Client模式下优于其它收集器[简单高效,不过一般都是用Server模式，64bit的jvm甚至没Client模式]
   ```

   [JVM的Client模式与Server模式](https://www.cnblogs.com/wxw7blog/p/7221756.html)

   > JVM有两种运行模式Server与Client。两种模式的区别在于，Client模式启动速度较快，Server模式启动较慢；但是启动进入稳定期长期运行之后Server模式的程序运行速度比Client要快很多。这是因为Server模式启动的JVM采用的是重量级的虚拟机，对程序采用了更多的优化；而Client模式启动的JVM采用的是轻量级的虚拟机。所以Server启动慢，但稳定后速度比Client远远要快。

   优点:对于Client模式下的jvm来说是个好的选择。适用于单核CPU【现在基本都是多核了】
   缺点:收集时要暂停其它线程，有点浪费资源，多核下显得。

2. ParNew收集器

   可以认为是Serial的升级版，因为它支持多线程[GC线程]，而且收集算法、Stop The World、回收策略和Serial一样，就是可以有多个GC线程并发运行，它是HotSpot第一个真正意义实现并发的收集器。默认开启线程数和当前cpu数量相同【几核就是几个，超线程cpu的话就不清楚了 - -】，如果cpu核数很多不想用那么多，可以通过-XX:ParallelGCThreads来控制垃圾收集线程的数量。

   优点:

   1. 支持多线程，多核CPU下可以充分的利用CPU资源
   2. 运行在Server模式下新生代首选的收集器【重点是因为新生代的这几个收集器只有它和Serial可以配合CMS收集器一起使用】

   缺点: 在单核下表现不会比Serial好，由于在单核能利用多核的优势，在线程收集过程中可能会出现频繁上下文切换，导致额外的开销。

3. Parallel Scavenge

   采用复制算法的收集器，和ParNew一样支持多线程。

   但是该收集器重点关心的是吞吐量【吞吐量 = 代码运行时间 / (代码运行时间 + 垃圾收集时间) 如果代码运行100min垃圾收集1min，则为99%】

   对于用户界面，适合使用GC停顿时间短,不然因为卡顿导致交互界面卡顿将很影响用户体验。

   对于后台，高吞吐量可以高效率的利用cpu尽快完成程序运算任务，适合后台运算

   > Parallel Scavenge注重吞吐量，所以也成为"吞吐量优先"收集器。
   >

##### 老年代收集器

1. Serial Old

   和新生代的Serial一样为单线程，Serial的老年代版本，不过它采用"标记-整理算法"，这个模式主要是给Client模式下的JVM使用。

   如果是Server模式有两大用途

   1. jdk5前和Parallel Scavenge搭配使用，jdk5前也只有这个老年代收集器可以和它搭配。

   2. 作为CMS收集器的后备。

2. Parallel Old

   支持多线程，Parallel Scavenge的老年版本，jdk6开始出现， 采用"标记-整理算法"【老年代的收集器大都采用此算法】

   在jdk6以前，新生代的Parallel Scavenge只能和Serial Old配合使用【根据图，没有这个的话只剩Serial Old，而Parallel Scavenge又不能和CMS配合使用】，而且Serial Old为单线程Server模式下会拖后腿【多核cpu下无法充分利用】，这种结合并不能让应用的吞吐量最大化。

   > Parallel Old的出现结合Parallel Scavenge，真正的形成“吞吐量优先”的收集器组合。

3. CMS

   CMS收集器(Concurrent Mark Sweep)是以一种获取最短回收停顿时间为目标的收集器。【重视响应，可以带来好的用户体验，被sun称为并发低停顿收集器】

   ```css
   启用CMS：-XX:+UseConcMarkSweepGC
   ```

   正如其名，CMS采用的是"标记-清除"(Mark Sweep)算法，而且是支持并发(Concurrent)的

   它的运作分为4个阶段

   1. 初始标记:标记一下GC Roots能直接关联到的对象，速度很快
   2. 并发标记:GC Roots Tarcing过程，即可达性分析
   3. 重新标记:为了修正因并发标记期间用户程序运作而产生变动的那一部分对象的标记记录，会有些许停顿，时间上一般 初始标记 < 重新标记 < 并发标记
   4. 并发清除

   以上初始标记和重新标记需要stw(停掉其它运行java线程)

   之所以说CMS的用户体验好，是因为CMS收集器的内存回收工作是可以和用户线程一起并发执行。

   总体上CMS是款优秀的收集器，但是它也有些缺点。

   1. cms堆cpu特别敏感，cms运行线程和应用程序并发执行需要多核cpu，如果cpu核数多的话可以发挥它并发执行的优势，但是cms默认配置启动的时候垃圾线程数为 (cpu数量+3)/4，它的性能很容易受cpu核数影响，当cpu的数目少的时候比如说为为2核，如果这个时候cpu运算压力比较大，还要分一半给cms运作，这可能会很大程度的影响到计算机性能。

   2. cms无法处理浮动垃圾，可能导致Concurrent Mode Failure（并发模式故障）而触发full GC

   3. 由于cms是采用"标记-清除“算法,因此就会存在垃圾碎片的问题，为了解决这个问题cms提供了 **-XX:+UseCMSCompactAtFullCollection**选项，这个选项相当于一个开关【默认开启】，用于CMS顶不住要进行full GC时开启内存碎片合并，内存整理的过程是无法并发的，且开启这个选项会影响性能(比如停顿时间变长)

4. G1收集器

   G1(garbage first:尽可能多收垃圾，避免full gc)收集器是当前最为前沿的收集器之一(1.7以后才开始有)，同cms一样也是关注降低延迟，是用于替代cms功能更为强大的新型收集器，因为它解决了cms产生空间碎片等一系列缺陷。

   > 摘自甲骨文:适用于 Java HotSpot VM 的低暂停、服务器风格的分代式垃圾回收器。G1 GC 使用并发和并行阶段实现其目标暂停时间，并保持良好的吞吐量。当 G1 GC 确定有必要进行垃圾回收时，它会先收集存活数据最少的区域（垃圾优先)
   >
   > g1的特别之处在于它强化了分区，弱化了分代的概念，是区域化、增量式的收集器，它不属于新生代也不属于老年代收集器。
   >
   > 用到的算法为标记-清理、复制算法

   ```css
   jdk1.7,1.8的都是默认关闭的，更高版本的还不知道
   开启选项 -XX:+UseG1GC 
   比如在tomcat的catania.sh启动参数加上
   ```

   g1是区域化的，它将java堆内存划分为若干个大小相同的区域【region】，jvm可以设置每个region的大小(1-32m,大小得看堆内存大小，必须是2的幂),它会根据当前的堆内存分配合理的region大小。

   g1通过并发(并行)标记阶段查找老年代存活对象，通过并行复制压缩存活对象【这样可以省出连续空间供大对象使用】。

   g1将一组或多组区域中存活对象以增量并行的方式复制到不同区域进行压缩，从而减少堆碎片，目标是尽可能多回收堆空间【垃圾优先】，且尽可能不超出暂停目标以达到低延迟的目的。

   g1提供三种垃圾回收模式 young gc、mixed gc 和 full gc,不像其它的收集器，根据区域而不是分代，新生代老年代的对象它都能回收。

   几个重要的默认值，更多的查看官方文档[oracle官方g1中文文档](https://www.oracle.com/technetwork/cn/articles/java/g1gc-1984535-zhs.html)

   ```
   g1是自适应的回收器，提供了若干个默认值，无需修改就可高效运作
   -XX:G1HeapRegionSize=n  设置g1 region大小，不设置的话自己会根据堆大小算，目标是根据最小堆内存划分2048个区域
   -XX:MaxGCPauseMillis=200 最大停顿时间 默认200毫秒
   ```

#### Minor GC、Major GC、FULL GC、mixed gc

1. Minor GC

   在年轻代Young space(包括Eden区和Survivor区)中的垃圾回收称之为 Minor GC,Minor GC只会清理年轻代.

2. Major GC

   Major GC清理老年代(old GC)，但是通常也可以指和Full GC是等价，因为收集老年代的时候往往也会伴随着升级年轻代，收集整个Java堆。所以有人问的时候需问清楚它指的是full GC还是old GC。

3. Full GC

   full gc是对新生代、老年代、永久代【jdk1.8后没有这个概念了】统一的回收。

   【知乎R大的回答:收集整个堆，包括young gen、old gen、perm gen（如果存在的话)、元空间(1.8及以上)等所有部分的模式】

4. mixed GC【g1特有】

   混合GC

   收集整个young gen以及部分old gen的GC。只有G1有这个模式

#### 查看GC日志

**简单日志查看**

要看得懂并理解GC，需要看懂GC日志。

这边我在idea上试了个小例子，需要在idea配置参数(-XX:+PrintGCDetails)。

```java
public class GCtest {
    public static void main(String[] args) {
        for(int i = 0; i < 10000; i++) {
            List<String> list = new ArrayList<>();
            list.add("aaaaaaaaaaaaa");
        }
        System.gc();
    }
}
```

#### 几个疑问

1. GC是怎么判断对象是被标记的

   通过枚举根节点的方式，通过jvm提供的一种oopMap的数据结构，简单来说就是不要再通过去遍历内存里的东西，而是通过OOPMap的数据结构去记录该记录的信息,比如说它可以不用去遍历整个栈，而是扫描栈上面引用的信息并记录下来。

   总结:通过OOPMap把栈上代表引用的位置全部记录下来，避免全栈扫描，加快枚举根节点的速度，除此之外还有一个极为重要的作用，可以帮HotSpot实现准确式GC【这边的准确关键就是类型，可以根据给定位置的某块数据知道它的准确类型，HotSpot是通过oopMap外部记录下这些信息，存成映射表一样的东西】。

2. 什么时候触发GC

   简单来说，触发的条件就是GC算法区域满了或将满了。

   ```css
   minor GC(young GC):当年轻代中eden区分配满的时候触发[值得一提的是因为young GC后部分存活的对象会已到老年代(比如对象熬过15轮)，所以过后old gen的占用量通常会变高]
   
   full GC:
   1.手动调用System.gc()方法 [增加了full GC频率，不建议使用而是让jvm自己管理内存，可以设置-XX:+ DisableExplicitGC来禁止RMI调用System.gc]
   2.发现perm gen（如果存在永久代的话)需分配空间但已经没有足够空间
   3.老年代空间不足，比如说新生代的大对象大数组晋升到老年代就可能导致老年代空间不足。
   4.CMS GC时出现Promotion Faield[pf]
   5.统计得到的Minor GC晋升到旧生代的平均大小大于老年代的剩余空间。
   这个比较难理解，这是HotSpot为了避免由于新生代晋升到老年代导致老年代空间不足而触发的FUll GC。
   比如程序第一次触发Minor GC后，有5m的对象晋升到老年代，姑且现在平均算5m，那么下次Minor GC发生时，先判断现在老年代剩余空间大小是否超过5m，如果小于5m，则HotSpot则会触发full GC(这点挺智能的)
   Promotion Faield:minor GC时 survivor space放不下[满了或对象太大]，对象只能放到老年代，而老年代也放不下会导致这个错误。
   Concurrent Model Failure:cms时特有的错误，因为cms时垃圾清理和用户线程可以是并发执行的，如果在清理的过程中
   可能原因：
   1 cms触发太晚，可以把XX:CMSInitiatingOccupancyFraction调小[比如-XX:CMSInitiatingOccupancyFraction=70 是指设定CMS在对内存占用率达到70%的时候开始GC(因为CMS会有浮动垃圾,所以一般都较早启动GC)]
   2 垃圾产生速度大于清理速度，可能是晋升阈值设置过小，Survivor空间小导致跑到老年代，eden区太小，存在大对象、数组对象等情况
   3.空间碎片过多，可以开启空间碎片整理并合理设置周期时间
   ```

   > full gc导致了concurrent mode failure，而不是因为concurrent mode failure错误导致触发full gc，真正触发full gc的原因可能是ygc时发生的promotion failure。

3. cms收集器是否会扫描年轻代

   会，在初始标记的时候会扫描新生代。

   虽然cms是老年代收集器，但是我们知道年轻代的对象是可以晋升为老年代的，为了空间分配担保，还是有必要去扫描年轻代。

4. 什么是空间分配担保

   在minor gc前，jvm会先检查老年代最大可用空间是否大于新生代所有对象总空间，如果是的话，则minor gc可以确保是安全的，

   > 如果担保失败,会检查一个配置(HandlePromotionFailire),即是否允许担保失败。
   >
   > 如果允许:继续检查老年代最大可用可用的连续空间是否大于之前晋升的平均大小，比如说剩10m，之前每次都有9m左右的新生代到老年代，那么将尝试一次minor gc(大于的情况)，这会比较冒险。
   >
   > 如果不允许，而且还小于的情况，则会触发full gc。【为了避免经常full GC 该参数建议打开】
   >
   > 这边为什么说是冒险是因为minor gc过后如果出现大对象，由于新生代采用复制算法，survivor无法容纳将跑到老年代，所以才会去计算之前的平均值作为一种担保的条件与老年代剩余空间比较，这就是分配担保。
   >
   > 这种担保是动态概率的手段，但是也有可能出现之前平均都比较低，突然有一次minor gc对象变得很多远高于以往的平均值，这个时候就会导致担保失败【Handle Promotion Failure】，这就只好再失败后再触发一次FULL GC，

5. 为什么复制算法要分两个Survivor，而不直接移到老年代

   这样做的话效率可能会更高，但是old区一般都是熬过多次可达性分析算法过后的存活的对象，要求比较苛刻且空间有限，而不能直接移过去，这将导致一系列问题(比如老年代容易被撑爆)

   分两个Survivor(from/to)，自然是为了保证复制算法运行以提高效率。

6. 各个版本的JVM使用的垃圾收集器是怎么样的

   准确来说，垃圾收集器的使用跟当前jvm也有很大的关系，比如说g1是jdk7以后的版本才开始出现。

   并不是所有的垃圾收集器都是默认开启的，有些得通过设置相应的开关参数才会使用。比如说cms，需设置(XX:+UseConcMarkSweepGC)

   这边有几个实用的命令，比如说server模式下

   ```shell
   #UnlockExperimentalVMOptions UnlockDiagnosticVMOptions解锁获取jvm参数，PrintFlagsFinal用于输出xx相关参数，以Benchmark类测试，这边会有很多结果 大都看不懂- - 在这边查(usexxxxxxgc会看到jvm不同收集器的开关情况)
   java -server -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal Benchmark
   
   #后面跟| grep ":"获取已赋值的参数[加:代表被赋值过]
   java -server -XX:+UnlockExperimentalVMOptions -XX:+UnlockDiagnosticVMOptions -XX:+PrintFlagsFinal Benchmark| grep ":"
   
   #获得用户自定义的设置或者jvm设置的详细的xx参数和值
   java -server -XX:+PrintCommandLineFlags Benchmark
   ```

   本人用的jdk8，这边UseParallelGC为true，参考深入理解jvm那本书说这个是Parallel Scavenge+Serial old搭配组合的开关，但是网上又说8默认是Parallel Scavenge+Parallel Old,我还是信书的吧 - -。

   ![MFTWDg.png](https://s2.ax1x.com/2019/11/07/MFTWDg.png)

   > 据说更高版本的jvm默认使用g1

7. stop the world具体是什么，有没有办法避免

   stop the world简单来说就是gc的时候，停掉除gc外的java线程。

   无论什么gc都难以避免停顿，即使是g1也会在初始标记阶段发生，stw并不可怕，可以尽可能的减少停顿时间。

8. 新生代什么样的情况会晋升为老年代

   对象优先分配在eden区，eden区满时会触发一次minor GC

   > 对象晋升规则
   >  1 长期存活的对象进入老年代，对象每熬过一次GC年龄+1(默认年龄阈值15，可配置)。
   >  2 对象太大新生代无法容纳则会分配到老年代
   >  3 eden区满了，进行minor gc后，eden和一个survivor区仍然存活的对象无法放到(to survivor区)则会通过分配担保机制放到老年代，这种情况一般是minor gc后新生代存活的对象太多。
   >  4 动态年龄判定，为了使内存分配更灵活，jvm不一定要求对象年龄达到MaxTenuringThreshold(15)才晋升为老年代，若survior区相同年龄对象总大小大于survior区空间的一半，则大于等于这个年龄的对象将会在minor gc时移到老年代

9. 怎么理解g1，适用于什么场景

   > G1 GC 是区域化、并行-并发、增量式垃圾回收器，相比其他 HotSpot 垃圾回收器，可提供更多可预测的暂停。增量的特性使 G1 GC 适用于更大的堆，在最坏的情况下仍能提供不错的响应。G1 GC 的自适应特性使 JVM 命令行只需要软实时暂停时间目标的最大值以及 Java 堆大小的最大值和最小值，即可开始工作。

   g1不再区分老年代、年轻代这样的内存空间，这是较以往收集器很大的差异，所有的内存空间就是一块划分为不同子区域，每个区域大小为1m-32m，最多支持的内存为64g左右，且由于它为了的特性适用于大内存机器。

   ![MFT5Us.png](https://s2.ax1x.com/2019/11/07/MFT5Us.png)

   适用场景:

   1. 像cms能与应用程序并发执行，GC停顿短【短而且可控】，用户体验好的场景。

   2. 面向服务端，大内存，高cpu的应用机器。【网上说差不多是6g或更大】

   3. 应用在运行过程中经常会产生大量内存碎片，需要压缩空间【比cms好的地方之一，g1具备压缩功能】。

### 参考资料：

1. https://blog.csdn.net/l540675759/article/details/73733763
2. https://www.iteye.com/topic/587995
3. https://www.geeksforgeeks.org/types-references-java/
4. https://blog.csdn.net/aitangyong/article/details/39453365
5. [详解 Java 中的四种引用](https://blog.csdn.net/hacker_zhidian/article/details/83043270)
6. [深入理解JVM-内存模型（jmm）和GC](https://www.jianshu.com/p/76959115d486)
7. [JVM内存模型、指令重排、内存屏障概念解析](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fchenyangyao%2Fp%2F5269622.html)
8. [Java对象头](https://www.jianshu.com/p/9c19eb0ea4d8)
9. [GC收集器](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fduke2016%2Fp%2F6250766.html)
10. [Major GC和Full GC的区别](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F41922036%2Fanswer%2F93079526)
11. [JVM 垃圾回收 Minor gc vs Major gc vs Full gc](https://links.jianshu.com/go?to=http%3A%2F%2Fm635674608.iteye.com%2Fblog%2F2236137)
12. [关于准确式GC、保守式GC](https://links.jianshu.com/go?to=http%3A%2F%2Frednaxelafx.iteye.com%2Fblog%2F1044951)
13. [关于CMS垃圾收集算法的一些疑惑](https://www.jianshu.com/p/55670407fdb9)
14. [图解cms](https://www.jianshu.com/p/2a1b2f17d3e4)
15. [G1垃圾收集器介绍](https://www.jianshu.com/p/0f1f5adffdc1)
16. [详解cms回收机制](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2FlittleLord%2Fp%2F5380624.html)

