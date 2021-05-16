---
title: 设计模式--行为型（下）
date: 2019-05-24 22:18:59
tags:
 - Java
 - 设计模式
categories:
 - Java
 - 设计模式
---

### 解释器模式

解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，它属于行为型模式。这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等。

<!--more-->

**意图：**给定一个语言，定义它的文法表示，并定义一个解释器，这个解释器使用该标识来解释语言中的句子。

**主要解决：**对于一些固定文法构建一个解释句子的解释器。

**何时使用：**如果一种特定类型的问题发生的频率足够高，那么可能就值得将该问题的各个实例表述为一个简单语言中的句子。这样就可以构建一个解释器，该解释器通过解释这些句子来解决该问题。

**如何解决：**构建语法树，定义终结符与非终结符。

**关键代码：**构建环境类，包含解释器之外的一些全局信息，一般是 HashMap。

**应用实例：**编译器、运算表达式计算。

**优点：**

1. 易于实现简易语法，一条语法规则用一个解释器对象解释执行
2. 易于扩展新的语法，只需创建相应的解释器对象，在创建抽象语法树的时候使用即可。
3. 增加了新的解释表达式的方式

**缺点：**

1.可使用场景较少

2.对于复杂的文法比较难维护

3.引起类膨胀

4.采用递归调用方法，效率，性能，维护问题

**使用场景：** 

1.重复发生的问题

2.一个简单语法需要解释的场景

3.将一个解释执行的语言中的句子表示为一个抽象语法树

**注意事项：**可利用场景比较少，JAVA 中如果碰到可以用 expression4J 代替。

**四个角色**

- AbstractExpression：抽象表达式，声明一个所有具体表达式都要实现的接口，
  接口中主要是一个interpret()方法，称为解释操作，具体的解释任务由他的各个实现
  类来完成，而具体的解释器又分别由 终结符解释器 与 非终结符解释器 完成。
- TerminalExpression：终结符表达式，实现与文法中的元素相关联的解释操作，
  通常一个解释器模式中只有一个终结符表达式，但有多个实例，对应不同的终结符。、
  终结符一半是文法中的运算单元，比如有一个简单的公式R=R1+R2，在里面R1和R2就是终结符，
  对应的解析R1和R2的解释器就是终结符表达式。

- NonterminalExpression ：非终结符表达式，文法中的每条规则对应于一个非终
  结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字，比如公式R=R1+R2中，
  +就是非终结符，解析+的解释器就是一个非终结符表达式。非终结符表达式根据逻辑的复杂
  程度而增加，原则上每个文法规则都对应一个非终结符表达式。

- Context：上下文环境，存放文法中各个终结符所对应的具体值，比如R=R1+R2，我们
  给R1赋值100，给R2赋值200。这些信息需要存放到环境角色中，很多情况下我们使用Map来充
  当环境角色就足够了。

**UML类图**

