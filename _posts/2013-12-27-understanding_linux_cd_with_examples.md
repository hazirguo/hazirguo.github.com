---
layout: post
title: "例解 Linux cd 命令"
description: ""
category: linux 
tags: [linux, command, cd]
---
{% include JB/setup %}

**cd** 命令是 *nix 系统中最基本的命令，它所做的事情是改变你当前所在的目录。本文详细介绍该命令，它所能完成的功能以及关于该命令内在的东西。

## cd 命令：一个内置命令
BASH Shell 是大多 Linux 发行版的默认 shell，BASH 有一些自己的内置命令，cd 就是其中的一个。我将解释什么是内置命令，以及为什么 cd 是一个内置命令。首先，用 `SHELL` 环境变量确认你当前的 shell：

![](http://linoxide.com/wp-content/uploads/2013/12/01.cd_shell.png)

现在用 `which` 命令检查 cd 命令二进制文件所在的路径（如果存在的话）：

![](http://linoxide.com/wp-content/uploads/2013/12/02.cd_which.png)

结果什么都没有输出，这是因为系统中不存在 cd 命令的二进制文件。但是你仍然可以运行该命令，这是因为 cd 是 BASH 的内置命令。内置命令就是内建在 shell 里的命令，另一个内置命令 `type` 会给你显示 cd 命令是一个内置命令的信息：

![](http://linoxide.com/wp-content/uploads/2013/12/03.cd_type.png)

如果你尝试获得任何内置命令的帮助文档，将不存在它们独立的帮助页：

![](http://linoxide.com/wp-content/uploads/2013/12/04.cd_man.png)

对于这些内置命令，不会创建独立的进程来运行它们，因此他们运行效率较高。

为了得到所有的内置命令，你可以使用 `help` 命令（这里 help 本身也是一个内置命令）：

![](http://linoxide.com/wp-content/uploads/2013/12/05.cd_help_1.png)

## 为什么 cd 是内置命令
为了描述简单，我就不讨论更多的细节了，但是要理解这个问题的答案，还需要知道一点 Unix 进程相关的知识。

BASH 创建的任何进程，它会由一个 BASH 的子 shell（当前 BASH 进程的子进程）来执行该进程，新建的进程运行实例、输出（如果需要的话），当该进程结束时，改子 shell 的任何属性都不会返回给父 shell。注意到的是，cd 命令用来改变 shell 当前所在的路径，如果 cd 是一个外部命令，它将改变子 shell 的当前路径，当运行完返回时，他所做的改变对父 shell 没有关系。因此，shell 的当前路径还是没有改变！所有改变当前 shell 环境的命令，在实现上都必须实现成内置命令。如果实现成外部命令，我们将不会得到预期的结果。

下面我们探索 cd 命令的用法：

## cd 命令用法
如果你直接输入 cd 命令而不带任何参数，它将切换到你的 home 目录下，不管你当前所在的目录是什么：

![](http://linoxide.com/wp-content/uploads/2013/12/07.cd_home.png)

波浪线（~）符号也代表 home 目录，你也可以使用它来切换到 home 目录下：

![](http://linoxide.com/wp-content/uploads/2013/12/08.cd_home_tilde.png)

如果你是 root 用户，你可以切换到任何用户的 home 目录，使用波浪线后跟用户名。在一些 Linux 发行版中，没有特权的用户默认没有权限切换到其它用户的 home 目录：

![](http://linoxide.com/wp-content/uploads/2013/12/09.cd_home_user.png)

点（.）代表当前目录，两个点（..）代表父目录，要想切换到父目录，只需要使用..:

![](http://linoxide.com/wp-content/uploads/2013/12/10.cd_parent.png)

只使用 . 大多情况下将不会将会你当前的目录，例如：

![](http://linoxide.com/wp-content/uploads/2013/12/11.cd_dot.png)

但是如果你当前目录重命名为其它名字，那么使用 . 将会改变当前目录：

![](http://linoxide.com/wp-content/uploads/2013/12/12.cd_dot_renamed.png)

在 BASH 以及大多数其它 shell 中，你可以提供两种类型的路径表示方式：绝对路径和相对路径。绝对路径使用 / 开始，和你当前所在目录无关；另一个相对路径不是以 / 开始，依赖于你当前所在的目录。

使用绝对路径改变当前目录：

![](http://linoxide.com/wp-content/uploads/2013/12/13.cd_abs_path.png)

使用相对路径改变当前目录：

![](http://linoxide.com/wp-content/uploads/2013/12/14cd_rel_path.png)

可以使用 `cd -` 命令，回到上一次工作的目录，实现在两个目录间来回切换：

![](http://linoxide.com/wp-content/uploads/2013/12/15.cd_toggle.png)

上次工作的目录保存在变量 `OLDPWD` 中，如果你试着在新的终端下使用该命令，它会显示下面的错误：

![](http://linoxide.com/wp-content/uploads/2013/12/16.cd_OLDPWD_not_set.png)

你还可以在 cd 命令中使用一些 bash 的技巧，例如使用通配符“？”、“*” 等。

![](http://linoxide.com/wp-content/uploads/2013/12/17.cd_question_mark_wild_card.png)
![](http://linoxide.com/wp-content/uploads/2013/12/18.cd_star_wild_card.png)


**编译自**：[http://linoxide.com/linux-command/linux-cd-command-examples/](http://linoxide.com/linux-command/linux-cd-command-examples/)
