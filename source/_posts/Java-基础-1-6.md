---
title: Java 包装类和常用数学类
date: 2019-04-05 10:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 包装类

虽然 Java 语言是典型的面向对象编程语言，但其中的八种基本数据类型并不支持面向对象编程，基本类型的数据不具备“对象”的特性——不携带属性、没有方法可调用。 沿用它们只是为了迎合人类根深蒂固的习惯，并的确能简单、有效地进行常规数据处理。

这种借助于非面向对象技术的做法有时也会带来不便，比如引用类型数据均继承了 Object 类的特性，要转换为 String 类型（经常有这种需要）时只要简单调用 Object 类中定义的toString()即可，而基本数据类型转换为 String 类型则要麻烦得多。为解决此类问题 ，Java为每种基本数据类型分别设计了对应的类，称之为包装类(Wrapper Classes)，也有教材称为外覆类或数据类型类。

<!--more-->

#### 自动装箱、拆箱

先抛出定义，Java中基础数据类型与它们的包装类进行运算时，编译器会自动帮我们进行转换，转换过程对程序员是透明的，这就是装箱和拆箱，装箱和拆箱可以让我们的代码更简洁易懂

![](https://i.loli.net/2019/04/10/5cad954f3490a.png)

每个包装类的对象可以封装一个相应的基本类型的数据，并提供了其它一些有用的方法。包装类对象一经创建，其内容（所封装的基本类型数据值）不可改变。

当表格中左边列出的基础类型与它们的包装类有如下几种情况时，编译器会**自动**帮我们进行装箱或拆箱.

- 进行 = 赋值操作（装箱或拆箱）
- 进行+，-，*，/混合运算 （拆箱）
- 进行>,<,==比较运算（拆箱）
- 调用equals进行比较（装箱）
- ArrayList,HashMap等集合类 添加基础类型数据时（装箱）

我们看一段平常很常见的代码

```
public void testAutoBox() {
    List<Float> list = new ArrayList<>();
    list.add(1.0f);
    float firstElement = list.get(0);
}
```

list集合存储的是Float包装类型，我传入的是float基础类型，所以需要进行装箱，而最后的get方法返回的是Float包装类型，我们赋值给float基础类型，所以需要进行拆箱，很简单，安排的明明白白

**手动装箱**

基本类型和对应的包装类可以相互装换：

 - 由基本类型向对应的包装类转换称为装箱，例如把 int 包装成 Integer 类的对象；
 - 包装类向对应的基本类型转换称为拆箱，例如把 Integer 类的对象重新简化为 int。

 ![](https://i.loli.net/2019/04/10/5cada18231a82.png)

以float和Float为例，装箱就是调用Float的valueOf方法new一个Float并赋值，拆箱就是调用Float对象的floatValue方法并赋值返回给float。其他基础类型都是大同小异的，具体可以查看源码。

区别：

- 基本类型是直接用来存储值的,放在**栈**中方便快速使用,包装类型是类,其实例是对象,放在**堆**中
- 基本类型不是对象,因此**没有方法**,包装类型是类,因此**有方法**
- 基本类型**直接赋值**即可,包装类型需要使用**new关键字**创建
- 包装类型**初始值为null**,基本类型初始值不为null,是**根据类型而定的**

#### 包装类的缓存

**Java对部分经常使用的数据采用缓存技术，在类第一次被加载时换创建缓存和数据。当使用等值对象时直接从缓存中获取，从而提高了程序执行性能。（通常只对常用数据进行缓存）**

包装器的缓存：

- Boolean：(全部缓存)，缓存TRUE、FALSE
- Byte：(全部缓存)
- Character(缓存0-127)
- Short(-128 — 127缓存)
- Long(-128 — 127缓存)
- Integer(-128 — 127缓存)，缓存上限可以通过配置jvm更改
- Float(没有缓存)
- Doulbe(没有缓存)

尤其需要特别注意的是，**只有valueOf方法构造对象时会用到缓存，new方法等不会使用缓存！**

同样对于垃圾回收器来说：

```java
ru：Integer i = 100;
i = null;
```

这里虽然i被赋予null，但它之前指向的是cache中的Integer对象，而cache没有被赋null，所以Integer(100)这个对象还是存在；然而如果这里是1000的话就符合回收的条件；其他的包装类也是。

**包装类和基础类型比较大小的问题**

1. int和int之间，用==比较，肯定为true。基本数据类型没有equals方法
2. int和Integer比较，Integer会自动拆箱，== 和 equals都肯定为true
3. int和new Integer比较，Integer会自动拆箱，调用intValue方法, 所以 == 和 equals都肯定为true
4. Integer和Integer比较的时候，由于直接赋值的话会进行自动的装箱。所以当值在[-128,127]中的时候，由于值缓存在IntegerCache中，那么当赋值在这个区间的时候，不会创建新的Integer对象，而是直接从缓存中获取已经创建好的Integer对象。而当大于这个区间的时候，会直接new Integer。
5. 当Integer和Integer进行==比较的时候，在[-128,127]区间的时候，为true。不在这个区间，则为false
6. 当Integer和Integer进行equals比较的时候，由于Integer的equals方法进行了重写，比较的是内容，所以为true
7. Integer和new Integer ： new Integer会创建对象，存储在堆中。而Integer在[-128,127]中，从缓存中取，否则会new Integer.
   所以 Integer和new Integer 进行==比较的话，肯定为false ; Integer和new Integer 进行equals比较的话，肯定为true
   new Integer和new Integer进行==比较的时候，肯定为false ; 进行equals比较的时候，肯定为true
   原因是new的时候，会在堆中创建对象，分配的地址不同，==比较的是内存地址，所以肯定不同

装箱过程是通过调用包装器的valueOf方法实现的
拆箱过程是通过调用包装器的xxxValue方法实现的（xxx表示对应的基本数据类型）


#### Number

Java.lang.Number是所有数字类的父类。Java在java.lang包中的抽象类Number下提供了各种数字包装子类。Number类下主要有6个子类。这些子类定义了一些在处理数字时经常使用的有用方法。byte, integer, double, short, float, long， 所有的包装类（Integer、Long、Byte、Double、Float、Short）都是抽象类Number的子类。

为什么要在原始数据上使用Number类对象？

 - 数字类定义的常量（如MIN_VALUE和MAX_VALUE）提供数据类型的上限和下限，非常有用。
 - Number类对象可以用作期望对象的方法的参数（通常用于处理数字集合）。
 - 类方法可用于将值转换为其他基本类型以及从其他基本类型转换值，用于转换字符串和从字符串转换，以及用于在数字系统（十进制，八进制，十六进制，二进制）之间进行转换。

**子类通用方法：**

1. xxx xxxValue（）：这里xxx表示原始数字数据类型（byte，short，int，long，float，double）。此方法用于将此 Number对象的值转换为指定的基本数据类型。
```
//句法 ：
byte byteValue（）
short shortValue（）
int intValue（）
long longValue（）
float floatValue（）
double doubleValue（）
//参数： 无
//返回：此对象表示的数值转换为指定类型后
```
示例：
```java
//Java program to demonstrate xxxValue() method
public class Test
{
    public static void main(String[] args)
    {
        // Creating a Double Class object with value "6.9685"
        Double d = new Double("6666.9685");
        // Converting this Double(Number) object to
        // different primitive data types
        byte b = d.byteValue();
        short s = d.shortValue();
        int i = d.intValue();
        long l = d.longValue();
        float f = d.floatValue();
        double d1 = d.doubleValue();
        System.out.println("value of d after converting it to byte : " + b);
        System.out.println("value of d after converting it to short : " + s);
        System.out.println("value of d after converting it to int : " + i);
        System.out.println("value of d after converting it to long : " + l);
        System.out.println("value of d after converting it to float : " + f);
        System.out.println("value of d after converting it to double : " + d1);
    }
}
```
运行结果：
```shell
$ java Hello
value of d after converting it to byte : 10
value of d after converting it to short : 6666
value of d after converting it to int : 6666
value of d after converting it to long : 6666
value of d after converting it to float : 6666.9683
value of d after converting it to double : 6666.9685
```
注意：转换时，可能会发生精度损失。例如，我们可以看到在从Double对象转换为int数据类型时，小数部分（“.9685”）已被省略。也有可能发生溢出，如Double对象转换成byte时。


2. int compareTo（NumberSubClass referenceName）：此方法用于将此 Number对象与指定的参数进行比较。但是，不能比较两种不同的类型，因此参数和调用方法的Number对象应该是相同的类型。referenceName可以是Byte，Double，Integer，Float，Long或Short。
```
句法 ：
public int compareTo（NumberSubClass referenceName）
参数：
referenceName  - 任何NumberSubClass类型值
返回：
如果Number等于参数，则值为0。
如果Number小于参数，则值为-1。
如果Number大于参数，则值为1。
```
示例：
```java
//Java program to demonstrate compareTo() method
public class Test
{
    public static void main(String[] args)
    {
        // creating an Integer Class object with value "10"
        Integer i = new Integer("10");  
        // comparing value of i
        System.out.println(i.compareTo(7));
        System.out.println(i.compareTo(11));
        System.out.println(i.compareTo(10));
    }
}
```
输出结果：
```
$ java Hello
1
-1
0
```

3. boolean equals（Object obj）：该方法确定此 Number对象是否等于参数。
```
句法 ：
public boolean equals（Object obj）
参数：
obj  - 任何对象
返回：
如果参数不为null，是相同类型和相同数值的对象，则该方法返回true
否则为false。
```
示例：
```java
//Java program to demonstrate equals() method
public class Test
{
    public static void main(String[] args)
    {
        // creating a Short Class object with value "15"
        Short s = new Short("15");
        // creating a Short Class object with value "10"
        Short x = 10;
        // creating an Integer Class object with value "15"
        Integer y = 15;
        // creating another Short Class object with value "15"
        Short z = 15;
        //comparing s with other objects
        System.out.println(s.equals(x));
        System.out.println(s.equals(y));
        System.out.println(s.equals(z));
    }
}
```
运行结果：
```
$ java Hello
false
false
true
```

4. int parseInt（String s，int radix）：此方法用于获取String的原始数据类型。基数用于返回十进制（10），八进制（8）或十六进制（16）等表示形式作为输出。
每个Number子类都包含其他方法，这些方法可用于将数字转换为字符串以及从数字转换为数字系统。以下是Integer类中的一些方法。我们将逐个讨论它们。其他 Number子类的方法类似。例如，对于Integer类的“parseInt”方法，我们有Short类的“parseShort”方法。
```
句法 ：
static int parseInt（String s，int radix）
参数：
s  - 小数的任何字符串表示
radix - 任何基数值
返回：
由十进制参数表示的整数值。
抛出：
NumberFormatException：如果该字符串不包含可分析的整数。
```
还有另外一个函数int parseInt（String s）：此方法是上述方法的另一种变体，默认情况下基数为10（十进制）。
示例：
```java
//Java program to demonstrate Integer.parseInt() method
public class Test
{
    public static void main(String[] args)
    {
        // parsing different strings
        int z = Integer.parseInt("654",8);
        int a = Integer.parseInt("-FF", 16);
        long l = Long.parseLong("2158611234",10);
        System.out.println(z);
        System.out.println(a);
        System.out.println(l);
        // run-time NumberFormatException will occur here
        // "Geeks" is not a parsable string
        int x = Integer.parseInt("Geeks",8);           
        // run-time NumberFormatException will occur here
        // (for octal(8),allowed digits are [0-7])
        int y = Integer.parseInt("99",8);
    }
}
```
运行结果：
```
$ java Hello
428
-255
2158611234
Exception in thread "main" java.lang.NumberFormatException: For input string: "Geeks"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:580)
	at Hello.main(Hello.java:13)
```

5. String toString（）：toString（）方法有两种变体。它们用于获取数字的字符串表示形式。这些方法的其他变体是Integer.toBinaryString（int i），Integer.toHexString（int i），Integer.toOctalString（int i），它将分别返回指定整数（i）的二进制，十六进制，八进制字符串表示形式。
```
句法 ：
String toString（）
String toString（int i）
参数：
String toString（） - 无参数
String toString（int i） -  i：任何整数值
返回
String toString（） -
返回一个表示Number对象的值的String对象
在其上调用它。
String toString（int i） -
返回一个表示指定整数（i）的十进制String对象，
```
示例：
```java
//Java program to demonstrate Integer.toString()
//and Integer.toString(int i) method
public class Test
{
    public static void main(String[] args)
    {
        // demonstrating toString() method
        Integer x = 12;        
        System.out.println(x.toString());       
        // demonstrating toString(int i) method
        System.out.println(Integer.toString(12));        
        System.out.println(Integer.toBinaryString(152));
        System.out.println(Integer.toHexString(152));
        System.out.println(Integer.toOctalString(152));
    }
}
```
运行结果：
```
$ java Hello
12
12
10011000
98
230
```

6. Integer valueOf（）：valueOf（）方法有三种变体。所有这三个方法都返回一个Integer对象，它保存一个原始整数的值。
```
句法 ：
Integer valueOf(int i)
Integer valueOf(String s)
Integer valueOf(String s, int radix)
参数：
i- 任何整数值
s  - 小数的任何字符串表示
radix - 任何基数值
退货：
valueOf（int i）：保存由int参数表示的值的Integer对象。
valueOf（String s）：保存由字符串参数表示的值的Integer对象。
valueOf（String s，int radix）：保存值的Integer对象
 由带有基数的字符串参数表示。
抛出：
valueOf（String s） -
NumberFormatException：如果该字符串不包含可分析的整数。
valueOf（String s，int radix） -
NumberFormatException：如果该字符串不包含可分析的整数。
```
示例:
```Java
// Java program to demonstrate valueOf() method
public class Test
{
    public static void main(String[] args)
    {
        // demonstrating valueOf(int i) method
        System.out.println("Demonstrating valueOf(int i) method");
        Integer i =Integer.valueOf(50);
        Double d = Double.valueOf(9.36);
        System.out.println(i);
        System.out.println(d);
            // demonstrating valueOf(String s) method
        System.out.println("Demonstrating valueOf(String s) method");
        Integer n = Integer.valueOf("333");
        Integer m = Integer.valueOf("-255");
        System.out.println(n);
        System.out.println(m);            
        // demonstrating valueOf(String s,int radix) method
        System.out.println("Demonstrating (String s,int radix) method");
        Integer y = Integer.valueOf("333",8);
        Integer x = Integer.valueOf("-255",16);
        Long l = Long.valueOf("51688245",16);
        System.out.println(y);
        System.out.println(x);
        System.out.println(l);
        // run-time NumberFormatException will occur in below cases
        Integer a = Integer.valueOf("Geeks");
        Integer b = Integer.valueOf("Geeks",16);
    }
}
```
运行结果：
```
$ java Hello
Demonstrating valueOf(int i) method
50
9.36
Demonstrating valueOf(String s) method
333
-255
Demonstrating (String s,int radix) method
219
-597
1365803589
Exception in thread "main" java.lang.NumberFormatException: For input string: "Geeks"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
	at java.lang.Integer.parseInt(Integer.java:580)
	at java.lang.Integer.valueOf(Integer.java:766)
	at Hello.main(Hello.java:25)
```

#### Integer
java.lang包中的Integer、Long和Short类，分别将基础类型int、long、short封装成一个类。Integer类在对象中包装了一个基本类型int的值。该类提供除了父类的方法，还提供int类型和String类型之间互相转换。

Integer类是基本数据类型int的包装器类，是抽象类Number的子类，位于java.lang包中。

Integer类在对象中包装了一个基本类型int的值，也就是每个Integer对象包含一个int类型的字段。在Integer源码中如下定义：private final int value;

**字段：**
```
[static int]  MAX_VALUE：值为 2^31－1 的常量，它表示 int 类型能够表示的最大值。
[static int]  MIN_VALUE：值为 －2^31 的常量，它表示 int 类型能够表示的最小值。
[static int]  SIZE： 用来以二进制补码形式表示 int 值的比特位数。
[static Class<Integer>]  TYPE：表示基本类型 int 的 Class 实例。
[static int]  BYTES：返回int值所占的字节数。
```
**构造方法：**

Integer类提供了两种构造方法：它们都会返回一个Integer对象

（1）Integer(int value);

（2）Integer(String s);    //要注意的是字符串不能包含非数字字符，否则会抛出NumberFormatException

（3）除此之外，还可以给Integer对象直接赋值，如：Integer a = 10;

```java
Integer w = new Integer(100);
Integer q = new Integer("100");
```

1. **“==”和equals()方法：** 在Integer类中，“==”用来比较对象地址是否相同，并且Integer类重写了equals(Object obj)方法，在equals(Object obj)方法中，会先判断参数中的对象obj是否是Integer类型的对象，如果是则判断值是否相同，值相同则返回true，值不同则返回false,如果obj不是Integer类的对象，则返回false。需要注意的是：**当参数是基本类型int时，编译器会给int自动装箱成Integer类，然后再进行比较。**
```java
//int和Integer对象用构造方法、直接赋值方法赋值比较
int a = 200;
Integer b = new Integer(200);
Integer c = 200;        
//b和c都是Integer对象，所以用“==”比较时，就是比较内存地址值是否相等
System.out.println(b == c);  //false
System.out.println(b.equals(c));  //true
//a在进行比较时，会自动装箱
System.out.println(b.equals(a));  //true
//因为b是Integer对象，a是基本数据类型int的变量，当进行“==”比较时，b会自动拆箱成int，从而比较的就是变量的值是否相等
System.out.println(b == a);   //true
```

2. Integer直接赋值和valueOf()方法，下面来看下面两段代码的不同
```java
Integer i02 = 59;
Integer i03 = Integer.valueOf(59);
Integer i04 = new Integer(59);    
System.out.println(i02 == i03);  //true
System.out.println(i02 == i04);  //false
System.out.println(i03 == i04);  //false
Integer i02 = 200;
Integer i03 = Integer.valueOf(200);
Integer i04 = new Integer(200);        
System.out.println(i02 == i03);  //false
System.out.println(i02 == i04);  //false
System.out.println(i03 == i04);  //false
```

我们可以看到上面两段代码只是赋给对象的值不同，但是在判断”i02 == i03“时得到的结果却不同，这是为何：

当使用直接赋值如”Integer i01 = 59“的时候，会调用Integer的valueOf()方法，这个方法就是返回一个Integer对象，但是在返回前，作了一个判断，判断要赋给对象的值i是否在[-128,127]区间中，且IntegerCache（是Integer类的内部类，里面有一个Integer对象数组，**用于存放已经存在的且范围在[-128,127]中的对象**）中是否存在此对象，如果存在，则直接**返回引用**，否则，创建**一个新对象返回**。

那么我们就可以知道，200这个数字不在[-128，127]中，所以会直接创建一个新对象返回，i02和i03就是两个不同的对象。而59属于[-128，127]中，当创建i03时，会直接返回引用，此时i02和i03都指向同一个地址。

以第一段代码为例：”Integer i02 = 59"：因为程序初次运行，没有59，所以直接创建一个对象返回；“Integer i03 = Integer.valueOf(59)”：因为IntegerCache中已经存在59，所以直接返回引用；“Integer i04 = new Integer(59)”：直接创建一个新对象。

**JVM中一个字节以下的整型数据（即[128,127]）会在JVM启动时加载进内存，除非用new Integer()显示的创建对象，否则都是同一对象。**


#### Boolean

Boolean是boolean的包装类。存储为 8位（1 个字节）的数值形式,但只能是 True 或是 False。

方法中存在以下几个静态方法，在一般情况下，我们可以不需要使用new来创建实例，使用Boolean.TRUE,Boolean.FALSE,或者是Boolean.valueOf(boolean)，可能还会提高空间和时间性能。

构造函数：
```
Boolean（boolean val）：表示val参数的布尔对象。
Boolean（String str）：根据字符串分配表示值true或false的布尔对象.
```

源码：
```（java）
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
}
 public static Boolean valueOf(String s) {
        return toBoolean(s) ? TRUE : FALSE;
 }
```

**java中的boolean是否有默认值？**

有默认类型，是false。但是应该声明为**成员变量或是静态变量**，如果在方法体中（如main）不会自动赋值，如果使用会提示为初始化。Boolean默认值为null。

#### Byte
Byte是byte的包装类，大小从-128到+127。

封装有几种好处，比如：
1. Byte可以将对象的引用传递，使得多个function共同操作一个byte类型的数据，而byte基本数据类型是赋值之后要在stack(栈区域)进行存储的；

2. 定义了和String之间互相转化的方法。Byte的大小是8个字节。因为Byte是需要通过关键字new来申请对象，而此部分申请出来的对象放在内存的heap(堆区域)。

常量：

```
MIN_VALUE =-128
MAX_VALUE=127
SIZE 用于以二进制补码形式表示byte值的位数
```

#### Float
Float是float的封装类。

常量：
```
MAX_VALUE：值为 1.4E38 的常量，它表示 float 类型能够表示的最大值。
MIN_VALUE：值为 3.4E-45 的常量，它表示 float 类型能够表示的最小值。
MAX_EXPONENT:有限 float 变量可能具有的最大指数。
MIN_EXPONENT：标准化 float 变量可能具有的最小指数。
MIN_NORMAL：保存 float 类型数值的最小标准值的常量，即 2-126。
NaN：保存 float 类型的非数字值的常量。
SIZE：用来以二进制补码形式表示 float 值的比特位数。
TYPE：表示基本类型 float 的 Class 实例
```
 注意：在实现将字符串转换为 float 类型数值的过程中，如果字符串中包含非数值类型的字符，则程序执行将出现异常。

#### Character

Character是char的包装类。char类型占2个字节、16位可以存放汉子，字母和数字占一个字节，一个字节8位，中文占2个字节，16位；

构造方法:

```
public Character(char value)
构造一个新分配的 Character 对象，用以表示指定的 char 值
```

Character类的判断功能:
```
public static boolean isDigit(char ch)
确定指定字符是否为数字。

public static boolean isLetter(char ch)
 确定指定字符是否为字母。

public static boolean isLowerCase(char ch)
确定是否是小写字母字符

public static boolean isUpperCase(char ch)
确定是否大写字母字符

Character类中的isWhitespace方法用来判断指定字符是否为空白字符，空白字符包括：空格、tab键、换行

```

两个转换功能:
```
public static int toLowerCase(char ch)
使用取自 UnicodeData 文件的大小写映射信息将字符参数转换为小写。
public static int toUpperCase(char ch)
使用取自 UnicodeData 文件的大小写映射信息将字符参数转换为大写。
```

### 数学处理类

为了应对数学问题、随机问题、大数问题、科学技术问题，Java提供了处理相关问题的类，包括DecimalFormat类（用于格式化数字）、Math类（数学计算工具类）、Random类（随机数类）、BigInteger和BigDecimal类（处理大数问题）。

#### 数字格式化

 java开发中经常会有数字、货币金钱等格式化需求，货币保留几位小数，货币前端需要加上货币符号等。可以用java.text.NumberFormat和java.text.DecimalFormat实现。

 DecimalFormat 是 NumberFormat 的一个子类，用于格式化十进制数字。他可以将一些数字格式化为整数、浮点数、百分数等。通过使用该类可以为要输出的数字加上单位或控制数字的精度。

DecimalFormat 类中特殊字符说明

![](https://i.loli.net/2019/04/10/5cadaf59e2898.png)

使用构造方法创建：DecimalFormat()创建初始化对象后可通过 applyPattern(String pattern)指定模板或者直接用DecimalFormat(String pattern)创建。

指定的模板不包含逗号则会将NumberFormat属性groupingUse设置为false，即使用setGroupingSize(int size)指定分组大小也不能起效，此时可用setGroupingUse方法让其起效。

同样可以用以下方式实例化：
```java
DecimalFormat df=(DecimalFormat)NumberFormat.getInstance();
DecimalFormat df1=(DecimalFormat) DecimalFormat.getInstance();
```

默认小数位数为3位，如：
```
DecimalFormat df=(DecimalFormat)NumberFormat.getInstance();
System.out.println(df.format(12.3456));
```
输出：12.346（四舍五入）

现在可以通过如下方法把小数为设为两位：
```
df.setMaximumFractionDigits(2);
System.out.println(df.format(12.3456789));
```
则输出为：12.35

将数字转化为百分比输出
```
df.applyPattern("##.##%");
System.out.println(df.format(12.3456789));
System.out.println(df.format(1));
System.out.println(df.format(0.015));
```
输出分别为：1234.57%    100%      1.5%

设置分组大小 ：
```
DecimalFormat df1=(DecimalFormat) DecimalFormat.getInstance();
df1.setGroupingSize(2);
System.out.println(df1.format(123456789));
```
输出：1,23,45,67,89

设置小数为必须为2位
```
DecimalFormat df2=(DecimalFormat) DecimalFormat.getInstance();
df2.applyPattern("0.00");
System.out.println(df2.format(1.2));
```
输出：1.20

```java
import java.text.DecimalFormat;

public class DecimalFormatSimpleDemo {
    //使用实例化对象时设置格式化模式
    static public void SimpleFormat(String pattern, double value)   {
        //实例化 DecimalFormat 对象
        DecimalFormat myFormat = new DecimalFormat(pattern);
        String output = myFormat.format(value);//将数字格式化
        System.out.println(value+" "+pattern+" "+output);
    }

    //使用 applyPattern() 方法对数字进行格式化
    static public void UseApplyPatternMethodFormat(String pattern, double value) {
        DecimalFormat myFormat = new DecimalFormat();
        myFormat.applyPattern(pattern);
        System.out.println(value+" "+pattern+" "+myFormat.format(value));
    }

    public static void main(String[] args) {
        SimpleFormat("###,###.###", 123456.789);
        SimpleFormat("00000000.###kg", 123456.789);
        //按照格式模式格式化数字，不存在的位以 0 显示
        SimpleFormat("000000.000", 123.78);
        //调用静态 UseApplyPatternMethodDormat()方法
        UseApplyPatternMethodFormat("#.###%", 0.789);
        //将小数点够格式化为两位
        UseApplyPatternMethodFormat("###.##", 123456.789);
        //将数字格式为 千分数形式
        UseApplyPatternMethodFormat("0.00\u2030", 0.789);
    }
}
```
运行结果：
```
$ java  DecimalFormatSimpleDemo
123456.789 ###,###.### 123,456.789
123456.789 00000000.###kg 00123456.789kg
123.78 000000.000 000123.780
0.789 #.###% 78.9%
123456.789 ###.## 123456.79
0.789 0.00‰ 789.00‰
```

#### 数学运算
Java的Math类封装了很多与数学有关的属性和方法。

示例：
```java
private void mathMethod(){

        /**
         * Math.sqrt()//计算平方根
         * Math.cbrt()//计算立方根
         * Math.hypot(x,y)//计算 (x的平方+y的平方)的平方根
         */

        Log.d("TAG","Math.sqrt(16)----:"+Math.sqrt(16));//4.0
        Log.d("TAG","Math.cbrt(8)----:"+Math.cbrt(8));//2.0
        Log.d("TAG","Math.hypot(3,4)----:"+Math.hypot(3,4));//5.0

        /**
         * Math.pow(a,b)//计算a的b次方
         * Math.exp(x)//计算e^x的值
         * */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.pow(3,2)----:"+Math.pow(3,2));//9.0
        Log.d("TAG","Math.exp(3)----:"+Math.exp(3));//20.085536923187668

        /**
         * Math.max();//计算最大值
         * Math.min();//计算最小值
         * */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.max(2.3,4.5)----:"+Math.max(7,15));//15
        Log.d("TAG","Math.min(2.3,4.5)----:"+Math.min(2.3,4.5));//2.3

        /**
         * Math.abs求绝对值
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.abs(-10.4)----:"+Math.abs(-10.4));//10.4
        Log.d("TAG","Math.abs(10.1)----:"+Math.abs(10.1));//10.1

        /**
         * Math.ceil天花板的意思，就是返回大的值
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.ceil(-10.1)----:"+Math.ceil(-10.1));//-10.0
        Log.d("TAG","Math.ceil(10.7)----:"+Math.ceil(10.7));//11.0
        Log.d("TAG","Math.ceil(-0.7)----:"+Math.ceil(-0.7));//-0.0
        Log.d("TAG","Math.ceil(0.0)----:"+Math.ceil(0.0));//0.0
        Log.d("TAG","Math.ceil(-0.0)----:"+Math.ceil(-0.0));//-0.0
        Log.d("TAG","Math.ceil(-1.7)----:"+Math.ceil(-1.7));//-1.0

        /**
         * Math.floor地板的意思，就是返回小的值
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.floor(-10.1)----:"+Math.floor(-10.1));//-11.0
        Log.d("TAG","Math.floor(10.7)----:"+Math.floor(10.7));//10.0
        Log.d("TAG","Math.floor(-0.7)----:"+Math.floor(-0.7));//-1.0
        Log.d("TAG","Math.floor(0.0)----:"+Math.floor(0.0));//0.0
        Log.d("TAG","Math.floor(-0.0)----:"+Math.floor(-0.0));//-0.0

        /**
         * Math.random 取得一个大于或者等于0.0小于不等于1.0的随机数[0,1)
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.random()----:"+Math.random());//输出[0,1)间的随机数 0.8979626325354049
        Log.d("TAG","Math.random()*100----:"+Math.random()*100);//输出[0,100)间的随机数 32.783762836248144

        /**
         * Math.rint 四舍五入
         * 返回double值
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.rint(10.1)----:"+Math.rint(10.1));//10.0
        Log.d("TAG","Math.rint(10.7)----:"+Math.rint(10.7));//11.0
        Log.d("TAG","Math.rint(-10.5)----:"+Math.rint(-10.5));//-10.0
        Log.d("TAG","Math.rint(-10.51)----:"+Math.rint(-10.51));//-11.0
        Log.d("TAG","Math.rint(-10.2)----:"+Math.rint(-10.2));//-10.0
        Log.d("TAG","Math.rint(9)----:"+Math.rint(9));//9.0


        /**
         * Math.round 四舍五入
         * float时返回int值，double时返回long值
         */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.round(10.1)----:"+Math.round(10.1));//10
        Log.d("TAG","Math.round(10.7)----:"+Math.round(10.7));//11
        Log.d("TAG","Math.round(-10.5)----:"+Math.round(-10.5));//-10
        Log.d("TAG","Math.round(-10.51)----:"+Math.round(-10.51));//-11
        Log.d("TAG","Math.round(-10.2)----:"+Math.round(-10.2));//-10
        Log.d("TAG","Math.round(9)----:"+Math.round(9));//9

        /**
         * Math.nextUp(a) 返回比a大一点点的浮点数
         * Math.nextDown(a) 返回比a小一点点的浮点数
         * Math.nextAfter(a,b) 返回(a,b)或(b,a)间与a相邻的浮点数 b可以比a小
         * */

        Log.d("TAG","------------------------------------------");
        Log.d("TAG","Math.nextUp(1.2)----:"+Math.nextUp(1.2));//1.2000000000000002
        Log.d("TAG","Math.nextDown(1.2)----:"+Math.nextDown(1.2));//1.1999999999999997
        Log.d("TAG","Math.nextAfter(1.2, 2.7)----:"+Math.nextAfter(1.2, 2.7));//1.2000000000000002
        Log.d("TAG","Math.nextAfter(1.2, -1)----:"+Math.nextAfter(1.2, -1));//1.1999999999999997
    }
```

#### 随机数

在Java中，随机数的概念从广义上将，有三种。

1. 通过System.currentTimeMillis()来获取一个当前时间毫秒数的long型数字。

2. 通过Math.random()返回一个0到1之间的double值。

3. 通过Random类来产生一个随机数，这个是专业的Random工具类，功能强大。


**Math.random()方法，** 默认返回[0,1)中的double值。生成的是伪随机数，是根据当前时间作为随机数生成器的参数而进行复杂运算的结果。

获得随机整数：

```
//获取[0,n)的int数
(int)(Math.random()*n)
//获取[m,m+n)的int数
(int)(Math.random()*n)+m
```

获取随机字符：
```
//生成a-z之间字符
(char)('a'+Math.random()*('z'-'a'+1))
//求任意两个字符之间的随机字符
(char)('char1'+Math.random()*('char1'-'char2'+1))
```

**通过System.currentTimeMillis()来获取一个当前时间毫秒数的long型数字**

```
final long l = System.currentTimeMillis();
```
若要获取int类型的整数，只需要将上面的结果转行成int类型即可。比如，获取[0, 100)之间的int整数。方法如下：
```
final long l = System.currentTimeMillis();
final int i = (int)( l % 100 );
```

**通过Random类来获取随机数**

1、java.util.Random类中实现的随机算法是伪随机，也就是有规则的随机，所谓有规则的就是在给定种(seed)的区间内随机生成数字；

2、相同种子数的Random对象，相同次数生成的随机数字是完全相同的；

3、Random类中各方法生成的随机数字都是均匀分布的，也就是说区间内部的数字生成的几率均等；


函数用法：
```java
// 构造函数(一)： 创建一个新的随机数生成器。
Random()
// 构造函数(二)： 使用单个 long 种子创建一个新随机数生成器： 
public Random(long seed) { setSeed(seed); } 
//next 方法使用它来保存随机数生成器的状态。
Random(long seed)

boolean nextBoolean()         // 返回下一个“boolean类型”伪随机数。
void    nextBytes(byte[] buf) // 生成随机字节并将其置于字节数组buf中。
double  nextDouble()          // 返回一个“[0.0, 1.0) 之间的double类型”的随机数。
float   nextFloat()           // 返回一个“[0.0, 1.0) 之间的float类型”的随机数。
int     nextInt()             // 返回下一个“int类型”随机数。
int     nextInt(int n)        // 返回一个“[0, n) 之间的int类型”的随机数。
long    nextLong()            // 返回下一个“long类型”随机数。
synchronized double nextGaussian()   // 返回下一个“double类型”的随机数，
//它是呈高斯（“正常地”）分布的 double 值，其平均值是 0.0，标准偏差是 1.0。
synchronized void setSeed(long seed) // 使用单个 long 种子设置此随机数生成器的种子。

```

示例：
```java
import java.util.Random;
import java.lang.Math;

/**
 * java 的随机数测试程序。共3种获取随机数的方法：
 *   (01)、通过System.currentTimeMillis()来获取一个当前时间毫秒数的long型数字。
 *   (02)、通过Math.random()返回一个0到1之间的double值。
 *   (03)、通过Random类来产生一个随机数，这个是专业的Random工具类，功能强大。
 *
 * @author skywang
 * @email kuiwu-wang@163.com
 */
public class RandomTest{

    public static void main(String args[]){

        // 通过System的currentTimeMillis()返回随机数
        testSystemTimeMillis();

        // 通过Math的random()返回随机数
        testMathRandom();

        // 新建“种子为1000”的Random对象，并通过该种子去测试Random的API
        testRandomAPIs(new Random(1000), " 1st Random(1000)");
        testRandomAPIs(new Random(1000), " 2nd Random(1000)");
        // 新建“默认种子”的Random对象，并通过该种子去测试Random的API
        testRandomAPIs(new Random(), " 1st Random()");
        testRandomAPIs(new Random(), " 2nd Random()");
    }

    /**
     * 返回随机数-01：测试System的currentTimeMillis()
     */
    private static void testSystemTimeMillis() {
        // 通过
        final long l = System.currentTimeMillis();
        // 通过l获取一个[0, 100)之间的整数
        final int i = (int)( l % 100 );

        System.out.printf("\n---- System.currentTimeMillis() ----\n l=%s i=%s\n", l, i);
    }


    /**
     * 返回随机数-02：测试Math的random()
     */
    private static void testMathRandom() {
        // 通过Math的random()函数返回一个double类型随机数，范围[0.0, 1.0)
        final double d = Math.random();
        // 通过d获取一个[0, 100)之间的整数
        final int i = (int)(d*100);

        System.out.printf("\n---- Math.random() ----\n d=%s i=%s\n", d, i);
    }


    /**
     * 返回随机数-03：测试Random的API
     */
    private static void testRandomAPIs(Random random, String title) {
        final int BUFFER_LEN = 5;

        // 获取随机的boolean值
        boolean b = random.nextBoolean();
        // 获取随机的数组buf[]
        byte[] buf = new byte[BUFFER_LEN];
        random.nextBytes(buf);
        // 获取随机的Double值，范围[0.0, 1.0)
        double d = random.nextDouble();
        // 获取随机的float值，范围[0.0, 1.0)
        float f = random.nextFloat();
        // 获取随机的int值
        int i1 = random.nextInt();
        // 获取随机的[0,100)之间的int值
        int i2 = random.nextInt(100);
        // 获取随机的高斯分布的double值
        double g = random.nextGaussian();
        // 获取随机的long值
        long l = random.nextLong();

        System.out.printf("\n---- %s ----\nb=%s, d=%s, f=%s, i1=%s, i2=%s, g=%s, l=%s, buf=[",
                title, b, d, f, i1, i2, g, l);
        for (byte bt:buf)
            System.out.printf("%s, ", bt);
        System.out.println("]");
    }
}
```
运行结果：
```
$ java  RandomTest

---- System.currentTimeMillis() ----
 l=1554888487975 i=75

---- Math.random() ----
 d=0.5375805102718979 i=53

----  1st Random(1000) ----
b=true, d=0.46028809169559504, f=0.015927613, i1=169247282, i2=45, g=-0.719106498075259, l=-7363680848376404625, buf=[47, -38, 53, 63, -72, ]

----  2nd Random(1000) ----
b=true, d=0.46028809169559504, f=0.015927613, i1=169247282, i2=45, g=-0.719106498075259, l=-7363680848376404625, buf=[47, -38, 53, 63, -72, ]

----  1st Random() ----
b=false, d=0.8938228115934014, f=0.8739582, i1=-1072778891, i2=51, g=-0.1970997024484297, l=3079912163885517301, buf=[-22, 127, -123, 122, 46, ]

----  2nd Random() ----
b=false, d=0.9775445491815494, f=0.81679565, i1=-2117941062, i2=58, g=-0.43305394609143333, l=1390203802753791438, buf=[-66, -120, -24, 77, -101, ]
```

**缺点：**

 Java的随机数产生是通过线性同余公式产生的,也就是说通过一个复杂的算法生成的。Java自带的随机数函数是很容易被黑客破解的,因为黑客可以通过获取一定长度的随机数序列来推出你的seed,然后就可以预测下一个随机数。

不用种子的不随机性会增大的原因:java.Math.Random()实际是在内部调用java.util.Random()的,使用一个和当前系统时间有关的数字作为种子数。两个随机数就很可能相同。

**总结**

1. 同一个种子,生成N个随机数,当你设定种子的时候,这N个随机数是什么已经确定。相同次数生成的随机数字是完全相同的。
2. 如果用相同的种子创建两个 Random 实例,如上面的r3,r4,则对每个实例进行相同的方法调用序列,它们将生成并返回相同的数字序列。
3. Java的随机数都是通过算法实现的,Math.random()本质上属于Random()类。
4. 使用java.util.Random()会相对来说比较灵活一些。

#### 大数运算

在算法竞赛中我们经常遇到大数问题，例如求一个很大的斐波那契数。住在这种情况下我们用常规解法（使用long long或long long int）肯定是不行的，而我们自己写一个大数的算法又过于麻烦且易于出错，在这种情况下使用java中自带的大数类是我们最好的选择

java中用于操作大数的类主要有两个，一个是BigInteger，代表大整数类用于对大整数进行操作，另一个是BigDecimal，代表大浮点型。因为这两种类的使用方法是一样的且通常情况下我们处理的数据是整数，所以下面我们以BigInteger为例进行讲解

java能处理大数的类有两个高精度大整数BigInteger 和高精度浮点数BigDecimal，这两个类位于java.math包内，要使用它们必须在类前面引用该包：`import java.math.BigInteger;`和`import java.math.BigDecimal;`或者`import java.math.*`;


**常量**

BigInteger:ONE,ZERO,TEN分别代表1,0,10.

其定义类似于:public static final BigInteger ONE = valueOf(1);

BigDecimal:除了以上三个常量外还有8个关于舍入的常量:ROUND_UP,ROUND_DOWN,ROUND_CEILING,ROUND_FLOOR,ROUND_HALF_UP,


**声明赋值**
```java
BigInteger:BigInteger bi = new BigInteger(byte[] val) ;
new BigInteger(int signum, byte[] magnitude) ;
new BigInteger(int bitLength, int certainty, Random rnd) 。
new BigInteger(int numBits, Random rnd) 。
new BigInteger(String val) 。
new BigInteger(String val, int radix) ；
```
构造函数仅仅能接受这几种类型，,比方这样定义就是错误的:BigInteger bi = new BigInteger(100);

或:BigInteger bi = BigInteger.valueOf(100);

数组定义与基本类型类似.
```java
//BigDecimal:
BigDecimal bd = new BigDecimal(100);
//或:
BigDecimal bd = BigDecimal.valueOf(100);
```

BigDecimal的构造函数比BigInteger多一些,感觉用起来更方便些

顺便说一下,java.util包中的Scanner类实现了nextBigInteger()和nextBigDecimal()方法,能够用来读入控制台输入的BigInteger和BigDecimal。


add(),subtract(),pow(),abs()，multiply（）等等这一类就不介绍了，奇妙的是probablePrime(int bitLength, Random rnd)，  nextProbablePrime()这一类竟然和素数扯得上关系。


BigDecimal关于格式控制的方法多了几个，这对处理各种不同格式的输出是非常实用的。

stripTraillingZeros():把不影响数值大小的0全去掉。
```
1.50 ->1.5；
1.00->1;
```

这功能非常实用吧。

大家都知道JAVA的类一般都要带toString这种方法的。BigDecimal则有toString,toPlainString和toEngineeringString三种表示成字符串的方法。

以下是这三种方法各自的特点：

```
toString: using scientific notation if an exponent is needed;
toEngineeringString:using engineering notation if an exponent is needed.
toPlainString:without an exponent field.
```

### 面试题

1. int和Integer有什么区别？

   答：Java是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入了基本数据类型，但是为了能够将这些基本数据类型当成对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class），int的包装类就是Integer，从Java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。
    Java 为每个原始类型提供了包装类型：

   - 原始类型: boolean，char，byte，short，int，long，float，double
   - 包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

   ```java
   class AutoUnboxingTest {
   
       public static void main(String[] args) {
           Integer a = new Integer(3);
           Integer b = 3;                  // 将3自动装箱成Integer类型
           int c = 3;
           System.out.println(a == b);     // false 两个引用没有引用同一对象
           System.out.println(a == c);     // true a自动拆箱成int类型再和c比较
       }
   }
   
   ```

   最近还遇到一个面试题，也是和自动装箱和拆箱有点关系的，代码如下所示：

   ```java
   public class Test03 {
   
       public static void main(String[] args) {
           Integer f1 = 100, f2 = 100, f3 = 150, f4 = 150;
   
           System.out.println(f1 == f2);
           System.out.println(f3 == f4);
       }
   }
   
   ```

   如果不明就里很容易认为两个输出要么都是true要么都是false。首先需要注意的是f1、f2、f3、f4四个变量都是Integer对象引用，所以下面的==运算比较的不是值而是引用。装箱的本质是什么呢？当我们给一个Integer对象赋一个int值的时候，会调用Integer类的静态方法valueOf，如果看看valueOf的源代码就知道发生了什么。

   ```java
   public static Integer valueOf(int i) {
       if (i >= IntegerCache.low && i <= IntegerCache.high)
           return IntegerCache.cache[i + (-IntegerCache.low)];
       return new Integer(i);
   }
   ```

   简单的说，如果整型字面量的值在-128到127之间，那么不会new新的Integer对象，而是直接引用常量池中的Integer对象，所以上面的面试题中`f1==f2`的结果是true，而`f3==f4`的结果是false。
   
2. 阅读下面程序，请写出输出结果。

   ```java
   package eclipse;
    
   public class Test_Integer {
    
      public static void main(String[] args) {
    
        Integer i1 = new Integer(97);   
        Integer i2 = new Integer(97); 
        System.out.println(i1 == i2);
        System.out.println(i1.equals(i2));
        System.out.println("----------------");
    
        Integer i3 = new Integer(197);  
        Integer i4 = new Integer(197);
        System.out.println(i3 == i4);
        System.out.println(i3.equals(i4));
        System.out.println("----------------");
    
        Integer i5 = 97;   
        Integer i6 = 97; 
        System.out.println(i5 == i6);
        System.out.println(i5.equals(i6));
        System.out.println("----------------");
    
        Integer i7 = 197;  
        Integer i8 = 197;
        System.out.println(i7 == i8);
        System.out.println(i7.equals(i8));
    
      }
   }
   ```

   运行结果：

   ```
   false
   true
   ----------------
   false
   true
   ----------------
   true
   true
   ----------------
   false
   true
   ```

   如果变量i大于-128并且小于127，Cache是缓存的意思，这里真实的意思就是去常量池去取。如果不在-128到127范围，就使用new关键字去新建一个Integer对象。

   