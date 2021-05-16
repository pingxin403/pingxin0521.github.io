---
title: JVM 垃圾回收机制
date: 2019-11-08 13:18:59
tags:
 - Java
categories:
 - Java
 - JVM
---
![l7lrPs.png](https://s2.ax1x.com/2020/01/13/l7lrPs.png)

<!--more-->

#### 哪些内存需要回收？

猿们都知道JVM的内存结构包括五大区域：程序计数器、虚拟机栈、本地方法栈、堆区、方法区。其中程序计数器、虚拟机栈、本地方法栈3个区域随线程而生、随线程而灭，因此这几个区域的内存分配和回收都具备确定性，就不需要过多考虑回收的问题，因为方法结束或者线程结束时，内存自然就跟随着回收了。而Java堆区和方法区则不一样、不一样!(怎么不一样说的朗朗上口)，这部分内存的分配和回收是动态的，正是垃圾收集器所需关注的部分。

垃圾收集器在对堆区和方法区进行回收前，首先要确定这些区域的对象哪些可以被回收，哪些暂时还不能回收，这就要用到判断对象是否存活的算法！

1. 引用计数算法

   引用计数是垃圾收集器中的早期策略。在这种方法中，堆中每个对象实例都有一个引用计数。当一个对象被创建时，就将该对象实例分配给一个变量，该变量计数设置为1。当任何其它变量被赋值为这个对象的引用时，计数加1（a = b,则b引用的对象实例的计数器+1），但当一个对象实例的某个引用超过了生命周期或者被设置为一个新值时，对象实例的引用计数器减1。任何引用计数器为0的对象实例可以被当作垃圾收集。当一个对象实例被垃圾收集时，它引用的任何对象实例的引用计数器减1。

   **优点**：引用计数收集器可以很快的执行，交织在程序运行中。对程序需要不被长时间打断的实时环境比较有利。

   **缺点**：无法检测出循环引用。如父对象有一个对子对象的引用，子对象反过来引用父对象。这样，他们的引用计数永远不可能为0。

2. 可达性分析算法

   可达性分析算法是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，从一个节点GC ROOT开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点，无用的节点将会被判定为是可回收的对象。

   ![l71pRI.png](https://s2.ax1x.com/2020/01/13/l71pRI.png)

   在Java语言中，可作为GC Roots的对象包括下面几种：

   - 虚拟机栈中引用的对象（栈帧中的本地变量表）；
   - 方法区中类静态属性引用的对象；
   - 方法区中常量引用的对象；
   - 本地方法栈中JNI（Native方法）引用的对象。

引用分类:<https://pingxin0521.coding.me/2019/07/01/Java-JVM-2-1/>

**对象死亡（被回收）前的最后一次挣扎**

即使在可达性分析算法中不可达的对象，也并非是“非死不可”，这时候它们暂时处于“缓刑”阶段，要真正宣告一个对象死亡，至少要经历两次标记过程。

第一次标记：如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它将会被第一次标记；

第二次标记：第一次标记后接着会进行一次筛选，筛选的条件是此对象是否有必要执行finalize()方法。在finalize()方法中没有重新与引用链建立关联关系的，将被进行第二次标记。

第二次标记成功的对象将真的会被回收，如果对象在finalize()方法中重新与引用链建立了关联关系，那么将会逃离本次回收，继续存活。

**方法区如何判断是否需要回收**

方法区存储内容是否需要回收的判断可就不一样咯。方法区主要回收的内容有：废弃常量和无用的类。对于废弃常量也可通过引用的可达性来判断，但是对于无用的类则需要同时满足下面3个条件：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；
- 加载该类的`ClassLoader`已经被回收；
- 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

#### 垃圾回收算法

1. 标记-清除算法

   标记-清除算法采用从根集合（GC Roots）进行扫描，对存活的对象进行标记，标记完毕后，再扫描整个空间中未被标记的对象，进行回收，如下图所示。标记-清除算法不需要进行对象的移动，只需对不存活的对象进行处理，在存活对象比较多的情况下极为高效，但由于标记-清除算法直接回收不存活的对象，因此会造成内存碎片。

   ![l71jXV.png](https://s2.ax1x.com/2020/01/13/l71jXV.png)

2. 复制算法

   复制算法的提出是为了克服句柄的开销和解决内存碎片的问题。它开始时把堆分成 一个对象 面和多个空闲面， 程序从对象面为对象分配空间，当对象满了，基于copying算法的垃圾 收集就从根集合（GC Roots）中扫描活动对象，并将每个 活动对象复制到空闲面(使得活动对象所占的内存之间没有空闲洞)，这样空闲面变成了对象面，原来的对象面变成了空闲面，程序会在新的对象面中分配内存。

   ![l73CtJ.png](https://s2.ax1x.com/2020/01/13/l73CtJ.png)

3. 标记-整理算法

   标记-整理算法采用标记-清除算法一样的方式进行对象的标记，但在清除时不同，在回收不存活的对象占用的空间后，会将所有的存活对象往左端空闲空间移动，并更新对应的指针。标记-整理算法是在标记-清除算法的基础上，又进行了对象的移动，因此成本更高，但是却解决了内存碎片的问题。具体流程见下图：

   ![l73k11.png](https://s2.ax1x.com/2020/01/13/l73k11.png)

4. 分代收集算法

   分代收集算法是目前大部分JVM的垃圾收集器采用的算法。它的核心思想是根据对象存活的生命周期将内存划分为若干个不同的区域。一般情况下将堆区划分为老年代（Tenured Generation）和新生代（Young Generation），在堆区之外还有一个代就是永久代（Permanet Generation）。老年代的特点是每次垃圾收集时只有少量对象需要被回收，而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。

   ![l73EX6.png](https://s2.ax1x.com/2020/01/13/l73EX6.png)

   **年轻代（Young Generation）的回收算法**

   - 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。

   - 新生代内存按照8:1:1的比例分为一个eden区和两个survivor(survivor0,survivor1)区。一个Eden区，两个 Survivor区(一般而言)。大部分对象在Eden区中生成。回收时先将eden区存活对象复制到一个survivor0区，然后清空eden区，当这个survivor0区也存放满了时，则将eden区和survivor0区存活对象复制到另一个survivor1区，然后清空eden和这个survivor0区，此时survivor0区是空的，然后将survivor0区和survivor1区交换，即保持survivor1区为空， 如此往复。
   - 当survivor1区不足以存放 eden和survivor0的存活对象时，就将存活对象直接存放到老年代。若是老年代也满了就会触发一次Full GC，也就是新生代、老年代都进行回收。
   - 新生代发生的GC也叫做Minor GC，MinorGC发生频率比较高(不一定等Eden区满了才触发)。

   **年老代（Old Generation）的回收算法**

   - 在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
   - 内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。

   **持久代（Permanent Generation）的回收算法**

   用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。持久代也称方法区

#### 垃圾收集器

下面一张图是HotSpot虚拟机包含的所有收集器，图是借用过来滴：

![l73Jnf.png](https://s2.ax1x.com/2020/01/13/l73Jnf.png)

- Serial收集器（复制算法)
  新生代单线程收集器，标记和清理都是单线程，优点是简单高效。是client级别默认的GC方式，可以通过`-XX:+UseSerialGC`来强制指定。
- Serial Old收集器(标记-整理算法)
  老年代单线程收集器，Serial收集器的老年代版本。
- ParNew收集器(停止-复制算法)　
  新生代收集器，可以认为是Serial收集器的多线程版本,在多核CPU环境下有着比Serial更好的表现。
- Parallel Scavenge收集器(停止-复制算法)
  并行收集器，追求高吞吐量，高效利用CPU。吞吐量一般为99%， 吞吐量= 用户线程时间/(用户线程时间+GC线程时间)。适合后台应用等对交互相应要求不高的场景。是server级别默认采用的GC方式，可用`-XX:+UseParallelGC`来强制指定，用`-XX:ParallelGCThreads=4`来指定线程数。
- Parallel Old收集器(停止-复制算法)
  Parallel Scavenge收集器的老年代版本，并行收集器，吞吐量优先。`- XX:+UseParallelOldGC`
- CMS(Concurrent Mark Sweep)收集器（标记-清理算法）
  高并发、低停顿，追求最短GC回收停顿时间，cpu占用比较高，响应时间快，停顿时间短，多核cpu 追求高响应时间的选择。

#### GC是什么时候触发的

由于对象进行了分代处理，因此垃圾回收区域、时间也不一样。GC有两种类型：Minor GC和Full GC。

**Minor GC**

一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。

**Full GC**

对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个堆进行回收，所以比Scavenge GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于Full GC的调节。有如下原因可能导致Full GC：

- 年老代（Tenured）被写满；
- 持久代（Perm）被写满；
- System.gc()被显示调用；
- 上一次GC之后Heap的各域分配策略动态变化；

#### 7种JVM垃圾收集器

##### JVM垃圾收集器发展历程

1. 第一阶段，Serial（串行）收集器

   在jdk1.3.1之前，java虚拟机仅仅能使用Serial收集器。 Serial收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅是说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作，更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。

2. 第二阶段，Parallel（并行）收集器

   Parallel收集器也称吞吐量收集器，相比Serial收集器，Parallel最主要的优势在于使用多线程去完成垃圾清理工作，这样可以充分利用多核的特性，大幅降低gc时间。

3. 第三阶段，CMS（并发）收集器

   CMS收集器在Minor GC时会暂停所有的应用线程，并以多线程的方式进行垃圾回收。在Full GC时不再暂停应用线程，而是使用若干个后台线程定期的对老年代空间进行扫描，及时回收其中不再使用的对象。

4. 第四阶段，G1（并发）收集器

   G1收集器（或者垃圾优先收集器）的设计初衷是为了尽量缩短处理超大堆（大于4GB）时产生的停顿。相对于CMS的优势而言是内存碎片的产生率大大降低。

##### 常见的垃圾收集器有3类

![l70GTI.png](https://s2.ax1x.com/2020/01/13/l70GTI.png)

1. 新生代的收集器包括

   Serial

   PraNew

   Parallel Scavenge

2. 老年代的收集器包括

   Serial Old

   Parallel Old

   CMS

3. 回收整个Java堆(新生代和老年代)

   G1收集器

##### 新生代垃圾收集器

1. Serial串行收集器-复制算法

   Serial收集器是新生代单线程收集器，优点是简单高效，算是最基本、发展历史最悠久的收集器。它在进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集完成。

   ![l70Ykt.png](https://s2.ax1x.com/2020/01/13/l70Ykt.png)

2. ParNew收集器-复制算法

   ParNew收集器是**新生代并行收集器**，其实就是Serial收集器的多线程版本。

   ![l70w6g.png](https://s2.ax1x.com/2020/01/13/l70w6g.png)

   除了使用多线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数、收集算法、Stop The World、对象分配规则、回收策略等都与Serial 收集器完全一样。

3. Parallel Scavenge（并行回收）收集器-复制算法

   Parallel Scavenge收集器是新生代并行收集器，追求高吞吐量，高效利用 CPU。

   该收集器的目标是达到一个可控制的吞吐量（Throughput）。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即 吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）

   停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可用高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

##### 老年代垃圾收集器

1. Serial Old 收集器-标记整理算法

   Serial Old是Serial收集器的老年代版本，它同样是一个单线程(串行)收集器，使用标记整理算法。这个收集器的主要意义也是在于**给Client模式下的虚拟机使用**。

   **如果在Server模式下，主要两大用途：**

   - 在JDK1.5以及之前的版本中与Parallel Scavenge收集器搭配使用
   - 作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用

2. Parallel Old 收集器-标记整理算法

   Parallel Old 是Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。这个收集器在1.6中才开始提供。

3. CMS收集器-标记整理算法

   CMS(Concurrent Mark Sweep)收集器是一种以获取最短回收停顿时间为目标的收集器。

   目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求。

   **CMS收集器是基于“标记-清除”算法实现的，**它的运作过程相对前面几种收集器来说更复杂一些，整个过程分为4个步骤：

   - 初始标记；
   - 并发标记；
   - 重新标记；
   - 并发清除。

   其中，初始标记、重新标记这两个步骤仍然需要“Stop The World”

   ![l7BmHs.png](https://s2.ax1x.com/2020/01/13/l7BmHs.png)

   CMS收集器主要优点：

   - 并发收集；

   - 低停顿。

   CMS明显的缺点：

   - CMS收集器无法处理浮动垃圾，可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。在JDK1.5的默认设置下，CMS收集器当老年代使用了68%的空间后就会被激活；
   - CMS是基于“标记-清除”算法实现的收集器，手机结束时会有大量空间碎片产生。空间碎片过多，可能会出现老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前出发FullGC。

##### 新生代和老年代垃圾收集器

1. G1收集器-标记整理算法

   JDK1.7后全新的回收器, 用于取代CMS收集器。

   G1收集器的优势：

   - 独特的分代垃圾回收器,分代GC: 分代收集器, 同时兼顾年轻代和老年代；

   - 使用分区算法, 不要求eden, 年轻代或老年代的空间都连续；

   - 并行性: 回收期间, 可由多个线程同时工作, 有效利用多核cpu资源；

   - 空间整理: 回收过程中, 会进行适当对象移动, 减少空间碎片；

   - 可预见性: G1可选取部分区域进行回收, 可以缩小回收范围, 减少全局停顿。

   G1收集器的运作大致可划分为一下步骤：

   ![l7B0C6.png](https://s2.ax1x.com/2020/01/13/l7B0C6.png)

   G1收集器的阶段分以下几个步骤：

   - 初始标记（它标记了从GC Root开始直接可达的对象）；
   - 并发标记（从GC Roots开始对堆中对象进行可达性分析，找出存活对象）；
   - 最终标记（标记那些在并发标记阶段发生变化的对象，将被回收）；
   - 筛选回收（首先对各个Regin的回收价值和成本进行排序，根据用户所期待的GC停顿时间指定回收计划，回收一部分Region）。

   更多G1收集器：[深入剖析G1收集器+回收流程+推荐用例](https://www.jianshu.com/p/4ef1524a396e)

下面给出配置回收器时，经常使用的参数：

```d
-XX:+UseSerialGC：在新生代和老年代使用串行收集器

-XX:+UseParNewGC：在新生代使用并行收集器

-XX:+UseParallelGC ：新生代使用并行回收收集器，更加关注吞吐量

-XX:+UseParallelOldGC：老年代使用并行回收收集器

-XX:ParallelGCThreads：设置用于垃圾回收的线程数

-XX:+UseConcMarkSweepGC：新生代使用并行收集器，老年代使用CMS+串行收集器

-XX:ParallelCMSThreads：设定CMS的线程数量

-XX:+UseG1GC：启用G1垃圾回收器

-XX:+PrintGCTimeStamps -XX:+PrintGCDetails -verbose:gc :GC详情 


-Xms :堆初始大小 
-Xmx 或 -XX:MaxHeapSize=size :
-Xmn 或 (-XX:NewSize=size + -XX:MaxNewSize=size ) :新生代大小 
-XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy: 幸存区比例(动态) 
-XX:NewRatio=n :新生代与老生代(new/old generation)的大小比例(Ratio). 默认值为 2
-XX:SurvivorRatio=ratio :eden/survivor 空间大小的比例(Ratio). 默认值为 8.
-XX:MaxTenuringThreshold=threshold :提升年老代的最大临界值(tenuring threshold). 默认值为 15.  
-XX:+PrintTenuringDistribution :晋升详情 
-XX:+ScavengeBeforeFullGC : FullGC 前 MinorGC 
-XX:GCTimeRatio=ratio :假设 GCTimeRatio 的值为 n，那么系统将花费不超过 1/(1+n) 的时间用于垃圾收集
-XX:MaxGCPauseMillis=ms :设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值
-XX:ParallelGCThreads=n ：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
-XX:ConcGCThreads=n :并发垃圾收集器使用的线程数量. 默认值随JVM运行的平台不同而不同.

##CMS
-XX:CMSInitiatingOccupancyFraction=percent :在上一次CMS并发GC执行过后，到底还要再执行多少次full GC才会做压缩。默认是0，也就是在默认配置下每次CMS GC顶不住了而要转入full GC的时候都会做压缩。
-XX:+CMSScavengeBeforeRemark :开启或关闭在CMS重新标记阶段之前的清除（YGC）尝试

##G1
-XX:G1ReservePercent=n :设置堆内存保留为假天花板的总量,以降低提升失败的可能性. 默认值是 10.
-XX:G1HeapRegionSize=n :使用G1时Java堆会被分为大小统一的的区(region)。此参数可以指定每个heap区的大小. 默认值将根据 heap size 算出最优解. 最小值为1Mb, 最大值为 32Mb.
-XX:InitiatingHeapOccupancyPercent=45 ：启动并发GC周期时的堆内存占用百分比. G1之类的垃圾收集器用它来触发并发GC周期,基于整个堆的使用率,而不只是某一代内存的使用比. 值为 0 则表示”一直执行GC循环”. 默认值为 45.
```

#### 知识点

1. 新生代转移到老年代的触发条件

   - 长期存活的对象
   - 大对象直接进入老年代
   - minor gc后，survivor仍然放不下
   - 动态年龄判断 ，大于等于某个年龄的对象超过了survivor空间一半 ，大于等于某个年龄的对象直接进入老年代

2. 不同阶段：

   - SerialGC
     - 新生代内存不足发生的垃圾收集 - minor gc
     - 老年代内存不足发生的垃圾收集 - full gc

   - ParallelGC
     - 新生代内存不足发生的垃圾收集 - minor gc
     - 老年代内存不足发生的垃圾收集 - full gc

   - CMS
     - 新生代内存不足发生的垃圾收集 - minor gc
     - 老年代内存不足，如果并发回收速度大于垃圾产生速度，则不是Full GC；否则是

   - G1
     - 新生代内存不足发生的垃圾收集 - minor gc
     - 老年代内存不足，如果并发回收速度大于垃圾产生速度，则不是Full GC；否则是

3. Young Collection 跨代引用：新生代回收的跨代引用(老年代引用新生代)问题

   采用了cart table的方式，对老年代进行细分，分成了许多个card,每个card大约是512K。如果老年代某个对象，引用了新生代的对象，我们把这个老年代的对象标记为脏card。这样，找老年代的根对象时，就不用遍历整个老年代了，只需要关注脏card，减小搜索范围，提高效率。如下图，粉色为脏card,绿色为伊甸园区，蓝色为幸存者区，橙色为老年代。
   ![UpQjJJ.png](https://s1.ax1x.com/2020/07/05/UpQjJJ.png)

   老年代有脏卡标记，而新生代则有remembered  Set记录外部对它的引用，记录都有哪些脏卡。将来对新生代进行垃圾回收时，先通过remembered Set 知道有哪些脏卡，然后通过脏卡区域遍历GC Root。

   在引用变更时通过post-write barrier + dirty card queue,在每次的引用变更时，都要更新标记脏卡（异步操作，把更新的指令放到一个队列（dirty card queue）之中，将来由一个线程，执行更新操作）。

4. G1并发标记阶段Remark阶段引用变化问题：

   使用写屏障技术+satb_mark_queue记录引用变化的对象

5. 优化选项：

   - 字符串去重

     优点:节省大量内存；缺点:略微多占用了 cpu 时间,新生代回收时间略微增加

     - `-XX:+UseStringDeduplication`

     将所有新分配的字符串放入一个队列；当新生代回收时, G1并发检查是否有字符串重复；如果它们值一样,让它们引用同一个 char[]

     注意,与 String.intern() 不一样，String.intern() 关注的是字符串对象，而字符串去重关注的是 char[]

     在 JVM 内部,使用了不同的字符串表

   - 并发标记类卸载

     所有对象都经过并发标记后,就能知道哪些类不再被使用,当一个类加载器的所有类都不再使用,则卸载它所加载的所有类 `-XX:+ClassUnloadingWithConcurrentMark` 默认启用

   - 回收巨型对象

     一个对象大于 region 的一半时,称之为巨型对象

     G1 不会对巨型对象进行拷贝

     回收时被优先考虑

     G1 会跟踪老年代所有 incoming 引用,这样老年代 incoming 引用为0 的巨型对象就可以在新生代垃圾回收时处理掉

   - JDK 9 并发标记起始时间的调整

     并发标记必须在堆空间占满前完成,否则退化为 FullGC；JDK 9 之前需要使用 `- XX:InitiatingHeapOccupancyPercent`

     JDK 9 可以动态调整

     - `- XX:InitiatingHeapOccupancyPercent 用来设置初始值`
     - 进行数据采样并动态调整
     - 总会添加一个安全的空档空间

#### 调优领域

最快的 GC是不发生 GC

查看 FullGC 前后的内存占用,考虑下面几个问题

- 数据是不是太多?

  `resultSet = statement.executeQuery("select * from 大表 limit n")`

- 数据表示是否太臃肿?
  - 对象图
  - 对象大小 16 Integer 24 int 4

- 是否存在内存泄漏?
  - static Map map =
  - 软引用
  - 弱引用
  - 第三方缓存实现

**新生代调优**

新生代的特点： 

- 所有的 new 操作的内存分配非常廉价
  - TLAB thread-local allocation buffer

- 死亡对象的回收代价是零
- 大部分对象用过即死
- Minor GC 的时间远远低于 Full GC

越大越好吗?

-Xmn Sets the initial and maximum size (in bytes) of the heap for the young generation (nursery).GC is performed in this region more often than in other regions. If the size for the young generation is too small, then a lot of minor garbage collections are performed. If the size is too large, then only full garbage collections are performed, which can take a long time to complete.Oracle recommends that you keep the size for the young generation greater than 25% and less than 50% of the overall heap size.

- 新生代能容纳所有【并发量 * (请求-响应)】的数据
- 幸存区大到能保留【当前活跃对象 +需要晋升对象】
- 晋升阈值配置得当,让长时间存活对象尽快晋升

`- XX:MaxTenuringThreshold=threshold`

`- XX:+PrintTenuringDistribution`：打印不同年龄的对象信息

**老年代调优**

推荐使用G1

以 CMS 为例

- CMS 的老年代内存越大越好

- 先尝试不做调优,如果没有 Full GC 那么已经...,否则先尝试调优新生代

- 观察发生 Full GC 时老年代内存占用,将老年代内存预设调大 1/4 ~ 1/3

  `- XX:CMSInitiatingOccupancyFraction=percent`

#### GCeasy

打印GC日志：

`-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:./gc.log `

- `-XX:+PrintGCDetails`：表示的是打印GC日志详情
- `-XX:+PrintGCTimeStamps`：表示打印GC时间戳
- `-Xloggc: ./gc.log`：表示在当前目录下生成gc.log文件

打印出GC日志之后，就可以拿去[GCeasy官网](https://www.gceasy.io/)上进行GC可视化分析了



### 参考

1. [JVM的4种垃圾回收算法、垃圾回收机制与总结](https://www.jianshu.com/p/7759c6f21ed1)
2. [7种JVM垃圾收集器特点，优劣势、及使用场景](https://www.jianshu.com/p/883a682dd25e)
3. [深入详解JVM内存模型与JVM参数详细配置](https://www.jianshu.com/p/895deef15808)
4. [JVM性能调优的6大步骤，及关键调优参数详解](https://www.jianshu.com/p/870b4ce45b80)
5. [深入剖析G1收集器+回收流程+推荐用例](https://www.jianshu.com/p/4ef1524a396e)
6. [G1](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html)