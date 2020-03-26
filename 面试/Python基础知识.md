# Python基础知识

## 1、语言特征及编码规范

### 1.1、Python的解释器有哪些？

* CPython：采用C语言开发的一种解释器，是目前最通用也是最多使用的解释器。
* IPython：是基于CPython之上的一个交互式解释器，增强了交互方式。其余和CPython一样。
* PyPy：目标是执行效率，采用JIT技术对Python代码进行动态编译，提高执行的速度。
* JPython：基于Java语言的解释器，可以将Python代码编译成Java字节码执行。
* IronPython：基于 `.NET` 语言的解释器，可以将Python编译成 `.NET` 字节码执行。

### 2.2、列举Python3和Python2的区别？

* Python3中默认使用UTF8编码，因此中文字符是合法的
* Python2中默认使用xrange，Python3中默认使用range
* Python3中的 `print` 是函数，必须使用括号
* Python2中默认字符串类型是ASCII，Python3中默认字符串类型是Unicode
* Python2中的 `/` 除法返回结果类型是整型，Python3中返回结果类型是浮点类型
* Python2中声明元类的方式为 `__metaclass__ = MetaClass` ，Python3中声明元类的方式为 `class NewClass(metaclass=MetaClass)` 
* Python3中处理异常使用 as 作为关键词 `except Exception as e` ，Python2中则为 `except Exception, e` 

### 2.3、Python中新式类和经典类的区别是什么？

在Python3中已经取消了经典类，默认都使用的是新式类，并且不必显示的继承`object` 。

Python3中的写法，下面三种写法效果都一样，皆使用了新式类

```python
class Fruit(object):
  pass

class Fruit:
  pass

class Fruit():
  pass
```

Python2中的写法

```python
# 使用新式类
class Fruit(object):
  pass

# 下面两种都使用经典类
class Fruit:
  pass

class Fruit():
  pass
```

在Python2版本中，新式类和经典类的主要区别为：

* 经典类是采用深度优先算法：当子类继承多个父类的情况时，如果继承的多个父类有属性相同的，根据深度优先方式进行搜索。
* 新式类是采用广度优先算法：当子类继承多个父类的情况时，如果继承的多个父类有属性相同的，根据广度优先方式进行搜素。

### 1.4、Python之禅是什么，Python中如何获取Python之禅？

Python之禅目的是告诉我们如何写出高效整洁的代码。

可以通过 `import this` 来获取Python之禅

```python
import this

# 具体输出的结果就是Python之禅

The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

### 1.5、Python中的DocStrings(解释文档)有什么作用？

DocStrings的主要作用就是解释代码的作用，让他人阅读你的代码时候，能够快速理解代码的作用是什么。

使用 `__doc___` 可查看函数中文档字符串属性

```python
def get_max(x,y):
    '''
    比较取出两个int类型中较大的那个数
    :param x: int类型数字
    :param y: int类型数字
    :return: 输出较大值
    '''
    if x > y:
        return x
    else:
        return y

get_max(3, 5)
print(get_max.__doc__) # 输出解释文档的具体内容
```

### 1.6、Python3中的类型注解有什么好处？如何使用？

Python是一种动态语言，在我们编写代码的时候，我们不需要定义变量的类型以及函数传参的类型。作为作者我们知道应该传递或返回什么类型的数据，但是读者并不能快速的知道。

针对上述的问题就可以使用类型注解。类型注解的好处就是让读者能够快速知道变量或者参数的类型是什么。

```python
# 一般写法
def func(x, y):
  return x + y

# 使用类型注解
def func(x: int, y: int) -> int:
  return x + y
