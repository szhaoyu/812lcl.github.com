---
layout: post
title: "Linux命令学习（2）：rsync"
date: 2014-05-04 15:18
comments: true
categories: Linux
tags: [linux, rsync]
---
rsync是Linux下进行文件同步到一个命令，可以同步两台计算机到文件与目录，利用查找文件中到不同块以减少数据传输。也可以在一台电脑到不同目录间同步，比如可以写个简单到脚本，将系统中你到一些配置文件备份到一个 dotfiles 文件夹，上传到 GitHub 以便以后新的电脑或系统再利用 rsync 恢复这些配置文件，这是很方便自动化的（我就是这么干的 [dotfiles](https://github.com/812lcl/dotfiles))。

###rsync的基本用法

    $ rsync [options] src dest

1. 目的端和源端文件内容不同，触发同步
2. rsync不同步文件到"modify time"，进行同步到文件，目的端到"modify time"总是被修改为最新时刻到时间。
3. rsync不会太关注目的端文件到rwx权限，如果目的端无此文件，则权限与源端保持一致；如果目的端有此文件，则权限不会随着源端变更。
4. rsync有对源文件到读权限，且对目标路径有写权限，rsync就能确保目的端文件同步到和源端一致。
5. rsync只能以登录目的端到帐号来创建文件，无法保持目的端文件到属主和属组和源端一致。

<!--more-->

####[-t选项]
1. 使用-t选项后，rsync总会想着一件事，那就是将源文件的“modify time”同步到目标机器。
2. 带有-t选项的rsync，会变得更聪明些，它会在同步前先对比两边文件的时间戳和文件大小，如果一致，则就认为两边文件一样，对此文件就不再采取更新动作了。
3. 因为rsync的聪明，也会反被聪明误。如果目的端的文件的时间戳、大小和源端完全一致，但是内容恰巧不一致时，rsync是发现不了的。这就是传说中的“坑”！
4. 对于rsync自作聪明的情况，解决办法就是使用-I选项。

####[-I选项]
1. -I选项会让rsync变得很乖很老实，它会挨个文件去发起数据同步。
2. -I选项可以确保数据的一致性，代价便是速度上会变慢，因为我们放弃了“quick check”策略。（quick check策略，就是先查看文件的时间戳和文件大小，依次先排除一批认为相同的文件）
3. 无论情况如何，目的端的文件的modify time总会被更新到当前时刻。

####[-v选项]
这个选项，简单易懂，就是让rsync输出更多的信息，你增加越多的v，就可以获得越多的日志信息。

####[-z选项]
这是个压缩选项，只要使用了这个选项，rsync就会把发向对端的数据先进行压缩再传输。对于网络环境较差的情况下建议使用。一般情况下，-z的压缩算法会和gzip的一样。

####[-r选项]
我们在第一次使用rsync时，往往会遇到这样的囧境：

    $ rsync superman machineB:/home/userB
    skipping directory superman

如果你不额外告诉rsync你需要它帮你同步文件夹的话，它是不会主动承担的，这也正是rsync的懒惰之处。所以，如果你真的想同步文件夹，那就要加上-r选项，即recursive（递归的、循环的），像这样：

    $  rsync -r superman machineB:/home/userB

对于文件夹，rsync是会明察秋毫的，只要你加了-r选项，它就会恪尽职守的进入到文件夹里去检查，而不会只对文件夹本身做“quick check”的。

####[-l选项]
rsync一旦发现某个文件是软链接，就会无视它，除非我们增加-l选项。使用了-l选项后，rsync会完全保持软链接文件类型，原原本本的将软链接文件复制到目的端，而不会“follow link”到指向的实体文件。如果我偏偏就想让rsync采取follow link的方式，那就用-L选项就可以了。你可以自己试试效果。

####[-p选项]
这个选项的全名是“perserve permissions”，顾名思义，就是保持权限。如果你不使用此选项的话，rsync是这样来处理权限问题的：

1. 如果目的端没有此文件，那么在同步后会将目的端文件的权限保持与源端一致；
2. 如果目的端已存在此文件，那么只会同步文件内容，权限保持原有不变。

如果你使用了-p选项，则无论如何，rsync都会让目的端保持与源端的权限一致的。

####[-g选项和-o选项]
这两个选项是一对，用来保持文件的属组(group)和属主（owner），作用应该很清晰明了。不过要注意的一点是，改变属主和属组，往往只有管理员权限才可以。

####[-D选项]
-D选项，原文解释是“preserve devices(root only)”，从字面意思看，就是保持设备文件的原始信息。

####[-a选项]
1. -a选项是rsync里比较霸道的一个选项，因为你使用-a选项，就相当于使用了`-rlptgoD`这一坨选项。以一敌七，唯-a选项也。
2. -a选项的学名应该叫做archive option，中文叫做归档选项。使用-a选项，就表明你希望采取递归方式来同步，且尽可能的保持各个方面的一致性。
3. 但是-a无法同步“硬链接”情况。如果有这方面需求，要加上-H选项。

####[--delete选项、--delete-excluded选项和--delete-after选项]
三个选项都是和“删除”有关的：

1. –delete：如果源端没有此文件，那么目的端也别想拥有，删除之。（如果你使用这个选项，就必须搭配-r选项一起）
2. –delete-excluded：专门指定一些要在目的端删除的文件。
3. –delete-after：默认情况下，rsync是先清理目的端的文件再开始数据同步；如果使用此选项，则rsync会先进行数据同步，都完成后再删除那些需要清理的文件。

这个学习可是要小心使用到，一不小心会删除很多东西哦。

可以使用-n选项，它会用受影响的文件列表来警告你，但不会真的去删除，这就让我们有了确认的机会和回旋的余地。

####[--exclude选项和--exclude-from选项]
如果你不希望同步一些东西到目的端的话，可以使用–exclude选项来隐藏，rsync还是很重视大家隐私的，你可以多次使用–exclude选项来设置很多的“隐私”。
如果你要隐藏的隐私太多的话，在命令行选项中设置会比较麻烦，rsync还是很体贴，它提供了–exclude-from选项，让你可以把隐私一一列在一个文件里，然后让rsync直接读取这个文件就好了。

####[--partial选项]
这就是传说中的断点续传功能。默认情况下，rsync会删除那些传输中断的文件，然后重新传输。但在一些特别情况下，我们不希望重传，而是续传。
我们在使用中，经常会看到有人会使用-P选项，这个选项其实是为了偷懒而设计的。以前人们总是要手动写–partial –progress，觉得太费劲了，倒不如用一个新的选项来代替，于是-P应运而生了。

####[--progress选项]
使用这个选项，rsync会显示出传输进度信息，有什么用呢，rsync给了一个很有意思的解释：
This gives a bored user something to watch.

###rsync核心算法
rsync的算法十分高效，酷壳上 [一篇文章](http://coolshell.cn/articles/7425.html) 介绍来其核心算法，就不再赘述来。

##rsync服务器架设
可以架设rsync服务器，rsync以守护进程运行，客户端将rsync指令写成一个shell脚本，通过crontab定期执行脚本，以实现服务器和客户端间特定文件或目录到同步，这样就不需要你每次手动同步来，而且rsync服务器配置也十分简单。

主要涉及三个文件：

1. /etc/rsyncd.conf 是rsync服务器主要配置文件
2. /etc/rsyncd.secrets 是登录rsync服务器到密码文件
3. /etc/rsyncd.motd 是定义用户登录时显示到信息

###rsyncd.conf
文件内容大体如下：

```
# Minimal configuration file for rsync daemon
# See rsync(1) and rsyncd.conf(5) man pages for help

# This line is required by the /etc/init.d/rsync script
pid file = /var/run/rsyncd.pid
motd file = /etc/rsyncd.motd
port = 873
address = 192.168.20.24
#uid = nobody
#gid = nobody
uid = root
gid = root
use chroot = yes
read only = no

# limit access to private LANs
hosts allow = *
hosts deny = *

max connections = 5

# This will give you a separate log file
log file = /var/log/rsync.log

# This will log every file transferred - up to 85,000+ per user, per sync
# transfer logging = yes

log format = %t %a %m %f %b
syslog facility = local3
timeout = 300

[test]
path = /home/lcl/test
list = yes
ignore errors
auth users = lcl
secrets file = /etc/rsyncd.secrets
comment = lcl test
#exclude = samba/
```

rsync使用到是873好端口，上面到配置中`hosts allow`是允许使用这台rsync服务到机器到IP地址，`hosts deny`则是拒绝到，不同IP以空格隔开。

[test]及之后到配置则是，配置rsync目录，包括目录到路径，使用到用户，密码文件，可以使用`exclude`排除不要同步到文件或目录。

###rsyncd.secrets
这是存储rsync服务用户到用户名和密码，是一个明文文本文件，非常重要，属性需要设为600，只允许所有者读写，文件格式如下：

    lcl:12345

###rsyncd.motd
文件记录来rsync服务到欢迎信息，当用户使用该服务时会显示，可以设置为任何文本信息，如：

    ++++++++++++++++++++++++++++++++++++++
    + Welcome to use lcl rsync services! +
    ++++++++++++++++++++++++++++++++++++++

配置完以上文件，可以启动rsync服务来：

    rsync --daemon

客户端可以通过rsync命令活脚本来进行同步数据来

rsync的命令格式可以为：

1. rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST]
2. rsync [OPTION]... [USER@]HOST:SRC DEST]
3. rsync [OPTION]... SRC [SRC]... DEST]
4. rsync [OPTION]... [USER@]HOST::SRC [DEST]
5. rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST]
6. rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]

rsync有六种不同的工作模式：

1. 拷贝本地文件；当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。
2. 使用一个远程shell程序（如rsh、ssh）来实现将本地机器的内容拷贝到远程机器。当DST地址包含单个冒号":"分隔符时启动该模式。
3. 使用一个远程shell程序（如rsh、ssh）来实现将远程机器的内容拷贝到本地机器。当SRC路径包含单个冒号":"分隔符时启动该模式。
4. 从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。
5. 从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。
6. 列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。

参考文章：

- [rsync同步的艺术](http://roclinux.cn/?p=2643)
- [rsync服务器架设（数据同步|文件增量备份)](https:www.centos.bz/2011/06/rsync-server-setup)
