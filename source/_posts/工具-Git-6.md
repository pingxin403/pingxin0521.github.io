---
title: Git 命令大全
date: 2019-04-21 09:18:59
tags:
 - 工具
 - Git
categories:
 - 工具
 - 版本控制
 - Git
---

前面是对git的通用介绍，这里，对工作和开发中常用的操作进行记录。

<!--more-->

#### 快速解决冲突

- 情景一：多个分支代码合并到一个分支时；
- 情景二：多个分支向同一个远端分支推送代码时；

实际上，push操作即是将本地代码merge到远端库分支上。

关于push和pull其实就分别是用本地分支合并到远程分支 和 将远程分支合并到本地分支

所以这两个过程中也可能存在冲突。

git的合并中产生冲突的具体情况：

1. 两个分支中修改了同一个文件（不管什么地方）
2. 两个分支中修改了同一个文件的名称

两个分支中分别修改了不同文件中的部分，不会产生冲突，可以直接将两部分合并。

**方法**

1. 使用工具

   安装beyond compare

   在个git中配置

   ```
   git config --global merge.tool bc3
   git config --global mergetool.path '/usr/bin/bcompare'
   git config --global mergetool.keepBackup false 
   ```

   应用beyond compare解决冲突

   ```
   git mergetool 
   ```

2. 把你修改的代码进行备份，然后执行命令

   ```
   git reset --hard origin/master
   git pull
   #从你备份好的文件当中把你写的代码拿过去，修改完成再进行git push
   ```

3. 在当前分支上，直接修改冲突代码--->add--->commit。

4. 在本地当前分支上，修改冲突代码--->add--->commit--->push

更多：<https://www.cnblogs.com/gavincoder/p/9071959.html>

#### 撤销代码修改

1. 如果代码尚未提交，即还未执git add命令，则可以对已经修改的代码进行撤销。

   ```
   git checkout <文件或者文件夹>
   ```

2. 如果代码已经添加到索引库，即执行了git add命令，那么需要先撤销添加，然后再撤销修改。

   ```
   #撤销添加
   git reset HEAD <文件或者文件夹>
   
   #撤销修改
   git checkout <文件或者文件夹>
   ```

3. 执行了git commit

   - 先找到之前提交的git commit的id ，通过：git log查看历史提交记录，找到想要撤销的id
   - 输入命令：`git reset id` ，含义：回退到某个版本，只保留源码，回退commit和index信息。
   - 或者输入命令：`git reset –hard id` ，含义：完成撤销，同时将代码恢复到前commit_id 对应的版本 。**该命令将彻底回退到某个版本，本地的源码也会变为上一个版本的内容，该命令慎用！**

   根据`–soft –mixed –hard`，会对working tree和index和HEAD进行重置：

   - `git reset --mixed`：此为默认方式，不带任何参数的git reset，即是这种方式，它回退到某个版本，只保留源码，回退commit和index信息
   - `git reset --soft`：回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
   - `git reset  --hard`：彻底回退到某个版本，本地的源码也会变为上一个版本的内容，此命令 慎用！

4. 回退到之前的倒数第N个提交

   ```
   git  reset   HEAD~N
   ```

5. 删除已经提交到服务器上的缓存文件

   ```
   1.先从github上拉取
   git pull 
   2.删除本地缓存
   git rm -r --cached target
   3.push到github
   git push origin master
    
   4.修改.gitignore文件，添加要过滤的文件夹或文件类型
   如，增加/target/
    
   5.添加修改
   git add .gitignore
    
   6.提交修改
   git commit -m "修改了.gitignore文件"
    
   7.push到github
   git push
   ```

   经过以上七个步骤，原来已经提交到github上的缓存文件可以被删除，并且之后再次提交代码也不会提交缓存到github上了，因为我们在.gitignore文件中增加了过滤：如/target/

   以下是一个.gitignore文件，我在其中增加了/target/来过滤掉IDEA生成的文件，这些是不需要提交的。

   ```
   # IntelliJ project files
   /target/ 
   .idea
   *.iml
   # Compiled class file
   *.class
    
   # Log file
   *.log
    
   # BlueJ files
   *.ctxt
    
   # Mobile Tools for Java (J2ME)
   .mtj.tmp/
    
   # Package Files #
   *.jar
   *.war
   *.nar
   *.ear
   *.zip
   *.tar.gz
   *.rar
   
   # virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
   hs_err_pid*
   ```

