---
layout: post
title: "Linux Kernel代码艺术——系统调用宏定义"
description: ""
category: Linux 
tags: [kernel, syscall, marco]
---
{% include JB/setup %}

我们习惯在SI（Source Insight)中阅读Linux内核，SI会建立符号表数据库，能非常方便地跳转到变量、宏、函数等的定义处。但在处理系统调用的函数时，却会遇到一些麻烦：我们知道系统调用函数名的特点是sys_×××，例如我们想找open函数的内核系统调用代码，在SI提供的符号表中搜索sys_open，能找到函数的声明：
{% highlight c %}
asmlinkage long sys_open(const char __user *filename, int flags, umode_t mode);
{% endhighlight %}
原本SI提供从函数名按住Ctrl单击鼠标左键能跳转到定义处的功能，但运用在系统调用函数sys_open上却失败了，这是什么回事呢？


## 系统调用宏定义展开
经过分析，原来内核中系统调用采用了宏定义，如这里的sys_open就被定义为：
{% highlight c %}
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode)
{% endhighlight %}

可以猜测出这个宏定义展开之后就是上面函数声明那样的，难怪SI不能跳转到系统调用的定义处呢！

下面以open系统调用为例分析这个宏是如何展开的：

首先在 `include/linux/syscall.h` 中有下面这样的宏定义：
{% highlight c %}
#define SYSCALL_DEFINE3(name, ...) 				\
	SYSCALL_DEFINEx(3, _##name, __VA_ARGS__)
{% endhighlight %}

针对这个宏定义有几点说明：

1. 反斜杠\\：当宏定义过长需要换行时，在行尾要加上换行标志“\”；
2. …：省略号代表可变的部分，下面用`__VA_AEGS__` 代表省略的变长部分；
3. \#\#：分隔连接方式，它的作用是先分隔，然后进行强制连接，例如：

{% highlight c %}
#define VAR(type, name) type name##_##type
VAR(int, var1);
展开之后就是：
int var1_int;
{% endhighlight %}

那么：
{% highlight c %}
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode)
展开之后是：
SYSCALL_DEFINEx(3, _open, __VA_ARGS__)
这又是一个宏，根据宏定义：
#define SYSCALL_DEFINEx(x, sname, ...)				\
	__SYSCALL_DEFINEx(x, sname, __VA_ARGS__)

SYSCALL_DEFINEx(3, _open, __VA_ARGS__)
展开为：
__SYSCALL_DEFINEx(3, _open, __VA_ARGS__)

再根据宏定义：
#define __SYSCALL_DEFINEx(x, name, ...)					\
	asmlinkage long sys##name(__SC_DECL##x(__VA_ARGS__))

__SYSCALL_DEFINEx(3, _open, __VA_ARGS__)
展开为：
asmlinkage long sys_name(__SC_DECL3(__VA_ARGS__))
{% endhighlight %}

这里 `__VA_ARGS__` 是 `const char __user *, filename, int, flags, umode_t, mode`，而同样`__SC_DECL3` 又是一组宏定义：

{% highlight c %}
#define __SC_DECL1(t1, a1)	t1 a1
#define __SC_DECL2(t2, a2, ...) t2 a2, __SC_DECL1(__VA_ARGS__)
#define __SC_DECL3(t3, a3, ...) t3 a3, __SC_DECL2(__VA_ARGS__)
{% endhighlight %}

这样，一步一步地展开：
{% highlight c %}
__SC_DECL3(const char __user *, filename, int, flags, umode_t, mode)
==>   __SC_DECL3(const char __user *, filename, int, flags, umode_t, mode)
==>   const char __user* filename, __SC_DECL2( int, flags, umode_t, mode)
==>   const char __user* filename, int flags, __SC_DECL1(umode_t, mode)
==>   const char __user* filename, int flags, umode_t mode
{% endhighlight %}

最终：
{% highlight c %}
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode)
宏定义展开之后就成为：
asmlinkage long sys_open(const char __user *filename, int flags, umode_t mode);
{% endhighlight %}

正如我们之前猜测的那样。

## 如何在 SI 中找到系统调用源代码
回到开始的话题，既然不能直接通过系统调用声明跳转到定义的代码处，那么怎样在 SI 快速找到系统调用的源码呢？通过上面的sys_open 的展开，相信大家已经知道带有三个参数的系统调用展开的过程，由于系统调用中最多可以带有六个参数，那么Linux 内核中定义了一组宏用来展开带有不同参数的系统调用：

