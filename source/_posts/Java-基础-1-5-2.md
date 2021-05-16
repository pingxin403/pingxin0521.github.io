---
title: Java 面对对象特性和类加载顺序
date: 2019-04-04 17:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---

### 封装

封装从字面上来理解就是包装的意思，专业点就是信息隐藏，是指利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体，数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。系统的其他对象只能通过包裹在数据外面的已经授权的操作来与这个封装的对象进行交流和交互。也就是说用户是无需知道对象内部的细节（当然也无从知道），但可以通过该对象对外的提供的接口来访问该对象。

对于封装而言，一个对象它所封装的是自己的属性和方法，所以它是不需要依赖其他对象就可以完成自己的操作。
<!--more-->
使用封装有四大好处：

1. 良好的封装能够减少耦合。

2. 类内部的结构可以自由修改。

3. 可以对成员进行更精确的控制。

4. 隐藏信息，实现细节。

首先我们先来看两个类：Husband.java、Wife.java

```(java)
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private int age ;
    private Wife wife;

    /*
     * setter()、getter()是该对象对外开发的接口
     */
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }
}

public class Wife {
    private String name;
    private int age;
    private String sex;
    private Husband husband;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }

    public Husband getHusband() {
        return husband;
    }

}
```
从上面两个实例我们可以看出Husband里面wife引用是没有getter()的，同时wife的age也是没有getter()方法的。至于理由我想各位都懂的，男人嘛深屋藏娇妻嘛，没有那个女人愿意别人知道她的年龄。

所以封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果不想被外界方法，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。

比如我们将一个房子看做是一个对象，里面的漂亮的装饰，如沙发、电视剧、空调、茶桌等等都是该房子的私有属性，但是如果我们没有那些墙遮挡，是不是别人就会一览无余呢？没有一点儿隐私！就是存在那个遮挡的墙，我们既能够有自己的隐私而且我们可以随意的更改里面的摆设而不会影响到其他的。但是如果没有门窗，一个包裹的严严实实的黑盒子，又有什么存在的意义呢？所以通过门窗别人也能够看到里面的风景。所以说门窗就是房子对象留给外界访问的接口。

通过这个我们还不能真正体会封装的好处。现在我们从程序的角度来分析封装带来的好处。如果我们不使用封装，那么该对象就没有setter()和getter()，那么Husband类应该这样写：

```（java）
public class Husband {
    public String name ;
    public String sex ;
    public int age ;
    public Wife wife;
}
```
我们应该这样来使用它：
```
Husband husband = new Husband();
husband.age = 30;
husband.name = "张三";
husband.sex = "男";    //貌似有点儿多余
```

但是那天如果我们需要修改Husband，例如将age修改为String类型的呢？你只有一处使用了这个类还好，如果你有几十个甚至上百个这样地方，你是不是要改到崩溃。如果使用了封装，我们完全可以不需要做任何修改，只需要稍微改变下Husband类的setAge()方法即可。

```（java）
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private String age ;    /* 改成 String类型的*/
    private Wife wife;

    public String getAge() {
        return age;
    }  
    public void setAge(int age) {
        //转换即可
        this.age = String.valueOf(age);
    }
    /** 省略其他属性的setter、getter **/    
}
```
其他的地方依然那样引用(husband.setAge(22))保持不变。

到了这里我们确实可以看出，**封装确实可以使我们容易地修改类的内部实现，而无需修改使用了该类的客户代码。**

我们在看这个好处：可以对成员变量进行更精确的控制。

还是那个Husband，一般来说我们在引用这个对象的时候是不容易出错的，但是有时你迷糊了，写成了这样：

```
Husband husband = new Husband();
        husband.age = 300;
```
也许你是因为粗心写成了，你发现了还好，如果没有发现那就麻烦大了，谁见过300岁的老妖怪啊！

但是使用封装我们就可以避免这个问题，我们对age的访问入口做一些控制(setter)如：

```（java）
public class Husband {

    /*
     * 对属性的封装
     * 一个人的姓名、性别、年龄、妻子都是这个人的私有属性
     */
    private String name ;
    private String sex ;
    private int age ;    /* 改成 String类型的*/
    private Wife wife;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(age > 120){
            System.out.println("ERROR：error age input....");    //提示错误信息
        }else{
            this.age = age;
        }    
    }
    /** 省略其他属性的setter、getter **/
}
```
上面都是对setter方法的控制，其实通过使用封装我们也能够对对象的出口做出很好的控制。例如性别我们在数据库中一般都是已1、0方式来存储的，但是在前台我们又不能展示1、0，这里我们只需要在getter()方法里面做一些转换即可。

