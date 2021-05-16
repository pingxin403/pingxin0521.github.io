---
title: Linux-expect
date: 2019-05-26 21:55:22
tags: 
 - Linux
 - 脚本
category: 
 - Linux
 - 脚本
---

Expect是一个用来处理交互的命令。借助Expect，我们可以将交互过程写在一个脚本上，使之自动化完成。形象的说，ssh登录，ftp登录等都符合交互的定义。

<!--more-->

查看是否安装了expect

```
rpm  -qa|grep expect
#否则，安装
yum install expect -y
```

#### 选项

- -c :执行脚本前先执行的命令，可多次使用。
- -d :debug模式，可以在运行时输出一些诊断信息，与在脚本开始处使用 exp_internal 1 相似。
- -D :启用交换调式器,可设一整数参数。
- -f :从文件读取命令，仅用于使用#!时。如果文件名为"-"，则从stdin读取(使用"./-"从文件名为-的文件读取)。
- -i :交互式输入命令，使用"exit"或"EOF"退出输入状态。
- -- :标示选项结束(如果你需要传递与expect选项相似的参数给脚本时)，可放到 #! 行: #!/usr/bin/expect -- 。
- -v :显示expect版本信息。

#### 常用命令

```bash
#!/usr/bin/expect
# 命令行参数
# $argv，参数数组，使用[lindex $argv n]获取，$argv 0为脚本名字
# $argc，参数个数
set username [lindex $argv 1]   # 获取第1个参数
set passwd [lindex $argv 2]     # 获取第2个参数

set timeout 30                 # 设置超时

# spawn是expect内部命令，开启ssh连接
spawn ssh -l username 192.168.1.1

# 判断上次输出结果里是否包含“password:”的字符串，如果有则立即返回，否则就等待一段时间(timeout)后返回
expect "password:"

# 发送内容ispass(密码、命令等)
send "ispass\r"

# 发送内容给用户
send_user "$argv0 [lrange $argv 0 2]\n"
send_user "It's OK\r"

# 执行完成后保持交互状态，控制权交给控制台(手工操作)。否则会完成后会退出。
interact
```

1. send命令

   send命令接收一个字符串参数，并将该参数发送到进程。

   ```
   expect1.1> send "hello world\n" hello world
   ```

2. expect命令

   和send命令正好相反，expect通常是用来等待一个进程的反馈。expect可以接收一个字符串参数，也可以接收正则表达式参数。

   ```
   expect "hi\n" send "hello there!\n"
   ```

   这两行代码的意思是：从标准输入中等到hi和换行键后，向标准输出输出hello there。

3. spawn命令

   上文的所有demo都是和标准输入输出进行交互，但是我们跟希望他可以和某一个进程进行交互。spawm命令就是用来启动新的进程的。spawn后的send和expect命令都是和spawn打开的进程进行交互的。

   ```bash
   set timeout -1
   spawn ftp ftp.test.com //打开新的进程，该进程用户连接远程ftp服务器
   expect "Name" //进程返回Name时
   send "user\r" //向进程输入anonymous\r
   expect "Password:" //进程返回Password:时
   send "123456\r" //向进程输入don@libes.com\r
   expect "ftp> " //进程返回ftp>时
   send "binary\r" //向进程输入binary\r
   expect "ftp> " //进程返回ftp>时
   send "get test.tar.gz\r" //向进程输入get test.tar.gz\r
   #这段代码的作用是登录到ftp服务器ftp ftp.uu.net上，并以二进制的方式下载服务器上的文件test.tar.gz。
   ```

4. interact

   到现在为止，我们已经可以结合spawn、expect、send自动化的完成很多任务了。但是，如何让人在适当的时候干预这个过程了。比如下载完ftp文件时，仍然可以停留在ftp命令行状态，以便手动的执行后续命令。interact可以达到这些目的。下面的demo在自动登录ftp后，允许用户交互。

   ```bash
   spawn ftp ftp.test.com
   expect "Name"
   send "user\r"
   expect "Password:"
   send "123456\r"
   interact
   ```

**其他命令**

- close:关闭当前进程的连接。
- debug:控制调试器。
- disconnect:断开进程连接(进程仍在后台运行)。
- 定时读取密码、执行priv_prog

