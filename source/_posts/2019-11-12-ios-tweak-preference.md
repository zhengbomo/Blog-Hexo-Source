---
title: 【iOS逆向】为Tweak插件添加设置
tags: [iOS, 逆向]
date: 2019-11-12 11:45:59
updated: 2019-11-12 11:45:59
categories: iOS
---


`THEOS`提供了`iphone/preference_bundle_modern`工具，可以为插件添加设置项，该设置项会被打包成bundle并和tweak插件合并打包到`deb`文件中

{% img /images/post/tweak-preference-entry.png 300 %}

<!-- more -->

## 创建preference bundle

先进入我们tweak插件的目录，然后通过`nic.pl`创建`preference_bundle_modern`

```sh
$ nic.pl
NIC 2.0 - New Instance Creator
------------------------------
  [1.] iphone/activator_event
  [2.] iphone/application_modern
  [3.] iphone/application_swift
  [4.] iphone/flipswitch_switch
  [5.] iphone/framework
  [6.] iphone/library
  [7.] iphone/preference_bundle_modern
  [8.] iphone/tool
  [9.] iphone/tool_swift
  [10.] iphone/tweak
  [11.] iphone/xpc_service
Choose a Template (required): 7
Project Name (required): testprereference
Package Name [com.yourcompany.testprereference]: com.bomo.testprereference
Author/Maintainer Name [bomo]:
[iphone/preference_bundle_modern] Class name prefix (three or more characters unique to this project) [XXX]: BM
Instantiating iphone/preference_bundle_modern in testprereference/...
Adding 'testprereference' as an aggregate subproject in Theos makefile 'Makefile'.
Done.
```

生成下面`testprereference`文件夹

```sh
- control
- Makefile
- Tweak.x
- tweaktest
- testprereference
        | - Resources
                | - Info.plist
                | - Root.plist              配置设置页面Root页面（从entry.plist进入）
        | - entry.plist                     定义在设置页面的入口按钮和文字
        | - Makefile
        | - BMRootListController.h
        | - BMRootListController.m          Root页面绑定的控制器，可以在这里实现action事件
```

### 图标处理

图标放在`Resources`目录下大小为`29*29`，可以使用2/3倍图

```sh
testprereference
        | - Resources
                | - icon.png
                | - icon@2x.png
                | - icon@3x.png
```

### entry.plist

我们看一下设置入口配置的定义`entry.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>entry</key>
    <dict>
        <key>bundle</key>
        <string>testprereference</string>
        <!-- cell类型 -->
        <key>cell</key>
        <string>PSLinkCell</string>
        <!-- 详情页绑定的控制器 -->
        <key>detail</key>
        <string>BMRootListController</string>
        <!-- 图标，放在Resources目录下，可以是icon@3x.png -->
        <key>icon</key>
        <string>icon.png</string>
        <!-- 是否绑定控制器 -->
        <key>isController</key>
        <true/>
        <!-- 显示的文本 -->
        <key>label</key>
        <string>测试设置</string>
    </dict>
</dict>
</plist>
```

编译后可以打开`设置.app`可以看到该项，如上图，接下来我们配置设置主页面`Root.plist`

### Root.plist

定义设置主页面的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>cell</key>
            <string>PSGroupCell</string>
            <key>label</key>
            <string>分组1</string>
        </dict>
        <dict>
            <!-- cell类型：开关类型 -->
            <key>cell</key>
            <string>PSSwitchCell</string>
            <!-- 默认值：true -->
            <key>default</key>
            <true/>
            <!-- userdefault关联的bundleId -->
            <key>defaults</key>
            <string>com.bomo.testprereference</string>
            <!-- 关联的userDefault的key -->
            <key>key</key>
            <string>testpreferenceMainSwitch</string>
            <!-- 显示的文本 -->
            <key>label</key>
            <string>插件开关</string>
            <!-- 当值改变的时候回发出通知 -->
            <key>PostNotification</key>
            <string>com.bomo.testprereference/reloadSettings</string>
        </dict>
        <dict>
            <key>cell</key>
            <string>PSButtonCell</string>
            <!-- 关联的事件，在BMRootListController定义 -->
            <key>action</key>
            <string>openHomePage</string>
            <!-- 按钮颜色 -->
            <key>isDestructive</key>
            <true/>
            <!-- 按钮文本 -->
            <key>label</key>
            <string>打开项目主页</string>
        </dict>
    </array>
    <!-- navigationbar标题 -->
    <key>title</key>
    <string>测试插件</string>
