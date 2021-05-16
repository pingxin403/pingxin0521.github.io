---
title: Redis 高级类型
date: 2019-10-19 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

Reids 在 Web 应用的开发中使用非常广泛，几乎所有的后端技术都会有涉及到 Redis 的使用。Redis 种除了常见的字符串 String、字典 Hash、列表 List、集合 Set、有序集合 SortedSet 等等之外，还有一些不常用的数据类型，这里着重介绍几个。下面话不多说了，来一起看看详细的介绍吧。

<!--more-->

#### BitMap

BitMap 就是通过一个 bit 位来表示某个元素对应的值或者状态, 其中的 key 就是对应元素本身，实际上底层也是通过对字符串的操作来实现。Redis 从 2.2 版本之后新增了setbit, getbit, bitcount 等几个 bitmap 相关命令。虽然是新命令，但是本身都是对字符串的操作，我们先来看看语法：

```
SETBIT key offset value
```

其中 offset 必须是数字，value 只能是 0 或者 1，咋一看感觉没啥用处，我们先来看看 bitmap 的具体表示，当我们使用命令 setbit key (0,2,5,9,12) 1后，它的具体表示为：

| byte  | bit0 | bit1 | bit2 | bit3 | bit4 | bit5 | bit6 | bit7 |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| byte0 | 1    | 0    | 1    | 0    | 0    | 1    | 0    | 0    |
| byte1 | 0    | 1    | 0    | 0    | 1    | 0    | 0    | 0    |

可以看出 bit 的默认值是 0，那么 BitMap 在实际开发的运用呢？这里举一个例子：储存用户在线状态。这里只需要一个 key，然后把用户 ID 作为 offset，如果在线就设置为 1，不在线就设置为 0。实例代码：

```shell
//设置在线状态
$redis->setBit('online', $uid, 1);
 
//设置离线状态
$redis->setBit('online', $uid, 0);
 
//获取状态
$isOnline = $redis->getBit('online', $uid);
 
//获取在线人数
$isOnline = $redis->bitCount('online');
```

#### Geohash

Redis 的 GEO 特性在 Redis 3.2 版本中推出， 这个功能可以将用户给定的地理位置信息储存起来， 并对这些信息进行操作。GEO 的数据结构总共有六个命令：geoadd、geopos、geodist、georadius、georadiusbymember、gethash,这里着重讲解几个。

**1.GEOADD**

```
GEOADD key longitude latitude member [longitude latitude member ...]
```

将给定的空间元素（纬度、经度、名字）添加到指定的键里面。 这些数据会以有序集合的形式被储存在键里面， 从而使得像 GEORADIUS 和 GEORADIUSBYMEMBER 这样的命令可以在之后通过位置查询取得这些元素。例子：

```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"(integer) 2
```

**2.GEOPOS**

```
GEOPOS key member [member ...]
```

从键里面返回所有给定位置元素的位置（经度和纬度），例子：

```
redis> GEOPOS Sicily Palermo Catania NonExisting1) 1) "13.361389338970184"2) "38.115556395496299"
```

**3.GEODIST**

```
GEODIST key member1 member2 [unit]
```

返回两个给定位置之间的距离。如果两个位置之间的其中一个不存在， 那么命令返回空值。指定单位的参数 unit 必须是以下单位的其中一个：（默认为m）

> m  表示单位为米。
> km 表示单位为千米。
> mi 表示单位为英里。
> ft 表示单位为英尺。

```
redis> GEODIST Sicily Palermo Catania"166274.15156960039"
```

**4.GEORADIUS**

```
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]
```

以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。距离单位和上面的一致，其中后面的选项：

> WITHDIST： 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。距离的单位和用户给定的范围单位保持一致。
> WITHCOORD： 将位置元素的经度和维度也一并返回。
> WITHHASH： 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。这个选项主要用于底层应用或者调试， 实际中的作用并不大。

```
redis> GEORADIUS Sicily 15 37 200 km WITHDIST1) 1) "Palermo"2) "190.4424"2) 1) "Catania"2) "56.4413"
```

##### Geohash算法

