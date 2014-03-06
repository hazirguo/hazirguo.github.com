---
layout: post
title: "【译】Web 框架是什么？"
description: ""
category: web 
tags: [web, framework, django]
---
{% include JB/setup %}



Web 应用框架，或者简单的说是“Web 框架”，其实是建立 web 应用的一种方式。从简单的博客系统到复杂的富 AJAX 应用，web 上每个页面都是通过写代码来生成的。我发现很多人都热衷于学习 web 框架技术，例如 Flask 或这 Django 之类的，但是很多人并不理解什么是 web 框架，或者它们是如何工作的。这篇文章中，我将探索反复被忽略的 web 框架基础的话题。阅读完这篇文章，你应该首先对什么是 web 框架以及它们为什么会存在有更深的认识。这会让你学习一个新的 web 框架变得简单的多，还会让你在使用不同的框架的时候做个明知的选择。


# Web 如何工作的？
在我们讨论框架之前，我们需要理解 Web 如何“工作”的。为此，我们将深入挖掘你在浏览器里输入一个 URL 按下 Enter 之后都发生了什么。在你的浏览器中打开一个新的标签，浏览 `http://www.jeffknupp.com` 。我们讨论为了显示这个页面，浏览器都做了什么事情（不关心 DNS 查询）。

## Web 服务器

每个页面都以 HTML 的形式传送到你的浏览器中，HTML 是一种浏览器用来描述页面内容和结构的语言。那些负责发送 HTML 到浏览器的应用称之为“Web 服务器”，会让你迷惑的是，这些应用运行的机器通常也叫做 web 服务器。

然而，最重要的是要理解，到最后所有的 web 应用要做的事情就是发送 HTML 到浏览器。不管应用的逻辑多么复杂，最终的结果总是将 HTML 发送到浏览器（我故意将应用可以响应像 `JSON` 或者 `CSS` 等不同类型的数据忽略掉，因为在概念上是相同的）。

web 应用如何知道发送什么到浏览器呢？它发送浏览器请求的任何东西。

### HTTP
浏览器从 web 服务器（或者叫应用服务器）上使用 HTTP 协议下载网站，HTTP 协议是基于一种 请求-响应（**`request-response`**）模型的。客户端（你的浏览器）从运行在物理机器上的 web 应用请求数据，web 应用反过来对你的浏览器请求进行响应。

重要的一点是，要记住通信总是由客户端（你的浏览器）发起的，服务器（也就是 web 服务器）没有办法创建一个链接，发送没有经过请求的数据给你的浏览器。如果你从 web 服务器上接收到数据，一定是因为你的浏览器显示地发送了请求。

#### HTTP Methods
在 HTTP 协议中，每条报文都关联方法（method 或者 verb），不同的 HTTP 方法对应客户端可以发送的逻辑上不同类型的请求，反过来也代表了客户端的不同意图。例如，请求一个 web 页面的 HTML，与提交一个表单在逻辑上是不同的，所以这两种行为就需要使用不同的方法。

##### HTTP GET
GET 方法就像其听起来的那样，从 web 服务器上 get（请求）数据。GET 请求是到目前位置最常见的一种 HTTP 请求，在一次 GET 请求过程中，web 应用对请求页面的 HTML 进行响应之外，就不需要做任何事情了。特别的，web 应用在 GET 请求的结果中，不应该改变应用的状态（比如，不能基于 GET 请求创建一个新帐号）。正是因为这个原因，GET 请求通常认为是“安全”的，因为他们不会导致应用的改变。

##### HTTP POST
显然，除了简单的查看页面之外，应该还有更多与网站进行交互的操作。我们也能够向应用发送数据，例如通过表单。为了达到这样的目的，就需要一种不同类型的请求方法：**POST**。POST 请求通常携带由用户输入的数据，web 应用收到之后会产生一些行为。通过在表单里输入你的信息登录一个网站，就是 POST 表单的数据给 web 应用的。

不同于 GET 请求，POST 请求通常会导致应用状态的改变。在我们的例子中，当表单 POST 之后，一个新的账户被创建。不同于 GET 请求，POST 请求不总是生成一个新的 HTML 页面发送到客户端，而是客户端使用响应的响应码（`response code`）来决定对应用的操作是否成功。

#### HTTTP Response Codes
通常来说，web 服务器返回 200 的响应码，意思是，“我已经完成了你要求我做的事情，一切都正常”。响应码总是一个三位数字的代号，web 应用在每个响应的同时都发送一个这样的代号，表明给定的请求的结果。响应码 200 字面意思是“OK”，是响应一个 GET 请求大多情况下都使用的代号。然而对于 POST 请求， 可能会有 204（“No Content”）发送回来，意思是“一切都正常，但是我不准备向你显示任何东西”。

