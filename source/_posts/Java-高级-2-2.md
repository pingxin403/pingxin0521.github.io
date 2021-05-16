---
title: Java  正则表达式
date: 2019-04-25 14:18:59
tags:
 - Java
categories:
 - Java
 - 高级
---

正则表达式(regular expression)是描述或表达在目标字符序列内匹配一定字符模式的字符序列。正则表达式在Unix世界已存在多年并凭借sed 、 grep(查找正则表达式)、awk等文本编辑工具而广为使用。在Unix平台上使用的正则表达式因其长久的历史已经成了大部分正则表达式处理器的基础。

<!--more-->

Perl享有较高水平的正则表达式集成,正当Java从未触及这一高度之际,java.util.regex包为Java带来了福音,它可以提供与Perl同等水平的表达能力。当然,Perl与Java的使用模型是不同的:Perl是过程式脚本语言,而Java是编译的、面向对象的语言。但是Java正则表达式API易于使用,并且现在允许Java轻松实现文本处理任务,而该任务在以往都是“外包”给Perl的.

java.util.regex 程序包只包含用于实现 Java 正则表达式处理技术的两个类,分别名为 Pattern 和 Matcher 。 自 然 而 然 你 会 想 到 正 则 表 达 式 由 模 式 匹 配 ( pattern matching)而成.java.lang 还定义了一个新接口,它支持这些新的类。

java.util.regex 包主要包括以下三个类：

- Pattern 类：

  pattern 对象是一个正则表达式的编译表示。Pattern 类没有公共构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态编译方法，它返回一个 Pattern 对象。该方法接受一个正则表达式作为它的第一个参数。

- Matcher 类：

  Matcher 对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。

- PatternSyntaxException：

  PatternSyntaxException 是一个非强制异常类，它表示一个正则表达式模式中的语法错误。



在研究 Patternt和 Matcher 之前,我们先快速浏览一下 CharSequence 这一新概念。


#### CharSequence接口

正则表达式是根据字符序列进行模式匹配的。虽然 String 对象封装了字符序列,但是它们并不能够这样做的唯一对象。
```
package java.lang;
public interface CharSequence
{
int length( );
char charAt (int index);
public String toString( );
CharSequence subSequence (int start, int end);
}
```
CharSequence 描述的每个字符序列通过 length( )方法会返回某个长度值。通过调用charAt( )可以得到序列的各个字符,其中索引是期望的字符位置(desired character position)。字符位置从零到字符序列的长度之间,与我们熟悉的 String.charAt( )基本一样。

toString( )方法返回的 String 对象包括所描述的字符序列。这可能很有用,如打印字符序列。正如之前提过的,String 现在实现了 CharSequence。String 和 CharSequence 同为不变的,因此如果 CharSequence 描述一个完整的 String,那么 CharSequence 的 toString( )方法返回的是潜在的 String 对象而不是副本。如果备份对象是 StringBuffer 或 CharBuffer,系统将创建一个新的 String 保存字符序列的副本。

最 后 通 过 调 用 subSequence( ) 方 法 会 创 建 一 个 新 的 CharSequence 描 述 子 范 围(subrange)。start 和 end 的指定方式与 String.substring( )的方式相同:start 必须是序列的有效索引(valid index);end 必须比 start 大,标志的是最末字符的索引加一。换句话说,start 是起始索引(计算在内),end 是结束索引(不计算在内)。