```（java）
public String getSexName() {
        if("0".equals(sex)){
            sexName = "女";
        }
        else if("1".equals(sex)){
            sexName = "男";
        }
        else{
            sexName = "人妖???";
        }
        return sexName;
    }
```

在使用的时候我们只需要使用sexName即可实现正确的性别显示。同理也可以用于针对不同的状态做出不同的操作。

```
public String getCzHTML(){
        if("1".equals(zt)){
            czHTML = "<a href='javascript:void(0)' onclick='qy("+id+")'>启用</a>";
        }
        else{
            czHTML = "<a href='javascript:void(0)' onclick='jy("+id+")'>禁用</a>";
        }
        return czHTML;
    }
```


参考：http://www.cnblogs.com/chenssy/p/3351835.html

### 继承

 在《Think in java》中有这样一句话：复用代码是Java众多引人注目的功能之一。但要想成为极具革命性的语言，仅仅能够复制代码并对加以改变是不够的，它还必须能够做更多的事情。在这句话中最引人注目的是“复用代码”,尽可能的复用代码使我们程序员一直在追求的，现在我来介绍一种复用代码的方式，也是java三大特性之一---继承。

 >所有的类都继承于java.lang.Object类

在讲解之前我们先看上面的例子。

Wife、Husband两个类除了各自的husband、wife外其余部分全部相同，作为一个想最大限度实现复用代码的我们是不能够忍受这样的重复代码，如果再来一个小三、小四、小五……我们是不是也要这样写呢？那么我们如何来实现这些类的可复用呢？利用继承！！

首先我们先离开软件编程的世界，从常识中我们知道丈夫、妻子、小三、小四…，他们都是人，而且都有一些共性，有名字、年龄、性别、头等等，而且他们都能够吃东西、走路、说话等等共同的行为，所以从这里我们可以发现他们都拥有人的属性和行为，同时也是从人那里继承来的这些属性和行为的。


 从上面我们就可以基本了解了继承的概念了，**继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。** 通过使用继承我们能够非常方便地复用以前的代码，能够大大的提高开发的效率。

```
public class Person{
protected String name;
protected int age;
protected String sex;
  //setter and getter
}

public class Wife extends Person{
private Husband husband;

}

 public class Husband extends Person{
   private Wife wife;
 }

```
对于Wife、Husband使用继承后，除了代码量的减少我们还能够非常明显的看到他们的关系。

继承所描述的是“is-a”的关系，如果有两个对象A和B，若可以描述为“A是B”，则可以表示A继承B，其中B是被继承者称之为父类或者超类，A是继承者称之为子类或者派生类。

实际上继承者是被继承者的特殊化，它除了拥有被继承者的特性外，还拥有自己独有得特性。例如猫有抓老鼠、爬树等其他动物没有的特性。同时在继承关系中，继承者完全可以替换被继承者，反之则不可以，例如我们可以说猫是动物，但不能说动物是猫就是这个道理，其实对于这个我们将其称之为“向上转型”，下面介绍。

诚然，继承定义了类如何相互关联，共享特性。对于若干个相同或者相识的类，我们可以抽象出他们共有的行为或者属相并将其定义成一个父类或者超类，然后用这些类继承该父类，他们不仅可以拥有父类的属性、方法还可以定义自己独有的属性或者方法。

同时在使用继承时需要记住三句话：

1. 子类拥有父类非private的属性和方法。

2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。

3. 子类可以用自己的方式实现父类的方法。

综上所述，使用继承确实有许多的优点，除了将所有子类的共同属性放入父类，实现代码共享，避免重复外，还可以使得修改扩展继承而来的实现比较简单。

诚然，讲到继承一定少不了这三个东西：构造器、protected关键字、向上转型。

#### 构造器

通过前面我们知道子类可以继承父类的属性和方法，除了那些private的外还有一样是子类继承不了的---构造器。对于构造器而言，它只能够被调用，而不能被继承。 调用父类的构造方法我们使用super()即可。

对于子类而已,其构造器的正确初始化是非常重要的,而且当且仅当只有一个方法可以保证这点：在构造器中调用父类构造器来完成初始化，而父类构造器具有执行父类初始化所需要的所有知识和能力。

```(java)
public class Person {
    protected String name;
    protected int age;
    protected String sex;

    Person(){
        System.out.println("Person Constrctor...");
    }
}

public class Husband extends Person{
    private Wife wife;

    Husband(){
        System.out.println("Husband Constructor...");
    }

    public static void main(String[] args) {
        Husband husband  = new Husband();
    }
}

//Output:
//Person Constrctor...
//Husband Constructor...
```

通过这个示例可以看出，构建过程是从父类“向外”扩散的，也就是从父类开始向子类一级一级地完成构建。而且我们并没有显示的引用父类的构造器，这就是java的聪明之处：编译器会默认给子类调用父类的构造器。

