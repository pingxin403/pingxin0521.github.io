---
title: Linux-文件查找和压缩
date: 2019-03-21 21:55:22
tags:
 - Linux
category:
 - Linux
 - 基础
---

### 前言

前面我们介绍了文本处理命令，本章我们就来学一下文件查找。

<!--more-->

在文件系统上查找符合条件的文件，linux上常用的实现工具有locate和find。其中locate是基于事先建立的索引库，并不能实时查找，使用模糊查找，查找速度快；find是实时查找工具，通过遍历指定起始路径下文件系统层级结构完成文件查找，特点是查找速度略慢、精确查找、实时查找。

### locate

 **locate**(locate) 命令用来查找文件或目录(是mlocate的命令)。 locate命令要比find -name快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db 。这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。整个locate工作其实是由四部分组成的:

1. /usr/bin/updatedb   主要用来更新数据库，通过crontab自动完成的

2. /usr/bin/locate         查询文件位置

3. /etc/updatedb.conf   updatedb的配置文件

   ```
   PRUNE_BIND_MOUNTS = "yes"
   PRUNEFS = "9p afs anon_inodefs auto autofs bdev binfmt_misc cgroup cifs coda configfs cpuset debugfs devpts ecryptfs exofs fuse fuse.sshfs fusectl gfs gfs2 gpfs hugetlbfs inotifyfs iso9660 jffs2 lustre mqueue ncpfs nfs nfs4 nfsd pipefs proc ramfs rootfs rpc_pipefs securityfs selinuxfs sfs sockfs sysfs tmpfs ubifs udf usbfs fuse.glusterfs ceph fuse.ceph"
   PRUNENAMES = ".git .hg .svn"
   PRUNEPATHS = "/afs /media /mnt /net /sfs /tmp /udev /var/cache/ccache /var/lib/yum/yumdb /var/spool/cups /var/spool/squid /var/tmp /var/lib/ceph"
   
   #PRUNE_BIND_MOUNTS = "yes"'
   #开启搜索限制
   #PRUNEFS = 
   #搜索时，不搜索的文件系统
   #PRUNENAMES=
   #搜索时，不搜索的文件类型
   #PRUNEPATHS=
   #搜索时，不搜索的路径
   ```

4. /var/lib/mlocate/mlocate.db  存放文件信息的文件

语法：`locate [options]... [PATTERN]...`

选项：

```
-b：只匹配路径中的基名；
-c：统计出共有多少个符合条件的文件；
-i, --ignore-case      忽略大小写
-r, --regexp REGEXP    使用基本正则表达式
      --regex            使用扩展正则表达式
```

注意：索引构建过程需要遍历整个根文件系统，极消耗资源；

### find

因为Linux下面一切皆文件，经常需要搜索某些文件来编写，所以对于linux来说find是一条很重要的命令。linux下面的find指令用于在目录结构中搜索文件，并执行指定的操作。它提供了相当多的查找条件，功能很强大。在不指定查找目录的情况下，find会在对整个系统进行遍历。即使系统中含有网络文件系统，find命令在该文件系统中同样有效。 在运行一个非常消耗资源的find命令时，很多人都倾向于把它放在后台执行，因为遍历一个大的文件系统可能会花费很长的时间。

语法：`find [查找目录] [查找规则] [查找完后的操作] `

即：`find pathname -option [-print -exec -ok …]`

命令参数
（1）pathname：表示所要查找的目录路径,例如”.”表示当前目录，”/”表示根目录。
（2）-print:将find找到的文件输出到标准输出。
（3）-exec:对找到的文件执行exec这个参数所指定的shell命令，相应的形式为：-exec command {} \; 将查到的文件进行command操作，”{}”就代替查到的文件。

注意：
1）”{}”和”\”之间有一个空格。

2）-ok:和-exec的作用相同，只不过-ok更加安全一点，在执行每一个命令之前，系统会让用户确定是否执行。

**(1). 最基础的打印操作**

