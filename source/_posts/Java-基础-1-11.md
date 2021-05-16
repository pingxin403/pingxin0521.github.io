---
title: Java  泛型和资源文件读取
date: 2019-04-06 16:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 泛型

泛型，即“参数化类型”。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

泛型的本质是为了参数化类型（**在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型**）。也就是说在泛型使用过程中操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。
<!--more-->

 - 适用于多种数据类型执行相同的代码
 - 泛型中的类型在使用时指定
 - 泛型归根到底就是“模版”

优点：使用泛型时，在实际使用之前类型就已经确定了，不需要强制类型转换。

**例子**

```java
List arrayList = new ArrayList();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    System.out.println("item = " + item);
}
```
毫无疑问，程序的运行结果会以崩溃结束：
```
java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```
ArrayList可以存放任意类型，例子中添加了一个String类型，添加了一个Integer类型，再使用时都以String的方式使用，因此程序崩溃了。为了解决类似这样的问题（在编译阶段就可以解决），泛型应运而生。

我们将第一行声明初始化list的代码更改一下，编译器会在编译阶段就能够帮我们发现类似这样的问题。
```
List<String> arrayList = new ArrayList<String>();
...
//arrayList.add(100); 在编译阶段，编译器就会报错
```
**特性**
泛型只在编译阶段有效。看下面的代码：

```java
List<String> stringArrayList = new ArrayList<String>();
List<Integer> integerArrayList = new ArrayList<Integer>();

Class classStringArrayList = stringArrayList.getClass();
Class classIntegerArrayList = integerArrayList.getClass();

if(classStringArrayList.equals(classIntegerArrayList)){
     System.out.println("泛型测试：类型相同");
}
```
输出结果：泛型测试: 类型相同。

通过上面的例子可以证明，**在编译之后程序会采取去泛型化的措施**。也就是说Java中的泛型，**只在编译阶段有效**。在编译过程中，正确检验泛型结果后，会将泛型的相关信息**擦除**，并且**在对象进入和离开方法的边界处添加类型检查和类型转换的方法**。也就是说，**泛型信息不会进入到运行时阶段**。

对此总结成一句话：**泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。**

#### 泛型的使用

泛型有三种使用方式，分别为：**泛型类、泛型接口、泛型方法**

**泛型字母**

 - 形式类型参数（formal type parameters）即泛型字母
 - 命名泛型字母可以随意指定，尽量使用单个的大写字母（有时候多个泛型类型时会加上数字，比如T1，T2）
 - 常见字母（见名知意）:T Type、K V Key Value、E Element
 - 当类被使用时，会使用具体的实际类型参数（actual type argument）代替

泛型类：只能用在成员变量上，只能使用引用类型

泛型接口：只能用在抽象方法上

泛型方法：返回值前面加上 T


**自定义泛型类：**

泛型类型用于类的定义中，被称为泛型类。通过泛型可以完成对一组类的操作对外开放相同的接口。最典型的就是各种容器类，如：List、Set、Map。

泛型类的最基本写法（这么看可能会有点晕，会在下面的例子中详解）：

```
class 类名称 <泛型标识：可以随便写任意标识号，标识指定的泛型的类型>{
  private 泛型标识 /*（成员变量类型）*/ var;
  .....

  }
}
```

一个最普通的泛型类：

