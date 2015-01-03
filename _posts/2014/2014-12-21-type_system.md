---
layout: post
title: "编程语言的类型系统"
description: ""
category: programming 
tags: [python, type]
---
{% include JB/setup %}


每学一门新的编程语言时，在看到介绍该门编程语言的特点时，经常会遇到 *静态*、*动态*、*强*、*弱* 、*隐式*、*显式* 类型等字样，似懂非懂，这里结合网上的资料总结一下它们的含义以及区别，描述不一定专业、准确，但求能进一步理解这些词的概念即可。

[类型系统（Type System）](http://en.wikipedia.org/wiki/Type_system)用于定义如何将编程语言中的数值和表达式归类为许多不同的类型，如何操作这些类型，这些类型如何互相作用。根据这些种种不同，可以将编程语言分为以下类别：


## 静态类型编程语言 vs 动态类型编程语言

在静态类型语言，每个变量名字都绑定到：

* 一个类型，编译时通过变量的定义来绑定
* 一个对象，这是可选的，如果变量名没绑定到一个对象，那么名字指向 *null*

例如下面是 Java 中定义字符串类型变量：

```
String str1;        //reference to null
String str2 = "Hello world";   
```

定义了两个变量 str1、str2，它们的类型均是 `String` 类型，而 str1 并没有指向特定的对象，但 str2 指向了一个 "Hello world" 的字符串对象。

注意的是，静态类型的编程语言，一旦变量名绑定到一个类型（通过声明语句），那么它就只能绑定到这种类型的对象上（通过赋值语句）。如果试图该某种类型的变量绑定到不同类型的对象上，将在编译时抛出类型异常的错误。

例如下面的代码将一个整型的对象赋值给 `String` 类型的变量会报错：

```
String str1;
str = 10;
```

而在动态类型语言中，每个变量名只于对象进行绑定。在程序运行时通过赋值操作来将名字绑定到对象上，如果再将名字绑定到另一种不同类型的对象上也是允许的。

例如下面是 Python 中代码：

```
var = "Hello world"
//...
var = 10
```

先将 var 变量赋值一个 "Hello world" 字符串对象，运行过程中随时可以将 var 赋值其它类型的对象。这在 Python 中并不会报错。

下图可以反应两者的区别：

![static-dynamic-typed-language](http://upload-images.jianshu.io/upload_images/13017-d7e2288a561238eb.png?imageView2/2/w/1240/q/100)


由上可以看出 Java、C/C++、C# 等属于静态类型编程语言，而 Python、PHP、Perl 等属于动态类型语言。


## 强类型编程语言 vs 弱类型编程语言

根据变量能否**隐式**（implicit）转换为**无关**(unrelated)的类型，可以将编程语言分成弱类型编程语言和强类型编程语言，前者可以完成转换，后者则需要通过**显式**(explicit)地转换。注意这里的**无关类型**转换指的是类似于数值类型与字符串类型的转换，而大多数语言允许**相关类型**之间的隐式转换，例如 int 类型到 float 类型的转换。

在弱类型编程语言中，数字 9 和字符串 "9" 是可以相互转换的，例如下面的代码是合法的：

```
a  = 9
b = "9"
c = concatenate(a, b)  // produces "99"
d = add(a, b)          // produces 18
```

但是在强类型编程语言中，后面两条语句会抛出类型异常，为了避免这些异常，我们往往需要通过一些显式类型转换操作来完成：

```
a  = 9
b = "9"
c = concatenate(str(a), b)
d = add(a, int(b) )
```

根据上面的描述，我们可以知道像 Python、Java 等属于强类型编程语言，而 Perl、PHP 等属于弱类型编程语言。


## 显式类型编程语言 vs 隐式类型编程语言

还有一种区分方法是，根据变量名是否需要显式给出类型的声明，来将语言分为显式类型语言和隐式类型语言。前者需要在定义变量时显式给出变量的类型，而后者可以使用[类型推论](http://en.wikipedia.org/wiki/Type_inference)来决定变量的类型。

大多数静态类型语言，例如 Java、C/C++ 都是显式类型语言，但是有些则不是，如 Haskell、ML 等，可以基于变量的操作来推断其类型；[Scala](http://www.scala-lang.org/node/25) 是一种新型的静态类型编程语言，它运行在 Java 虚拟机上，也是使用的是类型推断；[Boo](http://boo.codehaus.org/) 是一种类 Python 编程语言，运行在 .NET CLI 之上，使用的是类型推断。


## 总结

这篇文章只是给出了一个非形式化定义，旨在理解各种不同编程在类型系统的设计思想，当然本文只提到类型系统的冰山一角。

本文绝大数内容编译自这篇[文章](https://pythonconquerstheuniverse.wordpress.com/2009/10/03/static-vs-dynamic-typing-of-programming-languages/)，还参考 [知乎问答](http://www.zhihu.com/question/19918532)，wiki 相关词条，特致谢！

