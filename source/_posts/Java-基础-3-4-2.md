---
title: Map其他容器类--从源码理解原理（基于JDK 1.8）
date: 2020-1-18 18:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

上篇文章分析了HashMap，这篇文章将会分析其他的Map容器类。

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

### LinkedHashMap

LinkedHashMap 继承自 HashMap，在 HashMap 基础上，通过维护一条双向链表，解决了 HashMap 不能随时保持遍历顺序和插入顺序一致的问题。除此之外，LinkedHashMap 对访问顺序也提供了相关支持。在一些场景下，该特性很有用，比如缓存。在实现上，LinkedHashMap 很多方法直接继承自 HashMap，仅为维护双向链表覆写了部分方法。所以，要看懂 LinkedHashMap 的源码，需要先看懂 HashMap 的源码。

 LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构。该结构由数组和链表或红黑树组成，结构示意图大致如下：

![1pwB01.png](https://s2.ax1x.com/2020/01/18/1pwB01.png)

LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。其结构可能如下图：

![1pwytK.png](https://s2.ax1x.com/2020/01/18/1pwytK.png)

上图中，淡蓝色的箭头表示前驱引用，红色箭头表示后继引用。每当有新键值对节点插入，新节点最终会接在 tail 引用指向的节点后面。而 tail 引用则会移动到新的节点上，这样一个双向链表就建立起来了。

上面的结构并不是很难理解，虽然引入了红黑树，导致结构看起来略为复杂了一些。但大家完全可以忽略红黑树，而只关注链表结构本身。好了，接下来进入细节分析吧。

#### 源码分析

##### 节点结构

![1p0i3F.png](https://s2.ax1x.com/2020/01/18/1p0i3F.png)

LinkedHashMap 内部类 Entry 继承自 HashMap 内部类 Node，并新增了两个引用，分别是 before 和 after。这两个引用的用途不难理解，也就是用于维护双向链表。同时，TreeNode 继承 LinkedHashMap 的内部类 Entry 后，就具备了和其他 Entry 一起组成链表的能力。但是这里需要大家考虑一个问题。当我们使用 HashMap 时，TreeNode 并不需要具备组成链表能力。如果继承 LinkedHashMap 内部类 Entry ，TreeNode 就多了两个用不到的引用，这样做不是会浪费空间吗？简单说明一下这个问题（水平有限，不保证完全正确），这里这么做确实会浪费空间，但与 TreeNode 通过继承获取的组成链表的能力相比，这点浪费是值得的。在 HashMap 的设计思路注释中，有这样一段话：

> Because TreeNodes are about twice the size of regular nodes, we
> use them only when bins contain enough nodes to warrant use
> (see TREEIFY_THRESHOLD). And when they become too small (due to
> removal or resizing) they are converted back to plain bins. In
> usages with well-distributed user hashCodes, tree bins are
> rarely used.

大致的意思是 TreeNode 对象的大小约是普通 Node 对象的2倍，我们仅在桶（bin）中包含足够多的节点时再使用。当桶中的节点数量变少时（取决于删除和扩容），TreeNode 会被转成 Node。当用户实现的 hashCode 方法具有良好分布性时，树类型的桶将会很少被使用。

通过上面的注释，我们可以了解到。一般情况下，只要 hashCode 的实现不糟糕，Node 组成的链表很少会被转成由 TreeNode 组成的红黑树。也就是说 TreeNode 使用的并不多，浪费那点空间是可接受的。假如 TreeNode 机制继承自 Node 类，那么它要想具备组成链表的能力，就需要 Node 去继承 LinkedHashMap 的内部类 Entry。这个时候就得不偿失了，浪费很多空间去获取不一定用得到的能力。

```java
    /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * The iteration ordering method for this linked hash map: <tt>true</tt>
     * for access-order, <tt>false</tt> for insertion-order.
     *
     * @serial
     */
    final boolean accessOrder;
```

##### 链表的建立过程

链表的建立过程是在插入键值对节点时开始的，初始情况下，让 LinkedHashMap 的 head 和 tail 引用同时指向新节点，链表就算建立起来了。随后不断有新节点插入，通过将新节点接在 tail 引用指向节点的后面，即可实现链表的更新。

Map 类型的集合类是通过 put(K,V) 方法插入键值对，LinkedHashMap 本身并没有覆写父类的 put 方法，而是直接使用了父类的实现。但在 HashMap 中，put 方法插入的是 HashMap 内部类 Node 类型的节点，该类型的节点并不具备与 LinkedHashMap 内部类 Entry 及其子类型节点组成链表的能力。那么，LinkedHashMap 是怎样建立链表的呢？在展开说明之前，我们先看一下 LinkedHashMap 插入操作相关的代码：

```java
// HashMap 中实现
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// HashMap 中实现
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) {...}
    // 通过节点 hash 定位节点所在的桶位置，并检测桶中是否包含节点引用
    if ((p = tab[i = (n - 1) & hash]) == null) {...}
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) {...}
        else {
            // 遍历链表，并统计链表长度
            for (int binCount = 0; ; ++binCount) {
                // 未在单链表中找到要插入的节点，将新节点接在单链表的后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) {...}
                    break;
                }
                // 插入的节点已经存在于单链表中
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null) {...}
            afterNodeAccess(e);    // 回调方法，后续说明
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold) {...}
    afterNodeInsertion(evict);    // 回调方法
    return null;
}

// HashMap 中实现
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}

// LinkedHashMap 中覆写
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    // 将 Entry 接在双向链表的尾部
    linkNodeLast(p);
    return p;
}

// LinkedHashMap 中实现
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    // last 为 null，表明链表还未建立
    if (last == null)
        head = p;
    else {
        // 将新节点 p 接在链表尾部
        p.before = last;
        last.after = p;
    }
}
// LinkedHashMap 中实现
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 根据条件判断是否移除最近最少被访问的节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
// LinkedHashMap 中实现
// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

上面就是 LinkedHashMap 插入相关的源码，这里省略了部分非关键的代码。我根据上面的代码，可以知道 LinkedHashMap 插入操作的调用过程。如下：

![1p0M9O.png](https://s2.ax1x.com/2020/01/18/1p0M9O.png)

我把 newNode 方法红色背景标注了出来，这一步比较关键。LinkedHashMap 覆写了该方法。在这个方法中，LinkedHashMap 创建了 Entry，并通过 linkNodeLast 方法将 Entry 接在双向链表的尾部，实现了双向链表的建立。双向链表建立之后，我们就可以按照插入顺序去遍历 LinkedHashMap，大家可以自己写点测试代码验证一下插入顺序。

大家如果仔细看上面的代码的话，会发现有两个以`after`开头方法，在上文中没有被提及。在 JDK 1.8 HashMap 的源码中，相关的方法有3个：

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

根据这三个方法的注释可以看出，这些方法的用途是在增删查等操作后，通过回调的方式，让 LinkedHashMap 有机会做一些后置操作。上述三个方法的具体实现在 LinkedHashMap 中，本节先不分析这些实现

##### 链表节点的删除过程

与插入操作一样，LinkedHashMap 删除操作相关的代码也是直接用父类的实现。在删除节点时，父类的删除逻辑并不会修复 LinkedHashMap 所维护的双向链表，这不是它的职责。那么删除及节点后，被删除的节点该如何从双链表中移除呢？当然，办法还算是有的。上一节最后提到 HashMap 中三个回调方法运行 LinkedHashMap 对一些操作做出响应。所以，在删除及节点后，回调方法 `afterNodeRemoval` 会被调用。LinkedHashMap 覆写该方法，并在该方法中完成了移除被删除节点的操作。相关源码如下：

```java
// HashMap 中实现
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

// HashMap 中实现
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode) {...}
            else {
                // 遍历单链表，寻找要删除的节点，并赋值给 node 变量
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode) {...}
            // 将要删除的节点从单链表中移除
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);    // 调用删除回调方法进行后续操作
            return node;
        }
    }
    return null;
}

