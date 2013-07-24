---
layout: post
title: "C语言中结构体赋值"
description: ""
category: c_cplusplus 
tags: [c, struct]
---
{% include JB/setup %}

今天帮师姐调一个程序的BUG，师姐的程序中有个结构体直接赋值的语句，在我印象中结构体好像是不能直接赋值的，正如数组不能直接赋值那样，我怀疑这个地方有问题，但最后证明并不是这个问题。那么就总结一下C语言中结构体赋值的问题吧：
##结构体直接赋值的实现##
下面是一个实例：
{% highlight c %}
#include <stdio.h>

struct Foo {
	char a;
	int b;
	double c;
}foo1, foo2;          //define two structs with three different fields

void struct_assign(void)
{
	foo2 = foo1;       //structure directly assignment
}

int main()
{
	foo1.a = 'a';
	foo1.b = 1;
	foo1.c = 3.14;
	struct_assign();
	printf("%c %d %lf\n", foo2.a, foo2.b, foo2.c);

	return 0;	
}
{% endhighlight %}
我在Ubuntu 13.04下使用gcc 4.7.3 编译运行得到的结果，如下所示：
{% highlight bash %}
guohl@guohailin:~/Documents/c$ gcc struct_test1.c -o struct_test1
guohl@guohailin:~/Documents/c$ ./struct_test1 
a 1 3.140000
{% endhighlight %}
可以从结果上看出，结构体直接赋值在C语言下是可行的，我们看看`struct_assign()`函数的汇编实现，从而从底层看看C语言是如何实现两个结构体之间的赋值操作的：
{% highlight c %}
struct_assign:
	pushl	%ebp
	movl	%esp, %ebp
	movl	foo1, %eax
	movl	%eax, foo2      //copy the first 4 bytes from foo1 to foo2
	movl	foo1+4, %eax
	movl	%eax, foo2+4    //copy the second 4 bytes from foo1 to foo2       
	movl	foo1+8, %eax
	movl	%eax, foo2+8    //copy the third 4 bytes from foo1 to foo2  
	movl	foo1+12, %eax
	movl	%eax, foo2+12   //copy the forth 4 bytes from foo1 to foo2  
	popl	%ebp
	ret
{% endhighlight %}
这段汇编比较简单，由于结构体的对齐的特性，`sizeof(srtruct Foo)=16`,通过四次`movl`操作将foo1的结构体内容拷贝到结构体foo2中。从汇编上看出，结构体赋值，采用的类似于`memcpy`这种形式，而不是逐个字段的拷贝。
##复杂结构体的赋值##
如果结构体中含有其它复杂数据类型呢，例如数组、指针、结构体等，从上面的汇编实现可以看出，只要两个结构体类型相同，就可以实现赋值，如下例：
{% highlight c %}
#include <stdio.h>

struct Foo {
	int n;
	double d[2];
	char *p_c;
}foo1, foo2;

int main()
{
	char *c = (char *) malloc (4*sizeof(char));
	c[0] = 'a'; c[1] = 'b'; c[2] = 'c'; c[3] = '\0';
	
	foo1.n = 1;
	foo1.d[0] = 2; foo1.d[1] = 3;
	foo1.p_c = c;

	foo2 = foo1;     //assign foo1 to foo2
	
	printf("%d %lf %lf %s\n", foo2.n, foo2.d[0], foo2.d[1], foo2.p_c);
	
	return 0;
}
{% endhighlight %}
运行结果如下：

{% highlight bash %}
guohl@guohailin:~/Documents/c$ gcc struct_test2.c -o struct_test2
guohl@guohailin:~/Documents/c$ ./struct_test2
1 2.000000 3.000000 abc
{% endhighlight %}
可以看出结果和我们想象的是一样的。再次验证结构体的赋值，是直接结构体的内存的拷贝！但正是这个问题，如上面的实例，foo1 和 foo2 中p\_c 指针都是指向我们申请的一块大小为4个字节的内存区域，这里注意的是，结构体的拷贝只是浅拷贝，即指针p\_c的赋值并不会导致再申请一块内存区域，让foo2的p\_c指向它。那么，如果释放掉foo1中的p\_c指向的内存，此时foo2中p\_c变成野指针，这是对foo2的p\_c操作就会出现一些不可预见的问题！在C\+\+中引入了一种可以允许用户重载结构体赋值操作运算，那么我们就可以根据语义重载赋值操作。
##数组是二等公民##
[二等公民](http://zh.wikipedia.org/wiki/%E4%BA%8C%E7%AD%89%E5%85%AC%E6%B0%91 "二等公民")在维基百科上的解释是：
>二等公民不是一个正式的术语，用来描述一个社会体系内对一部分人的歧视或对外来人口的政治限制，即使他们作为一个公民或合法居民的地位。 二等公民虽然不一定是奴隶或罪犯，但他们只享有有限的合法权利、公民权利和经济机会，并经常受到虐待或忽视。法律无视二等公民，不向他们提供保护，甚至在制订法律时可能会根本不考虑他们的利益。划分出二等公民的行为，普遍被视为一种侵犯人权的行为。 典型的二等公民所面临的障碍包括但不仅限于（缺乏或丧失表决权）：权利被剥夺，限制民事或军事服务（不包括任何情况下的征兵），以及限制，语言，宗教，教育，行动和结社的自由，武器的所有权，婚姻，性别认同和表达，住房和财产所有权 。  


从词条上解释可以看出二等公民与一等公民在权利上是有差别的，这个词很有意思作为计算机专业术语，其含义也有异曲同工之妙！同样我们看看维基百科对计算机的术语”[first-class citizen](http://en.wikipedia.org/wiki/First-class_citizen "一等公民")"(一等公民）的定义，一般要满足以下几点，

+ can be stored in variables and data structures
+ can be passed as a parameter to a subroutine
+ can be returned as the result of a subroutine
+ can be constructed at run-time
+ has intrinsic identity (independent of any given name)

对比着上面的定义来看C语言数组，数组作为一个函数的参数传递时，退化成一个指针; 同时，数组无法作为函数的返回值; 也许让数组更不服气的是，数组之间不能直接赋值操作，如下面的操作就是非法的：
{% highlight c %}
int a[10];
int b[10];
a = b;
{% endhighlight %}
但是如果数组包装在结构体中，那么就能进行赋值了！相比之下，结构体可以作为函数参数和返回值，这就是一等公民的待遇！至于为什么数组必须是二等公民，这是有历史原因的，大家可以参考[C 语言的发展史](http://cm.bell-labs.com/cm/cs/who/dmr/chist.html "Chistory")来看，有时间这块内容我再补上！

---

参考资料：
- [维基百科](http://en.wikipedia.org/)   
- [Stackoverflow](http://stackoverflow.com)
- International Standard **ISO/IEC 9899**



