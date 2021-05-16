---
title: Java 数组
date: 2019-04-04 22:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### Java 数组
数组对于每一门编程语言来说都是重要的数据结构之一，当然不同语言对数组的实现及处理也不尽相同。
Java 语言中提供的数组是用来存储固定大小的同类型元素。

你可以声明一个数组变量，如 numbers[100] 来代替直接声明 100 个独立变量 number0，number1，....，number99。
<!--more-->

- 最小的下标是0，最大的下标是数组的元素个数-1；

- 编译器不会检查看你是否使用了有效的下标；
- 运行时出现了无效的下标，程序会终止进行

**声明数组变量**

首先必须声明数组变量，才能在程序中使用数组。下面是声明数组变量的语法：

```java
dataType[] arrayRefVar;   // 首选的方法
 或
dataType arrayRefVar[];  // 效果相同，但不是首选方法
```
注意: 建议使用 dataType[] arrayRefVar 的声明风格声明数组变量。 dataType arrayRefVar[] 风格是来自 C/C++ 语言 ，在Java中采用是为了让 C/C++ 程序员能够快速理解java语言。

**创建数组**

Java语言使用new操作符来创建数组，语法如下：
```
arrayRefVar = new dataType[arraySize];
```
上面的语法语句做了两件事：
1. 使用new dataType[arraySize] 创建了一个数组对象。
2. 把新创建的数组的引用赋值给变量 arrayRefVar。

数组变量的声明，和创建数组可以用一条语句完成，如下所示：
```
dataType[] arrayRefVar = new dataType[arraySize];
```
另外，你还可以使用如下的方式创建数组。
```
dataType[] arrayRefVar = {value0, value1, ..., valuek};
```
数组的元素是通过索引访问的。数组索引从 0 开始，所以索引值从 0 到 arrayRefVar.length-1。

示例：
```java

public class TestArray {
   public static void main(String[] args) {
      // 数组大小
      int size = 10;
      // 定义数组
      double[] myList = new double[size];
      myList[0] = 5.6;
      myList[1] = 4.5;
      myList[2] = 3.3;
      myList[3] = 13.2;
      myList[4] = 4.0;
      myList[5] = 34.33;
      myList[6] = 34.0;
      myList[7] = 45.45;
      myList[8] = 99.993;
      myList[9] = 11123;
      // 计算所有元素的总和
      double total = 0;
      for (int i = 0; i < size; i++) {
         total += myList[i];
      }
      System.out.println("总和为： " + total);
   }
}
```
运行结果：
```
$ java TestArray
总和为： 11367.373
```
这里 myList 数组里有 10 个 double 元素，它的下标从 0 到 9。


数组的元素类型和数组的大小都是确定的，所以当处理数组元素时候，我们通常使用基本循环或者 For-Each 循环。

示例：
```java

public class TestArray {
   public static void main(String[] args) {
      double[] myList = {1.9, 2.9, 3.4, 3.5};

      // 打印所有数组元素
      for (int i = 0; i < myList.length; i++) {
         System.out.println(myList[i] + " ");
      }
      // 计算所有元素的总和
      double total = 0;
      for (int i = 0; i < myList.length; i++) {
         total += myList[i];
      }
      System.out.println("Total is " + total);
      // 查找最大元素
      double max = myList[0];
      for (int i = 1; i < myList.length; i++) {
         if (myList[i] > max) max = myList[i];
      }
      System.out.println("Max is " + max);
   }
}
```
运行结果：
```
$ java TestArray
1.9
2.9
3.4
3.5
Total is 11.7
Max is 3.5
```

JDK 1.5 引进了一种新的循环类型，被称为 For-Each 循环或者加强型循环，它能在不使用下标的情况下遍历数组。但是不推荐在循环时插入或者删除数据。

```
for(type element: array)
{
    System.out.println(element);
}
```

示例：
```java
public class TestArray {
   public static void main(String[] args) {
      double[] myList = {1.9, 2.9, 3.4, 3.5};

      // 打印所有数组元素
      for (double element: myList) {
         System.out.println(element);
      }
   }
}
```
运行结果：
```
$ java TestArray
1.9
2.9
3.4
3.5
```

**多维数组**

