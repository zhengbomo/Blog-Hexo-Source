---
title: ubuntu安装mysql和python
categories: 技术
tags:
  - mysql
date: 2016-07-28 12:20:48
updated: 2016-07-28 12:20:48
---


前几天弄了一年免费亚马逊VPS服务器，这里记录一下配置python的环境和安装一些常用的工具

<!-- more -->

## 升级apt-get源
```bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

## 升级python
先看一下ubuntu自带的python的版本
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/56975859.jpg)
也可以通过`python --version`查看版本
自带的python版本为`2.7.6`，我们手动升级一下

升级前可能需要安装`gcc`, `make`, `zlib`, `ssl`
```bash
# 先更新一下源
$ sudo apt-get update
$ sudo apt-get install gcc
$ sudo apt-get install make
$ sudo apt-get install zlibc zlib1g-dev
$ sudo apt-get install libssl-dev
```

升级python（安装到`/usr/local/lib/python2.7.12`）
```bash
# 下载最新版
$ wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
# 解压
$ tar zxvf Python-2.7.12.tgz
# 进入目录
$ cd Python-2.7.12/
# 配置，注意带zlib，否则zlib要自己独立安装
$ sudo ./configure --prefix=/usr/local/lib/python2.7.12 --with-zlib-dir=/usr/local/lib
# 编译
$ make
# 执行安装
$ sudo make install
```

linux中安装程序基本上是`./configure`->`make`->`make install`三部曲，安装后的文件存放在`/usr/local/bin/python2.7.12`，需要链接到执行文件，安装完后发现python还是原来的版本，通过`which`命令看一下python的路径
```bash
$ which python
/usr/bin/python
```

进入`/usr/bin/`目录我们修改一下python文件换成我们新的python执行文件，在终端输入`python2.7.9`可以进入刚安装的版本，但是太麻烦了，这个时候改一下默认版本（有些版本安装后会自动改）
```bash
#  //对系统默认版本python进行操作，改名
$ sudo mv /usr/bin/python /usr/bin/python_old
$ sudo ln -s /usr/local/lib/python2.7.12/bin/python /usr/bin/python
```

到这里，我们就完成了python的升级，python安装在`/usr/local/lib/python2.7.12`目录下，python命令指向新的路径
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/66165983.jpg)

## 安装pip和easy_install
`easy_install`和`pip`是python最常用的两个包管理工具，通过源码编译安装时，并不会没有带这两个工具（windows和mac通过安装包安装时会自动安装pip和easy_install）
安装[easy_install]()
```
$ wget https://pypi.python.org/packages/1b/4f/e52b81c47396668deb139b628f4ebb499b3cd39fc05382851fa985d0b642/setuptools-24.3.0.tar.gz#md5=55d77ca2b1f783a71e330b0878da29ec
$ tar zxvf setuptools-24.3.0.tar.gz
$ cd setuptools-24.3.0
$ python setup.py build
$ sudo python setup.py install
```

安装pip
```bash
$ wget https://pypi.python.org/packages/e7/a8/7556133689add8d1a54c0b14aeff0acb03c64707ce100ecd53934da1aa13/pip-8.1.2.tar.gz#md5=87083c0b9867963b29f7aba3613e8f4a
$ tar zxvf pip-8.1.2.tar.gz
$ cd pip-8.1.2
$ sudo python setup.py install
```
默认安装在`/usr/local/lib/python2.7.12/bin`

这个时候不能直接用`easy_install`和`pip`命令，我们创建一下链接
```bash
$ sudo ln -s /usr/local/lib/python2.7.12/bin/easy_install /usr/bin/easy_install
$ sudo ln -s /usr/local/lib/python2.7.12/bin/pip /usr/bin/pip
```

如果系统已经有了pip和easy_install，我们需要改成新版本的pip和easy_install，通过which查看当前的路径，处理方法与python一样

删除原来的pip程序并链接新的pip程序
```bash
# pip
$ sudo mv /usr/bin/pip /usr/bin/pip_old
$ sudo ln -s /usr/local/python2.7.12/bin/pip /usr/bin/pip

# easy_install
$ sudo mv /usr/bin/easy_install /usr/bin/easy_install_old
$ sudo ln -s /usr/local/lib/python2.7.12/bin/easy_install /usr/bin/easy_install
```

## 安装virtualenv
```bash
$ sudo pip install virtualenv
```

## 安装flask
```
# flask依赖ssl库，需要先安装下面两个工具
$ sudo apt-get install openssl
$ sudo apt-get install libssl-dev

