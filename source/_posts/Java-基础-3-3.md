---
title: Java 集合类的秘密
date: 2019-10-06 08:20:59
tags:
 - Java
categories:
 - Java
 - 基础
---

![MSIXIU.png](https://s2.ax1x.com/2019/11/05/MSIXIU.png)

<!--more-->

#### 集合视图

集合视图并没有一个明确的定义。拿映射类`keySet`方法类比，初看起来，好像这个方法创建了一个新集，并将映射中的所有键都填进去，然后返回这个集。但是情况并非如此。取而代之的是，`keySet`方法返回一个实现`Set`接口的类对象，这个类的方法对原映射进行操作。这种集合称为视图。

**轻量级集包装器**

**Arrays类的静态方法asList 将返回一个包装了普通 java 数组的List 包装器。** 这个方法可以将数组传递给一个期望得到列表或集合变元的方法， 例如：

```
Card[] cardDeck = new Card[25];
List<Card> cardList = Arrays.asList(cardDeck); （数组转List视图对象）
```

对以上代码的分析（Analysis）：

- 返回的对象不是 ArrayList。 它是一个视图对象， 带有访问底层数组的get和set方法；而改变该数组大小的所有方法（与迭代器相关的add 和 remove方法） 都会抛出一个UnsupportedOperationException 异常；

- asList方法： 它已经被声明为一个具有可变参数的方法， 除了可以传递一个数组外， 还可以将各个元素直接传递给这个方法：

  ```
  List<String> names =  Arrays.asList<"a", "b", "c">;
  //（1）该方法不适用于基本数据类型（byte,short,int,long,float,double,boolean）
  //（2）该方法将数组与列表链接起来，当更新其中之一时，另一个自动更新
  //（3）不支持add和remove方法
  ```

- 以上这个方法调用 Collections.nCopies(n , anObject) ： 将返回一个实现了 List接口的**不可修改对象**， 并给人一种包含n个元素， 每个元素都是 Object的 错觉；

  ```
  List<String> settings = Collections.nCopies(100, "DEFAULT"); 
  System.out.println(settings.get(0)==settings.get(1));
  //true
  
  ```

**子范围**

1. 可以为很多集合建立子范围视图；

   如， `List g2 = staff.subList(10,20);` 从列表staff 中取出 第10个~第19个元素；（第一个 索引包含在内，第二个索引不包含在内）

2. 可以将任何操作应用到子范围， 并且能够自动地反应整个列表的情况；

   如， `g2.clear()；` 现在， 元素自动地从 staff 列表中清除了，并且g2 为空；

3. 对于有序集合映射表， 可以使用排序顺序而不是元素位置建立子范围。

   ```java
   //SortedSet 接口说明了3个方法
   SortedSet<E> subSet(E from ,E to)
   SortedSet<E> headSet(E to)
   SortedSet<E> tailSet(E from)
   //以上方法将返回 大于等于from 小于to的所有元素子集；
   
   //有序映射表有类似方法
   SortedSet<E> subMap(E from ,E to)
   SortedSet<E> headMap(E to)
   SortedSet<E> tailMap(E from)
   //返回映射表视图， 该映射表包含键落在指定范围内的所有元素；
   ```

    java SE6 引入的 NavigableSet 接口 赋予子范围操作更多 的控制能力。 可以指定是否包括边界：

   ```java
   NabigableSet<E> subSet(E from, boolean fromInclusive, E to, boolean toInclusive);
   NabigableSet<E> headSet(E to, boolean toInclusive);
   NabigableSet<E> tailSet(E from, boolean fromInclusive);
   ```

**不可修改视图**

Collections 还有几个方法， 用于产生集合的不可修改视图。 这些视图对现有集合增加了一个运行时的检查。 如果发现试图对集合进行修改， 就抛出一个异常， 同时这个集合将保持未修改状态；

（再次提醒：注意区分　Collection 和 Collections）

可以使用下列6种方法获得不可修改视图：

```java
Collections.unmodifiableCollection
Collections.unmodifiableList
Collections.unmodifiableSet
Collections.unmodifiableSortedSet
Collections.unmodifiableMap
Collections.unmodifiableSortedMap
```

每个方法都定义于一个接口。如， Collections.unmodifiableList　与 ArrayList、LinkedList 或者任何实现了 List接口的其他类一起协同工作；

1. Collections.unmodifiableList 方法返回一个实现List接口的类对象。当然，lookAt方法 可以调用 List 接口中的所有方法， 而不只是访问器。但是所有的更改器方法，已经被重新定义为 抛出一个 UnsuportedOperationException 异常，而不是 将调用传递给底层集合；

   ```
   List<String> staff = new LinkedList<>();
   lookAt（Collections.unmodifiableList(staff)）; // 返回一个不可修改视图， 传递给 loopAt方法；
   ```

2. **不可修改视图并不是 集合本身不可修改** **（干货——只是无法通过其投影出的视图修改原始集合）。** 仍然可以通过集合的原始引用对集合进行修改。并且仍然可以让集合的元素调用更改方法；

3. **由于视图只是包装了 接口而不是 实际的集合对象， 所以只能访问 接口中定义的方法；** 如， LinkedList 类有一些非常方便的方法， addFirst 和 addLast ， 它们都不是 List接口的方法，不能通过修改视图进行访问；

**同步视图**

如果由**多个线程访问**集合，就必须确保集不会被意外破坏， 类库的设计者使用视图及直接来确保常规集合的线程安全。

例如， Collections 类的 静态 synchronizedMap 方法将任何一个映射表转换成具有同步访问方法的 Map：

```java
Map<String, Employee> map = Collections.synchronizedMap(new HashMap<String, Employee>());
```

**检查视图**

出现的问题：

```java
ArrayList<String> strings = new ArrayList<>();
ArrayList rawList = strings;
rawList.add(new Date());//
```

这个错误的 add 命令在运行时检测不到。 相反，只有在 调用 get方法， 并将其 转换为 String 时，这个类才会抛出一个异常；

解决方法：检查视图可以探测到这类问题。下面定义了一个安全列表：`List safeString = Collections.checkedList(strings, String.class);`视图的add 方法将检测 插入的对象是否属于给定的类。 如果不属于给定的类， 就立即抛出一个ClassCastException。 这样做的好处是错误可以在正确的位置得以报告：

```
ArrayList rawList = safeString;
rawList.add(new Date()); // checked list chrows a ClassCastException
```

被检测视图受限于 虚拟机可以运行的运行时检查。例如， 对于ArrayList

更多的视图相关方法参考附录

### 原理

参考：<https://www.cnblogs.com/yysbolg/p/9230184.html>

#### Collections

**Collections.SynchronizedMap/List/Set/Collection底层实现原理**

Collections.synchronizedMap()实现原理是Collections定义了一个SynchronizedMap的内部类，并返回这个类的实例。

```java
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
  return new SynchronizedMap<>(m);
}
```

SynchronizedMap这个类实现了Map接口，**在调用方法时使用synchronized来保证线程同步**,当然了**实际上操作的还是我们传入的HashMap实例**，简单的说就是Collections.synchronizedMap()方法帮我们在操作HashMap时自动添加了synchronized来实现线程同步，类似的其它Collections.synchronizedXX方法也是类似原理）。

Mutex在构造时默认赋值为this，即所有方法都用的同一个锁，m就是我们传入的map。

**SynchronizedMap和ConcurrentHashMap 区别**

Collections.synchronizedMap()和Hashtable一样，实现上在调用map所有方法时，都对整个map进行同步，而ConcurrentHashMap的实现却更加精细，它对map中的所有桶加了锁。所以，只要要有一个线程访问map，其他线程就无法进入map，而如果一个线程在访问ConcurrentHashMap某个桶时，其他线程，仍然可以对map执行某些操作。这样，ConcurrentHashMap在性能以及安全性方面，明显比Collections.synchronizedMap()更加有优势。同时，同步操作精确控制到桶，所以，即使在遍历map时，其他线程试图对map进行数据修改，也不会抛出ConcurrentModificationException。

ConcurrentHashMap从类的命名就能看出，它必然是个HashMap。而Collections.synchronizedMap()可以接收任意Map实例，实现Map的同步

线程安全，并且锁分离。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hashtable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。 

#### Set

**元素无放入顺序，元素不可重复（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的）**

Set具有与Collection完全一样的接口，因此没有任何额外的功能,只是行为不同。这是继承与多态思想的典型应用：表现不同的行为。

Set不保存重复的元素(至于如何判断元素相同则较为负责)

存入Set的每个元素都必须是唯一的，因为Set不保存重复元素,加入Set的元素必须定义equals()方法以确保对象的唯一性。

Set 是基于对象的值来确定归属性的。

Set本身有去重功能是因为String内部重写了hashCode()和equals()方法，在add里实现了去重, Set集合是不允许重复元素的，但是集合是不知道我们对象的重复的判断依据的，默认情况下判断依据是判断两者是否为同一元素（euqals方法，依据是元素==元素），如果要依据我们自己的判断来判断元素是否重复，需要重写元素的equals方法（元素比较相等时调用）hashCode()的返回值是元素的哈希码，如果两个元素的哈希码相同，那么需要进行equals判断。【所以可以自定义返回值作为哈希码】 equals()返回true代表两元素相同，返回false代表不同。

- set集合没有索引，只能用迭代器或增强for循环遍历
- set的底层是map集合,可以看HashSet源码看到。
- Set最多有一个null元素

必须小心操作可变对象（Mutable Object）。如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。

Set具有与Collection完全一样的接口，没有额外的任何功能。所以把Set就是Collection，只是行为不同（这就是多态）；Set是基于对象的值来判断归属的，由于查询速度非常快速，HashSet使用了散列，HashSet维护的顺序与TreeSet或LinkedHashSet都不同，因为它们的数据结构都不同，元素存储方式自然也不同。TreeSet的数据结构是“红-黑树”，HashSet是散列函数，LinkedHashSet也用了散列函数；如果想要对结果进行排序，那么选择TreeSet代替HashSet是个不错的选择

#### HashSet

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{

    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    //因为底层是HashMap，这个PRESENT是默认的value
    private static final Object PRESENT = new Object();

    /**
     *  HashSet底层是HashMap
     */
    public HashSet() {
        map = new HashMap<>();
    }
    
    /**
     *  这个构造方法是用于LinkedHashSet来调用的，可以保证插入的顺序
     * 
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    
    public int size() {
        return map.size();
    }

    public boolean isEmpty() {
        return map.isEmpty();
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    /**
     * 这里可以看到PRESENT作为value值，而插入的对象作为key存入HashMap
     * HashMap的put方法会通过key计算除哈希值，找到对应的位置的数据，如果已经存在就会替换老的数据，并把老的数据返回，相当于没有插入key
     * HashSet以此来保证不会有重复的数据插入
     */
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    /**
     * 同样是调用HashMap的remove方法来删除数据
     */
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
    
    /**************equals方法在AbstractSet中，通过这个方法来判断是否是同一个对象***************/
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection c = (Collection) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }
    }
}
```

- HashSet底层是基于HashMap来实现，所以也不是线程安全的
- HashSet中不能插入相同的对象，且允许插入null
- HashSet仅仅存储对象，而不是键值对
- HashSet没有get方法，通过add方法存储对象
- LinkedHashSet底层是基于LinkedHashMap实现



#### Spliterator是什么？

Spliterator是一个可分割迭代器(splitable iterator)，可以和iterator顺序遍历迭代器一起看。jdk1.8发布后，对于并行处理的能力大大增强，Spliterator就是为了并行遍历元素而设计的一个迭代器，jdk1.8中的集合框架中的数据结构都默认实现了spliterator，后面我们也会结合ArrayList中的spliterator()一起解析。

**Spliterator内部结构**

```java
//单个对元素执行给定的动作，如果有剩下元素未处理返回true，否则返回false
boolean tryAdvance(Consumer<? super T> action);
 