```java
/**
 * 自定义泛型类
 *
 * 定义"模版"的时候，泛型用泛型字母：T 代替
 * 在使用的时候指定实际类型
 *
 * @author Administrator
 * @param <T>
 */
public class Student<T> {

  private T javase;

  //private static T javaee;   // 泛型不能使用在静态属性上

  public Student() {
  }

  public Student(T javase) {
    this();
    this.javase = javase;
  }

  public T getJavase() {
    return javase;
  }

  public void setJavase(T javase) {
    this.javase = javase;
  }

}
/**
 * 自定义泛型的使用
 * 在声明时指定具体的类型
 * 不能为基本类型
 *
 */
class Demo02 {
  public static void main(String[] args) {
    //Student<int>  Student = new Student<int>(); //不能为基本类型，编译时异常
    //传入的实参类型需与泛型的类型参数类型相同，即为Integer.
    Student<Integer> student = new Student<Integer>();
    student.setJavase(85);
    System.out.println(student.getJavase());  
  }
}
```
定义的泛型类，就一定要传入泛型类型实参么？并不是这样，**在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。**
```
Student student = new Student("111111");
Student student1 = new Student(4444);
Student student2 = new Student(55.55);
Student student3 = new Student(false);

System.out.println("泛型测试:key is " + student.getJavase());
System.out.println("泛型测试:key is " + student1.getJavase());
System.out.println("泛型测试:key is " + student2.getJavase());
System.out.println("泛型测试:key is " + student3.getJavase());


//结果
D/泛型测试: key is 111111
D/泛型测试: key is 4444
D/泛型测试: key is 55.55
D/泛型测试: key is false
```


注意：
- 泛型的类型参数**只能是类类型**，不能是简单类型。
- **不能对确切的泛型类型**使用instanceof操作。如下面的操作是非法的，编译时会出错。
```java
if(ex_num instanceof Student<Number>){ }
```

**自定义泛型接口**

**泛型接口与泛型类的定义及使用基本相同**。泛型接口常被用在各种类的生产器中，可以看一个例子：

```java
//定义一个泛型接口
// 接口中泛型字母只能使用在方法中，不能使用在全局常量中
public interface Generator<T> {
    public T next();
}
```
当实现泛型接口的类，未传入泛型实参时：
```java
/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 * 即：class FruitGenerator<T> implements Generator<T>{
 * 如果不声明泛型，如：class FruitGenerator implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
```
当实现泛型接口的类，传入泛型实参时：
```java
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

**非泛型类中定义泛型方法**

```java
import java.io.Closeable;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.List;


/**
 * 非泛型类中定义泛型方法
 *
 */
public class Method {

  // 泛型方法，在返回类型前面使用泛型字母
  public static <T> void test1(T t){
    System.out.println(t);
  }

  // T 只能是list 或者list 的子类
  public static <T extends List> void test2(T t){
    t.add("aa");
  }

  // T... 可变参数   --->   T[]
  public static <T extends Closeable> void test3(T...a) {
    for (T temp : a) {
     try {
       if (null != temp) {
         temp.close();
       }
     } catch (Exception e) {
       e.printStackTrace();
     }

    }
  }

  public static void main(String[] args) throws FileNotFoundException {
    test1("java 是门好语言");
    test3(new FileInputStream("a.txt"));
  }
}
```

**泛型的继承**

```java
/**
 * 泛型继承
 *
 * 保留父类泛型 ----》泛型子类
 * 不保留父类泛型 -----》子类按需实现
 *
 * 子类重写父类的方法，泛型类型随父类而定 子类使用父类的属性，该属性类型随父类定义的泛型
 *
 *
 * @param <T1>
 * @param <T2>
 */
public abstract class Father<T1, T2> {
  T1 age;

  public abstract void test(T2 name);
}

// 保留父类泛型 ----》泛型子类
// 1）全部保留
class C1<T1, T2> extends Father<T1, T2> {

  @Override
  public void test(T2 name) {

  }
}

// 2) 部分保留
class C2<T1> extends Father<T1, Integer> {

  @Override
  public void test(Integer name) {

  }
}

// 不保留父类泛型 -----》子类按需实现
// 1)具体类型
class C3 extends Father<String, Integer> {

  @Override
  public void test(Integer name) {

  }
}

// 2)没有具体类型
// 泛型擦除：实现或继承父类的子类，没有指定类型，类似于Object
class C4 extends Father {

  @Override
  public void test(Object name) {

  }

}
```

**泛型擦除**

```java

/**
 * 泛型擦除
 * 类似于Object,不等于Object
 *
 */
public class Demo03 {

  public static void test(Student<Integer> student){
    student.setJavase(100);
  }

