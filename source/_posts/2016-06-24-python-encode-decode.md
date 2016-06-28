---
title: python编解码
categories: python
tags: 编码
date: 2016-06-24 15:53:43
updated: 2016-06-24 15:53:43
---

python使用中文字符串的时候，经常会遇到乱码，升值是根本无法使用的问题，经常无端报错，有找不到原因，有时候使用decode或encode可以解决问题，但是并不知道为什么，今天在这里总结一下关于python编解码的一些相关要点

<!-- more -->
## 1. python的字符串
python有两种类型的字符串，分别是`unicode`和`str`，我们知道，大部分其他语言的字符串都只有一种，使用同一的编码方式，为什么python要搞出两个来坑人？

先来看看声明，以便下面区别，str字符串没有前缀，unicode字符串有前缀`u`
```python
s = 'str字符串'
u = u'unicode字符串'
```

在python中，str是字节串，而unicode才是真正意义上的字符串，str是unicode字符串经过编码后字节组成，而它们是可以互相转换的
```python
# unicode -> str
s = u'abc中文'.encode('utf-8')
print type(s)             # <type 'str'>

# str -> unicode
print s.decode('utf-8')   # abc中文
```

unicode编码是python处理字符串的中间编码，其他编码都是通过unicode进行转换的，所以为了避免混乱，这里推荐一个原则：**不要对unicode使用decode，不要对str使用encode**

如果要把gbk编码的str转为utf16编码的str，我们需要用unicode编码做中转
```python
s8 = u'abc中文'.encode('gbk')   # 'abc\xd6\xd0\xce\xc4'

# 转成unicode
temp = s8.decode('gbk')        # u'abc\u4e2d\u6587'
# 转成str
s16 = temp.encode('utf-16')      # '\xff\xfea\x00b\x00c\x00-N\x87e'

# 合起来
s16 = s8.decode('gbk').encode('utf-16')
```

s16使用的是`utf-16`编码，这时候我们在控制台或IDE输出一下
```python
print s16
# 输出：��abc-N�e
```
我们看到了乱码，这是由于控制台或IDE的编码与字符串的编码不一致导致的，如果一致，就正常输出

## 2. py文件编码
在python中，如果源文件使用了非Ascii字符，必须在文件头（前几行）声明文件编码格式，这样python解释器在解释的时候，会把相应的str字符串编码为响应的编码格式

声明py文件编码格式如下，下面四种方式可以，第一句`#!/usr/bin/python`是用来兼容linux，声明python解释器的位置，通常也加上
```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
#coding=utf-8
#coding:utf-16
#coding=gbk
```
**注意上面声明的编码格式必须与文件保存的编码格式一致，否则会读取失败或出现乱码**
文件头声明的编码是给解释器看的，它会把文件中的字符串用响应的格式进行编码

test.py（文件编码为GBK，py编码声明为GBK）
```python
#!/usr/bin/python
#coding=gbk

u = u'abc中文'
s = 'abc中文'
print u                     # abc中文
print s                     # 出现乱码

# 转为unicode字符串，正常
print s.decode('gbk')       # abc中文
```
上面代码运行的后，直接打印s会出现乱码，因为s是用GBK进行编码的str字符串，而IDE或控制台使用的编码为UTF8，不一致，所以输出乱码，解决办法可以吧s转换为unicode字符串

## 3. 读取文件
假定文件`test.txt`是用GBK编码存储的，我们现在读取其内容
```python
import io

io.open('test.txt')
f = io.open('test.txt', encoding='gbk')
print f.read()
# 这是一个GBK编码存储的文件
```
默认使用utf-8编码读取，不指定编码格式可能会读不了或出现乱码

## 4. getdefaultencoding/setdefaultencoding
上面我们说到，把gbk编码转换成utf16的时候，我们需要借助unicode编码进行中转，其实不中转也是可以的，python会自动帮我们转
```python
# 先拿到一个gbk编码的str
gbk = u'abc中文'.encode('gbk')

# 严谨的方式是这样的
gbk.decode('gbk').encode('utf16')

# 省去解码这一步（会报错）
gbk.encode('utf16')
```
上面我们直接对str进行encode，python解释器会自动帮我们吧str先转成unicode，而解码方式使用系统默认的方式，一般默认的解码方式是`ascii`，可以通过`sys.getdefaultencoding()`取到，所以上面语句相当于，显然会报错
```python
gbk.decode('ascii').encode('utf16')
```
这个时候我们可以设置`defaultencoding`为gbk就可以了，设置之前，需要reload一下sys库
```python
import sys

print sys.getdefaultencoding()      # ascii
reload(sys)
sys.setdefaultencoding('gbk')

gbk = u'abc中文'.encode('gbk')
print gbk.encode('utf16')           # 输出：'\xff\xfea\x00b\x00c\x00-N\x87e'
```
输出成功

## 5. 总结
* 文件编码，存储时的编码格式，与python无关
* python编码声明，放在py文件前几行，为了避免冲突，通常保持与文件编码一致
* IDE/控制台编码，当我们在IDE/或控制台输出的字符串的时候，如果IDE的编码与字符串的编码不一致，也导致输出乱码
* str字符串encode的时候，会自动先转为unicode，再进行编码，setdefaultencoding可以设置str字符串默认转换为unicode的编码
* 在python中，真正用来执行操作的还是unicode字符串，而str字符串仅用于前后的编码的转换而已


为了保证编码正确，我们可以尽量把其他编码格式的str字符串根据响应的编码格式，转换为unicode字符串，再使用

在开发中我们尽量都使用UTF8作为编码的格式，包括IDE，文件编码默认都是用UTF8
