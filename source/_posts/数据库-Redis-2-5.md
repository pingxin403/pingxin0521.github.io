---
title: Redis 常用客户端-redisson
date: 2019-11-20 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### 几种java客户端比较

> jedis、redisson、lettuce

<!--more-->

- Jedis是Redis的Java实现的客户端，其API提供了比较全面的Redis命令的支持；
- Jedis中的方法调用是比较底层的暴露的Redis的API，也即Jedis中的Java方法基本和Redis的API保持着一致，了解Redis的API，也就能熟练的使用Jedis。
- Redisson实现了分布式和可扩展的Java数据结构，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过Redis支持延迟队列。和Jedis相比，功能较为简单，不支持字符串操作，不支持排序、事务、管道、分区等Redis特性。Redisson的宗旨是促进使用者对Redis的关注分离，从而让使用者能够将精力更集中地放在处理业务逻辑上。
- Redisson中的方法则是进行比较高的抽象，每个方法调用可能进行了一个或多个Redis方法调用。
- Lettuce:高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。目前springboot默认使用的客户端。

**伸缩性**：

- Jedis：使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。
- Jedis仅支持基本的数据类型如：String、Hash、List、Set、Sorted Set。
- Redisson：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Redisson的API是线程安全的，所以可以操作单个Redisson连接来完成各种操作。
- Redisson不仅提供了一系列的分布式Java常用对象，基本可以与Java的基本数据结构通用，还提供了许多分布式服务，其中包括（BitSet, Set, Multimap, SortedSet, Map, List, Queue, BlockingQueue, Deque, BlockingDeque, Semaphore, Lock, AtomicLong, CountDownLatch, Publish / Subscribe, Bloom filter, Remote service, Spring cache, Executor service, Live Object service, Scheduler service）。
- Lettuce：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作。

### redisson

