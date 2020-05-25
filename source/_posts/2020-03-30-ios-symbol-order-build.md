---
title: iOS二进制重排对缺页和启动时间的优化实践
tags: [iOS]
date: 2020-03-30 15:45:56
updated: 2020-03-30 15:45:56
categories: iOS
---


抖音团队去年针对系统虚拟内存缺页的情况，基于二进制重排的方案，给App启动速度提升了15%，各路大神也随后分享了几篇优质的二进制重排的文章，这里基于自己的项目做一下实践

<!-- more -->

## 基本原理

1. 进程运行时使用的内存是操作系统提供的`虚拟内存`，而不是直接操作物理内存
2. 从`虚拟内存`到`物理内存`有一个映射表(页表)
3. 进程的内存会进行`分页`管理，以页为单位
4. 程序启动的时候，**并不会把所有内存都加载到物理内存中**，而是用到的时候才加载，没有用到的内存，可能并没有加载到物理内存中
5. 当程序访问到的内存地址（虚拟内存），如果还没有加载到物理内存时，就会触发`Page Fault`，（对应`System Trace`的`File Backed Page In`），然后操作系统把数据加载到物理内存中，如果已经已经加载到物理内存了，则会触发`Page Cache Hit`，后者是比较快的，这也是热启动比冷启动快的原因之一

> 1. 基于上面原理. 我们的目标就是在启动的时候增加`Page Cache Hit`，减少`Page Fault`，从而达到优化启动时间的目的
> 2. 我们需要确定，在启动的时候，执行了哪些符号，尽可能让这些符号的内存集中在一起，减少占用的页数，就能减少`Page Fault`的命中次数

## 测试Page Fault

通过`Instrument / System Trace`工具，可以查看我们的App，在启动过程中的`Page Fault`数量(File Breaked Page In)

![system trace page fault](/images/post/systemtrace-app-page-fault1.png)

> 如果App比较大，`Analizing`的过程会比较久，需要耐心等待

这里有个注意点，为了**确保App是真正的冷启动**，需要把内存清干净，不然结果会不太准，下图是我直接杀掉App，重新打开得到的结果

![_](/images/post/systemtrace-app-page-fault3.png)

可以看到，和第一次测试差的有点多，**我们可以在杀掉App后，重新打开多个其他的App（尽可能多），把原来的内存都覆盖掉，这样在重新打开App的时候，就会重新加载物理内存**

## 确定代码执行顺序

接下来需要确定App在启动的时候，调用了哪些函数（使用了哪些符号），这里我们使用[杨萧玉](http://yulingtianxia.com/)写的一个工具[AppOrderFiles](https://github.com/yulingtianxia/AppOrderFiles)，使用`Clang SanitizerCoverage`，通过编译器插装的方式，获取到调用函数的符号顺序

通过pod引入

```ruby
pod 'AppOrderFiles'
```

并且添加编译宏`OTHER_CFLAGS`和`OTHER_SWIFT_FLAGS`（只在Debug生效即可）

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      case config.name
        when "Debug"
        config.build_settings['OTHER_CFLAGS'] = '-fsanitize-coverage=func,trace-pc-guard'
        config.build_settings['OTHER_SWIFT_FLAGS'] = '-sanitize-coverage=func -sanitize=undefined'
        end
    end
  end
end
```

在App启动后，到第一个页面（HomePage）的viewDidLoad方法

```swift
import AppOrderFiles

override func viewDidLoad() {
    super.viewDidLoad()

    ...

    #if DEBUG
    // 延迟一下，让运行实践长一点，避免进入后因为PageFault造成卡顿
    DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.5, execute: {
        AppOrderFiles { (filePath) in
            if let p = filePath {
                print("output order file \(p)")
            }
        }
    })
    #endif
}
```

输出的文件在App沙盒，用模拟器运行更方便，得到文件`app.order`，这里面就是排好序的符号列表，根据App的执行顺序，如果项目比较大的话，会比较久

```txt
___swift_instantiateConcreteTypeFromMangledName
_main
_$s3jcm11AppDelegateCMa
_$s3jcm11AppDelegateCACycfcTo
_$s3jcm11AppDelegateCACycfc
_$s3jcm11AppDelegateC11application_29didFinishLaunchingWithOptionsSbSo13UIApplicationC_SDySo0j6LaunchI3KeyaypGSgtFTo
_$s3jcm11AppDelegateC11application_29didFinishLaunchingWithOptionsSbSo13UIApplicationC_SDySo0j6LaunchI3KeyaypGSgtF
_$s3jcm11AppDelegateC5setup13launchOptionsySDySo019UIApplicationLaunchF3KeyaypGSg_tF
_$s3jcm5ConstV11wechatAppIdSSvau
_globalinit_33_27D199AC10BAAE2783814C508183B809_func13
_$s3jcm5ConstV19wechatUniversalLinkSSvau
_globalinit_33_27D199AC10BAAE2783814C508183B809_func15
_$sSo12BaiduMobStatCMa
_$sSo12BaiduMobStatCs5Error_pIggzo_ABsAC_pIegnzo_TRTA
_$sSo12BaiduMobStatCs5Error_pIggzo_ABsAC_pIegnzo_TR

