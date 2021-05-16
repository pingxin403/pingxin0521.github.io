---
title: Java  JVM 入门
date: 2019-04-09 13:18:59
tags:
 - Java
categories:
 - Java
 - JVM
---
### 前言

JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

Java语言的一个非常重要的特点就是与平台的无关性。而使用Java虚拟机是实现这一特点的关键。一般的高级语言如果要在不同的平台上运行，至少需要编译成不同的目标代码。而引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。这就是Java的能够“一次编译，到处运行”的原因。
<!--more-->

每个使用Java的开发者都知道Java字节码是在JRE中运行(JRE: Java 运行时环境)。JVM则是JRE中的核心组成部分，承担分析和执行Java字节码的工作，而Java程序员通常并不需要深入了解JVM运行情况就可以开发出大型应用和类库。尽管如此，如果你对JVM有足够了解，就会对Java有更好的掌握，并且能解决一些看起来简单但又尚未解决的问题。

### 相关知识

这里进行简述，后面会进行详细分析

#### 虚拟机

JRE由Java API和JVM组成，JVM通过类加载器(Class Loader)加类Java应用，并通过Java API进行执行。

虚拟机(VM: Virtual Machine)是通过软件模拟物理机器执行程序的执行器。最初Java语言被设计为基于虚拟机器在而非物理机器，重而实现WORA(一次编写，到处运行)的目的，尽管这个目标几乎被世人所遗忘。所以，JVM可以在所有的硬件环境上执行Java字节码而无须调整Java的执行模式。

JVM的基本特性：

 - 基于栈(Stack-based)的虚拟机: 不同于Intel x86和ARM等比较流行的计算机处理器都是基于寄存器(register)架构，JVM是基于栈执行的。

 - 符号引用(Symbolic reference): 除基本类型外的所有Java类型(类和接口)都是通过符号引用取得关联的，而非显式的基于内存地址的引用。

 - 垃圾回收机制: 类的实例通过用户代码进行显式创建，但却通过垃圾回收机制自动销毁。

 - 通过明确清晰基本类型确保平台无关性: 像C/C++等传统编程语言对于int类型数据在同平台上会有不同的字节长度。JVM却通过明确的定义基本类型的字节长度来维持代码的平台兼容性，从而做到平台无关。

- 网络字节序(Network byte order): Java class文件的二进制表示使用的是基于网络的字节序(network byte order)。为了在使用小端(little endian)的Intel x86平台和在使用了大端(big endian)的RISC系列平台之间保持平台无关，必须要定义一个固定的字节序。JVM选择了网络传输协议中使用的网络字节序，即基于大端(big endian)的字节序。


Sun 公司开发了Java语言，但任何人都可以在遵循JVM规范的前提下开发和提供JVM实现。所以目前业界有多种不同的JVM实现，包括Oracle Hostpot JVM和IBM JVM。Google公司使用的Dalvik VM也是一种JVM实现，尽管其并未完全遵循JVM规范。与基于栈机制的Java 虚拟机不同的是Dalvik VM是基于寄存器的，Java 字节码也被转换为Dalvik VM使用的寄存器指令集。


#### Java代码编译和执行的整个过程
也正如前面所说，Java代码的编译和执行的整个过程大概是：开发人员编写Java代码(.java文件)，然后将之编译成字节码(.class文件)，再然后字节码被装入内存，一旦字节码进入虚拟机，它就会被解释器解释执行，或者是被即时代码发生器有选择的转换成机器码执行。

（1）Java代码编译是由Java源码编译器来完成，也就是Java代码到JVM字节码（.class文件）的过程。 流程图如下所示：

