---
title: 设计模式--行为型（上）
date: 2019-05-24 14:18:59
tags:
 - Java
 - 设计模式
categories:
 - Java
 - 设计模式
---

### 策略模式

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。

<!--more-->

**意图：**定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换，**让算法独立于使用它的客户而变化**。 

**主要解决：**在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护，一般用来替换if-else，个人感觉是面向过程与面向对象思想的 过渡

**何时使用：**一个系统有许多许多类，而区分它们的只是他们直接的行为。

**如何解决：**将这些算法封装成一个一个的类，任意地替换。

**关键代码：**实现同一个接口。

**应用实例：** 

1. 诸葛亮的锦囊妙计，每一个锦囊就是一个策略。
2. 旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。
3. JAVA AWT 中的 LayoutManager。

**优点：** 

1. 算法可以自由切换。 
2. 避免使用多重条件判断。 
3. 扩展性良好。 

**缺点：** 

1. 策略类会增多。 
2. 所有策略类都需要对外暴露。 

**使用场景：** 

1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。
2. 一个系统需要动态地在几种算法中选择一种。
3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。
4. 不希望客户端知道复杂的、与算法相关的数据结构，在具体策略类中封装算法 
   与相关的数据结构，可以提高算法的保密性与安全性。 

**注意事项：**如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

**三个角色**

- **Context**: **上下文环境类**，持有抽象策略角色的引用
- **Strategy**: **抽象策略类**，定义一系列抽象的算法策略
- **ConcreteStrategy**: **具体策略类**，实现具体的算法策略

UML：

