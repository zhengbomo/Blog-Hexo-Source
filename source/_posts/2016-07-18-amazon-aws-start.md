---
title: 亚马逊VPS免费一年试用
date: 2016-07-18 21:02:00
updated: 2016-07-18 21:02:00
categories: 技术
tags: [VPS]
---


一直想整个服务器玩玩，又苦于没时间没钱，最近看到亚马逊VPS服务器有免费一年的使用时间，于是整了一个，在这里记录申请和配置的过程

<!-- more -->

## 一、准备
需要用到VISA信用卡或者Master信用卡，钱的问题比较敏感，添加信用卡只是当预授权做验证用，会扣除`$1`，有个心理准备

## 二、开始
1. 到[亚马逊](https://aws.amazon.com/cn)注册账号
2. 按步骤填写，中间需要使用到信用卡，会扣除$1的手续费，在使用过程中需要注意，有很多服务是收费的，并且绑定信用卡后不会每次都询问你确认就直接扣除了（呵呵），操作的时候要小心，我使用完就直接到[账户中心](https://www.amazon.com/gp/wallet)把信用卡删除了
3. 到[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)创建实例，选择linux Ubuntu服务器（有部分服务器是要钱的）有两种版本HVM和PV，PV运行效率高一些，linux选择PV（半虚拟）就可以了，区别见[](http://stackoverflow.com/questions/22130214/amazon-ec2-ubuntupv-or-ubuntuhvm)

  ![启动实例](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/83846668.jpg)

  ![选择Ubuntu Server LTS (PV)](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/7930100.jpg)

  ![设置存储大小](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/83423717.jpg)

  ![选择实例类型](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/78094703.jpg)

  ![审核](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/59379720.jpg)

  ![启动](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/83309431.jpg)

4. 一直下一步走，会有提示免费的选项，选择免费的就行，在最后会让你创建秘钥，输入名字创建，并且下载到本地`xxx.pem`（**只有一次下载机会**）然后完成
  ![秘钥文件](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/72114717.jpg)
5. 到[https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/)查看正在运行的实例，这个时候应该只有一个，可以看到实例的公网ip，等信息
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/57044390.jpg)
6. 这个时候需要用秘钥与服务器配对连接
  * 进入终端修改秘钥文件的权限：chmod 400 xxx.pem
  * 默认用户为`ubuntu`，连接公网服务器：ssh -i xxx.pem ubuntu@[公网ip]（如：ssh -i "ABC.pem" ubuntu@52.39.168.235），以后也是通过这个连接的
  * ok，连接成功
  * 接下来就可以对服务器进行操作了
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-7-28/4340751.jpg)

  如果是windows下默认不支持`ssh`登陆，需要安装`PuTTY`，详情参见[这里](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/putty.html?icmpid=docs_ec2_console)

## 三、固定IP
默认情况下，服务器每次重启的的时候，IP可能会更变，我们在后台控制面板申请一个新的公网地址
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/22826182.jpg)

然后绑定到实例上，因为我这里已经绑定了，所以是解除关联
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/8236634.jpg)

这样在每次重启的的时候ip就不会变了

## 四、设置安全组
默认情况下服务器只能通过ssh访问，如果我们配置了web服务器，默认情况下是不能通过ip访问的，需要在安全组中添加入站规则，在后台找到实例对应的安全组
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/50658203.jpg)

找到添加入站规则（如：http）
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/24114515.jpg)
这样就可以通过http访问了，下面我们安装nginx测试一下
```bash
$ sudo add-apt-repository ppa:nginx/stable
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install nginx
```
启动nginx
```bash
$ sudo /etc/init.d/nginx start
start: Job is already running: nginx
```
这个时候我们就可以通过ip访问了
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/55317071.jpg)

## 五、绑定域名
有了固定ip绑定域名就很简单了，在dns域名解析服务器(如[dnspod](https://www.dnspod.cn/))添加一个A记录即可
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-3/17503114.jpg)
接下来就可以用域名访问了（可能有延迟）

## 六、iTerm快捷连接
iTerm基本上是Mac上必备的终端工具，功能比自带的终端强大的多，这里使用其中一个技巧来实现快速连接和登录服务器，打开iTerm配置（Cmd+,）添加Profile

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-9-6/79653608.jpg)

添加完后，通过快捷键（Cmd+o）打开profile列表，然后选择就可以自动打开新窗口执行了，特别方便
