---
layout: post
title: 使用crontab运行python脚本
date: '2016-10-06 22:52'
categories: python
tags:
  - crontab
  - python
---

之前使用crontab经常遇到一些问题，例如脚本不执行的问题，报错问题，在这里记录一下过程，先介绍一个python工具，可以自动更改和生成crontab运行脚本，使用起来非常友好：[plan](http://github.com/fengsp/plan)

<!-- more -->
## 一、安装
```bash
$ pip install plan
```
## 二、创建任务脚本
```python
from plan import Plan

cron = Plan()

# register one command, script or module
# cron.command('command', every='1.day')
# cron.script('script.py', path='/web/yourproject/scripts', every='1.month')
# cron.module('calendar', every='feburary', at='day.3')

if __name__ == "__main__":
    cron.run()
```

也可以使用下面命令自动创建，会在`schedule`目录下生成一个`schedule.py`文件
```
$ mkdir schedule
$ cd schedule
$ plan-quickstart
```

## 三、使用
直接在crontab配置文件中使用执行python脚本遇到了好多坑，最后还是使用sh脚本来运行，我们在schedule目录下创建一个文件夹`bash`，新增一个sh脚本文件`OneMinitesTask.sh`
```sh
#!/bin/sh
# 当前路径为: schedule/bash
cd ..;

# 激活虚拟环境
source env/bin/activate;

# 执行python脚本
python OneMinitesTask.py

# 退出虚拟环境
deactivate
```
创建完后我们还需要保证添加可执行权限
```bash
$ chmod +x OneMinitesTask.sh
```

下面是`schedule.py`
```python
from plan import Plan

# 获取bash路径（由于这里依赖于getcwd方法，所以要确保运行目录为当前目录）
bash_path = os.path.abspath(os.path.join(os.getcwd(), "bash"))

cron = Plan()

# 添加一个任务，每分钟执行一次TenMinutesTask.sh脚本
# 这里的every参数详细介绍见官网：http://plan.readthedocs.io/job_definition.html#every
job = Job('./OneMinutesTask.sh > hello.txt', path=bash_path, every='1.minute')

# 添加到plan上，可以添加多个job
cron.job(job)

if __name__ == "__main__":
    # 输出到文件，覆盖原有的任务，这里还有其他的参数，详情见官网：http://plan.readthedocs.io/run_types.html
    cron.run('white')
```

运行脚本
```bash
$ cd schedule
$ python schedule.py
[write] crontab file written
```

查看crontab任务
```bash
$ crontab -l
# Begin Plan generated jobs for: main
* * * * * cd /home/ubuntu/schedule/bash && ./OneMinutesTask.sh
# End Plan generated jobs for: main
```

ok，测试运行成功，使用python脚本配置crontab运行脚本可读性更强，更好维护


## 四、常见问题
### 1. 不确定任务是否运行，可以到`/var/log/cron.log`查看crontab的运行日志
```bash
$ vim /var/log/cron.log
```
使用`Shift+G`跳到最后一行

如果没有日志也可能是log服务没有开启，到`/etc/rsyslog.d/50-default.conf`查看`cron.*              /var/log/cron.log`一行是否被注释掉了，如果注释掉了，把注释关了
```bash
sudo vim /etc/rsyslog.d/50-default.conf

# 修改完后重启rsyslog
$ sudo service rsyslog restart
```


### 2. 关闭crontab任务
如果要关闭定时crontab，使用下面命令
```bash
# 删除crontab任务，删除后通过crontab -l查看则提示无任务
$ crontab -r
```

如果crontab的定时任务启动了，由于crontab只负责定时调用任务，crontab本身不支持任务的管理，如果需要关闭后台正在运行的任务，可以通过`ps aux`命令查看任务的`PID`，然后找到我们的任务，然后使用`kill {PID}`杀掉任务即可