find命令默认接的命令是-print，它默认以\n将找到的文件分隔。可以使用-print0来使用\0分隔，这样就不会分行了。但是一定要注意，-print0针对的是\n转\0，如果查找的文件名本身就含有空格，则find后-print0仍然会显示空格文件。所以-print0实现的是\n转\0的标记，可以使用其他工具将\0标记替换掉，如xargs，tr等。

```
[root@xuexi tmp]# mkdir /tmp/a
[root@xuexi tmp]# touch /tmp/a/{1..5}.log
[root@xuexi tmp]# find /tmp/a   # 等价于find /tmp/a  -print，表示搜索/tmp/a目录
/tmp/a
/tmp/a/4.log
/tmp/a/2.log
/tmp/a/5.log
/tmp/a/1.log
/tmp/a/3.log
[root@xuexi tmp]# find /tmp/a -print0    
/tmp/a/tmp/a/4.log/tmp/a/2.log/tmp/a/5.log/tmp/a/1.log/tmp/a/3.log
```

**(2).文件名搜索：-name或-path**

-name可以对文件的basename进行匹配，-path可以对文件的dirname+basename。查找的文件名最好使用引号包围，可以配合通配符进行查找。

```
[root@xuexi tmp]# find /tmp -name "*.log"
/tmp/screen.log
/tmp/x.log
/tmp/timing.log
/tmp/a/4.log
/tmp/a/2.log
/tmp/a/5.log
/tmp/a/1.log
/tmp/a/3.log
/tmp/b.log
```

但不能在-name的模式中使用"/"，除非文件名中包含了字符"/"，否则将匹配不到任何东西，因为-name只对basename进行匹配。例如，想要匹配/tmp目录下某包含字符a的目录下的log文件。

```
shell> find /tmp -name '*a*/*.log'
find: warning: Unix filenames usually don't contain slashes (though pathnames do).  That means that '-name ‘*a*/*.log’' will probably evaluate to
false all the time on this system.  You might find the '-wholename' test more useful, or perhaps '-samefile'.  Alternatively, if you are using GNU grep,
 you could use 'find ... -print0 | grep -FzZ ‘*a*/*.log’'.
```

所以想要在指定目录下搜索某目录中的某文件，应该使用-path而不是-name。

```
[root@server2 tmp]# find /tmp -path '*a*/*.log'
/tmp/abc/axyz.log
```

注意，配合通配符[]时应该注意是基于字符顺序的，大小写字母的顺序是a-z --> A-Z，指定[a-z]表示小写字母a-z，同理[A-Z]，而[a-zA-Z]和[a-Z]都表示所有大小写字母。当然还可以指定[a-A]表示a-z外加一个A。

字母的处理顺序较容易理解，关于数字的处理方法，见下面的示例。

```
[root@xuexi test]# ls
11.sh  1.sh  22.sh  2.sh  3.sh
[root@xuexi test]# find -name "[1-2].sh"
./2.sh
./1.sh
[root@xuexi test]# find -name "[1-23].sh"
./2.sh
./3.sh
./1.sh
[root@xuexi test]# touch 0.sh
[root@xuexi test]# find -name "[1-20].sh"
./2.sh
./0.sh
./1.sh
[root@xuexi test]# find -name "[1-22-3].sh"
./2.sh
./3.sh
./1.sh
```

从上面结果可以看出，其实[]只能匹配单个字符，[0-9]表示0-9的数字，[1-20]表示[1-2]外加一个0，[1-23]表示[1-2]外加一个3，[1-22-3]表示[1-2]或[2-3]，迷惑点就是看上去是大于10的整数，其实是两个或者更多的单个数字组合体。也可以用这种方法表示多种匹配：[1-2,2-3]。

**(3). 根据文件类型搜索：-type**

一般需要搜索的文件类型就只有普通文件(f)，目录(d)，链接文件(l)。

例如，搜索普通文件类的文件，且名称为a开头的sh文件。

[root@xuexi test]# find /tmp -type f -name "a*.sh"

搜索目录类文件，且目录名以a开头。

