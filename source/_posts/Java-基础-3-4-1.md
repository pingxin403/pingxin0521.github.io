---
title: 从HashMap 的源码理解原理（基于JDK 1.8）
date: 2020-1-16 18:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

### HashMap

HashMap是一个用于存储Key-Value键值对的集合，每一个键值对也叫做**Entry**。这些键值对（Entry）分散存储在一个数组中，这个数组就是HashMap的主干。

HashMap数组每一个元素的初始值都是Null。

HashMap 最早出现在 JDK 1.2中，底层基于散列算法实现。HashMap 允许 null 键和 null 值，在计算哈键的哈希值时，null 键哈希值为 0。

HashMap 并不保证键值对的顺序，这意味着在进行某些操作后，键值对的顺序可能会发生变化。另外，需要注意的是，HashMap 是非线程安全类，在多线程环境下可能会存在问题。

<!--more-->

对于HashMap，我们最常使用的是这几个方法：Get、Put、Contains、Remove、Replace。

下面依次介绍

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{
    //...
    }
```

看其继承结构，着重观察AbstractMap，

```java
public abstract class AbstractMap<K,V> implements Map<K,V> 
```

查看其源码，会发现这个类其实是对Map接口的通用接口的实现。

先有个概念，即Map为什么那么复杂，因为其内部管理的是K-V类型，因此有三个视图：entrySet、keySet、values。其余的方法都是通过entrySet的迭代来进行实现。

```java
transient Set<K>        keySet;
transient Collection<V> values;

public abstract Set<Entry<K,V>> entrySet();

public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new AbstractSet<K>() {
                public Iterator<K> iterator() {
                    return new Iterator<K>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public K next() {
                            return i.next().getKey();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object k) {
                    return AbstractMap.this.containsKey(k);
                }
            };
            keySet = ks;
        }
        return ks;
    }
    public Collection<V> values() {
        Collection<V> vals = values;
        if (vals == null) {
            vals = new AbstractCollection<V>() {
                public Iterator<V> iterator() {
                    return new Iterator<V>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public V next() {
                            return i.next().getValue();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object v) {
                    return AbstractMap.this.containsValue(v);
                }
            };
            values = vals;
        }
        return vals;
    }    
   
```

例如：

```java
    public V remove(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        Entry<K,V> correctEntry = null;
        if (key==null) {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    correctEntry = e;
            }
        } else {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    correctEntry = e;
            }
        }

        V oldValue = null;
        if (correctEntry !=null) {
            oldValue = correctEntry.getValue();
            i.remove();
        }
        return oldValue;
    }
```

只能说AbstractMap是其他Map的抽象实现，具体使用什么结构进行存储entrySet还是在子类中实现。

### HashMap

要搞清楚 HashMap ，首先要知道 HashMap 是什么，即它的存储结构-字段，其次要弄明白它能做什么，即它的功能实现-方法，下面分别展开分析。

从结构上来讲，HashMap 是数组 + 链表 + 红黑树（JDK1.8 中新增的红黑树部分）实现的，如下图所示

![lvGTjU.png](https://s2.ax1x.com/2020/01/16/lvGTjU.png)

对于拉链式的散列算法，其数据结构是由数组和链表（或树形结构）组成。在进行增删查等操作时，首先要定位到元素的所在桶的位置，之后再从链表中定位该元素。比如我们要查询上图结构中是否包含元素`35`，步骤如下：

1. 定位元素`35`所处桶的位置，`index = 35 % 16 = 3`
2. 在`3`号桶所指向的链表中继续查找，发现35在链表中。

上面就是 HashMap 底层数据结构的原理，HashMap 基本操作就是对拉链式散列算法基本操作的一层包装。不同的地方在于 JDK 1.8 中引入了红黑树，底层数据结构由`数组+链表`变为了`数组+链表+红黑树`，不过本质并未变。

链表长度大于8变为红黑树（在put和resize中），红黑树节点数小于等于6变为链表（在resize中）

这里我们首先要弄明白两个问题：数据底层到底存储的是什么？这样的存储有什么优点？

从源码中得知，HashMap 类有一个非常重要的字段，就是 `Node<K,V>[] table`，即哈希桶数组，我们看一下源码，即Node[JDK1.8] 

```java
   static class Node<K,V> implements Map.Entry<K,V> {
       //定位数组索引位置
        final int hash;
        final K key;
        V value;
       //链表的下一个节点
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            //返回旧值
            return oldValue;
        }
		//用于hash比较之后
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                //kv相等
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

Node 是 HashMap 的一个内部类，其实现了 Map.Entry 接口，本质就是一个映射（键值对），上图中的每一个黑点就是一个 Node 对象。

然后是链表转为红黑树之后的树节点

```java

    /**
     * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
     * extends Node) so can be used as extension of either regular or
     * linked node.
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
        //省略方法
}
```

HashMap 就是使用哈希表来存储的，哈希表为解决冲突，可以采用开放地址法和链地址法等来解决，Java 中的 HashMap 采用了链地址法。链地址法简单来说就是数组加链表的结合，在每个数组元素上都有一个链表结构，当数据被 hash 后，得到数组下标位置，把数据放在对应数组下标元素的链表上，例如程序执行下面的代码：

