---
title: python学习笔记
categories: python
tags: [python]
---

一直想学习一下python语言，拥有动态语言的特性，还是解释型语言，拥有高级数据结构，可以以简单而高效的方式进行面向对象编程，并且python类库众多，适合写脚本，特别是一些小工具，今天把python语法大概看了一遍，在这里记录学习的过程

<!-- more -->

> 我这里用的是mac，有些操作可能只适用于mac，不适用于windows


## 什么是python
python是一种编程语言，支持高级数据结构，结构化，python是一门解释性语言，无需编译即可运行，支持模块化，类库众多且重用性高，例如基础的IO，Socket，图形界面等，模块化使得python适用于更多领域，绝大多数其他编程语言能做到的python都能做到，甚至支持与二进制库连接起来

## 什么时候使用python
python比较使用于编写工具脚本，例如如果你有一些大量的重复的工作，希望计算机可以自动帮你完成，而又找不到相关的软件可以比较完美的实现，这个时候就可以考虑写一个python脚本来执行，例如：

  * 例如批量扫描文件内容，并替换文件中的文本（执行某些操作）
  * 批量修改文件名
  * 从多个excel文件中提取相关信息并输出到一个excel中
  * 批量抓取网页信息并保存到数据库（爬虫）

> 通常来说就是使用脚本自动完成一些自定义的行为，主要是python支持的库（module）众多，能帮助你节省不少时间  
> 当然如果你是开发人员，可能通过C++/C#/Java来写自动化的工具，但是这些工具可能会显得庞大而臃肿，可能会让人觉得这是烦躁而漫长的工作，那么你可以用python试试

当然python能做的事远不止这些，支持的类库涵盖了很多领域，如人工智能，机器学习，图像处理等，当然，很多功能其他语言也能做到，python最大的好处就是类库众多，我们很多时候都不需要重复造轮子，特别是对于追求效率的你来说，或许是个好选择，

> 语言只是个工具，没有语言是万能的，什么好用用什么

## 安装python
学习python的时候建议使用python 2.x版本，3.x版本相对于2.x改动比较大，并且一些老的库不支持3.x，我这里使用的是`Python 2.7.10`，mac系统自带python，可以通过下面命令查看python的版本

```bash
# 默认情况下使用python查看python2的版本
$ python -V
# 查看python3的版本
$ python3 -V
```

