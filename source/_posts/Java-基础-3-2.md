---
title: Java 集合类 (二)
date: 2019-04-06 08:20:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### Collections

Collections 是一个包装类。它包含有各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，服务于Java的Collection框架。

<!--more-->

##### 创建

Collections内部含有多个方法来生成容器对象。

##### 排序

- `static sort(List list);`
- `static sort(List list, Comparator<? super T> com);`

根据元素的自然顺序或自定义比较器指定顺序来对列表中的元素进行排序

##### 交换

- `static swap(List<?> list, int i, int j);`

对指定List的指定角标的两个元素进行位置交换

##### 折半查询

- `static binarySearch(List<? extends Comparable<? super T>> list,  T key);`

二分查找方法，根据指定key来查询List中的对应元素下标

##### 最值

    max (Collection<? extends T> coll);
    max (Collection<? extends T> coll, Comparator<? super T> comp);
该方法用于求集合元素的最大值，同时可以自定义比较器来规定大小的判定规则

##### 逆序

```
reverse (List<?> list)：反转指定列表中的元素的顺序
reverseOrder()：返回一个比较器，它强行逆转实现了Comparable接口对象Collection的自然顺序。
reverseOrder(Comparator cmp)：返回一个比较器，它强行逆转指定比较器的顺序。
```

##### 替换

- boolean replaceAll(List<?>  list,  T  oldVal,  T newVal)：替换失败则返回false，否则返回true
- fill (List<? super T>  list,  T  obj)：使用指定元素替换列表中的所有元素

##### 其他

- shuffle(List<?>  list)：对列表中的元素的位置进行随机变换，可以用于洗牌。
- List  synchronizedList(  List  list)：将非同步的集合转换成同步的集合。
- Collections.unmodifiableCollection(Collection c)方法创建一个只读集合，

#### 各种线性表选择策略

数组：是以一段连续内存保存数据的；随机访问是最快的，但不支持插入、删除、迭代等操作。

ArrayList与ArrayDeque：以数组实现；随机访问速度还行，插入、删除、迭代操作速度一般；线程不安全。

Vector：以数组实现；随机访问速度一般，插入、删除、迭代速度不太好；线程安全的。

LinkedList：以链表实现；随机访问速度不太好，插入、删除、迭代速度非常快。

#### fail-fast 机制

fail-fast的字面意思是“快速失败”。当我们在遍历集合元素的时候，经常会使用迭代器，但在迭代器遍历元素的过程中，如果集合的结构被改变的话，就会抛出异常，防止继续遍历。这就是所谓的快速失败机制。

当Iterator这个迭代器被创建后，除了迭代器本身的方法(remove)可以改变集合的结构外，其他的因素如若**改变了集合的结构**，都被抛出ConcurrentModificationException异常。

代器的快速失败行为是不一定能够得到保证的，一般来说，存在非同步的并发修改时，不可能做出任何坚决的保证的。但是快速失败迭代器会做出最大的努力来抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是不正确的。正确的做法应该是：迭代器的快速失败行为应该仅用于检测程序中的bug.

**稍微总结下：fail-fast,即快速失败机制，它是java集合中的一种错误检测机制，当多个线程（当个线程也是可以滴）,在结构上对集合进行改变时，就有可能会产生fail-fast机制。**

> 这里，我解释下什么是结构上的改变。 例如集合上的插入和删除就是结构上的改变，但是，如果是对集合中某个元素进行修改的话，并不是结构上的改变哦。

下面，我们来演示下在单线程的环境下，fail-fast抛出异常的实例：

```java
      for(int i = 10; i < 100; i++){
        map.put(i, i);
    }
    List<Integer> list = new ArrayList<>();
    for(int i = 0; i < 20; i++){
        list.add(i);
    }
    Iterator<Integer> it = list.iterator();
    int temp = 0;
    while(it.hasNext()){
        if(temp == 3){
            temp++;
            list.remove(3);
        }else{
            temp++;
            System.out.println(it.next());
        }
    }
}
```

**结果分析：**因为当temp==3的时候，执行list.remove()方法，集合的结构被改变了，所以再次遍历迭代器的时候，就会抛出异常。

**工作原理**

我们首先先来看下源码

