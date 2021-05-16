---
title: Java 补充知识
date: 2019-06-05 18:18:59
tags:
 - Java
 - 基础
categories:
 - Java
 - 基础
---

#### Console类:控制台输入输出

<!--more-->

1. **输出到控制台**

   - Console.Write(输出的值);  表示向控制台直接写入字符串，不进行换行，可继续接着前面的字符写入。
   - Console.WriteLine(输出的值);  表示向控制台写入字符串后换行。（常用）
   - Console.WriteLine("输出的格式字符串",变量列表);
   - Console.Write("输出的格式字符串",变量列表);

   注：Console.WrietLine()和Console.Write()的唯一却别就是前者输出后换行，后者不换行。Console.WriteLine("鹿鼎记中{0}的妻子有{1},{2},{3}等7个",strName[0],strName[1],strName[2],strName3]);

2. **从控制台输入**
   Console类提供的输入方法：

   - Console.ReadLine();  表示从控制台读取字符串后进行换行。(常用)
     这一句代码返回一个字符串型数据，可以把它直接赋值给字符串变量，如：

     ```
     String strname=console.ReadLine();
     ```

     有时需要从控制台输入数字，就用到前面介绍的内容，数据转换，如：

     ```
     int num=Integer.parseInt(console.ReadLine());
     int num=Convert.string2int(console.ReadLine(),10);
     ```

     上面两句代码效果相同，可以根据自己的习惯选择任意一种。

注意：   

- Console.ReadLine()和Console.Read()的输入结果完全不同，不能混用。

  Console.Read(),返回值为首字符的ASCII码； Console.ReadLine(),返回值为字符串。也就是说read方法只能读取第一个字符，而ReadLine能读多个字符也可以换行读取 

- Console.ReadKey()的作用，read是从控制台读取，key表示按下键盘，那么组合在一起的意思就是获取用户按下功能键显示在窗口中，用在前面的代码起到窗口暂停的功能，在调试状态下，只有按下任意键后窗口才会关闭。Console.ReadKey 获取用户按下的下一个字符或功能键，按下的键显示在控制台窗口中。
- Console.Beep 通过控制台扬声器播放提示音。
- Console.Clear 清除控制台缓冲区和相应的控制台窗口的显示信息。

##### 问题描述

学习如何从控制台输入中一般都会使用Scanner类，但是读取密码时JavaSE6引入了Console类，测试代码如下：

```
        Console console =  System.console();
        System.out.println(console);
        System.out.print("请输入你的名字：");
        String personName = console.readLine();
        System.out.print("请输入你的密码：");
        char[] password = console.readPassword();
```

该段代码在**Eclipse**的控制台中打印出来的结果是console的值为null.

##### 原因

如果Java程序要与windows下的cmd或者Linux下的Terminal交互，就可以使用这个Java Console类代劳。Java要与Console进行交互，不总是能得到可用的Java Console类的。一个JVM是否有可用的Console，依赖于底层平台和JVM如何被调用。如果JVM是在交互式命令行（比如Windows的cmd）中启动的，并且输入输出没有重定向到另外的地方，那么就可以得到一个可用的Console实例。

但当使用Eclipse等IDE运行以上代码时Console中将会为null。

表示Java程序无法获得Console实例，是因为JVM不是在命令行中被调用的，或者输入输出被重定向了。在Eclipse诸如类似的IDE工具中运行Console类。如果没有对Console实例判空操作，结果使用了该实例会抛出java.lang.NullPointerException异常。 

#### UML类图

虚线箭头指向依赖；

实线箭头指向关联；

虚线三角指向接口；

实线三角指向父类；

空心菱形能分离而独立存在，是聚合；

实心菱形精密关联不可分，是组合；

