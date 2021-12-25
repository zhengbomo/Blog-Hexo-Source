---
title: Xcode调试链优化
tags: [iOS]
date: 2021-11-15 14:48:37
updated: 2021-11-15 14:48:37
categories: iOS
---

Xcode是增量编译的，所以日常开发很多时候，我们都是改少量的代码或不改代码而重复调试，实际使用发现，从工程要跑到手机上调试仍然非常耗时，由于缓存的存在，编译可能不是最耗时的环节了，这里探究和优化影响`编译完成`到`App启动调试`速度的因素。

<!-- more -->

## 分析

这里只考虑DEBUG包，我们看一张Xcode打包日志细节图


{% img /images/post/xcodebuildopt/xcode_build_review.png 600 Xcode编译时间 %}

上图选中的为主要耗时的步骤，可以看出，编译成功后，还有下面步骤较为耗时

* Linking
* Embed Pod Frameworks
* Copy Pods Resources
* Custom Script（这里为自定义脚本，主要用于资源处理）
* Sign

> 还有一个较为耗时的时间是`Deploy`，但Xcode并不没有输出Deploy的时间，后面使用`ideviceinstaller`工具单独测试，时间总体上与Xcode基本一致

分析

1. Linking: 合并静态资源与地址修正，考虑使用第三方更快的link工具
2. Embed Pod Frameworks: 主要是处理动态库，拷贝到目标路径，主要影响是动态库数量
3. Copy Pods Resources: 主要对文件资源拷贝，主要影响是文件数量和大小，数量多则io耗时长
4. Custom Script：这里是自定义脚本，主要是的图片文件的处理，主要是对图片文件进行展开，文件多则io耗时长
5. Sign：对整个包签名，主要影响是文件数量和大小，文件多则ios耗时长
6. Deploy：主要是拷贝文件和签名校验，主要影响是文件数量，文件多则ios耗时长，文件大小影响首次安装，而二次安装会做差异拷贝，影响会减小

有上面可以看出，减少IO操作是一个可行的优化方向，项目中的图片资源是放到根目录的，由于项目中用到的图片非常多，直接使用目录管理，并且使用`Custom Script`脚本单独处理图片，把图片拷贝到包的根目录上，查看`xx直播`在编译完成后的包，总文件数为`9297`个, 其中`8821`个为图片，占95%，文件数量占比大，这意味着每次都要进行大量的IO操作，如果能减少文件的数量，就能减少整个流程的时间

## 优化

### IO问题

对于图片，我们知道除了可以放到根目录，还有可以放到`Images.xcassets`，放到`Images.xcassets`的图片最终会被编译成`Assets.car`，这里考虑把png和jpg图片提前制作成`Assets.car`，在日常开发迭代过程中，项目中用到的图片不会频繁的变动，这里我们考虑提前把图片做成`Assets.car`，如果有变动，再重新编译一次，操作步骤如下

1. 如果项目已经存在`Images.xcassets`，则拷贝一份出来，在它的基础上添加图片
2. 通过工具把所有图片构造成`xxx.imageset`，放到`Images.xcassets`里面，如

把`bottom_logo@2x.png`和`bottom_logo@2x.png`构造成下面目录结构

    ```sh
    ├── bottom_logo.imageset
    │   ├── Contents.json
    │   ├── bottom_logo@2x.png
    │   └── bottom_logo@3x.png
    ```

其中`Content.json`为

    ```json
    {
    "images" : [
        {
        "idiom" : "universal",
        "filename" : "bottom_logo@2x.png",
        "scale" : "2x"
        },
        {
        "idiom" : "universal",
        "filename" : "bottom_logo@3x.png",
        "scale" : "3x"
        }
    ],
    "info" : {
        "version" : 1,
        "author" : "xcode"
    }
    }
    ```

2. 使用`actool`编译`Images.xcassets`

```sh
outputPath="path/to/outputpath"
outputPath="path/to/outputpath/dependenciesPath"
outputPath="path/to/outputpath/generatedInfoPath"
xcassetPath="path/to/Images.xcassets"

/usr/bin/actool                                             \
      --output-format "human-readable-text"                 \
      --notices                                             \
      --export-dependency-info "$dependenciesPath"          \
      --output-partial-info-plist "$generatedInfoPath"      \
      --app-icon "AppIcon"                                  \
      --compress-pngs                                       \
      --enable-on-demand-resources "YES"                    \
      --development-region "English"                        \
      --target-device "iphone"                              \
      --minimum-deployment-target "9.0"                     \
      --platform "iphoneos"                                 \
      --compile "$outputPath" "$xcassetPath"
```

编译后得到`Assets.car`