## 工具
与其他脚本语言类似（php, js）使用普通的文本编辑器即可，我这里用的是[Sublime2](http://www.sublimetext.com/2)，当然也可以使用一些集成IDE
  * Atom：与Sublime类似，支持代码高亮，自动补齐
  * PyCharm：JetBrains出的python语言IDE，功能强大，收费
  * Eclipse：有python插件提供支持

如果是刚开始学习，推荐使用文本工具（Sublime或Atom）

## 交互式编程
交互式编程相当于一问一答的方式，在Terminal输入python进入交互式模式，与其他语言不通，python不以分号结尾，并且通过缩进识别代码块
```bash
localhost:~ bomo$ python
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

## 数据类型
python支持如下数据类型
  * [数值类型](https://docs.python.org/2.7/library/functions.html#complex)
    `int`, `long`, `float`, `decimal`数值, `fraction`有理数, `complex`复数
  * [迭代器类型](https://docs.python.org/2.7/library/stdtypes.html#iterator-types)
    迭代器类型包括，列表，字典，元组，字符串
    ```python
    # 列表
    squares = [1, 4, 9, 16, 25]             # 列表
    squares = ['spam', 'eggs', 100, 1234]   # 支持不同类型
    squares[1] = "eeee"                     # 支持索引，越界会报错
    last = squares[-1]                      # 复数表示从最后开始

    # 支持切片
    a = squares[1:3]        # ['eeee', 100]
    squares[1:3] = []       # 删除下标为1-2的项
    squares[-1:100] = []    # 切片范围越界不报错，自动取临界值
    squares                 # ['spam', 1234]

    # 支持运算符 +, *
    squares + ['a', 'b']    # ['spam', 1234, 'a', 'b']
    l = squares * 2         # ['spam', 1234, 'a', 'b', 'spam', 1234, 'a', 'b']
    ```

    字符串是特殊的列表
    ```python
    print '"Yes," he said.'               # 使用单双引号表示单行字符串："..." '...'
    print 'First line.\nSecond line.'     # 使用反斜杠`\`转义，

    print "C:\some\name"      # 换行
    print r'C:\some\name'     # 字符串前面添加`r`表示`\`不转义

    # 使用三个引号定义多行字符串 """...""" '''...'''
    print """\          # 行尾使用反斜杠`\`表示该行后面接下一行的开头
    aa
    bb
    """                 # 结束
    ```

    迭代器类型更多说明见[序列](#序列)

## 运算符
  * `+` `-` `*` `/` `%`
  * `//`, `**`
    ```python
    # 取整（不大于该数）
    11 // 3.0     # 3.0
    11 // -3      # -4

    # 幂运算，与其他语言的`^`类似
    5 ** 2        # 25
    ```
  * `in`, `not in`
    比较运算符，判断区间或集合是否存在某项
  * `is`, `is not`
    判断两个对象是否相同
    > is 与 == 的区别
    ```python
    a = 1
    b = 1.0
    a == b    # True
    a is b    # False
    ```


  在交互运算中，python会把最近一次的表达式的值赋值给`_`，__该变量是只读的，不要尝试给其赋值__，
  ```python
  a = 10
  10 + 32     # 42
  a + _       # 52
  ```

## 流程控制与循环
python的流程控制跟其他差不多也是 `if-else`, `for`, `while`, `break`, `continue`, `pass`
### if-else
```python
if x == 0:
    print 'Zero'
elif x == 1:
    print '1'
else:
    print 'Other'
```

### for
```python
words = ['mac', 'windows', 'linux']
for w in words:
   print w

# 支持切片
for w in words[1:3]:
   print w

for i in range(len(words)):
   print a[i]

# range函数
range(10)         # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
range(5, 10)      # [5, 6, 7, 8, 9]
range(0, 12, 3)   # [0, 3, 6, 9]
```

### pass
空语句，与C语言的分号`;`类似，什么都不执行

* 注意for, if-else, while 后面都有冒号`:`
* python没有`do-while`语句
* python没有`switch-case`，也可以参考：[http://blog.sina.com.cn/s/blog_6409e7eb01018chn.html](http://blog.sina.com.cn/s/blog_6409e7eb01018chn.html)


## 函数
### 函数定义
形式
```
def 函数名(函数参数列表):
  函数体
```
eg: 输出一个Fibonacci序列
```python
def fib(n):
   a, b = 0, 1
   while a < n:
       print b,
       a, b = b, a + b

fib(2000)             # 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597
```

### 默认参数
```python
i = 10
def test(a = 12, b = i):
  print b
i = 11;               # i为不可变对象，这个时候test函数的参数b的默认值仍为10

test(1,2)
test(a = 1, b = 2)    # 也可以指定参数
```

### 可变参数
```python
def test(a, *b):
   print a
   for i in b
     print i
test('hello', 'bob', 'james', 'adophi')     # 'hello', 'bob', 'james', 'adophi'
```

### 可变字典参数
```python
def test(a, *b, **c):
   keys = sorted(c.keys())
     for kw in keys:
       print kw, ":", keywords[kw]
test('box', 'value1', 'value2', width=100, height=200, color="red")
```

### lambda表达式
形式
```
lambda 函数参数列表: 函数体
```
eg: 使用Lambda表达式给列表排序

```python
pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]
pairs.sort(key = lambda pair: pair[1])
pairs
[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]
```

## 序列
### 常见函数
```python
squares = ['bomo', 'tobi', 'andy', 'leo']

squares.append('jobs')                    # 添加
squares.extend(['a', 'b', 'c', 'd'])      # 添加集合
squares.insert(0, x)                      # 插入元素
squares.remove('b')                       # 删除值为'b'的第一个元素
squares.pop([1])                          # 删除第1个元素，并返回该元素
squares.index('b')                        # 获取值为'b'的索引
squares.count('b')                        # 获取值为'b'的出现次数
```

更多函数参见：[`TODO`]()

### 列表推倒式
形式：`[expression for value in collection if condition]`  
相当于
```python
result = []  
for value in collection:  
    if condition:  
        result.append(expression)  
```
例如
```python
[i for i in range(1,100) if i > 90]         # [91, 92, 93, 94, 95, 96, 97, 98, 99]
```

### 元组tuple
元组相当于多维数组，用括号表示
```python
t = (1, 2)
t = ('a', 23)
t = (['a', 1], ['b', 2])

# 空元组
empty = ()

# 单元素元组
single = (11,)
single = 11,
```

元组各个元素可以同时赋值
```python
t = (1, 2)
x, y = t        # x = 1, y = 2
x, y = 1, 2     # 同上
```

> 与列表不同，元组是不可变的，只能链接和切割生成新的元组

### 字典dict
```python
tel = {'jack': 4098, 'sape': 4139}
tel['guido'] = 4127
tel = dict(jack=4098, sape=4139)

# 空字典
tel = {}
```

### 字典推倒式
`[key: value for value in tuple]`  
```python
{x: x**2 for x in (2, 4, 6)}      # {2: 4, 4: 16, 6: 36}
```

### 无序集set
```python
basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
fruit = set(basket)               # ['orange', 'pear', 'apple', 'banana']
```

### 集合操作
__del操作__
删除集合元素
```python
a = [-1, 1, 66.25, 333, 333, 1234.5]
del a[0]            # [1, 66.25, 333, 333, 1234.5]
del a[2:]           # [1, 66.25]
```

删除模块属性/函数
```python
class test:
  a = 10
  def f(self, i):
    print i

t = test()
t.a                     # 10
t.f('hello world')      # hello world

del test.a              # remove property a
del test.f              # remove method f

t.a                     # 报错：test instance has no attribute 'a'
t.f('hello world')      # 报错：test instance has no attribute 'f'
```

__与或操作__
```python
a = set('abracadabra')      # set(['a', 'r', 'b', 'c', 'd'])
b = set('alacazam')         # set(['a', 'c', 'z', 'm', 'l'])

a - b                       # set(['r', 'd', 'b'])
a | b                       # set(['a', 'c', 'b', 'd', 'm', 'l', 'r', 'z'])
a & b                       # set(['a', 'c'])
a ^ b                       # set(['r', 'd', 'b', 'm', 'z', 'l'])
```

__enumerate__  
使用enumerate函数可以同时得到索引和值
```python
for i, v in enumerate(['tic', 'tac', 'toe']):
   print(i, v)
```

__zip__
多个循环可以用zip打包

```python
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers):
   print 'What is your {0}?  It is {1}.'.format(q, a)

# 输出
# What is your name?  It is lancelot.
# What is your quest?  It is the holy grail.
# What is your favorite color?  It is blue.
```

__reversed__
反序
```python
for i in reversed([1,2,3]):
   print(i)       
# 输出：3, 2, 1
>>>
```

__iteritems__
遍历字典使用iteritems可以同事获得key, value
```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.iteritems():
   print k, v
# 输出
# gallahad the pure
# robin the brave
```

### 三个函数`filter`, `map`, `reduce`
* filter(function, sequence)
  过滤器，给定过滤函数和集合
  ```python
  def f(x):
    return x % 3 == 0 or x % 5 == 0
  filter(f, range(2, 25))     # [3, 5, 6, 9, 10, 12, 15, 18, 20, 21, 24]
  ```

* map(function, sequence)
  给定集合和操作函数，返回操作后的集合
  ```python
  def cube(x):
    return x*x
  map(cube, range(1, 6))      # [1, 4, 9, 16, 25, 36]
  ```

* reduce(function, sequence)
  首先操作前两个数，然后结果与第三个数操作，以此类推
  ```python
  def add(x,y):
    return x+y
  reduce(add, range(1, 11))   # 55
  ```

## 迭代器
迭代器统一了所有集合访问元素的方式，包括有序无序的，相比for遍历集合，其支持随机访问的集合，如`set`，`dict`，只能向前，不能后退，迭代器有下面两个基本方法
* `next`：返回迭代器下一个元素
* `__iter__`方法：返回一个迭代器

定义一个迭代器
```python
class Fabs(object):
    def __init__(self, max):
      self.max = max;
      self.n, self.a, self.b = 0, 0, 1

    def __iter__(self):
      return self;

    def next(self):
      if self.n < self.max:
        r = self.b
        self.a, self.b = self.b, self.a + self.b
        self.n = self.n + 1
        return r;

for i in Fabs(30):
  print i             # 1, 1, 2, 3, 5, 8, 13, 21
```

## 生成器


## 注释
### 普通注释

python注释以`#`开头
```python
# this is the first comment
SPAM = 1   # and this is the second comment
STRING = "# This is not a comment."
```

python多行注释使用三个引号`'''`或`"""`
```python
'''
这是一个多行注释
'''
```
> 三引号容易和多行字符串混淆，通常使用`#`开头表示注释就好了

### 文档注释
文档注释跟在相应的定义后面：函数定义，类定义，文件定义（文件头）
```python
#!usr/bin/env python

"""foo.py -- 模块注释"""

class Foo(object):
    """这里是类注释，这是一个类为空"""

def printmsg(msg):
    """这里是方法注释"""
    print msg
```

可以通过help或`__doc__`访问注释内容
```python
# 列出整个模块的所有注释
help(foo)

# 列出类Foo的类注释
Foo.__doc__
```

## 模块module, 类class, 包package
> python在处理功能复用组织结构切分为模块,包和面向对象的类，其结构类似于C#/Java的命名空间，用于

### 引用模块
```python
import math             # 引用math模块
print math.sqrt(2)      # 正确输出
print sqrt(2)           # 报错，未导入sqrt函数

##########################

import math as nmath    # 引用math模块，并取别名nmath
print nmath.sqrt(2)     # 正确输出
print math.sqrt(2)      # 报错，未找到math的定义

##########################

from math import sqrt   # 导入模块中的函数
print sqrt(2)           # 正确输出
print math.sqrt(2)      # 会报错，没有导入math

##########################

from math import *      # 不导入单下划线开头的函数和属性，不建议这么做，这么写语义不明确，并且在多模块中存在冲突的风险
print sqrt(2)
```

* 文件名就是模块的名字：`test.py`的模块名为`test`
* 第一次导入模块的时候会自动执行模块内的所有代码
* 通过python执行py模块的时候`__name__`变量会被设为`__main__`，如果是import，则不会，可以通过`__name__`变量判断是否是命令行执行
* 使用下划线`_`前缀定义类私有函数或变量
  1. `_xxx`        单下划线开头，在`from module import *`导入会忽略
  2. `__xxx__`     双下划线开头和结尾，系统定义名字
  3. `__xxx`       双下划线开头，类中的私有变量名

### dir函数
可以通过`dir`函数查看模块内定义的所有变量和函数
```python
import math
dir(math)
```

### 模块搜索路径
在导入模块的时候，python会自动从以下目录搜索该模块是否存在
* 输入脚本的目录(当前目录)。
* 环境变量 PYTHONPATH 表示的目录列表中搜索
* Python 默认安装路径中搜索

可以通过`sys.path`查看所有目录
```python
import sys
sys.path
```

### package包
多个模块组织成一个package，类似于java的jar包，C#的dll可执行文件，包在python中相当于一个文件夹，包文件夹包含`__init__.py`文件，该文件用于初始化package
```
sound/                        Top-level package
    __init__.py               Initialize the sound package
    formats/                  Subpackage for file format conversions
            __init__.py
            wavread.py
            wavwrite.py
            aiffread.py
```
使用package方式与module类似
```python
# 使用wavread需要引用sound.format
import sound.format.wavread

# 可以直接使用wavread
from wavread import sound.format
```

#### `__init__.py`文件
`__init__.py`文件可以包含一些包初始化的内容，可以定义默认导入的模块
```python
__all__ = ["wavread", "wavwrite"]
```
在使用`from sound.formats import *`的时候只会导入`wavread`, `wavwrite`两个模块，而不会导入`aiffread`模块

#### 包内引用
```python
from . import echo                # 当前包同目录下的echo模块
from .. import formats            # 当前包上级目录下的formats模块
from ..filters import equalizer   # 当前包上级目录下的filters目录下的equalizer模块
```


## 类
### 构造函数`__init__`
```python
class test:
  def __init__(self, x, y):
    self.x = x
    self.y = y

t = test(10, 20)
t.x + t.y
```

### 方法与变量
类变量定义在类中，实例变量定义在构造函数中，python中的类实例可以直接设置属性，如果属性不存在，则添加属性
```python
class test:  
  count = 0;                                                # 定义类变量   
  def __init__(self, c):                                    # 构造函数
    self.count = c;                                         # 定义实例变量count  
    self.name = 'bomo'                                      # 定义实例变量name
    self.__class__.count = self.__class__.count + 1;        # 操作类变量

  def sayHi(self):                                          # 实例方法
    print 'Hello, your name is?',self.name

  @staticmethod                       #声明静态，去掉则编译报错;还有静态方法不能访问类变量和实例变量
  def sayName():
      print "my name is king"



t = test(10)      
test.count        # 1
tt = test(20)
test.count        # 2
t.count           # 10
tt.count          # 20
```
### 实例方法，类方法，静态方法
在python中这几种方法特别容易
类属性和方法不能重名，否则会相互覆盖

```python
class Person:
    staticName = 'bomo'
    def __init__(self,name):
        self.name = name
    def sayHi(self):                                  # 实例方法加上self参数
        print 'Hello ' + self.name

    @staticmethod                                     # 声明静态方法，下面方法都为静态方法
    def sayName():                                    # 没有self参数
        print "static method" + Person.staticName     # 静态方法通过类名调用静态实例staticName

    @classmethod                                      # 声明类方法，下面方法都为类方法
    def classMethod(cls):                             # 有一个cls参数，类对象
        print "class method" + Person.staticName
```

其他语言如java和C#都只有静态方法和类方法，而python多了一个类方法，在大多数情况下，使用静态方法即可，在需要获取调用类的信息的时候，则需要使用类方法

> 方法的第一个参数被命名为 self。这仅仅是一个约定：对 Python 而言，名称 self 绝对没有任何特殊含义(但是请注意：如果不遵循这个约定，对其他的 Python 程序员而言你的代码可读性就会变差，而且有些类查看器程序也可能是遵循此约定编写的。)，在交互式命令行中会报错

### 继承
父类放在子类定义的类名后的括号内，python支持多继承
```python
class DerivedClassName(Base1, Base2, Base3):
  pass
class son(father):
  pass
```  

判断实例与继承关系
```python
s = son()
# 判断实例
isinstance(s, son)        # True
isinstance(s, father)     # True

# 判断继承关系
issubclass(son, father)   # True, son继承自father
```

## 异常
python的异常处理和其他语言类似:`try-except-finally`，抛出异常使用`raise`
```python
try:
    raise Exception('spam', 'eggs')
except Exception as inst:   # 设置异常实例别名，用于引用
    print type(inst)        # 获取异常类型
    print inst.args         # 获取异常实例
finally:                    # 无论是否抛出异常，都会执行，与JAVA,C#其他语言一样
    print "executing finally clause"
```

##


详情参见：TODO：