```

### 1.7、Python语言中的命名规范有哪些？

* 基本原则：Python的标识符必须以字母、下划线 开头，数字不能作为开头，后面可以以任意数目的字母、数字和下划线。此处的字母不限于26个英文字母，可以包含中文字符、日文字符等。、
* 禁忌：Python包含的一系列关键字和内置函数，不建议使用它们作为变量名，防止发生冲突。
* 常用命名规则
  * 项目名：首字母大写，其余单词小写。多单词组合则用下划线分割
  * 包名、模块名：全用小写字母
  * 类名：首字母大写，其他字母小写。多单词组合则用驼峰
  * 方法：小写单词。多单词组合则用下划线分割
  * 函数：函数参数名与保留关键词冲突，则在参数后加一个下划线
  * 全局变量：全采用大写。多单词使用下划线分割

### Python中各种下划线的作用？

* 一个前导下划线：表示非公有，也叫做保护变量，表示类对象和子类对象自己才能访问这些变量。采用 `from pkg import *` 的方法导入模块时，被保护的变量不会被导入。
* 一个后缀下划线：为了避免关键字冲突，采用的一种命名方法。
* 两个前导下划线：私有属性，无法在外部直接访问（名字已经被重写）。类对象可以访问，或者使用`类对象._类名__属性名`访问。
* 两个前导和后缀下划线：内置的魔法方法对象或者属性（有特殊的用途）。例如`__init__` 或者 `__name__` 。不要自己创造类似的名称，只需要使用他们即可。

### 1.9、单引号、双引号、三引号有什么区别？

* 单引号和双引号：单独使用单引号和双引号没什么区别，但是如果引号里面还需要使用引号的时候，需要两者配合使用。一般外部使用单引号，内部使用双引号。
* 三引号：三引号分为三单双引号和三双引号，一般用于函数功能描述。两者都可以在声明长字符串时候使用，如果使用docstring就需要使用三单引号。

***

## 2、文件I/O操作

### 2.1、Python中打开文件有哪些模式？

| 模式类型 | 简介                                                         |
| -------- | :----------------------------------------------------------- |
| r        | 以只读方式打开文件，文件的指针会指向文件的开头。             |
| w        | 打开一个文件只用于写入。如果该文件已存在则将其覆盖，如果不存在则创一个新文件。 |
| a        | 打开一个文件用于追加。如果该文件已存在则文件指针会放在文件的结尾。新的内容将会追加到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| rb       | 以二进制格式打开一个文件用于只读。文件指针会指向文件的开头。 |
| wb       | 以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖，如果不存在则创建一个新文件。 |
| ab       | 以二进制格式打开一个文件用于追加。如果该文件已存在则文件指针会放在文件的结尾。新的内容将会追加到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| r+       | 打开一个文件用于读写。文件指针指向文件的开头。               |
| w+       | 打开一个文件用于读写。如果该文件存在则将其覆盖，如果不存在则创建新文件。 |
| a+       | 打开一个文件用于读写。如果该文件存在，则文件指针指向文件的末尾。文件打开时会是追加模式。如果文件不存在，创建新文件用于读写。 |
| rb+      | 以二进制格式打开一个文件用于读写。文件指针指向文件的开头。   |
| wb+      | 以二进制格式打开一个文件用于读写。如果该文件存在则将其覆盖，如果不存在则创建新文件。 |
| ab+      | 以二进制格式打开一个文件用于读写。如果该文件存在，则文件指针指向文件的末尾。文件打开时会是追加模式。如果文件不存在，创建新文件用于读写。 |

### 2.2、Python中read、readline和readlines的区别？

* read：read表示一次性将文件的所有内容都读取到内存之中，如果文件占用空间很大，一次性读取会出现内存不足的问题。针对文件过大导致内存不足的问题，可以使用 `read(size)` 一次最多读取多少字节，读取大文件的时候可以多次调用该方法。
* readline：每次只读一行内容，如果是文本文件则建议使用此方法，逐行读取。
* readlines：和read方法一样，也是读取全部内容，但是该方法返回一个按行读取的列表，其中一个元素代表一行。

### 2.3、大文件只需读取部分内容，或者避免读取时候内存不足的解决方法？

方法一、直接for循环迭代协议，避免内存不足

```python
with open('test.txt', 'r') as fp:
  for line in fp:
    print(line)
```

方法二、使用`islice`进行迭代器切片操作

```python
with open('test.txt', 'r') as fp:
  # islice 返回一个生成器函数，进行切片操作，取出索引为101到105的值
  for line in islice(fp, 101, 105):
    print(line)
  
  # 一个参数表示结束的位置
  for line in islice(fp, 5):
    print(line)
```