[root@xuexi test]# find /tmp -type d -name "a*"

**(4). 根据文件的时间戳搜索：-atime/-mtime/-ctime**

例如搜索/tmp下3天内修改过内容的sh文件，因为是文件内容，所以不考虑搜索目录。

find /tmp -type f -mtime -3 -name "*.sh"

至于为什么是"-3"，见后面find理论部分内容。

**(5). 根据文件大小搜索：-size**

例如搜索/tmp下大于100K的sh文件。

find /tmp -type f -size +100k -name '*.sh'

**(6). 根据权限搜索：-perm**

例如搜索/tmp下所有者具有可读可写可执行权限的sh文件。

find /tmp -type f -perm -0700 -name '*.sh'

**(7). 搜索空文件**

空文件可以是没有任何内容的普通文件，也可以是没有任何内容的目录。

例如搜索目录中没有文件的空目录。

find /tmp -type d -empty

**(8). 搜索到文件后并删除**

例如搜索到/tmp下的".tmp"文件然后删除。

find /tmp -type f -name "*.tmp" -exec rm -rf '{}'\;

但是这是极不安全的方法，因为如果文件名有空白字符的话，会造成误删除，例如文件名为"a xy.tmp"，则直接-exec rm -rf '{}'将会删除a和xy.tmp和"a xy.tmp"，也就是说a这个文件或目录被误删除了。

**(9). 搜索指定日期范围的文件，例如搜索/test下2017-06-03到2017-06-06之间修改过的文件**

```
find /test -type f -newermt 2017-06-03 -a ! -newermt 2017-06-06
```

或者，创建两个临时文件，并用touch修改这两个文件的修改时间，然后`find -newer`去参照这两个文件

```
touch -m -d 2017-06-03 tmp1.txt
touch -m -d 2017-06-06 tmp2.txt
find /test -type f -newer tmp1.txt -a ! -newer tmp2.txt
```

不过这样会把tmp2.txt也搜索出来，因为newer搜索的是比xxx文件更新，取反则表示更旧或时间相同。

**(10).并行加速搜索**

有时候，想要搜索的内容并不知道在哪里，这时我们会从根"/"开始搜索，这样的搜索速度可能会稍微长那么一点点。为了加速搜索，使用xargs的并行功能。例如，搜索"/"下的所有"Find.pm"结尾的文件：

```
ls --hide proc / | xargs -i -P 0 find /{} -type f -name "*Find.pm"
```

可以使用time命令看看cpu利用率：149%

```shell
$ /usr/bin/time bash -c 'ls --hide proc / | xargs -i -P 0 find /{} -type f -name "*Find.pm"'
/perlapp/perl-5.26.2/cpan/Pod-Parser/lib/Pod/Find.pm
/perlapp/perl-5.26.2/ext/File-Find/lib/File/Find.pm
/usr/share/perl5/vendor_perl/Pod/Find.pm
/usr/share/perl5/File/Find.pm
0.04user 0.25system 0:00.19elapsed 149%CPU (0avgtext+0avgdata 5492maxresident)k
0inputs+0outputs (0major+12685minor)pagefaults 0swaps
```



find首先对整个命令行进行语法解析，并应用给定的options，然后定位到搜索路径path下开始对路径下的文件或子目录进行表达式评估或测试，评估或测试的过程是按照表达式的顺序从左向右进行的(此处不考虑操作符的影响)，如果最终表达式的表达式评估为true，则输出(默认)该文件的全路径名。

对于find来说，一个非常重要的概念：find的搜索机制是根据表达式返回的true/false决定的，每搜索一次都判断一次是否能确定最终评估结果为true，只有评估的最终结果为true才算是找到，并切入到下一个搜索点。

**expression operators**

操作符控制表达式运算方式。确切的说，是控制expression中的options/tests/actions的运算方式，无论是options、tests还是actions，它们都可以给定多个，例如find /tmp -type f -name "*.log" -exec ls '{}' \; -print，该find中给定了两个test，两个action，它们之间从前向后按顺序进行评估，所以如果想要改变运算逻辑，需要使用操作符来控制。