{% highlight c %}
#define SYSCALL_DEFINE1(name, ...) SYSCALL_DEFINEx(1, _##name, __VA_ARGS__)
#define SYSCALL_DEFINE2(name, ...) SYSCALL_DEFINEx(2, _##name, __VA_ARGS__)
#define SYSCALL_DEFINE3(name, ...) SYSCALL_DEFINEx(3, _##name, __VA_ARGS__)
#define SYSCALL_DEFINE4(name, ...) SYSCALL_DEFINEx(4, _##name, __VA_ARGS__)
#define SYSCALL_DEFINE5(name, ...) SYSCALL_DEFINEx(5, _##name, __VA_ARGS__)
#define SYSCALL_DEFINE6(name, ...) SYSCALL_DEFINEx(6, _##name, __VA_ARGS__)
{% endhighlight %}

那么有了这些宏之后，系统调用定义处的部分就可以用宏来代替了，如：

* fork系统调用，就可以定义为 `SYSCALL_DEFINE0(fork)`
* brk系统调用，就可以定义为 `SYSCALL_DEFINE1(brk, unsigned long)`
* creat 系统调用，就可以定义为 `SYSCALL_DEFINE2(creat, const char __user *, pathname, umode_t, mode)`
* ...

可以找出规律，SYSCALL_DEFINE 后面跟系统调用所带的参数个数n，第一个参数为系统调用的名字，然后接2*n个参数，每一对指明系统调用的参数类型及名字。那么下次我们想在 SI 中找某个系统调用的代码时，使用 SI 提供的全局搜索功能（快捷键 `Ctrl-/`），以open系统调用为例，输入 `SYSCALL_DEFINE3(open` ，如图1所示。有了这个方法以后就再不用担心找不到系统调用的内核代码了。

