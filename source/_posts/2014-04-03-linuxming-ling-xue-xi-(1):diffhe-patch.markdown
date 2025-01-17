---
layout: post
title: "Linux命令学习（1）：diff 和 patch"
date: 2014-04-03 21:03
comments: true
categories: Linux
tags: [linux, diff, patch]
---
diff 命令是 Linux 上非常重要的工具，用于比较文件甚至目录的内容，清晰的告诉你前后改动的地方。diff 可以输出为补丁(patch) ，Linux 中还有一条命令patch，可以根据补丁文件，对文件更新修改。当你和别人合作或想为开源项目提供贡献时，可以将自己的修改打成补丁，邮件给合作者，他即可合并你的代码。因此diff也是svn、cvs、git等版本控制工具不可或缺的一部分。

diff
====
diff的基本格式如下：

    $ diff [参数] <变动前的文件> <变动后的文件>

命令参数有以下常用的一些，可以根据不同的模式选用：

    -b 或--ignore-space-change 　   不检查空格字符的不同。
    -B 或--ignore-blank-lines 　    不检查空白行。
    -w 或--ignore-all-space     　  忽略全部的空格字符。
    -i 或--ignore-case 　           不检查大小写的不同。
    -q 或--brief 　                 仅显示有无差异，不显示详细的信息。

<!--more-->
在diff目录时常用的参数如下：

    -r 或--recursive 　             比较子目录中的文件。
    -N 或--new-file 　              文件A仅出现在某个目录中，预设会显示：Only in目录
                                    文件A若使用-N参数，则diff会将文件A与一个空白的文件比较。
    -P或--unidirectional-new-file 　与-N类似，只有当第二个目录包含了一个第一个目录所没有的文件时，
                                    才会将这个文件与空白的文件做比较。

diff有大概四种格式：**正常格式**、**并列格式**、**上下文格式**、**合并格式**。

还有git diff与vimdiff模式。

diff可以比较文本文件或目录之间的差异，首先来讲解文本文件之间的diff，示例文件如下：

文件f1

    a b
    b
    c
    d
    e

文件f2

    a b c
    b
    d
    e
    f

## 正常格式diff ##
正常格式不用加任何参数，`diff f1 f2`即可，显示结果如下：

    1c1
    < a b
    ---
    > a b c
    3d2
    < c
    5a5
    > f

`1c1`、`3d2`、`5a5`是说明变动的位置，前边数字代表f1中变化的行，后边的则代表f2中变化的行，中间的字母分别代表“改变”（c）、“删除”（d）和“增加”（a）。

`<`表示f1指定行的内容，`---`分割两个文件的，然后`>`表示f2指定行的内容。删除或增加时，则分别f2、f1中指定行无内容。

## 并列格式diff ##

其他格式diff都是先后显示两个文件的内容变化，并列格式可以并排显示两个文件的内容变化，更形象的看出文件的变化，和vimdiff显示的有些相似。

使用方法为加入`-y`参数，即可并列显示，`-W num`参数可设定并列的宽度，可以不使用。

    diff -y -W 50 f1 f2
    
结果如下：

    a b                   | a b c
    b                       b
    c                     <
    d                       d
    e                       e
                          > f

`|`说明此行有变化，`<`说明此行被删除了，`>`说明此行是后增加的。

## 上下文格式diff ##

标准格式diff显示的内容不够直观，上下文格式则通过显示变化的上下文，而更加的利于理解。

使用方法为使用参数`-c`参数： `diff -c f1 f2`，结果如下：

    *** f1  2014-04-03 21:24:23.581007082 +0800
    --- f2  2014-04-03 21:24:21.324995895 +0800
    ***************
    *** 1,5 ****
    ! a b
      b
    - c
      d
      e
    --- 1,5 ----
    ! a b c
      b
      d
      e
    + f

首先，显示两个文件的基本情况：

    *** f1  2014-04-03 21:24:23.581007082 +0800
    --- f2  2014-04-03 21:24:21.324995895 +0800
    
`***`表示变动前的文件f1，`---`表示变动后的文件f2。

然后，15个星号将文件的基本情况与变动内容分割开。

最后，显示f1和f2文件的内容变动情况。

`*** 1,5 ****`表示f1文件的1~5行

`--- 1,5 ----`表示f2文件的1~5行

`!`代表次行内容有变动，`+`表示此行为新增加，`-`表示此行被删除了。

上下文格式默认显示包括修改行前后的三行内容，可以使用`-num`来设置显示前后num行，如：

    diff -c -1 f1 f2

## 合并格式diff ##

两个文件大量内容重复，上下文格式将显示很多无用干扰信息，后来就退出了合并式diff。

使用方法为，加入`-u`参数，`diff -u f1 f2`，结果如下：

    --- f1  2014-04-03 21:24:23.581007082 +0800
    +++ f2  2014-04-03 21:24:21.324995895 +0800
    @@ -1,5 +1,5 @@
    -a b
    +a b c
     b
    -c
     d
     e
    +f

同样前两行表示两个文件的基本情况

然后`@@ -1,5 +1,5 @@`表示修改的位置，`-`代表 f1 的1~5行，`+`代表 f2 的1~5行。

最后是合并显示的变动具体内容，依旧是`-`代表f1，`+`代表f2。

同上下文格式一样，合并格式也是默认显示修改前后3行的内容，也可以使用`-num`来设置显示前后num行：

    diff -u -1 f1 f2

这里还要提到，git使用的是合并格式diff的变体。当前工作目录下`git add f1`后，修改f1的内容，可以使用如下命令，观察做出的修改

    git diff