> [Redisson](https://redisson.org/)是架设在[Redis](http://redis.cn/)基础上的一个Java驻内存数据网格（In-Memory Data Grid）。充分的利用了Redis键值数据库提供的一系列优势，基于Java实用工具包中常用接口，为使用者提供了一系列具有分布式特性的常用工具类。使得原本作为协调单机多线程并发程序的工具包获得了协调分布式多机多线程并发系统的能力，大大降低了设计和研发大规模分布式系统的难度。同时结合各富特色的分布式服务，更进一步简化了分布式环境中程序相互之间的协作。

github地址：https://github.com/redisson/redisson

官方文档:<https://github.com/redisson/redisson/wiki/Redisson项目介绍>

相对于Jedis而言，Redisson强大的一批。当然了，随之而来的就是它的复杂性。它里面也实现了分布式锁，而且包含多种类型的锁，更多请参阅[分布式锁和同步器](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器)

#### Jedis实现分布式锁

分布式锁，是一种思想，它的实现方式有很多。比如，我们将沙滩当做分布式锁的组件，那么它看起来应该是这样的：

- **加锁**

  在沙滩上踩一脚，留下自己的脚印，就对应了加锁操作。其他进程或者线程，看到沙滩上已经有脚印，证明锁已被别人持有，则等待。

- **解锁**

  把脚印从沙滩上抹去，就是解锁的过程。

- **锁超时**

  为了避免死锁，我们可以设置一阵风，在单位时间后刮起，将脚印自动抹去。

分布式锁的实现有很多，比如基于数据库、memcached、Redis、系统文件、zookeeper等。它们的核心的理念跟上面的过程大致相同。

我们先来看如何通过单节点Redis实现一个简单的分布式锁。

1. 加锁

   加锁实际上就是在redis中，给Key键设置一个值，为避免死锁，并给定一个过期时间。

   ```
   SET lock_key random_value NX PX 5000
   ```

   值得注意的是：
    `random_value` 是客户端生成的唯一的字符串。
    `NX` 代表只在键不存在时，才对键进行设置操作。
    `PX 5000` 设置键的过期时间为5000毫秒。

   这样，如果上面的命令执行成功，则证明客户端获取到了锁。

2. 解锁

   解锁的过程就是将Key键删除。但也不能乱删，不能说客户端1的请求将客户端2的锁给删除掉。这时候`random_value`的作用就体现出来。

   为了保证解锁操作的原子性，我们用LUA脚本完成这一操作。先判断当前锁的字符串是否与传入的值相等，是的话就删除Key，解锁成功。

   ```lua
   if redis.call('get',KEYS[1]) == ARGV[1] then 
      return redis.call('del',KEYS[1]) 
   else
      return 0 
   end
   ```

**实现**

首先，我们在pom文件中，引入Jedis。在这里，笔者用的是最新版本，注意由于版本的不同，API可能有所差异。

```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.11.6</version>
</dependency>  
```

加锁的过程很简单，就是通过SET指令来设置值，成功则返回；否则就循环等待，在timeout时间内仍未获取到锁，则获取失败。

解锁我们通过`jedis.eval`来执行一段LUA就可以。将锁的Key键和生成的字符串当做参数传进来。

```java
@Service
public class RedisLock {

    Logger logger = LoggerFactory.getLogger(this.getClass());

    private String lock_key = "redis_lock"; //锁键

    protected long internalLockLeaseTime = 30000;//锁过期时间

    private long timeout = 999999; //获取锁的超时时间

    
    //SET命令的参数 
    SetParams params = SetParams.setParams().nx().px(internalLockLeaseTime);

    @Autowired
    JedisPool jedisPool;

    
    /**
     * 加锁
     * @param id
     * @return
     */
    public boolean lock(String id){
        Jedis jedis = jedisPool.getResource();
        Long start = System.currentTimeMillis();
        try{
            for(;;){
                //SET命令返回OK ，则证明获取锁成功
                String lock = jedis.set(lock_key, id, params);
                if("OK".equals(lock)){
                    return true;
                }
                //否则循环等待，在timeout时间内仍未获取到锁，则获取失败
                long l = System.currentTimeMillis() - start;
                if (l>=timeout) {
                    return false;
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }finally {
            jedis.close();
        }
    }
        /**
     * 解锁
     * @param id
     * @return
     */
    public boolean unlock(String id){
        Jedis jedis = jedisPool.getResource();
        String script =
                "if redis.call('get',KEYS[1]) == ARGV[1] then" +
                        "   return redis.call('del',KEYS[1]) " +
                        "else" +
                        "   return 0 " +
                        "end";
        try {
            Object result = jedis.eval(script, Collections.singletonList(lock_key), 
                                    Collections.singletonList(id));
            if("1".equals(result.toString())){
                return true;
            }
            return false;
        }finally {
            jedis.close();
        }
    }
}

```

最后，我们可以在多线程环境下测试一下。我们开启1000个线程，对count进行累加。调用的时候，关键是唯一字符串的生成。这里，笔者使用的是`Snowflake`算法。

```java
@Controller
public class IndexController {

    @Autowired
    RedisLock redisLock;
    
    int count = 0;
    
    @RequestMapping("/index")
    @ResponseBody
    public String index() throws InterruptedException {

        int clientcount =1000;
        CountDownLatch countDownLatch = new CountDownLatch(clientcount);

        ExecutorService executorService = Executors.newFixedThreadPool(clientcount);
        long start = System.currentTimeMillis();
        for (int i = 0;i<clientcount;i++){
            executorService.execute(() -> {
            
                //通过Snowflake算法获取唯一的ID字符串
                String id = IdUtil.getId();
                try {
                    redisLock.lock(id);
                    count++;
                }finally {
                    redisLock.unlock(id);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        long end = System.currentTimeMillis();
        logger.info("执行线程数:{},总耗时:{},count数为:{}",clientcount,end-start,count);
        return "Hello";
    }
}
```

至此，单节点Redis的分布式锁的实现就已经完成了。比较简单，但是问题也比较大，最重要的一点是，锁不具有可重入性。

#### redisson

**可重入锁**

上面我们自己实现的Redis分布式锁，其实不具有可重入性。那么下面我们先来看看Redisson中如何调用可重入锁。

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.10.1</version>
</dependency>
```

首先，通过配置获取RedissonClient客户端的实例，然后`getLock`获取锁的实例，进行操作即可。

```java
public static void main(String[] args) {

    Config config = new Config();
    config.useSingleServer().setAddress("redis://127.0.0.1:6379");
    config.useSingleServer().setPassword("redis1234");
    
    final RedissonClient client = Redisson.create(config);  
    RLock lock = client.getLock("lock1");
    
    try{
        lock.lock();
    }finally{
        lock.unlock();
    }
}
```

#### 单机模式

**获取锁实例**

我们先来看`RLock lock = client.getLock("lock1");`  这句代码就是为了获取锁的实例，然后我们可以看到它返回的是一个`RedissonLock`对象。

```cpp
public RLock getLock(String name) {
    return new RedissonLock(connectionManager.getCommandExecutor(), name);
}
```

在`RedissonLock`构造方法中，主要初始化一些属性。

```kotlin
public RedissonLock(CommandAsyncExecutor commandExecutor, String name) {
    super(commandExecutor, name);
    //命令执行器
    this.commandExecutor = commandExecutor;
    //UUID字符串
    this.id = commandExecutor.getConnectionManager().getId();
    //内部锁过期时间
    this.internalLockLeaseTime = commandExecutor.
                getConnectionManager().getCfg().getLockWatchdogTimeout();
    this.entryName = id + ":" + name;
}
```

**加锁**

当我们调用`lock`方法，定位到`lockInterruptibly`。在这里，完成了加锁的逻辑。

```java
public void lockInterruptibly(long leaseTime, TimeUnit unit) throws InterruptedException {
    
    //当前线程ID
    long threadId = Thread.currentThread().getId();
    //尝试获取锁
    Long ttl = tryAcquire(leaseTime, unit, threadId);
    // 如果ttl为空，则证明获取锁成功
    if (ttl == null) {
        return;
    }
    //如果获取锁失败，则订阅到对应这个锁的channel
    RFuture<RedissonLockEntry> future = subscribe(threadId);
    commandExecutor.syncSubscription(future);

    try {
        while (true) {
            //再次尝试获取锁
            ttl = tryAcquire(leaseTime, unit, threadId);
            //ttl为空，说明成功获取锁，返回
            if (ttl == null) {
                break;
            }
            //ttl大于0 则等待ttl时间后继续尝试获取
            if (ttl >= 0) {
                getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
            } else {
                getEntry(threadId).getLatch().acquire();
            }
        }
    } finally {
        //取消对channel的订阅
        unsubscribe(future, threadId);
    }
    //get(lockAsync(leaseTime, unit));
}
```

如上代码，就是加锁的全过程。先调用`tryAcquire`来获取锁，如果返回值ttl为空，则证明加锁成功，返回；如果不为空，则证明加锁失败。这时候，它会订阅这个锁的Channel，等待锁释放的消息，然后重新尝试获取锁。流程如下：

![QT9NUf.md.png](https://s2.ax1x.com/2019/12/17/QT9NUf.md.png)

1. 获取锁

   获取锁的过程是怎样的呢？接下来就要看`tryAcquire`方法。在这里，它有两种处理方式，一种是带有过期时间的锁，一种是不带过期时间的锁。

   ```java
   private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, final long threadId) {
   
       //如果带有过期时间，则按照普通方式获取锁
       if (leaseTime != -1) {
           return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
       }
       
       //先按照30秒的过期时间来执行获取锁的方法
       RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(
           commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(),
           TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
           
       //如果还持有这个锁，则开启定时任务不断刷新该锁的过期时间
       ttlRemainingFuture.addListener(new FutureListener<Long>() {
           @Override
           public void operationComplete(Future<Long> future) throws Exception {
               if (!future.isSuccess()) {
                   return;
               }
   
               Long ttlRemaining = future.getNow();
               // lock acquired
               if (ttlRemaining == null) {
                   scheduleExpirationRenewal(threadId);
               }
           }
       });
       return ttlRemainingFuture;
   }
   ```

   接着往下看，`tryLockInnerAsync`方法是真正执行获取锁的逻辑，它是一段LUA脚本代码。在这里，它使用的是hash数据结构。

   ```java
   <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit,     
                               long threadId, RedisStrictCommand<T> command) {
   
           //过期时间
           internalLockLeaseTime = unit.toMillis(leaseTime);
   
           return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
                     //如果锁不存在，则通过hset设置它的值，并设置过期时间
                     "if (redis.call('exists', KEYS[1]) == 0) then " +
                         "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                         "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                         "return nil; " +
                     "end; " +
                     //如果锁已存在，并且锁的是当前线程，则通过hincrby给数值递增1
                     "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                         "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                         "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                         "return nil; " +
                     "end; " +
                     //如果锁已存在，但并非本线程，则返回过期时间ttl
                     "return redis.call('pttl', KEYS[1]);",
           Collections.<Object>singletonList(getName()), 
                   internalLockLeaseTime, getLockName(threadId));
       }
   ```

   这段LUA代码看起来并不复杂，有三个判断：

   - **通过exists判断，如果锁不存在，则设置值和过期时间，加锁成功**
   - **通过hexists判断，如果锁已存在，并且锁的是当前线程，则证明是重入锁，加锁成功**
   - **如果锁已存在，但锁的不是当前线程，则证明有其他线程持有锁。返回当前锁的过期时间，加锁失败**

   ![QTCAJS.md.png](https://s2.ax1x.com/2019/12/17/QTCAJS.md.png)

   加锁成功后，在redis的内存数据中，就有一条hash结构的数据。Key为锁的名称；field为随机字符串+线程ID；值为1。如果同一线程多次调用`lock`方法，值递增1。

   ```shell
   127.0.0.1:6379> hgetall lock1
   1) "b5ae0be4-5623-45a5-8faa-ab7eb167ce87:1"
   2) "1"
   ```

**解锁**

我们通过调用`unlock`方法来解锁。

```java
public RFuture<Void> unlockAsync(final long threadId) {
    final RPromise<Void> result = new RedissonPromise<Void>();
    
    //解锁方法
    RFuture<Boolean> future = unlockInnerAsync(threadId);

    future.addListener(new FutureListener<Boolean>() {
        @Override
        public void operationComplete(Future<Boolean> future) throws Exception {
            if (!future.isSuccess()) {
                cancelExpirationRenewal(threadId);
                result.tryFailure(future.cause());
                return;
            }
            //获取返回值
            Boolean opStatus = future.getNow();
            //如果返回空，则证明解锁的线程和当前锁不是同一个线程，抛出异常
            if (opStatus == null) {
                IllegalMonitorStateException cause = 
                    new IllegalMonitorStateException("
                        attempt to unlock lock, not locked by current thread by node id: "
                        + id + " thread-id: " + threadId);
                result.tryFailure(cause);
                return;
            }
            //解锁成功，取消刷新过期时间的那个定时任务
            if (opStatus) {
                cancelExpirationRenewal(null);
            }
            result.trySuccess(null);
        }
    });

    return result;
}
```

然后我们再看`unlockInnerAsync`方法。这里也是一段LUA脚本代码。

```java
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, EVAL,
    
            //如果锁已经不存在， 发布锁释放的消息
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; " +
            "end;" +
            //如果释放锁的线程和已存在锁的线程不是同一个线程，返回null
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
            //通过hincrby递减1的方式，释放一次锁
            //若剩余次数大于0 ，则刷新过期时间
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            //否则证明锁已经释放，删除key并发布锁释放的消息
            "else " +
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
    Arrays.<Object>asList(getName(), getChannelName()), 
        LockPubSub.unlockMessage, internalLockLeaseTime, getLockName(threadId));

}
```

如上代码，就是释放锁的逻辑。同样的，它也是有三个判断：

- **如果锁已经不存在，通过publish发布锁释放的消息，解锁成功**
- **如果解锁的线程和当前锁的线程不是同一个，解锁失败，抛出异常**
- **通过hincrby递减1，先释放一次锁。若剩余次数还大于0，则证明当前锁是重入锁，刷新过期时间；若剩余次数小于0，删除key并发布锁释放的消息，解锁成功**

![QTCwo6.md.png](https://s2.ax1x.com/2019/12/17/QTCwo6.md.png)

至此，Redisson中的可重入锁的逻辑，就分析完了。但值得注意的是，上面的两种实现方式都是针对单机Redis实例而进行的。如果我们有多个Redis实例，请参阅**Redlock算法**。该算法的具体内容，请参考http://redis.cn/topics/distlock.html

#### 哨兵模式

即sentinel模式，实现代码和单机模式几乎一样，唯一的不同就是Config的构造：

```java
Config config = new Config();
config.useSentinelServers().addSentinelAddress(
        "redis://172.29.3.245:26378","redis://172.29.3.245:26379", "redis://172.29.3.245:26380")
        .setMasterName("mymaster")
        .setPassword("a123456").setDatabase(0);
