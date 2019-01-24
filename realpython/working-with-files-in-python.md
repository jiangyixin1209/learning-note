[TOC]

Python中有几个内置模块和方法来处理文件。这些方法被分割到例如`os`,  `os.path` , `shutil`  和 `pathlib` 等等几个模块中。文章将列举Python中对文件最常用的操作和方法。

在这篇文章中，你将学习如何：

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

`open()` 接收一个文件名和一个模式作为它的参数，`r` 表示以只读模式打开文件。想要往文件中写数据的话，则用`w` 作为参数。

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
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
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
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
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
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
```

使用 `pathlib.Path()` 或 `os.scandir()` 来替代 `os.listdir()` 是获取目录列表的首选方法，尤其是当你需要获取文件类型和文件属性信息的时候。`pathlib.Path()` 提供了在 `os` 和 `shutil` 中大部分处理文件和路径的功能，并且它的方法比这些模块更加有效。我们将讨论如何快速的获取文件属性。

| 函数                     | 描述                                                     |
| ------------------------ | -------------------------------------------------------- |
| os.listdir()             | 以列表的方式返回目录中所有的文件和文件夹                 |
| os.scandir()             | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |
| pathlib.Path().iterdir() | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |

这些函数返回目录中所有内容的列表，包括子目录。这可能并总是你一直想要的结果，下一节将向你展示如何从目录列表中过滤结果。

## 列出目录中的所有文件

这节将向你展示如何使用 `os.listdir()` ，`os.scandir()` 和 `pathlib.Path()` 打印出目录中文件的名称。为了过滤目录并仅列出 `os.listdir()` 生成的目录列表的文件，要使用 `os.path` ：

```python
import os

basepath = 'my_directory'
for entry in os.listdir(basepath):
    # 使用os.path.isfile判断该路径是否是文件类型
    if os.path.isfile(os.path.join(base_path, entry)):
        print(entry)
```

在这里调用 `os.listdir()` 返回指定路径中所有内容的列表，接着使用 `os.path.isfile()` 过滤列表让其只显示文件类型而非目录类型。代码执行结果如下：

```shell
file1.py
file2.csv
file3.txt
```

一个更简单的方式来列出一个目录中所有的文件是使用 `os.scandir()` 或 `pathlib.Path()` :

```python
import os

basepath = 'my_directory'
with os.scandir(basepath) as entries:
    for entry in entries:
        if entry.is_file():
            print(entry.name)
```

使用 `os.scandir()` 比起 `os.listdir()` 看上去更清楚和更容易理解。对 `ScandirIterator` 的每一项调用 `entry.isfile()` ，如果返回 `True` 则表示这一项是一个文件。上述代码的输出如下：

```shell
file1.py
file3.txt
file2.csv
```

接着，展示如何使用 `pathlib.Path()` 列出一个目录中的文件：

```python
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    if entry.is_file():
        print(entry.name)
```

在 `.iterdir()` 产生的每一项调用 `.is_file()` 。产生的输出结果和上面相同：

```shell
file1.py
file3.txt
file2.csv
```

如果将for循环和if语句组合成单个生成器表达式，则上述的代码可以更加简洁。关于生成器表达式，推荐一篇[Dan Bader](https://dbader.org/blog/python-generator-expressions) 的文章。

修改后的版本如下：

```python
from pathlib import Path

basepath = Path('my_directory')
files_in_basepath = (entry for entry in basepath.iterdir() if entry.is_file())
for item in files_in_basepath:
    print(item.name)
```

上述代码的执行结果和之前相同。本节展示使用 `os.scandir()` 和 `pathlib.Path()` 过滤文件或目录比使用 `os.listdir()` 和 `os.path` 更直观，代码看起来更简洁。

## 列出子目录

如果要列出子目录而不是文件，请使用下面的方法。现在展示如何使用 `os.listdir()` 和 `os.path()` :

```python
import os

basepath = 'my_directory'
for entry in os.listdir(basepath):
    if os.path.isdir(os.path.join(basepath, entry)):
        print(entry)
```

当你多次调用 `os.path,join()` 时，以这种方式操作文件系统就会变得很笨重。在我电脑上运行此代码会产生以下输出：

```shell
sub_dir
sub_dir_b
sub_dir_c
```

下面是如何使用 `os.scandir()` ：

```python
import os

basepath = 'my_directory'
with os.scandir(basepath) as entries:
    for entry in entries:
        if entry.is_dir():
            print(entry.name)