</dict>
</plist>
```

编译后可以查看效果

{% img /images/post/tweak-preference-root.png 300 %}

### RootListController

关联的controller`BMRootListController`实现`PSButtonCell`绑定的action

```objc
@implementation BMRootListController

// ...

- (void)openHomePage {
    NSURL *url = [NSURL URLWithString:@"https://blog.bombox.org"];
    [UIApplication.sharedApplication openURL:url options:@{} completionHandler:nil];
}

@end
```

## 读取配置

在`Root.plist`里面，对于每一个设置项，[`这里`](https://iphonedevwiki.net/index.php/Preferences_specifier_plist)有详细的说明

![_](/images/post/preference_specifier_generalkeys.png)

其中有两个属性

* defaults：设置UserDefault关联的bundleId
* key：设置关联UserDefaults的key，例如PSSwitchCell开关变化的时候，会把值保存到plist文件里面

如上面的设置

```xml
<!-- userdefault关联的bundleId -->
<key>defaults</key>
<string>com.bomo.testprereference</string>
<!-- 关联的userDefault的key -->
<key>key</key>
<string>testpreferenceMainSwitch</string>
```

我们可以`NSUserDefault`在bundle的程序中读取（如：`BMRootListController`）

```objc
NSString *bundleId = @"com.bomo.testprereference";
NSString *switchStatusKey = @"testpreferenceMainSwitch";
NSDictionary *bundleDefaults = [[NSUserDefaults standardUserDefaults] persistentDomainForName:bundleId];
BOOL isSwitchOn = [[bundleDefaults objectForKey:switchStatusKey] boolValue];

NSLog(@"%@", @(isSwitchOn));
```

在Tweak项目中无法使用`NSUserDefault`的方式读取配置，我们的配置会被保存在`/var/mobile/Library/Preferences/{bundle_id}.plist`，我们在tweak插件中，可以通过该文件读取到设置的值，**但不要代码中写入值**

```objc
NSString *bundleId = @"com.bomo.testprereference";
NSString *plistPath = [NSString stringWithFormat:@"/var/mobile/Library/Preferences/%@.plist", bundleId];

NSDictionary *settings = [[NSDictionary alloc] initWithContentsOfFile:plistPath];
BOOL isOn = [settings[@"testpreferenceMainSwitch"] boolValue];
```

## 监听设置变化

通过下面方法监听系统通知

```objc
CFNotificationCenterAddObserver(CFNotificationCenterGetDarwinNotifyCenter(),        // 系统的notifycenter，固定
                    NULL,
                    reloadSettings,                                                 // 回调函数
                    CFSTR("com.bomo.testprereference/reloadSettings"),              // 通知Key
                    NULL,
                    CFNotificationSuspensionBehaviorCoalesce);
```

通知的key在配置cell的时候定义（`Root.plist`）

```xml
<dict>
    <!-- cell类型 -->
    <key>cell</key>
    <string>PSSwitchCell</string>
    ...
    <!-- 当值改变的时候回发出通知 -->
    <key>PostNotification</key>
    <string>com.bomo.testprereference/reloadSettings</string>
</dict>
```

监听函数

```objc
static void reloadSettings(CFNotificationCenterRef center, void *observer, CFStringRef name, const void *object, CFDictionaryRef userInfo) {
    // reload setting，这里需要重新读取配置
}
```

## 安装

插件编译打包后，preferencebundle会被编译成bundle放到deb里面，`PreferenceLoaders`是MobileSubstrate其中的一个工具，可以把tweak扩展PreferenceBundles注入到iOS的设置中

* 入口文件`entry.plist`会被装到`/Library/PreferenceLoader/Preferences`
* 我们编译好的`preference bundle`（可执行文件和资源）会被安装到`/Library/PreferenceBundles`下面
* 而配置数据文件`{bundle_id}.plist`，会被存放在`/var/mobile/Library/Preferences/`目录下

在编译的时候可能会报下面问题

```sh
ld: warning: directory not found for option '-F/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS13.4.sdk/System/Library/PrivateFrameworks'
ld: framework not found Preferences
```

这是由于`Preferences`是私有库，我们可以从[`theos/sdk`](https://github.com/theos/sdks)找到[`Preferences.framework`](https://github.com/theos/sdks/tree/master/iPhoneOS11.2.sdk/System/Library/PrivateFrameworks/Preferences.framework)，下载下来放到`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/System/Library/PrivateFrameworks`，这样就可以正常引用了

## 参考

* [Preferences_specifier_plist](https://iphonedevwiki.net/index.php/Preferences_specifier_plist)
* [PreferenceLoader](https://iphonedevwiki.net/index.php/PreferenceLoader)

