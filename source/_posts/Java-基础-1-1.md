---
title: Java 基础语法
date: 2019-04-04 08:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 前言

一个 Java 程序可以认为是一系列对象的集合，而这些对象通过调用彼此的方法来协同工作。下面简要介绍下类、对象、方法和实例变量的概念。

- 对象：对象是类的一个实例，有状态和行为。例如，一条狗是一个对象，它的状态有：颜色、名字、品种；行为有：摇尾巴、叫、吃等。
- 类：类是一个模板，它描述一类对象的行为和状态。
- 方法：方法就是行为，一个类可以有很多方法。逻辑运算、数据修改以及所有动作都是在方法中完成的。
- 实例变量：每个对象都有独特的实例变量，对象的状态由这些实例变量的值决定。
<!--more-->

#### java 思想

- 万物皆为对象。（封装）
- 程序是对象的集合，它们通过发送消息来告知彼此所要做的。（方法调用）
- 每个对象都有自己的由其他对象所构成的存储。（基本类型变量或自定义类型变量）
- 每个对象都拥有其类型。（每个对象都是该类型的实例，对应java中的class）

- 某一特定类型的所有对象都可以接收同样的消息。（多态）

#### 注意

编写 Java 程序时，应注意以下几点：

- 大小写敏感：Java 是大小写敏感的，这就意味着标识符 Hello 与 hello 是不同的。
- 类名：对于所有的类来说，类名的首字母应该大写。如果类名由若干单词组成，那么每个单词的首字母应该大写，例如 MyFirstJavaClass 。
- 方法名：所有的方法名都应该以小写字母开头。如果方法名含有若干单词，则后面的每个单词首字母大写。
- 源文件名：源文件名必须和类名相同。当保存文件的时候，你应该使用类名作为文件名保存（切记 Java 是大小写敏感的），文件名的后缀为 .java。（如果文件名和类名不相同则会导致编译错误）。
- 主方法入口：所有的 Java 程序由 public static void main(String []args) 方法开始执行。
- 语句以分号结尾

>以下示例以类名为文件名，默认使用javac编译源代码文件，使用java 运行class文件


### Java 标识符

Java 所有的组成部分都需要名字。类名、变量名以及方法名都被称为标识符。

关于 Java 标识符，有以下几点需要注意：
```
所有的标识符都应该以字母（A-Z 或者 a-z）,美元符（$）、或者下划线（_）开始
首字符之后可以是字母（A-Z 或者 a-z）,美元符（$）、下划线（_）或数字的任何字符组合
关键字不能用作标识符
标识符是大小写敏感的
合法标识符举例：age、$salary、_value、__1_value
非法标识符举例：123abc、-salary
```
一般类名、接口名使用大写字母开头，每个单词首字符大写，方法名、字段名使用驼峰命名法。具体参考命名规则。

#### 主类结构

java是面对对象的语言，基本组成单位是类，类中又包括属性和方法。默认类都是不能运行的，需要在main（）方法中使用，含有main（）方法的类称为主类，一个程序只能有一个主类。主类格式如下：
```（java）
package test;
public class Test{
public static void main(String [] args)
{
  ...
}  
}
```
在下面示例中，都是用到了main方法，所以才能够运行。
#### 包声明
java程序由若干个类组成，如果类是同名或者需要对不同的类进行分类，那就需要用到包，比如说在项目目录下有两个文件夹：dog、cat，每个文件夹中都有一个类。
```
./
├── cat
│   └── Cat.java
└── dog
    └── Dog.java
```
Cat.java中的内容：
```（java）
package cat; //包名
public class Cat{
public String  Whoami()
{
  retirn "I'm a Cat";
}  
}

```


### Java 基本数据类型
变量就是申请内存来存储值。也就是说，当创建变量的时候，需要在内存中申请空间。

内存管理系统根据变量的类型为变量分配存储空间，分配的空间只能用来储存该类型数据。

因此，通过定义不同类型的变量，可以在内存中储存整数、小数或者字符。


#### Java 的两大数据类型

```
内置数据类型
引用数据类型
```
**内置数据类型**

Java语言提供了八种基本类型。六种数字类型（四个整数型，两个浮点型），一种字符类型，还有一种布尔型。

byte：
```
byte 数据类型是8位、有符号的，以二进制补码表示的整数；
最小值是 -128（-2^7）；
最大值是 127（2^7-1）；
默认值是 0；
byte 类型用在大型数组中节约空间，主要代替整数，因为 byte 变量占用的空间只有 int 类型的四分之一；
例子：byte a = 100，byte b = -50。
```

short：
```
short 数据类型是 16 位、有符号的以二进制补码表示的整数
最小值是 -32768（-2^15）；
最大值是 32767（2^15 - 1）；
Short 数据类型也可以像 byte 那样节省空间。一个short变量是int型变量所占空间的二分之一；
默认值是 0；
例子：short s = 1000，short r = -20000。
```

int：
```
int 数据类型是32位、有符号的以二进制补码表示的整数；
最小值是 -2,147,483,648（-2^31）；
最大值是 2,147,483,647（2^31 - 1）；
一般地整型变量默认为 int 类型；
默认值是 0 ；
例子：int a = 100000, int b = -200000。
```

long：
```
long 数据类型是 64 位、有符号的以二进制补码表示的整数；
最小值是 -9,223,372,036,854,775,808（-2^63）；
最大值是 9,223,372,036,854,775,807（2^63 -1）；
这种类型主要使用在需要比较大整数的系统上；
默认值是 0L；
例子： long a = 100000L，Long b = -200000L。
"L"理论上不分大小写，但是若写成"l"容易与数字"1"混淆，不容易分辩。所以最好大写。
```

