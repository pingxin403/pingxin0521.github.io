---
title: java 枚举
date: 2019-04-05 17:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
enum 的全称为 enumeration， 是 JDK 1.5  中引入的新特性，存放在 java.lang 包中。
<!--more-->

原来接口定义常量：
```java
public interface IConstants {
    String MON = "Mon";
    String TUE = "Tue";
    String WED = "Wed";
    String THU = "Thu";
    String FRI = "Fri";
    String SAT = "Sat";
    String SUN = "Sun";
}
```
#### 语法（定义）

创建枚举类型要使用 enum 关键字，隐含了所创建的类型都是 java.lang.Enum 类的子类（java.lang.Enum 是一个抽象类）。枚举类型符合通用模式 Class Enum<E extends Enum<E\>>，而 E 表示枚举类型的名称。枚举类型的每一个值都将映射到 protected Enum(String name, int ordinal) 构造函数中，在这里，每个值的名称都被转换成一个字符串，并且序数设置表示了此设置被创建的顺序

```java
public enum EnumTest {
    MON, TUE, WED, THU, FRI, SAT, SUN;
}
```
这段代码实际上调用了7次 Enum(String name, int ordinal)：
```java
new Enum<EnumTest>("MON",0);
new Enum<EnumTest>("TUE",1);
new Enum<EnumTest>("WED",2);
    ... ...
```
#### 遍历、switch 等常用操作

```java
public class Test {
    public static void main(String[] args) {
        for (EnumTest e : EnumTest.values()) {
            System.out.println(e.toString());
        }

        System.out.println("----------------我是分隔线------------------");

        EnumTest test = EnumTest.TUE;
        switch (test) {
        case MON:
            System.out.println("今天是星期一");
            break;
        case TUE:
            System.out.println("今天是星期二");
            break;
        // ... ...
        default:
            System.out.println(test);
            break;
        }
    }
}
```
运行结果：
```
MON
TUE
WED
THU
FRI
SAT
SUN
----------------我是分隔线------------------
今天是星期二
```
#### 常用方法介绍

values()获取存储枚举中所有常量实例的数组。常配合foreach完成遍历

valueOf()通过常量名获取对应的枚举实例。

int compareTo(E o) :比较此枚举与指定对象的顺序。

Class<E\> getDeclaringClass() :返回与此枚举常量的枚举类型相对应的 Class 对象。

String name() :返回此枚举常量的名称，在其枚举声明中对其进行声明。

int ordinal() :返回枚举常量的序数（它在枚举声明中的位置，其中初始常量序数为零）。

String toString():返回枚举常量的名称，它包含在声明中。

static <T extends Enum<T\>> T valueOf(Class<T\> enumType, String name) :返回带指定名称的指定枚举类型的枚举常量。

```java
EnumTest test = EnumTest.TUE;

 //compareTo(E o)
 switch (test.compareTo(EnumTest.MON)) {
 case -1:
     System.out.println("TUE 在 MON 之前");
     break;
 case 1:
     System.out.println("TUE 在 MON 之后");
     break;
 default:
     System.out.println("TUE 与 MON 在同一位置");
     break;
 }

 //getDeclaringClass()
 System.out.println("getDeclaringClass(): " + test.getDeclaringClass().getName());

 //name() 和  toString()
 System.out.println("name(): " + test.name());
 System.out.println("toString(): " + test.toString());

 //ordinal()， 返回值是从 0 开始
 System.out.println("ordinal(): " + test.ordinal());
```
运行结果：
```
TUE 在 MON 之后
getDeclaringClass(): com.hyp.learn.EnumTest
name(): TUE
toString(): TUE
ordinal(): 1
```
#### 自定义属性和方法

枚举是一个特殊的类，因此它是可拓展的。这意味着他们可以有实例字段、构造器和方法（默认无参构造器不能够被声明并且所有的构造器必须被private修饰）。让我们使用枚举的实例和构造器添加一个isWeekend属性。

给 enum 对象加一下 value 的属性和 getValue() 的方法：
```java
public enum EnumTest {
    MON(1), TUE(2), WED(3), THU(4), FRI(5), SAT(6) {
        @Override
        public boolean isRest() {
            return true;
        }
    },
    SUN(0) {
        @Override
        public boolean isRest() {
            return true;
        }
    };

    private int value;

    private EnumTest(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    public boolean isRest() {
        return false;
    }
}


public class Test {
    public static void main(String[] args) {
        System.out.println("EnumTest.FRI 的 value = " + EnumTest.FRI.getValue());
    }
}
```
输出结果：
```
EnumTest.FRI 的 value = 5
```
Java中枚举的实例字段有很大的用处。在常规的类声明规则中，它们经常用来将一些额外的细节与每个值相关联。