多维数组可以看成是数组的数组，比如二维数组就是一个特殊的一维数组，其每一个元素都是一个一维数组，例如：
```
String str[][] = new String[3][4];
```
多维数组的动态初始化（以二维数组为例）

1. 直接为每一维分配空间，格式如下：
```
type[][] typeName = new type[typeLength1][typeLength2];
```
type 可以为基本数据类型和复合数据类型，arraylenght1 和 arraylenght2 必须为正整数，arraylenght1 为行数，arraylenght2 为列数。

2. 从最高维开始，分别为每一维分配空间，例如：
```
String s[][] = new String[2][];
s[0] = new String[2];
s[1] = new String[3];
s[0][0] = new String("Good");
s[0][1] = new String("Luck");
s[1][0] = new String("to");
s[1][1] = new String("you");
s[1][2] = new String("!");
```
解析：
s[0]=new String[2] 和 s[1]=new String[3] 是为最高维分配引用空间，也就是为最高维限制其能保存数据的最长的长度，然后再为其每个数组元素单独分配空间 s0=new String("Good") 等操作。

多维数组的引用（以二维数组为例）

对二维数组中的每个元素，引用方式为 `arrayName[index1][index2]`

#### For和For-Each
普通for循环在遍历集合时使用下标来定位集合中的元素。java在JDK1.5开始支持foreach循环，foreach在一定程度上简化了对集合的遍历。但某些情况下，foreach是不能完全代替for循环的。

限制场景：

1、foreach适用于数组或实现了iterator的集合类。foreach就是使用Iterator接口来实现对集合的遍历的。

2、在用foreach循环遍历一个集合时，不能改变集合中的元素，如增加元素、修改元素。否则会抛出ConcurrentModificationException异常。想了解原因的可以研究一下源码。也不能修改集合中的元素（不报异常），但可以修改元素的属性。


#### Arrays 类

java.util.Arrays 类能方便地操作数组，它提供的所有方法都是静态的。

具有以下功能：
```
数组的length属性可以获取数组长度
给数组赋值：通过 fill 方法。
对数组排序：通过 sort 方法,按升序。
比较数组：通过 equals 方法比较数组中元素值是否相等。
查找数组元素：通过 binarySearch(Object [] arr,Object key) 方法能对排序好的数组进行二分查找法操作。
复制数组：copyOf(arr,int  newlength)，从arr中复制newlength长度数据，如果大于arr长度，则int填充0，char填充null
复制数组：copyOfRange(arr,int fromIndex,int toIndex)。fromIndex从0到整个数组的长度，toIndex要复制的最后索引位置，可大于arr长度。新数组是不包括索引是toIndex的元素。
多维数组操作：deepToString（转化为字符串）、deepEquals（比较值）、deepHashCode（转化为hash code）
```
1. sort方法解析

   ```java
   /**
   * Arrays的Sort方法,对数组排序默认按升序,内部类必须实现Compare<T>接口，
   * 如果是基本数据类型只能按升序，类可以通过自定义的Compare<T>接口来改变排序方法
   源码
   * 打开Arrays.sort(）源码，还是以int型为例，其他类型也是大同小异
   * 从源码中发现，两种参数类型的sort方法都调用了 DualPivotQuicksort.sort（）方法
   * 结合文档以及源代码，我们发现，jdk中的Arrays.sort(）的实现是通过所谓的双轴快排的算法
   * Java1.8的快排是一种双轴快排，顾名思义：双轴快排是基于两个轴来进行比较，跟普通的选择一个点来作为轴点的快排是有很大的区别的，
   * 双轴排序利用了区间相邻的特性，对原本的快排进行了效率上的提高，
   * 很大程度上是利用了数学的一些特性。。。。。嗯。。。反正很高深的样子
   
   * 算法步骤
   *
   * 1.对于很小的数组（长度小于27），会使用插入排序。
   * 2.选择两个点P1,P2作为轴心，比如我们可以使用第一个元素和最后一个元素。
   * 3.P1必须比P2要小，否则将这两个元素交换，现在将整个数组分为四部分：
   * （1）第一部分：比P1小的元素。
   * （2）第二部分：比P1大但是比P2小的元素。
   * （3）第三部分：比P2大的元素。
   * （4）第四部分：尚未比较的部分。
   * 在开始比较前，除了轴点，其余元素几乎都在第四部分，直到比较完之后第四部分没有元素。
   * 4.从第四部分选出一个元素a[K]，与两个轴心比较，然后放到第一二三部分中的一个。
   * 5.移动L，K，G指向。
   * 6.重复 4 5 步，直到第四部分没有元素。
   * 7.将P1与第一部分的最后一个元素交换。将P2与第三部分的第一个元素交换。
   * 8.递归的将第一二三部分排序。
   */
       public static void testSort() {
           System.out.println("--------testSort--------");
           int[] arr = {3, 1, 3, 10, 5, 6};
           System.out.println("Before:" + Arrays.toString(arr));
           Arrays.sort(arr);
           System.out.println("After sort:" + Arrays.toString(arr));
           int[] arr01 = {3, 1, 3, 10, 5, 6};
           System.out.println("Before:" + Arrays.toString(arr01));
   
           Arrays.sort(arr01, 2, 5);
           System.out.println("from [2 to 5) Sort:" + Arrays.toString(arr01));
   
           System.out.println("\n减序排序后顺序"); //要实现减序排序，得通过包装类型数组，基本类型数组是不行滴
           Integer[] integers = new Integer[]{2, 324, 4, 4, 6, 1};
           System.out.println("Before:" + Arrays.toString(integers));
           Arrays.sort(integers, new Comparator<Integer>() {
               /*
                *此处与c++ 的比较函数构成不一致
                * c++ 返回bool型, 而Java返回的为int型
                * 当返回值 > 0 时
                * 进行交换，即排序(源码实现为两枢轴快速排序)
                */
               public int compare(Integer o1, Integer o2) {
                   return o2 - o1;
               }
   
               public boolean equals(Object obj) {
                   return false;
               }
           });
           System.out.println("After 减序排序:" + Arrays.toString(integers));
   
   
           System.out.println("--------testSort--------");
   
       }
   ```

