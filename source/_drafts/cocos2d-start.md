---
title: cocos2d-start
date: 2016-08-06 14:19:45
updated: 2016-08-06 14:19:45
categories:
tags:
---

cocos2d环境搭建

官网：[http://cocos2d-x.org/](http://cocos2d-x.org/)

如果需要使用Android，则需要安装如下依赖
* [Android SDK](http://developer.android.com/sdk/index.html)
* [NDK](http://developer.android.com/sdk/index.html)
* [ANT](http://ant.apache.org)

Windows平台需要安装Python

1. 到官网下载[cocos2d-x-3.12](http://www.cocos2d-x.org/filedown/start/339)
2. 解压，到终端进入解压出来的目录，执行
  ```bash
  $ ./setup.py
  ```
  过程中会提示设置AndroidSDK, NDK, ANT的安装目录，可跳过
3. 按提示执行下面命令，使配置生效
  ```bash
  $ source /Users/Zhengxiankai/.bash_profile
  ```
4. 通过命令`cocos -h`查看帮助命令
5. 查看生成项目的命令帮助`cocos new -h`
6. 生成项目
  ```bash
  $ cocos new -p com.bomo.coco -l cpp -d /Users/zhengxiankai/Desktop/Document/cocoa2dx/ hellocc3
  ```
7. 这时候会在设置的目录生成项目文件`/hellocc3/proj.ios_mac/hellocc3.xcodeproj`，用Xcode打开，运行成功

8. 在下载的cocos2d-x解压目录，有个`build`目录，里面有个`cocos2d_tests.xcodeproj`工程，里面的项目会演示cocos2d-x的所有功能demo，是个很好的学习例子
