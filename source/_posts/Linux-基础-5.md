---
title: Linux-文本处理三剑客--grep
date: 2019-03-20 16:55:22
tags: 
 - Linux
category: 
 - Linux
 - 基础
---

### 前言

linux的思想之一就是一切皆文件，因此linux的所有的配置和状态都是可以通过文件来访问。因此，文本处理命令是我们必须要掌握的，这章我们将学习文本处理有关的常用命令，希望大家多多加油，补充自己。

<!--more-->

### 文本过滤

**grep**

grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

Unix的grep家族包括grep、egrep和fgrep。egrep和fgrep的命令只跟grep有很小不同。egrep是grep的扩展，支持更多的re元字符， fgrep就是fixed grep或fast grep，它们把所有的字母都看作单词，也就是说，正则表达式中的元字符表示回其自身的字面意义，不再特殊。linux使用GNU版本的grep。它功能更强，可以通过-G、-E、-F命令行选项来使用egrep和fgrep的功能。

grep的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。grep可用于shell脚本，因为grep通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。

命令格式：grep [option] pattern file

```shell
-A<显示行数> 或 --after-context=<显示行数> : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
-B<显示行数> 或 --before-context=<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前的内容。
-c 或 --count : 计算符合样式的列数。
-d <动作> 或 --directories=<动作> : 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
-e<范本样式> 或 --regexp=<范本样式> : 指定字符串做为查找文件内容的样式。
-E 或 --extended-regexp : 将样式为延伸的普通表示法来使用。
-f<规则文件> 或 --file=<规则文件> : 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
-F 或 --fixed-regexp : 将样式视为固定字符串的列表。
-G 或 --basic-regexp : 将样式视为普通的表示法来使用。
-h 或 --no-filename : 在显示符合样式的那一行之前，不标示该行所属的文件名称。
-H 或 --with-filename : 在显示符合样式的那一行之前，表示该行所属的文件名称。
-i 或 --ignore-case : 忽略字符大小写的差别。
-l 或 --file-with-matches : 列出文件内容符合指定的样式的文件名称。
-L 或 --files-without-match : 列出文件内容不符合指定的样式的文件名称。
-n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的列数编号。
-o 或 --only-matching : 只显示匹配PATTERN 部分。
-v 或 --revert-match : 显示不包含匹配文本的所有行。
-w 或 --word-regexp : 只显示全字符合的列。
-x --line-regexp : 只显示全列符合的列。
```

**匹配**

```
字符匹配：
	.:匹配任意单个字符
	[]:匹配指定范围的任意单个字符
	[^]:匹配指定范围外的任意单个字符
	[:digit:]、[:lower:]、[:upper:]、[:alpha:]、[:alnum:]、[:punct:]、[:space:]
匹配次数：用于指定次数的字符后，用于指定前面字符出现次数，贪婪模式
	*：匹配前面字符0或多次
	.*：任意长度的任意字符
	\?：匹配前面字符0次或1次
	\+：匹配前面字符至少1次
	\{m\}:匹配前面字符必须出现m次
	\{m,n\}:匹配前面字符次数至少m，至多n次
		\{m,\}:匹配前面字符至少m次
		\{0,m\}:匹配前面字符至多m次
位置锚定：
	^:行首，模式最左边
	$:行尾，模式最右边
	^PATTERN$:模式匹配整行
		^$:空行
		^[[:space:]]*$
	\< 或 \b:词首锚定，单词模式左侧
	\> 或 \b:词尾锚定，单词模式右侧
	\<PATTERN\>:匹配整个单词
	分组：\(\) 将多个字符捆绑在一起
	后向引用：引用前面的分组括号中模式所匹配字符，（而非模式本身）
		\NUM:第NUM个左括号与与之匹配到的字符
```

示例：

