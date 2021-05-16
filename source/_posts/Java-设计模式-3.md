---
title: 设计模式--结构型（上）
date: 2019-05-23 14:18:59
tags:
 - Java
 - 设计模式
categories:
 - Java
 - 设计模式
---

### 装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

<!--more-->

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

我们通过下面的实例来演示装饰器模式的用法。其中，我们将把一个形状装饰上不同的颜色，同时又不改变形状类

装饰者模式是以对客户端透明的方式扩展对象的功能，是继承关系的一种替代方案！
以下情况可以考虑是想用对象组合（组合与委托）：

- 在不影响其他对象的情况下,以动态、透明的方式给单个对象添加职责；
- 处理那些可以撤消的职责；
- 当不能采用生成子类的方法进行扩充时：一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类



**意图：**动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

**主要解决：**一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

**何时使用：**在不想增加很多子类的情况下扩展类。

**如何解决：**将具体功能职责划分，同时继承装饰者模式。

**关键代码：** 

1. Component 类充当抽象角色，不应该具体实现。
2. 修饰类引用和继承 Component 类，具体扩展类重写父类方法。 

**应用实例：** 

1. 孙悟空有 72 变，当他变成"庙宇"后，他的根本还是一只猴子，但是他又有了庙宇的功能。 
2. 不论一幅画有没有画框都可以挂在墙上，但是通常都是有画框的，并且实际上是画框被挂在墙上。在挂在墙上之前，画可以被蒙上玻璃，装到框子里；这时画、玻璃和画框形成了一个物体。 

**优点：**装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。

**缺点：**多层装饰比较复杂。

**使用场景：**

1. 扩展一个类的功能。 
2. 动态增加功能，动态撤销。 

**注意事项：**可代替继承。

#### 示例

奶茶店列了一张单子，写出所有饮品的价格：

> 奶茶：
>
> - 原味奶茶：5块
> - 珍珠奶茶：7块
> - 椰果奶茶：7块
> - 珍珠椰果奶茶：9块
>
> 柠檬茶：
>
> - 原味柠檬茶：3块
> - 金桔柠檬茶：5块

然后顾客要什么点什么，按着菜单收费就好了，然而用户的 需求都是多变的，他们觉得配料那里可以加点红豆，然后你 的菜单需要新增：

> - 红豆奶茶：7块
> - 红豆珍珠奶茶：9块
> - 红豆椰果奶茶：9块
> - 红豆珍珠椰果奶茶：11块

食客又说，还可以加点其他的配料啊，黑钻，果冻，凉粉，奶盖， 烧仙草等，然后你的菜单就爆炸了，**奶盖果冻黑钻烧仙草珍珠椰果奶茶**

我们必须想一个更优的套路，这个时候可以考虑引入**装饰者模式**， 简单来说就是：**一层套一层**，比如说要椰果珍珠奶茶：

**奶茶**  –> 套一层珍珠 –> **珍珠(奶茶)** –> 套一层椰果 –> **椰果(珍珠(奶茶))**

1. 先定义一个抽象茶的父类，定义茶的名字与定义价格的抽象方法

   ```
   //Tea.java
   abstract class Tea {
       private String name = "茶";
   
       public String getName() { return name; }
   
       void setName(String name) { this.name = name; }
   
       public abstract int price();
   }
   ```

2. 接着定义配料的抽象类，所有配料都来继承这个东东

   ```
   //Decorator.java
   abstract class Decorator extends Tea{
       public abstract String getName();
   }
   ```

3. 接着定义基本茶品，奶茶和柠檬茶

   ```
   //MilkTea.java
   class MilkTea extends Tea {
       MilkTea() { setName("奶茶"); }
       @Override public int price() { return 5; }
   }
   //LemonTea.java
   class LemonTea extends Tea{
       LemonTea() { setName("柠檬茶"); }
       @Override public int price() { return 3; }
   }
   ```

