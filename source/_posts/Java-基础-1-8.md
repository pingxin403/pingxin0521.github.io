---
title: Java 异常处理
date: 2019-04-05 18:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 异常处理

**Java的基本理念是“结构不佳的代码不能运行”！！！！！**

> 大成若缺，其用不弊。
>
> 大盈若冲，其用不穷。
>

在这个世界不可能存在完美的东西，不管完美的思维有多么缜密，细心，我们都不可能考虑所有的因素，这就是所谓的智者千虑必有一失。同样的道理，计算机的世界也是不完美的，异常情况随时都会发生，我们所需要做的就是避免那些能够避免的异常，处理那些不能避免的异常。这里我将记录如何利用异常还程序一个“完美世界”。

<!--more-->

**为什么要使用异常**

首先我们可以明确一点就是异常的处理机制可以确保我们程序的健壮性，提高系统可用率。虽然我们不是特别喜欢看到它，但是我们不能不承认它的地位，作用。有异常就说明程序存在问题，有助于我们及时改正。在我们的程序设计当做，任何时候任何地方因为任何原因都有可能会出现异常，在没有异常机制的时候我们是这样处理的：通过函数的返回值来判断是否发生了异常（这个返回值通常是已经约定好了的），调用该函数的程序负责检查并且分析返回值。虽然可以解决异常问题，但是这样做存在几个缺陷：

1. 容易混淆。如果约定返回值为-11111时表示出现异常，那么当程序最后的计算结果真的为-1111呢？

2. 代码可读性差。将异常处理代码和程序代码混淆在一起将会降低代码的可读性。

3. 由调用函数来分析异常，这要求程序员对库函数有很深的了解。

在OO中提供的异常处理机制是提供代码健壮的强有力的方式。**使用异常机制它能够降低错误处理代码的复杂度，如果不使用异常，那么就必须检查特定的错误，并在程序中的许多地方去处理它，而如果使用异常，那就不必在方法调用处进行检查，因为异常机制将保证能够捕获这个错误，并且，只需在一个地方处理错误，即所谓的异常处理程序中。这种方式不仅节约代码，而且把“概述在正常执行过程中做什么事”的代码和“出了问题怎么办”的代码相分离。总之，与以前的错误处理方法相比，异常机制使代码的阅读、编写和调试工作更加井井有条。**（摘自《Think in java 》）。

在初学时，总是听老师说把有可能出错的地方记得加异常处理，刚刚开始还不明白，有时候还觉得只是多此一举，现在随着自己的不断深入，代码编写多了，渐渐明白了异常是非常重要的。

#### 基本定义

在《Think in java》中是这样定义异常的：**异常情形是指阻止当前方法或者作用域继续执行的问题。** 在这里一定要明确一点：异常代码某种程度的错误，尽管Java有异常处理机制，但是我们不能以“正常”的眼光来看待异常，异常处理机制的原因就是告诉你：这里可能会或者已经产生了错误，您的程序出现了不正常的情况，可能会导致程序失败！

**那么什么时候才会出现异常呢？** 只有在你当前的环境下程序无法正常运行下去，也就是说程序已经无法来正确解决问题了，这时它所就会从当前环境中跳出，并抛出异常。抛出异常后，它首先会做几件事。首先，它会使用new创建一个异常对象，然后在产生异常的位置终止程序，并且从当前环境中弹出对异常对象的引用，这时。异常处理机制就会接管程序，并开始寻找一个恰当的地方来继续执行程序，这个恰当的地方就是异常处理程序，它的任务就是将程序从错误状态恢复，以使程序要么换一种方法执行，要么继续执行下去。

总的来说异常处理机制就是当程序发生异常时，它强制终止程序运行，记录异常信息并将这些信息反馈给我们，由我们来确定是否处理异常。

异常发生的原因有很多，通常包含以下几大类：
```
用户输入了非法数据。
要打开的文件不存在。
网络通信时连接中断，或者JVM内存溢出
```
这些异常有的是因为用户错误引起，有的是程序错误引起的，还有其它一些是因为物理错误引起的。-