```shell
nl /etc/passwd | grep 'qmcui'   # 最好带引号

nl /etc/passwd | grep -v 'qmcui'

nl /etc/passwd | grep -w  "root"

nl /etc/passwd | grep  ROOT
nl /etc/passwd | grep -i ROOT


nl /etc/passwd | grep -ie 'Server' -e 'root' -e 'qmcui'
nl /etc/passwd | grep 'root\|qmcui'
nl /etc/passwd | grep -ie 'Server' -e 'root\|qmcui'
## 区别下面代码
# nl /etc/passwd | grep 'root\|qmcui'       # 转义成正则
nl /etc/passwd | grep 'root' | grep 'qmcui'

echo $PATH | grep ':'
echo $PATH | grep -o ':'

echo $PATH | sed 's/:/\n/g' | grep -A 1 '/sbin'
echo $PATH | sed 's/:/\n/g' | grep '/sbin' -A 1
echo $PATH | sed 's/:/\n/g' | grep '/sbin' -B 1
echo $PATH | sed 's/:/\n/g' | grep '/sbin' -C 1

nl /etc/passwd | grep  -c  'qmcui'
nl /etc/passwd | grep -ie 'Server' -e 'root\|qmcui' -c

nl /etc/passwd | grep -n 'qmcui'

# $ cat  match.txt      # cat >match.txt
# root
# qmcui
less /etc/passwd | grep -f match.txt

unalias grep
cat /etc/passwd | grep '^root'
cat /etc/passwd | grep --color=auto 'root'
nl /etc/passwd | grep --color=auto 'bash$'

# 进阶，不要求掌握！
nl /etc/passwd | grep -E 'root|qmcui'
grep  '[^a]bb'  1.txt       # 查bb前一个字符不为a的bb
grep   'a..b'  1.txt        # 开头为a 结尾为b 长度为4
cat /etc/passwd|grep --color=auto 'o\{2\}'
cat /etc/passwd|grep --color=auto 'o\{2,\}'
seq 20 | grep '[10]' 
```

练习：

```
练习：
	1、显示/etc/passwd文件中不以/bin/bash结尾的行；
		~]# grep -v "/bin/bash$" /etc/passwd
		
	2、找出/etc/passwd文件中的两位数或三位数；
		~]# grep  "\<[0-9]\{2,3\}\>"  /etc/passwd
		
	3、找出/etc/rc.d/rc.sysinit或/etc/grub2.cfg文件中，以至少一个空白字符开头，且后面非空白字符的行；
		~]# grep  "^[[:space:]]\+[^[:space:]]"  /etc/grub2.cfg
		
	4、找出"netstat -tan"命令的结果中以'LISTEN'后跟0、1或多个空白字符结尾的行；
		~]# netstat -tan | grep  "LISTEN[[:space:]]*$"
	
分组及引用
	\(\)：将一个或多个字符捆绑在一起，当作一个整体进行处理；
		\(xy\)*ab
		
	Note：分组括号中的模式匹配 到的内容会被正则表达式引擎自动记录于内部的变量中，这些变量为：
		\1：模式从左侧起，第一个左括号以及与之匹配的右括号之间的模式所匹配到的字符；
		\2：模式从左侧起，第二个左括号以及与之匹配的右括号之间的模式所匹配到的字符；
		\3
		...
	
			He loves his lover.
			He likes his lover.
			She likes her liker.
			She loves her liker.
			
			~]# grep  "\(l..e\).*\1"  lovers.txt
			
	后向引用：引用前面的分组括号中的模式所匹配到的字符；
```

**例1**

测试文件

```
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/bin/false,aaa,bbbb,cccc,aaaaaa
DADddd:x:2:2:daemon:/sbin:/bin/false
mail:x:8:12:mail:/var/spool/mail:/bin/false
ftp:x:14:11:ftp:/home/ftp:/bin/false
&nobody:$:99:99:nobody:/:/bin/false
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash
http:x:33:33::/srv/http:/bin/false
dbus:x:81:81:System message bus:/:/bin/false
hal:x:82:82:HAL daemon:/:/bin/false
mysql:x:89:89::/var/lib/mysql:/bin/false
aaa:x:1001:1001::/home/aaa:/bin/bash
ba:x:1002:1002::/home/zhangy:/bin/bash
test:x:1003:1003::/home/test:/bin/bash
@zhangying:*:1004:1004::/home/test:/bin/bash
policykit:x:102:1005:Po
```

a,匹配含有root的行

```
[root@krlcgcms01 test]# grep root test  
root:x:0:0:root:/root:/bin/bash  
```

b,匹配以root开头或者以zhang开头的行，注意反斜杠

```
[root@krlcgcms01 test]# cat test |grep '^\(root\|zhang\)'  
root:x:0:0:root:/root:/bin/bash  
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
```

c,匹配以root开头或者以zhang开头的行，注意反斜杠,根上面一个例子一样，-e默认是省去的

```
[root@krlcgcms01 test]# cat test |grep -e '^\(root\|zhang\)'  
root:x:0:0:root:/root:/bin/bash  
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
```

