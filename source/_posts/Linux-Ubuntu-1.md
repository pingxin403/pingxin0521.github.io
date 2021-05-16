---
title: Ubuntu 基础配置和美化
date: 2019-04-20 21:55:22
tags:
 - Linux
 - Ubuntu
category:
 - Linux
 - Ubuntu
---
### 前言

由于博主的电脑老化，运行Windows时风扇高速运转，电脑时不时卡顿，所以为了更好的体验和使用，我开始寻找是否有一些可以对用户友好，并且占用更少资源的操作系统，然后发现其实Ubuntu就是一个很好的操作系统，并且具有很高水平的可定制桌面，有QQ、百度网盘的解决方案，现在转入Ubuntu可以说风险更小一些，当然Ubuntu有它的不足，很多在Windows上常用的软件在Ubuntu上很少有适合的，所以有所取舍。

推荐安装Ubuntu 20.04 LTS，因为是长期支持版，更适合在个人电脑上安装，新版本在虚拟机安装玩玩就行了。

Ubuntu 镜像下载地址：<http://tel.mirrors.163.com/ubuntu-releases/20.04/>

<!--more-->


每两年的 4 月份，都会推出一个长期支持版本（LTS），其支持期长达五年，而非 LTS 版本的支持期通常只有半年。

### 配置脚本

目标：节省Ubuntu 18.04安装时间，在安装完系统后运行，自动换源和安装常用软件。

位置：<https://github.com/hanyunpeng0521/UbuntuAutoScript-18.04/>

缺点：博主自己写的，功能不是很大，后期会更改

另一个脚本：<https://github.com/the0demiurge/CharlesScripts>

### 优化

#### 设置 root 用户密码
在 Terminal 下输入 `sudo passwd root`
输入当前用户密码，回车
输入新密码，回车，这个密码就是 su 用户的密码。

#### 设置使用 sudo 时免输密码

每次使用 sudo 时都需要输入密码确实烦人, 毕竟是私人电脑, 安全性有锁屏密码保护就可以了, 为了使用方便, 不仿取消使用 sudo 时需要输入 root 用户密码的设定:

同时按下 ctrl + alt + t 打开终端, 输入 sudo visudo , 在打开的文件中, 将
```
%sudo ALL=(ALL:ALL) ALL
```
改为：

```
%sudo ALL=(ALL:ALL) NOPASSWD:ALL
```
即可。

#### 隐藏 grub 引导菜单

修改/etc/grub.d/30_os-prober中的timeout=10为timeout=0 （如果没有这一步，第二步中设置为0秒后会重置为10秒)

接下来修改/etc/default/gru

```
sudo vim /etc/default/grub
```
修改内容为:
```
GRUB_DEFAULT=0
# GRUB_HIDDEN_TIMEOUT=0
# GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
GRUB_DISABLE_OS_PROBER=true
```
更新 grub:
```
sudo update-grub
```
之后开机就不会再出现 grub 引导界面啦, 如果需要临时显示启动页面, 只需要在开机过程中长按 空格 键就好了.

#### 更改软件源

按 win 键召唤 bash 栏, 搜索 update 并在搜索结果中打开 "软件更新器", 软件打开时会自动检查更新, 点 "停止" 即可.

选择 "设置" - "Ubuntu 软件" , 在 "下载自" 下拉列表中选择合适的服务器, 个人推荐 "中国的服务器", 相对来说软件版本会比较新.

取消"ubuntu软件"-"可从互联网下载" - "有版权和合法性问题的软件"。

"更新"推荐，使用太新的软件可能会引入不必要的依赖