4. 接着是各种配料，珍珠，椰果，红豆，金桔，都是继承配料类

   ```java
   //ZhenZhu.java
   class ZhenZhu extends Decorator {
   
       Tea tea;
   
       ZhenZhu(Tea tea) { this.tea = tea; }
   
       @Override public String getName() { return "珍珠" + tea.getName(); }
   
       @Override public int price() { return 2 + tea.price(); }
   }
   //YeGuo.java
   class YeGuo extends Decorator{
   
       Tea tea;
   
       YeGuo(Tea tea) { this.tea = tea; }
   
       @Override public String getName() { return "椰果" + tea.getName(); }
   
       @Override  public int price() { return 2 + tea.price(); }
   }
   //HongDou.java
   class HongDou extends Decorator{
   
       Tea tea;
   
       HongDou(Tea tea) { this.tea = tea; }
   
       @Override public String getName() { return "红豆" + tea.getName(); }
   
       @Override  public int price() { return 2 + tea.price(); }
   }
   //JinJu.java
   class JinJu extends Decorator{
   
       Tea tea;
   
       JinJu(Tea tea) { this.tea = tea; }
   
       @Override public String getName() { return "金桔" + tea.getName(); }
   
       @Override public int price() { return 1 + tea.price(); }
   }
   
   ```

5. 接着开始自由搭配了：

   ```java
   //Store.java
   public class Store {
       public static void main(String[] args) {
           Tea tea1 = new MilkTea();
           System.out.println("你点的是：" + tea1.getName() + " 价格为：" + tea1.price());
   
           Tea tea2 = new LemonTea();
           tea2 = new JinJu(tea2);
           System.out.println("你点的是：" + tea2.getName() + " 价格为：" + tea2.price());
   
           Tea tea3 = new MilkTea();
           tea3 = new ZhenZhu(tea3);
           tea3 = new YeGuo(tea3);
           tea3 = new HongDou(tea3);
           tea3 = new JinJu(tea3);
           System.out.println("你点的是：" + tea3.getName() + " 价格为：" + tea3.price());
       }
   }
   ```

   可以，没毛病，如果你要**奶盖果冻黑钻烧仙草珍珠椰果奶茶**，也无压力， 没新增一个配料就建一个类而已,不用每次都去新建一堆类，然后又 去继承。

**四个角色**：

- Component：抽象组件，可以是接口或抽象类，具体组件与抽象装饰类的共同父类，声明了在具体组件中实现的业务方法，可以使客户端以一致的方式处理未修饰对象与修饰后的对象，实现了客户端的透明操作，比如这里的Tea类。
- ConcreteComponent：具体组件，实现抽象组件中生命的方法，装饰器类可以给他增加额外的责任(方法)，比如这里的MilkTea和LemonTea。
- Decorator：抽象装饰类，装饰组件对象的，内部一定要有一个指向组件对象的引用！！！通过该引用可以调用装饰前构建对象的方法，并通过其子类扩展该方法，已达到装饰的目的，比如这里的Decorator类。
- ConcreteDecorator：具体装饰类，抽象装饰类的具体实现，可以调用抽象
  装饰类中定义的方法，也可以新增新的方法来扩充对象的行为。

UML类图