3. 由于`Images.xcassets`已经编译好了，去掉工程对`Images.xcassets`的引用，并且把编译好的`Assets.car`引入工程，上面我们只把`jpg/png`图片编译进了`Assets.car`，对于其他文件（不多），这里我通过工具单独引入工程

{% img /images/post/xcodebuildopt/xcode_xcassets.png 800 %}

4. 把`Run Script`中的图片拷贝去掉，我这里的是去掉`Pods-xxx-resources.sh`文件中对图片资源的引用

经过上面一番处理后，重新测试，数据对比如下

|  | 优化前 | 优化后 |
| ---- | ---- | ---- | 
| Copy Pod Resources | 11s | 0.1s（减少97%） |    
| Run Custom Script | 32.5 | 0s(减少100%) |
| Sign | 2.2 | 1.5s（减少45%） |

从上面的测试结果来看，速度提升效果显著

> 因为优化后的方案有前置处理时间，需要把编译图片为`Assets.car`，这个时间没算上，上面的对比仅供参考，可以与其他操作并行

上面的操作过于繁琐，可以写了一个工具【一键操作】

{% img /images/post/xcodebuildopt/xcode_build_opt_tool.png 800 工具图片 %}

### 关于Linking

有一个第三方的linking工具[zld](https://github.com/michaeleisel/zld)，优化的缓存策略，在`Other Linker Flag`添加`-fuse-ld='/path/to/zld' -Wl,-zld_original_ld_path,$(DT_TOOLCHAIN_DIR)/usr/bin/ld`即可

经测试，Linking时间从`15s`减少到`10.1s`，减少了`32%`

### 关于CodeSign

1. Xcode默认使用`SHA-256`签名，我们可以改成更快的`SHA-1`，由于只是本地开发调试使用，不会有安全性问题，在`Other Code Signing Flags`添加`--digest-algorithm=sha1`
2. iOS校验签名只会二进制的签名，其他文件不签名也不会报错，可以通过过滤文件减少签名的文件，在`Other Code Signing Flags`添加`--resource-rules=/path/to/rule.plist`，`rule.plist`文件如下
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>rules</key>
        <dict>
                <key>.*</key>
                <false/>
        </dict>
    </dict>
    </plist>
    ```

上面处理并不会影响正常开发，通过上面处理编译后的包`_CodeSignature/CodeResources`文件从`2.88 MB`减少到`589 Byte`，减少了`99.98%`
Sign时间从`2.2s`减少到`0.5s`，减少了`77%`

### 关于Deploy

由于`APFS`的特性，相同的文件不会深拷贝，省去拷贝的时间，由于文件数量大大减少，IO次数也大幅提高，如果不改动文件的话，基本可以达到秒启动，由于Xcode没有输出安装所有的时间，这里使用`ideviceinstaller`进行测试，这里把优化前后打出来的`xxx.app`安装到手机上（卸载），测试时间

```sh
# 安装ideviceinstaller
# brew install -HEAD libimobiledevice

ideviceinstaller -i xxx.ipa
```

优化前：`40s`，优化后：`18s`，时间减少了`55%`

## 综合统计

### 微观统计

|      | 优化前 | 优化后 |
| ---- | ---- | ---- | 
| Linking | 15s | 10.1s（下降32%） |    
| Copy Pod Resources | 11s | 0.1s（减少97%） |    
| Run Custom Script | 32.5 | 0s（下降100%)）|
| Sign | 2.2 | 0.5s（下降77%） |
| Deploy |40s | 18s（下降55%） |

{% img /images/post/xcodebuildopt/xcode_io_time.png %}


### 宏观统计

为了减少编译时间的影响，工程中大多数组件都使用二进制库，数据来源为Xcode编译打包后显示的时间（不包含Deploy和Run），如下

{% img /images/post/xcodebuildopt/xcode_build_time.png 400 %}

|     | 首次 | 二次（修改代码）| 二次（不改代码） |
| --- | --- | --- | --- |
| 优化前 | 88.75s | 27.4s | 9.2s | 
| 优化后 | 37.35s | 14.2s | 2.2s |
| 对比 | 减少58% | 减少48% | 减少76% |

 {% img /images/post/xcodebuildopt/xcode_macro_time.png %}


> 测试设备：Mac Mini M1 16G

## 总结

经过一段时间的使用，上面所有操作都是基于对开发效率的提升（for DEBUG），通过一些细节优化，可以把项目入侵降到最小，可以做到提高效率的同时，基本不影响日常开发

市面上大多数App包里面都会带非常多图片资源，图片太多确实会影响调试性能，减少图片数量从而减少IO次数，提升开发体验明显
