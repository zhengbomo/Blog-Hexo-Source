---
title: python学习笔记
categories: python
tags:
  - python
date: 2016-06-14 20:01:29
updated: 2016-06-14 20:01:29
---

一直想学习一下Python语言，拥有动态语言的特性，还是解释型语言，拥有高级数据结构，可以以简单而高效的方式进行面向对象编程，并且Python类库众多，适合写脚本，特别是一些小工具，最近把Python语法大概捋了一遍，在这里记录学习的过程

<!-- more -->

# 一、简介
## 1. 什么是Python
Python是一种编程语言，支持高级数据结构，结构化，Python是一门解释性语言，无需编译即可运行，支持模块化，类库众多且重用性高，例如基础的IO，Socket，图形界面等，模块化使得Python适用于更多领域，绝大多数其他编程语言能做到的Python都能做到，甚至支持与二进制库连接起来，有一句话是这么说的：人生苦短，我用Python

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-6-14/69651795.jpg)

## 2. 什么时候使用Python
pPython比较使用于编写工具脚本，例如如果你有一些大量的重复的工作，希望计算机可以自动帮你完成，而又找不到相关的软件可以比较完美的实现，这个时候就可以考虑写一个Python脚本来执行，例如：

  * 例如批量扫描文件内容，并替换文件中的文本（执行某些操作）
  * 批量修改文件名
  * 从多个excel文件中提取相关信息并输出到一个excel中
  * 批量抓取网页信息并保存到数据库（爬虫）

> 通常来说就是使用脚本自动完成一些自定义的行为，主要是Python支持的库（module）众多，能帮助你节省不少时间
> 当然如果你是开发人员，可能通过C++/C#/Java来写自动化的工具，但是这些工具可能会显得庞大而臃肿，可能会让人觉得这是烦躁而漫长的工作，那么你可以用Python试试

当然Python能做的事远不止这些，支持的类库涵盖了很多领域，如人工智能，机器学习，图像处理，金融，物理学等，当然，很多功能其他语言也能做到，Python最大的优势就是类库众多，处理灵活，我们很多时候都不需要重复造轮子，特别是对于追求效率的你来说，或许是个好选择，

> 语言只是个工具，没有语言是万能的，什么好用用什么

## 3. 安装Python
学习Python的时候建议使用Python 2.x版本，大多第三方库都基于2.x版本，3.x版本相对于2.x改动比较大，并且一些老的库不支持3.x，我这里使用的是`Python 2.7.10`，mac系统自带Python，可以通过下面命令查看Python的版本

```bash
# 默认情况下使用Python查看Python2的版本
$ Python -V
# 查看Python3的版本
$ Python3 -V
```

