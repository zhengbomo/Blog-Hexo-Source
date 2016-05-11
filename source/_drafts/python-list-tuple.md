---
title: python-list-tuple
date: 2016-05-11 00:06:50
updated: 2016-05-11 00:06:50
categories:
tags:
---


## 列表与元组

python最基本的数据结构为序列（列表和元组）

序列支持下面操作
* 序列支持正向和反向索引，第一个索引为`0`，反向第一个为`-1`
* 切片(sliceing)：
  * 可以操作序列中的一定范围的子序列，如：list[1:3]表示序列list的2~3个元素
  * 可以省略索引表示起始和结束，如：list[:]
  * 可以指定步长（默认为1），如：list[1:10:2] 表示序列第2,4,6,8,10两个元素
  * 插入元素：list[1:1] = 'new' # 插入到第一个元素后面
* 加法(adding)：如：[1, 3, 4] + [2, 5, 6]
* 乘法(multiplying)：如得到包含10个1的序列：[1] * 10

### 列表方法
* list.append('new value')
* list.count()
* list.extend([1, 2])  # 追加序列
* list.index('firstOne')
* list.insert(0, 'firstOne')
* list.pop(0)
* list.remove('firstOne')
* list.reverse()
* list.sort()

### 元组
元组用逗号隔开（必须），可用括号括起来（可选）

* 元组大部分操作与列表一样
* 一般来说，列表可以满足序列所有的要求


## 字符串
字符串相当于特殊的列表，支持列表的所有操作，有一点需要注意是字符串不可变

### 格式化
使用`%`隔开格式化字符串和值
```python
from math import pi
format = 'Hello %s, pi is %10.2f'
value = ('World', pi)
print format % value      # Hello World, pi is 3.14
```

`in`运算符：判断元素是否在集合内
