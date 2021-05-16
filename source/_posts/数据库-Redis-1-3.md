---
title: Jedis 使用和原理
date: 2019-10-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### redis协议规范（Redis Protocol specification）

**官网文档**

https://redis.io/topics/protocol

http://www.redis.cn/topics/protocol.html

<!--more-->

Redis服务器与客户端通过RESP（REdis Serialization Protocol）协议通信。

redis协议在以下几点之间做出了折衷：

1. 简单的实现
2. 快速地被计算机解析
3. 简单得可以能被人工解析
4. 网络层，Redis在TCP端口6379上监听到来的连接（本质就是socket），客户端连接到来时，Redis服务器为此创建一个TCP连接。在客户端与服务器端之间传输的每个Redis命令或者数据都以\r\n结尾。

**RESP协议支持的数据类型**

`Simple String`第一个字节以`+`开头，随后紧跟内容字符串（不能包含`CR` `LF`），最后以`CRLF`结束。很多Redis命令执行成功时会返回"OK"，"OK"就是一个`Simple String`：

```bash
"+OK\\r\\n"
```

`Error`的结构与`Simple String`很像，但是第一个字节以`-`开头：

```bash
"-ERR unknown command 'foobar'"
```

`-`符号后的第一个单词代表错误类型，ERR代表一般错误，WRONGTYPE代表在某种数据结构上执行了不支持的操作。

`Integer`第一个字节以`:`开头，随后紧跟数字，以`CRLF`结束：

```bash
":1000\\r\\n"
```

很多Redis命令会返回`Integer`,例如`INCR` `LLEN`等。

`Bulk String`是一种二进制安全的字符串结构，整个结构包含两部分。第一部分以`$`开头，后面紧跟字符串的字节长度，`CRLF`结尾。第二部分是真正的字符串内容，`CRLF`结尾，最大长度限制为512MB。一个`Bulk String`结构的"Hello World!"是：

```bash
"$12\\r\\nHello World!\\r\\n"
```

空字符串是：

```bash
"$0\\r\\n\\r\\n"  
```

nil是：

```bash
"$-1\\r\\n"
```

`Array`也可以看成由两部分组成，第一部分以`*`开头，后面紧跟一个数字代表`Array`的长度，以`CRLF`结束。第二部分是每个元素的具体值，可能是`Integer`，可能是`Bulk String`。`Array`结构的["hello", "world"]是：

```bash
"*2\\r\\n$5\\r\\nhello\\r\\n$5\\r\\nworld\\r\\n"
```

空`Array`：

```bash
"*-1\\r\\n"
```

**基本通讯模型**

执行过程：发送指令--》执行命令--》返回结果

执行命令：单线程执行，所有命令进入队列，按顺序执行

单线程快的原因：纯内存访问，单线程避免线程切换和竞争产生资源消耗，RESP协议简单

问题：如果某个命令执行慢，会造成其他命令的阻塞