//对每个剩余元素执行给定的动作，依次处理，直到所有元素已被处理或被异常终止。默认方法调用tryAdvance方法
default void forEachRemaining(Consumer<? super T> action) {
   do { } while (tryAdvance(action));
}
 
//对任务分割，返回一个新的Spliterator迭代器
Spliterator<T> trySplit();
 
//用于估算还剩下多少个元素需要遍历
long estimateSize();
 
//当迭代器拥有SIZED特征时，返回剩余元素个数；否则返回-1
default long getExactSizeIfKnown() {
   return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
}
 
 //返回当前对象有哪些特征值
int characteristics();
 
//是否具有当前特征值
default boolean hasCharacteristics(int characteristics) {
   return (characteristics() & characteristics) == characteristics;
}
//如果Spliterator的list是通过Comparator排序的，则返回Comparator
//如果Spliterator的list是自然排序的 ，则返回null
//其他情况下抛错
default Comparator<? super T> getComparator() {
    throw new IllegalStateException();
}
```

特征值其实就是为表示该Spliterator有哪些特性，用于可以更好控制和优化Spliterator的使用。关于获取比较器getComparator这一个方法，目前我还没看到具体使用的地方，所以可能理解有些误差。特征值如下：





### 附录1

视图相关类、方法以及对应版本

```java
6.java.util.Collections 1.2：
static <E> Collection unmodifiableCollection(Collection<E> c)