![5.png](https://i.loli.net/2019/06/05/5cf79874b3ebe18496.png)

#### 示例

定义一个能够解释加减法的解释器作为示例

先定义**抽象表达式**

```
public abstract class Expression {

    public abstract int interpret(Context context);

    @Override public abstract String toString();
}
```

接着定义加减法两个**非终结符表达式**

```
public class PlusExpression extends Expression{

    private Expression leftExpression;
    private Expression rightExpression;

    public PlusExpression(Expression leftExpression, Expression rightExpression) {
        this.leftExpression = leftExpression;
        this.rightExpression = rightExpression;
    }

    @Override public int interpret(Context context) {
        return leftExpression.interpret(context) + rightExpression.interpret(context);
    }

    @Override public String toString() {
        return leftExpression.toString() + " + " + rightExpression.toString();
    }
}

public class MinusExpression extends Expression{

    private Expression leftExpression;
    private Expression rightExpression;

    public MinusExpression(Expression leftExpression, Expression rightExpression) {
        this.leftExpression = leftExpression;
        this.rightExpression = rightExpression;
    }

    @Override public int interpret(Context context) {
        return leftExpression.interpret(context) - rightExpression.interpret(context);
    }

    @Override public String toString() {
        return leftExpression.toString() + " - " + rightExpression.toString();
    }
}
```

再接着定义常量与变量两个**终结符表达式**

```
public class ConstantExpression extends Expression {

    private int value;

    public ConstantExpression(int value) {
        this.value = value;
    }

    @Override public int interpret(Context context) {
        return value;
    }

    @Override public String toString() {
        return Integer.toString(value);
    }
}

public class VariableExpression extends Expression {

    private String name;

    public VariableExpression(String name) {
        this.name = name;
    }

    @Override public int interpret(Context context) {
        return context.lookup(this);
    }

    @Override public String toString() {
        return name;
    }
}


```

然后定义**上下文环境**，用Map存放各个终结符对应的具体值

```
public class Context {

    private Map<Expression, Integer> map = new HashMap<>();

    public void addExpression(Expression expression, int value) {
        map.put(expression, value);
    }

    public int lookup(Expression expression) {
        return map.get(expression);
    }

}

```

最后客户端调用

```
public class Client {
    public static void main(String[] args) {
        Context context = new Context();

        VariableExpression a = new VariableExpression("a");
        VariableExpression b = new VariableExpression("b");
        ConstantExpression c = new ConstantExpression(6);

        context.addExpression(a, 2);
        context.addExpression(b, 3);

        Expression expression = new PlusExpression(new PlusExpression(a,b),new MinusExpression(a,c));
        System.out.println(expression.toString() + " = " + expression.interpret(context));

    }
}

```

输出结果

```
a + b + a - 6 = 1
```

### 中介者模式

中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。

**意图：**用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

**主要解决：**对象与对象之间存在大量的关联关系，这样势必会导致系统的结构变得很复杂，同时若一个对象发生改变，我们也需要跟踪与之相关联的对象，同时做出相应的处理。

**何时使用：**多个类相互耦合，形成了网状结构。

**如何解决：**将上述网状结构分离为星型结构。

**关键代码：**对象 Colleague 之间的通信封装到一个类中单独处理。

**应用实例：** 

1. 中国加入 WTO 之前是各个国家相互贸易，结构复杂，现在是各个国家通过 WTO 来互相贸易。
2. 机场调度系统。 
3. MVC 框架，其中C（控制器）就是 M（模型）和 V（视图）的中介者。

**优点：** 

1. 降低了类的复杂度，将一对多转化成了一对一。 
2. 各个类之间的解耦。 
3. 符合迪米特原则。 

**缺点：**中介者会庞大，变得复杂难以维护。

**使用场景：** 

1. 系统中对象之间存在比较复杂的引用关系，导致它们之间的依赖关系结构混乱而且难以复用该对象。
2. 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。 

**注意事项：**不应当在职责混乱的时候使用。

![1.png](https://i.loli.net/2019/06/05/5cf76e712e90586279.png)

这里有必要和前面学的**外观模式**，**代理模式**进行区分！

- **外观模式**

**结构型**，对子系统提供统一的接口，**单向**，所有请求都委托子系统完成，**树型**

![2.png](https://i.loli.net/2019/06/05/5cf76ee165db748226.png)

- **代理模式**

**结构型**，**引用代理对象的方式来访问目标对象**，**单向**

![3.png](https://i.loli.net/2019/06/05/5cf76ee163b9a20855.png)

- **中介者模式**

**行为型**，**用一个中介对象来封装一系列同事对象的交互行为**，**双向**，**一对多星型**

![4.png](https://i.loli.net/2019/06/05/5cf76ee18d8fd49451.png)

**四个角色**

- Mediator：抽象中介者
  定义了同事对象到中介者对象的接口
- ConcreteMediator：具体中介者
  实现相关方法，需要知道所有具体同事类，并从具体同事接手信息，像具体同事对象发出命令。
- Colleague：抽象同事
  定义了中介者对象接口，它只知道中介者而不知道其他同事对象。
- ConcreteColleague：具体同事
  实现相关方法，只知道自己的行为，不了解其他具体同事的状况，但都认识中介者对象。

#### 示例

这里用写个简单的房屋中介例子，帮助大家理解这个模式的使用。

首先是**抽象中介类**，里面只有一个连接同事进行交互的方法

```
public abstract class Mediator {
    abstract void contact(People people, String msg);
}
```

接着到**抽象同事类**，两个属性，姓名，还有一个**中介类的引用**，因为所有同事都知道中介

```
public abstract class People {
    protected String name;
    protected Mediator mediator;    //每个人都知道中介

    public People(String name, Mediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }
}
```

再接着是**具体同事类**，这里有房东和房客

```
public class Landlord extends People {

    public Landlord(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void contact(String msg) {
        mediator.contact(this, msg);
    }

    public void getMessage(String msg) {
        System.out.println("【房东】" + name + "：" + msg);
    }

}

public class Tenant extends People {

    public Tenant(String name, Mediator mediator) {
        super(name, mediator);
    }

    public void contact(String msg) {
        mediator.contact(this, msg);
    }

    public void getMessage(String msg) {
        System.out.println("【房客】" + name + "：" + msg);
    }

}
```

然后是**具体中介类**，因为**中介者知道所有的同事**，这里创建两个引用，另外 实现交互方法时对，调用者进行判断，完成对应的信息显示，比如这里房东发的， 房客收到。

```
public class HouseMediator extends Mediator {
    //中介者知道所有同事
    private Landlord landlord;
    private Tenant tenant;

    public Landlord getLandlord() {
        return landlord;
    }

    public void setLandlord(Landlord landlord) {
        this.landlord = landlord;
    }

    public Tenant getTenant() {
        return tenant;
    }

    public void setTenant(Tenant tenant) {
        this.tenant = tenant;
    }

    @Override void contact(People people, String msg) {
        if(people == tenant) {
            tenant.getMessage(msg);
        } else {
            landlord.getMessage(msg);
        }
    }
}

```

最后客户端调用

```
public class Client {
    public static void main(String[] args) {
        //实例化中介者
        HouseMediator mediator = new HouseMediator();
        //实例化同事对象，传入中介者实例
        Landlord landlord = new Landlord("包租婆",mediator);
        Tenant tenant = new Tenant("小猪",mediator);
        //为中介者传入同事实例
        mediator.setLandlord(landlord);
        mediator.setTenant(tenant);
        //调用
        landlord.contact("单间500一个月，有兴趣吗？");
        tenant.contact("热水器，空调，网线有吗？");
        landlord.contact("都有。");
        tenant.contact("好吧，我租了。");
    }
}
```

运行结果：

```
【房东】包租婆：单间500一个月，有兴趣吗？
【房客】小猪：热水器，空调，网线有吗？
【房东】包租婆：都有。
【房客】小猪：好吧，我租了。
```

### 访问者模式

在访问者模式（Visitor Pattern）中，我们使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。这种类型的设计模式属于行为型模式。根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。

**核心其实就是**：**数据结构不变**，**操作可变**，结构与操作解耦的一种模式。

**意图：**主要将数据结构与数据操作分离。

**主要解决：**稳定的数据结构和易变的操作耦合问题。

**何时使用：**需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中。

**如何解决：**在被访问的类里面加一个对外提供接待访问者的接口。

**关键代码：**在数据基础类里面有一个方法接受访问者，将自身引用传入访问者。

**应用实例：**您在朋友家做客，您是访问者，朋友接受您的访问，您通过朋友的描述，然后对朋友的描述做出一个判断，这就是访问者模式。

**优点：**

1.**好的扩展性**，能在不修改结构元素情况下，为元素添加新功能；

2.**好的复用性**，通过访问者来定义整个对象结构统一的功能，从而提高复用程度；

3.**分离无关行为**，把相关的行为封装在一起，构成一个访问者，这样每一个访问者的功能都比较单一。

**缺点：**

1.不适用于结构中类经常变化的情况

2.破坏封装性，具体元素对访问者公布细节

**使用场景：** 

1. 对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作
2. 需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，也不希望在增加新操作时修改这些类。

**注意事项：**访问者可以对功能进行统一，可以做报表、UI、拦截器与过滤器。

**五个角色**

- Visitor：抽象访问者，为每一个具体访问者角色创建一个访问操作接口。
- ConcreteVisitor：具体访问者，实现 Vistor 接口，并实现独自的操作。
- Element：元素角色，定义一个接收访问者方法，以访问者作为参数。
- ConcreteElement：具体元素角色，实现Element接口并实现接收操作。
- ObjectStructure：对象结构，使访问者能够访问到对应的元素。

**UML类图**

![6.png](https://i.loli.net/2019/06/05/5cf79a834341212207.png)

#### 示例

我们将创建一个定义接受操作的 *ComputerPart* 接口。*Keyboard*、*Mouse*、*Monitor* 和 *Computer* 是实现了 *ComputerPart* 接口的实体类。我们将定义另一个接口 *ComputerPartVisitor*，它定义了访问者类的操作。*Computer* 使用实体访问者来执行相应的动作。

*VisitorPatternDemo*，我们的演示类使用 *Computer*、*ComputerPartVisitor* 类来演示访问者模式的用法。

![1.jpg](https://i.loli.net/2019/06/05/5cf7a54ee37e711861.jpg)

1. 定义一个表示元素的接口。

   ```
   //ComputerPart.java
   public interface ComputerPart {
      public void accept(ComputerPartVisitor computerPartVisitor);
   }
   
   ```

2. 创建扩展了上述类的实体类。

   ```
   
   Keyboard.java
   public class Keyboard  implements ComputerPart {
    
      @Override
      public void accept(ComputerPartVisitor computerPartVisitor) {
         computerPartVisitor.visit(this);
      }
   }
   Monitor.java
   public class Monitor  implements ComputerPart {
    
      @Override
      public void accept(ComputerPartVisitor computerPartVisitor) {
         computerPartVisitor.visit(this);
      }
   }
   
   //Mouse.java
   public class Mouse  implements ComputerPart {
    
      @Override
      public void accept(ComputerPartVisitor computerPartVisitor) {
         computerPartVisitor.visit(this);
      }
   }
   
   //Computer.java
   public class Computer implements ComputerPart {
      
      ComputerPart[] parts;
    
      public Computer(){
         parts = new ComputerPart[] {new Mouse(), new Keyboard(), new Monitor()};      
      } 
    
    
      @Override
      public void accept(ComputerPartVisitor computerPartVisitor) {
         for (int i = 0; i < parts.length; i++) {
            parts[i].accept(computerPartVisitor);
         }
         computerPartVisitor.visit(this);
      }
   }
   
   ```

3. 定义一个表示访问者的接口。

   ```
   //ComputerPartVisitor.java
   public interface ComputerPartVisitor {
      public void visit(Computer computer);
      public void visit(Mouse mouse);
      public void visit(Keyboard keyboard);
      public void visit(Monitor monitor);
   }
   
   ```

4. 创建实现了上述类的实体访问者。

   ```
   //ComputerPartDisplayVisitor.java
   public class ComputerPartDisplayVisitor implements ComputerPartVisitor {
    
      @Override
      public void visit(Computer computer) {
         System.out.println("Displaying Computer.");
      }
    
      @Override
      public void visit(Mouse mouse) {
         System.out.println("Displaying Mouse.");
      }
    
      @Override
      public void visit(Keyboard keyboard) {
         System.out.println("Displaying Keyboard.");
      }
    
      @Override
      public void visit(Monitor monitor) {
         System.out.println("Displaying Monitor.");
      }
   }
   
   ```

5. 使用 *ComputerPartDisplayVisitor* 来显示 *Computer* 的组成部分。

   ```
   //VisitorPatternDemo.java
   public class VisitorPatternDemo {
      public static void main(String[] args) {
    
         ComputerPart computer = new Computer();
         computer.accept(new ComputerPartDisplayVisitor());
      }
   }
   ```

6. 运行结果

   ```
   Displaying Mouse.
   Displaying Keyboard.
   Displaying Monitor.
   Displaying Computer.
   ```

### 责任链模式

顾名思义，责任链模式（Chain of Responsibility Pattern）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。

在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

使多个对象都有机会处理请求，从而避免请求的发送者与接收者之间的耦合关系，将这个对象连成一条链，并沿着这条链传递该请求，知道有一个对象处理它为止。

**意图：**避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。

**主要解决：**职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。

**何时使用：**在处理消息的时候以过滤很多道。

**如何解决：**拦截的类都实现统一接口。

**关键代码：**Handler 里面聚合它自己，在 HandlerRequest 里判断是否合适，如果没达到条件则向下传递，向谁传递之前 set 进去。

**应用实例：** 

1. 红楼梦中的"击鼓传花"。 
2. JS 中的事件冒泡。 
3. JAVA WEB 中 Apache Tomcat 对 Encoding 的处理，Struts2 的拦截器，jsp servlet 的 Filter。 

**优点：** 

1. 降低耦合度。它将请求的发送者和接收者解耦。 
2. 简化了对象。使得对象不需要知道链的结构。
3. 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。 
4. 增加新的请求处理类很方便。 

**缺点：** 

1. 产生许多细颗粒对象

2. 不一定能被处理，可能到末端也没被处理或者中间写错

3. 比较长的责任链，可能涉及多个处理对象，性能问题，还有调试不方便

4. 建链不当，可能造成循环调用，导致系统进入死循环；

**使用场景：** 

1. 有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。
2. 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。 
3. 可动态指定一组对象处理请求。 

**注意事项：**在 JAVA WEB 中遇到很多应用。

**两个角色**

- Handler：抽象处理者，定义抽象请求处理方法，还定义一个抽象处理者对象作为
  其下家的引用，通过该引用，处理者可以连成一条链。
- ConcreteHandler：具体处理者，处理他所负责的请求，如果可处理就处理，不能
  处理就把请求转发给下家。

**UML类图**

![7.png](https://i.loli.net/2019/06/05/5cf7ab9ad0dc012929.png)



#### 示例

我们创建抽象类 AbstractLogger，带有详细的日志记录级别。然后我们创建三种类型的记录器，都扩展了 AbstractLogger。每个记录器消息的级别是否属于自己的级别，如果是则相应地打印出来，否则将不打印并把消息传给下一个记录器。

![2.jpg](https://i.loli.net/2019/06/05/5cf7ac3089d7869586.jpg)

1. 创建抽象的记录器类。

   ```
   //AbstractLogger.java
   public abstract class AbstractLogger {
      public static int INFO = 1;
      public static int DEBUG = 2;
      public static int ERROR = 3;
    
      protected int level;
    
      //责任链中的下一个元素
      protected AbstractLogger nextLogger;
    
      public void setNextLogger(AbstractLogger nextLogger){
         this.nextLogger = nextLogger;
      }
    
      public void logMessage(int level, String message){
         if(this.level <= level){
            write(message);
         }
        
           if (nextLogger !=null) {
               nextLogger.logMessage(level, message);
           }
       
      }
    
      abstract protected void write(String message);
      
   }
   
   ```

2. 创建扩展了该记录器类的实体类。

   ```
   //ConsoleLogger.java
   public class ConsoleLogger extends AbstractLogger {
    
      public ConsoleLogger(int level){
         this.level = level;
      }
    
      @Override
      protected void write(String message) {    
         System.out.println("Standard Console::Logger: " + message);
      }
   }
   //ErrorLogger.java
   public class ErrorLogger extends AbstractLogger {
    
      public ErrorLogger(int level){
         this.level = level;
      }
    
      @Override
      protected void write(String message) {    
         System.out.println("Error Console::Logger: " + message);
      }
   }
   //FileLogger.java
   public class FileLogger extends AbstractLogger {
    
      public FileLogger(int level){
         this.level = level;
      }
    
      @Override
      protected void write(String message) {    
         System.out.println("File::Logger: " + message);
      }
   }
   
   ```

3. 创建不同类型的记录器。赋予它们不同的错误级别，并在每个记录器中设置下一个记录器。每个记录器中的下一个记录器代表的是链的一部分。

   ```
   //ChainPatternDemo.java
   public class ChainPatternDemo {
      
      private static AbstractLogger getChainOfLoggers(){
    
         AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
         AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
         AbstractLogger consoleLogger = new ConsoleLogger(AbstractLogger.INFO);
    
         errorLogger.setNextLogger(fileLogger);
         fileLogger.setNextLogger(consoleLogger);
    
         return errorLogger;  
      }
    
      public static void main(String[] args) {
         AbstractLogger loggerChain = getChainOfLoggers();
    
         loggerChain.logMessage(AbstractLogger.INFO, "This is an information.");
    
         loggerChain.logMessage(AbstractLogger.DEBUG, 
            "This is a debug level information.");
    
         loggerChain.logMessage(AbstractLogger.ERROR, 
            "This is an error information.");
      }
   }
   
   ```

4. 执行程序，输出结果

   ```
   Standard Console::Logger: This is an information.
   File::Logger: This is a debug level information.
   Standard Console::Logger: This is a debug level information.
   Error Console::Logger: This is an error information.
   File::Logger: This is an error information.
   Standard Console::Logger: This is an error information.
   ```

另外责任链模式还分**纯与不纯**

- **纯责任链**，要么承担全部责任，要么责任推个下家，**不允许**在某处**承担**了部分 
   或者全部**责任**，然后**又把责任推给下家**。
- **不纯责任链**，责任在某处部分或全部被处理后，还向下传递。

参考：

1. [Java设计模式系列之责任链模式](https://www.cnblogs.com/ysw-go/p/5432921.html)
2. [责任链模式实现的三种方式](https://www.cnblogs.com/lizo/p/7503862.html)

### 状态模式

在状态模式（State Pattern）中，类的行为是基于它的状态改变的。这种类型的设计模式属于行为型模式。

在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象。

**意图：**允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。

**主要解决：**对象的行为依赖于它的状态（属性），并且可以根据它的状态改变而改变它的相关行为。

**何时使用：**代码中包含大量与对象状态有关的条件语句。

**如何解决：**将各种具体的状态类抽象出来。

**关键代码：**通常命令模式的接口中只有一个方法。而状态模式的接口中有一个或者多个方法。而且，状态模式的实现类的方法，一般返回值，或者是改变实例变量的值。也就是说，状态模式一般和对象的状态有关。实现类的方法有不同的功能，覆盖接口中的方法。状态模式和命令模式一样，也可以用于消除  if...else 等条件选择语句。

**应用实例：** 

1. 打篮球的时候运动员可以有正常状态、不正常状态和超常状态。 
2. 曾侯乙编钟中，'钟是抽象接口','钟A'等是具体状态，'曾侯乙编钟'是具体环境（Context）。

**优点：** 

1. 封装了转换规则。 
2. 枚举可能的状态，在枚举状态之前需要确定状态种类。
3. 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。 
4. 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。 
5. 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数。 

**缺点：** 

1. 状态模式的使用必然会增加系统类和对象的个数。 
2. 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱。
3. 状态模式对"开闭原则"的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态，而且修改某个状态类的行为也需修改对应类的源代码。 

**使用场景：** 

1. 对象的行为依赖于它的状态(属性)并且可以根据它的状态改变而改变它的相关行为；

2. 代码中包含大量与对象状态有关的条件语句

**注意事项：**在行为受状态约束的时候使用状态模式，而且状态不超过 5 个。

**三个角色**

- Context：上下文环境，定义客户感兴趣的接口，维护一个State子类的实例，该实例定义了对象的当前状态
- State：抽象状态，定义一个接口以封装与 Context 的一个特定状态相关的行为。
- ConcreteState：具体状态，实现抽象状态中定义的接口，从而达到不同状态下的不同行为

![8.png](https://i.loli.net/2019/06/05/5cf7b10280f2714414.png)

#### 示例

状态模式非常简单，就是抽象出状态State，然后实现该接口，然后具体化不同状态， 做不同的操作，然后写一个Context，里面存储一个State的实例，然后定义一个 可以修改State实例的方法，并在里面去调用实例的行为方法。写个不同时刻做 不同事情的例子帮助理解下~

1. 创建一个接口。

   ```
   //State.java
   public interface State {
      public void doAction(Context context);
   }
   
   ```

2. 创建实现接口的实体类。

   ```
   //StartState.java
   public class StartState implements State {
    
      public void doAction(Context context) {
         System.out.println("Player is in start state");
         context.setState(this); 
      }
    
      public String toString(){
         return "Start State";
      }
   }
   
   //StopState.java
   public class StopState implements State {
    
      public void doAction(Context context) {
         System.out.println("Player is in stop state");
         context.setState(this); 
      }
    
      public String toString(){
         return "Stop State";
      }
   }
   
   ```

3. 创建 *Context* 类。

   ```
   //Context.java
   public class Context {
      private State state;
    
      public Context(){
         state = null;
      }
    
      public void setState(State state){
         this.state = state;     
      }
    
      public State getState(){
         return state;
      }
   }
   
   ```

4. 使用 *Context* 来查看当状态 *State* 改变时的行为变化。

   ```
   //StatePatternDemo.java
   public class StatePatternDemo {
      public static void main(String[] args) {
         Context context = new Context();
    
         StartState startState = new StartState();
         startState.doAction(context);
    
         System.out.println(context.getState().toString());
    
         StopState stopState = new StopState();
         stopState.doAction(context);
    
         System.out.println(context.getState().toString());
      }
   }
   
   ```

5. 运行结果

   ```
   Player is in start state
   Start State
   Player is in stop state
   Stop State
   ```



好的，状态模式还是非常简单的，可能细心的你发现了状态模式和策略模式非常的类似，其实还是有些不同的，比如Context类，策略模式只是一个委托作用，负责算法替换；而状态模式不仅仅是委托作用，还具有等级状态变化的功能，与具体的状态类协作，共同完成状态切换的任务。

策略模式封装的是不同算法，算法间没有交互，已达到算法可以自由切换的目的；状态模式封装的是不同状态，以达到状态切换行为随之发生改变的目的。

### 模板模式

在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。这种类型的设计模式属于行为型模式。

**意图：**定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

**主要解决：**一些方法通用，却在每一个子类都重新写了这一方法。

**何时使用：**有一些通用的方法。

**如何解决：**将这些通用算法抽象出来。

**关键代码：**在抽象类实现，其他步骤在子类实现。

**应用实例：** 

1. 在造房子的时候，地基、走线、水管都一样，只有在建筑的后期才有加壁橱加栅栏等差异。
2. 西游记里面菩萨定好的 81 难，这就是一个顶层的逻辑骨架。
3. spring 中对 Hibernate 的支持，将一些已经定好的方法封装起来，比如开启事务、获取 Session、关闭 Session 等，程序员不重复写那些已经规范好的代码，直接丢一个实体就可以保存。 

**优点：** 

1. 封装不变部分，扩展可变部分。 
2. 提取公共代码，便于维护。 
3. 行为由父类控制，子类实现。 

**缺点：**每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

**使用场景：** 

- 对一些复杂的算法进行分割，将其算法中固定不变的部分设计为模板方法和父类具体方法， 
   而一些可以改变的细节由其子类来实现。
- 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。
- 需要通过子类来决定父类算法中某个步骤是否执行，实现子类对父类的反向控制。

**注意事项：**为防止恶意操作，一般模板方法都加上 final 关键词。

**两个角色**

- **AbstractClass**：**抽象模板**，定义了模板结构，让子类去具体实现
- **ConcreteClass**：**具体模板**，具体实现抽象类的抽象方法

#### 示例

我们将创建一个定义操作的 *Game* 抽象类，其中，模板方法设置为 final，这样它就不会被重写。*Cricket* 和 *Football* 是扩展了 *Game* 的实体类，它们重写了抽象类的方法。

*TemplatePatternDemo*，我们的演示类使用 *Game* 来演示模板模式的用法。

![3.jpg](https://i.loli.net/2019/06/05/5cf7b3e877fe936058.jpg)

1. 创建一个抽象类，它的模板方法被设置为 final。

   ```
   //Game.java
   public abstract class Game {
      abstract void initialize();
      abstract void startPlay();
      abstract void endPlay();
    
      //模板
      public final void play(){
    
         //初始化游戏
         initialize();
    
         //开始游戏
         startPlay();
    
         //结束游戏
         endPlay();
      }
   }
   
   ```

2. 创建扩展了上述类的实体类。

   ```
   //Cricket.java
   public class Cricket extends Game {
    
      @Override
      void endPlay() {
         System.out.println("Cricket Game Finished!");
      }
    
      @Override
      void initialize() {
         System.out.println("Cricket Game Initialized! Start playing.");
      }
    
      @Override
      void startPlay() {
         System.out.println("Cricket Game Started. Enjoy the game!");
      }
   }
   
   //Football.java
   public class Football extends Game {
    
      @Override
      void endPlay() {
         System.out.println("Football Game Finished!");
      }
    
      @Override
      void initialize() {
         System.out.println("Football Game Initialized! Start playing.");
      }
    
      @Override
      void startPlay() {
         System.out.println("Football Game Started. Enjoy the game!");
      }
   }
   
   ```

3. 使用 *Game* 的模板方法 play() 来演示游戏的定义方式。

   ```
   //TemplatePatternDemo.java
   public class TemplatePatternDemo {
      public static void main(String[] args) {
    
         Game game = new Cricket();
         game.play();
         System.out.println();
         game = new Football();
         game.play();      
      }
   }
   
   ```

4. 运行结果

   ```
   Cricket Game Initialized! Start playing.
   Cricket Game Started. Enjoy the game!
   Cricket Game Finished!
   
   Football Game Initialized! Start playing.
   Football Game Started. Enjoy the game!
   Football Game Finished!
   ```

### 空对象模式

在空对象模式（Null Object Pattern）中，一个空对象取代 NULL 对象实例的检查。Null 对象不是检查空值，而是反应一个不做任何动作的关系。这样的 Null 对象也可以在数据不可用的时候提供默认的行为。

在空对象模式中，我们创建一个指定各种要执行的操作的抽象类和扩展该类的实体类，还创建一个未对该类做任何实现的空对象类，该空对象类将无缝地使用在需要检查空值的地方。

#### 示例

我们将创建一个定义操作（在这里，是客户的名称）的 *AbstractCustomer* 抽象类，和扩展了 *AbstractCustomer* 类的实体类。工厂类 *CustomerFactory* 基于客户传递的名字来返回 *RealCustomer* 或 *NullCustomer* 对象。

*NullPatternDemo*，我们的演示类使用 *CustomerFactory* 来演示空对象模式的用法。

![4.jpg](https://i.loli.net/2019/06/05/5cf7b9954bb8180060.jpg)

1. 创建一个抽象类。

   ```
   //AbstractCustomer.java
   public abstract class AbstractCustomer {
      protected String name;
      public abstract boolean isNil();
      public abstract String getName();
   }
   
   ```

2. 创建扩展了上述类的实体类。

   ```
   //RealCustomer.java
   public class RealCustomer extends AbstractCustomer {
    
      public RealCustomer(String name) {
         this.name = name;    
      }
      
      @Override
      public String getName() {
         return name;
      }
      
      @Override
      public boolean isNil() {
         return false;
      }
   }
   //NullCustomer.java
   public class NullCustomer extends AbstractCustomer {
    
      @Override
      public String getName() {
         return "Not Available in Customer Database";
      }
    
      @Override
      public boolean isNil() {
         return true;
      }
   }
   
   ```

3. 创建 *CustomerFactory* 类。

   ```
   //CustomerFactory.java
   public class CustomerFactory {
      
      public static final String[] names = {"Rob", "Joe", "Julie"};
    
      public static AbstractCustomer getCustomer(String name){
         for (int i = 0; i < names.length; i++) {
            if (names[i].equalsIgnoreCase(name)){
               return new RealCustomer(name);
            }
         }
         return new NullCustomer();
      }
   }
   
   ```

4. 使用 *CustomerFactory*，基于客户传递的名字，来获取 *RealCustomer* 或 *NullCustomer* 对象。

   ```
   //NullPatternDemo.java
   public class NullPatternDemo {
      public static void main(String[] args) {
    
         AbstractCustomer customer1 = CustomerFactory.getCustomer("Rob");
         AbstractCustomer customer2 = CustomerFactory.getCustomer("Bob");
         AbstractCustomer customer3 = CustomerFactory.getCustomer("Julie");
         AbstractCustomer customer4 = CustomerFactory.getCustomer("Laura");
    
         System.out.println("Customers");
         System.out.println(customer1.getName());
         System.out.println(customer2.getName());
         System.out.println(customer3.getName());
         System.out.println(customer4.getName());
      }
   }
   
   ```

5. 运行结果

   ```
   Customers
   Rob
   Not Available in Customer Database
   Julie
   Not Available in Customer Database
   ```

   