```java
        HashMap<String,String> map=new HashMap<>();
        map.put("str","str");
```

系统将调用“str”这个key的hashCode（）方法得到其hashCode值，然后通过 hash 算法的后两步运算（高位运算和取模运算，下面将会介绍）来定位该键值对的存储位置，有时两个key对定位到相同的位置，表示发生了 Hash 碰撞，当然 Hash 算法计算结果越分散均匀，Hash 碰撞的概率就越小，map的存取效率就会越高。

如果哈希桶数组很大，即使较差的 Hash 算法也会比较分散，否则即使好的 Hash 算法也会出现较多碰撞，所以就需要在时间和空间成本之间权衡，其实就是根据实际情况确定哈希桶数组的大小，并在此基础上设计好的 Hash 算法减少 Hash 碰撞，那么通过什么方式来控制map使得 Hash 的碰撞概率低，哈希桶数组（Node[] table）占用较少的内存呢？答案就是好的 Hash 算法和扩容机制。

在理解 Hash 算法和扩容之前，先了解一下 HashMap 的一些重要的属性字段：

```java
    /**
     * 初始化容量(必须是二的n次幂)
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 集合最大容量(必须是二的幂)
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 负载因子，默认的0.75
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     	当链表的值超过8则会转红黑树
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
      当链表的值小于6则会从红黑树转回链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 当Map里面的数量超过这个值时，表中的桶才能进行树形化 ，
     否则，桶内元素太多时会扩容，而不是树形化 为了避免进行扩容、树形化选择的冲突，
     这个值不能小于 4 * TREEIFY_THRESHOLD
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
        /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;

    /**
    用来存放缓存
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * HashMap中存储节点的数量
     */
    transient int size;

    /**用来记录HashMap的修改次数，具体参考fast-fail
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     * 用来调整大小下一个容量的值计算方式为(容量*负载因子)
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
    	哈希表的加载因子
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;

```

1. HashMap 的 数组 Node[] table 初始化的长度 capacity 是16，哈希桶数组长度 length 大小必须是2的n次方，这种设计主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap 定位哈希桶索引位置时，也加入了高位参与运算的过程
2. loadFactor 为负载因子，默认值是 0.75，0.75 是对时间和空间效率的一个平衡选择，建议大家不要修改，除非在时间或者空间上比较特殊的情况下。例如：如果内存空间很多而又对时间效率要求很高，可以降低负载因子 loadFactor 的值，相反，如果内存空间较少而又对时间效率要求不高，可以增加负载因子 loadFactor 的值，这个值可以大于1
3. threshold 是 HashMap 所能容纳的最大的键值对的个数，threshold  = capacity  * loadFactor，也就是说 capacity  数组一定的情况下，负载因子越大，所能容纳的键值对个数越多，超出 threshold 这个数目就重新 resize（扩容），扩容后的 HashMap 的容量是之前的2倍。
4. size 是 HashMap 中实际存在的键值对的数量，注意和 Node[] table 的长度 capacity 、容纳最大键值对数量  threshold 的区别
5. modCount 主要用来记录 HashMap 内部结构发生变化的次数，主要用于迭代的快速失败。强调一点，内部结构发生变化是指结构发生变化，例如 put 新的键值对，但是某个 key 对应的 value 值被覆盖不属于结构变化
6. TREEIFY_THRESHOLD 和 MIN_TREEIFY_THRESHOLD，即使 hash 算法 和负载因子设计的再完美，也避免不了拉链过长的情况，一旦出现拉链过长，严重影响 HashMap 的性能，于是在 JDK1.8 中对数据结构做了进一步的优化，引入了红黑树。当链表长度太长（超过 TREEIFY_THRESHOLD = 8）时，当Node[] table 数组长度超过 64（MIN_TREEIFY_THRESHOLD = 64） 时，链表就转化为了红黑树，利用红黑树快速增删改查的特点提高 HashMap 的性能。

对于 HashMap 来说，负载因子是一个很重要的参数，该参数反应了 HashMap 桶数组的使用情况（假设键值对节点均匀分布在桶数组中）。通过调节负载因子，可使 HashMap 时间和空间复杂度上有不同的表现。当我们调低负载因子时，HashMap 所能容纳的键值对数量变少。扩容时，重新将键值对存储新的桶数组里，键的键之间产生的碰撞会下降，链表长度变短。此时，HashMap 的增删改查等操作的效率将会变高，这里是典型的拿空间换时间。相反，如果增加负载因子（负载因子可以大于1），HashMap 所能容纳的键值对数量变多，空间利用率高，但碰撞率也高。这意味着链表长度变长，效率也随之降低，这种情况是拿时间换空间。至于负载因子怎么调节，这个看使用场景了。一般情况下，我们用默认值就可以了。