![Java编译过程.jpeg](https://i.loli.net/2019/04/09/5cac60f8d7912.jpeg)

（2）Java字节码的执行是由JVM执行引擎来完成，流程图如下所示：

![java字节码执行过程.jpeg](https://i.loli.net/2019/04/09/5cac60f8c0a4f.jpeg)

Java代码编译和执行的整个过程包含了以下三个重要的机制:
1. Java源码编译机制

2. 类加载机制

3. 类执行机制

**Java源码编译机制**

Java 源码编译由以下三个过程组成：
分析和输入到符号表
注解处理
语义分析和生成class文件

流程图如下所示：

![java源码编译.jpeg](https://i.loli.net/2019/04/09/5cac620a2125a.jpeg)

最后生成的class文件由以下部分组成：

1. 结构信息：包括class文件格式版本号及各部分的数量与大小的信息
2. 元数据：对应于Java源码中声明与常量的信息。包含类/继承的超类/实现的接口的声明信息、域与方法声明信息和常量池
3. 方法信息：对应Java源码中语句和表达式对应的信息。包含字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试符号信息

##### 类加载机制

装载--》链接--》初始化

**装载**

JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：

![java类加载.png](https://i.loli.net/2019/04/09/5cac620a7be78.png)

1. Bootstrap ClassLoader
负责加载$JAVA_HOME中`jre/lib/rt.jar`里所有的class，由C++实现，不是ClassLoader子类

2. Extension ClassLoader
负责加载java平台中扩展功能的一些jar包，包括`$JAVA_HOME中jre/lib/*.jar`或-Djava.ext.dirs指定目录下的jar包

3. App ClassLoader
负责记载classpath中指定的jar包及目录中class

4. Custom ClassLoader
属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader


加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类。

**类执行机制**

JVM是基于堆栈的虚拟机。JVM为每个新创建的线程都分配一个堆栈.也就是说,对于一个Java程序来说，它的运行就是通过对堆栈的操作来完成的。堆栈以帧为单位保存线程的状态。JVM对堆栈只进行两种操作:以帧为单位的压栈和出栈操作。

JVM执行class字节码，线程创建后，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个栈帧，每个栈帧对应着每个方法的每次调用，而栈帧又是有局部变量区和操作数栈两部分组成，局部变量区用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。栈的结构如下图所示：

![java类执行.jpeg](https://i.loli.net/2019/04/09/5cac6209c64cb.jpeg)

#### JVM内存结构

![JVM结构.jpg](https://i.loli.net/2019/04/09/5cac35106fa3e.jpg)

下面我们了解下Java栈、Java堆、方法区和常量池：


1. Java栈（线程私有数据区）
每个Java虚拟机线程都有自己的Java虚拟机栈，Java虚拟机栈用来存放栈帧，每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

2. Java堆（线程共享数据区）
在虚拟机启动时创建，此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配。


3. 方法区（线程共享数据区）
方法区在虚拟机启动的时候被创建，它存储了每一个类的结构信息，例如运行时常量池、字段和方法数据、构造函数和普通方法的字节码内容、还包括在类、实例、接口初始化时用到的特殊方法。在JDK8之前永久代是方法区的一种实现，而JDK8元空间替代了永久代，永久代被移除，也可以理解为元空间是方法区的一种实现。


4. 常量池（线程共享数据区）
常量池常被分为两大类：静态常量池和运行时常量池。
静态常量池也就是Class文件中的常量池，存在于Class文件中。
运行时常量池（Runtime Constant Pool）是方法区的一部分，存放一些运行时常量数据。


#### JVM内存管理及垃圾回收机制
JVM内存结构分为：方法区（method），栈内存（stack），堆内存（heap），本地方法栈（java中的jni调用），结构图如下所示：

![java内存结构.jpeg](https://i.loli.net/2019/04/09/5cac628a64f4f.jpeg)

（1）堆内存（heap）

所有通过new创建的对象的内存都在堆中分配，其大小可以通过-Xmx和-Xms来控制。

操作系统有一个记录空闲内存地址的链表，当系统收到程序的申请时，会遍历该链表，寻找第一个空间大于所申请空间的堆结点，然后将该结点从空闲结点链表中删除，并将该结点的空间分配给程序，另外，对于大多数系统，会在这块内存空间中的首地址处记录本次分配的大小，这样代码中的delete语句才能正确的释放本内存空间。但由于找到的堆结点的大小不一定正好等于申请的大小，系统会自动的将多余的那部分重新放入空闲链表中。这时由new分配的内存，一般速度比较慢，而且容易产生内存碎片，不过用起来最方便。

另外，在WINDOWS下，最好的方式是用VirtualAlloc分配内存，它不是在堆，也不是在栈，而是直接在进程的地址空间中保留一块内存，虽然这种方法用起来最不方便，但是速度快，也是最灵活的。堆内存是向高地址扩展的数据结构，是不连续的内存区域。由于系统是用链表来存储的空闲内存地址的，自然是不连续的，而链表的遍历方向是由低地址向高地址。堆的大小受限于计算机系统中有效的虚拟内存。由此可见，**堆获得的空间比较灵活，也比较大。**

（2）栈内存（stack）

在Windows下, 栈是向低地址扩展的数据结构，是一块 **连续的内存区域** 这句话的意思是栈顶的地址和栈的最大容量是系统预先规定好的，在WINDOWS下，栈的大小是固定的（是一个编译时就确定的常数），如果申请的空间超过栈的剩余空间时，将提示overflow。

因此，能从栈获得的空间较小。只要栈的剩余空间大于所申请空间，系统将为程序提供内存，否则将报异常提示栈溢出。 由系统自动分配，速度较快。但程序员是无法控制的。

**堆内存与栈内存需要说明：**

基础数据类型直接在栈空间分配，方法的形式参数，直接在栈空间分配，当方法调用完成后从栈空间回收。引用数据类型，需要用new来创建，既在栈空间分配一个地址空间，又在堆空间分配对象的类变量 。方法的引用参数，在栈空间分配一个地址空间，并指向堆空间的对象区，当方法调用完成后从栈空间回收。局部变量new出来时，在栈空间和堆空间中分配空间，当局部变量生命周期结束后，栈空间立刻被回收，堆空间区域等待GC回收。方法调用时传入的literal参数，先在栈空间分配，在方法调用完成后从栈空间收回。字符串常量、static在DATA区域分配，this在堆空间分配。数组既在栈空间分配数组名称，又在堆空间分配数组实际的大小。

（3）本地方法栈（java中的jni调用）

用于支持native方法的执行，存储了每个native方法调用的状态。对于本地方法接口，实现JVM并不要求一定要有它的支持，甚至可以完全没有。Sun公司实现Java本地接口(JNI)是出于可移植性的考虑，当然我们也可以设计出其它的本地接口来代替Sun公司的JNI。但是这些设计与实现是比较复杂的事情，需要确保垃圾回收器不会将那些正在被本地方法调用的对象释放掉。

（4）方法区（method）

它保存方法代码(编译后的java代码)和符号表。存放了要加载的类信息、静态变量、final类型的常量、属性和方法信息。JVM用持久代（Permanet Generation）来存放方法区，可通过-XX:PermSize和-XX:MaxPermSize来指定最小值和最大值。

**垃圾回收机制**

堆里聚集了所有由应用程序创建的对象，JVM也有对应的指令比如 new, newarray, anewarray和multianewarray，然并没有向 C++ 的 delete，free 等释放空间的指令，Java的所有释放都由 GC 来做，GC除了做回收内存之外，另外一个重要的工作就是内存的压缩，这个在其他的语言中也有类似的实现，相比 C++ 不仅好用，而且增加了安全性，当然她也有弊端，比如性能这个大问题。


#### Java 字节码

JVM使用Java字节码—一种运行于Java(用户语言)和机器语言的中间语言，以达到WORA的目的。Java字节码是部署Java程序的最小单元。

Java 字节码是JVM的基本元素，JVM本身就是一个用于执行Java字节码的执行器。Java编译器并不会把像C/C++那样把高级语言转为机器语言(CPU执行指令)，而是把开发者能理解的Java语言转为JVM理解的Java字节码。因为Java字节码是平台无关的，所以它可以在安装了JVM(准确的说，是JRE环境)的任何硬件环境执行，即使它们的CPU和操作系统各不相同(所以在Windows PC机上开发和编译的class文件在不做任何调整的情况下就可以在Linux机器上执行)。编译后文件的大小与源文件大小基本一致，所以比较容易通过网络传输和执行Java字节码。

Java class文件本身是基于二进制的文件，所以我们很难直观的理解其中的指令。为了管理这些class 文件， JVM提供了javap命令来对二进制文件进行反编译。执行javap得到的是直观的java指令序列。

源码：
```（java）
public  class Hello
{
	public static void main(String[] args) {
		System.out.println("Hello World");
}
}
```
先使用javac编译成字节码，再使用javap -c 将class文件反编译成指令。

```
$ javac Hello.java
$ javap -c Hello
Compiled from "Hello.java"
public class Hello {
  public Hello();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #3                  // String Hello World
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```

invokevirtual是Java 字节码中最常用到的一个操作码，用于调用一个方法。另外，在Java字节码中有4个表示调用方法的操作码： invokeinterface, invokespecial, invokestatic, invokevirtual 。他们每个的含义如下：

```
invokeinterface: 调用接口方法
invokespecial: 调用初始化方法、私有方法、或父类中定义的方法
invokestatic: 调用静态方法
invokevirtual: 调用实例方法
```

Java 字节码的指令集包含操作码(OpCode)和操作数(Operand)。像invokevirtual这样的操作码需要一个2字节长度的操作数。


**在上面的反编译结果中，代码前面的数字是具有什么含义？**

它是一个一字节数字，也许正因此JVM执行的代码被称为“字节码”。像 aload0, getfield_ 和 invokevirtual 都被表示为一个单字节数字。(aload_0 = 0x2a, getfiled = 0xb4, invokevirtual = 0xb6)。因此Java字节码表示的最大指令码为256。

像aload0和aload1这样的操作码不需要任何操作数，因此aload_0的下一个字节就是下一个指令的操作码。而像getfield和invokevirtual这样的操作码却需要一个2字节的操作数，因此第一个字节里的第二个指令getfield指令的一下指令是在第4个字节，其中跳过了2个字节。通过16进制编辑器查看字节码如下:

```
  2a b4 00 0f 2b b6 00 17 57 b1
```
在Java字节码中，类实例表示为"L;"，而void表示为"V"，类似的其他类型也有各自的表示。下表列出了Java字节码中类型表示。

Java字节码里的类型表示:

![](https://i.loli.net/2019/04/09/5cac5e199131f.png)

示例：

```
double d[][][]  --->   [[[D

Object mymethod(int i,double d,Thread t) ----> mymethod(I,D,Ljava/lang/Thread;)Ljava/lang/Object

```

#### 类文件格式

我们通过对这一段符号的分析来了解一个类文件的具体格式。

 - magic: 类文件的前4个字节是一组魔数，是一个用于区分Java类文件的预定义值。如上所看到的，其值固定为0xCAFEBABE。也就是说一个文件的前4个字节如果是0xCAFABABE，就可以认为它是Java类文件。"CAFABABE"是与"JAVA"有关的一个有趣的魔数。
 - minorversion, majorversion: 接下来的4个字节表示类的版本号。如上所示，0x00000032表示的类版本号为50.0。由JDK 1.6编译而来的类文件的版本号是50.0，而由JDK 1.5编译而来的版本号则是49.0。JVM必须保持向后兼容，即保持对比其版本低的版本的类文件的兼容。而如果在一个低版本的JVM中运行高版本的类文件，则会出现java.lang.UnsupportedClassVersionError的发生。
 - constantpoolcount, constant_pool[]: 紧接着版本号的是类的常量池信息。这里的信息在运行时会被分配到运行时常量池区域，后面会有对内存分配的介绍。在JVM加载类文件时，类的常量池里的信息会被分配到运行时常量池，而运行时常量池又包含在方法区内。上面UserService.class文件的constantpoolcount为0x0028，所以按照定义contant_pool数组将有(40-1)即39个元素值。
 - access_flags: 2字节的类的修饰符信息，表示类是否为public, private, abstract或者interface。
 - thisclass, superclass: 分别表示保存在constant_pool数组中的当前类及父类信息的索引值。
 - interface_count, interfaces[]: interfacecount为保存在constantpool数组中的当前类实现的接口数的索引值，interfaces[]即表示当前类所实现的每个接口信息。
 - fields_count, fields[]: 类的字段数量及字段信息数组。字段信息包含字段名、类型、修饰符以及在constant_pool数组中的索引值。
 - methods_count, methods[]: 类的方法数量及方法信息数组。方法信息包括方法名、参数的类型及个数、返回值、修饰符、在constant_pool中的索引值、方法的可执行代码以及异常信息。
 - attributes_count, attributes[]: attributeinfo有多种不同的属性，分别被fieldinfo, method_into使用。

javap程序把class文件格式以可阅读的方式输出来。在对Hello.class文件使用"javap -verbose"命令分析时，输出内容如下：


```
Classfile /home/hyp/programes/IdeaProjects/Learn/Hello.class
  Last modified 2019-4-9; size 415 bytes
  MD5 checksum 660731507e3404560151325ceed123b4
  Compiled from "Hello.java"
public class Hello
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // Hello World
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // Hello
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Hello.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               Hello World
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               Hello
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public Hello();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 4: 0
        line 5: 8
}
SourceFile: "Hello.java"
```
方法的65535个字节的限制受到了结构体method_info的影响。如上面"javap -verbose"的输出所示，结构体Hello包括代码(Code)、行号表(LineNumberTable)以及本地变量表(LocalVariableTable)。其中行号表、本地变量表以及代码里的异常表(exceptiontable)的总长度为一个固定2字节的值。因此方法的大小不能超过行号表、本地变量表、异常表的长度，即不能超过65535个字节。

尽管很多人抱怨方法的大小限制，JVM规范也声称将会对此大小进行扩充，然而到目前为止并没有明确的进展。因为JVM技术规范里定义要把几乎整个类文件的内容都加载到方法区，因此如果方法长度将会对程序的向后兼容带来极大的挑战。

**对于一个由Java编译器错误而导致的错误的类文件将发生怎样的情况？如果是在网络传输或文件复制过程中，类文件被损坏又将发生什么？**
为了应对这些场景，Java类加载器的加载过程被设计为一个非常严谨的处理过程。JVM规范详细描述了这个过程。

我们如何验证JVM成功执行了类文件的验证过程？如何验证不同的JVM实现是否符合JVM规范？为此，Oracle提供了专门的测试工具：TCK(Technology Compatibility Kit)。TCK通过执行大量的测试用例(包括大量通过不同方式生成的错误类文件)来验证JVM规范。只有通过TCK测试的JVM才能被称作是JVM。

类似TCK，还有一个JCP(Java Community Process;` http://jcp.org`)，用于验证新的Java技术规范。对于一个JCP，必须具有详细的文档，相关的实现以及提交给JSR(Java Specification Request)的TCK测试。如果用户想像JSR一样使用新的Java技术，那他必须先从RI提供者那里得到许可，或者自己直接实现它并对之进行TCK测试。

### 面试题

1. GC是什么？为什么要有GC？

   答：GC是垃圾收集的意思，内存处理是编程人员容易出现问题的地方，忘记或者错误的内存回收会导致程序或系统的不稳定甚至崩溃，Java提供的GC功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的，Java语言没有提供释放已分配内存的显式操作方法。Java程序员不用担心内存管理，因为垃圾收集器会自动进行管理。要请求垃圾收集，可以调用下面的方法之一：System.gc() 或Runtime.getRuntime().gc() ，但JVM可以屏蔽掉显式的垃圾回收调用。

   垃圾回收可以有效的防止内存泄露，有效的使用可以使用的内存。垃圾回收器通常是作为一个单独的低优先级的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清除和回收，程序员不能实时的调用垃圾回收器对某个对象或所有对象进行垃圾回收。在Java诞生初期，垃圾回收是Java最大的亮点之一，因为服务器端的编程需要有效的防止内存泄露问题，然而时过境迁，如今Java的垃圾回收机制已经成为被诟病的东西。移动智能终端用户通常觉得iOS的系统比Android系统有更好的用户体验，其中一个深层次的原因就在于Android系统中垃圾回收的不可预知性。

   补充：垃圾回收机制有很多种，包括：分代复制垃圾回收、标记垃圾回收、增量垃圾回收等方式。标准的Java进程既有栈又有堆。栈保存了原始型局部变量，堆保存了要创建的对象。Java平台对堆内存回收和再利用的基本算法被称为标记和清除，但是Java对其进行了改进，采用“分代式垃圾收集”。这种方法会根据Java对象的生命周期将堆内存划分为不同的区域，在垃圾收集过程中，可能会将对象移动到不同区域：

   - 伊甸园（Eden）：这是对象最初诞生的区域，并且对大多数对象来说，这里是它们唯一存在过的区域。
   - 幸存者乐园（Survivor）：从伊甸园幸存下来的对象会被挪到这里。
   - 终身颐养园（Tenured）：这是足够老的幸存对象的归宿。年轻代收集（Minor-GC）过程是不会触及这个地方的。当年轻代收集不能把对象放进终身颐养园时，就会触发一次完全收集（Major-GC），这里可能还会牵扯到压缩，以便为大对象腾出足够的空间。

   与垃圾回收相关的JVM参数：

   - -Xms / -Xmx — 堆的初始大小 / 堆的最大大小
   - -Xmn — 堆中年轻代的大小
   - -XX:-DisableExplicitGC — 让System.gc()不产生任何作用
   - -XX:+PrintGCDetails — 打印GC的细节
   - -XX:+PrintGCDateStamps — 打印GC操作的时间戳
   - -XX:NewSize / XX:MaxNewSize — 设置新生代大小/新生代最大大小
   - -XX:NewRatio — 可以设置老生代和新生代的比例
   - -XX:PrintTenuringDistribution — 设置每次新生代GC后输出幸存者乐园中对象年龄的分布
   - -XX:InitialTenuringThreshold / -XX:MaxTenuringThreshold：设置老年代阀值的初始值和最大值
   - -XX:TargetSurvivorRatio：设置幸存区的目标使用率