![capture](https://f.cloud.github.com/assets/3265880/1172878/2ab65f50-212d-11e3-8f2f-a6d839ab020d.PNG)

*图1 SI中搜索sys_open函数代码*

到此，我们学习到了宏定义的一些高级用法（如…、\#\#等），还知道了如何在SI中通过搜索找系统调用代码，学习过程中还会不时感慨开发 Linux 内核这些大牛们怎能将宏运用得如此出神入化。如果只知道这些肯定还是不够的，我们试想一下为什么要用宏定义把系统调用搞得这么复杂？直接用展开的形式不好么？可以肯定的是内核开发者不是单纯地秀代码技巧，至于这样写带来的好处是什么？

## 漏洞 CVE-2009-0029 解析
如果我们查看2.6.28 之前的代码，系统调用确实没有这样写，但在2009年64位 Linux 内核在某些64位平台下被发现系统调用有漏洞，为了修复该漏洞系统调用才改写成现在这样的。该漏洞被命名为 [CVE-2009-0029](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2009-0029&cid=4 'CVE-2009-0029') ，对该漏洞的简单描述如下：

>The ABI in the Linux kernel 2.6.28 and earlier on s390, powerpc, sparc64, and mips 64-bit platforms requires that a 32-bit argument in a 64-bit register was properly sign extended when sent from a user-mode application, but cannot verify this, which allows local users to cause a denial of service (crash) or possibly gain privileges via a crafted system call.

意思是说，在Linux 2.6.28及以前版本内核中，IBM/S390、PowerPC、Sparc64以及MIPS 64位平台的ABI要求在系统调用时，用户空间程序将系统调用中32位的参数存放在64位的寄存器中要做到正确的符号扩展，但是用户空间程序却不能保证做到这点，这样就会可以通过向有漏洞的系统调用传送特制参数便可以导致系统崩溃或获得权限提升。

举例来说，假如下面是个系统调用的内核代码，参数是32位的无符号整型，但使用的64位寄存器传参，上面提及到平台的ABI要求32为参数存放在64位寄存器中要符号扩展，由程序的调用者来完成，在系统调用的函数中则由用户程序来保证进行了正确的寄存器符号扩展，但用户空间程序却无法保证。
{% highlight c %}
asmlinkage long sys_example(unsigned int index)
{
    if (index > 5)
        return -EINVAL;
    return example_array[index];
}
{% endhighlight %}

在上面程序中，调用程序必须将索引符号扩展为64位，如传入参数index=3，那么将寄存器的低32位赋值为3，并未修改高32位，此时该寄存器的高32位假设为0xFFFFFFFF，在进入该系统调用函数时，由于编译器认为你已经进行符号扩展了，所以直接引用64位寄存器的值代表index，此时index=-4294967293，判断不大于5，返回example_array[-4294967293]，很可能访问到一块没有权限访问的地址空间或者其他地址异常的错误而导致程序崩溃。

怎么去解决这个问题呢，也许你会想既然用户空间没有进行寄存器的符号扩展，那么我在系统调用函数之前加入一些汇编代码将寄存器进行符号扩展，但有个问题是，系统调用前代码都是公共的，因此并不能将某个寄存器一定符号扩展。

在Linux内核中，解决这个问题的办法很巧妙，它先将所有参数都当成long类型（64位），然后再强制转化到相应的类型，这样就能解决问题了。如果去每个系统调用中一一这么做，这是一般程序员选择的做法，但写内核的大牛们不仅要完成功能，而且完成得有艺术！这就出现了现在的做法，定义了下面的宏：

{% highlight c %}
#define __SYSCALL_DEFINEx(x, name, ...)					\
	asmlinkage long sys##name(__SC_DECL##x(__VA_ARGS__));		\
	static inline long SYSC##name(__SC_DECL##x(__VA_ARGS__));	\
	asmlinkage long SyS##name(__SC_LONG##x(__VA_ARGS__))		\
	{								\
		__SC_TEST##x(__VA_ARGS__);				\
		return (long) SYSC##name(__SC_CAST##x(__VA_ARGS__));	\
	}								\
	SYSCALL_ALIAS(sys##name, SyS##name);				\
	static inline long SYSC##name(__SC_DECL##x(__VA_ARGS__))
{% endhighlight %}

那么仍然是以open系统调用为例，

{% highlight c linenos %}
SYSCALL_DEFINE3(open, const char __user *, filename, int, flags, umode_t, mode)
{
	…
}

展开之后如下：
asmlinkage long sys_open(const char __user * filename, int flags, umode_t mode);
static inline long SYSC_open(const char __user * filename, int flags, umode_t mode);
asmlinkage long SyS_open((long)filename, (long)flags, (long)mode)
{
	__SC_TEST3(const, char __user * filename, int, flags, umode_t, mode);   
	return (long)SYSC_open(const char __user * filename, int flags, umode_t mode);
}
SYSCALL_ALIAS(sys_open, SyS_open);
static inline long SYSC_open(const char __user * filename, int flags, umode_t mode)
{
	…
}
{% endhighlight %}

* 第11行 `__SC_TEST3` 宏没展开，其实是编译时检查类型是否错误的代码，和我们这里讨论的关系不大，展开之后也是个很有意思的宏定义，可以参考我[这里的一篇文章](http://blog.csdn.net/hazir/article/details/9336299 'Linux Kernel 代码艺术——编译时断言') 。
* 第17行` SYSCALL_ALIAS` 宏夜没展开，意思即 sys_open 函数的别名是 SyS_open。
* 因此，系统调用调转到 sys_open 处即执行第三行的 SyS_open 函数，注意该函数的参数全为 long 类型，该函数又直接调用 SYSC_open，而 SYSC_open 函数的参数又转化为 sys_open 原来正确的类型。这样一来就消除了用户空间不保证参数符号扩展的问题了，因为此时实际上系统调用函数由 SyS_open 函数调用了，它来保证 32 位寄存器参数正确的符号扩展。

由于某些体系结构是不存在此类问题的，如x86_64等，Linux内核定义了一个配置选项`CONFIG_HAVE_SYSCALL_WRAPPERS`，一开始介绍的扩展的宏定义是在没有配置该选项扩展的结果，如果是S390、PowerPC、Sparc 64等平台就需要配置该选项。

## 说明
本文前面部分主要介绍一些表面的东西，比较简单，对于后面部分是我思考的部分，在去年读内核时我就有这个疑问——为什么内核要这样把简单的系统调用定义成这么复杂的宏？这两天通过查一下资料终于找到能解释这个问题的理由，原因在于漏洞 CVE-2009-0029 导致系统调用函数定义不能直接用那个原型，内核大牛们就写出了现在这样的代码。但需要说明的是，对于 CVE-2009-0029 产生的原因、解决方法，由于涉及到我并不熟悉的体系结构平台，所以上面只是我根据网上仅有很少的资料进行推断出来的，肯定不是很准确，希望大家能指正！

## 参考资料
* [http://wangcong.org/blog/archives/1306](http://wangcong.org/blog/archives/1306)  我找到的唯一一篇国人写的关于 CVE-2009-0029 的博客
* [https://bugzilla.redhat.com/show_bug.cgi?format=multiple&id=479969](https://bugzilla.redhat.com/show_bug.cgi?format=multiple&id=479969)  Redhat 公司报告的关于 CVE-2009-0029 漏洞的说明
* [http://stackoverflow.com/questions/15105313/linux-kernel-system-call-naming-convention](http://stackoverflow.com/questions/15105313/linux-kernel-system-call-naming-convention)  Stackoverflow上这个问题的回答让我引导我一步一步解决
* [http://sota.gen.nz/compat2/](http://sota.gen.nz/compat2/) 该博客描述了关于内核系统调用令一个著名的漏洞 CVE-2010-3301，CVE的说明[在这](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2010-3301&cid=1)

