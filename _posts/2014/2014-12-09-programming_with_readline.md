---
layout: post
title: "GNU Readline 库及编程简介"
description: ""
category: c_cplusplus 
tags: [c, gnu, readline]
---
{% include JB/setup %}

用过 Bash 命令行的一定知道，Bash 有几个特性：

* `TAB` 键可以用来命令补全
* `↑` 或 `↓` 键可以用来快速输入历史命令
* 还有一些交互式行编辑快捷键：
    * `C-A` / `C-E` 将光标移到行首/行尾
    * `C-B` / `C-F` 将光标向左/向右移动一个位置 
    * `C-D` 删除光标下的一个字符
    * `C-K` 删除光标及光标到行尾的所有字符
    * `C-U` 删除光标到行首的所有字符
    * ...
    
同样的操作在很多交互式程序都有类似的操作，例如 ftp、gdb 等等，那么你是否想过这些是如何实现的呢？如果我们要做一个命令行下的交互式开源软件，是否希望也能有这些命令补全、搜索历史命令、行编辑快捷键等等这些人性化的交互方式呢？

要想实现这些，你有两种途径：可以自己写程序实现，或者调用开源的库 Readline Lib。例如上面介绍的 bash、ftp、gdb 等等软件都使用了 GNU 的开源跨平台库，为其提供交互式的文本编辑功能。当然需要注意的是，Readline Library 是 GNU 自由软件，在 GNU GPL V3 协议下发布，因此如果你的程序中需要用到该库，也必须遵守相关协议。

本文首先简单介绍一下该库的基本使用方法，后面会稍微详细介绍下如何使用 Readline 来自定义命令补全功能。

## Readline 基本操作

很多命令行交互式程序交互方式都差不多，输出提示符，等待用户输入命令，用户输入命令之后按回车，程序开始解析命令并执行。那么这里面有个动作是读入用户的输入，以前我们也许使用 `gets()` 这样的函数来实现，当我们使用 Readline 库时，可以使用 `readline()` 函数来替换它，该函数在 ANSI C 中定义如下：

{% highlight c %}
char *readline (char *prompt);
{% endhighlight %}

该函数带有一个参数 *prompt*，表示命令提示符，例如 ftp 中就是 "**ftp>**"，用户在后面可以输入命令，当按下回车键时，程序读入该行（不包括最后的换行符）存入字符缓冲区中，`readline` 的返回值就是该行文本的指针。**注意：**当该行文本不需要使用时，需要释放该指针指向的空间，防止内存泄漏。当读入 `EOF` 时，如果还未读入其它字符，则返回 `(char *) NULL`，否则读入结束，与读入换行效果相同。

除了能读入用户的输入，我们有时希望交互更简单些，例如命令补全。当有很多命令时，如果希望用户都能准确记忆命令的拼写是困难的，那么一般做法是按下 `TAB` 键进行命令提示及补全，如 ftp 下输入一个字符 `c` 之后按下 `TAB` 键，会列出所有以 `c` 开头的命令：

{% highlight bash %}
ftp> c
case	cd	cdup	chmod	close	cr
{% endhighlight %}

`readline` 函数其实已经给用户默认的 `TAB` 补全的功能：**根据当前路径下文件名来补全**。

如果你不想 Readline 根据文件名补全，你可以通过 `rl_bind_key()` 函数来改变 `TAB` 键的行为。该函数的原型为：
 
{% highlight c %}
int rl_bind_key(int key, int (*function)());
{% endhighlight %}
 
该函数带有两个参数：*key* 是你想绑定键的 ASCII 码字符表示，*function* 是当 *key* 键按下时触发调用函数的地址。如果想按下 `TAB` 键就输入一个制表符本身，可以将 `TAB` 绑定到 `rl_insert()` 函数，这是 Readline 库提供的函数。如果 *key* 不是有效的 ASCII 码值（0~255之间），`rl_bind_key()` 返回非 0。

这样，禁止 `TAB` 的默认行为，下面这样做就可以了：

{% highlight c %} 
rl_bind_key('\t', rl_insert);
{% endhighlight %}

这个代码需要在你程序一开始就调用；你可以写一个函数叫 `initialize_readline()` 来执行这个动作和其它一些必要的初始化，例如安装用户自定义补全。

当我们希望输入 `TAB` 时不是列出当前路径下的所有文件，而是列出程序内置的一些命令，例如上面举到 ftp 的例子，这种行为称为**自定义补全**。 该操作较复杂，我们留在后面一节主要介绍。

基本操作还有一个——**搜索历史**。我们希望输入过的命令行，还可以通过 `C-p` 或者 `C-s` 来搜索到，那么就需要将命令行加入到历史列表中，可以调用 `add_history()` 函数来完成。但尽量将空行也加入到历史列表中，因为空行占用历史列表的空间而且也毫无用处。综上，我们可以写出一个 Readline 版的 `gets()` 函数 `rl_gets()`：