但是，这个默认调用父类的构造器是有前提的：父类有默认构造器。如果父类没有默认构造器，我们就要必须显示的使用super()来调用父类构造器，否则编译器会报错：无法找到符合父类形式的构造器。

```(java)
public class Person {
    protected String name;
    protected int age;
    protected String sex;

    Person(String name){
        System.out.println("Person Constrctor-----" + name);
    }
}

public class Husband extends Person{
    private Wife wife;

    Husband(){
        super("pingxin");
        System.out.println("Husband Constructor...");
    }

    public static void main(String[] args) {
        Husband husband  = new Husband();
    }
}

//Output:
//Person Constrctor-----pingxin
//Husband Constructor...
```
所以综上所述：**对于继承而已，子类会默认调用父类的构造器，但是如果没有默认的父类构造器，子类必须要显示的指定父类的构造器，而且必须是在子类构造器中做的第一件事(第一行代码)。**

#### protected关键字

private访问修饰符，对于封装而言，是最好的选择，但这个只是基于理想的世界，有时候我们需要这样的需求：我们需要将某些事物尽可能地对这个世界隐藏，但是仍然允许子类的成员来访问它们。这个时候就需要使用到protected。

**对于protected而言，它指明就类用户而言，他是private，但是对于任何继承与此类的子类而言或者其他任何位于同一个包的类而言，他却是可以访问的**

```
public class Person {
    private String name;
    private int age;
    private String sex;

    protected String getName() {
        return name;
    }

    protected void setName(String name) {
        this.name = name;
    }

    public String toString(){
        return "this name is " + name;
    }

    /** 省略其他setter、getter方法 **/
}

public class Husband extends Person{
    private Wife wife;

    public  String toString(){
        setName("pingxin");    //调用父类的setName();
        return  super.toString();    //调用父类的toString()方法
    }

    public static void main(String[] args) {
        Husband husband = new Husband();

        System.out.println(husband.toString());
    }
}

//Output：
//this name is pinxgin
```
从上面示例可以看书子类Husband可以明显地调用父类Person的setName()。

诚然尽管可以使用protected访问修饰符来限制父类属性和方法的访问权限，但是最好的方式还是 **将属性保持为private(我们应当一致保留更改底层实现)，通过protected方法来控制类的继承者的访问权限。**

#### 向上转型

在上面的继承中我们谈到继承是is-a的相互关系，猫继承与动物，所以我们可以说猫是动物，或者说猫是动物的一种。这样将猫看做动物就是向上转型。如下：

```(java)
public class Person {
    public void display(){
        System.out.println("Play Person...");
    }

    static void display(Person person){
        person.display();
    }
}

public class Husband extends Person{
    public static void main(String[] args) {
        Husband husband = new Husband();
        Person.display(husband);      //向上转型
    }
}
```
在这我们通过Person.display(husband)。这句话可以看出husband是person类型。

将子类转换成父类，在继承关系上面是向上移动的，所以一般称之为向上转型。由于向上转型是从一个叫专用类型向较通用类型转换，所以它总是安全的，唯一发生变化的可能就是属性和方法的丢失。这就是为什么编译器在“未曾明确表示转型”活“未曾指定特殊标记”的情况下，仍然允许向上转型的原因。

**向下转型**

向下转型需要显式声明，如：
```
Person person=new Person()
Husband husband = (Husband)person;
```
不过向下转型一般会出现问题。

可以使用instanceof操作符判断对象类型。结合向下转型，防止出现错误。

```
Husband husband = (Husband)person;
if(husband instanceof  Husband){} //true
if(husband instanceof Person){}//true

Person person=new Person()
if(person instanceof Person){}//true
if(person instanceof Husband) {}//true  
```


#### 谨慎继承

上面讲了继承所带来的诸多好处，那我们是不是就可以大肆地使用继承呢？送你一句话：慎用继承。

首先我们需要明确，继承存在如下缺陷：

1. 父类变，子类就必须变。

2. 继承破坏了封装，对于父类而言，它的实现细节对与子类来说都是透明的。

3. 继承是一种强耦合关系。      

所以说当我们使用继承的时候，我们需要确信使用继承确实是有效可行的办法。那么到底要不要使用继承呢？《Think in java》中提供了解决办法：**问一问自己是否需要从子类向父类进行向上转型。如果必须向上转型，则继承是必要的，但是如果不需要，则应当好好考虑自己是否需要继承**

少用继承，多用组合，继承是is-a关系，组合是has-a关系

参考：https://www.cnblogs.com/chenssy/p/3354884.html

### 多态

