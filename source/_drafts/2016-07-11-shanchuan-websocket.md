---
title: shanchuan-websocket
date: 2016-07-11 10:40:00
updated: 2016-07-11 10:40:00
categories:
tags:
---


闪传WebSocket相关探索

闪传网页端：[http://web.shanchuan.cn/](http://web.shanchuan.cn/)

## 一、数据传输
通过Chrome/Wireshark/Charles抓包查看网络请求

1、Web端与服务器
  * Web上的与服务器交互所有协议信息请求都走WebSocket，通过`ws://54.223.165.156:8080/wschannel?id=154a1a9c-aea0-42a9-b98d-f044b52915b4`，不管是局域网和广域网都会走这个通道
  * 心跳包也是通过上面socket信道发送，30秒一次
  * 所有的数据请求通过websocket信道传输，包括图片列表，文件列表，相册列表等，数据格式为json
  * 所有资源列表一次加载，后续操作不加载
  * 如果移动设备和Web端不在同一局域网，所有的资源请求走广域网，如：`http://web.shanchuan.cn/respImage/154a1a9c-aea0-42a9-b98d-f044b52915b4/image/thumbnail/IMG_3856.JPG`
  * Web上的所有的资源请求都走http，如果在同一局域网，走局域网，如果不是，走广域网


2、Web端与客户端（局域网）
  * Web端与客户端所有交互都走http，包括通信信道，和资源请求
  * 如果Web端和客户端在同一局域网（同一路由器），所有通信走客户端的http服务器
  * 客户端通过轮训向服务器请求实现与服务器的交互，如果超时，返回心跳


3、客户端(Android/iOS)与服务器
  * 前期信道数据使用Socket发送到服务器（非WebSocket）
  * 部分信道数据使用Http发送消息到服务器（如后期与Web交互的数据PlayTo等）
  * 资源数据使用http发送到服务器


## 二、WebSocket
> 针对Http解决Http无法进行长连接的短板而一项基于Socket链接的接口，用法与Socket类似，首次链接通过类似Http头链接，之后通过发送Frame数据维护双方连接状态（通过心跳包实现）
> http也可以通过http请求实现心跳包，达到长连接的目的，但是由于http协议头相对于心跳包本身的数据过于冗余，会很大程度上浪费双方通信的带宽，而WebSocket在传输过程中使用的数据只有一个2byte的头数据，不像http请求那么多

WebSocket是一种协议，与Http一样，基于TCP协议，解决http长连接的问题，解决http轮训数据包过于臃肿的问题，用法与Socket类似

首次连接通过http发送连接请求，并包装，连接成功后升级协议为websocket，之后所有数据走websocket请求

由于客户端可以直接使用Socket，不存在上述缺陷，故WebSocket是为Web端而生的，大部分情况下只用于web端

### iOS上相关的WebSocket库
* [SocketRocket](https://github.com/facebook/SocketRocket)
  该库只有3个源文件，使用方便，简单，github star数5000+
  加入库后包体积增加`44k`（45057字节）
      添加前：21,072,309 字节
      添加后：21,117,365 字节
* [RockemSockem](https://github.com/ReactiveCocoa/RockemSockem)
  依赖ReactiveCocoa，github star数200+，

### Android上相关WebSocket库
* [autobahn](https://github.com/crossbario/autobahn-android)
  通过一个空工程导入后包体积新增`117kb`
      加入库之前：279kb
      加入库之后：396kb

* [android-websockets](https://github.com/codebutler/android-websockets)
* [AndroidAsync](https://github.com/koush/AndroidAsync)
  包含socket, http (client+server), websocket, and socket.io，并提供Async支持


## 三、参考资料
* [http://www.open-open.com/lib/view/open1464313314082.html](http://www.open-open.com/lib/view/open1464313314082.html)
* [https://github.com/facebook/SocketRocket](https://github.com/facebook/SocketRocket)
* [https://www.zhihu.com/question/20215561](https://www.zhihu.com/question/20215561)
