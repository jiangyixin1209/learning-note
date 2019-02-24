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

# 读写已打开的文件

打开文件后，你将需要读取或写入文件。首先，让我们来阅读一个文件。可以在文件对象上调用多种方法：

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`.read(size=-1)`](https://docs.python.org/3.7/library/io.html#io.RawIOBase.read) | 这将根据`size`字节数从文件中读取。如果没有传递参数或`None`或`-1`，那么整个文件被读取。 |
| [`.readline(size=-1)`](https://docs.python.org/3.7/library/io.html#io.IOBase.readline) | 这将从该行读取最多`size`数量的字符。直到到行结尾，然后到下一行。如果没有参数被传递或`None`或`-1`，则整行（或行剩余的部分）被读出。 |
|                                                              |                                                              |

使用上面使用过的 `dog_breeds.txt` 文件，我们来看一些如何使用这些方法的示例。以下是如何使用 `.read()` 命令打开和读取整个文件的示例：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     # Read & print the entire file
>>>     print(reader.read())
Pug
Jack Russel Terrier
English Springer Spaniel
German Shepherd
Staffordshire Bull Terrier
Cavalier King Charles Spaniel
Golden Retriever
West Highland White Terrier
Boxer
Border Terrier
```

这是一个如何使用`.readline()`在一行中每次读取5个字节的示例：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     # Read & print the first 5 characters of the line 5 times
>>>     print(reader.readline(5))
>>>     # Notice that line is greater than the 5 chars and continues
>>>     # down the line, reading 5 chars each time until the end of the
>>>     # line and then "wraps" around
>>>     print(reader.readline(5))
>>>     print(reader.readline(5))
>>>     print(reader.readline(5))
>>>     print(reader.readline(5))
Pug

Jack 
Russe
l Ter
rier
```

> 译者注：第一次调用reader.readline(5) 实际打印出 Pug\r\n，因此可以看到有输出一个换行

以下是使用`.readlines()`将整个文件作为列表读取的示例：

```shell
>>> f = open('dog_breeds.txt')
>>> f.readlines()  # Returns a list object
['Pug\n', 'Jack Russel Terrier\n', 'English Springer Spaniel\n', 'German Shepherd\n', 'Staffordshire Bull Terrier\n', 'Cavalier King Charles Spaniel\n', 'Golden Retriever\n', 'West Highland White Terrier\n', 'Boxer\n', 'Border Terrier\n']
```

上面的例子也可以通过使用`list()`从文件对象创建列表来完成：

```python
>>> f = open('dog_breeds.txt')
>>> list(f)
['Pug\n', 'Jack Russel Terrier\n', 'English Springer Spaniel\n', 'German Shepherd\n', 'Staffordshire Bull Terrier\n', 'Cavalier King Charles Spaniel\n', 'Golden Retriever\n', 'West Highland White Terrier\n', 'Boxer\n', 'Border Terrier\n']
```

## 迭代文件中的每一行

读取文件时常见的事情是迭代每一行。以下是如何使用`.readline()`执行该迭代的示例：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     # Read and print the entire file line by line
>>>     line = reader.readline()
>>>     while line != '':  # The EOF char is an empty string
>>>         print(line, end='')
>>>         line = reader.readline()
Pug
Jack Russel Terrier
English Springer Spaniel
German Shepherd
Staffordshire Bull Terrier
Cavalier King Charles Spaniel
Golden Retriever
West Highland White Terrier
Boxer
Border Terrier
```

迭代文件中每一行的另一种方法是使用`.readlines()`文件对象。请记住，`.readlines()`返回一个列表，其中列表中的每个元素代表文件中的一行：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     for line in reader.readlines():
>>>         print(line, end='')
Pug
Jack Russell Terrier
English Springer Spaniel
German Shepherd
Staffordshire Bull Terrier
Cavalier King Charles Spaniel
Golden Retriever
West Highland White Terrier
Boxer
Border Terrier
```

但是，通过迭代文件对象本身可以进一步简化上述示例：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     # Read and print the entire file line by line
>>>     for line in reader:
>>>         print(line, end='')
Pug
Jack Russel Terrier
English Springer Spaniel
German Shepherd
Staffordshire Bull Terrier
Cavalier King Charles Spaniel
Golden Retriever
West Highland White Terrier
Boxer
Border Terrier
```

最后的方法更Pythonic，可以更快，更高效。因此，建议你改用它。

> **注意：**上面的一些示例包含`print('some text', end='')`。这`end=''`是为了防止Python为正在打印的文本添加额外的换行符，并仅打印从文件中读取的内容。

现在让我们深入研究文件。与读取文件一样，文件对象有多种方法可用于写入文件：

| 方法             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| .write(string)   | 将字符串写入文件。                                           |
| .writelines(seq) | 将序列写入文件。不会给每个序列项附加结尾符。这会由你来添加适当的结尾符。 |

以下是使用`.write()`和的简单示例`.writelines()`：

```python
with open('dog_breeds.txt', 'r') as reader:
    # Note: readlines doesn't trim the line endings
    dog_breeds = reader.readlines()

with open('dog_breeds_reversed.txt', 'w') as writer:
    # Alternatively you could use
    # writer.writelines(reversed(dog_breeds))

    # Write the dog breeds to the file in reversed order
    for breed in reversed(dog_breeds):
        writer.write(breed)
```

## 使用字节

有时，你可能需要使用[字节字符串](https://docs.python.org/3.7/glossary.html#term-bytes-like-object)处理文件。可以通过在`mode`参数中添加`'b'`字符来完成。适用于文件对象的所有相同方法。但是，每个方法都期望并返回一个`bytes`对象：

```shell
>>> with open(`dog_breeds.txt`, 'rb') as reader:
>>>     print(reader.readline())
b'Pug\n'
```

使用`b`标志打开文本文件并不那么有趣。假设我们有一张Jack Russell Terrier（`jack_russell.png`）的可爱图片：

![jack_russell.jpg](https://i.loli.net/2019/02/24/5c720730c47f9.jpg)

你可以在Python中打开该文件并检查内容！由于[`.png`文件格式](https://en.wikipedia.org/wiki/Portable_Network_Graphics)定义的那样，文件的标题是8个字节，如下所示：

| 值             | 描述                                    |
| -------------- | --------------------------------------- |
| 0x89           | 一个“魔术”数字，表示这是一个`PNG`的开始 |
| 0x50 0x4E 0x47 | `PNG` 的ASCII                           |
| 0x0D 0x0A      | DOS样式行结束 `\r\n`                    |
| 0x1A           | DOS风格的EOF字符                        |
| 0x0A           | 一个Unix风格的行结尾 `\n`               |

当打开文件并单独读取这些字节时，可以看到这确实是一个`.png`头文件：

```python
>>> with open('jack_russell.png', 'rb') as byte_reader:
>>>     print(byte_reader.read(1))
>>>     print(byte_reader.read(3))
>>>     print(byte_reader.read(2))
>>>     print(byte_reader.read(1))
>>>     print(byte_reader.read(1))
b'\x89'
b'PNG'
b'\r\n'
b'\x1a'
b'\n'
```

## 一个完整的例子: dos2unix.py

让我们把知识点整理一下，看看如何读取和写入文件的完整示例。下面是一个[`dos2unix`](https://en.wikipedia.org/wiki/Unix2dos)类似的工具，将其转换一个文件，将它的的行结束`\r\n`转为`\n`。

该工具分为三个主要部分。第一个是`str2unix()`将字符串从`\\r\\n`行结尾转换为`\\n`。第二个是`dos2unix()`将包含`\r\n`字符的字符串转换为`\n`。`dos2unix()`调用`str2unix()`。最后，有`__main__`块，只有当文件作为脚本执行时才会调用。

```python
"""
A simple script and library to convert files or strings from dos like
line endings with Unix like line endings.
"""

import argparse
import os


def str2unix(input_str: str) -> str:
    r"""\
    Converts the string from \r\n line endings to \n

    Parameters
    ----------
    input_str
        The string whose line endings will be converted

    Returns
    -------
        The converted string
    """
    r_str = input_str.replace('\r\n', '\n')
    return r_str


def dos2unix(source_file: str, dest_file: str):
    """\
    Coverts a file that contains Dos like line endings into Unix like

    Parameters
    ----------
    source_file
        The path to the source file to be converted
    dest_file
        The path to the converted file for output
    """
    # NOTE: Could add file existence checking and file overwriting
    # protection
    with open(source_file, 'r') as reader:
        dos_content = reader.read()

    unix_content = str2unix(dos_content)

    with open(dest_file, 'w') as writer:
        writer.write(unix_content)


if __name__ == "__main__":
    # Create our Argument parser and set its description
    parser = argparse.ArgumentParser(
        description="Script that converts a DOS like file to an Unix like file",
    )

    # Add the arguments:
    #   - source_file: the source file we want to convert
    #   - dest_file: the destination where the output should go

    # Note: the use of the argument type of argparse.FileType could
    # streamline some things
    parser.add_argument(
        'source_file',
        help='The location of the source '
    )

    parser.add_argument(
        '--dest_file',
        help='Location of dest file (default: source_file appended with `_unix`',
        default=None
    )

    # Parse the args (argparse automatically grabs the values from
    # sys.argv)
    args = parser.parse_args()

    s_file = args.source_file
    d_file = args.dest_file

    # If the destination file wasn't passed, then assume we want to
    # create a new file based on the old one
    if d_file is None:
        file_path, file_extension = os.path.splitext(s_file)
        d_file = f'{file_path}_unix{file_extension}'

    dos2unix(s_file, d_file)
```

***

# 技巧和窍门

现在你已经掌握了读取和写入文件的基础知识，这里有一些提示和技巧可以帮助你提高技能。

 ## `__file__`

该`__file__`属性是模块的[特殊属性](https://docs.python.org/3/reference/datamodel.html)，类似于`__name__`。它是：

> “如果是从文件加载的，它就为加载模块的文件的路径名，”（[来源](https://docs.python.org/3/reference/datamodel.html)）

> **注意：**`__file__`返回*相*对于调用初始Python脚本的路径。如果需要完整的系统路径，可以使用`os.getcwd()`获取执行代码的当前工作目录。

这是一个真实的例子。在我过去的一份工作中，我对硬件设备进行了多次测试。每个测试都是使用Python脚本编写的，测试脚本文件名用作标题。然后将执行这些脚本并使用`__file__`特殊属性打印其状态。这是一个示例文件夹结构：

```shell
project/
|
├── tests/
|   ├── test_commanding.py
|   ├── test_power.py
|   ├── test_wireHousing.py
|   └── test_leds.py
|
└── main.py
```

运行`main.py`产生以下内容：

```shell
>>> python main.py
tests/test_commanding.py Started:
tests/test_commanding.py Passed!
tests/test_power.py Started:
tests/test_power.py Passed!
tests/test_wireHousing.py Started:
tests/test_wireHousing.py Failed!
tests/test_leds.py Started:
tests/test_leds.py Passed!
```

## 追加文件内容

有时，你可能希望追加到文件或在已有文件的末尾开始写入。这可以通过在参数`mode`中追加`'a'`字符来完成：

```python
with open('dog_breeds.txt', 'a') as a_writer:
    a_writer.write('\nBeagle')
```

当对`dog_breeds.txt`再次检查时，你将看到文件的开头未更改，`Beagle`现在已添加到文件的末尾：

```shell
>>> with open('dog_breeds.txt', 'r') as reader:
>>>     print(reader.read())
Pug
Jack Russel Terrier
English Springer Spaniel
German Shepherd
Staffordshire Bull Terrier
Cavalier King Charles Spaniel
Golden Retriever
West Highland White Terrier
Boxer
Border Terrier
Beagle
```

## 同时使用两个文件

有时你可能想要读取文件并同时写入另一个文件。如果你使用在学习如何写入文件时显示的示例，它实际上可以合并到以下内容中：

```python
d_path = 'dog_breeds.txt'
d_r_path = 'dog_breeds_reversed.txt'
with open(d_path, 'r') as reader, open(d_r_path, 'w') as writer:
    dog_breeds = reader.readlines()
    writer.writelines(reversed(dog_breeds))
```

## 创建属于你自己的上下文管理器

有时候，你可能需要通过将文件对象放在自定义类中来更好地控制文件对象。执行此操作时，除非添加一些魔术方法，否则无法再使用`with`语句：通过添加`__enter__`和`__exit__`，你将创建所谓的[上下文管理器](https://docs.python.org/3/library/stdtypes.html#typecontextmanager)。

`__enter__()`调用`with`语句时调用。`__exit__()`从`with`语句块退出时被调用。

这是一个可用于制作自定义类的模板：

```python
class my_file_reader():
    def __init__(self, file_path):
        self.__path = file_path
        self.__file_object = None

    def __enter__(self):
        self.__file_object = open(self.__path)
        return self

    def __exit__(self, type, val, tb):
        self.__file_object.close()

    # Additional methods implemented below
```

现在你已经拥有了带有上下文管理器的自定义类，你可以与使用内置`open()`那样使用它：

```python
with my_file_reader('dog_breeds.txt') as reader:
    # Perform custom class operations
    pass
```

这是一个很好的例子。还记得我们有可爱的Jack Russell形象吗？也许你想打开其他`.png`文件，但不想每次都解析头文件。这是一个如何做到这一点的例子。此示例还使用自定义迭代器。如果你不熟悉它们，请查看[Python迭代器](https://dbader.org/blog/python-iterators)：

```python
class PngReader():
    # Every .png file contains this in the header.  Use it to verify
    # the file is indeed a .png.
    _expected_magic = b'\x89PNG\r\n\x1a\n'

    def __init__(self, file_path):
        # Ensure the file has the right extension
        if not file_path.endswith('.png'):
            raise NameError("File must be a '.png' extension")
        self.__path = file_path
        self.__file_object = None

    def __enter__(self):
        self.__file_object = open(self.__path, 'rb')

        magic = self.__file_object.read(8)
        if magic != self._expected_magic:
            raise TypeError("The File is not a properly formatted .png file!")

        return self

    def __exit__(self, type, val, tb):
        self.__file_object.close()

    def __iter__(self):
        # This and __next__() are used to create a custom iterator
        # See https://dbader.org/blog/python-iterators
        return self

    def __next__(self):
        # Read the file in "Chunks"
        # See https://en.wikipedia.org/wiki/Portable_Network_Graphics#%22Chunks%22_within_the_file

        initial_data = self.__file_object.read(4)

        # The file hasn't been opened or reached EOF.  This means we
        # can't go any further so stop the iteration by raising the
        # StopIteration.
        if self.__file_object is None or initial_data == b'':
            raise StopIteration
        else:
            # Each chunk has a len, type, data (based on len) and crc
            # Grab these values and return them as a tuple
            chunk_len = int.from_bytes(initial_data, byteorder='big')
            chunk_type = self.__file_object.read(4)
            chunk_data = self.__file_object.read(chunk_len)
            chunk_crc = self.__file_object.read(4)
            return chunk_len, chunk_type, chunk_data, chunk_crc
```

你现在可以打开`.png`文件，并使用自定义上下文管理器正确解析它们：

```shell
>>> with PngReader('jack_russell.png') as reader:
>>>     for l, t, d, c in reader:
>>>         print(f"{l:05}, {t}, {c}")
00013, b'IHDR', b'v\x121k'
00001, b'sRGB', b'\xae\xce\x1c\xe9'
00009, b'pHYs', b'(<]\x19'
00345, b'iTXt', b"L\xc2'Y"
16384, b'IDAT', b'i\x99\x0c('
16384, b'IDAT', b'\xb3\xfa\x9a$'
16384, b'IDAT', b'\xff\xbf\xd1\n'
16384, b'IDAT', b'\xc3\x9c\xb1}'
16384, b'IDAT', b'\xe3\x02\xba\x91'
16384, b'IDAT', b'\xa0\xa99='
16384, b'IDAT', b'\xf4\x8b.\x92'
16384, b'IDAT', b'\x17i\xfc\xde'
16384, b'IDAT', b'\x8fb\x0e\xe4'
16384, b'IDAT', b')3={'
01040, b'IDAT', b'\xd6\xb8\xc1\x9f'
00000, b'IEND', b'\xaeB`\x82'
```

***

# 不要重复造轮子

在处理文件时可能会遇到常见情况。大多数情况可以使用其他模块处理。您可能需要使用的两种常见文件类型是`.csv`和`.json`。*Real Python*已经汇总了一些关于如何处理这些内容的精彩文章：

- [用Python读写CSV文件](https://realpython.com/python-csv/)
- [在Python中使用JSON数据](https://realpython.com/python-json/)

此外，还有内置库，可以使用它们来帮助你：

- [**wave**](https://docs.python.org/3.7/library/wave.html)：读写WAV文件（音频）
- [**aifc**](https://docs.python.org/3/library/aifc.html)：读写AIFF和AIFC文件（音频）
- [**sunau**](https://docs.python.org/3/library/sunau.html)：读取和写入Sun AU文件
- [**tarfile**](https://docs.python.org/3/library/tarfile.html)：读取和写入tar归档文件
- [**zipfile**](https://docs.python.org/3/library/zipfile.html)：使用ZIP存档
- [**configparser**](https://docs.python.org/3/library/configparser.html)：轻松创建和解析配置文件
- [**xml.etree.ElementTree**](https://docs.python.org/3/library/xml.etree.elementtree.html)：创建或读取基于XML的文件
- [**msilib**](https://docs.python.org/3/library/msilib.html)：读取和写入Microsoft Installer文件
- [**plistlib**](https://docs.python.org/3/library/plistlib.html)：生成并解析Mac OS X `.plist`文件

还有更多的东西。此外，PyPI还有更多第三方工具可用。一些流行的是以下：

- [**PyPDF2**](https://pypi.org/project/PyPDF2/)：PDF工具包
- [**xlwings**](https://pypi.org/project/xlwings/)：读取和写入Excel文件
- [**Pillow**](https://pypi.org/project/Pillow/)：图像阅读和操作

***

# 总结

你现在知道如何使用Python处理文件，包括一些高级技术。使用Python中的文件现在比以往任何时候都更容易，当你开始这样做时，这是一种有益的感觉。

在本教程中，你已经了解到：

- 什么是文件
- 如何正确打开和关闭文件
- 如何读写文件
- 使用文件时的一些高级技术
- 一些库使用常见的文件类型