注意，理解and和or的评估方式非常重要，很可能写在and或or后面的表达式不起作用，而导致跟想象中的结果不一样。

下面的操作符优先级从高到低。

```
------------------------------------------------------------------------------------
( expr )         ：优先级最高。为防止括号被shell解释(进入子shell)，所以需要转义，即\(...\)
------------------------------------------------------------------------------------
! expr           ：对expr的true和false结果取反。同样需要使用引号包围
------------------------------------------------------------------------------------
-not expr        ：等价于"! expr"
------------------------------------------------------------------------------------
expr1 expr2      ：等同于and操作符。
------------------------------------------------------------------------------------
expr1 -a expr2   ：等同于and操作符。
------------------------------------------------------------------------------------
expr1 -and expr2 ：首先要求expr1为true，然后expr2以expr1搜索的结果为基础继续检测，然后再返回
                 ：检测值为true的文件。因为expr2是以expr1结果为基础的，所以如果expr1返回
                 ：false，则expr2直接被忽略而不会进行任何操作
------------------------------------------------------------------------------------
expr1 -o expr2   ：等同于or操作符
------------------------------------------------------------------------------------
expr1 -or expr2  ：只有expr1为假时才评估expr2。
------------------------------------------------------------------------------------
expr1 , expr2    ：逗号操作符表示列表的意思，expr1和expr2都会被评估，但expr1的true或false是被
                 ：无视的，只有expr2的结果才是最终状态值。
```

**expression-options**

options总是返回true。除了"-daystart"，options会影响所有指定的test表达式部分，哪怕是test部分写在options的前面。这是因为options是在命令行被解析完后立即处理的，而test是在检测到文件后才处理的。对于"-daystart"这个选项，它们仅仅影响写在它们后面的test部分，因此，建议将任何options部分写在expression的最前面，若不如此，会给出一个警告信息。

选项：

```
-daystart：指定以每天的开始(凌晨零点)计算关于天的时间，用于改变时间类(-amin,-atime,-cmin,-ctime,-mmin和-mtime)
         ：的计算方式。默认天的计算是从24小时前计算的。例如，当前时间为5月3日17:00，要求搜索出2天内修改过的文件，默认
         ：搜索文件的起点是5月1日17:00，如果使用-daystart，则搜索文件的起点是是5月1日00:00。
         ：注意，该选项只会影响写在它后面的test表达式。
---------------------------------------------------------------------------------------------------
-depth   ：搜索到目录时，先处理目录中的文件(子目录)，再处理目录本身。对于"-delete"这个action，它隐含"-depth"选项。
---------------------------------------------------------------------------------------------------
-maxdepth levels：指定tests和actions作用的最大目录深度，只能为非负整数。可以简单理解为目录搜索深度，但并非如此。当
                ：前path目录的层次为1，所以若指定-maxdepth 0将得不到任何结果。
---------------------------------------------------------------------------------------------------
-mindepth levels：tests和actions不会应用于小于指定深度的目录，"-mindepth 1"表示应用于所有的文件。
```

**expression-tests**

ind解析完命令行语法之后，开始搜索文件，在搜索过程中，每次检测到的文件都会被test expression进行测试，符合条件的将被保留。

数值部分可以设置为以下3种模式：n可以是小数。

> +n：大于n
>
> -n：小于n
>
> n ：精确的等于n

对于文件大小而言，文件的大小是精确的，指定100KB，则比对的值必定是100KB。此时(+ -)n和字面意思是一样的。但对于时间而言，时间是有时间段的，例如指定前第四天，第四天也整整占用了一天，所以(+ -)n和文件大小的计算方法是不一样的。

find在计算以天数为单位的时间时，默认会转换为24小时制，除非同时指定了"-daystart"这个选项，这在前面已经解释过了。例如当前时间为5月3号17:00，那么计算"atime +1"的时候，真正计算的是24*1=24小时之前，也就是5月2号17:00以前被访问过，若指定了"-daystart"，则计算的是5月2号00:00之前被访问过。

