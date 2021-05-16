---
title: 线程安全集合类分析 ConcurrentSkipListMap、ConcurrentHashMap（一）
date: 2020-1-20 20:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

### ConcurrentSkipListMap

跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。

<!--more-->

在4线程1.6万数据的条件下，ConcurrentHashMap 存取速度是ConcurrentSkipListMap 的4倍左右。

但ConcurrentSkipListMap有几个ConcurrentHashMap 不能比拟的优点：

1. ConcurrentSkipListMap 的key是有序的。
2. ConcurrentSkipListMap 支持更高的并发。ConcurrentSkipListMap 的存取时间是log（N），和线程数几乎无关。也就是说在数据量一定的情况下，并发的线程越多，ConcurrentSkipListMap越能体现出他的优势。

在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。
所以在多线程程序中，如果需要对Map的键值进行排序时，请尽量使用ConcurrentSkipListMap，可能得到更好的并发度。
注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作。

#### 主要内部类

内部类跟存储结构结合着来看，大概能预测到代码的组织方式。

```java
// 数据节点，典型的单链表结构
static final class Node<K,V> {
    final K key;
    // 注意：这里value的类型是Object，而不是V
    // 在删除元素的时候value会指向当前元素本身
    volatile Object value;
    volatile Node<K,V> next;
    
    Node(K key, Object value, Node<K,V> next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }
    
    Node(Node<K,V> next) {
        this.key = null;
        this.value = this; // 当前元素本身(marker)
        this.next = next;
    }
}

// 索引节点，存储着对应的node值，及向下和向右的索引指针
static class Index<K,V> {
    final Node<K,V> node;
    final Index<K,V> down;
    volatile Index<K,V> right;
    
    Index(Node<K,V> node, Index<K,V> down, Index<K,V> right) {
        this.node = node;
        this.down = down;
        this.right = right;
    }
}

// 头索引节点，继承自Index，并扩展一个level字段，用于记录索引的层级
static final class HeadIndex<K,V> extends Index<K,V> {
    final int level;
    
    HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
        super(node, down, right);
        this.level = level;
    }
}
```

#### 构造方法

```java
public ConcurrentSkipListMap() {
    this.comparator = null;
    initialize();
}

public ConcurrentSkipListMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
    initialize();
}

public ConcurrentSkipListMap(Map<? extends K, ? extends V> m) {
    this.comparator = null;
    initialize();
    putAll(m);
}

public ConcurrentSkipListMap(SortedMap<K, ? extends V> m) {
    this.comparator = m.comparator();
    initialize();
    buildFromSorted(m);
}
```

四个构造方法里面都调用了initialize()这个方法，那么，这个方法里面有什么呢？

```java
private static final Object BASE_HEADER = new Object();

private void initialize() {
    keySet = null;
    entrySet = null;
    values = null;
    descendingMap = null;
    // Node(K key, Object value, Node<K,V> next)
    // HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level)
    head = new HeadIndex<K,V>(new Node<K,V>(null, BASE_HEADER, null),
                              null, null, 1);
}
```

可以看到，这里初始化了一些属性，并创建了一个头索引节点，里面存储着一个数据节点，这个数据节点的值是空对象，且它的层级是1。

所以，初始化的时候，跳表中只有一个头索引节点，层级是1，数据节点是一个空对象，down和right都是null。

