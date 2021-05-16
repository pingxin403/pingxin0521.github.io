---
title: java  面试题
date: 2019-09-07 14:18:59
tags:
 - 面试题
 - java
categories:
 - 面试题
 - java
---

面试书籍：

1. [JavaGuide](https://github.com/Snailclimb/JavaGuide)
2. [CS-Notes](https://github.com/CyC2018/CS-Notes)

<!--more-->

### java面试题

熟练掌握java是很关键的，大公司不仅仅要求你会使用几个api，更多的是要你熟悉源码实现原理，甚至要你知道有哪些不足，怎么改进，还有一些java有关的一些算法，设计模式等等。

##### **（一） java基础面试知识点**

1. java中==和equals和hashCode的区别

   ==:基本数据类型比较值；引用类型比较指向对象的内存地址

   equals()：对象方法，继承自Object对象，用于对象比较，若不重写，则比较指向对象的地址值;若重写，则依据重写规则。

   hashCode():对象方法，继承自Object对象,用于集合类中的元素增删。

   ==和equals和hashCode的区别:<https://blog.csdn.net/hla199106/article/details/46907725>

   hashCode:<https://blog.csdn.net/m0_38101105/article/details/82420213>

2. int、char、long各占多少字节数

   <https://blog.csdn.net/qq_31615049/article/details/80574551>

3. int与Integer的区别

   <https://www.cnblogs.com/guodongdidi/p/6953217.html>

4. 探探对java多态的理解

   多态，顾名思义，就是意味着某一时刻程序对应着多个可能的状态，在面向对象里，分为两种多态，第一种是编译时多态，主要指方法的重载;第二种是运行时多态，通过动态绑定来实现，这是我们更常说的多态。

   在静态状态下，由父类引用指向子类对象，程序实际运行过程中，引用变量的具体类型以及编译方法唯一确定。Java的多态，核心思想就是，在不修改代码的前提下，让引用变量同时绑定在多个类的实现方法上，导致运行时该引用变量方法随之改变，让程序可以在多个运行状态中进行选择。

   多态发生的几个必要条件：

   - 继承，从而出现多个不同子类；
   - 重写，在子类中覆盖父类的方法；
   - 向上转型，引用变量只能访问父类中拥有的方法和属性，而对于子类中存在而父类中不存在的方法，是不能使用的；

   <https://www.jianshu.com/p/b5ecb7f836da>

5. String、StringBuffer、StringBuilder区别

   <https://blog.csdn.net/itchuxuezhe_yang/article/details/89966303>

6. 什么是内部类？内部类的作用

   在java语言中，可以把一个类定义到另外一个类的内部，在类里面的这个类就叫内部类，外面的类就叫外部类。

   1. 隐藏你不想让别人知道的操作，也即封装性。
   2. 一个内部类对象可以访问创建它的外部类对象的内容，甚至包括私有变量！

7. 抽象类和接口区别

   接口更多的是在系统架构设计方法发挥作用，主要用于定义模块之间的通信契约。而抽象类在代码实现方面发挥作用，可以实现代码的重用

8. 抽象类的意义

   抽象类：
    一个类中如果包含抽象方法，这个类应该用abstract关键字声明为抽象类。

   意义：

   - 为子类提供一个公共的类型；

   - 封装子类中重复内容（成员变量和方法）；

   - 定义有抽象方法，子类虽然有不同的实现，但该方法的定义是一致的。

9. 抽象类与接口的应用场景

   接口（interface）的应用场合：

   1. 类与类之前需要特定的接口进行协调，而不在乎其如何实现。
   2. 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识。
   3. 需要将一组类视为单一的类，而调用者只通过接口来与这组类发生联系。
   4. 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系。

   抽象类（abstract class）的应用场合：

   一句话，在既需要统一的接口，又需要实例变量或缺省的方法的情况下，就可以使用它。最常见的有：

   1. 定义了一组接口，但又不想强迫每个实现类都必须实现所有的接口。可以用abstract class定义一组方法体，甚至可以是空方法体，然后由子类选择自己所感兴趣的方法来覆盖。
   2. 某些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中表示状态的变量来区别不同的关系。abstract的中介作用可以很好地满足这一点。
   3. 规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能

10. 抽象类是否可以没有方法和属性？

    抽象类专用于派生出子类，子类必须实现抽象类所声明的抽象方法，否则，子类仍是抽象类。

    包含抽象方法的类一定是抽象类，但抽象类中的方法不一定是抽象方法。

    抽象类中可以没有抽象方法，但有抽象方法的一定是抽象类。所以，java中 抽象类里面可以没有抽象方法。比如HttpServlet类。抽象类和普通类的区别就在于，抽象类不能被实例化，就是不能被new出来，即使抽象类里面没有抽象方法。

    抽象类的作用在于子类对其的继承和实现，也就是多态；而没有抽象方法的抽象类的存在价值在于：实例化了没有意义，因为类已经定义好了，不能改变其中的方法体，但是实例化出来的对象却满足不了要求，只有继承并重写了他的子类才能满足要求。所以才把它定义为没有抽象方法的抽象类

11. 接口的意义

    抽象类和接口所反映的设计理念是不同的，抽象类所代表的是“is-a”的关系，而接口所代表的是“like-a”的关系。

12. 泛型中extends和super的区别

    泛型中extends的主要作用是设定类型通配符的上限，super与extends是完全相反的，其定义的是下界通配符。

13. 父类的静态方法能否被子类重写

    先给一个答案，不能，父类的静态方法能够被子类继承，但是不能够被子类重写，即使子类中的静态方法与父类中的静态方法完全一样，也是两个完全不同的方法。

    ```java
    class Fruit{
    
        static String color = "五颜六色";
        static public void call() {
            System.out.println("这是一个水果");
        }
    }
    
    public class Banana extends Fruit{
    
        static String color = "黄色";
        static public void call() {
            System.out.println("这是一个香蕉");
        }
    
        public static void main(String[] args) {
    
            Fruit fruit = new Banana();
            System.out.println(fruit.color);    //五颜六色
            fruit.call();         //这是一个水果
        }
    }
    ```

    重写指的是根据运行时对象的类型来决定调用哪个方法，而不是根据编译时的类型。

    对于静态方法和静态变量来说，虽然在上述代码中使用对象来进行调用，但是底层上还是使用父类来调用的，静态变量和静态方法在编译的时候就将其与类绑定在一起。既然它们在编译的时候就决定了调用的方法、变量，那就和重写没有关系了。

14. 为什么不能根据返回类型来区分重载

    因为调用时不能指定类型信息，编译器不知道你要调用哪个函数。

15. 进程和线程的区别

    https://www.jianshu.com/p/2dc01727be45

16. final，finally，finalize的区别

17. 序列化的方式

18. Serializable 和Parcelable 的区别

    https://www.cnblogs.com/berylqliu/p/6261479.html

19. 静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？

    父类的静态属性和方法可以被子类继承

    不可以被子类重写：当父类的引用指向子类时，使用对象调用静态方法或者静态变量，是调用的父类中的方法或者变量。并没有被子类改写。 

    原因：

    因为静态方法从程序开始运行后就已经分配了内存，也就是说已经写死了。所有引用到该方法的对象（父类的对象也好子类的对象也好）所指向的都是同一块内存中的数据，也就是该静态方法。

    子类中如果定义了相同名称的静态方法，并不会重写，而应该是在内存中又分配了一块给子类的静态方法，没有重写这一说。

20. 静态内部类的设计意图

    非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。

    没有这个引用就意味着：

    1. 它的创建是不需要依赖于外围类的。
    2. 它不能使用任何外围类的非static成员变量和方法。

21. 成员内部类、静态内部类、局部内部类和匿名内部类的理解，以及项目中的应用

    <https://pingxin0521.coding.me/2019/04/04/Java-基础-7/#内部类>

22. 谈谈对kotlin的理解

    <https://www.jianshu.com/p/0bae35037270>

23. 闭包和局部内部类的区别

    https://www.jianshu.com/p/f55b11a4cec2

24. string 转换成 integer的方式及原理

    https://www.jianshu.com/p/9eebb4f2ccb1

25. 什么是语法糖？请举例说明？

    语法糖的存在主要是方便开发人员使用。但其实，**Java虚拟机并不支持这些语法糖**，这些语法糖在**编译阶段就会被还原成简单的基础语法结构**，这个过程就是解语法糖。

    说到编译，大家肯定都知道，Java语言中，javac命令可以将后缀名为.java的源文件编译为后缀名为.class的可以运行于Java虚拟机的字节码。如果你去看com.sun.tools.javac.main.JavaCompiler的源码，你会发现在compile()中有一个步骤就是调用**desugar()**，这个方法就是负责解语法糖的实现的。

    Java 中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。

    Java语法糖系列一：可变长度参数和foreach循环
     [http://www.jianshu.com/p/628568f94ef8](https://www.jianshu.com/p/628568f94ef8)

    Java语法糖系列二：自动装箱/拆箱和条件编译
     [http://www.jianshu.com/p/946b3c4a5db6](https://www.jianshu.com/p/946b3c4a5db6)

    Java语法糖系列三：泛型与类型擦除
     [http://www.jianshu.com/p/4de08deb6ba4](https://www.jianshu.com/p/4de08deb6ba4)

    Java语法糖系列四：枚举类型
     [http://www.jianshu.com/p/ae09363fe734](https://www.jianshu.com/p/ae09363fe734)

    Java语法糖系列五：内部类和闭包
     [http://www.jianshu.com/p/f55b11a4cec2](https://www.jianshu.com/p/f55b11a4cec2)

##### **（二） java深入源码级的面试题（有难度）**

- 哪些情况下的对象会被垃圾回收机制处理掉？

- 讲一下常见编码方式？

- utf-8编码中的中文占几个字节；int型几个字节？

- 静态代理和动态代理的区别，什么场景使用？

- Java的异常体系

- 谈谈你对解析与分派的认识。

- 修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？

- Java中实现多态的机制是什么？

- 如何将一个Java对象序列化到文件里？

- 说说你对Java反射的理解

- 说说你对Java注解的理解

- 说说你对依赖注入的理解

- 说一下泛型原理，并举例说明

- Java中String的了解

- String为什么要设计成不可变的？

- Object类的equal和hashCode方法重写，为什么？ 

##### **（三） 数据结构**

- 常用数据结构简介
- 并发集合了解哪些？
- 列举java的集合以及集合之间的继承关系
- 集合类以及集合框架
- 容器类介绍以及之间的区别（容器类估计很多人没听这个词，Java容器主要可以划分为4个部分：List列表、Set集合、Map映射、工具类（Iterator迭代器、Enumeration枚举类、Arrays和Collections）。
- List,Set,Map的区别
- List和Map的实现方式以及存储方式
- HashMap的实现原理
- HashMap数据结构？
- HashMap源码理解
- HashMap如何put数据（从HashMap源码角度讲解）？
- HashMap怎么手写实现？
- ConcurrentHashMap的实现原理
- ArrayMap和HashMap的对比
- HashTable实现原理
- TreeMap具体实现
- HashMap和HashTable的区别
- HashMap与HashSet的区别
- HashSet与HashMap怎么判断集合元素重复？
- 集合Set实现Hash怎么防止碰撞
- ArrayList和LinkedList的区别，以及应用场景
- 数组和链表的区别
- 二叉树的深度优先遍历和广度优先遍历的具体实现
- 堆的结构
- 堆和树的区别
- 堆和栈在内存中的区别是什么(解答提示：可以从数据结构方面以及实际实现方面两个方面去回答)？
- 什么是深拷贝和浅拷贝
- 手写链表逆序代码
- 讲一下对树，B+树的理解
- 讲一下对图的理解
- 判断单链表成环与否？
- 链表翻转（即：翻转一个单项链表）
- 合并多个单有序链表（假设都是递增的）

##### **（四） 线程、多线程和线程池**

- 开启线程的三种方式？
- 线程和进程的区别？
- 为什么要有线程，而不是仅仅用进程？
- run()和start()方法区别
- 如何控制某个方法允许并发访问线程的个数？
- 在Java中wait和seelp方法的不同；
- 谈谈wait/notify关键字的理解
- 什么导致线程阻塞？
- 线程如何关闭？
- 讲一下java中的同步的方法
- 数据一致性如何保证？
- 如何保证线程安全？
- 如何实现线程同步？
- 两个进程同时要求写或者读，能不能实现？如何防止进程的同步？
- 线程间操作List
- Java中对象的生命周期
- Synchronized用法
- synchronize的原理
- 谈谈对Synchronized关键字，类锁，方法锁，重入锁的理解
- static synchronized 方法的多线程访问和作用
- 同一个类里面两个synchronized方法，两个线程同时访问的问题
- volatile的原理
- 谈谈volatile关键字的用法
- 谈谈volatile关键字的作用
- 谈谈NIO的理解
- synchronized 和volatile 关键字的区别
- synchronized与Lock的区别
- ReentrantLock 、synchronized和volatile比较
- ReentrantLock的内部实现
- lock原理
- 死锁的四个必要条件？
- 怎么避免死锁？
- 对象锁和类锁是否会互相影响？
- 什么是线程池，如何使用?
- Java的并发、多线程、线程模型
- 谈谈对多线程的理解
- 多线程有什么要注意的问题？
- 谈谈你对并发编程的理解并举例说明
- 谈谈你对多线程同步机制的理解？
- 如何保证多线程读写文件的安全？
- 多线程断点续传原理
- 断点续传的实现



















































































### 错误

1. This is probably not a problem with npm. There is likely additional logging output above.错误

   可能由于种种版本更新的原因需要执行

   ```
   npm install
   ```

    重新安装一次，如果还是不可以的话，在把之前装的都清空

   ```
   rm -rf node_modules
   rm package-lock.json
   npm cache clear --force
   npm install
   ```

   

