---
title: swift反射
date: 2016-07-14 11:51:24
updated: 2016-07-14 11:51:24
categories: iOS
tags: swift
---


* 在swift中的所有类都属于Any类型，但不支持元组
```swift
var things = [Any]()
things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
//things.append((3.0, 5.0))
things.append(NSObject())
```
