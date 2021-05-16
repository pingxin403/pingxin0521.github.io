---
title: Redis 数据结构
date: 2019-08-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### 五种数据结构

所有命令参考：

- <https://redis.io/commands>

- <http://www.redis.cn/commands.html>

#### 全局key操作

<!--more-->

```ruby
--删
flushdb          --清空当前选择的数据库
del mykey mykey2 --删除了两个 Keys

--改
move mysetkey 1        --将当前数据库中的 mysetkey 键移入到 ID 为 1 的数据库中
rename mykey mykey1    --将 mykey 改名为 mykey1
renamenx oldkey newkey --如果 newkey 已经存在，则无效
expire mykey 100 --将该键的超时设置为 100 秒
persist mykey    --将该 Key 的超时去掉,变成持久化的键

--查
keys my*     --获取当前数据库中所有以my开头的key
exists mykey --若不存在,返回0;存在返回1
select 0     --打开 ID 为 0 的数据库
ttl mykey    --查看还有多少秒过期，-1表示永不过期，-2表示已过期
type mykey   --返回mykey对应的值的类型
```

#### 字符串(String)

string是redis最基本的类型，一个key对应一个value。string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

与其它编程语言或其它键值存储提供的字符串非常相似，`键(key)------值(value)` (字符串格式),字符串拥有一些操作命令，如：`get set del` 还有一些比如自增或自减操作等等。

```shell
--增
set mykey "test"       --为键设置新值，并覆盖原有值
getset mycounter 0     -- 设置值,取值同时进行,先返回值，再赋值
setex mykey 10 "hello" -- 设置指定 Key 的过期时间为10秒,在存活时间可以获取value
setnx mykey "hello"    --若该键不存在，则为键设置新值，如果key已经存在则插入无效
mset key3 "stephen" key4 "liu" --批量设置键

--删
del mykey --删除已有键

--改
append mykey "hello" --拼接命令。若该键并不存在,返回当前 Value 的长度，该键已经存在，返回追加后 Value的长度
incr mykey           --值增加1,若该key不存在,创建key,初始值设为0,增加后结果为1，减少命令：DECR
decrby mykey 5       --值减少5，增加命令对应：INCRBY
setrange mykey 20 dd --把第21和22个字节,替换为dd, 超过value长度,自动补0

--查 
exists mykey  --判断该键是否存在，存在返回 1，否则返回0
get mykey     --获取Key对应的value
strlen mykey  --获取指定 Key 的字符长度
ttl mykey     --查看一下指定 Key 的剩余存活时间(秒数)
getrange mykey 1 20 --获取第2到第20个字节,若20超过value长度,则截取第2个和后面所有的的
mget key3 key4 --批量获取键
```

**底层原理**

string表示的是一个可变的字节数组，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

redis是使用C语言开发，但C中并没有字符串类型，只能使用指针或符数组的形式表示一个字符串，所以redis设计了一种简单动态字符串(SDS[Simple Dynamic String])作为底实现：

定义SDS对象，此对象中包含三个属性：

- len buf中已经占有的长度(表示此字符串的实际长度)
- free buf中未使用的缓冲区长度
- buf[] 实际保存字符串数据的地方，以字节\0结尾

所以取字符串的长度的时间复杂度为O(1)，另，buf[]中依然采用了C语言的以\0结尾可以直接使用C语言的部分标准C字符串库函数。

空间分配原则：**当len小于IMB（1024\*1024）时增加字符串分配空间大小为原来的2倍，当len大于等于1M时每次分配 额外多分配1M的空间。**(字符串最大长度为 **512M**)

