---
title: iOS动态库共享缓存
tags: [iOS]
date: 2020-04-08 16:16:20
updated: 2020-04-08 16:16:20
categories: iOS
---


我们在开发的过程中，经常会用到系统自带的库，如 Foundation，UIKit 等，这些库存放在什么地方呢，我们可以通过编译好的 MachO查看这些库的路径

![macho-framework](/images/post/macho-framework.png)

这里可以看到，动态库的路径为`/System/Library/Frameworks/UIKit.framework/UIKit`，我们连接到手机查看发现，`framework文件夹` 存在，但是并没有可执行文件

![system-lib-path](/images/post/system-lib-path.png)

## 动态库共享缓存
> 从iOS 3.1开始，为了提高系统的性能，所有的系统库文件都被打包合并成一个大的缓存文件中，而原来的动态库文件则被去除了，系统直接去缓存文件中加载动态库，该共享缓存文件保存在`/System/Library/Caches/com.apple.dyld/`目录下

![dyld-cache-path](/images/post/dyld-cache-path.png)






> https://www.jianshu.com/p/d225df2f1690

## 动态库共享缓存
问题：在`/System/Library/Framework/UIKit.framework`找不到对应的执行文件
1. 处于性能考虑，苹果把常用的动态库打包到同一个缓存位置`/System/Library/Caches/com.apple.dyld/dyld_shared_cache_armXX`


## 动态库的加载
在Mac/iOS中，使用`/usr/lib/dyld`加载动态库，`[NSBundle loadBundle]`内部也是使用`dyld`



### 导出手机上的UIKit
从手机上导出UIKit的共享缓存库`/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
```sh
scp root@xx.xx.xx.xx:/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64 ~/Desktop/dyld_shared_cache_arm64
```
这个包含所有的系统缓存库，我们需要提取出单独的动态库，dyld里面包含`dsc_extractor`工具，可以用于提取共享缓存动态库里面的子库
下载[dyld](`https://opensource.apple.com/source/dyld`)库，找到`/launch-cache/dsc_extractor.cpp`文件，使用clang编译
```sh
$ clang++ -o dsc_extractor dsc_extractor.cpp
```
得到`dsc_extractor`

提取
```sh
$ ./dsc_extractor ~/Desktop/dyld_shared_cache_arm64 arm64
```