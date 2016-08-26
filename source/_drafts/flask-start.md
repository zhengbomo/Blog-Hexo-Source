---
title: flask学习笔记
date: 2016-08-22 22:04:17
updated: 2016-08-22 22:04:17
categories: python
tags: [flask]
---

## 简介
Flask 是一个轻量级的 Web 应用框架, 使用 Python 编写。基于 WerkzeugWSGI工具箱和 Jinja2模板引擎。使用 BSD 授权。Flask也被称为 “microframework” ，因为它使用简单的核心，用 extension 增加其他功能。Flask没有默认使用的数据库、窗体验证工具。然而，Flask保留了扩增的弹性，可以用 Flask-extension 加入这些功能：ORM、窗体验证工具、文件上传、各种开放式身份验证技术。

## 使用
下面是一个最简单的flask的demo
```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```
