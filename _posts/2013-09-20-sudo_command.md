---
layout: post
title: "sudo 命令情景分析"
description: ""
category: Linux 
tags: [command, sudo]
---
{% include JB/setup %}


Linux 下使用 `sudo` 命令，可以让普通用户也能执行一些或者全部的 root 命令。本文就对我们常用到 sudo 操作情景进行简单分析，通过一些例子来了解 sudo 命令相关的技巧。

## 情景一：用户无权限执行 root 命令
普通用户登录 shell 之后，如果自身没有权限访问某个文件或执行某个命令时，若该用户获得root授权，那么就可以在需要执行的命令之前加上 sudo，临时切换到root用户的权限，完成相关的操作。

在sudo于1980年前后被写出之前，一般用户管理系统的方式是利用su切换为超级用户。但是使用su的缺点之一在于必须要先告知超级用户的密码，而sudo使一般用户不需要知道超级用户的密码即可获得权限。

那么哪些用户可以临时获得 root 权限呢？这就需要在 /etc/sudoers 文件中进行配置：

** 授权给单个用户：**
{% highlight bash %}
# User privilege specification
guohl   ALL=(ALL) ALL
{% endhighlight %}
上面这个例子中：

* guohl：允许使用 sudo 的用户名
* ALL：允许从任何终端（任何机器）使用 sudo
* (ALL)：允许以任何用户执行 sudo 命令
* ALL：允许 sudo 权限执行任何命令

如果我们想让用户 test 只能在本主机（主机名为guohl-pc）以 root 账户执行/bin/chown、/bin/chmod 两条命令，那么就应该这样配置：

{% highlight bash %}
# User privilege specification
test   guohl-pc=(root) /bin/chown,/bin/chmod
{% endhighlight %}

如果test 登录之后运行 sudo 命令，不满足上面三个条件命令均失败。

** 授权给用户组: **
{% highlight bash %}
# Allow members of group sudo to execute any command
# (Note that later entries override this, so you might need to move it further down)
%sudo ALL=(ALL) ALL
{% endhighlight %}
和授权给单个用户类似，只不过将用户名在这里换成`%组名`，所有在该组中的用户都按照此规则进行授权。对于该例，所有在 sudo 组内的用户都有在任何终端（第一个ALL）、以任何用户（第二个ALL）、执行任何命令（第三个ALL）的权限，查看 /etc/group 文件可以知道哪些用户属于 sudo 组。

** 举例: **

如果当前帐号在 /etc/sudoers 文件中被授予 sudo 的权限，那么你就可以将任何 root 命令作为 sudo 命令的参数，使用 root 权限来执行该命令。举例来说，挂载一个文件系统只能由 root 来执行，但是一个普通用户也可以使用 sudo 来挂载：
{% highlight bash %}
$sudo mount /dev/sda7 /mnt
[sudo] password for guohailin:
{% endhighlight %}

首次使用会要求你输入当前用户的密码，系统确实输入正确即以 root 权限来执行 mount 命令，接下来一段时间（默认为5分钟）再次使用 sudo 命令就不需要输密码了。


## 情景二：vim 编辑后发现忘记使用 sudo
我们经常会遇到这样的一个囧境：使用 vim 对某个文件进行编辑，编辑完之后，按 `ESC` 之后回到普通模式，再按 `:wq` 准备保存退出时，发现没有权限对该文件进行修改，我们在使用 vim 命令时忘记在前面加 sudo 了。我就经常出现这种问题，之前的做法是只能不保存强退，再加上 sudo 重新编辑。

但是今后我们再也不需要用这么愚蠢的做法了，我们可以在 vim 的普通模式下，按 `:w !sudo tee %` ，这样就可以 root 权限来保存文件了，你也无需因为自己一时忘记加个 sudo 而沮丧懊恼了！

## 情景三：执行 root 命令忘记加 sudo
我们还会遇到这样稍微好一点的情形：输入一个长长的命令，按 `Enter` 之后出现无权限操作，因为我们忘记加 sudo 了。大多人的做法是按 `↑` 回到上一条命令，在该命令之前加上 sudo，再执行该命令。