float：
```
float 数据类型是单精度、32位、符合IEEE 754标准的浮点数；
float 在储存大型浮点数组的时候可节省内存空间；
默认值是 0.0f；
浮点数不能用来表示精确的值，如货币；
例子：float f1 = 234.5f。
```

double：
```
double 数据类型是双精度、64 位、符合IEEE 754标准的浮点数；
浮点数的默认类型为double类型；
double类型同样不能表示精确的值，如货币；
默认值是 0.0d；
例子：double d1 = 123.4。
```

boolean：
```
boolean数据类型表示一位的信息；
只有两个取值：true 和 false；
这种类型只作为一种标志来记录 true/false 情况；
默认值是 false；
例子：boolean one = true。
```

char：
```
char类型是一个单一的 16 位 Unicode 字符；
最小值是 \u0000（即为0）；
最大值是 \uffff（即为65,535）；
char 数据类型可以储存任何字符；
例子：char letter = 'A';。
```

**实例**

对于数值类型的基本类型的取值范围，我们无需强制去记忆，因为它们的值都已经以常量的形式定义在对应的包装类中了。请看下面的例子：
```（java）

public class PrimitiveTypeTest {  
    public static void main(String[] args) {  
        // byte  
        System.out.println("基本类型：byte 二进制位数：" + Byte.SIZE);  
        System.out.println("包装类：java.lang.Byte");  
        System.out.println("最小值：Byte.MIN_VALUE=" + Byte.MIN_VALUE);  
        System.out.println("最大值：Byte.MAX_VALUE=" + Byte.MAX_VALUE);  
        System.out.println();  

        // short  
        System.out.println("基本类型：short 二进制位数：" + Short.SIZE);  
        System.out.println("包装类：java.lang.Short");  
        System.out.println("最小值：Short.MIN_VALUE=" + Short.MIN_VALUE);  
        System.out.println("最大值：Short.MAX_VALUE=" + Short.MAX_VALUE);  
        System.out.println();  

        // int  
        System.out.println("基本类型：int 二进制位数：" + Integer.SIZE);  
        System.out.println("包装类：java.lang.Integer");  
        System.out.println("最小值：Integer.MIN_VALUE=" + Integer.MIN_VALUE);  
        System.out.println("最大值：Integer.MAX_VALUE=" + Integer.MAX_VALUE);  
        System.out.println();  

        // long  
        System.out.println("基本类型：long 二进制位数：" + Long.SIZE);  
        System.out.println("包装类：java.lang.Long");  
        System.out.println("最小值：Long.MIN_VALUE=" + Long.MIN_VALUE);  
        System.out.println("最大值：Long.MAX_VALUE=" + Long.MAX_VALUE);  
        System.out.println();  

        // float  
        System.out.println("基本类型：float 二进制位数：" + Float.SIZE);  
        System.out.println("包装类：java.lang.Float");  
        System.out.println("最小值：Float.MIN_VALUE=" + Float.MIN_VALUE);  
        System.out.println("最大值：Float.MAX_VALUE=" + Float.MAX_VALUE);  
        System.out.println();  

        // double  
        System.out.println("基本类型：double 二进制位数：" + Double.SIZE);  
        System.out.println("包装类：java.lang.Double");  
        System.out.println("最小值：Double.MIN_VALUE=" + Double.MIN_VALUE);  
        System.out.println("最大值：Double.MAX_VALUE=" + Double.MAX_VALUE);  
        System.out.println();  

        // char  
        System.out.println("基本类型：char 二进制位数：" + Character.SIZE);  
        System.out.println("包装类：java.lang.Character");  
        // 以数值形式而不是字符形式将Character.MIN_VALUE输出到控制台  
        System.out.println("最小值：Character.MIN_VALUE="  
                + (int) Character.MIN_VALUE);  
        // 以数值形式而不是字符形式将Character.MAX_VALUE输出到控制台  
        System.out.println("最大值：Character.MAX_VALUE="  
                + (int) Character.MAX_VALUE);  
    }  
}
```

运行结果：
```（shell）
$ javac  PrimitiveTypeTest.java
$ java  PrimitiveTypeTest
基本类型：byte 二进制位数：8
包装类：java.lang.Byte
最小值：Byte.MIN_VALUE=-128
最大值：Byte.MAX_VALUE=127

基本类型：short 二进制位数：16
包装类：java.lang.Short
最小值：Short.MIN_VALUE=-32768
最大值：Short.MAX_VALUE=32767

基本类型：int 二进制位数：32
包装类：java.lang.Integer
最小值：Integer.MIN_VALUE=-2147483648
最大值：Integer.MAX_VALUE=2147483647

基本类型：long 二进制位数：64
包装类：java.lang.Long
最小值：Long.MIN_VALUE=-9223372036854775808
最大值：Long.MAX_VALUE=9223372036854775807

基本类型：float 二进制位数：32
包装类：java.lang.Float
最小值：Float.MIN_VALUE=1.4E-45
最大值：Float.MAX_VALUE=3.4028235E38

基本类型：double 二进制位数：64
包装类：java.lang.Double
最小值：Double.MIN_VALUE=4.9E-324
最大值：Double.MAX_VALUE=1.7976931348623157E308

基本类型：char 二进制位数：16
包装类：java.lang.Character
最小值：Character.MIN_VALUE=0
最大值：Character.MAX_VALUE=65535
```

