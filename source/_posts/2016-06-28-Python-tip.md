---
title: 《编写高质量代码 改善Python程序的91个建议》学习笔记
categories: python
date: 2016-06-28 11:16:46
updated: 2016-07-11 11:16:46
tags: python
---

最近在读《编写高质量代码 改善Python程序的91个建议》，在这里总结阅读中遇到的一些要点，和一些自己的理解

<!-- more -->

## 0. the Zen of Python
先来看看一个有趣的彩蛋，python的设计之禅，我们在Python控制台输入`import this`，可以看到
```python
>>> import this
The Zen of Python, by Tim Peters          # Python的禅宗

Beautiful is better than ugly.            # 优美胜于丑陋
Explicit is better than implicit.         # 明了胜于晦涩
Simple is better than complex.            # 简单胜于复杂
Complex is better than complicated.       # 复杂胜于凌乱
Flat is better than nested.               # 扁平胜于嵌套
Sparse is better than dense.              # 间隔胜于紧凑
Readability counts.                       # 可读性很重要
Special cases aren't special enough to break the rules.                     # 特例并不违背规则
Although practicality beats purity.                                         # 虽然实用性比完美
Errors should never pass silently.
Unless explicitly silenced.                                                 # 错误不应该被忽略，除非你明确要这样做
In the face of ambiguity, refuse the temptation to guess.                   # 在模棱两可的时候，拒绝胡乱猜测
There should be one-- and preferably only one --obvious way to do it.       # 应该有一个，最后只有一个方式可以做到
Although that way may not be obvious at first unless you're Dutch.          # 虽然好的方式可能不容易做到（但我心向之），除非你是Python之父
Now is better than never.                           # 立行胜于不做
Although never is often better than *right* now.    # 不做胜于鲁莽
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.    # 如果你无法向别人描述好你的实现，那这一定是个糟糕的想法，如果能，或许是个好想法
Namespaces are one honking great idea -- let's do more of those!    # 命名空间是一个很棒的理念，我们尽量多利用它
>>>
```

python的设计哲学可以归纳为两个单词：简单，易懂

## 1. 理解Pythonic
什么是Pythonic，最直观的解释就是Python风格的代码，那什么是Python风格的代码
看看下面这个C语言的例子
```c
for (i = 0; i < mylist_length; i++) {
   do_something(mylist[i]);
}
```
如果直接写成Python的风格，是这样的（Python的`for`语句只用于迭代，故我们把上面的写法转成`while`语句）
```python
i = 0
while i < mylist_length:
   do_something(mylist[i])
   i += 1
```

上面代码可以正确运行，但是并不被人为是Python的风格，我们稍作修改
```python
for i in range(mylist_length):
   do_something(mylist[i])
```
上面代码比之前的`while`更为简洁，但还不是完全的Pythonic风格，下面方式才是完全的Pythonic风格
```python
for element in mylist:
   do_something(element)
```

从上面例子中我们可以看出，Pythonic的代码，变量更少，更为短小，更为简单，读起来更为清晰

另外一个经常被提到的问题是，如何直接修改引用的变量（指针变量），我们再来看另外一个C语言的例子
```c
void foo(int* a, float* b) {
    *a = 3;
    *b = 5.5;
}

int alpha;
int beta;
foo(&alpha, &beta);
```

上面代码不能很好的描述其功能，并且晦涩难懂，在Python不鼓励这种写法，也不支持这种写法，Python使用输入输出的方式传值
```python
def foo():
    return 3, 5.5

alpha, beta = foo()
```

再看一个例子，在C语言中交换两个数
```c
int a = 1, b = 2
int tmp = a
a = b
b = tmp
```
Python中交换两个数
```
a, b = b, a
```

Pythonic是一种代码风格，以简单，易懂为宗旨