要理解Java异常处理是如何工作的，你需要掌握以下三种类型的异常：

 - 检查性异常：最具代表的检查性异常是用户错误或问题引起的异常，这是程序员无法预见的。例如要打开一个不存在文件时，一个异常就发生了，这些异常在编译时不能被简单地忽略。
 - 运行时异常： 运行时异常是可能被程序员避免的异常。与检查性异常相反，运行时异常可以在编译时被忽略。
 - 错误： 错误不是异常，而是脱离程序员控制的问题。错误在代码中通常被忽略。例如，当栈溢出时，一个错误就发生了，它们在编译也检查不到的。

 异常指不期而至的各种状况，如：文件找不到、网络连接失败、除0操作、非法参数等。异常是一个事件，它发生在程序运行期间，干扰了正常的指令流程。

 Java语言在设计的当初就考虑到这些问题，提出异常处理的框架的方案，所有的异常都可以用一个异常类来表示，不同类型的异常对应不同的子类异常（目前我们所说的异常包括错误概念），定义异常处理的规范，在JDK1.4版本以后增加了异常链机制，从而便于跟踪异常。

 Java异常是一个描述在代码段中发生异常的对象，当发生异常情况时，一个代表该异常的对象被创建并且在导致该异常的方法中被抛出，而该方法可以选择自己处理异常或者传递该异常。


#### 异常体系

 java为我们提供了非常完美的异常处理机制，使得我们可以更加专心于我们的程序，在使用异常之前我们需要了解它的体系结构。

 Java把异常当作对象来处理，并定义一个基类java.lang.Throwable作为所有异常的超类。

在Java API中已经定义了许多异常类，这些异常类分为两大类，错误Error和异常Exception。