结果如下：

    diff --git a/f1 b/f1
    index 924a897..c3b09ff 100644
    --- a/f1
    +++ b/f1
    @@ -1,5 +1,5 @@
    -a b
    +a b c
     b
    -c
     d
     e
    +f

和合并格式diff的区别在头部文件基本信息，`git diff`显示的是a、b两个版本的f1文件的内容变化，并且显示了两个版本的git哈希值（924a897..c3b09ff），与文件的权限（644）。

## vimdiff ##
Vim提供的diff模式称作imdiff，可以很清晰形象的观察到文件内容的变化，方便的进行合并工作。vimdiff的使用方法如下：

    vimdiff f1 f2
    或者
    vim -d f1 f2

下图为结果的画面

![vimdiff](http://812lcl.github.io/images/blog/vimdiff.png)

默认屏幕垂直分割，对比显示两个文件的不同，如果想要水平分割可以使用参数`-o`（不过怎么也是垂直的好看）。这里可以看到f1和f2中都有的但是内容改变的行被高亮为红色，次行内修改的具体位置被高亮为黄色；f1里有但是在f2中被删除的行，f1中显示为蓝色，f2中显示为绿色；相反，f2中增加的行显示为蓝色，f1中相应位置显示为绿色。这样更突出引起差异的地方，并且如果文件内容较多，连续相同的行会折叠起来，可以使用`zo`和`zc`打开和关闭折叠。

对不同修改内容的高亮显示，颜色可以自己自定义，在自己的Vim配置文件`~/.vimrc`中添加如下语句：

    hi DiffAdd ctermbg=blue ctermfg=white
    hi DiffDelete ctermbg=green ctermfg=none
    hi DiffChange ctermbg=red ctermfg=White
    hi DiffText ctermbg=yellow ctermfg=black

使用GUI的话可以配置`guibg`和`guifg`。

切换不同的窗口可以使用下列命令：

    Ctrl-w l        切换到右边窗口
    Ctrl-w h        切换到左边窗口
    Ctrl-w j        切换到下边窗口
    Ctrl-w k        切换到上边窗口

在编辑文件时也可以使用命令模式启动vimdiff模式：

    vim f1
    :vertical diffsplit f2

如果不加`vertical`默认使用的水平分割。在Vim中除了`diffsplit`还有一些其他的命令，利于对文件进行合并和其他操作。

    :diffget        把差异点中另一个文件对应的内容复制到当前行
    :diffput        把差异点中当前行的内容复制到另一个文件对应的位置
    :diffpatch      根据补丁文件更新文件内容，后面需要跟一个参数指定文件
    :diffupdate     手动刷新比较结果

    :qa             同时退出两个文件
    :wqa            同时保存并退出

如果Vim安装来[vim-fugitive](https://github.com/tpope/vim-fugitive)插件，还可以使用`:Gdiff`命令来比较当前文件和暫存区中的文件的不同，也是利用的vimdiff，显示和操作同上。

## 目录的diff ##

目录间的diff比较目录中相同文件名的文件，也可以使用正常格式、上下文格式、合并格式、并列格式。

示例目录为d1和d2目录，d1中有文件f1、f3，d2中有f1、f2，其中f1有一些改动。

如果不是用其他参数，不会递归比较子目录中的文件，会显示只存在目录一个目录中的文件，但不显示其详细信息，如下结果是使用`diff -u d1 d2`的结果：

    diff -u d1/f1 d2/f1
    --- d1/f1   2014-04-03 19:29:25.910803383 +0800
    +++ d2/f1   2014-04-03 19:28:52.918639783 +0800
    @@ -1,5 +1,5 @@
    -a b c
    +a b
    b
    +c
    d
    e
    -f
    只在 d2 存在：f2
    只在 d1 存在：f3

在目录的diff中常使用的参数是`-ruN`，r表示递归比较子目录中的文件，u是合并格式，N表示diff会将只存在与某个目录中的文件与一个空白的文件比较。

patch
====
将diff的输出重定向到文本文件中，即得到补丁文件(patch)，可以使用patch命令对文本文件或目录打补丁，从而进行内容更新。

patch的基本用法

    patch [参数] <patchfile>

参数：

    -p Num  忽略几层文件夹
    -E      选项说明如果发现了空文件，那么就删除它
    -R      取消打过的补丁。

如果使用参数`-p0`，表示从当前目录找打补丁的目标文件夹，再对该目录中的文件执行patch操作。
而使用参数`-p1`，表示忽略第一层目录，从当前目录寻找目标文件夹中的子目录和文件，进行patch操作。

## 处理单个文件补丁 ##

产生补丁

    diff -uN f1 f2 > file.patch

打补丁

    patch -p0 < file.patch
    或者
    patch f1 file.patch

取消补丁

    patch -RE -p0 < file.patch
    或者
    patch -RE f1 file.patch

## 处理目录补丁 ##

产生补丁

    diff -urN d1 d2 > dir.patch

打补丁

    cd d1
    patch -p1 < ../dir.patch

取消补丁

    patch -R -p1 < ../dir.patch

应用补丁时的目标代码和生成补丁时的代码未必相同，打补丁操作可能失败，补丁失败的文件会以.rej结尾。

参考文章：

- [读懂diff - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html)

- [每天一个linux命令（36）：diff 命令](http://www.cnblogs.com/peida/archive/2012/12/12/2814048.html)

- [补丁(patch)的制作与应用 - Linux Wiki](http://linux-wiki.cn/wiki)

- [技巧：Vimdiff 使用 - IBM](http://www.ibm.com/developerworks/cn/linux/l-vimdiff/index.html)
