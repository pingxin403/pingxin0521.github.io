---
title: Java 类和对象
date: 2019-04-04 12:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 前言


面向对象编程是当今主流的程序设计思想，已经取代了过程化程序开发技术，Java 是完全面向对象编程语言，所以必须熟悉面向对象才能够编写 Java 程序。

面向对象的程序核心是由对象组成的，每个对象包含着对用户公开的特定功能和隐藏的实现部分。程序中的很多对象来自 JDK 标准库，而更多的类需要我们程序员自定义。

从理论上讲，只要对象能够实现业务功能，其具体的实现细节不必特别关心。

面向对象有以下特点：

（1）面向对象是一种常见的思想，比较符合人们的思考习惯；

（2）面向对象可以将复杂的业务逻辑简单化，增强代码复用性；

（3）面向对象具有抽象、封装、继承、多态等特性。

面向对象的编程语言主要有：C++、Java、C#等。

<!--more-->


面向对象程序设计就是通过对象来进行程序设计，对象表示一个可以明确标识的实体。例如：一个人、一本书、一个学校或一台电脑等等。每个对象都有自己独特的标识、状态和行为。

对象的状态（特征或属性，即实例变量），由该对象的数据域来表示。 例如：一个人可以具有名字、年龄、身高、体重、家庭地址等等属性，这些就是“人这个对象的数据域”。

对象的行为（对象执行的动作，即功能），由方法来定义。例如：定义getName()来获取姓名， getHeight()获取身高，setAddress(String addr)修改地址。


**类是一种抽象的概念集合，是最基础的组织单位，作为对象的模板、合约或蓝图。**

类是对象的类型，使用一个通用类可以定义同一类型的对象，类中定义对象的数据域是什么以及方法是做什么的。 对象是类的实例，一个类可以拥有多个实例，创建实例的过程叫做实例化。实例也称为对象，两者说法一致。

