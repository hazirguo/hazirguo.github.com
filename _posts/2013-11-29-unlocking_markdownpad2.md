---
layout: post
title: "破解 Windows 平台下Markdown 编辑器 MarkdownPad2"
description: ""
category: miscellaneous
tags: [markdown]
---
{% include JB/setup %}

MarkdownPad 是 Windows 平台下一款优秀的 Markdown 编辑器，本文简单介绍 Markdown 以及使用一种方法破解 MarkdownPad 使其升级到专业版。该方法仅限于教育用途，我不为滥用该方法导致的商业纠纷负责！

## Markdown 简介
Markdown 是一种轻量级标记语言，有着及其简单、易学易用的语法，任何人在几分钟之内都可以掌握其基本语法并用来写作，你可以在 [**这里**](http://daringfireball.net/projects/markdown/syntax) 学习markdown的基本语法。其创始人 John Gruber 下面的话高度概括了 Markdown 是什么、优点以及适合哪些人使用：

>Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

相比 Word 等富文本编辑器，你会花很大精力去排版上，还有它所写的内容移植性、兼容性也很差，Markdown 这种纯文本的标记语言，就不存在这种问题，当然还有很多其它好处，你可以阅读下面几篇文章：

* [3 Reasons Why Everyone Needs to Learn Markdown](http://readwrite.com/2012/04/17/why-you-need-to-learn-markdown#awesm=~ooxRM0YgMdu35E)
* [Markdown写作浅谈](http://jianshu.io/p/PpDNMG)

而且当下最大的社交编程平台 [Github](https://github.com/) 以及问答网站 [Stackoverflow](http://stackoverflow.com/) 在其有编辑框的地方，都支持的是 Markdown，这也极大地推动了 Markdown 的普及。

随着 Markdown 在开发者中间逐渐流行，出现了很多 Markdown 编辑器，如 Mac 下的 [Mou](http://mouapp.com/)， Linux 下的 [ReTex](http://sourceforge.net/p/retext/home/ReText/)，Windows 下的 [MarkdownPad](http://markdownpad.com/) 等等。当然如果你不想下载这些编辑器，也可以用在线版的，如 [Dillinger](http://dillinger.io/) 等。你可以猛戳 [**这篇文章**](http://mashable.com/2013/06/24/markdown-tools/) ，介绍了 78 款 Markdown 编辑预览的工具。


## 破解 MarkdowPad 2
我目前已经习惯了写博客、做笔记等都使用 Markdown，在 Linux 使用的是 [Sublime Text](http://www.sublimetext.com/)，在 Windows 下使用的是 [MarkdownPad](http://markdownpad.com/)。MarkdowPad 是 Windows 下一款优秀的 Markdown 编辑器，有免费版和专业版，专业版提供了很多高级功能，如自动保存、PDF导出、GFM 语法、自定义CSS、语法高亮等等。当免费版进行一些高级设置时，如设置Markdown处理引擎为 GFM，会提示让你升级到专业版，图1所示。可以说专业版这些功能是非常吸引我的，但专业版需要付 $14.95！本能地去在网上找破解版的，如果百度甚至Google出的所有破解 MarkdownPad 的都让你替换一个名为`user.config` 文件，其实都是用一个已注册邮箱和密钥进行验证的。但是遗憾的是，目前这个帐号已经不能使用了，这里介绍的一种方法，可以真正意义上破解该软件。

![](http://i.imgur.com/MLmcISF.png)

***图1 MarkdownPad 高级设置需要升级到专业版***

### 准备
你首先需要安装以下软件：

* MarkdownPad 2, 你可以去 [官网](https://markdownpad.com/download.html) 下载最新版的，然后安装好。
* .NET 反编译器, 这里选用[ILSpy](http://ilspy.net/), 需要.NET Framework 4.0支持, 将可执行文件反编译成 C# 源代码。
* 反汇编工具, 无疑选用 [IDA](https://hex-rays.com/products/ida/index.shtml), 能将可执行文件文件反汇编成汇编文件。
* 十六进制编辑器, 有很多种，这里选用 [HxD](http://mh-nexus.de/en/hxd/), 可以以十六进制查看二进制文件，并且编辑。

### 破解步骤

1.**使用 ILSpy 反编译 MarkdownPad 出源文件，找到其验证授权的函数。**   
使用 ILSpy 打开 MarkdownPad2 安装目录下的 MarkdownPad2.exe 文件，在 MarkdowPad2.Licensing 命名空间下找到 LicenseEngine 类的 VerifyLicense 方法，如图2所示。

![](http://i.imgur.com/z0mhKkX.png)

***图2 使用ILSpy 反编译找到验证函数***

这个函数是用来验证你所填写的邮箱 email 和 许可证 licenseKey 是否合法，函数首先判断 email 和 licenseKey 是否为空，若有一个为空则直接返回 false，即验证不通过； 若均不为空，那么下面进行其它逻辑的验证。我们并不关心它是如何验证用户所填的 email 和 licenseKey 是否能匹配上，我们只需要将第一步判断若 email 或 licenseKey 为空返回 false 改为 返回 true，那么，就直接通过验证了。下面就是要使用 IDA 工具找到该代码片段的二进制代码的位置。

2.**使用 IDA 反汇编 Markdown，找到验证授权函数的汇编代码。**
使用 IDA 打开 MarkdownPad2 安装目录下的 MarkdownPad2.exe 文件，左侧点击 Function name，按 `ALT+T` 键搜索 **VerifyLicense** 函数名，能看到汇编代码逻辑图，如图3。

![](http://i.imgur.com/a17IM57.png)

***图3 通过IDA 反汇编找到验证函数的汇编代码***

上图中黄色标记的汇编代码 `ldc.i4.0` 的意思是将 0 作为32位整型数压到栈上，根据上面的分析，我们应该要把这条指令改成 `ldc.i4.1`，让其返回 true，那么我们还需要找到这条汇编代码对应的二进制代码。在 IDA 中就可以以十六进制视图查看，如图4。

![](http://i.imgur.com/ryjoUzD.png)

***图4 在 IDA 中以十六进制视图查看验证函数代码***

3.**使用 HxD 修改验证授权函数的二进制代码，使其通过验证。**
可以看到 `ldc.i4.0` 对应的二进制代码为 `0x16`，我们只需将其改成 `0x17`，这需要借助 HxD 软件来对可执行文件进行编辑。使用 HxD 打开 MarkdownPad2 安装目录下的 MarkdownPad2.exe 文件，根据图5搜索 `2C 02 16 2A 02 02 03`，找到 `16` 的位置，然后将其改成 `17` 即可。

![](http://i.imgur.com/iZDxHSh.png)

***图5 通过搜索找到`idc.i4.0`二进制代码位置***

## 再次声明
文章中方法来源于[该博客](http://iamjuza.blogspot.com/2013/09/unlocking-markdownpad-2.html)，只限于学习用途，如需使用还[**请支持正版**](https://markdownpad.com/buy.html)!
 
 