POST 请求仍然会发送一个特殊的 URL，这个 URL 可能和提交数据的页面不同，意识这一点是至关重要的。还是以我们的登录为例，表单可能是在 `www.foo.com/signup` 页面，然而点击 `submit`，可能会导致带有表单数据的 POST 请求发送到 `www.foo.com/process_sigup` 上。POST 请求要发送的位置在表单的 HTML 中有特别标明。


# Web 应用

你可以仅仅使用 HTTP GET 和 POST 做很多事情。一个应用程序负责去接收一个 HTTP 请求，同时给以 HTTP 响应，通常包含了请求页面的 HTML。POST 请求会引起 web 应用做出一些行为，可能是往数据库中添加一条记录这样的。还有很多其它的 HTTP 方法，但是我们目前只关注 GET 和 POST。

那么最简单的 web 应用是什么样的呢？我们可以写一个应用，让它一直监听 80 端口（著名的 HTTP 端口，几乎所有 HTTP 都发送到这个端口上）。一旦它接收到等待的客户端发送的请求连接，然后它就会回复一些简单的 HTML。

下面是程序的代码：

{% highlight python %} 
import socket
HOST = ''
PORT = 80
listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
listen_socket.bind((HOST, PORT))
listen_socket.listen(1)
connection, address = listen_socket.accept()
request = connection.recv(1024)
connection.sendall("""HTTP/1.1 200 OK
Content-type: text/html
<html>
    <body>
        <h1>Hello, World!</h1>
    </body>
</html>""")
connection.close()
{% endhighlight %} 

（如果上面的代码不工作，试着将 PORT 改为类似 8080 这样的端口。）

这个代码接收简单的链接和简单的请求，不管请求的 URL 是什么，它都会响应 HTTP 200（所以，这不是一个真正意义上的 web 服务器）。`Content-type:text/html` 行代码的是 header 字段，header 用来提供请求或者响应的元信息。这样，我们就告诉了客户端接下来的数据是 HTML。

## Anatomy of a Request  请求的剖析
如果看一下测试上面程序的 HTTP 请求，你会发现它和 HTTP 响应非常类似。第一行 `<HTTP Method> <URL> <HTTP version>`，在这个例子中是 `GET / HTTP/1.0`。第一行之后就是一些类似 `Accept: */*` 这样的头（意思是我们希望在响应中接收任何内容）。

我们响应和请求有着类似的第一行，格式是 `<HTTP version> <HTTP Status-code> <Status-code Reason Phrase>`，在外面的例子中是 `HTTP/1.1 200 OK` 。接下来是头部，与请求的头部有着相同的格式。最后是响应的实际包含的内容。注意，这会被解释为一个字符串或者二进制文件， **`Content-type`** 头告诉客户端怎样去解释响应。


## Web 服务器之殇

如果我们继续以上面的例子为基础建立 web 应用，我们还需要解决很多问题：

1. 我们怎样检测请求的 URL 以及返回正确的页面？
2. 除了简单的 GET 请求之外我们如何处理 POST 请求？
3. 我们如何理解更高级的概念，如 session 和 cookie？
4. 我们如何扩展程序以使其处理上千个并发连接？

就像你想的那样，没有人愿意每次建立一个 web 应用都要解决这些问题。正是这个原因，就有处理 HTTP 协议本身和有效解决上面问题的办法的包存在。然而，记住了，它们的核心功能和我们的例子是相同的：监听请求，带有一些 HTML 发回 HTTP 响应。


# 解决两大问题：路由和模板

围绕建立 web 应用的所有问题中，两个问题尤其突出：

1. 我们如何将请求的 URL 映射到处理它的代码上？
2. 我们怎样动态地构造请求的 HTML 返回给客户端，HTML 中带有计算得到的值或者从数据库中取出来的信息？

每个 web 框架都以某种方法来解决这些问题，也有很多不同的解决方案。用例子来说明更容易理解，所以我将针对这些问题讨论 Django 和 Flask 的解决方案。但是，首先我们还需要简单讨论一下 MVC 。

## Django 中的 MVC

Django 充分利用 MVC 设计模式。 MVC，也就是 **M**odel-**V**iew-**C**ontroller （模型-视图-控制器），是一种将应用的不同功能从逻辑上划分开。models 代表的是类似数据库表的资源（与 Python 中用 **`class`** 来对真实世界目标建模使用的方法大体相同）。controls 包括应用的业务逻辑，对 models 进行操作。为了动态生成代表页面的 HTML，需要 views 给出所有要动态生成页面的 HTML 的信息。

在 Django 中有点让人困惑的是，controllers 被称做 views，而 views 被称为 templates。除了名字上的有点奇怪，Django 很好地实现了 MVC 的体系架构。

## Django 中的路由

