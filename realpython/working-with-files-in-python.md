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

`os.scandir()` 被调用时返回一个迭代器而不是一个列表。

***