2. fill方法

   **Arrays.fill()并不能提高赋值的效率**，在函数的内部也是用for循环的方式 实现的

   fill()函数源码：

   ```java
       public static void fill(Object[] a, Object val) {
           for (int i = 0, len = a.length; i < len; i++)
               a[i] = val;
       }
   ```

   由此可见fill()函数只能填充一维数组，如果这样用，肯定会失败的。

   ```java
           int[][] map=new int[4][5];
           Arrays.fill(map,-1);//失败
   ```

   但是可以换一种方法实现，二维数组其实就是一维数组的数组，即，它本身只是一个一维数组，但是数组中的每个变量也是一个一维数组。 
   所以既然它是一维数组，就可以用对应类型的变量来填充它，即用一个一维数组来填充它：

   ```java
       int[][] map=new int[4][5];
       int[] ten=new int[10];
       Arrays.fill(ten, -1);
       Arrays.fill(map,ten);  //成功
   ```
   这里值得注意的是，一旦用ten填充了map,那map声明时候的“5”将起不到任何作用,每个map[i]都将等于ten。

   虽然成功填充了二维数组，但是感觉好像把问题变得更复杂了，可能并不如直接用for循环实现简单。

   不过在下面这种情况下，还是很实用的：

   ```java
           int[][] map=new int[4][5];
           int[] ten={1,2,6,3,6,1,7};
           Arrays.fill(map,ten);  
   ```

   当ten中的数值不固定，也不一定有规律时，可以用Arrays.fill()来填充二维数组，使其每一行都是｛1，2，6，3，6，1，7｝

3. equals 方法

   “为了能比较数组是否完全相等，Arrays提供了经重载的equals()方法。当然，也是针对各种primitive以及Object的。两个数组要想完全想等，它们必须有相同数量的元素，而且数组的每个元素必须与另一个数组的相对应位置上的元素相等。

   -->元素的相等性，用equals()判断<-- ”

   ```java
   //重写S对象的equals方法
   boolean equals(Object obj) {
       if(!obj instanceof S) return false;
       S b = (S) obj;
       if(this.val == b.val)
         return true;
       return false;
     } 
   /**重写Arrays的equals方法*/
   	public boolean equals(int[] array1, int[] array2) {
   		boolean isNull = array1 == null || array2 == null;
   		if (isNull) {
   			return false;
   		} else {
   			boolean isLenEqual = array1.length == array2.length;
   			if (!isLenEqual) {
   				return false;
   			} else {
   				for (int i = 0; i < array1.length; i++) {
   					boolean isNumEqual = array1[i] == array2[i];
   					if (!isNumEqual) {
                           return false;
                       }
   				}
                   return true;
   			}
   		}
   	}
   ```