以后，我们无需这样了，只要输入 `sudo !!` 即可，这里的 !! 代表上一条命令。如：

{% highlight bash %}
$ head -n 4 /etc/sudoers
head: cannot open `/etc/sudoers' for reading: Permission denied

$ sudo !!
sudo head -n 4 /etc/sudoers
# /etc/sudoers
#
# This file MUST be edited with the 'visudo' command as root.
#
{% endhighlight %}

## 情景四：shell 内置命令如何使用 sudo 
shell 是一个交互式的应用程序，在执行外部命令时通过 fork 来创建一个子进程，再通过 exec 来加载外部命令的程序来执行，但是如果一个命令是 shell 内置命令，那么只能直接由 shell 来运行。sudo 的意思是，以别的用户（如root）的权限来 fork 一个进程，加载程序并运行，因此 sudo 后面不能跟 shell 的内置命令，如：

{% highlight bash %}
$ sudo cd /sys/kernel/debugfs
sudo: cd: command not found
{% endhighlight %}
在这种情况，我们又没有 root 账户的密码，我们怎样执行该命令呢？有种办法就是使用 sudo 获得root shell 的权限，然后在root shell 中执行该命令。进入root shell 很简单，输入`sudo shell` 确认本用户的密码即可，此时你会发现命令提示符显示当前是 root。一旦获得root shell，你可以执行任何命令而不需要在每条命令前输入sudo了。

另外，常用的shell 内置命令在[这里](http://www.thegeekstuff.com/2010/08/bash-shell-builtin-commands/) 有简单介绍，我们可以使用 type 命令来查看命令的类型，如：
{% highlight bash %}
$ type ls
ls is /bin/ls
$ type umask
umask is a shell builtin
{% endhighlight %}

## 情景五：sudo 操作记录日志
作为一个 Linux 系统的管理员，不仅可以让指定的用户或用户组作为root用户或其它用户来运行某些命令，还能将指定的用户所输入的命令和参数作详细的记录。而sudo的日志功能就可以用户跟踪用户输入的命令，这不仅能增进系统的安全性，还能用来进行故障检修。但是要记录sudo的日志还要一些简单的配置：

* 创建sudo日志文件   
我们将sudo日志文件放置在 `/var/log/sudo.log` 文件中：   

		$ sudo touch /var/log/sudo.log

* 修改 `/etc/rsyslog.conf` 配置文件   
我使用系统为Ubuntu13.04 为改名字，但有些系统名为`/etc/syslog.conf`，注意不同发行版之间的差别，在该文件加入下面一行：   

		local2.debug	/var/log/sudo.log    #空白不能用空格，必须用tab
		
* 修改 `/etc/sudoers` 配置文件   
注意网上很多关于sudo日志文件配置都缺少这一步！在该文件中加入下面一行：

		Defaults	logfile=/var/log/sudo.
		
* 重启 syslog 服务：   

		$ sudo service rsyslog restart
		
* 查看 sudo 日志记录：    
经过上面的配置，sudo 的所有成功和不成功的sudo命令都记录到文件/var/log/sudo.log 中，例如我运行几条sudo 命令之后，查看该文件的记录如下：  
 
		$ cat sudo.log 
		Sep 20 22:10:51 : guohailin : TTY=pts/1 ; PWD=/var/log ; USER=root ;
	    	COMMAND=/bin/cat /etc/sudoers
		Sep 20 22:11:36 : guohailin : TTY=pts/1 ; PWD=/var/log ; USER=root ;
	    	COMMAND=/usr/sbin/service rsyslog restart
		Sep 20 22:11:45 : guohailin : TTY=pts/1 ; PWD=/var/log ; USER=root ;
	    	COMMAND=/bin/ls
		Sep 20 22:12:08 : guohailin : TTY=pts/1 ; PWD=/var/log ; USER=root ;
	    	COMMAND=/bin/ls /root/
		

## 参考资料：
* [sudo mannual](http://www.sudo.ws/sudoers.man.html)
* [7 Linux sudo Command Tips and Tricks](http://www.thegeekstuff.com/2010/09/sudo-command-examples/)
* [sudo 日志配置](http://firerat.blog.51cto.com/2371183/455524/)
