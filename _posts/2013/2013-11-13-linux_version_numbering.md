---
layout: post
title: "Linux 内核版本命名"
description: ""
category: linux 
tags: [linux, kernel]
---
{% include JB/setup %}


Linux 内核版本命名在不同的时期有其不同的规范，我们熟悉的也许是 2.x 版本奇数表示开发版、偶数表示稳定版，但到 2.6.x 以及 3.x 甚至将来的 4.x ，内核版本命名都不遵守这样的约定。本文就简单总结一下关于 Linux 内核版本号那点事：

## Linux 内核版本号命名四个不同的阶段

1. 从内核第一个0.01 版本发布到 1.0 版本。接下来是 0.02, 0.03, 0.10, 0.11, 0.12 (第一个 GPL 版本), 0.95, 0.96, 0.97, 0.98, 0.99，最后才到 1.0。

2. 1.0发布之后，直到2.6版本之前，命名格式为 “A.B.C”：
	* 数字 A 是内核版本号，版本号只有在代码和内核的概念有重大改变的时候才会改变，历史上有两次变化：
		* 第一次是1994年的 1.0 版
		* 第二次是1996年的 2.0 版
		* 2011年的 3.0 版发布，但这次在内核的概念上并没有发生大的变化
	* 数字 B 是内核主版本号，主版本号根据传统的奇-偶系统版本编号来分配：奇数为开发版，偶数为稳定版
	* 数字 C 是内核次版本号，次版本号是无论在内核增加安全补丁、修复bug、实现新的特性或者驱动时都会改变
	
3. 2004年 2.6 版本发布之后，内核开发者觉得基于更短的时间为发布周期更有益，所以大约七年的时间里，内核版本号的前两个数一直保持是“2.6”，第三个数随着发布次数增加，发布周期大约是两三个月。考虑到对某个版本的bug和安全漏洞的修复，有时也会出现第四个数字。

4. 2011年5月29号，Linus 宣布为了纪念Linux发布 20周年，在 2.6.39 版本发布之后，内核版本将升到 3.0 。Linux 继续使用在 2.6.0 版本引入的基于时间的发布规律，但是使用第二个数——例如在3.0发布的几个月之后发布3.1，同时当需要修复bug和安全漏洞的时候，增加一个数字（现在是第三个数）来表示，如 3.0.18。

**其它补充**：

* 内核版本命名第一次使用第四个数字是在 2.6.8 的 NFS 代码中出现一个严重的错误需要立即修复，然而还没有足够多的其它改变可以发布一个新的版本（也就是2.6.9），所以，2.6.8.1 发布了，仅仅修正了那个错误。直到 2.6.11，这种版本命名策略被官方正式采纳。接着，这种通过改变第四个数字来显示修复主要bug和安全补丁而发布新内核的做法，成为一种普遍的做法。

* 在正式发布之前，一般都冠以“待发布”（release candidates)字样，通过在内核版本的普通数字之后添加后缀 “rc”。

* 有些时候，版本号后面有类似于 “tip”这样的后缀，表明另一个开发分支，这些分支通常（但不总是）是一个人开始发起的。举例来说，“ck” 代表 Con Kolivas，“ac” 代表 Alan Cox 等等。有时，字母和内核建立分支的主要开发领域相关，例如“wl” 表示该分支主要测试无线网络的。同时，不同的发行版也会根据需要有自己的后缀。


## 4.0 版本什么时候发布？
2013年11月3日，Linus Torvalds宣布发布Linux 3.12，同时还讨论了Linux 4.0发布计划：他考虑在Linux 3.19 之后发布Linux 4.0，和Linux 3.0发布策略相同，4.0并不代表着巨大变化，他只是想避免3.x 的版本号超过20，因为小版本号记忆起来比较简单。

下面是他在内核开发邮件中的原文：

> we're getting to release numbers where I have to take off my socks to count that high again. I'm ok with3.<low teens>, but I don't want us to get to the kinds of crazy numbers we had in the 2.x series, so at some point we're going to cut over from 3.x to 4.x, just to keep the numbers small and easy to remember. We're not there yet, but I would actually prefer to not go into the twenties, so I can see it happening in a year or so, and we'll have 4.0 follow 3.19 or something like that.

按照 Linus 的发布 4.0 的预期以及现在每一个多月就更新一个版本的频率，大概在一年之内内核版本就可以变成 4.x。

## 内核版本分类

在 Linux 内核官网上你会看到主要有三种类型的内核版本，下图为我在2013.11.13 在官网的截图：

![linux_kernel](https://f.cloud.github.com/assets/3265880/1528044/a922f638-4c01-11e3-882f-3f57a5b20718.png)


1. **mainline** 是主线版本，目前主线版本为 3.12。
2. **stable** 是稳定版，由 mainline 在时机成熟时发布，稳定版也会在相应版本号的主线上提供 bug 修复和安全补丁，但内核社区人力有限，因此较老版本会停止维护，而标记为 **EOL** (End of Life)的版本表示不再支持的版本。
3. **longterm** 是长期支持版，目前还处在长期支持版的有五个版本的内核，分别为 3.10、3.4、3.2、2.6.34、2.6.32，长期支持版的内核等到不再支持时，也会标记**EOL**。

## 查看机器使用的内核版本号
我们安装了不同的 Linux 发行版，那么如何去查看该发行版使用的内核版本号呢？ 我们可以使用命令 `uname -r` 来查看：

{% highlight bash %}
[root@archlab-server2 ~]# cat /etc/issue
CentOS release 6.4 (Final)
[root@archlab-server2 ~]# uname -r
2.6.32-358.6.1.el6.i686
{% endhighlight %}

我测试的机器使用的是 CentOS 6.4 的发行版，显示的内核版本为 2.6.32。


## 参考资料
* [http://en.wikipedia.org/wiki/Linux_kernel#Version_numbering](http://en.wikipedia.org/wiki/Linux_kernel#Version_numbering)
* [https://lkml.org/lkml/2013/11/3/160](https://lkml.org/lkml/2013/11/3/160)
* [http://lxr.linux.no/](http://lxr.linux.no/)
* [https://kernel.org/](https://kernel.org/)
