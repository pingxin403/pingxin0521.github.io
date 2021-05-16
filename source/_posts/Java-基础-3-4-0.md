---
title: Java 集合类总结
date: 2020-1-06 08:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

#### List

List中的元素是有序的、可重复的，主要实现方式有动态数组和链表。

<!--more-->

![1FjnO0.png](https://s2.ax1x.com/2020/01/21/1FjnO0.png)

java中提供的List的实现主要有ArrayList、LinkedList、CopyOnWriteArrayList，另外还有两个古老的类Vector和Stack。

关于List相关的问题主要有：

1. ArrayList和LinkedList有什么区别？
2. ArrayList是怎么扩容的？
3. ArrayList插入、删除、查询元素的时间复杂度各是多少？
4. 怎么求两个集合的并集、交集、差集？
5. ArrayList是怎么实现序列化和反序列化的？
6. 集合的方法toArray()有什么问题？
7. 什么是fail-fast？
8. LinkedList是单链表还是双链表实现的？
9. LinkedList除了作为List还有什么用处？
10. LinkedList插入、删除、查询元素的时间复杂度各是多少？
11. 什么是随机访问？
12. 哪些集合支持随机访问？他们都有哪些共性？
13. CopyOnWriteArrayList是怎么保证并发安全的？
14. CopyOnWriteArrayList的实现采用了什么思想？
15. CopyOnWriteArrayList是不是强一致性的？
16. CopyOnWriteArrayList适用于什么样的场景？
17. CopyOnWriteArrayList插入、删除、查询元素的时间复杂度各是多少？
18. CopyOnWriteArrayList为什么没有size属性？
19. 比较古老的集合Vector和Stack有什么缺陷？

#### Map

Map是一种(key/value)的映射结构，其它语言里可能称作字典（Dictionary），包括java早期也是叫做字典，Map中的元素是一个key只能对应一个value，不能存在重复的key。

![1Fv939.png](https://s2.ax1x.com/2020/01/21/1Fv939.png)

java中提供的Map的实现主要有HashMap、LinkedHashMap、WeakHashMap、TreeMap、ConcurrentHashMap、ConcurrentSkipListMap，另外还有两个比较古老的Map实现HashTable和Properties。

关于Map的问题主要有：

1. 什么是散列表？
2. 怎么实现一个散列表？
3. java中HashMap实现方式的演进？
4. HashMap的容量有什么特点？
5. HashMap是怎么进行扩容的？
6. HashMap中的元素是否是有序的？
7. HashMap何时进行树化？何时进行反树化？
8. HashMap是怎么进行缩容的？
9. HashMap插入、删除、查询元素的时间复杂度各是多少？
10. HashMap中的红黑树实现部分可以用其它数据结构代替吗？
11. LinkedHashMap是怎么实现的？
12. LinkedHashMap是有序的吗？怎么个有序法？
13. LinkedHashMap如何实现LRU缓存淘汰策略？
14. WeakHashMap使用的数据结构？
15. WeakHashMap具有什么特性？
16. WeakHashMap通常用来做什么？
17. WeakHashMap使用String作为key是需要注意些什么？为什么？
18. 什么是弱引用？
19. 红黑树具有哪些特性？
20. TreeMap就有序的吗？怎么个有序法？
21. TreeMap是否需要扩容？
22. 什么是左旋？什么是右旋？
23. 红黑树怎么插入元素？
24. 红黑树怎么删除元素？
25. 为什么要进行平衡？
26. 如何实现红黑树的遍历？
27. TreeMap中是怎么遍历的？
28. TreeMap插入、删除、查询元素的时间复杂度各是多少？
29. HashMap在多线程环境中什么时候会出现问题？
30. ConcurrentHashMap的存储结构？
31. ConcurrentHashMap是怎么保证并发安全的？
32. ConcurrentHashMap是怎么扩容的？
33. ConcurrentHashMap的size()方法的实现知多少？
34. ConcurrentHashMap是强一致性的吗？
35. ConcurrentHashMap不能解决什么问题？
36. ConcurrentHashMap中哪些地方运用到分段锁的思想？
37. 什么是伪共享？怎么避免伪共享？
38. 什么是跳表？
39. ConcurrentSkipList是有序的吗？
40. ConcurrentSkipList是如何保证线程安全的？
41. ConcurrentSkipList插入、删除、查询元素的时间复杂度各是多少？
42. ConcurrentSkipList的索引具有什么特性？
43. 为什么Redis选择使用跳表而不是红黑树来实现有序集合？

#### Set

java里面的Set对应于数学概念上的集合，里面的元素是不可重复的，通常使用Map或者List来实现。

![1FvsET.png](https://s2.ax1x.com/2020/01/21/1FvsET.png)

java中提供的Set的实现主要有HashSet、LinkedHashSet、TreeSet、CopyOnWriteArraySet、ConcurrentSkipSet。

关于Set的问题主要有：

1. HashSet怎么保证添加元素不重复？
2. HashSet是有序的吗？
3. HashSet是否允许null元素？
4. Set是否有get()方法？
5. LinkedHashSet是有序的吗？怎么个有序法？
6. LinkedHashSet支持按元素访问顺序排序吗？
7. TreeSet真的是使用TreeMap来存储元素的吗？
8. TreeSet是有序的吗？怎么个有序法？
9. TreeSet和LinkedHashSet有何不同？
10. TreeSet和SortedSet有什么区别和联系？
11. CopyOnWriteArraySet是用Map实现的吗？
12. CopyOnWriteArraySet是有序的吗？怎么个有序法？
13. CopyOnWriteArraySet怎么保证并发安全？
14. CopyOnWriteArraySet以何种方式保证元素不重复？
15. 如何比较两个Set中的元素是否完全一致？
16. ConcurrentSkipListSet的底层是ConcurrentSkipListMap吗？
17. ConcurrentSkipListSet是有序的吗？怎么个有序法？

#### Queue

Queue是一种叫做队列的数据结构，队列是遵循着一定原则的入队出队操作的集合，一般来说，入队是在队列尾添加元素，出队是在队列头删除元素，但是，也不一定，比如优先级队列的原则就稍微有些不同。

![1FvOxA.png](https://s2.ax1x.com/2020/01/21/1FvOxA.png)

java中提供的Queue的实现主要有PriorityQueue、ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue、LinkedTransferQueue、DelayQueue、ConcurrentLinkedQueue。

关于Queue的问题主要有：

1. 什么是堆？什么是堆化？
2. 什么是优先级队列？
3. PriorityQueue是怎么实现的？
4. PriorityQueue是有序的吗？
5. PriorityQueue入队、出队的时间复杂度各是多少？
6. PriorityQueue是否需要扩容？扩容规则呢？
7. ArrayBlockingQueue的实现方式？
8. ArrayBlockingQueue是否需要扩容？
9. ArrayBlockingQueue怎么保证线程安全？
10. ArrayBlockingQueue有什么缺点？
11. LinkedBlockingQueue的实现方式？
12. LinkedBlockingQueue是有界的还是无界的队列？
13. LinkedBlockingQueue怎么保证线程安全？
14. LinkedBlockingQueue与ArrayBlockingQueue对比？
15. SynchronousQueue的实现方式？
16. SynchronousQueue真的是无缓冲的吗？
17. SynchronousQueue怎么保证线程安全？
18. SynchronousQueue的公平模式和非公平模式有什么区别？
19. SynchronousQueue在高并发情景下会有什么问题？
20. PriorityBlockingQueue的实现方式？
21. PriorityBlockingQueue是否需要扩容？
22. PriorityBlockingQueue怎么保证线程安全？
23. PriorityBlockingQueue为什么不需要notFull条件？
24. 什么是双重队列？
25. LinkedTransferQueue是怎么实现阻塞队列的？
26. LinkedTransferQueue是怎么控制并发安全的？
27. LinkedTransferQueue与SynchronousQueue有什么异同？
28. ConcurrentLinkedQueue是阻塞队列吗？
29. ConcurrentLinkedQueue如何保证并发安全？
30. ConcurrentLinkedQueue能用于线程池吗？
31. DelayQueue是阻塞队列吗？
32. DelayQueue的实现方式？
33. DelayQueue主要用于什么场景？

#### Deque

Deque是一种特殊的队列，它的两端都可以进出元素，故而得名双端队列（Double Ended Queue）。

![1FxnaT.png](https://s2.ax1x.com/2020/01/21/1FxnaT.png)

java中提供的Deque的实现主要有ArrayDeque、LinkedBlockingDeque、ConcurrentLinkedDeque、LinkedList。

关于Deque的问题主要有：

1. 什么是双端队列？
2. ArrayDeque是怎么实现双端队列的？
3. ArrayDeque是有界的吗？
4. LinkedList与ArrayDeque的对比？
5. 双端队列是否可以作为栈使用？
6. LinkedBlockingDeque是怎么实现双端队列的？
7. LinkedBlockingDeque是怎么保证并发安全的？
8. ConcurrentLinkedDeque是怎么实现双端队列的？
9. ConcurrentLinkedDeque是怎么保证并发安全的？
10. LinkedList是List和Deque的集合体？

#### 总结

其实上面的问题很多都具有共性，我觉得以下几个问题在看每个集合类的时候都要掌握清楚：

1. 使用的数据结构？

2. 添加元素、删除元素的基本逻辑？

3. 是否是fail-fast的？

4. 是否需要扩容？扩容规则？

5. 是否有序？是按插入顺序还是自然顺序还是访问顺序？

6. 是否线程安全？

7. 使用的锁？

8. 优点？缺点？

9. 适用的场景？

10. 时间复杂度？

11. 空间复杂度？

基本上所有集合类使用的数据结构都是数组和链表，包括树和跳表也可以看成是链表的一种方式。

对于并发安全的集合，还要再加上相应的锁策略，要不就是重入锁，要不就是CAS+自旋，偶尔也来个synchronized。

所以，掌握集合的源码不算什么，数据结构和锁才是王道。

### 线程安全的集合类

![](https://i.loli.net/2019/05/12/5cd81a084d40e.png)

常见并且常用的数据集合有map，hashmap，hashSet，TreeMap， TreeSet， List， ArrayList， LinkedList， StringBuilder

**线程安全的数据集合有**：Vector， HashTable， StringBuffer，  ConcurrentHashMap， Collections.synchronizedList， CopyOnWriteArrayList

**相关集合对象的比较：**

**Vector、ArrayList、LinkedList， Collections.synchronizedList， CopyOnWriteArrayList：** 

**1、Vector：**

Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。 

**2、ArrayList：** 

a. 当操作是在一列数据的后面添加数据而不是在前面或者中间，并需要随机地访问其中的元素时，使用ArrayList性能比较好。 

b. ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。 

**3、LinkedList：** 

a. 当对一列数据的前面或者中间执行添加或者删除操作时，并且按照顺序访问其中的元素时，要使用LinkedList。 

b. LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。

Vector和ArrayList在使用上非常相似，都可以用来表示一组数量可变的对象应用的集合，并且可以随机的访问其中的元素。 

**HashTable、HashMap、HashSet：** 

HashTable和HashMap采用的存储机制是一样的，不同的是： 

**1、HashMap：**

a. 采用数组方式存储key-value构成的Entry对象，无容量限制； 

b. 基于key hash查找Entry对象存放到数组的位置，对于hash冲突采用链表的方式去解决； 

c. 在插入元素时，可能会扩大数组的容量，在扩大容量时须要重新计算hash，并复制对象到新的数组中； 

d. 是非线程安全的； 

e. 遍历使用的是Iterator迭代器；

**2、HashTable：** 

a. 是线程安全的； 

b. 无论是key还是value都不允许有null值的存在；在HashTable中调用Put方法时，如果key为null，直接抛出NullPointerException异常； 

c. 遍历使用的是Enumeration列举；

**3、HashSet：** 

a. 基于HashMap实现，无容量限制； 

b. 是非线程安全的； 

c. 不保证数据的有序；



**TreeSet、TreeMap：**

TreeSet和TreeMap都是完全基于Map来实现的，并且都不支持get(index)来获取指定位置的元素，需要遍历来获取。另外，TreeSet还提供了一些排序方面的支持，例如传入Comparator实现、descendingSet以及descendingIterator等。 

**1、TreeSet：** 

a. 基于TreeMap实现的，支持排序； 

b. 是非线程安全的；

**2、TreeMap：** 

a. 典型的基于红黑树的Map实现，因此它要求一定要有key比较的方法，要么传入Comparator比较器实现，要么key对象实现Comparator接口； 

b. 是非线程安全的；



**StringBuffer和StringBulider：** 

StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串。

      　　  1. 在执行速度方面的比较：StringBuilder > StringBuffer ； 
      　　  2. 他们都是字符串变量，是可改变的对象，每当我们用它们对字符串做操作时，实际上是在一个对象上操作的，不像String一样创建一些对象进行操作，所以速度快； 
      　　  3. StringBuilder：线程非安全的； 
      　　  4. StringBuffer：线程安全的；

**对于String、StringBuffer和StringBulider三者使用的总结：**

　　 1.如果要操作少量的数据用 = String 

　 　2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 

　　 3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

**vector,hashtable是在Java1.0就引入的集合，两个都是线程安全的，但是现在已很少使用，原因就是内部实现的线程安全太消耗资源，因此新的安全的高效的数据集合出现了，这就是ConcurrentHashMap， Collections.synchronizedList， CopyOnWriteArrayList。**

**为什么我们想要新的线程安全的List类？为什么会出现CopyOnWriteArrayList？**

简单的答案是与迭代和并发修改之间的交互有关。使用 Vector 或使用同步的 List 封装器，返回的迭代器是 fail-fast 的，

这意味着如果在迭代过程中任何其他线程修改 List，迭代可能失败。Vector 的非常普遍的应用程序是存储通过组件注册的监听器的列表。当发生适合的事件时，该组件将在监听器的列表中迭代，调用每个监听器。

为了防止ConcurrentModificationException，迭代线程必须复制列表或锁定列表，一遍进行整体迭代，而这两种情况都需要大量的性能成本。CopyOnWriteArrayList类通过每次添加或删除元素时创建支持数组的新副本，避免了这个问题，但是进行中的迭代保持对创建迭代器时的当前副本进行操作。虽然复制也会有一些成本，但是在许多情况下，迭代要比修改多得多，在这些情况系，写入时复制要比其他备用方法具有更好的性能和并发性。

**CopyOnWriteArrayList如何做到线程安全的？**

**CopyOnWriteArrayList使用了一种叫写时复制的方法，当有新元素添加到CopyOnWriteArrayList时，先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。**

当有新元素加入的时候，如下图，创建新数组，并往新数组中加入一个新元素,这个时候，array这个引用仍然是指向原数组的。

![1.png](https://i.loli.net/2019/05/12/5cd81c49801cb.png)

当元素在新数组添加成功后，将array这个引用指向新数组。

![2.png](https://i.loli.net/2019/05/12/5cd81c4999183.png)

CopyOnWriteArrayList的整个add操作都是在**锁**的保护下进行的。 

这样做是为了避免在多线程并发add的时候，**复制出多个副本出来**,把数据搞乱了，导致最终的数组数据不是我们期望的。

CopyOnWriteArrayList的add操作的源代码如下：

```java
public boolean add(E e) {
//1、先加锁    final ReentrantLock lock = this.lock;

lock.lock();

try {

    Object[] elements = getArray();

    int len = elements.length;

    //2、拷贝数组        Object[] newElements = Arrays.copyOf(elements, len + 1);

    //3、将元素加入到新数组中        newElements[len] = e;

    //4、将array引用指向到新数组        setArray(newElements);

    return true;

} finally {

   //5、解锁        lock.unlock();

}
}
```

由于所有的写操作都是在新数组进行的，这个时候如果有线程并发的写，则通过锁来控制，如果有线程并发的读，则分几种情况： 

1. 如果写操作未完成，那么直接读取原数组的数据； 

2. 如果写操作完成，但是引用还未指向新数组，那么也是读取原数组数据； 

3. 如果写操作完成，并且引用已经指向了新的数组，那么直接从新数组中读取数据。

可见，CopyOnWriteArrayList的**读操作**是可以不用**加锁**的。

**CopyOnWriteArrayList的使用场景**

通过上面的分析，CopyOnWriteArrayList 有几个缺点： 

1. 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc

2. 不能用于**实时读**的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到**最终一致性**,但是还是没法满足实时性要求；

CopyOnWriteArrayList 合适**读多写少**的场景，不过这类慎用 

因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

**CopyOnWriteArrayList透露的思想**

如上面的分析CopyOnWriteArrayList表达的一些思想： 

1. 读写分离，读和写分开 

2. 最终一致性 

3. 使用另外开辟空间的思路，来解决并发冲突

**Collections.synchronizedList**

CopyOnWriteArrayList和Collections.synchronizedList是实现线程安全的列表的两种方式。两种实现方式分别针对不同情况有不同的性能表现，其中CopyOnWriteArrayList的写操作性能较差，而多线程的读操作性能较好。而Collections.synchronizedList的写操作性能比CopyOnWriteArrayList在多线程操作的情况下要好很多，而读操作因为是采用了synchronized关键字的方式，其读操作性能并不如CopyOnWriteArrayList。因此在不同的应用场景下，应该选择不同的多线程安全实现类。

 Collections.synchronizedList的源码可知，其实现线程安全的方式是建立了list的包装类，代码如下：

```
public static  List synchronizedList(List list) {  

return (list instanceof RandomAccess ?  

new SynchronizedRandomAccessList(list) :  

new SynchronizedList(list));//根据不同的list类型最终实现不同的包装类。  

   } 
```

其中，SynchronizedList对部分操作加上了synchronized关键字以保证线程安全。但其iterator()操作还不是线程安全的。部分SynchronizedList的代码如下：

```
public E get(int index) {  

synchronized(mutex) {return list.get(index);}  

}  

public E set(int index, E element) {  

synchronized(mutex) {return list.set(index, element);}  

}  

public void add(int index, E element) {  

synchronized(mutex) {list.add(index, element);}  

}  

public ListIterator listIterator() {  

return list.listIterator(); // Must be manually synched by user 需要用户保证同步，否则仍然可能抛出ConcurrentModificationException  

}  



public ListIterator listIterator(int index) {  

return list.listIterator(index); // Must be manually synched by user 需要用户保证同步，否则仍然可能抛出ConcurrentModificationException  

}
```

 写操作：在线程数目增加时CopyOnWriteArrayList的写操作性能下降非常严重，而Collections.synchronizedList虽然有性能的降低，但下降并不明显。

 读操作：在多线程进行读时，Collections.synchronizedList和CopyOnWriteArrayList均有性能的降低，但是Collections.synchronizedList的性能降低更加显著。

