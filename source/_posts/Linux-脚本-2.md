---
title: Linux-bash流程（二）
date: 2019-04-26 21:55:22
tags: 
 - Linux
 - 脚本
category: 
 - Linux
 - 脚本
---

### 运算符

#### 数值计算

<!--more-->

**整数运算工具**

1. 使用expr命令

   乘法操作应采用 \* 转义，避免被作为Shell通配符；参与运算的整数值与运算操作符之间需要以空格分开，引用变量时必须加$符号。

   首先定义变量X=1234，然后分别计算与78的加减乘除和求模运算结果：

   ```shell
   [root@svr5 ~]# X=1234  							//定义变量X
   [root@svr5 ~]# expr  $X  +  78  					//加法
   1312
   [root@svr5 ~]# expr  $X  -  78   					//减法
   1156
   [root@svr5 ~]# expr  $X  \*  78  					//乘法，操作符应添加\转义
   96252
   [root@svr5 ~]# expr  $X  /  78  					//除法，仅保留整除结果
   15
   [root@svr5 ~]# expr  $X  %  78 					//求模
   64
   ```

2. 使用[]或(())表达式
   乘法操作\*无需转义，运算符两侧可以无空格；引用变量可省略 $ 符号；计算结果替换表达式本身，可结合echo命令输出。
   同样对于变量X=1234，分别计算与78的加减乘除和求模运算结果：

   ```shell
   [root@svr5 ~]# X=1234   
   [root@svr5 ~]# echo $[X+78]
   1312
   [root@svr5 ~]# echo $[X-78]
   1156
   [root@svr5 ~]# echo $[X*78]
   96252
   [root@svr5 ~]# echo $[X/78]
   15
   [root@svr5 ~]# echo $[X%78]
   64
   ```

3. 使用let命令
   expr或[]、(())方式只进行运算，并不会改变变量的值；而let命令可以直接对变量值做运算再保存新的值。因此变量X=1234，在执行let运算后的值会变更；另外，let运算操作并不显示结果，但是可以结合echo命令来查看

   ```shell
   [root@svr5 ~]# X=1234  
   [root@svr5 ~]# let X+=78 ; echo $X
   1312
   [root@svr5 ~]# let X-=78 ; echo $X 
   1234
   [root@svr5 ~]# let X*=78 ; echo $X 
   96252
   [root@svr5 ~]# let X/=78 ; echo $X 
   1234
   [root@svr5 ~]# let X%=78 ; echo $X 
   64
   
   ```

**小数运算工具**

1. bc交互式运算
   先执行bc命令进入交互环境，然后再输入需要计算的表达式。以计算小数12.34与5.678的四则运算为例，相关操作如下：

   ```shell
   [root@svr5 ~]# bc 
   bc 1.06.95
   Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
   This is free software with ABSOLUTELY NO WARRANTY.
   For details type `warranty'. 
   12.34+56.78										//加法
   69.12
   12.34-56.78										//减法
   -44.44
   12.34*56.78										//乘法
   700.66
   12.34/56.78										//除法
   0
   quit  											//退出交互计算器
   [root@svr5 ~]#
   ```

2. bc非交互式运算
   将需要运算的表达式通过管道操作交给bc运算。注意，小数位的长度可采用scale=N限制，除此以外也受参与运算的数值的小数位影响。以计算小数12.34与5.678的四则运算为例，相关操作如下：

   ```shell
   [root@svr5 ~]# echo 'scale=4;12.34+5.678' | bc
   18.018
   [root@svr5 ~]# echo 'scale=4;12.34-5.678' | bc 
   6.662
   [root@svr5 ~]# echo 'scale=4;12.34*5.678' | bc 
   70.0665
   [root@svr5 ~]# echo 'scale=4;12.34/5.678' | bc 
   2.1733
   ```

#### 运算符

```
- 加法	`expr $a + $b` 结果为 30。
- 减法	`expr $a - $b` 结果为 -10。
- 乘法	`expr $a \* $b` 结果为  200。
/	除法	`expr $b / $a` 结果为 2。
%	取余	`expr $b % $a` 结果为 0。
=	赋值	a=$b 将把变量 b 的值赋给 a。
==	相等。用于比较两个数字，相同则返回 true。	[ $a == $b ] 返回 false。
!=	不相等。用于比较两个数字，不相同则返回 true。	[ $a != $b ] 返回 true。
 注意：条件表达式要放在方括号之间，并且要有空格，例如: [$a==$b] 是错误的，必须写成 [ $a == $b ]。