CharSequence 接口因为没有赋值方法(mutator method)看上去似乎是不变的,但是基本的实现对象可能不是不变的。CharSequence 方法反映了基本对象的现状。如果状态改变,CharSequence 方法返回的信息同样会发生变化。如何你依赖 CharSequence 保持稳定且不确认基础的实现,你可以调用 toString( )方法,对字符序列拍个真实不变的快照。
```
public class CharSeq
{
  public static void main (String [] argv)
  {
    StringBuffer stringBuffer = new StringBuffer ("Hello World");
    CharBuffer charBuffer = CharBuffer.allocate (20);
    CharSequence charSequence = "Hello World";
    //直接来源于String
    printCharSequence (charSequence);
    //来源于StringBuffer
    charSequence = stringBuffer;
    printCharSequence (charSequence);
    //更改StringBuffer
    stringBuffer.setLength (0);
    stringBuffer.append ("Goodbye cruel world");
    //相同、“不变的”CharSequence产生了不同的结果
    printCharSequence (charSequence);
    //从CharBuffer中导出CharSequence
    charSequence = charBuffer;
    charBuffer.put ("xxxxxxxxxxxxxxxxxxxx");
    charBuffer.clear( );
    charBuffer.put ("Hello World");
    charBuffer.flip( );
    printCharSequence (charSequence);
    charBuffer.mark( );
    charBuffer.put ("Seeya");
    charBuffer.reset( );
    printCharSequence (charSequence);
    charBuffer.clear( );
    printCharSequence (charSequence);
    //更改基础CharBuffer会反映在只读的CharSequence接口上
  }
  private static void printCharSequence (CharSequence cs)
  {
  System.out.println ("length=" + cs.length( )
  + ", content='" + cs.toString( ) + "'");
  }
}
```
#### Regix匹配规则

```
\\ 反斜杠
\t 间隔 ('\	')
\n 换行 ('\ ')
\r 回车 ('\ ')
\d 数字 等价于[0-9]
\D 非数字 等价于[^0-9]
\s 空白符号 [\t\n\x0B\f\r]
\S 非空白符号 [^\t\n\x0B\f\r]
\w 单独字符 [a-zA-Z_0-9]
\W 非单独字符 [^a-zA-Z_0-9]
\f 换页符
\e Escape
\b 一个单词的边界
\B 一个非单词的边界
\G 前一个匹配的结束

^为限制开头
^java     条件限制为以Java为开头字符
$为限制结尾
java$     条件限制为以java为结尾字符
.  条件限制除\n以外任意一个单独字符
java..     条件限制为java后除换行外任意两个字符


加入特定限制条件「[]」
[a-z]     条件限制在小写a to z范围中一个字符
[A-Z]     条件限制在大写A to Z范围中一个字符
[a-zA-Z] 条件限制在小写a to z或大写A to Z范围中一个字符
[0-9]     条件限制在小写0 to 9范围中一个字符
[0-9a-z] 条件限制在小写0 to 9或a to z范围中一个字符
[0-9[a-z]] 条件限制在小写0 to 9或a to z范围中一个字符(交集)

[]中加入^后加再次限制条件「[^]」
[^a-z]     条件限制在非小写a to z范围中一个字符
[^A-Z]     条件限制在非大写A to Z范围中一个字符
[^a-zA-Z] 条件限制在非小写a to z或大写A to Z范围中一个字符
[^0-9]     条件限制在非小写0 to 9范围中一个字符
[^0-9a-z] 条件限制在非小写0 to 9或a to z范围中一个字符
[^0-9[a-z]] 条件限制在非小写0 to 9或a to z范围中一个字符(交集)

在限制条件为特定字符出现0次以上时，可以使用「*」
J*     0个以上J
.*     0个以上任意字符
J.*D     J与D之间0个以上任意字符

在限制条件为特定字符出现1次以上时，可以使用「+」
J+     1个以上J
.+     1个以上任意字符
J.+D     J与D之间1个以上任意字符

在限制条件为特定字符出现有0或1次以上时，可以使用「?」
JA?     J或者JA出现

限制为连续出现指定次数字符「{a}」
J{2}     JJ
J{3}     JJJ
文字a个以上，并且「{a,}」
J{3,}     JJJ,JJJJ,JJJJJ,???(3次以上J并存)
文字个以上，b个以下「{a,b}」
J{3,5}     JJJ或JJJJ或JJJJJ
两者取一「|」
J|A     J或A
Java|Hello     Java或Hello

「()」中规定一个组合类型
比如，我查询<a href=\"index.html\">index</a>中<a href></a>间的数据，可写作<a.*href=\".*\">(.+?)</a>
```

