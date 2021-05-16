---
title: Java 字符串(一)
date: 2019-04-04 20:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 字符串

在java中所有以双引号包括的零个或多个字符都是字符串，可以认为是String类，例如：
```
"Hello"
""
"12315 \n"
```
单引号包括的就是字符，只能有一个字符，可以认为时Character类。例如：
```
'1'
'H'
''
```

<!--more-->

> 注:Java中一切都可以被当做对象

首先我们要明确，String并不是基本数据类型，而是一个对象，并且是不可变的对象。查看源码就会发现String类为final型的（当然也不可被继承），而且通过查看JDK文档会发现几乎每一个修改String对象的操作，实际上都是创建了一个全新的String对象。

**字符串的声明：**

```
String str；
```
默认是等于null，必须经过初始化才能使用。

**字符串创建：**
```
String str1="hello 123"; //直接声明，在堆中创建字符串常量"hello 123"，str1是该字符串的引用，
char c[]={'h','e','l','l','o'};
String str2=new String(c); //使用字符数组填充值，和下面一个等价
String str3=new String("hello");//创建对象

int offset=1;//偏移量
int length=2;//长度
String str4=new String(c,offset,length); //el
```


**字符串连接**

使用'+'运算符就可以实现链接多个字符串的功能，连接后创建一个新的String对象。
```
String str1 =new String("hello");
String str2=new String("123");

String s=str1+" "+str2;

```

> 一句相连的字符串不能分开在两行中

```
String str1 =new String("hello
        123 ");//错误

//推荐,使用这个控制代码行长度
String str1 =new String("hello"+
        "123 ");

```

上面显示的是拼接字符串，其实'+'可以拼接字符串和其他类型、其他类型和字符串、复杂类型等。

```
String s1="hello"+123;//字符串拼接数字，值"hello123"
String s2="hello"+123f;//字符串拼接float数字,值"hello123.0"
String s3="hello"+true;//字符串拼接布尔,值"hellotrue"

String s4=1+2+3+"hello";//数字拼接字符串，前面'+'是加法运算符，后面是字符串拼接符号,值"6hello"

String s5=1+2+"hello"+3+6;//复杂类型，前面'+'是加法运算符，后面是字符串拼接符号,值"3hello36"

String s6=1+3+"hello"+(3+6) //使用小括号，值"4hello9"。

```

### String类

字符串广泛应用 在Java 编程中，在 Java 中字符串属于对象，Java 提供了 String 类来创建和操作字符串。


#### 常用方法

 1. 构造方法：

 ```
* public String():创建String对象
*	public String(byte[] bytes):把字节数组转成字符串。
*	public String(byte[] bytes,int index,int length):把字节数组中的一部分转成字符串
*	public String(char[] value):把字符数组转成字符串
*	public String(char[] value,int index,int count):把字符数组的一部分转成字符串
* public String(String original):把字符串转成字符串
 ```
 注意问题：
* 输出语句输出任何对象名称的时候，默认调用的是该对象的toString()方法。

  而toString()方法默认输出的是包名...类名@哈希值的十六进制。

  如果，你用输出语句输出一个对象名称的时候，发现不是这个格式，说明了该类或者该类的父类重写了toString()方法。

- 返回此字符串的长度

```
public int length()
```

2. ==：比较的是引用类型，比较的是地址值即：
```
System.out.println(s1 == s2); // false
equal():默认比较的是地址值。String类重写了equals()方法，该方法的作用是比较字符串的内容是否相同
System.out.println(s1.equals(s2)); // true
```

3. 字符串变量相加：先开空间，再加内容；字符串常量相加：先加，再找，没有再开空间
```
String s1 = "hello";
String s2 = "world";
String s3 = "helloworld";
String s4 = s1 + s2;
String s5 = "hello"+"world";
// 即s4和s5的区别。
```

