---
title: Java  Lambda、Stream和函数式编程
date: 2019-05-25 18:18:59
tags:
 - Java
 - Lambda
categories:
 - Java
 - 高级
---

### 方法引用

**方法引用**是用来直接访问类或者实例的已经存在的方法或者构造方法。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。

<!--more-->

当Lambda表达式中只是执行一个方法调用时，不用Lambda表达式，直接通过方法引用的形式可读性更高一些。方法引用是一种更简洁易懂的Lambda表达式。

注意方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。

简单地说，就是一个Lambda表达式。在Java 8中，我们会使用Lambda表达式创建匿名方法，但是有时候，我们的Lambda表达式可能仅仅调用一个已存在的方法，而不做任何其它事，对于这种情况，通过一个方法名字来引用这个已存在的方法会更加清晰，Java 8的方法引用允许我们这样做。方法引用是一个更加紧凑，易读的Lambda表达式，注意方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号"::"。

**四种方法引用类型**

方法引用的标准形式是：`类名::方法名`。（**注意：只需要写方法名，不需要写括号**）

有以下四种形式的方法引用：

| **类型**                         | **示例**                             |
| -------------------------------- | ------------------------------------ |
| 引用静态方法                     | ContainingClass::staticMethodName    |
| 引用某个对象的实例方法           | containingObject::instanceMethodName |
| 引用某个类型的任意对象的实例方法 | ContainingType::methodName           |
| 引用构造方法                     | ClassName::new                       |

示例：

```
public class Person {
    public enum Sex{
        MALE,FEMALE
    }

    String name;
    LocalDate birthday;
    Sex gender;
    String emailAddress;

    public String getEmailAddress() {
        return emailAddress;
    }

    public Sex getGender() {
        return gender;
    }

    public LocalDate getBirthday() {
        return birthday;
    }

    public String getName() {
        return name;
    }

    public static int compareByAge(Person a,Person b){
        return a.birthday.compareTo(b.birthday);
    }

}


```

引用静态方法

```

 Person [] persons = new Person[10];

//使用匿名类
Arrays.sort(persons, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return o1.birthday.compareTo(o2.birthday);
            }
 });

//使用lambda表达式
Arrays.sort(persons, (o1, o2) -> o1.birthday.compareTo(o2.birthday));

//使用lambda表达式和类的静态方法
Arrays.sort(persons, (o1, o2) -> Person.compareByAge(o1,o2));

//使用方法引用
//引用的是类的静态方法
Arrays.sort(persons, Person::compareByAge);
```

引用对象的实例方法

```
class ComparisonProvider{
            public int compareByName(Person a,Person b){
                return a.getName().compareTo(b.getName());
            }

            public int compareByAge(Person a,Person b){
                return a.getBirthday().compareTo(b.getBirthday());
            }
        }

ComparisonProvider provider = new ComparisonProvider();

//使用lambda表达式
//对象的实例方法
Arrays.sort(persons,(a,b)->provider.compareByAge(a,b));

//使用方法引用
//引用的是对象的实例方法
Arrays.sort(persons, provider::compareByAge);
```

引用类型对象的实例方法

```
String[] stringsArray = {"Hello","World"};

//使用lambda表达式和类型对象的实例方法
Arrays.sort(stringsArray,(s1,s2)->s1.compareToIgnoreCase(s2));

//使用方法引用
//引用的是类型对象的实例方法
Arrays.sort(stringsArray, String::compareToIgnoreCase);
```

引用构造方法

```
public static <T, SOURCE extends Collection<T>, DEST extends Collection<T>>
    DEST transferElements(SOURCE sourceColletions, Supplier<DEST> colltionFactory) {
        DEST result = colltionFactory.get();
        for (T t : sourceColletions) {
            result.add(t);
        }
        return result;
    }
...
 
final List<Person> personList = Arrays.asList(persons);

//使用lambda表达式
Set<Person> personSet = transferElements(personList,()-> new HashSet<>());

//使用方法引用
//引用的是构造方法
Set<Person> personSet2 = transferElements(personList, HashSet::new); 
```

### Lambda

Java8最值得学习的特性就是Lambda表达式和Stream API，如果有python或者javascript的语言基础，对理解Lambda表达式有很大帮助，因为Java正在将自己变的更高（Sha）级（Gua），更人性化。--------可以这么说lambda表达式其实就是实现SAM接口的语法糖。

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

lambda表达式专门针对只有一个方法的接口（即函数式接口），Comparator就是一个函数式接口

```
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

@FunctionalInterface的作用就是标识一个接口为函数式接口，此时Comparator里只能有一个抽象方法，由编译器进行判定。

lambda写的好可以极大的减少代码冗余，同时可读性也好过冗长的内部类，匿名类。

示例：

使用() -> {} 替代匿名类：

```java
//Before Java 8:
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Before Java8 ");
    }
}).start();