Java正则表达式句法:链接：https://pan.baidu.com/s/1LWm2rKzrueyRxSQ-_gNYFA 密码：abx8

在使用Pattern.compile函数时，可以加入控制正则表达式的匹配行为的参数：
```
Pattern Pattern.compile(String regex, int flag)
```
flag的取值范围如下：
```
Pattern.CANON_EQ     当且仅当两个字符的"正规分解(canonical decomposition)"都完全相同的情况下，才认定匹配。比如用了这个标志之后，表达式"a\?"会匹配"?"。默认情况下，不考虑"规 范相等性(canonical equivalence)"。
Pattern.CASE_INSENSITIVE()     默认情况下，大小写不明感的匹配只适用于US-ASCII字符集。这个标志能让表达式忽略大小写进行匹配。要想对Unicode字符进行大小不明感的匹 配，只要将UNICODE_CASE与这个标志合起来就行了。
Pattern.COMMENTS()     在这种模式下，匹配时会忽略(正则表达式里的)空格字符(译者注：不是指表达式里的"\\s"，而是指表达式里的空格，tab，回车之类)。注释从#开始，一直到这行结束。可以通过嵌入式的标志来启用Unix行模式。
Pattern.DOTALL()     在这种模式下，表达式'.'可以匹配任意字符，包括表示一行的结束符。默认情况下，表达式'.'不匹配行的结束符。
Pattern.MULTILINE
()     在这种模式下，'^'和'$'分别匹配一行的开始和结束。此外，'^'仍然匹配字符串的开始，'$'也匹配字符串的结束。默认情况下，这两个表达式仅仅匹配字符串的开始和结束。
Pattern.UNICODE_CASE
()     在这个模式下，如果你还启用了CASE_INSENSITIVE标志，那么它会对Unicode字符进行大小写不明感的匹配。默认情况下，大小写不敏感的匹配只适用于US-ASCII字符集。
Pattern.UNIX_LINES()     在这个模式下，只有'\n'才被认作一行的中止，并且与'.'，'^'，以及'$'进行匹配。
```

#### 贪婪, 勉强和占有
假设待处理的字符串是  xfooxxxxxxfoo

1. 模式`.*foo` （贪婪模式）: 

  模式分为子模式`p1(.*)`和子模式`p2(foo)`两个部分. 其中p1中的量词匹配方式使用默认方式(贪婪型)。 

  匹配开始时,吃入所有字符xfooxxxxxx去匹配子模式p1。匹配成功,但这样以来就没有了字符串去匹配子模式p2。本轮匹配失败；第二轮：减少p1部分的匹配量，吐出最后一个字符, 把字符串分割成xfooxxxxxxfo和o两个子字符串s1和s2。 s1匹配p1, 但s2不匹配p2。本轮匹配失败；第三轮，再次减少p1部分匹配量，吐出两个字符, 字符串被分割成xfooxxxxxxfo和oo两部分。结果同上。第四轮，再次减少p1匹配量, 字符串分割成xfooxxxxxx和foo两个部分, 这次s1/s2分别和p1/p2匹配。停止尝试,返回匹配成功。


2. 模式`.*?foo` （勉强模式）: 最小匹配方式。

  第一次尝试匹配, p1由于是0或任意次，因此被忽略，用字符串去匹配p2,失败；第二次，读入第一个字符x, 尝试和p1匹配, 匹配成功; 字符串剩余部分fooxxxxxxfoo中前三个字符和p2也是匹配的. 因此, 停止尝试, 返回匹配成功。在这种模式下，如果对剩余字符串继续去寻找和模式相匹配的子字符串，还会找到字符串末尾的另一个xfoo，而在贪婪模式下，由于第一次匹配成功的子串就已经是所有字符，因此不存在第二个匹配子串。


