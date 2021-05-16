---
title: 设计模式--结构型（下）
date: 2019-05-23 16:18:59
tags:
 - Java
 - 设计模式
categories:
 - Java
 - 设计模式
---

### 外观模式

外观模式（Facade Pattern）隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。这种类型的设计模式属于结构型模式，它向现有的系统添加一个接口，来隐藏系统的复杂性。

这种模式涉及到一个单一的类，该类提供了客户端请求的简化方法和对现有系统类方法的委托调用。

<!--more-->

**意图：**为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

**主要解决：**降低访问复杂系统的内部子系统时的复杂度，简化客户端与之的接口。

**何时使用：** 1、客户端不需要知道系统内部的复杂联系，整个系统只需提供一个"接待员"即可。 2、定义系统的入口。 

**如何解决：**客户端不与系统耦合，外观类与系统耦合。

**关键代码：**在客户端和复杂系统之间再加一层，这一层将调用顺序、依赖关系等处理好。

**应用实例：** 

1. 去医院看病，可能要去挂号、门诊、划价、取药，让患者或患者家属觉得很复杂，如果有提供接待人员，只让接待人员来处理，就很方便。
2. JAVA 的三层开发模式。 

**优点：** 

1. 减少系统相互依赖。 
2. 提高灵活性。
3. 提高了安全性。 

**缺点：**不符合开闭原则，如果要改东西很麻烦，继承重写都不合适。

**使用场景：** 

1. 为复杂的模块或子系统提供外界访问的模块。 
2. 子系统相对独立
3. 预防低水平人员带来的风险。 

**注意事项：**在层次化结构中，可以使用外观模式定义系统中每一层的入口。

**两个角色**

- Facade：外观角色，客户端可以调用他的方法，在外观角色
  中可以知道相关子系统的功能和责任；在正常情况下，它将所有从客户
  端发来的请求委派到相应的子系统去，传递给相应的子系统对象处理。
- Subsystem：子系统角色，实现子系统的功能，处理外观类
  指派的任务，注意子系统类不含有外观类的引用

UML类图