具体的选项如下：

```
-type X：根据文件类型来搜索
    · b：块设备文件
    · c：字符设备文件
    · d：目录
    · p：命名管道文件(FIFO文件)
    · f：普通文件
    · l：符号链接文件，即软链接文件
    · s：套接字文件(socket)
```

【文件大小或内容类测试条件】

```
-size n[cwbkMG]：根据文件大小来搜索，可以是(+ -)n，单位可以是：
            · b：512字节的(默认单位)
            · c：1字节的
            · w：2字节
            · k：1024字节
            · M：1024k
            · G：1024M
empty：空文件，对于目录来说，则是空目录
```

【文件名或路径名匹配类测试条件】

```
------------------------------------------------------------------------------------------------
-name pattern   | 文件的basename(不包括其前导目录的纯文件名)能被通配符模式的pattern匹配到。由于前导目录被移除，
                | 所以find对包含"/"的pattern是绝对不可能匹配到内容的，例如"-name a/b"的结果一定是空且会给出
                | 警告信息，若要匹配这样的文件，可考虑使用"-path"或"-name b"。需要注意的是，在find中的通配元
                | 字符"*"、"?"和"[]"是能够匹配以点开头的文件的，之所以要在此说明这一点，是因为在bash中，这些通
                | 配元字符默认是无法匹配"."开头的文件的，例如"cp ~/* /tmp"不会把隐藏文件也拷贝走。若要忽略一个
                | 目录及其内的文件，可以配合"-prune"，它会跳过整个目录而不对此目录做任何检查。注意pattern要用
                | 引号包围防止被shell解释               
----------------|-------------------------------------------------------------------------------
-iname pattern  | 不区分大小的"-name"               
----------------|-------------------------------------------------------------------------------
-path pattern   | 文件名能被通配符模式的pattern匹配到。此模式下，通配元字符"*"、"?"和"[]"不认为字符"/"或"."是
                | 特殊字符，也就是说这两个字符也在通配范围内，所以能匹配这两个字符。例如find . -path "./sr*sc"
                | 可以匹配到名为"./src/misc"的目录。find会将"-path"的pattern与文件的dirname和basename的结
                | 合体进行比较，由于dirname和basename的结合体不包含尾随"/"，所以如果pattern中指定了尾随"/"是
                | 不可能匹配到任何东西的，例如find /tmp -path "/tmp/ab*/"，实际上它会给出警告信息，提示
                | pattern以"/"结尾。使用"-path"的时候，一定要注意"-path"后指定的路径起点属于
                | "find path expression"的path内，例如"find /bar -path /foo/bar/myfile -print"不可能
                | 匹配到任何东西。若要忽略目录及其内文件，可配合"-prune"，它会跳过整个目录而不对此目录做检查。如
                | "find . -path ./src/emacs -prune -o -print"将跳过对目录"./src/emacs"的检查。
                | 注意pattern要用引号包围防止被shell解释                              
----------------|---------------------------------------------------------------------------------
-ipath pattern  | 不区分大小写的"-path"                        
----------------|---------------------------------------------------------------------------------
-regex pattern  | 文件名能被正则表达式pattern匹配到的文件。正则匹配会匹配整个路径，例如要匹配文件名为"./fubar3"
                | 的文件，可以使用".*bar."或".*b.*3"，但不能是"f.*r3"，默认find使用的正则类型是Emacs正则，
                | 但可以使用-regextype来改变正则类型                            
----------------|---------------------------------------------------------------------------------
-iregex pattern | 不区分大小写的"-regex"
```

【权限类测试条件】

