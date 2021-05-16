---
title: java 日期
date: 2019-04-29 21:18:59
tags:
 - Java
categories:
 - Java
 - 基础
---
### 概述

日期在Java中是一块非常复杂的内容，对于一个日期在不同的语言国别环境中，日期的国际化，日期和时间之间的转换，日期的加减运算，日期的展示格式都是非常复杂的问题。

[java 8 日期时间](https://www.cnblogs.com/ark-blog/p/9694950.html)

<!--more-->

在Java中，操作日期主要涉及到一下几个类：

1. java.util.Date

   类 Date 表示特定的瞬间，精确到毫秒。从 JDK 1.1 开始，应该使用 Calendar 类实现日期和时间字段之间转换，使用 DateFormat 类来格式化和分析日期字符串。Date 中的把日期解释为年、月、日、小时、分钟和秒值的方法已废弃。

2. java.text.DateFormat（抽象类）

   DateFormat 是日期/时间格式化子类的抽象类，它以与语言无关的方式格式化并分析日期或时间。日期/时间格式化子类（如 SimpleDateFormat）允许进行格式化（也就是日期 -> 文本）、分析（文本-> 日期）和标准化。将日期表示为 Date 对象，或者表示为从 GMT（格林尼治标准时间）1970 年，1 月 1 日 00:00:00 这一刻开始的毫秒数。

3. java.text.SimpleDateFormat（DateFormat的直接子类）

   SimpleDateFormat 是一个以与语言环境相关的方式来格式化和分析日期的具体类。它允许进行格式化（日期 -> 文本）、分析（文本 -> 日期）和规范化。
   SimpleDateFormat 使得可以选择任何用户定义的日期-时间格式的模式。但是，仍然建议通过 DateFormat 中的 getTimeInstance、getDateInstance 或 getDateTimeInstance 来新的创建日期-时间格式化程序。

4. java.util.Calendar（抽象类）

   Calendar 类是一个抽象类，它为特定瞬间与一组诸如 YEAR、MONTH、DAY_OF_MONTH、HOUR 等 日历字段之间的转换提供了一些方法，并为操作日历字段（例如获得下星期的日期）提供了一些方法。瞬间可用毫秒值来表示，它是距历元（即格林威治标准时间 1970 年 1 月 1 日的 00:00:00.000，格里高利历）的偏移量。
   与其他语言环境敏感类一样，Calendar 提供了一个类方法 getInstance，以获得此类型的一个通用的对象。Calendar 的 getInstance 方法返回一个 Calendar 对象，其日历字段已由当前日期和时间初始化。

5. java.util.GregorianCalendar（Calendar的直接子类）

   GregorianCalendar 是 Calendar 的一个具体子类，提供了世界上大多数国家使用的标准日历系统。
   GregorianCalendar 是一种混合日历，在单一间断性的支持下同时支持儒略历和格里高利历系统，在默认情况下，它对应格里高利日历创立时的格里高利历日期（某些国家是在 1582 年 10 月 15 日创立，在其他国家要晚一些）。可由调用方通过调用 setGregorianChange() 来更改起始日期。

### java.util.Date

Date内部存储一个long类型的时间，无参构造函数使用`System.currentTimeMillis()`初始化。
```
Date date=new Date();
//相同
Date date=new Date(System.currentTimeMillis());

```
#### java.util.Date的API简介

类 java.util.Date 表示特定的瞬间，精确到毫秒。提供了很多的方法，但是很多已经过时，不推荐使用，下面仅仅列出没有过时的方法：

**构造方法摘要**

Date():分配 Date 对象并用当前时间初始化此对象，以表示分配它的时间（精确到毫秒）。
Date(long date):分配 Date 对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即 1970 年 1 月 1 日 00:00:00 GMT）以来的指定毫秒数。

**方法摘要**

boolean after(Date when):测试此日期是否在指定日期之后。比较long大小

boolean before(Date when):测试此日期是否在指定日期之前。

Object clone():返回此对象的副本。

int compareTo(Date anotherDate):比较两个日期的顺序。

boolean equals(Object obj):比较两个日期的相等性。

long getTime():返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象表示的毫秒数。

void setTime(long time):设置此 Date 对象，以表示 1970 年 1 月 1 日 00:00:00 GMT 以后 time 毫秒的时间点。

String toString():把此 Date 对象转换为以下形式的 String： `dow mon dd hh:mm:ss zzz yyyy` 。其中：
- dow 是一周中的某一天 (Sun, Mon, Tue, Wed, Thu, Fri, Sat)。
- mon 是月份 (Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec)。
- dd 是一月中的某一天（01 至 31），显示为两位十进制数。
- hh 是一天中的小时（00 至 23），显示为两位十进制数。
- mm 是小时中的分钟（00 至 59），显示为两位十进制数。
- ss 是分钟中的秒数（00 至 61），显示为两位十进制数。
- zzz 是时区（并可以反映夏令时）。标准时区缩写包括方法 parse 识别的时区缩写。如果不提供时区信息，则 zzz 为空，即根本不包括任何字符。
- yyyy 是年份，显示为 4 位十进制数。

示例：
```
public class TestDate {
    public static void main(String[] args) {
        TestDate testdate = new TestDate();
        testdate.getSystemCurrentTime();
        testdate.getCurrentDate();
    }
    /**
     * 获取系统当前时间
     * System.currentTimeMillis()返回系统当前时间，结果为1970年1月1日0时0分0秒开始，到程序执行取得系统时间为止所经过的毫秒数
     * 1秒＝1000毫秒
     */
    public void getSystemCurrentTime(){
        System.out.println("---获取系统当前时间---");
        System.out.println(System.currentTimeMillis());
    }
    public void getCurrentDate(){
        System.out.println("---获取系统当前时间---");
         //创建并初始化一个日期（初始值为当前日期）
        Date date = new Date();
        System.out.println("现在的日期是 = " + date.toString());
        System.out.println("自1970年1月1日0时0分0秒开始至今所经历的毫秒数 = " + date.getTime());
    }
}
```
运行结果：
```
---获取系统当前时间---
1556605207401
---获取系统当前时间---
现在的日期是 = Tue Apr 30 14:20:07 CST 2019
自1970年1月1日0时0分0秒开始至今所经历的毫秒数 = 1556605207403
```
#### java.text.DateFormat抽象类的使用

DateFormat 是日期/时间格式化子类的抽象类，它以与语言无关的方式格式化并分析日期或时间。日期/时间格式化子类（如 SimpleDateFormat）允许进行格式化（也就是日期 -> 文本）、分析（文本-> 日期）和标准化。将日期表示为 Date 对象，或者表示为从 GMT（格林尼治标准时间）1970 年，1 月 1 日 00:00:00 这一刻开始的毫秒数。

DateFormat 提供了很多类方法，以获得基于默认或给定语言环境和多种格式化风格的默认日期/时间 Formatter。格式化风格包括 FULL、LONG、MEDIUM 和 SHORT。方法描述中提供了使用这些风格的更多细节和示例。

DateFormat 可帮助进行格式化并分析任何语言环境的日期。对于月、星期，甚至日历格式（阴历和阳历），其代码可完全与语言环境的约定无关。

要格式化一个当前语言环境下的日期，可使用某个静态工厂方法：
```
myString = DateFormat.getDateInstance().format(myDate);
```
如果格式化多个日期，那么获得该格式并多次使用它是更为高效的做法，这样系统就不必多次获取有关环境语言和国家约定的信息了。
```
DateFormat df = DateFormat.getDateInstance();
for (int i = 0; i < myDate.length; ++i) {
 output.println(df.format(myDate[i]) + "; ");
}
```
要格式化不同语言环境的日期，可在 getDateInstance() 的调用中指定它。
```
 DateFormat df = DateFormat.getDateInstance(DateFormat.LONG, Locale.FRANCE);
```
还可使用 DateFormat 进行分析。
```
 myDate = df.parse(myString);
```
使用 getDateInstance 来获得该国家的标准日期格式。另外还提供了一些其他静态工厂方法。使用 getTimeInstance 可获得该国家的时间格式。使用 getDateTimeInstance 可获得日期和时间格式。可以将不同选项传入这些工厂方法，以控制结果的长度（从 SHORT 到 MEDIUM 到 LONG 再到 FULL）。确切的结果取决于语言环境，但是通常：
```
 SHORT 完全为数字，如 12.13.52 或 3:30pm
 MEDIUM 较长，如 Jan 12, 1952
 LONG 更长，如 January 12, 1952 或 3:30:32pm
 FULL 是完全指定，如 Tuesday, April 12, 1952 AD 或 3:30:42pm PST。
```
如果愿意，还可以在格式上设置时区。如果想对格式化或分析施加更多的控制（或者给予用户更多的控制），可以尝试将从工厂方法所获得的 DateFormat 强制转换为 SimpleDateFormat。这适用于大多数国家；只是要记住将其放入一个 try 代码块中，以防遇到特殊的格式。

还可以使用借助 ParsePosition 和 FieldPosition 的分析和格式化方法形式来：逐步地分析字符串的各部分。 对齐任意特定的字段，或者找出字符串在屏幕上的选择位置。

DateFormat 不是同步的。建议为每个线程创建独立的格式实例。如果多个线程同时访问一个格式，则它必须保持外部同步。

#### java.text.SimpleDateFormat

SimpleDateFormat 是一个以与语言环境相关的方式来格式化和分析日期的具体类。它允许进行格式化（日期 -> 文本）、分析（文本 -> 日期）和规范化。

SimpleDateFormat 使得可以选择任何用户定义的日期-时间格式的模式。但是，仍然建议通过 DateFormat 中的 getTimeInstance、getDateInstance 或 getDateTimeInstance 来新的创建日期-时间格式化程序。每一个这样的类方法都能够返回一个以默认格式模式初始化的日期/时间格式化程序。可以根据需要使用 applyPattern 方法来修改格式模式。

日期和时间模式

日期和时间格式由日期和时间模式 字符串指定。在日期和时间模式字符串中，未加引号的字母 'A' 到 'Z' 和 'a' 到 'z' 被解释为模式字母，用来表示日期或时间字符串元素。文本可以使用单引号 (') 引起来，以免进行解释。"''" 表示单引号。所有其他字符均不解释；只是在格式化时将它们简单复制到输出字符串，或者在分析时与输入字符串进行匹配。

定义了以下模式字母（所有其他字符 'A' 到 'Z' 和 'a' 到 'z' 都被保留）：

![](https://i.loli.net/2019/04/30/5cc720ae98dea.png)

示例：

```java
public class TestDateFormat {
    public static void main(String[] args) throws ParseException {
        TestDateFormat tdf = new TestDateFormat();
        tdf.dateFormat();
    }
    /**
     * 对SimpleDateFormat类进行测试
     * @throws ParseException
     */
    public void dateFormat() throws ParseException{
         //创建日期
        Date date = new Date();

        //创建不同的日期格式
        DateFormat df1 = DateFormat.getInstance();
        DateFormat df2 = new SimpleDateFormat("yyyy-MM-01 hh:mm:ss EE");
        DateFormat df3 = DateFormat.getDateInstance(DateFormat.FULL, Locale.CHINA);     //产生一个指定国家指定长度的日期格式，长度不同，显示的日期完整性也不同
        DateFormat df4 = new SimpleDateFormat("yyyy年MM月dd日 hh时mm分ss秒 EE", Locale.CHINA);
        DateFormat df5 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss EEEEEE", Locale.US);
        DateFormat df6 = new SimpleDateFormat("yyyy-MM-dd");  

        //将日期按照不同格式进行输出
        System.out.println("-------将日期按照不同格式进行输出------");
        System.out.println("按照Java默认的日期格式，默认的区域                      : " + df1.format(date));
        System.out.println("按照指定格式 yyyy-MM-dd hh:mm:ss EE ，系统默认区域      :" + df2.format(date));
        System.out.println("按照日期的FULL模式，区域设置为中文                      : " + df3.format(date));
        System.out.println("按照指定格式 yyyy年MM月dd日 hh时mm分ss秒 EE ，区域为中文 : " + df4.format(date));
        System.out.println("按照指定格式 yyyy-MM-dd hh:mm:ss EE ，区域为美国        : " + df5.format(date));
        System.out.println("按照指定格式 yyyy-MM-dd ，系统默认区域                  : " + df6.format(date));

        //将符合该格式的字符串转换为日期，若格式不相配，则会出错
        Date date1 = df1.parse("16-01-24 下午2:32");
        Date date2 = df2.parse("2016-01-24 02:51:07 星期日");
        Date date3 = df3.parse("2016年01月24日 星期五");
        Date date4 = df4.parse("2016年01月24日 02时51分18秒 星期日");
        Date date5 = df5.parse("2016-01-24 02:51:18 Sunday");
        Date date6 = df6.parse("2016-01-24");

        System.out.println("-------输出将字符串转换为日期的结果------");
        System.out.println(date1);
        System.out.println(date2);
        System.out.println(date3);
        System.out.println(date4);
        System.out.println(date5);
        System.out.println(date6);
    }
}
```
#### java.util.Calendar

java.util.Calendar是个抽象类，是系统时间的抽象表示，它为特定瞬间与一组诸如 YEAR、MONTH、DAY_OF_MONTH、HOUR 等 日历字段之间的转换提供了一些方法，并为操作日历字段（例如获得下星期的日期）提供了一些方法。瞬间可用毫秒值来表示，它是距历元（即格林威治标准时间 1970 年 1 月 1 日的 00:00:00.000，格里高利历）的偏移量。

与其他语言环境敏感类一样，Calendar 提供了一个类方法 getInstance，以获得此类型的一个通用的对象。Calendar 的 getInstance 方法返回一个 Calendar 对象，其日历字段已由当前日期和时间初始化。

一个Calendar的实例是系统时间的抽象表示，从Calendar的实例可以知道年月日星期月份时区等信息。Calendar类中有一个静态方法get(int x),通过这个方法可以获取到相关实例的一些值（年月日星期月份等）信息。参数x是一个产量值，在Calendar中有定义。

Calendar中些陷阱，很容易掉下去：
1. Calendar的星期是从周日开始的，常量值为0。
2. Calendar的月份是从一月开始的，常量值为0。
3. Calendar的每个月的第一天值为1。

#### java.util.GregorianCalendar
GregorianCalendar 是 Calendar 的一个具体子类，提供了世界上大多数国家使用的标准日历系统。结合Calendar抽象类使用。
```
public class TestCalendar {
    public static void main(String[] args) throws ParseException {
        TestCalendar testCalendar = new TestCalendar();
        testCalendar.testCalendar();
        testCalendar.testCalendar2();
    }
    public void testCalendar(){
        //创建Calendar的方式
        Calendar now1 = Calendar.getInstance();
        Calendar now2 = new GregorianCalendar();
        Calendar now3 = new GregorianCalendar(2016, 01, 24);
        Calendar now4 = new GregorianCalendar(2016, 01, 24, 15, 55);      //陷阱:Calendar的月份是0~11
        Calendar now5 = new GregorianCalendar(2016, 01, 24, 15, 55, 44);
        Calendar now6 = new GregorianCalendar(Locale.US);
        Calendar now7 = new GregorianCalendar(TimeZone.getTimeZone("GMT-8:00"));

        //通过日期和毫秒数设置Calendar
        now2.setTime(new Date());
        System.out.println(now2);

        now2.setTimeInMillis(new Date().getTime());
        System.out.println(now2);


        //定义日期的中文输出格式,并输出日期
        SimpleDateFormat df = new SimpleDateFormat("yyyy年MM月dd日 hh时mm分ss秒 E", Locale.CHINA);
        System.out.println("获取日期中文格式化化输出：" + df.format(now5.getTime()));
        System.out.println();

        System.out.println("--------通过Calendar获取日期中年月日等相关信息--------");
        System.out.println("获取年：" + now5.get(Calendar.YEAR));
        System.out.println("获取月(月份是从0开始的)：" + now5.get(Calendar.MONTH));
        System.out.println("获取日：" + now5.get(Calendar.DAY_OF_MONTH));
        System.out.println("获取时：" + now5.get(Calendar.HOUR));
        System.out.println("获取分：" + now5.get(Calendar.MINUTE));
        System.out.println("获取秒：" + now5.get(Calendar.SECOND));
        System.out.println("获取上午、下午：" + now5.get(Calendar.AM_PM));
        System.out.println("获取星期数值(星期是从周日开始的)：" + now5.get(Calendar.DAY_OF_WEEK));
        System.out.println();

        System.out.println("---------通用星期中文化转换---------");
        String dayOfWeek[] = {"", "日", "一", "二", "三", "四", "五", "六"};
        System.out.println("now5对象的星期是:" + dayOfWeek[now5.get(Calendar.DAY_OF_WEEK)]);
        System.out.println();

        System.out.println("---------通用月份中文化转换---------");
        String months[] = {"一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"};
        System.out.println("now5对象的月份是: " + months[now5.get(Calendar.MONTH)]);
    }

    public void testCalendar2() throws ParseException{
        //获取当前月份的最大天数
        Calendar cal = Calendar.getInstance();   
        int maxday=cal.getActualMaximum(Calendar.DAY_OF_MONTH);   
        int minday=cal.getActualMinimum(Calendar.DAY_OF_MONTH);   
        System.out.println(maxday);
        //取当月的最后一天  
        DateFormat formatter3=new SimpleDateFormat("yyyy-MM-"+maxday);   
        System.out.println(formatter3.format(cal.getTime()));
        //取当月的最后一天  
        DateFormat formatter4=new SimpleDateFormat("yyyy-MM-"+minday);   
        System.out.println(formatter4.format(cal.getTime()));
        //求两个日期之间相隔的天数
        java.text.SimpleDateFormat format = new java.text.SimpleDateFormat("yyyy-MM-dd");   
        java.util.Date beginDate= format.parse("2007-12-24");   
        java.util.Date endDate= format.parse("2007-12-25");   
        long day=(endDate.getTime()-beginDate.getTime())/(24*60*60*1000);   
        System.out.println("相隔的天数="+day);
        //一年前的日期
        java.text.Format formatter5=new java.text.SimpleDateFormat("yyyy-MM-dd");   
        java.util.Date todayDate=new java.util.Date();   
        long beforeTime=(todayDate.getTime()/1000)-60*60*24*365;   
        todayDate.setTime(beforeTime*1000);   
        String beforeDate=formatter5.format(todayDate);   
        System.out.println(beforeDate);
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.YEAR, -1);
        System.out.println(formatter5.format(calendar.getTime()));
        //当前星期的星期一和星期日
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMMdd");
        GregorianCalendar gregorianCalendar = new GregorianCalendar();
        int dayInWeek = gregorianCalendar.get(Calendar.DAY_OF_WEEK);
        int offset = 0;
        if (dayInWeek == 1) {
            // 星期天
            offset = 6;
        } else {
            // 星期一至星期六
            offset = dayInWeek - 2;
        }
        gregorianCalendar.add(GregorianCalendar.DAY_OF_MONTH, -offset);
        String sday = dateFormat.format(gregorianCalendar.getTime());
        gregorianCalendar.add(GregorianCalendar.DAY_OF_MONTH, 6);
        String eday = dateFormat.format(gregorianCalendar.getTime());

        System.out.println("这个星期的星期一:" + sday);
        System.out.println("这个星期的星期天:" + eday);
    }

}
```

#### 总结

Java中日期的经常有一下五个方面：
1、创建日期
2、日期格式化显示
3、日期的转换（主要是和字符串之间的相互转换）
4、日期中年、月、日、时、分、秒、星期、月份等获取。
5、日期的大小比较、日期的加减。

### 补充

#### java 8 新日期类

```
Instant         时间戳
Duration        持续时间、时间差
LocalDate       只包含日期，比如：2018-09-24
LocalTime       只包含时间，比如：10:32:10
LocalDateTime   包含日期和时间，比如：2018-09-24 10:32:10
Peroid          时间段
ZoneOffset      时区偏移量，比如：+8:00
ZonedDateTime   带时区的日期时间
Clock           时钟，可用于获取当前时间戳
java.time.format.DateTimeFormatter      时间格式化类
```

[java 8 日期时间](https://www.cnblogs.com/ark-blog/p/9694950.html)

#### SimpleDateFormat线程安全

日常开发中，我们经常需要使用时间相关类，说到时间相关类，想必大家对SimpleDateFormat并不陌生。主要是用它进行时间的格式化输出和解析，挺方便快捷的，但是SimpleDateFormat并**不是一个线程安全的类**。在多线程情况下，会出现异常，想必有经验的小伙伴也遇到过。下面我们就来分析分析SimpleDateFormat为什么不安全？是怎么引发的？以及多线程下有那些SimpleDateFormat的解决方案？

先看看**《阿里巴巴开发手册》**对于SimpleDateFormat是怎么看待的：

【强制】 SimpleDateFormat 是线程不安全的类,一般不要定义为 static 变量,如果定义为static ,必须加锁,或者使用 DateUtils 工具类。

正例:注意线程安全,使用 DateUtils 。亦推荐如下处理:

```
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() {
@ Override
protected DateFormat initialValue() {
return new SimpleDateFormat("yyyy-MM-dd");
}
};
```

说明:如果是 JDK 8 的应用,**可以使用 Instant 代替 Date , LocalDateTime 代替 Calendar ,DateTimeFormatter 代替 SimpleDateFormat** ,官方给出的解释: `simple beautiful strong immutable thread - safe` 。

一般我们使用SimpleDateFormat的时候会把它定义为一个**静态变量**，避免频繁创建它的对象实例，如下代码：

```
    public class SimpleDateFormatTest {
        private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        public static String formatDate(Date date) throws ParseException {
            return sdf.format(date);
        }
        public static Date parse(String strDate) throws ParseException {
            return sdf.parse(strDate);
        }
        public static void main(String[] args) throws InterruptedException, ParseException {
            System.out.println(sdf.format(new Date()));
        }
    }
```

是不是感觉没什么毛病？单线程下自然没毛病了，都是运用到多线程下就有大问题了。 测试下：

```
public static void main(String[] args) throws InterruptedException, ParseException {
        ExecutorService service = Executors.newFixedThreadPool(100);
        for (int i = 0; i < 20; i++) {
            service.execute(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        System.out.println(parse("2018-01-02 09:45:59"));
                    } catch (ParseException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        // 等待上述的线程执行完
        service.shutdown();
        service.awaitTermination(1, TimeUnit.DAYS);
    }
```

部分线程获取的时间不对，部分线程直接报 **java.lang.NumberFormatException:multiple points**错，线程直接挂死了。

**多线程不安全原因**

因为我们把SimpleDateFormat定义为静态变量，那么多线程下SimpleDateFormat的实例就会**被多个线程共享**，B线程会读取到A线程的时间，就会出现时间差异和其它各种问题。SimpleDateFormat和它继承的DateFormat类也不是线程安全的

来看看SimpleDateFormat的format()方法的源码

```
// Called from Format after creating a FieldDelegate
    private StringBuffer format(Date date, StringBuffer toAppendTo,
                                FieldDelegate delegate) {
        // Convert input date to time field list
        calendar.setTime(date);
        boolean useDateFormatSymbols = useDateFormatSymbols();
        for (int i = 0; i < compiledPattern.length; ) {
            int tag = compiledPattern[i] >>> 8;
            int count = compiledPattern[i++] & 0xff;
            if (count == 255) {
                count = compiledPattern[i++] << 16;
                count |= compiledPattern[i++];
            }
            switch (tag) {
            case TAG_QUOTE_ASCII_CHAR:
                toAppendTo.append((char)count);
                break;
            case TAG_QUOTE_CHARS:
                toAppendTo.append(compiledPattern, i, count);
                i += count;
                break;
            default:
                subFormat(tag, count, delegate, toAppendTo, useDateFormatSymbols);
                break;
            }
        }
        return toAppendTo;
    }
```

注意， calendar.setTime(date)，SimpleDateFormat的format方法实际操作的就是**Calendar**。

因为我们声明SimpleDateFormat为static变量，那么它的**Calendar**变量也就是一个共享变量，**可以被多个线程访问**。

假设线程A执行完calendar.setTime(date)，把时间设置成2019-01-02，这时候被挂起，线程B获得CPU执行权。线程B也执行到了calendar.setTime(date)，把时间设置为2019-01-03。线程挂起，线程A继续走，**calendar**还会被继续使用(subFormat方法)，而这时**calendar**用的是线程B设置的值了，而这就是引发问题的根源，出现时间不对，线程挂死等等。

其实SimpleDateFormat源码上作者也给过我们提示：

> Date formats are not synchronized. It is recommended to create separate format instances for each thread.If multiple threads access a format concurrently, it must be synchronized externally.

意思就是

> 日期格式不同步。 建议为每个线程创建单独的格式实例。 如果多个线程同时访问一种格式，则必须在外部同步该格式。

**解决方案**

1. 只在需要的时候创建新实例，不用static修饰

```
        public static String formatDate(Date date) throws ParseException {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return sdf.format(date);
        }
        public static Date parse(String strDate) throws ParseException {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return sdf.parse(strDate);
        }
```

如上代码，仅在需要用到的地方创建一个新的实例，就没有线程安全问题，不过也加重了创建对象的负担，会频繁地创建和销毁对象，效率较低。

2. synchronized

```
  private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    public static String formatDate(Date date) throws ParseException {
        synchronized(sdf){
            return sdf.format(date);
        }
    }
    public static Date parse(String strDate) throws ParseException {
        synchronized(sdf){
            return sdf.parse(strDate);
        }
    }
```

简单粗暴，synchronized往上一套也可以解决线程安全问题，缺点自然就是*并发量大的时候会对性能有影响，线程阻塞*。

3. ThreadLocal

   ```
   private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>() {
           @Override
           protected DateFormat initialValue() {
               return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
           }
       };
       public static Date parse(String dateStr) throws ParseException {
           return threadLocal.get().parse(dateStr);
       }
   
       public static String format(Date date) {
           return threadLocal.get().format(date);
       }
   ```

4. 基于JDK1.8的DateTimeFormatter

   ```
       public class SimpleDateFormatTest {
           private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
           public static String formatDate2(LocalDateTime date) {
               return formatter.format(date);
           }
   
           public static LocalDateTime parse2(String dateNow) {
               return LocalDateTime.parse(dateNow, formatter);
           }
   
           public static void main(String[] args) throws InterruptedException, ParseException {
               ExecutorService service = Executors.newFixedThreadPool(100);
               // 20个线程
               for (int i = 0; i < 20; i++) {
                   service.execute(() -> {
                       for (int j = 0; j < 10; j++) {
                           try {   
                           System.out.println(parse2(formatDate2(LocalDateTime.now())));
                           } catch (Exception e) {
                               e.printStackTrace();
                           }
                       }
                   });
               }
               // 等待上述的线程执行完
               service.shutdown();
               service.awaitTermination(1, TimeUnit.DAYS);
           }
       }
   ```

   DateTimeFormatter源码上作者也加注释说明了，**他的类是不可变的，并且是线程安全的**

### 面试题

1. 日期和时间：
   如何取得年月日、小时分钟秒？
   如何取得从1970年1月1日0时0分0秒到现在的毫秒数？
   如何取得某月的最后一天？
   如何格式化日期？

   答：
    问题1：创建java.util.Calendar 实例，调用其get()方法传入不同的参数即可获得参数所对应的值。Java 8中可以使用java.time.LocalDateTimel来获取，代码如下所示。

   ```
   public class DateTimeTest {
       public static void main(String[] args) {
           Calendar cal = Calendar.getInstance();
           System.out.println(cal.get(Calendar.YEAR));
           System.out.println(cal.get(Calendar.MONTH));    // 0 - 11
           System.out.println(cal.get(Calendar.DATE));
           System.out.println(cal.get(Calendar.HOUR_OF_DAY));
           System.out.println(cal.get(Calendar.MINUTE));
           System.out.println(cal.get(Calendar.SECOND));
   
           // Java 8
           LocalDateTime dt = LocalDateTime.now();
           System.out.println(dt.getYear());
           System.out.println(dt.getMonthValue());     // 1 - 12
           System.out.println(dt.getDayOfMonth());
           System.out.println(dt.getHour());
           System.out.println(dt.getMinute());
           System.out.println(dt.getSecond());
       }
   }
   ```

   问题2：以下方法均可获得该毫秒数。

   ```
   Calendar.getInstance().getTimeInMillis();
   System.currentTimeMillis();
   Clock.systemDefaultZone().millis(); // Java 8
   ```

   问题3：代码如下所示。

   ```
   Calendar time = Calendar.getInstance();
   time.getActualMaximum(Calendar.DAY_OF_MONTH);
   ```

   问题4：利用java.text.DataFormat 的子类（如SimpleDateFormat类）中的format(Date)方法可将日期格式化。Java 8中可以用java.time.format.DateTimeFormatter来格式化时间日期，代码如下所示。

   ```
   import java.text.SimpleDateFormat;
   import java.time.LocalDate;
   import java.time.format.DateTimeFormatter;
   import java.util.Date;
   
   class DateFormatTest {
   
       public static void main(String[] args) {
           SimpleDateFormat oldFormatter = new SimpleDateFormat("yyyy/MM/dd");
           Date date1 = new Date();
           System.out.println(oldFormatter.format(date1));
   
           // Java 8
           DateTimeFormatter newFormatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
           LocalDate date2 = LocalDate.now();
           System.out.println(date2.format(newFormatter));
       }
   }
   ```

   > 补充：Java的时间日期API一直以来都是被诟病的东西，为了解决这一问题，Java 8中引入了新的时间日期API，其中包括LocalDate、LocalTime、LocalDateTime、Clock、Instant等类，这些的类的设计都使用了不变模式，因此是线程安全的设计。

2. 打印昨天的当前时刻

   ```
   import java.util.Calendar;
   
   class YesterdayCurrent {
       public static void main(String[] args){
           Calendar cal = Calendar.getInstance();
           cal.add(Calendar.DATE, -1);
           System.out.println(cal.getTime());
       }
   }
   ```

   在Java 8中，可以用下面的代码实现相同的功能。

   ```
   import java.time.LocalDateTime;
   
   class YesterdayCurrent {
   
       public static void main(String[] args) {
           LocalDateTime today = LocalDateTime.now();
           LocalDateTime yesterday = today.minusDays(1);
   
           System.out.println(yesterday);
       }
   }
   ```