  public static void main(String[] args) {
    // 泛型擦除
    Student student = new Student();
    test(student);

    Student<Object> student2 = new Student<Object>();
    //test(student2);  //编译异常
  }
}
```


#### 通配符
`Generic<Integer>`不能被看作为`Generic<Number>`的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

```
T、K、V、E 等泛型字母为有类型，类型参数赋予具体的值
？未知类型 类型参数赋予不确定值，任意类型
只能用在声明类型、方法参数上，不能用在定义泛型类上
```

**上限（extends）**

指定的类必须是继承某个类，或者实现了某个接口（不是implements），即<=
```
    ? extends List
```

**下限（super）**
即父类或本身
```
    ？ super List
```

**泛型嵌套**

从外向里取

```java
import java.util.Map.Entry;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

/**
 * 泛型嵌套
 *
 */
public class Demo05 {
  public static void main(String[] args) {
    Student2<String> student = new Student2<String>();
    student.setScore("优秀");
    System.out.println(student.getScore());

    //泛型嵌套
    School<Student2<String>> school = new School<Student2<String>>();
    school.setStu(student);

    String s = school.getStu().getScore(); //从外向里取
    System.out.println(s);

    // hashmap 使用了泛型的嵌套
    Map<String, String> map =  new HashMap<String,String>();
    map.put("a", "张三");
    map.put("b", "李四");
    Set<Entry<String, String>> set = map.entrySet();
    for (Entry<String, String> entry : set) {
     System.out.println(entry.getKey()+":"+entry.getValue());
    }    
  }
}
```

泛型没有多态，泛型没有数组

```java
import java.util.ArrayList;
import java.util.List;

/**
* 泛型没有多态
* 泛型没有数组
* JDK1.7对泛型的简化
*
*/
public class Demo06 {

 public static void main(String[] args) {
   Fruit fruit = new Apple();  // 多态，父类的引用指向子类的对象
   //List<Fruit> list = new ArrayList<Apple>(); //泛型没有多态
   List<? extends Fruit> list = new ArrayList<Apple>();

   //泛型没有数组
   //Fruit<String>[] fruits = new Fruit<String>[10];

   //ArrayList底层是一个Object[],它放数据的时候直接放，取数据的时候强制类型转化为泛型类型
   /*public boolean add(E e) {
         ensureCapacityInternal(size + 1);  // Increments modCount!!
         elementData[size++] = e;
         return true;
     }*/

   /*E elementData(int index) {
         return (E) elementData[index];
     }*/
   //JDK1.7泛型的简化,1.6编译通不过
   List<Fruit> list2 = new ArrayList<>();
 }
}
```


#### java泛型中T和？和有什么区别

**T 代表一种类型**

加在类上:`class SuperClass<A>{}`

加在方法上:
```
public <T>void fromArrayToCollection(T[] a, Collection<T> c){}
```
方法上的<T\>代表括号里面要用到泛型参数，若类中传了泛型，此处可以不传，调用类型上面的泛型参数，前提是方法中使用的泛型与类中传来的泛型一致。
```java
class People<T>{

public void show(T a) {

   }

}
```
T extends T2 指传的参数为T2或者T2的子类型。

**?是通配符,泛指所有类型**

一般用于定义一个引用变量,这么做的好处是,如下所示,定义一个sup的引用变量，就可以指向多个对象。
```
SuperClass<?> sup = new SuperClass<String>("lisi");
sup = new SuperClass<People>(new People());
sup = new SuperClass<Animal>(new Animal());
```
若不用?,用固定的类型的话，则：
```
SuperClass<String> sup1 = new SuperClass<String>("lisi");
SuperClass<People> sup2 = new SuperClass<People>("lisi");
SuperClass<Animal> sup3 = new SuperClass<Animal>("lisi");
```
这就是?通配符的好处。
```
? extends T 指T类型或T的子类型