d,匹配以zhang开头，只含有字母

```
[root@krlcgcms01 test]# echo 'zhangying' |grep '^zhang[a-z]*$'  
zhangying  
```

e,匹配以bin开头的行,用的egrep，在这里可以换成-F,-G

```
[root@krlcgcms01 test]# cat test |grep -E '^bin'  
bin:x:1:1:bin:/bin:/bin/false,aaa,bbbb,cccc,aaaaaa  
```

f,在匹配的行前面加上该行在文件中，或者输出中所在的行号

```
[root@krlcgcms01 test]# cat test|grep -n zhangy  
7:zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
13:ba:x:1002:1002::/home/zhangy:/bin/bash  
15:@zhangying:*:1004:1004::/home/test:/bin/bash  
```

g,不匹配以bin开头的行,并显示行号

```
[root@krlcgcms01 test]# cat test|grep -nv '^bin'  
root:x:0:0:root:/root:/bin/bash
DADddd:x:2:2:daemon:/sbin:/bin/false
mail:x:8:12:mail:/var/spool/mail:/bin/false
ftp:x:14:11:ftp:/home/ftp:/bin/false
&nobody:$:99:99:nobody:/:/bin/false
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash
http:x:33:33::/srv/http:/bin/false
dbus:x:81:81:System message bus:/:/bin/false
hal:x:82:82:HAL daemon:/:/bin/false
mysql:x:89:89::/var/lib/mysql:/bin/false
aaa:x:1001:1001::/home/aaa:/bin/bash
ba:x:1002:1002::/home/zhangy:/bin/bash
test:x:1003:1003::/home/test:/bin/bash
@zhangying:*:1004:1004::/home/test:/bin/bash
policykit:x:102:1005:Po
```

h,显示匹配的个数，不显示内容

```
[root@krlcgcms01 test]#  cat test|grep -c zhang  
3  
```

i,匹配system，没有加-i没有匹配到东西。

```
[root@krlcgcms01 test]# grep  system test  
[root@krlcgcms01 test]# grep -ni  system test  
9:dbus:x:81:81:System message bus:/:/bin/false  
```

j,匹配zhan没有匹配到东西，匹配zhangy能匹配到，因为在test文件中，有zhangy这个单词

```
[root@krlcgcms01 test]#  cat test|grep -w zhan  
[root@krlcgcms01 test]#  cat test|grep -w zhangy  
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
ba:x:1002:1002::/home/zhangy:/bin/bash  
```

k,在这里-x后面东西，和输出中的整行相同时，才会输出

```
[root@krlcgcms01 test]# echo "aaaaaa" |grep -x aaa  
[root@krlcgcms01 test]# echo "aaaa" |grep -x aaaa  
aaaa  
```

l,最多只匹配一次，如果把-m 1去掉的话，会有三个

```
[root@krlcgcms01 test]# cat test |grep -m 1 zhang  
zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
```

m,匹配行的前面显示块号，这个块号是干什么的，不知道，有谁知道可否告诉我一下

```
[apacheuser@krlcgcms01 test]$ cat test |grep -b zha  
241:zhangy:x:1000:100:,,,:/home/zhangy:/bin/bash  
480:ba:x:1002:1002::/home/zhangy:/bin/bash  
558:@zhangying:*:1004:1004::/home/test:/bin/bash  
```

n,多文件匹配时，在匹配的行前面加上文件名

```
[apacheuser@krlcgcms01 test]$ grep -H 'root' test test2 testbak  
test:root:x:0:0:root:/root:/bin/bash  
test2:root  
testbak:root:x:0:0:root:/root:/bin/bash  
```

o,多文件匹配时，在匹配的行前面不加上文件名

```
[apacheuser@krlcgcms01 test]$ grep -h 'root' test test2 testbak  
root:x:0:0:root:/root:/bin/bash  
root  
root:x:0:0:root:/root:/bin/bash  
```

p,多文件匹配时，显示匹配文件的文件名

```
[apacheuser@krlcgcms01 test]$ grep -l 'root' test test2 testbak DAta  
test  
test2  
testbak  
```

q,没有-o时，有一行匹配，这一行里面有3个root，加上-o后，这个3个root就出来了

```
[apacheuser@krlcgcms01 test]$ grep  'root' test  
root:x:0:0:root:/root:/bin/bash  
[apacheuser@krlcgcms01 test]$ grep -o 'root' test  
root  
root  
root  
```

