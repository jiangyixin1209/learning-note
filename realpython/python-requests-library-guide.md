> 本文为译文，原文链接 [python-requests-library-guide](https://realpython.com/python-requests/) 

[requests](http://docs.python-requests.org/en/master/) 库是用来在Python中发出标准的HTTP请求。 它将请求背后的复杂性抽象成一个漂亮，简单的API，以便你可以专注于与服务交互和在应用程序中使用数据。

在本文中，你将看到 `requests` 提供的一些有用的功能，以及如何针对你可能遇到的不同情况来自定义和优化这些功能。 你还将学习如何有效的使用 `requests`，以及如何防止对外部服务的请求导致减慢应用程序的速度。

在本教程中，你将学习如何:

* 使用常见的HTTP方法发送请求
* 定制你的请求头和数据，使用查询字符串和消息体
* 检查你的请求和响应的数据
* 发送带身份验证的请求
* 配置你的请求来避免阻塞或减慢你的应用程序

虽然我试图包含尽可能多的信息来理解本文中包含的功能和示例，但阅读此文需要对HTTP有基础的了解。

现在让我们深入了解如何在你的应用程序中使用请求！

# 开始使用 *requests* 

让我们首先安装 `requests` 库。 为此，请运行以下命令:

```shell
pip install requests
```

如果你喜欢使用 [Pipenv](https://realpython.com/pipenv-guide/) 管理Python包，你可以运行下面的命令:

```shell
pipenv install requests
```

一旦安装了 `requests` ，你就可以在应用程序中使用它。像这样导入 `requests` :

```shell
import requests
```

现在你已经都准备完成了，那么是时候开始使用 `requests` 的旅程了。 你的第一个目标是学习如何发出GET请求。

***

# GET 请求

[HTTP方法](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)（如GET和POST）决定当发出HTTP请求时尝试执行的操作。 除了GET和POST之外，还有其他一些常用的方法，你将在本教程的后面部分使用到。

最常见的HTTP方法之一是GET。 GET方法表示你正在尝试从指定资源获取或检索数据。 要发送GET请求，请调用 `requests.get()` 。

你可以通过下面方式来向GitHub的 [Root REST API](https://developer.github.com/v3/#root-endpoint) 发出GET请求：

```shell
>>> requests.get('https://api.github.com')
<Response [200]>
```

恭喜！ 你发出了你的第一个请求。 接下来让我们更深入地了解该请求的响应。

***

# 响应

`Response` 是检查请求结果的强有力的对象。 让我们再次发出相同的请求，但这次将返回值存储在一个变量中，以便你可以仔细查看其属性和方法：

```shell
>>> response = requests.get('https://api.github.com')
```

在此示例中，你捕获了 `get()` 的返回值，该值是 `Response` 的实例，并将其存储在名为 `response` 的变量中。 你现在可以使用 `response` 来查看有关GET请求结果的全部信息。

## 状态码

您可以从 `Response` 获取的第一部分信息是状态码。 状态码会展示你请求的状态。

例如，`200 OK` 状态表示你的请求成功，而 `404 NOT FOUND` 状态表示找不到你要查找的资源。 还有许多[其它的状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) ，可以为你提供关于你的请求所发生的具体情况。

通过访问 ``.status_code`，你可以看到服务器返回的状态码：

```shell
>>> response.status_code
200
```



***

# 查询字符串参数

***

# 请求头

***

# 其他HTTP方法

***

# 消息体

***

# 检查你的请求

***

# 授权

***

# SSL证书验证

***

# 性能

## 超时

## Session对象

## 最大重试

***

# 总结

