---
title: 线程安全集合类分析 CopyOnWriteArrayList、CopyOnWriteArraySet、ConcurrentSkipListSet（二）
date: 2020-1-20 21:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

### CopyOnWriteArrayList

CopyOnWriteArrayList是ArrayList的线程安全版本，内部也是通过数组实现，每次对数组的修改都完全拷贝一份新的数组来修改，修改完了再替换掉老数组，这样保证了只阻塞写操作，不阻塞读操作，实现读写分离。

<!--more-->

![1iTqW6.png](https://s2.ax1x.com/2020/01/20/1iTqW6.png)

CopyOnWriteArrayList实现了List, RandomAccess, Cloneable, java.io.Serializable等接口。

CopyOnWriteArrayList实现了List，提供了基础的添加、删除、遍历等操作。

CopyOnWriteArrayList实现了RandomAccess，提供了随机访问的能力。

CopyOnWriteArrayList实现了Cloneable，可以被克隆。

CopyOnWriteArrayList实现了Serializable，可以被序列化。

#### 属性

```java
/** 用于修改时加锁，使用transient修饰表示不自动序列化。 */
final transient ReentrantLock lock = new ReentrantLock();

/** 真正存储元素的地方，只能通过getArray()/setArray()访问
使用transient修饰表示不自动序列化，使用volatile修饰表示一个线程对这个字段的修改另外一个线程立即可见。 */
private transient volatile Object[] array;
```

#### 构造方法

```java
//创建空数组。
public CopyOnWriteArrayList() {
    // 所有对array的操作都是通过setArray()和getArray()进行
    setArray(new Object[0]);
}

final void setArray(Object[] a) {
    array = a;
}
//如果c是CopyOnWriteArrayList类型，直接把它的数组赋值给当前list的数组，注意这里是浅拷贝，两个集合共用同一个数组。
//如果c不是CopyOnWriteArrayList类型，则进行拷贝把c的元素全部拷贝到当前list的数组中。
public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
//把toCopyIn的元素拷贝给当前list的数组。
    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
```

#### 添加方法

##### add(E e)方法

添加一个元素到末尾。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 将旧数组元素拷贝到新数组中
        // 新数组大小是旧数组大小加1
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 将元素放在最后一位
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

1. 加锁；
2. 获取元素数组；
3. 新建一个数组，大小为原数组长度加1，并把原数组元素拷贝到新数组；
4. 把新添加的元素放到新数组的末尾；
5. 把新数组赋值给当前对象的array属性，覆盖原数组；
6. 解锁；

##### add(int index, E element)方法

添加一个元素在指定索引处。

```java
public void add(int index, E element) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        // 检查是否越界, 可以等于len
        if (index > len || index < 0)
            throw new IndexOutOfBoundsException("Index: "+index+
                                                ", Size: "+len);
        Object[] newElements;
        int numMoved = len - index;
        if (numMoved == 0)
            // 如果插入的位置是最后一位
            // 那么拷贝一个n+1的数组, 其前n个元素与旧数组一致
            newElements = Arrays.copyOf(elements, len + 1);
        else {
            // 如果插入的位置不是最后一位
            // 那么新建一个n+1的数组
            newElements = new Object[len + 1];
            // 拷贝旧数组前index的元素到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index及其之后的元素往后挪一位拷贝到新数组中
            // 这样正好index位置是空出来的
            System.arraycopy(elements, index, newElements, index + 1,
                             numMoved);
        }
        // 将元素放置在index处
        newElements[index] = element;
        setArray(newElements);
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

1. 加锁；
2. 检查索引是否合法，如果不合法抛出IndexOutOfBoundsException异常，注意这里index等于len也是合法的；
3. 如果索引等于数组长度（也就是数组最后一位再加1），那就拷贝一个len+1的数组；
4. 如果索引不等于数组长度，那就新建一个len+1的数组，并按索引位置分成两部分，索引之前（不包含）的部分拷贝到新数组索引之前（不包含）的部分，索引之后（包含）的位置拷贝到新数组索引之后（不包含）的位置，索引所在位置留空；
5. 把索引位置赋值为待添加的元素；
6. 把新数组赋值给当前对象的array属性，覆盖原数组；
7. 解锁；

##### addIfAbsent(E e)方法

添加一个元素如果这个元素不存在于集合中。

```java
public boolean addIfAbsent(E e) {
    // 获取元素数组, 取名为快照
    Object[] snapshot = getArray();
    // 检查如果元素不存在,直接返回false
    // 如果存在再调用addIfAbsent()方法添加元素
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
        addIfAbsent(e, snapshot);
}

private boolean addIfAbsent(E e, Object[] snapshot) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 重新获取旧数组
        Object[] current = getArray();
        int len = current.length;
        // 如果快照与刚获取的数组不一致
        // 说明有修改
        if (snapshot != current) {
            // 重新检查元素是否在刚获取的数组里
            int common = Math.min(snapshot.length, len);
            for (int i = 0; i < common; i++)
                // 到这个方法里面了, 说明元素不在快照里面
                if (current[i] != snapshot[i] && eq(e, current[i]))
                    return false;
            if (indexOf(e, current, common, len) >= 0)
                    return false;
        }
        // 拷贝一份n+1的数组
        Object[] newElements = Arrays.copyOf(current, len + 1);
        // 将元素放在最后一位
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

1. 检查这个元素是否存在于数组快照中；
2. 如果存在直接返回false，如果不存在调用addIfAbsent(E e, Object[] snapshot)处理;
3. 加锁；
4. 如果当前数组不等于传入的快照，说明有修改，检查待添加的元素是否存在于当前数组中，如果存在直接返回false;
5. 拷贝一个新数组，长度等于原数组长度加1，并把原数组元素拷贝到新数组中；
6. 把新元素添加到数组最后一位；
7. 把新数组赋值给当前对象的array属性，覆盖原数组；
8. 解锁；

#### 获取

##### get(int index)

获取指定索引的元素，支持随机访问，时间复杂度为O(1)。

```java
public E get(int index) {
    // 获取元素不需要加锁
    // 直接返回index位置的元素
    // 这里是没有做越界检查的, 因为数组本身会做越界检查
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

1. 获取元素数组；
2. 返回数组指定索引位置的元素；

#### 删除

##### remove(int index)方法

删除指定索引位置的元素。

```java
public E remove(int index) {
    final ReentrantLock lock = this.lock;
    // 加锁
    lock.lock();
    try {
        // 获取旧数组
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            // 如果移除的是最后一位
            // 那么直接拷贝一份n-1的新数组, 最后一位就自动删除了
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            // 如果移除的不是最后一位
            // 那么新建一个n-1的新数组
            Object[] newElements = new Object[len - 1];
            // 将前index的元素拷贝到新数组中
            System.arraycopy(elements, 0, newElements, 0, index);
            // 将index后面(不包含)的元素往前挪一位
            // 这样正好把index位置覆盖掉了, 相当于删除了
            System.arraycopy(elements, index + 1, newElements, index,
                             numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        // 释放锁
        lock.unlock();
    }
}
```

1. 加锁；
2. 获取指定索引位置元素的旧值；
3. 如果移除的是最后一位元素，则把原数组的前len-1个元素拷贝到新数组中，并把新数组赋值给当前对象的数组属性；
4. 如果移除的不是最后一位元素，则新建一个len-1长度的数组，并把原数组除了指定索引位置的元素全部拷贝到新数组中，并把新数组赋值给当前对象的数组属性；
5. 解锁并返回旧值；

#### size()方法

返回数组的长度。

```java
public int size() {
    // 获取元素个数不需要加锁
    // 直接返回数组的长度
    return getArray().length;
}
```

#### 总结

1. CopyOnWriteArrayList使用ReentrantLock重入锁加锁，保证线程安全；
2. CopyOnWriteArrayList的写操作都要先拷贝一份新数组，在新数组中做修改，修改完了再用新数组替换老数组，所以空间复杂度是O(n)，性能比较低下；
3. CopyOnWriteArrayList的读操作支持随机访问，时间复杂度为O(1)；
4. CopyOnWriteArrayList采用读写分离的思想，读操作不加锁，写操作加锁，且写操作占用较大内存空间，所以适用于读多写少的场合；
5. CopyOnWriteArrayList只保证最终一致性，不保证实时一致性；

**为什么CopyOnWriteArrayList没有size属性？**

因为每次修改都是拷贝一份正好可以存储目标个数元素的数组，所以不需要size属性了，数组的长度就是集合的大小，而不像ArrayList数组的长度实际是要大于集合的大小的。

比如，add(E e)操作，先拷贝一份n+1个元素的数组，再把新元素放到新数组的最后一位，这时新数组的长度为len+1了，也就是集合的size了。

### CopyOnWriteArraySet

CopyOnWriteArraySet底层是使用CopyOnWriteArrayList存储元素的，所以它并不是使用Map来存储元素的。

但是，我们知道CopyOnWriteArrayList底层其实是一个数组，它是允许元素重复的，那么用它来实现CopyOnWriteArraySet怎么保证元素不重复呢？

#### 源码分析

Set类的源码一般都比较短，所以我们直接贴源码上来一行一行分析吧。

Set之类的简单源码适合泛读，主要是掌握一些不常见的用法，做到心里有说，坐个车三五分钟可能就看完了。

像ConcurrentHashMap、ConcurrentSkipListMap之类的比较长的我们还是倾向分析主要的方法，适合精读，主要是掌握实现原理以及一些不错的思想，可能需要一两个小时才能看完一整篇文章。

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    // 内部使用CopyOnWriteArrayList存储元素
    private final CopyOnWriteArrayList<E> al;

    // 构造方法
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
    
    // 将集合c中的元素初始化到CopyOnWriteArraySet中
    public CopyOnWriteArraySet(Collection<? extends E> c) {
        if (c.getClass() == CopyOnWriteArraySet.class) {
            // 如果c是CopyOnWriteArraySet类型，说明没有重复元素，
            // 直接调用CopyOnWriteArrayList的构造方法初始化
            @SuppressWarnings("unchecked") CopyOnWriteArraySet<E> cc =
                (CopyOnWriteArraySet<E>)c;
            al = new CopyOnWriteArrayList<E>(cc.al);
        }
        else {
            // 如果c不是CopyOnWriteArraySet类型，说明有重复元素
            // 调用CopyOnWriteArrayList的addAllAbsent()方法初始化
            // 它会把重复元素排除掉
            al = new CopyOnWriteArrayList<E>();
            al.addAllAbsent(c);
        }
    }
    
    // 获取元素个数
    public int size() {
        return al.size();
    }

    // 检查集合是否为空
    public boolean isEmpty() {
        return al.isEmpty();
    }
    
    // 检查是否包含某个元素
    public boolean contains(Object o) {
        return al.contains(o);
    }

    // 集合转数组
    public Object[] toArray() {
        return al.toArray();
    }

    // 集合转数组，这里是可能有bug的，详情见ArrayList中分析
    public <T> T[] toArray(T[] a) {
        return al.toArray(a);
    }
    
    // 清空所有元素
    public void clear() {
        al.clear();
    }
    
    // 删除元素
    public boolean remove(Object o) {
        return al.remove(o);
    }
    
    // 添加元素
    // 这里是调用CopyOnWriteArrayList的addIfAbsent()方法
    // 它会检测元素不存在的时候才添加
    // 还记得这个方法吗？当时有分析过的，建议把CopyOnWriteArrayList拿出来再看看
    public boolean add(E e) {
        return al.addIfAbsent(e);
    }
    
    // 是否包含c中的所有元素
    public boolean containsAll(Collection<?> c) {
        return al.containsAll(c);
    }
    
    // 并集
    public boolean addAll(Collection<? extends E> c) {
        return al.addAllAbsent(c) > 0;
    }

    // 单方向差集
    public boolean removeAll(Collection<?> c) {
        return al.removeAll(c);
    }

    // 交集
    public boolean retainAll(Collection<?> c) {
        return al.retainAll(c);
    }
    
    // 迭代器
    public Iterator<E> iterator() {
        return al.iterator();
    }
    
    // equals()方法
    public boolean equals(Object o) {
        // 如果两者是同一个对象，返回true
        if (o == this)
            return true;
        // 如果o不是Set对象，返回false
        if (!(o instanceof Set))
            return false;
        Set<?> set = (Set<?>)(o);
        Iterator<?> it = set.iterator();

        // 集合元素数组的快照
        Object[] elements = al.getArray();
        int len = elements.length;
        
        // 我觉得这里的设计不太好
        // 首先，Set中的元素本来就是不重复的，所以不需要再用个matched[]数组记录有没有出现过
        // 其次，两个集合的元素个数如果不相等，那肯定不相等了，这个是不是应该作为第一要素先检查
        boolean[] matched = new boolean[len];
        int k = 0;
        // 从o这个集合开始遍历
        outer: while (it.hasNext()) {
            // 如果k>len了，说明o中元素多了
            if (++k > len)
                return false;
            // 取值
            Object x = it.next();
            // 遍历检查是否在当前集合中
            for (int i = 0; i < len; ++i) {
                if (!matched[i] && eq(x, elements[i])) {
                    matched[i] = true;
                    continue outer;
                }
            }
            // 如果不在当前集合中，返回false
            return false;
        }
        return k == len;
    }

    // 移除满足过滤条件的元素
    public boolean removeIf(Predicate<? super E> filter) {
        return al.removeIf(filter);
    }

    // 遍历元素
    public void forEach(Consumer<? super E> action) {
        al.forEach(action);
    }

    // 分割的迭代器
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator
            (al.getArray(), Spliterator.IMMUTABLE | Spliterator.DISTINCT);
    }
    
    // 比较两个元素是否相等
    private static boolean eq(Object o1, Object o2) {
        return (o1 == null) ? o2 == null : o1.equals(o2);
    }
}
```

可以看到，在添加元素时调用了CopyOnWriteArrayList的addIfAbsent()方法来保证元素不重复。

#### 总结

1. CopyOnWriteArraySet是用CopyOnWriteArrayList实现的；
2. CopyOnWriteArraySet是有序的，因为底层其实是数组，数组是不是有序的？！
3. CopyOnWriteArraySet是并发安全的，而且实现了读写分离；
4. CopyOnWriteArraySet通过调用CopyOnWriteArrayList的addIfAbsent()方法来保证元素不重复；

#### 问题

1. CopyOnWriteArraySet是用Map实现的吗？
2. CopyOnWriteArraySet是有序的吗？
3. CopyOnWriteArraySet是并发安全的吗？
4. CopyOnWriteArraySet以何种方式保证元素不重复？
5. 如何比较两个Set中的元素是否完全一致？

### ConcurrentSkipListSet

ConcurrentSkipListSet底层是通过ConcurrentNavigableMap来实现的，它是一个有序的线程安全的集合。

#### 源码分析

它的源码比较简单，跟通过Map实现的Set基本是一致，只是多了一些取最近的元素的方法。

为了保持专栏的完整性，我还是贴一下源码，最后会对Set的整个家族作一个对比，有兴趣的可以直接拉到最下面。

```java
// 实现了NavigableSet接口，并没有所谓的ConcurrentNavigableSet接口
public class ConcurrentSkipListSet<E>
    extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2479143111061671589L;

    // 存储使用的map
    private final ConcurrentNavigableMap<E,Object> m;

    // 初始化
    public ConcurrentSkipListSet() {
        m = new ConcurrentSkipListMap<E,Object>();
    }

    // 传入比较器
    public ConcurrentSkipListSet(Comparator<? super E> comparator) {
        m = new ConcurrentSkipListMap<E,Object>(comparator);
    }
    
    // 使用ConcurrentSkipListMap初始化map
    // 并将集合c中所有元素放入到map中
    public ConcurrentSkipListSet(Collection<? extends E> c) {
        m = new ConcurrentSkipListMap<E,Object>();
        addAll(c);
    }
    
    // 使用ConcurrentSkipListMap初始化map
    // 并将有序Set中所有元素放入到map中
    public ConcurrentSkipListSet(SortedSet<E> s) {
        m = new ConcurrentSkipListMap<E,Object>(s.comparator());
        addAll(s);
    }
    
    // ConcurrentSkipListSet类内部返回子set时使用的
    ConcurrentSkipListSet(ConcurrentNavigableMap<E,Object> m) {
        this.m = m;
    }
    
    // 克隆方法
    public ConcurrentSkipListSet<E> clone() {
        try {
            @SuppressWarnings("unchecked")
            ConcurrentSkipListSet<E> clone =
                (ConcurrentSkipListSet<E>) super.clone();
            clone.setMap(new ConcurrentSkipListMap<E,Object>(m));
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }

    /* ---------------- Set operations -------------- */
    // 返回元素个数
    public int size() {
        return m.size();
    }

    // 检查是否为空
    public boolean isEmpty() {
        return m.isEmpty();
    }
    
    // 检查是否包含某个元素
    public boolean contains(Object o) {
        return m.containsKey(o);
    }
    
    // 添加一个元素
    // 调用map的putIfAbsent()方法
    public boolean add(E e) {
        return m.putIfAbsent(e, Boolean.TRUE) == null;
    }
    
    // 移除一个元素
    public boolean remove(Object o) {
        return m.remove(o, Boolean.TRUE);
    }

    // 清空所有元素
    public void clear() {
        m.clear();
    }
    
    // 迭代器
    public Iterator<E> iterator() {
        return m.navigableKeySet().iterator();
    }

    // 降序迭代器
    public Iterator<E> descendingIterator() {
        return m.descendingKeySet().iterator();
    }


    /* ---------------- AbstractSet Overrides -------------- */
    // 比较相等方法
    public boolean equals(Object o) {
        // Override AbstractSet version to avoid calling size()
        if (o == this)
            return true;
        if (!(o instanceof Set))
            return false;
        Collection<?> c = (Collection<?>) o;
        try {
            // 这里是通过两次两层for循环来比较
            // 这里是有很大优化空间的，参考上篇文章CopyOnWriteArraySet中的彩蛋
            return containsAll(c) && c.containsAll(this);
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }
    }
    
    // 移除集合c中所有元素
    public boolean removeAll(Collection<?> c) {
        // Override AbstractSet version to avoid unnecessary call to size()
        boolean modified = false;
        for (Object e : c)
            if (remove(e))
                modified = true;
        return modified;
    }

    /* ---------------- Relational operations -------------- */
    
    // 小于e的最大元素
    public E lower(E e) {
        return m.lowerKey(e);
    }

    // 小于等于e的最大元素
    public E floor(E e) {
        return m.floorKey(e);
    }
    
    // 大于等于e的最小元素
    public E ceiling(E e) {
        return m.ceilingKey(e);
    }

    // 大于e的最小元素
    public E higher(E e) {
        return m.higherKey(e);
    }

    // 弹出最小的元素
    public E pollFirst() {
        Map.Entry<E,Object> e = m.pollFirstEntry();
        return (e == null) ? null : e.getKey();
    }

    // 弹出最大的元素
    public E pollLast() {
        Map.Entry<E,Object> e = m.pollLastEntry();
        return (e == null) ? null : e.getKey();
    }


    /* ---------------- SortedSet operations -------------- */

    // 取比较器
    public Comparator<? super E> comparator() {
        return m.comparator();
    }

    // 最小的元素
    public E first() {
        return m.firstKey();
    }

    // 最大的元素
    public E last() {
        return m.lastKey();
    }
    
    // 取两个元素之间的子set
    public NavigableSet<E> subSet(E fromElement,
                                  boolean fromInclusive,
                                  E toElement,
                                  boolean toInclusive) {
        return new ConcurrentSkipListSet<E>
            (m.subMap(fromElement, fromInclusive,
                      toElement,   toInclusive));
    }
    
    // 取头子set
    public NavigableSet<E> headSet(E toElement, boolean inclusive) {
        return new ConcurrentSkipListSet<E>(m.headMap(toElement, inclusive));
    }

    // 取尾子set
    public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
        return new ConcurrentSkipListSet<E>(m.tailMap(fromElement, inclusive));
    }

    // 取子set，包含from，不包含to
    public NavigableSet<E> subSet(E fromElement, E toElement) {
        return subSet(fromElement, true, toElement, false);
    }
    
    // 取头子set，不包含to
    public NavigableSet<E> headSet(E toElement) {
        return headSet(toElement, false);
    }
    
    // 取尾子set，包含from
    public NavigableSet<E> tailSet(E fromElement) {
        return tailSet(fromElement, true);
    }
    
    // 降序set
    public NavigableSet<E> descendingSet() {
        return new ConcurrentSkipListSet<E>(m.descendingMap());
    }

    // 可分割的迭代器
    @SuppressWarnings("unchecked")
    public Spliterator<E> spliterator() {
        if (m instanceof ConcurrentSkipListMap)
            return ((ConcurrentSkipListMap<E,?>)m).keySpliterator();
        else
            return (Spliterator<E>)((ConcurrentSkipListMap.SubMap<E,?>)m).keyIterator();
    }

    // 原子更新map，给clone方法使用
    private void setMap(ConcurrentNavigableMap<E,Object> map) {
        UNSAFE.putObjectVolatile(this, mapOffset, map);
    }

    // 原子操作相关内容
    private static final sun.misc.Unsafe UNSAFE;
    private static final long mapOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ConcurrentSkipListSet.class;
            mapOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("m"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

可以看到，ConcurrentSkipListSet基本上都是使用ConcurrentSkipListMap实现的，虽然取子set部分是使用ConcurrentSkipListMap中的内部类，但是这些内部类其实也是和ConcurrentSkipListMap相关的，它们返回ConcurrentSkipListMap的一部分数据。

另外，这里的equals()方法实现的相当敷衍，有很大的优化空间，作者这样实现，应该也是知道几乎没有人来调用equals()方法吧。

#### 总结

1. ConcurrentSkipListSet底层是使用ConcurrentNavigableMap实现的；
2. ConcurrentSkipListSet有序的，基于元素的自然排序或者通过比较器确定的顺序；
3. ConcurrentSkipListSet是线程安全的；

Set大汇总：

| Set                   | 有序性 | 线程安全 | 底层实现               | 关键接口     | 特点               |
| --------------------- | ------ | -------- | ---------------------- | ------------ | ------------------ |
| HashSet               | 无     | 否       | HashMap                | 无           | 简单               |
| LinkedHashSet         | 有     | 否       | LinkedHashMap          | 无           | 插入顺序           |
| TreeSet               | 有     | 否       | NavigableMap           | NavigableSet | 自然顺序           |
| CopyOnWriteArraySet   | 有     | 是       | CopyOnWriteArrayList   | 无           | 插入顺序，读写分离 |
| ConcurrentSkipListSet | 有     | 是       | ConcurrentNavigableMap | NavigableSet | 自然顺序           |

从中我们可以发现一些规律：

1. 除了HashSet其它Set都是有序的；
2. 实现了NavigableSet或者SortedSet接口的都是自然顺序的；
3. 使用并发安全的集合实现的Set也是并发安全的；
4. TreeSet虽然不是全部都是使用的TreeMap实现的，但其实都是跟TreeMap相关的（TreeMap的子Map中组合了TreeMap）
5. ConcurrentSkipListSet虽然不是全部都是使用的ConcurrentSkipListMap实现的，但其实都是跟ConcurrentSkipListMap相关的（ConcurrentSkipListeMap的子Map中组合了ConcurrentSkipListMap）

#### 问题

1. ConcurrentSkipListSet的底层是ConcurrentSkipListMap吗？
2. ConcurrentSkipListSet是线程安全的吗？
3. ConcurrentSkipListSet是有序的吗？
4. ConcurrentSkipListSet和之前讲的Set有何不同？