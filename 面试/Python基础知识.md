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

