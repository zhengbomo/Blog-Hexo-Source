---
title: swift-cache-start
date: 2016-07-20 14:47:56
updated: 2016-07-20 14:47:56
categories:
tags:
---


## 思路
1. 调度器Dispatcher
  控制一对多，调度控制
2. 缓存管理器CacheManager
  实现一级缓存，二级缓存
3. 下载器Downloader
  下载器控制多线程下载，可以控制线程数量等
