---
title: 其他容器类--从源码理解原理（基于JDK 1.8）
date: 2020-1-18 20:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

上篇文章分析了Map类，这篇文章将会分析其他的容器类。

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

   - HashTable: 数组+链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的

   - TreeMap: 红黑树（自平衡的排序二叉树）

<!--more-->

### List

#### ArrayList

`ArrayList` 是一种变长的集合类，基于定长数组实现。ArrayList 允许空值和重复元素，当往 ArrayList 中添加的元素数量大于其底层数组容量时，其会通过扩容机制重新生成一个更大的数组。另外，由于 ArrayList 底层基于数组实现，所以其可以保证在 `O(1)` 复杂度下完成随机查找操作。其他方面，ArrayList 是非线程安全类，并发环境下，多个线程同时操作 ArrayList，会引发不可预知的错误。

ArrayList 是大家最为常用的集合类，作为一个变长集合类，其核心是扩容机制。所以只要知道它是怎么扩容的，以及基本的操作是怎样实现就够了。

##### 构造方法

ArrayList 有两个构造方法，一个是无参，另一个需传入初始容量值。大家平时最常用的是无参构造方法，相关代码如下：

```java
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData;
    
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

上面的代码比较简单，两个构造方法做的事情并不复杂，目的都是初始化底层数组 elementData。区别在于无参构造方法会将 elementData 初始化一个空数组，插入元素时，扩容将会按默认值重新初始化数组。而有参的构造方法则会将 elementData 初始化为参数值大小（>= 0）的数组。一般情况下，我们用默认的构造方法即可。倘若在可知道将会向 ArrayList 插入多少元素的情况下，应该使用有参构造方法。按需分配，避免浪费。

##### 插入

对于数组（线性表）结构，插入操作分为两种情况。一种是在元素序列尾部插入，另一种是在元素序列其他位置插入。ArrayList 的源码里也体现了这两种插入情况，如下：

```java
/** 在元素序列尾部插入 */
public boolean add(E e) {
    // 1. 检测是否需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 2. 将新元素插入序列尾部
    elementData[size++] = e;
    return true;
}

/** 在元素序列 index 位置处插入 */
public void add(int index, E element) {
    rangeCheckForAdd(index);

    // 1. 检测是否需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 2. 将 index 及其之后的所有元素都向后移一位
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 3. 将新元素插入至 index 处
    elementData[index] = element;
    size++;
}
```

对于在元素序列尾部插入，这种情况比较简单，只需两个步骤即可：

1. 检测数组是否有足够的空间插入
2. 将新元素插入至序列尾部

如下图：

![1pqRmj.png](https://s2.ax1x.com/2020/01/18/1pqRmj.png)

如果是在元素序列指定位置（假设该位置合理）插入，则情况稍微复杂一点，需要三个步骤：

1. 检测数组是否有足够的空间
2. 将 index 及其之后的所有元素向后移一位
3. 将新元素插入至 index 处

如下图：

![1pqIhV.png](https://s2.ax1x.com/2020/01/18/1pqIhV.png)

从上图可以看出，将新元素插入至序列指定位置，需要先将该位置及其之后的元素都向后移动一位，为新元素腾出位置。这个操作的时间复杂度为`O(N)`，频繁移动元素可能会导致效率问题，特别是集合中元素数量较多时。在日常开发中，若非所需，我们应当尽量避免在大集合中调用第二个插入方法。

以上是 ArrayList 插入相关的分析，上面的分析以及配图均未体现扩容机制。那么下面就来简单分析一下 ArrayList 的扩容机制。对于变长数据结构，当结构中没有空余空间可供使用时，就需要进行扩容。在 ArrayList 中，当空间用完，其会按照原数组空间的1.5倍进行扩容。相关源码如下：

```java
/** 计算最小容量 */
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/** 扩容的入口方法 */
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/** 扩容的核心方法 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // newCapacity = oldCapacity + oldCapacity / 2 = oldCapacity * 1.5
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 扩容
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    // 如果最小容量超过 MAX_ARRAY_SIZE，则将数组容量扩容至 Integer.MAX_VALUE
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

##### 删除

不同于插入操作，ArrayList 没有无参删除方法。所以其只能删除指定位置的元素或删除指定元素，这样就无法避免移动元素（除非从元素序列的尾部删除）。相关代码如下：

```java
/** 删除指定位置的元素 */
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    // 返回被删除的元素值
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将 index + 1 及之后的元素向前移动一位，覆盖被删除值
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 将最后一个元素置空，并将 size 值减1                
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

E elementData(int index) {
    return (E) elementData[index];
}

/** 删除指定元素，若元素重复，则只删除下标最小的元素 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        // 遍历数组，查找要删除元素的位置
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/** 快速删除，不做边界检查，也不返回删除的元素值 */
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

上面的删除方法并不复杂，这里以第一个删除方法为例，删除一个元素步骤如下：

1. 获取指定位置 index 处的元素值
2. 将 index + 1 及之后的元素向前移动一位
3. 将最后一个元素置空，并将 size 值减 1
4. 返回被删除值，完成删除操作

现在，考虑这样一种情况。我们往 ArrayList 插入大量元素后，又删除很多元素，此时底层数组会空闲处大量的空间。因为 ArrayList 没有自动缩容机制，导致底层数组大量的空闲空间不能被释放，造成浪费。对于这种情况，ArrayList 也提供了相应的处理方法，如下：

```java
/** 将数组容量缩小至元素数量 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

通过上面的方法，我们可以手动触发 ArrayList 的缩容机制。这样就可以释放多余的空间，提高空间利用率。

![1pLCcD.png](https://s2.ax1x.com/2020/01/18/1pLCcD.png)

##### 遍历

ArrayList 实现了 RandomAccess 接口（该接口是个标志性接口），表明它具有随机访问的能力。ArrayList 底层基于数组实现，所以它可在常数阶的时间内完成随机访问，效率很高。对 ArrayList 进行遍历时，一般情况下，我们喜欢使用 foreach 循环遍历，但这并不是推荐的遍历方式。ArrayList 具有随机访问的能力，如果在一些效率要求比较高的场景下，更推荐下面这种方式：

```java
for (int i = 0; i < list.size(); i++) {
    list.get(i);
}
```

至于原因也不难理解，foreach 最终会被转换成迭代器遍历的形式，效率不如上面的遍历方式。

##### 快速失败机制

在 Java 集合框架中，很多类都实现了快速失败机制。该机制被触发时，会抛出并发修改异常`ConcurrentModificationException`，这个异常大家在平时开发中多多少少应该都碰到过。关于快速失败机制，ArrayList 的注释里对此做了解释，这里引用一下：

> The iterators returned by this class’s iterator() and
> listIterator(int) methods are fail-fast
> if the list is structurally modified at any time after the iterator is
> created, in any way except through the iterator’s own
> ListIterator remove() or ListIterator add(Object) methods,
> the iterator will throw a ConcurrentModificationException. Thus, in the face of
> concurrent modification, the iterator fails quickly and cleanly, rather
> than risking arbitrary, non-deterministic behavior at an undetermined
> time in the future.

上面注释大致意思是，ArrayList 迭代器中的方法都是均具有快速失败的特性，当遇到并发修改的情况时，迭代器会快速失败，以避免程序在将来不确定的时间里出现不确定的行为。

以上就是 Java 集合框架中引入快速失败机制的原因，并不难理解，这里不多说了。

遍历时删除是一个不正确的操作，即使有时候代码不出现异常，但执行逻辑也会出现问题。关于这个问题，阿里巴巴 Java 开发手册里也有所提及。这里引用一下：

> 【强制】不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator 方式，如果并发操作，需要对 Iterator 对象加锁。

相关代码（稍作修改）如下：

```java
List<String> a = new ArrayList<String>();
	a.add("1");
	a.add("2");
	for (String temp : a) {
	    System.out.println(temp);
	    if("1".equals(temp)){
	        a.remove(temp);
	    }
	}
}
```

相信有些朋友应该看过这个，并且也执行过上面的程序。上面的程序执行起来不会虽不会出现异常，但代码执行逻辑上却有问题，只不过这个问题隐藏的比较深。我们把 temp 变量打印出来，会发现只打印了数字`1`，`2`没打印出来。初看这个执行结果确实很让人诧异，不明原因。如果死抠上面的代码，我们很难找出原因，此时需要稍微转换一下思路。我们都知道 Java 中的 foreach 是个语法糖，编译成字节码后会被转成用迭代器遍历的方式。所以我们可以把上面的代码转换一下，等价于下面形式：

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
Iterator<String> it = a.iterator();
while (it.hasNext()) {
    String temp = it.next();
    System.out.println("temp: " + temp);
    if("1".equals(temp)){
        a.remove(temp);
    }
}
```

