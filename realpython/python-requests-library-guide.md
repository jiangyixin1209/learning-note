> 本文为译文，原文链接 [python-requests-library-guide](https://realpython.com/python-requests/) 
> 本人博客: [编程禅师](http://blog.jiangyixin.top)

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

通过访问 `.status_code`，你可以看到服务器返回的状态码：

```shell
>>> response.status_code
200
```

`.status_code` 返回 `200` 意味着你的请求是成功的，并且服务器返回你要请求的数据。

有时，你可能想要在代码中使用这些信息来做判断:

```python
if response.status_code == 200:
    print('Success!')
elif response.status_code == 404:
    print('Not Found.')
```

按照这个逻辑，如果服务器返回 `200` 状态码，你的程序将打印 `Success!`  如果结果是 `404` ，你的程序将打印 `Not Found.` 。

`requests` 更进一步为你简化了此过程。 如果在条件表达式中使用 `Response` 实例，则在状态码介于 `200` 和 `400` 之间时将被计算为为 `True` ，否则为 `False` 。

因此，你可以通过重写 `if` 语句来简化上一个示例:

```python
if response:
    print('Success!')
else:
    print('An error has occurred.')
```

> **技术细节** : 因为 `__ bool __()` 是 `Response` 上的[重载方法](https://realpython.com/operator-function-overloading/#making-your-objects-truthy-or-falsey-using-bool) ，因此[真值测试](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)才成立。
>
> 这意味着重新定义了 `Response` 的默认行为，用来在确定对象的真值时考虑状态码。

请记住，此方法 `不会验证` 状态码是否等于 `200` 。原因是 `200` 到 `400` 范围内的其他状态代码，例如 `204 NO CONTENT` 和 `304 NOT MODIFIED` ，就意义而言也被认为是成功的响应。

例如，`204` 告诉你响应是成功的，但是下消息体中没有返回任何内容。

因此，通常如果你想知道请求是否成功时，请确保使用这方便的简写，然后在必要时根据状态码适当地处理响应。

假设你不想在 `if` 语句中检查响应的状态码。 相反，如果请求不成功，你希望抛出一个异常。 你可以使用 `.raise_for_status()`执行此操作:

```python
import requests
from requests.exceptions import HTTPError

for url in ['https://api.github.com', 'https://api.github.com/invalid']:
    try:
        response = requests.get(url)

        # If the response was successful, no Exception will be raised
        response.raise_for_status()
    except HTTPError as http_err:
        print(f'HTTP error occurred: {http_err}')  # Python 3.6
    except Exception as err:
        print(f'Other error occurred: {err}')  # Python 3.6
    else:
        print('Success!')
```

如果你调用 `.raise_for_status()`，将针对某些状态码引发 `HTTPError` 异常。 如果状态码指示请求成功，则程序将继续进行而不会引发该异常。

> 进一步阅读：如果你不熟悉Python 3.6的 [f-strings](https://realpython.com/python-f-strings/)，我建议你使用它们，因为它们是简化格式化字符串的好方法。

现在，你对于如何处理从服务器返回的响应的状态码了解了许多。 但是，当你发出GET请求时，你很少只关心响应的状态码。 通常，你希望看到更多。 接下来，你将看到如何查看服务器在响应正文中返回的实际数据。

## 响应内容

`GET` 请求的响应通常在消息体中具有一些有价值的信息，称为有效负载。 使用 `Response` 的属性和方法，你可以以各种不同的格式查看有效负载。

要以 [字节](https://realpython.com/python-strings/) 格式查看响应的内容，你可以使用 `.content`：

```shell
>>> response = requests.get('https://api.github.com')
>>> response.content
b'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}","commit_search_url":"https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}","emails_url":"https://api.github.com/user/emails","emojis_url":"https://api.github.com/emojis","events_url":"https://api.github.com/events","feeds_url":"https://api.github.com/feeds","followers_url":"https://api.github.com/user/followers","following_url":"https://api.github.com/user/following{/target}","gists_url":"https://api.github.com/gists{/gist_id}","hub_url":"https://api.github.com/hub","issue_search_url":"https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}","issues_url":"https://api.github.com/issues","keys_url":"https://api.github.com/user/keys","notifications_url":"https://api.github.com/notifications","organization_repositories_url":"https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}","organization_url":"https://api.github.com/orgs/{org}","public_gists_url":"https://api.github.com/gists/public","rate_limit_url":"https://api.github.com/rate_limit","repository_url":"https://api.github.com/repos/{owner}/{repo}","repository_search_url":"https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}","current_user_repositories_url":"https://api.github.com/user/repos{?type,page,per_page,sort}","starred_url":"https://api.github.com/user/starred{/owner}{/repo}","starred_gists_url":"https://api.github.com/gists/starred","team_url":"https://api.github.com/teams","user_url":"https://api.github.com/users/{user}","user_organizations_url":"https://api.github.com/user/orgs","user_repositories_url":"https://api.github.com/users/{user}/repos{?type,page,per_page,sort}","user_search_url":"https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"}'

```

虽然 `.content` 允许你访问响应有效负载的原始字节，但你通常希望使用 [UTF-8](https://en.wikipedia.org/wiki/UTF-8) 等字符编码将它们转换为[字符串](https://realpython.com/python-data-types/)。 当你访问 `.text` 时，`response` 将为你执行此操作：

```shell
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}"...}"}'

```

因为对 `bytes` 解码到 `str` 需要一个编码格式，所以如果你没有指定，请求将尝试根据响应头来猜测编码格式。 你也可以在访问 `.text` 之前通过 `.encoding` 来显式设置编码：

```shell
>>> response.encoding = 'utf-8' # Optional: requests infers this internally
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":"https://github.com/settings/connections/applications{/client_id}","authorizations_url":"https://api.github.com/authorizations","code_search_url":"https://api.github.com/search/code?q={query}{&page,per_page,sort,order}"...}"}'
```

如果你看看响应，你会发现它实际上是序列化的 `JSON` 内容。 要获取字典内容，你可以使用 `.text` 获取 `str` 并使用`json.loads()` 对其进行反序列化。 但是，完成此任务的更简单方法是使用 `.json()`：

```shell
>>> response.json()
{'current_user_url': 'https://api.github.com/user', 'current_user_authorizations_html_url': 'https://github.com/settings/connections/applications{/client_id}'...}'}
```

`.json()` 返回值的类型是字典类型，因此你可以使用键值对的方式访问对象中的值。

你可以使用状态码和消息体做许多事情。 但是，如果你需要更多信息，例如有关 `response` 本身的元数据，则需要查看响应头部。

## 响应头部

响应头部可以为你提供有用的信息，例如响应有效负载的内容类型以及缓存响应的时间限制。 要查看这些头部，请访问 `.headers` ：

```shell
>>> response.headers
{'Server': 'GitHub.com', 'Date': 'Mon, 10 Dec 2018 17:49:54 GMT', 'Content-Type': 'application/json; charset=utf-8',...}
```

`.headers` 返回类似字典的对象，允许你使用键来获取头部中的值。 例如，要查看响应有效负载的内容类型，你可以访问 `Content-Type` ：

```shell
>>> response.headers['Content-Type']
'application/json; charset=utf-8'
```

但是，这个类似于字典的头部对象有一些特别之处。 HTTP规范定义头部不区分大小写，这意味着我们可以访问这些头信息而不必担心它们的大小写：

```shell
>>> response.headers['content-type']
'application/json; charset=utf-8'
```

无论您使用 `'content-type'` 还是 `'Content-Type'`，你都将获得相同的值。

现在，你已经学习了有关 `Response` 的基础知识。 你已经看到了它最有用的属性和方法。 让我们退后一步，看看自定义 `GET` 请求时你的响应如何变化。

***

# 查询字符串参数

自定义 `GET` 请求的一种常用方法是通过URL中的 [查询字符串](https://en.wikipedia.org/wiki/Query_string) 参数传递值。 要使用 `get()` 执行此操作，请将数据传递给 `params` 。 例如，你可以使用GitHub的[Search](https://developer.github.com/v3/search/) API来查找 `requests` 库：

```python
import requests

# Search GitHub's repositories for requests
response = requests.get(
    'https://api.github.com/search/repositories',
    params={'q': 'requests+language:python'},
)

# Inspect some attributes of the `requests` repository
json_response = response.json()
repository = json_response['items'][0]
print(f'Repository name: {repository["name"]}')  # Python 3.6+
print(f'Repository description: {repository["description"]}')  # Python 3.6+
```

通过将字典 `{'q'：'requests + language：python'}` 传递给  `.get()` 的 `params` 参数，你可以修改从Search API返回的结果。

你可以像你刚才那样以字典的形式或以元组列表形式将 `params` 传递给 `get()`:

```shell
>>> requests.get(
...     'https://api.github.com/search/repositories',
...     params=[('q', 'requests+language:python')],
... )
<Response [200]>
```

你甚至可以传 `bytes` 作为值:

```shell
>>> requests.get(
...     'https://api.github.com/search/repositories',
...     params=b'q=requests+language:python',
... )
<Response [200]>
```

查询字符串对于参数化GET请求很有用。 你还可以通过添加或修改发送的请求的头部来自定义你的请求。

***

# 请求头

要自定义请求头，你可以使用 `headers` 参数将HTTP头部组成的字典传递给 `get()`。 例如，你可以通过 `Accept` 中指定文本匹配媒体类型来更改以前的搜索请求，以在结果中突出显示匹配的搜索字词：

```python
import requests

response = requests.get(
    'https://api.github.com/search/repositories',
    params={'q': 'requests+language:python'},
    headers={'Accept': 'application/vnd.github.v3.text-match+json'},
)

# View the new `text-matches` array which provides information
# about your search term within the results
json_response = response.json()
repository = json_response['items'][0]
print(f'Text matches: {repository["text_matches"]}')
```

`Accept` 告诉服务器你的应用程序可以处理哪些内容类型。 由于你希望突出显示匹配的搜索词，所以使用的是 `application / vnd.github.v3.text-match + json`，这是一个专有的GitHub的 `Accept` 标头，其内容为特殊的JSON格式。

在你了解更多自定义请求的方法之前，让我们通过探索其他HTTP方法来拓宽视野。

***

# 其他HTTP方法

除了 `GET` 之外，其他流行的HTTP方法包括 `POST` ，``PUT` ，`DELETE`，`HEAD`，`PATCH`和`OPTIONS`。 `requests` 为每个HTTP方法提供了一个方法，与 `get()` `具有类似的结构:

```python
>>> requests.post('https://httpbin.org/post', data={'key':'value'})
>>> requests.put('https://httpbin.org/put', data={'key':'value'})
>>> requests.delete('https://httpbin.org/delete')
>>> requests.head('https://httpbin.org/get')
>>> requests.patch('https://httpbin.org/patch', data={'key':'value'})
>>> requests.options('https://httpbin.org/get')
```

调用每个函数使用相应的HTTP方法向httpbin服务发出请求。 对于每种方法，你可以像以前一样查看其响应：

```python
>>> response = requests.head('https://httpbin.org/get')
>>> response.headers['Content-Type']
'application/json'

>>> response = requests.delete('https://httpbin.org/delete')
>>> json_response = response.json()
>>> json_response['args']
{}
```

每种方法的响应中都会返回头部，响应正文，状态码等。 接下来，你将进一步了解 `POST`，``PUT` 和 `PATCH` 方法，并了解它们与其他请求类型的区别。

***

# 消息体

根据HTTP规范，`POST`，``PUT`和不太常见的`PATCH`请求通过消息体而不是通过查询字符串参数传递它们的数据。 使用 `requests`，你将有效负载传递给相应函数的 `data` 参数。

`data` 接收字典，元组列表，字节或类文件对象。 你需要将在请求正文中发送的数据调整为与你交互的服务的特定格式。

例如，如果你的请求的内容类型是 `application / x-www-form-urlencoded` ，则可以将表单数据作为字典发送：

```shell
>>> requests.post('https://httpbin.org/post', data={'key':'value'})
<Response [200]>
```

你还可以将相同的数据作为元组列表发送:

```python
>>> requests.post('https://httpbin.org/post', data=[('key', 'value')])
<Response [200]>
```

但是，如果需要发送JSON数据，则可以使用 `json` 参数。 当你通过 `json` 传递JSON数据时，`requests` 将序列化你的数据并为你添加正确的 `Content-Type` 标头。

[httpbin.org](https://httpbin.org/) 是 `requests` 作者 [Kenneth Reitz](https://realpython.com/interview-kenneth-reitz/) 创建的一个很好的资源。 它是一种接收测试请求并响应有关请求数据的服务。 例如，你可以使用它来检查基本的POST请求:

```shell
>>> response = requests.post('https://httpbin.org/post', json={'key':'value'})
>>> json_response = response.json()
>>> json_response['data']
'{"key": "value"}'
>>> json_response['headers']['Content-Type']
'application/json'
```

你可以从响应中看到服务器在你发送请求时收到了请求数据和标头。 `requests` 还以 `PreparedRequest` 的形式向你提供此信息。

***

# 检查你的请求

当你发出请求时，`requests` 库会在将请求实际发送到目标服务器之前准备该请求。 请求准备包括像验证头信息和序列化JSON内容等。

你可以通过访问 `.request` 来查看 `PreparedRequest`:

```shell
>>> response = requests.post('https://httpbin.org/post', json={'key':'value'})
>>> response.request.headers['Content-Type']
'application/json'
>>> response.request.url
'https://httpbin.org/post'
>>> response.request.body
b'{"key": "value"}'
```

通过检查 `PreparedRequest` ，你可以访问有关正在进行的请求的各种信息，例如有效负载，URL，头信息，身份验证等。

到目前为止，你已经发送了许多不同类型的请求，但它们都有一个共同点：它们是对公共API的未经身份验证的请求。 你遇到的许多服务可能都希望你以某种方式进行身份验证。

***

# 身份验证

身份验证可帮助服务了解你的身份。 通常，你通过将数据传递到 `Authorization` 头信息或服务定义的自定义头信息来向服务器提供凭据。 你在此处看到的所有请求函数都提供了一个名为 `auth` 的参数，允许你传递凭据。

需要身份验证的一个示例API的是GitHub的 [Authenticated User](https://developer.github.com/v3/users/#get-the-authenticated-user) API。 此端点提供有关经过身份验证的用户配置文件的信息。 要向 `Authenticated User API` 发出请求，你可以将你的GitHub的用户名和密码以元组传递给 `get()` ：

```shell
>>> from getpass import getpass
>>> requests.get('https://api.github.com/user', auth=('username', getpass()))
<Response [200]>
```

如果你在元组中传递给 `auth` 的凭据有效，则请求成功。 如果你尝试在没有凭据的情况下发出此请求，你将看到状态代码为 `401 Unauthorized` :

```shell
>>> requests.get('https://api.github.com/user')
<Response [401]>
```

当你以元组形式吧用户名和密码传递给 `auth` 参数时，`rqeuests` 将使用HTTP的基本访问认证方案来应用凭据。

因此，你可以通过使用 `HTTPBasicAuth` 传递显式的基本身份验证凭据来发出相同的请求：

```shell
>>> from requests.auth import HTTPBasicAuth
>>> from getpass import getpass
>>> requests.get(
...     'https://api.github.com/user',
...     auth=HTTPBasicAuth('username', getpass())
... )
<Response [200]>
```

虽然你不需要明确进行基本身份验证，但你可能希望使用其他方法进行身份验证。 `requests` 提供了开箱即用的其他身份验证方法，例如 `HTTPDigestAuth` 和 `HTTPProxyAuth` 。

你甚至可以提供自己的身份验证机制。 为此，你必须首先创建AuthBase的子类。 然后，实现`__call __()`：

```python
import requests
from requests.auth import AuthBase

class TokenAuth(AuthBase):
    """Implements a custom authentication scheme."""

    def __init__(self, token):
        self.token = token

    def __call__(self, r):
        """Attach an API token to a custom auth header."""
        r.headers['X-TokenAuth'] = f'{self.token}'  # Python 3.6+
        return r


requests.get('https://httpbin.org/get', auth=TokenAuth('12345abcde-token'))
```

在这里，你自定义的 `TokenAuth` 接收一个令牌，然后在你的请求头中的 `X-TokenAuth` 头中包含该令牌。

错误的身份验证机制可能会导致安全漏洞，因此，除非服务因某种原因需要自定义身份验证机制，否则你始终希望使用像 `Basic` 或 `OAuth` 这样经过验证的身份验证方案。

在考虑安全性时，让我们考虑使用 `requests` 处理SSL证书。

***

# SSL证书验证

每当你尝试发送或接收的数据都很敏感时，安全性就很重要。 通过HTTP与站点安全通信的方式是使用SSL建立加密连接，这意味着验证目标服务器的SSL证书至关重要。

好消息是 `requests` 默认为你执行此操作。 但是，在某些情况下，你可能希望更改此行为。

如果要禁用SSL证书验证，请将 `False` 传递给请求函数的 `verify` 参数:

```shell
>>> requests.get('https://api.github.com', verify=False)
InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)
<Response [200]>
```

当你提出不安全的请求时，`requests` 甚至会发出警告来帮助你保护数据安全。

***

# 性能

在使用 `requests` 时，尤其是在生产应用程序环境中，考虑性能影响非常重要。 超时控制，会话和重试限制等功能可以帮助你保持应用程序平稳运行。

## 超时控制

当你向外部服务发出请求时，系统将需要等待响应才能继续。 如果你的应用程序等待响应的时间太长，则可能会阻塞对你的服务的请求，你的用户体验可能会受到影响，或者你的后台作业可能会挂起。

默认情况下，`requests` 将无限期地等待响应，因此你几乎应始终指定超时时间以防止这些事情发生。 要设置请求的超时，请使用 `timeout` 参数。 `timeout` 可以是一个整数或浮点数，表示在超时之前等待响应的秒数:

```shell
>>> requests.get('https://api.github.com', timeout=1)
<Response [200]>
>>> requests.get('https://api.github.com', timeout=3.05)
<Response [200]>
```

在第一个请求中，请求将在1秒后超时。 在第二个请求中，请求将在3.05秒后超时。

你还可以将元组传递给 `timeout`，第一个元素是连接超时（它允许客户端与服务器建立连接的时间），第二个元素是读取超时（一旦你的客户已建立连接而等待响应的时间）:

```shell
>>> requests.get('https://api.github.com', timeout=(2, 5))
<Response [200]>
```

如果请求在2秒内建立连接并在建立连接的5秒内接收数据，则响应将按原样返回。 如果请求超时，则该函数将抛出 `Timeout` 异常：

```python
import requests
from requests.exceptions import Timeout

try:
    response = requests.get('https://api.github.com', timeout=1)
except Timeout:
    print('The request timed out')
else:
    print('The request did not time out')
```

你的程序可以捕获 `Timeout` 异常并做出相应的响应。

## Session对象

到目前为止，你一直在处理高级请求API，例如 `get()` 和 `post()`。 这些函数是你发出请求时所发生的事情的抽象。 为了你不必担心它们，它们隐藏了实现细节，例如如何管理连接。

在这些抽象之下是一个名为 `Session` 的类。 如果你需要微调对请求的控制方式或提高请求的性能，则可能需要直接使用 `Session` 实例。

`Session` 用于跨请求保留参数。 例如，如果要跨多个请求使用相同的身份验证，则可以使用`session`：

```python
import requests
from getpass import getpass

# By using a context manager, you can ensure the resources used by
# the session will be released after use
with requests.Session() as session:
    session.auth = ('username', getpass())

    # Instead of requests.get(), you'll use session.get()
    response = session.get('https://api.github.com/user')

# You can inspect the response just like you did before
print(response.headers)
print(response.json())
```

每次使用 `session` 发出请求时，一旦使用身份验证凭据初始化，凭据将被保留。

`session` 的主要性能优化以持久连接的形式出现。 当你的应用程序使用 `Session` 建立与服务器的连接时，它会在连接池中保持该连接。 当你的应用程序想要再次连接到同一服务器时，它将重用池中的连接而不是建立新连接。

## 最大重试

请求失败时，你可能希望应用程序重试相同的请求。 但是，默认情况下，`requests` 不会为你执行此操作。 要应用此功能，您需要实现自定义 [Transport Adapter](http://docs.python-requests.org/en/master/user/advanced/#transport-adapters)。

通过 `Transport Adapters `，你可以为每个与之交互的服务定义一组配置。 例如，假设你希望所有对于https://api.github.com的请求在最终抛出 `ConnectionError` 之前重试三次。 你将构建一个 `Transport Adapter`，设置其 `max_retries` 参数，并将其装载到现有的 `Session`：

```python
import requests
from requests.adapters import HTTPAdapter
from requests.exceptions import ConnectionError

github_adapter = HTTPAdapter(max_retries=3)

session = requests.Session()

# Use `github_adapter` for all requests to endpoints that start with this URL
session.mount('https://api.github.com', github_adapter)

try:
    session.get('https://api.github.com')
except ConnectionError as ce:
    print(ce)
```

当您将 `HTTPAdapter(github_adapter)`挂载到 `session` 时，`session`将遵循其对https://api.github.com的每个请求的配置。

`Timeouts`，`Transport Adapters`和 `Sessions` 用于保持代码高效和应用程序的鲁棒性。

***

# 总结

在学习Python中强大的 `requests` 库方面，你已经走了很长的路。

你现在能够：

* 使用各种不同的HTTP方法发出请求，例如GET，POST和PUT
* 通过修改请求头，身份验证，查询字符串和消息体来自定义你的请求
* 检查发送到服务器的数据以及服务器发回给你的数据
* 使用SSL证书验证
* 高效的使用`requests` 通过使用 `max_retries`，`timeout`，`Sessions` 和 `Transport Adapters` 

因为您学会了如何使用 `requests`，所以你可以使用他们提供的迷人数据探索广泛的Web服务世界并构建出色的应用程序了。



[![代码与艺术](https://i.loli.net/2019/02/04/5c57ae3acb5c8.jpg)](https://i.loli.net/2019/02/04/5c57ae3acb5c8.jpg)


  关注公众号 <代码与艺术>，学习更多国外精品技术文章。