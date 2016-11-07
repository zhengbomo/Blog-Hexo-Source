---
title: YYCache-start
date: 2016-05-24 15:09:26
updated: 2016-05-24 15:09:26
categories:
tags:
---


# 缓存
内存缓存：高速，容量小
磁盘缓存：低俗，容量大

通常的缓存都是内存缓存与磁盘缓存结合使用

## API
NSCache系统自带的缓存工具，API与NSDictionary类似，但是有下面几个特点
* 并且是线程安全的
* 在内存不足的情况下能自动释放缓存的资源
* 不会retain key (键不需要实现 NSCopying 协议)
  [NSCache](http://nshipster.cn/nscache/)

```objc
NSCache *cache = p[NSCache alloc] init];
cache.countLimit = 30;      //限制数量为30
[cache setObject:data forKey:key];
[cache setObject:data forKey:key cost:2];

id value = cache[key];
```

## 缓存淘汰规则
* FIFO（First in First out）先进先出

* LRU（Least-Recently-Used）最近最少使用
  ![LRU淘汰过程](http://image76.360doc.com/DownloadImg/2014/07/0409/43139082_1.png)

* LFU（Least-Frequently-Used）最不常用
  ![LFU淘汰过程](http://doc.okbase.net/picture/addon/2014/07/06/A171451881-83654.png)

## YYKit

### 内存缓存
使用一个线程队列维护
* 缓存节点：YYLinkedMapNode（key,value, prev, next, cost, time）
* 缓存队列：YYLinkedMap：为缓存队列提供插入，移动，删除的操作  
  使用链表，而不是数组，快速移动节点，使用双向链表，可以快速改变方向

* 缓存列表管理：YYMemoryCache
  * 使用LRU淘汰机制
  * 使用Timer递归扫描缓存数据（容量，数量，时间）

### 磁盘缓存
磁盘缓存基本上有下面几种
* 基于文件读写（SDWebImage）
  基于文件缓存有点很明显，可以缓存大数据，性能相近，但也有缺点，结构单一，不利于扩展，不支持元数据，很难实现一些复杂的淘汰算法，数据统计慢

* 基于mmap文件内存映射（[FastImageCache](https://github.com/path/FastImageCache)）

* 基于数据库（NSURLCache）
  优缺点基本上与文件缓存相反，方便扩展，方便统计，但是存储大量会严重影响性能

比较好的方式是结合数据库和文件存储来实现磁盘缓存

YYKit实现
* YYKVStorageItem：缓存节点
* YYKVStorage：数据管理（文件与数据库存储）
* YYDiskCache：缓存管理



### 问题：
> CFMutableDictionaryRef 与MutableDictionary
> NSHashTable, NSMapTable
> NSSet, NSDictionary, NSCache


# 引用
* [专访YYKit作者郭曜源](http://www.infoq.com/cn/news/2015/11/ibireme-interview?utm_source=tuicool&utm_medium=referral)
* [YYCache设计细节](http://blog.ibireme.com/2015/10/26/yycache/)