路由是处理请求 URL 到负责生成相关的 HTML 的代码之间映射的过程。在简单的情形下，所有的请求都是有相同的代码来处理（就像我们之前的例子那样）。变得稍微复杂一点，每个 URL 对应一个 view function 。举例来说，如果请求 `www.foo.com/bar` 这样的 URL，调用 `handler_bar()` 这样的函数来产生响应。我们可以建立这样的映射表，枚举出我们应用支持的所有 URL 与它们相关的函数。

然而，当 URL 中包含有用的数据，例如资源的 ID（像这样 `www.foo.com/users/3/`） ，那么这种方法将变得非常臃肿。我们如何将 URL 映射到一个 view 函数，同时如何利用我们想显示 ID 为 3 的用户？

Django 的答案是，将 URL 正则表达式映射到可以带参数的 view 函数。例如，我假设匹配 `^/users/(?P<id>\d+)/$` 的 URL 调用 `display_user(id)` 这样的函数，这儿参数 id 是正则表达式中匹配的 id。这种方法，任何 `/users/<some_number>/` 这样的 URL 都会映射到 `display_user` 函数。这些正则表达式可以非常复杂，包含关键字和参数。

## Flask 中的路由

Flask 采取了一点不同的方法。将一个函数和请求的 URL 关联起来的标准方法是通过使用 `route()` 装饰器。下面是 Flask 代码，在功能上和上面正则表达式方法相同：

{% highlight python %}
@app.route('/users/<id:int>/')
def display_user(id):
    # ...
{% endhighlight %} 

就像你看到的这样，装饰器使用几乎最简单的正则表达式的形式来将 URL 映射到参数。通过传递给 `route()` 的 URL 中包含的 `<name:type>` 指令，可以提取到参数。路由像 `/info/about_us.html` 这样的静态 URL，可以像你预想的这样 `@app.route('/info/about_us.html')` 处理。

## 通过 Templates 产生 HTML

继续上面的例子，一旦我们有合适的代码映射到正确的 URL，我们如何动态生成 HTML？对于 Django 和 Flask，答案都是通过 **HTML Templating**。

HTML Templating 和使用 `str.format()` 类似：需要动态输出值的地方使用占位符填充，这些占位符后来通过 `str.format()` 函数用参数替换掉。想象一下，整个 web 页面就是一个字符串，用括号标明动态数据的位置，最后再调用 `str.format()` 。Django 模板和 Flask 使用的模板引擎 Jinja2 都使用的是这种方法。

然而，不是所有的模板引擎都能相同的功能。Django 支持在模板里基本的编程，而 Jinja2 只能让你执行特定的代码（不是真正意义上的代码，但也差不多）。Jinja2 可以缓存渲染之后的模板，让接下来具有相同参数的请求可以直接从缓存中返回结果，而不是用再次花大力气渲染。


## 数据库交互

Django 有着“功能齐全”的设计哲学，其中包含了一个 ORM(**O**bject **R**ealational **M**apper，对象关系映射)，ORM 的目的有两方面：一是将 Python 的 class 与数据库表建立映射，而是剥离出不同数据库引擎直接的差异。没人喜欢 ORM，因为在不同的域之间映射永远不完美，然而这还在承受范围之内。Django 是功能齐全的，而 Flask 是一个微框架，不包括 ORM，尽管它对 SQLAlchemy 兼容性非常好，SQLAlchemy 是 Django ORM 的最大也是唯一的竞争对手。

内嵌 ORM 让 Django 有能力创建一个功能丰富的 CRUD 应用，从服务器端角度来看，**CRUD**（**C**reate **R**ead **U**pdate **D**elete）应用非常适合使用 web 框架技术。Django 和 Flask-SQLchemy 可以直接对每个 model 进行不同的 CRUD 操作。


# 再谈 web 框架

到现在为止，web 框架的目的应该非常清晰了：向程序员隐藏了处理 HTTP 请求和响应相关的基础代码。至于隐藏多少这取决于不同的框架，Django 和 Flask 走向了两个极端：Django 包括了每种情形，几乎成了它致命的一点；Flask 立足于“微框架”，仅仅实现 web 应用需要的最小功能，其它的不常用的 web 框架任务交由第三方库来完成。

但是最后要记住的是，Python web 框架都以相同的方式工作的：它们接收 HTTP 请求，分派代码，产生 HTML，创建带有内容的 HTTP 响应。事实上，所有主流的服务器端框架都以这种方式工作的（ JavaScript 框架除外）。但愿了解了这些框架的目的，你能够在不同的框架之间选择适合你应用的框架进行开发。


文章来源于：[http://www.jeffknupp.com/blog/2014/03/03/what-is-a-web-framework/](http://www.jeffknupp.com/blog/2014/03/03/what-is-a-web-framework/)











































