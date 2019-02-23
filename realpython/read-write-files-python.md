> 本文为译文，原文链接 [read-write-files-python](https://realpython.com/read-write-files-python/) 
> 本人博客: [编程禅师](http://blog.jiangyixin.top)

使用Python做的最常见的任务是读取和写入文件。无论是写入简单的文本文件，读取复杂的服务器日志，还是分析原始的字节数据。所有这些情况都需要读取或写入文件。

**在本教程中，你将学习：**

* 文件的构成以及为什么这在Python中很重要
* Python中读取和写入文件的基础
* 用Python进行读取和写入文件的一些情景

本教程主要面向初学者到中级的Python开发者，但是这里有一些提示，更高级的程序员也可以从中获益。

# 什么是文件？

在我们开始研究如何使用Python中的文件之前，了解文件究竟是什么以及现代操作系统如何处理它们的某些方面是非常重要的。

从本质上讲，文件是[用于存储数据](https://en.wikipedia.org/wiki/Computer_file)的连续字节集。这些数据以特定格式组织，可以是任何像文本文件一样简单的数据，也可以像程序可执行文件一样复杂。最后，这些字节文件被翻译成二进制文件`1`，`0`以便计算机更容易处理。

大多数现代文件系统上的文件由三个主要部分组成：

1. **标题(Header)：**有关文件内容的元数据（文件名，大小，类型等）
2. **数据(Data)：**由创建者或编辑者编写的文件内容
3. **文件结束（EOF）：**表示文件结尾的特殊字符

[![FileForma](https://i.loli.net/2019/02/23/5c709d130c2ca.jpg)](https://i.loli.net/2019/02/23/5c709d130c2ca.jpg)

数据表示的内容取决于所使用的格式规范，通常由扩展名表示。例如，扩展名为`.gif`的文件最可能符合[图形交换格式](https://en.wikipedia.org/wiki/GIF)规范。有数百个（如果不是数千个）[文件扩展名](https://en.wikipedia.org/wiki/List_of_filename_extensions)。对于本教程，你将只处理`.txt`或`.csv`文件扩展名。

## 文件路径

在操作系统上访问文件时，需要文件路径。文件路径是表示文件位置的字符串。它分为三个主要部分：

1. **文件夹路径：**文件系统上的文件夹位置，后续文件夹由正斜杠`/`（Unix）或反斜杠`\`（Windows）分隔
2. **文件名：文件**的实际名称
3. **扩展名：**文件路径的末尾预先设置了句号（`.`），用于表示文件类型

这是一个简单的例子。假设你有一个位于文件结构中的文件，如下所示：

```shell
/
│
├── path/
|   │
│   ├── to/
│   │   └── cats.gif
│   │
│   └── dog_breeds.txt
|
└── animals.csv
```

假设您要访问该`cats.gif`文件，并且你当前的位置与文件夹中`path`平级。要访问该文件，你需要浏览该`path`文件夹，然后查看`to`文件夹，最后到达该`cats.gif`文件。文件夹路径是`path/to/`。文件名是`cats`。文件扩展名是`.gif`。所以完整的道路是`path/to/cats.gif`。

现在假设你当前的位置或当前工作目录（cwd）位于我们的示例文件夹结构的`to`文件夹中。可以通过文件名和扩展名简单地引用文件`cats.gif`，而不是引用`cats.gif`完整路径`path/to/cats.gif` 。

```shell
/
│
├── path/
|   │
|   ├── to/  ← Your current working directory (cwd) is here
|   │   └── cats.gif  ← Accessing this file
|   │
|   └── dog_breeds.txt
|
└── animals.csv
```

但对于`dog_breeds.txt`如何进行访问呢？如果不使用完整路径，你将如何访问？你可以使用特殊字符双点（`..`）来向前移动一个目录。这意味着可以在`to`目录使用`../dog_breeds.txt`引用`dog_breeds.txt`文件。

```shell
/
│
├── path/  ← Referencing this parent folder
|   │
|   ├── to/  ← Current working directory (cwd)
|   │   └── cats.gif
|   │
|   └── dog_breeds.txt  ← Accessing this file
|
└── animals.csv
```

双点（`..`）可以连接在一起以遍历当前目录之前的多个目录。例如，在`to`文件夹中要访问`animals.csv`，你将使用`../../animals.csv`。

## 行结尾

处理文件数据时经常遇到的一个问题是新行或行结尾的表示。行结尾起源于莫尔斯电码时代，[使用一个特定的符号被用来表示传输的结束或一行的结尾](https://en.wikipedia.org/wiki/Prosigns_for_Morse_code#Official_International_Morse_code_procedure_signs)。

后来，国际标准化组织（ISO）和美国标准协会（ASA）[对电传打字机](https://en.wikipedia.org/wiki/Newline#History)进行了标准化。ASA标准规定行尾应使用回车（序列`CR`或`\r`）*和*换行（`LF`或`\n`）字符（`CR+LF`或`\r\n`）。然而，ISO标准允许`CR+LF`字符或仅`LF`字符。

[Windows使用`CR+LF`字符](https://unix.stackexchange.com/a/411830)表示新行，而Unix和较新的Mac版本仅使用`LF`字符。当你处理来源于不同操作系统上的文件时，这可能会导致一些复杂情况。这是一个简单的例子。假设我们检查在Windows系统上创建的文件`dog_breeds.txt`：

```reStructuredText
Pug\r\n
Jack Russel Terrier\r\n
English Springer Spaniel\r\n
German Shepherd\r\n
Staffordshire Bull Terrier\r\n
Cavalier King Charles Spaniel\r\n
Golden Retriever\r\n
West Highland White Terrier\r\n
Boxer\r\n
Border Terrier\r\n
```

同样的输出将在Unix设备上以不同方式解释：

```reStructuredText
Pug\r
\n
Jack Russel Terrier\r
\n
English Springer Spaniel\r
\n
German Shepherd\r
\n
Staffordshire Bull Terrier\r
\n
Cavalier King Charles Spaniel\r
\n
Golden Retriever\r
\n
West Highland White Terrier\r
\n
Boxer\r
\n
Border Terrier\r
\n
```

这可能会使每行重复出现问题，你可能需要考虑这样的情况。

## 字符编码

你可能面临的另一个常见问题是字节数据的编码。编码是从字节数据到人类可读字符的转换。通常通过指定编码的格式来完成。两种最常见的编码是[ASCII](https://www.ascii-code.com/)和[UNICODE](https://unicode.org/)格式。[ASCII只能存储128个字符](https://en.wikipedia.org/wiki/ASCII)，而[Unicode最多可包含1,114,112个字符](https://en.wikipedia.org/wiki/Unicode)。

ASCII实际上是Unicode（UTF-8）的子集，这意味着ASCII和Unicode共享相同的数值字符值。重要的是要注意，使用不正确的字符编码解析文件可能会导致字符转换失败和出错。例如，如果文件是使用UTF-8编码创建的，并且你尝试使用ASCII编码对其进行解析，则如果存在超出这128个值的字符，则会引发错误。

***

# 在Python中打开和关闭文件

当你想使用文件时，首先要做的就是打开它。该操作通过调用 [open()](https://docs.python.org/3/library/functions.html#open) 内置函数完成的。`open()`有一个必需的参数，它是文件的路径。`open()`有一个返回，是这个文件的[文件对象](https://docs.python.org/3/glossary.html#term-file-object)：

```python
file = open('dog_breeds.txt')
```

打开文件后，接下来要学习的是如何关闭它。

> **警告：**你应*始终*确保正确关闭打开的文件。

重要的是要记住，关闭文件是你的责任。在大多数情况下，在应用程序或脚本终止时，文件最终将被关闭。但是，无法保证实际上将会发生什么。这可能导致不必要的行为，包括资源泄漏。这也是Python（Pythonic）中的最佳实践，以确保你的代码以明确定义的方式运行并减少任何不需要的行为。

当你操作文件时，有两种方法可以确保文件正确关闭，即使遇到错误也是如此。关闭文件的第一种方法是使用`try-finally`块：

```python
reader = open('dog_breeds.txt')
try:
    # Further file processing goes here
finally:
    reader.close()
```

如果你不熟悉`try-finally`块的内容，请查看[Python Exceptions：An Introduction](https://realpython.com/python-exceptions/)。

关闭文件的第二种方法是使用以下`with`语句：

```python
with open('dog_breeds.txt') as reader:
    # Further file processing goes here
```

使用 `with`语句，一旦离开了`with`块或甚至在错误的情况下，系统也会自动关闭文件。我强烈建议你尽可能使用`with`语句，因为它的代码更加清晰并使你更容易处理任何意外错误。

最有可能的是，你也想要使用第二个位置参数`mode`。此参数是一个字符串，其中包含多个字符以表示你要如何打开文件。默认值和最常见的是`'r'`，表示以只读模式将文件作为文本文件打开：

```python
with open('dog_breeds.txt', 'r') as reader:
    # Further file processing goes here
```

其他模式请看[在线文档](https://docs.python.org/3/library/functions.html#open)，但最常用的模式如下：

| 模式             | 含义                                  |
| ---------------- | ------------------------------------- |
| 'r'              | 只读模式打开(默认)                    |
| ‘w’              | 写入模式打开，会覆盖文件              |
| `'rb'` 或 `'wb'` | 以二进制模式打开（使用字节数据读/写） |

让我们回过头来谈谈文件对象。文件对象是：

> “将面向文件的API（使用`read()`or `write()` 等方法）暴露给底层资源的对象。”（[来源](https://docs.python.org/3/glossary.html#term-file-object)）

有三种不同类型的文件对象：

- 文本文件
- 缓冲的二进制文件
- 原始二进制文件

这些中每一种文件类型的都在`io`模块中定义。这里简要介绍了这三种类型。

## 文本文件

文本文件是你将遇到的最常见的文件。以下是一些如何打开这些文件的示例：

```python
open('abc.txt')

open('abc.txt', 'r')

open('abc.txt', 'w')
```

对于此类型的文件，`open()`将返回一个`TextIOWrapper`文件对象：

```shell
>>> file = open('dog_breeds.txt')
>>> type(file)
<class '_io.TextIOWrapper'>
```

这是`open()`默认返回的文件对象。

## 缓冲二进制文件类型

缓冲二进制文件类型用于读取和写入二进制文件。以下是一些如何打开这些文件的示例：

```python
open('abc.txt', 'rb')

open('abc.txt', 'wb')
```

对于此类型的文件，`open()`将返回一个`BufferedReader`或`BufferedWriter`文件对象：

```shell
>>> file  =  open （'dog_breeds.txt' ， 'rb' ）
>>> type （file ）
<class'_io.BufferedReader'> 
>>> file  =  open （'dog_breeds.txt' ， 'wb' ）
> >> type （file ）
<class'_io.BufferedWriter'>
```

## 原始文件类型

原始文件类型是：

> “通常用作二进制和文本流的低级构建块。”（[来源](https://docs.python.org/3.7/library/io.html#raw-i-o)）

因此通常不使用它。

以下是如何打开这些文件的示例：

```python
open('abc.txt', 'rb', buffering=0)
```

对于此类型的文件，`open()`将返回一个`FileIO`文件对象：

```shell
>>> file = open('dog_breeds.txt', 'rb', buffering=0)
>>> type(file)
<class '_io.FileIO'>
```

***

