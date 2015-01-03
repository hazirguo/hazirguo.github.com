---
layout: post
title: "为什么 C++ 中成员函数指针是 16 字节？"
description: ""
category: c_cplusplus 
tags: [cpp, pointer, OO]
---
{% include JB/setup %}

当我们讨论指针时，通常假设它是一种可以用 `void *` 指针来表示的东西，在 x86_64 平台下是 8 个字节大小。例如，下面是来自 [维基百科中关于 x86_64 的文章](http://en.wikipedia.org/wiki/X86-64#Architectural_features) 的摘录：

> Pushes and pops on the stack are always in 8-byte strides, and **pointers are 8 bytes wide**.

从 CPU 的角度来看，指针无非就是内存的地址，所有的内存地址在 x86_64 平台下都是由 64 位来表示，所以假设它是 8 个字节是正确的。通过简单输出不同类型指针的长度，这也不难验证我们所说的。

{% highlight cpp %}
#include <iostream>

int main() {
    std::cout <<
        "sizeof(int*)      == " << sizeof(int*) << "\n"
        "sizeof(double*)   == " << sizeof(double*) << "\n"
        "sizeof(void(*)()) == " << sizeof(void(*)()) << std::endl;
}
{% endhighlight %}

编译运行上面的程序，从结果中可以看出所有的指针的长度都是 8 个字节：

{% highlight bash %}
$ uname -i
x86_64
$ g++ -Wall ./example.cc
$ ./a.out
sizeof(int*)      == 8
sizeof(double*)   == 8
sizeof(void(*)()) == 8
{% endhighlight %}

然而在 C++ 中还有一种特例——成员函数的指针。很有意思吧，成员函数指针是其它任何指针长度的两倍。这可以通过下面简单的程序来验证，输出的结果是 “16”：

{% highlight cpp %}
#include <iostream>

struct Foo {
    void bar() const { }
};

int main() {
    std::cout << sizeof(&Foo::bar) << std::endl;
}
{% endhighlight %}

这是否以为着维基百科上错了呢？显然不是！从硬件的角度来看，所有的指针仍然是 8 个字节。既然如此，那么成员函数的指针是什么呢？这是 C++ 语言的特性，这里成员函数的指针不是直接映射到硬件上的，它由运行时（编译器）来实现，会带来一些额外的开销，通常会导致性能的损失。C++ 语言规范中并没有提到实现的细节，也没有解释这种类型指针。幸运的是，Itanium C++ ABI 规范中共享了 C++ 运行时实现的细节——举例来说，它解释了 Virtual Table、RTTI 和异常是如何实现的，在 §2.3 中也解释了成员指针：

> A pointer to member function is a pair as follows:
>
> ptr:
>
> For a non-virtual function, this field is a simple function pointer. For a virtual function, it is 1 plus the virtual table offset (in bytes) of the function, represented as a ptrdiff_t. The value zero represents a NULL pointer, independent of the adjustment field value below.
>
> adj:
>
> The required adjustment to this, represented as a ptrdiff_t.

所以，成员指针是 16 字节而不是 8 字节，因为在简单函数指针的后面还需要保存怎样调整 “this" 指针（总是隐式地传递给非静态成员函数）的信息。 ABI 规范并没有说为什么以及什么时候需要调整 this 指针。可能一开始并不是很明显，让我们先看下面类继承的例子：

{% highlight cpp %}
struct A {
    void foo() const { }
    char pad0[32];
};

struct B {
    void bar() const { }
    char pad2[64];
};

struct C : A, B
{ };
{% endhighlight %}

A 和 B 都有一个非静态成员函数以及一个数据成员。这两个方法可以通过隐式传递给它们的 “this" 指针来访问到它们类中的数据成员。为了访问到任意的数据成员，需要在 "this" 指针上加上一个偏移，偏移是数据成员到类对象基址的偏移，可以由 ptrdiff_t 来表示。然而事情在多重继承时将会变得更复杂。我们有一个类 C 继承了 A 和 B，将会发生什么呢？编译器将 A 和 B 同时放到内存中，B 在 A 之下，因此，A 类的方法和 B 类的方法看到的 this 指针的值是不一样的。这可以通过实践来简单验证，如：

{% highlight cpp %}
#include <iostream>

struct A {
    void foo() const {
        std::cout << "A's this: " << this << std::endl;
    }
    char pad0[32];
};

struct B {
    void bar() const {
        std::cout << "B's this: " << this << std::endl;
    }
    char pad2[64];
};

struct C : A, B
{ };

int main()
{
    C obj;
    obj.foo();
    obj.bar();
}
{% endhighlight %}

{% highlight bash %}
$ g++ -Wall -o test ./test.cc && ./test
A's this: 0x7fff57ddfb48
B's this: 0x7fff57ddfb68
{% endhighlight %}

正如你看到的，“this” 指针的值传给 B 的方法要比 A 的方法要大 32 字节——一个类 A 对象的实际大小。但是，当我们用下面的函数通过指针来调用类 C 的方法时，会发生什么呢？

{% highlight cpp %}
void call_by_ptr(const C &obj, void (C::*mem_func)() const) {
    (obj.*mem_func)();
}
{% endhighlight %}

与调用什么函数有关，不同的 "this" 指针值会被传递到这些函数中。但是 `call_by_ptr` 函数并不知道它的参数是 `foo()` 的指针还是 `bar()` 的指针，能知道该信息的唯一时机是这些方法使用时。这就是为什么成员函数的指针在调用之前需要知道如何调整 `this` 指针。现在，我们将所有的放到一个简单的程序，阐释了内部工作的机制：

{% highlight cpp %}
#include <iostream>

struct A {
    void foo() const {
        std::cout << "A's this:\t" << this << std::endl;
    }
    char pad0[32];
};

struct B {
    void bar() const {
        std::cout << "B's this:\t" << this << std::endl;
    }
    char pad2[64];
};

struct C : A, B
{ };

void call_by_ptr(const C &obj, void (C::*mem_func)() const)
{
    void *data[2];
    std::memcpy(data, &mem_func, sizeof(mem_func));
    std::cout << "------------------------------\n"
        "Object ptr:\t" << &obj <<
        "\nFunction ptr:\t" << data[0] <<
        "\nPointer adj:\t" << data[1] << std::endl;
    (obj.*mem_func)();
}

int main()
{
    C obj;
    call_by_ptr(obj, &C::foo);
    call_by_ptr(obj, &C::bar);
}
{% endhighlight %}

上面的程序输出如下：

{% highlight bash %}
------------------------------
Object ptr:    0x7fff535dfb28
Function ptr:  0x10c620cac
Pointer adj:   0
A's this:    0x7fff535dfb28
------------------------------
Object ptr:    0x7fff535dfb28
Function ptr:  0x10c620cfe
Pointer adj:   0x20
B's this:    0x7fff535dfb48
{% endhighlight %}

希望本文能使问题变得更明确一点。

---
文章出自：[http://741mhz.com/wide-pointers/](http://741mhz.com/wide-pointers/)

