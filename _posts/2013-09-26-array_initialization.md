---
layout: post
title: "Linux Kernel代码艺术——数组初始化"
description: ""
category: c_cplusplus 
tags: [linux, array]
---
{% include JB/setup %}

前几天看内核中系统调用代码，在系统调用向量表初始化中，有下面这段代码写的让我有点摸不着头脑：

{% highlight c %}
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
	/*
	 * Smells like a compiler bug -- it doesn't work
	 * when the & below is removed.
	 */
	[0 ... __NR_syscall_max] = &sys_ni_syscall,
#include <asm/syscalls_32.h>
};
{% endhighlight %}

咱先不管上面代码的意思，先来回顾一下 C 语言中数组初始化的相关知识，然后再回头来理解上面这段代码。

## 数组初始化 ##
C 语言中数组的初始化，可以在定义时就给出其初始值，以逗号隔开，用花括号括起来，例如：

{% highlight c %}
int my_array[5] = {0, 1, 2, 3, 4};
{% endhighlight %}
当然你可以不用显示地去初始化所有的元素，例如，下面的代码就是显示初始化了数组的前三项，后面两项默认为0：
 
{% highlight c %}
int my_array[5] = {0, 1, 2};
{% endhighlight %}

在 C89 标准中，要求按照数组中元素固定的顺序对数组的元素进行初始化；然而在 ISO C99 中，你可以以任意的顺序对数组元素初始化，只是需要给出数组元素所在的索引号；当然 GNU 编译器 GCC 对 C89 进行了扩展，也允许这么做。为了指明初始特殊的数组元素，需要在元素值前加上 `[index] = `，如：

{% highlight c %}
int my_array[6] = { [4] = 29, [2] = 15 };
或者写成：
int my_array[6] = { [4] 29, [2] 15 };     //省略到索引与值之间的=，GCC 2.5 之后该用法已经过时了，但 GCC 仍然支持
两者均等价于：
int my_array[6] = {0, 0, 15, 0, 29, 0};
{% endhighlight %}

GNU 还有一个扩展：在需要将一个范围内的元素初始化为同一值时，可以使用 `[first ... last] = value` 这样的语法：
{% highlight c %}
int my_array[100] = { [0 ... 9] = 1, [10 ... 98] = 2, 3 };
{% endhighlight %}
这是将my_array数组的第0~9个元素初始化为1， 第10~98个元素初始化为2， 第99个元素初始化为3（你也可以显示地写成[99] = 3)。**注意**：在语法中`...` 两边必须要留有空格符。

## 回到上面 ##
对数组特定元素进行初始化我之前还真没遇到过，但也是 C 标准所支持的。内核中系统调用表是指根据系统调用号来找到系统调用的函数入口地址，结合上面数组初始化这个语法点，再回头看看上面系统调用表的定义：

{% highlight c %}
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
	[0 ... __NR_syscall_max] = &sys_ni_syscall,
#include <asm/syscalls_32.h>
};
{% endhighlight %}

先对表中所有 __NR_syscall_max+1 项初始化为指向 sys_ni_syscall 的函数，该函数只返回 -ENOSYS，表示该系统调用未实现。接下来包含一个头文件`#include <asm/syscalls_32.h>`，该文件是在编译时生成的，内容为：

{% highlight c %}
__SYSCALL_I386(0, sys_restart_syscall, sys_restart_syscall)
__SYSCALL_I386(1, sys_exit, sys_exit)
__SYSCALL_I386(2, sys_fork, stub32_fork)
__SYSCALL_I386(3, sys_read, sys_read)
__SYSCALL_I386(4, sys_write, sys_write)
__SYSCALL_I386(5, sys_open, compat_sys_open)
...
{% endhighlight %}

__SYSCALL_I386 是一个宏定义：

{% highlight c %}
#define __SYSCALL_I386(nr, sym, compat) [nr] = sym,
{% endhighlight %}

这样上面的系统调用表定义就展开为：
{% highlight c %}
const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
	[0 ... __NR_syscall_max] = &sys_ni_syscall,
	[0] = sys_restart_syscall,
	[1] = sys_exit,
	[2] = sys_fork,
	[3] = sys_read,
	//...
};
{% endhighlight %}

当用户进程发生系统调用，通过软中断 int 0x80 或者 sysenter 指令陷入到内核态，首先保存寄存器，然后检查系统调用号是否合法，最后跳转到相应的内核系统调用函数中执行：
{% highlight c %}
ENTRY(system_call)
	pushl_cfi %eax			# 保存原始 eax
	SAVE_ALL                # 保存寄存器帧
	GET_THREAD_INFO(%ebp)
	testl $_TIF_WORK_SYSCALL_ENTRY,TI_flags(%ebp)    # 检查是否跟踪系统调用标志
	jnz syscall_trace_entry
	cmpl $(NR_syscalls), %eax    # 检查系统调用号是否合法
	jae syscall_badsys
syscall_call:
	call *sys_call_table(,%eax,4)   # 调用相应函数，等价于 call sys_call_table[%eax*4]
{% endhighlight %}

上面就是系统调用的进入过程，比较简单，这里只是说明了我们之前定义的系统调用表 sys_call_table 的用处。

## 再举一例 ##
内核中还有其他地方应用到此种初始化数组的方法：

{% highlight c %}
/* There are machines which are known to not boot with the GDT
   being 8-byte unaligned.  Intel recommends 16 byte alignment. */
static const u64 boot_gdt[] __attribute__((aligned(16))) = {
	/* CS: code, read/execute, 4 GB, base 0 */
	[GDT_ENTRY_BOOT_CS] = GDT_ENTRY(0xc09b, 0, 0xfffff),
	/* DS: data, read/write, 4 GB, base 0 */
	[GDT_ENTRY_BOOT_DS] = GDT_ENTRY(0xc093, 0, 0xfffff),
	/* TSS: 32-bit tss, 104 bytes, base 4096 */
	/* We only have a TSS here to keep Intel VT happy;
	   we don't actually use it for anything. */
	[GDT_ENTRY_BOOT_TSS] = GDT_ENTRY(0x0089, 4096, 103),
};
{% endhighlight %}
这是对系统启动时对全局符号表GDT的初始化。


---

参考资料：

* [http://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html#Initializing-Arrays](http://www.gnu.org/software/gnu-c-manual/gnu-c-manual.html#Initializing-Arrays)
* [http://stackoverflow.com/questions/201101/how-to-initialize-an-array-in-c](http://stackoverflow.com/questions/201101/how-to-initialize-an-array-in-c)
