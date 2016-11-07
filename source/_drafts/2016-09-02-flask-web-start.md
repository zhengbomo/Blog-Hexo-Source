---
title: flask-web学习笔记(草稿)
categories: python
date: 2016-09-08 20:30:48
updated: 2016-09-09 20:30:48
tags:
---


## 简介
Flask是轻量级的Web框架，通过扩展增加功能，当熟悉了Flask之后，很可能读懂其所有的源代码，Flask是扩展而设计的，有很强的定制型，各个模块均有扩展组成

Flask本身提供的功能不多，主要有两个依赖：Werkzeug（Web服务器路由，调试，网关）和Jinja2模板引擎

而例如数据库，表单验证，用户验证等高级的功能均由扩展组件实现，由用户自己定制

<!-- more -->

## 安装
1. 虚拟环境：virtualenv，安装和使用参考[这里](/2016-07-28/virtualenv-start/)


## 使用
### 1. 基本使用
```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

from flask import Flask
app = Flask(__name__)


@app.route('/')
def index():
    return '<h1>Hello World!</h1>'


# 路由支持动态参数,flask会将动态参数作为函数参数传入
@app.route(u'/user/id/<int:id>')
def user_with_id(id):
    return '<h1>Hello flask! %s </h1>' % id


# 支持声明类型(Flask 支持在路由中使用 int、float 和 path 类型)
@app.route(u'/user/name/<path:name>')
def user_with_name(name):
    return '<h1>Hello, %s!</h1>' % name

# 路由还支持制定请求方法
@app.route(u'/post', methods=['POST'])
def post_user_name(name):
    return '<h1>Hello, %s!</h1>' % name

# 如果是入口模块,启动服务
if __name__ == '__main__':
    app.run(debug=True)
```

### 2. 请求header处理
可以通过request对象取得请求头，每个线程的request对象是独立的
```python
from flask import request

@app.route('/header')
def request_with_header():
    user_agent = request.headers.get('User-Agent')
    return '<h2>Your browser is </h2> <p>%s</p>' % user_agent
```

### 3. 请求钩子
flask支持下面四种钩子
* before_first_request:注册一个函数,在处理第一个请求之前运行。
* before_request:注册一个函数,在每次请求之前运行。
* after_request:注册一个函数,如果没有未处理的异常抛出,在每次请求之后运行。
* teardown_request:注册一个函数,即使有未处理的异常抛出,也在每次请求之后运行。
TODO：

### 4. 响应
通过元组返回状态码
```python
@app.route('/response')
def response():
    return '<h1>Bad Request</h1>', 400
```
TODO通过元组返回response header


通过make_response返回数据
```python
from flask import make_response

@app.route('/makeresponse')
def make_resp():
    resp = make_response('<h1>This document carries a cookie!</h1>')
    resp.set_cookie('answer', '42')
    return resp
```

重定向
```python
@app.route('/redirect')
def redirect():
    return redirect('http://www.example.com')
```

abort函数，通过web服务器返回404错误
```python
from flask import abort

@app.route('/404')
def notfound():
    return abort(404)
```

## Flask扩展
### Flask-Script
安装
```bash
$ pip install flask-script
```

把上面的函数入口改为
```python
from flask_script import Manager

# 如果是入口模块,启动服务
if __name__ == '__main__':
    app.debug = True
    manager = Manager(app)
    manager.run()
```

假定文件名为`hello.py`，运行并制定绑定的ip地址，如果机器有外网ip，则可以直接绑定外网
```bash
$ python hello.py runserver --host 192.168.1.3
```


## 模板引擎
模板可以分离代码逻辑和ui，可以达到更好的重用，flask默认的模板目录为`templates`，恩定义一个简单模板`template/index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
</head>
<body>
    <h1>Hello world!</h1>
</body>
</html>
```

在视图函数渲染该模板
```python
from flask import render_template

@app.route('/')
def index():
    return render_template('index.html')
```
这样就可以吧代码逻辑和html描述分离开

### 参数
在模板中使用 `{{ 占位符 }}`
```html
<h1>Hello, {{ name }}!</h1>
```

