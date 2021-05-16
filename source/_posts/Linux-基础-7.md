---
title: Linux-文本处理三剑客--awk
date: 2019-03-20 16:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

AWK是一种处理文本文件的语言，是一个强大的文本分析工具。之所以叫AWK是因为其取了三位创始人 Alfred Aho，Peter Weinberger, 和 Brian Kernighan 的Family Name的首字符。
awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

<!--more-->

**参数**

```
awk [option] '{pattern + action}' {filenames} # sometims muti file is ok
awk [option] 'BEGIN{初始代码} {循环代码} END{最后代码}' filename
```

**常用基本用法总览**

```bash
echo 1 2 3 |awk '{ print "total pay for", $1, "is", $2 * $3 }'
# awk '{ printf("total pay for %s is $%.2f\n", $1, $2 * $3) }'

# awk -F  ：-F的意思就是指定分隔符
echo $PATH|awk -F ':' '{print $1}'
# awk -F '[;:]' 指定多个分隔符
# -F相当于内置变量FS, 指定分割字符
cat /etc/passwd |awk  -F ':'  '{print $1"\t"$7}'
cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print $1","$7} END {print "blue,/bin/nosh"}'
awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd
# less -S tmp.txt |awk 'BEGIN{sum=3}{sum=sum+1}END{print sum}'
awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd

awk '$3 == 0 { print $1 }' emp.data
#  less -S /public/reference/gtf/ensembl/homo_sapiens_86/Homo_sapiens.GRCh38.86.gtf.gz|awk '$3=="gene"{print $0}'|less -S
# 根据gtf测试文本来提取区域
# less -S /public/reference/gtf/ensembl/homo_sapiens_86/Homo_sapiens.GRCh38.86.gtf.gz|awk '$4>20000 {print $0}'|less -S
# less -S /public/reference/gtf/ensembl/homo_sapiens_86/Homo_sapiens.GRCh38.86.gtf.gz|awk '$1=="1" && $4>20000 && $5<30000 {print $0}'

awk '$3 >0 { print $1, $2 * $3 }' emp.data
awk '$3 > 15 { num = num + 1 } END{ print num" lines" }'
awk 'NF != 3     { print $0, "number of fields is not equal to 3" }'  # 与 && ， 或 || ， 以及非 !
awk '$2 < 3.35   { print $0, "rate is below minimum wage" }'  # Eg: !($2 < 4 && $3 < 20)
# 体会内置变量 NR NF
nl /etc/passwd |awk 'END { print NR, "employees" }'
nl /etc/passwd |awk '{print NF}'
# 体会赋值
cat /etc/passwd |awk -v FS=":" '{print NF}'


awk     '{ pay = pay + $2 * $3 }
END { print NR, "employees"
      print "total pay is", pay
      print "average pay is", pay/NR
    }'
```

#### 匹配

```
awk '/LISTEN/' netstat.txt
awk '/LISTEN/' netstat.txt
awk '$6 ~ /FIN|TIME/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt
awk '$6 !~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt
awk '!/WAIT/' netstat.txt
awk 'NR!=1{print > $6}' netstat.txt

$ awk 'NR!=1{if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
else if($6 ~ /LISTEN/) print > "2.txt";
else print > "3.txt" }' netstat.txt

$ awk 'NR!=1{a[$6]++;} END {for (i in a) print i ", " a[i];}' netstat.txt
```

**示例**

log.txt文本内容如下：

```
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

实例：

```shell
# 使用","分割
 $  awk -F, '{print $1,$2}'   log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 或者使用内建变量
 $ awk 'BEGIN{FS=","} {print $1,$2}'     log.txt
 ---------------------------------------------
 2 this is a test
 3 Are you like awk
 This's a test
 10 There are orange apple
 # 使用多个分隔符.先使用空格分割，然后对分割结果再使用","分割
 $ awk -F '[ ,]'  '{print $1,$2,$5}'   log.txt
 ---------------------------------------------
 2 this test
 3 Are awk
 This's a
 10 There apple
```

#### 运算符

| 运算符                  | 描述                             |
| ----------------------- | -------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ ~!                    | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加，减                           |
| * / %                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

重点：记忆匹配~，> ,==判断

**示例**

过滤第一列大于2的行

```ruby
$ awk '$1>2' log.txt    #命令
#输出
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

过滤第一列大于2并且第二列等于'Are'的行

```ruby
$ awk '$1>2 && $2=="Are" {print $1,$2,$3}' log.txt    #命令
#输出
3 Are you
```

#### 内建变量

赋值时结合-v一起使用

| 常见变量 | 描述                                                       |
| -------- | ---------------------------------------------------------- |
| $n       | 当前记录的第n个字段，字段间由FS分隔                        |
| $0       | 完整的输入记录                                             |
| FS       | 字段分隔符(默认是任何空格)                                 |
| OFS      | 输出记录分隔符（输出换行符），输出时用指定的符号代替换行符 |
| NF       | 一条记录的字段的数目                                       |
| NR       | 已经读出的记录数，就是行号，从1开始                        |
| RS       | 记录分隔符(默认是一个换行符)                               |
| ORS      | 输出记录分隔符(默认值是一个换行符)                         |

其他变量

| 变量        | 描述                                              |
| ----------- | ------------------------------------------------- |
| ARGC        | 命令行参数的数目                                  |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                 |
| ARGV        | 包含命令行参数的数组                              |
| CONVFMT     | 数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组 |
| ERRNO       | 最后一个系统错误的描述                            |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                        |
| FILENAME    | 当前文件名                                        |
| FNR         | 各文件分别计数的行号                              |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配                  |
| OFMT        | 数字的输出格式(默认值是%.6g)                      |
| RLENGTH     | 由match函数所匹配的字符串的长度                   |
| RSTART      | 由match函数所匹配的字符串的第一个位置             |
| SUBSEP      | 数组下标分隔符(默认值是/034)                      |

