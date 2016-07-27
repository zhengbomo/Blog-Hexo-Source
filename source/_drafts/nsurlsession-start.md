---
title: NSURLSession
date: 2016-07-20 16:58:18
updated: 2016-07-20 16:58:18
categories:
tags:
---


NSURLSession是子iOS7之后引入的网络库，而在这之前，我们使用NSURLConnection进行网络请求（iOS9以后弃用）

## 1. 基本使用
```swift
// 获取一个共享的session
let session = NSURLSession.sharedSession()
let request = NSURLRequest(URL: NSURL(string: "http://baidu.com")!)

// 构建一个task（默认为挂起状态）
let task = session.dataTaskWithRequest(request, completionHandler: { (data, response, error) -> Void in
    // 回调（后台线程）
    let string = NSString(data: data, encoding: NSUTF8StringEncoding)
    println(string)
})

// 启动task
task.resume()
```

共享的NSURLSession采用的是 “异步阻塞” 模型，即session不能同时进行多个请求，如果有多个，将会逐个执行



## 1. 三种工作模式
默认会话模式（default）：工作模式类似于原来的NSURLConnection，使用的是基于磁盘缓存的持久化策略，使用用户keychain中保存的证书进行认证授权。

瞬时会话模式（ephemeral）：该模式不使用磁盘保存任何数据。所有和会话相关的caches，证书，cookies等都被保存在RAM中，因此当程序使会话无效，这些缓存的数据就会被自动清空。

后台会话模式（background）：该模式在后台完成上传和下载，在创建Configuration对象的时候需要提供一个NSString类型的ID用于标识完成工作的后台会话。