render_template可以传入参数字典
```python
@app.route(u'/user/name/<path:name>')
def user_with_name(name):
    return render_template('user.html', name=name)
```
模板渲染的时候，会把占位符对应的变量渲染成字符串，在占位符中，我们还可以使用表达式
```html
<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>
```

### 过滤器
占位变量还支持过滤器，flask支持下面过滤器

过滤器名  | 说  明
--|--
safe  |  渲染值时不转义，默认情况下会转义
capitalize  |  首字母大写
lower  |  全部小写
upper  |  全部大写
title  |  每个单词的首字母大写
trim  |  去掉前后空白符
striptags  |  渲染之前把值中所有的 HTML 标签都删掉


### 流程控制
if-else
```html
{% if user %}
    Hello, {{ user }}!
{% else %}
    Hello, Stranger!
{% endif %}
```

for-in
```html
<ul>
    {% for comment in comments %}
        <li>{{ comment }}</li> {% endfor %}
</ul>
```

### 模板宏
模板宏相当于python中的函数，可以在多处重用
```html
{% macro render_comment(comment) %}
    <li>{{ comment }}</li>
{% endmacro %}

<ul>
    {% for comment in comments %}
        {{ render_comment(comment) }} {% endfor %}
</ul>
```

为了在更多地方使用模板宏，我们可以把宏定义到单独的文件`macros.html`，需要的时候通过`import`引用，这里的import与python中的import类似
```html
{% import 'macros.html' as macros %}
<ul>
    {% for comment in comments %}
        {{ macros.render_comment(comment) }}
    {% endfor %}
</ul>
```

### 外部模板
直接使用include可以引用外部模板渲染到此处
```html
{% include 'common.html' %}
```

### 模板继承
模板还支持继承，父模板定义模板结构，子模板定义模板中的模块


父模板
```html
<html>
    <head>
        {% block head %}
            <title>
                {% block title %}{% endblock %} - My Application
            </title>
        {% endblock %}
   </head>
   <body>
      {% block body %}
      {% endblock %}
    </body>
</html>
```

子模板使用extends声明父模板
```html
{% extends "base.html" %}

{% block title %}Index{% endblock %}

{% block head %}
    {{ super() }}
    <style>
    </style>
{% endblock %}

{% block body %}
    <h1>Hello, World!</h1>
{% endblock %}
```
如果需要使用父模板的内容，需要调用`super()`

## [Flask-Bootstrap](http://pythonhosted.org/Flask-Bootstrap/index.html)扩展
安装
```bash
$ pip install flask-bootstrap
```

使用flask-bootstrap之前需要配置app对象
```python
from flask_bootstrap import Bootstrap

if __name__ == '__main__':
    bootstrap = Bootstrap(app)
    app.run(debug=True)
```