封装隐藏了类的内部实现机制，可以在不影响使用的情况下改变类的内部结构，同时也保护了数据。对外界而已它的内部细节是隐藏的，暴露给外界的只是它的访问方法。

继承是为了重用父类代码。两个类若存在IS-A的关系就可以使用继承。同时继承也为实现多态做了铺垫。那么什么是多态呢？多态的实现机制又是什么？请看我一一为你揭开：

所谓 **多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。**

比如你是一个酒神，对酒情有独钟。某日回家发现桌上有几个杯子里面都装了白酒，从外面看我们是不可能知道这是些什么酒，只有喝了之后才能够猜出来是何种酒。你一喝，这是剑南春、再喝这是五粮液、再喝这是酒鬼酒….在这里我们可以描述成如下：
```
酒 a = 剑南春

酒 b = 五粮液

酒 c = 酒鬼酒
…
```

这里所表现的的就是多态。剑南春、五粮液、酒鬼酒都是酒的子类，我们只是通过酒这一个父类就能够引用不同的子类，这就是多态——我们只有在运行的时候才会知道引用变量所指向的具体实例对象。

诚然，要理解多态我们就必须要明白什么是“向上转型”。在继承中我们简单介绍了向上转型，这里就在啰嗦下：在上面的喝酒例子中，酒（Win）是父类，剑南春（JNC）、五粮液（WLY）、酒鬼酒（JGJ）是子类。我们定义如下代码：
```
JNC a = new  JNC();
```
对于这个代码我们非常容易理解无非就是实例化了一个剑南春的对象嘛！但是这样呢？
```
Wine a = new JNC();
```
在这里我们这样理解，这里定义了一个Wine 类型的a，它指向JNC对象实例。由于JNC是继承与Wine，所以JNC可以自动向上转型为Wine，所以a是可以指向JNC实例对象的。这样做存在一个非常大的好处，在继承中我们知道子类是父类的扩展，它可以提供比父类更加强大的功能，如果我们定义了一个指向子类的父类引用类型，那么它除了能够引用父类的共性外，还可以使用子类强大的功能。

但是向上转型存在一些缺憾，那就是它必定会导致一些方法和属性的丢失，而导致我们不能够获取它们。所以父类类型的引用可以调用父类中定义的所有属性和方法，对于只存在与子类中的方法和属性它就望尘莫及了。

```(java)
public class Wine {
    public void fun1(){
        System.out.println("Wine 的Fun.....");
        fun2();
    }

    public void fun2(){
        System.out.println("Wine 的Fun2...");
    }
}

public class JNC extends Wine{
    /**
     * @desc 子类重载父类方法
     *        父类中不存在该方法，向上转型后，父类是不能引用该方法的
     * @param a
     * @return void
     */
    public void fun1(String a){
        System.out.println("JNC 的 Fun1...");
        fun2();
    }

    /**
     * 子类重写父类方法
     * 指向子类的父类引用调用fun2时，必定是调用该方法
     */
    public void fun2(){
        System.out.println("JNC 的Fun2...");
    }
}

public class Test {
    public static void main(String[] args) {
        Wine a = new JNC();
        a.fun1();
    }
}
//-------------------------------------------------
//Output:
//Wine 的Fun.....
//JNC 的Fun2...
```

从程序的运行结果中我们发现，a.fun1()首先是运行父类Wine中的fun1().然后再运行子类JNC中的fun2()。

分析：在这个程序中子类JNC重载了父类Wine的方法fun1()，重写fun2()，而且重载后的fun1(String a)与 fun1()不是同一个方法，由于父类中没有该方法，向上转型后会丢失该方法，所以执行JNC的Wine类型引用是不能引用fun1(String a)方法。而子类JNC重写了fun2() ，那么指向JNC的Wine引用会调用JNC中fun2()方法。

所以对于多态我们可以总结如下：

**指向子类的父类引用由于向上转型了，它只能访问父类中拥有的方法和属性，而对于子类中存在而父类中不存在的方法，该引用是不能使用的，尽管是重载该方法。若子类重写了父类中的某些方法，在调用该些方法的时候，必定是使用子类中定义的这些方法（动态连接、动态调用）。**

对于面向对象而已，多态分为编译时多态和运行时多态。其中编辑时多态是静态的，主要是指方法的重载，它是根据参数列表的不同来区分不同的函数，通过编辑之后会变成两个不同的函数，在运行时谈不上多态。而运行时多态是动态的，它是通过动态绑定来实现的，也就是我们所说的多态性。

#### 多态的实现

在刚刚开始就提到了继承在为多态的实现做了准备。子类Child继承父类Father，我们可以编写一个指向子类的父类类型引用，该引用既可以处理父类Father对象，也可以处理子类Child对象，当相同的消息发送给子类或者父类对象时，该对象就会根据自己所属的引用而执行不同的行为，这就是多态。即多态性就是相同的消息使得不同的类做出不同的响应