6. 删除具体某个提交commit

   ```
   1.git log获取commit信息 
   2.git rebase -i (commit-id) 
   commit-id 为要删除的commit的下一个commit号 
   3.编辑文件，将要删除的commit之前的单词改为drop 
   4.保存文件退出大功告成 
   5.git log查看
   6.git push origin HEAD –force 然后推送到远程仓库
   ```

   

#### 删除未跟踪文件

1. 先查看有哪些文件可以删除,但是不真执行删除

   ```
   git rm -r -n target/*
   ```

   -r  递归移除目录

   -n 加上这个参数，执行命令时，是不会删除任何文件，而是展示此命令要删除的文件列表预览，所以一般用这个参数先看看要删除哪些文件，防止误删，确认之后，就去掉此参数，真正的删除文件。

   上面这个命令就是先查看 target/* 下有哪些可以删除的内容

2. 删除 untracked files

   ```
   git clean -f
   ```

3. 连 untracked 的目录也一起删掉

   ```
   git clean -fd
   ```

4. 连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）

   ```
   git clean -xfd
   ```

5. 在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删

   ```
   git clean -nxfd
   
   git clean -nf
   
   git clean -nfd
   ```



#### git代码统计

```shell
git log 参数说明： 
–author 指定作者 
–stat 显示每次更新的文件修改统计信息，会列出具体文件列表 
–shortstat 统计每个commit 的文件修改行数，包括增加，删除，但不列出文件列表： 
–numstat 统计每个commit 的文件修改行数，包括增加，删除，并列出文件列表： 
-p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新 
例如：git log -p -2 
–name-only 仅在提交信息后显示已修改的文件清单 
–name-status 显示新增、修改、删除的文件清单 
–abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符 
–relative-date 使用较短的相对时间显示（比如，“2 weeks ago”） 
–graph 显示 ASCII 图形表示的分支合并历史 
–pretty 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式） 
例如： git log –pretty=oneline ; git log –pretty=short ; git log –pretty=full ; git log –pretty=fuller 
–pretty=tformat: 可以定制要显示的记录格式，这样的输出便于后期编程提取分析 
例如：git log –pretty=format:”“%h - %an, %ar : %s”” 
下面列出了常用的格式占位符写法及其代表的意义。 
选项 说明 
%H 提交对象（commit）的完整哈希字串 
%h 提交对象的简短哈希字串 
%T 树对象（tree）的完整哈希字串 
%t 树对象的简短哈希字串 
%P 父对象（parent）的完整哈希字串 
%p 父对象的简短哈希字串 
%an 作者（author）的名字 
%ae 作者的电子邮件地址 
%ad 作者修订日期（可以用 -date= 选项定制格式） 
%ar 作者修订日期，按多久以前的方式显示 
%cn 提交者(committer)的名字 
%ce 提交者的电子邮件地址 
%cd 提交日期 
%cr 提交日期，按多久以前的方式显示 
%s 提交说明 
–since 限制显示输出的范围， 
例如： git log –since=2.weeks 显示最近两周的提交 
选项 说明 
-(n) 仅显示最近的 n 条提交 
–since, –after 仅显示指定时间之后的提交。 
–until, –before 仅显示指定时间之前的提交。 
–author 仅显示指定作者相关的提交。 
–committer 仅显示指定提交者相关的提交。
```

**git blame**

显示最后修改的版本和作者修改了文件的每一行

```shell
用法：git blame [<选项>] [<版本选项>] [<版本>] [--] <文件>

    <版本选项> 的文档记录在 git-rev-list(1) 中

    --incremental         增量式地显示发现的 blame 条目
    -b                    边界提交显示空的 SHA-1（默认：关闭）
    --root                不把根提交作为边界（默认：关闭）
    --show-stats          显示命令消耗统计
    --progress            强制进度显示
    --score-debug         显示判断 blame 条目位移的得分诊断信息
    -f, --show-name       显示原始文件名（默认：自动）
    -n, --show-number     显示原始的行号（默认：关闭）
    -p, --porcelain       显示为一个适合机器读取的格式
    --line-porcelain      为每一行显示机器适用的提交信息
    -c                    使用和 git-annotate 相同的输出模式（默认：关闭）
    -t                    显示原始时间戳（默认：关闭）
    -l                    显示长的 SHA1 提交号（默认：关闭）
    -s                    隐藏作者名字和时间戳（默认：关闭）
    -e, --show-email      显示作者的邮箱而不是名字（默认：关闭）
    -w                    忽略空白差异
    --indent-heuristic    使用一个试验性的启发式算法改进差异显示
    --minimal             花费额外的循环来找到更好的匹配
    -S <文件>             使用来自 <文件> 的修订集而不是调用 git-rev-list
    --contents <文件>     使用 <文件> 的内容作为最终的图片
    -C[<得分>]            找到文件内及跨文件的行拷贝
    -M[<得分>]            找到文件内及跨文件的行移动
    -L <n,m>              只处理行范围在 n 和 m 之间的，从 1 开始
    --abbrev[=<n>]        用 <n> 位数字显示 SHA-1 哈希值

```

其显示格式为：

```
 commit ID | 代码提交作者 | 提交时间 | 代码位于文件中的行数 | 实际代码
```

这样，我们就可以知道 commit ID 了，然后使用命令：`git show commitID` 来看具体的修改

**示例**

1. 查看git上的个人代码量

   ```shell
   git log  --since="2018-07-16" --before="2019-02-14" --author="username" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
   ```

   结果示例：(记得修改 username)

   ```
   added lines: 2160, removed lines: 424, total lines: 1736
   ```

2. 使用 ls-file 实现不指定用户版统计行数版，使用ruby

   ```shell
   git ls-files -z | xargs -0n1 git blame -w | ruby -n -e '$_ =~ /^.*\((.*?)\s[\d]{4}/; puts $1.strip' | sort -f | uniq -c | sort -n
   ```

3. 统计每个人增删行数

   ```shell
   git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
   ```

4. 查看仓库提交者排名前 5

   ```shell
   git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
   ```

5. 贡献值/开发者人数统计

   ```
   git log --pretty='%aN' | sort -u | wc -l
   ```

6. 提交数统计

   ```
   git log --oneline | wc -l
   ```

7. 添加或修改的代码行数：

   ```shell
   git log --stat|perl -ne 'END { print $c } $c += $1 if /(\d+) insertions/'
   ```

8. 第三方小工具版

   使用这个工具可以直接输出非常漂亮的统计表格：<https://github.com/oleander/git-fame-rb>

9. gitstats

   [GitStats项目](https://github.com/hoxu/gitstats)，用Python开发的一个工具，通过封装Git命令来实现统计出来代码情况并且生成可浏览的网页。

   ```shell
   git clone git://github.com/hoxu/gitstats.git
   cd gitstats
   ./gitstats <你的项目的位置> <生成统计的文件夹位置>
   ```

   可能会提示没有安装gnuplot画图程序，那么需要安装再执行：

   ```shell
   //mac osx
   brew install gnuplot
   //centos linux
   yum install gnuplot
   //ubuntu linux
   apt install gnuplot
   ```

   

10. 使用[cloc](https://github.com/AlDanial/cloc)

    ```
    npm install -g cloc
    cloc EtCampusService # 文件夹名
```
    

#### 全部命令

```shell
$ git help                 
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]

这些是各种场合常见的 Git 命令：

开始一个工作区（参见：git help tutorial）
   clone      克隆一个仓库到一个新目录
   init       创建一个空的 Git 仓库或重新初始化一个已存在的仓库

在当前变更上工作（参见：git help everyday）
   add        添加文件内容至索引
   mv         移动或重命名一个文件、目录或符号链接
   reset      重置当前 HEAD 到指定状态
   rm         从工作区和索引中删除文件

检查历史和状态（参见：git help revisions）
   bisect     通过二分查找定位引入 bug 的提交
   grep       输出和模式匹配的行
   log        显示提交日志
   show       显示各种类型的对象
   status     显示工作区状态

扩展、标记和调校您的历史记录
   branch     列出、创建或删除分支
   checkout   切换分支或恢复工作区文件
   commit     记录变更到仓库
   diff       显示提交之间、提交和工作区之间等的差异
   merge      合并两个或更多开发历史
   rebase     在另一个分支上重新应用提交
   tag        创建、列出、删除或校验一个 GPG 签名的标签对象

协同（参见：git help workflows）
   fetch      从另外一个仓库下载对象和引用
   pull       获取并整合另外的仓库或一个本地分支
   push       更新远程引用和相关的对象

命令 'git help -a' 和 'git help -g' 显示可用的子命令和一些概念帮助。
查看 'git help <命令>' 或 'git help <概念>' 以获取给定子命令或概念的
帮助。

$ git help -g
最常用的 Git 向导有：

   attributes   定义路径的属性
   everyday     每一天 Git 常用的约 20 条命令
   glossary     Git 词汇表
   ignore       忽略指定的未跟踪文件
   modules      定义子模组属性
   revisions    指定 Git 的版本和版本范围
   tutorial     一个 Git 教程（针对 1.5.1 或更新版本）
   workflows    Git 推荐的工作流概览

$ git help -a
用法：git [--version] [--help] [-C <path>] [-c <键名>=<值>]
           [--exec-path[=<路径>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<路径>] [--work-tree=<路径>] [--namespace=<名称>]
           <命令> [<参数>]

在 '/usr/lib/git-core' 下可用的 git 命令

  add                       get-tar-commit-id         remote
  add--interactive          grep                      remote-ext
  am                        hash-object               remote-fd
  annotate                  help                      remote-ftp
  apply                     http-backend              remote-ftps
  archive                   http-fetch                remote-http
  bisect                    http-push                 remote-https
  bisect--helper            imap-send                 remote-testsvn
  blame                     index-pack                repack
  branch                    init                      replace
  bundle                    init-db                   request-pull
  cat-file                  instaweb                  rerere
  check-attr                interpret-trailers        reset
  check-ignore              log                       rev-list
  check-mailmap             ls-files                  rev-parse
  check-ref-format          ls-remote                 revert
  checkout                  ls-tree                   rm
  checkout-index            mailinfo                  send-pack
  cherry                    mailsplit                 sh-i18n--envsubst
  cherry-pick               merge                     shell
  clean                     merge-base                shortlog
  clone                     merge-file                show
  column                    merge-index               show-branch
  commit                    merge-octopus             show-index
  commit-tree               merge-one-file            show-ref
  config                    merge-ours                stage
  count-objects             merge-recursive           stash
  credential                merge-resolve             status
  credential-cache          merge-subtree             stripspace
  credential-cache--daemon  merge-tree                submodule
  credential-store          mergetool                 submodule--helper
  daemon                    mktag                     subtree
  describe                  mktree                    symbolic-ref
  diff                      mv                        tag
  diff-files                name-rev                  unpack-file
  diff-index                notes                     unpack-objects
  diff-tree                 pack-objects              update-index
  difftool                  pack-redundant            update-ref
  difftool--helper          pack-refs                 update-server-info
  fast-export               patch-id                  upload-archive
  fast-import               prune                     upload-pack
  fetch                     prune-packed              var
  fetch-pack                pull                      verify-commit
  filter-branch             push                      verify-pack
  fmt-merge-msg             quiltimport               verify-tag
  for-each-ref              read-tree                 web--browse
  format-patch              rebase                    whatchanged
  fsck                      rebase--helper            worktree
  fsck-objects              receive-pack              write-tree
  gc                        reflog


```



#### 