![异常处理类.jpg](https://i.loli.net/2019/04/11/5caec960cc38f.jpg)


从上面这幅图可以看出，Throwable是java语言中所有错误和异常的超类（万物即可抛）。它有两个子类：Error、Exception。

下面将详细讲述这些异常之间的区别与联系：

 - Error：Error类对象由 **Java 虚拟机生成并抛出**，大多数错误与代码编写者所执行的操作无关。例如，Java虚拟机运行错误（Virtual MachineError），当JVM不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止；还有发生在虚拟机试图执行应用时，如类定义错误（NoClassDefFoundError）、链接错误（LinkageError）。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在Java中，错误通常是使用Error的子类描述。

 - Exception：在Exception分支中有一个重要的子类RuntimeException（运行时异常），该类型的异常自动为你所编写的程序定义ArrayIndexOutOfBoundsException（数组下标越界）、NullPointerException（空指针异常）、ArithmeticException（算术异常）、MissingResourceException（丢失资源）、ClassNotFoundException（找不到类）等异常，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由**程序逻辑错误**引起的，程序应该从逻辑角度尽可能避免这类异常的发生；而RuntimeException之外的异常我们统称为非运行时异常，类型上属于Exception类及其子类，从程序语法角度讲是**必须进行处理的异常**，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

 >Error和Exception的区别：Error通常是灾难性的致命的错误，是程序无法控制和处理的，当出现这些异常时，Java虚拟机（JVM）一般会选择终止线程；Exception通常情况下是可以被程序处理的，并且在程序中应该尽可能的去处理这些异常。


- 检查异常：在正确的程序运行过程中，很容易出现的、情理可容的异常状况，在一定程度上这种异常的发生是可以预测的，并且一旦发生该种异常，就必须采取某种方式进行处理。
>除了RuntimeException及其子类以外，其他的Exception类及其子类都属于检查异常，当程序中可能出现这类异常，要么使用try-catch语句进行捕获，要么用throws子句抛出，否则编译无法通过。

- 不受检查异常：包括RuntimeException及其子类和Error。
>不受检查异常为编译器不要求强制处理的异常，检查异常则是编译器要求必须处置的异常。



所以：对于可恢复的条件使用被检查的异常（CheckedException），对于程序错误（言外之意不可恢复，大错已经酿成）使用运行时异常（RuntimeException）。



#### 异常处理流程

Java的异常处理本质上是抛出异常和捕获异常。

![异常处理流程.jpeg](https://i.loli.net/2019/04/11/5caec960e55e0.jpeg)

**抛出异常：** 要理解抛出异常，首先要明白什么是异常情形（exception condition），它是指阻止当前方法或作用域继续执行的问题。其次把异常情形和普通问题相区分，普通问题是指在当前环境下能得到足够的信息，总能处理这个错误。对于异常情形，已经无法继续下去了，因为在当前环境下无法获得必要的信息来解决问题，你所能做的就是从当前环境中跳出，并把问题提交给上一级环境，这就是抛出异常时所发生的事情。

抛出异常后，会有几件事随之发生。首先，是像创建普通的java对象一样将使用new在堆上创建一个异常对象；然后，当前的执行路径（已经无法继续下去了）被终止，并且从当前环境中弹出对异常对象的引用。此时，异常处理机制接管程序，并开始寻找一个恰当的地方继续执行程序，这个恰当的地方就是异常处理程序或者异常处理器，它的任务是将程序从错误状态中恢复，以使程序要么换一种方式运行，要么继续运行下去。

**捕获异常：** 在方法抛出异常之后，运行时系统将转为寻找合适的异常处理器（exception handler）。潜在的异常处理器是异常发生时依次存留在调用栈中的方法的集合。当异常处理器所能处理的异常类型与方法抛出的异常类型相符时，即为合适的异常处理器。运行时系统从发生异常的方法开始，依次回查调用栈中的方法，直至找到含有合适异常处理器的方法并执行。当运行时系统遍历调用栈而未找到合适的异常处理器，则运行时系统终止。同时，意味着Java程序的终止。


对于运行时异常、错误和检查异常，Java技术所要求的异常处理方式有所不同。

 - 由于运行时异常及其子类的不可查性，为了更合理、更容易地实现应用程序，Java规定，运行时异常将由Java运行时系统自动抛出，允许应用程序忽略运行时异常。

 - 对于方法运行中可能出现的Error，当运行方法不欲捕捉时，Java允许该方法不做任何抛出声明。因为，大多数Error异常属于永远不能被允许发生的状况，也属于合理的应用程序不该捕捉的异常。

 - 对于所有的检查异常，Java规定：一个方法必须捕捉，或者声明抛出方法之外。也就是说，当一个方法选择不捕捉检查异常时，它必须声明将抛出异常。


#### 异常使用

在网上看了这样一个搞笑的话：世界上最真情的相依，是你在try我在catch。无论你发神马脾气，我都默默承受，静静处理。对于初学者来说异常就是try…catch，（鄙人刚刚接触时也是这么认为的，碰到异常就是try…catch）。个人感觉try…catch确实是用的最多也是最实用的。

Java异常处理涉及到五个关键字，分别是：try、catch、finally、throw、throws。下面将骤一介绍，通过认识这五个关键字，掌握基本异常处理知识。

![异常处理关键字.jpg](https://i.loli.net/2019/04/11/5caec960e6ee8.jpg)


 - try        -- 用于监听。将要被监听的代码(可能抛出异常的代码)放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
 -  catch   -- 用于捕获异常。catch用来捕获try语句块中发生的异常。
 -  finally  -- finally语句块总是会被执行。它主要用于回收在try块里打开的物力资源(如数据库连接、网络连接和磁盘文件)。只有finally块，执行完成之后，才会回来执行try或者catch块中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则就不会跳回执行，直接停止。
 -  throw   -- 用于抛出异常。
 -  throws -- 用在方法签名中，用于声明该方法可能抛出的异常。

**异常处理的基本语法**

```
try{
    //code that might generate exceptions
        }catch(Exception e){
    //the code of handling exception1
    }catch(Exception e){
    //the code of handling exception2
  }
```
要明白异常捕获，还要理解监控区域（guarded region）的概念。它是一段可能产生异常的代码，并且后面跟着处理这些异常的代码。

因而可知，上述try-catch所描述的即是监控区域，关键词try后的一对大括号将一块可能发生异常的代码包起来，即为监控区域。Java方法在运行过程中发生了异常，则创建异常对象。将异常抛出监控区域之外，由Java运行时系统负责寻找匹配的catch子句来捕获异常。若有一个catch语句匹配到了，则执行该catch块中的异常处理代码，就不再尝试匹配别的catch块了。

**匹配的原则是：** 如果抛出的异常对象属于catch子句的异常类，或者属于该异常类的子类，则认为生成的异常对象与catch块捕获的异常类型相匹配。

使用多重的catch语句：很多情况下，由单个的代码段可能引起多个异常。处理这种情况，我们需要定义两个或者更多的catch子句，每个子句捕获一种类型的异常，当异常被引发时，每个catch子句被依次检查，第一个匹配异常类型的子句执行，当一个catch子句执行以后，其他的子句将被旁路。

编写多重catch语句块注意事项:顺序问题：先小后大，即**先子类后父类**

Java通过异常类描述异常类型。对于有多个catch子句的异常程序而言，应该尽量将捕获底层异常类的catch子句放在前面，同时尽量将捕获相对高层的异常类的catch子句放在后面。否则，捕获底层异常类的catch子句将可能会被屏蔽。

RuntimeException异常类包括运行时各种常见的异常，ArithmeticException类和ArrayIndexOutOfBoundsException类都是它的子类。因此，RuntimeException异常类的catch子句应该放在最后面，否则可能会屏蔽其后的特定异常处理或引起编译错误。

**嵌套try语句：** try语句可以被嵌套。也就是说，一个try语句可以在另一个try块的内部。每次进入try语句，异常的前后关系都会被推入堆栈。如果一个内部的try语句不含特殊异常的catch处理程序，堆栈将弹出，下一个try语句的catch处理程序将检查是否与之匹配。这个过程将继续直到一个catch语句被匹配成功，或者是直到所有的嵌套try语句被检查完毕。如果没有catch语句匹配，Java运行时系统将处理这个异常。

注意：当有方法调用时，try语句的嵌套可以很隐蔽的发生。例如，我们可以将对方法的调用放在一个try块中。在该方法的内部，有另一个try语句。在这种情况下，方法内部的try仍然是嵌套在外部调用该方法的try块中的。


到目前为止，我们只是获取了被Java运行时系统引发的异常。然而，我们还可以用throw语句抛出明确的异常。Throw的语法形式如下：
```
throw ThrowableInstance;
```
这里的ThrowableInstance一定是Throwable类类型或者Throwable子类类型的一个对象。简单的数据类型，例如int，char,以及非Throwable类，例如String或Object，不能用作异常。有两种方法可以获取Throwable对象：在catch子句中使用参数或者使用new操作符创建。

程序执行完throw语句之后立即停止；throw后面的任何语句不被执行，最邻近的try块用来检查它是否含有一个与异常类型匹配的catch语句。如果发现了匹配的块，控制转向该语句；如果没有发现，次包围的try块来检查，以此类推。如果没有发现匹配的catch块，默认异常处理程序中断程序的执行并且打印堆栈轨迹。**throw抛出的异常对象必须得到处理**


如果一个方法可以导致一个异常但不处理它，它必须指定这种行为以使方法的调用者可以保护它们自己而不发生异常。要做到这点，我们可以在方法声明中包含一个throws子句。一个throws子句列举了一个方法可能引发的所有异常类型。这对于除了Error或RuntimeException及它们子类以外类型的所有异常是必要的。一个方法可以引发的所有其他类型的异常必须在throws子句中声明，否则会导致编译错误。

下面是throws子句的方法声明的通用形式：
```
public void info() throws Exception
{
   //body of method
}
```
Exception 是该方法可能引发的所有的异常,也可以是异常列表，中间以逗号隔开。

Throws抛出异常的规则：

 - 如果是不受检查异常（unchecked exception），即Error、RuntimeException或它们的子类，那么可以不使用throws关键字来声明要抛出的异常，编译仍能顺利通过，但在运行时会被系统抛出。
 - 必须声明方法可抛出的任何检查异常（checked exception）。即如果一个方法可能出现受可查异常，要么用try-catch语句捕获，要么用throws子句声明将它抛出，否则会导致编译错误
 - 仅当抛出了异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出，而不是囫囵吞枣。
 - 调用方法必须遵循任何可查异常的处理和声明规则。若覆盖一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或子类。


 当异常发生时，通常方法的执行将做一个陡峭的非线性的转向，它甚至会过早的导致方法返回。例如，如果一个方法打开了一个文件并关闭，然后退出，你不希望关闭文件的代码被异常处理机制旁路。finally关键字为处理这种意外而设计。

 finally创建的代码块在try/catch块完成之后另一个try/catch出现之前执行。finally块无论有没有异常抛出都会执行。如果抛出异常，即使没有catch子句匹配，finally也会执行。一个方法将从一个try/catch块返回到调用程序的任何时候，经过一个未捕获的异常或者是一个明确的返回语句，finally子句在方法返回之前仍将执行。这在关闭文件句柄和释放任何在方法开始时被分配的其他资源是很有用。

>    finally子句是可选项，可以有也可以无，但是每个try语句至少需要一个catch或者finally子句。

注：如果finally块与一个try联合使用，finally块将在try结束之前执行。

#### 自定义异常

Java确实给我们提供了非常多的异常，但是异常体系是不可能预见所有的希望加以报告的错误，所以Java允许我们自定义异常来表现程序中可能会遇到的特定问题，总之就是一句话：我们不必拘泥于Java中已有的异常类型。

Java自定义异常的使用要经历如下四个步骤：

1、定义一个类继承Throwable或其子类。

2、添加构造方法(当然也可以不用添加，使用默认构造方法)。

3、在某个方法类抛出该异常。

4、捕捉该异常。

```（java）
/** 自定义异常 继承Exception类 **/
class MyException extends Exception{
    public MyException(){

    }

    public MyException(String message){
        super(message);
    }
}

public class Test {
    public void display(int i) throws MyException{
        if(i == 0){
            throw new MyException("该值不能为0.......");
        }
        else{
            System.out.println( i / 2);
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        try {
            test.display(0);
            System.out.println("---------------------");
        } catch (MyException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：

```
$ java Test
MyException: 该值不能为0.......
	at Test.display(Test.java:15)
	at Test.main(Test.java:25)
```
#### 异常链

在设计模式中有一个叫做责任链模式，该模式是将多个对象链接成一条链，客户端的请求沿着这条链传递直到被接收、处理。同样Java异常机制也提供了这样一条链：异常链。

我们知道每遇到一个异常信息，我们都需要进行try…catch,一个还好，如果出现多个异常呢？分类处理肯定会比较麻烦，那就一个Exception解决所有的异常吧。这样确实是可以，但是这样处理势必会导致后面的维护难度增加。最好的办法就是将这些异常信息封装，然后捕获我们的封装类即可。

诚然在应用程序中，我们有时候不仅仅只需要封装异常，更需要传递。怎么传递？throws!！binge，正确！！但是如果仅仅只用throws抛出异常，那么你的封装类，怎么办？？

我们有两种方式处理异常，一是throws抛出交给上级处理，二是try…catch做具体处理。但是这个与上面有什么关联呢？try…catch的catch块我们可以不需要做任何处理，仅仅只用throw这个关键字将我们封装异常信息主动抛出来。然后在通过关键字throws继续抛出该方法异常。它的上层也可以做这样的处理，以此类推就会产生一条由异常构成的异常链。

**通过使用异常链，我们可以提高代码的可理解性、系统的可维护性和友好性。**

同理，我们有时候在捕获一个异常后抛出另一个异常信息，并且希望将原始的异常信息也保持起来，这个时候也需要使用异常链。

在异常链的使用中，throw抛出的是一个新的异常信息，这样势必会导致原有的异常信息丢失，如何保持？在Throwable及其子类中的构造器中都可以接受一个cause参数，该参数保存了原有的异常信息，通过getCause()就可以获取该原始异常信息。

```
public void test() throws XxxException{
        try {
            //do something:可能抛出异常信息的代码块
        } catch (Exception e) {
            throw new XxxException(e);
        }
    }
```

示例：
```（java）
public class Test {
    public void f() throws MyException{
         try {
            FileReader reader = new FileReader("G:\\myfile\\struts.txt");  
             Scanner in = new Scanner(reader);  
             System.out.println(in.next());
        } catch (FileNotFoundException e) {
            //e 保存异常信息
            throw new MyException("文件没有找到--01",e);
        }  
    }

    public void g() throws MyException{
        try {
            f();
        } catch (MyException e) {
            //e 保存异常信息
            throw new MyException("文件没有找到--02",e);
        }
    }

    public static void main(String[] args) {
        Test t = new Test();
        try {
            t.g();
        } catch (MyException e) {
            e.printStackTrace();
        }
    }
}
```
运行结果：
```
com.test9.MyException: 文件没有找到--02
    at com.test9.Test.g(Test.java:31)
    at com.test9.Test.main(Test.java:38)
Caused by: com.test9.MyException: 文件没有找到--01
    at com.test9.Test.f(Test.java:22)
    at com.test9.Test.g(Test.java:28)
    ... 1 more
Caused by: java.io.FileNotFoundException: G:\myfile\struts.txt (系统找不到指定的路径。)
    at java.io.FileInputStream.open(Native Method)
    at java.io.FileInputStream.<init>(FileInputStream.java:106)
    at java.io.FileInputStream.<init>(FileInputStream.java:66)
    at java.io.FileReader.<init>(FileReader.java:41)
    at com.test9.Test.f(Test.java:17)
    ... 2 more
```

#### 总结

异常使用指南（摘自：Think in java）

应该在下列情况下使用异常。

1. 在恰当的级别处理问题（在知道该如何处理异常的情况下才捕获异常）。

2. 解决问题并且重新调用产生异常的方法。

3. 进行少许修补，然后绕过异常发生的地方继续执行。

4. 用别的数据进行计算，以代替方法预计会返回的值。

5. 把当前运行环境下能做的事情尽量做完。然后把相同（不同）的异常重新抛到更高层。

6. 终止程序。

7. 进行简化。尽可能的减小try块。

8. 让类库和程序更加安全。（这既是在为调试做短期投资，也是在为程序的健壮做长期投资）

9. 保证所有资源都被正确释放。充分运用finally关键词。

10. catch语句应当尽量指定具体的异常类型，而不应该指定涵盖范围太广的Exception类。 不要一个Exception试图处理所有可能出现的异常。

11. 既然捕获了异常，就要对它进行适当的处理。不要捕获异常之后又把它丢弃，不予理睬。 不要做一个不负责的人
那么怎么改进呢？有四种选择：
```
1、处理异常。对所发生的的异常进行一番处理，如修正错误、提醒。再次申明ex.printStackTrace()算不上已经“处理好了异常”.
2、重新抛出异常。既然你认为你没有能力处理该异常，那么你就尽情向上抛吧！！！
3、封装异常。这是较好的处理方法，对异常信息进行分类，然后进行封装处理。
4、不要捕获异常。
```
12. 在异常处理模块中提供适量的错误原因信息，组织错误信息使其易于理解和阅读。

13. 不要在finally块中处理返回值。

14. 不要在构造函数中抛出异常。


### 错误控制

1. 函数参数检查
```
public String getMsg(String code,String key){
    if ("".equals(code)||"".equals(key))
    {
        //throw 自定义异常
        //或
        //return “”；
    }
    String msg="";
    //do something

    return msg;
}
```
2. 函数返回值检查
```
public void exec(String msg)//throws 自定义异常,如果getMsg会抛出异常
{
    if (null==msg||"".equals(msg))
    {
        //throw 自定义异常
        //或
        //return “”；
    }

    String s = getMsg(msg, KEY);

    if ("".equals(s))
    {
        //do Something
    }
    //do Something
}
```
3. 错误操作使用throw产生异常，底层函数throws抛异常到上一层，顶层进行try{}catch处理。

### 面试题

1. try{} 里有一个 return 语句，那么紧跟在这个 try 后的 finally{} 里的 code 会不会被执行，什么时候被执行，在 return 前还是后?

   答案：会执行，在方法返回调用者前执行。

   注意：在finally中改变返回值的做法是不好的，因为如果存在finally代码块，try中的return语句不会立马返回调用者，而是记录下返回值待finally代码块执行完毕之后再向调用者返回其值，然后如果在finally中修改了返回值，就会返回修改后的值。显然，在finally中返回或者修改返回值会对程序造成很大的困扰，C#中直接用编译错误的方式来阻止程序员干这种龌龊的事情，Java中也可以通过提升编译器的语法检查级别来产生警告或错误。

2. Java语言如何进行异常处理，关键字：throws、throw、try、catch、finally分别如何使用？

   答：Java通过面向对象的方法进行异常处理，把各种不同的异常进行分类，并提供了良好的接口。在Java中，每个异常都是一个对象，它是Throwable类或其子类的实例。当一个方法出现异常后便抛出一个异常对象，该对象中包含有异常信息，调用这个对象的方法可以捕获到这个异常并可以对其进行处理。

   Java的异常处理是通过5个关键词来实现的：try、catch、throw、throws和finally。

   一般情况下是用try来执行一段程序，如果系统会抛出（throw）一个异常对象，可以通过它的类型来捕获（catch）它，或通过总是执行代码块（finally）来处理；try用来指定一块预防所有异常的程序；catch子句紧跟在try块后面，用来指定你想要捕获的异常的类型；throw语句用来明确地抛出一个异常；throws用来声明一个方法可能抛出的各种异常（当然声明异常时允许无病呻吟）；finally为确保一段代码不管发生什么异常状况都要被执行；try语句可以嵌套，每当遇到一个try语句，异常的结构就会被放入异常栈中，直到所有的try语句都完成。如果下一级的try语句没有对某种异常进行处理，异常栈就会执行出栈操作，直到遇到有处理这种异常的try语句或者最终将异常抛给JVM。

3. 运行时异常与受检异常有何异同？

   答：异常表示程序运行过程中可能出现的非正常状态，运行时异常表示虚拟机的通常操作中可能遇到的异常，是一种常见运行错误，只要程序设计得没有问题通常就不会发生。受检异常跟程序运行的上下文环境有关，即使程序设计无误，仍然可能因使用的问题而引发。Java编译器要求方法必须声明抛出可能发生的受检异常，但是并不要求必须声明抛出未被捕获的运行时异常。异常和继承一样，是面向对象程序设计中经常被滥用的东西，在Effective Java中对异常的使用给出了以下指导原则：

   - 不要将异常处理用于正常的控制流（设计良好的API不应该强迫它的调用者为了正常的控制流而使用异常）

   - 对可以恢复的情况使用受检异常，对编程错误使用运行时异常

   - 避免不必要的使用受检异常（可以通过一些状态检测手段来避免异常的发生）

   - 优先使用标准的异常

   - 每个方法抛出的异常都要有文档

   - 保持异常的原子性

   - 不要在catch中忽略掉捕获到的异常

4. 列出一些你常见的运行时异常？

   答：

   - ArithmeticException（算术异常）

   - ClassCastException （类转换异常）

   - IllegalArgumentException （非法参数异常）

   - IndexOutOfBoundsException （下标越界异常）

   - NullPointerException （空指针异常）

   - SecurityException （安全异常）

5. 什么时候用断言（assert）

   答：断言在软件开发中是一种常用的调试方式，很多开发语言中都支持这种机制。一般来说，断言用于保证程序最基本、关键的正确性。断言检查通常在开发和测试时开启。为了保证程序的执行效率，在软件发布后断言检查通常是关闭的。断言是一个包含布尔表达式的语句，在执行这个语句时假定该表达式为true；如果表达式的值为false，那么系统会报告一个AssertionError。断言的使用如下面的代码所示：

   ```
   assert(a > 0); // throws an AssertionError if a <= 0
   ```

   断言可以有两种形式：
    assert Expression1;
    assert Expression1 : Expression2 ;
    Expression1 应该总是产生一个布尔值。
    Expression2 可以是得出一个值的任意表达式；这个值用于生成显示更多调试信息的字符串消息。

   要在运行时启用断言，可以在启动JVM时使用-enableassertions或者-ea标记。要在运行时选择禁用断言，可以在启动JVM时使用-da或者-disableassertions标记。要在系统类中启用或禁用断言，可使用-esa或-dsa标记。还可以在包的基础上启用或者禁用断言。

   > 注意：断言不应该以任何方式改变程序的状态。简单的说，如果希望在不满足某些条件时阻止代码的执行，就可以考虑用断言来阻止它。

6. Error和Exception有什么区别

   答：Error表示系统级的错误和程序不必处理的异常，是恢复不是不可能但很困难的情况下的一种严重问题；比如内存溢出，不可能指望程序能处理这样的情况；Exception表示需要捕捉或者需要程序进行处理的异常，是一种设计或实现问题；也就是说，它表示如果程序运行正常，从不会发生的情况。

7. 2005年摩托罗拉的面试中曾经问过这么一个问题“If a process reports a stack overflow run-time error, what’s the most possible cause?”，给了四个选项

   a. lack of memory; 

   b. write on an invalid memory space;

    c. recursive function calling; 

   d. array index out of boundary. 

   Java程序在运行时也可能会遭遇StackOverflowError，这是一个无法恢复的错误，只能重新修改代码了，这个面试题的答案是c。如果写了不能迅速收敛的递归，则很有可能引发栈溢出的错误，如下所示：

   ```java
   class StackOverflowErrorTest {
   
       public static void main(String[] args) {
           main(null);
       }
   }
   ```

   > 提示：用递归编写程序时一定要牢记两点：1. 递归公式；2. 收敛条件（什么时候就不再继续递归）。

9. 阐述final、finally、finalize的区别。

   - final：修饰符（关键字）有三种用法：如果一个类被声明为final，意味着它不能再派生出新的子类，即不能被继承，因此它和abstract是反义词。将变量声明为final，可以保证它们在使用中不被改变，被声明为final的变量必须在声明时给定初值，而在以后的引用中只能读取不可修改。被声明为final的方法也同样只能使用，不能在子类中被重写。

   - finally：通常放在try…catch…的后面构造总是执行代码块，这就意味着程序无论正常执行还是发生异常，这里的代码只要JVM不关闭都能执行，可以将释放外部资源的代码写在finally块中。

   - finalize：Object类中定义的方法，Java中允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在销毁对象时调用的，通过重写finalize()方法可以整理系统资源或者执行其他清理工作。

10. 类ExampleA继承Exception，类ExampleB继承ExampleA。

    有如下代码片断：

    ```
    try {
        throw new ExampleB("b")
    } catch（ExampleA e）{
        System.out.println("ExampleA");
    } catch（Exception e）{
        System.out.println("Exception");
    }
    ```

    请问执行此段代码的输出是什么?

    答：输出：ExampleA。（根据里氏代换原则[能使用父类型的地方一定能使用子类型]，抓取ExampleA类型异常的catch块能够抓住try块中抛出的ExampleB类型的异常）

11. 说出下面代码的运行结果。（此题的出处是《Java编程思想》一书）

    ```
    class Annoyance extends Exception {}
    class Sneeze extends Annoyance {}
    
    class Human {
    
        public static void main(String[] args) 
            throws Exception {
            try {
                try {
                    throw new Sneeze();
                } 
                catch ( Annoyance a ) {
                    System.out.println("Caught Annoyance");
                    throw a;
                }
            } 
            catch ( Sneeze s ) {
                System.out.println("Caught Sneeze");
                return ;
            }
            finally {
                System.out.println("Hello World!");
            }
        }
    }
    ```

11. [java异常面试常见题目](https://www.cnblogs.com/huajiezh/p/5236864.html)