```
-perm mode | 精确匹配给定权限的文件。"-perm g=w"将只匹配权限为0020的文件。当然，也可以写成三位数字的权限模式
-----------|-------------------------------------------------------------------------------------
-perm -mode| 匹配完全包含给定权限的文件，这是最可能用上的权限匹配方式。例如给定的权限"-0766"，则只能匹配"N767"、
           | "N777"和"N776"这几种权限的文件，如果使用字符模式的权限，则必须指定u/g/o/a，例如"-perm -u+x,a+r"
           | 表示至少所有人都有读权限，且所有者有执行权限的文件
-----------|-------------------------------------------------------------------------------------
-perm /mode| 匹配任意给定权限位的权限，例如"-perm /640"可以匹配出600，040,700,740等等，只要文件权限的任意位能
           | 包含给定权限的任意一位就满足
-----------|-------------------------------------------------------------------------------------
-perm +mode| 由于某些原因，此匹配模式被替换为"-perm /mode"，所以此模式已经废弃
-----------|-------------------------------------------------------------------------------------
-executable| 具有可执行权限的文件。它会考虑acl等的特殊权限，只要是可执行就满足。它会忽略掉-perm的测试
-----------|-------------------------------------------------------------------------------------
-readable  | 具有可读权限的文件。它会考虑acl等的特殊权限，只要是可读就满足。它会忽略掉-perm的测试
-----------|-------------------------------------------------------------------------------------
-writable  | 具有可写权限的文件。它会考虑acl等的特殊权限，只要是可写就满足。它会忽略掉-perm的测试(不是writeable)
```

【所有者所属组类测试条件】

```
-gid n      ：gid为n的文件
-group gname：组名为gname的文件
-uid n      ：文件的所有者的uid为n
-user uname ：文件的所有者为uname，也可以指定uid
-nogroup    ：匹配那些所属组为数字格式的gid，且此gid没有对应组名的文件
-nouser     ：匹配那些所有者为数字格式的uid，且此uid没有对应用户名的文件
```

【时间戳类测试条件】

```
-anewer file：atime比mtime更接近现在的文件。也就是说，文件修改过之后被访问过
-cnewer file：ctime比mtime更接近现在的文件
-newer  file：比给定文件的mtime更接近现在的文件。
-newer[acm]t TIME：atime/ctime/mtime比时间戳TIME更新的文件
-amin  n：文件的atime在范围n分钟内改变过。注意，n可以是(+ -)n，例如-amin +3表示在3分钟以前
-cmin  n：文件的ctime在范围n分钟内改变过
-mmin  n：文件的mtime在范围n分钟内改变过
-atime n：文件的atime在范围24*n小时内改变过
-ctime n：文件的ctime在范围24*n小时内改变过
-mtime n：文件的mtime在范围24*n小时内改变过
-used  n：最近一次ctime改变n天范围内，atime改变过的文件，即atime比ctime晚n天的文件，可以是(+ -)n
```

【软硬链接类测试条件】

```
-samefile name：找出指定文件同indoe的文件，即其硬链接文件
-inum  n：inode号为n的文件，可用来找出硬链接文件。但使用"-samefile"比此方式更方便
-links n：有n个软链接的文件
```

【杂项测试】

```
-false：总是返回false，这选项有奇用
-true ：总是返回true，这选项有奇用
```

**expression-actions**

actions部分一般都是执行某些命令，或实现某些功能。这部分是find的command line部分。

