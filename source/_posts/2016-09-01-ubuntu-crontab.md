---
title: ubuntu crontab定时任务
categories: linux
tags: linux
date: 2016-09-01 23:48:43
updated: 2016-09-01 23:48:43
---


最近在写一个爬虫，需要每天爬取一定量的数据，在自己机器做就太麻烦了，如果能放在服务器上自动跑就好了，找了一下linux系统有个`crontab`的工具可以用来设置定时任务，这里简单记录一下使用方法

<!-- more -->

## 一、定义任务
这里定义个python脚本（`daily_task.py`），假设需要每天凌晨12点执行一次
```python
#!/usr/bin/python
# -*- coding:utf-8 -*-

def do_something():
  print 'daily task by crontab!'

do_something()
```
假设文件存放在：`/home/ubuntu/daily_task.py`

## 二、创建任务
通过`crontab -e`配置任务，如果有多个编辑器的话，会让你选择一个编辑器
```
$ crontab -e
```

输入格式：`分 时 日 月 年 脚本`
第1列分钟：1～59
第2列小时：1～23（0表示子夜）
第3列日：1～31
第4列月：1～12
第5列星期：0～6（0表示星期天）
第6列脚本

### 1. 表示单个具体时间
每天7点30分执行daily_task.py脚本
```
30 7 * * * python /home/ubuntu/daily_task.py
```

### 2. 表示多个时间（通过都好隔开）
表示每月1、10、22日的4:45重启apache。
```
45 4 1,10,22 * * /usr/local/apache/bin/apachectl restart
```

### 3. 表示时间间隔
每一小时重启apache
```
* */1 * * * /usr/local/apache/bin/apachectl restart
```

### 4. 表示时间范围
晚上11点到早上7点之间，每隔一小时重启apache
```
* 23-7/1 * * * /usr/local/apache/bin/apachectl restart
```

### 5. 通过字母表示星期
每月的4号与每周一到周三的11点重启apache
```
0 11 4 * mon-wed /usr/local/apache/bin/apachectl restart
```
### 6. 通过字母表示月份
1月1号的4点重启apache
```
0 4 1 jan * /usr/local/apache/bin/apachectl restart
```
编辑完成后，退出保存，并保存cront文件


## 三、测试
我们创建一个简单的任务，没两分钟执行一次
```
*/2 * * * * python /home/ubuntu/python/daily_task.py >> /home/ubuntu/python/crontest.py.log
```

可以在该文件中配置多个任务，编辑完成后，我们查看一下cron的状态
```bash
$ sudo service cron status
cron start/running, process 29899
```

如果没有启动，我们重启一下cron服务
```bash
$ sudo service cron restart
cron stop/waiting
cron start/running, process 29899
```

可以通过``命令查看当前的配置（也就是刚刚配置的文件）
```bash
$ crontab -l
```

可以吧刚刚配置的文件保存到自定义的位置
```bash
$ crontab -l > /home/ubuntu/python/mycron.config
```

如果已经有了配置文件，可以设置文件路径
```bash
$ crontab < /home/ubuntu/python/mycron.config
```