![MHiMFJ.png](https://s2.ax1x.com/2019/11/22/MHiMFJ.png)

**请求**

Redis接收由不同参数组成的命令。一旦收到命令，将会立刻被处理，并回复给客户端

**新的统一请求协议**

在这个统一协议里，发送给Redis服务端的所有参数都是二进制安全的。以下是通用形式：

```
*后面数量表示存在几个$
$后面数量表示字符串的长度
```

例子：

```
*3
$3
SET
$5
mykey
$7
myvalue
```

上面的命令看上去像是单引号字符串，所以可以在查询中看到每个字节的准确值：

```
"*3\r\n$3\r\nSET\r\n$5\r\nmykey\r\n$7\r\nmyvalue\r\n"
```

在Redis的回复中也使用这样的格式。批量回复时，这种格式用于每个参数$6\r\nmydata\r\n。 实际的统一请求协议是Redis用于返回列表项，并调用 Multi-bulk回复。 仅仅是N个以以\*\r\n为前缀的不同批量回复，是紧随的参数（批量回复）数目。

**回复**

Redis用不同的回复类型回复命令。它可能从服务器发送的第一个字节开始校验回复类型(最小单元类型)：

1. 用单行回复，回复的第一个字节将是“+”
2. 错误消息，回复的第一个字节将是“-”
3. 整型数字，回复的第一个字节将是“:”
4. 批量回复，回复的第一个字节将是“$”
5. 多个批量回复，回复的第一个字节将是“\*”

注：每个单元结束时统一加上回车换行符号\r\n

```
#单行字符串 hello
+hello\r\n
#多行字符串 hello，也支持表示单行字符串
$11\r\nhello\r\n
#NULL 用多行字符串表示，不过长度要写成-1
$-1\r\n
#空串 用多行字符串表示，长度填 0（因为两个\r\n之间,隔的是空串）
$0\r\n\r\n
#整数 1024
:1024\r\n
#错误 参数类型错误（错误消息不需要\r\n结尾）
-WRONGTYPE unknown command
#数组 [1,2,3]
*3\r\n:1\r\n:2\r\n:3\r\n
```

**模拟Redis服务和客户端通讯，实现RESP协议通信**

枚举类

```java
public enum CommandRedis {
    SET, GET, SETNX
}
```

实现类

```java
import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

/**
 * @author hyp
 * Project name is JavaLearn
 * Include in com.hyp.learn.redis
 * hyp create at 19-11-22
 **/
public class RedisClientByResp {
    private Socket socket;

    public RedisClientByResp() {
        try {
            socket = new Socket("127.0.0.1", 6379);
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("连接失败" + e.getMessage());
        }
    }

    /**
     * 请求redis
     *
     * @param cr
     * @param key
     * @param value
     * @return
     * @throws IOException
     */
    private String postRequest(CommandRedis cr, String key, String value) throws IOException {
        StringBuffer sb = new StringBuffer();
        sb.append("*3").append("\r\n");
        sb.append("$").append(cr.name().length()).append("\r\n");
        sb.append(cr.name()).append("\r\n");
        // 注意中文汉字。一个汉字两个字节，长度为2
        sb.append("$").append(key.getBytes().length).append("\r\n");
        sb.append(key).append("\r\n");
        if (null != value) {
            sb.append("$").append(value.getBytes().length).append("\r\n");
            sb.append(value).append("\r\n");
        }
        System.out.println(sb.toString());
        socket.getOutputStream().write(sb.toString().getBytes());
        byte[] b = new byte[64];
        sb = new StringBuffer();
        InputStream is = socket.getInputStream();
        while (is.available() > 0) {
            int len = is.read(b);
            if (len <= 0) {
                break;
            }
            sb.append(new String(b, 0, len));
        }
        return sb.toString();
    }


    /**
     * 设置值
     *
     * @param key
     * @param value
     * @return
     * @throws IOException
     */
    public String set(String key, String value) throws IOException {
        return postRequest(CommandRedis.SET, key, value);
    }

    /**
     * 获取值
     *
     * @param key
     * @return
     * @throws Exception
     */
    public String get(String key) throws Exception {
        return postRequest(CommandRedis.GET, key, null);

    }

    /**
     * 设置值：不会覆盖存在的值
     *
     * @param key
     * @param value
     * @return
     * @throws Exception
     */
    public String setnx(String key, String value) throws Exception {
        return postRequest(CommandRedis.SETNX, key, value);
    }

    public static void main(String[] args) {
        RedisClientByResp resp = new RedisClientByResp();
        try {
            System.out.println(resp.set("mykey", "myvalue"));
            System.out.println(resp.get("mykey"));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (!resp.socket.isConnected()) {
                try {
                    resp.socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }


    }
}
```

### Jedis使用

Jedis是Redis的Java客户端，连接池使用commons-pool2

 Jedis Client是Redis官网推荐的一个面向java客户端，库文件实现了对redis各类API进行封装调用。redis通信协议是Redis客户端与Redis Server之间交流的语言，它规定了请求和返回值的格式。redis-cli与server端使用一种专门为redis设计的协议RESP(Redis Serialization Protocol)交互，Resp本身没有指定TCP，但redis上下文只使用TCP连接。

```xml
  <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.0.0</version>
        </dependency>
```

**Jedis对应Redis的四种工作模式**

![MbBo4K.png](https://s2.ax1x.com/2019/11/23/MbBo4K.png)

Jedis的主要模块，Jedis,JedisCluster,JedisSentinel和ShardedJedis对应了Redis的四种工作模式：Redis Standalone（单节点模式）,Redis Cluster（集群模式）,Redis Sentinel（哨兵模式）和Redis Sharding（分片模式）。

**Jedis的三种请求模式**

每个Jedis实例对应一个Redis节点，我们对Jedis实例的每个操作，都相当于使用`redis-cli`启动客户端的直接操作。无论是集群模式，哨兵模式，还是分片模式，内部均为对Jedis实例的操作。所以了解Jedis类的内部结构及Jedis实例的请求模式是掌握Jedis框架的基础。

 Jedis实例有3种请求模式，Pipeline，Transaction和Client。

![MbBjHI.png](https://s2.ax1x.com/2019/11/23/MbBjHI.png)



Jedis实例通过**Socket**建立客户端与服务端的长连接，往**outputStream**发送命令，从**inputStream**读取回复，

1. Client模式

    Client模式就是常用的“所见即所得”，客户端发一个命令，阻塞等待服务端执行，然后读取返回结果。优点是确保每次处理都有结果，一旦发现返回结果中有Error,就可以立即处理。

2. Pipeline模式

   Pipeline模式则是一次性发送多个命令，最后一次取回所有的返回结果，这种模式通过减少网络的往返时间和IO的读写次数，大幅度提高通信性能，但Pipeline不支持原子性，如果想保证原子性，可同时开启事务模式。

3. Transaction模式

   Transaction模式即开启Redis的事务管理，Pipeline可以在事务中，也可以不在事务中。事务模式开启后，所有的命令（除了 **EXEC** 、 **DISCARD** 、 **MULTI** 和 **WATCH** ）到达服务端以后，不会立即执行，会进入一个等待队列，等到收到下述四个命令时执行不同操作：

   - **EXEC**命令执行时， 服务器以先进先出（FIFO）的方式执行事务队列中的命令,当事务队列里的所有命令被执行完之后， 将回复队列作为自己的执行结果返回给客户端， 客户端从事务状态返回到非事务状态， 至此， 事务执行完毕。

   - **DISCARD**命令用于取消一个事务， 它清空客户端的整个事务队列， 然后将客户端从事务状态调整回非事务状态， 最后返回字符串 OK 给客户端， 说明事务已被取消。

   - **Redis** 的事务是不可嵌套的， 当客户端已经处于事务状态， 而客户端又再向服务器发送MULTI时， 服务器只是简单地向客户端发送一个错误， 然后继续等待其他命令的入队。 MULTI命令的发送不会造成整个事务失败， 也不会修改事务队列中已有的数据。

   - **WATCH**只能在客户端进入事务状态之前执行， 在事务状态下发送 WATCH命令会引发一个错误， 但它不会造成整个事务失败， 也不会修改事务队列中已有的数据（和前面处理 MULTI的情况一样）。

Jedis主要有两条业务逻辑：

1. 初始化的过程
2. 发送命令的过程。

#### Jedis对象

**Jedis**对象的继承关系：`Jedis`—>`BinaryJedis`- ->`BasicCommands` 、`BinaryJedisCommands`等，其中`BinaryJedis`组合了**Client**对象(`Client`—>`BinaryClient`—>`Connection`，**Connection**对象组合了**socket**、**输入输出**流等连接对象)

##### Jedis的初始化流程

```java
Jedis jedis = new Jedis("localhost", 6379, 15000);
Transaction t = jedis.multi();
Pipeline pipeline = jedis.pipelined();
```

 Jedis通过传入Redis服务器地址（host,port）开始初始化，然后在BinaryJedis里实例化Client。Client通过**Socket**维持客户端与Redis服务器的连接与沟通。
前文提到Transaction和Pipeline很相似，它们继承同一个基类`MultiKeyPipelineBase`。区别在于Transaction在实例化的时候，就自动发送**MULTI**命令，开启事务模式，而Pipeline则按情况手动开启，它们均依靠Client发送命令。以下是Transaction和Pipeline初始化的具体实现：

```java
//BinaryJedis类
public Transaction multi() {
client.multi();
transaction = new Transaction(client);
return transaction;
}
public Pipeline pipelined() {
pipeline = new Pipeline();
pipeline.setClient(client);
return pipeline;
}
```

 下面通过发送一个`get key` 的命令，看看，这三种模式是如何运转的。

#### JedisPool

##### JedisPool初始化

**JedisPool**的构造方法很多（可以改造成Builder Pattern，更清晰），可以通过**JedisConfig**进行配置

```java
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxActive(MAX_ACTIVE);
    config.setMaxIdle(MAX_IDLE);
    config.setMaxWait(MAX_WAIT);
    config.setMaxWait(MAX_WAIT);
    config.setTestOnBorrow(TEST_ON_BORROW);
    jedisPool = new JedisPool(config, ADDR, PORT, TIMEOUT, AUTH);
```

**JedisPool**的继承关系：`JedisPool` —>`JedisPoolAbstract`—>`Pool`

**JedisPool**的构造，依赖于抽象父类`Pool`的构造函数，如下：

```java
//Pool中内置的属性是commons-pool2的GenericObjectPool
    private final GenericObjectPool internalPool;

    public Pool(final GenericObjectPoolConfig poolConfig, PooledObjectFactory<T> factory) {
    initPool(poolConfig, factory);
  }

  public void initPool(final GenericObjectPoolConfig poolConfig, PooledObjectFactory<T> factory) {
        //...
    this.internalPool = new GenericObjectPool<T>(factory, poolConfig);
  }
```

**Pool**中内置的属性是**commons-pool2**的`GenericObjectPool`，即最终的连接池对象：

```java
    public GenericObjectPool(final PooledObjectFactory<T> factory,
            final GenericObjectPoolConfig config) {
                //...
        this.factory = factory;
        idleObjects = new LinkedBlockingDeque<>(config.getFairness());
                //...
    }
```

其中构造参数`final PooledObjectFactory factory`，是外层构造时传入的`JedisFactory`，其实现了接口`PoolObjectFactory`，并实现了`makeObject()`等相关方法，即产生**Jedis**类的真正工厂。

##### Jedis连接获取

上文提到的`makeObject()`真正被调用的时候，是在连接获取时，进行调用并创建Jedis对象 (当然是有条件的，即下文提到的存活队列中无可用对象，并未达到上限时)

```java
        private PooledObject<T> create() throws Exception {
      //判断连接池容量的逻辑...
        final PooledObject<T> p;
        try {
            p = factory.makeObject();
        } catch (final Exception e) {
            createCount.decrementAndGet();
            throw e;
        }finally {
            synchronized (makeObjectCountLock) {
                makeObjectCount--;
                //此处唤醒所有等待连接的线程
                makeObjectCountLock.notifyAll();
            }
        }
    }
```

##### Jedis连接关闭

这里必须要提到，在连接池对象中(`GenericObjectPool`)的一个重要属性：

```java
private final LinkedBlockingDeque<PooledObject<T>> idleObjects;
```

这个阻塞队列是维护存活**Jedis**对象的容器，当连接关闭时，**Jedis**对象归还连接池，即存入此队列，保持连接的存活。

```java
        final int maxIdleSave = getMaxIdle();
        if (isClosed() || maxIdleSave > -1 && maxIdleSave <= idleObjects.size()) {
                try {
                        destroy(p);
        } catch (final Exception e) {
            swallowException(e);
        }
        } else {        
                if (getLifo()) {
                        idleObjects.addFirst(p);
                } else {
                    idleObjects.addLast(p);
                }
    }
```

**工具类**

https://github.com/hanyunpeng0521/JavaLearn/blob/master/src/main/java/com/hyp/learn/redis/RedisUtils.java

#### Jedis工作模式的调用流程

##### client请求模式

以`get key` 为例，为突出主要步骤，部分代码略有缩减。
用法：

```
jedis.get("foo");
```

![MbrRT1.png](https://s2.ax1x.com/2019/11/23/MbrRT1.png)

调用流程:

![MbsXDJ.png](https://s2.ax1x.com/2019/11/23/MbsXDJ.png)

client模式下，发送请求，读取回复的具体实现。可以清楚看到，在每次发送命令前，会先通过`connect()`方法判断是否已经连接，若未连接则进行如下操作：

1. 实例化Socket，并配置，
2. 连接Socket,获取OutputStream和InputStream
3. 如果是SSL连接，则会通过SSLSocketFactory创建socket连接

 **Protocol**是一个通讯工具类，将Redis的各类执行关键字存储为静态变量，可以直观调用命令，例如`Protocol.Command.GET`。同时，将命令包装成符合[Redis的统一请求协议](http://www.redis.cn/topics/protocol.html)，回复消息的处理也是在这个类进行，先通过通讯协提取出当次请求的回复消息，将Object类型的消息，格式化为String,List等具体类型，如果回复消息有Error则以异常的形式抛出。

##### Pipeline和Transaction模式

Transaction和Pipeline两个类的的类结构。可以看到Pipeline和Transaction都继承自`MultiKeyPipelineBase`，其中，`MultiKeyPipelineBase`和`PipelineBase`的区别在于处理的命令不同，内部均调用Client发送命令。

从以下用例也可以看出两者的操作也十分类似。Pipeline有一个内部类对象`MultiResponseBuilder`，前文提到，当Redis事务结束时，会以List的形式，一次性返回所有命令的执行结果。`MultiResponseBuilder`对象就是用于，当Pipeline开始其实模式后，在事务结束时，存储所有返回结果。

Queable用一个LinkedList装入每个命令的返回结果，`Response`是一个泛型，`set(Object data)`方法传入格式化之前的结果，`get()`方法返回格式化之后的结果。

Pipeline的使用方法：

```java
Pipeline p = jedis.pipelined();
//只发送命令，不读取结果，此时的Response<T>没有数据
Response<String> string = p.get("string");
Response<String> list = p.lpop("list");
Response<String> hash = p.hget("hash", "foo");
Response<Set<String>> zset = p.zrange("zset", 0, -1);
Response<String> set = p.spop("set");
//一次读取所有response.此时的Response<T>有数据
p.sync();

assertEquals("foo", string.get());
assertEquals("foo", list.get());
assertEquals("bar", hash.get());
assertEquals("foo", zset.get().iterator().next());
assertEquals("foo", set.get());


//Transactions使用方法：
//开启事务
Transaction t = jedis.multi();
//命令进入服务端的待执行队列
Response<String> string = t.get("string");
Response<String> list = t.lpop("list");
Response<String> hash = t.hget("hash", "foo");
Response<Set<String>> zset = t.zrange("zset", 0, -1);
Response<String> set = t.spop("set");
//发送EXEC指令，执行所有命令，并返回结果
t.exec();

assertEquals("foo", string.get());
assertEquals("foo", list.get());
assertEquals("bar", hash.get());
assertEquals("foo", zset.get().iterator().next());
assertEquals("foo", set.get());
```

![MbywxU.png](https://s2.ax1x.com/2019/11/23/MbywxU.png)

Pipeline的调用流程:

![Mb6kWV.png](https://s2.ax1x.com/2019/11/23/Mb6kWV.png)

Pipeline从发送请求到读取回复的具体实现，Transaction的实现与其类似，因而没有另外做图说明。由上图可见，Pipeline通过Client发送命令，Client在`sendCommand`时，会同时执行`pipelinedCommands++`，记录发送命令的条数（参见图3-5）。之后，返回一个`Response`实例，并将这个实例塞入了`pipelinedResponses`队列中。`Response`主要有3个属性：

1. 格式化前的回复消息data,
2. 格式化后的回复消息response,
3. 格式化方式builder。

 刚发送消息后，`Response`里面的`data`和`response`是空值，只有格式化的方式`builder`。`Sync()`用于一次性读取所有回复，首先调用client的`getAll()`方法，`getAll()`方法根据之前记录的`pipelinedCommands`和Redis通讯协议，读取相同条数的回复消息到一个List，并返回给Pipeline。随后遍历这个List,逐个将回复消息赋给`pipelinedResponses`中每个`Response`的`data`。

 在执行`Response.get()`命令时，`Response`里面`data`已经有值了，但是是Object类型的，因而还要调用`build()`方法，做一次数据转换，返回格式化之后的数据。

以上就是Pipeline的主要工作流程。Transaction的`exec()`方法和`sync()`很相似，下文为`exec()`的具体实现。

```java
public List<Object> exec() {
  // 清空inputstream里面的所有数据，忽略QUEUED or ERROR回复
  client.getMany(getPipelinedResponseLength());
  //发送EXEC指令，让服务端执行所有命令
  client.exec();
  //事务结束
  inTransaction = false;
  //从inputStream中读取所有回复
  List<Object> unformatted = client.getObjectMultiBulkReply();
  if (unformatted == null) {
    return null;
  }
  //和sync（）一样
  List<Object> formatted = new ArrayList<Object>();
  for (Object o : unformatted) {
    try {
     formatted.add(generateResponse(o).get());
    } catch (JedisDataException e) {
    formatted.add(e);
    }
  }
  return formatted;
}
```

#### JedisCluster

![MbIo5D.png](https://s2.ax1x.com/2019/11/23/MbIo5D.png)

由于Jedis本身不是线程安全的，所以选择使用对象池JedisPool 来保证线程安全。在`JedisClusterInfoCache`中，除了要保存节点和槽的一一对应关系，还要为每个节点建立一个对象池`JedisPool`，并保存在`map`中。因而，这个类主要用于保存集群的配置信息，并且是JedisCluster初始化部分的核心所在。

`JedisClusterConnectionHandler`是cache类的一个窗口，cache类似数据管理层，而Handler就类似于操控数据提供服务的Service层。

**初始化**

初始化调用时序图：

![MbIH8H.png](https://s2.ax1x.com/2019/11/23/MbIH8H.png)



初始化详情：

![MbokMn.png](https://s2.ax1x.com/2019/11/23/MbokMn.png)

Jedis建立集群的过程很清晰，传入节点信息，通过其中一个节点从redis服务器拿到整个集群的信息信息，包括槽位对应关系，主从节点的信息，保存在JedisClusterInfoCache中。

**调用流程**

使用方法：

```java
Set<HostAndPort> jedisClusterNode = new HashSet<HostAndPort>();
jedisClusterNode.add(new HostAndPort("127.0.0.1", 7379));
JedisCluster jc = new JedisCluster(jedisClusterNode, DEFAULT_TIMEOUT, DEFAULT_TIMEOUT,
    DEFAULT_REDIRECTIONS, "cluster", DEFAULT_CONFIG);
jc.set("foo", "bar");
assertEquals("bar", jc.get("foo"));
```

![MboMRJ.png](https://s2.ax1x.com/2019/11/23/MboMRJ.png)



具体实现过程：

![MbotIO.png](https://s2.ax1x.com/2019/11/23/MbotIO.png)

在发送请求时，JedisCluster对象先从初始化得到的集群map中获取key对应的节点连接，即一个可用的Jedis对象。然后通过这个对象发送`get key` 命令。

通常，根据key计算槽位得到的节点不会报错。所以如果发生`connectionException`,或者`MovedDataException`,说明初始化得到的槽位与节点的对应关系有问题，即与实际的对应关系不符，应当重置map。 如果出现ASK异常，说明数据正在迁移，需要临时使用返回消息指定的地址，重新发送命令。在这里，Jedis通过异常反馈，智能地同步了客户端与服务端的集群信息。

#### JedisSentinel

JedisSentinel常用方式有两种：

1. 使用哨兵单节点拿到主节点，从节点的信息

   ```java
   //通过哨兵节点的信息新建Jedis实例，然后拿到Master节点的信息
   Jedis j = new Jedis(sentinel);
   List<Map<String, String>> masters = j.sentinelMasters();
   //拿到master的address，**MASTER_NAME**
   List<String> masterHostAndPort = j.sentinelGetMasterAddrByName(**MASTER_NAME**);
   HostAndPort masterFromSentinel = new HostAndPort(masterHostAndPort.get(0),Integer.parseInt(masterHostAndPort.get(1)));
   assertEquals(master, masterFromSentinel);
   //通过哨兵节点，拿到从节点的信息
   List<Map<String, String>> slaves = j.sentinelSlaves(**MASTER_NAME**);
   ```

2. 使用哨兵节点对象池

   ```java
   Set<String> sentinels = new HashSet<String>();
   sentinels.add(new HostAndPort("localhost", 65432).toString());
   sentinels.add(new HostAndPort("localhost", 65431).toString());
   JedisSentinelPool pool = new JedisSentinelPool(MASTER_NAME, sentinels);
   pool.destroy();
   ```

 `JedisSentinelPool`的结构清晰，内部使用对象池存放一个个`sentinel`实例。下图分别为`JedisSentinelPool`的类结构和初始化流程。在使用时，我们先根据，host,port等信息，初始化一个Jedis实例，然后可以通过`sentinelMasters()`，`sentinelGetMasterAddrByName(MASTER_NAME)`，`sentinelSlaves(MASTER_NAME)`等方法拿到这个哨兵节点监听的MASTER节点信息或对应的SLAVE节点信息。

![MboyeP.png](https://s2.ax1x.com/2019/11/23/MboyeP.png)

初始化流程：

![Mbo2FS.png](https://s2.ax1x.com/2019/11/23/Mbo2FS.png)

#### ShardedJedis

构建Jedis分片的方法（单节点与对象池的模式）

1. 单节点模式

   ```java
   List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>(2);
   //其中一个分片
   JedisShardInfo shard1 = new JedisShardInfo(redis1);
   shards.add(shard1);
   //另一个分片
   JedisShardInfo shard2 = new JedisShardInfo(redis2);
   shards.add(shard2);
   @SuppressWarnings("resource")
   //新建ShardedJedis实例
   ShardedJedis shardedJedis = new ShardedJedis(shards);
   shardedJedis.set("a", "bar");
   //通过key可以查看存储在哪个jedis
   JedisShardInfo ak = shardedJedis.getShardInfo("a");
   assertEquals(shard2, ak);
   ```

2. 对象池模式

   ```java
   ShardedJedisPool pool = new ShardedJedisPool(new GenericObjectPoolConfig(), shards);
   ShardedJedis jedis = pool.getResource();
   jedis.set("foo", "bar");
   assertEquals("bar", jedis.get("foo"));
   ```

![MbTeOI.png](https://s2.ax1x.com/2019/11/23/MbTeOI.png)

初始化流程：

![MbTQk8.png](https://s2.ax1x.com/2019/11/23/MbTQk8.png)

ShardedJedis的类结构（其内部保存一个对象池，与常规的JedisPool的不同之处在于，内部的`PooledObjectFactory`实现不同）,分片信息保存在基类`Sharded`中，`Sharded`保存了3个重要变量：

1. nodes是一个TreeMap,保存了每个分片节点和对应的hash值。
2. algo是计算hash值的函数，默认是MurmurHash,可替换。
3. resources是一个LinkedHashMap，存放着JedisShardinfo和一个Jedis实例的对应关系。

 `ShardedJedis`的初始化流程，通过传入待分片的节点信息，初始化好上述3个变量。在使用时，先根据key计算出hash值，在`nodes`中找到对应的分片信息，再在`resources`中找到对应的Jedis实例，然后通过这个Jedis实例才操作redis节点。

### 参考：

1. [Jedis源码分析（一）-Jedis介绍](https://segmentfault.com/a/1190000013052104)