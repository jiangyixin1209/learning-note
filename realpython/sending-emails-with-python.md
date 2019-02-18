> 本文为译文，原文链接 [Sending Emails With Python](https://realpython.com/python-send-email)   
> 本人博客: [编程禅师](http://blog.jiangyixin.top)

你可能因为想使用Python发送电子邮件而找到了本教程。 也许你希望写代码来接收邮件提醒，在用户创建帐户时向用户发送确认邮件，或向组织成员发送邮件以提醒他们支付会费。 发送邮件是一项耗时且容易出错的任务，但是使用Python可以轻松实现自动化。

在本教程中,你将了解如何:

* 使用 `SMTP_SSL()` 和 `.starttls()` 设置安全连接

* 使用Python的内置 `smtplib` 库发送基本电子邮件

* 使用 `email` 包发送包含HTML内容和附件的电子邮件

* 发送多份包含联系人数据的CSV文件的个性化电子邮件

* 使用 `Yagmail` 包只需几行代码即可通过Gmail帐户发送电子邮件

你将在本教程结束时找到一些事务性的电子邮件服务，当你想要发送大量电子邮件时，这些服务会很有用。

# 准备开始

Python内置了 [smtplib](https://docs.python.org/3/library/smtplib.html) 模块，用于使用简单邮件传输协议（SMTP）发送电子邮件。 `smtplib` 在SMTP中使用 [RFC 821](https://tools.ietf.org/html/rfc821)协议。 本教程中的示例将使用Gmail SMTP服务器发送电子邮件，但相同的原则适用于其他电子邮件服务。 虽然大多数电子邮件提供商使用与本教程中相同的连接端口，但你可以使用Google搜索来快速确认。

要开始本教程，请先建立用于开发的Gmail帐户，或者建立一个SMTP调试服务器，邮件将不会发送而是将其打印到控制台中。 下面列出了这两个选项。 本地SMTP调试服务器可用于修复电子邮件功能的任何问题，并确保在发送任何电子邮件之前你的电子邮件功能没有错误。

## 选项一: 建立用于开发的Gmail账户

如果你决定使用Gmail帐户发送电子邮件，我强烈建议你为开发代码设置一次性帐户。 这是因为你必须调整Gmail帐户的安全设置来允许从你的Python代码访问，而且你也可能会意外地泄露你的登录详细信息。 此外，我发现我的测试帐户的收件箱很快就填满了测试电子邮件，这也足以让你设置一个新的Gmail帐户进行开发。

Gmail的一个不错的功能是，你可以使用 `+` 号在@符号前面的电子邮件地址中添加任何修饰符。 例如，发送到 `my+person1@gmail.com` 和 `my+person2@gmail.com` 的邮件都将发送到 `my@gmail.com` 。 在测试电子邮件功能时，你可以使用此功能模拟所有指向同一收件箱的多个地址。

要创建用于测试代码的Gmail地址，请执行以下操作：

* [创建一个新的Google帐户](https://accounts.google.com/signup)
* 将[允许不够安全的应用设置](https://myaccount.google.com/lesssecureapps)为启用。请注意，这样可以让其他人更轻松地访问你的帐户。

如果你不想降低Gmail帐户的安全设置，请查看Google的[文档](https://developers.google.com/gmail/api/quickstart/python)，了解如何使用Python脚本进行OAuth2授权来获取访问凭据。

## 选项二:建立一个本地SMTP服务器

你可以使用Python预先安装的 `smptd` 模块运行一个本地SMTP调试服务器来测试电子邮件功能。 它不会将电子邮件发送到指定的地址，而是丢弃它们并将其内容打印到控制台。 运行本地调试服务器意味着无需处理消息加密或使用凭据登录电子邮件服务器。

你可以通过在命令提示符下输入以下内容来启动本地SMTP调试服务器：

```shell
python -m smtpd -c DebuggingServer -n localhost:1025
```

在Linux上，在相同的命令之前加上 `sudo`。

通过此服务器发送的任何电子邮件都将被丢弃，并在终端窗口中每行显示一个 `bytes` 对象：

```shell
---------- MESSAGE FOLLOWS ----------
b'X-Peer: ::1'
b''
b'From: my@address.com'
b'To: your@address.com'
b'Subject: a local test mail'
b''
b'Hello there, here is a test email'
------------ END MESSAGE ------------
```

对于本教程的其余部分，我假设你使用的是Gmail帐户，但如果你使用的是本地调试服务器，请确保使用 `localhost` 作为SMTP服务器地址并使用端口为1025而不是端口465或587。 除此之外，你不需要使用 `login()` 或使用 `SSL / TLS` 加密通信。

***

# 发送纯文本邮件

在我们深入发送包含HTML内容和附件的电子邮件之前，你将学会使用Python发送纯文本电子邮件。 这些是你可以在简单的文本编辑器中编写的电子邮件。 没有像文本格式或超链接这样的奇特东西。 这些稍后你会了解到。

## 开始一个安全SMTP连接

当你通过Python发送电子邮件时，应确保你的SMTP连接已加密，以便其他人无法轻松访问你的邮件和登录凭据。 SSL（安全套接字层）和TLS（传输层安全性）是两种可用于加密SMTP连接的协议。 使用本地调试服务器时，不必使用其中任何一个。

有两种方法可以与你的电子邮件服务器建立安全连接：

* 使用 `SMTP_SSL()` 建立安全的SMTP连接
* 使用不安全的SMTP连接，然后使用 `.starttls()` 加密

在这两种情况下，Gmail都会使用TLS加密电子邮件，因为这是SSL更安全的后续版本。 根据Python的[安全注意事项](https://docs.python.org/3/library/ssl.html#ssl-security)，强烈建议你使用 [ssl](https://docs.python.org/3/library/ssl.html) 模块中的 `create_default_context()`。 这将加载系统的可信CA证书，启用主机名检查和证书验证，并尝试选择适合的安全的协议和密码设置。

如果要检查Gmail收件箱中电子邮件的加密信息，请转到更多→显示原始内容来查看“已接收”标题下列出的加密类型。

[smtplib](https://docs.python.org/3/library/smtplib.html) 是Python的内置模块，用于使用SMTP或ESMTP监听器守护程序向任何Internet计算机发送电子邮件。

我将首先向你展示如何使用 `SMTP_SSL()`，因为它从一开始就实例化一个安全的连接，并且比使用 `.starttls()` 替代方案略微简洁。 请注意，如果使用 `SMTP_SSL()` ，则Gmail要求你连接到465端口，使用 `.starttls()` 时，要求连接到589端口。

### 选项1: 使用 **SMPT_SSL()** 

下面的代码示例使用 `smtplib` 的 `SMTP_SSL()` 初始化TLS加密连接，以此来与Gmail的SMTP服务器建立安全连接。 `ssl` 的默认上下文验证主机名及其证书，并优化连接的安全性。 请务必填写你自己的电子邮件地址而不是 `my@gmail.com`：

```python
import smtplib, ssl

port = 465  # For SSL
password = input("Type your password and press enter: ")

# Create a secure SSL context
context = ssl.create_default_context()

with smtplib.SMTP_SSL("smtp.gmail.com", port, context=context) as server:
    server.login("my@gmail.com", password)
    # TODO: Send email here
```


使用 `with smtplib.SMTP_SSL() as server` 确保连接在缩进代码块的末尾自动关闭。 如果port为零或未指定，则 `.SMTP_SSL()` 将使用标准端口（端口465）。

将电子邮件密码存储在代码中并不安全，特别是如果你打算与他人共享。 相反，`使用input()` 让用户在运行脚本时输入密码，如上例所示。 如果你在输入时不希望密码显示在屏幕上，则可以导入 [getpass](https://docs.python.org/3/library/getpass.html) 模块并使用 `.getpass()` 代替直接输入密码。

### 选项2: 使用 **.starttls()** 

我们可以创建一个不安全的SMTP连接并使用 `.starttls()` 加密它，而不是使用 `.SMTP_SSL()` 一开始就创建是安全的连接。

为此，创建一个 `smtplib.SMTP` 实例，该实例封装SMTP连接并允许你访问其方法。 我建议你在脚本开头定义SMTP服务器和端口，以便轻松配置它们。

下面的代码片段使用 `server= SMTP()` ，而不是使用 `with SMTP() as server:` 这个我们在上一个示例中使用了的格式。 为了确保在出现问题时代码不会崩溃，请将主代码放在 `try` 块中，让 `except` 块将任何错误消息打印到 `stdout` ：

```python
import smtplib, ssl

smtp_server = "smtp.gmail.com"
port = 587  # For starttls
sender_email = "my@gmail.com"
password = input("Type your password and press enter: ")

# Create a secure SSL context
context = ssl.create_default_context()

# Try to log in to server and send email
try:
    server = smtplib.SMTP(smtp_server,port)
    server.ehlo() # Can be omitted
    server.starttls(context=context) # Secure the connection
    server.ehlo() # Can be omitted
    server.login(sender_email, password)
    # TODO: Send email here
except Exception as e:
    # Print any error messages to stdout
    print(e)
finally:
    server.quit() 
```

要通知服务器让它知道自己，应在创建 `.SMTP()` 对象后调用 `.helo()` （SMTP）或 `.ehlo()` （ESMTP），并在 `.starttls()` 后再调用。 此函数由 `.starttls()` 和 `.sendmail()` 隐式调用，因此除非你要检查服务器的SMTP服务扩展，否则不必显式调用 `.helo()` 或 `.ehlo()` 。

## 发送你的纯文本邮件

使用上述任一方法发起一个安全SMTP连接后，你可以使用 `.sendmail()` 发送电子邮件：

```python
server.sendmail(sender_mail, receiver_email, message)
```

我建议在 `import` 后在脚本顶部定义电子邮件地址和邮件内容，以便你可以方便更改它们：

```python
sender_email = "my@gmail.com"
receiver_email = "your@gmail.com"
message = """\
Subject: Hi there

This message is sent from Python."""

# Send email here
```


`message` 字符串以 `“Subject：Hi there”` 开头，后跟两个换行符（\ n）。 这样可以确保 `Hi there` 为电子邮件的主题显示，并且新的一个行后面的文本将被视为邮件正文。

下面的代码示例使用 `SMTP_SSL()` 发送纯文本电子邮件：

```python
import smtplib, ssl

port = 465  # For SSL
smtp_server = "smtp.gmail.com"
sender_email = "my@gmail.com"  # Enter your address
receiver_email = "your@gmail.com"  # Enter receiver address
password = input("Type your password and press enter: ")
message = """\
Subject: Hi there

This message is sent from Python."""

context = ssl.create_default_context()
with smtplib.SMTP_SSL(smtp_server, port, context=context) as server:
    server.login(sender_email, password)
    server.sendmail(sender_email, receiver_email, message)
```

这是通过使用 `.starttls()` 加密SMTP连接发送纯文本电子邮件的一个代码示例。作为对照， `server.ehlo()` 行可以省略，因为如果需要，它们将由 `.starttls()` 和 `.sendmail()` 隐式调用：

```python
import smtplib, ssl

port = 587  # For starttls
smtp_server = "smtp.gmail.com"
sender_email = "my@gmail.com"
receiver_email = "your@gmail.com"
password = input("Type your password and press enter:")
message = """\
Subject: Hi there

This message is sent from Python."""

context = ssl.create_default_context()
with smtplib.SMTP(smtp_server, port) as server:
    server.ehlo()  # Can be omitted
    server.starttls(context=context)
    server.ehlo()  # Can be omitted
    server.login(sender_email, password)
    server.sendmail(sender_email, receiver_email, message)
```

***

# 发送花哨的电子邮件

Python的内置 `email` 包允许你构建更多花哨的电子邮件，然后可以使用 `smptlib` 进行传输。 下面，你将了解如何使用 email` 包发送包含HTML内容和附件的电子邮件。

## 包含HTML内容

如果你要格式化电子邮件中的文本（粗体，斜体等），或者如果要添加任何图像，超链接或响应式的内容，则使用HTML非常方便。 今天最常见的电子邮件类型是MIME（多用途因特网邮件扩展）多部分的电子邮件，它结合了HTML和纯文本。 MIME消息由Python的 `email.mime` 模块处理。 有关详细说明，请查看[文档](https://docs.python.org/3/library/email.mime.html) 。

由于并非所有电子邮件客户端都默认显示HTML内容，并且出于安全原因，有些人仅选择接收纯文本电子邮件，因此为HTML消息添加纯文本的替代非常重要。 由于电子邮件客户端将首先渲染最后一部分的附件，因此请确保在纯文本版本之后添加HTML消息。

在下面的示例中，我们的 `MIMEText()` 对象将包含消息的HTML和纯文本版本，`MIMEMultipart（"alternative"）` 实例将这些组合成一个带有两个备用渲染选项的消息：

```python
import smtplib, ssl
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

sender_email = "my@gmail.com"
receiver_email = "your@gmail.com"
password = input("Type your password and press enter:")

message = MIMEMultipart("alternative")
message["Subject"] = "multipart test"
message["From"] = sender_email
message["To"] = receiver_email

# Create the plain-text and HTML version of your message
text = """\
Hi,
How are you?
Real Python has many great tutorials:
www.realpython.com"""
html = """\
<html>
  <body>
    <p>Hi,<br>
       How are you?<br>
       <a href="http://www.realpython.com">Real Python</a> 
       has many great tutorials.
    </p>
  </body>
</html>
"""

# Turn these into plain/html MIMEText objects
part1 = MIMEText(text, "plain")
part2 = MIMEText(html, "html")

# Add HTML/plain-text parts to MIMEMultipart message
# The email client will try to render the last part first
message.attach(part1)
message.attach(part2)

# Create secure connection with server and send email
context = ssl.create_default_context()
with smtplib.SMTP_SSL("smtp.gmail.com", 465, context=context) as server:
    server.login(sender_email, password)
    server.sendmail(
        sender_email, receiver_email, message.as_string()
    )
```

在此示例中，首先将纯文本和HTML消息定义为字符串文字，然后将它们存储为 `plain / html MIMEText` 对象。 然后可以按顺序将这些添加到 `MIMEMultipart（"alternative"）` 消息中，并通过与电子邮件服务器的安全连接发送。 请记住在替代的纯文本后添加HTML消息，因为电子邮件客户端将尝试首先渲染最后一个子部分。