```

#### 集群模式

集群模式构造Config如下：

```java
Config config = new Config();
config.useClusterServers().addNodeAddress(
        "redis://172.29.3.245:6375","redis://172.29.3.245:6376", "redis://172.29.3.245:6377",
        "redis://172.29.3.245:6378","redis://172.29.3.245:6379", "redis://172.29.3.245:6380")
        .setPassword("a123456").setScanInterval(5000);
```

#### 总结

普通分布式实现非常简单，无论是那种架构，向Redis通过EVAL命令执行LUA脚本即可。

#### Redlock分布式锁

说道Redis分布式锁大部分人都会想到：`setnx+lua`，或者知道`set key value px milliseconds nx`。后一种方式的核心实现命令如下：

```lua
- 获取锁（unique_value可以是UUID等）
SET resource_name unique_value NX PX 30000

- 释放锁（lua脚本中，一定要比较value，防止误解锁）
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

这种实现方式有3大要点（也是面试概率非常高的地方）：

1. set命令要用`set key value px milliseconds nx`；
2. value要具有唯一性；
3. 释放锁时要验证value值，不能误解锁；

事实上这类琐最大的缺点就是它加锁时只作用在一个Redis节点上，即使Redis通过sentinel保证高可用，如果这个master节点由于某些原因发生了主从切换，那么就会出现锁丢失的情况：