$ sudo pip install flask
```

## 安装mysql
使用下面命令检查是否安装过，如果没有任何输出，说明没有安装
```bash
$ sudo netstat -tap | grep mysql
```
安装`mysql-server`, `mysql-client`
```bash
$ sudo apt-get install mysql-server mysql-client
$ sudo apt-get install libmysqlclient-dev
```
安装过程会让你输入`root`用户的密码，输入后按`Tap`键下一步
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/17932531.jpg)

安装完成，测试是否成功安装（成功）
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/41329824.jpg)

登陆看看
```bash
$ mysql -u root -p
```
然后输入密码，ok
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/96881626.jpg)

创建数据库
```mysql
CREATE DATABASE IF NOT EXISTS TestDb DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/44504183.jpg)


查看所有数据库
```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| TestDb             |
| mysql              |
| performance_schema |
+--------------------+
```

使用数据库
```bash
mysql> use TestDb
Database changed
```

## 安装MySQL-python库
```bash
$ sudo pip install MySQL-python
```
使用
```python
import MySQLdb

conn = MySQLdb.connect(host="127.0.0.1", user="root", passwd="111111", db="PaiPaiDai", charset="utf8")
```

> 如果没有安装`libmysqlclient-dev`，安装过程中可能会出现下面错误
```bash
sh: 1: mysql_config: not found
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/tmp/pip-build-IMiD72/MySQL-python/setup.py", line 17, in <module>
    metadata, options = get_config()
  File "/tmp/pip-build-IMiD72/MySQL-python/setup_posix.py", line 43, in get_config
    libs = mysql_config("libs_r")
  File "/tmp/pip-build-IMiD72/MySQL-python/setup_posix.py", line 25, in mysql_config
    raise EnvironmentError("%s not found" % (mysql_config.path,))
EnvironmentError: mysql_config not found
```

## 安装lxml
安装前需要先安装几个依赖包
```bash
$ sudo apt-get install libxml2 libxml2-dev
$ sudo apt-get install libxslt1-dev
$ sudo apt-get install python-libxml2
```

安装lxml（安装可能需要几分钟）
```bash
$ sudo pip install lxml
```

参考：[http://lxml.de/installation.html](http://lxml.de/installation.html)

## 安装scrapy
安装scrapy前需要安装依赖`Twisted`
```bash
$ wget https://pypi.python.org/packages/c0/7c/c1e5b61e30b7ffc96576d2a922615c8068e6996a622be813fc626cef07aa/Twisted-16.3.0.tar.bz2#md5=e044af844623e9fbcbe29f578db6053a
$ tar xjf Twisted-16.3.0.tar.bz2
$ cd Twisted-16.3.0
$ sudo python setup.py install
```
安装完成后再安装scrapy
```bash
$ sudo pip install scrapy
```

入门教程：[https://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html](https://scrapy-chs.readthedocs.io/zh_CN/0.24/intro/tutorial.html)

## 安装requests库
让你从痛苦的`urllib`中解脱
```bash
$ sudo pip install requests
```
详细介绍：[http://cn.python-requests.org/zh_CN/latest/](http://cn.python-requests.org/zh_CN/latest/)

## 安装git
```bash
$ sudo apt-get install git
```

## 安装nginx
```bash
# 添加Nginx库到apt-get source中
$ sudo add-apt-repository ppa:nginx/stable
# 更新apt源
$ sudo apt-get update && sudo apt-get upgrade
# 安装nginx
$ sudo apt-get install nginx
```

启动
```bash
$ sudo /etc/init.d/nginx start
start: Job is already running: nginx
```

启动后可以通过ip可以正常访问
![nginx](http://7xqzvt.com1.z0.glb.clouddn.com/16-11-1/68380067.jpg)

## ubuntu使用技巧
### 1. vim退出不保存
有时候使用vim编辑系统文件的时候，由于没有权限无法保存，又无法退出，只用`:q!`可以不保存退出

### 2. 开启crontab日志
默认定时任务crontab是不开启日志的，需要修改`/etc/rsyslog.d/50-default.conf`并将下面一行的前面的注释`#`去掉（编辑的时候需要`sudo`权限）
```conf
# cron.*                          /var/log/cron.log
```
然后重启`rsyslog`和`cron`服务
```bash
$ service rsyslog restart;
$ service cron restart;
```



<!-- ##


pip install MySQL-python
pip install Flask
pip install uwsgi


使用apt-get安装Nginx的话，我们需要添加Nginx库到apt-get source中
sudo add-apt-repository ppa:nginx/stable


升级已有的包，确保系统上有uWSGI所需的编译器和工具：

```bash
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install build-essential python python-dev
```


安装Yum
$ sudo apt-get yum
$ yum -y install vixie-cron -->