...
```

把`app.order`放到工程目录，配置到Xcode里面`Build Setting` -> `Order File` -> `$(PROJECT_DIR)/app.order`

![order file setting](/images/post/order-file-setting.png)

## 验证是否生效

Xcode里面`Build Setting`有个`Write Link Map File`，可以生成Link Map文件的选项，路径如下

```sh
# Link Map文件
Intermediates.noindex/xxxx.build/Debug-iphoneos/xxx.build/xxx-LinkMap-normal-arm64.txt

# 生成app文件路径
Products/Debug-iphoneos/xxx.app
```

文件内容其实是描述链接器连接的详情，对应的是MachO文件的内存分布，文件如下

```txt
# Path: /Users/bomo/Library/Developer/Xcode/DerivedData/SwiftScaffold-fdswirgebkkdidcxcpxdffxxvxye/Build/Products/Debug-iphoneos/jcm.app/jcm
# Arch: arm64
# Object files:
[  0] linker synthesized
[  1] /Users/bomo/Library/Developer/Xcode/DerivedData/SwiftScaffold-fdswirgebkkdidcxcpxdffxxvxye/Build/Intermediates.noindex/SwiftScaffold.build/Debug-iphoneos/jcm.build/Objects-normal/arm64/JHCollectionViewFlowLayout.o
[  2] /Users/bomo/Library/Developer/Xcode/DerivedData/SwiftScaffold-fdswirgebkkdidcxcpxdffxxvxye/Build/Intermediates.noindex/SwiftScaffold.build/Debug-iphoneos/jcm.build/Objects-normal/arm64/JHCollectionReusableView.o
...

# Sections:
# Address   Size        Segment Section
0x100004928 0x00ED5B08  __TEXT  __text
0x100EDA430 0x00005550  __TEXT  __stubs
0x100EDF980 0x00005190  __TEXT  __stub_helper
0x100EE4B10 0x000684D9  __TEXT  __cstring
...

# Symbols:
# Address   Size        File  Name
0x100004928 0x00000094  [  6] ___swift_instantiateConcreteTypeFromMangledName
0x1000049BC 0x00000088  [ 78] _main
0x100004A44 0x00000070  [ 78] _$s3jcm11AppDelegateCMa
0x100004AB4 0x00000044  [ 78] _$s3jcm11AppDelegateCACycfcTo
0x100004AF8 0x00000108  [ 78] _$s3jcm11AppDelegateCACycfc
0x100004C00 0x00000144  [ 78] _$s3jcm11AppDelegateC11application_29didFinishLaunchingWithOptionsSbSo13UIApplicationC_SDySo0j6LaunchI3KeyaypGSgtFTo
0x100004D44 0x00000430  [ 78] _$s3jcm11AppDelegateC11application_29didFinishLaunchingWithOptionsSbSo13UIApplicationC_SDySo0j6LaunchI3KeyaypGSgtF
...

# Dead Stripped Symbols:
#           Size        File  Name
<<dead>>    0x00000006  [  2] literal string: class
<<dead>>    0x00000014  [  2] literal string: setBackgroundColor:
<<dead>>    0x0000000B  [  2] literal string: v24@0:8@16
<<dead>>    0x00000010  [  3] literal string: backgroundColor
<<dead>>    0x00000014  [  3] literal string: setBackgroundColor:
<<dead>>    0x0000000E  [  3] literal string: .cxx_destruct
<<dead>>    0x00000008  [  3] literal string: @16@0:8
...
```

这里我们只关注符号表`Symbols`，这里的顺序就是MachO文件对应的顺序，如果与`app.order`的顺序一致，就表明改成功了

## 对比

通过`System Trace`工具测试修改前后对比

![system trace page fault diff](/images/post/trace-page-fault-diff.png)

`page fault`减少了900，速度提升`225ms`，这里的时间与具体的运行环境有关系，建议多次测试

## 引用

* [抖音研发实践：基于二进制文件重排的解决方案 APP启动速度提升超15%](https://mp.weixin.qq.com/s?__biz=MzI1MzYzMjE0MQ==&mid=2247485101&idx=1&sn=abbbb6da1aba37a04047fc210363bcc9&scene=21&token=2051547505&lang=zh_CN#wechat_redirect)
* [App 二进制文件重排已经被玩坏了](http://yulingtianxia.com/blog/2019/09/01/App-Order-Files/)