1. 在Redis的master节点上拿到了锁；
2. 但是这个加锁的key还没有同步到slave节点；
3. master故障，发生故障转移，slave节点升级为master节点；
4. 导致锁丢失。

正因为如此，Redis作者antirez基于分布式环境下提出了一种更高级的分布式锁的实现方式：**Redlock**。

antirez提出的redlock算法大概是这样的：

在Redis的分布式环境中，我们假设有N个Redis master。这些节点**完全互相独立，不存在主从复制或者其他集群协调机制**。我们确保将在N个实例上使用与在Redis单实例下相同方法获取和释放锁。现在我们假设有5个Redis master节点，同时我们需要在5台服务器上面运行这些Redis实例，这样保证他们不会同时都宕掉。

为了取到锁，客户端应该执行以下操作:

- 获取当前Unix时间，以毫秒为单位。
- 依次尝试从5个实例，使用相同的key和**具有唯一性的value**（例如UUID）获取锁。当向Redis请求获取锁时，客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试去另外一个Redis实例请求获取锁。
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。**当且仅当从大多数**（N/2+1，这里是3个节点）**的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
- 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在**所有的Redis实例上进行解锁**（即便某些Redis实例根本就没有加锁成功，防止某些节点获取到锁但是客户端没有得到响应而导致接下来的一段时间不能被重新获取锁）。