这个时候，我们再去分析一下 ArrayList 的迭代器源码就能找出原因。

```java
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        // 并发修改检测，检测不通过则抛出异常
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
    
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
    
    // 省略不相关的代码
}
```

我们一步一步执行一下上面的代码，第一次进入 while 循环时，一切正常，元素 1 也被删除了。但删除元素 1 后，就无法再进入 while 循环，此时 it.hasNext() 为 false。原因是删除元素 1 后，元素计数器 size = 1，而迭代器中的 cursor 也等于 1，从而导致 it.hasNext() 返回false。归根结底，上面的代码段没抛异常的原因是，循环提前结束，导致 next 方法没有机会抛异常。不信的话，大家可以把代码稍微修改一下，即可发现问题：

```java
List<String> a = new ArrayList<>();
a.add("1");
a.add("2");
a.add("3");
Iterator<String> it = a.iterator();
while (it.hasNext()) {
    String temp = it.next();
    System.out.println("temp: " + temp);
    if("1".equals(temp)){
        a.remove(temp);
    }
}
```

以上是关于遍历时删除的分析，在日常开发中，我们要避免上面的做法。正确的做法使用迭代器提供的删除方法，而不是直接删除。

#### Vector

1. Vector是线程安全的集合类，ArrayList并不是线程安全的类。Vector类对集合的元素操作时都加了synchronized，保证线程安全。
2. Vector与ArrayList本质上都是一个Object[] 数组，ArrayList提供了size属性，Vector提供了elementCount属性，他们的作用是记录集合内有效元素的个数。与我们平常调用的arrayList.size()和vector.size()一样返回的集合内有效元素的个数。
3. Vector与ArrayList的扩容并不一样，**Vector默认扩容是增长一倍的容量，Arraylist是增长50%的容量**。
4. Vector与ArrayList的remove,add(index,obj)方法都会导致内部数组进行数据拷贝的操作，这样在大数据量时，可能会影响效率。
5. Vector与ArrayList的add(obj)方法，如果新增的有效元素个数超过数组本身的长度，都会导致数组进行扩容。

**如果你说,Vector是同步的,你要在多线程使用.那你应该使用java.util.concurrent.CopyOnWriteArrayList等而不是Vector.**

#### Stack

```
public class Stack<E>extends Vector<E>
```

由于Vector是通过数组实现的，这就意味着，Stack也是通过数组实现的，而非链表。

Stack类表示后进先出（LIFO）的对象堆栈。它通过五个操作对类Vector进行了扩展 ，允许将向量视为堆栈。它提供了通常的push和pop操作，以及取堆栈顶点的peek方法、测试堆栈是否为空的empty方法、在堆栈中查找项并确定到堆栈顶距离的search方法。

首次创建堆栈时，它不包含项。

**Deque** **接口及其实现提供了 LIFO 堆栈操作的更完整和更一致的 set，应该优先使用此 set，而非此类**。例如：

```
Deque<Integer> stack = new ArrayDeque<Integer>();
```

**如果你要使用Stack做类似的业务.那么非线程的你可以选择linkedList,多线程情况你可以选择java.util.concurrent.ConcurrentLinkedDeque 或者java.util.concurrent.ConcurrentLinkedQueue**

**多线程情况下,应尽量使用java.util.concurrent包下的类.**

#### LinkedList

LinkedList 是 Java 集合框架中一个重要的实现，其底层采用的双向链表结构。和 ArrayList 一样，LinkedList 也支持空值和重复值。由于 LinkedList 基于链表实现，存储元素过程中，无需像 ArrayList 那样进行扩容。但有得必有失，LinkedList 存储元素的节点需要额外的空间存储前驱和后继的引用。另一方面，LinkedList 在链表头部和尾部插入效率比较高，但在指定位置进行插入时，效率一般。原因是，在指定位置插入需要定位到该位置处的节点，此操作的时间复杂度为`O(N)`。最后，LinkedList 是非线程安全的集合类，并发环境下，多个线程同时操作 LinkedList，会引发不可预知的错误。

