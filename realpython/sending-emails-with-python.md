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

# 发送纯文本的电子邮件

