---
title: Redis 事务、发布订阅
date: 2019-10-18 20:18:59
tags:
 - NoSQL
 - Redis
 - 数据库
categories:
 - NoSQL
 - Redis
---

### 事务

Redis 通过 MULTI 、EXEC、 DISCARD  和 WATCH 四个命令来实现事务功能。

1. MULTI ：标记一个事务块的开始。

2. EXEC： 执行所有事务块内的命令。
3. DISCARD ：取消事务，放弃执行事务块内的所有命令
4. WATCH key [key ...] ：监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

<!--more-->

1. 是什么？

   可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，不许加塞！

2. 能干什么？

    一个队列中，一次性的、顺序性的、排他性的执行一系列的命令。要么一起成功，要么一起失败。排好队，一次性的执行多个redis的命令！

3. 怎么玩？
   通过MULTI指令开启，之后输入多个命令！Redis将它们加入到队列当中，所有的命令通过EXEC来开启执行！通过DISCARD来放弃本次的批处理操作！

   - 正常执行（好比购物先加入购物车，最后EXEC统一结账。）

     ```
     127.0.0.1:6379[1]> MULTI
     OK
     127.0.0.1:6379[1]> set k1 val1
     QUEUED
     127.0.0.1:6379[1]> set k2 val2
     QUEUED
     127.0.0.1:6379[1]> set k3 val3
     QUEUED
     127.0.0.1:6379[1]> set k4 val4
     QUEUED
     127.0.0.1:6379[1]> EXEC
     OK
     OK
     OK
     OK
     ```

   - DISCARD 撤销所有操作！

   - 加入队列时不出错，下面的都将正常运行，执行时出错不影响其他语句！

**注意**：

1. 当遇到执行错误时，redis放过这种错误，保证事务执行完成。（所以说redis的事务并不是保证原子性）
2. 与mysql中事务不同，在redis事务遇到执行错误的时候，不会进行回滚，而是简单的放过了，并保证其他的命令正常执行。这个区别在实现业务的时候，需要自己保证逻辑符合预期。
3. 当事务的执行过程中，如果redis意外的挂了。很遗憾只有部分命令执行了，后面的也就被丢弃了。当然如果我们使用的append-only file方式持久化，redis会用单个write操作写入整个事务内容。即是是这种方式还是有可能只部分写入了事务到磁盘。发生部分写入事务的情况 下，redis重启时会检测到这种情况，然后失败退出。可以使用redis-check-aof工具进行修复，修复会删除部分写入的事务内容。修复完后就 能够重新启动了。

#### WATCH

WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回空多条批量回复（null multi-bulk reply）来表示事务已经失败。

**首先我们不加WATCH进行事务处理**

![UTOOLS1574606164142.png](https://i.loli.net/2019/11/24/65xKXNu1LPZk3ne.png)

**添加WARCH**

![UTOOLS1574606200362.png](https://i.loli.net/2019/11/24/8sWbAmPCokiNG4r.png)



### 发布订阅

Redis的列表类型键可以用来实现队列，并且支持阻塞式读取，可以很容易的实现一个高性能的优先队列。同时在更高层面上，Redis还支持"发布/订阅"的消息模式，可以基于此构建一个聊天系统。

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

![UTOOLS1574606986152.png](https://i.loli.net/2019/11/24/9ZogvPukQxhp3Gb.png)

Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![UTOOLS1574606745620.png](https://i.loli.net/2019/11/24/DBEtmLUFPq9Sk6c.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![UTOOLS1574606773214.png](https://i.loli.net/2019/11/24/CWJj6hnP9dQuyq2.png)

Redis通过publish和subscribe命令实现订阅和发布的功能。订阅者可以通过subscribe向redis server订阅自己感兴趣的消息类型。redis将信息类型称为通道(channel)。当发布者通过publish命令向redis server发送特定类型的信息时，订阅该消息类型的全部订阅者都会收到此消息。

1. RedisServer中可以创建若干channel

2. 一个订阅者可以订阅多个channel

3. 当发布者向一个频道中发布一条消息时，所有的订阅者都将会收到消息

4. Redis的发布订阅模型没有消息积压功能，即新加入的订阅者收不到发布者之前发布的消息

5. 当订阅者收到消息时，消息内容如下 

   - 第一行：固定内容message

   - 第二行：channel的名称
   - 第三行：收到的新消息

**发布订阅的 API**

|                 命令                  |                含义                |
| :-----------------------------------: | :--------------------------------: |
|        publish channel message        |     向指定的channel中发布消息      |
|   subscribe channel1 [channel2...]    |   订阅给定的一个或多个渠道的消息   |
|  unsubcribe [channel1 [channel2...]]  | 取消订阅给定的一个或多个渠道的消息 |
|   psubscribe pattern1 [pattern2...]   |  订阅一个或多个符合给定模式的频道  |
| punsubscribe [pattern1 [pattern2...]] |       退订所有给定模式的频道       |
|            pubsub channel             |     列出至少有一个订阅者的频道     |
|      pubsub numsub [channel...]       |      列出给定频道的订阅者数量      |

**示例**

客户端1订阅CCTV1:

```
127.0.0.1:6379> subscribe CCTV1
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "CCTV1"
3) (integer) 1
```

客户端2订阅CCTV1和CCTV2:

```
127.0.0.1:6379> subscribe CCTV1 CCTV2
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "CCTV1"
3) (integer) 1
1) "subscribe"
2) "CCTV2"
3) (integer) 2
```

此时这两个客户端分别监听这指定的频道。现在另一个客户端向服务器推送了关于这两个频道的信息。

```
127.0.0.1:6379> publish CCTV1 "cctv1 is good"
(integer) 2
//返回2表示两个客户端接收了次消息。被接收到消息的客户端如下所示。
1) "message"
2) "CCTV1"
3) "cctv1 is good"
----
1) "message"
2) "CCTV1"
3) "cctv1 is good"
```

如上的订阅/发布也称订阅发布到频道(使用publish与subscribe命令)，此外还有订阅发布到模式(使用psubscribe来订阅一个模式)

订阅CCTV的全部频道

```
127.0.0.1:6379> psubscribe CCTV*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "CCTV*"
3) (integer) 1
```

当依然先如上推送一个CCTV1的消息时，该客户端正常接收。

#### 消息队列和发布订阅区别

我们来看一张消息队列通信模型的图：

![UTOOLS1574607098051.png](https://i.loli.net/2019/11/24/er9HcVLwg2tMxPG.png)

可以看到：

> 发布订阅模式是将消息通知每一个订阅者，消息队列是消息发布者发表消息后只有一个消息订阅者收得到，订阅者争抢接受消息，这是一个抢的功能。

**发布订阅 - Jedis**

```java
//订阅
public void testSubscribe() {
    Jedis jedis = new Jedis("127.0.0.1" , 6381);
    jedis.subscribe(new JedisPubSub() {
        @Override
        public void onMessage(String channel, String message) {
            System.out.println("receive channel ["+channel+"] message ["+message+"]");
        }
    } , "aliTV" , "googleTV");
}

//发布
public void testPublish() {
    Jedis jedis = new Jedis("127.0.0.1" , 6381);
    jedis.publish("aliTV" , "I am xxx");
    jedis.publish("googleTV" , "My age is 21");
}
```

### 参考

1. [订阅与发布](https://redisbook.readthedocs.io/en/latest/feature/pubsub.html)
2. [redis 学习（12）-- redis 发布订阅](https://www.jianshu.com/p/10d70352d132)