**实现**

redisson已经有对redlock算法封装，接下来对其用法进行简单介绍，并对核心源码进行分析（假设5个redis实例）。

首先，我们来看一下redission封装的redlock算法实现的分布式锁用法，非常简单，跟重入锁（ReentrantLock）有点类似：

```java
Config config = new Config();
config.useSentinelServers().addSentinelAddress("127.0.0.1:6369","127.0.0.1:6379", "127.0.0.1:6389")
        .setMasterName("masterName")
        .setPassword("password").setDatabase(0);
RedissonClient redissonClient = Redisson.create(config);
// 还可以getFairLock(), getReadWriteLock()
RLock redLock = redissonClient.getLock("REDLOCK_KEY");
boolean isLock;
try {
    isLock = redLock.tryLock();
    // 500ms拿不到锁, 就认为获取锁失败。10000ms即10s是锁失效时间。
    isLock = redLock.tryLock(500, 10000, TimeUnit.MILLISECONDS);
    if (isLock) {
        //TODO if get lock success, do something;
    }
} catch (Exception e) {
} finally {
    // 无论如何, 最后都要解锁
    redLock.unlock();
}
```

最核心的变化就是`RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);`，因为我这里是以三个节点为例。

那么如果是哨兵模式呢？需要搭建3个，或者5个sentinel模式集群（具体多少个，取决于你）。

那么如果是集群模式呢？需要搭建3个，或者5个cluster模式集群（具体多少个，取决于你）。

