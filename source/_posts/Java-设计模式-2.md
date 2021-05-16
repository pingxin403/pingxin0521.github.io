---
title: 设计模式--创造型
date: 2019-05-22 14:18:59
tags:
 - Java
 - 设计模式
categories:
 - Java
 - 设计模式
---

### 单例模式

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

<!--more-->

这种模式涉及到一个单一的类，该类负责创建自己的对象，同时确保只有单个对象被创建。这个类提供了一种访问其唯一的对象的方式，可以直接访问，不需要实例化该类的对象。

**注意：** 

- 单例类只能有一个实例：私有化该类的构造函数
- 单例类必须自己创建自己的唯一实例:通过new在本类中创建一个本类对象
- 单例类必须给所有其他对象提供这一实例:定义一个公有的方法，将在该类中所创建的对象返回

**意图：**保证一个类仅有一个实例，并提供一个访问它的全局访问点。

**主要解决：**一个全局使用的类频繁地创建与销毁。

**何时使用：**当您想控制实例数目，节省系统资源的时候。

**如何解决：**判断系统是否已经有这个单例，如果有则返回，如果没有则创建。

**关键代码：**构造函数是私有的。

**应用实例：**

- 一个班级只有一个班主任。
- Windows 是多进程多线程的，在操作一个文件的时候，就不可避免地出现多个进程或线程同时操作一个文件的现象，所以所有文件的处理必须通过唯一的实例来进行。
- 一些设备管理器常常设计为单例模式，比如一个电脑有两台打印机，在输出的时候就要处理不能两台打印机打印同一个文件。 

**优点：**

- 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。
- 避免对资源的多重占用（比如写文件操作）。 

**缺点：**没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

**使用场景：**

-  要求生产唯一序列号。 
- WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。
-  创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。

**注意事项：**getInstance() 方法中需要使用同步锁 synchronized (Singleton.class) 防止多线程同时进入造成 instance 被多次实例化。

#### 示例

1. 饿汉模式

   ```java
   public class Singleton () {
       private static Singleton instance = new Singleton()
       private Singleton(){ }
       public static Singleton getInstance() {
           return instance;
       }
   }
   ```

   获取单例对象：`Singleton instace = Singleton.getInstance();`

   优点：类加载的时候就完成实例化，避免线程同步问题

   缺点：由于在类加载的时候就进行了实例化，所以没有达到Lazy Loading(懒加载)的效果，即使我们没用到这个实例，但是他还是会加载，从而造成内存浪费(可以忽略，最常用的一种)。

2. 懒汉模式(线程不安全，不可用)

   ```java
   public class Singleton {
       private static Singleton instance = null;
       private Singleton() { }
       private static Singleton getInstance() {
           if(instance == null) {
               instance = new Singleton();
           }
           return instance;
       }
   }
   ```

   尽管达到了懒加载，但是却存在线程安全问题，比如有两个线程，刚好都执行完`if(instance == null)`，接着准备执行`instance = new Singleton()`语句，这样的结果会导致我们实例化了两个Singleton对象，这就是懒汉单例模式可能会引发的线程安全问题，解决这个方法，我们可以对getInstance方法加锁。

3. 懒汉模式(线程安全，但效率低，不推荐使用)

   ```java
   public class Singleton { 
       private Singleton instance = null;
       private Singleton() { }
       public static synchronized Singleton getInstance() {
           if(instance == null) {
               instance = new Singleton();
           }
           return instance;
       }
   }
   ```

   尽管保证了线程安全，但是每个线程在想要获得实例时，执行getInstance()方法都需要进行同步，而实例化代码只需执行一次就够了，后面获取该实例，直接return即可，方法进行同步效率太低，需要改进。还有一种写法是：`synchronized (Singleton.class) { instance = new Singleton(); }`这样一样是线程不安全的，如果你想使用懒汉模式的话，推荐使用下面的：DCL单例（双重检查锁定）。

4. 懒汉模式双重校验锁

   ```java
   public class Singleton {
       private static volatile Singleton instance = null;
       private Singleton() { }
       public static Singleton getInstance() {
           if(instance == null) {
               synchronized(Singleton.class) {
                   if(instance == null) {
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   }
   ```

   代码中进行了两次if检查，这样就可以保证线程安全，初始化一次后，后面再次访问时，if检查，直接return 实例化对象。volatile是1.5后引入的，volatile关键字会屏蔽Java虚拟机所做的一些代码优化，会导致系统运行效率降低，而更好的写法是使用静态内部类来实现单例！

