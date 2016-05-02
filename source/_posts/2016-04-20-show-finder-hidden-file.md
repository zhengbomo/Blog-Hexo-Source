---
title: Mac上让Finder显示隐藏文件
date: 2016-04-20 12:00:58
tags: [Mac]
categories: 技术
---


> 在Mac上默认不显示隐藏文件，对于开发人员来说，有时候需要修改一些隐藏文件中的配置，或是删除隐藏文件，在Finder上操作显得特别麻烦，可以在终端用下面命令让Finder的显示/不显示隐藏文件

<!-- more -->

## OSX Mavericks或 OSX Yosemite 系统以上

显示隐藏文件
```shell
$ defaults write com.apple.finder AppleShowAllFiles -boolean true ; killall Finder
```

不显示隐藏文件
```shell
$ defaults write com.apple.finder AppleShowAllFiles -boolean false ; killall Finder
```

## OSX Mountain Lion之前

显示隐藏文件
```shell
$ defaults write com.apple.finder AppleShowAllFiles TRUE ; killall Finder
```

不显示隐藏文件
```shell
$ defaults write com.apple.finder AppleShowAllFiles FALSE ; killall Finder
```