![M9RTr6.png](https://s2.ax1x.com/2019/11/05/M9RTr6.png)

**分析：**从源码我们可以发现，迭代器在执行next()等方法的时候，都会调用checkForComodification()这个方法，查看modCount==expectedModCount?如果相等则抛出异常。

expectedModcount:这个值在对象被创建的时候就被赋予了一个固定的值modCount。也就是说这个值是不变的。也就是说，如果在迭代器遍历元素的时候，如果modCount这个值发生了改变，那么再次遍历时就会抛出异常。
**什么时候modCount会发生改变呢？**

其实当我们对集合的元素的个数做出改变的时候，modCount的值就会被改变，如果删除，插入。但修改则不会。

**一些处理方法**

如果我们不希望在迭代器遍历的时候因为并发等原因，导致集合的结构被改变，进而可能抛出异常的话，我们可以在涉及到会影响到modCount值改变的地方，加上同步锁(synchronized),或者直接使用Collections.synchronizedList来解决,或者使用CopyOnWriteCollection。

**fail-safe与fail-fast的区别**

当我们对集合结构上做出改变的时候，fail-fast机制就会抛出异常。但是，对于采用fail-safe机制来说，就不会抛出异常(大家估计看到safe两个字就知道了)。

这是因为，当集合的结构被改变的时候，fail-safe机制会在复制原集合的一份数据出来，然后在复制的那份数据遍历。

因此，虽然fail-safe不会抛出异常，但存在以下缺点：

1. 复制时需要额外的空间和时间上的开销。
2. 不能保证遍历的是最新内容。

在java.util.concurrent 包中集合的迭代器，如 ConcurrentHashMap, CopyOnWriteArrayList等默认为都是Fail-Safe。

```java
// 1.foreach迭代，fail-safe，不会抛出异常
for (String s : list) {
  System.out.println(s);
  if ("s1".equals(s)) {
  list.remove(s);
  }
}

// 2.iterator迭代，fail-safe，不会抛出异常
Iterator<String> iterator = list.iterator();
  while (iterator.hasNext()) {
  String s = iterator.next();
  System.out.println(s);
  if ("s1".equals(s)) {
  list.remove(s);
  }
}
```

1. **CopyOnWriteArrayList** 在修改时会创建一个原数组副本（只是新建一个数组，浅克隆）在新数组上修改后再将集合的数组指针指向新数组对象，而此时迭代器中的指针仍指向原数组对象。迭代过程中的修改，不会反映到迭代上。

   源码解析：

   ```java
   public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
       //！只放关键代码
   
       /** The array, accessed only via getArray/setArray. */
       private transient volatile Object[] array; //存放数据数组
   
       //添加方法
       public boolean add(E e) {
           final ReentrantLock lock = this.lock;
           lock.lock();
           try {
               Object[] elements = getArray();
               int len = elements.length;
               Object[] newElements = Arrays.copyOf(elements, len + 1); //浅克隆原数组并长度+1
               newElements[len] = e; //在新数组上进行添加
               setArray(newElements); //将数组指针指向新数组
               return true;
           } finally {
               lock.unlock();
           }
       }
   
       //迭代器方法
       public Iterator<E> iterator() {
           return new COWIterator<E>(getArray(), 0); //直接用原来的数据数组
       }
   
       final Object[] getArray() {
           return array;
       }
   
       static final class COWIterator<E> implements ListIterator<E> {
           /** Snapshot of the array */
           private final Object[] snapshot;
           /** Index of element to be returned by subsequent call to next.  */
           private int cursor; //始终指向下一个元素
   
           private COWIterator(Object[] elements, int initialCursor) {
               cursor = initialCursor;
               snapshot = elements; //将引用snapshot指向传入的原数组
           }
   //…
           public E next() {
               if (! hasNext())
                   throw new NoSuchElementException();
               return (E) snapshot[cursor++];
           }
       }
   }
   ```

2. ConcurrentHashMap对fail-safe实现

   参考：java高级

### 面试题

1. List、Set、Map是否继承自Collection接口？

   答：List、Set 是，Map 不是。Map是键值对映射容器，与List和Set有明显的区别，而Set存储的零散的元素且不允许有重复元素（数学中的集合也是如此），List是线性结构的容器，适用于按数值索引访问元素的情形。

2. 阐述ArrayList、Vector、LinkedList的存储性能和特性。

   答：ArrayList 和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector中的方法由于添加了synchronized修饰，因此Vector是线程安全的容器，但性能上较ArrayList差，因此已经是Java中的遗留容器。

   LinkedList使用双向链表实现存储（将内存中零散的内存单元通过附加的引用关联起来，形成一个可以按序号索引的线性结构，这种链式存储方式与数组的连续存储方式相比，内存的利用率更高），按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。

   Vector属于遗留容器（Java早期的版本中提供的容器，除此之外，Hashtable、Dictionary、BitSet、Stack、Properties都是遗留容器），已经不推荐使用，但是由于ArrayList和LinkedListed都是非线程安全的，如果遇到多个线程操作同一个容器的场景，则可以通过工具类Collections中的synchronizedList方法将其转换成线程安全的容器后再使用（这是对装潢模式的应用，将已有对象传入另一个类的构造器中创建新的对象来增强实现）。

   补充：遗留容器中的Properties类和Stack类在设计上有严重的问题，Properties是一个键和值都是字符串的特殊的键值对映射，在设计上应该是关联一个Hashtable并将其两个泛型参数设置为String类型，但是Java API中的Properties直接继承了Hashtable，这很明显是对继承的滥用。这里复用代码的方式应该是Has-A关系而不是Is-A关系，另一方面容器都属于工具类，继承工具类本身就是一个错误的做法，使用工具类最好的方式是Has-A关系（关联）或Use-A关系（依赖）。同理，Stack类继承Vector也是不正确的。Sun公司的工程师们也会犯这种低级错误，让人唏嘘不已。

3. Collection和Collections的区别？

   答：Collection是一个接口，它是Set、List等容器的父接口；Collections是个一个工具类，提供了一系列的静态方法来辅助容器操作，这些方法包括对容器的搜索、排序、线程安全化等等。

4. List、Map、Set三个接口存取元素时，各有什么特点？

   答：List以特定索引来存取元素，可以有重复元素。Set不能存放重复元素（用对象的equals()方法来区分元素是否重复）。Map保存键值对（key-value pair）映射，映射关系可以是一对一或多对一。Set和Map容器都有基于哈希存储和排序树的两种实现版本，基于哈希存储的版本理论存取时间复杂度为O(1)，而基于排序树版本的实现在插入或删除元素时会按照元素或元素的键（key）构成排序树从而达到排序和去重的效果。

5. TreeMap和TreeSet在排序时如何比较元素？Collections工具类中的sort()方法如何比较元素？

   答：TreeSet要求存放的对象所属的类必须实现Comparable接口，该接口提供了比较元素的compareTo()方法，当插入元素时会回调该方法比较元素的大小。TreeMap要求存放的键值对映射的键必须实现Comparable接口从而根据键对元素进行排序。Collections工具类的sort方法有两种重载的形式，第一种要求传入的待排序容器中存放的对象必须实现Comparable接口以实现元素的比较；第二种不强制性的要求容器中的元素必须可比较，但是要求传入第二个参数，参数是Comparator接口的子类型（需要重写compare方法实现元素的比较），相当于一个临时定义的排序规则，其实就是通过接口注入比较元素大小的算法，也是对回调模式的应用（Java中对函数式编程的支持）。

6. HashMap 的长度为什么是2的幂次方

   为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀，每个链表/红黑树长度大致相同。这个实现就是把数据存到哪个链表/红黑树中的算法。

   **这个算法应该如何设计呢？**

   我们首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。**

 7. ConcurrentHashMap 和 Hashtable 的区别

    ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

    - **底层数据结构：** JDK1.7的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
    - **实现线程安全的方式（重要）：** ① **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。（默认分配16个Segment，比Hashtable效率提高16倍。） **到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **Hashtable(同一把锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

    HashTable:

    ![2.jpg](https://i.loli.net/2019/06/25/5d11db295d81427916.jpg)

    JDK1.7的ConcurrentHashMap：

    ![3.jpg](https://i.loli.net/2019/06/25/5d11db2975eb036228.jpg)

    JDK1.8的ConcurrentHashMap（TreeBin: 红黑二叉树节点 Node: 链表节点）：

    ![4.jpg](https://i.loli.net/2019/06/25/5d11db29809eb83027.jpg)

8. ConcurrentHashMap线程安全的具体实现方式/底层具体实现

   **JDK1.7（上面有示意图）**

   首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

   **ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成。**

   Segment 实现了 ReentrantLock,所以 Segment 是一种可重入锁，扮演锁的角色。HashEntry 用于存储键值对数据。

   一个 ConcurrentHashMap 里包含一个 Segment 数组。Segment 的结构和HashMap类似，是一种数组和链表结构，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个HashEntry数组里的元素，当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment的锁。

   **JDK1.8 （上面有示意图）**

   ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全。数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。

   synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

9. [fail-fast与fail-safe有什么区别](https://www.cnblogs.com/kubidemanong/articles/9113820.html)？

   Iterator的fail-fast属性与当前的集合共同起作用，因此它不会受到集合中任何改动的影响。Java.util包中的所有集合类都被设计为fail-fast的，

   而java.util.concurrent中的集合类都为fail-safe的。

   Fall—fast迭代器抛出ConcurrentModificationException，

   fall—safe迭代器从不抛出ConcurrentModificationException。

10. Map接口提供了哪些不同的集合视图

     Map接口提供三个集合视图：

     - Set keyset()：返回map中包含的所有key的一个Set视图。集合是受map支持的，map的变化会在集合中反映出来，反之亦然。当一个迭代器正在遍历一个集合时，若map被修改了（除迭代器自身的移除操作以外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。

       它不支持add和addAll操作。

     - Collection values()：返回一个map中包含的所有value的一个Collection视图。这个collection受map支持的，map的变化会在collection中反映出来，反之亦然。当一个迭代器正在遍历一个collection时，若map被修改了（除迭代器自身的移除操作以外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。它不支持add和addAll操作。

     - Set<Map.Entry<K, V>> entrySet()：返回一个map钟包含的所有映射的一个集合视图。这个集合受map支持的，map的变化会在collection中反映出来，反之亦然。当一个迭代器正在遍历一个集合时，若map被修改了（除迭代器自身的移除操作，以及对迭代器返回的entry进行setValue外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。它不支持add和addAll操作。

11. 集合框架底层数据结构总结

     Collection
     1. List

         - Arraylist： Object数组
         - Vector： Object数组
         - LinkedList： 双向循环链表

     2. Set（依赖Map的key来实现）

         - HashSet（无序，唯一）: 基于 HashMap 实现的，底层采用 HashMap 来保存元素,使用其key唯一性，扩容方式与HashMap相同
         - LinkedHashSet： LinkedHashSet 继承与 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 Hashmap 实现一样，不过还是有一点点区别的。
         - TreeSet（有序，唯一）： 内部使用TreeMap，红黑树(自平衡的排序二叉树)

    3. Map
    
         - HashMap： JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）.JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
    
         - LinkedHashMap: LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
    
         - HashTable: 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
    
         - TreeMap: 红黑树（自平衡的排序二叉树）
    
12. Comparable和Comparator接口有何区别

     Comparable和Comparator接口被用来对对象集合或者数组进行排序。Comparable接口被用来提供对象的自然排序，我们可以使用它来提供基于单个逻辑的排序。

     Comparator接口被用来提供不同的排序算法，我们可以选择需要使用的Comparator来对给定的对象集合进行排序。

13. 并发集合类是什么

     Java1.5并发包（java.util.concurrent）包含线程安全集合类，允许在迭代时修改集合。迭代器被设计为fail-fast的，会抛出ConcurrentModificationException。一部分类为：CopyOnWriteArrayList、 ConcurrentHashMap、CopyOnWriteArraySet。

14. 我们如何对一组对象进行排序

    如果我们需要对一个对象数组进行排序，我们可以使用Arrays.sort()方法。如果我们需要排序一个对象列表，我们可以使用Collection.sort()方法。

    两个类都有用于自然排序（使用Comparable）或基于标准的排序（使用Comparator）的重载方法sort()。Collections内部使用数组排序方法，所有它们两者都有相同的性能，只是Collections需要花时间将列表转换为数组。

15. 当一个集合被作为参数传递给一个函数时，如何才可以确保函数不能修改它

    在作为参数传递之前，我们可以使用Collections.unmodifiableCollection(Collection c)方法创建一个只读集合，这将确保改变集合的任何操作都会抛出UnsupportedOperationException。





### 参考

1. [ 40个Java集合面试问题和答案 ](https://www.cnblogs.com/dengcl/p/8046739.html)
2. [合集](https://www.cnblogs.com/WPF0414/p/9885619.html)
3. [面试必备：30个Java集合面试问题及答案](https://www.jianshu.com/p/e6076b43fdfb)