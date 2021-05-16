---
title: Java 流程控制
date: 2019-04-04 10:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 循环结构

顺序结构的程序语句只能被执行一次。如果您想要同样的操作执行多次,，就需要使用循环结构。
Java中有三种主要的循环结构：

```
while 循环
do…while 循环
for 循环
```
<!--more-->
在Java5中引入了一种主要用于数组的增强型for循环。

#### while 循环

while是最基本的循环，它的结构为：
```java
while( 布尔表达式 ) {
  //循环内容
}
```
只要布尔表达式为 true，循环就会一直执行下去。

示例：
```java
public class Test {
   public static void main(String args[]) {
      int x = 10;
      while( x < 20 ) {
         System.out.print("value of x : " + x );
         x++;
         System.out.print("\n");
      }
   }
}
```
运行结果：
```
$ java  Test
value of x : 10
value of x : 11
value of x : 12
value of x : 13
value of x : 14
value of x : 15
value of x : 16
value of x : 17
value of x : 18
value of x : 19
```
#### do…while 循环
对于 while 语句而言，如果不满足条件，则不能进入循环。但有时候我们需要即使不满足条件，也至少执行一次。

do…while 循环和 while 循环相似，不同的是，do…while 循环至少会执行一次。
```
do {
       //代码语句
}while(布尔表达式);
```
注意：布尔表达式在循环体的后面，所以语句块在检测布尔表达式之前已经执行了。 如果布尔表达式的值为 true，则语句块一直执行，直到布尔表达式的值为 false。

示例：
```java
public class Test {
   public static void main(String args[]){
      int x = 10;

      do{
         System.out.print("value of x : " + x );
         x++;
         System.out.print("\n");
      }while( x < 20 );
   }
}
```
运行结果：
```
$ java  Test
value of x : 10
value of x : 11
value of x : 12
value of x : 13
value of x : 14
value of x : 15
value of x : 16
value of x : 17
value of x : 18
value of x : 19
```
#### for循环
虽然所有循环结构都可以用 while 或者 do...while表示，但 Java 提供了另一种语句 —— for 循环，使一些循环结构变得更加简单。

for循环执行的次数是在执行前就确定的。语法格式如下：
```
for(初始化; 布尔表达式; 更新) {
    //代码语句
}
```
关于 for 循环有以下几点说明：
- 最先执行初始化步骤。可以声明一种类型，但可初始化一个或多个循环控制变量，也可以是空语句。
- 然后，检测布尔表达式的值。如果为 true，循环体被执行。如果为false，循环终止，开始执行循环体后面的语句。
- 执行一次循环后，更新循环控制变量。
- 再次检测布尔表达式。循环执行上面的过程。

示例：
```java
public class Test {
   public static void main(String args[]) {

      for(int x = 10; x < 20; x = x+1) {
         System.out.print("value of x : " + x );
         System.out.print("\n");
      }
   }
}
```
运行结果：
```
$ java  Test
value of x : 10
value of x : 11
value of x : 12
value of x : 13
value of x : 14
value of x : 15
value of x : 16
value of x : 17
value of x : 18
value of x : 19
```
#### Java 增强 for 循环

Java5 引入了一种主要用于数组的增强型 for 循环。

Java 增强 for 循环语法格式如下:
```
for(声明语句 : 表达式)
{
   //代码句子
}
```
声明语句：声明新的局部变量，该变量的类型必须和数组元素的类型匹配。其作用域限定在循环语句块，其值与此时数组元素的值相等。

表达式：表达式是要访问的数组名，或者是返回值为数组的方法。

示例：
```java
public class Test {
   public static void main(String args[]){
      int [] numbers = {10, 20, 30, 40, 50};
      for(int x : numbers ){
         System.out.print( x );
         System.out.print(",");
      }
      System.out.print("\n");
      String [] names ={"James", "Larry", "Tom", "Lacy"};
      for( String name : names ) {
         System.out.print( name );
         System.out.print(",");
      }
      System.out.println();
   }
}
```
运行结果：
```
$ java  Test
10,20,30,40,50,
James,Larry,Tom,Lacy,
```
#### break 关键字

