---
layout: post
title: Linux 创建文件别名
date: 2015-01-01 23:13:00
---

在做项目的时候操作服务器或者常常使用命令行操作系统的开发人员肯定会很多时候经常进入同一个繁琐的目录或者运行固定位置的shell脚本。



为了方便使用，Linux系统提供了一个有用的工具叫alias，可以让我们将一些需要频繁使用的但又过于冗长的命令设置一个别名，这样一来，以后只需输入一个简短的别名就可以达到同样的作用。、



用法：alias [-p] [name[=value] ... ]注意‘=’和字符串之间不能包含空格

显示当前设置的别名：

shell>alias-p

aliasl.='ls -d .* --color=tty'

aliasll='ls -l --color=tty'

aliasls='ls --color=tty'

aliasvi='vim'

aliaswhich='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'

或者直接输入：

shell>alias-p

aliasl.='ls -d .* --color=tty'

aliasll='ls -l --color=tty'

aliasls='ls --color=tty'

aliasvi='vim'

aliaswhich='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'



若只想显示某个别名代表的含义可输入alias name，比如:

shell>aliasll

aliasll='ls -l --color=tty'

若想为某个命令设置别名可输入 alias新命令='原命令 选项/参数'，比如：

shell>aliassite='cd /var/www/site/mycitsm/'

若想取消某个别名可输入unalias name，比如

shell>unaliassite



PS：以上的设置只是对当前的有效，当重新连接和重新启动的情况下就别名就不可以再次被使用。



可以通过将设置别名的命令写进启动文件使别名持久生效。大多数Linux发行版使用下述三个启动文件中的一个：

$HOME/.bash_profile

$HOME/.bash_login

$HOME/.profile

可将设置别名的命令写进启动文件内，这样，每次连入系统的时候别名都会生效。若想在命令写入启动文件后立即生效记得执行source命令，比如：

source$HOME/.bash_profile

通过上述方式设置命令别名解决了命令别名只针对回话生效的问题，但是，写进每个用户特定的主目录下的启动文件中的命令别名只针对该用户有效。对其他用户没有什么效果，这通常也是正常情况下期望看到的情况。但如果确实像使设置的别名对任意用户有效则可将设置别名的命令写进全局启动文件中，如/etc/profile。