4. binarySearch方法

   binarySearch()方法提供了多种重载形式，用于满足各种类型数组的查找需要，binarySearch()有两种参数类型

   注：此法为二分搜索法，故查询前需要用sort()方法将数组排序，如果数组没有排序，则结果是不确定的，另外

   如果数组中含有多个指定值的元素，则无法保证找到的是哪一个。

   - binarySearch(object[ ], object key);

     如果key在数组中，则返回搜索值的索引；否则返回-1或者”-“(插入点)。插入点是索引键将要插入数组的那一点，即第一个大于该键的元素索引。

     - 搜索值不是数组元素，且在数组范围内，从1开始计数，得“ - 插入点索引值”；
     - 搜索值是数组元素，从0开始计数，得搜索值的索引值；
     - 搜索值不是数组元素，且大于数组内元素，索引值为 – (arr.length + 1);
     - 搜索值不是数组元素，且小于数组内元素，索引值为 – 1。

   - binarySearch(object[ ], int fromIndex, int endIndex, object key);

     如果要搜索的元素key在指定的范围内，则返回搜索键的索引；否则返回-1或者”-“(插入点)。

     - 该搜索键在范围内，但不是数组元素，由1开始计数，得“ - 插入点索引值”；
     - 该搜索键在范围内，且是数组元素，由0开始计数，得搜索值的索引值；
     - 该搜索键不在范围内，且小于范围（数组）内元素，返回–(fromIndex + 1)；
     - 该搜索键不在范围内，且大于范围（数组）内元素，返回 –(toIndex + 1)。

   参考：[Java之数组查询Arrays类的binarySearch()方法详解](https://blog.csdn.net/cxhply/article/details/49423501)

5. arrayOf方法

   Array.copyOf() 用于复制指定的数组内容以达到扩容的目的，该方法对不同的基本数据类型都有对应的重载方法。本文只介绍其中的`copyOf(U[] original, int newLength, Class<? extends T[]> newType)`方法，基本类型的重载方法比该方法少一个 Class 类型的入参而已，该方法源码如下：

   ```java
   /**
    * @Description 复制指定的数组, 如有必要用 null 截取或填充，以使副本具有指定的长度
    * 对于所有在原数组和副本中都有效的索引，这两个数组相同索引处将包含相同的值
    * 对于在副本中有效而在原数组无效的所有索引，副本将填充 null，当且仅当指定长度大于原数组的长度时，这些索引存在
    * 返回的数组属于 newType 类
    
    * @param original 要复制的数组
    * @param newLength 副本的长度
    * @param newType 副本的类
    * 
    * @return T 原数组的副本，截取或用 null 填充以获得指定的长度
    * @throws NegativeArraySizeException 如果 newLength 为负
    * @throws NullPointerException 如果 original 为 null
    * @throws ArrayStoreException 如果从 original 中复制的元素不属于存储在 newType 类数组中的运行时类型
   
    * @since 1.6
    */
   public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
       T[] copy = ((Object)newType == (Object)Object[].class)
           ? (T[]) new Object[newLength]
           : (T[]) Array.newInstance(newType.getComponentType(), newLength);
       System.arraycopy(original, 0, copy, 0,
                        Math.min(original.length, newLength));
       return copy;
   }
   
   ```

   从代码可知，数组拷贝时调用的是本地方法 System.arraycopy() ；

   Arrays.copyOf()方法**返回的数组是新的数组对象，原数组对象仍是原数组对象**，不变，该拷贝不会影响原来的数组。

   **System.arraycopy()** 源码如下：

   ```java
   public static native void arraycopy(Object src,  int  srcPos,
                                           Object dest, int destPos,
                                           int length);
   //arraycopy()方法的主要缺点是必须事先创建好目的数组。
   ```

   参数说明：

   - src：源对象
   - srcPos：源数组中的起始位置，0>=srcPos&&(srcPos+length)<=src.length
   - dest：目标数组对象
   - destPos：目标数据中的起始位置，0>=destPos&&(destPos+length)<=dest.length
   - length：要拷贝的数组元素的数量

   ```java
   import java.util.Arrays;//注意这里
   public class Example5_20_3 {
   	public static void main(String[] args) {
   		        int[] a = {1, 4, 6, 45, 3, 2, 56, 76, 5, 4, 3, 4, 5, 6};
           int[] b = new int[8];//注意数组的长度
           int[] c=new int[20];
           System.out.println("数组a：" + Arrays.toString(a));
           System.arraycopy(a, 0, b, 0, 8);
           System.out.println("数组b：" + Arrays.toString(b));
           System.arraycopy(a,10,c,10,4);
           System.out.println("数组c：" + Arrays.toString(c));
   }
   }
   
   ```

#### 动态数组

ArrayList是一个容量可以动态增长的动态数组。它继承了AbstractList。同时ArrayList不是线程安全的，一般在单线程中才使用。

**遍历方式**

```(java)
//1、迭代器遍历：
Iterator<Integer> it = myArrayList.iterator();
while(it.hasNext()){
    System.out.print(it.next() + " ");
}
//2、索引值遍历：
for(int i = 0; i < myArrayList.size(); i++){
   System.out.print(myArrayList.get(i) + " ");
}
//3、foreach遍历：
for(Integer number : myArrayList){
   System.out.print(number + " ");
}
```

**基本操作**
```
添加元素到尾部：add();
添加元素到指定位置：add(index,value);
删除指定位置上面的元素并返回该位置上的元素：remove(index);
返回元素个数：size();
返回指定位置上的元素：get(index);
按适当顺序（从第一个到最后一个元素）返回包含此列表中所有元素的数组:toArray();
替换指定位置上的元素：set(index,value);
其他：contains()等等。
```

**注意**

1. ArrayList//此处要声明整数类型需要使用int的封装类Integer。
2. List myArrayList = new ArrayList();
如果像这样使用默认的构造方法，初始容量大小10，超出之后会重新分配内存。（如果直接指定大小则不会重新分配，除非超出现有范围）
JDK1.6 int newCapacity = (oldCapacity * 3)/2 + 1;//1.5倍+1
JDK1.8 int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍
（有没有什么方法可以查看Java中内存分配情况，或者能够返回ArrayList实际占用内存大小的函数也行。使用jvisualvm.exe只能看见性能，不能看见具体变量情况）
3. 如果想ArrayList中添加大量元素，可使用ensureCapacity方法一次性增加capacity，可以减少增加重分配的次数提高性能。
4. ArrayList是可以存放null值的。

#### 需要注意的问题

Java中的数组中既可以存储基本的值类型，也可以存储对象。对象数组和原始数据类型数组在使用方法上几乎是完全一致的，唯一的差别在于对象数组容纳的是引用而原始数据类型数组容纳的是具体的数值。这一点要特别注意，在讨论关于数组的问题时，一定要先确定数组中存储的是基本值类型还是对象。特别是在调试程序时，要注意这方面。

例如：
Arrays提供了一个fill()方法将一个值复制到一个位置，如果是对象数组则将引用复制到每一个位置。

Java 标准库提供了一个静态方法名为System.arraycopy() 专门用于数组的复制它复制数组的速度比自己亲自动手写一个for 循环来复制快得多System.arraycopy()已进行了重载可对所有类型进行控制。无论原始数据类型数组还是对象数组我们都可对它们进行复制。但是假如复制的对象数组，那么真正复制的只是引用对象本身可不会复制。

**为什么使用数组而不使用ArrayList等容器类？**

效率：

对于Java 来说要想保存和随机访问一系列对象实际是对象引用效率最高的方法莫过于数组。

类型：

Java标准库中的容器类都把对象当作没有具体类型那样对待，换言之它们将其当作Object 类型处理。Object 类型是Java 中所有类的根类，从某种角度看这种处理方法是非常合理的，我们只需构建一个容器然后所有Java 对象都可进入那个容器。原始数据类型除外，可用Java 的基类型封装器类将其作为常数置入容器或自建一个类把它们封装到里面当作可变值进行对待。这再一次体现出数组相较于普通容器的优越性，创建一个数组时可让它容纳一种特定的类型。这意味着可进行编译时间的类型检查防范自己设置了错误的类型或者错误地提取了一种类型，而不是运行时的Exception。

总结：在你想容纳一组对象的时候第一个也是最有效的一个选择便是数组。

#### 面试题

1. 声明数组`int[ ]x,y[ ];`下列不能编译通过的是：（int[ ]x,y[ ]可以写成“int [ ]x”和“int [ ]y[ ]”）

   ```
   x[0]=y;//不能编译通过，因为y表示为一个“一维数组”，而x[0]为一个整形的变量值，类型不匹配
   y[0]=x;//能编译通过，因为x和y[0]都表示为一个一维数组
   y[0][0]=x;//不能编译通过，y[0][0]表示为值，x表示为一维数组
   x[0][0]=y;//不能编译通过，表示错误
   y[0][0]=x[0];//能编译通过，因为双方都表示为值
   ```

2.  在java中，声明一个数组过程中，是如何分配内存的？

   - 当声明数组类型变量时，为其分配了（32位）引用空间，由于未赋值，因此并不指向任何对象；
   - 当创建了一个数组对象（也就是new出来的）并将其地址赋值给了变量，其中创建出来的那几个数组元素相当于引用类型变量，因此各自占用（32位的）引用空间并按其默认初始化规则被赋值为null
   - 程序继续运行，当创建新的对象并（将其地址）赋值给各数组元素，此时堆内存就会有值了

3.  Java变量一定要初始化吗？

   不一定。Java数组变量是引用数据类型变量，它并不是数组对象本身，只要让数组变量指向有效的数组对象，即可使用该数组变量。对数组执行初始化，并不是对数组变量进行初始化，而是对数组对象进行初始化——也就是为该数组对象分配一块连续的内存空间，这块连续的内存空间就是数组的长度。

4.  基本类型变量都放在栈内存中？

    错。应该这样说：所有局部变量都放在栈内存里保存的，不管其是基本类型的变量，还是引用类型变量，都是存储在各自的方法栈区中；但是引用类型变量所引用的对象（包括数组、普通java对象）则总是存储在堆内存中。

5.  引用变量何时只是栈内存中的变量本身，何时又变为引用实例的java对象？

    引用变量本质上只是一个指针，只要程序通过引用变量访问属性，或者通过引用变量来调用方法，该引用变量将会由他所引用的对象代替。

6. 数组是如何随机访问数组元素

   数组是如何实现根据下标随机访问数组元素的吗？

   例如： int[]a=newint[10]

   - 计算机给数组a[10]，分配了一组连续的内存空间。
   - 比如内存块的首地址为 base_address=1000。
   - 当计算给每个内存单元分配一个地址，计算机通过地址来访问数据。

   当计算机需要访问数组的某个元素的时候，会通过一个寻址公式来计算存储的内存地址。

   ```
   a[i]_address = base_address + i * data_type_size
   //arr[i] 首地址 = 数组内存块首地址 + 数据类型大小 * i ，其中i为偏移量。
   //baseaddress：内存块的首地址。datatype_size：数组中每个元素的大小，比如每个元素大小是4个字节。
   ```

   数组使用二分法查找元素，时间复杂度是O(logn)。

   根据下标随机访问的时间复杂度是O(1)。

7. 数组如何提高效率？

   将多次删除操作中集中在一起执行，可以先记录已经删除的数据，但是不进行数据迁移，而仅仅是记录，当发现没有更多空间存储时，再执行真正的删除操作，这样减少数据搬移次数节省耗时。

   这也是跟 JVM 标记清除垃圾回收算法的核心思想相似。

   - 在垃圾收集时此算法分为“标记”、“清除”两个阶段，先标记出需要回收的对象，再统一清除标记的对象。清除之后会产生大量不连续的内存碎片。

   - 在标记完成之后让所有存活的对象都向一端移动，然后直接清理掉边界以外的内存。

8. 为什么数组要从 0 开始编号，而不是1

   从偏移角度理解a[0] 0为偏移量，如果从1计数，会多出K-1,增加cpu负担。

   