{% highlight c %}
/* A static variable for holding the line. */
static char *line_read = (char *)NULL;

/* Read a string, and return a pointer to it.  Returns NULL on EOF. */
char *
rl_gets ()
{
  /* If the buffer has already been allocated, return the memory
     to the free pool. */
  if (line_read)
    {
      free (line_read);
      line_read = (char *)NULL;
    }

  /* Get a line from the user. */
  line_read = readline ("");

  /* If the line has any text in it, save it on the history. */
  if (line_read && *line_read)
    add_history (line_read);

  return (line_read);
}
{% endhighlight %}

## 自定义补全

上面也提到了什么是自定义补全，无疑这在命令行交互式程序中是非常重要的，直接影响到用户体验。Readline 库提供了两种比较常用的补全方式——按照文件名补全和按照用户名补全，分别对应 Readline 中已经实现的两个函数 `rl_filename_completion_function` 和 `rl_username_completion_function`。如果我们既不希望按照文件名和用户名来补全，希望按照程序的命令补全，应该怎么做呢？也很容易想到，只要实现自己的补全函数就好了。

Readline 补全的工作原理如下：

* 用户接口函数 `rl_complete()` 调用 `rl_completion_matches()` 来产生可能的补全列表；
* 内部函数 `rl_completion_matches()` 使用程序提供的 *generator*  函数来产生补全列表，并返回这些匹配的数组，在此之前需要将 *generator* 函数的地址放到 `rl_completion_entry_function` 变量中，例如上面提到的按文件名或用户名补全函数就是不同的 *generators*；
* *generator* 函数在 `rl_completion_matches()` 中不断被调用，每次返回一个字符串。*generator* 函数带有两个参数：*text* 是需要补全的单词的部分，*state* 在函数第一次调用时为 0，接下来调用时非 0。*generator* 函数返回 `(char *)NULL` 通知 `rl_completion_matches()` 没有剩下可能的匹配。 

Readline 库中有个变量 `rl_attempted_completion_function`，改变量类型是一个函数指针 `rl_completion_func_t *`，我们可以将该变量设置我们自定义的产生匹配的函数，该按下 `TAB` 键时会调用该函数，函数具有三个参数：

* *text*: 该参数是待补全的单词的部分，例如在 Bash 提示符后输入一个 `c` 字符，按下 `TAB`，此时 *text* 指向的是 "c" 字符串的指针；在 Bash 提示符后输入一个 `cd /home/gu` 字符串，按下 `TAB`，此时 *text* 指向的是 "/home/gu" 字符串的指针；
* *start*: *text* 字符串在该行输入中的起始位置，例如对于上面的例子，第一种情况下是 0，第二种情况下是 3；
* *end*: *text* 字符串在该行输入中的结束位置，例如对于上面的例子，第一种情况下是 1，第二种情况下是 11。

我们自定义的补全函数可以根据传入的参数来设置我们希望按照什么方式补全，例如对于 Bash 下的 `cd` 命令，我们希望开始是命令补全，当命令补全之后，后面接着跟的是文件名补全，这样可以使用 `rl_completion_matches()` 来绑定使用哪种 *generator*，`rl_completion_matches()` 函数的原型是：

{% highlight c %}
char ** rl_completion_matches (const char *text, rl_compentry_func_t *entry_func)
{% endhighlight %}

带有两个参数：*text* 就是上面介绍的传入的待补全的单词，第二个参数 *entry_func* 是上面反复介绍的 *generator* 函数的指针。该函数的返回值是 *generator* 产生的可能匹配 *text* 的字符串数组指针，该数组的最后一项是 `NULL` 指针。

好了，上面说了这么多关于自定义补全的函数和变量，到底怎么用呢，估计还是比较模糊，那么看一个例子估计就很清楚了，这个例子是 Readline 官方提供的示例程序，由于比较长，就不在这里贴出来了，你可以在 [http://cnswww.cns.cwru.edu/php/chet/readline/readline.html#SEC49](http://cnswww.cns.cwru.edu/php/chet/readline/readline.html#SEC49) 找到。

## 总结

其实，虽然说了很多，但还只是 Readline 库的皮毛，这个库的功能远远比这强大的多，如果想深入了解并且运用，你必须要做三件事：

* **Read The Fucking Manual**：阅读官方的 [文档](http://cnswww.cns.cwru.edu/php/chet/readline/readline.html)
* **Read The Fucking Source Code**：阅读官方提供的例子代码，如果想了解更深入可以去看 Readline 的源码
* **Show Your Code**：自己动手写几个例子试试，如果有机会运用到你的项目中。
