---
title: Java 集合类(一)
date: 2019-04-06 08:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
在编写java程序中，我们最常用的除了八种基本数据类型，String对象外还有一个集合类，在我们的的程序中到处充斥着集合类的身影！java中集合大家族的成员实在是太丰富了，有常用的ArrayList、HashMap、HashSet，也有不常用的Stack、Queue，有线程安全的Vector、HashTable，也有线程不安全的LinkedList、TreeMap等等！

<!--more-->

![1.jpg](https://i.loli.net/2019/06/25/5d11cf909d22a28956.jpg)

数组（可以存储基本数据类型）是用来存现对象的一种容器，但是数组的长度固定，不适合在对象数量未知的情况下使用。

集合（只能存储对象，对象类型可以不一样）的长度可变，可在多数情况下使用。

1. Iterator：Collection（值）、Map（键值对）；

2. Collection：Set（无序不重复）、List（有序可重复）、Queue；

3. Set：HashSet（基于HashMap实现）、LinkedHashSet（继承自HashSet）、TreeSet（底层基于HashMap实现，升序排列）；

4. List：ArrayList（基于数组实现，默认初始容量为10，增速1.5倍）、Vector（初始容量为10，增速2倍）、Stack（继承自Vector）、LinkedList（基于链表实现）；

5. Map：HashMap、HashTable、TreeMap（基于红黑树实现，按key值排序）；

6. 线程安全的集合类：（喂，SHE），Vector、Stack、HashTable、Enumeration。以及使用Collections.synchronizedXXX()包装的安全集合。

7. 不论Collection的实际类型如何，它都支持一个iterator()的方法，该方法返回一个迭代子，使用该迭代子即可逐一访问Collection中每一个元素。典型的用法如下：

   ```java
   Iterator it = collection.iterator(); // 获得一个迭代子
   while(it.hasNext()) {
   Object obj = it.next(); // 得到下一个元素
   
   }
   ```

   List还提供一个listIterator()方法，返回一个 ListIterator接口（实现Iterator），和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素，还能向前或向后遍历。

8. 快速失败（Fail-Fast）机制：它是Java集合的一种错误检测机制。**当多个线程对集合进行结构上的改变的操作时，有可能会产生fail-fast机制。记住是有可能，而不是一定。**例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。

   ```java
   public static void main(String[] args) {
           Vector<Integer> vector=new Vector<>();
           for (int i = 0; i < 10; i++) {
               vector.add(i);
           }
   
           Thread t1=new Thread(new Runnable() {
               @Override
               public void run() {
                   System.out.println(vector);
                   Iterator<Integer> iterator =
                           vector.iterator();
                   while (iterator.hasNext())
                   {
                       System.out.println(iterator.next());
                       try {
                           Thread.sleep(1000);
                       } catch (InterruptedException e) {
                           e.printStackTrace();
                       }
                   }
                   System.out.println(vector);
               }
           });
           Thread t2=new Thread(new Runnable() {
               @Override
               public void run() {
                   System.out.println(vector);
                   vector.add(100);
               }
           });
           t1.start();
           t2.start();
       }
   ```

   

### Collection接口

Collection接口是最基本的集合接口，它不提供直接的实现，Java SDK提供的类都是继承自Collection的“子接口”如List和Set。Collection所代表的是一种规则，它所包含的元素都必须遵循一条或者多条规则。如有些允许重复而有些则不能重复、有些必须要按照顺序插入而有些则是散列，有些支持排序但是有些则不支持。

在Java中所有实现了Collection接口的类都必须提供两套标准的构造函数，一个是无参，用于创建一个空的Collection，一个是带有Collection参数的有参构造函数，用于创建一个新的Collection，这个新的Collection与传入进来的Collection具备相同的元素。

Collection接口是Set、List和Queue接口的父接口，基本操作包括：

  - add(Object o)：增加元素
  - addAll(Collection c)：将另一个集合内的成员添加到该集合
  - clear()：清除集合内所有对象
  - contains(Object o)：是否包含指定元素
  - containsAll(Collection c)：是否包含集合c中的所有元素
  - iterator()：返回Iterator对象，用于遍历集合中的元素
  - remove(Object o)：移除元素
  - removeAll(Collection c)：相当于减集合c
  - retainAll(Collection c)：相当于求与c的交集
  - size()：返回元素个数
  - toArray()：把集合转换为一个数组

Collection的遍历可以使用Iterator接口或者是foreach循环来实现

**为什么Collection的遍历可以使用Iterator接口或者是foreach循环来实现**

容器之间的关系：

![容器分类.jpg](https://i.loli.net/2019/04/12/5cafdf96557d4.jpg)

可以看到Collection可以产生Iterator，那么Iterator又是如何工作的呢？

Iterator有三个函数：
```
  boolean hasNext();//指向的集合是否有下一个元素
  E next(); //获取指向集合的元素，并指向下一元素
  void remove();
```

以下例子是利用了Iterator接口的着三个方法，实现遍历ArrayList<String\>类型。
一开始迭代器在所有元素的左边，调用next()之后，迭代器移到第一个和第二个元素之间，next()方法返回迭代器刚刚经过的元素。
hasNext()若返回True，则表明接下来还有元素，迭代器不在尾部。
remove()方法必须和next方法一起使用，功能是去除刚刚next方法返回的元素。

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;

public class ForEachDemo {
    public static void main(String... arg) {
        Collection<String> a = new ArrayList<String>();
        a.add("Bob");
        a.add("Alice");
        a.add("Lisy");

        Iterator<String> iterator = a.iterator();
        while (iterator.hasNext()) {
            String ele = iterator.next();
            System.out.println(ele);//Bob  Alice  Lisy    
        }
        System.out.println(a);//[Bob, Alice, Lisy]  
        iterator = a.iterator();
        iterator.next();
        iterator.remove();
        System.out.println(a);//[Alice, Lisy]  
    }
}
```

for-each循环可以与任何实现了Iterable接口的对象一起工作。

Iterable接口只有一个方法，就是返回一个Iterator。

```
Iterator<T> iterator();
```

而java.util.Collection接口继承java.lang.Iterable，故标准类库中的任何集合都可以使用for-each循环,也可以使用iterator()来返回iterator。

**为什么一定要去实现Iterable这个接口呢？ 为什么不直接实现Iterator接口呢？**

看一下JDK中的集合类，比如List一族或者Set一族，都是继承了Iterable接口，但并不直接继承Iterator接口。

仔细想一下这么做是有道理的。因为Iterator接口的核心方法next()或者hasNext()，是依赖于迭代器的当前迭代位置的。

如果Collection直接继承Iterator接口，势必导致集合对象中包含当前迭代位置的数据(指针)。当集合在不同方法间被传递时，由于当前迭代位置不可预置，那么next()方法的结果会变成不可预知。除非再为Iterator接口添加一个reset()方法，用来重置当前迭代位置。但即时这样，Collection也只能同时存在一个当前迭代位置。

而Iterable则不然，**每次调用都会返回一个从头开始计数的迭代器。多个迭代器是互不干扰的**。

通过上面的框架图，其各个类、接口如下：

- Collection：Collection 层次结构 中的根接口。它表示一组对象，这些对象也称为 collection 的元素。对于Collection而言，它不提供任何直接的实现，所有的实现全部由它的子类负责。
- AbstractCollection：提供 Collection 接口的骨干实现，以最大限度地减少了实现此接口所需的工作。对于我们而言要实现一个不可修改的 collection，只需扩展此类，并提供 iterator 和 size 方法的实现。但要实现可修改的 collection，就必须另外重写此类的 add 方法（否则，会抛出 UnsupportedOperationException），iterator 方法返回的迭代器还必须另外实现其 remove 方法。
- Iterator：迭代器。
- ListIterator：系列表迭代器，允许程序员按任一方向遍历列表、迭代期间修改列表，并获得迭代器在列表中的当前位置。
- List：继承于Collection的接口。它代表着有序的队列。
- AbstractList：List 接口的骨干实现，以最大限度地减少实现“随机访问”数据存储（如数组）支持的该接口所需的工作。
- Queue：队列。提供队列基本的插入、获取、检查操作。
- Deque：一个线性 collection，支持在两端插入和移除元素。大多数 Deque 实现对于它们能够包含的元素数没有固定限制，但此接口既支持有容量限制的双端队列，也支持没有固定大小限制的双端队列。
- AbstractSequentialList：提供了 List 接口的骨干实现，从而最大限度地减少了实现受“连续访问”数据存储（如链接列表）支持的此接口所需的工作。从某种意义上说，此类与在列表的列表迭代器上实现“随机访问”方法。
- LinkedList：List 接口的链接列表实现。它实现所有可选的列表操作。
- ArrayList：List 接口的大小可变数组的实现。它实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。
- Vector：实现可增长的对象数组。与数组一样，它包含可以使用整数索引进行访问的组件。
- Stack：后进先出（LIFO）的对象堆栈。它通过五个操作对类 Vector 进行了扩展 ，允许将向量视为堆栈。
- Enumeration：枚举，实现了该接口的对象，它生成一系列元素，一次生成一个。连续调用 nextElement 方法将返回一系列的连续元素。这种传统接口已被迭代器取代，虽然Enumeration 还未被遗弃，但在现代代码中已经被很少使用了。尽管如此，它还是使用在诸如Vector和Properties这些传统类所定义的方法中，除此之外，还用在一些API类，并且在应用程序中也广泛被使用。
- Map：“键值”对映射的抽象接口。该映射不包括重复的键，一个键对应一个值。
- SortedMap：有序的键值对接口，继承Map接口。
- NavigableMap：继承SortedMap，具有了针对给定搜索目标返回最接近匹配项的导航方法的接口。
- AbstractMap：实现了Map中的绝大部分函数接口。它减少了“Map的实现类”的重复编码。
- Dictionary：任何可将键映射到相应值的类的抽象父类。目前被Map接口取代。
- TreeMap：有序散列表，实现SortedMap 接口，底层通过红黑树实现。
- HashMap：是基于“拉链法”实现的散列表。底层采用“数组+链表”实现。
- WeakHashMap：基于“拉链法”实现的散列表。
- HashTable：基于“拉链法”实现的散列表。

### List

List接口为Collection直接接口。List所代表的是有序的Collection，即它用某种特定的插入顺序来维护元素顺序。用户可以对列表中每个元素的插入位置进行精确地控制，同时可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。实现List接口的集合主要有：ArrayList、LinkedList、Vector、Stack。

List子接口是有序集合，所以与Set相比，增加了与索引位置相关的操作：

  - add(int index, Object o)：在指定位置插入元素
  - addAll(int index, Collection c)：...
  - get(int index)：取得指定位置元素
  - indexOf(Obejct o)：返回对象o在集合中第一次出现的位置
  - lastIndexOf(Object o)：...
  - remove(int index)：删除并返回指定位置的元素
  - set(int index, Object o)：替换指定位置元素
  - subList(int fromIndex, int endIndex)：返回子集合

#### ArrayList

ArrayList是一个动态数组，也是我们最常用的集合。它允许任何符合规则的元素插入甚至包括null。每一个ArrayList都有一个初始容量（10），该容量代表了数组的大小。随着容器中的元素不断增加，容器的大小也会随着增加。在每次向容器中增加元素的同时都会进行容量检查，当快溢出时，就会进行扩容操作。所以如果我们明确所插入元素的多少，最好指定一个初始容量值，避免过多的进行扩容操作而浪费时间、效率。

size、isEmpty、get、set、iterator 和 listIterator 操作都以固定时间运行。add 操作以分摊的固定时间运行，也就是说，添加 n 个元素需要 O(n) 时间（由于要考虑到扩容，所以这不只是添加元素会带来分摊固定时间开销那样简单）。

ArrayList擅长于随机访问。同时ArrayList是非同步的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。所以为了保证同步，最好的办法是在创建时完成，以防止意外对列表进行不同步的访问：

```
List list = Collections.synchronizedList(new ArrayList(...));
```

ArrayList是实现List接口的，底层采用数组实现，所以它的操作基本上都是基于对数组的操作。

ArrayList提供了三个构造函数：

```
ArrayList()：默认构造函数，提供初始容量为10的空列表。
ArrayList(int initialCapacity)：构造一个具有指定初始容量的空列表。
ArrayList(Collection<? extends E> c)：构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。
```

ArrayList提供了add(E e)、add(int index, E element)、addAll(Collection<? extends E> c)、addAll(int index, Collection<? extends E> c)、set(int index, E element)这个五个方法来实现ArrayList增加。允许null值。

```
add(E e)：将指定的元素添加到此列表的尾部。
add(int index, E element)：将指定的元素插入此列表中的指定位置
addAll(Collection<? extends E> c)：按照指定 collection 的迭代器所返回的元素顺序，将该 collection 中的所有元素添加到此列表的尾部。
addAll(int index, Collection<? extends E> c)：从指定的位置开始，将指定 collection 中的所有元素插入到此列表中。
set(int index, E element)：用指定的元素替代此列表中指定位置上的元素。
```

ArrayList提供了remove(int index)、remove(Object o)、removeRange(int fromIndex, int toIndex)、removeAll()四个方法进行元素的删除。

ArrayList提供了get(int index)用读取ArrayList中的元素。由于ArrayList是动态数组，所以我们完全可以根据下标来获取ArrayList中的元素，而且速度还比较快，故**ArrayList长于随机访问**

在前面就提过ArrayList每次新增元素时都会需要进行容量检测判断，若新增元素后元素的个数会超过ArrayList的容量，就会进行扩容操作来满足新增元素的需求，扩容步长为1.5。所以当我们清楚知道业务数据量或者需要插入大量元素前，我可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

处理这个ensureCapacity()这个扩容数组外，ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能。它可以通过trimToSize()方法来实现。该方法可以最小化ArrayList实例的存储量。


#### LinkedList

同样实现List接口的LinkedList与ArrayList不同，ArrayList是一个动态数组，而LinkedList是一个双向链表。所以它除了有ArrayList的基本操作方法外还额外提供了get，remove，insert方法在LinkedList的首部或尾部。**这些操作使LinkedList可被用作堆栈（stack），队列（queue）或双向队列（deque）。**

由于实现的方式不同，LinkedList不能随机访问，它所有的操作都是要**按照双重链表的需要执行**。在列表中索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。这样做的好处就是可以通过较低的代价在List中进行插入和删除操作。

与ArrayList一样，LinkedList也是**非同步的**。如果多个线程同时访问一个List，则必须自己实现访问同步。一种解决方法是在创建List时构造一个同步的List：

```java
List list = Collections.synchronizedList(new LinkedList(...));
```

除了实现 List 接口外，LinkedList 类还为在列表的开头及结尾 get、remove 和 insert 元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。

此类实现 Deque 接口，为 add、poll 提供先进先出队列操作，以及其他堆栈和双端队列操作。

所有操作都是按照双重链接列表的需要执行的。在列表中编索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。

LinkedList提高了两个构造方法：LinkedLis()和LinkedList(Collection<? extends E> c)。

```
LinkedList()构造一个空列表。里面没有任何元素，仅仅只是将header节点的前一个元素、后一个元素都指向自身。
LinkedList(Collection<? extends E> c)： 构造一个包含指定 collection 中的元素的列表，这些元素按其 collection 的迭代器返回的顺序排列。该构造函数首先会调用LinkedList()，构造一个空列表，然后调用了addAll()方法将Collection中的所有元素添加到列表中。
```

增加方法：

```
 add(E e): 将指定元素添加到此列表的结尾，该方法调用addBefore方法，然后直接返回true，对于addBefore()而已，它为LinkedList的私有方法。
add(int index, E element)：在此列表中指定的位置插入指定的元素。
addAll(Collection<? extends E> c)：添加指定 collection 中的所有元素到此列表的结尾，顺序是指定 collection 的迭代器返回这些元素的顺序。
addAll(int index, Collection<? extends E> c)：将指定 collection 中的所有元素从指定位置开始插入此列表。
AddFirst(E e): 将指定元素插入此列表的开头。
addLast(E e): 将指定元素添加到此列表的结尾。
```

 移除方法：

 ```
 remove(Object o)：从此列表中移除首次出现的指定元素（如果存在）。 该方法首先会判断移除的元素是否为null，然后迭代这个链表找到该元素节点，最后调用remove(Entry<E> e)，remove(Entry<E> e)为私有方法，是LinkedList中所有移除方法的基础方法
 clear()： 从此列表中移除所有元素。
remove()：获取并移除此列表的头（第一个元素）。
remove(int index)：移除此列表中指定位置处的元素。
remove(Objec o)：从此列表中移除首次出现的指定元素（如果存在）。
removeFirst()：移除并返回此列表的第一个元素。
removeFirstOccurrence(Object o)：从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表时）。
removeLast()：移除并返回此列表的最后一个元素。
removeLastOccurrence(Object o)：从此列表中移除最后一次出现的指定元素（从头部到尾部遍历列表时）。
 ```

查找方法：

```
get(int index)：返回此列表中指定位置处的元素。
getFirst()：返回此列表的第一个元素。
getLast()：返回此列表的最后一个元素。
indexOf(Object o)：返回此列表中首次出现的指定元素的索引，如果此列表中不包含该元素，则返回 -1。
lastIndexOf(Object o)：返回此列表中最后出现的指定元素的索引，如果此列表中不包含该元素，则返回 -1。
```



**Aarraylist和Linkedlist**

1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。
   这一点要看实际情况的。**若只对单条数据插入或删除，ArrayList的速度反而优于LinkedList。但若是批量随机的插入删除数据，LinkedList的速度大大优于ArrayList**. 因为ArrayList每插入一条数据，要移动插入点及之后的所有数据。

#### Vector

与ArrayList相似，但是Vector是同步的。所以说Vector是**线程安全的动态数组**。它的操作与ArrayList几乎一样。

**Vector和ArrayList**

1. vector是线程同步的，所以它也是线程安全的，而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高。
2. 如果集合中的元素的数目大于目前集合数组的长度时，vector增长率为目前数组长度的100%,而arraylist增长率为目前数组长度的50%.如过在集合中使用数据量比较大的数据，用vector有一定的优势。
3. 如果查找一个指定位置的数据，vector和arraylist使用的时间是相同的，都是0(1),这个时候使用vector和arraylist都可以。而如果移动一个指定位置的数据花费的时间为0(n-i)n为总长度，这个时候就应该考虑到使用linklist,因为它移动一个指定位置的数据所花费的时间为0(1),而查询一个指定位置的数据时花费的时间为0(i)。

ArrayList 和Vector是采用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要设计到数组元素移动等内存操作，所以索引数据快插入数据慢，Vector由于使用了synchronized方法（线程安全）所以性能上比ArrayList要差，LinkedList使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快！

**不推荐使用Vector类，即使需要考虑同步，即也可以通过其它方法实现。同样我们也可以通过ArrayDeque类或LinkedList类实现“栈”的相关功能。所以Vector与子类Stack，建议放进历史吧**

```java
ArrayList<Integer> list = new ArrayList<Integer>();
list.iterator();
list.listIterator();
Collections.synchronizedList(list);

LinkedList<Integer> linkedList=new LinkedList<Integer>();
//栈
linkedList.push(1);
linkedList.pop();
//队列
linkedList.offer(2);
linkedList.poll();

//双向队列
linkedList.offerFirst(3);
linkedList.pollFirst();

linkedList.offerLast(4);
linkedList.pollLast();

//使用信号量进行加锁，synchronize加锁mutex变量，默认为集合的this
Collections.synchronizedList(linkedList);

linkedList.forEach(System.out::println);

Vector<Integer> vector=new Vector<>();
```

#### Stack

 Stack继承自Vector，实现一个后进先出的堆栈。Stack提供5个额外的方法使得Vector得以被当作堆栈使用。基本的push和pop 方法，还有peek方法得到栈顶的元素，empty方法测试堆栈是否为空，search方法检测一个元素在堆栈中的位置。Stack刚创建后是空栈。

### Map接口

Map与List、Set接口不同，它是由一系列键值对组成的集合，提供了key到Value的映射。同时它也没有继承Collection。在Map中它保证了key与value之间的一一对应关系。也就是说一个key对应一个value，所以它不能存在相同的key值，当然value值可以相同。实现map的有：HashMap、TreeMap、HashTable、Properties、EnumMap。

JDK1.8 之前 HashMap 由 **数组+链表** 组成的（**“链表散列”** 即数组和链表的结合体），数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（HashMap 采用 **“拉链法也就是链地址法”** 解决冲突），如果定位到的数组位置不含链表（当前  entry 的 next 指向 null  ）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为 O(1)，因为最新的 Entry  会插入链表头部，急需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过 key 对象的 equals 方法逐一比对查找.

> 所谓 **“拉链法”** 就是将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

相比于之前的版本， JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。

> TreeMap、TreeSet以及JDK1.8之后的HashMap底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

#### HashMap

以哈希表数据结构实现，查找对象时通过哈希函数计算其位置，它是为快速查询而设计的，其内部定义了一个hash表数组（Entry[] table），元素会通过哈希转换函数将元素的哈希地址转换成数组中存放的索引，如果有冲突，则使用**散列链表的形式将所有相同哈希地址的元素串起来**，可能通过查看HashMap.Entry的源码它是一个单链表结构。

#### TreeMap

键以某种排序规则排序，内部以red-black（红-黑）树数据结构实现，实现了SortedMap接口


**HashMap与TreeMap**

1. HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap。HashMap中元素的排列顺序是不固定的）。

2. HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。集合框架”提供两种常规的Map实现：HashMap和TreeMap (TreeMap实现SortedMap接口)。

3. 在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 equals()的实现。 这个TreeMap没有调优选项，因为该树总处于平衡状态。

#### HashTable

也是以哈希表数据结构实现的，解决冲突时与HashMap也一样也是采用了**散列链表**的形式，不过性能比HashMap要低

**hashtable与hashmap**

1. **线程是否安全：** HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过 `synchronized` 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；
2. **效率：** 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；
3. **对Null key 和Null value的支持：** HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null，但是在 HashTable 中 put 进的键值只要有一个 null，直接抛NullPointerException。
4. **初始容量大小和每次扩充容量大小的不同 ：** ①创建时如果不指定容量初始值，Hashtable  默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap  默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而  HashMap 会将其扩充为2的幂次方大小。也就是说 HashMap 总是使用2的幂作为哈希表的大小,后面会介绍到为什么是2的幂次方。
5. **底层数据结构：** JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

对于在Map中插入、删除和定位元素这类操作，HashMap是最好的选择。然而，假如你需要对一个有序的key集合进行遍历，TreeMap是更好的选择。基于你的collection的大小，也许向HashMap中添加元素会更快，将map换为TreeMap进行有序key的遍历。

**Map优化**

使用一个较大的数组让元素能够均匀分布。在Map有两个会影响到其效率，一是容器的初始化大小、二是负载因子。

在哈希映射表中，内部数组中的每个位置称作“存储桶”(bucket)，而可用的存储桶数（即内部数组的大小）称作容量 (capacity)，我们为了使Map对象能够有效地处理任意数的元素，将Map设计成可以调整自身的大小。我们知道当Map中的元素达到一定量的时候就会调整容器自身的大小，但是这个调整大小的过程其开销是非常大的。调整大小需要将原来所有的元素插入到新数组中。我们知道index = hash(key) % length。这样可能会导致原先冲突的键不在冲突，不冲突的键现在冲突的，重新计算、调整、插入的过程开销是非常大的，效率也比较低下。所以，如果我们开始知道Map的预期大小值，将Map调整的足够大，则可以大大减少甚至不需要重新调整大小，这很有可能会提高速度。

为了确认何时需要调整Map容器，Map使用了一个额外的参数并且粗略计算存储容器的密度。在Map调整大小之前，使用”负载因子”来指示Map将会承担的“负载量”，也就是它的负载程度，当容器中元素的数量达到了这个“负载量”，则Map将会进行扩容操作。负载因子、容量、Map大小之间的关系如下：负载因子 * 容量 > map大小  ----->调整Map大小。

例如：如果负载因子大小为0.75（HashMap的默认值），默认容量为11，则 11 * 0.75 = 8.25 = 8，所以当我们容器中插入第八个元素的时候，Map就会调整大小。

负载因子本身就是在控件和时间之间的折衷。当我使用较小的负载因子时，虽然降低了冲突的可能性，使得单个链表的长度减小了，加快了访问和更新的速度，但是它占用了更多的控件，使得数组中的大部分控件没有得到利用，元素分布比较稀疏，同时由于Map频繁的调整大小，可能会降低性能。但是如果负载因子过大，会使得元素分布比较紧凑，导致产生冲突的可能性加大，从而访问、更新速度较慢。所以我们一般推荐不更改负载因子的值，采用默认值0.75.

### Set

Set是一种不包括重复元素的Collection。它维持它自己的内部排序，所以随机访问没有任何意义。与List一样，它同样运行null的存在但是仅有一个。由于Set接口的特殊性，所有传入Set集合中的元素都必须不同，同时要注意任何可变对象，如果在对集合中元素进行操作时，导致e1.equals(e2)==true，则必定会产生某些问题。实现了Set接口的集合有：EnumSet、HashSet、TreeSet。

#### EnumSet

是枚举的专用Set。所有的元素都是枚举类型。

```java
// EnumSet - a modern replacement for bit fields - Page 160
import java.util.*;

public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) {
        // Body goes here
    }

    // Sample use
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

#### HashSet

HashSet堪称查询速度最快的集合，因为其内部是以HashCode来实现的。它内部元素的顺序是由哈希码来决定的，所以它**不保证set 的迭代顺序**；特别是它**不保证该顺序恒久不变**。

特点：

  - 不能保证元素的排列顺序，加入的元素要特别注意hashCode()方法的实现。
  - HashSet**不是同步**的，多线程访问同一步HashSet对象时，需要手工同步。
  - 集合元素值**可以是null**。

####  LinkedHashSet

LinkedHashSet类也是根据元素的hashCode值来决定元素的存储位置，但它同时使用链表维护元素的次序。与HashSet相比，特点：

  - 对集合迭代时，按增加顺序返回元素。
  - 性能略低于HashSet，因为需要维护元素的插入顺序。但迭代访问元素时会有好性能，因为它采用链表维护内部顺序。

#### SortedSet接口及TreeSet实现类

基于TreeMap，生成一个总是处于排序状态的set，内部以TreeMap来实现。它是使用元素的自然顺序对元素进行排序，或者根据创建Set 时提供的 Comparator 进行排序，具体取决于使用的构造方法。

TreeSet类是SortedSet接口的实现类。因为需要排序，所以**性能肯定差于HashSet**。与HashSet相比，额外增加的方法有：

  - first()：返回第一个元素
  - last()：返回最后一个元素
  - lower(Object o)：返回指定元素之前的元素
  - higher(Obect o)：返回指定元素之后的元素
  - subSet(fromElement, toElement)：返回子集合

可以定义比较器（Comparator）来实现自定义的排序。默认自然升序排序。


### Queue

队列，它主要分为两大类，一类是阻塞式队列，队列满了以后再插入元素则会抛出异常，主要包括ArrayBlockQueue、PriorityBlockingQueue、LinkedBlockingQueue。另一种队列则是双端队列，支持在头、尾两端插入和移除元素，主要包括：ArrayDeque、LinkedBlockingDeque、LinkedList。

Queue用于模拟队列这种数据结构，实现“FIFO”等数据结构。通常，队列不允许随机访问队列中的元素。

Queue 接口并未定义阻塞队列的方法，而这在并发编程中是很常见的。BlockingQueue 接口定义了那些等待元素出现或等待队列中有可用空间的方法，这些方法扩展了此接口。

Queue 实现通常不允许插入 null 元素，尽管某些实现（如 LinkedList）并不禁止插入 null。即使在允许 null 的实现中，也不应该将 null 插入到 Queue 中，因为 null 也用作 poll 方法的一个特殊返回值，表明队列不包含元素。

基本操作：

  - boolean add(E e) : 将元素加入到队尾，不建议使用
  - boolean offer(E e): 将指定的元素插入此队列（如果立即可行且不会违反容量限制），当使用有容量限制的队列时，此方法通常要优于 add(E)，后者可能无法插入元素，而只是抛出一个异常。推荐使用此方法取代add
  - E remove(): 获取头部元素并且删除元素，不建议使用
  - E poll(): 获取头部元素并且删除元素，队列为空返回null；推荐使用此方法取代remove
  - E element(): 获取但是不移除此队列的头
  - E peek(): 获取队列头部元素却不删除元素，队列为空返回null

#### PriorityQueue

PriorityQueue保存队列元素的顺序并不是按照加入队列的顺序，而是按队列元素的大小重新排序。当调用peek()或者是poll()方法时，返回的是队列中最小的元素。当然你可以与TreeSet一样，可以自定义排序。


#### Deque子接口与ArrayDeque类

Deque代表一个双端队列，可以当作一个双端队列使用，也可以当作“栈”来使用，因为它包含出栈pop()与入栈push()方法。

ArrayDeque类为Deque的实现类，数组方式实现。方法有：

  - addFirst(Object o)：元素增加至队列开头
  - addLast(Object o)：元素增加至队列末尾
  - poolFirst()：获取并删除队列第一个元素，队列为空返回null
  - poolLast()：获取并删除队列最后一个元素，队列为空返回null
  - pop()：“栈”方法，出栈，相当于removeFirst()
  - push(Object o)：“栈”方法，入栈，相当于addFirst()
  - removeFirst()：获取并删除队列第一个元素
  - removeLast()：获取并删除队列最后一个元素

#### 实现List接口与Deque接口的LinkedList类

LinkedList类是List接口的实现类，同时它也实现了Deque接口。因此它也可以当做一个双端队列来用，也可以当作“栈”来使用。并且，它是以链表的形式来实现的，这样的结果是它的随机访问集合中的元素时性能较差，但插入与删除操作性能非常出色。

### 参考

1. [ 40个Java集合面试问题和答案 ](https://www.cnblogs.com/dengcl/p/8046739.html)
2. [合集](https://www.cnblogs.com/WPF0414/p/9885619.html)
3. [面试必备：30个Java集合面试问题及答案](https://www.jianshu.com/p/e6076b43fdfb)