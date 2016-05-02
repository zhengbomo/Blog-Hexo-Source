---
title: ReactiveCocoa初识
tags:
  - MVVM
categories: iOS
date: 2016-04-19 01:11:12
---


## 简介
ReactiveCocoa简称RAC
在讲RAC之前，我们先来看看iOS上处理事件的方式，在iOS上，处理事件主要有一下几种方式

<!-- more -->

* Delegate
* Notification
* KVO
* block
* AddTarget

应用程序的开发，基本上也是在处理不同的事件，例如应用程序启动事件（AppDelegate），用户响应事件（TouchEvent），后台服务响应事件（Service）
上面五种行为都可以视为事件处理，但是有些使用起来特别麻烦或者不方便，例如Notification和KVO更是反人类的用字符串处理，RAC统一包含上面五种方式的了绝大多数的事件处理方式，把事件封装成信号量，然后信号流入管道进行处理，下面先对比普通的事件处理和RAC的事件处理

## 事件机制

##

## 信号传输

## 原理