#### 枚举与接口

另外一个强大的特性，我们再次确认一下枚举是是一个特殊的类，所以它能够实现接口（然而枚举不能够继承任何类）。比如，让我们引入接口DayOfWeek。
```java
interface DayOfWeek {
    boolean isWeekend();
}
```
然后使用接口实现代替常规实例字段的方式重写前面的枚举例子。
```java
public enum DaysOfTheWeekInterfaces implements DayOfWeek {
    MONDAY() {
        @Override
        public boolean isWeekend() {
            return false;
        }
    },
    TUESDAY() {
        @Override
        public boolean isWeekend() {
            return false;
        }
    },
    WEDNESDAY() {
        @Override
        public boolean isWeekend() {
            return false;
        }
    },
    THURSDAY() {
        @Override
        public boolean isWeekend() {
            return false;
        }
    },
    FRIDAY() {
        @Override
        public boolean isWeekend() {
            return false;
        }
    },
    SATURDAY() {
        @Override
        public boolean isWeekend() {
            return true;
        }
    },
    SUNDAY() {
        @Override
        public boolean isWeekend() {
            return true;
        }
    };
}
```
我们实现接口的这种方式显得代码有些冗长，然而合并实例字段和接口实现可以解决这个问题，比如：
```java
public enum DaysOfTheWeekFieldsInterfaces implements DayOfWeek {
    MONDAY( false ),
    TUESDAY( false ),
    WEDNESDAY( false ),
    THURSDAY( false ),
    FRIDAY( false ),
    SATURDAY( true ),
    SUNDAY( true );

    private final boolean isWeekend;

    private DaysOfTheWeekFieldsInterfaces(final boolean isWeekend){
        this.isWeekend = isWeekend;
    }

    @Override
    public boolean isWeekend() {
        return isWeekend;
    }
}
```
通过支持实例字段和接口，枚举可以以更加面向对象的方式使用，从而带来一定程度的抽象。

#### 枚举与泛型

在Java中，虽然咋一看并看不出来枚举和泛型的关系，但是他们之间存在一种关系。Java中的每一个单独的枚举自动继承自泛型类Enum<T\>，在这里T就是枚举类型本身。Java编译器在编译时代表开发者做了这个转换，拓展一下枚举声明public enum DaysOfTheWeek 如下：

```java
public class DaysOfTheWeek extends Enum< DaysOfTheWeek > {
    // Other declarations here
}
```
这也就说明了为什么枚举可以实现接口但不能继承其他类：因为它隐式的继承自Enum<T>并且我们在使用对象的公共方法时已经讨论过，Java中不支持多继承。

实际上每一个继承自Enum<T\>的枚举允许定义泛型类、接口和方法，通过这种方式可以让枚举类型的实例参数化或者类型参数化。比如：

```java
public<T extends Enum< ? >> void performAction(final T instance) {
    // Perform some action here
}
```
在上面的方法声明中，类型T被约定为任意枚举类型的实例并且Java编译器将会对其做验证。


#### EnumSet和EnumMap

和所有其他类一样，枚举的实例也可以和标准Java集合库一起使用。然而，某些集合类型针对枚举做了优化，并且在大多数情况下推荐使用这些优化过后的集合代替通用的集合。

EnumSet<T\>集合是常规的集合优化过后高效存储枚举类型的一个集合，EnumSet<T\>不能够使用构造器进行实例化，但是它提供了很多非常有用的工厂方法。

示例：
```java
final Set<DaysOfTheWeek> enumSetAll = EnumSet.allOf(DaysOfTheWeek.class);//包含了所有枚举类型所枚举的常量
final Set<DaysOfTheWeek> enumSetNone = EnumSet.noneOf(DaysOfTheWeek.class);//创建的是一个空的EnumSet<T>实例
final Set< DaysOfTheWeek > enumSetSome = EnumSet.of(//指定枚举类型中那些枚举常量应该包含在EnumSet<T>中
    DaysOfTheWeek.SUNDAY,
    DaysOfTheWeek.SATURDAY
);
```
EnumMap<T, ?>是最接近于一般的map的，唯一的不同就是EnumMap<T, ?>的key是枚举类型的枚举常量。比如;
```java
final Map<DaysOfTheWeek, String> enumMap = new EnumMap<>(DaysOfTheWeek.class);
enumMap.put(DaysOfTheWeek.MONDAY, "Lundi");
enumMap.put(DaysOfTheWeek.TUESDAY, "Mardi");
```
注意，和大多数集合实现一样，EnumSet<T>和EnumMap<T, ?>不是线程安全的所以不能在多线程环境下使用。


#### Enum用法
对上面的用法进行总结。