4. String类的判断功能：
```
boolean equals(Object obj):比较字符串的内容是否相同，严格区分大小写
boolean equalsIgnoreCase(String str):比较字符串的内容是否相同，不考虑大小写
boolean contains(String str):判断是否包含指定的小串
boolean startsWith(String str):判断是否以指定的字符串开头
boolean endsWith(String str):判断是否以指定的字符串结尾
boolean isEmpty():判断字符串的内容是否为空
```

5. String类的获取功能：
```
int length():返回字符串的长度。字符的个数。
char charAt(int index):返回字符串中指定位置的字符。
int indexOf(int ch):返回指定字符在字符串中第一次出现的位置
int indexOf(String str):返回指定字符串在字符串中第一次出现的位置
int indexOf(int ch,int fromIndex):返回指定字符从指定位置开始在字符串中第一次出现的位置
int indexOf(String str,int fromIndex):返回指定字符串从指定位置开始在字符串中第一次出现的位置
String substring(int start):返回从指定位置开始到末尾的子串
String substring(int start,int end):返回从指定位置开始到指定位置结束的子串----注意左包右不包,
即[start,end);
```

6. String的转换功能：
```
 byte[] getBytes():把字符串转换为字节数组
 char[] toCharArray():把字符串转换为字符数组
 static String valueOf(char[] chs):把字符数组转成字符串
 static String valueOf(int i):把int类型的数据转成字符串
 //把任意类型转换为字符串的方法,省略
 String toLowerCase():把字符串转小写
 String toUpperCase():把字符串转大写
 String concat(String str):字符串的连接
```

7. 替换功能：
```
String replace(char old,char new)
String replace(String old,String new)
```

8. 去除字符串两侧空格：
```
 String trim()
```

9. 按字典顺序比较两个字符串  a-z
```
 int compareTo(String str)
 int compareToIgnoreCase(String str)
```

#### 格式化字符串

**参考即可,不用记忆**

