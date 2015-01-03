---
layout: post
title: "Hash 函数及其重要性"
description: ""
category: translation 
tags: [python, hash, cryptology]
---
{% include JB/setup %}



不时会爆出网站的服务器和数据库被盗取，考虑到这点，就要确保用户一些敏感数据（例如密码）的安全性。今天，我们要学的是 hash 背后的基础知识，以及如何用它来保护你的 web 应用的密码。

## 申明

密码学是非常复杂的一门学科，我不是这方面的专家，在很多大学和安全机构，在这个领域都有长期的研究。

本文我试图使事情简单化，呈现给大家的是一个 web 应用中安全存储密码的合理方法。

## “Hashing” 做的是什么？
Hashing 将一段数据（无论长还是短）转成相对较短的一段数据，例如一个字符串或者一个整数。

这是通过使用单向哈希函数来完成的。“单向” 意味着逆转它是困难的，或者实际上是不可能的。

加密可以保证信息的安全性，避免被拦截到被破解。Python 的加密支持包括使用 hashlib 的标准算法（例如 MD5 和 SHA），根据信息的内容生成签名，HMAC 用来验证信息在传送过程中没有被篡改。

一个通常使用的 hash 函数的例子是 [**md5()**](http://docs.python.org/3.3/library/hashlib.html#module-hashlib)，这也是当前在很多不同语言和系统中比较流行的：

{% highlight python %}
import hashlib
 
data = "Hello World"
 
h = hashlib.md5()
h.update(data)
print(h.hexdigest())
# b10a8db164e0754105b7a99be72e3fe5
{% endhighlight %}

为了计算一个数据块（这儿是 ASCII 字符串）的 `MD5` 哈希值或摘要，首先需要创建一个 hash 对象，然后添加数据，再调用 `digest()` 或者 `hexdigest()` 函数。本例使用的是 `hexdigest()` 方法来代替 `digest()` ，是因为为了更清晰地输出而对结果进行了格式化。如果你能接受输出二进制的摘要值，那么就用 `digest()`。

## 使用 Hash 函数来存储密码
** 用户注册的过程通常是这样的：**

* 用户填写注册表单，包括密码这一项
* web 脚本将所有的信息存储在数据库中
* 然而，密码在存储之前需要通过 hash 函数进行转化
* 最原始版本的密码并没有保存在任何地方，因此从技术上讲它消失了

** 用户登录的过程：**

* 用户输入用户名和密码
* 脚本用同样的 hash 函数来转化密码
* 脚本找到记录在数据库中的用户信息，读取保存 hash 之后的密码
* 比较两者的值，如果匹配了就完成了登录

注意原始密码不会存储在任何地方！那么如果数据库被盗，那么用户的登录信息不会被盗，是吗？答案是“根据情况来定”。让我们看看一些潜在的问题：

## 问题1：Hash 冲突

当两个不同的输入数据产生相同的 Hash 结果时，这就发生了 Hash 冲突。发生的概率依赖于你所使用的函数。

### 如果利用呢？

作为例子，我使用一些老的脚本，它们使用 [**`crc32()`**](http://docs.python.org/3.3/library/binascii.html#binascii.crc32) 来 Hash 密码。这个函数会产生 32 位整数的结果，这意味着仅仅有 `2^32 (i.e. 4,294,967,296)` 种结果。

让我们来 hash 一个密码：

{% highlight python %}
import binascii
result = binascii.crc32('supersecretpassword')
print(result) #323322056
{% endhighlight %}

现在，我们假设有人盗取了数据库，有了 hash 值。我们也许并不能将 `323322056` 转成 `supersecretpassword`，然而我们能用一个简单的脚本，来找到另一个密码可以转化为相同的 hash 值：

{% highlight python %}
import binascii,base64
 
i = 0
while True:
    if binascii.crc32(base64.encodestring(bytes(i,))) == 323322056:
        print(base64.encodestring(i))
        i += 1
{% endhighlight %}

这可能需要运行好一会，但最终会返回一个字符串。我们可以用返回的字符串来代替 `supersecretpassword`，也同样能登录进入那个用户的账户。

举例来说，在我电脑运行那个脚本一会之后，得到字符串 `MTIxMjY5MTAwNg==`，让我们测试一下：

{% highlight python %}
import binascii
 
print(binascii.crc32("supersecretpassword"))
#323322056
 
print(binascii.crc32("MTIxMjY5MTAwNg=="))
#323322056
{% endhighlight %}

### 如何避免呢？

现在，一个强大的家庭 PC 机就可以用来每秒钟运行一个哈希函数十亿次之多，那么我们需要一个产生非常大范围数的哈希函数。

举例来说，`md5()` 可能就比较适合，因为它产生 128 位哈希值，也就是 340,282,366,920,938,463,463,374,607,431,768,211,456 可能的结果。通过遍历找到冲突不可能的，然而有些人仍然这样做（参考[**这里**](http://www.mscs.dal.ca/~selinger/md5collision/))。

### Sha1

**[Sha1()](http://docs.python.org/2/library/sha.html)** 是一个更好的替代方案，它会产生甚至长达 160 位的 hash 值。


## 问题2：彩虹表

甚至我们解决了冲突的问题，我们还不能确保安全。

彩虹表（rainbow table）是通过计算一些常用的单词和它们的组合的 hash 值而创建的。这些表有多达上百万或上亿项。

举例来说，你可以遍历一个字典，为每个单词产生一个 hash 值。你也可以将它们进行组合，也为组合的单词产生 hash 值。这还没完，你甚至可以以数字插入单词的开始、结尾、中间，将它们也存入表中。

考虑到现在存储系统非常廉价，可以产生和使用上 G 量级的彩虹表。

### 如何利用呢？

让我们想象一下，一个大的数据库被盗，里面有一千万的密码哈希值。在彩虹表中搜索与数据库中密码哈希值的匹配是件相当简单的事，不是所有密码都能找到，但也不是都找不到！它们中的一些肯定可以找到！

### 如何避免呢？

我们尝试添加 “salt” 来解决，下面是个例子：

{% highlight python %}
import hashlib
 
password = "EasyPassword"
 
print(hashlib.sha1(password).hexdigest())
# ff166c2477f864d609ca8111680bfa387eb4e509
 
salt = "f#@V)Hu^%Hgfds"
 
print(hashlib.sha1(salt + password).hexdigest())
# 3e7edaceb96becaf69ae7e73073812ea136188e2
{% endhighlight %}

我们做的很简单，在 hash 密码之前将 “salt” 字符串与用户密码连接，这样很显然 hash 的结果和之前建立的彩虹表没有一个匹配。但是，我们还不够安全！

## 问题3：彩虹表问题（续）

记住，在数据库被盗之后，还可以从 scratch 中重建彩虹表。

### 如何利用呢？

即使使用了 salt，仍然有可能随着数据库被盗而破解。他们所要做的是从 scratch 中产生新的彩虹表，但这次他们会连接 salt 到每个密码上。

举例来说，通常彩虹表中 `easypassword` 可能存在，但在新的彩虹表中，也存在 `f#@V)Hu^%Hgfdseasypassword` 这样的密码。当他们将上千万条盗来的经过 salted 的哈希值与这张新彩虹表比较时，他们也会能找到一些相同的匹配。

### 如何避免呢？

我们使用唯一的 “salt” 替代，每个用户都不一样。

一种备选 salt 是从数据库中取得用户的 ID：

{% highlight python %}
hashlib.sha1(userid + password).hexdigest()
{% endhighlight %}

基于的假设是用户的 ID 号永远不会改变，一般这都是成立的。

我们也可以为每个用户产生一个随机字符串，把它作为这个唯一的 salt。但是我们需要保证要将这个唯一的 salt 保存在用户记录的某个位置。

{% highlight python %}
import hashlib, os
 
def unique_salt():
    return hashlib.sha1(os.urandom(10)).hexdigest()[:22]
 
salt = unique_salt()
password = "" # str or int
hash = hashlib.sha1(salt + str(password)).hexdigest()
print(hash)
# 37dec03d2761122819f8708e6d5c8392ee02b40d
{% endhighlight %}

这种方法有效预防了使用彩虹表破解，因为现在每一个密码都经过不同的值 salted 过，攻击者需要生成一千万个独立的彩虹表，这实际上是不现实的。



## 问题4：Hash 速度

大多 Hash 函数在设计时都注重速度，因为它们常用于计算大数据集和文件的 checksum 值，来检查数据的完整性。

### 如何利用呢？

就像我之前提到的，一个现代 PC 机带有强大的 GPU（或者显卡）可以完成每秒钟上千次的 hash 运算。这种方法，他们可以使用暴力攻击法，尝试每个可能的密码。

你可能认为需要不少于 8 位长度的密码可能避免暴力破解，让我们看下面的分析来决定是否真的能避免：

* 如果密码包含小写字母、大写字母以及数字，也就是有 `62 (26+26+10)` 种可能的字符。
* 一个 8 位长的字符串就有 `62^8` 可能的组合，比 218 万亿略小一点。
* 以每秒十亿的 hash 速率，大约在 60 个小时内就可以破解。

如果是 6 位长的密码，这也相当普遍，只需要在 1 分钟之内就可以破解。

如果要求 9 位或 10 位长度的密码，这样就会让你的用户体验非常不好。

### 如何避免呢？

使用一个低速的 hash 函数。

假设你是用一个 hash 函数，在同样的硬件下，每秒钟只能进行 100 万次 hash 运算，而不是 10 亿次，暴力破解将会花费比以前多出 1000 倍的时间。那么 60 个小时将会变成将近 7 年！

一种你可以实现的方法是：

{% highlight python %}
import hashlib
 
def my_hash(password, salt):
    hash = hashlib.sha1(salt + password).hexdigest()
 
    for i in range(1000):
        hash = hashlib.sha1(hash).hexdigest()
    return hash
 
print(my_hash("12345", "f#@V)Hu^%Hgfds"))
{% endhighlight %}

或者，你可以使用支持 “开销参数”（例如 **[BLOWFISH](https://www.dlitz.net/software/pycrypto/api/current/)**）的算法。在 Python 中，可以利用 **[`py-crypt`](https://bitbucket.org/alexandrul/py-bcrypt/downloads)** 库。

{% highlight python %}
import bcrypt
 
def my_hash(password):
    return bcrypt.hashpw(password, bcrypt.gensalt(10))
 
print(my_hash("atdk"))
#$2a$10$WNhGOdVhoZrrKgwxGa2VIuzfAvm9oFWZF9PIVtLIoU5LQOVGLuLrq
{% endhighlight %}

** 注意输出：**

1. 第一个值是 `$2a`，表明我们使用的是 BLOWFISH 算法。
2. 这种情形下第二个值是 `$10`，是“开销参数”。是执行迭代的次数以 2 为对数的结果，它将会迭代 `(10 => 2^10 = 1024)` 次。这个数值可以在 4 到 31 范围内变化。

让我们运行例子：

{% highlight python %}
import bcrypt, os, hashlib
 
def my_hash(password, unique_salt):
    return bcrypt.hashpw(password, bcrypt.gensalt(10) + unique_salt)
 
def unique_salt():
    return hashlib.sha1(os.urandom(10)).hexdigest()[:22]
 
password = "verysecret"
 
print(my_hash(password, unique_salt()))
# $2a$10$aHx0q.FE/tGvGWzlm6yePemYx9SAsBP2iSiy/uFx7pyjpy980Hita
{% endhighlight %}

结果包括算法（$2a），开销参数（$10），使用的 22 位 salt，剩下的是计算的 hash 值。让我们测试一下：

{% highlight python %}
import bcrypt, os, hashlib
 
# assume this was pulled from the database
hash = "$2a$10$6XDaX/3kNby0jI9Ih/Re7.478DOMZK9OnA2mTxKUP0My.39N.jdky"
 
# assume this is the password the user entered to log back in
password = "verysecret"
 
def check_password(hash, password):
    salt = hash[:29]
    new_hash = bcrypt.hashpw(password, salt)
    return hash == new_hash
 
if check_password(hash, password):
    print("Access Granted")
else:
    print("Access Denied")
{% endhighlight %}

当我们运行时，我们看到输出 "Access Granted!"。

## 整合所有的问题

如果考虑到上面的所有问题，根据我们目前所学的，写一个实用类：

{% highlight python %}
import bcrypt, os, hashlib
 
class PassHash():
    def unique_salt(self):
        return hashlib.sha1(os.urandom(10)).hexdigest()[:22]
 
    def hash(self, password):
        return bcrypt.hashpw(password, bcrypt.gensalt(10) + self.unique_salt())
 
    def check_password(self, hash, password):
        full_salt = hash[:29]
        new_hash = bcrypt.hashpw(password, full_salt)
        return hash == new_hash
 
obj = PassHash()
 
a = obj.hash("12345")
print(a) # $2a$10$gBSbmXKanQJOTSabtX4wfOE2RT2mKDFbCY6r7cqCJSk2YPGjIDrou
 
b = obj.check_password(a, "12345")
print(b) # True
{% endhighlight %}

现在，我们可以在我们的表单中使用该类来 hash 我们密码，确保安全性。

## 结论

这种 hash 密码的方法对大多 web 应用已经足够了。别忘记你还可以要求你的用户使用更强的密码，通过强制最小密码长度，组合字符、数字和特殊字符等方法。



----

编译自：[http://pypix.com/python/hash-functions/](http://pypix.com/python/hash-functions/)