Geohash算法一共有三步。

1. 首先将经纬度变成二进制。

   比如这样一个点（39.923201, 116.390705）
    纬度的范围是（-90，90），其中间值为0。对于纬度39.923201，在区间（0，90）中，因此得到一个1；（0，90）区间的中间值为45度，纬度39.923201小于45，因此得到一个0，依次计算下去，即可得到纬度的二进制表示，如下表：

   ![lQSVv4.png](https://s2.ax1x.com/2019/12/30/lQSVv4.png)

   最后得到纬度的二进制表示为：

   ```undefined
    10111000110001111001
   ```

   同理可以得到经度116.390705的二进制表示为：

   ```
     11010010110001000100
   ```

2. 第2步，就是将经纬度合并。

   经度占偶数位，纬度占奇数位，注意，0也是偶数位。

   ```
     11100 11101 00100 01111 00000 01101 01011 00001
   ```

3. 第3步，按照Base32进行编码

   Base32编码表的其中一种如下，是用0-9、b-z（去掉a, i, l, o）这32个字母进行编码。具体操作是先将上一步得到的合并后二进制转换为10进制数据，然后对应生成Base32码。需要注意的是，将5个二进制位转换成一个base32码。上例最终得到的值为

   

   ```undefined
     wx4g0ec1
   ```

   Geohash比直接用经纬度的高效很多，而且使用者可以发布地址编码，既能表明自己位于北海公园附近，又不至于暴露自己的精确坐标，有助于隐私保护。

   - GeoHash用一个字符串表示经度和纬度两个坐标。在数据库中可以实现在一列上应用索引（某些情况下无法在两列上同时应用索引）
   - GeoHash表示的并不是一个点，而是一个矩形区域
   - GeoHash编码的前缀可以表示更大的区域。例如wx4g0ec1，它的前缀wx4g0e表示包含编码wx4g0ec1在内的更大范围。 这个特性可以用于附近地点搜索

   编码越长，表示的范围越小，位置也越精确。因此我们就可以通过比较GeoHash匹配的位数来判断两个点之间的大概距离。

   ![lQSRZn.png](https://s2.ax1x.com/2019/12/30/lQSRZn.png)

**GeoHash算法会对上述编码的整数继续做一次base32编码(0 ~ 9,a ~ z)变成一个字符串。Redis中经纬度使用52位的整数进行编码，放进zset中，zset的value元素是key，score是GeoHash的52位整数值。在使用Redis进行Geo查询时，其内部对应的操作其实只是zset(skiplist)的操作。通过zset的score进行排序就可以得到坐标附近的其它元素，通过将score还原成坐标值就可以得到元素的原始坐标**

**总之，Redis中处理这些地理位置坐标点的思想是: 二维平面坐标点  --> 一维整数编码值 --> zset(score为编码值) --> zrangebyrank(获取score相近的元素)、zrangebyscore --> 通过score(整数编码值)反解坐标点 --> 附近点的地理位置坐标。**

推荐一篇牛逼的讲解高效多维空间点索引算法文章：https://halfrost.com/go_spatial_search/

**问题**

geohash算法有两个问题。首先是边缘问题。

![lQSvi6.png](https://s2.ax1x.com/2019/12/30/lQSvi6.png)

如图，如果车在红点位置，区域内还有一个黄点。相邻区域内的绿点明显离红点更近。但因为黄点的编码和红点一样，最终找到的将是黄点。这就有问题了。

要解决这个问题，很简单，只要再查找周边8个区域内的点，看哪个离自己更近即可。

另外就是曲线突变问题。

本文第2张图片比较好地解释了这个问题。其中0111和1000两个编码非常相近，但它们的实际距离确很远。所以编码相近的两个单位，并不一定真实距离很近，这需要实际计算两个点的距离才行。

**代码实现**

geohash原理清楚后，代码实现就比较简单了。不过仍然有一个问题需要解决，就是如何计算周边的8个区域key值呢

假设我们计算的key值是6位，那么二进制位数就是 6*5 = 30位，所以经纬度分别是15位。我们以纬度为例，纬度会均分15次。这样我们很容易能够算出15次后，划分的最小单位是多少

```cpp
  private void setMinLatLng() {
    minLat = MAXLAT - MINLAT;
    for (int i = 0; i < numbits; i++) {
        minLat /= 2.0;
    }
    minLng = MAXLNG - MINLNG;
    for (int i = 0; i < numbits; i++) {
        minLng /= 2.0;
    }
}
```

得到了最小单位，那么周边区域的经纬度也可以计算得到了。比如说左边区域的经度肯定是自身经度减去最小经度单位。纬度也可以通过加减，得到上下的纬度值，最终周围8个单位也可以计算得到。

可以到 [http://geohash.co/](https://links.jianshu.com/go?to=http%3A%2F%2Fgeohash.co%2F) 进行geohash编码，以确定自己代码是否写错

整体代码如下所示：

```cpp
public class GeoHash {
public static final double MINLAT = -90;
public static final double MAXLAT = 90;
public static final double MINLNG = -180;
public static final double MAXLNG = 180;

private static int numbits = 3 * 5; //经纬度单独编码长度

private static double minLat;
private static double minLng;

private final static char[] digits = { '0', '1', '2', '3', '4', '5', '6', '7', '8',
        '9', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'j', 'k', 'm', 'n', 'p',
        'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z' };

//定义编码映射关系
final static HashMap<Character, Integer> lookup = new HashMap<Character, Integer>();
//初始化编码映射内容
static {
    int i = 0;
    for (char c : digits)
        lookup.put(c, i++);
}

public GeoHash(){
    setMinLatLng();
}

public String encode(double lat, double lon) {
    BitSet latbits = getBits(lat, -90, 90);
    BitSet lonbits = getBits(lon, -180, 180);
    StringBuilder buffer = new StringBuilder();
    for (int i = 0; i < numbits; i++) {
        buffer.append( (lonbits.get(i))?'1':'0');
        buffer.append( (latbits.get(i))?'1':'0');
    }
    String code = base32(Long.parseLong(buffer.toString(), 2));
    //Log.i("okunu", "encode  lat = " + lat + "  lng = " + lon + "  code = " + code);
    return code;
}

public ArrayList<String> getArroundGeoHash(double lat, double lon){
    //Log.i("okunu", "getArroundGeoHash  lat = " + lat + "  lng = " + lon);
    ArrayList<String> list = new ArrayList<>();
    double uplat = lat + minLat;
    double downLat = lat - minLat;

    double leftlng = lon - minLng;
    double rightLng = lon + minLng;

    String leftUp = encode(uplat, leftlng);
    list.add(leftUp);

    String leftMid = encode(lat, leftlng);
    list.add(leftMid);

    String leftDown = encode(downLat, leftlng);
    list.add(leftDown);

    String midUp = encode(uplat, lon);
    list.add(midUp);

    String midMid = encode(lat, lon);
    list.add(midMid);

    String midDown = encode(downLat, lon);
    list.add(midDown);

    String rightUp = encode(uplat, rightLng);
    list.add(rightUp);

    String rightMid = encode(lat, rightLng);
    list.add(rightMid);

    String rightDown = encode(downLat, rightLng);
    list.add(rightDown);

    //Log.i("okunu", "getArroundGeoHash list = " + list.toString());
    return list;
}

//根据经纬度和范围，获取对应的二进制
private BitSet getBits(double lat, double floor, double ceiling) {
    BitSet buffer = new BitSet(numbits);
    for (int i = 0; i < numbits; i++) {
        double mid = (floor + ceiling) / 2;
        if (lat >= mid) {
            buffer.set(i);
            floor = mid;
        } else {
            ceiling = mid;
        }
    }
    return buffer;
}

//将经纬度合并后的二进制进行指定的32位编码
private String base32(long i) {
    char[] buf = new char[65];
    int charPos = 64;
    boolean negative = (i < 0);
    if (!negative){
        i = -i;
    }
    while (i <= -32) {
        buf[charPos--] = digits[(int) (-(i % 32))];
        i /= 32;
    }
    buf[charPos] = digits[(int) (-i)];
    if (negative){
        buf[--charPos] = '-';
    }
    return new String(buf, charPos, (65 - charPos));
}

private void setMinLatLng() {
    minLat = MAXLAT - MINLAT;
    for (int i = 0; i < numbits; i++) {
        minLat /= 2.0;
    }
    minLng = MAXLNG - MINLNG;
    for (int i = 0; i < numbits; i++) {
        minLng /= 2.0;
    }
}

//根据二进制和范围解码
private double decode(BitSet bs, double floor, double ceiling) {
    double mid = 0;
    for (int i=0; i<bs.length(); i++) {
        mid = (floor + ceiling) / 2;
        if (bs.get(i))
            floor = mid;
        else
            ceiling = mid;
    }
    return mid;
}

//对编码后的字符串解码
public double[] decode(String geohash) {
    StringBuilder buffer = new StringBuilder();
    for (char c : geohash.toCharArray()) {
        int i = lookup.get(c) + 32;
        buffer.append( Integer.toString(i, 2).substring(1) );
    }

    BitSet lonset = new BitSet();
    BitSet latset = new BitSet();

    //偶数位，经度
    int j =0;
    for (int i=0; i< numbits*2;i+=2) {
        boolean isSet = false;
        if ( i < buffer.length() )
            isSet = buffer.charAt(i) == '1';
        lonset.set(j++, isSet);
    }

    //奇数位，纬度
    j=0;
    for (int i=1; i< numbits*2;i+=2) {
        boolean isSet = false;
        if ( i < buffer.length() )
            isSet = buffer.charAt(i) == '1';
        latset.set(j++, isSet);
    }

    double lon = decode(lonset, -180, 180);
    double lat = decode(latset, -90, 90);

    return new double[] {lat, lon};
}

public static void main(String[] args)  throws Exception{
    GeoHash geohash = new GeoHash();
//        String s = geohash.encode(40.222012, 116.248283);
//        System.out.println(s);
    geohash.getArroundGeoHash(40.222012, 116.248283);
//        double[] geo = geohash.decode(s);
//        System.out.println(geo[0]+" "+geo[1]);
}
}
```

#### HyperLogLog

Redis 的基数统计，这个结构可以非常省内存的去统计各种计数，比如注册 IP 数、每日访问 IP 数、页面实时UV）、在线用户数等。但是它也有局限性，就是只能统计数量，而没办法去知道具体的内容是什么。

当然用集合也可以解决这个问题。但是一个大型的网站，每天 IP 比如有 100 万，粗算一个 IP 消耗 15 字节，那么 100 万个 IP 就是 15M。而 HyperLogLog 在 Redis 中每个键占用的内容都是 12K，理论存储近似接近 2^64 个值，不管存储的内容是什么，它一个基于基数估算的算法，只能比较准确的估算出基数，可以使用少量固定的内存去存储并识别集合中的唯一元素。而且这个估算的基数并不一定准确，是一个带有 0.81% 标准错误的近似值。

这个数据结构的命令有三个：PFADD、PFCOUNT、PFMERGE

1.PFADD

```
redis> PFADD databases "Redis" "MongoDB" "MySQL"
(integer) 1
redis> PFADD databases "Redis" # Redis 已经存在，不必对估计数量进行更新
(integer) 0
```

2.PFCOUNT

```
redis> PFCOUNT databases(integer) 3
```

3.PFMERGE

```
PFMERGE destkey sourcekey [sourcekey ...]
```

将多个 HyperLogLog 合并为一个 HyperLogLog， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合的并集。合并得出的 HyperLogLog 会被储存在 destkey 键里面， 如果该键并不存在，那么命令在执行之前， 会先为该键创建一个空的 HyperLogLog 。

```
redis> PFADD nosql "Redis" "MongoDB" "Memcached"
(integer) 1
redis> PFADD RDBMS "MySQL" "MSSQL" "PostgreSQL"
(integer) 1
redis> PFMERGE databases nosql RDBMSOK
redis> PFCOUNT databases
(integer) 6
```