break 主要用在循环语句或者 switch 语句中，用来跳出整个语句块。

break 跳出最里层的循环，并且继续执行该循环下面的语句。

break 的用法很简单，就是循环结构中的一条语句：
```
break;
```
示例：
```java
public class Test {
   public static void main(String args[]) {
      int [] numbers = {10, 20, 30, 40, 50};

      for(int x : numbers ) {
         // x 等于 30 时跳出循环
         if( x == 30 ) {
            break;
         }
         System.out.print( x );
         System.out.print("\n");
      }
   }
}
```
运行结果：
```
$ java  Test
10
20
```

#### continue 关键字

continue 适用于任何循环控制结构中。作用是让程序立刻跳转到下一次循环的迭代。

在 for 循环中，continue 语句使程序立即跳转到更新语句。

在 while 或者 do…while 循环中，程序立即跳转到布尔表达式的判断语句。

continue 就是循环体中一条简单的语句：
```
continue;
```
示例：
```java
public class Test {
   public static void main(String args[]) {
      int [] numbers = {10, 20, 30, 40, 50};

      for(int x : numbers ) {
         if( x == 30 ) {
        continue;
         }
         System.out.print( x );
         System.out.print("\n");
      }
   }
}
```
运行结果：
```
$ java  Test
10
20
40
50
```

### 条件语句

常用语法：
```java
//1
if(布尔表达式)
{
   //如果布尔表达式为true将执行的语句
}

//2
if(布尔表达式){
   //如果布尔表达式的值为true
}else{
   //如果布尔表达式的值为false
}

//3
if(布尔表达式 1){
   //如果布尔表达式 1的值为true执行代码
}else if(布尔表达式 2){
   //如果布尔表达式 2的值为true执行代码
}else if(布尔表达式 3){
   //如果布尔表达式 3的值为true执行代码
}else {
   //如果以上布尔表达式都不为true执行代码
}

//4


if(布尔表达式 1){
   ////如果布尔表达式 1的值为true执行代码
   if(布尔表达式 2){
      ////如果布尔表达式 2的值为true执行代码
   }
}


```

示例：
```java
public class Test {
  public static void main(String args[]){
     int x = 30;

     if( x == 10 ){
        System.out.println("Value of X is 10");
     }else if( x == 20 ){
        System.out.println("Value of X is 20");
     }else if( x == 30 ){
        System.out.println("Value of X is 30");
     }else{
        System.out.println("这是 else 语句");
     }

     int x1 = 30;
	int y1 = 10;

if( x1 == 30 ){
   if( y1 == 10 ){
       System.out.println("X1 = 30 and Y1 = 10");
    }
 }
  }
}
```

运行结果：
```
$ java  Test
Value of X is 30
X1 = 30 and Y1 = 10
```

#### Java switch case 语句

switch case 语句判断一个变量与一系列值中某个值是否相等，每个值称为一个分支。
```java
switch(expression){
    case value :
       //语句
       break; //可选
    case value :
       //语句
       break; //可选
    //你可以有任意数量的case语句
    default : //可选
       //语句
}
```
switch case 语句有如下规则：
- switch 语句中的变量类型可以是： byte、short、int 或者 char。从 Java SE 7 开始，switch 支持字符串 String 类型了，同时 case 标签必须为字符串常量或字面量。

- switch 语句可以拥有多个 case 语句。每个 case 后面跟一个要比较的值和冒号。

- case 语句中的值的数据类型必须与变量的数据类型相同，而且只能是常量或者字面常量。

- 当变量的值与 case 语句的值相等时，那么 case 语句之后的语句开始执行，直到 break 语句出现才会跳出 switch 语句。

- 当遇到 break 语句时，switch 语句终止。程序跳转到 switch 语句后面的语句执行。case 语句不必须要包含 break 语句。如果没有 break 语句出现，程序会继续执行下一条 case 语句，直到出现 break 语句。

