---
title: Python装饰器
date: 2016-06-17 16:32:35
updated: 2016-06-17 16:32:35
categories: Python
tags: Python
---

Python从语法级别提供了对装饰器模式的支持，有时候需要为一些函数添加一些额外的操作，如在执行前后打印执行时间，由于Python是函数式编程语言，支持高阶函数（函数可以作为参数和返回值使用），这样我们可以定义一个函数对原有的函数进行包装，比如在函数执行前后进行打印

<!-- more -->

```python
def sayHello(name):
    print 'hello %s' % (name,)

def log(func):
    # 定义一个转换器,接收任意参数最后传给func
    def wrapper(*args, **kv):
        print 'begin %s' % (time.ctime(),)
        func(*args, **kv)
        print 'end %s' % (time.ctime(),)
    return wrapper

# 直接调用
sayHello('bomo')

# 包装后调用
f = log(sayHello)
f('bomo')
```
打印结果
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-6-17/57647661.jpg)

这样做还是很麻烦，Python提供了一种装饰器的语法，直接包装方法，而不改变原有的函数调用，我们在sayHello函数定义上加上`@log`
```python
# 把log函数的定义放在sayHello之前
@log
def sayHello(name):
    print 'hello %s' % (name,)

sayHello('bomo')
```
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-6-17/6205983.jpg)
这里的`@log`的功能就相当于
```python
sayHello = log(sayHello)
```

Python的装饰器本质上就是函数调用，除了上面的方式，还支持三层嵌套，也支持参数，我们可以修改log函数为下面形式如下
```python
def log(name):
    def wrapper2(func):
        # 定义一个转换器,接收任意参数最后传给func
        def wrapper(*args, **kv):
            print 'begin %s %s' % (name, time.ctime())
            func(*args, **kv)
            print 'end %s %s' % (name, time.ctime())
        return wrapper

@log('timelog')
def say_hello(name):
    print 'hello %s' % (name,)
```
这时候就相当于
```python
say_hello = log('timelog')(say_hello)
```

装饰器是替换了原来的方法，所以使用装饰器后方法名也会变化，由上面代码可以看出
```python
say_hello = log('timelog')(say_hello)
# 相当于
say_hello = wrapper2(say_hello)
# 相当于
say_hello = wrapper
```

故`say_hello.__name__`为`wrapper`

有时候我们可能会有些操作依赖于原来的方法名，这时候我们就不希望方法名被改了，或者是我们在使用装饰器后还能访问到原来函数的一些属性（函数也是对象，也有属性），Python提供了`functools.wraps`装饰器，用于把外部方法的相关属性赋值给内部方法，如
```python
def log(func):
    # 定义一个转换器,接收任意参数最后传给func
    @functools.wraps(func)
    def wrapper(*args, **kv):
        print 'begin %s' % (time.ctime(),)
        func(*args, **kv)
        print 'end %s' % (time.ctime(),)
    return wrapper


def log(name):
    def wrapper2(func):
        @functools.wraps(func)
        def wrapper(*args, **kv):
            print 'begin %s %s' % (name, time.ctime())
            func(*args, **kv)
            print 'end %s %s' % (name, time.ctime())
        return wrapper
```

Python还只是同事声明几个装饰器，如
```python
def log1(func):
    def wrapper(*args, **kv):
        print 'log1 begin %s' % (time.ctime(),)
        func(*args, **kv)
        print 'log1 end %s' % (time.ctime(),)
    return wrapper

def log2(func):
    def wrapper(*args, **kv):
        print 'log2 begin %s' % (time.ctime(),)
        func(*args, **kv)
        print 'log2 end %s' % (time.ctime(),)
    return wrapper

@log1
@log2
def say_hello(name):
    print 'hello %s' % (name,)

say_hello('bomo')
```
输出，先定义的在外层
```bash
log1 begin Fri Jun 17 19:00:59 2016
log2 begin Fri Jun 17 19:00:59 2016
hello bomo
log2 end Fri Jun 17 19:00:59 2016
log1 end Fri Jun 17 19:00:59 2016
```

在面向对象设计模式中有个叫装饰器模式的，与Python的类似，而Python直接从语法级实现了支持
