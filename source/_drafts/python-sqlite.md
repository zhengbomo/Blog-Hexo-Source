---
title: python-sqlite
date: 2016-06-02 21:55:58
updated: 2016-06-02 21:55:58
categories:
tags:
---

与其他语言一样，python也能进行数据库操作，并且在2.5版本以后，python就内置了sqlite3，所以在使用python的时候不需要装任何东西，可以直接使用

下面是一个最简单的操作数据库的例子
```python
import sqlite3
# 连接数据库
conn = sqlite3.connect('test.db')
# 创建一个游标，用于操作
cursor = conn.cursor()
# 执行一个sql语句，创建一个表
sql = 'CREATE TABLE if not exists user (id varchar(20) primary key AUTOINCREMENT, name varchar(20))'
cursor.execute(sql)

#插入一条语句
sql = 'INSERT INTO user (name) values (\'bomo\')'
cursor.execute(sql)

#查询
print 'row count = %d' % cursor.rowcount

# 查询结果集，参数为元组
sql = 'select * from user where id = ?'
cursor.execute(sql, ('1',))

# 查询的结果保存在游标上，通过fetchall方法拿到
values = cursor.fetchall()

# 使用完成后需要关闭游标
cursor.close()

# 必须调用commit才能生效
connect.commit()

# 关闭连接
conn.close()


```