- switch 语句可以包含一个 default 分支，该分支一般是 switch 语句的最后一个分支（可以在任何位置，但建议在最后一个）。default 在没有 case 语句的值和变量值相等的时候执行。default 分支不需要 break 语句。

switch case 执行时，一定会先进行匹配，匹配成功返回当前 case 的值，再根据是否有 break，判断是否继续输出，或是跳出判断。


示例：
```java
public class Test {
   public static void main(String args[]){
      //char grade = args[0].charAt(0);
      char grade = 'C';

      switch(grade)
      {
         case 'A' :
            System.out.println("优秀");
            break;
         case 'B' :
         case 'C' :
            System.out.println("良好");
            break;
         case 'D' :
            System.out.println("及格");
            break;
         case 'F' :
            System.out.println("你需要再努力努力");
            break;
         default :
            System.out.println("未知等级");
      }
      System.out.println("你的等级是 " + grade);
   }
}
```

运行结果：
```
$ java  Test
良好
你的等级是 C
```

### 方法

在我们的日常生活中，方法可以理解为要做某件事情，而采取的解决办法。

如：小明同学在路边准备坐车来学校学习。这就面临着一件事情（坐车到学校这件事情）需要解决，解决办法呢？可采用坐公交车或坐出租车的方式来学校，那么，这种解决某件事情的办法，我们就称为方法。

在java中，方法就是用来完成解决某件事情或实现某个功能的办法。

方法实现的过程中，会包含很多条语句用于完成某些有意义的功能——通常是处理文本，控制输入或计算数值。

我们可以通过在程序代码中引用方法名称和所需的参数，实现在该程序中执行（或称调用）该方法。方法，一般都有一个返回值，用来作为事情的处理结果。

#### 格式

在Java中，声明一个方法的具体语法格式如下：

```
修饰符 返回值类型 方法名(参数类型 参数名1,参数类型 参数名2,．．．．．．){
执行语句
………
return 返回值;
}
```

对于上面的语法格式中具体说明如下：

- 修饰符：方法的修饰符比较多，有对访问权限进行限定的，有静态修饰符static，还有最终修饰符final等，这些修饰符在后面的学习过程中会逐步介绍

- 返回值类型：用于限定方法返回值的数据类型

- 参数类型：用于限定调用方法时传入参数的数据类型

- 参数名：是一个变量，用于接收调用方法时传入的数据

- return关键字：用于结束方法以及返回方法指定类型的值

- 返回值：被return语句返回的值，该值会返回给调用者

需要特别注意的是，方法中的“参数类型 参数名1，参数类型 参数名2”被称作参数列表，它用于描述方法在被调用时需要接收的参数，如果方法不需要接收任何参数，则参数列表为空，即()内不写任何内容。方法的返回值必须为方法声明的返回值类型，如果方法中没有返回值，返回值类型要声明为void，此时，方法中return语句可以省略。

接下来通过一个案例来演示方法的定义与使用。MethodDemo01.java

```java
public class MethodDemo01 {
public static void main(String[] args) {
int area = getArea(3, 5); // 调用 getArea方法
System.out.println(" The area is " + area);
}
// 下面定义了一个求矩形面积的方法，接收两个参数，其中x为高，y为宽
public static int getArea(int x, int y) {
int temp = x * y; // 使用变量temp记住运算结果
return temp; // 将变量temp的值返回
}
}
```

在上述代码中，定义了一个getArea()方法用于求矩形的面积，参数x和y分别用于接收调用方法时传入的高和宽，return语句用于返回计算所得的面积。在main()方法中通过调用getArea()方法，获得矩形的面积，并将结果打印。

#### 方法调用图解

接下来通过一个图例演示getArea()方法的整个调用过程，如下图所示。