5. 静态内部类实现单例模式

   ```java
   public class Singleton {
       private Singleton() { }
       public static final Singleton getInstance() {
           return SingletonHolder.INSTANCE;
       }
       private static class SingletonHolder {
           private static final Singleton INSTANCE = new Singleton();
       }
   }
   ```

   和饿汉式类似，两者都是通过类装载机制来保证初始化实例的时候只有一个线程，从而避免线程安全问题，饿汉式的Singleton类被加载，就会实例化，而静态内部类这种，当Singleton类被加载时，不会立即实例化，调用getInstance方法，才会装载SingletonHolder类，从而完成Singleton的实例化。

6. 枚举实现单例模式

   ```java
   public enum SingletonEnum {
       INSTANCE;
       private Singleton instance;
       SingletonEnum() { 
           instance = new Singleton();
       }
       public Singleton getInstance() {
           return instance;
       }
   }
   ```

   访问方式：`SingletonEnum.INSTANCE.method();`

   INSTANCE即为SingletonEnum类型的引用，得到它就可以调用枚举中的方法。既避免了线程安全问题，还能防止反序列化重新创建新的对象，但是失去了类的一些特性，没有延时加载，推荐使用。

7. 使用容器实现单例模式

   ```java
   public class SingletonManager {
       private static Map<String,Object> objMap=new ConcurrentHashMap<>();
       public static void registerService(String key,Object instance) {
           if(!objMap.containsKey(key)) {
               objMap.put(key,instance);
           }
       }
       public static Object getService(String key) {
           return objMap.get(key);
       }
   }
   ```
   
将多种单例类型注入到一个统一的管理类中，在使用时根据key获取对象对应类型的对象。这种方式使得我们可以管理多种类型的单例，并且在使用时可以通过统一的接口进行获取操作，降低了用户的使用成本，也对用户隐藏了具体实现，降低了耦合度。
   
**优点**：保持类对象唯一性，对于频繁创建和销毁的对象可以提高性能。 
   **缺点**：扩展困难，单例的方法无法生成子类对象，要扩展的话基本要重写这个类。

经验之谈：一般情况下，不建议使用第 2 种和第 3 种懒汉方式，建议使用第 1 种饿汉方式。只有在要明确实现 lazy loading 效果时，才会使用第 5 种登记方式。如果涉及到反序列化创建对象时，可以尝试使用第 6 种枚举方式。如果有其他特殊的需求，可以考虑使用第 4 种双检锁方式。

### 工厂模式

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

**意图：**定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

**主要解决：**主要解决接口选择的问题。

**何时使用：**我们明确地计划不同条件下创建不同实例时。

**如何解决：**让其子类实现工厂接口，返回的也是一个抽象的产品。

**关键代码：**创建过程在其子类执行。

**应用实例：** 

1. 您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 
2. Hibernate 换数据库只需换方言和驱动就可以。 

**优点：** 

1. 一个调用者想创建一个对象，只要知道其名称就可以了
2. 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。
3. 屏蔽产品的具体实现，调用者只关心产品的接口。 

**缺点：**每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。

**使用场景：** 

1. 日志记录器：记录可能记录到本地硬盘、系统事件、远程服务器等，用户可以选择记录日志到什么地方。
2. 数据库访问，当用户不知道最后系统采用哪一类数据库，以及数据库可能有变化时。 
3. 设计一个连接服务器的框架，需要三个协议，"POP3"、"IMAP"、"HTTP"，可以把这三个作为产品类，共同实现一个接口。 

**注意事项：**作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过  new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。

#### 示例

我们将创建一个 *Shape* 接口和实现 *Shape* 接口的实体类。下一步是定义工厂类 *ShapeFactory*。

*FactoryPatternDemo*，我们的演示类使用 *ShapeFactory* 来获取 *Shape* 对象。它将向 *ShapeFactory* 传递信息（*CIRCLE / RECTANGLE / SQUARE*），以便获取它所需对象的类型。