![9.png](https://i.loli.net/2019/06/05/5cf7452f96e1851220.png)

#### 示例

每个Computer都有CPU、Memory、Disk。在Computer开启和关闭的时候，相应的部件也会开启和关闭，所以，使用了该外观模式后，会使用户和部件之间解耦。如：

![10.png](https://i.loli.net/2019/06/05/5cf745710f73b53321.png)

1. 首先是子系统类：

   ```
   //CPU.java
   import org.apache.log4j.Logger;
   
   /**
    * cpu子系统类
    *
    */
   public class CPU 
   {
       public static final Logger LOGGER = Logger.getLogger(CPU.class);
       public void start()
       {
           LOGGER.info("cpu is start...");
       }
       
       public void shutDown()
       {
           LOGGER.info("CPU is shutDown...");
       }
   }
   
   //Disk.java
   import org.apache.log4j.Logger;
   
   /**
    * Disk子系统类
    *
    */
   public class Disk
   {
       public static final Logger LOGGER = Logger.getLogger(Disk.class);
       public void start()
       {
           LOGGER.info("Disk is start...");
       }
       
       public void shutDown()
       {
           LOGGER.info("Disk is shutDown...");
       }
   }
   
   //Memory.java
   import org.apache.log4j.Logger;
   
   /**
    * Memory子系统类
    *
    */
   public class Memory
   {
       public static final Logger LOGGER = Logger.getLogger(Memory.class);
       public void start()
       {
           LOGGER.info("Memory is start...");
       }
       
       public void shutDown()
       {
           LOGGER.info("Memory is shutDown...");
       }
   }
   
   
   ```

2. 然后是，门面类Facade

   ```
   //Computer.java
   import org.apache.log4j.Logger;
   
   import com.huawei.facadeDesign.children.CPU;
   import com.huawei.facadeDesign.children.Disk;
   import com.huawei.facadeDesign.children.Memory;
   
   
   /**
    * 门面类（核心）
    *
    */
   public class Computer
   {
       public static final Logger LOGGER = Logger.getLogger(Computer.class);
       
       private CPU cpu;
       private Memory memory;
       private Disk disk;
       public Computer()
       {
           cpu = new CPU();
           memory = new Memory();
           disk = new Disk();
       }
       public void start()
       {
           LOGGER.info("Computer start begin");
           cpu.start();
           disk.start();
           memory.start();
           LOGGER.info("Computer start end");
       }
       
       public void shutDown()
       {
           LOGGER.info("Computer shutDown begin");
           cpu.shutDown();
           disk.shutDown();
           memory.shutDown();
           LOGGER.info("Computer shutDown end...");
       }
   }
   ```

3. 最后为，客户角色。

   ```
   import org.apache.log4j.Logger;
   
   import com.huawei.facadeDesign.facade.Computer;
   
   /**
    * 客户端类
    *
    */
   public class Cilent {
       public static final Logger LOGGER = Logger.getLogger(Cilent.class);
       public static void main(String[] args) 
       {
           Computer computer = new Computer();
           computer.start();
           LOGGER.info("=================");
           computer.shutDown();
       }
   
   }
   ```

参考：

1. [JAVA设计模式之门面模式（外观模式）](https://www.runoob.com/w3cnote/facade-pattern-3.html)
2. [如何让孩子爱上设计模式 ——11.外观模式(Facade Pattern)](https://blog.csdn.net/coder_pig/article/details/55002971)

### 享元模式

享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。

享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象。

**意图：**运用共享技术有效地支持大量细粒度的对象。

**主要解决：**在有大量对象时，有可能会造成内存溢出，我们把其中共同的部分抽象出来，如果有相同的业务请求，直接返回在内存中已有的对象，避免重新创建。

**何时使用：** 

1. 系统中有大量对象。
2. 这些对象消耗大量内存。 
3. 这些对象的状态大部分可以外部化。 
4. 这些对象可以按照内蕴状态分为很多组，当把外蕴对象从对象中剔除出来时，每一组对象都可以用一个对象来代替。 
5. 系统不依赖于这些对象身份，这些对象是不可分辨的。 

**如何解决：**用唯一标识码判断，如果在内存中有，则返回这个唯一标识码所标识的对象。

**关键代码：**用 HashMap 存储这些对象。

**应用实例：** 

1. JAVA 中的 String，如果有则返回，如果没有则创建一个字符串保存在字符串缓存池里面。 
2. 数据库的数据池。 

**优点：**大大减少对象的创建，降低系统的内存，使效率提高。

**缺点：**提高了系统的复杂度，需要分离出外部状态和内部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱。

**使用场景：** 

1. 系统有大量相似对象。 
2. 需要缓冲池的场景。 

**注意事项：** 

1. 注意划分外部状态和内部状态，否则可能会引起线程安全问题。
2. 这些类必须有一个工厂对象加以控制。 

**内部状态与外部状态**

内部状态：固定不变可共享的的部分，存储在享元对象内部，比如这里的花色。
外部状态：可变不可共享的部分，一般由客户端传入享元对象内部，比如这里的大小。

**三个角色**

- Flyweight：享元对象的抽象父类或者接口，通过这个接口，享元对象可以接受并作用于外部状态；
- ConcreteFlyweight：具体享元实现对象，继承或实现Flyweight并为内部状态增加存储空间。
- FlyweightFactory：享元工厂，创建并管理共享的享元对象，并对外提供访问共享享元对象的接口。

**UML类图**

![3.png](https://i.loli.net/2019/06/05/5cf74854aa5c812752.png)



#### 示例

有时在开发中，可能我们需要创建大量的相同的重复对象，比如游戏开发中，场景贴图的，一个森林的场景，要有有成千上万的树，如果为每棵树都实例化不同的模型，估计会把你电脑给炸了。使用享元模式可以解决这个问题，抽取出所有树对象的共有属性，并转移到一个单独的类中，然后只需要一个示例就可以了，然后森林里的每棵树对这个实例做一次引用：

![1.png](https://i.loli.net/2019/06/05/5cf7475f7d03863726.png)
![2.png](https://i.loli.net/2019/06/05/5cf7475f948ac86687.png)



这里举个扑克牌的例子来帮助理解享元模式。 （这里假定没有大小王，只有52张牌，四种花色）

**普通套路实现扑克牌程序**

如果让你来实现一个简单的扑克牌程序，你的代码可能是这样：牌有花色和大小，先创建一个牌类；然后初始化52张牌，然后随机发五张牌；好的，正常实现，但是却初始化了52个Card对象，真的有必要 创建那么多对象吗？如果使用享元模式需要创建几个对象？ 

```
//Card.java
class Card {
    private String color;
    private String num;

    public Card() { }

    Card(String color, String num) {
        this.color = color;
        this.num = num;
    }

    public String getColor() { return color; }

    public void setColor(String color) { this.color = color; }

    public String getNum() { return num; }

    public void setNum(String num) { this.num = num; }

    @Override public String toString() {
        return "Card{" + "花色='" + color + '\'' + ", 大小='" + num + '\'' + '}';
    }
}

//Player.java
import java.util.ArrayList;
import java.util.List;

/**
 * 描述：
 *
 */


public class Player {
    public static void main(String[] args) {
        String[] colors = new String[] {"黑桃","红心","梅花","方块"};
        List<Card> cards = new ArrayList<>();
        for(int i = 0;i < 4; i++ ) {
            for (int j = 1;j <= 13;j ++) {
                switch (j) {
                    case 11: cards.add(new Card(colors[i],"J")); break;
                    case 12: cards.add(new Card(colors[i],"Q")); break;
                    case 13: cards.add(new Card(colors[i],"K")); break;
                    default: cards.add(new Card(colors[i],j + "")); break;
                }
            }
        }
        System.out.println("扑克牌初始化完毕，共：" + cards.size() + "张");
        System.out.println("随机分5张牌：");
        for (int k = 0; k < 5; k ++){
            System.out.println(cards.get((int)(Math.random()*52)));
        }
    }
}
```

**享元模式实现扑克牌程序**

抽取下牌共有的属性是：花色和大小，花色固定四种，不同是大小， 这里涉及到享元模式的内部状态和外部状态，这个等下讲。 写一个卡牌的父类，然后写四个花色的类继承父类

```
abstract class Card {
    abstract void showCard(String num);  //传入外部状态参数，大小
}

public class ClubCard extends Card{

    public ClubCard() { super(); }

    @Override public void showCard(String num) { System.out.println("梅花：" + num); }
}


public class HeartCard extends Card{

    public HeartCard() { super(); }

    @Override public void showCard(String num) { System.out.println("红桃：" + num); }
}

public class DiamondCard extends Card{

    public DiamondCard() { super(); }

    @Override public void showCard(String num) { System.out.println("方块：" + num); }
}

public class SpadeCard extends Card{

    public SpadeCard() { super(); }

    @Override public void showCard(String num) { System.out.println("黑桃：" + num); }
}

```

接着是最关键的**享元工厂**，创建并管理共享的享元对象，并提供访问享元对象的接口：

```
//PokerFactory.java
import java.util.HashMap;
import java.util.Map;


public class PokerFactory {

    static final int Spade = 0;  //黑桃
    static final int Heart  = 1; //红桃
    static final int Club  = 2; //梅花
    static final int Diamond  = 3;   //方块

    public static Map<Integer, Card> pokers = new HashMap<>();

    public static Card getPoker(int color) {
        if (pokers.containsKey(color)) {
            System.out.print("对象已存在，对象复用...");
            return pokers.get(color);
        } else {
            System.out.print("对象不存在，新建对象...");
            Card card;
            switch (color) {
                case Spade: card = new SpadeCard(); break;
                case Heart: card = new HeartCard(); break;
                case Club: card = new ClubCard(); break;
                case Diamond: card = new DiamondCard(); break;
                default: card = new SpadeCard(); break;
            }
            pokers.put(color,card);
            return card;
        }
    }
}
```

接着客户端调用，模拟发十张牌

```
//Player
public class Player {
    public static void main(String[] args) {
        for (int k = 0; k < 10; k ++){
            Card card = null;
            //随机花色
            switch ((int)(Math.random()*4)) {
                case 0: card = PokerFactory.getPoker(PokerFactory.Spade); break;
                case 1: card = PokerFactory.getPoker(PokerFactory.Heart); break;
                case 2: card = PokerFactory.getPoker(PokerFactory.Club); break;
                case 3: card = PokerFactory.getPoker(PokerFactory.Diamond); break;
            }
            if(card != null) {
                //随机大小
                int num = (int)(Math.random()*13 + 1);
                switch (num) {
                    case 11: card.showCard("J"); break;
                    case 12: card.showCard("Q"); break;
                    case 13: card.showCard("K"); break;
                    default: card.showCard(num+""); break;
                }
            }
        }
    }
}


```

### 代理模式

在代理模式（Proxy Pattern）中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式。

在代理模式中，我们创建具有现有对象的对象，以便向外界提供功能接口。

**意图：**为其他对象提供一种代理以控制对这个对象的访问。

**主要解决：**在直接访问对象时带来的问题，比如说：要访问的对象在远程的机器上。在面向对象系统中，有些对象由于某些原因（比如对象创建开销很大，或者某些操作需要安全控制，或者需要进程外的访问），直接访问会给使用者或者系统结构带来很多麻烦，我们可以在访问此对象时加上一个对此对象的访问层。

**何时使用：**想在访问一个类时做一些控制。

**如何解决：**增加中间层。

**关键代码：**实现与被代理类组合。

**应用实例：** 

1. Windows 里面的快捷方式。
2. 猪八戒去找高翠兰结果是孙悟空变的，可以这样理解：把高翠兰的外貌抽象出来，高翠兰本人和孙悟空都实现了这个接口，猪八戒访问高翠兰的时候看不出来这个是孙悟空，所以说孙悟空是高翠兰代理类。 
3. 买火车票不一定在火车站买，也可以去代售点。 
4. 一张支票或银行存单是账户中资金的代理。支票在市场交易中用来代替现金，并提供对签发人账号上资金的控制。
5. spring aop。 

**优点：** 

1. 职责清晰。 
2. 高扩展性。 
3. 智能化。 

**缺点：** 

1. 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。 
2. 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。 

**使用场景：**按职责来划分，通常有以下使用场景： 

1. 远程代理:为一个对象在不同的地址空间提供局部代表，可以隐藏 对象存在于不同地址空间的事实，把占资源和操作移到更好的硬件上。
2. 虚拟代理:用消耗资源较小的对象替代多的对象。
3. Copy-on-Write 代理。
4. 保护（Protect or Access）代理:控制目标对象访问，给不同用户提供不同访问权限
5. Cache代理。 
6. 防火墙（Firewall）代理。
7. 同步化（Synchronization）代理。 
8. 智能引用（Smart Reference）代理:访问对象时附加额外操作

**注意事项：** 

1. 和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口。 
2. 和装饰器模式的区别：装饰器模式为了增强功能，而代理模式是为了加以控制。

**三个角色**

- **Subject**：**抽象对象**，声明真实角色与代理角色的公共接口。
- **RealSubject**：**真实对象**，代理角色所代表的真实对象，最终引用对象。
- **Proxy**：**代理对象**，包含对真实对象的引用从而操作真实对象， 
   相当于对真实对象进行封装。

**UML类图**

![4.png](https://i.loli.net/2019/06/05/5cf74b39332c078433.png)

#### 示例 

**静态代理**

静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类.

下面举个案例来解释:

模拟保存动作,定义一个保存动作的接口:IUserDao.java,然后目标对象实现这个接口的方法UserDao.java,此时如果使用静态代理方式,就需要在代理对象(UserDaoProxy.java)中也实现IUserDao接口.调用的时候通过调用代理对象的方法来调用目标对象.

需要注意的是,代理对象与目标对象要实现相同的接口,然后通过调用相同的方法来调用目标对象的方法

代码示例:
 接口:IUserDao.java

```
/**
 * 接口
 */
public interface IUserDao {

    void save();
}
```

目标对象:UserDao.java

```
/**
 * 接口实现
 * 目标对象
 */
public class UserDao implements IUserDao {
    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

代理对象:UserDaoProxy.java

```
/**
 * 代理对象,静态代理
 */
public class UserDaoProxy implements IUserDao{
    //接收保存目标对象
    private IUserDao target;
    public UserDaoProxy(IUserDao target){
        this.target=target;
    }

    public void save() {
        System.out.println("开始事务...");
        target.save();//执行目标对象的方法
        System.out.println("提交事务...");
    }
}
```

测试类:App.java

```
/**
 * 测试类
 */
public class App {
    public static void main(String[] args) {
        //目标对象
        UserDao target = new UserDao();

        //代理对象,把目标对象传给代理对象,建立代理关系
        UserDaoProxy proxy = new UserDaoProxy(target);

        proxy.save();//执行的是代理的方法
    }
}
```

**静态代理总结:**
 1.可以做到在不修改目标对象的功能前提下,对目标功能扩展.
 2.缺点:

- 因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护.

如何解决静态代理中的缺点呢?答案是可以使用动态代理方式

#### 动态代理

**动态代理有以下特点:**

1. 代理对象,不需要实现接口
2. 代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)
3. 动态代理也叫做:JDK代理,接口代理

**JDK 自带的动态代理**

-  java.lang.reflect.Proxy:生成动态代理类和对象；
-  java.lang.reflect.InvocationHandler（处理器接口）：可以通过invoke方法实现对真实角色的代理访问。

JDK实现代理只需要使用newProxyInstance方法,但是该方法需要接收三个参数,完整的写法是:

```
static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,InvocationHandler h )
```

注意该方法是在Proxy类中是静态方法,且接收的三个参数依次为:

- `ClassLoader loader,`:指定当前目标对象使用类加载器,获取加载器的方法是固定的
- `Class<?>[] interfaces,`:目标对象实现的接口的类型,使用泛型方式确认类型
- `InvocationHandler h`:事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入

代码示例:
 接口类IUserDao.java以及接口实现类,目标对象UserDao是一样的,没有做修改.在这个基础上,增加一个代理工厂类(ProxyFactory.java),将代理类写在这个地方,然后在测试类(需要使用到代理的代码)中先建立目标对象和代理对象的联系,然后代用代理对象的中同名方法

代理工厂类:ProxyFactory.java

```
/**
 * 创建动态代理对象
 * 动态代理不需要实现接口,但是需要指定接口类型
 */
public class ProxyFactory{

    //维护一个目标对象
    private Object target;
    public ProxyFactory(Object target){
        this.target=target;
    }

   //给目标对象生成代理对象
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开始事务2");
                        //执行目标对象方法
                        Object returnValue = method.invoke(target, args);
                        System.out.println("提交事务2");
                        return returnValue;
                    }
                }
        );
    }

}
```

测试类:App.java

```
/**
 * 测试类
 */
public class App {
    public static void main(String[] args) {
        // 目标对象
        IUserDao target = new UserDao();
        // 【原始的类型 class cn.itcast.b_dynamic.UserDao】
        System.out.println(target.getClass());

        // 给目标对象，创建代理对象
        IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
        // class $Proxy0   内存中动态生成的代理对象
        System.out.println(proxy.getClass());

        // 执行方法   【代理对象】
        proxy.save();
    }
}
```

**总结:**
 代理对象不需要实现接口,但是目标对象一定要实现接口,否则不能用动态代理

**Cglib 动态代理**

[Cglib](https://www.runoob.com/w3cnote/cglibcode-generation-library-intro.html) 动态代理是针对代理的类, 动态生成一个子类, 然后子类覆盖代理类中的方法, 如果是private或是final类修饰的方法,则不会被重写。

CGLIB是一个功能强大，高性能的代码生成包。它为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。通常可以使用Java的动态代理创建代理，但当要代理的类没有实现接口或者为了更好的性能，CGLIB是一个好的选择。

Cglib代理,也叫作子类代理,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.

- JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
- Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的interception(拦截)
- Cglib包的底层是通过使用一个小而块的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

CGLIB作为一个开源项目，其代码托管在github，地址为：<https://github.com/cglib/cglib>

Cglib子类代理实现方法:

1. 需要引入cglib的jar文件,但是Spring的核心包中已经包括了Cglib功能,所以直接引入`pring-core-3.2.5.jar`即可.
2. 引入功能包后,就可以在内存中动态构建子类
3. 代理的类不能为final,否则报错
4. 目标对象的方法如果为final/static,那么就不会被拦截,即不会执行目标对象额外的业务方法.

代码示例:
 目标对象类:UserDao.java

```
/**
 * 目标对象,没有实现任何接口
 */
public class UserDao {

    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

Cglib代理工厂:ProxyFactory.java

```
/**
 * Cglib子类代理工厂
 * 对UserDao在内存中动态构建一个子类对象
 */
public class ProxyFactory implements MethodInterceptor{
    //维护目标对象
    private Object target;

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance(){
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();

    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开始事务...");

        //执行目标对象的方法
        Object returnValue = method.invoke(target, args);

        System.out.println("提交事务...");

        return returnValue;
    }
}
```

测试类:

```
/**
 * 测试类
 */
public class App {

    @Test
    public void test(){
        //目标对象
        UserDao target = new UserDao();

        //代理对象
        UserDao proxy = (UserDao)new ProxyFactory(target).getProxyInstance();

        //执行代理对象的方法
        proxy.save();
    }
}
```

在Spring的AOP编程中:

- 如果加入容器的目标对象有实现接口,用JDK代理

- 如果目标对象没有实现接口,用Cglib代理

参考：

1. [如何让孩子爱上设计模式 ——13.代理模式(Proxy Pattern)](https://blog.csdn.net/coder_pig/article/details/60138896)
2. [Java的三种代理模式](https://www.cnblogs.com/cenyu/p/6289209.html)
3. [设计模式之---代理模式(AOP的原理)](https://blog.csdn.net/qq_34178598/article/details/78630934)
4. [CGLIB原理及实现机制](https://blog.csdn.net/gyshun/article/details/81000997)

### 过滤器模式

过滤器模式（Filter Pattern）或标准模式（Criteria  Pattern）是一种设计模式，这种模式允许开发人员使用不同的标准来过滤一组对象，通过逻辑运算以解耦的方式把它们连接起来。这种类型的设计模式属于结构型模式，它结合多个标准来获得单一标准。

#### 示例

我们将创建一个 *Person* 对象、*Criteria* 接口和实现了该接口的实体类，来过滤 *Person* 对象的列表。*CriteriaPatternDemo*，我们的演示类使用 *Criteria* 对象，基于各种标准和它们的结合来过滤 *Person* 对象的列表。

![5.jpg](https://i.loli.net/2019/06/05/5cf7baf81870060267.jpg)

1. 创建一个类，在该类上应用标准。

   ```
   //Person.java
   public class Person {
      
      private String name;
      private String gender;
      private String maritalStatus;
    
      public Person(String name,String gender,String maritalStatus){
         this.name = name;
         this.gender = gender;
         this.maritalStatus = maritalStatus;    
      }
    
      public String getName() {
         return name;
      }
      public String getGender() {
         return gender;
      }
      public String getMaritalStatus() {
         return maritalStatus;
      }  
   }
   
   ```

2. 为标准（Criteria）创建一个接口。

   ```
   //Criteria.java
   import java.util.List;
    
   public interface Criteria {
      public List<Person> meetCriteria(List<Person> persons);
   }
   
   ```

3. 创建实现了 *Criteria* 接口的实体类。

   ```java
   //CriteriaMale.java
   
   import java.util.ArrayList;
   import java.util.List;
    
   public class CriteriaMale implements Criteria {
    
      @Override
      public List<Person> meetCriteria(List<Person> persons) {
         List<Person> malePersons = new ArrayList<Person>(); 
         for (Person person : persons) {
            if(person.getGender().equalsIgnoreCase("MALE")){
               malePersons.add(person);
            }
         }
         return malePersons;
      }
   }
   
   //CriteriaFemale.java
   import java.util.ArrayList;
   import java.util.List;
    
   public class CriteriaFemale implements Criteria {
    
      @Override
      public List<Person> meetCriteria(List<Person> persons) {
         List<Person> femalePersons = new ArrayList<Person>(); 
         for (Person person : persons) {
            if(person.getGender().equalsIgnoreCase("FEMALE")){
               femalePersons.add(person);
            }
         }
         return femalePersons;
      }
   }
   
   //CriteriaSingle.java
   import java.util.ArrayList;
   import java.util.List;
    
   public class CriteriaSingle implements Criteria {
    
      @Override
      public List<Person> meetCriteria(List<Person> persons) {
         List<Person> singlePersons = new ArrayList<Person>(); 
         for (Person person : persons) {
            if(person.getMaritalStatus().equalsIgnoreCase("SINGLE")){
               singlePersons.add(person);
            }
         }
         return singlePersons;
      }
   }
   
   //AndCriteria.java
   import java.util.List;
    
   public class AndCriteria implements Criteria {
    
      private Criteria criteria;
      private Criteria otherCriteria;
    
      public AndCriteria(Criteria criteria, Criteria otherCriteria) {
         this.criteria = criteria;
         this.otherCriteria = otherCriteria; 
      }
    
      @Override
      public List<Person> meetCriteria(List<Person> persons) {
         List<Person> firstCriteriaPersons = criteria.meetCriteria(persons);     
         return otherCriteria.meetCriteria(firstCriteriaPersons);
      }
   }
   
   //OrCriteria.java
   import java.util.List;
    
   public class OrCriteria implements Criteria {
    
      private Criteria criteria;
      private Criteria otherCriteria;
    
      public OrCriteria(Criteria criteria, Criteria otherCriteria) {
         this.criteria = criteria;
         this.otherCriteria = otherCriteria; 
      }
    
      @Override
      public List<Person> meetCriteria(List<Person> persons) {
         List<Person> firstCriteriaItems = criteria.meetCriteria(persons);
         List<Person> otherCriteriaItems = otherCriteria.meetCriteria(persons);
    
         for (Person person : otherCriteriaItems) {
            if(!firstCriteriaItems.contains(person)){
              firstCriteriaItems.add(person);
            }
         }  
         return firstCriteriaItems;
      }
   }
   
   ```

4. 使用不同的标准（Criteria）和它们的结合来过滤 *Person* 对象的列表。

   ```
   //CriteriaPatternDemo.java
   
   import java.util.ArrayList; 
   import java.util.List;
    
   public class CriteriaPatternDemo {
      public static void main(String[] args) {
         List<Person> persons = new ArrayList<Person>();
    
         persons.add(new Person("Robert","Male", "Single"));
         persons.add(new Person("John","Male", "Married"));
         persons.add(new Person("Laura","Female", "Married"));
         persons.add(new Person("Diana","Female", "Single"));
         persons.add(new Person("Mike","Male", "Single"));
         persons.add(new Person("Bobby","Male", "Single"));
    
         Criteria male = new CriteriaMale();
         Criteria female = new CriteriaFemale();
         Criteria single = new CriteriaSingle();
         Criteria singleMale = new AndCriteria(single, male);
         Criteria singleOrFemale = new OrCriteria(single, female);
    
         System.out.println("Males: ");
         printPersons(male.meetCriteria(persons));
    
         System.out.println("\nFemales: ");
         printPersons(female.meetCriteria(persons));
    
         System.out.println("\nSingle Males: ");
         printPersons(singleMale.meetCriteria(persons));
    
         System.out.println("\nSingle Or Females: ");
         printPersons(singleOrFemale.meetCriteria(persons));
      }
    
      public static void printPersons(List<Person> persons){
         for (Person person : persons) {
            System.out.println("Person : [ Name : " + person.getName() 
               +", Gender : " + person.getGender() 
               +", Marital Status : " + person.getMaritalStatus()
               +" ]");
         }
      }      
   }
   
   ```

5. 运行结果

   ```
   Males: 
   Person : [ Name : Robert, Gender : Male, Marital Status : Single ]
   Person : [ Name : John, Gender : Male, Marital Status : Married ]
   Person : [ Name : Mike, Gender : Male, Marital Status : Single ]
   Person : [ Name : Bobby, Gender : Male, Marital Status : Single ]
   
   Females: 
   Person : [ Name : Laura, Gender : Female, Marital Status : Married ]
   Person : [ Name : Diana, Gender : Female, Marital Status : Single ]
   
   Single Males: 
   Person : [ Name : Robert, Gender : Male, Marital Status : Single ]
   Person : [ Name : Mike, Gender : Male, Marital Status : Single ]
   Person : [ Name : Bobby, Gender : Male, Marital Status : Single ]
   
   Single Or Females: 
   Person : [ Name : Robert, Gender : Male, Marital Status : Single ]
   Person : [ Name : Diana, Gender : Female, Marital Status : Single ]
   Person : [ Name : Mike, Gender : Male, Marital Status : Single ]
   Person : [ Name : Bobby, Gender : Male, Marital Status : Single ]
   Person : [ Name : Laura, Gender : Female, Marital Status : Married ]
   ```

   