// LinkedHashMap 中覆写
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    // 将 p 节点的前驱后后继引用置空
    p.before = p.after = null;
    // b 为 null，表明 p 是头节点
    if (b == null)
        head = a;
    else
        b.after = a;
    // a 为 null，表明 p 是尾节点
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

删除的过程并不复杂，上面这么多代码其实就做了三件事：

1. 根据 hash 定位到桶位置
2. 遍历链表或调用红黑树相关的删除方法
3. 从 LinkedHashMap 维护的双链表中移除要删除的节点

举个例子说明一下，假如我们要删除下图键值为 3 的节点。

![1p0BvQ.png](https://s2.ax1x.com/2020/01/18/1p0BvQ.png)

根据 hash 定位到该节点属于3号桶，然后在对3号桶保存的单链表进行遍历。找到要删除的节点后，先从单链表中移除该节点。如下：

![1p0cEq.png](https://s2.ax1x.com/2020/01/18/1p0cEq.png)

然后再双向链表中移除该节点：

![1p0HV1.png](https://s2.ax1x.com/2020/01/18/1p0HV1.png)

删除及相关修复过程并不复杂，结合上面的图片，大家应该很容易就能理解，这里就不多说了。

##### 访问顺序的维护过程

前面说了插入顺序的实现，本节来讲讲访问顺序。默认情况下，LinkedHashMap 是按插入顺序维护链表。不过我们可以在初始化 LinkedHashMap，指定 accessOrder 参数为 true，即可让它按访问顺序维护链表。访问顺序的原理上并不复杂，当我们调用`get/getOrDefault/replace`等方法时，只需要将这些方法访问的节点移动到链表的尾部即可。相应的源码如下：

```java
// LinkedHashMap 中覆写
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    // 如果 accessOrder 为 true，则调用 afterNodeAccess 将被访问节点移动到链表最后
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

// LinkedHashMap 中覆写
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        // 如果 b 为 null，表明 p 为头节点
        if (b == null)
            head = a;
        else
            b.after = a;
            
        if (a != null)
            a.before = b;
        /*
         * 这里存疑，父条件分支已经确保节点 e 不会是尾节点，
         * 那么 e.after 必然不会为 null，不知道 else 分支有什么作用
         */
        else
            last = b;
    
        if (last == null)
            head = p;
        else {
            // 将 p 接在链表的最后
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

上面就是访问顺序的实现代码，并不复杂。下面举例演示一下，帮助大家理解。假设我们访问下图键值为3的节点，访问前结构为：

![1pBpqA.png](https://s2.ax1x.com/2020/01/18/1pBpqA.png)

访问后，键值为3的节点将会被移动到双向链表的最后位置，其前驱和后继也会跟着更新。访问后的结构如下：

![1pBPat.png](https://s2.ax1x.com/2020/01/18/1pBPat.png)

##### 基于 LinkedHashMap 实现缓存

前面介绍了 LinkedHashMap 是如何维护插入和访问顺序的，大家对 LinkedHashMap 的原理应该有了一定的认识。本节我们来写一些代码实践一下，这里通过继承 LinkedHashMap 实现了一个简单的 LRU 策略的缓存

```java
// LinkedHashMap 中实现
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 根据条件判断是否移除最近最少被访问的节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
// LinkedHashMap 中实现
// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

当我们基于 LinkedHashMap 实现缓存时，通过覆写`removeEldestEntry`方法可以实现自定义策略的 LRU 缓存。比如我们可以根据节点数量判断是否移除最近最少被访问的节点，或者根据节点的存活时间判断是否移除该节点等。本节所实现的缓存是基于判断节点数量是否超限的策略。在构造缓存对象时，传入最大节点数。当插入的节点数超过最大节点数时，移除最近最少被访问的节点。实现代码如下：

```java
public class SimpleCache<K, V> extends LinkedHashMap<K, V> {

    private static final int MAX_NODE_NUM = 100;

    private int limit;

    public SimpleCache() {
        this(MAX_NODE_NUM);
    }

    public SimpleCache(int limit) {
        super(limit, 0.75f, true);
        this.limit = limit;
    }

    public V save(K key, V val) {
        return put(key, val);
    }

    public V getOne(K key) {
        return get(key);
    }

    public boolean exists(K key) {
        return containsKey(key);
    }

    /**
     * 判断节点数是否超限
     * @param eldest
     * @return 超限返回 true，否则返回 false
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > limit;
    }

    public static void main(String[] args) {
        SimpleCache<Integer, Integer> cache = new SimpleCache<>(3);

        for (int i = 0; i < 10; i++) {
            cache.save(i, i * i);
        }

        System.out.println("插入10个键值对后，缓存内容：");
        System.out.println(cache + "\n");

        System.out.println("访问键值为7的节点后，缓存内容：");
        cache.getOne(7);
        System.out.println(cache + "\n");

        System.out.println("插入键值为1的键值对后，缓存内容：");
        cache.save(1, 1);
        System.out.println(cache);
    }
//    插入10个键值对后，缓存内容：
//    {7=49, 8=64, 9=81}
//
//    访问键值为7的节点后，缓存内容：
//    {8=64, 9=81, 7=49}
//
//    插入键值为1的键值对后，缓存内容：
//    {9=81, 7=49, 1=1}
}

```

在测试代码中，设定缓存大小为3。在向缓存中插入10个键值对后，只有最后3个被保存下来了，其他的都被移除了。然后通过访问键值为7的节点，使得该节点被移到双向链表的最后位置。当我们再次插入一个键值对时，键值为7的节点就不会被移除。

##### 遍历

前面讲到，HashMap的遍历时通过遍历table数组和node节点的next进行遍历，对于红黑树TreeNode来说，也是通过维护next、prev来进行遍历，根节点也就会是链表的首节点

LinkedHashMap因为实现了双向链表，因此遍历时使用该链表进行遍历。

```java
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new LinkedKeySet();
            keySet = ks;
        }
        return ks;
    }

    final class LinkedKeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { LinkedHashMap.this.clear(); }
        public final Iterator<K> iterator() {
            return new LinkedKeyIterator();
        }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator()  {
            return Spliterators.spliterator(this, Spliterator.SIZED |
                                            Spliterator.ORDERED |
                                            Spliterator.DISTINCT);
        }
        public final void forEach(Consumer<? super K> action) {
            if (action == null)
                throw new NullPointerException();
            int mc = modCount;
            for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
                action.accept(e.key);
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
    // Iterators

    abstract class LinkedHashIterator {
        LinkedHashMap.Entry<K,V> next;
        LinkedHashMap.Entry<K,V> current;
        int expectedModCount;

        LinkedHashIterator() {
            next = head;
            expectedModCount = modCount;
            current = null;
        }

        public final boolean hasNext() {
            return next != null;
        }

        final LinkedHashMap.Entry<K,V> nextNode() {
            LinkedHashMap.Entry<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            current = e;
            next = e.after;
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }

    final class LinkedKeyIterator extends LinkedHashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().getKey(); }
    }
```

#### 总结

1. LinkedHashMap继承自HashMap，具有HashMap的所有特性；
2. LinkedHashMap内部维护了一个双向链表存储所有的元素；
3. 如果accessOrder为false，则可以按插入元素的顺序遍历元素；
4. 如果accessOrder为true，则可以按访问元素的顺序遍历元素；
5. LinkedHashMap的实现非常精妙，很多方法都是在HashMap中留的钩子（Hook），直接实现这些Hook就可以实现对应的功能了，并不需要再重写put()等方法；
6. 默认的LinkedHashMap并不会移除旧元素，如果需要移除旧元素，则需要重写removeEldestEntry()方法设定移除策略；
7. LinkedHashMap可以用来实现LRU缓存淘汰策略；

### HashTable

和HashMap一样，Hashtable 也是一个散列表，它存储的内容是键值对(key-value)映射。Hashtable 继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口。

Hashtable 的函数都是同步的，这意味着它是线程安全的。它的key、value都不可以为null。

Hashtable中的映射不是有序的。

数组+链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    /**
    table是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。 
    Entry结构跟HashMap.Node相同
     * The hash table data.
     */
    private transient Entry<?,?>[] table;

    /**
    count是Hashtable的大小，它是Hashtable保存的键值对的数量。 
     * The total number of entries in the hash table.
     */
    private transient int count;

    /**
     * The table is rehashed when its size exceeds this threshold.  (The
     * value of this field is (int)(capacity * loadFactor).)
     *threshold是Hashtable的阈值，用于判断是否需要调整Hashtable的容量。threshold的值="容量*加载因子"。
     * @serial
     */
    private int threshold;

    /**
    loadFactor就是加载因子。 
     * The load factor for the hashtable.
     *
     * @serial
     */
    private float loadFactor;

    /**
    modCount是用来实现fail-fast机制的
     * The number of times this Hashtable has been structurally modified
     * Structural modifications are those that change the number of entries in
     * the Hashtable or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the Hashtable fail-fast.  (See ConcurrentModificationException).
     */
    private transient int modCount = 0;
    /**
    散列表容量经过n次扩容之后，设置上限的阈值
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

}
```

#### 构造函数

```java
	public Hashtable(int initialCapacity, float loadFactor) {//可指定初始容量和加载因子  
        if (initialCapacity < 0)  
            throw new IllegalArgumentException("Illegal Capacity: "+  
                                               initialCapacity);  
        if (loadFactor <= 0 || Float.isNaN(loadFactor))  
            throw new IllegalArgumentException("Illegal Load: "+loadFactor);  
        if (initialCapacity==0)  
            initialCapacity = 1;//初始容量最小值为1  
        this.loadFactor = loadFactor;  
        table = new Entry[initialCapacity];//创建桶数组  
        threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);//初始化容量阈值  
        useAltHashing = sun.misc.VM.isBooted() &&  
                (initialCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);  
    }  
    /** 
     * Constructs a new, empty hashtable with the specified initial capacity 
     * and default load factor (0.75). 
     */  
    public Hashtable(int initialCapacity) {  
        this(initialCapacity, 0.75f);//默认负载因子为0.75  
    }  
    public Hashtable() {  
        this(11, 0.75f);//默认容量为11，负载因子为0.75  
    }  
    /** 
     * Constructs a new hashtable with the same mappings as the given 
     * Map.  The hashtable is created with an initial capacity sufficient to 
     * hold the mappings in the given Map and a default load factor (0.75). 
     */  
    public Hashtable(Map<? extends K, ? extends V> t) {  
        this(Math.max(2*t.size(), 11), 0.75f);  
        putAll(t);  
    }  
```

需注意的点：

1. Hashtable的默认容量为11，默认负载因子为0.75.(HashMap默认容量为16，默认负载因子也是0.75)
2. Hashtable的容量可以为任意整数，最小值为1，而HashMap的容量始终为2的n次方。
3. 为避免扩容带来的性能问题，建议指定合理容量。
4. 跟HashMap一样，Hashtable内部也有一个静态类叫Entry，其实是个键值对对象，保存了键和值的引用。
5. HashMap和Hashtable存储的是键值对对象，而不是单独的键或值。

#### put

```java
	public synchronized V put(K key, V value) {//向哈希表中添加键值对  
        // Make sure the value is not null  
        if (value == null) {//确保值不能为空  
            throw new NullPointerException();  
        }  
        // Makes sure the key is not already in the hashtable.  
        Entry tab[] = table;  
        int hash = hash(key);//根据键生成hash值---->若key为null，此方法会抛异常  
        int index = (hash & 0x7FFFFFFF) % tab.length;//通过hash值找到其存储位置  
        for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {//遍历链表  
            if ((e.hash == hash) && e.key.equals(key)) {//若键相同，则新值覆盖旧值  
                V old = e.value;  
                e.value = value;  
                return old;  
            }  
        }  
        addEntry(hash, key, value, index);
        return null;  
    }
    private void addEntry(int hash, K key, V value, int index) {
        modCount++;
        if (count >= threshold) {//当前容量超过阈值。需要扩容  
            // Rehash the table if the threshold is exceeded  
            rehash();//重新构建桶数组，并对数组中所有键值对重哈希，耗时！  
            tab = table;  
            hash = hash(key);  
            index = (hash & 0x7FFFFFFF) % tab.length;//这里是取摸运算  
        }  
        // Creates the new entry.  
        Entry<K,V> e = tab[index];  
        //将新结点插到链表首部  
        tab[index] = new Entry<>(hash, key, value, e);//生成一个新结点  
        count++;  
    }
```

1. Hasbtable并不允许值和键为空（null），若为空，会抛空指针。
2. HashMap计算索引的方式是h&(length-1),而Hashtable用的是模运算，效率上是低于HashMap的。
3. 另外Hashtable计算索引时将hash值先与上0x7FFFFFFF,这是为了保证hash值始终为正数。
4. 特别需要注意的是这个方法包括下面要讲的若干方法都加了synchronized关键字，也就意味着这个Hashtable是个线程安全的类，这也是它和HashMap最大的不同点.

#### rehash

```java
	protected void rehash() {  
        int oldCapacity = table.length;//记录旧容量  
        Entry<K,V>[] oldMap = table;//记录旧的桶数组  
        // overflow-conscious code  
        int newCapacity = (oldCapacity << 1) + 1;//新容量为老容量的2倍加1  
        if (newCapacity - MAX_ARRAY_SIZE > 0) {  
            if (oldCapacity == MAX_ARRAY_SIZE)//容量不得超过约定的最大值  
                // Keep running with MAX_ARRAY_SIZE buckets  
                return;  
            newCapacity = MAX_ARRAY_SIZE;  
        }  
        Entry<K,V>[] newMap = new Entry[newCapacity];//创建新的数组  
        modCount++;  
        threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);  
        boolean currentAltHashing = useAltHashing;  
        useAltHashing = sun.misc.VM.isBooted() &&  
                (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);  
        boolean rehash = currentAltHashing ^ useAltHashing;  
        table = newMap;  
        for (int i = oldCapacity ; i-- > 0 ;) {//转移键值对到新数组  
            for (Entry<K,V> old = oldMap[i] ; old != null ; ) {  
                Entry<K,V> e = old;  
                old = old.next;  
                if (rehash) {  
                    e.hash = hash(e.key);  
                }  
                int index = (e.hash & 0x7FFFFFFF) % newCapacity;  
                e.next = newMap[index];  
                newMap[index] = e;  
            }  
        }  
    }  	
```

Hashtable每次扩容，容量都为原来的2倍加1，而HashMap为原来的2倍。

#### get

```java
  public synchronized V get(Object key) {//根据键取出对应索引  
      Entry tab[] = table;  
      int hash = hash(key);//先根据key计算hash值  
      int index = (hash & 0x7FFFFFFF) % tab.length;//再根据hash值找到索引  
      for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {//遍历entry链  
          if ((e.hash == hash) && e.key.equals(key)) {//若找到该键  
              return e.value;//返回对应的值  
          }  
      }  
      return null;//否则返回null  
	}
```

当然，如果你传的参数为null，是会抛空指针的。

### TreeMap

`TreeMap`最早出现在`JDK 1.2`中，是 Java 集合框架中比较重要一个的实现。TreeMap 底层基于`红黑树`实现，可保证在`log(n)`时间复杂度内完成 containsKey、get、put 和 remove 操作，效率很高。另一方面，由于 TreeMap 基于红黑树实现，这为 TreeMap 保持键的有序性打下了基础。总的来说，TreeMap 的核心是红黑树，其很多方法也是对红黑树增删查基础操作的一个包装。所以只要弄懂了红黑树，TreeMap 就没什么秘密了。

`TreeMap`继承自`AbstractMap`，并实现了 `NavigableMap`接口。NavigableMap 接口继承了`SortedMap`接口，SortedMap 最终继承自`Map`接口，同时 AbstractMap 类也实现了 Map 接口。以上就是 TreeMap 的继承体系，描述起来有点乱，不如看图了：

![1pDPOJ.png](https://s2.ax1x.com/2020/01/18/1pDPOJ.png)

上图就是 TreeMap 的继承体系图，比较直观。这里来简单说一下继承体系中不常见的接口`NavigableMap`和`SortedMap`，这两个接口见名知意。先说 NavigableMap 接口，NavigableMap 接口声明了一些列具有导航功能的方法，比如：

```java
/**
 * 返回红黑树中最小键所对应的 Entry
 */
Map.Entry<K,V> firstEntry();

/**
 * 返回最大的键 maxKey，且 maxKey 仅小于参数 key
 */
K lowerKey(K key);

/**
 * 返回最小的键 minKey，且 minKey 仅大于参数 key
 */
K higherKey(K key);

// 其他略
```

通过这些导航方法，我们可以快速定位到目标的 key 或 Entry。至于 SortedMap 接口，这个接口提供了一些基于有序键的操作，比如

```java
/**
 * 返回包含键值在 [minKey, toKey) 范围内的 Map
 */
SortedMap<K,V> headMap(K toKey);();

/**
 * 返回包含键值在 [fromKey, toKey) 范围内的 Map
 */
SortedMap<K,V> subMap(K fromKey, K toKey);

// 其他略
```

以上就是两个接口的介绍，很简单。至于 AbstractMap 和 Map 这里就不说了，大家有兴趣自己去看看 Javadoc 吧。

#### 源码分析

`JDK 1.8`中的`TreeMap`源码有两千多行，还是比较多的。本文并不打算逐句分析所有的源码，而是挑选几个常用的方法进行分析。这些方法实现的功能分别是查找、遍历、插入、删除等，其他的方法小伙伴们有兴趣可以自己分析。`TreeMap`实现的核心部分是关于`红黑树`的实现，其绝大部分的方法基本都是对底层红黑树增、删、查操作的一个封装。

##### 属性

```java
/**
 * 比较器，如果没传则key要实现Comparable接口
 按key的大小排序有两种方式，一种是key实现Comparable接口，一种方式通过构造方法传入比较器。
 */
private final Comparator<? super K> comparator;

/**
 * 根节点，TreeMap没有桶的概念，所有的元素都存储在一颗树中。
 */
private transient Entry<K,V> root;

/**
 * 元素个数
 */
private transient int size = 0;

/**
 * 修改次数
 */
private transient int modCount = 0;
```

##### Entry内部类

存储节点，典型的红黑树结构。

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;
}
```

##### 查找

TreeMap基于红黑树实现，而红黑树是一种自平衡二叉查找树，所以 TreeMap 的查找操作流程和二叉查找树一致。二叉树的查找流程是这样的，先将目标值和根节点的值进行比较，如果目标值小于根节点的值，则再和根节点的左孩子进行比较。如果目标值大于根节点的值，则继续和根节点的右孩子比较。在查找过程中，如果目标值和二叉树中的某个节点值相等，则返回 true，否则返回 false。TreeMap 查找和此类似，只不过在 TreeMap 中，节点（Entry）存储的是键值对<k,v>。在查找过程中，比较的是键的大小，返回的是值，如果没找到，则返回null。TreeMap 中的查找方法是get，具体实现在getEntry方法中，相关源码如下：

```java
public V get(Object key) {
    // 根据key查找元素
    Entry<K,V> p = getEntry(key);
    // 找到了返回value值，没找到返回null
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // 如果comparator不为空，使用comparator的版本获取元素
    if (comparator != null)
        return getEntryUsingComparator(key);
    // 如果key为空返回空指针异常
    if (key == null)
        throw new NullPointerException();
    // 将key强转为Comparable
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;
    // 从根元素开始遍历
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            // 如果小于0从左子树查找
            p = p.left;
        else if (cmp > 0)
            // 如果大于0从右子树查找
            p = p.right;
        else
            // 如果相等说明找到了直接返回
            return p;
    }
    // 没找到返回null
    return null;
}
    
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
    K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 从根元素开始遍历
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                // 如果小于0从左子树查找
                p = p.left;
            else if (cmp > 0)
                // 如果大于0从右子树查找
                p = p.right;
            else
                // 如果相等说明找到了直接返回
                return p;
        }
    }
    // 没找到返回null
    return null;
}
```

查找操作的核心逻辑就是`getEntry`方法中的`while`循环

1. 从root遍历整个树；
2. 如果待查找的key比当前遍历的key小，则在其左子树中查找；
3. 如果待查找的key比当前遍历的key大，则在其右子树中查找；
4. 如果待查找的key与当前遍历的key相等，则找到了该元素，直接返回；
5. 从这里可以看出是否有comparator分化成了两个方法，但是内部逻辑一模一样，因此可见笔者`comparator = (k1, k2) -> ((Comparable)k1).compareTo(k2);`这种改造的必要性

##### 遍历

遍历操作也是大家使用频率较高的一个操作，对于`TreeMap`，使用方式一般如下：

```java
for(Object key : map.keySet()) {
    // do something
}

for(Map.Entry entry : map.entrySet()) {
    // do something
}
```

从上面代码片段中可以看出，大家一般都是对 TreeMap 的 key 集合或 Entry 集合进行遍历。上面代码片段中用 foreach 遍历keySet 方法产生的集合，在编译时会转换成用迭代器遍历，等价于：

```java
Set keys = map.keySet();
Iterator ite = keys.iterator();
while (ite.hasNext()) {
    Object key = ite.next();
    // do something
}
```

另一方面，TreeMap 有一个特性，即可以保证键的有序性，默认是正序。所以在遍历过程中，大家会发现 TreeMap 会从小到大输出键的值。那么，接下来就来分析一下`keySet`方法，以及在遍历 keySet 方法产生的集合时，TreeMap 是如何保证键的有序性的。相关代码如下：

```java
public Set<K> keySet() {
    return navigableKeySet();
}

public NavigableSet<K> navigableKeySet() {
    KeySet<K> nks = navigableKeySet;
    return (nks != null) ? nks : (navigableKeySet = new KeySet<>(this));
}

static final class KeySet<E> extends AbstractSet<E> implements NavigableSet<E> {
    private final NavigableMap<E, ?> m;
    KeySet(NavigableMap<E,?> map) { m = map; }

    public Iterator<E> iterator() {
        if (m instanceof TreeMap)
            return ((TreeMap<E,?>)m).keyIterator();
        else
            return ((TreeMap.NavigableSubMap<E,?>)m).keyIterator();
    }

    // 省略非关键代码
}

Iterator<K> keyIterator() {
    return new KeyIterator(getFirstEntry());
}

final class KeyIterator extends PrivateEntryIterator<K> {
    KeyIterator(Entry<K,V> first) {
        super(first);
    }
    public K next() {
        return nextEntry().key;
    }
}

abstract class PrivateEntryIterator<T> implements Iterator<T> {
    Entry<K,V> next;
    Entry<K,V> lastReturned;
    int expectedModCount;

    PrivateEntryIterator(Entry<K,V> first) {
        expectedModCount = modCount;
        lastReturned = null;
        next = first;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Entry<K,V> nextEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        // 寻找节点 e 的后继节点
        next = successor(e);
        lastReturned = e;
        return e;
    }

    // 其他方法省略
}
```

上面的代码比较多，keySet 涉及的代码还是比较多的，大家可以从上往下看。从上面源码可以看出 keySet 方法返回的是`KeySet`类的对象。这个类实现了`Iterable`接口，可以返回一个迭代器。该迭代器的具体实现是`KeyIterator`，而 KeyIterator 类的核心逻辑是在`PrivateEntryIterator`中实现的。上面的代码虽多，但核心代码还是 KeySet 类和 PrivateEntryIterator 类的 `nextEntry`方法。KeySet 类就是一个集合，这里不分析了。而 nextEntry 方法比较重要，下面简单分析一下。

在初始化 KeyIterator 时，会将 TreeMap 中包含最小键的 Entry 传给 PrivateEntryIterator。当调用 nextEntry 方法时，通过调用 successor 方法找到当前 entry 的后继，并让 next 指向后继，最后返回当前的 entry。通过这种方式即可实现按正序返回键值的的逻辑。

##### 插入

相对于前两个操作，插入操作明显要复杂一些。当往 TreeMap 中放入新的键值对后，可能会破坏红黑树的性质。这里为了描述方便，把 Entry 称为节点。并把新插入的节点称为`N`，N 的父节点为`P`。P 的父节点为`G`，且 P 是 G 的左孩子。P 的兄弟节点为`U`。在往红黑树中插入新的节点 N 后（新节点为红色），会产生下面5种情况：

1. N 是根节点
2. N 的父节点是黑色
3. N 的父节点是红色，叔叔节点也是红色
4. N 的父节点是红色，叔叔节点是黑色，且 N 是 P 的右孩子
5. N 的父节点是红色，叔叔节点是黑色，且 N 是 P 的左孩子

上面5中情况中，情况2不会破坏红黑树性质，所以无需处理。情况1 会破坏红黑树性质2（根是黑色），情况3、4、和5会破坏红黑树性质4（每个红色节点必须有两个黑色的子节点）。这个时候就需要进行调整，以使红黑树重新恢复平衡。

至于怎么调整，可以参考[红黑树详细分析](http://www.coolblog.xyz/2018/01/11/红黑树详细分析/)

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    // 1.如果根节点为 null，将新节点设为根节点
    if (t == null) {
        compare(key, key);
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        // 2.为 key 在红黑树找到合适的位置
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    } else {
        // 与上面代码逻辑类似，省略
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    // 3.将新节点链入红黑树中
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    // 4.插入新节点可能会破坏红黑树性质，这里修正一下
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

put 方法代码如上，逻辑和二叉查找树插入节点逻辑一致。重要的步骤我已经写了注释，并不难理解。插入逻辑的复杂之处在于插入后的修复操作，对应的方法`fixAfterInsertion`，该方法的源码和说明如下：

![1pHvr9.png](https://s2.ax1x.com/2020/01/18/1pHvr9.png)

##### 删除

删除操作是红黑树最复杂的部分，原因是该操作可能会破坏红黑树性质5（从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点），修复性质5要比修复其他性质（性质2和4需修复，性质1和3不用修复）复杂的多。当删除操作导致性质5被破坏时，会出现8种情况。为了方便表述，这里还是先做一些假设。我们把`最终被删除`的节点称为 X，X 的替换节点称为 N。N 的父节点为`P`，且 N 是 P 的左孩子。N 的兄弟节点为`S`，S 的左孩子为 SL，右孩子为 SR。这里特地强调 X 是 `最终被删除` 的节点，是原因**二叉查找树会把要删除有两个孩子的节点的情况转化为删除只有一个孩子的节点的情况**，该节点是欲被删除节点的前驱和后继。

接下来，简单列举一下删除节点时可能会出现的情况，先列举较为简单的情况：

1. 最终被删除的节点 X 是红色节点
2. X 是黑色节点，但该节点的孩子节点是红色

比较复杂的情况：

1. 替换节点 N 是新的根
2. N 为黑色，N 的兄弟节点 S 为红色，其他节点为黑色。
3. N 为黑色，N 的父节点 P，兄弟节点 S 和 S 的孩子节点均为黑色。
4. N 为黑色，P 是红色，S 和 S 孩子均为黑色。
5. N 为黑色，P 可红可黑，S 为黑色，S 的左孩子 SL 为红色，右孩子 SR 为黑色
6. N 为黑色，P 可红可黑，S 为黑色，SR 为红色，SL 可红可黑

上面列举的8种情况中，前两种处理起来比较简单，后6种情况中情况2~6较为复杂。接下来我将会对情况26展开分析，删除相关的源码如下：

```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}

private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    /* 
     * 1. 如果 p 有两个孩子节点，则找到后继节点，
     * 并把后继节点的值复制到节点 P 中，并让 p 指向其后继节点
     */
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        /*
         * 2. 将 replacement parent 引用指向新的父节点，
         * 同时让新的父节点指向 replacement。
         */ 
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // 3. 如果删除的节点 p 是黑色节点，则需要进行调整
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // 删除的是根节点，且树中当前只有一个节点
        root = null;
    } else { // 删除的节点没有孩子节点
        // p 是黑色，则需要进行调整
        if (p.color == BLACK)
            fixAfterDeletion(p);

        // 将 P 从树中移除
        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

从源码中可以看出，`remove`方法只是一个简单的保证，核心实现在`deleteEntry`方法中。deleteEntry 主要做了这么几件事：

1. 如果待删除节点 P 有两个孩子，则先找到 P 的后继 S，然后将 S 中的值拷贝到 P 中，并让 P 指向 S
2. 如果最终被删除节点 P（P 现在指向最终被删除节点）的孩子不为空，则用其孩子节点替换掉
3. 如果最终被删除的节点是黑色的话，调用 fixAfterDeletion 方法进行修复

上面说了 replacement 不为空时，deleteEntry 的执行逻辑。上面说的略微啰嗦，如果简单说的话，7个字即可总结：`找后继 -> 替换 -> 修复`。这三步中，最复杂的是修复操作。修复操作要重新使红黑树恢复平衡，修复操作的源码分析如下：

fixAfterDeletion 方法分析如下：

![1pbPPK.png](https://s2.ax1x.com/2020/01/18/1pbPPK.png)

### WeakHashMap

WeakHashMap是一种弱引用map，内部的key会存储为弱引用，当jvm gc的时候，如果这些key没有强引用存在的话，会被gc回收掉，下一次当我们操作map的时候会把对应的Entry整个删除掉，基于这种特性，WeakHashMap特别适用于缓存处理。

WeakHashMap因为gc的时候会把没有强引用的key回收掉，所以注定了它里面的元素不会太多，因此也就不需要像HashMap那样元素多的时候转化为红黑树来处理了。

因此，WeakHashMap的存储结构只有（数组 + 链表）。

```java
public class WeakHashMapTest {
    public static void main(String[] args) {
        Map<String, Integer> map = new WeakHashMap<>(3);

        // 放入3个new String()声明的字符串
        //key引用堆中对象
        map.put(new String("1"), 1);
        map.put(new String("2"), 2);
        map.put(new String("3"), 3);

        // 放入不用new String()声明的字符串
        //key引用字符串常量池中“6”对象
        map.put("6", 6);

        // 使用key强引用"3"这个字符串
        String key = null;
        for (String s : map.keySet()) {
            // 这个"3"和new String("3")不是一个引用
            if (s.equals("3")) {
                key = s;
            }
        }

        // 输出{6=6, 1=1, 2=2, 3=3}，未gc所有key都可以打印出来
        System.out.println(map);

        // gc一下
        System.gc();

        //{6=6, 3=3},GC Roots可以查到“6”、“3”
        System.out.println(map);


        // 放一个new String()声明的字符串
        map.put(new String("4"), 4);

        // 输出{4=4, 6=6, 3=3}，gc后放入的值和强引用的key可以打印出来
        System.out.println(map);

        // key与"3"的引用断裂
        key = null;

        // gc一下
        System.gc();

        // 输出{6=6}，gc后强引用的key可以打印出来
        System.out.println(map);
    }
}
```



1. WeakHashMap使用（数组 + 链表）存储结构；
2. WeakHashMap中的key是弱引用，gc的时候会被清除；
3. 每次对map的操作都会剔除失效key对应的Entry；
4. 使用String作为key时，一定要使用new String()这样的方式声明key，才会失效，其它的基本类型的包装类型是一样的；
5. WeakHashMap常用来作为缓存使用；

### 参考

1. [ LinkedHashMap 源码详细分析（JDK1.8）](http://www.tianxiaobo.com/2018/01/24/LinkedHashMap-源码详细分析（JDK1-8）/)
2. [jdk1.8的Hashtable源码解析](https://blog.csdn.net/qq_29864971/article/details/79326106)
3. [死磕 java集合之WeakHashMap源码分析](https://www.cnblogs.com/tong-yuan/p/10639404.html)