String类的format()方法用于创建格式化的字符串以及连接多个字符串对象。熟悉C语言的同学应该记得C语言的sprintf()方法，两者有类似之处。format()方法有两种重载形式。
```
format(String format, Object... args) 新字符串使用本地语言环境，制定字符串格式和参数生成格式化的新字符串。
format(Locale locale, String format, Object... args) 使用指定的语言环境，制定字符串格式和参数生成格式化的字符串。
```
显示不同转换符实现不同数据类型到字符串的转换，如图所示
![](https://i.loli.net/2019/04/09/5cac38caf12a9.png)
示例：
```java
public static void main(String[] args) {
    String str=null;
    str=String.format("Hi,%s", "平心");
    System.out.println(str);
    str=String.format("Hi,%s:%s.%s", "平","心","韩");
    System.out.println(str);
    System.out.printf("字母a的大写是：%c %n", 'A');
    System.out.printf("3>7的结果是：%b %n", 3>7);
    System.out.printf("100的一半是：%d %n", 100/2);
    System.out.printf("100的16进制数是：%x %n", 100);
    System.out.printf("100的8进制数是：%o %n", 100);
    System.out.printf("50元的书打8.5折扣是：%f 元%n", 50*0.85);
    System.out.printf("上面价格的16进制数是：%a %n", 50*0.85);
    System.out.printf("上面价格的指数表示：%e %n", 50*0.85);
    System.out.printf("上面价格的指数和浮点数结果的长度较短的是：%g %n", 50*0.85);
    System.out.printf("上面的折扣是%d%% %n", 85);
    System.out.printf("字母A的散列码是：%h %n", 'A');
}
```
运行结果：
```（shell）
$ java TestString
Hi,平心
Hi,平:心.韩
字母a的大写是：A
3>7的结果是：false
100的一半是：50
100的16进制数是：64
100的8进制数是：144
50元的书打8.5折扣是：42.500000 元
上面价格的16进制数是：0x1.54p5
上面价格的指数表示：4.250000e+01
上面价格的指数和浮点数结果的长度较短的是：42.5000
上面的折扣是85%
字母A的散列码是：41
```
**搭配转换符的标志**

![](https://i.loli.net/2019/04/09/5cac3b223da50.png)
示例：
```（java）
public static void main(String[] args) {
    String str=null;
    //$使用
    str=String.format("格式参数$的使用：%1$d,%2$s", 99,"abc");
    System.out.println(str);
    //+使用
    System.out.printf("显示正负数的符号：%+d与%d%n", 99,-99);
    //补O使用
    System.out.printf("最牛的编号是：%03d%n", 7);
    //空格使用
    System.out.printf("Tab键的效果是：% 8d%n", 7);
    //.使用
    System.out.printf("整数分组的效果是：%,d%n", 9989997);
    //空格和小数点后面个数
    System.out.printf("一本书的价格是：% 50.5f元%n", 49.8);
}
```
运行结果：
```
$ java TestString
格式参数$的使用：99,abc
显示正负数的符号：+99与-99
最牛的编号是：007
Tab键的效果是：       7
整数分组的效果是：9,989,997
一本书的价格是：                                          49.80000元
```
**日期和事件字符串格式化**
在程序界面中经常需要显示时间和日期，但是其显示的 格式经常不尽人意，需要编写大量的代码经过各种算法才得到理想的日期与时间格式。字符串格式中还有%tx转换符没有详细介绍，它是专门用来格式化日期和时间的。%tx转换符中的x代表另外的处理日期和时间格式的转换符，它们的组合能够将日期和时间格式化成多种格式。

常见日期和时间组合的格式，如图所示。

![](https://i.loli.net/2019/04/09/5cac3c926dbe0.png)


示例：
```（java）
//需要导入`import java.util.Date;`

public static void main(String[] args) {
    Date date=new Date();
    //c的使用
    System.out.printf("全部日期和时间信息：%tc%n",date);
    //f的使用
    System.out.printf("年-月-日格式：%tF%n",date);
    //d的使用
    System.out.printf("月/日/年格式：%tD%n",date);
    //r的使用
    System.out.printf("HH:MM:SS PM格式（12时制）：%tr%n",date);
    //t的使用
    System.out.printf("HH:MM:SS格式（24时制）：%tT%n",date);
    //R的使用
    System.out.printf("HH:MM格式（24时制）：%tR",date);
}
```
运行结果：
```
$ java TestString
全部日期和时间信息：星期二 四月 09 14:42:50 CST 2019
年-月-日格式：2019-04-09
月/日/年格式：04/09/19
HH:MM:SS PM格式（12时制）：02:42:50 下午
HH:MM:SS格式（24时制）：14:42:50
HH:MM格式（24时制）：14:42
```
定义日期格式的转换符可以使日期通过指定的转换符生成新字符串。这些日期转换符如图所示。

![Ziivxe.png](https://s2.ax1x.com/2019/06/23/Ziivxe.png)

示例：

```java
public static void main(String[] args) {
    Date date=new Date();
    //b的使用，月份简称
    String str=String.format(Locale.US,"英文月份简称：%tb",date);
    System.out.println(str);
    System.out.printf("本地月份简称：%tb%n",date);
    //B的使用，月份全称
    str=String.format(Locale.US,"英文月份全称：%tB",date);
    System.out.println(str);
    System.out.printf("本地月份全称：%tB%n",date);
    //a的使用，星期简称
    str=String.format(Locale.US,"英文星期的简称：%ta",date);
    System.out.println(str);
    //A的使用，星期全称
    System.out.printf("本地星期的简称：%tA%n",date);
    //C的使用，年前两位
    System.out.printf("年的前两位数字（不足两位前面补0）：%tC%n",date);
    //y的使用，年后两位
    System.out.printf("年的后两位数字（不足两位前面补0）：%ty%n",date);
    //j的使用，一年的天数
    System.out.printf("一年中的天数（即年的第几天）：%tj%n",date);
    //m的使用，月份
    System.out.printf("两位数字的月份（不足两位前面补0）：%tm%n",date);
    //d的使用，日（二位，不够补零）
    System.out.printf("两位数字的日（不足两位前面补0）：%td%n",date);
    //e的使用，日（一位不补零）
    System.out.printf("月份的日（前面不补0）：%te",date);
  }
```
输出结果：
```
英文月份简称：Sep
本地月份简称：九月
英文月份全称：September
本地月份全称：九月
英文星期的简称：Mon
本地星期的简称：星期一
年的前两位数字（不足两位前面补0）：20
年的后两位数字（不足两位前面补0）：12
一年中的天数（即年的第几天）：254
两位数字的月份（不足两位前面补0）：09
两位数字的日（不足两位前面补0）：10
月份的日（前面不补0）：10
```
和日期格式转换符相比，时间格式的转换符要更多、更精确。它可以将时间格式化成时、分、秒甚至时毫秒等单位。格式化时间字符串的转换符如图所示。
![Zik9lF.png](https://s2.ax1x.com/2019/06/23/Zik9lF.png)

示例：
```（java）
public static void main(String[] args) {
    Date date = new Date();
    //H的使用
    System.out.printf("2位数字24时制的小时（不足2位前面补0）:%tH%n", date);
    //I的使用
    System.out.printf("2位数字12时制的小时（不足2位前面补0）:%tI%n", date);
    //k的使用
    System.out.printf("2位数字24时制的小时（前面不补0）:%tk%n", date);
    //l的使用
    System.out.printf("2位数字12时制的小时（前面不补0）:%tl%n", date);
    //M的使用
    System.out.printf("2位数字的分钟（不足2位前面补0）:%tM%n", date);
    //S的使用
    System.out.printf("2位数字的秒（不足2位前面补0）:%tS%n", date);
    //L的使用
    System.out.printf("3位数字的毫秒（不足3位前面补0）:%tL%n", date);
    //N的使用
    System.out.printf("9位数字的毫秒数（不足9位前面补0）:%tN%n", date);
    //p的使用
    String str = String.format(Locale.US, "小写字母的上午或下午标记(英)：%tp", date);
    System.out.println(str);
    System.out.printf("小写字母的上午或下午标记（中）：%tp%n", date);
    //z的使用
    System.out.printf("相对于GMT的RFC822时区的偏移量:%tz%n", date);
    //Z的使用
    System.out.printf("时区缩写字符串:%tZ%n", date);
    //s的使用
    System.out.printf("1970-1-1 00:00:00 到现在所经过的秒数：%ts%n", date);
    //Q的使用
    System.out.printf("1970-1-1 00:00:00 到现在所经过的毫秒数：%tQ%n", date);
}
```
输出结果：
```
2位数字24时制的小时（不足2位前面补0）:11
2位数字12时制的小时（不足2位前面补0）:11
2位数字24时制的小时（前面不补0）:11
2位数字12时制的小时（前面不补0）:11
2位数字的分钟（不足2位前面补0）:03
2位数字的秒（不足2位前面补0）:52
3位数字的毫秒（不足3位前面补0）:773
9位数字的毫秒数（不足9位前面补0）:773000000
小写字母的上午或下午标记(英)：am
小写字母的上午或下午标记（中）：上午
相对于GMT的RFC822时区的偏移量:+0800
时区缩写字符串:CST
1970-1-1 00:00:00 到现在所经过的秒数：1347246232
1970-1-1 00:00:00 到现在所经过的毫秒数：1347246232773
```

#### StringBuilder

在程序开发过程中，我们常常碰到字符串连接的情况，方便和直接的方式是通过”+”符号来实现，但是这种方式达到目的的效率比较低，且每执行一次都会创建一个String对象，即耗时，又浪费空间。使用StringBuilder类就可以避免这种问题的发生，下面就Stringbuilder的使用做个简要的总结：

创建Stringbuilder对象
```
StringBuilder strB = new StringBuilder();
StringBuilder strB = new StringBuilder("abc");
```

1.  append(String str)/append(Char c)：字符串连接
```
System.out.println("StringBuilder:"+strB.append("ch").append("111").append('c'));
//return “StringBuilder:ch111c”
```

2. toString()：返回一个与构建起或缓冲器内容相同的字符串
```
System.out.println("String:"+strB.toString());
//return “String:ch111c”
```

3. appendcodePoint(int cp)：追加一个代码点，并将其转换为一个或两个代码单元并返回this
```
System.out.println("StringBuilder.appendCodePoint:"+strB.appendCodePoint(2));
//return “StringBuilder.appendCodePoint:ch111c[]”
```

4. setCharAt(int i, char c)：将第 i 个代码单元设置为 c（可以理解为替换）
```
strB.setCharAt(2, 'd');
System.out.println("StringBuilder.setCharAt:" + strB);
//return “StringBuilder.setCharAt:chd11c[]”
```

5. insert(int offset, String str)/insert(int offset, Char c)：在指定位置之前插入字符(串)
```
System.out.println("StringBuilder.insertString:"+ strB.insert(2, "LS"));
//return “StringBuilder.insertString:chLSd11c[]”
System.out.println("StringBuilder.insertString:"+ strB.insert(2, 'L'));
//return “StringBuilder.insertChar:chLLSd11c[]”
```

6. delete(int startIndex,int endIndex)：删除起始位置（含）到结尾位置（不含）之间的字符串
```
System.out.println(“StringBuilder.delete:”+ strB.delete(2, 4));
//return “StringBuilder.delete:chSd11c[]”
```

**StringBuilder是线程不安全的。**

#### StringBuffer类

StringBuffer类和String一样，也用来代表字符串，只是由于StringBuffer的内部实现方式和String不同，所以StringBuffer在进行字符串处理时，不生成新的对象，在内存使用上要优于String类。

所以在实际使用时，如果经常需要对一个字符串进行修改，例如插入、删除等操作，使用StringBuffer要更加适合一些。

在StringBuffer类中存在很多和String类一样的方法，这些方法在功能上和String类中的功能是完全一样的。

但是有一个最显著的区别在于，对于StringBuffer对象的每次修改都会改变对象自身，这点是和String类最大的区别。

另外由于StringBuffer是线程安全的，关于线程的概念后续有专门的章节进行介绍，所以在多线程程序中也可以很方便的进行使用，但是程序的执行效率相对来说就要稍微慢一些。

**StringBuffer对象的初始化**

StringBuffer对象的初始化不像String类的初始化一样，Java提供的有特殊的语法，而通常情况下一般使用构造方法进行初始化。

例如：
```
StringBuffer s = new StringBuffer();
```
这样初始化出的StringBuffer对象是一个空的对象。
如果需要创建带有内容的StringBuffer对象，则可以使用：
```
 StringBuffer s = new StringBuffer(“abc”);
```
这样初始化出的StringBuffer对象的内容就是字符串”abc”。
需要注意的是，StringBuffer和String属于不同的类型，也不能直接进行强制类型转换，下面的代码都是错误的：
```
StringBuffer s = “abc”;               //赋值类型不匹配
StringBuffer s = (StringBuffer)”abc”;    //不存在继承关系，无法进行强转
```
StringBuffer对象和String对象之间的互转的代码如下：
```
String s = “abc”;
StringBuffer sb1 = new StringBuffer(“123”);
StringBuffer sb2 = new StringBuffer(s);   //String转换为StringBuffer
String s1 = sb1.toString();              //StringBuffer转换为String
```
StringBuffer() 构造一个空的字符串缓冲区，并且初始化为 16 个字符的容量。
StringBuffer(int length) 创建一个空的字符串缓冲区，并且初始化为指定长度 length 的容量。
StringBuffer(String str) 创建一个字符串缓冲区，并将其内容初始化为指定的字符串内容 str，字符串缓冲区的初始容量为 16 加上字符串 str 的长度。

对于StringBuffer而言不可直接强制类型转化，即StringBuffer a=（StringBuffer）‘ac’是错误的使用方法。
对于StringBuffer转化为String可使用 String b=a.toString(），对于String转为StringBuffer可使用StringBuffer b=new StringBuffer(string)


**常用函数使用：**

```java
public class UsingStringBuffer {
	/**
	 * 查找匹配字符串
	 */
	public static void testFindStr() {
	StringBuffer sb = new StringBuffer();
	sb.append("This is a StringBuffer");
	// 返回子字符串在字符串中最先出现的位置，如果不存在，返回负数
	System.out.println("sb.indexOf(\"is\")=" + sb.indexOf("is"));
	// 给indexOf方法设置参数，指定匹配的起始位置
	System.out.println("sb.indexOf(\"is\")=" + sb.indexOf("is", 3));
	// 返回子字符串在字符串中最后出现的位置，如果不存在，返回负数
	System.out.println("sb.lastIndexOf(\"is\") = " + sb.lastIndexOf("is"));
	// 给lastIndexOf方法设置参数，指定匹配的结束位置
	System.out.println("sb.lastIndexOf(\"is\", 1) = "
			+ sb.lastIndexOf("is", 1));
	}

	/**
	 * 截取字符串
	 */
	public static void testSubStr() {
	StringBuffer sb = new StringBuffer();
	sb.append("This is a StringBuffer");
	// 默认的终止位置为字符串的末尾
	System.out.print("sb.substring(4)=" + sb.substring(4));
	// substring方法截取字符串，可以指定截取的起始位置和终止位置
	System.out.print("sb.substring(4,9)=" + sb.substring(4, 9));
	}

	/**
	 * 获取字符串中某个位置的字符
	 */
	public static void testCharAtStr() {
	StringBuffer sb = new StringBuffer("This is a StringBuffer");
	System.out.println(sb.charAt(sb.length() - 1));
	}

	/**
	 * 添加各种类型的数据到字符串的尾部
	 */
	public static void testAppend() {
	StringBuffer sb = new StringBuffer("This is a StringBuffer!");
	sb.append(1.23f);
	System.out.println(sb.toString());
	}

	/**
	 * 删除字符串中的数据
	 */
	public static void testDelete() {
	StringBuffer sb = new StringBuffer("This is a StringBuffer!");
	sb.delete(0, 5);
	sb.deleteCharAt(sb.length() - 1);
	System.out.println(sb.toString());
	}

	/**
	 * 向字符串中插入各种类型的数据
	 */
	public static void testInsert() {
		StringBuffer sb = new StringBuffer("This is a StringBuffer!");
		// 能够在指定位置插入字符、字符数组、字符串以及各种数字和布尔值
		sb.insert(2, 'W');
		sb.insert(3, new char[] { 'A', 'B', 'C' });
		sb.insert(8, "abc");
		sb.insert(2, 3);
		sb.insert(3, 2.3f);
		sb.insert(6, 3.75d);
		sb.insert(5, 9843L);
		sb.insert(2, true);
		System.out.println("testInsert: " + sb.toString());
	}

	/**
	 * 替换字符串中的某些字符
	 */
	public static void testReplace() {
		StringBuffer sb = new StringBuffer("This is a StringBuffer!");
		// 将字符串中某段字符替换成另一个字符串
		sb.replace(10, sb.length(), "Integer");
		System.out.println("testReplace: " + sb.toString());
	}

	/**
	 * 将字符串倒序
	 */
	public static void reverseStr() {
		StringBuffer sb = new StringBuffer("This is a StringBuffer!");
		System.out.println(sb.reverse()); // reverse方法将字符串倒序
	}
    public static void main(String[] args) {
       UsingStringBuffer.testAppend();
       UsingStringBuffer.testCharAtStr();
       UsingStringBuffer.testDelete();
       UsingStringBuffer.testFindStr();
       UsingStringBuffer.testInsert();
       UsingStringBuffer.testReplace();
       UsingStringBuffer.testSubStr();
       UsingStringBuffer.reverseStr();
    }
}
```
运行结果：
```
This is a StringBuffer!1.23
r
is a StringBuffer
sb.indexOf("is")=2
sb.indexOf("is")=5
sb.lastIndexOf("is") = 5
sb.lastIndexOf("is", 1) = -1
testInsert: Thtrue32.984333.75WABCisabc is a StringBuffer!
testReplace: This is a Integer
sb.substring(4)= is a StringBuffer
sb.substring(4,9)= is a
!reffuBgnirtS a si sihT
```

 StringBuffer类的replace方法同String类的replace方法不同,它的replace方法有三个参数,第一个参数指定被替换子串的起始位置，第二个参数指定被替换子串的终止位置，第三个参数指定新子串


 示例：

```
public class Main
{
  public static void main(String[] args)
  {
    String a = "a";
    long start = System.currentTimeMillis();
    String string = a;
    for (int i = 0; i < 100000; i++) {
         string += a;
    }
    if (string.equals("abc")) {}
    System.out.println("string+ cost time:" + (System.currentTimeMillis() - start) + "ms");
    start = System.currentTimeMillis();
    StringBuffer stringBuffer = new StringBuffer();
    for (int i = 0; i < 100000; i++) {
        stringBuffer.append(a);
    }
    if (stringBuffer.toString().equals("abc")) {}
    System.out.println("stringbuffer cost time:" + (System.currentTimeMillis() - start) + "ms");
  }
}
```

运行结果：
```
$ java  Main
string+ cost time:4367ms
stringbuffer cost time:3ms
```
string+事实上是由stringbuilder完成的。我们来看一下这个程序的class文件内容就可以看出来了。

#### 不可变类

**不可变类**：所谓的不可变类是指这个类的实例一旦创建完成后，就不能改变其成员变量值。如JDK内部自带的很多不可变类：Interger、Long和String等。
**可变类**：相对于不可变类，可变类创建实例后可以改变其成员变量值，开发中创建的大部分类都属于可变类。

**不可变类的优点**

说完可变类和不可变类的区别，我们需要进一步了解为什么要有不可变类？这样的特性对JAVA来说带来怎样的好处？

1. 线程安全
   不可变对象是线程安全的，在线程之间可以相互共享，不需要利用特殊机制来保证同步问题，因为对象的值无法改变。可以降低并发错误的可能性，因为不需要用一些锁机制等保证内存一致性问题也减少了同步开销。
2. 易于构造、使用和测试

**不可变类的设计方法**

对于设计不可变类，个人总结出以下原则：

1. 类添加final修饰符，保证类不被继承。
   如果类可以被继承会破坏类的不可变性机制，只要继承类覆盖父类的方法并且继承类可以改变成员变量值，那么一旦子类以父类的形式出现时，不能保证当前类是否可变。
2. 保证所有成员变量必须私有，并且加上final修饰
   通过这种方式保证成员变量不可改变。但只做到这一步还不够，因为如果是对象成员变量有可能再外部改变其值。所以第4点弥补这个不足。
3. 不提供改变成员变量的方法，包括setter
   避免通过其他接口改变成员变量的值，破坏不可变特性。

4. 通过构造器初始化所有成员，进行深拷贝(deep copy)

如果构造器传入的对象直接赋值给成员变量，还是可以通过对传入对象的修改进而导致改变内部变量的值。例如：

```
public final class ImmutableDemo {  
    private final int[] myArray;  
    public ImmutableDemo(int[] array) {  
        this.myArray = array; // wrong  
    }  
}
```

这种方式不能保证不可变性，myArray和array指向同一块内存地址，用户可以在ImmutableDemo之外通过修改array对象的值来改变myArray内部的值。
为了保证内部的值不被修改，可以采用深度copy来创建一个新内存保存传入的值。正确做法：

```
public final class MyImmutableDemo {  
    private final int[] myArray;  
    public MyImmutableDemo(int[] array) {  
        this.myArray = array.clone();   
    }   
}
```

5. 在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝

   这种做法也是防止对象外泄，防止通过getter获得内部可变成员对象后对成员变量直接操作，导致成员变量发生改变。

**String对象的不可变性**

string对象在内存创建后就不可改变，不可变对象的创建一般满足以上5个原则，我们看看String代码是如何实现的。

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
{
    /** The value is used for character storage. */
    private final char value[];
    /** The offset is the first index of the storage that is used. */
    private final int offset;
    /** The count is the number of characters in the String. */
    private final int count;
    /** Cache the hash code for the string */
    private int hash; // Default to 0
    ....
    public String(char value[]) {
         this.value = Arrays.copyOf(value, value.length); // deep copy操作
     }
    ...
     public char[] toCharArray() {
     // Cannot use Arrays.copyOf because of class initialization order issues
        char result[] = new char[value.length];
        System.arraycopy(value, 0, result, 0, value.length);
        return result;
    }
    ...
}
```

如上代码所示，可以观察到以下设计细节:

1. String类被final修饰，不可继承
2. string内部所有成员都设置为私有变量
3. 不存在value的setter
4. 并将value和offset设置为final。
5. 当传入可变数组value[]时，进行copy而不是直接将value[]复制给内部变量.
6. 获取value时不是直接返回对象引用，而是返回对象的copy.

这都符合上面总结的不变类型的特性，也保证了String类型是不可变的类。

**String对象的不可变性的优缺点**

从上一节分析，String数据不可变类，那设置这样的特性有什么好处呢？我总结为以下几点：

1. 字符串常量池的需要.

   字符串常量池可以将一些字符常量放在常量池中重复使用，避免每次都重新创建相同的对象、节省存储空间。但如果字符串是可变的，此时相同内容的String还指向常量池的同一个内存空间，当某个变量改变了该内存的值时，其他遍历的值也会发生改变。所以不符合常量池设计的初衷。

2. 线程安全考虑。
   同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。
3. 类加载器要用到字符串，不可变性提供了安全性，以便正确的类被加载。譬如你想加载java.sql.Connection类，而这个值被改成了myhacked.Connection，那么会对你的数据库造成不可知的破坏。
4. 支持hash映射和缓存。
   因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

缺点：如果有对String对象值改变的需求，那么会创建大量的String对象。

**String对象的是否真的不可变**

虽然String对象将value设置为final,并且还通过各种机制保证其成员变量不可改变。但是还是可以通过反射机制的手段改变其值。例如：

```
    //创建字符串"Hello World"， 并赋给引用s
    String s = "Hello World"; 
    System.out.println("s = " + s); //Hello World

    //获取String类中的value字段
    Field valueFieldOfString = String.class.getDeclaredField("value");
    //改变value属性的访问权限
    valueFieldOfString.setAccessible(true);

    //获取s对象上的value属性的值
    char[] value = (char[]) valueFieldOfString.get(s);
    //改变value所引用的数组中的第5个字符
    value[5] = '_';
    System.out.println("s = " + s);  //Hello_World
```

打印结果为：

```
s = Hello World
s = Hello_World
```

发现String的值已经发生了改变。也就是说，通过反射是可以修改所谓的“不可变”对象的

**总结**

不可变类是实例创建后就不可以改变成员遍历的值。这种特性使得不可变类提供了线程安全的特性但同时也带来了对象创建的开销，每更改一个属性都是重新创建一个新的对象。JDK内部也提供了很多不可变类如Integer、Double、String等。String的不可变特性主要为了满足常量池、线程安全、类加载的需求。合理使用不可变类可以带来极大的好处。

### 参考：

1. https://www.cnblogs.com/xiaoxi/p/6036701.html
2. https://blog.csdn.net/qq_34490018/article/details/82110578
3. [理解Java字符串常量池与intern()方法](https://www.cnblogs.com/justcooooode/p/7603381.html)