![image.png](https://i.loli.net/2019/08/29/YqQPnENjp7US2dI.png)

从上图中可以看出，在程序运行期间，参数x和y相当于在内存中定义的两个变量。当调用getArea()方法时，传入的参数3和5分别赋值给变量x和y，并将x*y的结果通过return语句返回，整个方法的调用过程结束，变量x和y被释放。

#### 方法定义练习

分别定义如下方法：

1. 定义无返回值无参数方法，如打印3行，每行3个*号的矩形
2. 定义有返回值无参数方法，如键盘录入得到一个整数
3. 定义无返回值有参数方法，如打印指定M行，每行N个*号的矩形
4. 定义有返回值有参数方法，如求三个数的平均值

```java
//l 无返回值无参数方法，如打印3行，每行3个*号的矩形
public static void printRect(){
//打印3行星
for (int i=0; i<3; i++) {
//System.out.println("***"); 相当于是打印3颗星，换行
//每行打印3颗星
for (int j=0; j<3; j++) {
System.out.print("*");  // ***
}
System.out.println();
}
}

//l 有返回值无参数方法，如键盘录入得到一个整数
public static int getNumber(){
Scanner sc = new Scanner(System.in);
int number = sc.nextInt();
return number;
}

//l 无返回值有参数方法，如打印指定M行，每行N个*号的矩形
public static void printRect2(int m, int n){
//打印M行星
for (int i=0; i<m; i++) {
//每行中打印N颗星
for (int j=0; j<n; j++) {
System.out.print("*");  
}
System.out.println();
}
}
//l 有返回值有参数方法，如求三个数的平均值
public static double getAvg(double a, double b, double c) {
double result = (a+b+c)/3;
return result;
}
```

#### 方法的重载

我们假设要在程序中实现一个对数字求和的方法，由于参与求和数字的个数和类型都不确定，因此要针对不同的情况去设计不同的方法。接下来通过一个案例来实现对两个整数相加、对三个整数相加以及对两个小数相加的功能，具体实现如下所示。MethodDemo02.java

```java
public class MethodDemo02 {
public static void main(String[] args) {
// 下面是针对求和方法的调用
int sum1 = add01(1, 2);
int sum2 = add02(1, 2, 3);
double sum3 = add03(1.2, 2.3);

// 下面的代码是打印求和的结果
System.out.println("sum1=" + sum1);
System.out.println("sum2=" + sum2);
System.out.println("sum3=" + sum3);
}

// 下面的方法实现了两个整数相加
public static int add01(int x, int y) {
return x + y;
}
// 下面的方法实现了三个整数相加
public static int add02(int x, int y, int z) {
return x + y + z;
}
// 下面的方法实现了两个小数相加
public static double add03(double x, double y) {
return x + y;
}
}
```

从上述代码不难看出，程序需要针对每一种求和的情况都定义一个方法，如果每个方法的名称都不相同，在调用时就很难分清哪种情况该调用哪个方法。

为了解决这个问题，Java允许在一个类中定义多个名称相同的方法，但是参数的类型或个数必须不同，这就是方法的重载。

下面的三个方法互为重载关系

```
public static int add(int x,int y) {逻辑} //两个整数加法
public static int add(int x,int y,int z) {逻辑} //三个整数加法
public static int add(double x,double y) {逻辑} //两个小数加法
```

接下来通过方法重载的方式进行修改，如下所示。MethodDemo03.java

```java
public class MethodDemo03 {
public static void main(String[] args) {
// 下面是针对求和方法的调用
int sum1 = add(1, 2);
int sum2 = add(1, 2, 3);
double sum3 = add(1.2, 2.3);
// 下面的代码是打印求和的结果
System.out.println("sum1=" + sum1);
System.out.println("sum2=" + sum2);
System.out.println("sum3=" + sum3);
}
// 下面的方法实现了两个整数相加
public static int add(int x, int y) {
return x + y;
}
// 下面的方法实现了三个整数相加
public static int add(int x, int y, int z) {
return x + y + z;
}
// 下面的方法实现了两个小数相加
public static double add(double x, double y) {
return x + y;
}
}
```

上述代码中定义了三个同名的add()方法，它们的参数个数或类型不同，从而形成了方法的重载。

在main()方法中调用add()方法时，**通过传入不同的参数便可以确定调用哪个重载的方法**，如add(1,2)调用的是两个整数求和的方法。值得注意的是，**方法的重载与返回值类型无关**，它只有两个条件，**一是方法名相同，二是参数个数或参数类型不相同**。

**重载的注意事项**

- 重载方法参数必须不同：

  参数个数不同，如method(int x)与method(int x,int y)不同

  参数类型不同，如method(int x)与method(double x)不同g

  参数顺序不同，如method(int x,double y)与method(double x,int y)不同

- 重载只与方法名与参数类型相关与返回值无关

  如void method(int x)与int method(int y)不是方法重载，不能同时存在

- 重载与具体的变量标识符无关

  如method(int x)与method(int y)不是方法重载，不能同时存在

**参数传递**

参数传递，可以理解当我们要调用一个方法时，我们会把指定的数值，传递给方法中的参数，这样方法中的参数就拥有了这个指定的值，可以使用该值，在方法中运算了。这种传递方式，我们称为参数传递。

- 在这里，定义方法时，参数列表中的变量，我们称为形式参数

- 调用方法时，传入给方法的数值，我们称为实际参数

我们看下面的两段代码，来明确下参数传递的过程：

```java
public class ArgumentsDemo01 {
public static void main(String[] args) {
int a=5;
int b=10;
change(a, b);//调用方法时，传入的数值称为实际参数
System.out.println("a=" + a);
System.out.println("b=" + b);
}
public static void change(int a, int b){//方法中指定的多个参数称为形式参数
a=200;
b=500;
}
}
```

再看另一段代码

```java
public class ArgumentsDemo02 {
public static void main(String[] args) {
int[] arr = { 1, 2, 3 };
change(arr);// 调用方法时，传入的数值称为实际参数
for (int i = 0; i < arr.length; i++) {
System.out.println(arr[i]);
}
}
public static void change(int[] arr) {// 方法中指定的多个参数称为形式参数
for (int i = 0; i < arr.length; i++) {
arr[i] *= 2;
}
}
}
```

**参数传递图解与结论**

![image.png](https://i.loli.net/2019/08/29/arJiqPU7fzL3sl9.png)

通过上面的两段程序可以得出如下结论：

- 当调用方法时，如果传入的数值为基本数据类型（包含String类型），形式参数的改变对实际参数不影响

- 当调用方法时，如果传入的数值为引用数据类型（String类型除外），形式参数的改变对实际参数有影响

#### Java是值传递还是引用传递

观点：Java没有引用传递，只有值传递

**基本概念**

- 实参：实际参数，是提前准备好并赋值完成的变量。分配到栈上。如果是基本类型直接分配到栈上，如果是引用类型，栈上分配引用空间存储指向堆上分配的对象本身的指针。String等基本类型的封装类型比较特殊，后续讨论。
- 形参：形式参数，方法调用时在栈上分配的实参的拷贝。
- 值传递：方法调用时，实际参数把它的值传递给对应的形式参数，形参接收的是原始值的一个拷贝，此时内存中存在两个相等的变量
- 引用传递：方法调用时将实参的地址传递给对应的形参，实参和形参指向相同的内容

Java数据有两个类型:

- 基本类型
- 引用类型

**基本类型**

基本类型传递时，线程在栈上分配形式参数并拷贝实际参数的值。

```java
public class Test {
    public static void main(String[] args) {
        int value = 100;
        change(value);
        System.out.println("outer: " +  value);
    }
 
    static void change(int value) {
        value = 200;
        System.out.println("inner: " +  value);
    }
 
}
//结果输出：
//inner: 200
//outer: 100
```

方法修改的只是形式参数，对实际参数没有作用。方法调用结束后形式参数随着栈帧回收。

**引用类型**

引用类型传递时，传递的是引用的值，从这个角度来讲还是**值传递**。如果是引用传递的话，**传递的应该是引用的地址，而不是引用的值**。

```java
public class Test {
    public static void main(String[] args) {
        Person person = new Person();
        person.setAge(17);
        person.setName("Tom");
        change(person);
 
        System.out.println("outer: " +  person.getAge());
 
    }
 
    static void change(Person value) {
        value.setAge(18);
        System.out.println("inner: " +  value.getAge());
    }
 
    static class Person {
        private String name;
        private Integer age;
 
        public String getName() {
            return name;
        }
 
        public void setName(String name) {
            this.name = name;
        }
 
        public Integer getAge() {
            return age;
        }
 
        public void setAge(Integer age) {
            this.age = age;
        }
    }
 
}
//结果输出：
//inner: 18
//outer: 18
```

方法修改的是引用所指向的数据空间的数据，所以方法外部也能看到修改的结果。

基本类型的数组也是对象，所以int[] 传递的也是对象应用的值。

```java
public class Test {
    public static void main(String[] args) {
        int[] intArray = new int[10];
        change(intArray);
 
        System.out.println("outer: " +  intArray[0]);
 
    }
    static void change(int[] value) {
        value[0] = 200;
        System.out.println("inner: " +  value[0]);
    }
}
//运行结果：
//inner: 200
//outer: 200


```

如果对引用类型的传递稍作修改

```java
    static void change(Person value) {
        value = new Person();
        value.setAge(18);
        System.out.println("inner: " +  value.getAge());
    }
//运行结果：
//inner: 18
//outer: 17
```

同理String，Integer等类型的封装类型为final类型，对数据的修改操作实际上是创建了一个新的对象

```java
public class Test {
    public static void main(String[] args) {
        String value = "123";
        change(value);
 
        System.out.println("outer: " + value);
 
    }
 
    static void change(String value) {
        value = "abc";
        System.out.println("inner: " +  value);
    }
}
//运行结果：
//inner: abc
//outer: 123
```

### 面试题

1. switch 是否能作用在byte 上，是否能作用在long 上，是否能作用在String上

   答：在Java 5以前，switch(expr)中，expr只能是byte、short、char、int。从Java 5开始，Java中引入了枚举类型，expr也可以是enum类型，从Java 7开始，expr还可以是字符串（String），但是长整型（long）在目前所有的版本中都是不可以的。

2. 在JAVA中如何跳出当前的多重嵌套循环

   在java中，要想跳出多重循环，可以在外面的循环语句前定义一个标号，然后在里层循环体的代码中使用带有标号的的break语句，即可跳出外层循环

   ```java
   ok:
   for(int i=0;i<10;i++)
   {
        for(int j=0;j<10;j++)
   {
        system.out.println("i="+i+",j="+j);
        if(j==5)break ok;
   }
   }
   ```

   另外，我个人觉得通常不使用标号这种方式，而是让外层的循环条件表达式的结果可以收到里层循环体代码的控制，例如，要在二维数组中查找到某个数字

   ```java
   int arr[][]={{1,2,2},{2,2,5},{4,4}};
   boolean found =false;
   for(int i=0;i<arr.length&&!found;i++)
   {
       for(int j=0;j<arr[i].length;j++)
   {
       system.out.println("i="+i+",j="+j);
       if(arr[i][j]==5)
   {     
       found=true;
       break;
   }
   }
   }
   
   ```

3. Exit（）函数 和return 区别

   - return返回函数值，是关键字；  exit 是一个函数。
   - return是语言级别的，它表示了调用堆栈的返回；而exit是系统调用级别的，它表示结束一个进程。

   - return是函数的退出(返回)；exit是进程的退出。

   - return是C语言提供的，exit是操作系统提供的（或者函数库中给出的）。

   - return用于结束一个函数的执行，将函数的执行信息传出个其他调用函数使用；exit函数是退出应用程序，删除进程使用的内存空间，并将应用程序的一个状态返回给OS，这个状态标识了应用程序的一些运行信息，这个信息和机器和操作系统有关，一般是 0 为正常退出， 非0 为非正常退出。

   - 非主函数中调用return和exit效果很明显，但是在main函数中调用return和exit的现象就很模糊，多数情况下现象都是一致的