//Java 8 way:
new Thread( () -> System.out.println("In Java8!") ).start();
```

你可以使用 下面语法实现Lambda:

```
(params) -> expression
(params) -> statement
(params) -> { statements; }
```

如果你的方法并不改变任何方法参数，比如只是输出，那么可以简写如下：

```
() -> System.out.println("Hello Lambda Expressions");
```

如果你的方法接受两个方法参数，如下：

```
(int even, int odd) -> even + odd
```

以下是lambda表达式的重要特征:

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。 	
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。 	
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。 	
-  **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。 	

Lambda 表达式的简单例子:

```java

public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```

 执行以上脚本，输出结果为：

```bash
$ javac Java8Tester.java 
$ java Java8Tester
10 + 5 = 15
10 - 5 = 5
10 x 5 = 50
10 / 5 = 2
Hello Runoob
Hello Google
```

使用 Lambda 表达式需要注意以下两点：

-  Lambda 表达式主要用来定义行内执行的方法类型接口，例如，一个简单方法接口。在上面例子中，我们使用各种类型的Lambda表达式来定义MathOperation接口的方法。然后我们定义了sayMessage的执行。
-  Lambda 表达式免去了使用匿名方法的麻烦，并且给予Java简单但是强大的函数化的编程能力。

#### 变量作用域

lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

```java

public class Java8Tester {
 
   final static String salutation = "Hello! ";
   
   public static void main(String args[]){
      GreetingService greetService1 = message -> 
      System.out.println(salutation + message);
      greetService1.sayMessage("Runoob");
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
}

```

执行以上脚本，输出结果为：

```bash
$ javac Java8Tester.java 
$ java Java8Tester
Hello! Runoob
```

我们也可以直接在 lambda 表达式中访问外层的局部变量：

```java

public class Java8Tester {
    public static void main(String args[]) {
        final int num = 1;
        Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
        s.convert(2);  // 输出结果为 3
    }
 
    public interface Converter<T1, T2> {
        void convert(int i);
    }
}

```

lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）

```java
int num = 1;  
Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
s.convert(2);
num = 5;  
//报错信息：Local variable num defined in an enclosing scope must be final or effectively 
```

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```java
String first = "";  
Comparator<String> comparator = (first, second) -> Integer.compare(first.length(), second.length());  //编译会出错 
```

#### 示例

1. 使用Java 8 lambda表达式进行事件处理

   如果你用过Swing API编程，你就会记得怎样写事件监听代码。这又是一个旧版本简单匿名类的经典用例，但现在可以不这样了。你可以用lambda表达式写出更好的事件监听代码，如下所示：

   ```java
   // Java 8之前：
   JButton show =  new JButton("Show");
   show.addActionListener(new ActionListener() {
       @Override
       public void actionPerformed(ActionEvent e) {
       System.out.println("Event handling without lambda expression is boring");
       }
   });
   
   // Java 8方式：
   show.addActionListener((e) -> {
       System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
   });
   ```

   Java开发者经常使用匿名类的另一个地方是为 Collections.sort() 定制 [Comparator](http://javarevisited.blogspot.sg/2014/01/java-comparator-example-for-custom.html)。在Java 8中，你可以用更可读的lambda表达式换掉丑陋的匿名类。我把这个留做练习，应该不难，可以按照我在使用lambda表达式实现 [Runnable](http://javarevisited.blogspot.sg/2012/01/difference-thread-vs-runnable-interface.html) 和 ActionListener 的过程中的套路来做。

2. 使用lambda表达式对列表进行迭代

   如果你使过几年Java，你就知道针对集合类，最常见的操作就是进行迭代，并将业务逻辑应用于各个元素，例如处理订单、交易和事件的列表。由于Java是命令式语言，Java 8之前的所有循环代码都是顺序的，即可以对其元素进行并行化处理。如果你想做并行过滤，就需要自己写代码，这并不是那么容易。

   通过引入lambda表达式和默认方法，将做什么和怎么做的问题分开了，这意味着Java集合现在知道怎样做迭代，并可以在API层面对集合元素进行并行处理。下面的例子里，我将介绍如何在使用lambda或不使用lambda表达式的情况下迭代列表。你可以看到列表现在有了一个 forEach()  方法，它可以迭代所有对象，并将你的lambda代码应用在其中。

   ```java
   // Java 8之前：
   List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
   for (String feature : features) {
       System.out.println(feature);
   }
   
   // Java 8之后：
   List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
   features.forEach(n -> System.out.println(n));
    
   // 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
   // 看起来像C++的作用域解析运算符
   features.forEach(System.out::println);
   ```

   列表循环的最后一个例子展示了如何在Java 8中使用方法引用（method reference）。你可以看到C++里面的双冒号、范围解析操作符现在在Java 8中用来表示方法引用。

3. 使用lambda表达式和函数式接口Predicate

   除了在语言层面支持函数式编程风格，Java 8也添加了一个包，叫做 java.util.function。它包含了很多类，用来支持Java的函数式编程。其中一个便是Predicate，使用 java.util.function.Predicate 函数式接口以及lambda表达式，可以向API方法添加逻辑，用更少的代码支持更多的动态行为。下面是Java 8 Predicate 的例子，展示了过滤集合数据的多种常用方法。Predicate接口非常适用于做过滤。

   ```java
   public static void main(args[]){
       List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
    
       System.out.println("Languages which starts with J :");
       filter(languages, (str)->str.startsWith("J"));
    
       System.out.println("Languages which ends with a ");
       filter(languages, (str)->str.endsWith("a"));
    
       System.out.println("Print all languages :");
       filter(languages, (str)->true);
    
       System.out.println("Print no language : ");
       filter(languages, (str)->false);
    
       System.out.println("Print language whose length greater than 4:");
       filter(languages, (str)->str.length() > 4);
   }
    
