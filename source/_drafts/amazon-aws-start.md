---
title: amazon-aws-start
date: 2016-07-18 21:02:00
updated: 2016-07-18 21:02:00
categories:
tags:
---

亚马逊服务器


## 准备
* VISA信用卡


## 开始
1. 到[亚马逊](https://aws.amazon.com/cn)注册账号
2. 按步骤，需要使用信用卡，会扣除$1的手续费，有个心理准备
3. 到[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)创建实例，选择linux Ubuntu服务器，有部分服务器是要钱的
4. 一直下一步走，会有提示免费的选项，选择免费的就行，在最后会让你创建秘钥，输入名字创建，并且下载到本地`xxx.pem`（只有一次下载机会）然后完成
5. 到[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)查看正在运行的实例，这个时候应该只有一个，可以看到实例的公网ip，等信息
6. 这个时候需要用秘钥与服务器配对连接
  * 进入终端修改秘钥文件的权限：chmod 400 xxx.pem
  * 默认用户为`ubuntu`，连接公网服务器：ssh -i xxx.pem ubuntu@[公网ip]（如：ssh -i ABC.pem ubuntu@52.37.197.133），以后也是通过这个连接的
  * ok，连接成功
  * 接下来就可以对服务器进行操作了



## 折腾python
### 1. 升级python
执行命令`python --version`可以看到ubuntu系统的自带python2.7.6，升级一下
```bash
$ python --version
Python 2.7.6
```

升级前可能需要安装`gcc`和`make`
```bash
# 安装yum
$ sudo apt-get install yum


# 先更新一下源
$ sudo apt-get update
$ sudo apt-get install gcc
$ sudo apt-get install make
$ sudo apt-get install zlibc zlib1g-dev
$ sudo apt-get install libssl-dev
```

升级python
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


进入`/usr/local/bin/`目录发现我们的python程序就在这个目录`python2.7`，在终端输入`python2.7.9`可以进入刚安装的版本，但是太麻烦了，这个时候改一下默认版本（有些版本安装后会自动改）
```bash
#  //对系统默认版本python进行操作，改名
$ sudo mv /usr/bin/python /usr/bin/python_old
$ ln -s /usr/local/lib/python2.7.12/bin/python /usr/bin/python
```



## 2. 安装pip和easy_install
安装easy_install
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

这个时候不能直接用`pip`命令，我们链接一下
```bash
$ ln -s /usr/local/lib/python2.7.12/bin/pip /usr/bin/pip
```

注意，python2.7.12自带pip和easy_install，如果系统已经有了pip和easy_install，我们需要改成新版本的pip和easy_install，通过which查看当前的路径
```bash
$ which pip
/usr/bin/pip
```

删除原来的pip程序并链接新的pip程序
```bash
# pip
$ sudo rm /usr/bin/pip
$ sudo ln -s /usr/local/python2.7.12/bin/pip /usr/bin/pip

# easy_install
$ sudo rm /usr/bin/easy_install
$ sudo ln -s /usr/local/lib/python2.7.12/bin/easy_install /usr/bin/easy_install

```


下载gcc
```
$ wget http://ftp.gnu.org/gnu/gcc/gcc-6.1.0/gcc-6.1.0.tar.bz2
```

## 3. 安装lxml
* Python 2.3+
* 安装依赖包`libxml2`, `libxslt`, `python-libxml2`
  ```bash
  $ sudo apt-get install libxml2 libxml2-dev
  $ sudo apt-get install libxslt1-dev
  $ sudo apt-get install python-libxml2
  ```
  > 未安装上面依赖会报gcc编译错误
  > [http://www.cnblogs.com/lyroge/archive/2013/02/22/2922515.html](http://www.cnblogs.com/lyroge/archive/2013/02/22/2922515.html)

* 安装lxml
  ```bash
  $ sudo easy_install lxml
  ```


## 3. 安装scrapy爬虫框架
```bash
$ sudo pip install Scrapy
```

## 4. 安装mysql
可以同yum命令查看是否安装过，如果没有任何输出，说明没有安装
```bash
$ sudo netstat -tap | grep mysql
```

安装`mysql-server`, `mysql-client`
```bash
$ sudo apt-get install mysql-server mysql-client
```
安装过程会让你输入`root`用户的密码，输入后按`Tap`键下一步


安装完成，测试是否成功安装


登陆看看
```bash
$ mysql -uroot -p
```
然后输入密码，ok



https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz

安装MYSQL for python
```bash
$ sudo pip install MySQL-python
```



```bash
$ wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.13-linux-glibc2.5-x86_64.tar.gz

$ tar vzxf mysql-5.7.13-linux-glibc2.5-x86_64.tar.gz
$ sudo cp mysql-5.7.13-linux-glibc2.5-x86_64 /usr/local/mysql -r

$ cd /usr/local/mysql/

# 修改当前目录拥有者为mysql用户
$ sudo chown -R mysql:mysql ./

# 安装数据库：执行命令
$ sudo bin/mysqld --initialize-insecure --console

# 修改当前目录拥有者为root用户：执行命令
$ sudo chown -R root:root ./

# 修改当前data目录拥有者为mysql用户：执行命令
$ sudo chown -R mysql:mysql data

# 把启动脚本放到开机初始化目录
$ sudo cp support-files/mysql.server /etc/init.d/mysql
```

安装可以通过`apt-get`命令直接安装
```bash
$ sudo apt-get -f install
$ sudo apt-get -f install mysql-server
```

```bash
# 先装common
$ sudo dpkg -i mysql-common_5.7.13-1ubuntu16.04_amd64.deb
# 装
$ sudo dpkg -i mysql-community-server_5.7.13-1ubuntu16.04_amd64.deb
```
