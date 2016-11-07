---
title: flask-jinja模板引擎
date: 2016-09-25 15:26:43
updated: 2016-09-25 15:26:43
categories: python
tags: [flask, jinja]
---

最近在学flask开发，其中最重要的组件之一就是jinja模板引擎，绝大多数Web开发框架中都有各自的模板引擎，模板可以大大降低数据与UI的耦合，甚至可以达到完全分离的程度，模板引擎相当于定义好一个模板，然后使用占位的符的方式在最终渲染的时候吧占位符渲染成对应的变量，jinja模板引擎还提供了一些逻辑操作，例如循环，流程控制，判断，函数宏等，并且还支持继承和块，让模板也想代码一样高度重用并且更好的维护

<!-- more -->
## 一、安装
通过pip安装即可
```bash
$ pip install jinja2
```

## 二、介绍
先来看看最基本的使用
```python
from jinja2 import Template

template = Template('Hello {{ name }}!')
template.render(name='John Doe')
# u'Hello John Doe!'
```
上面只是jinja引擎通过api的执行过程，通过`render`函数渲染模板，渲染的过程或根据相关语法，把相应的变量进行替换

我们先来看个简单的模板
```html
<title>{% block title %}{% endblock %}</title>
<ul>
{% for user in users %}
  <li><a href="{{ user.url }}">{{ user.username }}</a></li>
{% endfor %}
</ul>
```
这里用到了块定义，一个for循环，还有占位变量，关于具体的语法，后面介绍

## 三、API
### 1. Environment
jinja引擎使用一个Environment对象定义如何加载和渲染模板的配置


## 相关链接
* [中文文档](http://docs.jinkan.org/docs/jinja2/)
*