![](https://i.loli.net/2019/04/09/5caca396d519c.png)

类是同一类事物的统称，如：鸟类、人类等。类是构造对象时所依赖的规范，具有相同特性和行为的一类事物称为类，对象就是符合某个类定义所产生出来的实例。类是封装对象的属性和行为的载体，反过来说，具有相同属性和行为的一类实体被称为类。

在Java中，类中对象的行为是以方法的形式定义的，对象的属性是以成员变量的形式定义的，而类的包括对象的属性和方法。

**类：** 对某类事物的普遍一致性特征、功能的抽象、描述和封装，是构造对象的模版或蓝图，用 Java 编写的代码都会在某些类的内部。类之间主要有：依赖、聚合、继承等关系。


**对象：** 使用 new 关键字或反射技术创建的某个类的实例。同一个类的所有对象，都具有相似的数据（比如人的年龄、性别）和行为（比如人的吃饭、睡觉），但是每个对象都保存着自己独特的状态，对象状态会随着程序的运行而发生改变，需要注意状态的变化必须通过调用方法来改变，这就是封装的基本原则。

### 面向对象思想

Java 是面向对象的高级编程语言，类和对象是 Java 程序的构成核心。围绕着 Java 类和 Java 对象，有三大基本特性：

**封装是 Java 类的编写规范**

**继承是类与类之间联系的一种形式**

**而多态为系统组件或模块之间解耦提供了解决方案。**

#### 封装

核心思想就是“隐藏细节”、“数据安全”：将对象不需要让外界访问的成员变量和方法私有化，只提供符合开发者意愿的公有方法来访问这些数据和逻辑，保证了数据的安全和程序的稳定。

![](https://i.loli.net/2019/04/09/5cacb4c29bf16.png)

具体的实现方式就是：

使用 private 修饰符把成员变量设置为私有，防止外部程序直接随意调用或修改成员变量，然后对外提供 public 的 set 和 get 方法按照开发者的意愿（可以编写一些业务逻辑代码，虽然很少这样做）设置和获取成员变量的值。

也可以把只在本类内部使用的方法使用 private，这就是封装的思想，是面向对象最基本的开发规范之一。

在此，我们有必要说一下 Java 的访问权限修饰关键字。Java 中主要有 private、protected、public 和 默认访问权限 四种：

```
public 修饰符，具有最大的访问权限，可以访问任何一个在 CLASSPATH 下的类、接口、异常等。
protected 修饰符，主要作用就是用来保护子类，子类可以访问这些成员变量和方法，其余类不可以。
default 修饰符，主要是本包的类可以访问。
private 修饰符，访问权限仅限于本类内部，在实际开发过程中，大多数的成员变量和方法都是使用 private 修饰的。
```

Java 的访问控制是停留在编译层的，只在编译时进行访问权限检查，不会在类文件中留下痕迹。*通过反射机制，还是可以访问类的私有成员的。*

示例：
```java
public class MobilePhone {
    // 使用 private 关键字把成员变量私有化
    private String os;
    private String phoneNumber;
    private String brand;
    private double dumpEnergy;
    // 对外提供访问、设置成员变量的 public 方法
    // 这样就可以按照我们自己的意愿来访问、设置成员变量
    // 而且也有助于在方法内部对数据有效性进行验证
    public void setOs(String os){
        this.os = os;
    }
    public String getOs(){
        return this.os;
    }
    public void setPhoneNumber(String phoneNumber){
        this.phoneNumber = phoneNumber;
    }
    public String getPhoneNumber(){
        return this.phoneNumber;
    }
    public void setBrand(String brand){
        this.brand = brand;
    }
    public String getBrand(){
        return this.brand;
    }
    public void setDumpEnergy(double dumpEnergy){
        this.dumpEnergy = dumpEnergy;
    }
    public double getDumpEnergy(){
        return this.dumpEnergy;
    }
    // 发短信的方法，不需要做修改
    public void sendMessage(String message, String targetPhoneNumber){
        System.out.println("发给" + targetPhoneNumber + ", 内容是：" + message);
    }
    // 充电方法，不需要做修改
    public double charge(){
        System.out.println("正在充电, 剩余电量：" + dumpEnergy * 100 + "%");
        return dumpEnergy;
    }
    // 对外提供的开机方法
    public void startup(){app
        System.out.println("正在开机......");
        // 调用私有开机方法
        startup2();
        System.out.println("完成开机");
    }
    // 私有的开机方法，封装开机细节
    private void startup2(){
        System.out.println("启动操作系统......");
        System.out.println("加载开机启动项......");
        System.out.println("......");
    }
}
```

在实际的开发过程中，这样的封装方式已经成了 Java Bean 代码编写的规范。现在主流的框架在使用反射技术为对象赋值、取值时使用的都是 set 和 get 方法，而不是直接操作字段的值。

#### 继承

（1）在多个不同的类中抽取出共性的数据和逻辑，对这些共性的内容进行封装一个新的类即父类（也叫做超类或基类），让之前的类来继承这个类，那些共性的内容在子类中就不必重复定义，比如 BaseDAO、BaseAction 等。

（2）Java 的继承机制是单继承，即一个类只能有一个直接父类，如：`public class Husband extends Person{}`。

（3）如果子类和父类有同名成员变量和方法，子类可以使用 super 关键字调用父类的成员变量和方法，上述使用方式前提是成员在子类可见。

 （4）在调用子类构造方法时，会隐式的调用父类的构造方法 super()。如果父类没有无参构造方法，为了避免编译错误，需要在子类构造方法中显式的调用父类的含参构造方法。

（5）子类创建时调用父类构造方法：子类需要使用父类的成员变量和方法，所以就要调用父类构造方法来初始化，之后再进行子类成员变量和方法的初始化。因此，构造方法是无法覆盖的。

 （6）当子类需要扩展父类的某个方法时，可以覆盖父类方法，但是子类方法访问权限必须大于或等于父类权限。

（7）继承提高了程序的复用性、扩展性，也是 Java 语言多态特征的前提。

（8）在实际开发、程序设计过程中，并非先有的父类，而是先有了子类中通用的数据和逻辑，然后再抽取封装出来的父类。

图形类层次结构：

![](https://i.loli.net/2019/04/09/5cacb51690b78.png)


#### 多态

多态指允许不同类的对象对同一“消息”做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。可以用于消除类型之间的耦合关系。

（1）Java 中可以使用父类、接口变量引用子类、实现类对象；

（2）在这个过程中，会对子类、实现类对象做自动类型提升，其特有功能就无法访问了，如果需要使用，可以做强制类型转换。

### 类

**引用变量访问对象和调用数据域和方法**

对象是通过对象类型变量来访问，该变量包含了对对象的引用。对象类型变量使用操作符（.）来 访问对象数据和方法。

创建的对象会在内存中分配空间， 然后通过引用变量来访问。 包含一个引用地址的变量就是引用变量， 即引用类型。 Java中，除了基本类型外，就是引用类型（对象）。

声明对象类型变量的两种形式
```
Fruit f ; //只声明，未指向一个引用地址
Fruit f = null; //f指向一个空地址
//两者基本无区别，null是引用类型的默认值
```

本质上来看，类是一种自定义类型，是一种引用类型，所以该类类型的变量可以引用该类的一个实例。

```
Fruit f = new Fruit("西瓜", "“甜味”)
//表示创建一个Fruit对象，并返回该对象的引用，赋给Fruit类型的f变量。  变量f包含了一个Fruit对象的引用地址。 但通常情况下，直接称变量f为Fruit对象。
```
**引用类型变量和基本类型变量的区别**

每一个变量都代表一个存储值的内存位置。 声明变量时，就是告知编译器该变量可以存储什么类型的值。**对基本类型变量来说，对应内存所存储的值就是基本类型值。而对于引用类型变量来说，对应内存所存储的值是一个引用，指向对象在内存中的位置。**

除了基本类型，就是引用类型，引用类型包含对象引用，可以将引用类型看作对象。

```
int a = 6;
int b = a; //将a的实际值赋给b        
TestReferance tr = new TestReferance();
TestReferance t2 = tr; //将tr的实际值(对TestReferance对象的引用)赋给t2 , tr和t2指向同一对象
t2.a = 10;
System.out.println(tr.a); // 10
```
所以，将引用类型变量赋值给另一个同类型引用变量，两者会指向同一个对象，而不是独立的对象。 如果想要指向一个具有同样内容，但不是同一个对象，可以使用clone()方法。

【技巧】：如果不再需要某个对象时，也就是不引用该对象，可以将引用类型变量赋值null，表示引用为空。 若创建的对象没有被任何变量所引用，JVM会自动回收它所占的空间。

```
TestReferance t1 = new TestReferance();
t1 = new TestReferance(); //t1指向一个新的对象。 t1原来指向的对象会被回收
//创建一个匿名对象，执行完构造方法后，就会被回收。
new TestReferance();
```
PS：关于NullPointerException异常，一般都是因为操作的引用类型变量指向null，所以对引用类型变量操作时，最好先判断一下是否为null。

**静态与实例的区别**

上面的类中定义的都是实例变量和实例方法，这些数据域和方法属于类的某个特定实例，只有创建该类的实例后，才可以访问对象的数据域和方法。

静态变量和方法属于类本身，静态变量被类中的所有对象共享，在静态方法中不能直接访问实例变量和调用实例方法。
```java
public class Test {
    public int a = 5; //实例变量
    public static int staB = 10; //静态变量
    public Test(int a) { this.a = a; }
    public static void staMethod() {
        System.out.println(a); //error, 不允许直接访问实例变量
        insMethod(); //不允许直接调用实例方法
        //通过对象来调用
        System.out.println(new Test(5).a);
        new Test(5).insMethod();
    }
    //实例方法中,可直接访问静态变量和调用静态方法
    public void insMethod() {
        staMethod();
        System.out.println(staB);
    }
}
```
因为静态变量将变量值存储在一个公共地址，被该类的所有对象共享，当某个对象对其修改时，会影响到其他对象。而实例的实例变量则是存储在不同的内存位置中，不会相互影响。

访问静态变量和静态方法时，可以不用创建对象，通过“类名.静态变量/静态方法”来访问调用。 虽然能通过对象来访问静态变量和方法，但为了可读性，方便分辨静态变量，应该通过类名来调用。

【技巧】：若想要某个数据被所有对象共享，就可以使用static修饰，例如常量，修饰常量使用public static final。

【技巧】：如果某个变量或方法依赖于类的某个实例，则应该定义成实例变量或实例方法。若某个变量或方法不依赖于类的某个实例，则应该定义成静态变量和静态方法。 例如Math类，只有静态方法和静态变量，禁止创建Math对象。


**不可变对象和类**

 通过定不可变类来产生不可变对象，不可变对象的内容不能被改变。就像文件中的“只读”概念。

通常情况下，创建一个对象后，该对象的内容可以允许之后修改。但有时候我们需要一个一旦创建，其内容就不能改变的对象。

定义不可变类的要素：

类中所有数据域都是私有的，并且没有提供任何一个数据域的setXxx()方法。若数据域是可变的引用类型，不提供返回该引用类型变量的getXxx()方法。
```
public class Test {
    private int a = 5;
    private String str; //String是不可变对象，它的更改值的方法都是返回一个新的String对象
    private Date date;
    public Test(int a, String str, Date date) {
        super();
        this.a = a;
        this.str = str;
        this.date = date;
    }
    public static void main(String[] args) {
        Test t = new Test(5, "ABC", new Date());
        System.out.println(t.toString());
        //获取引用类型的数据域，其中Date是可变的
        String s = t.getStr();
        Date d = t.getDate();
        s.toLowerCase();
        //可变数据域
        d.setDate(541313);
        System.out.println(t.toString());
    }
    public int getA() {
        return a;
    }
    public String getStr() {
        return str;
    }
    }
```
从以上案例看出，如果数据域是一个可变的引用类型，那么不要返回该数据域，不然对返回的引用变量进行操作，会导致对象内容改变。

**this关键字的使用**

关键字this表示当前对象，引用对象自身。 可以用于访问实例的数据域， 尤其是实例变量和局部变量同名时，进行分辨。除此之外，可以在构造方法内部调用同一个类的其他构造方法。

```java
public class  TestThis{
    public int a;
    public String str;
    //构造方法1
       public TestThis(int a, int b) {
           this.a = a;  
           this.b = b;
       }
    //构造方法2
public TestThis(int a, int b, int c) {
    this(a, b); //调用构造方法1来简化初始化操作
    this.c = c;
}
  }
```

形参名和全局变量同名，但形参是局部变量，所以在方法中优先使用局部变量，这里的赋值是赋值给形参了。解决这个问题，可以修改形参名，但形参名和要初始化的变量名不相等容易引起歧义。

super 代表父类，所有的类的顶级父类就是java.lang.Object，可以使用`super.<变量名\方法名>`访问父类的protected或public的变量或方法。

**引用参数传递给方法**

方法是一种功能集合，表明可以做什么，封装了实现功能的代码，要实现某个功能只需要调用相关方法即可，无需关注功能实现细节。大部分方法都需要传递参数来实现相关功能，但是传递基本类型和传递引用类型有什么区别呢？

给方法传递一个对象参数，实际上是将对象的引用传递给方法。

```java
public class TestThis {
    public static void main(String[] args) {
        int[] arr = {1, 2};
        swap(arr[0], arr[1]);
        System.out.println(Arrays.toString(arr)); //[1, 2]
        swap(arr);
        System.out.println(Arrays.toString(arr)); //[2, 1]  
    }
    public static void swap(int a, int b) {
        int temp = a;
        a = b;
        b = temp;
    }
    public static void swap(int[] a) {
        int temp = a[0];
        a[0] = a[1];
        a[1] = temp;
    }
}
```
第一个swap()方法，接收基本类型参数，所以通过访问下标获取数组元素传递给形参，相当于赋一个实际值给形参，所以对形参交换，不会影响实际参数。

第二个swap()接收一个int类型数组， 数组也是一个对象， 所以此时形参a包含一个该数组的引用，在方法内部对a数组操作，会影响实际参数 arr。

上面方法中使用到了方法重载，方法重载就是使用同样的名字，但根据方法签名来定义多个方法。
方法重载只与方法签名有关，和修饰符以及返回类型无关。被重载的方法必须有不同的参数列表或者参数类型不同。

参数列表不同必然重载，若参数列表相同，但类型相近时，会引发匹配歧义，因为类型不明确。

```
//因为double类型可以匹配int类型，所以会导致编译器匹配歧义
     public static void sum(int a, double b) { }
     public static void sum(double a, int b) { }
```

#### 抽象类
在面向对象的概念中，所有的对象都是通过类来描绘的，但是反过来，并不是所有的类都是用来描绘对象的，如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。

抽象类除了不能实例化对象之外，类的其它功能依然存在，成员变量、成员方法和构造方法的访问方式和普通类一样。

由于抽象类不能实例化对象，所以抽象类必须被继承，才能被使用。也是因为这个原因，通常在设计阶段决定要不要设计抽象类。

父类包含了子类集合的常见的方法，但是由于父类本身是抽象的，所以不能使用这些方法。

在Java中抽象类表示的是一种继承关系，一个类只能继承一个抽象类，而一个类却可以实现多个接口。

```java
/* 文件名 : Employee.java */
public abstract class Employee
{
   private String name;
   private String address;
   private int number;
   public Employee(String name, String address, int number)
   {
      System.out.println("Constructing an Employee");
      this.name = name;
      this.address = address;
      this.number = number;
   }
   public double computePay()
   {
     System.out.println("Inside Employee computePay");
     return 0.0;
   }
   public void mailCheck()
   {
      System.out.println("Mailing a check to " + this.name
       + " " + this.address);
   }
   public String toString()
   {
      return name + " " + address + " " + number;
   }
   public String getName()
   {
      return name;
   }
   public String getAddress()
   {
      return address;
   }
   public void setAddress(String newAddress)
   {
      address = newAddress;
   }
   public int getNumber()
   {
     return number;
   }
}
```
如果你想设计这样一个类，该类包含一个特别的成员方法，该方法的具体实现由它的子类确定，那么你可以在父类中声明该方法为抽象方法。

Abstract 关键字同样可以用来声明抽象方法，抽象方法只包含一个方法名，而没有方法体。

抽象方法没有定义，方法名后面直接跟一个分号，而不是花括号。

声明抽象方法会造成以下两个结果：

- 如果一个类包含抽象方法，那么该类必须是抽象类。
- 任何子类必须重写父类的抽象方法，或者声明自身为抽象类。

继承抽象方法的子类必须重写该方法。否则，该子类也必须声明为抽象类。最终，必须有子类实现该抽象方法，否则，从最初的父类到最终的子类都不能用来实例化对象。
![](https://i.loli.net/2019/04/11/5caeb1f11a2ac.png)

**规定**

1. 抽象类不能被实例化(初学者很容易犯的错)，如果被实例化，就会报错，编译无法通过。只有抽象类的非抽象子类可以创建对象。

2. 抽象类中不一定包含抽象方法，但是有抽象方法的类必定是抽象类。

3. 抽象类中的抽象方法只是声明，不包含方法体，就是不给出方法的具体实现也就是方法的具体功能。

4. 构造方法，类方法（用 static 修饰的方法）不能声明为抽象方法。

5. 抽象类的子类必须给出抽象类中的抽象方法的具体实现，除非该子类也是抽象类。

6. abstract不能与final并列修饰同一个类

7. abstract 不能与private、static、final或native并列修饰同一个方法。

 >创建抽象类和抽象方法非常有用,因为他们可以使类的抽象性明确起来,并告诉用户和编译器打算怎样使用他们.
 >抽象类还是有用的重构器,因为它们使我们可以很容易地将公共方法沿着继承层次结构向上移动。（From:Think in java ）

#### 接口

接口（英文：Interface），在JAVA编程语言中是一个抽象类型，是抽象方法的集合，接口通常以interface来声明。一个类通过继承接口的方式，从而来继承接口的抽象方法。

接口是用来建立类与类之间的协议，它所提供的只是一种形式，而没有具体的实现。同时实现该接口的实现类必须要实现该接口的所有方法，通过使用implements关键字，他表示该类在遵循某个或某组特定的接口，同时也表示着“interface只是它的外貌，但是现在需要声明它是如何工作的”。

接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念。类描述对象的属性和方法。接口则包含类要实现的方法。

除非实现接口的类是抽象类，否则该类要定义接口中的所有方法。

接口无法被实例化，但是可以被实现。一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类。另外，在 Java 中，接口类型可用来声明一个变量，他们可以成为一个空指针，或是被绑定在一个以此接口实现的对象。

>接口中定义方法默认为public，可以为public或protected。定义字段默认为 static final类型.

接口与类相似点：
```
一个接口可以有多个方法。
接口文件保存在 .java 结尾的文件中，文件名使用接口名。
接口的字节码文件保存在 .class 结尾的文件中。
接口相应的字节码文件必须在与包名称相匹配的目录结构中。
```

接口与类的区别：
```
接口不能用于实例化对象。
接口没有构造方法。
接口中所有的方法必须是抽象方法。
接口不能包含成员变量，除了 static 和 final 变量。
接口不是被类继承了，而是要被类实现。
接口支持多继承。
```

接口特性
```
接口中每一个方法也是隐式抽象的,接口中的方法会被隐式的指定为 public abstract（只能是 public abstract，其他修饰符都会报错）。
接口中可以含有变量，但是接口中的变量会被隐式的指定为 public static final 变量（并且只能是 public，用 其他修饰会报编译错误）。
接口中的方法是不能在接口中实现的，只能由实现接口的类来实现接口中的方法。
```

抽象类和接口的区别

1. 抽象类中的方法可以有方法体，就是能实现方法的具体功能，但是接口中的方法不行。
2. 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是 public static final 类型的。
3. 接口中不能含有静态代码块以及静态方法(用 static 修饰的方法)，而抽象类是可以有静态代码块和静态方法。
4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

接口的声明
```
[可见度] interface 接口名称 [extends 其他的接口名] {
        // 声明变量
        // 抽象方法
}
```
示例：
```
/* 文件名 : NameOfInterface.java */
import java.lang.*;
//引入包


/* 文件名 : Animal.java */
interface Animal {
   public void eat();
   public void travel();
}

```
接口有以下特性：

1. 接口是隐式抽象的，当声明一个接口的时候，不必使用abstract关键字。

2. 接口中每一个方法也是隐式抽象的，声明时同样不需要abstract关键字。

3. 接口中的方法都是公有的。


**接口的实现**

当类实现接口的时候，类要实现接口中所有的方法。否则，类必须声明为抽象的类。

类使用implements关键字实现接口。在类声明中，Implements关键字放在class声明后面。

实现一个接口的语法，可以使用这个公式：
```
...implements 接口名称[, 其他接口名称, 其他接口名称..., ...] ...
```

重写接口中声明的方法时，需要注意以下规则：
```
    类在实现接口的方法时，不能抛出强制性异常，只能在接口中，或者继承接口的抽象类中抛出该强制性异常。
    类在重写方法时要保持一致的方法名，并且应该保持相同或者相兼容的返回值类型。
    如果实现接口的类是抽象类，那么就没必要实现该接口的方法。
```

在实现接口的时候，也要注意一些规则：
```
一个类可以同时实现多个接口。
一个类只能继承一个类，但是能实现多个接口。
一个接口能继承另一个接口，这和类之间的继承比较相似。
```

**接口的继承**

一个接口能继承另一个接口，和类之间的继承方式比较相似。接口的继承使用extends关键字，子接口继承父接口的方法。

下面的Sports接口被Hockey和Football接口继承：

```(java)
// 文件名: Sports.java
public interface Sports
{
   public void setHomeTeam(String name);
   public void setVisitingTeam(String name);
}

// 文件名: Football.java
public interface Football extends Sports
{
   public void homeTeamScored(int points);
   public void visitingTeamScored(int points);
   public void endOfQuarter(int quarter);
}

// 文件名: Hockey.java
public interface Hockey extends Sports
{
   public void homeGoalScored();
   public void visitingGoalScored();
   public void endOfPeriod(int period);
   public void overtimePeriod(int ot);
}

```
Hockey接口自己声明了四个方法，从Sports接口继承了两个方法，这样，实现Hockey接口的类需要实现六个方法。

相似的，实现Football接口的类需要实现五个方法，其中两个来自于Sports接口。

**标记接口**

最常用的继承接口是没有包含任何方法的接口。

标记接口是没有任何方法和属性的接口.它仅仅表明它的类属于一个特定的类型,供其他代码来测试允许做一些事情。

标记接口作用：简单形象的说就是给某个对象打个标（盖个戳），使对象拥有某个或某些特权。

没有任何方法的接口被称为标记接口。标记接口主要用于以下两种目的：

1. 建立一个公共的父接口：如EventListener接口，这是由几十个其他接口扩展的Java API，你可以使用一个标记接口来建立一组接口的父接口。例如：当一个接口继承了EventListener接口，Java虚拟机(JVM)就知道该接口将要被用于一个事件的代理方案。

2. 向一个类添加数据类型：这种情况是标记接口最初的目的，实现标记接口的类不需要定义任何接口方法(因为标记接口根本就没有方法)，但是该类通过多态性变成一个接口类型。

>ISP（Interface Segregation Principle）：面向对象的一个核心原则。它表明使用多个专门的接口比使用单一的总接口要好。
>一个类对另外一个类的依赖性应当是建立在最小的接口上的。
>一个接口代表一个角色，不应当将不同的角色都交给一个接口。没有关系的接口合并在一起，形成一个臃肿的大接口，这是对角色和接口的污染。

![](https://i.loli.net/2019/04/11/5caeb1f119f24.png)


**总结**

1.  抽象类在java语言中所表示的是一种继承关系，一个子类只能存在一个父类，但是可以存在多个接口。

2.  在抽象类中可以拥有自己的成员变量和非抽象类方法，但是接口中只能存在静态的不可变的成员数据（不过一般都不在接口中定义成员数据），而且它的所有方法都是抽象的。

3. 抽象类和接口所反映的设计理念是不同的，抽象类所代表的是“is-a”的关系，而接口所代表的是“like-a”的关系。

抽象类和接口是java语言中两种不同的抽象概念，他们的存在对多态提供了非常好的支持，虽然他们之间存在很大的相似性。但是对于他们的选择往往反应了您对问题域的理解。只有对问题域的本质有良好的理解，才能做出正确、合理的设计。


#### Object类

Object 是 Java 类库中的一个特殊类，也是所有类的父类。当一个类被定义后，如果没有指定继承的父类，那么默认父类就是 Object 类。因此，以下两个类是等价的。

```
    public class MyClass{…}
    public class MyClass extends Object {…}
```
由于 Java 中的所有类都是由 Object 类派生出来的，因此在 Object 类中定义的方法，在其他类中都可以使用。

![](https://i.loli.net/2019/04/10/5cadf0551b3cc.png)

 其中，equals() 方法和 getClass() 方法在 Java 程序中比较常用。getClass()、notify()、notifyAll()、wait()等方法使用final定义，不能被子类修改。

 equals() 方法的作用与运算符类似，用于值与值的比较和值与对象的比较，而 equals() 方法用于对象与对象之间的比较

 ```
     boolean result=obj.equals(Object o);
 ```
其中，Obj 表示要进行比较的一个对象，o 表示另一个对象。

编写一个 Java 程序，实现用户登录的验证功能。要求，用户从键盘输入登录用户名和密码，当用户输入的用户名等于 admin 并且密码也等于 admin 时，则表示该用户为合法用户，提示登录成功，否则提示用户名或者密码错误信息。

在这里使用 equals() 方法将用户输入的字符串与保存 admin 的字符串对象进行比较，具体的代码如下：

 ```java
 import java.util.Scanner;
 public class Test01
 {
     //验证用户名和密码
     public static boolean validateLogin(String uname,String upwd)
     {
         boolean con=false;
         if(uname.equals("admin")&&upwd.equals("admin"))
         {    //比较两个 String 对象
             con=true;
         }
         else
         {
             con=false;
         }
         return con;
     }
     public static void main(String[] args)
     {
         Scanner input=new Scanner(System.in);
         System.out.println("------欢迎使用大数据管理平台------");
         System.out.println("用户名：");
         String username=input.next();    //获取用户输入的用户名
         System.out.printin("密码：");
         String pwd=input.next();    //获取用户输入的密码
         boolean con=validateLogin(username,pwd);
         if(con)
         {
             System.out.println("登录成功！");
         }
         else
         {
             System.out.println("用户名或密码有误！");
         }
     }
 }
 ```

getClass() 方法返回对象所属的类，是一个 Class 对象。通过 Class 对象可以获取该类的各种信息，包括类名、父类以及它所实现接口的名字等。

编写一个实例，演示如何对 String 类型调用 getClass() 方法，然后输出其父类及实现的接口信息。具体实现代码如下：

 ```java
 public class Test02
{
    public static void printClassinfo(Object obj)
    {
        //获取类名
        System.out.println("类名："+obj.getClass().getName());

        //获取父类名
        System.out.println("父类："+obj.getClass().getSuperclass().getName());
        System.out.println("实现的接口有：");

        //获取实现的接口并输出
        for(int i=0;i<obj.getClass().getInterfaces().length;i++)
        {
            System.out.println(obj.getClass().getInterfaces()[i]);
        }
    }
    public static void main(String[] args)
    {
        String strObj=new String();
        printClassInfo(strObj);
    }
}
 ```
 该程序的运行结果如下：

 ```
 类名：java.lang.String
父类：java.lang.Object
实现的接口有：
interface java.io.Serializable
interface java.lang.Comparable
interface java.lang.CharSequence
 ```

### 面试题

1. Java值传递还是引用传递？

   Java总是采用按值调用。对于基本类型变量，Java是传递值的副本；对于引用类型变量，即所有的对象型变量，Java都是传引用的副本（依然指向同一对象）。

   - 一个方法不能修改一个基本数据类型的参数(即数值型和布尔型)
   - 一个方法可以改变一个对象参数的状态
   - 一个方法不能让对象参数引用一个新的对象

2. 简述执行过程，不考虑继承

   ```java
   public class PartyMember {
   
       /**
        * 非静态变量（实例属性）
        */
       int apple = 8;
       int banana = 9;
   
       /**
        * 构造方法
        */
       public PartyMember(int apple, int banana) {
           this.apple = apple;
           this.banana = banana;
           System.out.println("构造方法中：");
           System.out.println(this.apple + " " + this.banana);
       }
   
       /**
        * 非静态代码块
        */
       {
           this.apple--;
           this.banana--;
           System.out.println("非静态代码块中：");
           System.out.println(this.apple + " " + this.banana);
       }
   
       /**
        * main方法
        */
       public static void main(String[] args) {
           new PartyMember(1, 2);
       }
   }
   ```
   
   如若不考虑静态代码块static{ }中调用构造方法创建对象后，再赋给实例属性值的情况，实例属性（非静态变量）的初始化过程简述为：

   用new创建并初始化对象步骤：

   1. 给对象的实例变量（非“常量”）分配内存空间，默认初始化成员变量；
   2. 成员变量声明时的初始化；
   
   3. 初始化块初始化（又称为构造代码块或非静态代码块）；
   4. 构造方法初始化
   
   即 **默认值 –> 初始值 –> 非静态代码块中的值 –> 构造方法中的值**
   
   参考：[Java变量初始化过程详解](https://blog.csdn.net/HNAKXR/article/details/81464182)
3. 重载（Overload）和重写（Override）的区别。重载的方法能否根据返回类型进行区分？

   答：方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

   重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；

   重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。

   重载对返回类型没有特殊的要求。

4. 华为的面试题中曾经问过这样一个问题 - "为什么不能根据返回类型来区分重载"，快说出你的答案吧！

   **因为调用时不能指定类型信息，编译器不知道你要调用哪个函数。**

   例如：

   ```java
   float max(int a, int b);
   int max(int a, int b);
   ```

   当调用`max(1, 2);`时无法确定调用的是哪个，单从这一点上来说，仅返回值类型不同的重载是不应该允许的。

5. 构造器（constructor）是否可被重写（override）？

   答：构造器不能被继承，因此不能被重写，但可以被重载。

6. 两个对象值相同(x.equals(y) == true)，但却可有不同的hash code，这句话对不对？

   答：不对，如果两个对象x和y满足x.equals(y) == true，它们的哈希码（hash code）应当相同。Java对于eqauls方法和hashCode方法是这样规定的：

   - 如果两个对象相同（equals方法返回true），那么它们的hashCode值一定要相同；
   - 如果两个对象的hashCode相同，它们并不一定相同。当然，你未必要按照要求去做，但是如果你违背了上述原则就会发现在使用容器时，相同的对象可以出现在Set集合中，同时增加新元素的效率会大大下降（对于使用哈希存储的系统，如果哈希码频繁的冲突将会造成存取性能急剧下降）。

   首先equals方法必须满足自反性（x.equals(x)必须返回true）、对称性（x.equals(y)返回true时，y.equals(x)也必须返回true）、传递性（x.equals(y)和y.equals(z)都返回true时，x.equals(z)也必须返回true）和一致性（当x和y引用的对象信息没有被修改时，多次调用x.equals(y)应该得到同样的返回值），而且对于任何非null值的引用x，x.equals(null)必须返回false。

   实现高质量的equals方法的诀窍包括：

   - 使用==操作符检查"参数是否为这个对象的引用"；
   - 使用instanceof操作符检查"参数是否为正确的类型"；
   - 对于类中的关键属性，检查参数传入对象的属性是否与之相匹配
   - 编写完equals方法后，问自己它是否满足对称性、传递性、一致性；
   - 重写equals时总是要重写hashCode；
   - 不要将equals方法参数中的Object对象替换为其他的类型，在重写时不要忘掉@Override注解。

7. 抽象类（abstract class）和接口（interface）有什么异同？

   答：抽象类和接口都不能够实例化，但可以定义抽象类和接口类型的引用。

   一个类如果继承了某个抽象类或者实现了某个接口都需要对其中的抽象方法全部进行实现，否则该类仍然需要被声明为抽象类。

   接口比抽象类更加抽象，因为抽象类中可以定义构造器，可以有抽象方法和具体方法，而接口中不能定义构造器而且其中的方法全部都是抽象方法。

   抽象类中的成员可以是private、默认、protected、public的，而接口中的成员全都是public的。抽象类中可以定义成员变量，而接口中定义的成员变量实际上都是常量。有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法。

8. 抽象的（abstract）方法是否可同时是静态的（static）,是否可同时是本地方法（native），是否可同时被synchronized修饰？

   答：都不能。抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的。本地方法是由本地代码（如C代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。synchronized和方法的实现细节有关，抽象方法不涉及实现细节，因此也是相互矛盾的。

9. 如何实现对象克隆

   答：有两种方式：

   - 实现Cloneable接口并重写Object类中的clone()方法；
   -  实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆

   ```java
   import java.io.ByteArrayInputStream;
   import java.io.ByteArrayOutputStream;
   import java.io.ObjectInputStream;
   import java.io.ObjectOutputStream;
   import java.io.Serializable;
   
   public class MyUtil {
   
       private MyUtil() {
           throw new AssertionError();
       }
   
       @SuppressWarnings("unchecked")
       public static <T extends Serializable> T clone(T obj) throws Exception {
           ByteArrayOutputStream bout = new ByteArrayOutputStream();
           ObjectOutputStream oos = new ObjectOutputStream(bout);
           oos.writeObject(obj);
   
           ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());
           ObjectInputStream ois = new ObjectInputStream(bin);
           return (T) ois.readObject();
   
           // 说明：调用ByteArrayInputStream或ByteArrayOutputStream对象的close方法没有任何意义
           // 这两个基于内存的流只要垃圾回收器清理对象就能够释放资源，这一点不同于对外部资源（如文件流）的释放
       }
   }
   
   import java.io.Serializable;
   
   /**
    * 人类
    * @author nnngu
    *
    */
   class Person implements Serializable {
       private static final long serialVersionUID = -9102017020286042305L;
   
       private String name;    // 姓名
       private int age;        // 年龄
       private Car car;        // 座驾
   
       public Person(String name, int age, Car car) {
           this.name = name;
           this.age = age;
           this.car = car;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public Car getCar() {
           return car;
       }
   
       public void setCar(Car car) {
           this.car = car;
       }
   
       @Override
       public String toString() {
           return "Person [name=" + name + ", age=" + age + ", car=" + car + "]";
       }
   
   }
   
   /**
    * 小汽车类
    * @author nnngu
    *
    */
   class Car implements Serializable {
       private static final long serialVersionUID = -5713945027627603702L;
   
       private String brand;       // 品牌
       private int maxSpeed;       // 最高时速
   
       public Car(String brand, int maxSpeed) {
           this.brand = brand;
           this.maxSpeed = maxSpeed;
       }
   
       public String getBrand() {
           return brand;
       }
   
       public void setBrand(String brand) {
           this.brand = brand;
       }
   
       public int getMaxSpeed() {
           return maxSpeed;
       }
   
       public void setMaxSpeed(int maxSpeed) {
           this.maxSpeed = maxSpeed;
       }
   
       @Override
       public String toString() {
           return "Car [brand=" + brand + ", maxSpeed=" + maxSpeed + "]";
       }
   
   }
   class CloneTest {
   
       public static void main(String[] args) {
           try {
               Person p1 = new Person("郭靖", 33, new Car("Benz", 300));
               Person p2 = MyUtil.clone(p1);   // 深度克隆
               p2.getCar().setBrand("BYD");
               // 修改克隆的Person对象p2关联的汽车对象的品牌属性
               // 原来的Person对象p1关联的汽车不会受到任何影响
               // 因为在克隆Person对象时其关联的汽车对象也被克隆了
               System.out.println(p1);
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   ```

   注意：基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是好过把问题留到运行时。

10. 接口是否可继承（extends）接口？抽象类是否可实现（implements）接口？抽象类是否可继承具体类（concrete class）？

    答：接口可以继承接口，而且支持多重继承。抽象类可以实现(implements)接口，抽象类可继承具体类也可以继承抽象类。

    举一个多继承的例子，我们定义一个动物（类）既是狗（父类1）也是猫（父类2），两个父类都有“叫”这个方法。那么当我们调用“叫”这个方法时，它就不知道是狗叫还是猫叫了，这就是多重继承的冲突。
    而接口没有具体的方法实现，所以多继承接口也不会出现这种冲突。

    

