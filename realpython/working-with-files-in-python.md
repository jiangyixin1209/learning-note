Python中有几个内置模块和方法来处理文件。这些方法被分割到例如`os`,  `os.path` , `shutil`  和 `pathlib` 等等几个模块中。文章将列举Python中对文件最常用的操作和方法。

在这篇指南中，你将学习如何：

* 获取文件属性
* 创建目录
* 匹配文件名中的模式
* 遍历目录树
* 创建临时文件和目录
* 删除文件和目录
* 复制、移动和重命名文件和目录
* 创建和解压ZIP和TAR档案
* 使用`fileinput` 模块打开多个文件

# Python中文件数据的读和写

使用Python对文件进行读和写是十分简单的。为此，你首先必须使用合适的模式打开文件。这里有一个如何打开文本文件并读取其内容的例子。

```python
with open('data.txt', 'r') as f:
    data = f.read()
    print('context: {}'.format(data))
```

open()接收一个文件名和一个模式作为它的参数，`r` 表示以只读模式打开文件。想要往文件中写数据的话，则用`w` 作为参数。

```python
with open('data.txt', 'w') as f:
    data = 'some data to be written to the file'
    f.write(data)
```

在上述例子中，open()打开用于读取或写入的文件并返回文件句柄(本例子中的 `f` )，该句柄提供了可用于读取或写入文件数据的方法。阅读 [Working With File I/O in Python](https://dbader.org/blog/python-file-io) 获取更多关于如何读写文件的信息。

***

# 获取目录列表

假设你当前的工作目录有一个叫 `my_directory` 的子目录，该目录包含如下内容：

```shell
.
├── file1.py
├── file2.csv
├── file3.txt
├── sub_dir
│   ├── bar.py
│   └── foo.py
├── sub_dir_b
│   └── file4.txt
└── sub_dir_c
    ├── config.py
    └── file5.txt
```

Python内置的 `os` 模块有很多有用的方法能被用来列出目录内容和过滤结果。为了获取文件系统中特定目录的所有文件和文件夹列表，可以在遗留版本的Python中使用 `os.listdir()` 或 在Python 3.x 中使用 `os.scandir()` 。 如果你还想获取文件和目录属性(如文件大小和修改日期)，那么 `os.scandir()` 则是首选的方法。

## 使用遗留版本的Python获取目录列表

```python
import os
entries = os.listdir('my_directory')
```

`os.listdir()` 返回一个Python列表，其中包含path参数所指目录的文件和子目录的名称。

```python
['file1.py', 'file2.csv', 'file3.txt', 'sub_dir', 'sub_dir_b', 'sub_dir_c']
```

目录列表现在看上去不容易阅读，对 `os.listdir()` 的调用结果使用循环打印有助于查看。

```python
for entry in entries:
    print(entry)

"""
sub_dir_c
file1.py
sub_dir_b
file3.txt
file2.csv
sub_dir
"""

```

## 使用现代版本的Python获取目录列表

在现代Python版本中，可以使用 `os.scandir()` 和 `pathlib.Path` 来替代 `os.listdir()` 。

`os.scandir()` 在Python 3.5 中被引用，其文档为 [PEP 471](https://www.python.org/dev/peps/pep-0471/) 。

`os.scandir()` 调用时返回一个迭代器而不是一个列表。

```python
import os
entries = os.scandir('my_directory')
print(entries)
# <posix.ScandirIterator at 0x105b4d4b0>
```

ScandirIterator 指向了当前目录中的所有条目。你可以遍历迭代器的内容，并打印文件名。

```python
import os
with os.scandir('my_directory') as entries:
    for entry in entries:
        print(entry.name)
```

这里 `os.scandir()` 和with语句一起使用，因为它支持上下文管理协议。使用上下文管理器关闭迭代器并在迭代器耗尽后自动释放获取的资源。在 `my_directory` 打印文件名的结果就和在 `os.listdir()` 例子中看到的一样：

```shell
sub_dir_c
file1.py
sub_dir_b
file3.txt
file2.csv
sub_dir
```

另一个获取目录列表的方法是使用 `pathlib` 模块：

```python
from pathlib import Path

entries = Path('my_directory')
for entry in entries.iterdir():
    print(entry.name)
```

`pathlib.Path()` 返回的是 `PosixPath` 或 `WindowsPath` 对象，这取决于操作系统。

`pathlib.Path()` 对象有一个 `.iterdir()` 的方法用于创建一个迭代器包含该目录下所有文件和目录。由 `.iterdir()` 生成的每个条目都包含文件或目录的信息，例如其名称和文件属性。`pathlib` 在Python3.4时被第一次引入，并且是对Python一个很好的加强，它为文件系统提供了面向对象的接口。

在上面的例子中，你调用 `pathlib.Path()` 并传入了一个路径参数。然后调用 `.iterdir()` 来获取 `my_directory` 下的所有文件和目录列表。

`pathlib` 提供了一组类，以简单并且面向对象的方式提供了路径上的大多数常见的操作。使用 `pathlib` 比起使用 `os` 中的函数更加有效。和 `os` 相比，使用 `pathlib` 的另一个好处是减少了操作文件系统路径所导入包或模块的数量。想要了解更多信息，可以阅读 [Python 3’s pathlib Module: Taming the File System](https://realpython.com/python-pathlib/) 。

运行上述代码会得到如下结果:

```python
sub_dir_c
file1.py
sub_dir_b
file3.txt
file2.csv
sub_dir
```

使用 `pathlib.Path()` 或 `os.scandir()` 来替代 `os.listdir()` 是获取目录列表的首选方法，尤其是当你需要获取文件类型和文件属性信息的时候。`pathlib.Path()` 提供了在 `os` 和 `shutil` 中大部分处理文件和路径的功能，并且它的方法比这些模块更加有效。我们将讨论如何快速的获取文件属性。

| 函数                     | 描述                                                     |
| ------------------------ | -------------------------------------------------------- |
| os.listdir()             | 以列表的方式返回目录中所有的文件和文件夹                 |
| os.scandir()             | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |
| pathlib.Path().iterdir() | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |



***