? super T   指T类型或T的父类型
```

这个两个一般也是和?一样用在定义引用变量中，但是传值范围不一样，T和？运用的地方有点不同,?是定义在引用变量上,T是类上或方法上，如果有泛型方法和非泛型方法,都满足条件,会执行非泛型方法。


```java
public void show(String s){
      System.out.println("1");
   }
   @Override
   public void show(T a) {
      System.out.println("2");
   }
```
1. 在整个类中只有一处使用了泛型,使用时注意加了泛型了参数不能调用与参数类型有关的方法比如“+”，比如打印出任意参数化类型集合中的所有内容，就适合用通配符泛型<?>
```java
public static void printCollecton(Collection <?> collection)
{
for(Object obj: collection)
{
System.out.println(obj);
}
}
```
2. 当一个类型变量用来表达两个参数之间或者参数与返回值之间的关系时，即统一各类型变量在方法签名的两处被使用，或者类型变量在方法体代码中也被使用而不仅仅在签名的时候使用，这是应该用自定义泛型<T\>。泛型方可以调用一些时间类型的方法。比如集合的add方法。
```java
public static <T> T autoConvertType(T obj)
{
     return(T)obj;
}
```

泛型三种：
```
[1]ArrayList<T> al=new ArrayList<T>();指定集合元素只能是T类型
[2]ArrayList<?> al=new ArrayList<?>();集合元素可以是任意类型，这种没有意义，一般是方法中，只是为了说明用法
[3]ArrayList<? extends E> al=new ArrayList<? extends E>();
```
泛型的限定：

```
? extends E:接收E类型或者E的子类型。
？super E:接收E类型或者E的父类型。
```

java泛型的两种用法：List<T\>是泛型方法，List<?\>是限制通配符

List<T\>一般有两种用途：

1. 定义一个通用的泛型方法。

```java
public interface Dao{
  List<T> getList(){};
}

List<String> getStringList(){
  return dao.getList();//dao是一个实现类实例
}

List<Integer> getIntList(){
  return dao.getList();
}
```
上面接口的getList方法如果定义成List<?> ，后面就会报错。

2、限制方法的参数之间或参数和返回结果之间的关系。

```java
List<T> getList<T param1,T param2>
```

这样可以限制返回结果的类型以及两个参数的类型一致。List<?>一般就是在泛型起一个限制作用。

```java
public Class Fruit(){}

public Class Apple extends Fruit(){}

public void test(? extends Fruit){};

test(new Fruit());
test(new Apple());
test(new String()); //这个就会报错,
//参数必须是Fruit或其子类。
```

**“<T\>"和"<?>"，首先要区分开两种不同的场景：**

```
第一，声明一个泛型类或泛型方法。
第二，使用泛型类或泛型方法。
类型参数“<T>”主要用于第一种，声明泛型类或泛型方法。
无界通配符“<?>”主要用于第二种，使用泛型类或泛型方法
```

#### 泛型数组

```java
  public GenericArrayWithTypeToken(Class<T> type,int sz) {
       T[]  array = (T[]) Array.newInstance(type, sz);
        //array = (T[])new Object[sz]; 会出现类型转换异常
    }
