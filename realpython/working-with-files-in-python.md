[TOC]

Python中有几个内置模块和方法来处理文件。这些方法被分割到例如`os`,  `os.path` , `shutil`  和 `pathlib` 等等几个模块中。文章将列举Python中对文件最常用的操作和方法。

在这篇文章中，你将学习如何：

* 获取文件属性
* 创建目录
* 文件名模式匹配
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

你编写的程序迟早需要创建目录以便在其中存储数据。 `os` 和 `pathlib` 包含了创建目录的函数。我们将会考虑如下方法：

| 方法                 | 描述                       |
| -------------------- | -------------------------- |
| os.mkdir()           | 创建单个子目录             |
| os.makedirs()        | 创建多个目录，包括中间目录 |
| Pathlib.Path.mkdir() | 创建单个或多个目录         |

## 创建单个目录

要创建单个目录，把目录路径作为参数传给 `os.mkdir()` :

```python
import os

os.mkdir('example_directory')
```

如果该目录已经存在，`os.mkdir()` 将抛出 `FileExistsError` 异常。或者，你也可以使用 `pathlib` 来创建目录:

```python
from pathlib import Path

p = Path('example_directory')
p.mkdir()
```

如果路径已经存在，`mkdir()` 会抛出 `FileExistsError` 异常:

```shell
FileExistsError: [Errno 17] File exists: 'example_directory'
```

为了避免像这样的错误抛出， 当发生错误时捕获错误并让你的用户知道:

```python
from pathlib import Path

p = Path('example_directory')
try:
    p.mkdir()
except FileExistsError as e:
    print(e)
```

或者，你可以给 `.mkdir()` 传入 `exist_ok=True` 参数来忽略 `FileExistsError` 异常:

```python
from pathlib import Path

p = Path('example_directory')
p.mkdir(exist_ok=True)
```

如果目录已存在，则不会引起错误。

## 创建多个目录

`os.makedirs()` 和 `os.mkdir()` 类似。两者之间的区别在于，`os.makedirs()` 不仅可以创建单独的目录，还可以递归的创建目录树。换句话说，它可以创建任何必要的中间文件夹，来确保存在完整的路径。

`os.makedirs()` 和在bash中运行 `mkdir -p` 类似。例如，要创建一组目录像 2018/10/05，你可以像下面那样操作:

```python
import os

os.makedirs('2018/10/05', mode=0o770)
```