Java实现多态有三个必要条件：继承、重写、向上转型。

 - 继承：在多态中必须存在有继承关系的子类和父类。

 - 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。

 - 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。


 只有满足了上述三个条件，我们才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而达到执行不同的行为。

对于Java而言，它多态的实现机制遵循一个原则：**当超类对象引用变量引用子类对象时，被引用对象的类型(而不是引用变量的类型)决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法。**


在Java中有两种形式可以实现多态。继承和接口。

##### 基于继承实现的多态.
基于继承的实现机制主要表现在父类和继承该父类的一个或多个子类对某些方法的重写，多个子类对同一方法的重写可以表现出不同的行为。
```(java)
public class Wine {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Wine(){
    }

    public String drink(){
        return "喝的是 " + getName();
    }

    /**
     * 重写toString()
     */
    public String toString(){
        return null;
    }
}

public class JNC extends Wine{
    public JNC(){
        setName("JNC");
    }

    /**
     * 重写父类方法，实现多态
     */
    public String drink(){
        return "喝的是 " + getName();
    }

    /**
     * 重写toString()
     */
    public String toString(){
        return "Wine : " + getName();
    }
}

public class JGJ extends Wine{
    public JGJ(){
        setName("JGJ");
    }

    /**
     * 重写父类方法，实现多态
     */
    public String drink(){
        return "喝的是 " + getName();
    }

    /**
     * 重写toString()
     */
    public String toString(){
        return "Wine : " + getName();
    }
}

public class Test {
    public static void main(String[] args) {
        //定义父类数组
        Wine[] wines = new Wine[2];
        //定义两个子类
        JNC jnc = new JNC();
        JGJ jgj = new JGJ();

        //父类引用子类对象
        wines[0] = jnc;
        wines[1] = jgj;

        for(int i = 0 ; i < 2 ; i++){
            System.out.println(wines[i].toString() + "--" + wines[i].drink());
        }
        System.out.println("-------------------------------");

    }
}
//OUTPUT:
//Wine : JNC--喝的是 JNC
//Wine : JGJ--喝的是 JGJ
//-------------------------------
```
在上面的代码中JNC、JGJ继承Wine，并且重写了drink()、toString()方法，程序运行结果是调用子类中方法，输出JNC、JGJ的名称，这就是多态的表现。不同的对象可以执行相同的行为，但是他们都需要通过自己的实现方式来执行，这就要得益于向上转型了。

我们都知道所有的类都继承自超类Object，toString()方法也是Object中方法，当我们这样写时：
```
Object o = new JGJ();
System.out.println(o.toString());
```
输出的结果是Wine : JGJ。

Object、Wine、JGJ三者继承链关系是：JGJ—>Wine—>Object。所以我们可以这样说：当子类重写父类的方法被调用时，只有对象继承链中的最末端的方法才会被调用。但是注意如果这样写：

```
Object o = new Wine();
System.out.println(o.toString());
```
输出的结果应该是Null，因为JGJ并不存在于该对象继承链中。

所以基于继承实现的多态可以总结如下：**对于引用子类的父类类型，在处理该引用时，它适用于继承该父类的所有子类，子类对象的不同，对方法的实现也就不同，执行相同动作产生的行为也就不同。**


如果父类是抽象类，那么子类必须要实现父类中所有的抽象方法，这样该父类所有的子类一定存在统一的对外接口，但其内部的具体实现可以各异。这样我们就可以使用顶层类提供的统一接口来处理该层次的方法。

##### 基于接口实现的多态

继承是通过重写父类的同一方法的几个不同子类来体现的，那么就可就是通过实现接口并覆盖接口中同一方法的几不同的类体现的。

在接口的多态中，指向接口的引用必须是指定这实现了该接口的一个类的实例程序，在运行时，根据对象引用的实际类型来执行对应的方法。

继承都是单继承，只能为一组相关的类提供一致的服务接口。但是接口可以是多继承多实现，它能够利用一组相关或者不相关的接口进行组合与扩充，能够对外提供一致的服务接口。所以它相对于继承来说有更好的灵活性。

接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念。类描述对象的属性和方法。接口则包含类要实现的方法。

#### 经典实例

