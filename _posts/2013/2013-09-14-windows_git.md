---
layout: post
title: "如何在windows下使用git"
description: ""
category: git 
tags: [git]
---
{% include JB/setup %}

在 Windows 下使用Git基本上有两种方法：通过安装 Cygwin 或 msysGit 来使用Git。在两种不同的方案下，Git 的使用与 Linux 环境下的使用基本相同，另外还可以通过安装 msysGit 的图形界面软件 TortoiseGit (用过 SVN 的已经非常熟悉 TortoiseSVN，这里 TortoiseGit 和它有着非常相似的操作界面和方法）来使用 Git。这里我只介绍一下如何安装和使用 msysGit，让大家对 Git 的使用有个基础的了解。
## 在windows下安装 msysGit
msysGit 名字前面四个字母来源与 MSYS 项目。MSYS项目源自于 MinGW（Minimaliest GNU for Windows，最简GNU工具集），通过增加一个 bash 提供的 shell 环境和其他相关的工具软件组成一个最简系统（Minimal SYStem），简称MSYS。利用 MinGW 提供的工具和 Git 针对 MinGW 的一个分支版本，在 Windows 平台为 Git 编译出一个原生应用，结合 MSYS 就组成了 msysGit。
安装 Git 很简单，去 msysGit 的官网 http://msysgit.github.io 选择下载名为 `Git-<VERSION>-preview<DATE>.exe` 的版本，这里我们选择下载 Git-1.8.3-preview20130601.exe，选择安装路径，其他选项默认即可完成安装。
安装完成之后，鼠标右键会有 `Git Bash` 和 `Git GUI` 两种接口，为了能与 Linux 下保持一致且熟悉 Git 基本命令，推荐大家选择 Git Bash，和 Linux 下命令完全一致。
## msysGit 中 shell 环境的中文支持
msysGit 中 Shell 环境的中文支持很不好，需要进行一些配置。
（待完善...）

## 如何操作资源库
我们实验室已经建立了一个 Git 服务器（ip地址为1.0.0.89，Git 服务器专用帐号为 git，服务器上有个测试版本库 test.git），大家可以将需要共享的文件和实验室项目托管在该服务器上，这样大家不仅共享方便，而且数据安全性更可靠。Git 支持多种协议，如 HTTP、SSH、Git-daemon、Gitosis 等等，在这里我们使用 SSH 协议来进行对我们的资源库进行读写操作。

### SSH协议公钥认证
所有用户都使用同一个专用的 SSH 帐号访问版本库，访问时通过公钥认证的方式。我们已经安装了 msysGit，启动 Git Bash，输入命令 `ssh-keygen` 命令即可产生公私钥，如图1：
![ssh-keygen](http://i.imgur.com/4wiYYzM.png)
图1 产生SSH协议公私钥

在用户主目录下（`c:\Users\***\.ssh`）生成公钥/私钥对，名字分别为 id_rsa.pub 和 id_rsa。
将你们的**公钥**发送给管理员，管理员将公钥添加到你能访问到资源库中，这样你就可以无口令登录到远程服务器，即用公钥认证代替口令认证。

### 克隆版本库
下面均使用我们的测试版本库 test.git 来做实验。
首先我们需要将服务器上的版本库克隆到本地机器，例如我们要将此版本克隆到 E 盘下，我们进入 E 盘，鼠标右键启动 Git Bash，输入命令：
{% highlight bash %}
$git clone git@1.0.0.89:test.git 
{% endhighlight %}
即将远程服务器上 test.git 库拷贝到本地，进入该目录，可以看到现在版本库中已有的文件。

### 同步远程版本库
我们在本地向版本库添加文件如何更新到服务器上呢？同样如何从服务器上获得最新更新呢？

1.简单配置   
开始之前，这里只介绍一下 Git 的简单配置，例如下面命令：
{% highlight bash %}
  $git config --global user.name "guohailin"
  $git config --global user.email hazirguo@gmail.com
{% endhighlight %}
可以用来配置提交时使用的姓名和邮箱，当然配置一次就够了，以后提交如果不修改则不用再配置；更复杂的配置参见 [git-config(1) Manual Page](https://www.kernel.org/pub/software/scm/git/docs/git-config.html) 。

2.提交到本地库   
我们需要将新增文件提交到本地的版本库中，如下命令新增一个文件greeting，内容为“hello”，提交到本地的版本库：
{% highlight bash %}
  $touch greeting
  $echo "hello" >> greeting
  $git add greeting        #添加greeting文件到暂存区
  $git commit -m "add file greeting by guohl"		#提交到本地版本库，-m 后加上注释
{% endhighlight %}

3.更新到远程服务器   
提交到本地服务器之后，还需要提交到远程服务器，很简单，只需要使用 `git push` 命令即可。

4.从远程服务器上获得最新版本   
每个人都有可能向远程服务器提交，我们可以通过在本地使用命令 `git pull` 即可获得远程服务器的别人的最新提交。

## 结语
总结一下，我们开始使用 git 时，只需掌握基本的简单操作即可，至于更复杂的操作等今后再慢慢探讨。
开始可以使用服务器上的一个测试库 test.git 来练习练习，我们还创建一个公共库 eslab2013.git，大家可以克隆下来，如果有什么需要共享的软件、资源就可以上传到这上面，该版本库的维护靠大家。
如果谁有特殊需求需要建立其它库来备份资料，也可以联系我。
出现任何问题都可以在下方评论中提出，大家一起解决！


----
参考资料：

* Git 权威指南    蒋鑫 著      机械工业出版社
* Pro Git     http://git-scm.com/book



