---
title: virtualenv学习笔记
categories: python
tags: [virtualenv]
date: 2016-07-28 16:05:38
updated: 2016-07-28 16:05:38
---

`virtualenv`是python的一个第三方模块，用来配置独立的python环境，特别是在服务器端需要进行版本控制时使用，有些第三方库向前或向后兼容性很差，这时候可用针对不同的版本配置不同的python环境更为方便

<!-- more -->

## 一、安装
可以直接通过pip安装，也可以自行下载源码编译安装
```bash
$ sudo pip install virtualenv
```

安装完后需要连接到执行命令目录
```bash
$ sudo ln -s /usr/local/lib/python2.7.12/bin/virtualenv /usr/bin/virtualenv
```
在终端使用`virtualenv -h`查看帮助

## 二、基本使用
我们可以使用`virtualenv`创建一个独立的python环境，默认也会包含pip, easy_install, wheel等工具：
```bash
$ virtualenv envname  # 创建一个新的隔离环境，会安装Installing setuptools, pip, wheel...done.
$ cd envname
```

### 常见命令参数
* `--system-site-packages`: 使用系统的全局的python库
* `--no-site-packages`: 不使用系统的全局的python库（默认）(废弃)
* `--download`: 从网上下载包预安装的包
* `--no-download`: 使用本地包，不从网上下载，如果不存在会报错

更多参数见官网说明：[https://virtualenv.pypa.io/en/stable/reference/#cmdoption--system-site-packages](https://virtualenv.pypa.io/en/stable/reference/#cmdoption--system-site-packages)


### 我们查看一下有哪些文件
```bash
$ ls
bin  include  lib  pip-selfcheck.json

$ ls bin
activate       activate_this.py  pip     python     python-config
activate.csh   easy_install      pip2    python2    wheel
activate.fish  easy_install-2.7  pip2.7  python2.7

$ ls include
python2.7

$ ls lib
python2.7
```
文件与python安装目录下的文件类似，即独立环境所使用的package和一些可执行程序

### 激活
使用下面命令激活当前的环境（这里用的是mac），之后使用的python环境就是刚创建的虚拟环境，命令行前面会带虚拟环境的名字：`(envname)`
```bash
$ source bin/activate
(envname) localhost:envname zhengxiankai$
```

我们通过which查看一下当前环境下的python执行文件的路径，我们发现当前的环境变成了刚刚激活的路径，而不是系统的python路径了，而使用pip安装的路径包也会在这个环境的路径下
```bash
(envname) localhost:envname zhengxiankai$ which python
/home/ubuntu/envname/bin/python
```

进入python交互解释器
```bash
(envname) localhost:envname zhengxiankai$ python
Python 2.7.12 (default, Jul 28 2016, 07:03:11)
[GCC 4.8.4] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

查看搜索路径
```bash
>>> import sys
>>> sys.path
['', '/home/ubuntu/envname/lib/python27.zip', '/home/ubuntu/envname/lib/python2.7', '/home/ubuntu/envname/lib/python2.7/plat-linux2', '/home/ubuntu/envname/lib/python2.7/lib-tk', '/home/ubuntu/envname/lib/python2.7/lib-old', '/home/ubuntu/envname/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7.12/lib/python2.7', '/usr/local/lib/python2.7.12/lib/python2.7/plat-linux2', '/usr/local/lib/python2.7.12/lib/python2.7/lib-tk', '/home/ubuntu/envname/lib/python2.7/site-packages']
>>>
```

使用完成之后通过`deactivate`命令退出虚拟环境，前面的虚拟环境名`(envname)`没有了，说明退出了
```bash
(envname) localhost:envname zhengxiankai$ deactivate
localhost:envname zhengxiankai$
```

## 三、与Pycharm结合
Pycharm是python最常用的开发工具，当然也提供了virtualenv的支持，到设置里面的`Project Interpreter`添加本地已经存在的虚拟环境，也可以直接创建，然后应用到工程即可
![](http://7xqzvt.com1.z0.glb.clouddn.com/001.png)

## 四、总结
virtualenv可以创建python的独立环境，可以包含一整套python的环境（除了外部依赖，如mysql等），特别是在服务器部署时可以连同环境一块部署，不需要为每一台服务器安装所有的库，而在多人开发过程中，为了保证环境的一致，也可以把独立环境也通过git维护，这样可以保证所有人的环境一致，而不用在所有的机器上配置