![5.png](https://i.loli.net/2019/06/04/5cf6888976e0359120.png)



### 适配器模式

适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。

这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。举个真实的例子，读卡器是作为内存卡和笔记本之间的适配器。您将内存卡插入读卡器，再将读卡器插入笔记本，这样就可以通过笔记本来读取内存卡。

**意图：**将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

**主要解决：**主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

**何时使用：**

1. 系统需要使用现有的类，而此类的接口不符合系统的需要。 
2. 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。
3. 通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。） 

**如何解决：**继承或依赖（推荐）。

**关键代码：**适配器继承或依赖已有的对象，实现想要的目标接口。

**应用实例：** 

1. 美国电器 110V，中国 220V，就要有一个适配器将 110V 转化为 220V。 
2. JAVA JDK 1.1 提供了 Enumeration 接口，而在 1.2 中提供了 Iterator 接口，想要使用 1.2 的 JDK，则要将以前系统的 Enumeration 接口转化为 Iterator 接口，这时就需要适配器模式。 
3. 在 LINUX 上运行 WINDOWS 程序。 
4. JAVA 中的 jdbc。 
5. 在 JAVA的IO类库中有很多这样的需求，如将字符串数据转变成字节数据保存到文件中，将字节数据转变成流数据等。

**优点：** 

1. 可以让任何两个没有关联的类一起运行。 
2. 提高了类的复用。
3. 增加了类的透明度。
4. 灵活性好。 

**缺点：**

1. 过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。
2. 由于 JAVA 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。 

**使用场景：**有动机地修改一个正常运行的系统的接口，这时应该考虑使用适配器模式。

**注意事项：**适配器不是在详细设计时添加的，而是解决正在服役的项目的问题。

**两种适配器模式**：

根据适配器类与适配者类的关系不同，适配器模式可分为 类适配器 和对象适配器两种，尽管都是三个角色，但还是有些差别的！

另外，类适配器的适配器和适配者是 继承 关系，而对象适配器则是 引用 关系。

**类适配器的三个角色：**

- 目标接口(Target)：客户所期待的接口，目标是接口；
- 需要适配的类(Adaptee)：需要适配的类或适配者类；
- 适配器(Adapter)：实现了抽象目标类接口Target，并继承了适配者类Adaptee

**对象适配器的三个角色**：

- **目标接口**(Target)：客户所期待的接口，可以是**具体的或抽象的类**，也可以是**接口**
- **需要适配的类**(Adaptee)：需要适配的类或适配者类；
- **适配器**(Adapter)：通过包装一个需要适配的对象，把原接口转换成目标接口；

UML类图

![2.png](https://i.loli.net/2019/06/04/5cf67daf87e0e86725.png)

![3.png](https://i.loli.net/2019/06/04/5cf67dafa341a63110.png)

**对象适配器**支持传入一个被适配器对象，因此可以做到对多种被适 配接口进行适配。而类适配器直接继承，无法动态修改，所以一般情况 下对象适配器使用得更多！（Java不支持多重继承！！！）

当不需要实现一个接口所提供的所有方法时，可先设计一个抽象类实现该接口，并为接口中每个方法提供一个默认实现（空方法），那么该抽象类的子类可以选择性地覆盖父类的某些方法来实现需求，它适用于不想使用一个接口中的所有方法的情况，又称为单接口适配器模式。

简单点说就是：不想实现接口里的所有方法，写个基类去继承这个接口，然后重写所有方法，子类再去继承这个基类，按需重写！

#### 示例

简单的抽象一个场景：手机充电需要将220V的交流电转化为手机锂电池需要的5V直流电，我们的demo就是写一个电源适配器，将 AC220v ——> DC5V，其实适配器模式可以简单的分为三类：类适配器模式、对象的适配器模式、接口的适配器模式。我们就以这三种模式来实现上述步骤。

**类适配器模式**

就上面提到的功能，简单的使用类适配器模式，Source类如下：

```
public class AC220 {
    public int output220V(){
        int output = 220;
        return output;
    }
}

```

我们的目标类Destination,只需要定义方法，由适配器来转化：

```
public interface DC5 {
    int output5V();
}
```

Adapter类如下：

```
public class PowerAdapter extends AC220 implements DC5 {
    @Override
    public int output5V() {
        int output = output220V();
        return (output / 44);
    }
}

```

对于使用，也很简单：

```
 private void initClassAdapter() {
        DC5 dc5 = new PowerAdapter();
        dc5.output5V();
    }

```

因为java单继承的缘故，Destination类必须是接口，以便于Adapter去继承Source并实现Destination，完成适配的功能，但这样就导致了Adapter里暴露了Source类的方法，使用起来的成本就增加了。

**对象适配器模式**

对于同样的逻辑，我们在以对象适配器模式实现。我们保留AC220和DC5两个基本类，我们让Adapter持有Destination类的实例，然后再实现DC5，以这种持有对象的方式来实现适配器功能：

```
public class PowerAdapter implements DC5{
    private AC220 mAC220;

    public PowerAdapter(AC220 ac220){
        this.mAC220 = ac220;
    }

    @Override
    public int output5V() {
        int output = 0;
        if (mAC220 != null) {
            output = mAC220.output220V() / 44;
        }
        return output;
    }
}

```

使用

```
    /**
     * 对象适配器模式demo
     */
    private void initObjAdapter() {
  PowerAdapter adapter = new PowerAdapter(new AC220());
        adapter.output5V();
    }

```

对象适配器和类适配器其实算是同一种思想，只不过实现方式不同。再回想装饰者模式，装饰者是对Source的装饰，使用者毫无察觉到Source被装饰，也就是用法不变。而对于适配器模式用法还是有改变的。

**接口适配器模式**

对于接口适配器模式，我们就不用担着眼于220->5,我们的接口可以有更多的抽象方法，这一点在android开发中有很多影子，动画的适配器有很多接口，但我们只需要关心我们需要的回调方法（详见AnimatorListenerAdapter类），我们把接口比作万能适配器：

```
public interface DCOutput {
    int output5V();
    int output9V();
    int output12V();
    int output24V();
}

```

然后我们要用的是5V的电压，所以关心5V的适配：

```
public class Power5VAdapter extends PowerAdapter {

    public Power5VAdapter(AC220 ac220) {
        super(ac220);
    }

    @Override
    public int output5V() {
        int output = 0;
        if (mAC220 != null) {
            output = mAC220.output220V() / 44;
        }
        return output;
    }
}

```

但是我们必须存在一个中间适配器，用于实现默认的接口方法，以至于减少我们适配器的代码量，让代码更加清晰：

```
/**
 * Created by italkbb on 2018/1/24.
 */
public abstract class PowerAdapter implements DCOutput{
    protected AC220 mAC220;

    public PowerAdapter(AC220 ac220){
        this.mAC220 = ac220;
    }

    @Override
    public int output5V() {
        return mAC220.output220V();
    }

    @Override
    public int output9V() {
        return mAC220.output220V();
    }

    @Override
    public int output12V() {
        return mAC220.output220V();
    }

    @Override
    public int output24V() {
        return mAC220.output220V();
    }
}

```

这样一来我们就只需要重写父类我们关心的方法了，当然我们有时候可以省略Power5VAdapter类，因为内部类可以实现我们的方法，就跟使用setOnClickOnLintener(new OnClickOnLintener(){…})一样，我们来看使用：

```
    /**
     * 接口适配器模式demo
     */
    private void initinterfaceAdapter() {
        // 已经实现了子类
        Power5VAdapter power5VAdapter = new Power5VAdapter(new AC220());
        power5VAdapter.output5V();

        // 直接实现子类
        PowerAdapter powerAdapter = new PowerAdapter(new AC220()) {
            @Override
            public int output5V() {
                int output = 0;
                if (mAC220 != null) {
                    output = mAC220.output220V() / 44;
                }
                return output;
            }
        };
        powerAdapter.output5V();
    }
```

这样也实现了这个适配功能，而且可以说易于扩展。

android开发中，ListView、RecyclerView等列表展示组件填充数据都会用到Adapter，这里面的Source相当于Datas，Destination相当于Views，然后Adapter去协调数据与显示。使用适配器模式会拥有更高的复用性以及更好的扩展性，但是过多的使用适配器模式，代码的可阅读性会有所下降，但是设计模式应该是对于相对应的情况，而不是盲目的使用。

装饰器与适配器都有一个别名叫做 包装模式(Wrapper)，它们看似都是起到包装一个类或对象的作用，但是使用它们的目的很不一一样。适配器模式的意义是要将一个接口转变成另一个接口，它的目的是通过改变接口来达到重复使用的目的。

而装饰器模式不是要改变被装饰对象的接口，而是恰恰要保持原有的接口，但是增强原有对象的功能，或者改变原有对象的处理方式而提升性能。所以这两个模式设计的目的是不同的。

### 组合模式

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

这种模式创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式。

组合模式，就是在一个对象中包含其他对象，这些被包含的对象可能是终点对象（不再包含别的对象），也有可能是非终点对象（其内部还包含其他对象，或叫组对象），我们将对象称为节点，即一个根节点包含许多子节点，这些子节点有的不再包含子节点，而有的仍然包含子节点，以此类推。

所谓组合模式，其实说的是对象包含对象的问题，通过组合的方式（在对象内部引用对象）来进行布局，我认为这种组合是区别于继承的，而另一层含义是指树形结构子节点的抽象（将叶子节点与数枝节点抽象为子节点），区别于普通的分别定义叶子节点与数枝节点的方式

**意图：**将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

**主要解决：**它在我们树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以向处理简单元素一样来处理复杂元素，从而使得客户程序与复杂元素的内部结构解耦。

**何时使用：** 

1. 您想表示对象的部分-整体层次结构（树形结构）。
2. 您希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象。 

**如何解决：**树枝和叶子实现统一接口，树枝内部组合该接口。

**关键代码：**树枝内部组合该接口，并且含有内部属性 List，里面放 Component。

**应用实例：** 

1. 算术表达式包括操作数、操作符和另一个操作数，其中，另一个操作符也可以是操作数、操作符和另一个操作数。
2. 在 JAVA AWT 和 SWING 中，对于 Button 和 Checkbox 是树叶，Container 是树枝。 

**优点：** 

1. 高层模块调用简单。 
2. 节点自由增加。 

**缺点：**在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则。

**使用场景：**部分、整体场景，如树形菜单，文件、文件夹的管理。

**注意事项：**定义时为具体类。

#### 示例

举个简单例子，菜单和菜品，同样是以小猪的奶茶店为例子：

![6.png](https://i.loli.net/2019/06/05/5cf72dc7407f322439.png)

假设这两类需求如下：

**菜单**：菜单名，描述信息，添加，添加删除子菜单或菜品 。递归打印出所有的子菜单与菜品！

**菜品**：菜名，描述信息，价格，打印信息

1. 抽象出即可代表菜单又可代表菜品的类，这里我们只需要一个 add，get，getString三个抽象方法，让菜单和菜品去继承，菜品 只需完成getString方法重写，菜单需要重写add和get方法。

   ```
   //AbstractMenu.java
   abstract class AbstractMenu {
       public abstract void add(AbstractMenu menu);
       public abstract AbstractMenu get(int index);
       public abstract String getString();
   }
   ```

2. 四个菜品类

   ```
   //FishBall.java
   class FishBall extends AbstractMenu {
   
       private String name;
       private String desc;
       private int price;
   
       FishBall(String name, String desc, int price) {
           this.name = name;
           this.desc = desc;
           this.price = price;
       }
   
       @Override public void add(AbstractMenu menu) { /*未使用*/ }
   
       @Override public AbstractMenu get(int index) { return null; }
   
       @Override public String getString() {
           return " - 【鱼蛋】* " + name + " 标注：" + desc + " 价格：" + price;
       }
   }
   //HandCake.java
   class HandCake extends AbstractMenu {
   
       private String name;
       private String desc;
       private int price;
   
       HandCake(String name, String desc, int price) {
           this.name = name;
           this.desc = desc;
           this.price = price;
       }
   
       @Override public void add(AbstractMenu menu) { /*未使用*/ }
   
       @Override public AbstractMenu get(int index) { return null; }
   
       @Override public String getString() {
           return " - 【手抓饼】* " + name + " 标注：" + desc + " 价格：" + price;
       }
   }
   //Juice.java
   class Juice extends AbstractMenu {
   
       private String name;
       private String desc;
       private int price;
   
       Juice(String name, String desc, int price) {
           this.name = name;
           this.desc = desc;
           this.price = price;
       }
   
       @Override public void add(AbstractMenu menu) { /*未使用*/ }
   
       @Override public AbstractMenu get(int index) { return null; }
   
       @Override public String getString() {
           return " - 【果汁】* " + name + " 标注：" + desc + " 价格：" + price;
       }
   }
   //MilkTea.java
   class MilkTea extends AbstractMenu {
       private String name;
       private String desc;
       private int price;
   
       MilkTea(String name, String desc, int price) {
           this.name = name;
           this.desc = desc;
           this.price = price;
       }
   
       @Override public void add(AbstractMenu menu) { /*未使用*/ }
   
       @Override public AbstractMenu get(int index) { return null; }
   
       @Override public String getString() {
           return " - 【奶茶】* " + name + " 标注：" + desc + " 价格：" + price;
       }
   }
   //MilkTea.java
   class MilkTea extends AbstractMenu {
       private String name;
       private String desc;
       private int price;
   
       MilkTea(String name, String desc, int price) {
           this.name = name;
           this.desc = desc;
           this.price = price;
       }
   
       @Override public void add(AbstractMenu menu) { /*未使用*/ }
   
       @Override public AbstractMenu get(int index) { return null; }
   
       @Override public String getString() {
           return " - 【奶茶】* " + name + " 标注：" + desc + " 价格：" + price;
       }
   }
   
   
   ```

3. 菜单类

   ```
   //Menu.java
   class Menu extends AbstractMenu {
       private String name;
       private String desc;
       private List<AbstractMenu> menus = new ArrayList<>();
   
       Menu(String name, String desc) {
           this.name = name;
           this.desc = desc;
       }
   
       @Override public void add(AbstractMenu menu) { menus.add(menu); }
   
       @Override public AbstractMenu get(int index) { return menus.get(index); }
   
       @Override public String getString() {
           StringBuilder sb = new StringBuilder("\n【菜单】：" + name + " 信息：" + desc + "\n");
           for (AbstractMenu menu: menus) { sb.append(menu.getString()).append("\n"); }
           return sb.toString();
       }
   }
   ```

4. 客户端调用

   ```
   //Store
   public class Store {
       public static void main(String[] args) {
           AbstractMenu mainMenu = new Menu("大菜单", "包含所有子菜单");
           AbstractMenu drinkMenu = new Menu("饮品菜单", "都是喝的");
           AbstractMenu eatMenu = new Menu("小吃菜单", "都是吃的");
           AbstractMenu milkTea = new MilkTea("珍珠奶茶", "奶茶+珍珠", 5);
           AbstractMenu juice = new Juice("鲜榨猕猴桃枝", "无添加即榨", 8);
           AbstractMenu ball = new FishBall("咖喱鱼蛋", "微辣", 6);
           AbstractMenu cake = new HandCake("培根手抓饼", "正宗台湾风味", 8);
   
           drinkMenu.add(milkTea);
           drinkMenu.add(juice);
           eatMenu.add(ball);
           eatMenu.add(cake);
           mainMenu.add(drinkMenu);
           mainMenu.add(eatMenu);
   
           System.out.println(mainMenu.getString());
       }
   }
   ```

   

**三个角色**

上面也说了合并模式是用一种树状的结构来组合对象，三个名词根节点，枝结点，叶子结点，类比上面那个菜单的图，根节点是菜单，枝结点是饮料菜单和小吃菜单，叶子结点是奶茶，果汁，手抓饼和鱼蛋！

- Component：抽象组件，为组合中的对象声明接口，让客户端可以通过这个接口来访问和管理整个对象结构，可以在里面为定义的功能提供缺省的实现，比如上面的AbstractMenu类。
- Composite：容器组件，继承抽象组件，实现抽象组件中与叶子组件相关的操作，比如上面的Menu类重写了get，set方法。
- Leaf：叶子组件，定义和实现叶子对象的行为，不再包含其它的子节点对象，比如上面的MilkTea，Juice，HandCake，FishBall。

UML类图

![7.png](https://i.loli.net/2019/06/05/5cf72e8beed5e46413.png)

### 桥接模式

桥接（Bridge）是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。

这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。

**意图：**将抽象部分与实现部分分离，使它们都可以独立的变化。

**主要解决：**在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活。

**何时使用：**实现系统可能有多个角度分类，每一种角度都可能变化。

**如何解决：**把这种多角度分类分离出来，让它们独立变化，减少它们之间耦合。

**关键代码：**抽象类依赖实现类。

**应用实例：** 

1. 猪八戒从天蓬元帅转世投胎到猪，转世投胎的机制将尘世划分为两个等级，即：灵魂和肉体，前者相当于抽象化，后者相当于实现化。生灵通过功能的委派，调用肉体对象的功能，使得生灵可以动态地选择。
2. 墙上的开关，可以看到的开关是抽象的，不用管里面具体怎么实现的。

**优点：** 

1. 抽象和实现的分离。
2. 优秀的扩展能力。 
3. 实现细节对客户透明。 

**缺点：**桥接模式的引入会增加系统的理解与设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计与编程。

**使用场景：**

1. 如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。
2. 对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。 
3. 一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。 

**注意事项：**对于两个独立变化的维度，使用桥接模式再适合不过了。

#### 示例

1. 创建桥接实现接口。

   ```
   //DrawAPI.java
   public interface DrawAPI {
      public void drawCircle(int radius, int x, int y);
   }
   ```

2. 创建实现了 *DrawAPI* 接口的实体桥接实现类。

   ```
   //RedCircle.java
   public class RedCircle implements DrawAPI {
      @Override
      public void drawCircle(int radius, int x, int y) {
         System.out.println("Drawing Circle[ color: red, radius: "
            + radius +", x: " +x+", "+ y +"]");
      }
   }
   
   //GreenCircle.java
   public class GreenCircle implements DrawAPI {
      @Override
      public void drawCircle(int radius, int x, int y) {
         System.out.println("Drawing Circle[ color: green, radius: "
            + radius +", x: " +x+", "+ y +"]");
      }
   }
   ```

3. 使用 *DrawAPI* 接口创建抽象类 *Shape*。

   ```
   //Shape.java
   public abstract class Shape {
      protected DrawAPI drawAPI;
      protected Shape(DrawAPI drawAPI){
         this.drawAPI = drawAPI;
      }
      public abstract void draw();  
   }
   
   ```

4. 创建实现了 *Shape* 接口的实体类。

   ```
   //Circle.java
   public class Circle extends Shape {
      private int x, y, radius;
    
      public Circle(int x, int y, int radius, DrawAPI drawAPI) {
         super(drawAPI);
         this.x = x;  
         this.y = y;  
         this.radius = radius;
      }
    
      public void draw() {
         drawAPI.drawCircle(radius,x,y);
      }
   }
   
   ```

5. 使用 *Shape* 和 *DrawAPI* 类画出不同颜色的圆。

   ```
   //BridgePatternDemo.java
   public class BridgePatternDemo {
      public static void main(String[] args) {
         Shape redCircle = new Circle(100,100, 10, new RedCircle());
         Shape greenCircle = new Circle(100,100, 10, new GreenCircle());
    
         redCircle.draw();
         greenCircle.draw();
      }
   }
   
   ```

6. 执行程序，输出结果：

   ```
   Drawing Circle[ color: red, radius: 10, x: 100, 100]
   Drawing Circle[  color: green, radius: 10, x: 100, 100]
   ```

桥接模式基于单一职责原则，如果系统中的类存在多个变化的维度，通过该模式可以将这几个维度分离出来，然后进行独立扩展。这些分离开来的维度，通过在抽象层持有其他维度的引用来进行关联，就好像在两个维度间搭了桥一样，所以叫桥接模式。

把**变化的因素都分离出来**(抽象类)，让他**独立变化**(继承)，然后在程序运行时 动态地将抽象与实现部分组合起来。而多维桥接需要互相持有

**四个角色**

- Abstraction：抽象部分的接口。通常在这个对象里面，要维护一个实现部分
  的对象引用，在抽象对象里面的方法，需要调用实现部分的对象来完成。
  这个对象里面的方法，通常都是跟具体的业务相关的方法。
- Refined Abstraction：扩展抽象部分的接口，通常在这些对象里面，
  定义跟实际业务相关的方法，这些方法的实现通常会使用Abstraction中
  定义的方法，也可能需要调用实现部分的对象来完成。

- Implementor：定义实现部分的接口，这个接口不用和Abstraction
  里面的方法一致，通常是由Implementor接口提供基本的操作，而
  Abstraction里面定义的是基于这些基本操作的业务方法，也就是说
  Abstraction定义了基于这些基本操作的较高层次的操作。

- ConcreteImplementor：真正实现Implementor接口的对象。

UML类图

![8.png](https://i.loli.net/2019/06/05/5cf7428680e7671247.png)

参考：[如何让孩子爱上设计模式 ——10.桥接模式(Bridge Pattern)](https://blog.csdn.net/coder_pig/article/details/54885435)