```

#### 关系运算符

```
-eq	检测两个数是否相等，相等返回 true。	[ $a -eq $b ] 返回 false。
-ne	检测两个数是否不相等，不相等返回 true。	[ $a -ne $b ] 返回 true。
-gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ] 返回 false。
-lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ] 返回 true。
-ge	检测左边的数是否大于等于右边的，如果是，则返回 true。	[ $a -ge $b ] 返回 false。
-le	检测左边的数是否小于等于右边的，如果是，则返回 true。	[ $a -le $b ] 返回 true。
```

实例演示：

```shell
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```

输出结果：

```
两个数相等！
```

#### 布尔运算符

```
!	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ] 返回 true。
-o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ] 返回 false。
```

另外，Shell还提供了与( -a )、或( -o )、非( ! )三个逻辑操作符用于将测试条件连接起来，其优先级为："!"最高，"-a"次之，"-o"最低。例如：

```shell
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

输出结果：

```
至少有一个文件存在!
```

#### 逻辑运算符

```
&&	逻辑的 AND	[[ $a -lt 100 && $b -gt 100 ]] 返回 false
||	逻辑的 OR	[[ $a -lt 100 || $b -gt 100 ]] 返回 true
```

#### 字符串运算符

```
=	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
!=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
-z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
-n	检测字符串长度是否为0，不为0返回 true。	[ -n "$a" ] 返回 true。
str	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。
```

实例演示：

```shell
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
```

输出结果：

```
两个字符串不相等!
```

#### 文件运算符

```
-b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。
-c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -c $file ] 返回 false。
-d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ] 返回 false。
-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。
-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] 返回 false。
-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。
-p file	检测文件是否是有名管道，如果是，则返回 true。	[ -p $file ] 返回 false。
-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。
-r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。
-w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。
-x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。
-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。
-e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。
```

实例演示：

```shell
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
```

输出结果：

```
文件已存在!
```

### 流程

if 条件判断在语言中最为常见，主要用于判断条件是否成立，比如在课堂上，并不是所有的学员都可以进入教室，而是必须符合条件（如必须是本班级学员）才能进入教室。当然，在上课时，是通过人的大脑进行判断的；如果在程序语言中，就要通过 if 条件判断语句来判断了。

#### 单分支if条件语句

单分支 if 条件语句最为简单，就是只有一个判断条件，如果符合条件则执行某个程序，否则什么事情都不做。语法如下：

```shell
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

在使用单分支 if 条件查询时需要注意几点：

- if 语句使用 fi 结尾，和一般语言使用大括号结尾不同。
- [条件判断式] 就是使用 test 命令判断，所以中括号和条件判断式之间必须有空格。

```shell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

单分支 if 条件语句非常简单，但是千万不要小看它，这是流程控制语句最基本的语法。而且在实现 Linux管理时，我们的管理脚本一般都不复杂，单分支 if 条件语句使用的概率还是很大的。

```bash
#!/bin/bash
# 使用单分支的if条件语句来判断/home/huanyu/shell/zz文件是否存在，若存在就结束条件判断和整个Shell脚本，反之则去创建这个目录
DIR="/tmp/zz"
if [ ! -e $DIR ]
then
        mkdir -p $DIR
fi
ll -d $DIR
```

#### 双分支if条件语句

语法格式：

```shell
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

示例

```bash
#!/bin/bash
#使用ping命令，做基本的网络连接状态判断

