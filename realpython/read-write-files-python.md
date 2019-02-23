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