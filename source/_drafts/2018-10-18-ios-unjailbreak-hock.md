---
title: iOS非越狱手机hock
tags: [iOS]
date: 2018-10-18 14:27:00
updated:
categories: iOS
---

由于iOS的封闭性，相比Android来说，相对安全，iOS开发者的安全意识总的来说并不高，很多时候并不考虑太多安全的问题，由于ObjC语言的动态性，是的我们很容易对OC代码进行hock操作，对ipa包进行hock，重签到非越狱手机上，可以很方便的做到，各种微商软件，就是利用这点做到篡改App的

<!-- more -->

## 准备
1. 砸壳IPA：可以到越狱市场（PP助手）下载，可以用越狱手机导出，企业签，AdHoc的包不需要砸壳的
2. [yololib](https://github.com/KJCracks/yololib): 用于将`dylib`注入可执行文件的加载列表中
3. [app-signer](https://github.com/zhengbomo/ios-app-signer): 重签名工具
4. [Theos](https://github.com/theos/theos.git): 越狱开发工具包，用于编写`dylib`

## 流程
1. 获取已砸壳的IPA包
2. 解压得到app包，得到可执行程序
3. 编写`dylib`动态库和`hock`的逻辑
4. 把`dylib`注入到可执行文件的加载库中
5. 把`dylib`拷贝到`.app`包的根目录
5. 使用开发证书重签名，安装

## 砸壳
### 1. 通过越狱手机砸壳
这里意见有人写过了一键砸壳的教程：[frida-ios-dump一键砸壳详细版](https://www.jianshu.com/p/cfe852110e8a)


### 2. 通过越狱市场下载，常见的由PP助手，这里有一个快速搜索和下载IPA的工具[IPASearch](https://github.com/ytx0574/IPASearch)，用的就是PP助手的的接口
