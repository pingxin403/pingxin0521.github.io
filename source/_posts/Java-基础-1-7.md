---
title: Java 包、static、final和内部类
date: 2019-04-04 13:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 包

问题：当定义了多个类的时候，可能会发生类名的重复问题。在java中采用包机制处理开发者定义的类名冲突问题。怎么使用java的包机制呢？

这就需要package了。Java 中的包package, 就是电脑中的文件夹。我们平时在工作中，文件太多时，都会新建文件夹进行分类管理，java 中的包也是类似的道理，当我们的类太多时，也需要进行分类管理，这时我们就会把类文件放到包中，就是把这个.class文件放到了一个文件夹中，这样也能有效地避免了命名冲突。
<!--more-->

java包的作用是为了区别类名的命名空间:

1. 能相似或相关的类或接口组织在同一个包中，方便类的查找和使用。、

2. 如同文件夹一样，包也采用了树形目录的存储方式。同一个包中的类名字是不同的，不同的包中的类的名字是可以相同的，当同时调用两个不同包中相同类名的类时，应该加上包名加以区别。因此，包可以避免名字冲突。

3. 包也限定了访问权限，拥有包访问权限的类才能访问某个包中的类。

当我们对java源文件进行编译时，它会生成一个.class 文件，如果我们在java源文件的顶部，指定一个包名（package net;）, 编译时，这个包名会生成一个文件夹，在这里就是net文件夹，编译好的.class文件则会放到该文件夹下。

```java
package net;   // 指定包名net

public class Test {
    public static void main(String[] args) {
        System.out.println("hello");
    }
}
```

当包名很长时，如package net.com.cn，这时它就会生成多级文件夹，先生成net文件夹，然后在net文件夹中生成com文件夹，最后在com文件夹中生成cn文件夹，我们编译生成的.class 文件就会放到cn 文件夹中。我们写一个Animal 类，指定包名为net.com.cn

```java
package net.com.cn;

public class Animal {

    private String name;
    private int age;

    public Animal( String name, int age) {
        this.name = name;
        this.age = age;
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
}
```

编译之后，你会发现我们当前文夹中多了一个net文件夹，打开net文件夹，发现com文件夹，再打开com文件夹，发现cn 文件夹，在cn文件里面才有我们的Animal.class 类文件。 有了包之后，这个Animal类就有了所属，这个类实际上叫做net.com.cn.Animal, 如果在java源文件中使用这个类创建对象，就要使用类的全称net.com.cn.Anima
```
net.com.cn.Animal cat = new net.com.cn.Animal();
```

这样使用类创建对象就非常麻烦了，所以出现了import导包。 把要使用的类直接导进来，我们在源代码中就直接可以使用类名进行书写。 我们写一个test.java文件
```java
import net.com.cn.Animal;  // 引入包

public class Test {
    public static void main(String[] args) {
        Animal cat = new Animal();  // 直接使用类名进行书写
        cat.setName("miaomiao");
        System.out.println(cat.getName());
    }

}
```
当引入包之后，就是一个类文件有了所属之后，这又带来权限问题。如果包中的class类文件能够被访问，它必须是public的，包中的方法，如果能够被访问，它也必须是public.  包与包之间的类进行访问，被访问的包中的类必须是public权限的，被访问的包中的类的方法也必须是public 权限的。

```
import package1[.package2…].(classname|*);
```
如果在一个包中，一个类想要使用本包中的另一个类，那么该包名可以省略。

通常，一个公司使用它互联网域名的颠倒形式来作为它的包名.例如：互联网域名是 runoob.com，所有的包名都以 com.runoob 开头。包名中的每一个部分对应一个子目录。

在jdk1.5以上版本可以使用import导入静态成员。
```java
import static 静态成员;
```
例如：
```java
import static java.lang.System.out;
```
### static

在《Java编程思想》P86页有这样一段话：

>“static方法就是没有this的方法。在static方法内部不能调用非静态方法，反过来是可以的。
>而且可以在没有创建任何对象的前提下，仅仅通过类本身来调用static方法。这实际上正是static方法的主要用途。”

这段话虽然只是说明了static方法的特殊之处，但是可以看出static关键字的基本作用，简而言之，一句话来描述就是：

**方便在没有创建对象的情况下来进行调用（方法/变量）。**

很显然，被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问。

static可以用来修饰类的成员方法、类的成员变量，另外可以编写static代码块来优化程序性能

#### static 变量
static变量也称作静态变量，静态变量和非静态变量的区别是：静态变量被所有的对象所共享，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化。而非静态变量是对象所拥有的，在创建对象的时候被初始化，存在多个副本，各个对象拥有的副本互不影响。

两者的区别是：

 - 对于静态变量在内存中只有一个拷贝（节省内存），JVM只为静态分配一次内存，在加载类的过程中完成静态变量的内存分配，可用类名直接访问（方便），当然也可以通过对象来访问（但是这是不推荐的）。

 - 对于实例变量，没创建一个实例，就会为实例变量分配一次内存，实例变量可以在内存中有多个拷贝，互不影响（灵活）。

static成员变量的**初始化顺序按照定义的顺序进行初始化**。

#### static代码块

static关键字还有一个比较关键的作用就是 用来形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会**按照static块的顺序来执行每个static块**，并且**只会执行一次。**