r,递归显示匹配的内容，在test目录下面建个mytest目录，copy test目录下面的test文件到mytest下面，能看到上面的结果

```
[root@krlcgcms01 test]# grep test -R /tmp/test/mytest  
/tmp/test/mytest/test:test:x:1003:1003::/home/test:/bin/bash  
/tmp/test/mytest/test:@zhangying:*:1004:1004::/home/test:/bin/bash  
```

s,显示匹配root后面的3行

```
[root@krlcgcms01 test]# cat test |grep -A 3 root  
root:x:0:0:root:/root:/bin/bash  
bin:x:1:1:bin:/bin:/bin/false,aaa,bbbb,cccc,aaaaaa  
daemon:x:2:2:daemon:/sbin:/bin/false  
mail:x:8:12:mail:/var/spool/mail:/bin/false  
```

**例2**

```
# 递归从所有文件中查询匹配的内容，文件名可不同

# grep -R C1079651000621  *   
20150727/503/20150701000104001317.xml:            C1079651000621
20150727/503/20150701000104001317.xml:            C1079651000621
20150727/503/20150701000104001333.xml:            C1079651000621
```

**例3**

```
# ifconfig |grep -Po "(?<=inet ).*(?= n)" 
10.23.121.78 
127.0.0.1 
```

#### egrep

支持扩展的正则表达式实现类似于grep文本过滤功能；grep -E

egrep [OPTIONS] PATTERN [FILE...]
选项：
-i, -o, -v, -q, -A, -B, -C
-G：支持基本正则表达式

扩展正则表达式的元字符：

```
字符匹配：
	.：任意单个字符
	[]：指定范围内的任意单个字符
	[^]：指定范围外的任意单个字符
		
次数匹配：
	*：任意次，0,1或多次；
	?：0次或1次，其前的字符是可有可无的；
	+：其前字符至少1次；
	{m}：其前的字符m次；
	{m,n}：至少m次，至多n次; 
		{0,n}
		{m,}
位置锚定
	^：行首锚定；
	$：行尾锚定；
	\<, \b：词首锚定；
	\>, \b：词尾锚定；
分组及引用：
	()：分组；括号内的模式匹配到的字符会被记录于正则表达式引擎的内部变量中；
	后向引用：\1, \2, ...
或：
	a|b：a或者b；
		C|cat：C或cat
		(c|C)at：cat或Cat
```

练习：

```
1、找出/proc/meminfo文件中，所有以大写或小写S开头的行；至少有三种实现方式；
	~]# grep -i "^s" /proc/meminfo
	~]# grep "^[sS]" /proc/meminfo
	~]# grep -E "^(s|S)" /proc/meminfo
	
2、显示肖前系统上root、centos或user1用户的相关信息；
	~]# grep -E "^(root|centos|user1)\>" /etc/passwd
	
3、找出/etc/rc.d/init.d/functions文件中某单词后面跟一个小括号的行；
	~]# grep  -E  -o  "[_[:alnum:]]+\(\)"  /etc/rc.d/init.d/functions
	
4、使用echo命令输出一绝对路径，使用egrep取出基名；
	~]# echo /etc/sysconfig/ | grep  -E  -o  "[^/]+/?$"
	
	进一步：取出其路径名；类似于对其执行dirname命令的结果；
	
5、找出ifconfig命令结果中的1-255之间的数值；
	~]# ifconfig | grep  -E  -o  "\<([1-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\>"
	
6、课外作业：找出ifconfig命令结果中的IP地址；
	
7、添加用户bash, testbash, basher以及nologin(其shell为/sbin/nologin)；而后找出/etc/passwd文件中用户名同shell名的行；
	~]# grep  -E  "^([^:]+\>).*\1$"  /etc/passwd
```

#### fgrep

不支持正则表达式元字符；
当无需要用到元字符去编写模式时，使用fgrep必能更好

#### 参考：

1. [https://blog.csdn.net/successdd/article/details/79088926](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fsuccessdd%2Farticle%2Fdetails%2F79088926)
2. [https://blog.csdn.net/shenhuan1104/article/details/75852822](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fshenhuan1104%2Farticle%2Fdetails%2F75852822)
3. [练习1](https://www.cnblogs.com/yajing-zh/p/4878273.html)
4. [练习2](https://www.cnblogs.com/datieli/p/10491024.html)
5. [练习3](https://www.jianshu.com/p/01be26fcfe4e)