static <E> List unmodifiableList(List<E> c)

static <E> Set unmodifiableSet(Set<E> c)

static <E> SortedSet unmodifiableSortedSet(SortedSet<E> c)

static <E> SortedSet unmodifiableNavigableSet(NavigableSet<E> c) 8

static <K,V> Map unmodifiableMap(Map<K,V> c)

static <K,V> SortedMap unmodifiableSortedMap(SortedMp<K,V> c)

static <K,V> SortedMap unmodifiableNavigableMap(NavigableMap <K,V> c) 8

构造一个集合视图；视图的更改器方法抛出一个UnsupportedOperationException

static <E> Collection<E> synchronizedCollection(Collection<E> c)

static <E> List synchronizedList(List<E> c)

static <E> Set synchronizedSet(Set<E> c)

static <E> SortedSet synchronizedSortedSet(SortedSet<E> c)

static <E> NavigableSet synchronizedNavigableSet(NavigableSet<E> c) 8

static <K,V> Map<K,V> synchronizedMap(Map<K,V> c)

static <K,V> SortedMap<K,V> synchronizedSortedMap(SortedMap<K,V> c)

static <K,V> NavigableMap<K,V> synchronizedNavigableMap(NavigableMap<K,V> c) 8

构造一个集合视图；视图的方法同步