if [ $# -eq 0 ] ;then
    echo "usage: `basename ${0}` file "
    exit
fi

if [  ! -f $1  ] ;then
    echo "error file "
    exit
fi

echo "begin ping..."

#read -p "请输入ip或域名： " ip
for ip in  `cat ${1}`
do
    ping -c1 $ip &>/dev/null
    if [ $? -eq 0 ] ;then
        echo "$ip is up"
    else
        echo "$ip is down"
    fi
done

```

#### 多分支if条件语句

语法格式：

```shell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

实例：

```shell
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

输出结果：

```
a 小于 b
```

#### for循环

语法格式：

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

写成一行：

```shell
for var in item1 item2 ... itemN; do command1; command2… done;
```

当变量值在列表里，for循环即执行一次所有命令，使用变量名获取列表中的当前取值。命令可为任何有效的shell命令和语句。in列表可以包含替换、字符串和文件名。

in列表是可选的，如果不用它，for循环使用命令行的位置参数。

例如，顺序输出当前列表中的数字：

```shell
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

输出结果：

```
The value is: 1
The value is: 2
The value is: 3
The value is: 4
The value is: 5
```

顺序输出字符串中的字符：

```shell
for str in 'This is a string'
do
    echo $str
done
```

输出结果：

```
This is a string
```

示例：

```bash
#!/bin/bash
############################################
#user add  from file                                                                #
# v1.0 by pingxin 2019/10/07                                               #
############################################

#判断脚本是否有参数
if [ $# -eq 0 ]; then
    echo "usage: $(basename $0) file"
    exit 1
fi

#判断是否是文件
if [ ! -f $1 ]; then
    echo "error file"
    exit 2
fi
#for循环中参数之间使用空行、空格、tab进行分割
#希望for处理文件按回车分割，而不是空格或者tab空格
#重新定义分隔符
#IFS内部字段分隔符
IFS=$'\n'
for line in $(cat $1); do
    if [ ${#line} -eq 0 ];then
        echo "Nothing to do."
        continue
    fi
    echo ${line}
    user=`echo ${line}| awk  '{print $1}'`
    pass=`echo ${line}| awk  '{print $2}'`
    id ${user} &>/dev/null
    if [ $? -eq 0 ]; then
        echo "user ${user} already exists."
    else
        useradd ${user}
        echo "${pass}" | passwd --stdin ${user} &>/dev/null
        if [ $? -eq 0 ]; then
            echo "${user} is created."
        fi
    fi
done
```

#### while语句

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：

```shell
while condition
do
    command
done
```

以下是一个基本的while循环，测试条件是：如果int小于等于5，那么条件返回真。int从0开始，每次循环处理时，int加1。运行上述脚本，返回数字1到5，然后终止。

```shell
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

运行脚本，输出：

```
1
2
3
4
5
```

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<Ctrl-D\>结束循环。

```bash
#!/bin/bash
# 用while循环做一个猜价格游戏

PRICE=$(expr $RANDOM % 1000)  # 随机生成一个数，并把它赋给PRICE
TIMES=0   #用来计算猜测的次数
echo "商品实际价格在0－999之间，猜猜看是多少？"
while true
do
read -p "请输入你猜测的价格数：" INT
let TIMES++
if [ $INT -eq $PRICE ]
then
        echo "恭喜你猜对了，实际价格是：$PRICE"
        echo "你总共猜了 $TIMES 次。"
        exit 0
elif [ $INT -gt $PRICE ]
then
        echo "太高了"
else
        echo "太低了"

fi
done
```

**创建用户**

```bash
#!/bin/bash
############################################
# while更适合从文件逐行读取数据
# 批量创建用户                                                #
# v1.0 by pingxin 2019/10/07                                               #
############################################

#判断脚本是否有参数
if [ $# -eq 0 ]; then
    echo "usage: $(basename $0) file"
    exit 1
fi

#判断是否是文件
if [ ! -f $1 ]; then
    echo "error file"
    exit 2
fi

while read line; do
    if [ ${#line} -eq 0 ]; then
        echo "Nothing to do."
        continue
    fi
    echo ${line}
    user=$(echo ${line} | awk '{print $1}')
    pass=$(echo ${line} | awk '{print $2}')
    id ${user} &>/dev/null
    if [ $? -eq 0 ]; then
        echo "user ${user} already exists."
    else
        useradd ${user}
        echo "${pass}" | passwd --stdin ${user} &>/dev/null
        if [ $? -eq 0 ]; then
            echo "${user} is created."
        fi
    fi
done <$1

```

#### 无限循环

无限循环语法格式：

```shell
while :
do
    command
done
```

或者

```shell
while true
do
    command
done
```

或者

```shell
for (( ; ; ))
```

#### until 循环

until 循环执行一系列命令直至条件为 true 时停止。

until 循环与 while 循环在处理方式上刚好相反。

一般 while 循环优于 until 循环，但在某些时候—也只是极少数情况下，until 循环更加有用。

until 语法格式:

```shell
until condition
do
    command
done
```

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

以下实例我们使用 until 命令来输出 0 ~ 9 的数字：

```shell
#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

运行结果：

输出结果为：

```
0
1
2
3
4
5
6
7
8
9
```

#### case

case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。case语句格式如下：

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```

case工作方式如上所示。取值后面必须为单词in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 \* 捕获该值，再执行后面的命令。

```bash
#!/bin/bash
# 提示用户输入一个字符并将其赋值给变量KEY，然后根据变量KEY的值向用户显示其值是字母、数字还是其他字符

read -p "请输入一个字符，并按Enter键确认：" KEY
case "$KEY" in
        [a-z]|[A-Z])
                echo "你输入的是字母。"
                ;;
        [0-9])
                echo "你输入的是数字。"
                ;;
        *)
                echo "你输入的是 空格、功能键或是其他控制字符。"
esac
```

菜单示例

```bash
#!/bin/bash

menu() {
    cat <<-EOF
##########################
#    h. help                                        
#    f. disk partition                       
#    d. filesystem mount
#    m. memory                 
#    u. system load      
#    q. exit             
##########################
EOF
}
if [ ${UID} -ne 0 ]; then
    echo "must used by root"
    exit
fi
menu
while [ 2 -gt 1 ]; do
    read -p "Please input[h for help]: " action

    case "${action}" in
    h)
        menu
        ;;
    f)
        fdisk -l
        ;;

    d)
        df -Th
        ;;

    m)
        free -m
        ;;

    u)
        uptime
        ;;

    q)
        break
        ;;
    "") ;;
    *)
        echo "error choose"
        menu
        ;;

    esac
