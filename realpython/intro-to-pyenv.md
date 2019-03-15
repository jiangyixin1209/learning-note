> 本文为译文，原文链接 [https://realpython.com/intro-to-pyenv/](https://realpython.com/intro-to-pyenv/) 
> 本人博客: [编程禅师](http://blog.jiangyixin.top)

你是否曾想为一个支持多个Python版本的项目做出贡献，但不确定如何轻松[测试](https://realpython.com/python-testing/)所有的版本？你是否曾对最新和最好的Python版本感到好奇？也许你想尝试这些新功能，但你担心会破坏你的开发环境。幸运的是，使用`pyenv`管理多个Python版本可以解决这个难题。

本文将为你提供一个很好的介绍，如何最大限度地利用你在项目上工作的时间，并最大限度地减少探索合适Python版本的时间。

**在本文中，你将学习如何：**

1. 安装多个版本的Python
2. 安装最新的Python开发版本
3. 在已安装的版本之间切换
4. 通过`pyenv`使用虚拟环境 
5. 自动激活不同的Python版本和虚拟环境

# 为何使用pyenv？

`pyenv`是管理多个Python版本的绝佳工具。即使你已经在系统上安装了Python，也值得安装`pyenv`，以便你可以轻松地尝试新的语言功能或帮助你创建基于不同Python版本的项目。

## 为什么不使用System Python？

“System Python”是安装在操作系统上的Python。如果你使用的是Mac或Linux，那么默认情况下，当在终端输入`python`时，你将获得一个Python REPL。

那么，为什么不使用它呢？一种看法是这个Python真的*属于*操作系统。毕竟，它是操作系统自带的。当你运行时`which` 命令时会反映出来：

```shell
$ which python
/usr/bin/python
```

在这里，所有用户都可以看到`python`的位置是`/usr/bin/python`。但这可能不是你想要的Python版本：

```python
$ python -V
Pyhton 2.7.12
```