3. 模式`.*+foo` （侵占模式）: 也叫占用模式。

  匹配开始时读入所有字符串, 和p1匹配成功, 但没有剩余字符串去和p2匹配。因此, 匹配失败。返回。

  简单地说, 贪婪模式和占有模式相比, 贪婪模式会在只有部分匹配成功的条件下, 依次从多到少减少匹配成功部分模式的匹配数量, 将字符留给模式其他部分去匹配; 而占用模式则是占有所有能匹配成功部分, 绝不留给其他部分使用。

 

再看下面一个例子：贪婪模式与侵占模式的比较
```
正则：\w+[a-z]与\w++[a-z]
目标串：232hjdhfd7474$
分析：

\w+[a-z]：\w+属于贪婪模式，会一次性吃掉它所能吃掉的所有的字符，也就是子串232hjdhfd7474，此时[a-z]不能够找到匹配了，故\w+匹配的串会吐出一个字符4，但此时还是得不到匹配。反复的这样吐出回退，直到吐出字符d时，此时[a-z]能够匹配h，所以这时正则表达式会返回一次成功的匹配结果，为232hjdhfd

\w++[a-z]：\w++属于侵占模式，它会一次性吃掉它所能够吃掉的所有字符，即子串232hjdhfd7474，而且不留给其他部分使用，故不会回退。此时[a-z]不能够找到匹配，所以此次匹配失败。在余下的子串中也找不到能匹配成功的子串。所以整个正则表达式是找不到匹配结果的！
```
![](https://i.loli.net/2019/04/26/5cc28b3d43daf.png)



#### String 正则表达式
所有新的 String 方法是 Pattern 或 Matcher 类的传递调用(pass-through call)。现在你知道了 Pattern 和 Matcher 是如何使用的,并且利用这些 String 公用程序的交互操作(interoperate)应当是傻瓜式的。

![](https://i.loli.net/2019/04/26/5cc283f058e01.png)

示例：[面向对象的文件Grep](http://www.javanio.info/filearea/bookexamples/unpacked/com/ronsoft/books/nio/regex/Grep.java)


#### 常用模式

```
一个或多个汉字	^[\u0391-\uFFE5]+$
邮政编码	^[1-9]\d{5}$（问题：邮政编码可以0开头。修正后为：^[0-9]{6}$或\d{6}等）
QQ号码	^[1-9]\d{4,10}$
邮箱	^[a-zA-Z_]{1,}[0-9]{0,}@(([a-zA-z0-9]-*){1,}\.){1,3}[a-zA-z\-]{1,}$（问题：转义字符本身需要转义，即\.应为\\,其它地方同理.）
用户名（字母开头 + 数字/字母/下划线）	^[A-Za-z][A-Za-z1-9_-]+$
手机号码	^1[3|4|5|8][0-9]\d{8}$（手机号码第二位只能为3458？那也应该修正为：^1[3458]\d{9}）
URL	^((http|https)://)?([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$
匹配只包含协议名、主机名的url：^((http://)|(https://))?((w|W){3}\\.)?[A-Za-z0-9]+\\.((com)|(cn)|(org))$
18位身份证号	^(\d{6})(18|19|20)?(\d{2})([01]\d)([0123]\d)(\d{3})(\d|X|x)?$
```

示例：
```(java)
//比如，在字符串包含验证时

//查找以Java开头,任意结尾的字符串
  Pattern pattern = Pattern.compile("^Java.*");
  Matcher matcher = pattern.matcher("Java不是人");
  boolean b= matcher.matches();
  System.out.println(b);


//以多条件分割字符串时
Pattern pattern = Pattern.compile("[, |]+");
String[] strs = pattern.split("Java Hello World  Java,Hello,,World|Sun");
for (int i=0;i<strs.length;i++) {
    System.out.println(strs[i]);
}

//文字替换（首次出现字符）
Pattern pattern = Pattern.compile("正则表达式");
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World");
//替换第一个符合正则的数据
System.out.println(matcher.replaceFirst("Java"));

//文字替换（全部）
Pattern pattern = Pattern.compile("正则表达式");
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World");
//替换第一个符合正则的数据
System.out.println(matcher.replaceAll("Java"));


//文字替换（置换字符）
Pattern pattern = Pattern.compile("正则表达式");
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World ");
StringBufferr sbr = new StringBuffer();
while (matcher.find()) {
    matcher.appendReplacement(sbr, "Java");
}
matcher.appendTail(sbr);
System.out.println(sbr.toString());

//验证是否为邮箱地址

String str="ceponline@yahoo.com.cn";
Pattern pattern = Pattern.compile("[\\w\\.\\-]+@([\\w\\-]+\\.)+[\\w\\-]+",Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher(str);
System.out.println(matcher.matches());

//去除html标记
Pattern pattern = Pattern.compile("<.+?>", Pattern.DOTALL);
Matcher matcher = pattern.matcher("<a href=\"index.html\">主页</a>");
String string = matcher.replaceAll("");
System.out.println(string);

//查找html中对应条件字符串
Pattern pattern = Pattern.compile("href=\"(.+?)\"");
Matcher matcher = pattern.matcher("<a href=\"index.html\">主页</a>");
if(matcher.find())
  System.out.println(matcher.group(1));
}

//截取http://地址
//截取url
Pattern pattern = Pattern.compile("(http://|https://){1}[\\w\\.\\-/:]+");
Matcher matcher = pattern.matcher("dsdsds<http://dsds//gfgffdfd>fdf");
StringBufferr buffer = new StringBuffer();
while(matcher.find()){
buffer.append(matcher.group());
buffer.append("\r\n");
System.out.println(b?r.toString());
}

//替换指定{}中文字

String str = "Java目前的发展史是由{0}年-{1}年";
String[][] object={new String[]{"\\{0\\}","1995"},new String[]{"\\{1\\}","2007"}};
System.out.println(replace(str,object));

public static String replace(final String sourceString,Object[] object) {
    String temp=sourceString;
    for(int i=0;i<object.length;i++){
	      String[] result=(String[])object[i];
       Pattern    pattern = Pattern.compile(result[0]);
       Matcher matcher = pattern.matcher(temp);
       temp=matcher.replaceAll(result[1]);
    }
    return temp;
}


//以正则条件查询指定目录下文件

//用于缓存文件列表
private ArrayList files = new ArrayList();
//用于承载文件路径
private String _path;
//用于承载未合并的正则公式
private String _regexp;

class MyFileFilter implements FileFilter {

      /**
       * 匹配文件名称
       */
      public boolean accept(File file) {
	try {
	  Pattern pattern = Pattern.compile(_regexp);
	  Matcher match = pattern.matcher(file.getName());
	  return match.matches();
	} catch (Exception e) {
	  return true;
	}
      }
    }

/**
 * 解析输入流
 * @param inputs
 */
FilesAnalyze (String path,String regexp){
    getFileName(path,regexp);
}

/**
 * 分析文件名并加入files
 * @param input
 */
private void getFileName(String path,String regexp) {
    //目录
      _path=path;
      _regexp=regexp;
      File directory = new File(_path);
      File[] filesFile = directory.listFiles(new MyFileFilter());
      if (filesFile == null) return;
      for (int j = 0; j < filesFile.length; j++) {
	files.add(filesFile[j]);
      }
      return;
    }

/**
 * 显示输出信息
 * @param out
 */
public void print (PrintStream out) {
    Iterator elements = files.iterator();
    while (elements.hasNext()) {
	File file=(File) elements.next();
	    out.println(file.getPath());
    }
}

public static void output(String path,String regexp) {

    FilesAnalyze fileGroup1 = new FilesAnalyze(path,regexp);
    fileGroup1.print(System.out);
}

public static void main (String[] args) {
    output("C:\\","[A-z|.]*");
}
```


参看：
1. [菜鸟教程](http://www.runoob.com/java/java-regular-expressions.html)
2. [Java 正则表达式匹配模式](https://www.cnblogs.com/tannerBG/p/5756892.html)
3. [java regex 匹配规则](https://www.cnblogs.com/lixin890808/archive/2013/04/18/3028444.html)
