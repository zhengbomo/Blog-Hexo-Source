---
title: Python使用sqlite
categories: python
tags: sqlite
date: 2016-06-22 21:55:58
updated: 2016-06-22 21:55:58
---


与其他语言一样，Python也能进行数据库操作，并且在2.5版本以后，Python就内置了sqlite3，所以在使用Python的时候不需要装任何东西，可以直接使用

<!-- more -->

下面是一个最简单的操作数据库的例子

## 1. 增删改
```python
import sqlite3
# 连接数据库，在当前目录下找数据库文件，如果不能再，则会创建
conn = sqlite3.connect('test.db')
# 使用绝对路径
# conn = sqlite3.connect('/Users/bomo/Documents/Code/Python/test.db')
# 连接内存数据库
# conn = sqlite3.connect(":memory:")

# 获得一个游标，通过这个游标对数据库操作，使用完成后需要关闭
cursor = conn.cursor()

# 执行一个sql语句，创建一个表
sql = 'CREATE TABLE if not exists user (id varchar(20) primary key AUTOINCREMENT, name varchar(20), age integer)'
cursor.execute(sql)
# 增删改必须调用commit才能生效
conn.commit()

# 插入一条记录
sql = 'INSERT INTO user (name, age) values (\'bomo\', 18)'
cursor.execute(sql)
conn.commit()

# 删除一条记录
sql = 'DELETE FROM user WHERE name=\'bomo\''
cursor.execute(sql)
conn.commit()

# 使用完成后需要关闭游标
cursor.close()

# 关闭连接
conn.close()
```
> 如果对数据库进行增删改操作的时候，需要调用`connection.commit()`方法才能生效

## 2. 查询
```python
conn = sqlite3.connect('test.db')
cursor = conn.cursor()

# 查询结果集，参数为元组
sql = 'select * from user where id = ?'
cursor.execute(sql, ('1',))
values = cursor.fetchall()    # 获取所有的行记录，得到一个list，list元素为tuple
print values
# 输出：[(1, u'bomo', 18), (2, u'tobi', 6)]

# 查询单个结果
sql = 'select count(*) from user'
cursor.execute(sql)
result = cursor.fetchone()    # 得到一个元组
print result
# 输出：(2,)

cursor.close()
conn.close()
```
### 3. row_factory
上面查询结果可以看到，输出结果为元组，只包含值信心，我们可以通过row_factory配置更丰富的结果集，python的sqlite自带`sqlite.Row`工厂可以生成可以通过索引和列名访问值的结果集，如下
```python
connection = sqlite3.connect('spider.db')
# 查询结果集使用Row构造, sqlite.Row提供了基于索引和列名索引的方式
connection.row_factory = sqlite3.Row
cu = connection.cursor()

# 查询多行
cu.execute('select * from user limit 1')
rows = cu.fetchall()
for row in rows:
    # print type(row)   # <type 'sqlite3.Row'>
    # 取得所有的列
    for col in row.keys():
        print '%s=%s ' % (col, i[col])
# 输出：
# id=1
# name=bomo
# age=18

# 查询单行
cu = connection.cursor()
cu.execute('select count(*) as rowcount count from user')
row = cu.fetchone()
for col in row.keys():
    print '%s=%s ' % (col, i[col])
# 输出：
# rowcount=2

cu.close()
connection.close()
```

当然，我们也可以自定义`row_factory`
```python
# 自定义row构造器，返回字典对象，可以通过列名索引
def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

connection.row_factory = dict_factory
```

## 4. 字符串编码
数据库默认的编码格式为UTF-8，可以通过下面命令指定编码格式，如果主数据库还没创建，则可以指定编码，否则使用原有编码格式
* PRAGMA encoding = "UTF-8";
* PRAGMA encoding = "UTF-16";
* PRAGMA encoding = "UTF-16le";
* PRAGMA encoding = "UTF-16be";　

```python
connection = sqlite3.connect('spider.db')
conn.executescript('PRAGMA encoding = "UTF-16";')
cu = connection.cursor()

# do something

cu.close()
connection.close()
```

> 问题：主数据库（main database）是指什么??

## 5. text_factory
从sqlite数据库读取出来的字符串需要转换成unicode对象，text_factory可以用于编码的转换，构建unicode字符串，默认保存的就是unicode编码，如果我们需要把字符串存成UTF8编码的，我们需要修改text_factory的值

```python
conn = sqlite3.connect('test')
conn.text_factory = str                                     # 默认为utf8编码
conn.text_factory = lambda t: unicode(t, 'gbk', 'ignore')   # 设为gbk编码
```

参见官网的[介绍](https://docs.python.org/2/library/sqlite3.html#sqlite3.Connection.text_factory)

关于python的编解码和unicode与str可以看我[另一篇文章](/2016-06-24/python-encode-decode/)