![1FFUfK.png](https://s2.ax1x.com/2020/01/20/1FFUfK.png)

#### 添加元素

跳表插入元素的时候会通过抛硬币的方式决定出它需要的层级，然后找到各层链中它所在的位置，最后通过单链表插入的方式把节点及索引插入进去来实现的。

那么，**ConcurrentSkipListMap**中是这么做的吗？让我们一起来探个究竟：

```java
public V put(K key, V value) {
    // 不能存储value为null的元素
    // 因为value为null标记该元素被删除（后面会看到）
    if (value == null)
        throw new NullPointerException();

    // 调用doPut()方法添加元素
    return doPut(key, value, false);
}

private V doPut(K key, V value, boolean onlyIfAbsent) {
    // 添加元素后存储在z中
    Node<K,V> z;             // added node
    // key也不能为null
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;

    // Part I：找到目标节点的位置并插入
    // 这里的目标节点是数据节点，也就是最底层的那条链
    // 自旋
    outer: for (;;) {
        // 寻找目标节点之前最近的一个索引对应的数据节点，存储在b中，b=before
        // 并把b的下一个数据节点存储在n中，n=next
        // 为了便于描述，我这里把b叫做当前节点，n叫做下一个节点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            // 如果下一个节点不为空
            // 就拿其key与目标节点的key比较，找到目标节点应该插入的位置
            if (n != null) {
                // v=value，存储节点value值
                // c=compare，存储两个节点比较的大小
                Object v; int c;
                // n的下一个数据节点，也就是b的下一个节点的下一个节点（孙子节点）
                Node<K,V> f = n.next;
                // 如果n不为b的下一个节点
                // 说明有其它线程修改了数据，则跳出内层循环
                // 也就是回到了外层循环自旋的位置，从头来过
                if (n != b.next)               // inconsistent read
                    break;
                // 如果n的value值为空，说明该节点已删除，协助删除节点
                if ((v = n.value) == null) {   // n is deleted
                    // todo 这里为啥会协助删除？后面讲
                    n.helpDelete(b, f);
                    break;
                }
                // 如果b的值为空或者v等于n，说明b已被删除
                // 这时候n就是marker节点，那b就是被删除的那个
                if (b.value == null || v == n) // b is deleted
                    break;
                // 如果目标key与下一个节点的key大
                // 说明目标元素所在的位置还在下一个节点的后面
                if ((c = cpr(cmp, key, n.key)) > 0) {
                    // 就把当前节点往后移一位
                    // 同样的下一个节点也往后移一位
                    // 再重新检查新n是否为空，它与目标key的关系
                    b = n;
                    n = f;
                    continue;
                }
                // 如果比较时发现下一个节点的key与目标key相同
                // 说明链表中本身就存在目标节点
                if (c == 0) {
                    // 则用新值替换旧值，并返回旧值（onlyIfAbsent=false）
                    if (onlyIfAbsent || n.casValue(v, value)) {
                        @SuppressWarnings("unchecked") V vv = (V)v;
                        return vv;
                    }
                    // 如果替换旧值时失败，说明其它线程先一步修改了值，从头来过
                    break; // restart if lost race to replace value
                }
                // 如果c<0，就往下走，也就是找到了目标节点的位置
                // else c < 0; fall through
            }

            // 有两种情况会到这里
            // 一是到链表尾部了，也就是n为null了
            // 二是找到了目标节点的位置，也就是上面的c<0

            // 新建目标节点，并赋值给z
            // 这里把n作为新节点的next
            // 如果到链表尾部了，n为null，这毫无疑问
            // 如果c<0，则n的key比目标key大，相妆于在b和n之间插入目标节点z
            z = new Node<K,V>(key, value, n);
            // 原子更新b的下一个节点为目标节点z
            if (!b.casNext(n, z))
                // 如果更新失败，说明其它线程先一步修改了值，从头来过
                break;         // restart if lost race to append to b
            // 如果更新成功，跳出自旋状态
            break outer;
        }
    }

    // 经过Part I，目标节点已经插入到有序链表中了

    // Part II：随机决定是否需要建立索引及其层次，如果需要则建立自上而下的索引

    // 取个随机数
    int rnd = ThreadLocalRandom.nextSecondarySeed();
    // 0x80000001展开为二进制为10000000000000000000000000000001
    // 只有两头是1
    // 这里(rnd & 0x80000001) == 0
    // 相当于排除了负数（负数最高位是1），排除了奇数（奇数最低位是1）
    // 只有最高位最低位都不为1的数跟0x80000001做&操作才会为0
    // 也就是正偶数
    if ((rnd & 0x80000001) == 0) { // test highest and lowest bits
        // 默认level为1，也就是只要到这里了就会至少建立一层索引
        int level = 1, max;
        // 随机数从最低位的第二位开始，有几个连续的1则level就加几
        // 因为最低位肯定是0，正偶数嘛
        // 比如，1100110，level就加2
        while (((rnd >>>= 1) & 1) != 0)
            ++level;

        // 用于记录目标节点建立的最高的那层索引节点
        Index<K,V> idx = null;
        // 取头索引节点（这是最高层的头索引节点）
        HeadIndex<K,V> h = head;
        // 如果生成的层数小于等于当前最高层的层级
        // 也就是跳表的高度不会超过现有高度
        if (level <= (max = h.level)) {
            // 从第一层开始建立一条竖直的索引链表
            // 这条链表使用down指针连接起来
            // 每个索引节点里面都存储着目标节点这个数据节点
            // 最后idx存储的是这条索引链表的最高层节点
            for (int i = 1; i <= level; ++i)
                idx = new Index<K,V>(z, idx, null);
        }
        else { // try to grow by one level
            // 如果新的层数超过了现有跳表的高度
            // 则最多只增加一层
            // 比如现在只有一层索引，那下一次最多增加到两层索引，增加多了也没有意义
            level = max + 1; // hold in array and later pick the one to use
            // idxs用于存储目标节点建立的竖起索引的所有索引节点
            // 其实这里直接使用idx这个最高节点也是可以完成的
            // 只是用一个数组存储所有节点要方便一些
            // 注意，这里数组0号位是没有使用的
            @SuppressWarnings("unchecked")Index<K,V>[] idxs =
                    (Index<K,V>[])new Index<?,?>[level+1];
            // 从第一层开始建立一条竖的索引链表（跟上面一样，只是这里顺便把索引节点放到数组里面了）
            for (int i = 1; i <= level; ++i)
                idxs[i] = idx = new Index<K,V>(z, idx, null);

            // 自旋
            for (;;) {
                // 旧的最高层头索引节点
                h = head;
                // 旧的最高层级
                int oldLevel = h.level;
                // 再次检查，如果旧的最高层级已经不比新层级矮了
                // 说明有其它线程先一步修改了值，从头来过
                if (level <= oldLevel) // lost race to add level
                    break;
                // 新的最高层头索引节点
                HeadIndex<K,V> newh = h;
                // 头节点指向的数据节点
                Node<K,V> oldbase = h.node;
                // 超出的部分建立新的头索引节点
                for (int j = oldLevel+1; j <= level; ++j)
                    newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);
                // 原子更新头索引节点
                if (casHead(h, newh)) {
                    // h指向新的最高层头索引节点
                    h = newh;
                    // 把level赋值为旧的最高层级的
                    // idx指向的不是最高的索引节点了
                    // 而是与旧最高层平齐的索引节点
                    idx = idxs[level = oldLevel];
                    break;
                }
            }
        }

        // 经过上面的步骤，有两种情况
        // 一是没有超出高度，新建一条目标节点的索引节点链
        // 二是超出了高度，新建一条目标节点的索引节点链，同时最高层头索引节点同样往上长

        // Part III：将新建的索引节点（包含头索引节点）与其它索引节点通过右指针连接在一起

        // 这时level是等于旧的最高层级的，自旋
        splice: for (int insertionLevel = level;;) {
            // h为最高头索引节点
            int j = h.level;

            // 从头索引节点开始遍历
            // 为了方便，这里叫q为当前节点，r为右节点，d为下节点，t为目标节点相应层级的索引
            for (Index<K,V> q = h, r = q.right, t = idx;;) {
                // 如果遍历到了最右边，或者最下边，
                // 也就是遍历到头了，则退出外层循环
                if (q == null || t == null)
                    break splice;
                // 如果右节点不为空
                if (r != null) {
                    // n是右节点的数据节点，为了方便，这里直接叫右节点的值
                    Node<K,V> n = r.node;
                    // 比较目标key与右节点的值
                    int c = cpr(cmp, key, n.key);
                    // 如果右节点的值为空了，则表示此节点已删除
                    if (n.value == null) {
                        // 则把右节点删除
                        if (!q.unlink(r))
                            // 如果删除失败，说明有其它线程先一步修改了，从头来过
                            break;
                        // 删除成功后重新取右节点
                        r = q.right;
                        continue;
                    }
                    // 如果比较c>0，表示目标节点还要往右
                    if (c > 0) {
                        // 则把当前节点和右节点分别右移
                        q = r;
                        r = r.right;
                        continue;
                    }
                }

                // 到这里说明已经到当前层级的最右边了
                // 这里实际是会先走第二个if

                // 第一个if
                // j与insertionLevel相等了
                // 实际是先走的第二个if，j自减后应该与insertionLevel相等
                if (j == insertionLevel) {
                    // 这里是真正连右指针的地方
                    if (!q.link(r, t))
                        // 连接失败，从头来过
                        break; // restart
                    // t节点的值为空，可能是其它线程删除了这个元素
                    if (t.node.value == null) {
                        // 这里会去协助删除元素
                        findNode(key);
                        break splice;
                    }
                    // 当前层级右指针连接完毕，向下移一层继续连接
                    // 如果移到了最下面一层，则说明都连接完成了，退出外层循环
                    if (--insertionLevel == 0)
                        break splice;
                }

                // 第二个if
                // j先自减1，再与两个level比较
                // j、insertionLevel和t(idx)三者是对应的，都是还未把右指针连好的那个层级
                if (--j >= insertionLevel && j < level)
                    // t往下移
                    t = t.down;

                // 当前层级到最右边了
                // 那只能往下一层级去走了
                // 当前节点下移
                // 再取相应的右节点
                q = q.down;
                r = q.right;
            }
        }
    }
    return null;
}

// 寻找目标节点之前最近的一个索引对应的数据节点
private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
    // key不能为空
    if (key == null)
        throw new NullPointerException(); // don't postpone errors
    // 自旋
    for (;;) {
        // 从最高层头索引节点开始查找，先向右，再向下
        // 直到找到目标位置之前的那个索引
        for (Index<K,V> q = head, r = q.right, d;;) {
            // 如果右节点不为空
            if (r != null) {
                // 右节点对应的数据节点，为了方便，我们叫右节点的值
                Node<K,V> n = r.node;
                K k = n.key;
                // 如果右节点的value为空
                // 说明其它线程把这个节点标记为删除了
                // 则协助删除
                if (n.value == null) {
                    if (!q.unlink(r))
                        // 如果删除失败
                        // 说明其它线程先删除了，从头来过
                        break;           // restart
                    // 删除之后重新读取右节点
                    r = q.right;         // reread r
                    continue;
                }
                // 如果目标key比右节点还大，继续向右寻找
                if (cpr(cmp, key, k) > 0) {
                    // 往右移
                    q = r;
                    // 重新取右节点
                    r = r.right;
                    continue;
                }
                // 如果c<0，说明不能再往右了
            }
            // 到这里说明当前层级已经到最右了
            // 两种情况：一是r==null，二是c<0
            // 再从下一级开始找

            // 如果没有下一级了，就返回这个索引对应的数据节点
            if ((d = q.down) == null)
                return q.node;

            // 往下移
            q = d;
            // 重新取右节点
            r = d.right;
        }
    }
}

// Node.class中的方法，协助删除元素
void helpDelete(Node<K,V> b, Node<K,V> f) {
    /*
     * Rechecking links and then doing only one of the
     * help-out stages per call tends to minimize CAS
     * interference among helping threads.
     */
    // 这里的调用者this==n，三者关系是b->n->f
    if (f == next && this == b.next) {
        // 将n的值设置为null后，会先把n的下个节点设置为marker节点
        // 这个marker节点的值是它自己
        // 这里如果不是它自己说明marker失败了，重新marker
        if (f == null || f.value != f) // not already marked
            casNext(f, new Node<K,V>(f));
        else
            // marker过了，就把b的下个节点指向marker的下个节点
            b.casNext(this, f.next);
    }
}

// Index.class中的方法，删除succ节点
final boolean unlink(Index<K,V> succ) {
    // 原子更新当前节点指向下一个节点的下一个节点
    // 也就是删除下一个节点
    return node.value != null && casRight(succ, succ.right);
}

// Index.class中的方法，在当前节点与succ之间插入newSucc节点
final boolean link(Index<K,V> succ, Index<K,V> newSucc) {
    // 在当前节点与下一个节点中间插入一个节点
    Node<K,V> n = node;
    // 新节点指向当前节点的下一个节点
    newSucc.right = succ;
    // 原子更新当前节点的下一个节点指向新节点
    return n.value != null && casRight(succ, newSucc);
}
```

我们这里把整个插入过程分成三个部分：

1. Part I：找到目标节点的位置并插入
   1. 这里的目标节点是数据节点，也就是最底层的那条链；
   2. 寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；
   3. 从这个数据节点开始往后遍历，直到找到目标节点应该插入的位置；
   4. 如果这个位置有元素，就更新其值（onlyIfAbsent=false）；
   5. 如果这个位置没有元素，就把目标节点插入；
   6. 至此，目标节点已经插入到最底层的数据节点链表中了；

2. Part II：随机决定是否需要建立索引及其层次，如果需要则建立自上而下的索引
   1. 取个随机数rnd，计算(rnd & 0x80000001)；
   2. 如果不等于0，结束插入过程，也就是不需要创建索引，返回；
   3. 如果等于0，才进入创建索引的过程（只要正偶数才会等于0）
   4. 计算`while (((rnd >>>= 1) & 1) != 0)`，决定层级数，level从1开始；
   5. 如果算出来的层级不高于现有最高层级，则直接建立一条竖直的索引链表（只有down有值），并结束Part II；
   6. 如果算出来的层级高于现有最高层级，则新的层级只能比现有最高层级多1；
   7. 同样建立一条竖直的索引链表（只有down有值）；
   8. 将头索引也向上增加到相应的高度，结束Part II；
   9. 也就是说，如果层级不超过现有高度，只建立一条索引链，否则还要额外增加头索引链的高度（脑补一下，后面举例说明）；

3. Part III：将新建的索引节点（包含头索引节点）与其它索引节点通过右指针连接在一起（补上right指针）
   1. 从最高层级的头索引节点开始，向右遍历，找到目标索引节点的位置；
   2. 如果当前层有目标索引，则把目标索引插入到这个位置，并把目标索引前一个索引向下移一个层级；
   3. 如果当前层没有目标索引，则把目标索引位置前一个索引向下移一个层级
   4. 同样地，再向右遍历，寻找新的层级中目标索引的位置，回到第（2）步；
   5. 依次循环找到所有层级目标索引的位置并把它们插入到横向的索引链表中；

总结起来，一共就是三大步：

1. 插入目标节点到数据节点链表中；
2. 建立竖直的down链表；
3. 建立横向的right链表；

#### 删除元素

删除元素，就是把各层级中对应的元素删除即可，真的这么简单吗？来让我们上代码：

```java
public V remove(Object key) {
    return doRemove(key, null);
}

final V doRemove(Object key, Object value) {
    // key不为空
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    // 自旋
    outer: for (;;) {
        // 寻找目标节点之前的最近的索引节点对应的数据节点
        // 为了方便，这里叫b为当前节点，n为下一个节点，f为下下个节点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            // 整个链表都遍历完了也没找到目标节点，退出外层循环
            if (n == null)
                break outer;
            // 下下个节点
            Node<K,V> f = n.next;
            // 再次检查
            // 如果n不是b的下一个节点了
            // 说明有其它线程先一步修改了，从头来过
            if (n != b.next)                    // inconsistent read
                break;
            // 如果下个节点的值奕为null了
            // 说明有其它线程标记该元素为删除状态了
            if ((v = n.value) == null) {        // n is deleted
                // 协助删除
                n.helpDelete(b, f);
                break;
            }
            // 如果b的值为空或者v等于n，说明b已被删除
            // 这时候n就是marker节点，那b就是被删除的那个
            if (b.value == null || v == n)      // b is deleted
                break;
            // 如果c<0，说明没找到元素，退出外层循环
            if ((c = cpr(cmp, key, n.key)) < 0)
                break outer;
            // 如果c>0，说明还没找到，继续向右找
            if (c > 0) {
                // 当前节点往后移
                b = n;
                // 下一个节点往后移
                n = f;
                continue;
            }
            // c=0，说明n就是要找的元素
            // 如果value不为空且不等于找到元素的value，不需要删除，退出外层循环
            if (value != null && !value.equals(v))
                break outer;
            // 如果value为空，或者相等
            // 原子标记n的value值为空
            if (!n.casValue(v, null))
                // 如果删除失败，说明其它线程先一步修改了，从头来过
                break;

            // P.S.到了这里n的值肯定是设置成null了

            // 关键！！！！
            // 让n的下一个节点指向一个market节点
            // 这个market节点的key为null，value为marker自己，next为n的下个节点f
            // 或者让b的下一个节点指向下下个节点
            // 注意：这里是或者||，因为两个CAS不能保证都成功，只能一个一个去尝试
            // 这里有两层意思：
            // 一是如果标记market成功，再尝试将b的下个节点指向下下个节点，如果第二步失败了，进入条件，如果成功了就不用进入条件了
            // 二是如果标记market失败了，直接进入条件
            if (!n.appendMarker(f) || !b.casNext(n, f))
                // 通过findNode()重试删除（里面有个helpDelete()方法）
                findNode(key);                  // retry via findNode
            else {
                // 上面两步操作都成功了，才会进入这里，不太好理解，上面两个条件都有非"!"操作
                // 说明节点已经删除了，通过findPredecessor()方法删除索引节点
                // findPredecessor()里面有unlink()操作
                findPredecessor(key, cmp);      // clean index
                // 如果最高层头索引节点没有右节点，则跳表的高度降级
                if (head.right == null)
                    tryReduceLevel();
            }
            // 返回删除的元素值
            @SuppressWarnings("unchecked") V vv = (V)v;
            return vv;
        }
    }
    return null;
}
```

1. 寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；
2. 从这个数据节点开始往后遍历，直到找到目标节点的位置；
3. 如果这个位置没有元素，直接返回null，表示没有要删除的元素；
4. 如果这个位置有元素，先通过`n.casValue(v, null)`原子更新把其value设置为null；
5. 通过`n.appendMarker(f)`在当前元素后面添加一个marker元素标记当前元素是要删除的元素；
6. 通过`b.casNext(n, f)`尝试删除元素；
7. 如果上面两步中的任意一步失败了都通过`findNode(key)`中的`n.helpDelete(b, f)`再去不断尝试删除；
8. 如果上面两步都成功了，再通过`findPredecessor(key, cmp)`中的`q.unlink(r)`删除索引节点；
9. 如果head的right指针指向了null，则跳表高度降级；

#### 查找元素

经过上面的插入和删除，查找元素就比较简单了，直接上代码：

```java
public V get(Object key) {
    return doGet(key);
}

private V doGet(Object key) {
    // key不为空
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    // 自旋
    outer: for (;;) {
        // 寻找目标节点之前最近的索引对应的数据节点
        // 为了方便，这里叫b为当前节点，n为下个节点，f为下下个节点
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            // 如果链表到头还没找到元素，则跳出外层循环
            if (n == null)
                break outer;
            // 下下个节点
            Node<K,V> f = n.next;
            // 如果不一致读，从头来过
            if (n != b.next)                // inconsistent read
                break;
            // 如果n的值为空，说明节点已被其它线程标记为删除
            if ((v = n.value) == null) {    // n is deleted
                // 协助删除，再重试
                n.helpDelete(b, f);
                break;
            }
            // 如果b的值为空或者v等于n，说明b已被删除
            // 这时候n就是marker节点，那b就是被删除的那个
            if (b.value == null || v == n)  // b is deleted
                break;
            // 如果c==0，说明找到了元素，就返回元素值
            if ((c = cpr(cmp, key, n.key)) == 0) {
                @SuppressWarnings("unchecked") V vv = (V)v;
                return vv;
            }
            // 如果c<0，说明没找到元素
            if (c < 0)
                break outer;
            // 如果c>0，说明还没找到，继续寻找
            // 当前节点往后移
            b = n;
            // 下一个节点往后移
            n = f;
        }
    }
    return null;
}
```

1. 寻找目标节点之前最近的一个索引对应的数据节点（数据节点都是在最底层的链表上）；
2. 从这个数据节点开始往后遍历，直到找到目标节点的位置；
3. 如果这个位置没有元素，直接返回null，表示没有找到元素；
4. 如果这个位置有元素，返回元素的value值；

### ConcurrentHashMap

ConcurrentHashMap是HashMap的线程安全版本，内部也是使用（数组 + 链表 + 红黑树）的结构来存储元素。

相比于同样线程安全的HashTable来说，效率等各方面都有极大地提高。

1. ConcurrentHashMap与HashMap的数据结构是否一样？
2. HashMap在多线程环境下何时会出现并发安全问题？
3. ConcurrentHashMap是怎么解决并发安全问题的？
4. ConcurrentHashMap使用了哪些锁？
5. ConcurrentHashMap的扩容是怎么进行的？
6. ConcurrentHashMap是否是强一致性的？
7. ConcurrentHashMap不能解决哪些问题？
8. ConcurrentHashMap中有哪些不常见的技术值得学习？

#### 原理

在ConcurrentHashMap中通过一个Node<K,V>[]数组来保存添加到map中的键值对，而在同一个数组位置是通过链表和红黑树的形式来保存的。但是这个数组只有在第一次添加元素的时候才会初始化，否则只是初始化一个ConcurrentHashMap对象的话，只是设定了一个sizeCtl变量，这个变量用来判断对象的一些状态和是否需要扩容，后面会详细解释。

第一次添加元素的时候，默认初期长度为16，当往map中继续添加元素的时候，通过hash值跟数组长度取与来决定放在数组的哪个位置，如果出现放在同一个位置的时候，优先以链表的形式存放，在同一个位置的个数又达到了8个以上，如果数组的长度还小于64的时候，则会扩容数组。如果数组的长度大于等于64了的话，在会将该节点的链表转换成树。

通过扩容数组的方式来把这些节点给分散开。然后将这些元素复制到扩容后的新的数组中，同一个链表中的元素通过hash值的数组长度位来区分，是还是放在原来的位置还是放到扩容的长度的相同位置去 。在扩容完成之后，如果某个节点的是树，同时现在该节点的个数又小于等于6个了，则会将该树转为链表。

取元素的时候，相对来说比较简单，通过计算hash来确定该元素在数组的哪个位置，然后在通过遍历链表或树来判断key和key的hash，取出value值。

往ConcurrentHashMap中添加元素的时候，里面的数据以数组的形式存放的样子大概是这样的：

![1iLTGn.png](https://s2.ax1x.com/2020/01/20/1iLTGn.png)

这个时候因为数组的长度才为16，则不会转化为树，而是会进行扩容。

扩容后数组大概是这样的：

![1iL72q.png](https://s2.ax1x.com/2020/01/20/1iL72q.png)

需要注意的是，扩容之后的长度不是32，扩容后的长度在后面细说。

如果数组扩张后长度达到64了，且继续在某个节点的后面添加元素达到8个以上的时候，则会出现转化为红黑树的情况。

转化之后大概是这样：

![1iLHx0.png](https://s2.ax1x.com/2020/01/20/1iLHx0.png)

#### 各种锁简介

这里先简单介绍一下各种锁，以便下文讲到相关概念时能有个印象。

1. synchronized

   java中的关键字，内部实现为监视器锁，主要是通过对象监视器在对象头中的字段来表明的。

   synchronized从旧版本到现在已经做了很多优化了，在运行时会有三种存在方式：偏向锁，轻量级锁，重量级锁。

   偏向锁，是指一段同步代码一直被一个线程访问，那么这个线程会自动获取锁，降低获取锁的代价。

   轻量级锁，是指当锁是偏向锁时，被另一个线程所访问，偏向锁会升级为轻量级锁，这个线程会通过自旋的方式尝试获取锁，不会阻塞，提高性能。

   重量级锁，是指当锁是轻量级锁时，当自旋的线程自旋了一定的次数后，还没有获取到锁，就会进入阻塞状态，该锁升级为重量级锁，重量级锁会使其他线程阻塞，性能降低。

2. CAS

   CAS，Compare And Swap，它是一种乐观锁，认为对于同一个数据的并发操作不一定会发生修改，在更新数据的时候，尝试去更新数据，如果失败就不断尝试。

3. volatile（非锁）

   java中的关键字，当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。（这里牵涉到java内存模型的知识，感兴趣的同学可以自己查查相关资料）

   volatile只保证可见性，不保证原子性，比如 volatile修改的变量 i，针对i++操作，不保证每次结果都正确，因为i++操作是两步操作，相当于 i = i +1，先读取，再加1，这种情况volatile是无法保证的。

4. 自旋锁

   自旋锁，是指尝试获取锁的线程不会阻塞，而是循环的方式不断尝试，这样的好处是减少线程的上下文切换带来的开锁，提高性能，缺点是循环会消耗CPU。

5. 分段锁

   分段锁，是一种锁的设计思路，它细化了锁的粒度，主要运用在ConcurrentHashMap中，实现高效的并发操作，当操作不需要更新整个数组时，就只锁数组中的一项就可以了。

6. ReentrantLock

   可重入锁，是指一个线程获取锁之后再尝试获取锁时会自动获取锁，可重入锁的优点是避免死锁。

   其实，synchronized也是可重入锁。

#### 属性

下面是几个重要的属性

```java
private static final int MAXIMUM_CAPACITY = 1 << 30;
private static final int DEFAULT_CAPACITY = 16;
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;
static final int MOVED     = -1; // 表示正在转移
static final int TREEBIN   = -2; // 表示已经转换成树
static final int RESERVED  = -3; // hash for transient reservations
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
transient volatile Node<K,V>[] table;//默认没初始化的数组，用来保存元素
private transient volatile Node<K,V>[] nextTable;//转移的时候用的数组
/**
     * 用来控制表初始化和扩容的，默认值为0，当在初始化的时候指定了大小，这会将这个大小保存在sizeCtl中，大小为数组的0.75
     * 当为负的时候，说明表正在初始化或扩张，
     *     -1表示初始化
     *     -(1+n) n:表示活动的扩张线程
     */
    private transient volatile int sizeCtl;
```

**几个重要的类**

```java
//Node<K,V>,这是构成每个元素的基本类。
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //key的hash值
        final K key;       //key
        volatile V val;    //value
        volatile Node<K,V> next; //表示链表中的下一个节点

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    }
// TreeNode，构造树的节点
    static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next,
                 TreeNode<K,V> parent) {
            super(hash, key, val, next);
            this.parent = parent;
        }
}
//TreeBin 用作树的头结点，只存储root和first节点，不存储节点的key、value值。
static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
}
//ForwardingNode在转移的时候放在头部的节点，是一个空节点
static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }
}
```

#### 初始化

```java
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
            MAXIMUM_CAPACITY :
            tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}

public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}

public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

构造方法与HashMap对比可以发现，没有了HashMap中的threshold和loadFactor，而是改用了sizeCtl来控制，而且只存储了容量在里面，那么它是怎么用的呢？官方给出的解释如下：

1. -1，表示有线程正在进行初始化操作
2. -(1 + nThreads)，表示有n个线程正在一起扩容
3. 0，默认值，后续在真正初始化的时候使用默认容量
4. 0，初始化或扩容完成后下一次的扩容门槛

至于，官方这个解释对不对我们后面再讨论。

#### 重要方法

在ConcurrentHashMap中使用了unSafe方法，通过直接操作内存的方式来保证并发处理的安全性，使用的是硬件的安全机制。

```java
/*
     * 用来返回节点数组的指定位置的节点的原子操作
     */
    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    /*
     * cas原子操作，在指定位置设定值
     */
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
    /*
     * 原子操作，在指定位置设定值
     */
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

#### 添加元素

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key和value都不能为null
    if (key == null || value == null) throw new NullPointerException();
    // 计算hash值
    int hash = spread(key.hashCode());
    // 要插入的元素所在桶的元素个数
    int binCount = 0;
    // 死循环，结合CAS使用（如果CAS失败，则会重新取整个桶进行下面的流程）
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            // 如果桶未初始化或者桶个数为0，则初始化桶
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 如果要插入的元素所在的桶还没有元素，则把这个元素插入到这个桶中
            if (casTabAt(tab, i, null,
                    new Node<K,V>(hash, key, value, null)))
                // 如果使用CAS插入元素时，发现已经有元素了，则进入下一次循环，重新操作
                // 如果使用CAS插入元素成功，则break跳出循环，流程结束
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            // 如果要插入的元素所在的桶的第一个元素的hash是MOVED，则当前线程帮忙一起迁移元素
            tab = helpTransfer(tab, f);
        else {
            // 如果这个桶不为空且不在迁移元素，则锁住这个桶（分段锁）
            // 并查找要插入的元素是否在这个桶中
            // 存在，则替换值（onlyIfAbsent=false）
            // 不存在，则插入到链表结尾或插入树中
            V oldVal = null;
            synchronized (f) {
                // 再次检测第一个元素是否有变化，如果有变化则进入下一次循环，从头来过
                if (tabAt(tab, i) == f) {
                    // 如果第一个元素的hash值大于等于0（说明不是在迁移，也不是树）
                    // 那就是桶中的元素使用的是链表方式存储
                    if (fh >= 0) {
                        // 桶中元素个数赋值为1
                        binCount = 1;
                        // 遍历整个桶，每次结束binCount加1
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                            (ek != null && key.equals(ek)))) {
                                // 如果找到了这个元素，则赋值了新值（onlyIfAbsent=false）
                                // 并退出循环
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                // 如果到链表尾部还没有找到元素
                                // 就把它插入到链表结尾并退出循环
                                pred.next = new Node<K,V>(hash, key,
                                        value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        // 如果第一个元素是树节点
                        Node<K,V> p;
                        // 桶中元素个数赋值为2
                        binCount = 2;
                        // 调用红黑树的插入方法插入元素
                        // 如果成功插入则返回null
                        // 否则返回寻找到的节点
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                value)) != null) {
                            // 如果找到了这个元素，则赋值了新值（onlyIfAbsent=false）
                            // 并退出循环
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // 如果binCount不为0，说明成功插入了元素或者寻找到了元素
            if (binCount != 0) {
                // 如果链表元素个数达到了8，则尝试树化
                // 因为上面把元素插入到树中时，binCount只赋值了2，并没有计算整个树中元素的个数
                // 所以不会重复树化
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                // 如果要插入的元素已经存在，则返回旧值
                if (oldVal != null)
                    return oldVal;
                // 退出外层大循环，流程结束
                break;
            }
        }
        }
        // 成功插入元素，元素个数加1（是否要扩容在这个里面）
        addCount(1L, binCount);
        // 成功插入元素返回null
        return null;
    }
```

整体流程跟HashMap比较类似，大致是以下几步：

1. 如果桶数组未初始化，则初始化；
2. 如果待插入的元素所在的桶为空，则尝试把此元素直接插入到桶的第一个位置；
3. 如果正在扩容，则当前线程一起加入到扩容的过程中；
4. 如果待插入的元素所在的桶不为空且不在迁移元素，则锁住这个桶（分段锁）；
5. 如果当前桶中元素以链表方式存储，则在链表中寻找该元素或者插入元素；
6. 如果当前桶中元素以红黑树方式存储，则在红黑树中寻找该元素或者插入元素；
7. 如果元素存在，则返回旧值；
8. 如果元素不存在，整个Map的元素个数加1，并检查是否需要扩容；

添加元素操作中使用的锁主要有（自旋锁 + CAS + synchronized + 分段锁）。

为什么使用synchronized而不是ReentrantLock？

因为synchronized已经得到了极大地优化，在特定情况下并不比ReentrantLock差。

#### 初始化桶数组

第一次放元素时，初始化桶数组。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            // 如果sizeCtl<0说明正在初始化或者扩容，让出CPU
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            // 如果把sizeCtl原子更新为-1成功，则当前线程进入初始化
            // 如果原子更新失败则说明有其它线程先一步进入初始化了，则进入下一次循环
            // 如果下一次循环时还没初始化完毕，则sizeCtl<0进入上面if的逻辑让出CPU
            // 如果下一次循环更新完毕了，则table.length!=0，退出循环
            try {
                // 再次检查table是否为空，防止ABA问题
                if ((tab = table) == null || tab.length == 0) {
                    // 如果sc为0则使用默认值16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    // 新建数组
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // 赋值给table桶数组
                    table = tab = nt;
                    // 设置sc为数组长度的0.75倍
                    // n - (n >>> 2) = n - n/4 = 0.75n
                    // 可见这里装载因子和扩容门槛都是写死了的
                    // 这也正是没有threshold和loadFactor属性的原因
                    sc = n - (n >>> 2);
                }
            } finally {
                // 把sc赋值给sizeCtl，这时存储的是扩容门槛
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

1. 使用CAS锁控制只有一个线程初始化桶数组；
2. sizeCtl在初始化后存储的是扩容门槛；
3. 扩容门槛写死的是桶数组大小的0.75倍，桶数组大小即map的容量，也就是最多存储多少个元素。

#### 判断是否需要扩容

每次添加元素后，元素数量加1，并判断是否达到扩容门槛，达到了则进行扩容或协助扩容。

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // 这里使用的思想跟LongAdder类是一模一样的（后面会讲）
    // 把数组的大小存储根据不同的线程存储到不同的段上（也是分段锁的思想）
    // 并且有一个baseCount，优先更新baseCount，如果失败了再更新不同线程对应的段
    // 这样可以保证尽量小的减少冲突

    // 先尝试把数量加到baseCount上，如果失败再加到分段的CounterCell上
    if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        // 如果as为空
        // 或者长度为0
        // 或者当前线程所在的段为null
        // 或者在当前线程的段上加数量失败
        if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                        U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            // 强制增加数量（无论如何数量是一定要加上的，并不是简单地自旋）
            // 不同线程对应不同的段都更新失败了
            // 说明已经发生冲突了，那么就对counterCells进行扩容
            // 以减少多个线程hash到同一个段的概率
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        // 计算元素个数
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        // 如果元素个数达到了扩容门槛，则进行扩容
        // 注意，正常情况下sizeCtl存储的是扩容门槛，即容量的0.75倍
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                (n = tab.length) < MAXIMUM_CAPACITY) {
            // rs是扩容时的一个邮戳标识
            int rs = resizeStamp(n);
            if (sc < 0) {
                // sc<0说明正在扩容中
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                    // 扩容已经完成了，退出循环
                    // 正常应该只会触发nextTable==null这个条件，其它条件没看出来何时触发
                    break;

                // 扩容未完成，则当前线程加入迁移元素中
                // 并把扩容线程数加1
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                    (rs << RESIZE_STAMP_SHIFT) + 2))
                // 这里是触发扩容的那个线程进入的地方
                // sizeCtl的高16位存储着rs这个扩容邮戳
                // sizeCtl的低16位存储着扩容线程数加1，即(1+nThreads)
                // 所以官方说的扩容时sizeCtl的值为 -(1+nThreads)是错误的

                // 进入迁移元素
                transfer(tab, null);
            // 重新计算元素个数
            s = sumCount();
        }
    }
}
```

1. 元素个数的存储方式类似于LongAdder类，存储在不同的段上，减少不同线程同时更新size时的冲突；
2. 计算元素个数时把这些段的值及baseCount相加算出总的元素个数；
3. 正常情况下sizeCtl存储着扩容门槛，扩容门槛为容量的0.75倍；
4. 扩容时sizeCtl高位存储扩容邮戳(resizeStamp)，低位存储扩容线程数加1（1+nThreads）；
5. 其它线程添加元素后如果发现存在扩容，也会加入的扩容行列中来；

#### 协助扩容（迁移元素）

线程添加元素时发现正在扩容且当前元素所在的桶元素已经迁移完成了，则协助迁移其它桶的元素。

```java
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    // 如果桶数组不为空，并且当前桶第一个元素为ForwardingNode类型，并且nextTab不为空
    // 说明当前桶已经迁移完毕了，才去帮忙迁移其它桶的元素
    // 扩容时会把旧桶的第一个元素置为ForwardingNode，并让其nextTab指向新桶数组
    if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
        int rs = resizeStamp(tab.length);
        // sizeCtl<0，说明正在扩容
        while (nextTab == nextTable && table == tab &&
                (sc = sizeCtl) < 0) {
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            // 扩容线程数加1
            if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                // 当前线程帮忙迁移元素
                transfer(tab, nextTab);
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```

当前桶元素迁移完成了才去协助迁移其它桶元素；

#### 迁移元素

扩容时容量变为两倍，并把部分元素迁移到其它桶中。

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        // 如果nextTab为空，说明还没开始迁移
        // 就新建一个新桶数组
        try {
            // 新桶数组是原桶的两倍
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    // 新桶数组大小
    int nextn = nextTab.length;
    // 新建一个ForwardingNode类型的节点，并把新桶数组存储在里面
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 整个while循环就是在算i的值，过程太复杂，不用太关心
        // i的值会从n-1依次递减，感兴趣的可以打下断点就知道了
        // 其中n是旧桶数组的大小，也就是说i从15开始一直减到1这样去迁移元素
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                    (this, TRANSFERINDEX, nextIndex,
                            nextBound = (nextIndex > stride ?
                                    nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            // 如果一次遍历完成了
            // 也就是整个map所有桶中的元素都迁移完成了
            int sc;
            if (finishing) {
                // 如果全部迁移完成了，则替换旧桶数组
                // 并设置下一次扩容门槛为新桶数组容量的0.75倍
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                // 当前线程扩容完成，把扩容线程数-1
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    // 扩容完成两边肯定相等
                    return;
                // 把finishing设置为true
                // finishing为true才会走到上面的if条件
                finishing = advance = true;
                // i重新赋值为n
                // 这样会再重新遍历一次桶数组，看看是不是都迁移完成了
                // 也就是第二次遍历都会走到下面的(fh = f.hash) == MOVED这个条件
                i = n; // recheck before commit
            }
        }
        else if ((f = tabAt(tab, i)) == null)
            // 如果桶中无数据，直接放入ForwardingNode标记该桶已迁移
            advance = casTabAt(tab, i, null, fwd);
        else if ((fh = f.hash) == MOVED)
            // 如果桶中第一个元素的hash值为MOVED
            // 说明它是ForwardingNode节点
            // 也就是该桶已迁移
            advance = true; // already processed
        else {
            // 锁定该桶并迁移元素
            synchronized (f) {
                // 再次判断当前桶第一个元素是否有修改
                // 也就是可能其它线程先一步迁移了元素
                if (tabAt(tab, i) == f) {
                    // 把一个链表分化成两个链表
                    // 规则是桶中各元素的hash与桶大小n进行与操作
                    // 等于0的放到低位链表(low)中，不等于0的放到高位链表(high)中
                    // 其中低位链表迁移到新桶中的位置相对旧桶不变
                    // 高位链表迁移到新桶中位置正好是其在旧桶的位置加n
                    // 这也正是为什么扩容时容量在变成两倍的原因
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        // 第一个元素的hash值大于等于0
                        // 说明该桶中元素是以链表形式存储的
                        // 这里与HashMap迁移算法基本类似
                        // 唯一不同的是多了一步寻找lastRun
                        // 这里的lastRun是提取出链表后面不用处理再特殊处理的子链表
                        // 比如所有元素的hash值与桶大小n与操作后的值分别为 0 0 4 4 0 0 0
                        // 则最后后面三个0对应的元素肯定还是在同一个桶中
                        // 这时lastRun对应的就是倒数第三个节点
                        // 至于为啥要这样处理，我也没太搞明白
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        // 看看最后这几个元素归属于低位链表还是高位链表
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        // 遍历链表，把hash&n为0的放在低位链表中
                        // 不为0的放在高位链表中
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 低位链表的位置不变
                        setTabAt(nextTab, i, ln);
                        // 高位链表的位置是原位置加n
                        setTabAt(nextTab, i + n, hn);
                        // 标记当前桶已迁移
                        setTabAt(tab, i, fwd);
                        // advance为true，返回上面进行--i操作
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        // 如果第一个元素是树节点
                        // 也是一样，分化成两颗树
                        // 也是根据hash&n为0放在低位树中
                        // 不为0放在高位树中
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        // 遍历整颗树，根据hash&n是否为0分化成两颗树
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 如果分化的树中元素个数小于等于6，则退化成链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        // 低位树的位置不变
                        setTabAt(nextTab, i, ln);
                        // 高位树的位置是原位置加n
                        setTabAt(nextTab, i + n, hn);
                        // 标记该桶已迁移
                        setTabAt(tab, i, fwd);
                        // advance为true，返回上面进行--i操作
                        advance = true;
                    }
                }
            }
        }
    }
}
```

1. 新桶数组大小是旧桶数组的两倍；
2. 迁移元素先从靠后的桶开始；
3. 迁移完成的桶在里面放置一ForwardingNode类型的元素，标记该桶迁移完成；
4. 迁移时根据hash&n是否等于0把桶中元素分化成两个链表或树；
5. 低位链表（树）存储在原来的位置；
6. 高们链表（树）存储在原来的位置加n的位置；
7. 迁移元素时会锁住当前桶，也是分段锁的思想；

#### 删除元素

删除元素跟添加元素一样，都是先找到元素所在的桶，然后采用分段锁的思想锁住整个桶，再进行操作。

```java
public V remove(Object key) {
    // 调用替换节点方法
    return replaceNode(key, null, null);
}

final V replaceNode(Object key, V value, Object cv) {
    // 计算hash
    int hash = spread(key.hashCode());
    // 自旋
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
                (f = tabAt(tab, i = (n - 1) & hash)) == null)
            // 如果目标key所在的桶不存在，跳出循环返回null
            break;
        else if ((fh = f.hash) == MOVED)
            // 如果正在扩容中，协助扩容
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 标记是否处理过
            boolean validated = false;
            synchronized (f) {
                // 再次验证当前桶第一个元素是否被修改过
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        // fh>=0表示是链表节点
                        validated = true;
                        // 遍历链表寻找目标节点
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                            (ek != null && key.equals(ek)))) {
                                // 找到了目标节点
                                V ev = e.val;
                                // 检查目标节点旧value是否等于cv
                                if (cv == null || cv == ev ||
                                        (ev != null && cv.equals(ev))) {
                                    oldVal = ev;
                                    if (value != null)
                                        // 如果value不为空则替换旧值
                                        e.val = value;
                                    else if (pred != null)
                                        // 如果前置节点不为空
                                        // 删除当前节点
                                        pred.next = e.next;
                                    else
                                        // 如果前置节点为空
                                        // 说明是桶中第一个元素，删除之
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            // 遍历到链表尾部还没找到元素，跳出循环
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        // 如果是树节点
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        // 遍历树找到了目标节点
                        if ((r = t.root) != null &&
                                (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            // 检查目标节点旧value是否等于cv
                            if (cv == null || cv == pv ||
                                    (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    // 如果value不为空则替换旧值
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    // 如果value为空则删除元素
                                    // 如果删除后树的元素个数较少则退化成链表
                                    // t.removeTreeNode(p)这个方法返回true表示删除节点后树的元素个数较少
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            // 如果处理过，不管有没有找到元素都返回
            if (validated) {
                // 如果找到了元素，返回其旧值
                if (oldVal != null) {
                    // 如果要替换的值为空，元素个数减1
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    // 没找到元素返回空
    return null;
}
```

1. 计算hash；
2. 如果所在的桶不存在，表示没有找到目标元素，返回；
3. 如果正在扩容，则协助扩容完成后再进行删除操作；
4. 如果是以链表形式存储的，则遍历整个链表查找元素，找到之后再删除；
5. 如果是以树形式存储的，则遍历树查找元素，找到之后再删除；
6. 如果是以树形式存储的，删除元素之后树较小，则退化成链表；
7. 如果确实删除了元素，则整个map元素个数减1，并返回旧值；
8. 如果没有删除元素，则返回null；

#### 获取元素

获取元素，根据目标key所在桶的第一个元素的不同采用不同的方式获取元素，关键点在于find()方法的重写。

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算hash
    int h = spread(key.hashCode());
    // 如果元素所在的桶存在且里面有元素
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果第一个元素就是要找的元素，直接返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            // hash小于0，说明是树或者正在扩容
            // 使用find寻找元素，find的寻找方式依据Node的不同子类有不同的实现方式
            return (p = e.find(h, key)) != null ? p.val : null;

        // 遍历整个链表寻找元素
        while ((e = e.next) != null) {
            if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

1. hash到元素所在的桶；
2. 如果桶中第一个元素就是该找的元素，直接返回；
3. 如果是树或者正在迁移元素，则调用各自Node子类的find()方法寻找元素；
4. 如果是链表，遍历整个链表寻找元素；
5. 获取元素没有加锁；

#### 获取元素个数

元素个数的存储也是采用分段的思想，获取元素个数时需要把所有段加起来。

```java
public int size() {
    // 调用sumCount()计算元素个数
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                    (int)n);
}

final long sumCount() {
    // 计算CounterCell所有段及baseCount的数量之和
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

1. 元素的个数依据不同的线程存在在不同的段里；（见addCounter()分析）
2. 计算CounterCell所有段及baseCount的数量之和；
3. 获取元素个数没有加锁；

#### 总结

1. ConcurrentHashMap是HashMap的线程安全版本；
2. ConcurrentHashMap采用（数组 + 链表 + 红黑树）的结构存储元素；
3. ConcurrentHashMap相比于同样线程安全的HashTable，效率要高很多；
4. ConcurrentHashMap采用的锁有 synchronized，CAS，自旋锁，分段锁，volatile等；
5. ConcurrentHashMap中没有threshold和loadFactor这两个字段，而是采用sizeCtl来控制；
6. sizeCtl = -1，表示正在进行初始化；
7. sizeCtl = 0，默认值，表示后续在真正初始化的时候使用默认容量；
8. sizeCtl > 0，在初始化之前存储的是传入的容量，在初始化或扩容后存储的是下一次的扩容门槛；
9. sizeCtl = (resizeStamp << 16) + (1 + nThreads)，表示正在进行扩容，高位存储扩容邮戳，低位存储扩容线程数加1；
10. 更新操作时如果正在进行扩容，当前线程协助扩容；
11. 更新操作会采用synchronized锁住当前桶的第一个元素，这是分段锁的思想；
12. 整个扩容过程都是通过CAS控制sizeCtl这个字段来进行的，这很关键；
13. 迁移完元素的桶会放置一个ForwardingNode节点，以标识该桶迁移完毕；
14. 元素个数的存储也是采用的分段思想，类似于LongAdder的实现；
15. 元素个数的更新会把不同的线程hash到不同的段上，减少资源争用；
16. 元素个数的更新如果还是出现多个线程同时更新一个段，则会扩容段（CounterCell）；
17. 获取元素个数是把所有的段（包括baseCount和CounterCell）相加起来得到的；
18. 查询操作是不会加锁的，所以ConcurrentHashMap不是强一致性的；
19. ConcurrentHashMap中不能存储key或value为null的元素；

#### 值得学习的技术

ConcurrentHashMap中有哪些值得学习的技术呢？

我认为有以下几点：

1. CAS + 自旋，乐观锁的思想，减少线程上下文切换的时间；
2. 分段锁的思想，减少同一把锁争用带来的低效问题；
3. CounterCell，分段存储元素个数，减少多线程同时更新一个字段带来的低效；
4. @sun.misc.Contended（CounterCell上的注解），避免伪共享；（p.s.伪共享我们后面也会讲的^^）
5. 多线程协同进行扩容；