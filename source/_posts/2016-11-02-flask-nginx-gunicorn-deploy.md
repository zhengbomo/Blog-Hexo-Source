---
layout: post
title: flask+nginx+gunicorn部署
date: 2016-11-02 10:43:09
updated: 2016-11-02 10:43:09
categories: python
tags:
  - flask
  - python
---

最近学习了flask，准备把flask部署到服务器上，这里记录部署的过程和期间遇到的一些问题

<!-- more -->

## 一、安装
### 1. flask
我们建立一个最简单的flask应用，目录如下
```bash
/home/ubuntu/python/flask
    www
      app
        run.py
```
为`run.py`模块添加一个helloworld示例代码
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

通常我们的应用是放在virtualenv环境下的，这里我的虚拟环境路径为
```bash
$ cd /home/ubuntu/python/env
```

### 2. nginx
nginx不用多介绍，高性能web服务器，通常用来在前端做反向代理服务器，下面是安装
```bash
# 先更新一下源
$ sudo apt-get update
$ sudo apt-get install nginx
```

安装完成后重启nginx服务
```bash
$ sudo service nginx restart

# 其他命令
$ sudo service nginx stop
$ sudo service nginx restart
```

就可以通过ip访问了，如果是外网并绑定了域名，也可以通过域名访问，nginx默认访问的是`/usr/share/nginx/html`这个文件
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-11-1/68380067.jpg)

### 3. [gunicorn](http://gunicorn.org/)
flask自带的服务器只适合在开发调试时候用，并不能满足性能的要求，我们这里采用 gunicorn做wsgi容器，用来部署python，安装很简单，进入虚拟环境安装，使用pip安装
```
(env) $ sudo pip install gunicorn
```

安装完后运行
```bash
# 先进到目录
(env) $ cd /home/ubuntu/python/flask/www/app
(env) $ gunicorn -w 4 -b 127.0.0.1:8080 run:app
```
> 后面的`run:app`的run表示模块，app表示模块里面的对象，即Flask实例，接着就可以访问了：http://127.0.0.1:8080


## 二、配置gunicorn到nginx
### 1. 配置nginx
修改nginx默认配置之前，我们先备份一下
```bash
$ sudo cp /etc/nginx/site-avalidable/default /etc/nginx/site-avalidable/default.bak
```

修改配置
```bash
$ sudo vim /etc/nginx/site-avalidable/default
```

内容如下
```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # 这是HOST机器的外部域名，用地址也行
    server_name example.org;

    location / {
        # 这里是指向 gunicorn host 的服务地址
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
配置完重启nginx
```bash
$ sudo service nginx restart
```
当访问ip或域名的时候，nginx会自动代理到`http://127.0.0.1:8080`，即gunicorn服务


### 2. 将gunicorn作为系统服务启动
上面配置完成后需要启动gunicorn才能看到hello world页面，我们需要让gunicorn在后台运行，而不是在控制台手动启动

#### 2.1 Ubuntu15.04系统版本以上
由于这里用到了`virtualenv`，为了减少一些配置的问题，这里我把gunicorn服务的启动包装到一个`myflask.sh`文件里面

```bash
#!/bin/sh

# 进入主目录
cd /home/ubuntu/python/

# 进入virtualenv环境
source env/bin/activate;

# 进入flask项目目录
cd flask/www/app;

# 启动gunicorn服务
gunicorn -w 4 -b 127.0.0.1:8080 run:app;
```

这里我用的是`Ubuntu16.04`，需要通过`systemd`来配置系统服务，我们先定义一个服务配置`myflask.service`

```
[Unit]
Description=The myflask service
After=network.target
[Service]
WorkingDirectory=/home/ubuntu/python/flask/www/app
ExecStart=/bin/bash myflask.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

拷贝服务到`/etc/systemd/system/`目录下
```bash
$ sudo cp myflask.service /etc/systemd/system/myflask.service
```

启动服务
```bash
# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 启动服务
$ sudo systemctl start myflask.service

# 停止服务
$ sudo systemctl stop myflask.service

# 重启服务
$ sudo systemctl restart myflask.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill myflask.service

# 查看服务的状态，可以看到一些错误
$ sudo systemctl status myflask.service
```

启动完后访问成功
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-11-2/62538421.jpg)

#### 2.2 Ubuntu 15.05系统版本以下
如果系统为12.04的版本，可以把服务配置文件加到`/etc/init.d/`目录中，服务配置`myflaskserver`如下
```bash
description "The myflask service"

# 关于runlevel(运行级别)的更多说明，参见：http://www.cnblogs.com/dkblog/archive/2011/08/30/2160191.html

# 在下面4种级别的时候开启
start on runlevel [2345]

# 在非下面4种级别的时候关闭
stop on runlevel [!2345]

respawn
setuid root
setgid www-data

# 设置虚拟环境路径
env PATH= /home/ubuntu/python/env/bin

# 修改当前目录
chdir /home/ubuntu/python/flask/www/app

# 执行gunicorn服务
exec gunicorn -w 4 -b 127.0.0.1:8080 run:app
```

拷贝到`/etc/init.d/`目录
```bash
$ sudo cp myflaskserver /etc/init.d/myflaskserver

# 为文件添加可执行权限
$ sudo chmod +x /etc/init.d/myflaskserver

# 链接到启动目录，系统启动的时候会运行
# S99表示优先级，系统核心服务用 0-19，20-39 是一般系统服务，40-59 好像是进行系统设置居多，60-79 是一些核心应用服务，80-99 就是最终用户接触的应用服务。
$ sudo ln -sf /etc/init.d/myflaskserver /etc/rc3.d/S99myflaskserver
```