   public static void filter(List names, Predicate condition) {
       for(String name: names)  {
           if(condition.test(name)) {
               System.out.println(name + " ");
           }
       }
   }
   
   
   // 更好的办法
   //public static void filter(List names, Predicate condition) {
   //    names.stream().filter((name) -> (condition.test(name))).forEach((name) -> {
   //        System.out.println(name + " ");
   //    });
   //}
   ```

   可以看到，Stream API的过滤方法也接受一个Predicate，这意味着可以将我们定制的 filter() 方法替换成写在里面的内联代码，这就是lambda表达式的魔力。另外，Predicate接口也允许进行多重条件的测试，下个例子将要讲到。

4. 如何在lambda表达式中加入Predicate

   上个例子说到，java.util.function.Predicate 允许将两个或更多的 Predicate 合成一个。它提供类似于逻辑操作符AND和OR的方法，名字叫做and()、or()和xor()，用于将传入 filter() 方法的条件合并起来。例如，要得到所有以J开始，长度为四个字母的语言，可以定义两个独立的 Predicate 示例分别表示每一个条件，然后用 Predicate.and() 方法将它们合并起来，如下所示：

   ```
   // 甚至可以用and()、or()和xor()逻辑函数来合并Predicate，
   // 例如要找到所有以J开始，长度为四个字母的名字，你可以合并两个Predicate并传入
   Predicate<String> startsWithJ = (n) -> n.startsWith("J");
   Predicate<String> fourLetterLong = (n) -> n.length() == 4;
   names.stream()
       .filter(startsWithJ.and(fourLetterLong))
       .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));
   ```

   类似地，也可以使用 or() 和 xor() 方法。本例着重介绍了如下要点：可按需要将 Predicate 作为单独条件然后将其合并起来使用。简而言之，你可以以传统Java命令方式使用 Predicate 接口，也可以充分利用lambda表达式达到事半功倍的效果。

5. Java 8中使用lambda表达式的Map和Reduce示例

   本例介绍最广为人知的函数式编程概念map。它允许你将对象进行转换。例如在本例中，我们将 costBeforeTax 列表的每个元素转换成为税后的值。我们将 x -> x*x lambda表达式传到 map() 方法，后者将其应用到流中的每一个元素。然后用 forEach() 将列表元素打印出来。使用流API的收集器类，可以得到所有含税的开销。有 toList() 这样的方法将 map 或任何其他操作的结果合并起来。由于收集器在流上做终端操作，因此之后便不能重用流了。你甚至可以用流API的 reduce() 方法将所有数字合成一个，下一个例子将会讲到。

   ```
   // 不使用lambda表达式为每个订单加上12%的税
   List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
   for (Integer cost : costBeforeTax) {
       double price = cost + 0.12*cost;
       System.out.println(price);
   }
    
   // 使用lambda表达式
   List<Double> costBeforeTax = Arrays.asList(100.0, 200.3, 300.5, 400.0, 500.0);
           costBeforeTax.stream().map((cost) -> cost + 0.12*cost).forEach(System.out::println);
   
   ```

   在上个例子中，可以看到map将集合类（例如列表）元素进行转换的。还有一个 reduce() 函数可以将所有值合并成一个。Map和Reduce操作是函数式编程的核心操作，因为其功能，reduce 又被称为折叠操作。另外，reduce 并不是一个新的操作，你有可能已经在使用它。

   SQL中类似 sum()、avg() 或者 count() 的聚集函数，实际上就是 reduce 操作，因为它们接收多个值并返回一个值。流API定义的 reduceh() 函数可以接受lambda表达式，并对所有值进行合并。IntStream这样的类有类似 average()、count()、sum() 的内建方法来做 reduce 操作，也有mapToLong()、mapToDouble() 方法来做转换。这并不会限制你，你可以用内建方法，也可以自己定义。在这个Java 8的Map Reduce示例里，我们首先对所有价格应用 12% 的VAT，然后用 reduce() 方法计算总和。

   ```
   // 为每个订单加上12%的税
   // 老方法：
   List<Integer> costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
   double total = 0;
   for (Integer cost : costBeforeTax) {
       double price = cost + .12*cost;
       total = total + price;
   }
   System.out.println("Total : " + total);
    
   // 新方法：
   List<Integer> costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
   double bill = costBeforeTax.stream().map((cost) -> cost + .12*cost).reduce((sum, cost) -> sum + cost).get();
   System.out.println("Total : " + bill);
   ```

6. 通过过滤创建一个String列表

   过滤是Java开发者在大规模集合上的一个常用操作，而现在使用lambda表达式和流API过滤大规模数据集合是惊人的简单。流提供了一个 filter() 方法，接受一个 Predicate 对象，即可以传入一个lambda表达式作为过滤逻辑。下面的例子是用lambda表达式过滤Java集合，将帮助理解。

   ```
   // 创建一个字符串列表，每个字符串长度大于2
   List<String> strList = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
   
   List<String> filtered = strList.stream().filter(x -> x.length()> 2).collect(Collectors.toList());
   System.out.printf("Original List : %s, filtered list : %s %n", strList, filtered);
   ```

   另外，关于 filter() 方法有个常见误解。在现实生活中，做过滤的时候，通常会丢弃部分，但使用filter()方法则是获得一个新的列表，且其每个元素符合过滤原则。

7. 对列表的每个元素应用函数

   我们通常需要对列表的每个元素使用某个函数，例如逐一乘以某个数、除以某个数或者做其它操作。这些操作都很适合用 map() 方法，可以将转换逻辑以lambda表达式的形式放在 map() 方法里，就可以对集合的各个元素进行转换了，如下所示。

   ```
   // 将字符串换成大写并用逗号链接起来
   List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
   String G7Countries = G7.stream().map(String::toUpperCase).collect(Collectors.joining(", "));
   System.out.println(G7Countries);
   ```

8. 复制不同的值，创建一个子列表

   本例展示了如何利用流的 distinct() 方法来对集合进行去重。

   ```
   // 用所有不同的数字创建一个正方形列表
   List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
   List<Integer> distinct = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
   System.out.printf("Original List : %s,  Square Without duplicates : %s %n", numbers, distinct);
   ```

9. 计算集合元素的最大值、最小值、总和以及平均值

   IntStream、LongStream 和 DoubleStream 等流的类中，有个非常有用的方法叫做 summaryStatistics() 。可以返回 IntSummaryStatistics、LongSummaryStatistics 或者 DoubleSummaryStatistic s，描述流中元素的各种摘要数据。在本例中，我们用这个方法来计算列表的最大值和最小值。它也有 getSum() 和 getAverage() 方法来获得列表的所有元素的总和及平均值。

   ```
   //获取数字的个数、最小值、最大值、总和以及平均值
   List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
   IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
   System.out.println("Highest prime number in List : " + stats.getMax());
   System.out.println("Lowest prime number in List : " + stats.getMin());
   System.out.println("Sum of all prime numbers : " + stats.getSum());
   System.out.println("Average of all prime numbers : " + stats.getAverage());
   ```

既然lambda表达式即将正式取代Java代码中的匿名内部类，那么有必要对二者做一个比较分析。一个关键的不同点就是关键字 this。匿名类的 this 关键字指向匿名类，而lambda表达式的 this 关键字指向包围lambda表达式的类。另一个不同点是二者的编译方式。Java编译器将lambda表达式编译成类的私有方法。使用了Java 7的 invokedynamic 字节码指令来动态绑定这个方法。

### Stream

![2.png](https://i.loli.net/2019/05/31/5cf0d3694d86b29939.png)

#### 为什么需要Stream ?

Stream作为Java8的一大亮点，它与java.io包里的InputStream和OutputStream是完全不同的概念。它也不同于StAX对XML解析的Stream，也不是Amazon  Kinesis对大数据实时处理的Stream。Java8中的Stream是对容器对象功能的增强，它专注于对容器对象进行各种非常便利、高效的 **聚合操作（aggregate operation）**，或者大批量数据操作  (bulk data operation)。Stream  API借助于同样新出现的Lambda表达式，极大的提高编程效率和程序可读性。同时，它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用fork/join并行方式来拆分任务和加速处理过程。通常，编写并行代码很难而且容易出错,  但使用Stream API无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java8中首次出现的 **java.util.stream是一个函数式语言+多核时代综合影响的产物。**

#### 什么是聚合操作

在传统的J2EE应用中，Java代码经常不得不依赖于关系型数据库的聚合操作来完成诸如：

- 客户每月平均消费金额
- 最昂贵的在售商品
- 本周完成的有效订单（排除了无效的）
- 取十个数据样本作为首页推荐

这类的操作。但在当今这个数据大爆炸的时代，在数据来源多样化、数据海量化的今天，很多时候不得不脱离 RDBMS，或者以底层返回的数据为基础进行更上层的数据统计。而Java的集合API中，仅仅有极少量的辅助型方法，更多的时候是程序员需要用Iterator来遍历集合，完成相关的聚合应用逻辑，这是一种远不够高效、笨拙的方法。在Java7中，如果要发现type为grocery的所有交易，然后返回以交易值降序排序好的交易ID集合，我们需要这样写：

```java
List<Transaction> groceryTransactions = new Arraylist<>();
for(Transaction t: transactions){
 if(t.getType() == Transaction.GROCERY){
 groceryTransactions.add(t);
 }
}