![5.png](https://i.loli.net/2019/06/05/5cf75c658840b40677.png)

**与状态模式的比较**

状态模式的类图和策略模式类似，并且都是能够动态改变对象的行为。但是状态模式是通过状态转移来改变 Context 所组合的 State 对象，而策略模式是通过 Context 本身的决策来改变组合的 Strategy 对象。所谓的状态转移，是指 Context 在运行过程中由于一些条件发生改变而使得 State 对象发生改变，注意必须要是在运行过程中。

状态模式主要是用来解决状态转移的问题，当状态发生转移了，那么 Context 对象就会改变它的行为；而策略模式主要是用来封装一组可以互相替代的算法族，并且可以根据需要动态地去替换 Context 使用的算法。



#### 示例

策略模式实现简易计算器

其实就是把if-else涉及到的算法，策略行为抽取出来，统一的接口，然后各自实现，比如这里我们把抽取计算的接口，然后继承分别实现加减乘除：

```

public interface Compute {
    String compute(int first, int second);
}
public class Add implements Compute{
    @Override public String compute(int first, int second) {
        return "输出结果：" + first + " + " + second + " = " + (first + second);
    }
}

public class Div implements Compute{
    @Override public String compute(int first, int second) {
        return "输出结果：" + first + " / " + second + " = " + (first / second);
    }
}

public class Mul implements Compute{
    @Override public String compute(int first, int second) {
        return "输出结果：" + first + " * " + second + " = " + (first * second);
    }
}

public class Sub implements Compute{
    @Override public String compute(int first, int second) {
        return "输出结果：" + first + " - " + second + " = " + (first - second);
    }
}
```

编写**上下文对象**，**负责与具体的策略类交互**

```
public class Context {
    private Compute compute;

    public Context() { compute = new Add(); }

    public void setCompute(Compute compute) { this.compute = compute; }

    public void calc(int first, int second) { System.out.println(compute.compute(first, second)); }

}
```

**客户端调用**：

```

public class Client {
    public static void main(String[] args) {
        Context context = new Context();

        context.setCompute(new Add());
        context.calc(1,2);

        context.setCompute(new Sub());
        context.calc(3,4);

        context.setCompute(new Mul());
        context.calc(5,6);

        context.setCompute(new Div());
        context.calc(7,8);
    }
}
```

### 观察者模式

当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知它的依赖对象。观察者模式属于行为型模式。

**意图：**定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

**主要解决：**一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

**何时使用：**一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。

**如何解决：**使用面向对象技术，可以将这种依赖关系弱化。

**关键代码：**在抽象类里有一个 ArrayList 存放观察者们。

**应用实例：** 

1. 拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价。
2. 西游记里面悟空请求菩萨降服红孩儿，菩萨洒了一地水招来一个老乌龟，这个乌龟就是观察者，他观察菩萨洒水这个动作。 

**优点：** 

1. 观察者和被观察者是抽象耦合的。 
2. 建立一套触发机制。 

**缺点：** 

1. 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
2. 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。 
3. 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。 

**使用场景：** 

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
- 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
- 一个对象必须通知其他对象，而并不知道这些对象是谁。
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。

**注意事项：** 

1. JAVA 中已经有了对观察者模式的支持类。
2. 避免循环引用。
3. 如果顺序执行，某一观察者错误会导致系统卡壳，一般采用异步方式。 

首先是使用场景，举个最简单的例子，你放学到家，很饿，这个时候你妈在厨房做饭，因为你和妈妈达成了写完作业才可以玩农药的约定，所以你要先写作业，但是你又很饿，你想当饭做好了第一时间能吃到，你可以：

- 每隔一段时间跑去厨房找你妈，问她饭做好没（轮询）
- 跟你妈说一声，”老妈，饭做好了马上叫我”（订阅或注册），你妈答应了你的请求，等她做完饭后立马通知你，这样你就不可以一心一意的做作业了，等你妈通知你，然后放下手上的作业，去吃饭。

上面的场景，使用观察者模式，比起反复的检索，显得更加优雅。

这个模式，有两个主要角色：被观察者 与 观察者，这里的老妈就属于被观察者，而你属于观察者，可能你还有个妹妹和弟弟，一起等饭吃，即：一个被观察者可以对应多个观察者，你妈做好饭的时候，会把你们都通知一遍。

主要角色就这两个，当然，按照设计模式的套路，肯定是会抽象的，所以有了另外的两个抽象被观察者，抽象观察者，四个角色责任如下：

**四个角色**

- Subject：抽象被观察者，把所有观察者对象的引用保存到集合中，然后
  提供添加，移除，和通知观察者对象更新的方法。
- ConcreteSubject：被观察者，集合存放观察者，重写增删和通知观察者
  的方法，当发生变化时通知观察者更新。
- Observer：抽象观察者，定义一个更新接口，给被观察者更新的时候调
- ConcreteObserver：具体观察者，继承抽象观察者，实现具体的更新方法

![6.png](https://i.loli.net/2019/06/05/5cf75f06c0a2c89200.png)

#### 示例

有一个微信公众号服务，不定时发布一些消息，关注公众号就可以收到推送消息，取消关注就收不到推送消息。

1. 定义一个抽象被观察者接口

   ```
   /***
    * 抽象被观察者接口
    * 声明了添加、删除、通知观察者方法
    *
    */
   public interface Observerable {
       
       public void registerObserver(Observer o);
       public void removeObserver(Observer o);
       public void notifyObserver();
       
   }
   ```

2. 定义一个抽象观察者接口

   ```
   /***
    * 抽象观察者
    * 定义了一个update()方法，当被观察者调用notifyObservers()方法时，观察者的update()方法会被回调。
    *
    */
   public interface Observer {
       public void update(String message);
   }
   ```

3. 定义被观察者，实现了Observerable接口，对Observerable接口的三个方法进行了具体实现，同时有一个List集合，用以保存注册的观察者，等需要通知观察者时，遍历该集合即可。

   ```
   import java.util.ArrayList;
   import java.util.List;
   
   /**
    * 被观察者，也就是微信公众号服务
    * 实现了Observerable接口，对Observerable接口的三个方法进行了具体实现
    *
    */
   public class WechatServer implements Observerable {
       
       //注意到这个List集合的泛型参数为Observer接口，设计原则：面向接口编程而不是面向实现编程
       private List<Observer> list;
       private String message;
       
       public WechatServer() {
           list = new ArrayList<Observer>();
       }
       
       @Override
       public void registerObserver(Observer o) {
           
           list.add(o);
       }
       
       @Override
       public void removeObserver(Observer o) {
           if(!list.isEmpty())
               list.remove(o);
       }
   
       //遍历
       @Override
       public void notifyObserver() {
           for(int i = 0; i < list.size(); i++) {
               Observer oserver = list.get(i);
               oserver.update(message);
           }
       }
       
       public void setInfomation(String s) {
           this.message = s;
           System.out.println("微信服务更新消息： " + s);
           //消息更新，通知所有观察者
           notifyObserver();
       }
   
   }
   ```

4. 定义具体观察者，微信公众号的具体观察者为用户User

   ```
   /**
    * 观察者
    * 实现了update方法
    *
    */
   public class User implements Observer {
   
       private String name;
       private String message;
       
       public User(String name) {
           this.name = name;
       }
       
       @Override
       public void update(String message) {
           this.message = message;
           read();
       }
       
       public void read() {
           System.out.println(name + " 收到推送消息： " + message);
       }
       
   }
   ```

5. 编写一个测试类

   首先注册了三个用户，ZhangSan、LiSi、WangWu。公众号发布了一条消息"PHP是世界上最好用的语言！"，三个用户都收到了消息。

   用户ZhangSan看到消息后颇为震惊，果断取消订阅，这时公众号又推送了一条消息，此时用户ZhangSan已经收不到消息，其他用户还是正常能收到推送消息。

   ```
   public class Test {
       
       public static void main(String[] args) {
           WechatServer server = new WechatServer();
           
           Observer userZhang = new User("ZhangSan");
           Observer userLi = new User("LiSi");
           Observer userWang = new User("WangWu");
           
           server.registerObserver(userZhang);
           server.registerObserver(userLi);
           server.registerObserver(userWang);
           server.setInfomation("PHP是世界上最好用的语言！");
           
           System.out.println("----------------------------------------------");
           server.removeObserver(userZhang);
           server.setInfomation("JAVA是世界上最好用的语言！");
           
       }
   }
   ```

**java内置的观察者模式**

在java.util包中包含有基本的Observer接口和Observable抽象类.功能上和Subject接口和Observer接口类似.不过在使用上,就方便多了,因为许多功能比如说注册,删除,通知观察者的那些功能已经内置好了.

使用javaAPI的观察者模式需要明白这么几件事情:

1. 如何使对象变为观察者?

   实现观察者接口(java.util.Observer),然后调用Observable对象的addObserver()方法.不想再当观察者时,调用deleteObserver()就可以了.

2. 被观察者(主题)如何发出通知?

   第一步:先调用setChanged()方法,标识状态已经改变的事实.

   第二步:调用notifyObservers()方法或者notifyObservers(Object arg),这就牵扯到推(push)和拉(pull)的方式传送数据.如果想用push的方式"推"数据给观察者,可以把数据当做数据对象传送给notifyObservers(Object arg)方法,其中的arg可以为任意对象,意思是你可以将任意对象传送给每一个观察者.如果调用不带参数的notifyObserver()方法,则意味着你要使用pull的方式去主题对象中"拉"来所需要的数据.

3. 观察者如何接收通知?

   观察者只需要实现一个update(Observable o,Object arg)方法,第一个参数o,是指定通知是由哪个主题下达的,第二个参数arg就是上面notifyObserver(Object arg)里传入的数据,如果不传该值,arg为null.

这里以师生关系为例,老师和学生是一对多的关系,老师给学生布置作业,这个动作作为主题事件,每当老师布置一道题时,就要自动通知到所有的学生把该题记下来,然后再布置下一道题...

![9.png](https://i.loli.net/2019/06/05/5cf762222786c58875.png)

被观察者TeacherSubject:

```
import java.util.Observable;

public class JTeacher extends Observable {
        //布置作业的状态信息字符串
    private String info;
    public void setHomework(String info) {

        this.info=info;
        System.out.println("布置的作业是"+info);

        setChanged();
        notifyObservers();
    }
    public String getInfo() {
        return info;
    }
}
```

观察者StudentObserver:

```
import java.util.Observable;
import java.util.Observer;

public class JStudent implements Observer{

    private Observable ob;
    private String name;

    public JStudent(String name,Observable ob) {
        this.ob = ob;
        this.name=name;
        ob.addObserver(this);
    }

    @Override
    public void update(Observable o, Object arg) {
        JTeacher t=(JTeacher)o;
        System.out.println(name+"得到作业信息:"+t.getInfo());
        
    }

}
```

测试类TestObserver:

```
public class Test02 {
    public static void main(String[] args) {

        JTeacher teacher=new JTeacher();
        JStudent zhangSan=new JStudent("张三", teacher);
        JStudent LiSi=new JStudent("李四", teacher);
        JStudent WangWu=new JStudent("王五", teacher);

        teacher.setHomework("第二页第六题");
        teacher.setHomework("第三页第七题");
        teacher.setHomework("第五页第八题");
    }
}
```

运行结果：

```
布置的作业是第二页第六题
王五得到作业信息:第二页第六题
李四得到作业信息:第二页第六题
张三得到作业信息:第二页第六题
布置的作业是第三页第七题
王五得到作业信息:第三页第七题
李四得到作业信息:第三页第七题
张三得到作业信息:第三页第七题
布置的作业是第五页第八题
王五得到作业信息:第五页第八题
李四得到作业信息:第五页第八题
张三得到作业信息:第五页第八题
```

**Java 9起淘汰了Observable及相关接口。**

- 如果用Observable写的对象事件订阅机制，推荐用java.beans相关的类来代替。
- 如果用Observable写的进程间通信机制，文档推荐用java.util.concurrent并行库相关的类来代替。
- 如果用Observable写的反应式流水线机制，文档推荐用并行库的Flow及Future等类来代替。
- 如果仅仅是为了实现观察者设计模式，可以写自定义的类...或直接剪贴源码 

#### 观察者模式与发布/订阅模式区别

**观察者模式**

比较概念的解释是，目标和观察者是基类，目标提供维护观察者的一系列方法，观察者提供更新接口。具体观察者和具体目标继承各自的基类，然后具体观察者把自己注册到具体目标里，在具体目标发生变化时候，调度观察者的更新方法。

比如有个“天气中心”的具体目标A，专门监听天气变化，而有个显示天气的界面的观察者B，B就把自己注册到A里，当A触发天气变化，就调度B的更新方法，并带上自己的上下文。

![7.png](https://i.loli.net/2019/06/05/5cf760308ee6c45287.png)

**发布/订阅模式**

比较概念的解释是，订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心（顺带上下文），由调度中心统一调度订阅者注册到调度中心的处理代码。

比如有个界面是实时显示天气，它就订阅天气事件（注册到调度中心，包括处理程序），当天气变化时（定时获取数据），就作为发布者发布天气信息到调度中心，调度中心就调度订阅者的天气处理程序。

![8.png](https://i.loli.net/2019/06/05/5cf760308caaa37728.png)

1. 从两张图片可以看到，最大的区别是调度的地方。

   虽然两种模式都存在订阅者和发布者（具体观察者可认为是订阅者、具体目标可认为是发布者），但是观察者模式是由具体目标调度的，而发布/订阅模式是统一由调度中心调的，所以观察者模式的订阅者与发布者之间是存在依赖的，而发布/订阅模式则不会。

2. 两种模式都可以用于松散耦合，改进代码管理和潜在的复用。

#### PUSH和PULL区别

在观察者模式中，又分为推模型和拉模型两种方式。

- 推模型:主题对象向观察者推送主题的详细信息，不管观察者是否需要，推送的信息通常是主题对象的全部或部分数据。

- 拉模型:主题对象在通知观察者的时候，只传递少量信息。如果观察者需要更具体的信息，由观察者主动到主题对象中获取，相当于是观察者从主题对象中拉数据。一般这种模型的实现中，会把主题对象自身通过update()方法传递给观察者，这样在观察者需要获取数据的时候，就可以通过这个引用来获取了。

推送(Push)技术是根据用户需要，有目的、按时将用户感兴趣的信息主动发送到用户的计算机中。Push技术的主要优点是对用户要求低，  普遍适用于广大公众，不要求有专门的技术；二是及时性好，信源及时地向用户“推送”不断更新的动态信息。但是，在随后实际应用中，因为存在以下几方面不  足，Push技术并没有取得预期的成功：

- 不能确保发送成功。由于Push技术采用广播方式，当网络信息中心发送信息时，只有接收器打开并正好切换到同一频道上，传输才能发生作用，用户才能获取信息。这对于那些要确保能收到信息的应用领域是不太适合的。
- 没有信息状态跟踪。Push技术采用的是“开环控制”模式，一个信息发布以后的状态，如用户是否接收，或客户端收到后是否按信息的提示执行了任务等，这些“反馈信息”发布者无从得知。
- 针对性差。推送的信息内容缺乏针对性，不能满足用户的个性化需求。有价值的重要信息，通常都是要针对一些特定的群组来发送的，即只送给相关的人士。Push技术不能满足上述需求。
- 信源任务重。信源系统要主动地、快速地、不断地将大量信息推送给用户。 

拉取(Pull)技术指用户有目的地在网络上主动查询信息，用户从浏览器给Web发出请求，由Web获取所需信息。面对拥有海量信息的  Internet环境，搜索引擎是有效的网络信息“拉取”(查询)的检索工具。Pull技术的主要优点是针对性强，能满足用户的个性化需求；信息传输量  小，网络上所传输的只是用户的请求和服务器针对该请求所作的响应；信源任务轻，信息系统只是被动接受查询，提供用户所需的部分信息。其主要缺点是及时性  差，由于用户只会基于自己的知识水平(或专业水平)提出请求，当信源中信息更新变化时，用户难以及时拉取新的动态信息，虽然可以通过定时查询来解决这个问  题，但是会浪费大量的网络资源和人力，而且，仍不能保证最好的实时性。对用户要求高，要求用户对信源系统有相应的专业知识，掌握相关的检索技术。

|            | push模型                                                     | pull模型                                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 描述       | 服务端主动发送数据给客户端                                   | 客户端主动从服务端拉取数据，通常客户端会定时拉取             |
| 实时性     | 较好，收到数据后可立即发送给客户端                           | 一般，取决于pull的间隔时间                                   |
| 服务端状态 | 需要保存push状态，哪些客户端已经发送成功，哪些发送失败       | 服务端无状态                                                 |
| 客户端状态 | 无需额外保存状态                                             | 需保存当前拉取的信息的状态，以便在故障或者重启的时候恢复     |
| 状态保存   | 集中式，集中在服务端                                         | 分布式，分散在各个客户端                                     |
| 负载均衡   | 服务端统一处理和控制                                         | 客户端之间做分配，需要协调机制，如使用zookeeper              |
| 其他       | 服务端需要做流量控制，无法最大化客户端的处理能力。     其次，在客户端故障情况下，无效的push对服务端有一定负载。 | 客户端的请求可能很多无效或者没有数据可供传输，浪费带宽和服务器处理能力 |
| 缺点方案   | 服务器端的状态存储是个难点，可以将这些状态转移到DB或者key-value存储，来减轻server压力。 | 针对实时性的问题，可以将push加入进来，push小数据的通知信息，让客户端再来主动pull。     针对无效请求的问题，可以设置逐渐延长间隔时间的策略，以及合理设计协议尽量缩小请求数据包来节省带宽。 |

**PUSH和PULL两种模式结合**

将信息推送与拉取两种模式结合能做到取长补短，使二者优势互补。根据推、拉结合顺序及结合方式的差异，又分以下四种不同推拉模式：

- 先推后拉——先由信源及时推送公共信息，再由用户有针对性地拉取个性化信息；

- 先拉后推——根据用户拉取的信息，信源进一步主动提供(推送)与之相关的信息；

- 推中有拉——在信息推送过程中，允许用户随时中断并定格在感兴趣的网页上，以拉取更有针对性的信息；

- 拉中有推——根据用户搜索(即拉取)过程中所用的关键字，信源主动推送相关的最新信息。 

在面对大量甚至海量客户端的时候，使用push模型，保存大量的状态信息是个沉重的负担，加上复制N份数据分发的压力，也会使得实时性这唯 一的优点也被放小。使用pull模型，通过将客户端状态保存在客户端，大大减轻了服务器端压力，通过客户端自身做流量控制也更容易，更能发挥客户端的处理 能力，但是需要面对如何在这些客户端之间做协调的难题。

客户端和服务端的交互有推和拉两种方式：如果是客户端拉的话，通常就是Polling；如果是服务端[推](http://en.wikipedia.org/wiki/Push_technology)的话，一般就是[Comet](http://en.wikipedia.org/wiki/Comet_(programming))，目前比较流行的Comet实现方式是Long Polling。

注：如果不清楚相关名词含义，可以参考：[Browser 與 Server 持續同步的作法介紹](http://josephj.com/entry.php?id=358)。

先来看看Polling，它其实就是我们平常所说的轮询，大致如下所示：

![2.png](https://i.loli.net/2019/06/07/5cfa183e4c68910824.png)

因为服务端不会主动告诉客户端它是否有新数据，所以Polling的实时性较差。虽然可以通过加快轮询频率的方式来缓解这个问题，但相应付出的代价也不小：一来会使负载居高不下，二来也会让带宽捉襟见肘。

再来说说Long Polling，如果使用传统的LAMP技术去实现的话，大致如下所示：

![3.png](https://i.loli.net/2019/06/07/5cfa1906258a198078.png)

客户端不会频繁的轮询服务端，而是对服务端发起一个长连接，服务端通过轮询数据库来确定是否有新数据，一旦发现新数据便给客户端发出响应，这次交互便结束了。客户端处理好新数据后再重新发起一个长连接，如此周而复始。

在上面这个Long Polling方案里，我们解决了Polling中客户端轮询造成的负载和带宽的问题，但是依然存在服务端轮询，数据库的压力可想而知，此时我们虽然可以通过针对数据库使用主从复制，分片等技术来缓解问题，但那毕竟只是治标不治本。

我们的目标是实现一个简单的服务端推方案，但简单绝对不意味着简陋，轮询数据库是不可以接受的，下面我们来看看如何解决这个问题。在这里我们放弃了传统的LAMP技术，转而使用[Nginx与Lua](http://huoding.com/2012/08/31/156)来实现。

![4.png](https://i.loli.net/2019/06/07/5cfa19272e30923981.png)

此方案的主要思路是这样的：使用Nginx作为服务端，通过Lua协程来创建长连接，一旦数据库里有新数据，它便主动通知Nginx，并把  相应的标识(比如一个自增的整数ID)保存在Nginx共享内存中，接下来，Nginx不会再去轮询数据库，而是改为轮询本地的共享内存，通过比对标识来  判断是否有新消息，如果有便给客户端发出响应。

​    注：服务端维持大量长连接时内核参数的调整请参考：[http长连接200万尝试及调优](http://blog.lifeibo.com/?p=269)。

### 迭代器模式

迭代器模式（Iterator Pattern）是 Java 和 .Net 编程环境中非常常用的设计模式。这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。

迭代器模式属于行为型模式。

**意图：**提供一种方法顺序访问一个聚合对象中各个元素, 而又无须暴露该对象的内部表示。

**主要解决：**不同的方式来遍历整个整合对象。

**何时使用：**遍历一个聚合对象。

**如何解决：**把在元素之间游走的责任交给迭代器，而不是聚合对象。

**关键代码：**定义接口：hasNext, next。

**应用实例：**JAVA 中的 iterator。

**优点：** 

1. 它支持以不同的方式遍历一个聚合对象。
2. 迭代器简化了聚合类。
3. 在同一个聚合上可以有多个遍历。 
4. 在迭代器模式中，增加新的聚合类和迭代器类都很方便，无须修改原有代码。 

**缺点：**由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性。

**使用场景：** 

1. 访问一个聚合对象的内容而无须暴露它的内部表示。 
2. 需要为聚合对象提供多种遍历方式。 
3. 为遍历不同的聚合结构提供一个统一的接口。 

**注意事项：**迭代器模式就是分离了集合对象的遍历行为，抽象出一个迭代器类来负责，这样既可以做到不暴露集合的内部结构，又可让外部代码透明地访问集合内部的数据。

**四个角色**：

- Iterator：迭代器角色，定义访问和遍历元素的接口；
- ConcreteIterator：具体迭代器角色，实现接口中的方法，并且记录遍历的当前位置；
- Container：容器角色，提供创建具体迭代器角色的接口；
- ConcreteContainer：具体容器角色，具体迭代器角色与容器相关联

#### 示例

这里我们举个遍历歌单的例子，首先是容器中的元素，歌曲

```
public class Song {

    public Song(String name, String singer) {
        this.name = name;
        this.singer = singer;
    }

    private String name;

    private String singer;

    public String getName() { return name; }

    public void setName(String name) { this.name = name;}

    public String getSinger() { return singer; }

    public void setSinger(String singer) { this.singer = singer; }

    @Override public String toString() {
        return "【歌名】" + name + " - " + singer;
    }

}
```

接着写个**抽象迭代器**，第一项，下一个，判断是否能下一个，获取当前项。

```

public interface Iterator {

    Song first();

    Song next();

    boolean hashNext();

    Song currentItem();
}
```

再接着是**抽象容器**，定义一个生成迭代器的方法

```
interface SongList {
    Iterator getIterator();
}
```

然后定义**具体容器**，集成抽象容器，并定义一个**具体迭代器内部类**

```
import sun.rmi.runtime.Log;

import java.util.ArrayList;
import java.util.List;


public class MyStoryList implements SongList{
    private List<Song> list = new ArrayList<>();

    public MyStoryList(List<Song> list) {
        this.list = list;
    }

    @Override public Iterator getIterator() {
        return new SongListIterator();
    }

    private class SongListIterator implements Iterator {

        private int cursor;

        @Override public Song first() {
            cursor = 0;
            return list.get(cursor);
        }

        @Override public Song next() {
            Song song = null;
            cursor++;
            if(hashNext()) {
                song = list.get(cursor);
            }
            return song;
        }

        @Override public boolean hashNext() {
            return !(cursor == list.size());
        }

        @Override public Song currentItem() {
            return list.get(cursor);
        }
    }


}
```

最后客户端调用

```
import java.util.*;
public class Client {
    public static void main(String[] args) {
        List<Song> list = new ArrayList<>();
        list.add(new Song("空白格","杨宗纬"));
        list.add(new Song("那时候的我","刘惜君"));
        list.add(new Song("黑泽明","陈奕迅"));
        list.add(new Song("今天只做一件事","陈奕迅"));
        list.add(new Song("童话镇","陈一发儿"));

        MyStoryList songList = new MyStoryList(list);

        Iterator iterator = songList.getIterator();

        while (iterator.hashNext()) {
            System.out.println(iterator.currentItem().toString());
            iterator.next();
        }
    }
}
```

好的，迭代器模式的例子就那么简单，其实Java中的容器类已经为我们提供了 相应的迭代器，而不需要我们另外去实现了。比如Util包中的Iterator接口。

### 命令模式

命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。 

**意图：**将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。

**主要解决：**在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。

**何时使用：**在某些场合，比如要对行为进行"记录、撤销/重做、事务"等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将"行为请求者"与"行为实现者"解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

**如何解决：**通过调用者调用接受者执行命令，顺序：调用者→接受者→命令。

**关键代码：**定义三个角色：

1. received 真正的命令执行对象 
2. Command 
3. invoker 使用命令对象的入口

**应用实例：**struts 1 中的 action 核心控制器 ActionServlet 只有一个，相当于 Invoker，而模型层的类会随着不同的应用有不同的模型类，相当于具体的 Command。

**优点：** 

1. 更松散的耦合，请求者无需知道执行者是谁，如何执行指令。
2. 更动态的控制，将请求封装，可以动态进行参数化，队列化，日志化等操作。
3. 命令可以复合，即一个命令可以由多个命令组合而成，又叫宏命令。
4. 更好的扩展性，因为命令发起者与执行者解耦，扩展新命令，只需实现新的命令对象，在需要的时候把具体的实现对象传入到命令对象中，就可以使用这个命令对象了，已有的实现完全不用变化。

5. 为请求的撤销(Undo)和恢复(Redo)操作提供了一种设计和实现方案。

**缺点：**使用命令模式可能会导致某些系统有过多的具体命令类。

**使用场景：**认为是命令的地方都可以使用命令模式，比如： 

1. 系统需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互。请求
   调用者无须知道接收者的存在，也无须知道接收者是谁，接收者也无须关心何时被调用。
2. 系统需要在不同的时间指定请求和执行请求。一个命令对象和请求的初始调用者可
   以有不同的生命期，换言之，最初的请求发出者可能已经不在了，而命令对象本身仍然
   是活动的，可以通过该命令对象去调用请求接收者，而无须关心请求调用者的存在性，
   可以通过请求日志文件等机制来具体实现。
3. 系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作。

4. 系统需要将一组操作组合在一起形成宏命令。

**注意事项：**系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作，也可以考虑使用命令模式，见命令模式的扩展。

**四个角色**：

- Command：命令，声明具体命令的抽象接口。
- ConcreteCommand：具体命令，接收者对象绑定与一个动作。
- Receiver：接收者，执行与请求相关的操作，具体实现对请求的业务处理。
- Invoker：调用者，负责调用命令对象执行请求，相关的方法叫做行动方法。

![10.png](https://i.loli.net/2019/06/05/5cf76904e597d94017.png)



#### 示例

摆地摊：顾客 –点餐-> 老板 –收到点餐指令-> 制作菜肴

写成代码的话：就是顾客中持有对老板的引用，然后执行指令的时候调用老板中制作菜肴的方法。这里的请求者直接与实现者进行交互，耦合度比较高！

开店：

增加了一个服务员的角色，接收用户指令，处理指令，然后调用老板进行菜肴制作，中途还可以记录命令请求，或者做撤销命令的操作。

这里用命令模式写个简单的播放控制例子吧。

首先创建播放对象，就一个故事名和播放URL的属性

```
public class Story {
    private String sName;
    private String sUrl;

    public Story(String sName, String sUrl) {
        this.sName = sName;
        this.sUrl = sUrl;
    }

    public String getsUrl() {
        return sUrl;
    }

    public void setsUrl(String sUrl) {
        this.sUrl = sUrl;
    }

    public String getsName() {

        return sName;
    }

    public void setsName(String sName) {
        this.sName = sName;
    }
}
```

然后创建**命令执行者**，一个音乐播放器，提供设置播放列表，播放，暂停等方法

```
public class StoryPlayer {

    private int cursor = 0; //当前播放项

    private int pauseCursor = -1;   //暂停播放项

    private List<Story> playList = new ArrayList<>();   //播放列表

    public void setPlayList(List<Story> list) {
        this.playList = list;
        cursor = 0;
        System.out.println("更新播放列表...");
    }

    public void play() {
        cursor = 0;
        play(cursor);
    }

    public void play(int cursor) {
        if(playList.size() == 0) {
            System.out.println("当前播放列表为空，请先设置播放列表！");
        } else {
            if(pauseCursor == cursor) {
                System.out.println("继续播放第" + pauseCursor + "个故事: 《" + playList.get(pauseCursor).getsName() + "》");
            } else  {
                this.cursor = cursor;
                System.out.println("开始播放第" + cursor + "个故事: 《" + playList.get(cursor).getsName() + "》");
            }
        }
    }

    public void next() {
        cursor++;
        if(cursor == playList.size()) cursor = 0;
        play(cursor);
    }

    public void pre() {
        cursor--;
        if(cursor < 0) cursor = playList.size() - 1;
        play(cursor);
    }

    public void pause() {
        pauseCursor = cursor;
        System.out.println("暂停播放！");
    }
}
```

接着创建**抽象命令接口**，就一个**execute()**的方法

```
public interface Command {
   void execute();
}
```

接着继承这个接口，写各种**具体命令类**，设置列表，播放，暂停，下一首，上一首 都是照葫芦画瓢，实现execute()方法，调用执行者里面的对应方法而已。

```
public class SetListCommand implements Command {

    private StoryPlayer mPlayer;

    private List<Story> mList = new ArrayList<>();

    public SetListCommand(StoryPlayer mPlayer) {
        this.mPlayer = mPlayer;
    }

    @Override public void execute() {
        mPlayer.setPlayList(mList);
    }

    public void setPlayList(List<Story> list) {
        this.mList = list;
    }
}


public class PlayCommand implements Command {

    private StoryPlayer mPlayer;

    public PlayCommand(StoryPlayer mPlayer) {
        this.mPlayer = mPlayer;
    }

    @Override public void execute() {
        mPlayer.play();
    }

}

public class PauseCommand implements Command {

    private StoryPlayer mPlayer;

    public PauseCommand(StoryPlayer mPlayer) {
        this.mPlayer = mPlayer;
    }

    @Override public void execute() {
        mPlayer.pause();
    }

}
public class NextCommand implements Command {

    private StoryPlayer mPlayer;

    public NextCommand(StoryPlayer mPlayer) {
        this.mPlayer = mPlayer;
    }

    @Override public void execute() {
        mPlayer.next();
    }

}
public class PreCommand implements Command {

    private StoryPlayer mPlayer;

    public PreCommand(StoryPlayer mPlayer) {
        this.mPlayer = mPlayer;
    }

    @Override public void execute() {
        mPlayer.pre();
    }

}


```

再接着是**请求者类**，低啊用命令对象执行具体的操作

```
public class Invoker {
    private SetListCommand setListCommand;
    private PlayCommand playCommand;
    private PauseCommand pauseCommand;
    private NextCommand nextCommand;
    private PreCommand preCommand;

    public void setSetListCommand(SetListCommand setListCommand) {
        this.setListCommand = setListCommand;
    }

    public void setPlayCommand(PlayCommand playCommand) {
        this.playCommand = playCommand;
    }

    public void setPauseCommand(PauseCommand pauseCommand) {
        this.pauseCommand = pauseCommand;
    }

    public void setNextCommand(NextCommand nextCommand) {
        this.nextCommand = nextCommand;
    }

    public void setPreCommand(PreCommand preCommand) {
        this.preCommand = preCommand;
    }

    /* 设置播放列表 */
    public void setPlayList(List<Story> list) {
        setListCommand.setPlayList(list);
        setListCommand.execute();
    }

    /* 播放 */
    public void play() {
        playCommand.execute();
    }

    /* 暂停 */
    public void pause() {
        pauseCommand.execute();
    }

    /* 下一首 */
    public void next() {
        nextCommand.execute();
    }

    /* 上一首 */
    public void pre() {
        preCommand.execute();
    }
}
```

最后客户端调用

```
public class Client {
    public static void main(String[] args) {
        //实例化播放列表
        List<Story> mList = new ArrayList<>();
        mList.add(new Story("白雪公主",""));
        mList.add(new Story("青蛙的愿望",""));
        mList.add(new Story("驴和妈",""));
        mList.add(new Story("小青蛙的烦恼",""));
        mList.add(new Story("三字经",""));

        //实例化接收者
        StoryPlayer mPlayer = new StoryPlayer();

        //实例化命令对象
        Command setListCommand = new SetListCommand(mPlayer);
        Command playCommand = new PlayCommand(mPlayer);
        Command pauseCommand = new PauseCommand(mPlayer);
        Command nextCommand = new NextCommand(mPlayer);
        Command preCommand = new PreCommand(mPlayer);

        //实例化请求者
        Invoker invoker = new Invoker();
        invoker.setSetListCommand((SetListCommand) setListCommand);
        invoker.setPlayList(mList);
        invoker.setPlayCommand((PlayCommand) playCommand);
        invoker.setPauseCommand((PauseCommand) pauseCommand);
        invoker.setNextCommand((NextCommand) nextCommand);
        invoker.setPreCommand((PreCommand) preCommand);

        //测试调用
        invoker.play();
        invoker.next();
        invoker.next();
        invoker.next();
        invoker.next();
        invoker.next();
        invoker.pause();
        invoker.play();
    }
}
```

运行结果

```
更新播放列表...
开始播放第0个故事: 《白雪公主》
开始播放第1个故事: 《青蛙的愿望》
开始播放第2个故事: 《驴和妈》
开始播放第3个故事: 《小青蛙的烦恼》
开始播放第4个故事: 《三字经》
开始播放第0个故事: 《白雪公主》
暂停播放！
继续播放第0个故事: 《白雪公主》
```

### 备忘录模式

备忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象。备忘录模式属于行为型模式。

**意图：**在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。

**主要解决：**所谓备忘录模式就是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。

**何时使用：**很多时候我们总是需要记录一个对象的内部状态，这样做的目的就是为了允许用户取消不确定或者错误的操作，能够恢复到他原先的状态，使得他有"后悔药"可吃。

**如何解决：**通过一个备忘录类专门存储对象状态。

**关键代码：**客户不与备忘录类耦合，与备忘录管理类耦合。

**应用实例：** 

1. 后悔药。
2. 打游戏时的存档。 
3. Windows 里的 ctri + z。 
4. IE 中的后退。
5. 数据库的事务管理。 

**优点：** 

1. **更好的封装性**，不把发起人对象的内部实现细节暴露给外部；

2. **简化了发起人**，把备忘录对象保存到发起人对象之外，让客户来管理请求的状态；

3. **窄接口和宽接口**，窄接口保证了只有发起者才能访问备忘录对象的状态； 

**缺点：**消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存。

**使用场景：** 

1. 需要保存/恢复数据的相关状态场景。 
2. 提供一个可回滚的操作。 

**注意事项：**  

1. 为了符合迪米特原则，还要增加一个管理备忘录的类。 
2. 为了节约内存，可使用原型模式+备忘录模式。 

**三个角色：**

- Originator：发起人，创建一个备忘录，可以记录，恢复自身的内部状态，还可根据
  需求决定存储那些内部状态。
- Memento：备忘录，存储发起人角色的内部状态，并防止外部对象访问备忘录。
- Caretaker：管理者，存储备忘录，不能对备忘录内容进行访问，只能将其传递
  给其他对象。

**UML类图**

![11.png](https://i.loli.net/2019/06/05/5cf76ba3f32ad70529.png)

#### 示例

这里举个简单的RPG游戏的例子，存档保存当前血量，蓝量，以及有用金币。 
定义**备忘录类**，即存档的内容

```
public class Memento {
    private int hp;
    private int mp;
    private int money;

    public Memento(int hp, int mp, int money) {
        this.hp = hp;
        this.mp = mp;
        this.money = money;
    }

    public int getHp() {
        return hp;
    }

    public int getMp() {
        return mp;
    }

    public int getMoney() {
        return money;
    }
}

```

接着定义一个**角色类**（发起人角色），除了属性定义，还有两件关键的事 要做：**定义保存方法**，保存自身状态；**定义恢复方法**，传入备忘录对象， 自行回复需要回复的项。

```
public class Character {
    private int hp;
    private int mp;
    private int money;

    public Character(int hp, int mp, int money) {
        this.hp = hp;
        this.mp = mp;
        this.money = money;
    }

    public int getHp() {
        return hp;
    }

    public void setHp(int hp) {
        this.hp = hp;
    }

    public int getMp() {
        return mp;
    }

    public void setMp(int mp) {
        this.mp = mp;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public void showMsg() {
        System.out.println("当前状态：| HP：" + hp + " | MP：" + mp + " | 金钱：" + money + "\n");
    }

    //创建一个备忘录，保存当前自身状态
    public Memento save() {
        return new Memento(hp, mp, money);
    }

    //传入一个备忘录对象，恢复内部状态
    public void restore(Memento memento) {
        this.hp = memento.getHp();
        this.mp = memento.getMp();
        this.money = memento.getMoney();
    }
}
```

再接着是备忘录管理者类，**只负责备忘录对象的传递**！ 
PS：如果是多个存档的，**可以用一个集合存备忘录**，然后根据一个索引来获取对应的备忘录！

```
public class Caretaker {
    private Memento memento;

    public Memento getMemento() {
        return memento;
    }

    public void setMemento(Memento memento) {
        this.memento = memento;
    }
}
```

最后，客户端调用

```
public class Client {
    public static void main(String[] args) {
        Caretaker caretaker= new Caretaker();
        Character character = new Character(2000,1000,500);
        //存档
        System.out.println("=== 存档中... ===");
        character.showMsg();
        caretaker.setMemento(character.save());

        System.out.println("=== 单挑Boss，不敌，金钱扣除一半... ===");
        character.setHp(0);
        character.setHp(0);
        character.setHp(250);
        character.showMsg();

        //读档
        System.out.println("=== 读取存档中... ===");
        character.restore(caretaker.getMemento());
        character.showMsg();

    }
}
```

运行结果：

```
=== 存档中... ===
当前状态：| HP：2000 | MP：1000 | 金钱：500

=== 单挑Boss，不敌，金钱扣除一半... ===
当前状态：| HP：250 | MP：1000 | 金钱：500

=== 读取存档中... ===
当前状态：| HP：2000 | MP：1000 | 金钱：500
```