1. 常量
在JDK1.5 之前，我们定义常量都是： public static fianl.... 。现在好了，有了枚举，可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。
```java
public enum Color {  
  RED, GREEN, BLANK, YELLOW  
}
```
2. switch
JDK1.6之前的switch语句只支持int,char,enum类型，使用枚举，能让我们的代码可读性更强。
```java
enum Signal {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    Signal color = Signal.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = Signal.GREEN;  
            break;  
        case YELLOW:  
            color = Signal.RED;  
            break;  
        case GREEN:  
            color = Signal.YELLOW;  
            break;  
        }  
    }  
}  
```

3. 向枚举中添加新方法
如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号。而且 Java 要求必须先定义 enum 实例。
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    // 普通方法  
    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    // get set 方法  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  
}  
```
4. 覆盖枚举的方法
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //覆盖方法  
    @Override  
    public String toString() {  
        return this.index+"_"+this.name;  
    }  
}  
```
5. 实现接口
所有的枚举都继承自java.lang.Enum类。由于Java 不支持多继承，所以枚举对象不能再继承其他类。
```java
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
//接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}  
```
6. 使用接口组织枚举
```java
public interface Food {  
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}  
```
7. 枚举集合使用

```java
public class Test {
    public static void main(String[] args) {
        // EnumSet的使用
        EnumSet<EnumTest> weekSet = EnumSet.allOf(EnumTest.class);
        for (EnumTest day : weekSet) {
            System.out.println(day);
        }

        // EnumMap的使用
        EnumMap<EnumTest, String> weekMap = new EnumMap(EnumTest.class);
        weekMap.put(EnumTest.MON, "星期一");
        weekMap.put(EnumTest.TUE, "星期二");
        // ... ...
        for (Iterator<Entry<EnumTest, String>> iter = weekMap.entrySet().iterator(); iter.hasNext();) {
            Entry<EnumTest, String> entry = iter.next();
            System.out.println(entry.getKey().name() + ":" + entry.getValue());
        }
    }
}
```

#### 原理分析


 enum 的语法结构尽管和 class 的语法不一样，但是经过编译器编译之后产生的是一个class文件。该class文件经过反编译可以看到实际上是生成了一个类，该类继承了java.lang.Enum<E\>。EnumTest 经过反编译(javap com.hmw.test.EnumTest 命令)之后得到的内容如下：
 ```java

public class com.hmw.test.EnumTest extends java.lang.Enum{
    public static final com.hmw.test.EnumTest MON;
    public static final com.hmw.test.EnumTest TUE;
    public static final com.hmw.test.EnumTest WED;
    public static final com.hmw.test.EnumTest THU;
    public static final com.hmw.test.EnumTest FRI;
    public static final com.hmw.test.EnumTest SAT;
    public static final com.hmw.test.EnumTest SUN;
    static {};
    public int getValue();
    public boolean isRest();
    public static com.hmw.test.EnumTest[] values();
    public static com.hmw.test.EnumTest valueOf(java.lang.String);
    com.hmw.test.EnumTest(java.lang.String, int, int, com.hmw.test.EnumTest);
}
 ```
所以，实际上 enum 就是一个 class，只不过 java 编译器帮我们做了语法的解析和编译而已。

#### 总结

**枚举的好处以及与常量类的区别**

- 枚举型可以直接与数据库打交道，我通常使用varchar类型存储，对应的是枚举的常量名。(数据库中好像也有枚举类型，不过也没用过)

-  switch语句支持枚举型，当switch使用int、String类型时，由于值的不稳定性往往会有越界的现象，对于这个的处理往往只能通过if条件筛选以及default模块来处理。而使用枚举型后，在编译期间限定类型，不允许发生越界的情况

- 当你使用常量类时，往往得通过equals去判断两者是否相等，使用枚举的话由于常量值地址唯一，可以用==直接对比，性能会有提高

-  常量类编译时，是直接把常量的值编译到类的二进制代码里，常量的值在升级中变化后，需要重新编译引用常量的类，因为里面存的是旧值。枚举类编译时，没有把常量值编译到代码里，即使常量的值发生变化，也不会影响引用常量的类。

- 枚举类编译后默认为final class，不允许继承可防止被子类修改。常量类可被继承修改、增加字段等，容易导致父类的不兼容。

常量的定义在开发中是必不可少的,虽然无论是通过常量类定义常量还是枚举定义常量都可以满足常量定义的需求。但个人建议最好是使用枚举类型。


可以把 enum 看成是一个普通的 class，它们都可以定义一些属性和方法，不同之处是：enum 不能使用 extends 关键字继承其他类，因为 enum 已经继承了 java.lang.Enum（java是单一继承）