## 4. 工具
与其他脚本语言类似（php, js）使用普通的文本编辑器即可，我这里用的是[Sublime2](http://www.sublimetext.com/2)，当然也可以使用一些集成IDE
  * Atom：与Sublime类似，支持代码高亮，自动补齐
  * PyCharm：JetBrains出的Python语言IDE，功能强大，收费
  * Eclipse：有Python插件提供支持

如果是刚开始学习，推荐使用文本工具（Sublime或Atom）

## 5. 交互式编程
交互式编程相当于一问一答的方式，在Terminal输入Python进入交互式模式，与其他语言不同，Python不以分号结尾，并且通过缩进识别代码块
```bash
localhost:~ bomo$ Python
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

# 二、基础与语法
## 1. 数据类型
与其他高级语言一样，除了基础数据类型，Python也支持高级数据类型
Python支持如下数据类型
  * [数值类型](https://docs.Python.org/2.7/library/functions.html#complex)
    `int`, `long`, `float`, `decimal`数值, `fraction`有理数, `complex`复数
  * [迭代器类型](https://docs.Python.org/2.7/library/stdtypes.html#iterator-types)
    迭代器类型包括，list, dict, tuple, str, set等

## 2. 运算符
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
    a, b = 1, 1.0
    a == b    # True
    a is b    # False
    ```

  python（2.5+）也支持三元运算符，用法与C语言有点出入，形式为
  ```python
  x if x > y else y     # 等价于C语言的 x > y ? x : y
  ```

  在交互运算中，Python会把最近一次的表达式的值赋值给`_`，__该变量是只读的，不要尝试给其赋值__，一些不需要的变量有时也用`_`表示（其实跟普通的变量i一样），如for
  ```python
  a = 10
  10 + 32     # 42
  a + _       # 52

  for _ in range(0, 10):
      print '这里不需要用到迭代的值'
  ```

## 3. 流程控制与循环
Python的流程控制跟其他语言差不多，关键字有：`if-else`, `for`, `while`, `break`, `continue`, `pass`
### 3.1 if-else
```python
if x == 0:
    print 'Zero'
    print 'ff'
elif x == 1:
    print '1'
else:
    print 'Other'
```

### 3.2 for
```python
words = ['mac', 'windows', 'linux']
for w in words:
   print w
```

### 3.3 pass
空语句，与C语言的分号`;`类似，什么都不执行，通常用来表示一个空的代码块

* 注意for, if-else, while 后面都有冒号`:`
* Python使用缩进来识别块（其他语言多数使用大括号来识别代码块）
* Python没有`do-while`语句
* Python没有`switch-case`，也可以参考：[http://blog.sina.com.cn/s/blog_6409e7eb01018chn.html](http://blog.sina.com.cn/s/blog_6409e7eb01018chn.html)


## 4. 函数
### 4.1 函数定义
形式
```
def 函数名(函数参数列表):
  函数体
```
eg: 输出一个Fibonacci斐波那契序列
```python
def fib(n):
  a, b = 0, 1
  while a < n:
       print b,
       a, b = b, a + b

fib(2000)             # 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597
```

### 4.2 参数默认值
```python
i = 10
def test(a = 12, b = i):
  print b
i = 11;               # i为不可变对象，这个时候test函数的参数b的默认值仍为10

test(1,2)
test(b = 2, a = 12)    # 也可以指定参数
```

从上面例子可以看到，参数的默认值在解释器运行到函数的时候就确定了，如上面的变量`b`，后面修改的的变量i与函数test没有关联了，有一点需要注意的是，如果默认参数是一个对象的时候，也是同样的，如下面例子
```python
def test(data=[]):      # 默认参数为空list
    data.append(1)
    return data

print test()        # 输出：[1]
print test()        # 输出：[1, 1]
print test()        # 输出：[1, 1, 1]
```
Python解释器在解释道函数test的时候就确定好了函数test的参数默认值，所以每次调用使用的参数对象都是同一个，上面函数应该定义为如下方式
```python
def test(data=None):      # 默认参数为空list
    if data is None:
        data = []
    data.append(1)
    return data
```

### 4.3 可变参数
```python
def test(a, *b):
   print a
   for i in b
     print i

test('hello', 'bob', 'james', 'adophi')     # 'hello', 'bob', 'james', 'adophi'
```
> 可变参数必须定义为最后一个（在可变字典参数前）

### 4.4 可变字典参数
```python
def test(a, *b, **c):
   keys = sorted(c.keys())
     for kw in keys:
       print kw, ":", keywords[kw]


test('box', 'value1', 'value2', width=100, height=200, color="red", rttt = 21)
```
> 可变字典参数必须定义在最后一个

### 4.5. lambda表达式
形式
```
lambda 函数参数列表: 函数体
```
eg: 使用Lambda表达式给列表排序

```python
pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]


pairs.sort(key = lambda pair: pair[0])
pairs
[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]
```
## 5. 装饰器
参见[这里](/2016-06-17/Python-decorator)

## 6. 序列
首先看看迭代器类型，Python常用的序列类型有list, tuple, str
```python
squares = [1, 4, 9, 16, 25]             # 列表
squares = ['spam', 'eggs', 100, 1234]   # 支持不同类型
squares[1] = "eeee"                     # 支持索引，越界会报错
last = squares[-1]                      # 复数表示从最后开始

## 支持切片（支持list,tuple,str）
squares[a:b:c]          # 表示从下标a到下标b且不包含下标b，取步长为c的集合，步长为正向右，步长为负向左

a = squares[1:3]        # ['eeee', 100]
squares[1:3] = []       # 删除下标为1-2的项
squares[-2:100] = []    # 切片范围越界不报错，自动取临界值
squares                 # ['spam', 1234]
squares[1:5:2]          # 支持步长，每次进两个元素，默认为一个[4, 16]
squares[5:1:-2]         # 步长为负数，表示从后往前：[25, 9]

# range函数
range(10)         # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
range(5, 10)      # [5, 6, 7, 8, 9]
range(0, 12, 3)   # [0, 3, 6, 9]
```

### 6.1. 字符串
字符串是特殊的列表
```python
print '"Yes," he said.'               # 使用单双引号表示单行字符串："..." '...'
print 'First line.\nSecond line.'     # 使用反斜杠`\`转义，

print "C:\some\name"      # 换行
print r'C:\some\name'     # 字符串前面添加`r`表示`\`不转义

# 使用三个引号定义多行字符串 """...""" '''...'''
print """\          # 行尾使用反斜杠`\`表示该行后面接下一行
aafdsafds
bb
"""                 # 结束
```

格式化字符串，Python使用`%`隔开格式化字符串和值
```python
from math import pi
print 'Hello %s, pi is %10.2f' % ('World', pi)
# Hello World, pi is 3.14
```

当像`%s`这种符号变多了之后，就很难分清楚哪个和哪个了，Python支持使用dict作为格式化参数，如下
```python
print '%(name)s is %(age)d years old' % {'name': 'bomo', 'age': 18}
# bomo is 18 years old
```
还有这种方式
```python
print '{name} is {age:2d} years old'.format(name='bomo', age=18)
# bomo is 18 years old
```
Python格式化字符串还支持索引
```python
print '{name} is {1:2d} years old'.format(12, 18, name='bomo')
# bomo is 18 years old
```
最后一种方式是最Pythonic的，也是最推荐的方式

### 6.2 序列常用操作
1. 加法乘法
```python
squares + ['a', 'b']    # ['spam', 1234, 'a', 'b']
l = squares * 2         # ['spam', 1234, 'a', 'b', 'spam', 1234, 'a', 'b']
```

2. 元素操作
```python
squares.append('jobs')                    # 添加
squares.extend(['a', 'b', 'c', 'd'])      # 添加集合
squares.insert(0, x)                      # 插入元素
squares.remove('b')                       # 删除值为'b'的第一个元素
squares.pop([1])                          # 删除第1个元素，并返回该元素
squares.index('b')                        # 获取值为'b'的索引
squares.count('b')                        # 获取值为'b'的出现次数
```

3. 统计操作
```python
len(squares)
max(squares)
min(squares)
```

4. 删除操作
```python
a = [-1, 1, 66.25, 333, 333, 1234.5]
del a[0]            # [1, 66.25, 333, 333, 1234.5]
del a[2:]           # [1, 66.25]
```

### 6.3 切片的原理
切片内部是调用`__getitem__`，`__setitem__`,`__delitem__`和`slice`函数
```python
a = [1, 2, 3, 4, 5, 6]
x = a[1: 5]             # x = a.__getitem__(slice( 1, 5, None ))
a[1: 3] = [10, 11, 12]  # a.__setitem__(slice(1, 3, None), [10, 11, 12])
del a[1: 4]         # a.__delitem__(slice(1, 4, None))
```

## 7. 元组tuple
元组相当于多维数组，与序列一样，元组元素支持任意类型，用括号表示，括号有时可以省略（不产生歧义的情况下）
```python
t = (1, 2)
t = 1, 2            # 省略括号
t = ('a', 23)
t = (['a', 1], ['b', 2])

# 空元组
empty = ()

# 单元素元组，逗号不能少
single = (11,)
single = 11,

# 如sqlite操作的时候
cursor.execute('select * from user where gender = ?',
              ('female',))
```

元组各个元素可以同时赋值
```python
t = (1, 2)
x, y = t        # x = 1, y = 2
x, y = 1, 2     # 同上
```

元组使得函数返回多个值变得更加方便
```python
def get_point():
    return (2, 4)
```

> 与列表不同，元组是不可变的，如果想修改，只能生成新的元组

## 8. 字典dict
用法与其他语言类似
```python
tel = {'jack': 4098, 'sape': 4139, 'bomo': 10086}
tel['guido'] = 4127
del d['bomo']
tel = dict(jack=4098, sape=4139)

# 空字典
tel = {}

# 迭代
for key in tel:
    print key

for value in tel.itervalues():
    print value

for (key, value) in tel.iteritems():
    print 'key=%s, value=%s' % (key, value)
```

### 列表推倒式和字典推导式
1. 列表推导式
形式：`[expression for value in collection if condition]`，相当于
```python
result = []
for value in collection:
    if condition:
        result.append(expression)
return result
```
例如
```python
[i for i in range(1,100) if i > 90]         # [91, 92, 93, 94, 95, 96, 97, 98, 99]
```

2. 字典推导式
形式：`{key_expression: value_expression for value in tuple}` ，相当于
  ```python
  result = {}
  for value in collection:
      if condition:
          result[key_expression] = value_expression
  return result
  ```
  例如：
  ```python
  {x+1: x**2 for x in (2, 4, 6, 8) if x <= 6}      # {3: 4, 5: 16, 7: 36}
  ```

## 9. 无序集set
```python
basket = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
fruit = set(basket)               # ['orange', 'pear', 'apple', 'banana']

# 操作符（自动去重）
a = set('abracadabra')      # set(['a', 'r', 'b', 'c', 'd'])
b = set('alacazam')         # set(['a', 'c', 'z', 'm', 'l'])

a - b                       # set(['r', 'd', 'b'])
a | b                       # set(['a', 'c', 'b', 'd', 'm', 'l', 'r', 'z'])
a & b                       # set(['a', 'c'])
a ^ b                       # set(['r', 'd', 'b', 'm', 'z', 'l'])
```

## 10. 迭代器类型常用操作
* __enumerate__
使用enumerate函数可以同时得到索引和值
```python
for i, v in enumerate(['tic', 'tac', 'toe']):
   print(i, v)
```

* __zip__
多个循环可以用zip打包，以最短的list为准
```python
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot']
cc = ['lancelot', 'the holy grail', 'blue', 'eeeee']
for q, a, c in zip(questions, answers, cc):
   print 'What is your {0}?  It is {1}.{2}'.format(q, a, c)

# 输出
# What is your name?  It is lancelot.
# What is your quest?  It is the holy grail.
# What is your favorite color?  It is blue.
```

* __reversed__
反序
```python
for i in reversed([1,2,3]):
   print(i)
# 输出：3, 2, 1
>>>
```

* __iteritems__
遍历字典使用iteritems可以同事获得key, value
```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.iteritems():
   print k, v
# 输出
# gallahad the pure
# robin the brave
```

* __filter__
过滤器，给定过滤函数和集合: `filter(function, sequence)`
```python
def f(x):
  return x % 3 == 0 or x % 5 == 0
filter(f, range(2, 25))     # [3, 5, 6, 9, 10, 12, 15, 18, 20, 21, 24]
```

* __map__
给定集合和操作函数，返回操作后的集合：`map(function, sequence)`
```python
def cube(x):
  return x*x
map(i, range(1, 6))      # [1, 4, 9, 16, 25, 36]
```

* __reduce__
首先操作前两个数，然后结果与后一个数运算，以此类推：`reduce(function, sequence)`
```python
def add(x,y):
    return x+y
reduce(add, range(1, 11))   # 1+2+3+4+5+6+7+8+9+10=55

reduce(lambda (x, y): x + y, range(1, 11))   # 55
```

## 11. 迭代器与生成器
参见[这里](/2016-06-21/python-iterator-generator/)

## 12 类
### 12.1 构造函数`__init__`，析构函数`__del__`
```python
class test:
  def __init__(self, x, y):
    self.x = x
    self.y = y

  def __del__(self):
    print '对象被释放'

t = test(10, 20)
t.x + t.y
```
> 注意，Python的构造函数不支持重载，只能有一个构造函数，可以通过可变参数实现多构造函数
> 在调用父类的构造方法见后面新式类和经典类

### 12.2 方法与变量
类变量定义在类中，实例变量定义在构造函数中，Python中的类实例可以直接设置属性，如果属性不存在，则添加属性
```python
class test:
  count = 0;                                                # 定义类变量
  def __init__(self, c):                                    # 构造函数
    self.count = c;                                         # 定义实例变量count
    self.name = 'bomo'                                      # 定义实例变量name
    self.__class__.count = self.__class__.count + 1;        # 操作类变量

  def sayHi(self):                                          # 实例方法
    print 'Hello, your name is?',self.name

  @staticmethod                       #声明静态，去掉则编译报错;还有静态方法不能直接访问类变量和实例变量
  def sayName():
      print "my name is king"

  @classmethod              # 静态变量无法直接访问到类变量，类方法可以，通过cls参数
  def test2(cls):
      print cls

t = test(10)
test.count        # 1
tt = test(20)
test.count        # 2
t.count           # 10
tt.count          # 20
```

### 12.3 类里面引用全局变量
当类里面需要引用外部的全局变量的时候需要，加上global关键字
```python
global_count = 0
global_count2 = 0

def test():
    global global_count   # 声明全局变量
    global_count += 10

    global_count2 = 10    # 局部变量
    print '%d, %d' % (global_count, global_count2)

print '%d, %d' % (global_count, global_count2)
test()
print '%d, %d' % (global_count, global_count2)
```

### 12.4 实例方法，类方法，静态方法
在Python中这几种方法特别容易，类属性和方法不能重名，否则会相互覆盖

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

其他语言如java和C#都只有静态方法和类方法，而Python多了一个类方法，在大多数情况下，使用静态方法即可，在需要获取调用类的信息的时候（如类变量，类类型，类名等），则需要使用类方法

> 方法的第一个参数被命名为 self。这仅仅是一个约定：对 Python 而言，名称 self 绝对没有任何特殊含义(但是请注意：如果不遵循这个约定，对其他的 Python 程序员而言你的代码可读性就会变差，而且有些类查看器程序也可能是遵循此约定编写的。)，在交互式命令行中会报错

### 12.5 继承
父类放在子类定义的类名后的括号内，Python支持多继承
```python
class DerivedClassName(Base1, Base2, Base3):
  pass
class son(father):
  pass
```

> 单继承：如果子类有自己实现的构造函数，则不会自动调用父类的构造函数，如果子类没有实现构造函数，则会继承父类的构造函数
> 多继承：如果子类有自己实现的构造函数，同单继承，不会主动调用父类的构造函数，如果子类没有实现自己的构造函数，则会从父类中优先选择有构造函数的父类（__深度搜索__）

判断实例与继承关系
```python
s = son()
# 判断实例
isinstance(s, son)        # True
isinstance(s, father)     # True

# 判断继承关系
issubclass(son, father)   # True, son继承自father
isinstance(obj, Class)    # 判断实例是否是某个类
```

### 12.6 新式类和经典类
新式类：从object类继承的类，继承顺序广度优先
经典类：不从object继承的类，继承顺序深度优先，不支持super
```python
# 新式类
class Parent(object):
    def __init__(self):
        print 'parent'
    def hi(self):
        print 'parent hi'

class Son(Parent):
    def __init__(self):
        # 新式类支持super
        super(Son, self).__init__()
        print 'son'
    def hi(self):
        super(Son, self).hi()
        print 'son hi'

# 经典类
class Parent:
    def __init__(self):
        print 'parent'
    def hi(self):
        print 'parent hi'

class Son(Parent):
    def __init__(self):
        # 不支持super
        Parent.__init__(self)
        print 'son'
    def hi(self):
        Parent.hi(self)
        print 'son hi'
```
子类可以通过`super(类, 对象)`获取父对象父类对象的实例，然后可以调用父类的（同名）方法，就相当于Java中的`super`，C#中的`base`

## 13. 模块module, 包package
> Python在处理功能复用组织结构切分为模块,包和面向对象的类，其结构类似于C#/Java的命名空间，用于

### 13.1 引用模块
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
* 通过Python执行py模块的时候`__name__`变量会被设为`__main__`，如果是import，则不会，可以通过`__name__`变量判断是否是命令行执行
* 使用下划线`_`前缀定义类私有函数或变量
  1. `_xxx`        单下划线开头，在`from module import *`导入会忽略
  2. `__xxx__`     双下划线开头和结尾，系统定义名字
  3. `__xxx`       双下划线开头，类中的私有变量名

### 13.2 dir函数查看模块
可以通过`dir`函数查看模块内定义的所有变量和函数
```python
import math
dir(math)
```

### 13.3 模块搜索路径
在导入模块的时候，Python会自动从以下目录搜索该模块是否存在
* 输入脚本的目录(当前目录)。
* 环境变量 PYTHONPATH 表示的目录列表中搜索
* Python 默认安装路径中搜索

可以通过`sys.path`查看所有目录
```python
import sys
sys.path
```

### 13.4 package包
多个模块组织成一个package，类似于java的jar包，C#的dll可执行文件，包在Python中相当于一个文件夹，包文件夹包含`__init__.py`文件，该文件用于初始化package
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

### 13.5 `__init__.py`文件
`__init__.py`文件可以包含一些包初始化的内容，可以定义默认导入的模块
```python
__all__ = ["wavread", "wavwrite"]
```
在使用`from sound.formats import *`的时候只会导入`wavread`, `wavwrite`两个模块，而不会导入`aiffread`模块

* 导入包的时候，先初始化`__init__.py`文件，再初始化模块文件
*

### 13.6 包内引用
```python
from . import echo                # 当前包同目录下的echo模块
from .. import formats            # 当前包上级目录下的formats模块
from ..filters import equalizer   # 当前包上级目录下的filters目录下的equalizer模块
```


### 13.7 模块操作
删除模块属性/函数
```python
class test:
  a = 10
  def f(self, i):
    print i

t = test()
t.a = 10                # 10
t.f('hello world')      # hello world

del test.a              # remove property a
del test.f              # remove method f

t.a                     # 报错：test instance has no attribute 'a'
t.f('hello world')      # 报错：test instance has no attribute 'f'
```
Python算是动态语言，属性和函数即写即用，不需要提前定义好，模块也可以即时导入即时卸载

## 14. 文档注释
文档注释跟在相应的定义后面：函数定义，类定义，文件定义（文件头）
```python
#!usr/bin/env Python
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

## 15. 异常
Python的异常处理和其他语言类似:`try-except-finally`，抛出异常使用`raise`
```python
try:
    raise Exception('spam', 'eggs')   # 手动抛出异常
except Exception as inst:             # 设置异常实例别名，用于引用
    print type(inst)                  # 获取异常类型
    print inst.args                   # 获取异常实例
finally:                              # 无论是否抛出异常，都会执行，与JAVA,C#其他语言一样
    print "始终会执行"
```

## 16. 多线程
参见[这里](/2016-06-14/Python-multithread/)


## 17. Python常用模块
### time
```python
import time
import datetime

# 格式化输出当前本地时间：[https://docs.Python.org/2/library/time.html#time.strftime](https://docs.Python.org/2/library/time.html#time.strftime)
print = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())


today = datetime.date.today()
oneday = datetime.datetime(2016, 06, 08, 13, 14, 15, 16)

# 时间差
timespan = today - oneday
print timespan.total_seconds()

# 构造时间差
timespan = datetime.timedelta(day=1)
```

### IO
```python
# 打印到屏幕
print 'hello world'

# 输入一行，输入的值被当做变量
str = raw_input("请输入变量str：");

# 输入一个表达式
str = input('请输入一个表达式：')

# 打开并写入文件
fo = open("foo.txt", "wb")
fo.write("hello world!\n");

# 打开并读取文件
fo = open("foo.txt", "r+")
str = fo.read(10);
position = fo.tell();         # 获取文件当前读取到的位置
position = fo.seek(2, 1);     # 从当前位置向前移动2个字节

# 关闭文件
fo.close()

# 重命名文件
import os
os.rename('old.txt', 'new.txt')
os.remove('old.txt')
os.mkdir('newFolder')

# 修改当前目录
os.chdir("/Users/bomo/Downloads")

# 获取当前目录
print os.getcwd()

# 删目录
os.rmdir('/Users/bomo/Downloads/newFolder')
```

### 正则表达式
```python
import re
# 匹配从头匹配
print(re.match(r'www', 'www.runoob.com').span())
# 查找匹配
print(re.search(r'www', 'www.runoob.com').span())
# 替换
print re.sub(pattern, replaceString, inputString)
```

match和search有第三个参数flag，`match(pattern, input, flag)`
```python
print(re.search(r'WWW', 'www.runoob.com', flags=re.I).string)
```
flag|用法
-|------
I|忽略大小写
M|多行模式
S|单选模式——点任意匹配模式
L|使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
U|使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
X|详细模式。该模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。

本文记录自己在学习过程中总结的一些Python要点，接触过Java，C#，OC，Python的语法确实简单优雅，很少多余的东西，人生苦短，Python是岸