done
echo "finish .... "
```

#### 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。

**break命令**

break命令允许跳出所有循环（终止执行后面的所有循环）。

下面的例子中，脚本进入死循环直至用户输入数字大于5。要跳出这个循环，返回到shell提示符下，需要使用break命令。

```shell
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

执行以上代码，输出结果为：

```
输入 1 到 5 之间的数字:3
你输入的数字为 3!
输入 1 到 5 之间的数字:7
你输入的数字不是 1 到 5 之间的! 游戏结束
```

**continue**

continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

对上面的例子进行修改：

```shell
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

运行代码发现，当输入大于5的数字时，该例中的循环不会结束，语句 **echo "游戏结束"** 永远不会被执行。

##### 猜数字游戏

```bash
#!/bin/bash
END=0
until [ $END -ne 0 ]; do
   PRICE=$(expr $RANDOM % 1000) #随机生成一个数，并把它赋给PRICE
   TIMES=10                     #用来计算猜测的次数
   LIMIT=10                     #最大次数
   WIN=0                        #是否赢
   echo "商品实际价格在0－999之间，猜猜看是多少？"
   while [ $TIMES -gt 0 ]; do
      echo "剩余次数： ${TIMES}"
      read -p "请输入你猜测的价格数：" INT
      let TIMES--
      if [ $INT -eq $PRICE ]; then
         WIN=1
         break
      elif [ $INT -gt $PRICE ]; then
         echo "太高了"
      else
         echo "太低了"
      fi
   done
   if [ $WIN -eq 0 ]; then
      echo "失败，实际价格为: ${PRICE}"
   else
      echo "恭喜你猜对了，实际价格是：$PRICE"
      echo "你总共猜了 $((LIMIT - TIMES)) 次。"
   fi
   read -p "是否退出游戏？(Y/N)  " action
   case "${action}" in
   y | Y | yes | YES)
      echo "BYE BYE"
      exit 0
      ;;
   *)
      echo "good"
      exit 2
      ;;
   esac

done

```

### 函数

linux shell 可以用户定义函数，然后在shell脚本中可以随便调用。

shell中函数的定义格式如下：

```shell
[ function ] funname [()]

{

    action;

    [return int;]

}
```

说明：

- 1、可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。
- 2、参数返回，可以显示加：return 返回

实例：

```shell
#!/bin/bash
demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

输出结果：

```
-----函数开始执行-----
这是我的第一个 shell 函数!
-----函数执行完毕-----
```

下面定义一个带有return语句的函数：