Collections.sort(groceryTransactions, new Comparator(){
 public int compare(Transaction t1, Transaction t2){
 return t2.getValue().compareTo(t1.getValue());
 }
});

List<Integer> transactionIds = new ArrayList<>();
for(Transaction t: groceryTransactions){
 transactionsIds.add(t.getId());
}
```

而在 Java 8 使用 Stream，代码更加简洁易读；而且使用并发模式，程序执行速度更快。

```java
List<Integer> transactionsIds = transactions.parallelStream()
.filter(t -> t.getType() == Transaction.GROCERY)
.sorted(comparing(Transaction::getValue).reversed())
.map(Transaction::getId).collect(toList());
```

#### 什么是流？

Stream不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的Iterator。原始版本的Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的Stream，用户只要给出需要对其包含的元素执行什么操作，比如，“过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream会隐式地在内部进行遍历，做出相应的数据转换。Stream就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

而和迭代器又不同的是，Stream可以**并行化**操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个item读完后再读下一个item。**而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。**Stream的并行操作依赖于Java7中引入的Fork/Join框架（JSR166y）来拆分任务和加速处理过程。

Stream 的另外一大特点是，**数据源本身可以是无限的。**

#### 流与集合

**什么时候计算**

Stream 和集合的其中一个差异在于什么时候进行计算。
一个集合，它会包含当前数据结构中所有的值，你可以随时增删，但是集合里面的元素毫无疑问地都是已经计算好了的。
流则是按需计算，按照使用者的需要计算数据，你可以想象我们通过搜索引擎进行搜索，搜索出来的条目并不是全部呈现出来的，而且先显示最符合的前 10 条或者前 20 条，只有在点击 “下一页” 的时候，才会再输出新的 10 条。

**外部迭代和内部迭代**

Stream 和集合的另一个差异在于迭代。

我们可以把集合比作一个工厂的仓库，一开始工厂比较落后，要对货物作什么修改，只能工人亲自走进仓库对货物进行处理，有时候还要将处理后的货物放到一个新的仓库里面。在这个时期，我们需要亲自去做迭代，一个个地找到需要的货物，并进行处理，这叫做**外部迭代**。

后来工厂发展了起来，配备了流水线作业，只要根据需求设计出相应的流水线，然后工人只要把货物放到流水线上，就可以等着接收成果了，而且流水线还可以根据要求直接把货物输送到相应的仓库。这就叫做**内部迭代**，流水线已经帮你把迭代给完成了，你只需要说要干什么就可以了（即设计出合理的流水线）。

Java 8 引入 Stream 很大程度是因为，流的内部迭代可以自动选择一种合适你硬件的数据表示和并行实现；而以往程序员自己进行 foreach 之类的时候，则需要自己去管理并行等问题。

#### 流的构成

当我们使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）-> 数据转换 -> 执行操作获取想要的结果。**每次转换原有Stream对象不改变，返回一个新的Stream对象（可以有多次转换）**，这就允许对其操作可以像链条一样排列，变成一个管道，如下图所示:

![1.png](https://i.loli.net/2019/05/31/5cf0d130d49bf66216.png)

**Stream的生成方式**

（1）从Collection和数组获得

- Collection.stream()
- Collection.parallelStream()
- Arrays.stream(T array) or Stream.of()

（2）从BufferedReader获得

- java.io.BufferedReader.lines()

（3）静态工厂

- java.util.stream.IntStream.range()
- java.nio.file.Files.walk()

（4）自己构建

- java.util.Spliterator

（5）其他

- Random.ints()
- BitSet.stream()
- Pattern.splitAsStream(java.lang.CharSequence)
- JarFile.stream()

**流的操作类型**

流的操作类型分为两种：

- Intermediate：一个流可以后面跟随零个或多个intermediate操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- Terminal：一个流只能有一个terminal操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以,这必定是流的最后一个操作。Terminal操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个side effect。

在对一个Stream进行多次转换操作(Intermediate 操作)，每次都对Stream的每个元素进行转换，而且是执行多次，这样时间复杂度就是N（转换次数）个for循环里把所有操作都做掉的总和吗？其实不是这样的，**转换操作都是lazy的，多个转换操作只会在Terminal操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在Terminal  操作的时候循环Stream对应的集合，然后对每个元素执行所有的函数。**

还有一种操作被称为**short-circuiting**。用以指：对于一个intermediate操作，如果它接受的是一个无限大（infinite/unbounded）的Stream，但返回一个有限的新Stream；对于一个terminal操作，如果它接受的是一个无限大的Stream，但能在有限的时间计算出结果。  
 当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个short-circuiting操作是必要非充分条件。

#### 使用详解

简单说，**对Stream的使用就是实现一个filter-map-reduce过程，产生一个最终结果，或者导致一个副作用（side effect）。**

1. 流的构造与转换，下面提供最常见的几种构造Stream的例子:

   ```
   // 1. Individual values
   Stream stream = Stream.of("a", "b", "c");
   
   // 2. Arrays
   String [] strArray = new String[] {"a", "b", "c"};
   stream = Stream.of(strArray);
   stream = Arrays.stream(strArray);
   
   // 3. Collections
   List<String> list = Arrays.asList(strArray);
   stream = list.stream();
   ```

   需要注意的是，对于基本数值型，目前有三种对应的包装类型Stream：IntStream、LongStream、DoubleStream。当然我们也可以用 Stream<Integer\>、Stream<Long\>和Stream<Double\>，但是boxing/unboxing会很耗时，所以特别为这三种基本数值型提供了对应的Stream。

   Java8中还没有提供其它数值型Stream，因为这将导致扩增的内容较多。而常规的数值型聚合运算可以通过上面三种Stream进行。

   ```
   IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
   IntStream.range(1, 3).forEach(System.out::println);
   IntStream.rangeClosed(1, 3).forEach(System.out::println);
   ```

   流也可以转换为其它数据结构，例如：

   ```
   // 1. Array
   String[] strArray1 = stream.toArray(String[]::new);
   // 2. Collection
   List<String> list1 = stream.collect(Collectors.toList());
   List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));
   Set set1 = stream.collect(Collectors.toSet());
   Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));
   // 3. String
   String str = stream.collect(Collectors.joining()).toString();
   ```

2. 流的操作

   接下来，当把一个数据结构包装成Stream后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下：

   - Intermediate 操作

     map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

   - Terminal 操作

     forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

   - Short-circuiting 操作

     anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

##### 具体介绍

1. stream() / parallelStream()

   最常用到的方法，将集合转换为流

   ```
   List list = new ArrayList();
   // return Stream<E>
   list.stream();
   ```

   而 parallelStream() 是并行流方法，能够让数据集执行并行操作，后面会更详细地讲解

2. filter(T -> boolean)

   保留 boolean 为 true 的元素

   ```
   保留年龄为 20 的 person 元素
   list = list.stream()
               .filter(person -> person.getAge() == 20)
               .collect(toList());
   
   打印输出 [Person{name='jack', age=20}]
   ```

   collect(toList()) 可以把流转换为 List 类型，这个以后会讲解

3. distinct()

   去除重复元素，这个方法是通过类的 equals 方法来判断两个元素是否相等的

   如例子中的 Person 类，需要先定义好 equals 方法，不然类似`[Person{name='jack', age=20}, Person{name='jack', age=20}]` 这样的情况是不会处理的

4. sorted() / sorted((T, T) -> int)

   如果流中的元素的类实现了 Comparable 接口，即有自己的排序规则，那么可以直接调用 sorted() 方法对元素进行排序，如 Stream<Integer\>

   反之, 需要调用 `sorted((T, T) -> int)` 实现 Comparator 接口

   ```
   根据年龄大小来比较：
   list = list.stream()
              .sorted((p1, p2) -> p1.getAge() - p2.getAge())
              .collect(toList());
   ```

   当然这个可以简化为

   ```
   list = list.stream()
              .sorted(Comparator.comparingInt(Person::getAge))
              .collect(toList());
   ```

5. limit(long n)

   返回前 n 个元素

   ```
   list = list.stream()
               .limit(2)
               .collect(toList());
   
   打印输出 [Person{name='jack', age=20}, Person{name='mike', age=25}]
   ```

6. skip(long n)

   去除前 n 个元素

   ```
   list = list.stream()
               .skip(2)
               .collect(toList());
   
   打印输出 [Person{name='tom', age=30}]
   ```

   **tips**:

   用在 limit(n) 前面时，先去除前 m 个元素再返回剩余元素的前 n 个元素

   limit(n) 用在 skip(m) 前面时，先返回前 n 个元素再在剩余的 n 个元素中去除 m 个元素

   ```
   list = list.stream()
               .limit(2)
               .skip(1)
               .collect(toList());
   
   打印输出 [Person{name='mike', age=25}]
   ```

7. map(T -> R)

   将流中的每一个元素 T 映射为 R（类似类型转换）

   ```
   List<String> newlist = list.stream().map(Person::getName).collect(toList());
   ```

   newlist 里面的元素为 list 中每一个 Person 对象的 name 变量

8. flatMap(T -> Stream<R\>)

   将流中的每一个元素 T 映射为一个流，再把每一个流连接成为一个流

   ```
   List<String> list = new ArrayList<>();
   list.add("aaa bbb ccc");
   list.add("ddd eee fff");
   list.add("ggg hhh iii");
   
   list = list.stream().map(s -> s.split(" ")).flatMap(Arrays::stream).collect(toList());
   ```

   上面例子中，我们的目的是把 List 中每个字符串元素以" "分割开，变成一个新的 List<String\>。
    首先 map 方法分割每个字符串元素，但此时流的类型为 Stream<String[ ]>，因为 split 方法返回的是 String[ ] 类型；所以我们需要使用 flatMap 方法，先使用`Arrays::stream`将每个 String[ ] 元素变成一个 Stream<String\> 流，然后 flatMap 会将每一个流连接成为一个流，最终返回我们需要的  Stream<String\>

9. anyMatch(T -> boolean)

   流中是否有一个元素匹配给定的 `T -> boolean` 条件

   ```
   是否存在一个 person 对象的 age 等于 20：
   boolean b = list.stream().anyMatch(person -> person.getAge() == 20);
   ```

10. allMatch(T -> boolean)

    流中是否所有元素都匹配给定的 `T -> boolean` 条件

11. noneMatch(T -> boolean)

    流中是否没有元素匹配给定的 `T -> boolean` 条件

12. findAny() 和 findFirst()

    findAny()：找到其中一个元素 （使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素）

    findFirst()：找到第一个元素

    **值得注意的是，这两个方法返回的是一个 Optional<T\> 对象**，它是一个容器类，能代表一个值存在或不存在，这个后面会讲到

13. reduce((T, T) -> T) 和 reduce(T, (T, T) -> T)

    用于组合流中的元素，如求和，求积，求最大值等

    ```
    计算年龄总和：
    int sum = list.stream().map(Person::getAge).reduce(0, (a, b) -> a + b);
    与之相同:
    int sum = list.stream().map(Person::getAge).reduce(0, Integer::sum);
    ```

    其中，reduce 第一个参数 0 代表起始值为 0，lambda `(a, b) -> a + b` 即将两值相加产生一个新值

    同样地：

    ```
    计算年龄总乘积：
    int sum = list.stream().map(Person::getAge).reduce(1, (a, b) -> a * b);
    ```

    当然也可以

    ```
    Optional<Integer> sum = list.stream().map(Person::getAge).reduce(Integer::sum);
    ```

    即不接受任何起始值，但因为没有初始值，需要考虑结果可能不存在的情况，因此返回的是 Optional 类型

14. count()

    返回流中元素个数，结果为 long 类型

15. collect()

    收集方法，我们很常用的是 `collect(toList())`，当然还有 `collect(toSet())` 等，参数是一个收集器接口，这个后面会另外讲

16. forEach()

    返回结果为 void，很明显我们可以通过它来干什么了，比方说：

    ```
    list.stream().forEach(System.out::println);
    ```

    再比如说 MyBatis 里面访问数据库的 mapper 方法：

    ```
    向数据库插入新元素：
    list.stream().forEach(PersonMapper::insertPerson);
    ```

17. unordered()

    还有这个比较不起眼的方法，返回一个等效的无序流，当然如果流本身就是无序的话，那可能就会直接返回其本身

#### 小结

总之，Stream 的特性可以归纳为：

- 不是数据结构;
- 它没有内部存储，它只是用操作管道从source（数据结构、数组、generator function、IO channel）抓取数据;
- 它也绝不修改自己所封装的底层数据结构的数据。例如Stream的filter操作会产生一个不包含被过滤元素的新Stream，而不是从source删除那些元素;
- 所有Stream的操作必须以lambda表达式为参数;
- 不支持索引访问;
- 你可以请求第一个元素，但无法请求第二个，第三个，或最后一个;
- 很容易生成数组或者List;
- 惰性化;
- 很多Stream操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始;
- Intermediate操作永远是惰性化的;
- 并行能力;
- 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的;
- 可以是无限的。集合有固定大小，Stream 则不必。limit(n)和findFirst()这类的short-circuiting操作可以对无限的Stream进行运算并很快完成。

### Optional类

NullPointException可以说是所有java程序员都遇到过的一个异常，虽然java从设计之初就力图让程序员脱离指针的苦海，但是指针确实是实际存在的，而java设计者也只能是让指针在java语言中变得更加简单、易用，而不能完全的将其剔除，所以才有了我们日常所见到的关键字`null`。

空指针异常是一个运行时异常，对于这一类异常，如果没有明确的处理策略，那么最佳实践在于让程序早点挂掉，但是很多场景下，不是开发人员没有具体的处理策略，而是根本没有意识到空指针异常的存在。当异常真的发生的时候，处理策略也很简单，在存在异常的地方添加一个if语句判定即可，但是这样的应对策略会让我们的程序出现越来越多的null判定，我们知道一个良好的程序设计，应该让代码中尽量少出现null关键字，而java8所提供的`Optional`类则在减少NullPointException的同时，也提升了代码的美观度。但首先我们需要明确的是，它并 **不是对null关键字的一种替代，而是对于null判定提供了一种更加优雅的实现，从而避免NullPointException**。

Optional 类比较常用的几个方法有：

- isPresent() ：值存在时返回 true，反之 flase
- get() ：返回当前值，若值不存在会抛出异常
- orElse(T) ：值存在时返回该值，否则返回 T 的值

Optional 类还有三个特化版本 OptionalInt，OptionalLong，OptionalDouble，刚刚讲到的数值流中的 max 方法返回的类型便是这个

Optional 类其中其实还有很多学问，讲解它说不定也要开一篇文章，这里先讲那么多，先知道基本怎么用就可以。

假设我们需要返回一个字符串的长度，如果不借助第三方工具类，我们需要调用`str.length()`方法：

```
if(null == str) { // 空指针判定
    return 0;
}
return str.length();
```

如果采用Optional类，实现如下：

```
return Optional.ofNullable(str).map(String::length).orElse(0);
```

Optional的代码相对更加简洁，当代码量较大时，我们很容易忘记进行null判定，但是使用Optional类则会避免这类问题。

#### 对象创建

**创建空对象**

```
Optional<String> optStr = Optional.empty();
```

上面的示例代码调用`empty()`方法创建了一个空的`Optional<String>`对象型。

**创建对象：不允许为空**
Optional提供了方法`of()`用于创建非空对象，该方法要求传入的参数不能为空，否则抛`NullPointException`，示例如下：

```
Optional<String> optStr = Optional.of(str);  // 当str为null的时候，将抛出NullPointException
```

**创建对象：允许为空**
如果不能确定传入的参数是否存在null值的可能性，则可以用Optional的`ofNullable()`方法创建对象，如果入参为null，则创建一个空对象。示例如下：

```
Optional<String> optStr = Optional.ofNullable(str);  // 如果str是null，则创建一个空对象
```

#### 使用

为了演示，我们设计了一个`User`类，如下：

```java

