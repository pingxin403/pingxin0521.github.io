---
title: Java 常见异常和处理方法（二）
date: 2020-07-05 08:58:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

<!--more-->

### JVM常见异常

| 区域                 | 作用                                                         | 异常                                                         | 控制参数                                                     | 解决思路                                                     |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| java堆               | 存放对象的实例。                                             | java.lang.OutOfMemory Error:Java heap space                  | -Xms（初始化堆）,-Xmx（最大堆）,-Xmn（新生代），-XX:+PrintStringTableStatistics（1.8，显示字符串池数据），-XX:StringTableSize（1.8，字符串池大小） | 1、先查看是不是内存泄漏（内存中的对象是不是必须的），如果是泄漏，则找到与GC root 的路径解决泄漏。2、看物理内存是否允许加大-Xms,-Xmx。3、检查堆中是不是有对象实例一直在内存中没有释放。4、技巧让-Xms = -Xmx，减少内存扩展的开销。 |
| 虚拟机栈和本地方法栈 | 控制方法的执行。                                             | 如果线程请求的栈深度大于虚拟机所允许的最大深度：java.lang.StackOverflow Error如果续集你在扩展栈时无法申请到足够的内存空间：java.lang.OUtOfMemory Error | -Xss                                                         | 1、加大-Xss参数。2、减少线程。3、更换64位虚拟机。4、减少最大堆（Xmx）。 |
| 方法区               | 用于存放Class的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。 | java.OutOfMemory Error:PermGen space                         | -XX:PermSize-XX:MaxPermSize                                  | 这是最常见的内存溢出异常。1、加大参数（治标不治本）。2、增加参数：-XX:PrintGCDetails，-XX:+PrintGCTimeStamps和-XX:+PrintGCDateStamps，还可以指定日志输出到文件：`-Xloggc：<fileName>`。通过查看GC日志来查看新生代GC(Minor GC)和老年代GC（Full GC或MajorGC）的回收效率和回收频率。通常Minor GC会很频繁，回收效率高才是对。3、稍微的加大Survivor1 Survivor2的空间。 |
| 本机直接内存         | 如NIO，就是直接通过此内存实现。                              | java.lang.OutOfMemory Error,有allocate、Native字样           | -MaxDirectMemorySize如不指定则与-Xmx一致。                   | 合理规划服务器内存，预留足够的内存给本地内存。加大参数。     |
| 元空间               | 1.8以后的持久代改为元空间                                    | java.lang.OutOfMemoryError: Metaspace                        | -XX:MaxMetaspaceSize指定元空间大小                           |                                                              |

空间关系：

- java堆 = 新生代 + 老年代。
- 新生代 = Eden（伊甸园） + Survivor1 + Survivor2。

大多数情况下，对象在新生代Eden区中分配。当Eden中没有足够的空间时，就触发一次Minor GC。Survivor2 是Minor GC时用于复制用的。当发生Minor GC时如果发现所要辅助的对象（活着的对象）无法存放到Survivor2中，则通过分配担保机制提前转移到老年代中去。

如配置`-Xms20M，-XMx20M，-Xmn10M，-XX:SurvivorRatio=8`。

含义：java堆大小为20M，不可扩展，其中10MB分配给新生代，剩下的10MB分给老年代。-XX:SurvivorRatio=8 表示新生代中Eden区与其中一个Survivor区的空间比例为8：1。即 `eden space 8192k，from space 1024k，to space 1024k`。新生代总可用空间为9216kb（Eden区+1个Survivor区的总量）。

#### 其他错误

1. **java.lang.OutOfMemoryError:GC over head limit exceeded**

   这种情况是当系统处于高频的GC状态，而且回收的效果依然不佳的情况，就会开始报这个错误，这种情况一般是产生了很多不可以被释放的对象，有可能是引用使用不当导致，或申请大对象导致，但是java heap space的内存溢出有可能提前不会报这个错误，也就是可能内存就直接不够导致，而不是高频GC.

   可以使用`-XX:-UseGCOverheadLimit`控制输出堆错误而不是GC错误

2. **java.lang.OutOfMemoryError: unable to create new native thread**：内存不够

   上面第四种溢出错误，已经说明了线程的内存空间，其实线程基本只占用heap以外的内存区域，也就是这个错误说明除了heap以外的区域，无法为线程分配一块内存区域了，这个要么是内存本身就不够，要么heap的空间设置得太大了，导致了剩余的内存已经不多了，而由于线程本身要占用内存，所以就不够用了，说明了原因，如何去修改，不用我多说，你懂的。

3. **java.lang.OutOfMemoryError: request {} byte for {}out of swap**：地址空间不够

4. **Exception in thread “main”: java.lang.OutOfMemoryError: Requested array size exceeds VM limit**：创建的数组大于堆内存的空间