参考：[http://blog.startifact.com/posts/older/what-is-pythonic.html](http://blog.startifact.com/posts/older/what-is-pythonic.html)


## 2. 编写Pythonic代码
* 变量名不与内建方法重名，如dict, list, element等，
* 由于Python使用缩进识别代码块，所以在代码里面，多余的空格和Tab尽量不要随便使用，**不推荐**对齐等号的方式（下面方式）
  ```python
  a        = 10                 # some comment
  some_str = 'hello world'      # some comment
  ```
* 注释
  ```python
  # 下面第一种比较第二行更好
  x = x + 1         # increase x by 1
  x = x + 1 # increase x by 1
  ```
* 函数详细注释
  ```python
  def func(a, b):
      """summary desctiption

      more detail comments for the function

      Args:
          a: some comment for parameter a
          b: some comment for parameter b

      Returns:
          return type, return value desctiption

      Raises:
          IOError: IOError exception may raise in the function
          IndexError: IndexError exception may raise in the function
      """
  ```

* 函数设计
  * 函数长度不宜过长，通常以小于一屏为准
  * 函数嵌套不宜过多，通常保持在3层以内（for, if-else等）
  * 参数不宜过多
  * 函数只做一件事
  * 使用异常抛出错误，而不通过返回值错误
  * 尽量不要在函数中定义可变对象，除非特殊需要

## 3. 常量
Python没有提供常量的支持，通常使用命名规范识别常量，所有字母大写，如`MAX_OVERFLOW`，当然，这只是一种约定，实际上与变量一样，是可以改变的

还有一种方式来模拟实现常量的功能，使用类来限制对属性的赋值
```python
class _const:
    class ConstError(TypeError): pass
    class ConstCaseError(ConstError): pass

    def __setattr__(self, name, value):
        if self.__dict__.has_key(name):
            raise self.ConstError, 'Can\'t change const.%s' % name
        if not name.isupper():
            raise self.ConstCaseError, 'const name "%s" is not all uppercase' % name
        self.__dict__[name] = value


import const
const.COMPANY = 'Google'
```

## 4. 使用断言
断言在其他很多语言都存在，可以方便用于测试和调试程序，使用断言格式如下
```python
assert expression, some_error_info

# 如下
x, y = 1, 2
assert x == y, 'not equals'
```
上面例子相当于
```python
x, y = 1, 2
if __debug__ and not x == y:
    raise AssertionError('not equals')
```
断言会带来一定的性能消耗，由于Python没有严格意义上的Debug和Release模式，故它并不优化字节码，只是忽略相关代码的执行，在执行脚本的时候添加`-O`参数可以禁用断言


## 5. 使用枚举
Python本身并不提供枚举的功能，关于Python是否要加入枚举功能，也引发了很多讨论，最后被组织拒绝了(在Python3.4以后又支持了😅)，但是因为Python强大的动态性，我们可以通过很多方式实现枚举的功能

```python
class Seasons:
    Spring = 0
    Summer = 1
    Autumn = 2
    Winter = 3

# 简写为
class Seasons:
    Spring, Summer, Autumn, Winter = range(4)

# 使用函数动态构造一个对象
def enum(*posarg, **kvarg):
    return type('Enum', (object,), dict(zip(posarg, xrange(len(posarg)))))

season = enum('spring', 'summer', 'autumn', 'winter')
print season.summer
```
我们还可以用第三方模块实现枚举的功能`flufl.enum`
//TODO:


## 6. 不推荐使用type()来判断类型相等
* 在经典类中，所有对象执行type()都相等
* 在新式类中，type()无法用于判断子类与父类的关系
  ```python
  type(son) is type(parent)       # 正常逻辑应该为True，但是结果是False
  ```

* 通常使用isinstance()方法判断类型
  ```python
  isinstance(son, Son)                  # True
  isinstance(son, (Son, list, tuple))   # True
```

## 7. 注意运算时候的精度问题
Python与C语言一样，计算精度取决于计算的值的类型，如两个整数相除，结果是整数，如果需要获得高精度的结果，需要转换为float类型在进行计算，在python3里，这个问题不存在
```python
a, b = 1, 3

print a / b             # 0
print a / float(b)      # 0.33333333
```

## 8. 尽量避免浮点类型的比较
浮点类型，在计算过程中可能有精度损失的风险，应尽量避免，如果可以转成整形再计算
```python
i = 1.0
while i != 1.5:
    i += 0.1
    print i
```
上面语句会一直在while循环，而不能正确跳出

## 9. 避免使用eval
用过JS或PHP的可能都知道eval函数，可以直接执行字符串脚本，然而，字符串有注入的风险，有安全性问题，如果需要，可以考虑使用`ast.literal_eval`代替

//TODO


## 10. 使用enumerate枚举索引和变量
Python语言很灵活，同一功能有多种实现方式，如索引列表
```python
l = [1, 2, 3]

# 方式一
index = 0
for i in l:
    do_something(i, index)
    index+=1

# 方式二
for index in range(len(l))
    do_something(l[index], index)

# 方式三
for index, i in zip(range(len(l)), l)
    do_something(i, index)

# 方式四
for index, i in enumerate(l):
    do_something(i, index)
```
推荐使用方式四，支持延迟加载，不会一次枚举出所有的值，性能最优，书写也简洁，enumerate不适用于dict对象

## 11. 分清is和==
is：比较内存地址，而不是内容，`a is b`相当于`id(a) == id(b)`
==：`a == b`相当于`a.__eq__(b)`，可以重载`__eq__`方法实现等于的逻辑

```python
class person(object):
    def __init__(self, pid, name):
        self.pid = pid
        self.name = name

    def __eq__(self, p):
        return self.pid == p.pid

p1 = person(1, 'bomo')
p2 = person(1, 'tobi')

print p1 == p2    # True
```

## 12. 尽量使用Unicode编码
python编码见[这里](/2016-06-24/python-encode-decode/)

## 13. 多使用模块和包来管理文件
* 尽量减少使用`from pack import *`这种导入方式，会污染命名空间，容易导致命名冲突，如果冲突，则后导入的覆盖先导入的
* TODO: absolute import, relative import


## 14. Python不支持++i语法
Python会把`++i`解释为`+(+i)`
```python
a = -1
print ++a
# 相当于
print +(+a)
# 输出-1
```

## 15. 使用with自动释放资源
用过C#的朋友应该都知道using，可以在代码块结束后自动释放资源，而Python也支持类似的语法
```python
import io
f = open('test.txt', 'r')
print f.read()
f.close()
```
像上面这种IO资源，在使用完成后需要开发者自己调用释放资源的方法，通常也会使用`try-except-finall`y来保证释放资源，而通常情况下，释放资源很容易遗漏，可以使用`with`语法，把相关操作放在代码块中，当离开代码块的时候会自动调用释放资源的方法，这样就可以避免人为的遗漏问题，上面代码可以写成下面方式
```python
with open('test.txt', 'r') as f:
    print f.read()
# 离开代码块后，f会自动释放
```
无论代码块中是否会抛出异常，离开代码块的时候，资源f都会被释放，其实只需要实现`__enter__`和`__exit__`方法就能支持这种行为
```python
class MyObj(object):
    def open(self):
        print 'open'
        return self

    def __enter__(self):
        print 'enter'

    def __exit__(self, exc_type, exc_val, exc_tb):
        print 'close'

        if exc_type is None:
            print '没有异常'
            return False
        else:
            print '出现异常'
            return False

obj = MyObj()

with obj.open() as v:
    print str(v) + '执行一些操作'

# 输出
# open
# enter
# Nonedo something
# close
# 没有异常
```
