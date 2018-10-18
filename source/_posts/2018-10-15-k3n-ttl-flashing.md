---
title: 斐讯K3N通过ttl刷K3固件
tags: [路由器, 斐讯]
date: 2018-10-15 08:01:56
updated:
categories: 路由器
---


K3N刚发布，斐讯就翻车了，K3N基本没有讨论的帖子，论坛基本是K3的帖子，好在K3和K3N的固件是通用的，通过TTL可以直接刷入K33的固件，在这里记录一下刷机的过程

<!-- more -->

## 准备
1. K3N和K3的固件是通用的
2. K3N目前只能通过TTL刷机（需要拆机）
3. TTL刷机需要用到USB转TTL刷机板，由于K3主板上的TTL是圆孔的，所以还需要插针，淘宝，下面是我购买的链接：  
    * 刷机板：[https://detail.tmall.com/item.htm?id=577003848649](https://detail.tmall.com/item.htm?id=577003848649)
    * 插针：[https://detail.tmall.com/item.htm?id=41428876908](https://detail.tmall.com/item.htm?id=41428876908)
4. PC一台

## 工具
1. 固件：只要是k3的固件都可以，我用的abcc的官改固件（其他K3固件也可以），下载后得到`k3_v18.bin`    
[http://www.right.com.cn/forum/thread-259012-1-1.html](http://www.right.com.cn/forum/thread-259012-1-1.html)，
2. [Tftpd64](http://tftpd32.jounin.net/tftpd32_download.html)：用于从PC传输固件到路由器上
3. [SecureCRT](): 用于连接刷机板
4. 单片机驱动，购买了刷机板的可以找厂家要，我这里用的是CH340：[http://www.winchiphead.com/download/CH341/CH341SER.ZIP](http://www.winchiphead.com/download/CH341/CH341SER.ZIP)，需要先卸载驱动，再安装

工具打包：  
链接: https://pan.baidu.com/s/185d8QLJNuF88pXIjJNaLhA 提取码: 6dxm


## 拆机
第一步肯定是要拆机了，下面是几个K3的拆机贴，跟K3N是一样的，可以参考下

* [小白K3拆机教程，易](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=261298&page=1)
* [K3拆机高清图片，给即将拆机的朋友一些参考 ](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=313112&page=1)
* [斐讯k3拆机 ttl救砖 教程](https://www.bilibili.com/video/av25547854)

最后得到主板如下
![](/images/post/1501539561946_.pic.jpg)

非接口一端有四个圆孔
![](/images/post/1521539561960_.pic.jpg)

四个孔分别是`TX`, `RX`, `GND`, `VCC`，我们通过这四个孔连接刷机版

刷机板也有这四个接口
![](/images/post/9f7dec8a-91fa-4f31-b02c-7a38d50cf7f0.png)

## 连接

1. 插入刷机版到电脑（注意先安装驱动，安装驱动的时候不要插刷机板），在设备管理器中可以看到，我这里是`COM3`
![](/images/post/5261539508714_.pic_hd.jpg)
2. 打开`SecureCRT`，选择快速连接，设置如下，然后点击连接
    Protocol -> Serial
    Port -> COM3 USB-SERIAL CH340
    Baud rate -> 115200
    Data bits -> 8

![]()
3. 这个时候可以可以看到session为绿色，是连上了，但是没有数据
![](/images/post/WechatIMG527.png)

4. 连接刷机版，刷机板一端连接PC，另一端连接路由器的TTL
```
RXD -> RX
TXD -> TX
GND -> GND
```
    还有一种接线方式GND接GND，RXD接TXD，TXD接RXD，我的板就是这种，如果接反了，会读不到数据
![](/images/post/1539572232493.jpg)

    我在刷机板的线接了四个插针，插入K3N主板的四个孔，用东西卡住，避免接触不良，可以不用焊接
![](/images/post/1539571592462.jpg)

    只需要接三条线，`VCC`口不用接

5. 路由器接上网线（网线连接PC）和电源，电源先不要开，路由器主板最好把屏幕也连接上，方便看进度
![](/images/post/1539572736618.jpg)

## 刷机
1. 按住reset键，打开路由器电源
可以看到`SecureCRT`有数据输出，并且可以看到路由器的IP（`172.16.10.1`）不同路由器IP可能不一样，最后一行为`CFE>`，这时候可以放开reset键
![](/images/post/1539572872063.jpg)

    如果能看到IP地址，就说明成功不远了

2. 电脑设置网卡的IP，如下，ip地址可以设置为`172.16.10.100`，不要跟路由器的IP一样，这时候电脑的网卡可能显示没有网络或者断开，不用管，设置就行了
![](/images/post/1539573194674.jpg)

3. 打开`Tftpd64`，选择固件的目录和网卡，如下
之前准备的固件`k3_18.bin`放在选择的目录下，并改名为`k3.trx`
选择网卡的时候可以看到刚刚设置的IP（`172.16.10.100`）
![](/images/post/1539573297662.jpg)

4. 在`SecureCRT`输入下面命令
注意替换成上面设置的IP，文件名我们改为了`k3.trx`
```shell
flash -noheader 172.16.10.100:k3.trx nflash0.trx
```
    这个过程有点久，路由器会先从PC下载固件（通过`Tftpd64`），可以在`Tftpd64`看到进度
    下载完成后在`SecureCRT`会看到输出
![](/images/post/1539573633154.jpg)

5. 执行完成之后路由器会重启，如果连接了显示器，可以看到路由器缓慢的启动，启动完会提示系统升级，然后再重启，到这里刷机完成，可以愉快的玩耍了
![](/images/post/1539573821336.jpg)






