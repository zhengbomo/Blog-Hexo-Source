---
title: ijkplayer编译
date: 2016-04-20 12:00:58
update: 2016-04-29 18:59:37
tags: [ffmpeg]
categories: iOS
---

编译哔哩哔哩开源的[ijkplayer](https://github.com/Bilibili/ijkplayer) iOS版本记录，只是为了更方便使用
ijkplayer基于[ffmpeg](https://ffmpeg.org/)，几乎支持所有视频，音频格式，最低支持到iOS6，在低端机如iphone4，itouch4上测试运行效果良好，搞播放器的同学可以基于这个来做，节省不少时间

<!-- more -->

### 编译前
编译前的准备，需要安装`homebrew`, `git`, `yasm`

```bash
# install homebrew, git, yasm
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install git
brew install yasm
```

### 设置编译脚本
使用`module-default.sh`脚本，默认情况下`module.sh`指向的是`module-lite-hevc.sh`
```bash
cd config
rm module.sh
ln -s module-default.sh module.sh

cd ../ios
sh compile-ffmpeg.sh clean
```

### 编译
编译ffmpeg库
```bash
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
git checkout -B latest k0.5.1

./init-ios.sh
# 这一步会等待一段时间，需要下载ffmpeg源码

cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```

接下来是漫长的等待...  

编译完成后得到6个lib
```bash
├── ios/build/universal/lib
|                        ├── libavcodec.a    
|                        ├── libavfilter.a    
|                        ├── libavformat.a    
|                        ├── libavutil.a    
|                        ├── libswresample.a    
|                        ├── libswscale.a    
```

## 编译ijkplayer库
打开ios/IJKMediaDemo/IJKMediaDemo.xcodeproj，编译通过，默认编译为当前architecture（CPU架构）的库，为了方便使用，我们需要framework编译成多架构的库（armv7 i386 x86_64 arm64）我们只需要引用一个文件，就可以在不同的CPU架构使用了

关于库的合并，可以参考[这里](/2016-04-29/merge-ios-lib)

编译完成，得到`IJKMediaFramework.framework`，支持 armv7 i386 x86_64 arm64

由于官方的pod失效了，编译一次比较麻烦费时间，这里编译好放在github上可以直接下载来用

[https://github.com/zhengbomo/ijkplayer.framework](https://github.com/zhengbomo/ijkplayer.framework)