使用bootstrap模板（模板继承）
```html
{% extends "bootstrap/base.html" %}

{% block title %}Flasky{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle"
            data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Flasky</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
    <div class="container">
         <div class="page-header">
             <h1>Hello, {{ name }}!</h1>
         </div>
    </div>
{% endblock %}
```
上面使用了`title`, `navbar`, `content`三个block，bootstrap模板还支持其他block，详情见[这里](http://pythonhosted.org/Flask-Bootstrap/basic-usage.html#available-blocks)


## 自定义错误页面
```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```

## 链接
使用url_for生成链接可以更加灵活的根据配置的路由生成url，如：

> url_for ('user', name='john', _external=True)
返回结果：http://localhost:5000/user/john。
> url_for('index', page=2)
返回结果：http://localhost:5000/?page=2

flask默认保留了一个特殊的路由，即`static`
> url_for('static', filename='css/styles.css', _external=True)
返回结果：http://localhost:5000/static/css/styles.css。

## 处理时间
如果需要兼容世界各地的时间，通常可以使用UTC时间，UTC时间保留了时区信息，可以根据用户的时区转换成当地的时间，`moment.js`可以很好的处理这一转化，把服务器的UTC时间转化成用户当地的时间

安装[flask-moment](https://github.com/miguelgrinberg/Flask-Moment)
```bash
$ pip install flask-moment
```

在程序入口处配置
```python
from flask_moment import Moment
from datetime import datetime

@app.route('/utctime')
def utctime():
  return render_template('index.html', current_time=datetime.utcnow())

# 如果是入口模块,启动服务
if __name__ == '__main__':
    moment = Moment(app)
    app.run(debug=True)
```

在模板文件中引入`moment.js`库
```html
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
{% endblock %}

{% block page_content %}
    <div class="page-header">
        <p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
        <p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
    </div>
{% endblock %}
```

## 表单
使用表单之前，我们先安装一个插件[Flask-WTF](http://pythonhosted.org/Flask-WTF/)，改扩展提供了一些表单的基础功能，让flask的表单处理过程变得更丝滑和强大
```bash
$ pip install flask-wtf
```

### 跨站请求伪造保护
通过秘钥验证表单数据的真伪，我们首先定义一个表单类
```python
from flask_wtf import Form

# 字段类型和字段验证从wtforms导入
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired

# 表单类继承自Form
class HeloForm(Form):
    name = StringField('What is your name?', validators=[DataRequired()])
    submit = SubmitField('Submit')
```
wtforms支持的字段和验证方法可以见官网：[http://wtforms.readthedocs.io/en/latest/fields.html](http://wtforms.readthedocs.io/en/latest/fields.html)和[http://wtforms.readthedocs.io/en/latest/validators.html](http://wtforms.readthedocs.io/en/latest/validators.html)


在视图函数处理表单
```python
from HelloForm import HelloForm

@app.route('/form', methods=['GET', 'POST'])
def hello_form():
    name = None
    form = HelloForm()
    # 判断是否是submit提交的表单
    if form.validate_on_submit():
        name = form.name.data
    return render_template('helloform.html', form=form, name=name)
```

`helloform.html`模板
```html
{% extends "base.html" %}

{% block title %}form{% endblock %}

{% block page_content %}
    <form method="POST">
        {{ form.hidden_tag() }}
        {{ form.name.label }} {{ form.name(id='my-text-field') }}
        {{ form.submit() }}
    </form>
{% endblock %}
```

也可以使用bootstrap提供的表单模板宏
```html
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}form{% endblock %}

{% block page_content %}
    {% if name %}
        <h1>hello {{ name }} !</h1>
    {% else %}
        <h1>hello Stranger !</h1>
    {% endif %}

    {{ wtf.quick_form(form) }}
{% endblock %}
```

## Session
session的数据可以保存在cookie里，可以用来保存用户状态，用法非常简单
```python
from flask import session

# 存
session['name'] = form.name.data

# 取
name = session.get('name')
```

## url_for
使用url_for来构造url

```python
# /
url_for('index')

# /login
url_for('login')

# /login?next=/
url_for('login', next='/')

# /profile?username=John%20Doe
url_for('profile', username='John Doe')
```

## flash
在视图函数可以通过`flash`函数把数据发给模板，模板通过`get_flashed_messages`拿到消息列表进行显示

```python
from flask import flash

@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('Looks like you have changed your name!')
            session['name'] = form.name.data
            return redirect(url_for('index'))
        return render_template('index.html', form = form, name = session.get('name'))
```

在模板中
```html
{% for message in get_flashed_messages() %}
    <div class="alert alert-warning">
        <button type="button" class="close" data-dismiss="alert">&times;</button>
            {{ message }}
    </div>
{% endfor %}
```

flash的信息是一次性的，通过get_flashed_messages函数使用完后会被清掉，第二次调用会返回空

## 数据库
大多数数据库引擎都有对应的Python库，如MySQL、Postgres、SQLite、 Redis、MongoDB 或者 CouchDB等，Python自带支持SQLite数据库，这里介绍一个Flask用的ORM框架库`Flask-SQLAlchemy`，SQLAlchemy提供了以对象的方式操作数据库，而不用复杂的复杂的sql语句

### 配置数据库
在SQLAlchemy中，数据库通过URL指定，常用的数据库URL有
```bash
mysql://username:password@hostname/database
postgresql://username:password@hostname/database
sqlite:////absolute/path/to/database
sqlite:///c:/absolute/path/to/database
```

创建数据库操作对象SQLAlchemy
```python
from flask_sqlalchemy import SQLAlchemy

# 配置数据库
basedir = os.path.abspath(os.path.dirname(__file__))
dbpath = os.path.join(basedir, 'data.sqlite')
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + dbpath
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
db = SQLAlchemy(app)

if __name__ == '__main__':
    manager = Manager(app)
    manager.run()
```
拿到db对象，然后就可以使用SQLAlchemy的所有功能了

### 定义数据模型
导入刚刚创建的SQLAlchemy对象db
```python
from hello import db

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)
    age = db.Column(db.Integer, default=0)

class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
```

列选项

选项名|说明
--|--
主键|primary_key
唯一|unique
索引|index
空置|nullable
默认值| default

关系
//TODO

### 增删改查
```python
from hello import Role, User

admin_role = Role(name='Admin')
mod_role = Role(name='Moderator')
user_role = Role(name='User')
user_john = User(username='john', role=admin_role)
user_susan = User(username='susan', role=user_role)
user_david = User(username='david', role=user_role)

# 通过session属性添加模型数据
db.session.add(admin_role)
db.session.add(mod_role)
db.session.add(user_role)
db.session.add(user_john)
db.session.add(user_susan)
db.session.add(user_david)
# 提交更新
db.session.commit()

# 如果失败了，也可以回滚
# db.session.rollback

# 修改
admin_role.name = 'Administrator'
db.session.add(admin_role)
db.session.commit()

# 删除
db.session.delete(mod_role)
db.session.commit()

# 每个模型都提供了Query属性用于数据库查询
Role.query.all()
User.query.filter_by(role=user_role).all()
user_role = Role.query.filter_by(name='User').first()

# 查看SQLAlchemy生成的sql语句
str(User.query.filter_by(role=user_role))

```


## 部署到nginx
安装和部署nginx参见[这里](/2016-07-28/ubuntu-install-mysql-python/index.html#安装nginx)
运行nginx
```bash
$ sudo /etc/init.d/nginx start
```

安装uwsgi
```bash
$ sudo pip install uwsgi
$ sudo apt-get install uwsgi uwsgi-plugin-python
```

在我们工程目录下新建一个nginx配置文件（`flask_nginx.conf`）
```
server {
    listen      80;
    server_name localhost;
    charset     utf-8;
    client_max_body_size 75M;

    location /static {
        alias /home/ubuntu/python/python_practice/project/flask_start/hello/static;
    }
    location / {
        include uwsgi_params;
        uwsgi_pass 0.0.0.0:9000;
        uwsgi_param UWSGI_PYHOME /home/ubuntu/python/python_practice/project/flask_start/env/;
		    uwsgi_param UWSGI_CHDIR /home/ubuntu/python/python_practice/project/flask_start/hello/;
		    uwsgi_param UWSGI_MODULE hello;
		    uwsgi_param UWSGI_CALLABLE app;
    }
}
```

链接到nginx目录
```bash
$ ln -s /home/ubuntu/python/python_practice/project/flask_start/flask_nginx.conf /etc/nginx/sites-enabled/flask_nginx
```

新建配置文件（`flask_uwsgi.ini`）
```
[uwsgi]
plugins=python
vhost=true
socket=0.0.0.0:9000
```

链接到uwsgi目录
```
ln -s /home/ubuntu/python/python_practice/project/flask_start/flask_uwsgi.ini /etc/uwsgi/apps-enabled/flask_uwsgi.ini
```

重启nginx和uwsgi
```bash
$ sudo service nginx restart
$ sudo service uwsgi restart
```


## 参考链接
* [http://www.pythondoc.com/flask/index.html](http://www.pythondoc.com/flask/index.html)



## 其他链接
* [基于Flask和MongoDB写的多人博客系统 OctBlog](https://github.com/flyhigher139/OctBlog)