```(java)
class A {
    public String show(D obj) {
        return ("A and D");
    }

    public String show(A obj) {
        return ("A and A");
    }

}

class B extends A{
    public String show(B obj){
        return ("B and B");
    }

    public String show(A obj){
        return ("B and A");
    }
}

class C extends B{

}

class D extends B{

}

public class Test {
    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new B();
        B b = new B();
        C c = new C();
        D d = new D();

        System.out.println("1--" + a1.show(b));
        System.out.println("2--" + a1.show(c));
        System.out.println("3--" + a1.show(d));
        System.out.println("4--" + a2.show(b));//this是A
        System.out.println("5--" + a2.show(c));
        System.out.println("6--" + a2.show(d));
        System.out.println("7--" + b.show(b));
        System.out.println("8--" + b.show(c));
        System.out.println("9--" + b.show(d));      
    }
}
```

运行结果：
```
1--A and A
2--A and A
3--A and D
4--B and A
5--B and A
6--A and D
7--B and B
8--B and B
9--A and D
```
在这里看结果1、2、3还好理解，从4开始就开始糊涂了，对于4来说为什么输出不是“B and B”呢？

首先我们先看一句话：**当超类对象引用变量引用子类对象时，被引用对象的类型而不是引用变量的类型决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法。这句话对多态进行了一个概括。其实在继承链中对象方法的调用存在一个优先级：this.show(O)->super.show(O)->this.show((super)O)->super.show((super)O)。**

从上面的程序中我们可以看出A、B、C、D存在如下关系。