为什么说static块可以用来优化程序性能，是因为它的特性:只会在类加载的时候执行一次。下面看个例子:
```java
class Person{
    private Date birthDate;

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBoomer() {
        Date startDate = Date.valueOf("1946");
        Date endDate = Date.valueOf("1964");
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```
isBornBoomer是用来这个人是否是1946-1964年出生的，而每次isBornBoomer被调用的时候，都会生成startDate和birthDate两个对象，造成了空间浪费，如果改成这样效率会更好：
```java
class Person{
    private Date birthDate;
    private static Date startDate,endDate;
    static{
        startDate = Date.valueOf("1946");
        endDate = Date.valueOf("1964");
    }

    public Person(Date birthDate) {
        this.birthDate = birthDate;
    }

    boolean isBornBoomer() {
        return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
    }
}
```
因此，很多时候会将一些**只需要进行一次的初始化操作都放在static代码块中进行**。

#### static方法

static方法一般称作静态方法，由于静态方法不依赖于任何对象就可以进行访问，因此对于静态方法来说，是没有this的，因为它不依附于任何对象，既然都没有对象，就谈不上this了。并且由于这个特性，**在静态方法中不能访问类的非静态成员变量和非静态成员方法**，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用。

但是要注意的是，虽然在静态方法中不能访问非静态成员方法和非静态成员变量，但是 **在非静态成员方法中是可以访问静态成员方法/变量的。**

而对于非静态成员方法，它访问静态成员方法/变量显然是毫无限制的。

因此，如果说想在不创建对象的情况下调用某个方法，就可以将这个方法设置为static。我们最常见的static方法就是main方法，至于为什么main方法必须是static的，现在就很清楚了。因为程序在执行main方法的时候没有创建任何对象，因此只有通过类名来访问。

#### 深入理解static

**static关键字会改变类中成员的访问权限吗？**

有些初学的朋友会将java中的static与C/C++中的static关键字的功能混淆了。在这里只需要记住一点：与C/C++中的static不同，Java中的static关键字不会影响到变量或者方法的作用域。在Java中能够影响到访问权限的只有private、public、protected（包括包访问权限）这几个关键字。

**能通过this访问静态成员变量吗？**

虽然对于静态方法来说没有this，那么在非静态方法中能够通过this访问静态成员变量吗？先看下面的一个例子，这段代码输出的结果是什么？

```(java)
public class Main {　　
    static int value = 33;

    public static void main(String[] args) throws Exception{
        new Main().printValue();
    }

    private void printValue(){
        int value = 3;
        System.out.println(this.value);
    }
}
```
这里面主要考察this和static的理解。this代表什么？this代表当前对象，那么通过new Main()来调用printValue的话，当前对象就是通过new Main()生成的对象。而static变量是被对象所享有的，因此在printValue中的this.value的值毫无疑问是33。在printValue方法内部的value是局部变量，根本不可能与this关联，所以输出结果是33。

在这里永远要记住一点：静态成员变量虽然独立于对象，但是不代表不可以通过对象去访问，所有的静态方法和静态变量都可以通过对象访问（只要访问权限足够）。

**static能作用于局部变量么？**

在C/C++中static是可以作用域局部变量的，但是在Java中切记：**static是不允许用来修饰局部变量。** 不要问为什么，这是Java语法的规定。

**static和final一块用表示什么**

static final用来修饰成员变量和成员方法，可简单理解为“全局常量”！

**对于变量，表示一旦给值就不可修改，并且通过类名可以访问。**

**对于方法，表示不可覆盖，并且可以通过类名直接访问。**

有时你希望定义一个类成员，使它的使用完全独立于该类的任何对象。通常情况下，类成员必须通过它的类的对象访问，但是可以创建这样一个成员，它能够被它自己使用，而不必引用特定的实例。在成员的声明前面加上关键字static(静态的)就能创建这样的成员。如果一个成员被声明为static，它就能够在它的类的任何对象创建之前被访问，而不必引用任何对象。你可以将方法和变量都声明为static。static 成员的最常见的例子是main( ) 。因为在程序开始执行时必须调用main() ，所以它被声明为static。

声明为static的变量实质上就是全局变量。当声明一个对象时，并不产生static变量的拷贝，而是该类所有的实例变量共用同一个static变量。声明为static的方法有以下几条限制：

 - 它们仅能调用其他的static 方法。

 - 它们只能访问static数据。

 - 它们不能以任何方式引用this 或super

### final

final在Java中是一个保留的关键字，可以声明成员变量、方法、类以及本地变量。一旦你将引用声明作final，你将不能改变这个引用了，编译器会检查代码，如果你试图将变量再次初始化的话，编译器会报编译错误。

#### final 变量

final成员变量表示常量，只能被赋值一次，赋值后值不再改变。

**当final修饰一个基本数据类型时，表示该基本数据类型的值一旦在初始化后便不能发生变化；如果final修饰一个引用类型时，则在对其初始化之后便不能再让其指向其他对象了，但该引用所指向的对象的内容是可以发生变化的。本质上是一回事，因为引用的值是一个地址，final要求值，即地址的值不发生变化。**

“常量”主要应用与以下两个地方：

1. 编译期常量，永远不可改变。

2. 运行期初始化时，我们希望它不会被改变。

