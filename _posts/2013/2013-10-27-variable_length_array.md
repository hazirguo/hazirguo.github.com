---
layout: post
title: "GCC 中零长数组与变长数组"
description: ""
category: c_cplusplus 
tags: [c, array]
---
{% include JB/setup %}


前两天看程序，发现在某个函数中有下面这段程序：

{% highlight c %}
int n;				//define a variable n
int array[n];		//define an array with length n
{% endhighlight %}
在我所学的C语言知识中，这种数组的定义在编译时就应该有问题的，因为定义数组时，数组的长度必须要是一个大于0的整型字面值或定义为 const 的常量。例如下面这样

{% highlight c %}
int array1[10];		//valid
int const N = 10;
int array2[N];		//valid
int n = 10;
int array3[n]; 		//invalid
{% endhighlight %}
但从上面看第三种定义数组的方法也是正确的，于是，我用 gcc 去编译这段程序，发现确实没报错，而且我对此数组进行一些操作，结果也都是正确！这简直颠覆了我的知识框架！难道大学老师教我的、我平时看的书，都是错误的吗？！我开始寻找答案...

## C 语言中变长数组
最官方的解释应该是 C 语言的规范和编译器的规范说明了。

* 在 ISO/IEC9899 标准的 [6.7.5.2 Array declarators](http://busybox.net/~landley/c99-draft.html#6.7.5.2) 
中明确说明了数组的长度可以为变量的，称为**变长数组**（VLA，variable length array）。（*注：这里的变长指的是数组的长度是在运行时才能决定，但一旦决定在数组的生命周期内就不会再变。*）
* 在 GCC 标准规范的 [6.19 Arrays of Variable Length](http://gcc.gnu.org/onlinedocs/gcc/Variable-Length.html) 中指出，作为编译器扩展，GCC 在 C90 模式和 C++ 编译器下遵守 ISO C99 关于变长数组的规范。 

这下，终于安心了，原来这种语法确实是 C 语言规范，GCC 非常完美的支持了 ISO C99。但令人遗憾的是，我们的大学老师教给我们的还是老一套，虽然关系不是很大，但这也从侧面反映了我们的教育是多么地滞后！而且我们读的 C 语言书，在不加任何限定的条件下，就说某某语法是不对的，读书的人只能很痛苦地记下！小小吐槽一下，下面继续...

这种变长数组有什么好处呢？你可以使用 `alloca` 函数达到类似的动态分配数组的效果，但 alloca 函数分配的空间在函数退出时还依然存在，你需要手动地去释放所分配的空间；VLA 就不一样了，在数组名生命周期结束之后，所分配的空间也就随之释放。

当然，关于 VLA 还有很多限制，例如 ISO/IEC9899 给出了下面这个例子：

{% highlight c %}
extern int n;
int A[n]; 							// invalid: file scope VLA
extern int (*p2)[n]; 				// invalid: file scope VM
int B[100]; 						// valid: file scope but not VM
void fvla(int m, int C[m][m]); 		// valid: VLA with prototype scope
void fvla(int m, int C[m][m]) 		// valid: adjusted to auto pointer to VLA
{
	typedef int VLA[m][m]; 			// valid: block scope typedef VLA
	struct tag {
		int (*y)[n]; 				// invalid: y not ordinary identi?er
		int z[n]; 					// invalid: z not ordinary identi?er
	};
	int D[m]; 						// valid: auto VLA
	static int E[m]; 				// invalid: static block scope VLA
	extern int F[m]; 				// invalid: F has linkage and is VLA
	int (*s)[m]; 					// valid: auto pointer to VLA
	extern int (*r)[m]; 			// invalid: r has linkage and points to VLA
	static int (*q)[m] = &B; 		// valid: q is a static block pointer to VLA
}
{% endhighlight %}
至于上面语法的原因，请参考 [ISO/IEC9899](http://busybox.net/~landley/c99-draft.html) 。


## GCC 中零长数组
GCC 中允许使用零长数组，把它作为结构体的最后一个元素非常有用，下面例子出自 gcc [官方文档](http://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/Zero-Length.html#Zero-Length)。

{% highlight c %}
struct line {
	int length;
	char contents[0];
};

struct line *thisline = (struct line *) malloc (sizeof (struct line) + this_length);
thisline->length = this_length;
{% endhighlight %}

从上例就可以看出，零长数组在有固定头部的可变对象上非常适用，我们可以根据对象的大小动态地去分配结构体的大小。

在 Linux 内核中也有这种应用，例如由于 [PID 命名空间的存在](http://hazirguo.github.io/kernel/2013/10/03/linux_kernel_pid/)，每个进程 PID 需要映射到所有能看到其的命名空间上，但该进程所在的命名空间在开始并不确定（但至少为 init 命名空间），需要在运行是根据 level 的值来确定，所以在该结构体后面增加了一个长度为 1 的数组（因为至少在一个init命名空间上），使得该结构体 pid 是个可变长的结构体，在运行时根据进程所处的命名空间的 level 来决定 numbers 分配多大。（*注：虽然不是零长度的数组，但用法是一样的*）

{% highlight c %}
struct pid
{
	atomic_t count;
	unsigned int level;
	/* lists of tasks that use this pid */
	struct hlist_head tasks[PIDTYPE_MAX];
	struct rcu_head rcu;
	struct upid numbers[1];
};
{% endhighlight %}

## 参考资料
* **ISO/IEC9899**
* **GCC Online Documents**