![](https://i.loli.net/2019/04/10/5cadf27482abd.png)

首先我们分析5，a2.show(c)，a2是A类型的引用变量，所以this就代表了A，a2.show(c),它在A类中找发现没有找到，于是到A的超类中找(super)，由于A没有超类（Object除外），所以跳到第三级，也就是this.show((super)O)，C的超类有B、A，所以(super)O为B、A，this同样是A，这里在A中找到了show(A obj)，同时由于a2是B类的一个引用且B类重写了show(A obj)，因此最终会调用子类B类的show(A obj)方法，结果也就是B and A。


方法已经找到了但是我们这里还是存在一点疑问，我们还是来看这句话：当超类对象引用变量引用子类对象时，被引用对象的类型而不是引用变量的类型决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法。这我们用一个例子来说明这句话所代表的含义：a2.show(b)；

这里a2是引用变量，为A类型，它引用的是B对象，因此按照上面那句话的意思是说有B来决定调用谁的方法,所以a2.show(b)应该要调用B中的show(B obj)，产生的结果应该是“B and B”，但是为什么会与前面的运行结果产生差异呢？这里我们忽略了后面那句话“但是这儿被调用的方法必须是在超类中定义过的”，那么show(B obj)在A类中存在吗？根本就不存在！所以这句话在这里不适用？那么难道是这句话错误了？非也！其实这句话还隐含这这句话：它仍然要按照继承链中调用方法的优先级来确认。所以它才会在A类中找到show(A obj)，同时由于B重写了该方法所以才会调用B类中的方法，否则就会调用A类中的方法。

所以多态机制遵循的原则概括为：**当超类对象引用变量引用子类对象时，被引用对象的类型而不是引用变量的类型决定了调用谁的成员方法，但是这个被调用的方法必须是在超类中定义过的，也就是说被子类覆盖的方法，但是它仍然要根据继承链中方法调用的优先级来确认方法，该优先级为：this.show(O)、super.show(O)、this.show((super)O)、super.show((super)O)。**

### 类加载顺序

Java代码中一个类初始化顺序：static变量 --  其他成员变量  --  构造函数

三者的调用先后顺序：

父类的静态字段——>父类静态代码块——>子类静态字段——>子类静态代码块——>父类成员变量（非静态字段）——>父类非静态代码块——>父类构造器——>子类成员变量——>子类非静态代码块——>子类构造器

系统默认值的给予比通过等号的赋予先执行。

一个类中的static变量或成员变量的初始化顺序，是按照声明的顺序初始化的。


```（java）
public class ClassWithStatic extends Super{
    private static int iTest0 = 0;
    private static ClassWithStatic instance = new ClassWithStatic();  //4
    private static int iTest1;
    private static int iTest2 = 0;
    static {
        System.out.println(ClassWithStatic.class.getName() + " : static{}"); //5
        iTest0++;
        iTest1++;
        iTest2++;
    }

    public ClassWithStatic() { //先调用父类默认构造函数，再使用本类的构造函数
        System.out.println(ClassWithStatic.class.getName() + " : Constuctor with this = " + this);
        iTest0++;
        iTest1++;
        iTest2++;
    }

    /**
     * @return the instance
     */
    public static ClassWithStatic getInstance() {
        return instance;
    }

    public void doSomeThing() {
      System.out.println(" ClassWithStatic");
        System.out.println("iTest0 = " + iTest0);
        System.out.println("iTest1 = " + iTest1);
        System.out.println("iTest2 = " + iTest2);
    }

    public static void main(String[] args) {
        //private static void main(String[] args)
        //Error: Main method not found in class san.ClassWithStatic, please define the main method as:
        //   public static void main(String[] args)
        //   or a JavaFX application class must extend javafx.application.Application

        //类加载完毕，实现static变量和代码块

        System.out.println("public static void main(String[] args)"); //6
        Super.getInstance().doSomeThing();
        ClassWithStatic.getInstance().doSomeThing();

    }
}

class Super {
    private static Super instance = new Super();  // 1
    private static int iTest1;
    private int iTest2 = 0;
    static {    
        System.out.println(Super.class.getName() + " : static{}");  //2
        iTest1++;
    }

    public Super() {
        System.out.println(Super.class.getName() + " : Constuctor with this = " + this);
        iTest2++;
    }
    public static Super getInstance() {
        return instance;
    }
    public void doSomeThing() {
        System.out.println(" Super");
        System.out.println("iTest1 = " + iTest1);
        System.out.println("iTest2 = " + iTest2);
    }
}
```
运行结果：

```
$ java  ClassWithStatic
Super : Constuctor with this = Super@2a139a55
Super : static{}
Super : Constuctor with this = ClassWithStatic@15db9742
ClassWithStatic : Constuctor with this = ClassWithStatic@15db9742
ClassWithStatic : static{}
public static void main(String[] args)
 Super
iTest1 = 1
iTest2 = 1
 ClassWithStatic
iTest0 = 2
iTest1 = 2
iTest2 = 1
```

上述加载步骤：

1. 加载主类ClassWithStatic的父类Super其中的静态成员，按照声明顺序：初始化instance，即构建Super的实例，执行构造函数（Super : Constuctor with this = Super@2a139a55，实例变量iTest2=1）;Super中的其他static成员还未加载，现在按照声明顺序将其加载完成后（Super : static{}，静态变量iTest1=1）。

2. 加载ClassWithStatic其中的静态变量，按照声明顺序：iTest0=0;初始化instance，即构建ClassWithStatic实例，先执行Super中的默认构造函数（Super : Constuctor with this = ClassWithStatic@15db9742），再执行本类的默认构造函数，因为会使用到iTest1和iTest2，所以将其按照默认值0加载（ClassWithStatic : Constuctor with this = ClassWithStatic@15db9742，iTest0=1,iTest1=1,iTest2=1）;按照声明顺序，继续执行iTest1不变，iTest2=0;执行static块（iTest0=2,iTest1=2,iTest2=1）

3. 执行主函数（public static void main(String[] args)），执行调用函数。

示例二

```java
public class ClassWithStatic extends Super {
    protected static int iTest0 = Super.iTest0 + 1;
    private static ClassWithStatic instance = new ClassWithStatic();
    protected static int iTest1;
    private static int iTest2 = 0;
    static {
        System.out.println(ClassWithStatic.class.getName() + " : static{}");
        iTest1++;
        iTest2++;
    }

    public ClassWithStatic() {
        System.out.println(ClassWithStatic.class.getName() + " : Constuctor with this = " + this);
        iTest1++;
        iTest2++;
    }

    /**
     * @return the instance
     */
    public static ClassWithStatic getInstance() {
        return instance;
    }

    public void doSomeThing() {
        System.out.println("ClassWithStatic.iTest0 = " + iTest0);
        System.out.println("ClassWithStatic.iTest1 = " + iTest1);
        System.out.println("ClassWithStatic.iTest2 = " + iTest2);
    }

    public static void main(String[] args) {
        //private static void main(String[] args)
        //Error: Main method not found in class san.ClassWithStatic, please define the main method as:
        //   public static void main(String[] args)
        //   or a JavaFX application class must extend javafx.application.Application
        System.out.println("public static void main(String[] args)");

        ClassWithStatic.getInstance().doSomeThing();
        System.out.println("Super.iTest0 = " + Super.iTest0);
        System.out.println(Const.constanceString);//对类的静态变量进行读取、赋值操作的。static,final且值确定是常量,是编译时确定的，调用的时候直接用，不会加载对应的类
        System.out.println("------------------------");
        Const.doStaticSomeThing();
    }
}

class Super {
    protected static int iTest0;
    private static Super instance = new Super();
    protected static int iTest1 = 0;
    static {
        System.out.println(Super.class.getName() + " : static{}");
        iTest0 = ClassWithStatic.iTest0 + 1;//1
    }

    public Super() {
        System.out.println(Super.class.getName() + " : Constuctor with this = " + this + ", iTest0 = " + iTest0);
        iTest1++;
    }
}

class Const {
    public static final String constanceString = "Const.constanceString";
    static {
        System.out.println(Const.class.getName() + " : static{}");
    }
    public static void doStaticSomeThing() {
        System.out.println(Const.class.getName() + " : doStaticSomeThing();");
    }
}
```
运行结果：

```
Super : Constuctor with this = Super@2a139a55, iTest0 = 0
Super : static{}
Super : Constuctor with this = ClassWithStatic@15db9742, iTest0 = 1
ClassWithStatic : Constuctor with this = ClassWithStatic@15db9742
ClassWithStatic : static{}
public static void main(String[] args)
ClassWithStatic.iTest0 = 2
ClassWithStatic.iTest1 = 2
ClassWithStatic.iTest2 = 1
Super.iTest0 = 1
Const.constanceString
------------------------
Const : static{}
Const : doStaticSomeThing();

```
请按照上述规则分析。

#### 内部类与加载顺序

**内部类中不能含有静态变量**

静态变量是要占用内存的，在编译时只要是定义为静态变量了，系统就会自动分配内存给他，而内部类是在宿主类编译完编译的，也就是说，必须有宿主类存在后才能有内部类，这也就和编译时就为静态变量分配内存产生了冲突，因为系统执行：**运行宿主类->静态变量内存分配->内部类**，而此时内部类的静态变量先于内部类生成，这显然是不可能的，所以不能定义静态变量！


#### 使用局部代码块的优点

初始化代码块/构造代码块 --- 定义在类内 --- 在创建对象的时候先于构造方法来执行一次。

局部代码块 --- 定义方法中 --- 限制变量的生命周期，以提高栈内存的利用率

#### 静态变量

static修饰变量---静态变量/类变量。静态变量在类加载的时候加载到方法区，并且在方法区中被赋予默认值。由于静态变量先于对象出现，所以可以通过类名来调用静态变量，也可以通过对象调用。这个类的所有对象存储的是这个静态变量在方法区的地址，所以所有对象是共享这个静态变量。

1. 类是加载到方法区中---类中的所有的信息都会加载方法区中

2. 类是第一次使用的时候加载到方法区，加载之后不在移除 --- 意味着类只加载一次

静态变量能否定义到构造方法中？

---不可以。--- 静态变量在类加载的时候加载到方法区；构造方法是在创建对象的时候调用，在栈内存中执行。


**关于static，abstract，final的共存**

![](https://i.loli.net/2019/04/10/5cadff51117c4.png)

### 面试题

1. 指出下面程序的运行结果。

   ```java
   class A {
   
       static {
           System.out.print("1");
       }
   
       public A() {
           System.out.print("2");
       }
   }
   
   class B extends A{
   
       static {
           System.out.print("a");
       }
   
       public B() {
           System.out.print("b");
       }
   }
   
   public class Hello {
   
       public static void main(String[] args) {
           A ab = new B();
           ab = new B();
       }
   
   }
   
   ```

   答：执行结果：1a2b2b。创建对象时构造器的调用顺序是：先初始化静态成员，然后调用父类构造器，再初始化非静态成员，最后调用自身构造器。

2. 描述一下JVM加载class文件的原理机制？

   答：JVM中类的装载是由类加载器（ClassLoader）和它的子类来实现的，Java中的类加载器是一个重要的Java运行时系统组件，它负责在运行时查找和装入类文件中的类。

   由于Java的跨平台性，经过编译的Java源程序并不是一个可执行程序，而是一个或多个类文件。当Java程序需要使用某个类时，JVM会确保这个类已经被加载、连接（验证、准备和解析）和初始化。类的加载是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件，然后产生与所加载类对应的Class对象。加载完成后，Class对象还不完整，所以此时的类还不可用。当类被加载后就进入连接阶段，这一阶段包括验证、准备（为静态变量分配内存并设置默认的初始值）和解析（将符号引用替换为直接引用）三个步骤。最后JVM对类进行初始化，包括：1)如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；2)如果类中存在初始化语句，就依次执行这些初始化语句。

   类的加载是由类加载器完成的，类加载器包括：根加载器（BootStrap）、扩展加载器（Extension）、系统加载器（System）和用户自定义类加载器（java.lang.ClassLoader的子类）。从Java 2（JDK 1.2）开始，类加载过程采取了父亲委托机制（PDM）。PDM更好的保证了Java平台的安全性，在该机制中，JVM自带的Bootstrap是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。JVM不会向Java程序提供对Bootstrap的引用。下面是关于几个类加载器的说明：

   - Bootstrap：一般用本地代码实现，负责加载JVM基础核心类库（rt.jar）；
   - Extension：从java.ext.dirs系统属性所指定的目录中加载类库，它的父加载器是Bootstrap；
   - System：又叫应用类加载器，其父类是Extension。它是应用最广泛的类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中加载类，是用户自定义加载器的默认父加载器。