final修饰一个成员变量（属性），必须要显示初始化。这里有两种初始化方式，一种是在变量声明的时候初始化；第二种方法是在声明变量的时候不赋初值，但是要在这个变量所在的类的所有的构造函数中对这个变量赋初值。

凡是对成员变量或者本地变量(在方法中的或者代码块中的变量称为本地变量)声明为final的都叫作final变量。final变量经常和static关键字一起使用，作为常量。下面是final变量的例子：
```
public static final String LOAN="loan";
LOAN=new String("loan");//invalid compilation error
```
**final变量是只读的。**

#### final 方法

下面这段话摘自《Java编程思想》第四版第143页：

>“使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。
>在早期的Java实现版本中，会将final方法转为内嵌调用。
>但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升。
>在最近的Java版本中，不需要使用final方法进行这些优化了。“


final也可以声明方法。方法前面加上final关键字，代表这个方法不可以被子类的方法重写。如果你认为一个方法的功能已经足够完整了，子类中不需要改变的话，你可以声明此方法为final。**final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时再动态绑定。** 下面是final方法的例子：

```
class Person{
  public final String getName(){
        return "personal loan";
    }
}

class loan extends Person{
      @Override
        public final Stirng getName(){
                return "cheap personal loan";//compilation error: overridden method is final
        }
}
```

final修饰的方法表示此方法已经是“最后的、最终的”含义，亦即此方法不能被重写（可以重载多个final修饰的方法）。此处需要注意的一点是：因为重写的前提是子类可以从父类中继承此方法，**如果父类中final修饰的方法同时访问控制权限为private，将会导致子类中不能直接继承到此方法，因此，此时可以在子类中定义相同的方法名和参数，此时不再产生重写与final的矛盾，而是在子类中重新定义了新的方法。**（注：**类的private方法会隐式地被指定为final方法。**）

#### final 类

使用final来修饰的类叫作final类。**final类通常功能是完整的，它们不能被继承。** Java中有许多类是final的，譬如String, Interger以及其他包装类。下面是final类的实例：
```
final class PersonalLoan{}
class CheapPersonalLoan extends PersonalLoan{}  //compilation error: cannot inherit from final class
```

**final关键字的好处**

下面总结了一些使用final关键字的好处

1. final关键字提高了性能。JVM和Java应用都会缓存final变量。

2. final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。

3. 使用final关键字，JVM会对方法、变量及类进行优化。

**不可变类**

创建不可变类要使用final关键字。不可变类是指它的对象一旦被创建了就不能被更改了。String是不可变类的代表。不可变类有很多好处，譬如它们的对象是只读的，可以在多线程环境下安全的共享，不用额外的同步开销等等。

