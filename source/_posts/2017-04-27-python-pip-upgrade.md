---
title: python-pip-upgrade
categories: python
tags:
  - python
date: 2017-04-27 14:52:45
updated: 2017-04-27 14:52:45
---


## 升级单个类库
```bash
$ pip install --upgrade flask
```

## 升级所有类库
由于pip不支持批量操作，这里通过python代码升级
```python
import pip
from subprocess import call

for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

## 导出已安装的类库配置
```bash
$ pip freeze > requirements.txt
```

## 安装类库配置
```bash
$ pip install -r requirements.txt
```