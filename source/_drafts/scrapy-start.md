---
title: scrapy-start
date: 2016-06-21 16:56:45
updated: 2016-06-21 16:56:45
categories:
tags:
---


scrapy是python最有名的爬虫框架之一，这里记录简单学习的过程，下面的一些例子摘自[官方教程](http://doc.scrapy.org/en/latest/intro/tutorial.html)

1. 创建scrapy项目
2. 定义需要提取的模型
3. 定义一个spider爬去网页数据并提取结构化数据
4. 保存结构化数据


使用下面命令创建一个scrapy项目
```bash
$ scrapy startproject tutorial
```

会在当前目录创建一个`tutorial`文件夹，下面是目录结构
```
tutorial/
    scrapy.cfg            # deploy configuration file

    tutorial/             # project's Python module, you'll import your code from here
        __init__.py
        items.py          # project items file
        pipelines.py      # project pipelines file
        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

在`tutorial`目录下可以看到`items.py`文件，在这里可以定义结构化数据
```
```