上述代码创建了 `2018/10/05` 的目录结构并为所有者和组用户提供读、写和执行权限。默认的模式为 `0o777` ，增加了其他用户组的权限。有关文件权限以及模式的应用方式的更多详细信息，请参考 [文档](https://docs.python.org/3/library/os.html#os.makedirs) 。

运行 `tree` 命令确认我们应用的权限:

```shell
$ tree -p -i .
.
[drwxrwx---]  2018
[drwxrwx---]  10
[drwxrwx---]  05
```

上述代码打印出当前目录的目录树。 `tree` 通常被用来以树形结构列出目录的内容。传入 `-p` 和 `-i` 参数则会以垂直列表打印出目录名称以及其文件权限信息。`-p` 用于输出文件权限，`-i` 则用于让 `tree` 命令产生一个没有缩进线的垂直列表。

正如你所看到的，所有的目录都拥有 770 权限。另一个方式创建多个目录是使用 `pathlib.Path` 的 `.mkdir()` :

```python
from pathlib import Path

p = Path('2018/10/05')
p.mkdir(parents=True, exist_ok=True)
```

通过给 `Path.mkdir()` 传递 `parents=True` 关键字参数使它创建 `05` 目录和使其路径有效的所有父级目录。

在默认情况下，`os.makedirs()` 和 `pathlib.Path.mkdir()` 会在目标目录存在的时候抛出 `OSError` 。通过每次调用函数时传递 `exist_ok=True` 作为关键字参数则可以覆盖此行为（从Python3.2开始）。

运行上述代码会得到像下面的结构：

```shell
└── 2018
    └── 10
        └── 05
```

我更喜欢在创建目录时使用 `pathlib` ，因为我可以使用相同的函数方法来创建一个或多个目录。

***

# 文件名模式匹配

使用上述方法之一获取目录中的文件列表后，你可能希望搜索和特定的模式匹配的文件。

下面这些是你可以使用的方法和函数：

* `endswith()` 和 `startswith()` 字符串方法
* `fnmatch.fnmatch()`
* `glob.glob()`
* `pathlib.Path.glob()`

这些方法和函数是下面要讨论的。本小节的示例将在名为 `some_directory` 的目录下执行，该目录具有以下的结构：

```shell
.
├── admin.py
├── data_01_backup.txt
├── data_01.txt
├── data_02_backup.txt
├── data_02.txt
├── data_03_backup.txt
├── data_03.txt
├── sub_dir
│   ├── file1.py
│   └── file2.py
└── tests.py
```

如果你正在使用 Bash shell，你可以使用以下的命令创建上述目录结构:

```shell
mkdir some_directory
cd some_directory
mkdir sub_dir
touch sub_dir/file1.py sub_dir/file2.py
touch data_{01..03}.txt data_{01..03}_backup.txt admin.py tests.py
```

这将会创建 `some_directory` 目录并进入它，接着创建 `sub_dir` 。下一行在 `sub_dir` 创建 `file1.py` 和 `file2.py` ，最后一行使用扩展创建其它所有文件。想要学习更多关于shell扩展，请阅读 [这里](http://linuxcommand.org/lc3_lts0080.php) 。

## 使用字符串方法

Python有几个内置 [修改和操作字符串](https://realpython.com/python-strings/) 的方法。当在匹配文件名时，其中的两个方法 `.startswith()` 和 `.endswith()` 非常有用。要做到这点，首先要获取一个目录列表，然后遍历。 

```python
import os

for f_name in os.listdir('some_directory'):
    if f_name.endswith('.txt'):
        print(f_name)
```

上述代码找到 `some_directory` 中的所有文件，遍历并使用 `.endswith()` 来打印所有扩展名为 `.txt` 的文件名。运行代码在我的电脑上输出如下:

```shell
data_01.txt
data_01_backup.txt
data_02.txt
data_02_backup.txt
data_03.txt
data_03_backup.txt
```

## 使用 `fnmatch` 进行简单文件名模式匹配

字符串方法匹配的能力是有限的。`fnmatch` 有对于模式匹配有更先进的函数和方法。我们将考虑使用 `fnmatch.fnmatch()` ，这是一个支持使用 `*` 和 `?` 等通配符的函数。例如，使用 `fnmatch` 查找目录中所有 `.txt` 文件，你可以这样做:

```python
import os
import fnmatch

for f_name in os.listdir('some_directory'):
    if fnmatch.fnmatch(f_name, '*.txt'):
        print(f_name)
```

迭代 `some_directory` 中的文件列表，并使用 `.fnmatch()` 对扩展名为 `.txt` 的文件执行通配符搜索。

## 更先进的模式匹配

假设你想要查找符合特定掉件的 `.txt` 文件。例如，你可能指向找到包含单次 `data` 的  `.txt`文件，一组下划线之间的数字，以及文件名中包含单词 `backup` 。就类似于 `data_01_backup`, `data_02_backup`, 或 `data_03_backup` 。

你可以这样使用 `fnmatch.fnmatch()` :

```python
import os
import fnmatch

for f_name in os.listdir('some_directory'):
    if fnmatch.fnmatch(f_name, 'data_*_backup.txt'):
        print(f_name)
```

这里就仅仅打印出匹配 `data_*_backup.txt` 模式的文件名称。模式中的 `*` 将匹配任何字符，因此运行这段代码则将查找文件名以 `data` 开头并以 `backup.txt` 的所有文本文件，就行下面的输出所示 :

```python
data_01_backup.txt
data_02_backup.txt
data_03_backup.txt
```

## 使用 `glob` 进行文件名模式匹配

另一个有用的模式匹配模块是 `glob` 。

`.glob()` 在 `glob` 模块中的左右就像 `fnmatch.fnmatch()`，但是与 `fnmach.fnmatch()` 不同的是，它将以 `.` 开头的文件视为特殊文件。

UNIX和相关系统在文件列表中使用通配符像 `?` 和 `*` 表示全匹配。

例如，在UNIX shell中使用 `mv *.py python_files` 移动所有 `.py` 扩展名 的文件从当前目录到 `python_files` 。这 `*` 是一个通配符表示任意数量的字符，`*.py` 是一个全模式。Windows操作系统中不提供此shell功能。但 `glob` 模块在Python中添加了此功能，使得Windows程序可以使用这个特性。

这里有一个使用 `glob` 模块在当前目录下查询所有Python代码文件:

```python
import glob

print(glob.glob('*.py'))
```

`glob.glob('*.py')` 搜索当前目录中具有 `.py` 扩展名的文件，并且将它们以列表的形式返回。 `glob` 还支持 shell 样式的通配符来进行匹配 :

```shell
import glob

for name in glob.glob('*[0-9]*.txt'):
    print(name)
```

这将找到所有文件名中包含数字的文本文件(`.txt`) :

```shell
data_01.txt
data_01_backup.txt
data_02.txt
data_02_backup.txt
data_03.txt
data_03_backup.txt
```

`glob` 也很容易在子目录中递归的搜索文件:

```python
import glob

for name in glob.iglob('**/*.py', recursive=True):
    print(name)
```