![1.png](https://i.loli.net/2019/06/06/5cf914ce31a6550123.png)

在画类图的时候，理清类和类之间的关系是重点。类的关系有泛化(Generalization)、实现（Realization）、依赖(Dependency)和关联(Association)。其中关联又分为一般关联关系和聚合关系(Aggregation)，合成关系(Composition)。下面我们结合实例理解这些关系。

**基本概念**

类图（Class Diagram）: 类图是面向对象系统建模中最常用和最重要的图，是定义其它图的基础。类图主要是用来显示系统中的类、接口以及它们之间的静态结构和关系的一种静态模型。

类图的3个基本组件：类名、属性、方法。 

![2.png](https://i.loli.net/2019/06/06/5cf91588b4b6047818.png)



泛化(generalization)：表示is-a的关系，是对象之间耦合度最大的一种关系，子类继承父类的所有细节。直接使用语言中的继承表达。在类图中使用带三角箭头的实线表示，箭头从子类指向父类。

![1.jpg](https://i.loli.net/2019/06/06/5cf915785153b78087.jpg)

实现（Realization）:在类图中就是接口和实现的关系。这个没什么好讲的。在类图中使用带三角箭头的虚线表示，箭头从实现类指向接口。

![2.jpg](https://i.loli.net/2019/06/06/5cf915886500695689.jpg)

依赖(Dependency)：对象之间最弱的一种关联方式，是临时性的关联。代码中一般指由局部变量、函数参数、返回值建立的对于其他对象的调用关系。一个类调用被依赖类中的某些方法而得以完成这个类的一些职责。在类图使用带箭头的虚线表示，箭头从使用类指向被依赖的类。

![3.jpg](https://i.loli.net/2019/06/06/5cf915890505941517.jpg)

关联(Association) : 对象之间一种引用关系，比如客户类与订单类之间的关系。这种关系通常使用类的属性表达。关联又分为一般关联、聚合关联与组合关联。后两种在后面分析。在类图使用带箭头的实线表示，箭头从使用类指向被关联的类。可以是单向和双向。

![4.jpg](https://i.loli.net/2019/06/06/5cf91588e960396612.jpg)

聚合(Aggregation) : 
表示has-a的关系，是一种不稳定的包含关系。较强于一般关联,有整体与局部的关系,并且没有了整体,局部也可单独存在。如公司和员工的关系，公司包含员工，但如果公司倒闭，员工依然可以换公司。在类图使用空心的菱形表示，菱形从局部指向整体。

![5.jpg](https://i.loli.net/2019/06/06/5cf91588ecaf280893.jpg)

组合(Composition) : 
表示contains-a的关系，是一种强烈的包含关系。组合类负责被组合类的生命周期。是一种更强的聚合关系。部分不能脱离整体存在。如公司和部门的关系，没有了公司，部门也不能存在了；调查问卷中问题和选项的关系；订单和订单选项的关系。在类图使用实心的菱形表示，菱形从局部指向整体。

![6.jpg](https://i.loli.net/2019/06/06/5cf915892060440386.jpg)

多重性(Multiplicity) : 通常在关联、聚合、组合中使用。就是代表有多少个关联对象存在。使用数字..星号（数字）表示。如下图，一个割接通知可以关联0个到N个故障单。

![7.jpg](https://i.loli.net/2019/06/06/5cf915893d1a651178.jpg)

**聚合和组合的区别**

这两个比较难理解，重点说一下。聚合和组合的区别在于：聚合关系是“has-a”关系，组合关系是“contains-a”关系；聚合关系表示整体与部分的关系比较弱，而组合比较强；聚合关系中代表部分事物的对象与代表聚合事物的对象的生存期无关，一旦删除了聚合对象不一定就删除了代表部分事物的对象。组合中一旦删除了组合对象，同时也就删除了代表部分事物的对象。 

**实例分析**

联通客户响应OSS。系统有故障单、业务开通、资源核查、割接、业务重保、网络品质性能等功能模块。现在我们抽出部分需求做为例子讲解。

大家可以参照着类图，好好理解。 

![8.jpg](https://i.loli.net/2019/06/06/5cf915898999181779.jpg)

1. 通知分为一般通知、割接通知、重保通知。这个是继承关系。
2. NoticeService和实现类NoticeServiceImpl是实现关系。
3. NoticeServiceImpl通过save方法的参数引用Notice,是依赖关系。同时调用了BaseDao完成功能，也是依赖关系。
4. 割接通知和故障单之间通过中间类(通知电路)关联，是一般关联。
5.  重保通知和预案库间是聚合关系。因为预案库可以事先录入，和重保通知没有必然联系，可以独立存在。在系统中是手工从列表中选择。删除重保通知，不影响预案。
6. 割接通知和需求单之间是聚合关系。同理，需求单可以独立于割接通知存在。也就是说删除割接通知，不影响需求单。
7. 通知和回复是组合关系。因为回复不能独立于通知存在。也就是说删除通知，该条通知对应的回复也要级联删除。

经过以上的分析，相信大家对类的关系已经有比较好的理解了。大家有什么其它想法或好的见解，欢迎拍砖。

需要绘图的童鞋可以选择使用[ProcessOn](https://www.processon.com/i/5ba47363e4b06fc64affadcd)，网页作图，功能强大，实时保存，不联网时，也可以在本地缓存。

#### maven编译报错：java.lang.ExceptionInInitializerError: com.sun.tools.javac.code.TypeTags

错误日志:

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project helloworld: Fatal error compiling: java.lang.ExceptionInInitializerError: com.sun.tools.javac.code.TypeTags -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :helloworld

```

原因是lombok版本太低，不支持java10以上。
到https://mvnrepository.com查询新版本即可

```
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
    <scope>provided</scope>
</dependency>

```

#### Java中计算对象的大小

Java中如何计算对象的大小呢，找到了3种方法：

1. java.lang.instrument.Instrumentation的getObjectSize方法；

2. BTraceUtils的sizeof方法；
3. https://github.com/mingbozhang/memory-measurer提供的工具包；

本质上java.lang.instrument.Instrumentation的使用是其他三种方法的基础，但是该类中的方法getObjectSize只是计算了对象本身,JDK注释描述：

> Returns an implementation-specific approximation of the amount of storage consumed by
> the specified object. The result may include some or all of the object's overhead,
> and thus is useful for comparison within an implementation but not between implementations.

第2种是使用BTrace的方法，可以对生产环境的程序进行检测，但是BTraceUtils的sizeof实现上直接调用的还是java.lang.instrument.Instrumentation的getObjectSize方法，所以还是存在第1种方法的问题。第3种方法支持对HashMap等常见对象的大小计算。

#### 将enum转换成json

```java
//JDK1.8及以上
//转换方法
public List<SelectForm> getConfig(){
    return Stream.of(Interval.values()).map(
        (interval) -> SelectForm.builder().value(interval.name()).label(interval.getDesc()).build())
        .collect(Collectors.toList());
}

//数据类
public class SelectForm {
  private String value;
  private String label;
}

//时间间隔枚举类
public enum Interval {
  D_1("今天"), D_2("最近3天"), D_7("最近7天"), M_1("最近一个月"), M_3("最近三个月");

  private String desc;

  private Interval(String desc) {
    this.desc = desc;
  }

  public String getDesc() {
    return desc;
  }
}

//返回前台结果
{
  "data": [
    {
      "value": "D_1",
      "label": "今天"
    },
    {
      "value": "D_2",
      "label": "最近3天"
    },
    {
      "value": "D_7",
      "label": "最近7天"
    },
    {
      "value": "M_1",
      "label": "最近一个月"
    },
    {
      "value": "M_3",
      "label": "最近三个月"
    }
  ],
  "code": 200,
  "message": "OK"
}
```

#### JPA之映射mysql text类型问题

**问题背景**

jpa如果直接映射mysql的text/longtext/tinytext类型到String字段会报错。需要设置一下@Lob和@Column。

@Lob代表是长字段类型，默认的话，是longtext类型，所以需要下面这个属性来指定对应的类型。

columnDefinition="text"里面的类型可以随意改，后面mysql可能会有新的类型，只要是对应java的String类型，就可以在这里动态配置。

**解决方案**

```java
@Data
@Entity
@Table(name="question")
public class Question {
    @Id
    @GeneratedValue
    public int questionId;

    //....省略其他字段
    @Lob
    @Column(columnDefinition="text")
    public String explainStr;
    public Date createTime;
}
```