Float和Double的最小值和最大值都是以科学记数法的形式输出的，结尾的"E+数字"表示E之前的数字要乘以10的多少次方。比如3.14E3就是3.14 × 103 =3140，3.14E-3 就是 3.14 x 10-3 =0.00314。

实际上，JAVA中还存在另外一种基本类型void，它也有对应的包装类 java.lang.Void，不过我们无法直接对它们进行操作。

**引用类型**

在Java中，引用类型的变量非常类似于C/C++的指针。引用类型指向一个对象，指向对象的变量是引用变量。这些变量在声明时被指定为一个特定的类型，比如 Employee、Puppy 等。变量一旦声明后，类型就不能被改变了。
对象、数组都是引用数据类型。

所有引用类型的默认值都是null。

一个引用变量可以用来引用任何与之兼容的类型。

例子：`Site site = new Site("HYP")`。

#### Java 常量
常量在程序运行时是不能被修改的。

在 Java 中使用 final 关键字来修饰常量，声明方式和变量类似：
```
final double PI = 3.1415927;
```

虽然常量名也可以用小写，但为了便于识别，通常使用大写字母表示常量。

字面量可以赋给任何内置类型的变量。例如：
```
byte a = 68;
char a = 'A'
```
byte、int、long、和short都可以用十进制、16进制以及8进制的方式来表示。

当使用常量的时候，前缀 0 表示 8 进制，而前缀 0x 代表 16 进制, 例如：
```
int decimal = 100;
int octal = 0144;
int hexa =  0x64;
```

和其他语言一样，Java的字符串常量也是包含在两个引号之间的字符序列。下面是字符串型字面量的例子：
```
"Hello World"
"two\nlines"
"\"This is in quotes\""
```

字符串常量和字符常量都可以包含任何Unicode字符。例如：
```
char a = '\u0001';
String a = "\u0001";
```