![3.jpg](https://i.loli.net/2019/06/04/5cf66b2c438bf68732.jpg)

1. 创建一个接口

   ```java
   //Shape.java
   public interface Shape {
      void draw();
   }
   ```

   

2. 创建实现接口的实体类。

   ```java
   //Rectangle.java
   public class Rectangle implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Rectangle::draw() method.");
      }
   }
   
   //Square.java
   public class Square implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Square::draw() method.");
      }
   }
   
   //Circle.java
   public class Circle implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Circle::draw() method.");
      }
   }
   
   ```

3. 创建一个工厂，生成基于给定信息的实体类的对象。

   ```java
   //ShapeFactory.java
   public class ShapeFactory {
       
      //使用 getShape 方法获取形状类型的对象
      public Shape getShape(String shapeType){
         if(shapeType == null){
            return null;
         }        
         if(shapeType.equalsIgnoreCase("CIRCLE")){
            return new Circle();
         } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
            return new Rectangle();
         } else if(shapeType.equalsIgnoreCase("SQUARE")){
            return new Square();
         }
         return null;
      }
   }
   ```
   
4. 使用该工厂，通过传递类型信息来获取实体类的对象。

   ```java
   
   public class FactoryPatternDemo {
    
      public static void main(String[] args) {
         ShapeFactory shapeFactory = new ShapeFactory();
    
         //获取 Circle 的对象，并调用它的 draw 方法
         Shape shape1 = shapeFactory.getShape("CIRCLE");
    
         //调用 Circle 的 draw 方法
         shape1.draw();
    
         //获取 Rectangle 的对象，并调用它的 draw 方法
         Shape shape2 = shapeFactory.getShape("RECTANGLE");
    
         //调用 Rectangle 的 draw 方法
         shape2.draw();
    
         //获取 Square 的对象，并调用它的 draw 方法
         Shape shape3 = shapeFactory.getShape("SQUARE");
    
         //调用 Square 的 draw 方法
         shape3.draw();
      }
   }
   
   ```

5. 执行程序，输出结果：

   ```
   Inside Circle::draw() method.
   Inside Rectangle::draw() method.
   Inside Square::draw() method.
   ```

上面示例是简单工厂模式，**简单工厂模式**又叫静态工厂模式

三个角色：

- **Product**(抽象产品)：封装了各种产品对象的公有方法，比如这里的Shape;
- **ConcreteProduct**(具体产品)：抽象产品的具体化，比如这里的Circle、Rectangle和Square;
- **Factory**(工厂)：实现创建所有产品实例的内部逻辑，比如这里的ShapeFactory；

**工厂方法模式**，在简单工厂基础上， 把**工厂**创建不同产品的内部逻辑抽取出来，生成一个 **抽象工厂**，然后创建具体的工厂类，来产生不同的产品！

UML类图：

![6.png](https://i.loli.net/2019/06/04/5cf66eed0ba8d11804.png)



### 抽象工厂模式

抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。该超级工厂又称为其他工厂的工厂。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

 在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

**意图：**提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

**主要解决：**主要解决接口选择的问题。

**何时使用：**系统的产品有多于一个的产品族，而系统只消费其中某一族的产品。

**如何解决：**在一个产品族里面，定义多个产品。

**关键代码：**在一个工厂里聚合多个同类产品。

**应用实例：**工作了，为了参加一些聚会，肯定有两套或多套衣服吧，比如说有商务装（成套，一系列具体产品）、时尚装（成套，一系列具体产品），甚至对于一个家庭来说，可能有商务女装、商务男装、时尚女装、时尚男装，这些也都是成套的，即一系列具体产品。假设一种情况（现实中是不存在的，要不然，没法进入共产主义了，但有利于说明抽象工厂模式），在您的家中，某一个衣柜（具体工厂）只能存放某一种这样的衣服（成套，一系列具体产品），每次拿这种成套的衣服时也自然要从这个衣柜中取出了。用  OOP  的思想去理解，所有的衣柜（具体工厂）都是衣柜类的（抽象工厂）某一个，而每一件成套的衣服又包括具体的上衣（某一具体产品），裤子（某一具体产品），这些具体的上衣其实也都是上衣（抽象产品），具体的裤子也都是裤子（另一个抽象产品）。

**优点：**当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。

**缺点：**产品族扩展非常困难，要增加一个系列的某一产品，既要在抽象的 Creator 里加代码，又要在具体的里面加代码。

**使用场景：** 

1. QQ 换皮肤，一整套一起换。
2. 生成不同操作系统的程序。 

**注意事项：**产品族难扩展，产品等级易扩展。

抽象工厂模式就是由四个角色组成：

- **抽象工厂**：声明一组用于创建产品族的方法，每个方法对应一种产品；
- **具体工厂**：实现抽象工厂创建产品的方法，生成具体产品；
- **抽象产品**：为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法；
- **具体产品**：抽象产品的具体化，实现抽象产品的相关方法；

#### 示例

我们将创建 Shape 和 Color 接口和实现这些接口的实体类。下一步是创建抽象工厂类 AbstractFactory。接着定义工厂类 ShapeFactory 和 ColorFactory，这两个工厂类都是扩展了 AbstractFactory。然后创建一个工厂创造器/生成器类 FactoryProducer。

AbstractFactoryPatternDemo，我们的演示类使用 FactoryProducer 来获取 AbstractFactory 对象。它将向 AbstractFactory 传递形状信息 Shape（CIRCLE / RECTANGLE / SQUARE），以便获取它所需对象的类型。同时它还向 AbstractFactory 传递颜色信息 Color（RED / GREEN / BLUE），以便获取它所需对象的类型。

![4.jpg](https://i.loli.net/2019/06/04/5cf66cb39089e10749.jpg)

1. 为形状创建一个接口。

   ```java
   //Shape.java
   public interface Shape {
      void draw();
   }
   ```

2. 创建实现接口的实体类。

   ```java
   
   //Rectangle.java
   public class Rectangle implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Rectangle::draw() method.");
      }
   }
   
   //Square.java
   public class Square implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Square::draw() method.");
      }
   }
   
   //Circle.java
   public class Circle implements Shape {
    
      @Override
      public void draw() {
         System.out.println("Inside Circle::draw() method.");
      }
   }
   
   ```

3. 为颜色创建一个接口。

   ```java
   //Color.java
   
   public interface Color {
      void fill();
   }
   
   ```

4. 创建实现接口的实体类。

   ```java
   //Red.java
   public class Red implements Color {
    
      @Override
      public void fill() {
         System.out.println("Inside Red::fill() method.");
      }
   }
   
   //Green.java
   public class Green implements Color {
    
      @Override
      public void fill() {
         System.out.println("Inside Green::fill() method.");
      }
   }
   
   //Blue.java
   public class Blue implements Color {
    
      @Override
      public void fill() {
         System.out.println("Inside Blue::fill() method.");
      }
   }
   
   ```

5. 为 Color 和 Shape 对象创建抽象类来获取工厂。

   ```java
   //AbstractFactory.java
   public abstract class AbstractFactory {
      public abstract Color getColor(String color);
      public abstract Shape getShape(String shape) ;
   }
   
   ```

6. 创建扩展了 AbstractFactory 的工厂类，基于给定的信息生成实体类的对象。

   ```java
   
   //ShapeFactory.java
   public class ShapeFactory extends AbstractFactory {
       
      @Override
      public Shape getShape(String shapeType){
         if(shapeType == null){
            return null;
         }        
         if(shapeType.equalsIgnoreCase("CIRCLE")){
            return new Circle();
         } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
            return new Rectangle();
         } else if(shapeType.equalsIgnoreCase("SQUARE")){
            return new Square();
         }
         return null;
      }
      
      @Override
      public Color getColor(String color) {
         return null;
      }
   }
   
   //ColorFactory.java
   public class ColorFactory extends AbstractFactory {
       
      @Override
      public Shape getShape(String shapeType){
         return null;
      }
      
      @Override
      public Color getColor(String color) {
         if(color == null){
            return null;
         }        
         if(color.equalsIgnoreCase("RED")){
            return new Red();
         } else if(color.equalsIgnoreCase("GREEN")){
            return new Green();
         } else if(color.equalsIgnoreCase("BLUE")){
            return new Blue();
         }
         return null;
      }
   }
   
   ```

   

7. 创建一个工厂创造器/生成器类，通过传递形状或颜色信息来获取工厂。

   ```java
   //FactoryProducer.java
   
   public class FactoryProducer {
      public static AbstractFactory getFactory(String choice){
         if(choice.equalsIgnoreCase("SHAPE")){
            return new ShapeFactory();
         } else if(choice.equalsIgnoreCase("COLOR")){
            return new ColorFactory();
         }
         return null;
      }
   }
   
   ```

8. 使用 FactoryProducer 来获取 AbstractFactory，通过传递类型信息来获取实体类的对象。

   ```
   
   AbstractFactoryPatternDemo.java
   public class AbstractFactoryPatternDemo {
      public static void main(String[] args) {
    
         //获取形状工厂
         AbstractFactory shapeFactory = FactoryProducer.getFactory("SHAPE");
    
         //获取形状为 Circle 的对象
         Shape shape1 = shapeFactory.getShape("CIRCLE");
    
         //调用 Circle 的 draw 方法
         shape1.draw();
    
         //获取形状为 Rectangle 的对象
         Shape shape2 = shapeFactory.getShape("RECTANGLE");
    
         //调用 Rectangle 的 draw 方法
         shape2.draw();
         
         //获取形状为 Square 的对象
         Shape shape3 = shapeFactory.getShape("SQUARE");
    
         //调用 Square 的 draw 方法
         shape3.draw();
    
         //获取颜色工厂
         AbstractFactory colorFactory = FactoryProducer.getFactory("COLOR");
    
         //获取颜色为 Red 的对象
         Color color1 = colorFactory.getColor("RED");
    
         //调用 Red 的 fill 方法
         color1.fill();
    
         //获取颜色为 Green 的对象
         Color color2 = colorFactory.getColor("Green");
    
         //调用 Green 的 fill 方法
         color2.fill();
    
         //获取颜色为 Blue 的对象
         Color color3 = colorFactory.getColor("BLUE");
    
         //调用 Blue 的 fill 方法
         color3.fill();
      }
   }
   ```

9. 执行程序，输出结果：

   ```
   Inside Circle::draw() method.
   Inside Rectangle::draw() method.
   Inside Square::draw() method.
   Inside Red::fill() method.
   Inside Green::fill() method.
   Inside Blue::fill() method.
   ```

#### 三种工厂模式区别

下面例子中鼠标，键盘，耳麦为产品，惠普，戴尔为工厂。

**简单工厂模式**

简单工厂模式不是 23 种里的一种，简而言之，就是有一个专门生产某个产品的类。

比如下图中的鼠标工厂，专业生产鼠标，给参数 0，生产戴尔鼠标，给参数 1，生产惠普鼠标。

![7.png](https://i.loli.net/2019/06/04/5cf670fd2f6e087605.png)

**工厂模式**

工厂模式也就是鼠标工厂是个父类，有生产鼠标这个接口。

戴尔鼠标工厂，惠普鼠标工厂继承它，可以分别生产戴尔鼠标，惠普鼠标。

生产哪种鼠标不再由参数决定，而是创建鼠标工厂时，由戴尔鼠标工厂创建。

后续直接调用鼠标工厂.生产鼠标()即可

![8.png](https://i.loli.net/2019/06/04/5cf670fd7714127553.png)

**抽象工厂模式**

抽象工厂模式也就是不仅生产鼠标，同时生产键盘。

也就是 PC 厂商是个父类，有生产鼠标，生产键盘两个接口。

戴尔工厂，惠普工厂继承它，可以分别生产戴尔鼠标+戴尔键盘，和惠普鼠标+惠普键盘。

创建工厂时，由戴尔工厂创建。

后续工厂.生产鼠标()则生产戴尔鼠标，工厂.生产键盘()则生产戴尔键盘。

![9.png](https://i.loli.net/2019/06/04/5cf670fd7b4c863507.png)

在抽象工厂模式中，假设我们需要增加一个工厂

假设我们增加华硕工厂，则我们需要增加华硕工厂，和戴尔工厂一样，继承 PC 厂商。

之后创建华硕鼠标，继承鼠标类。创建华硕键盘，继承键盘类即可。

![10.png](https://i.loli.net/2019/06/04/5cf670fd9beaf39173.png)



在抽象工厂模式中，假设我们需要增加一个产品

假设我们增加耳麦这个产品，则首先我们需要增加耳麦这个父类，再加上戴尔耳麦，惠普耳麦这两个子类。

之后在PC厂商这个父类中，增加生产耳麦的接口。最后在戴尔工厂，惠普工厂这两个类中，分别实现生产戴尔耳麦，惠普耳麦的功能。 

![11.png](https://i.loli.net/2019/06/04/5cf670fd66a4b61305.png)



### 建造者模式

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。

**意图：**将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

**主要解决：**主要解决在软件系统中，有时候面临着"一个复杂对象"的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。

**何时使用：**一些基本部件不会变，而其组合经常变化的时候。

**如何解决：**将变与不变分离开。

**关键代码：**建造者：创建和提供实例，导演：管理建造出来的实例的依赖关系。

**应用实例：** 

1. 去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的"套餐"。 
2. JAVA 中的 StringBuilder。 

**优点：** 

1. 建造者独立，易扩展。
2. 便于控制细节风险。 

**缺点：** 

1. 产品必须有共同点，范围有限制。 
2. 如内部变化复杂，会有很多的建造类。 

**使用场景：**

1.  需要生成的对象具有复杂的内部结构。 
2. 需要生成的对象内部属性本身相互依赖。 

**注意事项：**与工厂模式的区别是：建造者模式更加关注与零件装配的顺序。

**组成部分(四个角色)**

- Product —— 产品类
- Builder —— 抽象的Builder类，规范产品的组建，一般由子类实现具体过程
- ConcreteBuilder —— Builder的实现类，实现Builder类中定义方法，并返回组建好的对象
- Director——统一组装过程(现实开发中，一般被省略，直接使用一个Builder来进行对象的组装

UML类图：

![5.png](https://i.loli.net/2019/06/04/5cf66795a127439973.png)

#### 示例

我们假设一个快餐店的商业案例，其中，一个典型的套餐可以是一个汉堡（Burger）和一杯冷饮（Cold drink）。汉堡（Burger）可以是素食汉堡（Veg Burger）或鸡肉汉堡（Chicken Burger），它们是包在纸盒中。冷饮（Cold drink）可以是可口可乐（coke）或百事可乐（pepsi），它们是装在瓶子中。

我们将创建一个表示食物条目（比如汉堡和冷饮）的 Item 接口和实现 Item 接口的实体类，以及一个表示食物包装的 Packing 接口和实现 Packing 接口的实体类，汉堡是包在纸盒中，冷饮是装在瓶子中。

然后我们创建一个 Meal 类，带有 Item 的 ArrayList 和一个通过结合 Item 来创建不同类型的 Meal 对象的 MealBuilder。BuilderPatternDemo，我们的演示类使用 MealBuilder 来创建一个 Meal。

![2.jpg](https://i.loli.net/2019/06/04/5cf668cca3fea97508.jpg)

1. 创建一个表示食物条目和食物包装的接口。

   ```java
   //Item.java
   public interface Item {
      public String name();
      public Packing packing();
      public float price();    
   }
   
   //Packing.java
   
   public interface Packing {
      public String pack();
   }
   
   ```

2. 创建实现 Packing 接口的实体类。

   ```java
   //Wrapper.java
   public class Wrapper implements Packing {
    
      @Override
      public String pack() {
         return "Wrapper";
      }
   }
   
   //Bottle.java
   public class Bottle implements Packing {
    
      @Override
      public String pack() {
         return "Bottle";
      }
   }
   
   ```

3. 创建实现 Item 接口的抽象类，该类提供了默认的功能。

   ```java
   //Burger.java
   public abstract class Burger implements Item {
    
      @Override
      public Packing packing() {
         return new Wrapper();
      }
    
      @Override
      public abstract float price();
   }
   
   //ColdDrink.java
   public abstract class ColdDrink implements Item {
    
       @Override
       public Packing packing() {
          return new Bottle();
       }
    
       @Override
       public abstract float price();
   }
   
   ```

4. 创建扩展了 Burger 和 ColdDrink 的实体类。

   ```java
   
   //VegBurger.java
   public class VegBurger extends Burger {
    
      @Override
      public float price() {
         return 25.0f;
      }
    
      @Override
      public String name() {
         return "Veg Burger";
      }
   }
   
   //ChickenBurger.java
   public class ChickenBurger extends Burger {
    
      @Override
      public float price() {
         return 50.5f;
      }
    
      @Override
      public String name() {
         return "Chicken Burger";
      }
   }
   
   //Coke.java
   public class Coke extends ColdDrink {
    
      @Override
      public float price() {
         return 30.0f;
      }
    
      @Override
      public String name() {
         return "Coke";
      }
   }
   
   //Pepsi.java
   public class Pepsi extends ColdDrink {
    
      @Override
      public float price() {
         return 35.0f;
      }
    
      @Override
      public String name() {
         return "Pepsi";
      }
   }
   
   ```

5. 创建一个 Meal 类，带有上面定义的 Item 对象。

   ```java
   //Meal.java
   import java.util.ArrayList;
   import java.util.List;
    
   public class Meal {
      private List<Item> items = new ArrayList<Item>();    
    
      public void addItem(Item item){
         items.add(item);
      }
    
      public float getCost(){
         float cost = 0.0f;
         for (Item item : items) {
            cost += item.price();
         }        
         return cost;
      }
    
      public void showItems(){
         for (Item item : items) {
            System.out.print("Item : "+item.name());
            System.out.print(", Packing : "+item.packing().pack());
            System.out.println(", Price : "+item.price());
         }        
      }    
   }
   
   ```

6. 创建一个 MealBuilder 类，实际的 builder 类负责创建 Meal 对象。

   ```java
   
   public class MealBuilder {
    
      public Meal prepareVegMeal (){
         Meal meal = new Meal();
         meal.addItem(new VegBurger());
         meal.addItem(new Coke());
         return meal;
      }   
    
      public Meal prepareNonVegMeal (){
         Meal meal = new Meal();
         meal.addItem(new ChickenBurger());
         meal.addItem(new Pepsi());
         return meal;
      }
   }
   
   ```

7. BuiderPatternDemo 使用 MealBuider 来演示建造者模式（Builder Pattern）。

   ```java
   
   public class BuilderPatternDemo {
      public static void main(String[] args) {
         MealBuilder mealBuilder = new MealBuilder();
    
         Meal vegMeal = mealBuilder.prepareVegMeal();
         System.out.println("Veg Meal");
         vegMeal.showItems();
         System.out.println("Total Cost: " +vegMeal.getCost());
    
         Meal nonVegMeal = mealBuilder.prepareNonVegMeal();
         System.out.println("\n\nNon-Veg Meal");
         nonVegMeal.showItems();
         System.out.println("Total Cost: " +nonVegMeal.getCost());
      }
   }
   
   ```

8. 执行程序，输出结果：

   ```
   Veg Meal
   Item : Veg Burger, Packing : Wrapper, Price : 25.0
   Item : Coke, Packing : Bottle, Price : 30.0
   Total Cost: 55.0
   
   
   Non-Veg Meal
   Item : Chicken Burger, Packing : Wrapper, Price : 50.5
   Item : Pepsi, Packing : Bottle, Price : 35.0
   Total Cost: 85.5
   ```

### 原型模式

原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。

**意图：**用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

**主要解决：**在运行期建立和删除原型。

**何时使用：** 

1. 当一个系统应该独立于它的产品创建，构成和表示时。
2. 当要实例化的类是在运行时刻指定时，例如，通过动态装载。
3. 为了避免创建一个与产品类层次平行的工厂类层次时。
4. 当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。

**如何解决：**利用已有的一个原型对象，快速地生成和原型对象一样的实例。

**关键代码：** 

1. 实现克隆操作，在 JAVA 继承 Cloneable，重写 clone()，在 .NET 中可以使用 Object 类的 MemberwiseClone() 方法来实现对象的浅拷贝或通过序列化的方式来实现深拷贝。
2. 原型模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系，它同样要求这些"易变类"拥有稳定的接口。 

**应用实例：** 

1. 细胞分裂。 
2. JAVA 中的 Object clone() 方法。 

**优点：** 

1. 性能提高。 
2. 逃避构造函数的约束。 

**缺点：** 

1. 配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。 
2. 必须实现 Cloneable 接口。  

**使用场景：** 

1. 资源优化场景。 
2. 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。
3. 性能和安全要求的场景。 
4. 通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。
5. 一个对象多个修改者的场景。 
6. 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。
7. 在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。 

**注意事项：**与通过对一个类进行实例化来构造新对象不同的是，原型模式是通过拷贝一个现有对象生成新对象的。浅拷贝实现 Cloneable，重写，深拷贝是通过实现 Serializable 读取二进制流。

**组成部分（三个角色）：**

- Prototype —— 声明一个克隆自身的接口，用于约束想要
  克隆自己的类，要求实现定义的克隆方法。
- ConcretePrototype —— 实现Prototype接口的类，这些类
  真正实现克隆自身的相关代码。
- Client —— 客户端用户，调用类

![1.png](https://i.loli.net/2019/06/04/5cf67721cffeb82261.png)

**Java中的 == 与 equals 的区别**

- ==，如果是对比的基本数据类型(int,long等)，比较存储的值是否相等，
  如果对比的是引用型的变量，比较的是所指向的对象地址是否相等

- equals，不能用于比较基本数据类型，如果没对equals()方法进行
  重写，比较的是指向的对象地址，如果想要比较对象内容，需要自行重写
  方法，做相应的判断！！！！String调equals是可以判断内容是否一样，是
  因为对equals()方法进行了重写，

**克隆必须满足的条件（三个）**

1. 对任何的对象x，都有：x.clone()!=x ，即不是同一对象
2. 对任何的对象x，都有：x.clone().getClass==x.getClass()，即对象类型一致
3. 如果对象obj的equals()方法定义恰当的话，那么obj.clone().equals(obj)应当是成立的。(推荐，不强制)

**Java中如何使用：**

Prototype原型类(想被克隆的类)实现Cloneable接口，重写clone()方法。

调用：

```java
ConcretePrototype cp1 = new ConcretePrototype();
ConcretePrototype cp2 = (ConcretePrototype)cp1.clone();
```

#### 示例

我们将创建一个抽象类 Shape 和扩展了 Shape 类的实体类。下一步是定义类 ShapeCache，该类把 shape 对象存储在一个 Hashtable 中，并在请求的时候返回它们的克隆。

PrototypePatternDemo，我们的演示类使用 ShapeCache 类来获取 Shape 对象。



1. 创建一个实现了 *Cloneable* 接口的抽象类。

   ```java
   
   //Shape.java
   public abstract class Shape implements Cloneable {
      
      private String id;
      protected String type;
      
      abstract void draw();
      
      public String getType(){
         return type;
      }
      
      public String getId() {
         return id;
      }
      
      public void setId(String id) {
         this.id = id;
      }
      
      public Object clone() {
         Object clone = null;
         try {
            clone = super.clone();
         } catch (CloneNotSupportedException e) {
            e.printStackTrace();
         }
         return clone;
      }
   }
   
   ```

2. 创建扩展了上面抽象类的实体类。

   ```java
   
   //Rectangle.java
   public class Rectangle extends Shape {
    
      public Rectangle(){
        type = "Rectangle";
      }
    
      @Override
      public void draw() {
         System.out.println("Inside Rectangle::draw() method.");
      }
   }
   
   //Square.java
   public class Square extends Shape {
    
      public Square(){
        type = "Square";
      }
    
      @Override
      public void draw() {
         System.out.println("Inside Square::draw() method.");
      }
   }
   
   //Circle.java
   public class Circle extends Shape {
    
      public Circle(){
        type = "Circle";
      }
    
      @Override
      public void draw() {
         System.out.println("Inside Circle::draw() method.");
      }
   }
   
   ```

3. 创建一个类，从数据库获取实体类，并把它们存储在一个 *Hashtable* 中。

   ```java
   //ShapeCache.java
   import java.util.Hashtable;
    
   public class ShapeCache {
       
      private static Hashtable<String, Shape> shapeMap 
         = new Hashtable<String, Shape>();
    
      public static Shape getShape(String shapeId) {
         Shape cachedShape = shapeMap.get(shapeId);
         return (Shape) cachedShape.clone();
      }
    
      // 对每种形状都运行数据库查询，并创建该形状
      // shapeMap.put(shapeKey, shape);
      // 例如，我们要添加三种形状
      public static void loadCache() {
         Circle circle = new Circle();
         circle.setId("1");
         shapeMap.put(circle.getId(),circle);
    
         Square square = new Square();
         square.setId("2");
         shapeMap.put(square.getId(),square);
    
         Rectangle rectangle = new Rectangle();
         rectangle.setId("3");
         shapeMap.put(rectangle.getId(),rectangle);
      }
   }
   
   ```

4. *PrototypePatternDemo* 使用 *ShapeCache* 类来获取存储在 *Hashtable* 中的形状的克隆。

   ```java
   //PrototypePatternDemo.java
   public class PrototypePatternDemo {
      public static void main(String[] args) {
         ShapeCache.loadCache();
    
         Shape clonedShape = (Shape) ShapeCache.getShape("1");
         System.out.println("Shape : " + clonedShape.getType());        
    
         Shape clonedShape2 = (Shape) ShapeCache.getShape("2");
         System.out.println("Shape : " + clonedShape2.getType());        
    
         Shape clonedShape3 = (Shape) ShapeCache.getShape("3");
         System.out.println("Shape : " + clonedShape3.getType());        
      }
   }
   
   ```

5. 执行程序，输出结果：

   ```
   Shape : Circle
   Shape : Square
   Shape : Rectangle
   ```