LinkedList 的继承体系较为复杂，继承自 AbstractSequentialList，同时又实现了 List 和 Deque 接口。

![1pOrQS.png](https://s2.ax1x.com/2020/01/18/1pOrQS.png)

LinkedList 继承自 AbstractSequentialList，AbstractSequentialList 又是什么呢？从实现上，AbstractSequentialList 提供了一套基于顺序访问的接口。通过继承此类，子类仅需实现部分代码即可拥有完整的一套访问某种序列表（比如链表）的接口。深入源码，AbstractSequentialList 提供的方法基本上都是通过 ListIterator 实现的，比如：

```java
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

public void add(int index, E element) {
    try {
        listIterator(index).add(element);
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}

// 留给子类实现
public abstract ListIterator<E> listIterator(int index);
```

所以只要继承类实现了 listIterator 方法，它不需要再额外实现什么即可使用。对于随机访问集合类一般建议继承 AbstractList 而不是 AbstractSequentialList。LinkedList 和其父类一样，也是基于顺序访问。所以 LinkedList 继承了 AbstractSequentialList，但 LinkedList 并没有直接使用父类的方法，而是重新实现了一套的方法。

另外，LinkedList 还实现了 Deque (double ended queue)，Deque 又继承自 Queue 接口。这样 LinkedList 就具备了队列的功能。比如，我们可以这样使用：

```
Queue<T> queue = new LinkedList<>();
```

除此之外，我们基于 LinkedList 还可以实现一些其他的数据结构，比如栈，以此来替换 Java 集合框架中的 Stack 类（该类实现的不好，《Java 编程思想》一书的作者也对此类进行了吐槽）。

##### 成员

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

##### 构造函数

```java
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

##### 查找

LinkedList 底层基于链表结构，无法向 ArrayList 那样随机访问指定位置的元素。LinkedList 查找过程要稍麻烦一些，需要从链表头结点（或尾节点）向后查找，时间复杂度为 `O(N)`。相关源码如下：

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

Node<E> node(int index) {
    /*
     * 则从头节点开始查找，否则从尾节点查找
     * 查找位置 index 如果小于节点数量的一半，
     */    
    if (index < (size >> 1)) {
        Node<E> x = first;
        // 循环向后查找，直至 i == index
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

上面的代码比较简单，主要是通过遍历的方式定位目标位置的节点。获取到节点后，取出节点存储的值返回即可。这里面有个小优化，即通过比较 index 与节点数量 size/2 的大小，决定从头结点还是尾节点进行查找。

##### 遍历

链表的遍历过程也很简单，和上面查找过程类似，我们从头节点往后遍历就行了。但对于 LinkedList 的遍历还是需要注意一些，不然可能会导致代码效率低下。通常情况下，我们会使用 foreach 遍历 LinkedList，而 foreach 最终转换成迭代器形式。所以分析 LinkedList 的遍历的核心就是它的迭代器实现，相关代码如下：

```java
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}

