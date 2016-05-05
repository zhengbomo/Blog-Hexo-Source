---
title: 为自己的库添加CocoaPods支持
categories: iOS
tags:
  - cocoapod
date: 2016-05-04 16:39:17
---


做iOS开发的基本上都知道cocoapod，几乎每一种语言都有一种包管理工具，如C#的Nuget，Ruby的Brew，Nodejs的npm等，当然cocoapod就是objc/swift的包管理的工具了，几乎所有的objc/swift的开源类库都挂在cocoapod上，cocoapod可以让项目很方便的引用第三方类库，今天介绍一下如果把自己的写的库挂到cocoapod上，像SDWebImage, AFNetworking一样

<!-- more -->

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-5-4/34437182.jpg)

## 1. 创建仓库
首先我们需要建立仓库用于存放我们的类库，cocoapod支持git仓库，大多数类库都存放在github上，当然也可以用别的git仓库，如[OSChina](http://git.oschina.net/)，[coding](https://coding.net)，当然，仓库必须是public的

新建仓库后，需要添加LICENCE，大多数git工具在仓库初始化的时候都可以选择添加
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-5-4/28672226.jpg)

当然最好也添加`README.md`文件，对项目做一些描述

接下来是把Responsitory clone下来，然后添加工程，添加Demo
```
├── xxx             # 库相关文件：.h文件，.m文件，.a库，bundle资源，.framework库等
├── xxxDemo         # 最好提供Demo，方便别人快速了解和熟悉你的类库，可选
├── .gitignore	    # 过滤文件规则
├── xxx.podspec	    # cocoapod库描述文件，后面说到
├── LICENCE         # 许可说明，通常用的是 MIT
└── README.md       # 仓库说明文件，可选
```

## 2. 创建podspec文件
podspec是一个库描述文件，描述一个库如何被加入到工程中，下面举个例子如JSONModel的podspec文件

```ruby
Pod::Spec.new do |s|
  s.name         = "JSONModel"
  s.version      = "1.2.0"
  s.summary      = "Magical Data Modelling Framework for JSON. Create rapidly powerful, atomic and smart data model classes."

  s.homepage     = "http://www.jsonmodel.com"
  s.license      = { :type => 'MIT', :file => 'LICENSE_jsonmodel.txt' }
  s.author       = { "Marin Todorov" => "touch-code-magazine@underplot.com" }
  s.source       = { :git => "https://github.com/icanzilb/JSONModel.git", :tag => "1.2.0" }

  # 最低支持版本
  s.ios.deployment_target = '6.0'
  s.osx.deployment_target = '10.7'
  s.watchos.deployment_target = '2.0'
  s.tvos.deployment_target = '9.0'

  # 源文件
  s.source_files = 'JSONModel/**/*.{m,h}'

  # bundle资源文件
  # s.resources = 'Assets'

  # public头文件
  s.public_header_files = 'JSONModel/**/*.h'

  # 是否开启ARC模式
  s.requires_arc = true

  # 系统依赖框架
  s.frameworks = 'Foundation', 'UIKit'

  # 第三方依赖库
  #s.dependency 'AFNetworking', '~> 2.6'

  # 静态库.a或.framework
  # s.vendored_libraries = 'lib/Mipush.a'

  # xcconfig配置
  #s.xcconfig     = {'OTHER_LDFLAGS' => '-ObjC'}
end
```
podspec可以通过下面命令创建，也可以直接拿上面内容或其他第三方库的podspec进行修改
```
$ pod spec create podspec_name
```

podspec文件是用ruby写的，基本上可以直接看得懂，关于podspec文件官方有详细的说明：
[https://guides.cocoapods.org/syntax/podspec.html](https://guides.cocoapods.org/syntax/podspec.html)

## 3. 推送到github
编辑完podspec文件后，我们把所有文件push到github，push之前，如果在podspec设置source关联tag，见[这里](https://guides.cocoapods.org/syntax/podspec.html#source)，那么我们需要给当前的仓库打一个版本标签，版本与podspec描述的版本一致
```git
$ git tag '1.2.0'  
$ git push --tags
$ git push origin master
```

## 4. 在cocoapod注册trunk信息
cocoapod使用trunk的方式管理和提交podspec文件，关于trunk的更多说明，参见[官网](http://blog.cocoapods.org/CocoaPods-Trunk)

注册trunk
```
$ pod trunk register zhengbomo@hotmail.com 'bomo' --description='iOS player' --verbose
```
执行完成后，你的邮箱会受到一个验证邮件，打开验证通过，验证通过后，可以通过`pod trunk me`查看自己的信息，下面是我的信息

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-5-4/35982476.jpg)

```
ACA80164:Responsitory zhengxiankai$ pod trunk me
  - Name:     bomo
  - Email:    zhengbomo@hotmail.com
  - Since:    May 3rd, 01:23
  - Pods:     None
  - Sessions:
    - May 3rd, 01:23 - September 8th, 01:25. IP: 106.120.250.167
    Description: iOS player
```
如果能看到自己注册的信息，就可以了，cocoapod不通过密码验证用户，而是通过session token，这个时候就可以提交pod了


## 5. 提交podspec文件
注册完成后，先设置一下需要验证的版本
```
set the new version to 1.2.0
set the new tag to 1.2.0
```
cd到`xxx.podspec`文件所在的目录，验证一下本地的podspec文件是否正确
```
$ pod lib lint

# 验证成功会有提示
xxx passed validation.
```
上面过程都没有问题，上传podspec到pod服务器
```
$ pod trunk push xxx.podspec
```
等待几分钟，如果验证通过，就直接发布了，如果是更新库，则需要改版本号，方法与发布一样


自己发布的库只有自己才能修改/更新，可以添加所有者授权别人修改，xxx为库名称如SDWebImage
```
$ pod trunk add-owner xxx zhengbomo@hotmail.com
```

## 6. 验证
如果提交完成，就可以通过`pod search`搜索出来了
```
$ pod search xxx
```
如果搜索不出来，可以尝试更新一下本地的pod库列表
```
$ pod setup
```

## 7. 参考链接
* [http://blog.csdn.net/wzzvictory/article/details/20067595](http://blog.csdn.net/wzzvictory/article/details/20067595)
* [http://yulingtianxia.com/blog/2014/05/26/publish-your-pods-on-cocoapods-with-trunk/](http://yulingtianxia.com/blog/2014/05/26/publish-your-pods-on-cocoapods-with-trunk/)