使用方法

```shell
awk -v  # 设置变量
```

示例：

```linux
seq 1 10|awk 'BEGIN{ ORS=" " }{ print $0 }'
1 2 3 4 5 6 7 8 9 10 $

seq 1 10|awk 'BEGIN{ RS="\t" }{ print $0"OK" }'
1
2
3
4
5
6
7
8
9
10
OK

seq 1 10|awk 'BEGIN{ RS="\n" }{ print $0"OK" }'
1OK
2OK
3OK
4OK
5OK
6OK
7OK
8OK
9OK
10OK
```

实例：

```ruby
$ awk -va=1 '{print $1,$1+a}' log.txt
 ---------------------------------------------
 2 3
 3 4
 This's 1
 10 11
$ awk 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"} {printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}'  log.txt
FILENAME ARGC  FNR   FS   NF   NR  OFS  ORS   RS
---------------------------------------------
log.txt    2    1         5    1
log.txt    2    2         5    2
log.txt    2    3         3    3
log.txt    2    4         4    4
$ awk -F\' 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"} {printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}'  log.txt
FILENAME ARGC  FNR   FS   NF   NR  OFS  ORS   RS
---------------------------------------------
log.txt    2    1    '    1    1
log.txt    2    2    '    1    2
log.txt    2    3    '    2    3
log.txt    2    4    '    1    4
# 输出顺序号 NR, 匹配文本行号
$ awk '{print NR,FNR,$1,$2,$3}' log.txt
---------------------------------------------
1 1 2 this is
2 2 3 Are you
3 3 This's a test
4 4 10 There are
# 指定输出分割符
$  awk '{print $1,$2,$5}' OFS=" $ "  log.txt
---------------------------------------------
2 $ this $ test
3 $ Are $ awk
This's $ a $
10 $ There $
```

#### 正则匹配

```csharp
# 输出第二列包含 "th"，并打印第二列与第四列
$ awk '$2 ~ /th/ {print $2,$4}' log.txt
---------------------------------------------
this a
```

**~ 表示模式开始。// 中是模式。**

```ruby
# 输出包含"re" 的行
$ awk '/re/' log.txt
---------------------------------------------
3 Are you like awk
10 There are orange,apple,mongo
```

不匹配：

![awk '](https://math.jianshu.com/math?formula=awk%20%27)2 !~ /th/ {print ![2,](https://math.jianshu.com/math?formula=2%2C)4}' log.txt

#### 忽略大小写

```bash
$ awk 'BEGIN{IGNORECASE=1} /this/' log.txt
---------------------------------------------
2 this is a test
This's a test
```

#### awk脚本

关于awk脚本，我们需要注意两个关键词BEGIN和END。

- BEGIN{ 这里面放的是执行前的语句 }
- END {这里面放的是处理完所有的行后要执行的语句 }
- {这里面放的是处理每一行时要执行的语句}

假设有这么一个文件（学生成绩表）：

```ruby
$ cat score.txt
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62
```

我们的awk脚本如下：

```bash
$ cat cal.awk
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
 
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}
```

我们来看一下执行结果：

```shell
$ awk -f cal.awk score.txt
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
```

#### 另外一些实例

AWK的hello world程序为：

```shell
BEGIN { print "Hello, world!" }
```

计算文件大小

```shell
$ ls -l *.txt | awk '{sum+=$6} END {print sum}'
--------------------------------------------------
666581
```

从文件中找出长度大于80的行

```shell
awk 'length>80' log.txt
```

打印九九乘法表

```shell
seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'
```

##### 彩蛋

awk相当于一个语言，它能实现判断句等其他语言共有的特点
**if-else语句**

如下程序将计算时薪超过6美元的员工的总薪酬与平均薪酬。它使用一个 if 来防范计算平均薪酬时的零除问题

```shell
$2 > 6 { n = n + 1; pay = pay + $2 * $3 }
END    { if (n > 0)
            print n, "employees, total pay is", pay,
                     "average pay is", pay/n
         else
             print "no employees are paid more than $6/hour"
        }
    { line[NR] = $0 }  # 记下每个输入行

END { i = NR           # 逆序打印
      while (i > 0) {
        print line[i]
        i = i - 1
      }
    }
```

#### 进阶函数

###### split

```shell
echo ‘abcd’ | awk ‘{len=split($0,a,””);for(i=1;i<=len;i++)print “a[“i”]=”a[i];print “length=”len}’
# a[1]=a  a[2]=b  a[3]=c  a[4]=d  length=4
awk '{split($2,a,"-");if(a[2]==01){b[$1]+=$4}}END{for(i in b)print i,b[i]}' test.txt 

ipstr="192.168.1.2,192.168.1.3"
awk 'BEGIN{split('"\"$ipstr\""',a,",");for(i in a)print "sa["i"]="a[i]}'

cat config |awk -v FS="\t" '{split($0,a);split(a[1],b,"/"); print $0","b[5]}'


netstat | awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}'
awk '$3==0 && $6=="LISTEN" || NR==1 '
```

##### 示例

https://coolshell.cn/articles/9070.html

https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_72/com.ibm.aix.cmds1/awk.htm

##### 函数示例多

http://linuxcommand.org/lc3_adv_awk.php

##### 学习//匹配

https://www.cnblogs.com/sunada2005/p/3493941.html