**转义字符**
![](https://i.loli.net/2019/04/08/5cab1b43b188a.png)

#### 自动类型转换

**整型、实型（常量）、字符型数据可以混合运算。运算中，不同类型的数据先转化为同一类型，然后进行运算。**

转换从低级到高级。
```
低  ------------------------------------>  高
byte,short,char—> int —> long—> float —> double
```
数据类型转换必须满足如下规则：

1. 不能对boolean类型进行类型转换。

2. 不能把对象类型转换成不相关类的对象。

3. 在把容量大的类型转换为容量小的类型时必须使用强制类型转换。

4. 转换过程中可能导致溢出或损失精度，例如：
```
int i =128;   
byte b = (byte)i;
```
因为 byte 类型是 8 位，最大值为127，所以当 int 强制转换为 byte 类型时，值 128 时候就会导致溢出。

5. 浮点数到整数的转换是通过舍弃小数得到，而不是四舍五入，例如：
```
(int)23.7 == 23;        
(int)-45.89f == -45
```

**自动类型转换**

必须满足转换前的数据类型的位数要低于转换后的数据类型，例如: short数据类型的位数为16位，就可以自动转换位数为32的int类型，同样float数据类型的位数为32，可以自动转换为64位的double类型。
```(java)
public class ZiDongLeiZhuan{
        public static void main(String[] args){
            char c1='a';//定义一个char类型
            int i1 = c1;//char自动类型转换为int
            System.out.println("char自动类型转换为int后的值等于"+i1);
            char c2 = 'A';//定义一个char类型
            int i2 = c2+1;//char 类型和 int 类型计算
            System.out.println("char类型和int计算后的值等于"+i2);
        }
}

```
执行结果如下：
```（shell）
$ java ZiDongLeiZhuan
char自动类型转换为int后的值等于97
char类型和int计算后的值等于66
```
解析：c1 的值为字符 a ,查 ASCII 码表可知对应的 int 类型值为 97， A 对应值为 65，所以 i2=65+1=66。

**强制类型转换**

1. 条件是转换的数据类型必须是兼容的。
2. 格式：(type)value type是要强制类型转换后的数据类型 实例：
```(java)
public class QiangZhiZhuanHuan{
    public static void main(String[] args){
        int i1 = 123;
        byte b = (byte)i1;//强制类型转换为byte
        System.out.println("int强制类型转换为byte后的值等于"+b);
    }
}
```
运行结果：
```（shell）
$ java QiangZhiZhuanHuan
int强制类型转换为byte后的值等于123
```

**隐含强制类型转换**

1. 整数的默认类型是 int。
2. 浮点型不存在这种情况，因为在定义 float 类型时必须在数字后面跟上 F 或者 f，默认为double类型。


### Java 变量类型
在Java语言中，所有的变量在使用前必须声明。声明变量的基本格式如下：
```
type identifier [ = value][, identifier [= value] ...] ;
```
格式说明：type为Java数据类型。identifier是变量名。可以使用逗号隔开来声明多个同类型变量。

以下列出了一些变量的声明实例。注意有些包含了初始化过程。
```（java）
int a, b, c;         // 声明三个int型整数：a、 b、c
int d = 3, e = 4, f = 5; // 声明三个整数并赋予初值
byte z = 22;         // 声明并初始化 z
String s = "runoob";  // 声明并初始化字符串duix s
double pi = 3.14159; // 声明了双精度浮点型变量 pi
char x = 'x';        // 声明变量 x 的值是字符 'x'。
```
Java 中主要有如下几种类型的变量
```
局部变量 ：函数中
类变量（静态变量）：使用static标识的变量
成员变量（非静态变量） ：类成员变量
```

示例：
```（java）
public class Variable{
    static int allClicks=0;    // 类变量
    String str="hello world";  // 实例变量
    public void method(){
        int i =0;  // 局部变量
    }
}
```

#### Java局部变量

局部变量声明在方法、构造方法或者语句块中；

局部变量在方法、构造方法、或者语句块被执行的时候创建，当它们执行完成后，变量将会被销毁；

访问修饰符不能用于局部变量；

局部变量只在声明它的方法、构造方法或者语句块中可见；

局部变量是在栈上分配的。

局部变量没有默认值，所以局部变量被声明后，必须经过初始化，才可以使用。

示例:
``` java
public class Test{
   public void pupAge(){
      int age = 0;
      age = age + 7;
      System.out.println("小狗的年龄是: " + age);
   }   
   public static void main(String[] args){
      Test test = new Test();
      test.pupAge();
   }
}
```
结果如下：
```
$ java Test
小狗的年龄是: 7
```

#### 实例变量

实例变量声明在一个类中，但在方法、构造方法和语句块之外；

当一个对象被实例化之后，每个实例变量的值就跟着确定；

实例变量在对象创建的时候创建，在对象被销毁的时候销毁；

实例变量的值应该至少被一个方法、构造方法或者语句块引用，使得外部能够通过这些方式获取实例变量信息；

实例变量可以声明在使用前或者使用后；

访问修饰符可以修饰实例变量；

实例变量对于类中的方法、构造方法或者语句块是可见的。一般情况下应该把实例变量设为私有。通过使用访问修饰符可以使实例变量对子类可见；

实例变量具有默认值。数值型变量的默认值是0，布尔型变量的默认值是false，引用类型变量的默认值是null。变量的值可以在声明时指定，也可以在构造方法中指定；

实例变量可以直接通过变量名访问。但在静态方法以及其他类中，就应该使用完全限定名：ObejectReference.VariableName。

示例：

```(java)

import java.io.*;
public class Employee{
   // 这个实例变量对子类可见
   public String name;
   // 私有变量，仅在该类可见
   private double salary;
   //在构造器中对name赋值
   public Employee (String empName){
      name = empName;
   }
   //设定salary的值
   public void setSalary(double empSal){
      salary = empSal;
   }  
   // 打印信息
   public void printEmp(){
      System.out.println("名字 : " + name );
      System.out.println("薪水 : " + salary);
   }

   public static void main(String[] args){
      Employee empOne = new Employee("pingxin");
      empOne.setSalary(1000);
      empOne.printEmp();
   }
}
```
运行结果：

```（shell）
$ java  Employee
名字 : pingxin
薪水 : 1000.0
```

#### 类变量（静态变量）

类变量也称为静态变量，在类中以 static 关键字声明，但必须在方法之外。

无论一个类创建了多少个对象，类只拥有类变量的一份拷贝。

静态变量除了被声明为常量外很少使用。常量是指声明为public/private，final和static类型的变量。常量初始化后不可改变。

静态变量储存在静态存储区。经常被声明为常量，很少单独使用static声明变量。

静态变量在第一次被访问时创建，在程序结束时销毁。

与实例变量具有相似的可见性。但为了对类的使用者可见，大多数静态变量声明为public类型。

默认值和实例变量相似。数值型变量默认值是0，布尔型默认值是false，引用类型默认值是null。变量的值可以在声明的时候指定，也可以在构造方法中指定。此外，静态变量还可以在静态语句块中初始化。

静态变量可以通过：ClassName.VariableName的方式访问。

类变量被声明为public static final类型时，类变量名称一般建议使用大写字母。如果静态变量不是public和final类型，其命名方式与实例变量以及局部变量的命名方式一致。

```(java)
import java.io.*;
public class Employee {
    //salary是静态的私有变量
    private static double salary;
    // DEPARTMENT是一个常量
    public static final String DEPARTMENT = "开发人员";
    public static void main(String[] args){
    salary = 10000;
        System.out.println(DEPARTMENT+"平均工资:"+salary);
    }
}
```
运行结果：
```（shell）
$ java  Employee
开发人员平均工资:10000.0

```
>注意：如果其他类想要访问该变量，可以这样访问：Employee.DEPARTMENT。


### Java修饰符

像其他语言一样，Java可以使用修饰符来修饰类中方法和属性。主要有两类修饰符：
```
访问控制修饰符 : default, public , protected, private
非访问控制修饰符 : final, abstract, static, synchronized
```
 修饰符用来定义类、方法或者变量，通常放在语句的最前端。我们通过下面的例子来说明：

 ```(java)
 public class className {
    // ...
 }
 private boolean myFlag;
 static final double weeks = 9.5;
 protected static final int BOXWIDTH = 42;
 public static void main(String[] arguments) {
    // 方法体
 }
 ```
#### 访问控制修饰符
Java中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限。
```
default (即缺省，什么也不写）: 在同一包内可见，不使用任何修饰符。使用对象：类、接口、变量、方法。
private : 在同一类内可见。使用对象：变量、方法。 注意：不能修饰类（外部类）
public : 对所有类可见。使用对象：类、接口、变量、方法
protected : 对同一包内的类和所有子类可见。使用对象：变量、方法。 注意：不能修饰类（外部类）。
```

![ZFniS1.png](https://s2.ax1x.com/2019/06/23/ZFniS1.png)

protected 当前类，同包，异包子类可见。

**默认访问修饰符-不使用任何关键字**

使用默认访问修饰符声明的变量和方法，对同一个包内的类是可见的。**接口里的变量都隐式声明为 public static final,而接口里的方法默认情况下访问权限为 public。**

如下例所示，变量和方法的声明可以不使用任何修饰符。
```(java)
String version = "1.5.1";
boolean processOrder() {
   return true;
}
```
**私有访问修饰符-private**

私有访问修饰符是最严格的访问级别，所以被声明为 private 的方法、变量和构造方法只能被所属类访问，并且类和接口不能声明为 private。

声明为私有访问类型的变量只能通过类中公共的 getter 方法被外部类访问。

Private 访问修饰符的使用主要用来隐藏类的实现细节和保护类的数据。

下面的类使用了私有访问修饰符：
```java
public class Logger {
   private String format;
   public String getFormat() {
      return this.format;
   }
   public void setFormat(String format) {
      this.format = format;
   }
}
```
实例中，Logger 类中的 format 变量为私有变量，所以其他类不能直接得到和设置该变量的值。为了使其他类能够操作该变量，定义了两个 public 方法：getFormat() （返回 format的值）和 setFormat(String)（设置 format 的值）

**公有访问修饰符-public**

被声明为 public 的类、方法、构造方法和接口能够被任何其他类访问。

如果几个相互访问的 public 类分布在不同的包中，则需要导入相应 public 类所在的包。

由于类的继承性，类所有的公有方法和变量都能被其子类继承。

以下函数使用了公有访问控制：
```(java)
public static void main(String[] arguments) {
   // ...
}
```
Java 程序的 main() 方法必须设置成公有的，否则，Java 解释器将不能运行该类。

**受保护的访问修饰符-protected**

protected 需要从以下两个点来分析说明：
```
子类与基类在同一包中：被声明为 protected 的变量、方法和构造器能被同一个包中的任何其他类访问；
子类与基类不在同一包中：那么在子类中，子类实例可以访问其从基类继承而来的 protected 方法，而不能访问基类实例的protected方法。
```

protected 可以修饰数据成员，构造方法，方法成员，不能修饰类（内部类除外）。
子类能访问 protected 修饰符声明的方法和变量，这样就能保护不相关的类使用这些方法和变量。

下面的父类使用了 protected 访问修饰符，子类重写了父类的 openSpeaker() 方法。
```(java)
class AudioPlayer {
   protected boolean openSpeaker(Speaker sp) {
      // 实现细节
   }
}
class StreamingAudioPlayer extends AudioPlayer {
   protected boolean openSpeaker(Speaker sp) {
      // 实现细节
   }
}
```
如果把 openSpeaker() 方法声明为 private，那么除了 AudioPlayer 之外的类将不能访问该方法。
如果把 openSpeaker() 声明为 public，那么所有的类都能够访问该方法。
如果我们只想让该方法对其所在类的子类可见，则将该方法声明为 protected。

- 基类的 protected 成员是包内可见的，并且对子类可见；
- 若子类与基类不在同一包中，那么在子类中，子类实例可以访问其从基类继承而来的protected方法，而不能访问基类实例的protected方法。

protected细节查看[此处](http://www.runoob.com/w3cnote/java-protected-keyword-detailed-explanation.html)。

#### 非访问控制修饰符

Java除了提供public、protected等访问修饰符之外，为了实现一些其他的功能，Java 也提供了许多非访问修饰符。
```
static 修饰符，用来修饰类方法和类变量。
final 修饰符，用来修饰类、方法和变量，final 修饰的类不能够被继承，修饰的方法不能被继承类重新定义，修饰的变量为常量，是不可修改的。
abstract 修饰符，用来创建抽象类和抽象方法。
synchronized 和 volatile 修饰符，主要用于线程的编程。
```
**static 修饰符**

*静态变量*：static 关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝。 静态变量也被称为类变量。局部变量不能被声明为 static 变量。

*静态方法*：static 关键字用来声明独立于对象的静态方法。静态方法不能使用类的非静态变量。静态方法从参数列表得到数据，然后计算这些数据。

对类变量和方法的访问可以直接使用 类名.变量名 和 类名.方法名 的方式访问。

如下例所示，static修饰符用来创建类方法和类变量。
```java
public class InstanceCounter {
   private static int numInstances = 0; //计数InstanceCounter的实例化对象数
   protected static int getCount() {
      return numInstances;
   }

   private static void addInstance() {
      numInstances++;
   }

   InstanceCounter() {
      InstanceCounter.addInstance();
   }

   public static void main(String[] arguments) {
      System.out.println("Starting with " +
      InstanceCounter.getCount() + " instances"); // Starting with 0 instances
      for (int i = 0; i < 5; ++i){
         new InstanceCounter();
          }
      System.out.println("Created " +
      InstanceCounter.getCount() + " instances");  // Created 5 instances
   }
}
```

**final 修饰符**

*final 变量*

final 变量能被显式地初始化并且只能初始化一次。被声明为 final 的对象的引用不能指向不同的对象。但是 final 对象里的数据可以被改变。也就是说 final 对象的引用不能改变，但是里面的值可以改变。

final 修饰符通常和 static 修饰符一起使用来创建类常量。
```java
public class Test{
    final int value = 10;
    // 下面是声明常量的实例
    public static final int BOXWIDTH = 6;
    static final String TITLE = "Manager";

    public void changeValue(){
        value = 12;
    }
}
// error: cannot assign a value to final variable value
//        value = 12; //将输出一个错误
//        ^
// 1 error
```
*final 方法*

类中的 final 方法可以被子类继承，但是不能被子类修改。
声明 final 方法的主要目的是防止该方法的内容被修改。
如下所示，使用 final 修饰符声明方法。
```(java)
public class Test{
    public final void testMethod(){
       // 方法体
    }
}
```
*final 类*

final 类不能被继承，没有类能够继承 final 类的任何特性。

```（java）
public final class Test {
    // 类体
}
```

**abstract 修饰符**

*抽象类*

抽象类不能用来实例化对象，声明抽象类的唯一目的是为了将来对该类进行扩充。
一个类不能同时被 abstract 和 final 修饰。如果一个类包含抽象方法，那么该类一定要声明为抽象类，否则将出现编译错误。
抽象类可以包含抽象方法和非抽象方法。
```java
abstract class Test{ // 抽象类
   private double height;
   private String name;
   private Date birth;
   public abstract void hear(); // 抽象方法
   public abstract void eat(); // 抽象方法
}
```
*抽象方法*

抽象方法是一种没有任何实现的方法，该方法的的具体实现由子类提供。
任何继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类。
如果一个类包含若干个抽象方法，那么该类必须声明为抽象类。抽象类可以不包含抽象方法。
抽象方法的声明以分号结尾，例如：public abstract sample();。
```(java)
public abstract class SuperClass{
    abstract void testMethod(); // 抽象方法
}

class SubClass extends SuperClass{
    // 实现抽象方法
    void testMethod(){
        //.........
    }
}
```
 final不能同时和abstract使用，因为abstract是需要被子类继承覆盖的，否则毫无意义,而final 用是禁止继承的，两者相互排斥，所以不能共用。

 static 和 abstract 也是不能连用的，因为 static 是类级别的不能被子类覆盖， 而 abstract 需要被继承实现，两者相互矛盾。


 **synchronized 修饰符**

 synchronized 关键字声明的方法同一时间只能被一个线程访问。synchronized 修饰符可以应用于四个访问修饰符。
 可以用于修饰方法、代码段、变量。

 **volatile 修饰符**

volatile 修饰的**成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值**。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

volatile可以用在任何变量前面，但**不能用于final变量前面**，因为final型的变量是禁止修改的。

一个 volatile 对象引用可能是 null。

**[transient 修饰符](https://blog.csdn.net/u010188178/article/details/83581506)**

序列化的对象包含被 transient 修饰的实例变量时，java 虚拟机(JVM)跳过该特定的变量。

如果用**transient**声明一个实例变量，当对象存储时，它的值不需要维持。换句话来说就是，用**transient**关键字标记的成员变量不参与序列化过程。

Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。当一个对象被序列化的时候，transient型变量的值不包括在序列化的表示中，然而非transient型的变量是被包括进去的。

### Java 运算符

计算机的最基本用途之一就是执行数学运算，作为一门计算机语言，Java也提供了一套丰富的运算符来操纵变量。我们可以把运算符分成以下几组：
- 算术运算符
- 关系运算符
- 位运算符
- 逻辑运算符
- 赋值运算符
- 其他运算符

####  算术运算符

算术运算符用在数学表达式中，它们的作用和在数学中的作用一样。下表列出了所有的算术运算符。

表格中的实例假设整数变量A的值为10，变量B的值为20：
![](https://i.loli.net/2019/04/08/5cab3cde30bc2.png)

示例：
```java
public class Test {
  public static void main(String[] args) {
     int a = 10;
     int b = 20;
     int c = 25;
     int d = 25;
     System.out.println("a + b = " + (a + b) );
     System.out.println("a - b = " + (a - b) );
     System.out.println("a * b = " + (a * b) );
     System.out.println("b / a = " + (b / a) );
     System.out.println("b % a = " + (b % a) );
     System.out.println("c % a = " + (c % a) );
     System.out.println("a++   = " +  (a++) ); //返回原值a=10，改变后a=a+1=11
     System.out.println("a--   = " +  (a--) ); //返回原值a=11，改变后a=a-1=10
     // 查看  d++ 与 ++d 的不同
     System.out.println("d++   = " +  (d++) );//返回原值d=25,改变后d=d+1=26
     System.out.println("++d   = " +  (++d) ); //返回改变后值，原值d=26,改变后值d=d+1=27
  }
}
```
运行结果：
```（shell）
$ java Test
a + b = 30
a - b = -10
a * b = 200
b / a = 2
b % a = 0
c % a = 5
a++   = 10
a--   = 11
d++   = 25
++d   = 27
```

前缀自增自减法(++a,--a): 先进行自增或者自减运算，再进行表达式运算。
后缀自增自减法(a++,a--): 先进行表达式运算，再进行自增或者自减运算


#### 关系运算符
下表为Java支持的关系运算符

表格中的实例整数变量A的值为10，变量B的值为20：

![](https://i.loli.net/2019/04/27/5cc4135c26326.png)


示例：
```java
public class Test {
  public static void main(String[] args) {
     int a = 10;
     int b = 20;
     System.out.println("a == b = " + (a == b) );
     System.out.println("a != b = " + (a != b) );
     System.out.println("a > b = " + (a > b) );
     System.out.println("a < b = " + (a < b) );
     System.out.println("b >= a = " + (b >= a) );
     System.out.println("b <= a = " + (b <= a) );
  }
}
```
运行结果：
```shell
$ java  Test
a == b = false
a != b = true
a > b = false
a < b = true
b >= a = true
b <= a = false
```

#### 位运算符

Java定义了位运算符，应用于整数类型(int)，长整型(long)，短整型(short)，字符型(char)，和字节型(byte)等类型。

位运算符作用在所有的位上，并且按位运算。假设a = 60，b = 13;它们的二进制格式表示将如下

```
A = 0011 1100
B = 0000 1101
-----------------
A&b = 0000 1100
A | B = 0011 1101
A ^ B = 0011 0001
~A= 1100 0011
```
下表列出了位运算符的基本运算,假设整数变量A的值为60和变量B的值为13：

![](https://i.loli.net/2019/04/08/5cab40baeda98.png)

示例：

```java
public class Test {
  public static void main(String[] args) {
     int a = 60; /* 60 = 0011 1100 */
     int b = 13; /* 13 = 0000 1101 */
     int c = 0;
     c = a & b;       /* 12 = 0000 1100 */
     System.out.println("a & b = " + c );
     c = a | b;       /* 61 = 0011 1101 */
     System.out.println("a | b = " + c );
     c = a ^ b;       /* 49 = 0011 0001 */
     System.out.println("a ^ b = " + c );
     c = ~a;          /*-61 = 1100 0011 */
     System.out.println("~a = " + c );
     c = a << 2;     /* 240 = 1111 0000 */
     System.out.println("a << 2 = " + c );
     c = a >> 2;     /* 15 = 1111 */
     System.out.println("a >> 2  = " + c );
     c = a >>> 2;     /* 15 = 0000 1111 */
     System.out.println("a >>> 2 = " + c );
  }
}
```
运行结果：
```shell
$ java  Test
a & b = 12
a | b = 61
a ^ b = 49
~a = -61
a << 2 = 240
a >> 2  = 15
a >>> 2 = 15
```

#### 逻辑运算符

下表列出了逻辑运算符的基本运算，假设布尔变量A为真，变量B为假

![](https://i.loli.net/2019/04/08/5cab428e4f2d0.png)

示例：
```java

public class Test {
  public static void main(String[] args) {
     boolean a = true;
     boolean b = false;
     System.out.println("a && b = " + (a&&b));
     System.out.println("a || b = " + (a||b) );
     System.out.println("!(a && b) = " + !(a && b));
  }
}
```
运行结果：
```
$ java  Test
a && b = false
a || b = true
!(a && b) = true
```
逻辑运算符的优先级为：**！运算级别最高，&& 运算高于 || 运算。！运算符的优先级高于算术运算符，而 && 和 || 运算则低于关系运算符。结合方向是：逻辑非（单目运算符）具有右结合性，逻辑与和逻辑或（双目运算符）具有左结合性。**

短路逻辑运算符：**&&和&都表示与，&&表示第一个条件为false时，后面的条件就不执行，&要对所有的条件都进行判断。||和|都表示或，||表示第一个条件为true时，后面的条件都不判断；| 对所有的条件进行判断。**


#### 赋值运算符

![](https://i.loli.net/2019/04/27/5cc413f48ffc0.png)

示例：

```java
public class Test {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        int c = 0;
        c = a + b;
        System.out.println("c = a + b = " + c );
        c += a ;
        System.out.println("c += a  = " + c );
        c -= a ;
        System.out.println("c -= a = " + c );
        c *= a ;
        System.out.println("c *= a = " + c );
        a = 10;
        c = 15;
        c /= a ;
        System.out.println("c /= a = " + c );
        a = 10;
        c = 15;
        c %= a ;
        System.out.println("c %= a  = " + c );
        c <<= 2 ;
        System.out.println("c <<= 2 = " + c );
        c >>= 2 ;
        System.out.println("c >>= 2 = " + c );
        c >>= 2 ;
        System.out.println("c >>= 2 = " + c );
        c &= a ;
        System.out.println("c &= a  = " + c );
        c ^= a ;
        System.out.println("c ^= a   = " + c );
        c |= a ;
        System.out.println("c |= a   = " + c );
    }
}
```
运行结果：
```
$ java  Test
c = a + b = 30
c += a  = 40
c -= a = 30
c *= a = 300
c /= a = 1
c %= a  = 5
c <<= 2 = 20
c >>= 2 = 5
c >>= 2 = 1
c &= a  = 0
c ^= a   = 10
c |= a   = 10
```

#### 条件运算符（?:）

条件运算符也被称为三元运算符。该运算符有3个操作数，并且需要判断布尔表达式的值。该运算符的主要是决定哪个值应该赋值给变量。
```
variable x = (expression) ? value if true : value if false
```
示例：
```java
public class Test {
   public static void main(String[] args){
      int a , b;
      a = 10;
      // 如果 a 等于 1 成立，则设置 b 为 20，否则为 30
      b = (a == 1) ? 20 : 30;
      System.out.println( "Value of b is : " +  b );

      // 如果 a 等于 10 成立，则设置 b 为 20，否则为 30
      b = (a == 10) ? 20 : 30;
      System.out.println( "Value of b is : " + b );
   }
}
```
运行结果：
```
$ java  Test
Value of b is : 30
Value of b is : 20
```
#### instanceof 运算符
该运算符用于操作对象实例，检查该对象是否是一个特定类型（类类型或接口类型）。
instanceof运算符使用格式如下：

```
( Object reference variable ) instanceof  (class/interface type)
```
如果运算符左侧变量所指的对象，是操作符右侧类或接口(class/interface)的一个对象，那么结果为真。
如果被比较的对象兼容于右侧类型,该运算符仍然返回true。

示例：
```java
class Vehicle {}
 public class Car extends Vehicle {
   public static void main(String[] args){
      Vehicle a = new Car();
      boolean result =  a instanceof Car;
      System.out.println( result);
   }
}
```

#### Java运算符优先级
当多个运算符出现在一个表达式中，谁先谁后呢？这就涉及到运算符的优先级别的问题。在一个多运算符的表达式中，运算符优先级不同会导致最后得出的结果差别甚大。

例如，`（1+3）＋（3+2）*2`，这个表达式如果按加号最优先计算，答案就是 18，如果按照乘号最优先，答案则是 14。
再如，x = 7 + 3 * 2;这里x得到13，而不是20，因为乘法运算符比加法运算符有较高的优先级，所以先计算3 * 2得到6，然后再加7。
下表中具有最高优先级的运算符在的表的最上面，最低优先级的在表的底部。

![](https://i.loli.net/2019/04/08/5cab46a8b3be8.png)


### Java注释

类似于 C/C++、Java 也支持单行以及多行注释。注释中的字符将被 Java 编译器忽略。空白行或者有注释的行，Java 编译器都会忽略掉。

```java

public class HelloWorld {
   /* 这是第一个Java程序
    *它将打印Hello World
    * 这是一个多行注释的示例
    */
    public static void main(String []args){
       // 这是单行注释的示例
       /* 这个也是单行注释的示例 */
       System.out.println("Hello World");
    }
}
```

### 面试题

1. 面向对象的特征有哪些方面？

   抽象：将同类对象的共同特征提取出来构造类。
   继承：基于基类创建新类。
   封装：将数据隐藏起来，对数据的访问只能通过特定接口。
   多态性：不同子类型对象对相同消息作出不同响应。

2. float f=3.4;是否正确？

   答:不正确。3.4是双精度数，将双精度型（double）赋值给浮点型（float）属于下转型（down-casting，也称为窄化）会造成精度损失，因此需要强制类型转换float f =(float)3.4; 或者写成float f =3.4F;

3. `short s1 = 1; s1 = s1 + 1;`有错吗?`short s1 = 1; s1 += 1;`有错吗？

   答：对于`short s1 = 1; s1 = s1 + 1;`由于1是int类型，因此s1+1运算结果也是int 型，需要强制转换类型才能赋值给short型。而`short s1 = 1; s1 += 1;`可以正确编译，因为s1+= 1;相当于s1 = (short)(s1 + 1);其中有隐含的强制类型转换。

4. Java有没有goto？

   答：goto 是Java中的保留字，在目前版本的Java中没有使用。（根据James Gosling（Java之父）编写的《The Java Programming Language》一书的附录中给出了一个Java关键字列表，其中有goto和const，但是这两个是目前无法使用的关键字，因此有些地方将其称之为保留字，其实保留字这个词应该有更广泛的意义，因为熟悉C语言的程序员都知道，在系统类库中使用过的有特殊意义的单词或单词的组合都被视为保留字）

5. &和&&的区别？

   答：&运算符有两种用法：(1)按位与；(2)逻辑与。

   &&运算符是短路与运算。

   逻辑与跟短路与的差别是非常巨大的，虽然二者都要求运算符左右两端的布尔值都是true整个表达式的值才是true。

   &&之所以称为短路运算是因为，如果&&左边的表达式的值是false，右边的表达式会被直接短路掉，不会进行运算。

   很多时候我们可能都需要用&&而不是&，例如在验证用户登录时判定用户名不是null而且不是空字符串，应当写为：`username != null &&!username.equals("")`，二者的顺序不能交换，更不能用&运算符，因为第一个条件如果不成立，根本不能进行字符串的equals比较，否则会产生NullPointerException异常。

   注意：逻辑或运算符（|）和短路或运算符（||）的差别也是如此。
   
6. 用最有效率的方法计算2乘以8？

   答： 2 << 3（左移3位相当于乘以2的3次方，右移3位相当于除以2的3次方）。

7. char 型变量中能不能存贮一个中文汉字，为什么？

   答：char类型可以存储一个中文汉字，因为Java中使用的编码是Unicode（不选择任何特定的编码，直接使用字符在字符集中的编号，这是统一的唯一方法），一个char类型占2个字节（16比特），所以放一个中文是没问题的。

   补充：使用Unicode意味着字符在JVM内部和外部有不同的表现形式，在JVM内部都是Unicode，当这个字符被从JVM内部转移到外部时（例如存入文件系统中），需要进行编码转换。所以Java中有字节流和字符流，以及在字符流和字节流之间进行转换的转换流，如InputStreamReader和OutputStreamReader，这两个类是字节流和字符流之间的适配器类，承担了编码转换的任务；对于C程序员来说，要完成这样的编码转换恐怕要依赖于union（联合体/共用体）共享内存的特征来实现了。

8. java变量是否需要初始化

   1. 成员变量：无需初始化。

      成员变量无需初始化，系统在创建实例的过程中默认初始化。

      补充：以final修饰的成员变量。必须显性的初始化赋值。

   2. 方法里面的形参变量： 无需初始化。

      Java类方法，属于按值传递机制，调用方法的时候，完成参数的传递，相当形参被初始化。

   3. 局部变量：必须初始化。

      与成员变量不同，局部变量必须显示初始化，才能够使用。
   
9. 阐述静态变量和实例变量的区别。

   答：静态变量是被static修饰符修饰的变量，也称为类变量，它属于类，不属于类的任何一个对象，一个类不管创建多少个对象，静态变量在内存中有且仅有一个拷贝；实例变量必须依存于某一实例，需要先创建对象然后通过对象才能访问到它。静态变量可以实现让多个对象共享内存。

    补充：在Java开发中，上下文类和工具类中通常会有大量的静态成员。

10. 是否可以从一个静态（static）方法内部发出对非静态（non-static）方法的调用？

    答：不可以，静态方法只能访问静态成员，因为非静态方法的调用要先创建对象，在调用静态方法时可能对象并没有被初始化。

11. 一个".java"源文件中是否可以包含多个类（不是内部类）？有什么限制？

    答：可以，但一个源文件中最多只能有一个公开类（public class）而且文件名必须和公开类的类名完全保持一致。

    