**唯一ID**

实现分布式锁的一个非常重要的点就是set的value要具有唯一性，redisson的value是怎样保证value的唯一性呢？答案是**UUID+threadId**。入口在redissonClient.getLock("REDLOCK_KEY")，源码在Redisson.java和RedissonLock.java中：

```java
protected final UUID id = UUID.randomUUID();
String getLockName(long threadId) {
    return id + ":" + threadId;
}
```

**获取锁**

获取锁的代码为redLock.tryLock()或者redLock.tryLock(500, 10000, TimeUnit.MILLISECONDS)，两者的最终核心源码都是下面这段代码，只不过前者获取锁的默认租约时间（leaseTime）是LOCK_EXPIRATION_INTERVAL_SECONDS，即30s：

```java
<T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
    internalLockLeaseTime = unit.toMillis(leaseTime);
    // 获取锁时向5个redis实例发送的命令
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, command,
              // 首先分布式锁的KEY不能存在，如果确实不存在，那么执行hset命令（hset REDLOCK_KEY uuid+threadId 1），并通过pexpire设置失效时间（也是锁的租约时间）
              "if (redis.call('exists', KEYS[1]) == 0) then " +
                  "redis.call('hset', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              // 如果分布式锁的KEY已经存在，并且value也匹配，表示是当前线程持有的锁，那么重入次数加1，并且设置失效时间
              "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                  "redis.call('hincrby', KEYS[1], ARGV[2], 1); " +
                  "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                  "return nil; " +
              "end; " +
              // 获取分布式锁的KEY的失效时间毫秒数
              "return redis.call('pttl', KEYS[1]);",
              // 这三个参数分别对应KEYS[1]，ARGV[1]和ARGV[2]
                Collections.<Object>singletonList(getName()), internalLockLeaseTime, getLockName(threadId));
}
```

获取锁的命令中，

- **KEYS[1]**就是Collections.singletonList(getName())，表示分布式锁的key，即REDLOCK_KEY；
- **ARGV[1]**就是internalLockLeaseTime，即锁的租约时间，默认30s；
- **ARGV[2]**就是getLockName(threadId)，是获取锁时set的唯一值，即UUID+threadId：

**释放锁**

释放锁的代码为redLock.unlock()，核心源码如下：

```java
protected RFuture<Boolean> unlockInnerAsync(long threadId) {
    // 向5个redis实例都执行如下命令
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            // 如果分布式锁KEY不存在，那么向channel发布一条消息
            "if (redis.call('exists', KEYS[1]) == 0) then " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; " +
            "end;" +
            // 如果分布式锁存在，但是value不匹配，表示锁已经被占用，那么直接返回
            "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then " +
                "return nil;" +
            "end; " +
            // 如果就是当前线程占有分布式锁，那么将重入次数减1
            "local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); " +
            // 重入次数减1后的值如果大于0，表示分布式锁有重入过，那么只设置失效时间，还不能删除
            "if (counter > 0) then " +
                "redis.call('pexpire', KEYS[1], ARGV[2]); " +
                "return 0; " +
            "else " +
                // 重入次数减1后的值如果为0，表示分布式锁只获取过1次，那么删除这个KEY，并发布解锁消息
                "redis.call('del', KEYS[1]); " +
                "redis.call('publish', KEYS[2], ARGV[1]); " +
                "return 1; "+
            "end; " +
            "return nil;",
            // 这5个参数分别对应KEYS[1]，KEYS[2]，ARGV[1]，ARGV[2]和ARGV[3]
            Arrays.<Object>asList(getName(), getChannelName()), LockPubSub.unlockMessage, internalLockLeaseTime, getLockName(threadId));

}
```

##### 实现原理

既然核心变化是使用了**RedissonRedLock**，那么我们看一下它的源码有什么不同。这个类是**RedissonMultiLock**的子类，所以调用tryLock方法时，事实上调用了**RedissonMultiLock**的tryLock方法，精简源码如下：