```bash
    send_user "password?\ "
    expect_user -re "(.*)\n"
    for {} 1 {} {
     if {[fork]!=0} {sleep 3600;continue}
     disconnect
     spawn priv_prog
     expect Password:
     send "$expect_out(1,string)\r"
     . . .
     exit
    }
```

- exit:退出expect。
- exp_continue [-continue_timer]:继续执行下面的匹配。
- xp_internal [-f file] value:

#### expect范例

1. 自动telnet会话

   ```bash
   #!/usr/bin/expect -f
   set ip [lindex $argv 0 ] # 接收第1个参数,作为IP
   set userid [lindex $argv 1 ] # 接收第2个参数,作为userid
   set mypassword [lindex $argv 2 ] # 接收第3个参数,作为密码
   set mycommand [lindex $argv 3 ] # 接收第4个参数，作为命令
   set timeout 10 # 设置超时时间
   
   # 向远程服务器请求打开一个telnet会话，并等待服务器询问用户名
   spawn telnet $ip
   expect "username:"
   # 输入用户名，并等待服务器询问密码
   send "$userid\r"
   expect "password:"
   # 输入密码，并等待键⼊需要运行的命令
   send "$mypassword\r"
   expect "%"
   # 输入预先定好的密码，等待运行结果
   send "$mycommand\r"
   expect "%"
   # 将运行结果存入到变量中，显示出来或者写到磁盘中
   set results $expect_out(buffer)
   # 退出telnet会话，等待服务器的退出提示EOF
   send "exit\r"
   expect eof
   ```

2. 自动建立FTP会话

   ```bash
   #!/usr/bin/expect -f
   set ip [lindex $argv 0 ]            # 接收第1个参数,作为IP
   set userid [lindex $argv 1 ]        # 接收第2个参数,作为Userid
   set mypassword [lindex $argv 2 ]    # 接收第3个参数,作为密码
   set timeout 10                     # 设置超时时间
   
   # 向远程服务器请求打开一个FTP会话，并等待服务器询问用户名
   spawn ftp $ip
   expect "username:"
   # 输入用户名，并等待服务器询问密码
   send "$userid\r"
   expect "password:"
   # 输入密码，并等待FTP提示符的出现
   send "$mypassword\r"
   expect "ftp>"
   # 切换到二进制模式，并等待FTP提示符的出现
   send "bin\r"
   expect "ftp>"
   # 关闭ftp的提示符
   send "prompt\r"
   expect "ftp>"
   # 下载所有文件
   send "mget *\r"
   expect "ftp>"
   # 退出此次ftp会话，并等待服务器的退出提示EOF
   send "bye\r"
   expect eof
   ```

3. 自动登录SSH

   ```bash
   #!/usr/bin/expect -f 
   set ip [lindex $argv 0 ]            # 接收第1个参数,作为IP
   set username [lindex $argv 1 ]  # 接收第2个参数,作为username
   set mypassword [lindex $argv 2 ]   # 接收第3个参数,作为密码
   set timeout 10                      # 设置超时时间
   
   
   spawn ssh $username@$ip         # 发送ssh请求
   expect {                      # 返回信息匹配
       "*yes/no" { send "yes\r"; exp_continue}     # 第一次ssh连接会提示yes/no,继续 
       "*password:" { send "$mypassword\r" }        # 出现密码提示,发送密码 
   } 
   interact    # 交互模式,⽤户会停留在远程服务器上⾯
   ```

4. 批量登录ssh服务器执行操作范例，设定增量的for循环

   ```bash
   #!/usr/bin/expect
   for {set i 10} {$i <= 12} {incr i} {
        set timeout 30
        set ssh_user [lindex $argv 0]
        spawn ssh -i .ssh/$ssh_user abc$i.com
   
        expect_before "no)?" {
        send "yes\r" }
        sleep 1
        expect "password*"
        send "hello\r"
        expect "*#"
        send "echo hello expect! > /tmp/expect.txt\r"
        expect "*#"
        send "echo\r"
   }
   exit
   ```

