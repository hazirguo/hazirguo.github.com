---
layout: post
title: "Python 数据类型及其用法"
description: ""
category: python 
tags: [python]
---
{% include JB/setup %}


本文总结一下Python中用到的各种数据类型，以及如何使用可以使得我们的代码变得简洁。
##基本结构##
我们首先要看的是几乎任何语言都具有的数据类型，包括字符串、整型、浮点型以及布尔类型。这些基本数据类型组成了基本控制块，从而创建我们的Python应用程序。
###字符串###
[字符串(String)](http://docs.python.org/3.3/library/string.html)  是一段文本字符，通常以某种形式向用户输出。如果我们打开Python的解释器，我们试着最常见的输出“Hello World!"应用:

{% highlight python %}
>>> print ("Hello, world!")
Hello, world!
{% endhighlight %}

Python 中的数据类型不需要像Java或C语言那样显示的定义，这就意味着在Python中字符串就是简单地用引号括起来来标识，向上面的“Hello, world!"那样。我们也可以使用单引号而不是双引号，当我们字符串中就有双引号时，使用单引号来表示整个字符串更加方便，如：
{% highlight python %}
>>> print ("This is David's program")
This is David's program

>>> print ('"Hello", said David')
"Hello", said David
{% endhighlight %}
从上面你就可以看出在不同的条件下如何交叉使用不同的引号。

字符串提供了许多内置的函数，这些在很多Python程序中很有用处，它们包括：

* endswith()  - 检查字符串是否以给定字符串结尾
* startswith() - 检查字符串是否以给定字符串开始
* upper() - 将字符串所有字符变成大写
* lower() - 将字符串所有字符变成小写
* isupper()/islower() - 检测字符串是否全是大写/小写
* 我们也可以将字符串作为一个参数传入函数`len()`来返回字符串的长度，例如 len("david")

Python中字符串也是可迭代的，迭代的概念待会在列表和字典数据类型中可以做更深入的了解。这意味着我们可以依次循环字符串中每个字符，这在其它语言一般是不行的。

另一个关于字符串的技巧是，我们可以对字符串进行格式化。最好的方法是用 `format`语句来完成，这个函数为你处理你传入对象的类型以字符串形式显示，所以你可以传入任何类型的对象到 `format` 方法用来输出。如下：
{% highlight python %}
>>> string_1 = "Hello, {}, here is your string".format("David")
'Hello, David, here is your string'
>>>
>>> string_2 = "Hello, {}, your are {} years old".format("David", 23)
'Hello, David, you are 23 years old'
>>>
>>> string_3 = "Hello, {}, your are {} years old".format("David", 23.5)
'Hello, David, you are 23.5 years old'
{% endhighlight %}
从上面的代码可以清晰地看出，`format` 方法可以用来将任意的 Python 数据类型替换成字符串。

###布尔类型以及if语句###
[布尔值](http://docs.python.org/3.3/library/stdtypes.html#boolean-values) (True or False) 在任何语言中都是至关重要的，它们可以使我们根据变量的真假值来做出判断，通过代码可以用来控制程序的路径。在 Python 中，布尔值的首字母是大写的：`True` 和 `False`。举例来说，使用上面提到的字符串的一些方法，我们可以测试一个字符串是否是大写，并输出正确的结果：
{% highlight python %}
>>> if "david".isupper():
	print ('"david" is uppercase')
else:
	print ('"david" is lowercase')

	
"david" is lowercase
{% endhighlight %}
Python 中`if` 语句用来检查第一个条件是否为真，如果为True就会打印出 "david" is uppercase。然而在这个例子中，它返回False，所以 else 块执行，打印出 "david" is lowercase 。

在Python中，数据有其隐式的真假值的，这对于使代码简短、准确非常有帮助，就不需要每个地方都做判断了。下面就是Python中使用隐式布尔值的例子：
{% highlight python %}
>>> bool('')
False
>>> bool('hello')
True
>>> a = None
>>> b = 1
>>> bool(a)
False
>>> bool(b)
True
>>> if a:
     	print ("Yes")

>>> if b:
       	print ("Yes")
Yes
{% endhighlight %}
在这个例子中，你可以看到我们可以检查变量的真假值。通过调用`bool`方法，传入我们的变量给其做参数，返回True 或者 False。空值或者None（Python中类似其它语言Null 或者 Nil 的值）都会被认为是 False ，而其它情形则被认为是 True。

###整型、浮点型、复数###
[数值（Numbers）](http://docs.python.org/3.3/library/numbers.html) 在Python中就像通常格式所代表的那样，整型如1，10，56，1045，100433 等等。

浮点型，通常我们需要小数时使用，例如1.23、0.34532、23.4667 等等。

Python 支持简单和高级的数学计算，有着广泛的用途，最常用的还是基本的算术运算：
{% highlight python %}
>>> 1 + 1
2
>>> 1 - 1
0
>>> 2 * 2
4
>>> 2 / 2
1
{% endhighlight %}
查看 [Python Docs](http://www.python.org/doc//current/tutorial/introduction.html#numbers),复数类型解释如下：
>Complex numbers are also supported; imaginary numbers are written with a suffix of j or J. Complex numbers with a nonzero real component are written as (real+imagj), or can be created with the complex(real, imag) function.
{% highlight python %}
>>> 1j * 1J
(-1+0j)
>>> 1j * complex(0,1)
(-1+0j)
>>> 3+1j*3
(3+3j)
>>> (3+1j)*3
(9+3j)
>>> (1+2j)/(1+1j)
(1.5+0.5j)
{% endhighlight %}
>Complex numbers are always represented as two floating point numbers, the real and imaginary part. To extract these parts from a complex number z, use z.real and z.imag.
{% highlight python %}
>>> a=1.5+0.5j
>>> a.real
1.5
>>> a.imag
0.5
{% endhighlight %}
##更多高级结构##
###列表以及for循环###
[列表（list）](http://docs.python.org/3.3/tutorial/datastructures.html) 是Python以及其它语言中最常用到的数据结构之一。Python 使用中括号(\[\])用来解析列表，允许你以任意的顺序存储数据，从而方便处理。举例来说，如果我们需要从一个对象中抽取一些数据，我们可以将这些数据存在列表中以备后面使用：
{% highlight python %}
>>> my_list = []
>>> for object in objects:
		my_list.append(object.name)
>>> print (my_list)
['name_1', 'name_2', 'name_3']
{% endhighlight %}
上面的例子并不好，但你可以看到我们是如果从对象中提取数据，并将它们放到我们的列表中。类似的，如果我们有两个列表，使用`extend`方法我们可以将它们合并成一个：
{% highlight python %}
>>> list_1 = [1,2,3]
>>> list_2 = [4,5,6]
>>> print (list_1.append(list_2))
[1,2,3,[4,5,6]]
>>> print (list_1.extend(list_2))
[1,2,3,4,5,6]
{% endhighlight %}
这能清楚地看出这两个重要方法的不同。

在我们的第一个例子中，使用到了for循环来从我们的数据中构成列表，一种更简洁的解决方法可以使用列表扩展，这可以将整个过程压缩成一行。这是Python提供的一种高效的解决方法：
{% highlight python %}
>>> my_list = [object.name for object in objects]
>>> print my_list
['name_1', 'name_2', 'name_3']
{% endhighlight %}
再来看看这是如何映射到第一中方法的：在列表中，我们遍历objects，使用object作为从objects取出每个元素的变量名，最后我们取出object的name构成我们的列表。

除了使用方括号用来构造列表对象之外，我们还可以使用列表构造函数`list()`来建立我们的列表，如下：
{% highlight python %}
>>> list('a')
['a']
>>> values = (1,2,3,4) # tuple object see below
>>> list(values)
[1,2,3,4]
{% endhighlight %}
###字典###
另一个普遍使用的Python数据结构是[字典（Dictionaries）](http://docs.python.org/3.3/tutorial/datastructures.html#dictionaries)。字典用来存储大量数据，并且提供快速处理的方法。一种常见的字典应用的例子是通讯录系统。在字典这种数据结构中，你需要一个唯一的‘key’用来查询它对应的‘value’。在通讯录中，那个唯一的‘key’可以是电话号码，因此，字典结构是这样的：
{% highlight python %}
>>> phonebook = {'012345678': 'A Person',
                 '987654321': 'A.N Other'}
{% endhighlight %}
这儿我们有一个字典，电话号码作为‘key’，对应人名作为‘value’。现在我们可以使用这个结构来查询，如下：

{% highlight python %}
>>> phonebook['012345678']
'A Person'
{% endhighlight %}
Python的字典同样提供迭代每个数的标准方法，它们是：iteritems、iterkeys以及itervalues。这些函数分别允许我们迭代keys和values,仅迭代keys以及仅迭代values。下面是例子：

迭代所有的项：
{% highlight python %}
>>> for key, value in phonebook.iteritems():
        print (key, value)
987654321 A.N Other
012345678 A Person
{% endhighlight %}
仅迭代所有的keys：
{% highlight python %}
>>> for key in phonebook.iterkeys():
        print (key)
987654321
012345678
{% endhighlight %}
仅迭代所有的values:
{% highlight python %}
>>> for value in phonebook.itervalues():
        print (value)
A.N Other
A Person
{% endhighlight %}
与列表一样，也有不止一种创建字典的方法。到目前为止，我们只使用了逐个列举并用花括号括起来的方法来创建字典。然而，我们还可以像下面这样使用字典构造函数创建，需要传入关键字参数来创建我们字典的键值对：
{% highlight python %}
>>> dict(a=1, b=2, c=3)
{'a': 1, 'c': 3, 'b': 2}
{% endhighlight %}
###Sets 和 Frozensets###
类似与列表，我们还有[集合（Sets）](http://docs.python.org/3.3/library/sets.html) 以及 [Frozensets](http://docs.python.org/3.3/library/stdtypes.html#frozenset) 的数据结构。集合允许我们像列表那样存储数据，但不同的是集合里不允许有重复元素。当我们想确保每个数据只有一份拷贝的时候，这个是非常棒的。Frozensets 几乎和普通的集合是一样的，但它是不可变的类型(immutable)，这意味着一旦它被创建，它就不能再以任何方式改变。

同样，我们创建集合的方法有列举法（注意：相同的元素会自动删除）：
{% highlight python %}	
>>> a = {1,2,3,3}
set([1, 2, 3])
{% endhighlight %}
也可以使用构造函数：
{% highlight python %}
>>> set([1,2,3,3])
set([1, 2, 3])
{% endhighlight %}
Python 提供了强大的集合操作方法，我们可以完成数学中集合的并集、交集、差集等操作，如下：
{% highlight python %}
>>> a = {1,2,3}
>>> b = {3,4,5}
>>> a.union(b)
set([1, 2, 3, 4, 5])
>>> 
>>> a.difference(b)
set([1, 2])
>>> 
>>> a.intersection(b)
set([3])
{% endhighlight %}
###元组###
最后介绍的 Python 数据类型是[元组（tuples）](http://docs.python.org/3.3/tutorial/datastructures.html#tuples-and-sequences)。元组类似与列表，用逗号（,）来分隔存储的数据，与列表不同的是元组是不可变类型（immutable），列表可以任你插入或改变，而元组不行。所以，元组适用于你的数据是固定且不需改变的情形。从内存的角度来看，使用元组有一大好处是，Python可以明确地知道需要分配多少内存给元组（同样 Frozensets 也有这个好处）。

Python 同样也提供列举法和构造法来创建一个元组：
{% highlight python %}
>>> my_tuple = (1,2,3,4)
(1,2,3,4)
>>> 
>>> a = tuple([1,2,3,4])
>>> type(a)
<class 'tuple'>
{% endhighlight %}
关于元组这儿有几点特殊的情形需要说明。你可以使用空小括号或者使用不传参的构造函数来创建一个空的元组，但创建仅有一个元素的元组时还需要一个额外的逗号，乍一看好像是错误的，但这才是正确的语法。这是因为如果没有这个逗号，Python 会忽略小括号，仅仅只看到里面的值。下面的例子说明了这点：
{% highlight python %}
>>> empty_tup = ()
>>> type(empty_tup)
<class 'tuple'>
>>> 
>>> empty_tuple = tuple()
<class 'tuple'>
>>>
>>> single_tup = (1)
>>> type(single_tup)
<class 'int'>
>>>
>>> single_tup = (1,)
>>> type(single_tup)
<class 'tuple'>
{% endhighlight %}
可以看到一个小逗号作用这么大！

##总结##
本文主要总结了Python中我们使用的一些数据结构的不同，也罗列了一些有趣的高级的用法。
　　　
　　　
　　　
--------
参考资料：   
http://tech.pro/tutorial/1151/python-data-types-and-their-usage  　　　　 主要译自該篇文章   
http://docs.python.org/3.3/　　　　　   Python 官方文档，最权威最全面的Python学习资料