```

与文件列表中的示例一样，此处在 `os.scandir()` 返回的每一项上调用 `.is_dir()` 。如果这项是目录，则 `is_dir()` 返回 True，并打印出目录的名称。输出结果和上面相同：

```shell
sub_dir_c
sub_dir_b
sub_dir
```

下面是如何使用 `pathlib.Path()` ：

```python
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    if entry.is_dir():
        print(entry.name)
```

在 `.iterdir()` 迭代器返回的每一项上调用 `is_dir()` 检查是文件还是目录。如果该项是目录，则打印其名称，并且生成的输出与上一示例中的输出相同：

```shell
sub_dir_c
sub_dir_b
sub_dir
```

***

# 获取文件属性

Python可以很轻松的获取文件大小和修改时间等文件属性。可以通过使用 `os.stat()` ， `os.scandir()` 或 `pathlib.Path` 来获取。

`os.scandir()` 和 `pathlib.Path()` 能直接获取到包含文件属性的目录列表。这可能比使用 `os.listdir()` 列出文件然后获取每个文件的文件属性信息更加有效。

下面的例子显示了如何获取 `my_directory` 中文件的最后修改时间。以时间戳的方式输出：

```python
import os

with os.scandir('my_directory') as entries:
    for entry in entries:
        info = entry.stat()
        print(info.st_mtime)
        
"""
1548163662.3952665
1548163689.1982062
1548163697.9175904
1548163721.1841028
1548163740.765162
1548163769.4702623
"""
```

`os.scandir()` 返回一个 `ScandirIterator` 对象。`ScandirIterator` 对象中的每一项有 `.stat()` 方法能获取关于它指向文件或目录的信息。`.stat()` 提供了例如文件大小和最后修改时间的信息。在上面的示例中，代码打印了 `st_time` 属性，该属性是上次修改文件内容的时间。

`pathlib` 模块具有相应的方法，用于获取相同结果的文件信息:

```python
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    info = entry.stat()
    print(info.st_mtime)

"""
1548163662.3952665
1548163689.1982062
1548163697.9175904
1548163721.1841028
1548163740.765162
1548163769.4702623
"""
```

在上面的例子中，循环 `.iterdir()` 返回的迭代器并通过对其中每一项调用 `.stat()` 来获取文件属性。`st_mtime` 属性是一个浮点类型的值，表示的是时间戳。为了让 `st_time` 返回的值更容易阅读，你可以编写一个辅助函数将其转换为一个 `datetime` 对象：

```python
import datetime                                                                   
from pathlib import Path                                                          
                                                                                  
                                                                                  
def timestamp2datetime(timestamp, convert_to_local=True, utc=8, is_remove_ms=True)
    """                                                                           
    转换 UNIX 时间戳为 datetime对象                                                       
    :param timestamp: 时间戳                                                         
    :param convert_to_local: 是否转为本地时间                                             
    :param utc: 时区信息，中国为utc+8                                                     
    :param is_remove_ms: 是否去除毫秒                                                   
    :return: datetime 对象                                                          
    """                                                                           
    if is_remove_ms:                                                              
        timestamp = int(timestamp)                                                
    dt = datetime.datetime.utcfromtimestamp(timestamp)                            
    if convert_to_local:                                                          
        dt = dt + datetime.timedelta(hours=utc)                                   
    return dt                                                                     
                                                                                  
                                                                                  
def convert_date(timestamp, format='%Y-%m-%d %H:%M:%S'):                          
    dt = timestamp2datetime(timestamp)                                            
    return dt.strftime(format)                                                    
                                                                                  
                                                                                  
basepath = Path('my_directory')                                                   
for entry in basepath.iterdir():
    if entry.is_file()
    	info = entry.stat()                                                           
    	print('{} 上次修改时间为 {}'.format(entry.name, timestamp2datetime(info.st_mtime)))  
```

首先得到 `my_directory` 中文件的列表以及它们的属性，然后调用 `convert_date()` 来转换文件最后修改时间让其以一种人类可读的方式显示。`convert_date()` 使用 `.strftime()` 将datetime类型转换为字符串。

上述代码的输出结果：

```shell
file3.txt 上次修改时间为 2019-01-24 09:04:39
file2.csv 上次修改时间为 2019-01-24 09:04:39
file1.py 上次修改时间为 2019-01-24 09:04:39
```

将日期和时间转换为字符串的语法可能会让你感到混乱。如果要了解更多的信息，请查询相关的[官方文档](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior) 。另一个方式则是阅读 [http://strftime.org](http://strftime.org) 。

***

# 创建目录