private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;
    private int nextIndex;
    private int expectedModCount = modCount;

    /** 构造方法将 next 引用指向指定位置的节点 */
    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();

        lastReturned = next;
        next = next.next;    // 调用 next 方法后，next 引用都会指向他的后继节点
        nextIndex++;
        return lastReturned.item;
    }
    
    // 省略部分方法
}
```

我们都知道 LinkedList 不擅长随机位置访问，如果大家用随机访问的方式遍历 LinkedList，效率会很差。比如下面的代码：

```java
List<Integet> list = new LinkedList<>();
list.add(1)
list.add(2)
......
for (int i = 0; i < list.size(); i++) {
    Integet item = list.get(i);
    // do something
}
```

当链表中存储的元素很多时，上面的遍历方式对于效率来说就是灾难。原因在于，通过上面的方式每获取一个元素，LinkedList 都需要从头节点（或尾节点）进行遍历，效率不可谓不低。上面的遍历方式在大数据量情况下，效率很差。大家在日常开发中应该尽量避免这种用法。

##### 插入

LinkedList 除了实现了 List 接口相关方法，还实现了 Deque 接口的很多方法，所以我们有很多种方式插入元素。但这里，我只打算分析 List 接口中相关的插入方法，其他的方法大家自己看吧。LinkedList 插入元素的过程实际上就是链表链入节点的过程，学过数据结构的同学对此应该都很熟悉了。这里简单分析一下，先看源码吧：

```java
/** 在链表尾部插入元素 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/** 在链表指定位置插入元素 */
public void add(int index, E element) {
    checkPositionIndex(index);

    // 判断 index 是不是链表尾部位置，如果是，直接将元素节点插入链表尾部即可
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

/** 将元素节点插入到链表尾部 */
void linkLast(E e) {
    final Node<E> l = last;
    // 创建节点，并指定节点前驱为链表尾节点 last，后继引用为空
    final Node<E> newNode = new Node<>(l, e, null);
    // 将 last 引用指向新节点
    last = newNode;
    // 判断尾节点是否为空，为空表示当前链表还没有节点
    if (l == null)
        first = newNode;
    else
        l.next = newNode;    // 让原尾节点后继引用 next 指向新的尾节点
    size++;
    modCount++;
}

/** 将元素节点插入到 succ 之前的位置 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    // 1. 初始化节点，并指明前驱和后继节点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 2. 将 succ 节点前驱引用 prev 指向新节点
    succ.prev = newNode;
    // 判断尾节点是否为空，为空表示当前链表还没有节点    
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;   // 3. succ 节点前驱的后继引用指向新节点
    size++;
    modCount++;
}
```

上面是插入过程的源码，我对源码进行了比较详细的注释，应该不难看懂。上面两个 add 方法只是对操作链表的方法做了一层包装，核心逻辑在 linkBefore 和 linkLast 中。这里以 linkBefore 为例，它的逻辑流程如下：

1. 创建新节点，并指明新节点的前驱和后继
2. 将 succ 的前驱引用指向新节点
3. 如果 succ 的前驱不为空，则将 succ 前驱的后继引用指向新节点

对应于下图：

![1pXel8.png](https://s2.ax1x.com/2020/01/18/1pXel8.png)

##### 删除

如果大家看懂了上面的插入源码分析，那么再看删除操作实际上也很简单了。删除操作通过解除待删除节点与前后节点的链接，即可完成任务。过程比较简单，看源码吧：

```java
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        // 遍历链表，找到要删除的节点
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);    // 将节点从链表中移除
                return true;
            }
        }
    }
    return false;
}

public E remove(int index) {
    checkElementIndex(index);
    // 通过 node 方法定位节点，并调用 unlink 将节点从链表中移除
    return unlink(node(index));
}

/** 将某个节点从链表中移除 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    
    // prev 为空，表明删除的是头节点
    if (prev == null) {
        first = next;
    } else {
        // 将 x 的前驱的后继指向 x 的后继
        prev.next = next;
        // 将 x 的前驱引用置空，断开与前驱的链接
        x.prev = null;
    }

    // next 为空，表明删除的是尾节点
    if (next == null) {
        last = prev;
    } else {
        // 将 x 的后继的前驱指向 x 的前驱
        next.prev = prev;
        // 将 x 的后继引用置空，断开与后继的链接
        x.next = null;
    }

    // 将 item 置空，方便 GC 回收
    x.item = null;
    size--;
    modCount++;
    return element;
}
```

和插入操作一样，删除操作方法也是对底层方法的一层保证，核心逻辑在底层 unlink 方法中。所以长驱直入，直接分析 unlink 方法吧。unlink 方法的逻辑如下（假设删除的节点既不是头节点，也不是尾节点）：

1. 将待删除节点 x 的前驱的后继指向 x 的后继
2. 将待删除节点 x 的前驱引用置空，断开与前驱的链接
3. 将待删除节点 x 的后继的前驱指向 x 的前驱
4. 将待删除节点 x 的后继引用置空，断开与后继的链接

对应下图：

![1pXYlT.png](https://s2.ax1x.com/2020/01/18/1pXYlT.png)

### Set

#### HashSet

基于 HashMap 实现的，底层采用 HashMap 来保存元素,使用其key唯一性，扩容方式与HashMap相同

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;
//使用HashMap充当存储，k为存储值，v为PRESENT
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
}
```

**构造函数**

```java
public HashSet() {
    map = new HashMap<>();
}

public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

// 非public，主要是给LinkedHashSet使用的
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

构造方法都是调用HashMap对应的构造方法。

最后一个构造方法有点特殊，它不是public的，意味着它只能被同一个包或者子类调用，这是LinkedHashSet专属的方法。

HashSet就是使用HashMap的KeySet进行存储和操作。内部操作跟HashMap相同

**总结**

1. HashSet内部使用HashMap的key存储元素，以此来保证元素不重复；
2. HashSet是无序的，因为HashMap的key是无序的；
3. HashSet中允许有一个null元素，因为HashMap允许key为null；
4. HashSet是非线程安全的；
5. HashSet是没有get()方法的；

**问题**

1. 集合（Collection）和集合（Set）有什么区别？

2. HashSet怎么保证添加元素不重复？

3. HashSet是否允许null元素？

4. HashSet是有序的吗？

5. HashSet是同步的吗？

6. 什么是fail-fast？

   fail-fast机制是java集合中的一种错误机制。

   当使用迭代器迭代时，如果发现集合有修改，则快速失败做出响应，抛出ConcurrentModificationException异常。

   这种修改有可能是其它线程的修改，也有可能是当前线程自己的修改导致的，比如迭代的过程中直接调用remove()删除元素等。

   另外，并不是java中所有的集合都有fail-fast的机制。比如，像最终一致性的ConcurrentHashMap、CopyOnWriterArrayList等都是没有fast-fail的。

   那么，fail-fast是怎么实现的呢？

   细心的同学可能会发现，像ArrayList、HashMap中都有一个属性叫`modCount`，每次对集合的修改这个值都会加1，在遍历前记录这个值到`expectedModCount`中，遍历中检查两者是否一致，如果出现不一致就说明有修改，则抛出ConcurrentModificationException异常。

#### LinkedHashSet

 LinkedHashSet 继承与 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 Hashmap 实现一样，不过还是有一点点区别的。

LinkedHashSet继承自HashSet，让我们直接上源码来看看它们有什么不同。

```java
package java.util;

// LinkedHashSet继承自HashSet
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2851667679971038690L;

    // 传入容量和装载因子
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
    
    // 只传入容量, 装载因子默认为0.75
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
    
    // 使用默认容量16, 默认装载因子0.75
    public LinkedHashSet() {
        super(16, .75f, true);
    }

    // 将集合c中的所有元素添加到LinkedHashSet中
    // 好奇怪, 这里计算容量的方式又变了
    // HashSet中使用的是Math.max((int) (c.size()/.75f) + 1, 16)
    // 这一点有点不得其解, 是作者偷懒？
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
    
    // 可分割的迭代器, 主要用于多线程并行迭代处理时使用
    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT | Spliterator.ORDERED);
    }
}
```

完了，结束了，就这么多，这是全部源码了，真的。

可以看到，LinkedHashSet中一共提供了5个方法，其中4个是构造方法，还有一个是迭代器。

4个构造方法都是调用父类的`super(initialCapacity, loadFactor, true);`这个方法。

这个方法长什么样呢？

```java
    // HashSet的构造方法
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

如上所示，这个构造方法里面使用了LinkedHashMap来初始化HashSet中的map。

现在这个逻辑应该很清晰了，LinkedHashSet继承自HashSet，它的添加、删除、查询等方法都是直接用的HashSet的，唯一的不同就是它使用LinkedHashMap存储元素。

LinkedHashSet是不支持按访问顺序对元素排序的，只能按插入顺序排序。

**问题**

1. LinkedHashSet的底层使用什么存储元素？

2. LinkedHashSet与HashSet有什么不同？

3. LinkedHashSet是有序的吗？

4. LinkedHashSet支持按元素访问顺序排序吗？

   首先，LinkedHashSet所有的构造方法都是调用HashSet的同一个构造方法，如下：

   ```
       // HashSet的构造方法
       HashSet(int initialCapacity, float loadFactor, boolean dummy) {
           map = new LinkedHashMap<>(initialCapacity, loadFactor);
       }
   ```

   然后，通过调用LinkedHashMap的构造方法初始化map，如下所示：

   ```
       public LinkedHashMap(int initialCapacity, float loadFactor) {
           super(initialCapacity, loadFactor);
           accessOrder = false;
       }
   ```

   可以看到，这里把accessOrder写死为false了。

   所以，LinkedHashSet是不支持按访问顺序对元素排序的，只能按插入顺序排序。

#### TreeSet

内部使用TreeMap，红黑树(自平衡的排序二叉树)

```java
package java.util;

// TreeSet实现了NavigableSet接口，所以它是有序的
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    // 元素存储在NavigableMap中
    // 注意它不一定就是TreeMap
    private transient NavigableMap<E,Object> m;

    // 虚拟元素, 用来作为value存储在map中
    private static final Object PRESENT = new Object();

    // 直接使用传进来的NavigableMap存储元素
    // 这里不是深拷贝,如果外面的map有增删元素也会反映到这里
    // 而且, 这个方法不是public的, 说明只能给同包使用
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }

    // 使用TreeMap初始化
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }

    // 使用带comparator的TreeMap初始化
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }

    // 将集合c中的所有元素添加的TreeSet中
    public TreeSet(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    // 将SortedSet中的所有元素添加到TreeSet中
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }

    // 迭代器
    public Iterator<E> iterator() {
        return m.navigableKeySet().iterator();
    }

    // 逆序迭代器
    public Iterator<E> descendingIterator() {
        return m.descendingKeySet().iterator();
    }

    // 以逆序返回一个新的TreeSet
    public NavigableSet<E> descendingSet() {
        return new TreeSet<>(m.descendingMap());
    }

    // 元素个数
    public int size() {
        return m.size();
    }

    // 判断是否为空
    public boolean isEmpty() {
        return m.isEmpty();
    }

    // 判断是否包含某元素
    public boolean contains(Object o) {
        return m.containsKey(o);
    }

    // 添加元素, 调用map的put()方法, value为PRESENT
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
    
    // 删除元素
    public boolean remove(Object o) {
        return m.remove(o)==PRESENT;
    }

    // 清空所有元素
    public void clear() {
        m.clear();
    }

    // 添加集合c中的所有元素
    public  boolean addAll(Collection<? extends E> c) {
        // 满足一定条件时直接调用TreeMap的addAllForTreeSet()方法添加元素
        if (m.size()==0 && c.size() > 0 &&
            c instanceof SortedSet &&
            m instanceof TreeMap) {
            SortedSet<? extends E> set = (SortedSet<? extends E>) c;
            TreeMap<E,Object> map = (TreeMap<E, Object>) m;
            Comparator<?> cc = set.comparator();
            Comparator<? super E> mc = map.comparator();
            if (cc==mc || (cc != null && cc.equals(mc))) {
                map.addAllForTreeSet(set, PRESENT);
                return true;
            }
        }
        // 不满足上述条件, 调用父类的addAll()通过遍历的方式一个一个地添加元素
        return super.addAll(c);
    }

    // 子set（NavigableSet中的方法）
    public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                                  E toElement,   boolean toInclusive) {
        return new TreeSet<>(m.subMap(fromElement, fromInclusive,
                                       toElement,   toInclusive));
    }
    
    // 头set（NavigableSet中的方法）
    public NavigableSet<E> headSet(E toElement, boolean inclusive) {
        return new TreeSet<>(m.headMap(toElement, inclusive));
    }

    // 尾set（NavigableSet中的方法）
    public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
        return new TreeSet<>(m.tailMap(fromElement, inclusive));
    }

    // 子set（SortedSet接口中的方法）
    public SortedSet<E> subSet(E fromElement, E toElement) {
        return subSet(fromElement, true, toElement, false);
    }

    // 头set（SortedSet接口中的方法）
    public SortedSet<E> headSet(E toElement) {
        return headSet(toElement, false);
    }
    
    // 尾set（SortedSet接口中的方法）
    public SortedSet<E> tailSet(E fromElement) {
        return tailSet(fromElement, true);
    }

    // 比较器
    public Comparator<? super E> comparator() {
        return m.comparator();
    }

    // 返回最小的元素
    public E first() {
        return m.firstKey();
    }
    
    // 返回最大的元素
    public E last() {
        return m.lastKey();
    }

    // 返回小于e的最大的元素
    public E lower(E e) {
        return m.lowerKey(e);
    }

    // 返回小于等于e的最大的元素
    public E floor(E e) {
        return m.floorKey(e);
    }
    
    // 返回大于等于e的最小的元素
    public E ceiling(E e) {
        return m.ceilingKey(e);
    }
    
    // 返回大于e的最小的元素
    public E higher(E e) {
        return m.higherKey(e);
    }
    
    // 弹出最小的元素
    public E pollFirst() {
        Map.Entry<E,?> e = m.pollFirstEntry();
        return (e == null) ? null : e.getKey();
    }

    public E pollLast() {
        Map.Entry<E,?> e = m.pollLastEntry();
        return (e == null) ? null : e.getKey();
    }

    // 克隆方法
    @SuppressWarnings("unchecked")
    public Object clone() {
        TreeSet<E> clone;
        try {
            clone = (TreeSet<E>) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }

        clone.m = new TreeMap<>(m);
        return clone;
    }

    // 序列化写出方法
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden stuff
        s.defaultWriteObject();

        // Write out Comparator
        s.writeObject(m.comparator());

        // Write out size
        s.writeInt(m.size());

        // Write out all elements in the proper order.
        for (E e : m.keySet())
            s.writeObject(e);
    }

    // 序列化写入方法
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden stuff
        s.defaultReadObject();

        // Read in Comparator
        @SuppressWarnings("unchecked")
            Comparator<? super E> c = (Comparator<? super E>) s.readObject();

        // Create backing TreeMap
        TreeMap<E,Object> tm = new TreeMap<>(c);
        m = tm;

        // Read in size
        int size = s.readInt();

        tm.readTreeSet(size, s, PRESENT);
    }

    // 可分割的迭代器
    public Spliterator<E> spliterator() {
        return TreeMap.keySpliteratorFor(m);
    }

    // 序列化id
    private static final long serialVersionUID = -2479143000061671589L;
}
```

源码比较简单，基本都是调用map相应的方法。

**总结**

1. TreeSet底层使用NavigableMap存储元素；
2. TreeSet是有序的；
3. TreeSet是非线程安全的；
4. TreeSet实现了NavigableSet接口，而NavigableSet继承自SortedSet接口；
5. TreeSet实现了SortedSet接口；

**问题**

1. TreeSet真的是使用TreeMap来存储元素的吗？

   通过源码分析我们知道TreeSet里面实际上是使用的NavigableMap来存储元素，虽然大部分时候这个map确实是TreeMap，但不是所有时候都是TreeMap。

   因为有一个构造方法是`TreeSet(NavigableMap m)`，而且这是一个非public方法，通过调用关系我们可以发现这个构造方法都是在自己类中使用的，比如下面这个：

   ```
       public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
           return new TreeSet<>(m.tailMap(fromElement, inclusive));
       }
   ```

   而这个m我们姑且认为它是TreeMap，也就是调用TreeMap的tailMap()方法：

   ```
       public NavigableMap<K,V> tailMap(K fromKey, boolean inclusive) {
           return new AscendingSubMap<>(this,
                                        false, fromKey, inclusive,
                                        true,  null,    true);
       }
   ```

   可以看到，返回的是AscendingSubMap对象，这个类的继承链是怎么样的呢？

   ![1FABdA.png](https://s2.ax1x.com/2020/01/20/1FABdA.png)

   可以看到，这个类并没有继承TreeMap，不过通过源码分析也可以看出来这个类是组合了TreeMap，也算和TreeMap有点关系，只是不是继承关系。

   所以，TreeSet的底层不完全是使用TreeMap来实现的，更准确地说，应该是NavigableMap。

2. TreeSet是有序的吗？

3. TreeSet和LinkedHashSet有何不同？

   LinkedHashSet并没有实现SortedSet接口，它的有序性主要依赖于LinkedHashMap的有序性，所以它的有序性是指按照插入顺序保证的有序性；

   而TreeSet实现了SortedSet接口，它的有序性主要依赖于NavigableMap的有序性，而NavigableMap又继承自SortedMap，这个接口的有序性是指按照key的自然排序保证的有序性，而key的自然排序又有两种实现方式，一种是key实现Comparable接口，一种是构造方法传入Comparator比较器。

### 参考

1. [ LinkedHashMap 源码详细分析（JDK1.8）](http://www.tianxiaobo.com/2018/01/24/LinkedHashMap-源码详细分析（JDK1-8）/)
2. [jdk1.8的Hashtable源码解析](https://blog.csdn.net/qq_29864971/article/details/79326106)
3. [Java中ArrayList与Vector差异](https://www.jianshu.com/p/9a2d67d88506)