```shell
#!/bin/bash
funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

输出类似下面：

```
这个函数会对输入的两个数字进行相加运算...
输入第一个数字: 
1
输入第二个数字: 
2
两个数字分别为 1 和 2 !
输入的两个数字之和为 3 !
```

函数返回值在调用该函数后通过 $? 来获得。

> 注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

#### 函数参数

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

带参数的函数示例：

```
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

输出结果：

```
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```

注意，$10 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用${n}来获取参数。

另外，还有几个特殊字符用来处理参数：

| 参数处理 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数                       |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。         |
| $-       | 显示Shell使用的当前选项，与set命令功能相同。                 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

### shell后台并发执行

#### shell如何在后台执行

1. nohup命令
   通常我们都是远程登录linux终端，而当我们退出终端时在之前终端运行的程序都会终止，有时候先想要退出终端也要程序继续执行这时nohup就登场了。nohup命令可以将程序以忽略挂起信号的方式运行起来，被运行的程序的输出信息将不会显示到终端。

   ```
   nohup command > myout.file 2>&1 &
   ```

2. &后台执行
   在命令后面加 & 可以让程序在后台执行

   ```
   command &
   ```

3. Ctrl + z
   当一个程序正在执行并且占用当前终端时我们同时按下 Ctrl + z ,这样就会把正在执行的前台程序放到后台挂起。

#### 常规并发执行

1. 正常执行

   ```bash
   #!/bin/bash
   Njob=15    #任务总数
   for ((i=0; i<$Njob; i++)); do
   {
             echo  "progress $i is sleeping for 3 seconds zzz…"
             sleep  3
   }
   done
   echo -e "time-consuming: $SECONDS    seconds"    #显示脚本执行耗时
   ```

2. 并发后台执行

   ```bash
   #!/bin/bash
   Njob=15
   for ((i=0; i<$Njob; i++)); do
             echo  "progress $i is sleeping for 3 seconds zzz…"
             sleep  3 &       #循环内容放到后台执行
   done
   wait      #等待循环结束再执行wait后面的内容
   echo -e "time-consuming: $SECONDS    seconds"    #显示脚本执行耗时
   ```

   以这种方式进行后台执行执行的效率较高、速度快，但是当并发数过多时就有可能会造成系统奔溃。

3. 以队列方式并发后台执行

   就上面的并发执行的缺点，我们可以分批并行的方式并发执行。每批的执行进程数固定并不会引起系统奔溃。

   ```bash
   #!/bin/bash
   NQ=3
   num=5
   for ((i = 0; i < $NQ; i++)); do
       for ((j = 0; j < $num; j++)); do
           echo  "progress $i is sleeping for 3 seconds zzz…"
           sleep 3 &
       done
       wait
   done
   #等待循环结束再执行wait后面的内容
   echo -e "time-consuming: $SECONDS    seconds"    #显示脚本执行耗时
   ```

#### 最佳实践

以队列的形式并发执行固然很好，每次并发的进程是可控的，可以提高效率还能防止系统奔溃。但是存在一个问题就是每一批的并发进程的执行时间是由这些进程里面执行最慢的决定，先前执行完的进程要等待没有执行完的进程。下面介绍并发执行的最佳实践

```bash
#!/bin/bash
# 并发运行的最佳实践
 
# 总进程数
Sp=15
# 并发数,并发数过大可能造成系统崩溃
Qp=5
# 存放进程的队列
Qarr=();
# 运行进程数
run=0
# 将进程的添加到队列里的函数
function push() {
	Qarr=(${Qarr[@]} $1)
	run=${#Qarr[@]}
}
# 检测队列里的进程是否运行完毕
function check() {
	oldQ=(${Qarr[@]})
	Qarr=()
	for p in "${oldQ[@]}";do
		if [[ -d "/proc/$p" ]];then
			Qarr=(${Qarr[@]} $p)	
		fi
	done
	run=${#Qarr[@]}
}
 
# main
for((i=0; i<$Sp; i++));do
	echo "running $i " 
	sleep 3 &
	push $!
	while [[ $run -gt $Qp ]];do
		check
		sleep 0.1
	done
done
echo -e "time-consuming: $SECONDS   seconds"    #显示脚本执行耗时
```

大家可以看见同样是可控并发数量，总数量同为15个进程，每次都并发执行5个进程，效率提高了进30%。