static <E> Collection checkCollection(Collection<E> c,Class<E> elementType)

static <E> List checkedList(List<E> c,Class<E> elementType)

static <E> Set checkedSet(Set<E> c,Class<E> elementType)

static <E> SortedSet checkedSortedSet(SortedSet<E> c,Class<E> elementType)

static <E> NavigableSet checkedNavigableSet(NavigableSet<E> c,Class<E> elementType) 8

static <K,V> Map checkedMap(Map<K,V> c,Class<K> keyType,Class<V> valueType)

static <K,V> SortedMap checkedSortedMap(SortedMap<K,V> c,Class<K> keyType,Class<V> valueType)

static <K,V> NavigableMap checkedNavigableMap(NavigableMap<K,V>,Class<K> keyType,Class<V> valueType) 8

static <E> Queue<E> checkedQueue(Queue<E> queue,Class<E> elementType) 8

构造一个集合视图；如果插入一个错误类型的元素，视图的方法抛出一个ClassCastException

static <E> List<E> nCopies(int n,E value)

static <E> Set<E> singleton(E value)

static <E> List<E> singletonList(E value)

static <K,V> Map<K,V> singletonMap(K key,V value)

构造一个对象视图，可以是一个包含n个元素的不可修改列表，也可以是一个单元素集、列表或映射

static <E> List<E> emptyList()

static <T> Set<T> emptySet()

static <E> SortedSet<E> emptySortedSet()

static NavigableSet<E> emptyNavigableSet()

static <K,V> Map<K,V> emptyMap()

static <K,V> SortedMap<K,V> emptySortedMap()

static <K,V> NavigableMap<K,V> emptySortedMap()

static <T> Enumeration<T> emptyEnumeration()

static <T> Iterator<T> emptyIterator()

static <T> ListIterator<T> emptyListIterator()

生成一个空集合、映射或迭代器

7.java.util.Arrays 1.2：
static <E> List<E> asList(E... array)

返回一个数组元素的列表。这个数组是可修改的，但其大小不可变

8.java.util.List<E> 1.2：
List<E> subList(int firstIncluded,int firstExcluded)

返回给定位置范围内的所有元素的列表视图

9.java.util.SortedSet<E> 1.2：
SortedSet<E> subSet(E firstIncluded,int firstExcluded)

SortedSet<E> headSet(E firstExcluded)

SortedSet<E> tailSet(E firstIncluded)

返回给定范围内的元素视图

10.java.util.Navigable<E> 6：
NavigableSet<E> subSet(E from,boolean fromIncluded,E to,boolean toIncluded)

NavigableSet<E> headSet(E to,boolean toIncluded)

NavigableSet<E> tailSet(E from,boolean fromIncluded)

返回给定范围内的元素视图。boolean标志决定视图是否包含边界

11.java.util.SortedMap<K,V> 1.2：
SortedMap<K,V> subMap(K firstIncluded,K firstExcluded)

SortedMap<K,V> headMap(K firstExcluded)

SortedMap<K,V> tailMap(K firstIncluded)

返回在给定范围内的键条目的映射视图

12.java.util.NavigableMap<K,V> 6：
NavigableMap<K,V> subMap(K from,boolean fromIncluded,K,to,boolean toIncluded)

NavigableMap<K,V> headMap(K from,boolean fromIncluded)

NavigableMap<K,V> tailMap(K,to,boolean toIncluded)

返回在给定范围内的键条目的映射视图。boolean标志决定视图是否包含边界
```

