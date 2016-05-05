---
title: 函数式编程
categories: 技术
tags:
  - 技术
date: 2016-05-05 11:24:28
updated: 2016-05-05 11:24:28
---


> 函数式编程（英语：functional programming）或称函数程序设计，又称泛函编程，是一种编程范型，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。而且λ演算的函数可以接受函数当作输入（引数）和输出（传出值）。


上面是[维基百科](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)上的解释，可以把编程语言中的函数视为数学函数一样，函数可以与普通变量一样，作为另一个函数的参数，如下
```
f(x) = x * x + 2
g(x) = f(x) + f(x + 2)
h(x) = g(x) + f(x - 1)
```
函数式编程有以下几个特点
* 避免状态变量，注重状态传递，而不是状态维护  
  非函数式
  ```python
  a = 0
  def increment1():
      global a
      a += 1
  ```
  函数式
  ```python
  def increment2(a):
      return a + 1
  ```
* 函数是一等公民  
  即函数与变量一样，可以作为参数传递
  ```python
  def myFilter(x):
    return x < 5

  def myMap(filter, array):
    result = []
    for i in array:
      if filter(i):
        result.append(i)
    return result

  print myMap(myFilter, [1,2,5,7,8,9])
  # 输出 [1, 2]
  ```

* 高阶函数  
  高阶函数满足以下两个特点
  * 函数可以作为参数被传递
  * 函数可以作为返回值输出  

  ```python
  # 函数作为参数
  def add(x, y, f):
    return f(x) + f(y)

  # 函数作为返回值
  def calc(x):                # 接收第一个参数     
    def square(n):            # 接收第二个参数
        return n * n + x;     # 计算
    return s(x);

  c = calc(4)     # n*n+4
  print c(3)      # 3*3+4
  ```
  从上面例子可以看出来，一个n * n + x计算是通过两步进行的，并且参数也分成多个单一的参数，这种过程称为函数的`柯里化`(Currying)，下面是[维基百科](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)上的解释
  > 在计算机科学中，柯里化（英语：Currying），又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。



* 惰性求职  
    在使用函数处可以不直接执行函数，在需要的时候执行函数，如上高阶函数以函数作为返回值，在调用calc的时候并没有进行计算，而是后面调用返回的函数才进行计算

* 闭包  
  闭包就是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。有时候你不需要知道闭包的概念，也能很好的应用它，有一个直观的解释：可以自由访问变量(局部变量和内部变量)的代码块
  ```python
  outa = 200
  def calc(x):
    a = 100
    def square(n):
        # 这里可以访问局部变量a，也可以访问函数外的自由变量outa
        return n * n + a + outa + x;
    return s(x);
  ```
  例如objc的block，就是对闭包的支持

* 面向功能，而不面向过程，重视功能粒度化，函数化   
  函数式
  ```python
  names = ["Mary", "Isla", "Sam"]
  # 使用map，reduce
  name_lengths = map(len, names)
  print name_lengths
  # => [4, 4, 3]
  ```

  非函数式
  ```python
  # 而不是for
  name_lengths = []
  for n in names:
      name_lengths.append(len(n))
  print name_lengths
  ```

函数式编程就是功能函数化，粒度化，注重行为，而不注重过程，注重操作，而不注重状态的一种方编程思想，可以很大程度的提高代码的阅读性，因为函数本身就是一种注释，这种粒度使得代码可以更好的自解释，而闭包的特性使得代码拥有更高的聚合度，减少函数多带来的代码松散问题

> 我的理解是编程思想只是思考问题的一些不同的角度，并不是绝对的，可能一些代码中既有函数式编程的特点也有面向对象的思想，例如Python就是一种多范式语言，所以在实际应用中不需要过于刻意的区分，而是根据这些特点和具体业务找到合适的方式