#### 构造方法源码

 下面我们列出 HashMap 中的所有构造方法的源码，特别注意的是 tableSizeFor 方法，**该方法将传入的 capacity 转化为 2的n次方的 threshold（容量的临界值，构造时数组长度 == 临界值）**：

```java
    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //初始容量不能大于MAXIMUM_CAPACITY
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //负载因子限制
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        //根据容量计算扩容的临界值，比如容量是30,30<2^5，所以临界值为2^5=32,必须为2的次方，当实际大小大于临界值时进行扩容。
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    /**
     * Implements Map.putAll and Map constructor.
     *
     * @param m the map
     * @param evict false when initially constructing this map, else
     * true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

#### 确定哈希桶数组索引位置

HashMap的本质就是通过key值的hash值进行存取，那么hash值是怎么获取的呢？

增加、删除、查找键值对需要定位到哈希桶数组索引位置都是很关键的第一步，HashMap 的数据结构是 链表+数组 /红黑树的结合，所以希望 HashMap 里的元素位置尽量分布均匀些，使得每个位置上的元素只有一个，那么当我们用 hash 算法求得这个位置的时候，马上就知道对应位置的元素就是我们要找的，不用遍历链表，大大优化了查询的效率。HashMap 定位数组索引位置，直接决定了 hash 方法的离散性能，下面我们看看源码：

```java
    /**
     * Computes key.hashCode() and spreads (XORs) higher bits of hash
     * to lower.  Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int hash(Object key) {
        int h;
        //让高位数据与低位数据进行异或，以此加大低位信息的随机性，变相的让高位数据参与到计算中
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
//求下标，n为Node节点数组长度
//index = (n - 1) & hash
```

Hash 算法的本质就三步：**获取key的hashCode值、高位运算、取模运算**

HashMap 允许 null 键和 null 值，在计算哈键的哈希值时，null 键哈希值为 0。

在JDK1.8 中优化了高位运算的算法，通过hashCode的高16位异或低16位实现的：`（h = key.hashCode()）^ （h >>>16）`，主要从速度、功效、质量来考虑的，这么做可以在数组table的length较小的时候，也能保证考虑到高低Bit都参与到 Hash 的计算中，同时不会有太大的开销，下面具体说明，n 为table的长度：

![lvlVZn.png](https://s2.ax1x.com/2020/01/16/lvlVZn.png)

在hashMap中认为为同一个key的条件：

```java
e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k)))
 //1.比较hash值
 //2.比较k值
```

#### put 方法

**总结**：put方法是存储/更新k-v的方法，结果是新增或者更新一个节点，因此只需要处理扩容机制。

1. 如果table数组为空则调用resize方法，会创建数组，**在第一次调用put方法时，HashMap才会进行初始化**

2. **i = (n - 1) & hash 计算下标索引**

   ```java
   //此情况比较是否是同一k值
   p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k)))
   ```

   1. 数组下标为空则创建Node节点
   2. 不为空则说明已存在或者地址冲突，已存在直接更新v值，地址冲突使用**开地址**方法处理
      - 如果该节点是TreeNode，说明该位置为红黑树，使用红黑树的添加方法putTreeVal
      - 如果该节点链表Node，从头部开始往后找，如果找到则覆盖v旧值，返回v旧值；没找到则添加到末尾,如果链表长度大于TREEIFY_THRESHOLD（8）,则使用treeifyBin进行树化
        - 如果数组长度小于MIN_TREEIFY_CAPACITY(64)，则进行resize操作（扩容），不进行树化
        - 否则进行树化，需要先将链表转为双向链表，同时将每个节点包装成TreeNode，再调用数组节点（TreeNode）的树化方法

```java
    public V put(K key, V value) {
        //计算hash值
        return putVal(hash(key), key, value, false, true);
    }
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //table为空，调用resize扩容并创建table
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //i = (n - 1) & hash 计算下标索引，为空则创建Node节点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //该数组下标不为空
        else {
            Node<K,V> e; K k;
            //k存在且为Node链表的成员，直接替换，不用遍历
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //如果是红黑树，则使用红黑树的处理方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //如果是链表
            else {
                //binCount计数链表长度，
                for (int binCount = 0; ; ++binCount) {
                    
                    if ((e = p.next) == null) {
                        //添加到末尾
                        p.next = newNode(hash, key, value, null);
                        //链表长度大于8时，执行树化方法，并不一定会转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //找到节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //指向下一个寻找的节点
                    p = e;
                }
            }
            //map中存在对应k的节点，则覆盖旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
        //空方法,允许LinkedHashMap后处理的回调                
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //修改操作次数加一
        ++modCount;
        //键值对数量达到阀值，进行resize操作
        if (++size > threshold)
            resize();
        //空方法,允许LinkedHashMap后处理的回调   
        afterNodeInsertion(evict);
        return null;
    }

    // Create a regular (non-tree) node
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
        return new Node<>(hash, key, value, next);
    }

```

**链表树化**

JDK 1.8 对 HashMap 实现进行了改进。最大的改进莫过于在引入了红黑树处理频繁的碰撞，代码复杂度也随之上升。比如，以前只需实现一套针对链表操作的方法即可。而引入红黑树后，需要另外实现红黑树相关的操作。

让我们看一下treeifyBin这个方法，树化之前会先将Node链表转为TreeNode双向链表

在扩容过程中，树化要满足两个条件：

1. 链表长度大于等于 TREEIFY_THRESHOLD
2. 桶数组容量大于等于 MIN_TREEIFY_CAPACITY

```java
    /**
     * Replaces all linked nodes in bin at index for given hash unless
     * table is too small, in which case resizes instead.
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果数组长度小于MIN_TREEIFY_CAPACITY(64)，则进行resize操作
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)           
            resize();
        //否则进行树化
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            //链表转为红黑树需要先将链表转为双向链表
            do {
       // 将每个节点包装成TreeNode。
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    // 将所有TreeNode连接在一起此时只是链表结构
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                //再调用数组节点的树化方法
                hd.treeify(tab);
        }
    }

```

我们来看一下节点转换方法replacementTreeNode,实现很简单，返回即可

```java
    // For treeifyBin
    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
    }
```

我们看一下treeify代码，treeify只是在原有TreeNode双向链表上构建红黑树。

TreeNode 继承自 Node 类，所以 TreeNode 仍然包含 next 引用，原链表的节点顺序最终通过 next 引用被保存下来。

```java
        /**
        TreeNode的成员方法        
         * Forms tree of the nodes linked from this node.
         */
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
         // 以for循环的方式遍历刚才我们创建的链表。           
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                // next向前推进。
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                // 为树根节点赋值。
                if (root == null) {
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    // x即为当前访问链表中的项。
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    // 此时红黑树已经有了根节点，上面获取了当前加入红黑树的项的key和hash值进入核心循环。
                // 这里从root开始，是以一个自顶向下的方式遍历添加。
                // for循环没有控制条件，由代码内break跳出循环。
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        // dir：directory，比较添加项与当前树中访问节点的hash值判断加入项的路径，-1为左子树，+1为右子树。
                    // ph：parent hash。
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);
// xp：x parent。
                        TreeNode<K,V> xp = p;
                    // 找到符合x添加条件的节点。                        
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            // 如果xp的hash值大于x的hash值，将x添加在xp的左边。
                            if (dir <= 0)
                                xp.left = x;
                            // 反之添加在xp的右边。
                            else
                                xp.right = x;
                            // 维护添加后红黑树的红黑结构。
                            root = balanceInsertion(root, x);
                            
                             // 跳出循环当前链表中的项成功的添加到了红黑树中。
                            break;
                        }
                    }
                }
            }
           // 确保给定根是其bin的第一个节点
            moveRootToFront(tab, root);
        }
```

第一次循环会将链表中的首节点作为红黑树的根，而后的循环会将链表中的的项通过比较hash值然后连接到相应树节点的左边或者右边，插入可能会破坏树的结构所以接着执行balanceInsertion。

我们假设树化前，链表结构如下：

![1pdQIK.png](https://s2.ax1x.com/2020/01/18/1pdQIK.png)

HashMap 在设计之初，并没有考虑到以后会引入红黑树进行优化。所以并没有像 TreeMap 那样，要求键类实现 comparable 接口或提供相应的比较器。但由于树化过程需要比较两个键对象的大小，在键类没有实现 comparable 接口的情况下，怎么比较键与键之间的大小了就成了一个棘手的问题。为了解决这个问题，HashMap 是做了三步处理，确保可以比较出两个键的大小，如下：

1. 比较键与键之间 hash 的大小，如果 hash 相同，继续往下比较
2. 检测键类是否实现了 Comparable 接口，如果实现调用 compareTo 方法进行比较
3. 如果仍未比较出大小，就需要进行仲裁了，仲裁方法为 tieBreakOrder（大家自己看源码吧）

tie break 是网球术语，可以理解为加时赛的意思，起这个名字还是挺有意思的。

通过上面三次比较，最终就可以比较出孰大孰小。比较出大小后就可以构造红黑树了，最终构造出的红黑树如下：

![1pd1PO.png](https://s2.ax1x.com/2020/01/18/1pd1PO.png)

橙色的箭头表示 TreeNode 的 next 引用。由于空间有限，prev 引用未画出。可以看出，链表转成红黑树后，原链表的顺序仍然会被引用仍被保留了（红黑树的根节点会被移动到链表的第一位），我们仍然可以按遍历链表的方式去遍历上面的红黑树。这样的结构为后面红黑树的切分以及红黑树转成链表做好了铺垫

put 方法执行逻辑的图：

![lvsQq1.png](https://s2.ax1x.com/2020/01/16/lvsQq1.png)

#### get 方法

我们都知道，HashMap中最常用的方法就是 put 和 get 方法，上面介绍了put方法的源码，下面分析一下 get 方法的源码：

```java
    public V get(Object key) {
        Node<K,V> e;
        //通过k的hash值和k找到node节点，不为空则返回v
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        // 1. 定位键值对所在桶的位置
        //node数组不为空，且首节点不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 2. 如果 first 是 TreeNode 类型，则调用黑红树查找方法
            //首节点就是要找的节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //首节点不为空且不是要找的节点
            if ((e = first.next) != null) {
                //首节点为红黑树节点，使用红黑树的查找方法
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    // 2. 对链表进行查找
                    //首节点为链表节点，使用链表的查找方法
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //没有找到
        return null;
    }
```

#### contains

```java
    /**
     * Returns <tt>true</tt> if this map contains a mapping for the
     * specified key.
     *
     * @param   key   The key whose presence in this map is to be tested
     * @return <tt>true</tt> if this map contains a mapping for the specified
     * key.
     */
    public boolean containsKey(Object key) {
        //使用get，如果能get到就说明有
        return getNode(hash(key), key) != null;
    }
    /**
    全部遍历，效率低
     * Returns <tt>true</tt> if this map maps one or more keys to the
     * specified value.
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     *         specified value
     */
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                //使用Node/TreeNode的next
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```

#### remove

**总结：**本质上就是删除对应的k，结果就是删除一个Node或者什么也不做（不存在对应的key），并不进行红黑树转为链表的操作（就算长度小于UNTREEIFY_THRESHOLD（6））

1. 如果没有找到对应k值的node则返回null
2. 找到则调用对应删除方法

```java
    /**
     * Removes the mapping for the specified key from this map if present.
     *
     * @param  key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }
    /**
     * Implements Map.remove and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to match if matchValue, else ignored
     * @param matchValue if true only remove if value is equal
     * @param movable if false do not move other nodes while removing
     * @return the node, or null if none
     */
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
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
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
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

#### 扩容机制

扩容（resize）就是重新计算容量，向 HashMap 中不断的添加数据，HashMap 内部对象的数组无法承载更多的元素时就需要对象扩大数组的长度，以便能装入更多的元素。Java的数组是无法自动扩容的，方法就是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水就需要换大桶一样。

JDK1.7采用的是单链表的头插入方式，也就是同一位置上新元素总会被放在链表的头位置，在旧数组中同一条Node链表上的元素，通过重新计算索引位置后，有可能放到新数组的不同的位置上。 JDK1.7的源码：

![1SSARP.png](https://s2.ax1x.com/2020/01/17/1SSARP.png)

下面举个例子说明扩容的过程，假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。其中的哈希桶数组table的size=2， 所以key = 3、7、5，put顺序依次为 5、7、3。在mod 2以后都冲突在table[1]这里了。这里假设负载因子 loadFactor=1，即当键值对的实际大小size 大于 table的实际大小时进行扩容。接下来的三个步骤是哈希桶数组 resize成4，然后所有的Node重新rehash的过程。

![1SS8zV.png](https://s2.ax1x.com/2020/01/17/1SS8zV.png)

下面我们讲解下JDK1.8做了哪些优化。经过观测可以发现，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![1SSWod.png](https://s2.ax1x.com/2020/01/17/1SSWod.png)

元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![1SpimF.png](https://s2.ax1x.com/2020/01/17/1SpimF.png)

因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：

![1SpGtA.png](https://s2.ax1x.com/2020/01/17/1SpGtA.png)

这个设计非常的巧妙，即省去了重新计算hash值的时间，而且同时，由于新增的 1 bit 是0还是1可以认为是随机的，因此 resize 的过程，均匀的把之前的冲突的节点分散到新的 bucket 了，这就是JDK1.8新增的优化点。有一点注意区别，JDK1.7中 rehash 的时候，旧链表迁移新链表的时候，如果新表的数组索引位置相同，则链表元素会倒置，但是从上图可以看出，JDK1.8不会倒置。

**总结：**resize方法本质就是修改table数组的大小，结果为数组大小为原来的二倍或者不变。

1. 如果旧容量大于0

   - 如果旧的容量超出最大数组长度MAXIMUM_CAPACITY，无法扩容，容量默认设置为2^31-1，直接返回旧的数组
   - 否则，则容量扩大为原来的两倍。若扩容后容量（新容量）小于MAXIMUM_CAPACITY且旧容量大于DEFAULT_INITIAL_CAPACITY（16），临界值也扩大为原来的两倍

2. 如果旧数组容量小于等于0且旧临界值大于0，新容量直接设置为旧的临界值的大小

3. 如果旧数组容量小于等于0且旧临界值小于等于0，新容量直接设置为DEFAULT_INITIAL_CAPACITY（16），新临界值设置为(int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY)

4. 如果新临界值为0，重新设置

5. 继续修改临界值和table数组

   1. 如果原table数组为空，则返回新table数组（**新建/初始化**）

   2. 如果原table首节点为空则跳过

   3. 如果原table首节点不为空，**e.hash & oldCap==0则节点在原位置(j)，为1则转移到新位置(j+oldCap)**,不用重新通过计算。

      1. 如果就一个节点，直接转移，否则进行下面操作

      2. 如果该节点是TreeNode，则说明是红黑树，使用红黑树的split方法，也是分为新位置(j+oldCap)旧位置（j）,因为为红黑树，节点数一定大于TREEIFY_THRESHOLD(8)，拆分后有可能小于等于UNTREEIFY_THRESHOLD（6）

         先对TreeNode转为双向链表结构，如果小于等于6则进行untreeify；否则进行treeify

      3. 如果是链表，则进行链表的拆分，因为为数组，所以原来数组长度小于TREEIFY_THRESHOLD(8)，拆分后也一定小于8

下面分析一下JDK1.8中resize的源码：

```java
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        //旧的数组
        Node<K,V>[] oldTab = table;
        //旧的容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //旧的临界值
        int oldThr = threshold;
        //新的容量，新的临界值
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //超出最大数组长度，无法扩容，容量默认设置为2^31-1，直接返回旧的数组
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //数组长度和容量扩展为原来的2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        //如果旧数组容量小于等于0且旧临界值大于0，新容量直接设置为旧的临界值的大小
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //如果旧数组容量小于等于0且旧临界值小于等于0，新容量直接设置为DEFAULT_INITIAL_CAPACITY（16）
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            //临界值设置为
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //新临界值为0，重新设置
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //临界值修改
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //新的数组
        table = newTab;
        //转移
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                //数组节点不为空
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    //只有一个首节点，直接转移
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    //首节点为红黑树节点,使用红黑树的处理方法
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //首节点为链表节点
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //0为原位置，为1则移动到新位置
                            if ((e.hash & oldCap) == 0) {
                                //原位置链表处理
                                if (loTail == null)
                                    loHead = e;
                                else	
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                //新位置链表处理
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            //原位置
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            //新位置
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        //返回新数组
        return newTab;
    }

```

**红黑树拆分**

扩容后，普通节点需要重新映射，红黑树节点也不例外。按照一般的思路，我们可以先把红黑树转成链表，之后再重新映射链表即可。这种处理方式是大家比较容易想到的，但这样做会损失一定的效率。不同于上面的处理方式，HashMap 实现的思路则是上好佳。如上节所说，在将普通链表转成红黑树时，HashMap 通过两个额外的引用 next 和 prev 保留了原链表的节点顺序。这样再对红黑树进行重新映射时，完全可以按照映射链表的方式进行。这样就避免了将红黑树转成链表后再进行映射，无形中提高了效率。

红黑树的扩容的处理与链表的处理大同小异，源码如下：

```java
        /**
        TreeNode的成员方法
         * Splits nodes in a tree bin into lower and upper tree bins,
         * or untreeifies if now too small. Called only from resize;
         * see above discussion about split bits and indices.
         	
         * this, newTab, j, oldCap
         * @param map the map
         * @param tab the table for recording bin heads
         * @param index the index of the table being split
         * @param bit the bit of hash to split on
         */
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            //当前TreeNode
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                //使用链表记录
                //hash&bit(oldCap)==0，说明是原位置
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                //hash&bit(oldCap)==1，新位置
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }
            //旧位置
            if (loHead != null) {
 //元素大小小于等于UNTREEIFY_THRESHOLD（6）,map不需要转为红黑树，使用untreeify方法
                if (lc <= UNTREEIFY_THRESHOLD)
                    //非树化使用HashMap对象
                    tab[index] = loHead.untreeify(map);
                else {
 //否则则还是红黑树，使用treeify树化                   
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        //树化使用Node<数组>
                        loHead.treeify(tab);
                }
            }
            //新位置
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }
```

从源码上可以看得出，重新映射红黑树的逻辑和重新映射链表的逻辑基本一致。不同的地方在于，重新映射后，会将红黑树拆分成两条由 TreeNode 组成的链表。如果链表长度小于 UNTREEIFY_THRESHOLD，则将链表转换成普通链表。否则根据条件重新将 TreeNode 链表树化。举个例子说明一下，假设扩容后，重新映射上图的红黑树，映射结果如下：

![1pdaZt.png](https://s2.ax1x.com/2020/01/18/1pdaZt.png)

**红黑树链化**

treeify的分析可以看上面，下面分析untreeify，将TreeNode的双向链表转为Node的单向链表

前面说过，红黑树中仍然保留了原链表节点顺序。有了这个前提，再将红黑树转成链表就简单多了，仅需将 TreeNode 链表转成 Node 类型的链表即可。

```java
        /**
        TreeNode的成员方法
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            //从当前TreeNode开始进行转换为链表
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }
```

我们再来看一下replacementNode方法

```java
// For conversion from TreeNodes to plain nodes
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
```

**总结**

1. 扩容是一个特别耗性能的操作，所以当程序员在使用 HashMap，正确估算 map 的大小，初始化的时候给一个大致的数值，避免 map 进行频繁的扩容

2. 负载因子 loadFactor 是可以修改的，也可以大于1，但是建议不要轻易修改，除非情况特殊

3. HashMap 是非线程安全的，不要在并发的情况下使用 HashMap，建议使用 ConcurrentHashMap

4. JDK1.8 中引入了红黑树，大大的提高了 HashMap 的性能
5. **树化之后，TreeNode依然维护着next和prev**，红黑树会将根节点移到链表的首节点位置

#### 遍历

和查找查找一样，遍历操作也是大家使用频率比较高的一个操作。对于 遍历 HashMap，我们一般都会用下面的方式：

```
for(Object key : map.keySet()) {
    // do something
}
```

或

```
for(HashMap.Entry entry : map.entrySet()) {
    // do something
}
```

从上面代码片段中可以看出，大家一般都是对 HashMap 的 key 集合或 Entry 集合进行遍历。上面代码片段中用 foreach 遍历 keySet 方法产生的集合，在编译时会转换成用迭代器遍历，等价于：

```
Set keys = map.keySet();
Iterator ite = keys.iterator();
while (ite.hasNext()) {
    Object key = ite.next();
    // do something
}
```

大家在遍历 HashMap 的过程中会发现，多次对 HashMap 进行遍历时，遍历结果顺序都是一致的。但这个顺序和插入的顺序一般都是不一致的。产生上述行为的原因是怎样的呢？大家想一下原因。我先把遍历相关的代码贴出来，如下：

```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

/**
 * 键集合
 */
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    // 省略部分代码
}

/**
 * 键迭代器
 */
final class KeyIterator extends HashIterator 
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        if (t != null && size > 0) { // advance to first entry 
            // 寻找第一个包含链表节点引用的桶
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        if ((next = (current = e).next) == null && (t = table) != null) {
            // 寻找下一个包含链表节点引用的桶
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }
    //省略部分代码
}
```

如上面的源码，遍历所有的键时，首先要获取键集合`KeySet`对象，然后再通过 KeySet 的迭代器`KeyIterator`进行遍历。KeyIterator 类继承自`HashIterator`类，核心逻辑也封装在 HashIterator 类中。HashIterator 的逻辑并不复杂，在初始化时，HashIterator 先从桶数组中找到包含链表节点引用的桶。然后对这个桶指向的链表进行遍历。遍历完成后，再继续寻找下一个包含链表节点引用的桶，找到继续遍历。找不到，则结束遍历。举个例子，假设我们遍历下图的结构：

![1p8eWn.png](https://s2.ax1x.com/2020/01/18/1p8eWn.png)

HashIterator 在初始化时，会先遍历桶数组，找到包含链表节点引用的桶，对应图中就是3号桶。随后由 nextNode 方法遍历该桶所指向的链表。遍历完3号桶后，nextNode 方法继续寻找下一个不为空的桶，对应图中的7号桶。之后流程和上面类似，直至遍历完最后一个桶。以上就是 HashIterator 的核心逻辑的流程，对应下图：

![1p8KyV.png](https://s2.ax1x.com/2020/01/18/1p8KyV.png)

遍历上图的最终结果是 `19 -> 3 -> 35 -> 7 -> 11 -> 43 -> 59`，为了验证正确性，简单写点测试代码跑一下看看。测试代码如下：

```java
/**
 * 应在 JDK 1.8 下测试，其他环境下不保证结果和上面一致
 */
public class HashMapTest {

    @Test
    public void testTraversal() {
        HashMap<Integer, String> map = new HashMap(16);
        map.put(7, "");
        map.put(11, "");
        map.put(43, "");
        map.put(59, "");
        map.put(19, "");
        map.put(3, "");
        map.put(35, "");
//19 -> 3 -> 35 -> 7 -> 11 -> 43 -> 59
        System.out.println("遍历结果：");
        for (Integer key : map.keySet()) {
            System.out.print(key + " -> ");
        }
    }
}
```

在 JDK 1.8 版本中，为了避免过长的链表对 HashMap 性能的影响，特地引入了红黑树优化性能。但在上面的源码中并没有发现红黑树遍历的相关逻辑，这是为什么呢？对于被转换成红黑树的链表该如何遍历呢？

其实在TreeNode内部有着next和prev的引用，且一直在维护，树化操作之前会先将TreeNode变为双向链表。之后在进行转为红黑树，可以从treeifyBin方法中看到链表转为红黑树的逻辑，也可以从TreeNode.split方法中看到红黑树分裂为红黑树或者链表的逻辑。本质上都是先使用TreeNode维护好链表，再调用首节点的untreeify或者treeify方法，其实treeify只是在原有TreeNode双向链表上构建红黑树，可以从上面的源码分析看出来；而untreeify更简单，将TreeNode的双向链表转为Node的单向链表。**红黑树会将根节点移到链表的首节点位置**

#### 其他视图

HashMap支持keySet、values、entrySet三种视图，视图就是说不允许进行修改，只允许进行访问和删除，本质就是使用迭代器进行访问

举例：

```java
    public static void main(String [] args)
    {

        HashMap<String,String> map=new HashMap<>();
        int i=100;
        for (int j = 0; j < i; j++) {
            map.put("str"+j,""+j);
        }
        //更改视图则报错
        Set<String> keySet =
                map.keySet();
 //       keySet.add("str101");
        //true
       System.out.println(keySet.remove("str0"));


    }
```

源码实现

```java

    /**
     * Returns a {@link Set} view of the keys contained in this map.
     * The set is backed by the map, so changes to the map are
     * reflected in the set, and vice-versa.  If the map is modified
     * while an iteration over the set is in progress (except through
     * the iterator's own <tt>remove</tt> operation), the results of
     * the iteration are undefined.  The set supports element removal,
     * which removes the corresponding mapping from the map, via the
     * <tt>Iterator.remove</tt>, <tt>Set.remove</tt>,
     * <tt>removeAll</tt>, <tt>retainAll</tt>, and <tt>clear</tt>
     * operations.  It does not support the <tt>add</tt> or <tt>addAll</tt>
     * operations.
     *
     * @return a set view of the keys contained in this map
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new KeySet();
            keySet = ks;
        }
        return ks;
    }

    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
    /**
     * Returns a {@link Collection} view of the values contained in this map.
     * The collection is backed by the map, so changes to the map are
     * reflected in the collection, and vice-versa.  If the map is
     * modified while an iteration over the collection is in progress
     * (except through the iterator's own <tt>remove</tt> operation),
     * the results of the iteration are undefined.  The collection
     * supports element removal, which removes the corresponding
     * mapping from the map, via the <tt>Iterator.remove</tt>,
     * <tt>Collection.remove</tt>, <tt>removeAll</tt>,
     * <tt>retainAll</tt> and <tt>clear</tt> operations.  It does not
     * support the <tt>add</tt> or <tt>addAll</tt> operations.
     *
     * @return a view of the values contained in this map
     */
    public Collection<V> values() {
        Collection<V> vs = values;
        if (vs == null) {
            vs = new Values();
            values = vs;
        }
        return vs;
    }
    /**
     * Returns a {@link Set} view of the mappings contained in this map.
     * The set is backed by the map, so changes to the map are
     * reflected in the set, and vice-versa.  If the map is modified
     * while an iteration over the set is in progress (except through
     * the iterator's own <tt>remove</tt> operation, or through the
     * <tt>setValue</tt> operation on a map entry returned by the
     * iterator) the results of the iteration are undefined.  The set
     * supports element removal, which removes the corresponding
     * mapping from the map, via the <tt>Iterator.remove</tt>,
     * <tt>Set.remove</tt>, <tt>removeAll</tt>, <tt>retainAll</tt> and
     * <tt>clear</tt> operations.  It does not support the
     * <tt>add</tt> or <tt>addAll</tt> operations.
     *
     * @return a set view of the mappings contained in this map
     */
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
    }
```

#### 其他细节

##### 被 transient 所修饰 table 变量

如果大家细心阅读 HashMap 的源码，会发现桶数组 table 被申明为 transient。transient 表示易变的意思，在 Java 中，被该关键字修饰的变量不会被默认的序列化机制序列化。我们再回到源码中，考虑一个问题：桶数组 table 是 HashMap 底层重要的数据结构，不序列化的话，别人还怎么还原呢？

这里简单说明一下吧，HashMap 并没有使用默认的序列化机制，而是通过实现`readObject/writeObject`两个方法自定义了序列化的内容。这样做是有原因的，试问一句，HashMap 中存储的内容是什么？不用说，大家也知道是`键值对`。所以只要我们把键值对序列化了，我们就可以根据键值对数据重建 HashMap。有的朋友可能会想，序列化 table 不是可以一步到位，后面直接还原不就行了吗？这样一想，倒也是合理。但序列化 talbe 存在着两个问题：

1. table 多数情况下是无法被存满的，序列化未使用的部分，浪费空间
2. 同一个键值对在不同 JVM 下，所处的桶位置可能是不同的，在不同的 JVM 下反序列化 table 可能会发生错误。

以上两个问题中，第一个问题比较好理解，第二个问题解释一下。HashMap 的`get/put/remove`等方法第一步就是根据 hash 找到键所在的桶位置，但如果键没有覆写 hashCode 方法，计算 hash 时最终调用 Object 中的 hashCode 方法。但 Object 中的 hashCode 方法是 native 型的，不同的 JVM 下，可能会有不同的实现，产生的 hash 可能也是不一样的。也就是说同一个键在不同平台下可能会产生不同的 hash，此时再对在同一个 table 继续操作，就会出现问题。

综上所述，大家应该能明白 HashMap 不序列化 table 的原因了。



### 参考

1. [HashMap 源码详细分析(JDK1.8)](https://segmentfault.com/a/1190000012926722)
2. [HashMap分析之红黑树树化过程](https://www.cnblogs.com/finite/p/8251587.html)
3. [JDK 源码中 HashMap 的 hash 方法原理是什么？- 知乎](https://www.zhihu.com/question/20733617)
4. [Java 8系列之重新认识HashMap - 美团技术博客](https://tech.meituan.com/java-hashmap.html)
5. [python内置的hash函数对于字符串来说，每次得到的值不一样？- 知乎](https://www.zhihu.com/question/57526436/answer/153262129)
6. [Java中HashMap关键字transient的疑惑 - segmentFault](https://segmentfault.com/q/1010000000630486)