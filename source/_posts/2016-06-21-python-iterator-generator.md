---
title: Python迭代器和生成器
categories: Python
date: 2016-06-21 14:55:49
updated: 2016-06-21 14:55:49
tags:
---

与其他高级语言一样，Python也提供了迭代器的功能，迭代器统一了访问的集合的方式，Python中所有的集合数据类型（list, str, dict, set, tuple）都支持使用for进行迭代，当然我们也可以为自己定义的类或函数实现这种迭代的功能

<!-- more -->

## 例子
我们先来看一个例子，我们生成一个斐波那契序列
```python
def fabs(count):
    n, a, b = 0, 0, 1
    while n < max:
        print b
        a, b = b, a + b
        n = n + 1
```

上面在函数里面直接print结果，显然这样做的复用性特别差，于是我们想到了用列表

```python
def fabs(count):
    l = []
    n, a, b = 0, 0, 1
    while n < max:
        l.append(b)
        a, b = b, a + b
        n = n + 1
    return l

for i in fabs(10):
    print i
```

使用list返回可以解决了复用性问题，但是有时候我们需要使用的数据量非常大的时候，返回整个list会占用大量内存，这个时候，我们希望，函数返回的值不要一次性全部返回，而是用到的时候计算再返回，这样数据量再打也只占用一份内存而已了，Python提供了两种方式实现这种逐步迭代的方式，于是就有了下面迭代器和生成器

## 迭代器
迭代器统一了所有集合访问元素的方式，包括有序无序的，相比for遍历集合，其支持随机访问的集合，如`set`，`dict`，只能向前，不能后退，迭代器有下面两个基本方法，大多数高级语言都会定义统一的迭代器操作，Python中的迭代器类型需要实现下面两个方法
* `next`：返回迭代器下一个元素
* `__iter__`方法：返回一个迭代器

给一个类实现迭代器功能，生成斐波那契序列
```python
class Fabs(object):
    def __init__(self, max):
        """传入斐波那契数列的个数"""
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
        raise StopIteration()   # 迭代结束需要抛出异常

fab = Fabs(8)
for i in fab:
    print i             # 1, 1, 2, 3, 5, 8, 13, 21
```

## 生成器yield
上面我们看到每次迭代通过调用next取得值，Python还提供了另一个关键字yield用于更方便的迭代每个值，C#也支持这种语法，带有yield关键的函数被称为生成器（generator）
> yield：在函数使用yield返回每次迭代的值，而不直接返回，yield返回值后等待下次迭代，而保留当前的状态（局部变量不变），直到下次迭代的时候，接着后面的代码继续执行

```python
def Foo():
    yield 1   # 返回后等待第二次迭代
    yield 3   # 第二次迭代从这里开始，等待第三次迭代
    yield 2   # 第三次迭代从这里开始，迭代完成

def myrange(max):
    print 'iter start'      # 迭代开始时执行
    i = 0
    while i < max:
        yield i
        i += 1
    print 'iter end'      # 迭代函数执行完没yield返回时，抛出StopIteration异常表示迭代结束

# 不执行迭代过程（不输出iter start）
print type(myrange(10))   # 输出：<type 'generator'>

for _ in Foo():
    print str(_)    # 输出：1 3 2

for _ in myrange(10):
    print str(_)    # 输出：0 1 2 3 4 5 6 7 8 9 iter end

```
* 上面函数Foo可以迭代三次，分别返回1,3,2，与普通的列表一样
* 使用了生成器的函数，不能使用return，编译器会报错
* 生成器返回的对象是`generator`
* 生成器在迭代完最后一个值之后，当迭代函数执行完，没有yield返回迭代值的话，会抛出StopIteration异常表示迭代结束
* 只有生成器调用`next`方法的时候才会运行迭代过程


## 生成器支持与外部函数交互send
生成器可以通过yield返回值给外部函数，也可以接受外部函数传递的值，通过`send`方法传值
```python
for _ in test():
    print _

# 相当于
try:
    while True:
        print ite.next()
except StopIteration, e:
    pass
```
迭代器每次迭代实际上相当于调用了next方法，然后从yield取值，Python迭代器还提供了send方法，功能与next类似，但是可以传递参数作为yield的返回值在迭代器内部使用

```python
def test():
    n = 1
    p = yield n
    if p:         # 调用send时，p接收参数值，调用next时，p为None
        n += p

    p = yield n
    if p:
        n += p

    yield n

ite = test()
try:
    print ite.next()    # 第一次不允许调用send
    print ite.send(1)   # 传递参数1给迭代器
    print ite.send(2)   # 传递参数2给迭代器
    print ite.send(3)   # 最后一次传至无效，因为迭代已经完成，触发StopIteration
except StopIteration, e:
    pass
```

## yield的原理
//TODO:
