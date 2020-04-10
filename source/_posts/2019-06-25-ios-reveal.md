---
title: 使用Reveal查看AppUI结构
tags: [iOS, 逆向, Reveal]
date: 2019-06-25 23:37:56
updated: 2019-06-25 23:37:56
categories: iOS
---

在开发中，我们可能需要参考其它app界面的实现方式来寻找开发思路，通过Reveal工具，我们可以很方便的查看App在内存中的视图结构，如下(AppStore)

<!-- more -->

![RevealAppstore](/images/post/reveal_appstore.png)

## 准备

1. 一台越狱的手机
2. [Reveal](https://xclient.info/s/reveal.html)，推荐使用v4以上的版本，支持USB链接，速度快

## 手机安装Reveal2Loader插件

在Cydia搜索`Reveal2Loader`，该插件在`BissBoss`源，直接就能搜到，安装

{% img /images/post/reveal2loader.jpg 300 Reveal2Loader %}

安装完成后重启SpringBoard

{% img /images/post/restartspringboard.jpg 300 RestartSpringBoard %}

## 拷贝Reveal服务文件到iPhone中

打开mac上的Reveal，`Help`->`Show Reveal Library in Finder`->`iOS Library`
![Reveal2Loader](/images/post/reveal_ios_library.png)

进入目录`/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework`，也可以直接打开这个目录

将`RevealServer.framework`库中的`RevealServer`拷贝到手机`Library/RHRevealLoader/`并重命名为`libReveal.dylib`

```sh
# 进入目录
cd /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework

# 如果手机上没有/Library/RHRevealLoader这个目录，需要先创建一下
scp RevealServer root@xx.xx.xx.xx:/Library/RHRevealLoader/libReveal.dylib
```

将`RevealServer.framework`复制到手机的`/System/Library`中

```sh
# 进入目录
$ cd /Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries

# 远程拷贝目录
$ scp -r RevealServer.framework root@xx.xx.xx.xx://System/Library/RevealServer.framework
```

## 重启手机

```sh
killall SpringBoard
```

这时候设置里面会出现`Reveal`选项
{% img /images/post/setting_reveal.jpg 300 SettingReveal %}

我们进入`Enabled Applications`打开`AppStore`
{% img /images/post/reveal_list_appstore.jpg 300 EnableAppStore %}

打开Mac上的Reveal，打开手机上的AppStore，可以看到Reveal识别到AppStore
![RevealList](/images/post/reveal_list.png)

进入查看视图
![RevealAppstore](/images/post/reveal_appstore.png)

Reveal可以看到视图结构，内存地址，还能看到`View`对应的`ViewController`