![](https://i.loli.net/2019/04/20/5cbaf6424c90d.png)

在"附加驱动"查询适合自己电脑的驱动，这样会加速电脑的运行。

关闭"软件和更新"时会更新缓存。根据需要进行更新软件。

#### 更新软件
```
sudo apt update #更新缓存
sudo apt upgrade #更新软件
sudo apt dist-upgrade #系统更新
```
#### 安装wine

建议使用deepin-wine:<https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu>

#### 运行Android应用

20.04上安装有坑

安装[xDroid](https://www.linzhuotech.com/index.php/home/index/xdroid.html)，安装方法很简单，下载最新版安装包，解压后使用命令行，在解压文件夹目录下，运行 `./install.sh`,之后安装方法和Windows应用相同，安装包会安装到 `/opt/xdroid`下，安装完成后可以通过应用中的商店进行安装Android应用。

![image.png](https://i.loli.net/2019/10/10/AamNoT2YRyvJjwh.png)

#### WeChat & Tim

我个人推荐 deepin-wine-ubuntu 移植的 WeChat 和 Tim, electronic 用户体验不太好.

[wszqkzqk/deepin-wine-ubuntu](https://github.com/wszqkzqk/deepin-wine-ubuntu)

这个项目是 Deepin-wine 环境的 Ubuntu 移植版, 可以在 Ubuntu 上运行 Tim, 微信, 网易云音乐, 百度云网盘, 迅雷等 Windows 软件, 可以说是很良心了. 使用方法参见项目文档.

[electronic-wechat](https://github.com/geeeeeeeeek/electronic-wechat)

Linux 下一种退而求其次的 Wechat 客户端解决方案吧, 虽然不像官方客户端那么好用, 但毕竟比网页版强很多.

下载地址：[Download | electronic-wechat](https://github.com/geeeeeeeeek/electronic-wechat/releases)

#### 安装 Chrome 浏览器

Ubuntu 自带的是 Firefox 浏览器, 直接下载 Chrome 安装包 [Chrome](https://www.google.com/chrome/), 然后使用 Ubuntu 自带软件管理器安装或者使用`sudo dpkg -i  <安装包>`安装。

>Chrome好像在Ubuntu上占用更多内存，不太推荐

#### firefox优化

没有扩展的Firefox，就不是Firefox。——wistone

（但是不要装太多的扩展。）

（1）Adblock Plus（广告已成往事！）这是所有浏览器中广告过滤最好的！！比Chrome强太多了。

（2） IE Tab（让Firefox也能内嵌IE，用双核心）

（3） DownThemAll!（Firefox的批量下载工具）应该花一些时间看看怎么使用，然后你会觉得这是神器！！

（4）如意淘 如果你喜欢购物，这个不可或缺。

（5） all in one gesture ,通过鼠标手势能控制整个浏览器。

#### opera

轻量级可多端同步的浏览器，已经使用三个月，性能功能良好，但功能上还是不是很舒服，比如翻译和截图等都不是很舒服。

官网下载：https://www.opera.com/zh-cn/download?os=linux

直接下载安装即可，建议不要安装安装包里面的源，更新速度超慢，如果需要更新，请从官网下载新的安装包。

#### 安装flash

Adobe Flash 2020年将不再更新。

```
#火狐
$ sudo apt-get install flashplugin-installer
#其他
$ sudo add-apt-repository "deb http://archive.canonical.com/ bionic partner"
$ sudo apt update
$ sudo apt-get install adobe-flashplugin
```

#### 安装 apt-fast
apt-fast是一个为 apt-get 和 aptitude 做的“ shell脚本封装 ”，通过对每个包进行并发下载的方式可以大大减少APT的下载时间。apt-fast使用aria2c下载管理器来减少APT下载时间。就像传统的apt-get包管理器一样，apt-fast支持几乎所有的apt-get功能，如， install , remove , update , upgrade , dist-upgrade 等等，并且更重要的是它也支持proxy。

直白点说, apt-fast 就是一个多线程的 apt-get , 对于我们通过 apt-get 安装软件时尤其有用.

安装命令:
```
sudo add-apt-repository ppa:apt-fast/stable
sudo apt-get update
sudo apt-get -y install apt-fast
```
使用时, 将对应命令中的 apt-get 替换为 apt-fast 即可. 享受多线程飞一般的速度吧!
#### 安装 aptitude
aptitude是一个能够自行解决安装包依赖的命令。

```
sudo apt install aptitude
```

#### 安装 Git / Vim / NeoVim
执行命令:
```
sudo apt install git vim neovim -y
```
即可.

neoVim是vim的扩展项目，功能强大，具体介绍查看官网：https://github.com/neovim/neovim

vim美化：

如果你需要配置vim，只需在Home目录创建一个**~/.vimrc**文件即可以配置vim了，如需安装插件，在~/.vim目录下创建一个bundle文件夹，插件装在里面。（我通过Vundle管理插件，自行百度Vundle怎么使用）,可以参考我的vimrc配置文件：

```
set nocompatible
filetype on
 
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
 
 
" 这里根据自己需要的插件来设置，以下是我的配置 "
"
" YouCompleteMe:语句补全插件
set runtimepath+=~/.vim/bundle/YouCompleteMe
autocmd InsertLeave * if pumvisible() == 0|pclose|endif "离开插入模式后自动关闭预览窗口"
let g:ycm_collect_identifiers_from_tags_files = 1           " 开启 YCM基于标签引擎
let g:ycm_collect_identifiers_from_comments_and_strings = 1 " 注释与字符串中的内容也用于补全
let g:syntastic_ignore_files=[".*\.py$"]
let g:ycm_seed_identifiers_with_syntax = 1                  " 语法关键字补全
let g:ycm_complete_in_comments = 1
let g:ycm_confirm_extra_conf = 0                            " 关闭加载.ycm_extra_conf.py提示
let g:ycm_key_list_select_completion = ['<c-n>', '<Down>']  " 映射按键,没有这个会拦截掉tab, 导致其他插件的tab不能用.
let g:ycm_key_list_previous_completion = ['<c-p>', '<Up>']
let g:ycm_complete_in_comments = 1                          " 在注释输入中也能补全
let g:ycm_complete_in_strings = 1                           " 在字符串输入中也能补全
let g:ycm_collect_identifiers_from_comments_and_strings = 1 " 注释和字符串中的文字也会被收入补全
let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'
let g:ycm_show_diagnostics_ui = 0                           " 禁用语法检查
inoremap <expr> <CR> pumvisible() ? "\<C-y>" : "\<CR>"             " 回车即选中当前项
nnoremap <c-j> :YcmCompleter GoToDefinitionElseDeclaration<CR>     " 跳转到定义处
let g:ycm_min_num_of_chars_for_completion=2                 " 从第2个键入字符就开始罗列匹配项
"
" github 仓库中的插件 "
Plugin 'VundleVim/Vundle.vim'
Plugin 'vim-airline/vim-airline'
"vim-airline配置:优化vim界面"
"let g:airline#extensions#tabline#enabled = 1
" airline设置
" 显示颜色
set t_Co=256
set laststatus=2
" 使用powerline打过补丁的字体
let g:airline_powerline_fonts = 1
" 开启tabline
let g:airline#extensions#tabline#enabled = 1
" tabline中当前buffer两端的分隔字符
let g:airline#extensions#tabline#left_sep = ' '
" tabline中未激活buffer两端的分隔字符
let g:airline#extensions#tabline#left_alt_sep = ' '
" tabline中buffer显示编号
let g:airline#extensions#tabline#buffer_nr_show = 1
" 映射切换buffer的键位
nnoremap [b :bp<CR>
nnoremap ]b :bn<CR>
" 映射<leader>num到num buffer
map <leader>1 :b 1<CR>
map <leader>2 :b 2<CR>
map <leader>3 :b 3<CR>
map <leader>4 :b 4<CR>
map <leader>5 :b 5<CR>
map <leader>6 :b 6<CR>
map <leader>7 :b 7<CR>
map <leader>8 :b 8<CR>
map <leader>9 :b 9<CR>
" vim-scripts 中的插件 "
Plugin 'taglist.vim'
"ctags 配置:F3快捷键显示程序中的各种tags，包括变量和函数等。
map <F3> :TlistToggle<CR>
let Tlist_Use_Right_Window=1
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
let Tlist_WinWidt=25
 
Plugin 'The-NERD-tree'
"NERDTree 配置:F2快捷键显示当前目录树
map <F2> :NERDTreeToggle<CR>
let NERDTreeWinSize=25 
Plugin 'indentLine.vim'
Plugin 'delimitMate.vim'
" 非 github 仓库的插件"
" Plugin 'git://git.wincent.com/command-t.git'
" 本地仓库的插件 "
 
call vundle#end()
 
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"""""新文件标题
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"新建.c,.h,.sh,.java文件，自动插入文件头 
autocmd BufNewFile *.cpp,*.[ch],*.sh,*.java exec ":call SetTitle()" 
""定义函数SetTitle，自动插入文件头 
func SetTitle() 
	"如果文件类型为.sh文件 
	if &filetype == 'sh' 
		call setline(1, "##########################################################################") 
		call append(line("."), "# File Name: ".expand("%")) 
		call append(line(".")+1, "# Author: amoscykl") 
		call append(line(".")+2, "# mail: amoscykl980629@163.com") 
		call append(line(".")+3, "# Created Time: ".strftime("%c")) 
		call append(line(".")+4, "#########################################################################") 
		call append(line(".")+5, "#!/bin/zsh")
		call append(line(".")+6, "PATH=/home/edison/bin:/home/edison/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/work/tools/gcc-3.4.5-glibc-2.3.6/bin")
		call append(line(".")+7, "export PATH")
		call append(line(".")+8, "")
	else 
		call setline(1, "/*************************************************************************") 
		call append(line("."), "	> File Name: ".expand("%")) 
		call append(line(".")+1, "	> Author: amoscykl") 
		call append(line(".")+2, "	> Mail: amoscykl@163.com ") 
		call append(line(".")+3, "	> Created Time: ".strftime("%c")) 
		call append(line(".")+4, " ************************************************************************/") 
		call append(line(".")+5, "")
	endif
	if &filetype == 'cpp'
		call append(line(".")+6, "#include<iostream>")
    	call append(line(".")+7, "using namespace std;")
		call append(line(".")+8, "")
	endif
	if &filetype == 'c'
		call append(line(".")+6, "#include<stdio.h>")
		call append(line(".")+7, "")
	endif
	"	if &filetype == 'java'
	"		call append(line(".")+6,"public class ".expand("%"))
	"		call append(line(".")+7,"")
	"	endif
	"新建文件后，自动定位到文件末尾
	autocmd BufNewFile * normal G
endfunc 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"键盘命令
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
 
nmap <leader>w :w!<cr>
nmap <leader>f :find<cr>
 
" 映射全选+复制 ctrl+a
map <C-A> ggVGY
map! <C-A> <Esc>ggVGY
map <F12> gg=G
" 选中状态下 Ctrl+c 复制
vmap <C-c> "+y
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
""实用设置
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
" 设置当文件被改动时自动载入
set autoread
" quickfix模式
autocmd FileType c,cpp map <buffer> <leader><space> :w<cr>:make<cr>
"代码补全 
set completeopt=preview,menu 
"允许插件  
filetype plugin on
"共享剪贴板  
set clipboard=unnamed 
"从不备份  
set nobackup
"make 运行
:set makeprg=g++\ -Wall\ \ %
"自动保存
set autowrite
set ruler                   " 打开状态栏标尺
set cursorline              " 突出显示当前行
set magic                   " 设置魔术
set guioptions-=T           " 隐藏工具栏
set guioptions-=m           " 隐藏菜单栏
"set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\
" 设置在状态行显示的信息
set foldcolumn=0
set foldmethod=indent 
set foldlevel=3 
set foldenable              " 开始折叠
" 不要使用vi的键盘模式，而是vim自己的
set nocompatible
" 语法高亮
set syntax=on
" 去掉输入错误的提示声音
set noeb
" 在处理未保存或只读文件的时候，弹出确认
set confirm
" 自动缩进
set autoindent
set cindent
" Tab键的宽度
set tabstop=4
" 统一缩进为4
set softtabstop=4
set shiftwidth=4
" 不要用空格代替制表符
set noexpandtab
" 在行和段开始处使用制表符
set smarttab
" 显示行号
set number
" 历史记录数
set history=1000
"禁止生成临时文件
set nobackup
set noswapfile
"搜索忽略大小写
set ignorecase
"搜索逐字符高亮
set hlsearch
set incsearch
"行内替换
set gdefault
"编码设置
set enc=utf-8
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
"语言设置
set langmenu=zh_CN.UTF-8
set helplang=cn
" 我的状态行显示的内容（包括文件类型和解码）
set statusline=%F%m%r%h%w\ [FORMAT=%{&ff}]\ [TYPE=%Y]\ [POS=%l,%v][%p%%]\ %{strftime(\"%d/%m/%y\ -\ %H:%M\")}
set statusline=[%F]%y%r%m%*%=[Line:%l/%L,Column:%c][%p%%]
" 总是显示状态行
set laststatus=2
" 命令行（在状态行下）的高度，默认为1，这里是2
set cmdheight=2
" 侦测文件类型
filetype on
" 载入文件类型插件
filetype plugin on
" 为特定文件类型载入相关缩进文件
filetype indent on
" 保存全局变量
set viminfo+=!
" 带有如下符号的单词不要被换行分割
set iskeyword+=_,$,@,%,#,-
" 字符间插入的像素行数目
set linespace=0
" 增强模式中的命令行自动完成操作
set wildmenu
" 使回格键（backspace）正常处理indent, eol, start等
set backspace=2
" 允许backspace和光标键跨越行边界
set whichwrap+=<,>,h,l
" 可以在buffer的任何地方使用鼠标（类似office中在工作区双击鼠标定位）
set mouse=a
set selection=exclusive
set selectmode=mouse,key
" 通过使用: commands命令，告诉我们文件的哪一行被改变过
set report=0
" 在被分割的窗口间显示空白，便于阅读
set fillchars=vert:\ ,stl:\ ,stlnc:\
" 高亮显示匹配的括号
set showmatch
" 匹配括号高亮的时间（单位是十分之一秒）
set matchtime=1
" 光标移动到buffer的顶部和底部时保持3行距离
set scrolloff=3
" 为C程序提供自动缩进
set smartindent
" 高亮显示普通txt文件（需要txt.vim脚本）
 au BufRead,BufNewFile *  setfiletype txt
"自动补全
:inoremap ( ()<ESC>i
:inoremap ) <c-r>=ClosePair(')')<CR>
":inoremap { {<CR>}<ESC>O
":inoremap } <c-r>=ClosePair('}')<CR>
:inoremap [ []<ESC>i
:inoremap ] <c-r>=ClosePair(']')<CR>
:inoremap " ""<ESC>i
:inoremap ' ''<ESC>i
function! ClosePair(char)
	if getline('.')[col('.') - 1] == a:char
		return "\<Right>"
	else
		return a:char
	endif
endfunction
filetype plugin indent on 
"打开文件类型检测, 加了这句才可以用智能补全
set completeopt=longest,menu
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
```

**安装 Vundle**

它的使用方法很简单，安装一个插件只需要在 ~/.vimrc 按照规则中添加 Plugin 的名称，某些需要添加路径，之后在 Vim 中使用:PluginInstall既可以自动化安装。

1. 先新建目录

```
mkdir ~/.vim/bundle/Vundle.vim
```

2. git 克隆 Vundle 工程到本地

```
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

3. 修改 ~/.vimrc 配置 Plugins。在 ~/.vimrc 文件中添加如下内容并保存(已配置)

   ```
   set nocompatible " be iMproved, required
   filetype off " required
   
   " set the runtime path to include Vundle and initialize
   set rtp+=~/.vim/bundle/Vundle.vim
   call vundle#begin()
   " alternatively, pass a path where Vundle should install plugins
   "call vundle#begin('~/some/path/here')
   
   " let Vundle manage Vundle, required
   Plugin 'VundleVim/Vundle.vim'
   
   " The following are examples of different formats supported.
   " Keep Plugin commands between vundle#begin/end.
   
   " All of your Plugins must be added before the following line
   call vundle#end() " required
   filetype plugin indent on " required
   " To ignore plugin indent changes, instead use:
   "filetype plugin on
   "
   " Brief help
   " :PluginList - lists configured plugins
   " :PluginInstall - installs plugins; append `!` to update or just :PluginUpdate
   " :PluginSearch foo - searches for foo; append `!` to refresh local cache
   " :PluginClean - confirms removal of unused plugins; append `!` to auto-approve removal
   "
   " see :h vundle for more details or wiki for FAQ
   " Put your non-Plugin stuff after this line
   ```

4. 进入 `vim` 运行命令

   ```
   :PluginInstall
   ```

Vundle 命令

```
# 安装插件
:BundleInstall [插件名]
# 更新插件
:BundleUpdate
# 清除不需要的插件
:BundleClean
# 列出当前的插件
:BundleList
# 搜索插件
:BundleSearch
```

**注意**

插件配置不要在 `call vundle#end()` 之前，不然插件无法生效
如果配置错误，需要重新配置后，在vim中运行命令 `:PluginInstall`

#### Git 优化

如果 git log 等命令中中文显示乱码, 可以尝试设置 git config --global core.quotepath false 修复.

另外可以使用以下命令美化 git log :
```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
设置之后运行 git lg , 即可体验更好的 git log 效果.

#### 搜狗输入法
略微考虑了一下，姑且把输入法也算作系统增强的一部分吧。 首先安装 fcitx:
```
sudo apt-get install fcitx-bin fcitx fcitx-table
```
然后下载搜狗输入法 [Download | sougouinput](https://pinyin.sogou.com/linux/) 并安装；

dash 搜索 language 打开“语言支持”，将“键盘输入法系统”改为“fcitx”，重启电脑； 点击屏幕右上角的键盘图标，选择“配置当前输入法”，选择搜狗即可。 参见[搜狗输入法 for linux 安装指南](https://pinyin.sogou.com/linux/help.php)。

#### 使用 alias 简化常用命令
在 Linux 我们可以使用 alias 别名来简化常用命令，直接在 terminal 下输入 alias 就可以查看系统现有别名。

#### 设置快捷键
继承了我在 Windows 下的操作习惯，总是习惯用 win + E 快捷键来打开文件管理器，不妨在 Ubuntu 中设置一样的快捷键，方便日常操作：

- 依次打开 “设置” - “设备” - “键盘”，拉到页面最下方，添加自定义快捷键；
- “名称” 写 “打开文件管理器”， “命令” 写 “nautilus”，不怕自己误操作的可以在 nautilus 前加上 sudo 以管理员权限运行；
- 最后设置快捷键即可；
- 其他命令同理。

#### 安装 unar
Ubuntu 18.04 下使用 nautilus 解压 .zip 文件会产生乱码, 可以使用 unar 工具解压. (ubuntu 18.10 已经修复此问题).

安装命令:
```
sudo apt-get install unar
```
#### 安装 TimeShift

TimeShift 是一款 Linux 下的系统备份还原工具, 支持自动备份和增量备份.

安装命令:
```
sudo add-apt-repository -y ppa:teejee2008/ppa
sudo apt-get update
sudo apt-get install timeshift
```
#### 用TLP降低发热

TLP 是一款有助于系统冷却的应用程序，可以让 Ubuntu 系统运行得更快、更顺畅。安装完成后，运行命令启动它即可，而无需任何配置。
```
sudo add-apt-repository ppa:linrunner/tlp
sudo apt update
sudo apt install tlp tlp-rdw
sudo tlp start
```

#### 安装Preload

Preload（预加载）会在后台工作，以「研究」您如何使用计算机并增强计算机的应用程序处理能力。安装好 Preload 后，您使用频率最高的应用程序的加载速度就会明显快于不经常使用的应用程序。
```
sudo apt install preload
```


### 软件安装
想了一下, 软件也算是系统的一部分, 软件好用了系统用起来自然更舒畅, 所以这里收集了一些自我感觉良好的软件.

#### 系统监控indicator-sysmonitor

安装**indicator-sysmonitor**:https://github.com/fossfreedom/indicator-sysmonitor

```
sudo add-apt-repository ppa:fossfreedom/indicator-sysmonitor
sudo apt-get update
sudo apt-get install indicator-sysmonitor
```

启动：

indicator-sysmonitor &

添加

cpu: {cpu} 内存: {mem} 网络: {net}保存

#### 下载软件:motrix

一款全能的下载工具:https://motrix.app/zh-CN/

#### 截图工具

**deepin-screenshot & deepin-screen-recorder**

之前使用过 deepin 系统的人应该都知道, 深度截图可以说是目前 linux 下最好用的截图工具了, 没有之一.

今天意外发现深度截图居然上架 Ubuntu 应用商店了, 惊喜满满, 直接到 Ubuntu 应用商店下载安装即可.

此外, 深度录屏也可以在 Ubuntu 应用商店安装, 可用于录制屏幕录像或者 GIF 动图.

##### peek

```
sudo add-apt-repository ppa:peek-developers/stable
sudo apt-get update
sudo apt-get install peek
```

#### 音视频软件
**网易云音乐**

网易云音乐算是目前为止 Linux 下最好用的音乐客户端了吧, 直接到 [网易云音乐官网](http://music.163.com/#/download) 下载 deb 安装包，在安装包所在目录运行:
```
sudo dpkg -i ${网易云音乐安装包文件名}
```
即可.

目前网易云音乐客户端有一个小 bug, 必须以 root 身份运行才可以正常使用. 可以通过 alias 别名简化操作, 或者直接在桌面快捷方式中的 Exec 命令前加上 sudo 使快捷方式正常运行.

#### 视频播放器 VLC
支持倍速播放, 界面相对来说也比较美观, 安装命令:
```
 sudo apt-get install vlc
```
#### Thunderbird

```
sudo add-apt-repository ppa:ubuntu-mozilla-security/ppa
sudo apt-get update
sudo apt-get install thunderbird thunderbird-locale-zh-cn
```

#### 办公软件

**XMind ZEN**

超赞的思维导图软件, 下载对应的安装包安装即可

下载地址: [Download | XMind ZEN deb](https://www.xmind.cn/zen/)

当然也可以选择使用在线的作图工具：[ProcessON](https://www.processon.com/i/5ba47363e4b06fc64affadcd)

**WPS Office**

虽然不及 Windows 上面的 Office 那般强大, 但这也确实是 Linux 下的最好选择了.

下载地址: [Download | WPS Office](http://community.wps.cn/download/)

字体文件: [IamDH4/ttf-wps-fonts | Download| WPS Fonts](https://github.com/IamDH4/ttf-wps-fonts)

```
sudo apt-get install -y ttf-wqy-microhei xfonts-wqy ttf-wqy-zenhei --fix-missing
```

当然也可以使用在线的文档编辑：[金山文档](https://account.wps.cn/)

#### MarkDown 编辑器
用户体验上来讲我个人首推 Typora, 但是毕竟 Haroopad 支持 vim 快捷键, 程序员可以尝试一下

**Typora**

Typora 是一款轻量、优雅、跨平台、实时预览的 MarkDown 编辑器。并且可以将 Markdown 文件转化为多种格式输出.

下载地址：[Download | Typora](https://typora.io/#linux)

```
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -

# add Typora's repository

sudo add-apt-repository 'deb https://typora.io/linux ./'

sudo apt-get update

# install typora

sudo apt-get install typora
```

**Vnote**
VNote 是一个受Vim启发开发的专门为 Markdown 而优化、设计的笔记软件。VNote是一个更了解程序员和Markdown的笔记软件。

Vnote 的定义是一款笔记软件, 配合 Github 或者 gitee 使用可以当做云笔记来使用.

下载地址: [Download | Vnote](https://github.com/tamlok/vnote/releases)http://pad.haroopress.com/user.html#download)

**Atom**

Atom 是github专门为程序员推出的一个跨平台文本编辑器。具有简洁和直观的图形用户界面，并有很多有趣的特点：支持CSS，HTML，JavaScript等网页编程语言。它支持宏，自动完成分屏功能，集成了文件管理器。

下载地址:[Download | Atom](https://github.com/atom/atom)

```
sudo add-apt-repository ppa:webupd8team/atom  
sudo apt-get update  
sudo apt-get install atom  
```

#### 卸载使用以下命令：

```
sudo apt-get remove Atom
sudo add-apt-repository --remove ppa:webupd8tem/atom
```

以上只会卸载该软件，要卸载附加的一些软件包，请使用命令：

```
sudo apt-get autoremove
```


#### 创建软链
有一些软件下载之后就是可执行文件, 比如 Telegram, electronic-wechat 等等, 每次运行都要 cd 到软件所在目录也是麻烦, 除了 alias 别名之外还有一种方法就是创建软链, 在 /usr/bin/ 目录下创建软链之后就可以在系统任何地方执行命令了.

创建软链的命令如下:
```
sudo ln -s ${file_path}/${file_name} /usr/bin/${new_command}
```
其中:
```
${file_path} 代表可执行文件所在的路径;
${file_name} 代表可执行文件的文件名;
${new_command} 代表新的命令;
```
之后, 就可以在人以终端输入 ${new_command} 来打开软链指向的程序了.

#### 为应用添加启动图标
依然是针对极个别的可执行文件, 安装之后在 dash 栏是搜索不到的, 因为在 /usr/share/applications/ 目录下没有他们的 .desktop 文件呀, 既然没有, 创建一个便是.

以 electronic-wechat 为例:
```
sudo vim /usr/share/application/electronic-wechat.desktop
```
填入以下内容:
```
[Desktop Entry]
Name=Electronic Wechat
Name[zh_CN]=微信电脑版
Name[zh_TW]=微信电脑版
Exec=/opt/electronic-wechat-linux-x64/electronic-wechat
Icon=/opt/icons_customer/wechat1.png
Terminal=false
X-MultipleArgs=false
Type=Application
Encoding=UTF-8
Categories=Application;Utility;Network;InstantMessaging;
StartupNotify=false
StartupWMClass=electronic-wechat
```
其中:
```
Name 是应用名称, 也就是在 dash 栏搜索是需要输入的内容;
Exec 是可执行程序路径;
Icon 是应用图标, 当 Type=Application 时有效;
StartupWMClass 是图标分类依据, 这个字段值相同的图标会自动被分为一组.
```
其他字段不多说, 自行查找资料吧.

#### 添加托盘图标

Elementary OS颜值很高，但它有个反人类的设计（取消系统托盘）;本文提供一种解决方法，可以给Elementary OS添加系统托盘

```bash
$ sudo add-apt-repository ppa:yunnxx/elementary
$ sudo apt update
$ sudo apt install indicator-application wingpanel-indicator-ayatana
```

编辑配置

```
sudo vim /etc/xdg/autostart/indicator-application.desktop
```

找到这一行

```
OnlyShowIn=Unity;GNOME;
```

在后面添加Pantheon;

```
OnlyShowIn=Unity;GNOME;Pantheon;
```

安装完需要重启一下，回来应该就能看见托盘图标了(*^__^*) 嘻嘻……

#### Utools：多功能搜索控件

你的生产力工具集:https://www.u.tools/

#### 编辑器 VS Code

[微软官网](https://code.visualstudio.com/download)下载deb安装即可。

#### 安装navicat

**相关工具**

- navicat15-premium-cs.AppImage：Navicat 15 premium 官方简体中文试用版
- navicat-patcher：补丁
- navicat-keygen ：注册机
- appimagetool-x86_64.AppImage：Linux 独立运行软件打包工具

[相关工具百度网盘下载地址](https://pan.baidu.com/s/1u01pL0Fz7A0L1sfvU6_ZHg) 『提取码』：jjtr

**系统环境配置**

```
sudo apt-get install libcapstone-dev
sudo apt-get install cmake
git clone https://github.com/keystone-engine/keystone.git
cd keystone
mkdir build
cd build
../make-share.sh
sudo make install
sudo ldconfig
sudo apt-get install rapidjson-dev
```

**操作步骤**

1. 赋予执行权限

   ```
   chmod +x appimagetool-x86_64.AppImage
   chmod +x navicat-patcher
   chmod +x navicat-keygen
   ```

2. 解包官方软件

   ```
   mkdir navicat15
   sudo mount -o loop navicat15-premium-cs.AppImage navicat15
   cp -r navicat15 navicat15-patched
   ```

3. 运行补丁

   ```
   ./navicat-patcher navicat15-patched
   ```

   生成RegPrivateKey.pem文件

   ```
   **********************************************************
   *       Navicat Patcher (Linux) by @DoubleLabyrinth      *
   *                  Version: 1.0                          *
   **********************************************************
   [*] New RSA-2048 private key has been saved to
       ./RegPrivateKey.pem
   *******************************************************
   *           PATCH HAS BEEN DONE SUCCESSFULLY!         *
   *                  HAVE FUN AND ENJOY~                *
   *******************************************************
   
   ```

4. 打包成独立运行软件

   ```
   ./appimagetool-x86_64.Appimage navicat15-patched navicat15-premium-cs-pathed.AppImage
   ```

5. 运行补丁后软件包

   ```
   chmod +x navicat15-premium-cs-pathed.AppImage
   ./navicat15-premium-cs-pathed.AppImage
   ```

6. 运行注册机

   ```
   navicat-keygen --text ./RegPrivateKey.pem 
   ```

   **选择产品类型：** *1 Premium*

   **选择语言：** *1 Simplified Chinese*

   **选择版本：** *15*

   **生成序列号：** *填写至软件注册页面，并在断网后选择手工激活*

   ```
   [*] Serial number:
   NAVC-PJWW-BKN4-C4YW
   ```

   **填写个人信息：**

   ```
   [*] Your name: zenghaiming
   [*] Your organization: hh
   ```

   **输入注册码：** *复制软件激活页面注册码并粘贴此处*

   **生成激活码：** *复制Activation Code内容粘贴至软件激活页面*

打完收工

#### 微信开发者工具安装

前提：

1. `GUI`环境
2. 需要[安装`wine`](https://gitee.com/MrxR/wechat_web_devtools#安装Wine)

```
$ git clone https://git.dev.tencent.com/pingxin0521/wechat_web_devtools.git
$ sudo mv wechat_web_devtools /opt/
$ cd /opt/wechat_web_devtools
# 自动下载最新 `nw.js` , 同时部署目录 `~/.config/wechat_web_devtools/`
$ ./bin/wxdt install
$ ./bin/wxdt # 启动
```

更多参考：https://gitee.com/CodeHero1024/wechat_web_devtools

#### MacOS软件包管理器

MacOS也是类unix操作系统，其brew包管理器也可以在Linux上使用，而且还比较方便安装一些开发环境 

Homebrew（以前称为Linuxbrew）软件包管理器推出的新版本已经支持Linux和Windows 10中的Linux子系统（WSL），也就是说你可以在这两个Linux平台上使用Homebrew了。在Linux或WSL上运行时，Homebrew以前被称为Linuxbrew，它可以安装在你的主目录中，在这种情况下它不使用sudo，Homebrew不使用主机系统提供的任何库，除非glibc和gcc足够新，Homebrew可以为旧的Linux发行版安装自己当前版本的glibc和gcc。

- 可以将软件安装到你的主目录，因此不需要sudo。
- 安装未由主机分发包装的软件。
- 在主机分发较旧时安装最新版本的软件。
- 使用相同的软件包管理器来管理macOS、Linux和Windows系统。

Homebrew目前不支持32位x86平台。

官网：<https://github.com/Homebrew/brew>

**安装**

参考：

1. <https://brew.sh/index_zh-cn>

2. [Ubuntu下查看glibc版本](https://blog.csdn.net/xibeichengf/article/details/48290297)

```bash
sudo apt-get install build-essential curl file git gcc -y
sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"

Next steps:
- Install the Homebrew dependencies if you have sudo access:
  Debian, Ubuntu, etc.
    sudo apt-get install build-essential
  Fedora, Red Hat, CentOS, etc.
    sudo yum groupinstall 'Development Tools'
  See https://docs.brew.sh/linux for more information.
- Configure Homebrew in your ~/.profile by running
    echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >>~/.profile
- Add Homebrew to your PATH
    eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
- We recommend that you install GCC by running:
    brew install gcc
- Run `brew help` to get started


brew install hello
```

**用法**

- `brew install package` - 安装软件包

```bash
$ brew install redis
```

- `brew uninstall package` - 卸载软件包

```bash
$ brew uninstall redis
```

- `brew search package` - 查找软件包

```bash
$ brew search redis
```

- `brew list` - 显示所有已安装软件

```bash
$ brew list
redis
```

- `brew services start package` - 启动服务

```bash
$ brew services start redis
```

- `brew services stop package` - 停止服务

```bash
$ brew services stop redis
```

- `brew services restart package` - 重启服务

```bash
$ brew services restart redis
```

- `brew services list` - 查看所有服务状态

```bash
$ brew services list
Name       Status  User      Plist
redis      started zhangjian /Users/zhangjian/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

- `brew update` - 更新Homebrew到最新版本

```bash
$ brew update
```

**链接**

- [Homebrew官网](https://brew.sh/)
- [Homebrew支持软件列表](https://formulae.brew.sh/formula/)
- [安装](https://www.cnblogs.com/hongdada/p/9528560.html)

### 美化

#### Terminal

在终端中右键->preferences中，禁止菜单，禁止滚动条，以及在配色中选择透明度等配置。

![](https://i.loli.net/2019/04/20/5cbaff2072b34.png)

#### 换用zsh

**bash**这个是目前大多数Linux系统默认使用的shell，全名是BourneAgain Shell，一共有40个命令。包含的功能几乎可以涵盖shell所具有的功能，所以一般的shell脚本都会指定它为执行路径。

在 Linux 里执行这个命令和 Mac 略有不同，你会发现 Mac 多了一个 zsh，也就是说 OS X 系统预装了个 zsh，它是什么呢？

zsh 是一款功能强大的 shell 软件，它可以兼容 bash，并且提供了很多高效的改进。它是Linux里最庞大的一种shell，它有84个内部命令，也提供了更为强大的功能:

- 更好的自动补全
- 更好的文件名展开
- 丰富的插件
- 强大的定制性

但是由于配置过于复杂，一般情况下，我们不会使用该shell，直到「oh my zsh」的出现。

如果你用 Mac，Mac默认已经安装；
如果你用 Redhat Linux，执行：sudo yum install zsh；
如果你用 Ubuntu Linux，执行：sudo apt-get install zsh；

Oh My Zsh(http://ohmyz.sh/)是一款社区驱动的命令行工具，正如它的主页上说的，Oh My Zsh 是一种生活方式。它基于zsh命令行，提供了主题配置，插件机制，已经内置的便捷操作。给我们一种全新的方式使用命令行。

Oh My Zsh只是一个对zsh命令行环境的配置包装框架，但它不提供命令行窗口，更不是一个独立的APP。

**安装**

官网推荐安装方式：

Via curl:

```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Via wget:

```
$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

切换系统shell：

```
sudo chsh -s /bin/zsh $USER
```

**配置**

zsh的配置文件存在当前用户目录中的.zshrc文件，如果你发现切换了shell之后，以前的配置的环境变量不生效了，可以打开 .zshrc文件，找到：

```
# User configuration
source ~/.bash_profile
```

指定配置的环境变量文件，之后运行：

```
source .zshrc
```

更多参考：<https://www.cnblogs.com/monsterdev/p/11166720.html>

后续需要更改vscode、idea终端的字体

**优化**

sh 的交互式体验堪称是最强的——丰富的插件，强大的框架，将 zsh 的交互式体验推向了极致。然而另一方面，过多的插件，臃肿的主题，也让 zsh 变得反应迟钝，反过来破坏了交互式体验。

很多人反映的 zsh 的慢主要体现在两个方面：1. 启动速度，2. prompt 反应速度慢

影响第一点的主要是插件的数量和质量，影响第二点的则是主题。 因此不少人想到的第一个解决方案就是精简插件和主题。诚然，这是最有效的解决方案，但是这就和使用 zsh 的初衷相违背了：使用 zsh，不就是希望自己的 shell 能够用起来更方便么？

众所周知，尽管不少人会给 zsh 配上一堆插件，但很多插件都不是启动一个 shell 以后立马就需要使用的。 比如说 thefuck，这个插件只有在你打错的命令的时候才会被激活。然而为此你却需要在启动时花费数百毫秒来加载，这显然是一种浪费，为什么不能将这类插件的加载延迟到 zsh 启动以后呢？

zinit 就是在这个方向上的一次成功尝试。它提供了 “Turbo Mode”，允许延迟加载插件，一般来讲可以加速 50% 到 70%！

1. 安装

   刚正朴实的安装方式，官方推荐

   ```bash
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/zdharma/zinit/master/doc/install.sh)"
   ```

2. 配置

   作者的配置：[https://github.com/zdharma/zinit-configs/blob/master/psprint/zshrc.zsh](https://link.zhihu.com/?target=https%3A//github.com/zdharma/zplugin-configs/blob/master/psprint/zshrc.zsh)
   非常复杂，用到了很多 zinit 的高级功能，好处是兼容性强，不必依赖包管理来安装插件（比如 fzf, ripgrep 一类的）

#### 主题

首先安装 tweaks，使用该软件统一管理主题中的各个部分

```
sudo apt install gnome-tweak-tool
```
从Ubuntu软件中安装常用插件：
```
shell拓展blyr 类似Mac模糊的拓展
dash to dock 比Ubuntu dock更强大
hide top bar 顶部栏隐藏
netspeed 网速显示
topicons plus 托盘显示
```

删除扩展：

1. 找到扩展[文件](http://www.onlinedown.net/soft/267407.htm)。打开 /usr/share/gnome-shell/extensions目录，用root权限删除目录中的文件。

　2. 将~/.local/share/gnome-shell目录下的[文件删除](http://www.onlinedown.net/soft/629588.htm)。

　3. alt+f2后输入r重新启动gnome-shell后。

而具体的各种图标/shell theme/gtk theme等可以在[https://www.gnome-look.org/](https://www.gnome-look.org/)找到，非常多，下面选择几个我觉得好看的设置。

**Icon**

使用[McMojave-circle](https://github.com/vinceliuice/McMojave-circle),配置非常简单：

```bash
git clone https://github.com/vinceliuice/McMojave-circle.git
cd McMojave-circle
sudo ./install.sh
```

**主题**

使用**[ Mojave-gtk-theme](https://github.com/vinceliuice/Mojave-gtk-theme)**，安装配置很简单

```
sudo apt-get install gtk2-engines-murrine gtk2-engines-pixbuf
sudo apt install sassc libcanberra-gtk-module libglib2.0-dev
git clone https://github.com/vinceliuice/Mojave-gtk-theme.git
cd Mojave-gtk-theme
sudo ./install.sh
```
安装主题，并在TWeaks->外观-> 应用程序(或者Shell) 中打开

如果Shell显示不可修改，
```
sudo apt install gnome-shell-extensions
```

即可，更多配置参考文档

**Dock**

```
sudo apt install gnome-shell-extension-dash-to-dock
```
安装插件，并在TWeaks->extensions中打开
然后启动dash to Dock,按照自己的喜好配置。

**grup**

Flat Design themes for Grub2:<https://github.com/vinceliuice/grub2-themes>

安装方法：`sudo ./install.sh `

**cursors**

McMojave-cursors: <https://github.com/vinceliuice/McMojave-cursors>

```
git clone https://github.com/vinceliuice/McMojave-cursors.git
sudo ./install.sh
```

#### 登录桌面美化

我已经从将脚本上传到网上，需要的请下载后进行配置,选择stwaskpass。

链接：https://pan.baidu.com/s/1WDk8Le0LcAE7b0vYwlAYag 密码：bmf9


下载之后解压，执行里面的`./install.sh`文件。然后右键选择图片，使用脚本->SetAsWallpaper,当前图片就会变成壁纸，顺便会把图片模糊处理并设置成锁屏壁纸，输入密码后完成，退出登录后查看效果。

如果没有起效果，请更改`/usr/share/gnome-shell/theme/gdm3.css`，找到指定位置更改内部内容即可：
```
#lockDialogGroup {
    background: #2c001e url(file:///usr/share/backgrounds/gdmlock.jpg);
    /*lockscreen wallpaper*/
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
}
```

#### 隐藏登录界面中的用户

首先跳转到以下目录
```
cd /var/lib/AccountsService/users/
```
ls后可以看到你想要隐藏的某个用户文件，然后打开该文件
```
vim username
```
进行更改
```
[User]
SystemAccount=false  //将false改为true
```

#### 安装windows字体

Windows的字体一般存放在c:\windows\fonts目录下，打开以后看到那么多的字体文件，不管那么多了全部Copy过来，我是直接复制到了Ubuntu中/usr/share/fonts/zh_CN/这个文件夹下面，其中zh_CN这个文件夹是我自己新建的，拷贝完成后更新字体缓存，命令如下：

` sudo fc-cache`

现在，差不多在Ubuntu系统中就可以使用刚才你拷贝的那些XP字体了，为了让整体显示更加好看，还可以修改/etc/fonts/fonts.conf这个文件，对字体的渲染顺序进行调整。