```

java创建泛型数组可以通过Array类的newInstance方法创建,包含两个参数,第一个是数组类型,第二个是长度.

如果使用T[ ]创建数组会编译错误.

如果使用 (T[ ])new Object[SIZE] 虽然编译器不会出错,但是运行期会出错,毕竟创建的是Object数组,array实际指向的是Object数组,无法强转化为T

### 资源文件

java开发中，常见的resource文件有：.xml,.properties,.txt文件等，后台开发中经常用到读取资源文件，处理业务逻辑，然后返回结果。

获取资源文件的方法说明

```
getResource()返回:URL                     
getResourceAsStream ()   返回的是inputstream，需要定义一个InputStream接收            
//Class.getResource和Class.getResourceAsStream在使用时，路径选择上是一样的。
相当于你用getResource()取得File文件后，再new InputStream(file)一样的结果.
```

有4种读取文件URL的方法：

1. 通过本类的class类的getResource方法。path不以’/'开头时，默认是从此类所在的包下取资源；path  以’/'开头时，则是从ClassPath根下获取；
2. 通过本类的ClassLoader的getResource方法。 path不能以’/'开头，path是从ClassPath根下获取；所以可以认为：`类名.class.getResource("/") == 类名.class.getClassLoader().getResource("")`
3. 通过ClassLoader的getSystemResource(),路径和2一致
4. 通过Thread方式,路径和2一致(推荐此种)，`Thread.currentThread().getContextClassLoader()`。

WEB程序，里面的jar、resources都是由Tomcat内部来加载的，所以你在代码中动态加载jar、资源文件的时候，首先应该是使用Thread.currentThread().getContextClassLoader()。如果你使用Test.class.getClassLoader()，可能会导致和当前线程所运行的类加载器不一致（因为Java天生的多线程）

web程序中的资源文件会出现加载问题，不能通过上述方法获得，可通过下面进行获取，假设是资源文件夹下的/db.properties

```java
String path = DruidUtils.class.getClassLoader().getResource("/db.properties").getPath();
properties.load(new FileInputStream(path));
```



### 面试

1. Java的泛型是如何工作的 ? 什么是类型擦除 ?

   这是一道更好的泛型面试题。泛型是通过类型擦除来实现的，编译器在编译时擦除了所有类型相关的信息，所以在运行时不存在任何类型相关的信息。例如 List<String\>在运行时仅用一个List来表示。这样做的目的，是确保能和Java 5之前的版本开发二进制类库进行兼容。你无法在运行时访问到类型参数，因为编译器已经把泛型类型转换成了原始类型。根据你对这个泛型问题的回答情况，你会 得到一些后续提问，比如为什么泛型是由类型擦除来实现的或者给你展示一些会导致编译器出错的错误泛型代码。

   - 类型检查：在生成字节码之前提供类型检查

   - 类型擦除：所有类型参数都用他们的限定类型替换，包括类、变量和方法（类型擦除）

   - 如果类型擦除和多态性发生了冲突时，则在子类中生成桥方法解决

   - 如果调用泛型方法的返回类型被擦除，则在调用该方法时插入强制类型转换 

   类型擦除：

   所有类型参数都用他们的限定类型替换，比如T->Object   ? extends  BaseClass->BaseClass

2. 什么是泛型中的限定通配符和非限定通配符 ?

   这是另一个非常流行的Java泛型面试题。限定通配符对类型进行了限制。有两种限定通配符，一种是`<? extends T>`它通过确保类型必须是T的子类来设定类型的上界，另一种是`<? super T>`它通过确保类型必须是T的父类来设定类型的下界。泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。另一方面`<?>`表 示了非限定通配符，因为`<?>`可以用任意类型来替代。

   List<? extends T\>可以接受任何继承自T的类型的List，而List<? super T\>可以接受任何T的父类构成的List。例如List<? extends Number\>可以接受List<Integer\>或List<Float\>。

3. Java中的泛型是什么 ? 使用泛型的好处是什么?JDK 不同版本的泛型有什么区别？

   泛型是 Java SE 1.5 的新特性，泛型的本质是参数化类型，这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。在 Java SE 1.5 之前没有泛型的情况的下只能通过对类型 Object 的引用来实现参数的任意化，其带来的缺点是要做显式强制类型转换，而这种强制转换编译期是不做检查的，容易把问题留到运行时，所以 泛型的好处是在编译时检查类型安全，并且所有的强制转换都是自动和隐式的，提高了代码的重用率，避免在运行时出现 ClassCastException。

   好处：

   - 类型安全，提供编译期间的类型检测

   - 前后兼容

   - 泛化代码,代码可以更多的重复利用

   - 性能较高，用GJ(泛型JAVA)编写的代码可以为java编译器和虚拟机带来更多的类型信息，这些信息对java程序做进一步优化提供条件。

   JDK 1.5 引入了泛型来允许强类型在编译时进行类型检查；JDK 1.7 泛型实例化类型具备了自动推断能力，譬如 `List<String> list = new ArrayList<String>();` 可以写成 `List<String> llist = new ArrayList<>();` 了，JDK 具备自动推断能力。下面几种写法可以说是不同版本的兼容性了：

   ```
   //JDK 1.5 推荐使用的写法
   List<String> list =new ArrayList<String>();
   //JDK 1.7 推荐使用的写法
   List<String> list =new ArrayList<>();
   //可以使用，但不推荐，是为了兼容老版本，IDE 会提示警告，可以通过注解屏蔽警告
   List<String> list =new ArrayList();
   //可以使用，但不推荐，是为了兼容老版本，IDE 会提示警告，可以通过注解屏蔽警告
   List list =new ArrayList<String>();
   ```

4. 编写一段泛型程序来实现LRU缓存

   对于喜欢Java编程的人来说这相当于是一次练习。给你个提示，LinkedHashMap可以用来实现固定大小的LRU缓存，当LRU缓存已经满 了的时候，它会把最老的键值对移出缓存。LinkedHashMap提供了一个称为removeEldestEntry()的方法，该方法会被put() 和putAll()调用来删除最老的键值对。当然，如果你已经编写了一个可运行的JUnit测试，你也可以随意编写你自己的实现代码。

5. Java 泛型类、泛型接口、泛型方法有什么区别？

   泛型类是在实例化类的对象时才能确定的类型，其定义譬如 `class Test<T> {}`，在实例化该类时必须指明泛型 T 的具体类型。泛型接口与泛型类一样，其定义譬如 `interface Generator<E> { E dunc(E e); }`。

   泛型方法所在的类可以是泛型类也可以是非泛型类，是否拥有泛型方法与所在的类无关，所以在我们应用中应该尽可能使用泛型方法，不要放大作用空间，尤其是在 static 方法时 static 方法无法访问泛型类的类型参数，所以更应该使用泛型的 static 方法（声明泛型一定要写在 static 后返回值类型前）。泛型方法的定义譬如 `<T> void func(T val) {}`。

6. Java 如何优雅的实现元组？

   元组其实是关系数据库中的一个学术名词，一条记录就是一个元组，一个表就是一个关系，纪录组成表，元组生成关系，这就是关系数据库的核心理念。很多语言天生支持元组，譬如 Python 等，在语法本身支持元组的语言中元组是用括号表示的，如 (int, bool, string) 就是一个三元组类型，不过在 Java、C 等语言中就比较坑爹，语言语法本身不具备这个特性，所以在 Java 中我们如果想优雅实现元组就可以借助泛型类实现，如下是一个三元组类型的实现：

   ```java
   Triplet<A,B,C>{
   
      private A a;
   
       private B a;
   
       private C a;
   
       public Triplet(A a,B b,C c){
   
           this.a =a;
   
           this.b =b;
   
           this.c =c;
   
   }
   
   }
   ```

   

7. 你可以把List<String\>传递给一个接受List<Object\>参数的方法吗？

   对任何一个不太熟悉泛型的人来说，这个Java泛型题目看起来令人疑惑，因为乍看起来String是一种Object，所以 List<String\>应当可以用在需要List<Object\>的地方，但是事实并非如此。真这样做的话会导致编译错误。如 果你再深一步考虑，你会发现Java这样做是有意义的，因为List<Objec\t>可以存储任何类型的对象包括String, Integer等等，而List<String\>却只能用来存储Strings。

   ```
   List<Object> objectList;
   
   List<String> stringList;
   
   objectList = stringList; //compilation error incompatible types
   ```

8. Array中可以用泛型吗

   这可能是Java泛型面试题中最简单的一个了，当然前提是你要知道Array事实上并不支持泛型，这也是为什么Joshua Bloch在Effective Java一书中建议使用List来代替Array，因为List可以提供编译期的类型安全保证，而Array却不能。

9. 如何阻止Java中的类型未检查的警告?

   如果你把泛型和原始类型混合起来使用，例如下列代码，Java 5的javac编译器会产生类型未检查的警告，例如

   ```
   List<String> rawList = new ArrayList()
   ```

   注意: Hello.java使用了未检查或称为不安全的操作;

   这种警告可以使用@SuppressWarnings(“unchecked”)注解来屏蔽。

10. Java中List<Object\>和原始类型List之间的区别?

    原始类型和带参数类型<Object\>之间的主要区别是，在编译时编译器不会对原始类型进行类型安全检查，却会对带参数的类型进行检 查，通过使用Object作为类型，可以告知编译器该方法可以接受任何类型的对象，比如String或Integer。这道题的考察点在于对泛型中原始类 型的正确理解。它们之间的第二点区别是，**你可以把任何带参数的类型传递给原始类型List，但却不能把List<String\>传递给接受 List<Object\>的方法，因为会产生编译错误。**

11. Java中List<?\>和List<Object\>之间的区别是什么?

    这道题跟上一道题看起来很像，实质上却完全不同。`List<?>` 是一个未知类型的List，而`List<Object>`  其实是任意类型的List。你可以把`List<String>`, ` List<Integer>`赋值给`List<?>`，却不能把`List<String>`赋值给  `List<Object>`。     

    ```java
    List<?> listOfAnyType;
    
    List<Object> listOfObject = new ArrayList<Object>();
    
    List<String> listOfString = new ArrayList<String>();
    
    List<Integer> listOfInteger = new ArrayList<Integer>();
    
    listOfAnyType = listOfString; //legal
    
    listOfAnyType = listOfInteger; //legal
    
    listOfObjectType = (List<Object>) listOfString; //compiler error – in-convertible types
    ```

12. List<String\>和原始类型List之间的区别.

    该题类似于“原始类型和带参数类型之间有什么区别”。带参数类型是类型安全的，而且其类型安全是由编译器保证的，但原始类型List却不是类型安全 的。你不能把String之外的任何其它类型的Object存入String类型的List中，而你可以把任何类型的对象存入原始List中。使用泛型的 带参数类型你不需要进行类型转换，但是对于原始类型，你则需要进行显式的类型转换。

    ```java
    List listOfRawTypes = new ArrayList();
    
    listOfRawTypes.add(“abc”);
    
    listOfRawTypes.add(123); //编译器允许这样 – 运行时却会出现异常
    
    String item = (String) listOfRawTypes.get(0); //需要显式的类型转换
    
    item = (String) listOfRawTypes.get(1); //抛ClassCastException，因为Integer不能被转换为String
    
    List<String> listOfString = new ArrayList();
    
    listOfString.add(“abcd”);
    
    listOfString.add(1234); //编译错误，比在运行时抛异常要好
    
    item = listOfString.get(0); //不需要显式的类型转换 – 编译器自动转换
    ```

13. C++模板和java泛型之间有何不同？

    java泛型实现根植于“类型消除”这一概念。当源代码被转换为Java虚拟机字节码时，这种技术会消除参数化类型。有了Java泛型，我们可以做的事情也并没有真正改变多少；他只是让代码变得漂亮些。鉴于此，Java泛型有时也被称为“语法糖”。

    这和 C++模板截然不同。在 C++中，模板本质上就是一套宏指令集，只是换了个名头，编译器会针对每种类型创建一份模板代码的副本。

    由于架构设计上的差异，Java泛型和C++模板有很多不同点：

    - C++模板可以使用int等基本数据类型。Java则不行，必须转而使用Integer。

    - 在Java中，可以将模板的参数类型限定为某种特定类型。

    - 在C++中，类型参数可以实例化，但java不支持。

    - 在Java中，类型参数不能用于静态方法(?)和变量，因为它们会被不同类型参数指定的实例共享。在C++，这些类时不同的，因此类型参数可以用于静态方法和静态变量。

    - 在Java中，不管类型参数是什么，所有的实例变量都是同一类型。类型参数会在运行时被抹去。在C++中，类型参数不同，实例变量也不同。

### 参考

1. [java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一](https://www.cnblogs.com/coprince/p/8603492.html)