```
-delete        | 删除文件，如果删除成功则返回true，如果删除失败，将给出错误信息。"-delete"动作隐含"-depth"。
---------------------------------------------------------------------------------------------------
-exec command ;| 注意有个分号";"结尾，该action是用于执行给定的命令。如果命令的返回状态码为0则该action返回true。
               | command后面的所有内容都被当作command的参数，直到分号";"为止，其中参数部分使用字符串"{}"时，它
               | 表示find找到的文件名，即在执行命令时，"{}"会被逐一替换为find到的文件名，"{}"可以出现在参数中的
               | 任何位置，只要出现，它都会被文件名替换。
               | 注意，分号";"需要转义，即"\;"，如有需要，可以将"{}"用引号包围起来
---------------|-----------------------------------------------------------------------------------
-ok command ;  | 类似于-exec，但在执行命令前会交互式进行询问，如果不同意，则不执行命令并返回false，如果同意，则执
               | 行命令，但执行的命令是从/dev/null读取输入的
---------------|-----------------------------------------------------------------------------------
-print | 总是返回true。这是默认的action，输出搜索到文件的全路径名，并尾随换行符"\n"。由于在使用"-print"时所有的结
       | 果都有换行符，如果直接将结果通过管道传递给管道右边的程序，应该要考虑到这一点：文件名中有空白字符(换行符、制表
       | 符、空格)将会被右边程序误分解，如文件"ab c.txt"将被认为是ab和c.txt两个文件，如不想被此分解影响，可考虑使
       | 用"-print0"替代"-print"将所有换行符替换为"\0"
-------|-------------------------------------------------------------------------------------------
-printf| 输出格式太多，所以具体用法见man文档
-------|-------------------------------------------------------------------------------------------
-print0| 总是返回true。输出搜索到文件的全路径名，并尾随空字符"\0"。由于尾随的是空字符，所以管道传递给右边的程序，然后
       | 只需对这个空字符进行识别分隔就能保证文件名不会因为其中的空白字符被误分解
---------------------------------------------------------------------------------------------------
-prune | 不进入目录，所以可用于忽略目录，但不会忽略普通文件。没有给定-depth时，总是返回true，如果给定-depth，则直接
       | 返回false，所以-delete(隐含了-depth)是不能和-prune一起使用的
-------|-------------------------------------------------------------------------------------------
-ls    | 总是返回true。将找到的文件以"ls -dils"的格式打印出来，其中文件的size部分以KB为单位
```

一定要注意，**action是可以写在tests表达式前面的，它并不一定是在test表达式之后执行**。