```csharp
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    // 实现要点之允许加锁失败节点限制
    int failedLocksLimit = failedLocksLimit();
    List<RLock> acquiredLocks = new ArrayList<RLock>(locks.size());
    // 实现要点之遍历所有节点通过EVAL命令执行lua加锁
    for (ListIterator<RLock> iterator = locks.listIterator(); iterator.hasNext();) {
        RLock lock = iterator.next();
        boolean lockAcquired;
        try {
            // 对节点尝试加锁
            lockAcquired = lock.tryLock(awaitTime, newLeaseTime, TimeUnit.MILLISECONDS);
        } catch (RedisConnectionClosedException|RedisResponseTimeoutException e) {
            // 如果抛出这类异常，为了防止加锁成功，但是响应失败，需要解锁
            unlockInner(Arrays.asList(lock));
            lockAcquired = false;
        } catch (Exception e) {
            // 抛出异常表示获取锁失败
            lockAcquired = false;
        }
        
        if (lockAcquired) {
            // 成功获取锁集合
            acquiredLocks.add(lock);
        } else {
            // 如果达到了允许加锁失败节点限制，那么break，即此次Redlock加锁失败
            if (locks.size() - acquiredLocks.size() == failedLocksLimit()) {
                break;
            }               
        }
    }
    return true;
}
```

很明显，这段源码就是上一篇文章《[Redlock：Redis分布式锁最牛逼的实现](https://mp.weixin.qq.com/s/JLEzNqQsx-Lec03eAsXFOQ)》提到的Redlock算法的完全实现。

以sentinel模式架构为例，如下图所示，有sentinel-1，sentinel-2，sentinel-3总计3个sentinel模式集群，如果要获取分布式锁，那么需要向这3个sentinel集群通过EVAL命令执行LUA脚本，需要3/2+1=2，即至少2个sentinel集群响应成功，才算成功的以Redlock算法获取到分布式锁：

![QTPhu9.png](https://s2.ax1x.com/2019/12/17/QTPhu9.png)

##### 问题合集

![QTiXGT.md.png](https://s2.ax1x.com/2019/12/17/QTiXGT.md.png)

根据上面实现原理的分析，这位同学应该是对Redlock算法实现有一点点误解，假设我们用5个节点实现Redlock算法的分布式锁。那么**要么是5个redis单实例，要么是5个sentinel集群，要么是5个cluster集群**。而不是一个有5个主节点的cluster集群，然后向每个节点通过EVAL命令执行LUA脚本尝试获取分布式锁，如上图所示。

- 失效时间如何设置

  这个问题的场景是，假设设置失效时间10秒，如果由于某些原因导致10秒还没执行完任务，这时候锁自动失效，导致其他线程也会拿到分布式锁。

  这确实是Redis分布式最大的问题，不管是普通分布式锁，还是Redlock算法分布式锁，都没有解决这个问题。也有一些文章提出了对失效时间续租，即延长失效时间，很明显这又提升了分布式锁的复杂度。另外就笔者了解，没有现成的框架有实现，如果有哪位知道，可以告诉我，万分感谢。

- redis分布式锁的高可用

  关于Redis分布式锁的安全性问题，在分布式系统专家Martin Kleppmann和Redis的作者antirez之间已经发生过一场争论。有兴趣的同学，搜索"**基于Redis的分布式锁到底安全吗**"就能得到你想要的答案，需要注意的是，有上下两篇（这应该就是传说中的神仙打架吧，哈）。

- zookeeper or redis

  没有绝对的好坏，只有更适合自己的业务。就性能而言，redis很明显优于zookeeper；就分布式锁实现的健壮性而言，zookeeper很明显优于redis。如何选择，取决于你的业务！

### 参考

1. [分布式锁之Redis实现](https://www.jianshu.com/p/47fd7f86c848)
2. Jedis api 在线网址：http://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html
3. redisson 官网地址：https://redisson.org/
4. redisson git项目地址：https://github.com/redisson/redisson
5. lettuce 官网地址：https://lettuce.io/
6. lettuce git项目地址：https://github.com/lettuce-io/lettuce-core
7. https://www.cnblogs.com/yangzhilong/p/7605807.html
8. <https://github.com/redisson/redisson/wiki/目录>
9. [Redlock：Redis分布式锁最牛逼的实现](https://mp.weixin.qq.com/s/JLEzNqQsx-Lec03eAsXFOQ)
10. [Redisson实现Redis分布式锁的N种姿势](https://www.jianshu.com/p/f302aa345ca8)