相关阅读：[为什么String是不可变的以及如何写一个不可变类](https://www.cnblogs.com/wcyBlog/p/4073725.html)。

**关于final的重要知识点**

1. final关键字可以用于成员变量、本地变量、方法以及类。

2. final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。

3. 你不能够对final变量再次赋值。

4. 本地变量必须在声明时赋值。

5. 在匿名类中所有变量都必须是final变量。

6. final方法不能被重写。

7. final类不能被继承。

8. final关键字不同于finally关键字，后者用于异常处理。

9. final关键字容易与finalize()方法搞混，后者是在Object类中定义的方法，是在垃圾回收之前被JVM调用的方法。

10. 接口中声明的所有变量本身是final的。类和抽象类中private 方法默认就是final。

11. final和abstract这两个关键字是反相关的，final类就不可能是abstract的。

12. final方法在编译阶段绑定，称为静态绑定(static binding)。

13. 没有在声明时初始化final变量的称为空白final变量(blank final variable)，它们必须在构造器中初始化，或者调用this()初始化。不这么做的话，编译器会报错“final变量(变量名)需要进行初始化”。

14. 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化。

15. 按照Java代码惯例，final变量就是常量，而且通常 **常量名要大写**。

16. 对于集合对象声明为final指的是引用不能被更改，但是你可以向其中增加，删除或者改变内容。

```
private final List<String> Loans = new ArrayList<String>();
private void test01()
{
    Loans.add("home loan");  //valid
    Loans.add("personal loan"); //valid
    Loans = new Vector<String>();  //not valid
}
```

#### 深入理解final关键字

在了解了final关键字的基本用法之后，这一节我们来看一下final关键字容易混淆的地方。

**类的final变量和普通变量有什么区别？**

当用final作用于类的成员变量时，成员变量（注意是类的成员变量，局部变量只需要保证在使用之前被初始化赋值即可）必须在定义时或者构造器中进行初始化赋值，而且final变量一旦被初始化赋值之后，就不能再被赋值了。

那么final变量和普通变量到底有何区别呢？下面请看一个例子：
```(java)
public class Test {
    public static void main(String[] args)  {
        String a = "hello2";   
        final String b = "hello";
        String d = "hello";
        String c = b + 2;   
        String e = d + 2;
        System.out.println((a == c));
        System.out.println((a == e));
    }
}
```
输出结果：true、false

大家可以先想一下这道题的输出结果。为什么第一个比较结果为true，而第二个比较结果为fasle。这里面就是final变量和普通变量的区别了，当final变量是基本数据类型以及String类型时，如果在编译期间能知道它的确切值，则编译器会把它当做编译期常量使用。也就是说在用到该final变量的地方，相当于直接访问的这个常量，不需要在运行时确定。这种和C语言中的宏替换有点像。因此在上面的一段代码中，由于变量b被final修饰，因此会被当做编译器常量，所以在使用到b的地方会直接将变量b 替换为它的值。而对于变量d的访问却需要在运行时通过链接来进行。

想必其中的区别大家应该明白了，不过要注意，**只有在编译期间能确切知道final变量值的情况下，编译器才会进行这样的优化**，比如下面的这段代码就不会进行优化：

```(java)
public class Test {
    public static void main(String[] args)  {
        String a = "hello2";   
        final String b = getHello();
        String c = b + 2;   
        System.out.println((a == c));

    }

    public static String getHello() {
        return "hello";
    }
}
```
这段代码的输出结果为false。这里要注意一点就是：**不要以为某些数据是final就可以在编译期知道其值，通过变量b我们就知道了，在这里是使用getHello()方法对其进行初始化，他要在运行期才能知道其值**。

**被final修饰的引用变量指向的对象内容可变吗？**

在上面提到被final修饰的引用变量一旦初始化赋值之后就不能再指向其他的对象，那么该引用变量指向的对象的内容可变吗？看下面这个例子：
```(java)
public class Test {
    public static void main(String[] args)  {
        final MyClass myClass = new MyClass();
        System.out.println(++myClass.i);

    }
}  
class MyClass {
    public int i = 0;
}
```
这段代码可以顺利编译通过并且有输出结果，输出结果为1。**这说明引用变量被final修饰之后，虽然不能再指向其他对象，但是它指向的对象的内容是可变的。**


**final参数的问题**

在实际应用中，我们除了可以用final修饰成员变量、成员方法、类，还可以修饰参数、若某个参数被final修饰了，则代表了该参数是不可改变的。如果在方法中我们修改了该参数，则编译器会提示你：The final local variable i cannot be assigned. It must be blank and not using a compound assignment。看下面的例子：

```(java)
public class TestFinal {

    public static void main(String[] args){
        TestFinal testFinal = new TestFinal();
        int i = 0;
        testFinal.changeValue(i);
        System.out.println(i);

    }

    public void changeValue(final int i){
        //final参数不可改变
        //i++;
        System.out.println(i);
    }
}
```

上面这段代码changeValue方法中的参数i用final修饰之后，就不能在方法中更改变量i的值了。值得注意的一点，方法changeValue和main方法中的变量i根本就不是一个变量，因为java参数传递采用的是值传递，对于基本类型的变量，相当于直接将变量进行了拷贝。所以即使没有final修饰的情况下，在方法内部改变了变量i的值也不会影响方法外的i。

再看下面这段代码：
```(java)
public class TestFinal {

    public static void main(String[] args){
        TestFinal testFinal = new TestFinal();
        StringBuffer buffer = new StringBuffer("hello");
        testFinal.changeValue(buffer);
        System.out.println(buffer);

    }

    public void changeValue(final StringBuffer buffer){
        //final修饰引用类型的参数，不能再让其指向其他对象，但是对其所指向的内容是可以更改的。
        //buffer = new StringBuffer("hi");
        buffer.append("world");
    }
}
```
运行这段代码就会发现输出结果为 helloworld。很显然，用final进行修饰虽不能再让buffer指向其他对象，但对于buffer指向的对象的内容是可以改变的。现在假设一种情况，如果把final去掉，结果又会怎样？看下面的代码：

```(java)
public class TestFinal {

    public static void main(String[] args){
        TestFinal testFinal = new TestFinal();
        StringBuffer buffer = new StringBuffer("hello");
        testFinal.changeValue(buffer);
        System.out.println(buffer);

    }

    public void changeValue(StringBuffer buffer){
        //buffer重新指向另一个对象
        buffer = new StringBuffer("hi");
        buffer.append("world");
        System.out.println(buffer);
    }
}
```
运行结果：
```
hiworld
hello
```
从运行结果可以看出，将final去掉后，同时在changeValue中让buffer指向了其他对象，并不会影响到main方法中的buffer，**原因在于java采用的是值传递，对于引用变量，传递的是引用的值，也就是说让实参和形参同时指向了同一个对象，因此让形参重新指向另一个对象对实参并没有任何影响。**


### 内部类

内部类我们从外面看是非常容易理解的，无非就是在一个类的内部在定义一个类。

```(java)
public class OuterClass {
    private String name ;
    private int age;

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

    class InnerClass{
        public InnerClass(){
            name = "pingxin";
            age = 23;
        }
    }
}
```
 在这里InnerClass就是内部类，对于初学者来说内部类实在是使用的不多，但是随着编程能力的提高，我们会领悟到它的魅力所在，它可以使用能够更加优雅的设计我们的程序结构。在使用内部类之间我们需要明白为什么要使用内部类，内部类能够为我们带来什么样的好处。

为什么要使用内部类？在《Think in java》中有这样一句话：

**使用内部类最吸引人的原因是：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。**

在我们程序设计中有时候会存在一些使用接口很难解决的问题，这个时候我们可以利用内部类提供的、可以继承多个具体的或者抽象的类的能力来解决这些程序设计问题。可以这样说，**接口只是解决了部分问题，而内部类使得多重继承的解决方案变得更加完整。**

```java
public interface Father {

}

public interface Mother {

}

public class Son implements Father, Mother {

}

public class Daughter implements Father{

    class Mother_ implements Mother{

    }
}
```
其实对于这个实例我们确实是看不出来使用内部类存在何种优点，但是如果Father、Mother不是接口，而是抽象类或者具体类呢？这个时候我们就只能使用内部类才能实现多重继承了。

其实使用内部类最大的优点就在于它能够非常好的解决多重继承的问题，但是如果我们不需要解决多重继承问题，那么我们自然可以使用其他的编码方式，但是使用内部类还能够为我们带来如下特性（摘自《Think in java》）：

1. 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。

2. 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。

3. 创建内部类对象的时刻并不依赖于外围类对象的创建。

4. 内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。

5. 内部类提供了更好的封装，除了该外围类，其他类都不能访问。


当我们在创建一个内部类的时候，它无形中就与外围类有了一种联系，依赖于这种联系，它可以无限制地访问外围类的元素。

```java
public class OuterClass {
    private String name ;
    private int age;

    /**省略getter和setter方法**/

    public class InnerClass{
        public InnerClass(){
            name = "pingxin";
            age = 23;
        }

        public void display(){
            System.out.println("name：" + getName() +"   ;age：" + getAge());
        }
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        innerClass.display();
    }
}
//--------------
//Output：
//name：pingxin   ;age：23
```

在这个应用程序中，我们可以看到内部类InnerClass可以对外围类OuterClass的属性进行无缝的访问，尽管它是private修饰的。这是因为当我们在创建某个外围类的内部类对象时，此时内部类对象必定会捕获一个指向那个外围类对象的引用，只要我们在访问外围类的成员时，就会用这个引用来选择外围类的成员。

其实在这个应用程序中我们还看到了如何来引用内部类：引用内部类我们需要指明这个对象的类型：OuterClasName.InnerClassName。同时如果我们需要创建某个内部类对象，必须要利用外部类的对象通过.new来创建内部类： OuterClass.InnerClass innerClass = outerClass.new InnerClass();。

同时如果我们需要生成对外部类对象的引用，可以使用OuterClassName.this，这样就能够产生一个正确引用外部类的引用了。当然这点实在编译期就知晓了，没有任何运行时的成本。

```java
public class OuterClass {
    public void display(){
        System.out.println("OuterClass...");
    }

    public class InnerClass{
        public OuterClass getOuterClass(){
            return OuterClass.this;
        }
    }

    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        innerClass.getOuterClass().display();
    }
}
//-------------
//Output:
//OuterClass...
```
到这里了我们需要明确一点，内部类是个编译时的概念，一旦编译成功后，它就与外围类属于两个完全不同的类（当然他们之间还是有联系的）。对于一个名为OuterClass的外围类和一个名为InnerClass的内部类，在编译成功后，会出现这样两个class文件：OuterClass.class和OuterClass$InnerClass.class。

在Java中内部类主要分为**成员内部类、局部内部类、匿名内部类、静态内部类。**

#### 成员内部类

成员内部类也是最普通的内部类，它**是外围类的一个成员**，所以他是可以**无限制的访问外围类的所有 成员属性和方法，尽管是private的**，但是外围类要访问内部类的成员属性和方法则需要**通过内部类实例**来访问。

在成员内部类中要注意两点:

**第一：成员内部类中不能存在任何static的变量和方法；**

**第二：成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类。**

```java
public class OuterClass {
    private String str;

    public void outerDisplay(){
        System.out.println("outerClass...");
    }

    public class InnerClass{
        public void innerDisplay(){
            //使用外围内的属性
            str = "pingxin...";
            System.out.println(str);
            //使用外围内的方法
            outerDisplay();
        }
    }

    /*推荐使用getxxx()来获取成员内部类，尤其是该内部类的构造函数无参数时 */
    public InnerClass getInnerClass(){
        return new InnerClass();
    }

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        OuterClass.InnerClass inner = outer.getInnerClass();
        inner.innerDisplay();
    }
}
//--------------------
//pingxin...
//outerClass...
```

推荐使用getxxx()来获取成员内部类，尤其是该内部类的构造函数无参数时 。


#### 局部内部类

有这样一种内部类，它是**嵌套在方法和作用域内**的，对于这个类的使用主要是应用与解决比较复杂的问题，想创建一个类来辅助我们的解决方案，到那时又不希望这个类是公共可用的，所以就产生了局部内部类，局部内部类和成员内部类一样被编译，只是它的作用域发生了改变，它**只能在该方法和属性中被使用**，出了该方法和属性就会失效。

定义在方法里：
```java
public class Parcel5 {

  public Destionation destionation(String str){

      class PDestionation implements Destionation{
          private String label;

          public PDestionation(String whereTo){
              label = whereTo;

          }
          @Override
          public String readLabel(){
              return label;
          }
      }

      return new PDestionation(str);
  }
  public static void main(String[] args) {
      Parcel5 parcel5 = new Parcel5();
      Destionation d = parcel5.destionation("pingxin...");
      System.out.println(d.readLabel());
  }
}
interface Destionation {
  String readLabel();
}
```

定义在作用域内:

```java
public class Parcel6 {
    private void internalTracking(boolean b){
        if(b){
            class TrackingSlip{
                private String id;
                TrackingSlip(String s) {
                    id = s;
                }
                String getSlip(){
                    return id;
                }
            }
            TrackingSlip ts = new TrackingSlip("pingxin...");
            String string = ts.getSlip();
        }
    }

    public void track(){
        internalTracking(true);
    }

    public static void main(String[] args) {
        Parcel6 parcel6 = new Parcel6();
        parcel6.track();
    }
}
```
#### 匿名内部类

在做Swing编程中，我们经常使用这种方式来绑定事件

```java
button2.addActionListener(  
                new ActionListener(){  
                    public void actionPerformed(ActionEvent e) {  
                        System.out.println("你按了按钮二");  
                    }  
                });
```

我们咋一看可能觉得非常奇怪，因为这个内部类是没有名字的，在看如下这个例子：

```(java)
public class OuterClass {
    public InnerClass getInnerClass(final int num,String str2){
        return new InnerClass(){
            int number = num + 3;
            public int getNumber(){
                return number;
            }
        };        /* 注意：分号不能省 */
    }

    public static void main(String[] args) {
        OuterClass2 out = new OuterClass2();
        InnerClass inner = out.getInnerClass(2, "pingxin");
        System.out.println(inner.getNumber());
    }
}

interface InnerClass {
    int getNumber();
}

//----------------
//Output:
```
这里我们就需要看清几个地方

1. 匿名内部类是**没有访问修饰符**的。

2.  new 匿名内部类，这个类首先是要存在的。如果我们将那个InnerClass接口注释掉，就会出现编译出错。

3.  注意getInnerClass()方法的形参，第一个形参是用final修饰的，而第二个却没有。同时我们也发现第二个形参在匿名内部类中没有使用过，所以**当所在方法的形参需要被匿名内部类使用，那么这个形参就必须为final**。

4.  匿名内部类是**没有构造方法**的。因为它连名字都没有何来构造方法。

5. 我们给匿名内部类传递参数的时候，若该形参在内部类中需要被使用，那么该形参必须要为final。也就是说：当所在的方法的形参需要被内部类里面使用时，该形参必须为final。

6. 使用匿名内部类时，我们必须是**继承一个类或者实现一个接口**，但是两者不可兼得，同时也只能继承一个类或者实现一个接口。

7. 匿名内部类中不能存在任何的静态成员变量和静态方法。

8. 匿名内部类为局部内部类，所以局部内部类的所有限制同样对匿名内部类生效。

9. 匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的所有抽象方法。

#### 静态内部类

Static可以修饰成员变量、方法、代码块，它还可以修饰内部类，**使用static修饰的内部类**我们称之为静态内部类，不过我们更喜欢称之为嵌套内部类。静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。没有这个引用就意味着：

1.  它的创建是**不需要依赖于外围类**的。

2.  它**不能使用任何外围类的非static成员变量和方法**。

```(java)
public class OuterClass {
    private String sex;
    public static String name = "pingxin";

    /**
     *静态内部类
     */
    static class InnerClass1{
        /* 在静态内部类中可以存在静态成员 */
        public static String _name1 = "pingxin_static";

        public void display(){
            /*
             * 静态内部类只能访问外围类的静态成员变量和方法
             * 不能访问外围类的非静态成员变量和方法
             */
            System.out.println("OutClass name :" + name);
        }
    }

    /**
     * 非静态内部类
     */
    class InnerClass2{
        /* 非静态内部类中不能存在静态成员 */
        public String _name2 = "pingxin_inner";
        /* 非静态内部类中可以调用外围类的任何成员,不管是静态的还是非静态的 */
        public void display(){
            System.out.println("OuterClass name：" + name);
        }
    }

    /**
     * @desc 外围类方法
     */
    public void display(){
        /* 外围类访问静态内部类：内部类. */
        System.out.println(InnerClass1._name1);
        /* 静态内部类 可以直接创建实例不需要依赖于外围类 */
        new InnerClass1().display();

        /* 非静态内部的创建需要依赖于外围类 */
        OuterClass.InnerClass2 inner2 = new OuterClass().new InnerClass2();
        /* 方位非静态内部类的成员需要使用非静态内部类的实例 */
        System.out.println(inner2._name2);
        inner2.display();
    }

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.display();
    }
}
//----------------
//Output:
//pingxin_static
//OutClass name :pingxin
//pingxin_inner
//OuterClass name：pingxin
```

#### 深入理解

**为什么成员内部类可以无条件访问外部类的成员？**

在此之前，我们已经讨论过了成员内部类可以无条件访问外部类的成员，那具体究竟是如何实现的呢？下面通过反编译字节码文件看看究竟。事实上，编译器在进行编译的时候，会将成员内部类单独编译成一个字节码文件，下面是Outter.java的代码：
```(java)
public class Outter {
    private Inner inner = null;
    public Outter() {

    }

    public Inner getInnerInstance() {
        if(inner == null)
            inner = new Inner();
        return inner;
    }

    protected class Inner {
        public Inner() {

        }
    }
}
```
编译之后出现两个字节码文件：Outter.class、Outter$Inner.class

```
$ javap -v Outter\$Inner.class
Classfile /home/hyp/programes/IdeaProjects/Learn/Outter$Inner.class
  Last modified 2019-4-11; size 304 bytes
  MD5 checksum 732ba89287828251572200e940699e56
  Compiled from "Outter.java"
public class Outter$Inner
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Fieldref           #3.#13         // Outter$Inner.this$0:LOutter;
   #2 = Methodref          #4.#14         // java/lang/Object."<init>":()V
   #3 = Class              #16            // Outter$Inner
   #4 = Class              #19            // java/lang/Object
   #5 = Utf8               this$0
   #6 = Utf8               LOutter;
   #7 = Utf8               <init>
   #8 = Utf8               (LOutter;)V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               SourceFile
  #12 = Utf8               Outter.java
  #13 = NameAndType        #5:#6          // this$0:LOutter;
  #14 = NameAndType        #7:#20         // "<init>":()V
  #15 = Class              #21            // Outter
  #16 = Utf8               Outter$Inner
  #17 = Utf8               Inner
  #18 = Utf8               InnerClasses
  #19 = Utf8               java/lang/Object
  #20 = Utf8               ()V
  #21 = Utf8               Outter
{
  final Outter this$0;
    descriptor: LOutter;
    flags: ACC_FINAL, ACC_SYNTHETIC

  public Outter$Inner(Outter);
    descriptor: (LOutter;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: putfield      #1                  // Field this$0:LOutter;
         5: aload_0
         6: invokespecial #2                  // Method java/lang/Object."<init>":()V
         9: return
      LineNumberTable:
        line 14: 0
        line 16: 9
}
SourceFile: "Outter.java"
InnerClasses:
     protected #17= #3 of #15; //Inner=class Outter$Inner of class Outter
```
可以看到有一句：`  final Outter this$0;`，这行是一个指向外部类对象的指针，看到这里想必大家豁然开朗了。也就是说编译器会**默认为成员内部类添加了一个指向外部类对象的引用**，那么这个引用是如何赋初值的呢？

下面接着看内部类的构造器：`  public Outter$Inner(Outter);`

从这里可以看出，**虽然我们在定义的内部类的构造器是无参构造器，编译器还是会默认添加一个参数，该参数的类型为指向外部类对象的一个引用，**所以成员内部类中的Outter this&0 指针便指向了外部类对象，因此可以在成员内部类中随意访问外部类的成员。从这里也**间接说明了成员内部类是依赖于外部类的**，如果没有创建外部类的对象，则无法对Outter this&0引用进行初始化赋值，也就无法创建成员内部类的对象了。

**为什么局部内部类和匿名内部类只能访问局部final变量？**

```
public class Test {
    public static void main(String[] args)  {

    }

    public void test(final int b) {
        final int a = 10;
        new Thread(){
            public void run() {
                System.out.println(a);
                System.out.println(b);
            };
        }.start();
    }
}
```
这段代码会被编译成两个class文件：Test.class和Test1.class。默认情况下，编译器会为匿名内部类和局部内部类起名为Outterx.class（x为正整数）。

上段代码中，如果把变量a和b前面的任一个final去掉，这段代码都编译不过。我们先考虑这样一个问题：

当test方法执行完毕之后，变量a的生命周期就结束了，而此时Thread对象的生命周期很可能还没有结束，那么在Thread的run方法中继续访问变量a就变成不可能了，但是又要实现这样的效果，怎么办呢？Java采用了 复制  的手段来解决这个问题。将这段代码的字节码反编译可以得到下面的内容：
```
...
{
  final int val$b;
    descriptor: I
    flags: ACC_FINAL, ACC_SYNTHETIC

  final Test this$0;
    descriptor: LTest;
    flags: ACC_FINAL, ACC_SYNTHETIC

  Test$1(Test, int);
    descriptor: (LTest;I)V
    flags:
    Code:
      stack=2, locals=3, args_size=3
         0: aload_0
         1: aload_1
         2: putfield      #1                  // Field this$0:LTest;
         5: aload_0
         6: iload_2
         7: putfield      #2                  // Field val$b:I
        10: aload_0
        11: invokespecial #3                  // Method java/lang/Thread."<init>":()V
        14: return
      LineNumberTable:
        line 8: 0

  public void run();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: bipush        10
         5: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
         8: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_0
        12: getfield      #2                  // Field val$b:I
        15: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
        18: return
      LineNumberTable:
        line 10: 0
        line 11: 8
        line 12: 18
}
SourceFile: "Test.java"
EnclosingMethod: #21.#22                // Test.test
InnerClasses:
     #6; //class Test$1
```
我们看到在run方法中有一条指令：

`bipush 10`

这条指令表示将操作数10压栈，表示使用的是一个本地局部变量。这个过程是在编译期间由编译器默认进行，如果这个变量的值在编译期间可以确定，则编译器默认会在匿名内部类（局部内部类）的常量池中添加一个内容相等的字面量或直接将相应的字节码嵌入到执行字节码中。这样一来，匿名内部类使用的变量是另一个局部变量，只不过值和方法中局部变量的值相等，因此和方法中的局部变量完全独立开。

我们看到匿名内部类Test$1的构造器含有两个参数，一个是指向外部类对象的引用，一个是int型变量，很显然，这里是将变量test方法中的形参a以参数的形式传进来对匿名内部类中的拷贝（变量a的拷贝）进行赋值初始化。

**也就说如果局部变量的值在编译期间就可以确定，则直接在匿名内部里面创建一个拷贝。如果局部变量的值无法在编译期间确定，则通过构造器传参的方式来对拷贝进行初始化赋值。**

从上面可以看出，在run方法中访问的变量a根本就不是test方法中的局部变量a。这样一来就解决了前面所说的 生命周期不一致的问题。但是新的问题又来了，既然在run方法中访问的变量a和test方法中的变量a不是同一个变量，当在run方法中改变变量a的值的话，会出现什么情况？

对，**会造成数据不一致性**，这样就达不到原本的意图和要求。为了解决这个问题，java编译器就限定必须将变量a限制为final变量，不允许对变量a进行更改（对于引用类型的变量，是不允许指向新的对象），这样数据不一致性的问题就得以解决了。

到这里，想必大家应该清楚为何 方法中的局部变量和形参都必须用final进行限定了。


**静态内部类有特殊的地方吗？**

从前面可以知道，静态内部类是不依赖于外部类的，也就说可以在不创建外部类对象的情况下创建内部类的对象。另外，静态内部类是不持有指向外部类对象的引用的，这个读者可以自己尝试反编译class文件看一下就知道了，是没有Outter this&0引用的。

**为什么在Java中需要内部类？**

总结一下主要有以下四点：

1. 每个内部类都能独立的继承一个接口的实现，所以无论外部类是否已经继承了某个(接口的)实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整，

2. 方便将存在一定逻辑关系的类组织在一起，又可以对外界隐藏。

3. 方便编写事件驱动程序

4. 方便编写线程代码

### 面试题

1. 静态嵌套类(Static Nested Class)和内部类（Inner Class）的不同？

   答：Static Nested Class是被声明为静态（static）的内部类，它可以不依赖于外部类实例被实例化：`OuterClass.InnerClass ic=new OuterClass.InnerClass()`

   而通常的内部类需要在外部类实例化后才能实例化，其语法看起来挺诡异的:

   ```java
   OuterClass oc=new OuterClass();
   OuterClass.InnerClass ic=oc.new InnerClass()
   ```

2. 下面的代码哪些地方会产生编译错误？

   ```java
   class Outer {
   
       class Inner {}
   
       public static void foo() { new Inner(); }
   
       public void bar() { new Inner(); }
   
       public static void main(String[] args) {
           new Inner();
       }
   }
   ```

   注意：Java中非静态内部类对象的创建要依赖其外部类对象，上面的面试题中foo和main方法都是静态方法，静态方法中没有this，也就是说没有所谓的外部类对象，因此无法创建内部类对象，如果要在静态方法中创建内部类对象，可以这样做：

   ```java
   new Outer().new Inner();
   ```

3. Java 中会存在内存泄漏吗，请简单描述。

   答：理论上Java因为有垃圾回收机制（GC）不会存在内存泄露问题（这也是Java被广泛使用于服务器端编程的一个重要原因）；

   然而在实际开发中，可能会存在无用但可达的对象，这些对象不能被GC回收，因此也会导致内存泄露的发生。例如Hibernate的Session（一级缓存）中的对象属于持久态，垃圾回收器是不会回收这些对象的，然而这些对象中可能存在无用的垃圾对象，如果不及时关闭（close）或清空（flush）一级缓存就可能导致内存泄露。下面例子中的代码也会导致内存泄露。

   ```java
   import java.util.Arrays;
   import java.util.EmptyStackException;
   
   public class MyStack<T> {
       private T[] elements;
       private int size = 0;
   
       private static final int INIT_CAPACITY = 16;
   
       public MyStack() {
           elements = (T[]) new Object[INIT_CAPACITY];
       }
   
       public void push(T elem) {
           ensureCapacity();
           elements[size++] = elem;
       }
   
       public T pop() {
           if(size == 0) 
               throw new EmptyStackException();
           return elements[--size];
       }
   
       private void ensureCapacity() {
           if(elements.length == size) {
               elements = Arrays.copyOf(elements, 2 * size + 1);
           }
       }
   }
   ```

   上面的代码实现了一个栈（先进后出（FILO））结构，乍看之下似乎没有什么明显的问题，它甚至可以通过你编写的各种单元测试。然而其中的pop方法却存在内存泄露的问题，当我们用pop方法弹出栈中的对象时，该对象不会被当作垃圾回收，即使使用栈的程序不再引用这些对象，因为栈内部维护着对这些对象的过期引用（obsolete reference）。在支持垃圾回收的语言中，内存泄露是很隐蔽的，这种内存泄露其实就是无意识的对象保持。如果一个对象引用被无意识的保留起来了，那么垃圾回收器不会处理这个对象，也不会处理该对象引用的其他对象，即使这样的对象只有少数几个，也可能会导致很多的对象被排除在垃圾回收之外，从而对性能造成重大影响，极端情况下会引发Disk Paging（物理内存与硬盘的虚拟内存交换数据），甚至造成OutOfMemoryError。

4. Anonymous Inner Class(匿名内部类)是否可以继承其它类？是否可以实现接口？

   答：可以继承其他类或实现其他接口，在Swing编程和Android开发中常用此方式来实现事件监听和回调。

5. 内部类可以引用它的包含类（外部类）的成员吗？有没有什么限制？

   答：一个内部类对象可以访问创建它的外部类对象的成员，包括私有成员。

   