![MHAcm6.png](https://s2.ax1x.com/2019/11/22/MHAcm6.png)

由此可以得出以下特性：

- redis为字符分配空间的次数是小于等于字符串的长度N，而原C语言中的分配原则必为N。降低了分配次数提高了追加速度，代价就是多占用一些内存空间，且这些空间不会自动释放。
- 二进制安全的
- 高效的计算字符串长度(时间复杂度为O(1))
- 高效的追加字符串操作。

#### List类型

List类型是按照插入顺序排序的字符串链表（所以它这里的list指的相当于java中的linkesdlist）。和数据结构中的普通链表一样，我们可以在其头部(left)和尾部(right)添加新的元素。在插入时，如果该键并不存在，Redis将为该键创建一个新的链表。与此相反，如果链表中所有的元素均被移除，那么该键也将会被从数据库中删除。List类型:(链表:**最后一个插入的元素,位置索引为o**)

```shell
--增 
lpush mykey a b --若key不存在,创建该键及与其关联的List,依次插入a ,b， 若List类型的key存在,则插入value中
lpushx mykey2 e --若key不存在,此命令无效， 若key存在,则插入value中
linsert mykey before a a1 --在 a 的前面插入新元素 a1
linsert mykey after e e2  --在e 的后面插入新元素 e2
rpush mykey a b --在链表尾部先插入b,在插入a（lpush list a b那么读的时候是b,a的顺序，而rpush是怎么放怎么读出来
rpushx mykey e  --若key存在,在尾部插入e, 若key不存在,则无效
rpoplpush mykey mykey2 -- 将mykey的尾部元素弹出,再插入到mykey2 的头部(原子性的操作)

--删
del mykey       --删除已有键 
lrem mykey 2 a  --从头部开始找,按先后顺序,值为a的元素,删除数量为2个,若存在第3个,则不删除
ltrim mykey 0 2 --从头开始,索引为0,1,2的3个元素,其余全部删除

--改
lset mykey 1 e        --从头开始, 将索引为1的元素值,设置为新值 e,若索引越界,则返回错误信息
rpoplpush mykey mykey --将 mykey 中的尾部元素移到其头部

--查
lrange mykey 0 -1 --取链表中的全部元素，其中0表示第一个元素,-1表示最后一个元素。
lrange mykey 0 2  --从头开始,取索引为0,1,2的元素
lpop mykey        --获取头部元素,并且弹出头部元素,出栈
lindex mykey 6    --从头开始,获取索引为6的元素 若下标越界,则返回nil
```

**原理**

Redis将列表数据结构命名为list而不是array，是因为列表的存储结构用的是链表而不是数组，而且链表还是双向链表。因为它是链表，所以随机定位性能较弱O(n)，首尾插入删除性能较优O(1)。如果list的列表长度很长，使用时我们一定要关注链表相关操作的时间复杂度。

在3.2版本之前，列表是使用ziplist和linkedlist实现的，在这些老版本中，当列表对象同时满足以下两个条件时，列表对象使用ziplist编码：

- 列表对象保存的所有字符串元素的长度都小于64字节
- 列表对象保存的元素数量小于512个
- 当有任一条件 不满足时将会进行一次转码，使用linkedlist。

而在3.2版本之后，重新引入了一个quicklist的数据结构，列表的底层都是由quicklist实现的，它结合了ziplist和linkedlist的优点。【A doubly linked list of ziplists】意思就是一个由ziplist组成的双向链表。那么这两种数据结构怎么样结合的呢？

**[ziplist的结构](http://zhangtielei.com/posts/blog-redis-ziplist.html)**：由表头和N个entry节点和压缩列表尾部标识符zlend组成的一个连续的内存块。然后通过一系列的编码规则，提高内存的利用率，主要用于存储整数和比较短的字符串。可以看出在插入和删除元素的时候，都需要对内存进行一次扩展或缩减，还要进行部分数据的移动操作，这样会造成更新效率低下的情况。

**quicklist：**

![MHEwHf.png](https://s2.ax1x.com/2019/11/22/MHEwHf.png)

Redis底层存储的还不是一个简单的linkedlist，而是称之为快速链表quicklist的一个结构。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。所以Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

“ziplist组成的双向链表”是什么意思？实际上，它整体宏观上就是一个链表结构，只不过每个节点都是以压缩列表ziplist的结构保存着数据，而每个ziplist又可以包含多个entry。也可以说一个quicklist节点保存的是一片数据，而不是一个数据。总结：

- 整体上quicklist就是一个双向链表结构，和普通的链表操作一样，插入删除效率很高，但查询的效率却是O(n)。不过，这样的链表访问两端的元素的时间复杂度却是O(1)。所以，对list的操作多数都是poll和push。
- 每个quicklist节点就是一个ziplist，具备压缩列表的特性。

在redis.conf配置文件中，有两个参数可以优化列表：

1. list-max-ziplist-size 表示每个quicklistNode的字节大小。默认为-2 表示8KB
2. list-compress-depth 表示quicklistNode节点是否要压缩。默认是0 表示不压缩

#### 哈希(hash)

我们可以将Redis中的Hash类型看成具有<key,<key1,value>>,其中同一个key可以有多个不同key值的<key1,value>，所以该类型非常适合于存储值对象的信息。如Username、Password和Age等。如果Hash中包含很少的字段，那么该类型的数据也将仅占用很少的磁盘空间。

```
--案例解释:
--Map类型:
hset key field1 "s" 
redis.key=key redis.value=( map.key=field1 map.value=s ) 

--增
hset key field1 "s"   --若字段field1不存在,创建该键及与其关联的Hash, Hash中,key为field1 ,并设value为s ，若字段field1存在,则覆盖 
hsetnx key field1 s   --若字段field1不存在,创建该键及与其关联的Hash, Hash中,key为field1 ,并设value为s， 若字段field1存在,则无效
hmset key field1 "hello" field2 "world --一次性设置多个字段    

--删
hdel key field1 --删除 key 键中字段名为 field1 的字段
del key  -- 删除键    

--改 
hincrby key field 1 --给field的值加1

--查
hget key field1 --获取键值为 key,字段为 field1 的值
hlen key        --获取key键的字段数量
hexists key field1 --判断 key 键中是否存在字段名为 field1 的字段
hmget key field1 field2 field3 --一次性获取多个字段
hgetall key --返回 key 键的所有field值及value值
hkeys key   --获取key 键中所有字段的field值
hvals key   --获取 key 键中所有字段的value值
```

类似java的HashSet的内部实现使用的是HashMap。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

redis的散列可以存储多个键 值 对之间的映射，散列存储的值既可以是字符串又可以是数字值，并且用户同样可以对散列存储的数字值执行自增操作或者自减操作。散列可以看作是一个文档或关系数据库里的一行。hash底层的数据结构实现有两种：

- 一种是ziplist，上面已经提到过。当存储的数据超过配置的阀值时就是转用hashtable的结构。这种转换比较消耗性能，所以应该尽量避免这种转换操作。同时满足以下两个条件时才会使用这种结构：
  - 当键的个数小于hash-max-ziplist-entries（默认512）
  - 当所有值都小于hash-max-ziplist-value（默认64）
- 另一种就是hashtable。这种结构的时间复杂度为O(1)，但是会消耗比较多的内存空间。

#### set类型

Set类型看作为没有排序的字符集合。如果多次添加相同元素，Set中将仅保留该元素的一份拷贝。

```shell
--增
sadd myset a b c --若key不存在,创建该键及与其关联的set,依次插入a ,b,c。若key存在,则插入value中,若a 在myset中已经存在,则插入了 b 和 c 两个新成员。

--删
spop myset       --尾部的b被移出,事实上b并不是之前插入的第一个或最后一个成员
srem myset a d f --若f不存在, 移出 a、d ,并返回2

--改
smove myset myset2 a --将a从 myset 移到 myset2，

--查
sismember myset a --判断 a 是否已经存在，返回值为 1 表示存在。
smembers myset    --查看set中的内容
scard myset       --获取Set 集合中元素的数量
srandmember myset --随机的返回某一成员
sdiff myset1 myset2        --显示myset1和myset2比较后myset1独有的值（例：myset1有1,2,3,4。myset2有2,3,5,6，那最终显示1,4。
sdiff myset1 myset2 myset3 --显示myset1和myset2，myset3比较后myset1独有的值
sdiffstore diffkey myset myset2 myset3   --3个集和比较,获取独有的元素,并存入diffkey 关联的Set中
sinter myset myset2 myset3               --获得3个集合中都有的元素（交集）
sinterstore interkey myset myset2 myset3 --把交集存入interkey 关联的Set中
sunion myset myset2 myset3               --获取3个集合中的成员的并集
sunionstore unionkey myset myset2 myset3 --把并集存入unionkey 关联的Set中
```

 redis的集合和列表都可以存储多个字符串，它们之间的不同在于，列表可以存储多个相同的字符串，而集合则通过使用散列表（hashtable）来保证自已存储的每个字符串都是各不相同的(这些散列表只有键，但没有与键相关联的值)，redis中的集合是无序的。还可能存在另一种集合，那就是intset，它是用于存储整数的有序集合，里面存放同一类型的整数。共有三种整数：int16_t、int32_t、int64_t。查找的时间复杂度为O(logN)，但是插入的时候，有可能会涉及到升级（比如：原来是int16_t的集合，当插入int32_t的整数的时候就会为每个元素升级为int32_t）这时候会对内存重新分配，所以此时的时间复杂度就是O(N)级别的了。注意：intset只支持升级不支持降级操作。

intset在redis.conf中也有一个配置参数set-max-intset-entries默认值为512。表示如果entry的个数小于此值，则可以编码成REDIS_ENCODING_INTSET类型存储，节约内存。否则采用dict的形式存储。

#### Sorted-Sets类型

 Sorted-Sets中的每一个成员都会有一个分数(score)与之关联，Redis正是通过分数来为集合中的成员进行从小到大的排序。成员是唯一的，但是分数(score)却是可以重复的。

**分数:按分数高低排序。位置索引:分数最低的索引为0**

```shell
--增
zadd myzset 2 "two" 3 "three" --添加两个分数分别是 2 和 3 的两个成员

--删
zrem myzset one two  --删除多个成员变量,返回删除的数量

--改
zincrby myzset 2 one --将成员 one 的分数增加 2，并返回该成员更新后的分数（分数改变后相应它的index也会改变）

--查 
zrange myzset 0 -1 WITHSCORES --返回所有成员和分数,不加WITHSCORES,只返回成员
zrank myzset one   --获取成员one在Sorted-Set中的位置索引值。0表示第一个位置（分数越后，index就越后，所以它是有序的）
zcard myzset       --获取 myzset 键中成员的数量
zcount myzset 1 2  --获取分数满足表达式 1 <= score <= 2 的成员的数量
zscore myzset three --获取成员 three 的分数
zrangebyscore myzset 1 2 --获取分数满足表达式 1 < score <= 2 的成员


#-inf 表示第一个成员，+inf最后一个成员
#limit限制关键字
#2 3 是索引号
zrangebyscore myzset -inf +inf limit 2 3 --返回索（index）是2和3的成员
zremrangebyscore myzset 1 2      -- 删除分数 1<= score <= 2 的成员，并返回实际删除的数量
zremrangebyrank myzset 0 1       --删除位置索引满足表达式 0 <= rank <= 1 的成员
zrevrange myzset 0 -1 WITHSCORES --按位置索引从高到低,获取所有成员和分数

#原始成员:位置索引从小到大
one 0 
two 1
#执行顺序:把索引反转
位置索引:从大到小
one 1
two 0
#输出结果: two 
one
zrevrange myzset 1 3 --获取位置索引,为1,2,3的成员

#相反的顺序:从高到低的顺序
zrevrangebyscore myzset 3 0 --获取分数 3>=score>=0的成员并以相反的顺序输出
zrevrangebyscore myzset 4 0 limit 1 2 --获取索引是1和2的成员,并反转位置索引
```

Redis 的 zset 是一个复合结构，第一个是hash，第二个是跳跃列表skiplist。hash 结构来存储 value 和 score 的对应关系，保障元素value的唯一性，可以通过元素value找到相应的score值。跳跃列表的目的在于给按照score排序，根据score的范围获取value列表。

![MHVGZV.png](https://s2.ax1x.com/2019/11/22/MHVGZV.png)

**跳跃列表**

跳表(skip List)是一种随机化的数据结构，基于并联的链表，实现简单，插入、删除、查找的复杂度均为O(logN)。简单说来跳表也是链表的一种，只不过它在链表的基础上增加了跳跃功能，正是这个跳跃的功能，使得在查找元素时，跳表能够提供O(logN)的时间复杂度。

因为 zset 要支持随机的插入和删除，所以它不好使用数组来表示。而如果我们用一个有序链表来实现，那么我们要查找某个数据，就需要从头开始逐个进行比较（二分查找的对象必须是数组，只有数组才可以支持快速位置定位，链表做不到），也就是说，时间复杂度为O(n)。同样，当我们要插入新数据的时候，也要经历同样的查找过程，从而确定插入位置。

![MHVJaT.png](https://s2.ax1x.com/2019/11/22/MHVJaT.png)

假如我们每相邻两个节点增加一个指针，让指针指向下下个节点，如下图：

![MHVNiF.png](https://s2.ax1x.com/2019/11/22/MHVNiF.png)

现在当我们想查找数据的时候，可以先沿着这个新链表进行查找。当碰到比待查数据大的节点时，再回到原来的链表中进行查找。比如，我们想查找23，查找的路径是沿着下图中标红的指针所指向的方向进行的：

- 23首先和7比较，再和19比较，比它们都大，继续向后比较。
- 但23和26比较的时候，比26要小，因此回到下面的链表（原链表），与22比较。
- 23比22要大，沿下面的指针继续向后和26比较。23比26小，说明待查数据23在原链表中不存在，而且它的插入位置应该在22和26之间。

利用同样的方式，我们可以在上层新产生的链表上，继续为每相邻的两个节点增加一个指针，从而产生第三层链表。如下图：

![MHVBs1.png](https://s2.ax1x.com/2019/11/22/MHVBs1.png)

skiplist正是受这种多层链表的想法的启发而设计出来的。实际上，按照上面生成链表的方式，上面每一层链表的节点个数，是下面一层的节点个数的一半，这样查找过程就非常类似于一个二分查找，使得查找的时间复杂度可以降低到O(log n)。但是，这种方法在插入数据的时候有很大的问题。新插入一个节点之后，就会打乱上下相邻两层链表上节点个数严格的2:1的对应关系。如果要维持这种对应关系，就必须把新插入的节点后面的所有节点（也包括新插入的节点）重新进行调整，这会让时间复杂度重新蜕化成O(n)。删除数据也有同样的问题。

skiplist为了避免这一问题，它不要求上下相邻两层链表之间的节点个数有严格的对应关系，而是为每个节点随机出一个层数(level)。比如，一个节点随机出的层数是3，那么就把它链入到第1层到第3层这三层链表中。为了表达清楚，下图展示了如何通过一步步的插入操作从而形成一个skiplist的过程：

![MHVsZ6.png](https://s2.ax1x.com/2019/11/22/MHVsZ6.png)

实际应用中的skiplist每个节点应该包含key和value两部分。前面的描述中我们没有具体区分key和value，但实际上列表中是按照key(score)进行排序的，查找过程也是根据key在比较。

执行插入操作时计算随机数的过程，是一个很关键的过程，它对skiplist的统计特性有着很重要的影响。这并不是一个普通的服从均匀分布的随机数，它的计算过程如下：

- 首先，每个节点肯定都有第1层指针（每个节点都在第1层链表里）。
- 如果一个节点有第i层(i>=1)指针（即节点已经在第1层到第i层链表中），那么它有第(i+1)层指针的概率为p（redis中为25%）。
- 节点最大的层数不允许超过一个最大值，记为MaxLevel（redis中为64）。

计算随机层数的伪码如下所示：

```
randomLevel()
   level := 1
   // random()返回一个[0...1)的随机数
   while random() < p and level < MaxLevel do
       level := level + 1
   return level

```

因为每层的晋升的概率都只有25%，所以官方的跳跃列表更加的扁平化，层高相对较低，在单个层上需要遍历的节点数量会稍多一点。也正是因为层数一般不高，所以遍历的时候从顶层开始往下遍历会非常浪费。跳跃列表会记录一下当前的最高层数maxLevel，遍历时从这个 maxLevel 开始遍历性能就会提高很多

在一个极端的情况下，zset 中所有的 score 值都是一样的，zset 的查找性能会退化为 O(n) 么？Redis 作者自然考虑到了这一点，所以 zset 的排序元素不只看 score 值，如果 score 值相同还需要再比较 value 值 (字符串比较)。

**元素排名**

如果仅仅使用上面的结构，rank 是不能算出来的。Redis 在 skiplist 的 forward 指针上进行了优化，给每一个 forward 指针都增加了 span 属性，span 是「跨度」的意思，表示从前一个节点沿着当前层的 forward 指针跳到当前这个节点中间会跳过多少个节点。Redis 在插入删除操作时会小心翼翼地更新 span 值的大小。这样当我们要计算一个元素的排名时，只需要将「搜索路径」上的经过的所有节点的跨度 span 值进行叠加就可以算出元素的最终 rank 值。

**skiplist与平衡树的比较：**

- 查找单个key，skiplist和平衡树的时间复杂度都为O(log n)。
- 在做范围查找的时候，平衡树比skiplist操作要复杂。
- 平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。
- 从内存占用上来说，skiplist比平衡树更灵活一些。一般来说，平衡树每个节点包含2个指针（分别指向左右子树），
- 从算法实现难度上来比较，skiplist比平衡树要简单得多。

### 对应数据类型适应场景

**字符串**

1. 缓存功能

   典型使用场景：Redis作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从Redis中获取，由于Redis具有支撑高并发的特性，所以缓存通常能起到加速读写和降低后端压力的作用。

   开发提示：与MySQL等关系型数据库不同的是，Redis没有命令空间，而且也没有对键名有强制要求，但设计合理的键名，有利于防止键冲突和项目的可维护性，比较推荐的方式是使用“`业务名:对象名:id:[属性]`”作为键名。例如MySQL的数据库名为vs，用户表名为user，那么对应的键可以用"vs:user:1"，"vs:user:1:name"来表示，如果当前Redis只被一个业务使用，甚至可以去掉vs。

   如果键名比较长，例如"user:{uid}:friends:message:{mid}"，可以在能描述含义的前提下适当减少键的长度，例如采用缩写形式，从而减少由于键过长的内存浪费。

   ```shell
   > set User:1:name hyp EX 100 NX
   OK
   > get User:1:name
   "hyp"
   ```

   缓存是 `redis` 出镜率最高的一种使用场景，仅仅使用 `set/get` 就可以实现，不过也有一些需要考虑的点

   - 如何更好地设置缓存
   - 如何保持缓存与上游数据的一致性
   - 如何解决缓存血崩，缓存击穿问题

2. 计数

   典型应用场景：视频播放数计数的基础组件，用户每播放一次视频，相应的视频播放数就会自增1。Redis可以实现快速计数、查询缓存的功能，同时数据可以异步落地到其他数据源。

   开发提示：实际上一个真实的计数系统要考虑的问题会很多，防作弊、按照不同维度计数，数据持久化到底层数据源等。

3. 共享Session

   典型应用场景：用户登陆信息，Redis将用户的Session进行集中管理，每次用户更新或查询登陆信息都直接从Redis中集中获取。

   ```shell
   > set 5d27e60e6fb9a07f03576687 '{"id": 10086, role: "ADMIN"}' EX 7200
   OK
   > get 5d27e60e6fb9a07f03576687
   "{\"id\": 10086, role: \"ADMIN\"}"
   ```

   这也是很常用的一种场景，不过相对于有状态的 session，也可以考虑使用 JWT，各有利弊

   - [json web token 实践登录以及校验码验证](https://juejin.im/post/5cc459976fb9a032212cc73b)

4. 限速

   限流即在单位时间内只允许通过特定数量的请求，有两个关键参数

   - window，单位时间
   - max，最大请求数量

   最常见的场景: 短信验证码一分钟只能发送两次

   ```
   FUNCTION LIMIT_API_CALL(ip):
   current = GET(ip)
   IF current != NULL AND current > 10 THEN
       ERROR "too many requests per second"
   ELSE
       value = INCR(ip)
       IF value == 1 THEN
           EXPIRE(ip,1)
       END
       PERFORM_API_CALL()
   END
   ```

   可以使用计数器对 API 的请求进行限流处理，但是要注意几个问题

   1. 在平滑的滑动窗口时间内在极限情况下会有两倍数量的请求数
   2. 条件竞争 (Race Condition)

   这时候可以通过编程，根据 `TTL key` 进行进一步限制，或者使用一个 `LIST` 来维护每次请求打来的时间戳进行实时过滤。以下是 `node` 实现的一个 `Rate Limter`。参考源码 [node-rate-limiter-flexible](https://github.com/animir/node-rate-limiter-flexible)

   ```
   this.client
     .multi()
     .set(rlKey, 0, 'EX', secDuration, 'NX')
     .incrby(rlKey, points)
     .pttl(rlKey)
     .exec((err, res) => {
       if (err) {
         return reject(err);
       }
   
       return resolve(res);
     })
   
   if (res.consumedPoints > this.points) {
     // ...
   } else if (this.execEvenly && res.msBeforeNext > 0 && !res.isFirstInDuration) {
     // ...
     setTimeout(resolve, delay, res);
   } else {
     resolve(res);
   }
   ```

   - [node-rate-limiter-flexible](https://github.com/animir/node-rate-limiter-flexible)
   - [邮件发送，限流，漏桶与令牌桶](https://juejin.im/post/5cceafe5f265da039d32966d)

5. 分布式锁

   ```lua
   set Lock:User:10086 06be97fc-f258-4202-b60b-8d5412dd5605 EX 60 NX
   
   # 释放锁，一段 LUA 脚本
   if redis.call("get",KEYS[1]) == ARGV[1] then
       return redis.call("del",KEYS[1])
   else
       return 0
   end
   ```

   这是一个最简单的单机版的分布式锁，有以下要点

   - `EX` 表示锁会过期释放
   - `NX` 保证原子性
   - 解锁时对比资源对应产生的 UUID，避免误解锁

   当你使用分布式锁是为了解决一些性能问题，如分布式定时任务防止执行多次 (做好幂等性)，而且鉴于单点 `redis` 挂掉的可能性很小，可以使用这种单机版的分布式锁。

**哈希**

1. 缓存用户信息

   相比于使用字符串序列化缓存用户信息，哈希类型变得更加直观，并且在更新操作上会更加便捷。可以将每个用户的id定义为键后缀，多对field-value对应每个用户的属性。

   哈希类型和关系型数据库不同之处：

   - 哈希类型是稀疏的，而关系型数据库是完全结构化的，例如哈希类型每个键可以有不同的field，而关系型数据库一旦添加新的列，所有行都要为其设置值(即使为NULL)。

   - 关系型数据库可以做复杂的关系查询，而Redis去模拟关系型复杂查询开发困难，维护成本高。

   三种缓存用户信息优缺点比较：

   - 原生字符串类型：每个属性一个键

     优点：简单直观，每个属性都支持更新操作。

     缺点：占用过多的键，内存占用量较大，同时用户信息内聚性比较差，所以此种方案一般不会在生产环境使用。

   - 序列化字符串类型：将用户信息序列化后用一个键保存。

     优点：简化编程，如果合理的使用序列化可以提高内存的使用效率。

     缺点：序列化和反序列化有一定的开销，同时每次更新属性都需要把全部数据取出进行反序列化，更新后再序列化到Redis中。

   - 哈希类型：每个用户属性使用一对field-value，但是只用一个键保存。

     优点：简单直观，如果使用合理可以减少内存空间的使用。

     缺点：要控制哈希在ziplist和hashtable两种内部编码的转换，hashtable会消耗更多内存。

**列表**

1. 消息队列

   Redis的lpush+brpop命令组合即可实现阻塞队列，生产者客户端使用lrpush从列表左侧插入元素，多个消费者客户端使用brpop命令阻塞式的"抢"列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性。

   ```shell
   > lpush UserEmailQueue 1 2 3 4
   lpop UserEmailQueue
   > rpop UserEmailQueue
   1
   > rpop UserEmailQueue
   2
   ```

   可以把 `redis` 的队列视为分布式队列，作为消息队列时，生产者在一头塞数据，消费者在另一头出数据: (lpush/rpop, rpush/lpop)。不过也有一些不足，而这些不足有可能是致命的，不过对于一些丢几条消息也没关系的场景还是可以考虑的

   - 没有 ack，有可能丢消息

   - 需要做 `redis` 的持久化配置

2. 文章列表

   每个用户有属于自己的文章列表，现在需要分页展示文章列表。此时可以考虑使用列表，因为列表不但是有序的，同时支持按照索引范围获取元素。

4. 开发提示

   ```
   lpush + lpop = Stack(栈)
   
   lpush + rpop = Queue(队列)
   
   lpush + ltrim = Capped Collection(有限集合)
   
   lpush + brpop = Message Queue(消息队列)
   ```

**集合**

1. 标签(tag)

   集合类型比较典型的使用场景是标签(tag)，例如一个用户可能对娱乐、体育比较感兴趣，另一个用户可能对历史、新闻比较感兴趣，这些兴趣就是标签。 开发提示：用户和标签的关系维护应该在一个事物执行，防止部分命令失败造成的数据不一致。

2. 过滤器 (dupefilter)

   ```shell
   > sadd UrlSet http://1
   (integer) 1
   > sadd UrlSet http://2
   (integer) 1
   > sadd UrlSet http://2
   (integer) 0
   > smembers UrlSet
   1) "http://1"
   2) "http://2"
   ```

   [scrapy-redis](https://github.com/rmax/scrapy-redis) 作为分布式的爬虫框架，便是使用了 `redis` 的 `Set` 这个数据结构来对将要爬取的 url 进行去重处理。

   ```python
   # https://github.com/rmax/scrapy-redis/blob/master/src/scrapy_redis/dupefilter.py
   def request_seen(self, request):
       """Returns True if request was already seen.
       Parameters
       ----------
       request : scrapy.http.Request
       Returns
       -------
       bool
       """
       fp = self.request_fingerprint(request)
       added = self.server.sadd(self.key, fp)
       return added == 0
   ```

   不过当 `url` 过多时，会有内存占用过大的问题

**有序集合**

1. 排行榜系统

   有序集合比较典型的使用场景就是排行榜系统，例如视频网站需要对用户上传的视频做排行榜，榜单的维度可能是多个方面的：按照时间、按照播放数量、按照获得的赞数

**PUB/SUB**

1. 分布式 websocket

   可以通过 redis 的 `PUB/SUB` 来在 websocket server 间进行交流。可以参考以下项目

   - [socket.io-redis](https://github.com/socketio/socket.io-redis)

### 参考

1. [Redis内部数据结构详解](http://zhangtielei.com/posts/blog-redis-dict.html)
2. [Redis内部数据结构详解之跳跃表(skiplist)](https://blog.csdn.net/Acceptedxukai/article/details/17333673)
3. [文章投票网站的redis相关实现](https://segmentfault.com/a/1190000010741281)