5. 批量登录ssh并执⾏命令，foreach语法

   ```bash
   #!/usr/bin/expect
   if {$argc!=2} {
        send_user "usage: ./expect ssh_user password\n"
        exit
   }
   
   foreach i {11 12} {
        set timeout 30
        set ssh_user [lindex $argv 0]
        set password [lindex $argv 1]
        spawn ssh -i .ssh/$ssh_user root@xxx.yy.com
        expect_before "no)?" {
        send "yes\r" }
        sleep 1
        expect "Enter passphrase for key*"
        send "password\r"
        expect "*#"
        send "echo hello expect! > /tmp/expect.txt\r"
        expect "*#"
        send "echo\r"
   }
   exit
   ```

6. 另一自动ssh范例

   ```bash
   #!/usr/bin/expect
   # 使用方法: script_name ip1 ip2 ip3 ...
   set timeout 20
   if {$argc < 1} {
        puts "Usage: script IPs"
        exit 1
   }
   # 替换你自己的用户名
   set user "username"
   #替换你自己的登录密码
   
   set password "yourpassword"
   foreach IP $argv {
   spawn ssh $user@$IP
   
   expect \
     "(yes/no)?" {
        send "yes\r"
        expect "password:?" {
            send "$password\r"
        }
     } "password:?" {
        send "$password\r"
   }
   
   expect "\$?"
   
   # 替换你要执⾏的命令
   send "last\r"
   expect "\$?"
   sleep 10
   send "exit\r"
   expect eof
   }
   ```

7. ssh实现自动登录，并停留在登录服务器上

   ```bash
   #!/usr/bin/expect -f  
   #接收第一个参数,并设置IP
   set ip [ lindex $argv 0 ]
   
   #接收第二个参数,并设置密码
   set password [ lindex $argv 1 ]
   
   #设置超时时间
   set timeout 10
   
   #发送ssh请求
   spawn ssh -p2002 root@$ip
    expect {
    "*yes/no" { send "yes\r"; exp_continue}
    "*password:" { send "$password\r" }
    }
   
   # 交互模式,用户会停留在远程服务器上面.
   interact
   ```

   **exp_continue**是表示继续匹配下一次输入。

8. 批量拷贝公钥至目标主机

   实现免密码登录，通过shell脚本去调用expect脚本，达到循环拷贝公钥的目的。将expect脚本和shell脚本写在一起的话，容易出现问题。

   ```bash
#!/bin/bash
   
   function package_install() {
       if [ -z $1 ]; then
           exit
       fi
       rpm -q $1 &>/dev/null
       if [ $? -ne 0 ]; then
           yum install -y $1 &>/dev/null
           if [ $? -ne 0 ]; then
               echo "install $1  error"
            exit 1
           fi
    fi
   }
   
   function update_remote_ssh_key() {
       packages=("expect" "openssh-clients" "openssh")
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
    #判断网络是否可用
       ips=$(ip -o -4 addr show up primary scope global | awk '{print $2,$4}' | grep "^[e*|w*]" | cut -d' ' -f2 | cut -d. -f1-3)
       if [ -z ips ]; then
           echo "网络不可用"
       fi
   
       for p in ${packages[@]}; do
           package_install ${p}
       done
   
       if [ ! -f ~/.ssh/id_rsa ]; then
           echo "创建密钥"
           ssh-keygen -P "" -f ~/.ssh/id_rsa &>/dev/null
           if [ $? -eq 0 ]; then
               echo "创建成功"
           else
               echo "创建失败"
               exit 2
           fi
       fi
    name=$2
       password=$3
       for ip in `cat $1`
       do
               {
                   ping -c1 -W1 $ip &>/dev/null
                   if [ $? -eq 0 ];then
                       /usr/bin/expect <<-EOF
                           set timeout 20
                           spawn ssh-copy-id  ${name}@${ip}
                           expect  {
                               "(yes/no)"  { send "yes\r";exp_continue }
                               "password:" { send "$password\r" }
                           }
                           expect eof
   EOF
                       echo "$ip " >> ok_ip.txt
                   fi
               }&
       done
       wait
   
   }
   
   update_remote_ssh_key $1  root 123456
   ```
   
   
   
   