参考：[Linux find运行机制详解](https://www.cnblogs.com/f-ck-need-u/p/6995529.html)

#### 练习题

```
1、找出/tmp目录下属主为非root的所有文件；
[hyp@localhost ~]$ find /tmp -not -user  root -ls
2、找出/tmp目录下文件名中不包含fstab字符串的文件；
[hyp@localhost ~]$ sudo find /tmp -not -name "*fstab*" -ls
3、找出/tmp目录下属主为非root，而且文件名不包含fstab字符串的文件；
[hyp@localhost ~]$ sudo find /tmp  \( ! -name "*fstab*" -a ! -user root \) -ls
[hyp@localhost ~]$ sudo find /tmp !  \(  -name "*fstab*" -o  -user root \) -ls

1、查找/var目录下属主为root，且属组为mail的所有文件或目录；
	~]# find /var -user root -a -group mail -ls

2、查找/usr目录下不属于root, bin或hadoop的所有文件或目录；用两种方法；
	~]# find /usr -not -user root -a -not -user bin -a -not -user hadoop
	~]# find /usr -not \( -user root -o -user bin -o -user hadoop \) -ls

3、查找/etc目录下最近一周内其内容修改过，且属主不是root用户也不是hadoop用户的文件或目录；
	~]# find /etc -mtime -7 -a -not \( -user root -o -user hadoop \) -ls
	~]# find /etc -mtime -7 -a -not -user root -a -not -user hadoop -ls

4、查找当前系统上没有属或属组，且最近一周内曾被访问过的文件或目录；
	~]# find  /  \( -nouser -o -nogroup \)  -atime  -7  -ls

5、查找/etc目录下大于1M且类型为普通文件的所有文件；
	~]# find /etc -size +1M -type f -exec ls -lh {} \;

6、查找/etc目录下所有用户都没有写权限的文件；
	~]# find /etc -not -perm /222 -type f -ls					

7、查找/etc目录至少有一类用户没有执行权限的文件；
	~]# find /etc -not -perm -111 -type f -ls

8、查找/etc/init.d/目录下，所有用户都有执行权限，且其它用户有写权限的所有文件；
	~]# find /etc -perm -113 -type f -ls

```

### 文件打包和压缩

压缩的目的是用时间来换空间。总的来说，linux上常用的压缩包和对应命令如下：

```
	compress/uncompress, .Z
	gzip/gunzip,  .gz
	bzip2/bunzip2,  .bz2
	xz/unxz,  .xz
	zip/unzip  .zip
	tar, cpio  归档
```

下面先介绍压缩命令：

1、`.gz`

涉及命令gzip、gunzip、 zcat，gzip可用于解压缩，gunzip用于解压，zcat用于查看压缩文件信息。

语法：gzip  [OPTION]...  FILE...

选项：

```
-d：解压缩，相当于gunzip；
-#：指定压缩比，默认是6；数字越大压缩比越大（1-9）；
-c：将压缩结果输出至标准输出；
```


常用：gzip  -c  FILE > /PATH/TO/SOMEFILE.gz

2、`.bz2`

涉及命令bzip2、bunzip2、bzcat，

语法：bzip2  [OPTION]...  FILE...

选项：-d：解压缩
	-#：指定压缩比；默认是6；数字越大压缩比越大（1-9）；
	-k：keep，保留原文件；



3、 `.xz`

涉及命令xz、unxz、xzcat，

语法：xz  [OPTION]...  FILE...

选项：-d：解压缩
	   -#：指定压缩比；默认是6；数字越大压缩比越大（1-9）；
	   -k：保留原文件；

4、`.zip`

涉及命令zip、unzip

语法：zip  [OPTION]...  FILE...

选项：
	   -#：指定压缩比；默认是6；数字越大压缩比越大（1-9）；

5、`.gz`

涉及命令gzip、gunzip。

#### tar

打包命令，语法：`tar  [OPTION]...  FILE...`

(1) 创建归档
	-c -f /PATH/TO/SOMEFILE.tar  FILE...
	-cf /PATH/TO/SOMEFILE.tar  FILE...

(2) 展开归档
	-xf  /PATH/FROM/SOMEFILE.tar
	-xf  /PATH/FROM/SOMEFILE.tar  -C  /PATH/TO/SOMEDIR

(3) 查看归档文件的文件列表
	-tf  /PATH/TO/SOMEFILE.tar

归档完成后通常需要压缩，结果此前的压缩工具，就能实现压缩多个文件了；
(4) 归档压缩
	-z：gzip2
		-zcf   /PATH/TO/SOMEFILE.tar.gz  FILE...
		解压缩并展开归档：-zxf  /PATH/TO/SOMEFILE.tar.gz
	-j：bzip2
		-jcf 归档并压缩
		-jxf 解压缩并展开归档
	-J: xz
		-Jcf 归档并压缩
		-Jxf 解压缩并展开归档

#### 常见解压/压缩命令

tar
解包：tar xvf FileName.tar
打包：tar cvf FileName.tar DirName
（注：tar是打包，不是压缩！）


.gz
解压1：gunzip FileName.gz
解压2：gzip -d FileName.gz
压缩：gzip FileName

.tar.gz 和 .tgz
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName

.bz2
解压1：bzip2 -d FileName.bz2
解压2：bunzip2 FileName.bz2
压缩： bzip2 -z FileName

.tar.bz2
解压：tar jxvf FileName.tar.bz2
压缩：tar jcvf FileName.tar.bz2 DirName

.bz
解压1：bzip2 -d FileName.bz
解压2：bunzip2 FileName.bz
压缩：未知

.tar.bz
解压：tar jxvf FileName.tar.bz
压缩：未知

.Z
解压：uncompress FileName.Z
压缩：compress FileName

.tar.Z
解压：tar Zxvf FileName.tar.Z
压缩：tar Zcvf FileName.tar.Z DirName

.zip
解压：unzip FileName.zip
压缩：zip FileName.zip DirName

.rar
解压：rar x FileName.rar
压缩：rar a FileName.rar DirName

> 中文解压有乱码,指定编码集：unzip -O CP936 xxx.zip


.tar.xz
解压： tar xvJf  xxx.tar.xz
压缩： 只要先 tar cvf xxx.tar xxx/ 这样创建xxx.tar文件先，然后使用 xz -z xxx.tar 来将 xxx.tar压缩成为 xxx.tar.xz
