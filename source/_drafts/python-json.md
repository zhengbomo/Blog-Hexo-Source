---
title: Python的Json处理
date: 2016-06-24 01:24:22
updated: 2016-06-24 01:24:22
categories: python
tags:
---

将一个对象序列化

```python
import json

# 序列化
data = [{'a':"A",'b':(2,4),'c':3.0}]  #list对象
data_string = json.dumps(data)

# 反序列化
decoded = json.loads(data_string)
```
