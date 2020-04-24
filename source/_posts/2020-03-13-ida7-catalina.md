---
title: MacOS 10.15安装IDA7
tags: [iOS, 逆向]
date: 2020-03-13 22:01:20
updated: 2020-03-13 22:01:20
categories: iOS
---

MacOS 10.15对系统做了比较大的改动，之前的`IDA7.0`无法在10.15上安装，而在10.14安装是正常的，解决方案是通过`MacOS 10.14`安装，然后把安装好的文件放到10.15系统运行

<!-- more -->

## Mac 10.14安装IDA

1. 下载并安装虚拟机[`VMware Fusion`](https://xclient.info/s/vmware-fusion.html)

2. 下载并安装`MacOS 10.14`镜像
    > 链接: [https://pan.baidu.com/s/1XVPCyecg4xIbxnvR6_ukDA](https://pan.baidu.com/s/1XVPCyecg4xIbxnvR6_ukDA) 提取码: y8xn

3. 下载[`IDA7.0`](http://www.pc6.com/mac/566964.html)，安装到MacOS10.14上

安装完得到

{% img /images/post/ida-app.png 500 %}

## 复制到Mac 10.15

拷贝10.14系统下的到`/Applications/IDA Pro 7.0`目录到10.15系统的`/Applications/`下

1. 直接打开会直接崩溃
    {% img /images/post/ida-catalina-crash.png 500 %}

    需要替换一下`libqcocoa.dylib`文件，在[这里](https://github.com/fjh658/IDA7.0_SP)可以找到

    ```sh
    /Applications/IDA Pro 7.0/ida.app/Contents/PlugIns/platforms/libqcocoa.dylib
    ```

2. 每次打开会报授权过期
    解压`Fixes/IDA Mac 7 pacth.7z`并替换下面文件

    ```sh
    /Applications/IDA Pro 7.0/ida.app/Contents/MacOS/ida
    /Applications/IDA Pro 7.0/ida.app/Contents/MacOS/ida64
    /Applications/IDA Pro 7.0/ida.app/Contents/MacOS/libida.dylib
    /Applications/IDA Pro 7.0/ida.app/Contents/MacOS/libida64.dylib
    ```

3. 打开`ida64`如果出现无法打开的情况
   {% img /images/post/ida-catalina2.png 500 %}

   可能是权限问题

   ```sh
    # 如果没有执行权限，需要先添加，之后就能直接启动了
    cd /Applications/IDA Pro 7.0/ida.app/Contents/MacOS
    chmod +x ida
    chmod +x ida64
    ```

    {% img /images/post/ida-catalina3.png 600 %}

下面是我处理后的文件，可以直接拿到`Mac10.15`使用

> 链接:[https://pan.baidu.com/s/1DwkW2ICWev5FhIqOHuq6BA](https://pan.baidu.com/s/1DwkW2ICWev5FhIqOHuq6BA)  密码:kzxx