public class User {

    /** 用户编号 */
    private long id;

    private String name;

    private int age;

    private Optional<Long> phone;

    private Optional<String> email;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 省略setter和getter
}
```

手机和邮箱不是一个人的必须有的，所以我们利用Optional定义。

**映射：map与flatMap**
映射是将输入转换成另外一种形式的输出的操作，比如前面例子中，我们输入字符串，而输出的是字符串的长度，这就是一种隐射，我们利用方法`map()`得以实现。假设我们希望获得一个人的姓名，那么我们可以如下实现：

```
String name = Optional.ofNullable(user).map(User::getName).orElse("no name")
```

这样当入参user不为空的时候则返回其name，否则返回`no name` 如我我们希望通过上面方式得到phone或email，利用上面的方式则行不通了，因为map之后返回的是Optional，我们把这种称为Optional嵌套，我们必须在map一次才能拿到我们想要的结果：

```
long phone = optUser.map(User::getPhone).map(Optional::get).orElse(-1L);
```

其实这个时候，更好的方式是利用flatMap，一步拿到我们想要的结果：

```
long phone = optUser.flatMap(User::getPhone).orElse(-1L);
```

flapMap可以将方法返回的各个流扁平化成为一个流。

**过滤：fliter**
filiter，顾名思义是过滤的操作，我们可以将过滤操作做为参数传递给该方法，从而实现过滤目的，加入我们希望筛选18周岁以上的成年人，则可以实现如下：

```
optUser.filter(u -> u.getAge() >= 18).ifPresent(u -> System.out.println("Adult:" + u));
```

#### 类方法

| 序号 | 方法 & 描述                                                  |
| ---- | ------------------------------------------------------------ |
| 1    | **static <T\> Optional<T\> empty()** 返回空的 Optional 实例。 |
| 2    | **boolean equals(Object obj)** 判断其他对象是否等于 Optional。 |
| 3    | **Optional<T\> filter(Predicate<? super <T\> predicate)** 如果值存在，并且这个值匹配给定的 predicate，返回一个Optional用以描述这个值，否则返回一个空的Optional。 |
| 4    | **<U\> Optional<U\> flatMap(Function<? super T,Optional<U\>> mapper)** 如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional |
| 5    | **T get()** 如果在这个Optional中包含这个值，返回值，否则抛出异常：NoSuchElementException |
| 6    | **int hashCode()** 返回存在值的哈希码，如果值不存在 返回 0。 |
| 7    | **void ifPresent(Consumer<? super T> consumer)** 如果值存在则使用该值调用 consumer , 否则不做任何事情。 |
| 8    | **boolean isPresent()** 如果值存在则方法会返回true，否则返回 false。 |
| 9    | **<U\>Optional<U\> map(Function<? super T,? extends U> mapper)**  如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。 |
| 10   | **static <T\> Optional<T\> of(T value)** 返回一个指定非null值的Optional。 |
| 11   | **static <T\> Optional<T\> ofNullable(T value)** 如果为非空，返回 Optional 描述的指定值，否则返回空的 Optional。 |
| 12   | **T orElse(T other)** 如果存在该值，返回值， 否则返回 other。 |
| 13   | **T orElseGet(Supplier<? extends T> other)** 如果存在该值，返回值， 否则触发 other，并返回  other 调用的结果。 |
| 14   | **<X extends Throwable\> T orElseThrow(Supplier<? extends X> exceptionSupplier)** 如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常 |
| 15   | **String toString()** 返回一个Optional的非空字符串，用来调试 |

**注意：** 这些方法是从 **java.lang.Object** 类继承来的。

#### 注意

Optional是一个final类，未实现任何接口，所以当我们在利用该类包装定义类的属性的时候，如果我们定义的类有序列化的需求，那么因为Optional没有实现Serializable接口，这个时候执行序列化操作就会有问题

```
public class User implements Serializable{

    /** 用户编号 */
    private long id;

    private String name;

    private int age;

    private Optional<Long> phone;  // 不能序列化

    private Optional<String> email;  // 不能序列化
```

不过我们可以采用如下替换策略：

```
private long phone;

public Optional<Long> getPhone() {
    return Optional.ofNullable(this.phone);
}
```

### 参考

1. [浅析Java8 Stream原理](https://www.jianshu.com/p/dd5fb725331b)
2. [JAVA8如何妙用Optional解决NPE问题详解](https://www.jb51.net/article/141787.htm)
3. [如何更好地使用Java 8的Optional](https://www.jdon.com/idea/java/using-optional-effectively-in-java-8.html)