---
title: 使用Cycript调试应用
categories: iOS
tags: [iOS, 逆向]
date: 2019-07-03 20:36:24
updated: 2019-07-03 20:36:24
---

Cycript是由Cydia创始人Saurik推出的一款脚本语言，Cycript混合了OC、JavaScript语法的解释器，这意味着我们能够在一个命令中使用Oc或者JavaScript，甚至两者并用。它能够附加到正在运行的进程，能够查看和修改运行时的数据

<!-- more -->
## 基本使用
1. 通过cydia安装，自带的源`https://apt.bingner.com/`就有cycript

{% img /images/post/cydia-cycript.png 300 cydia-cycript %}


2. 打开 App ，通过 ssh 连接设备，然后进入Cycrypt调试模式
```sh
# 调试进程（PID=323）
cycript -p 323

# 调试进程（进程名）
cycript -p SpringBoard
cycript -p neteasemusic

# `Control+D`退出cycript模式
```

3. 获取进程Id

需要先安装插件：`adv-cmds`（在自带源`https://apt.bingner.com/`可找到）
```sh
# 查看所有进程
ps -A 

# 搜索进程
ps -A | grep neteasemusic
```

4. 常用语法

* UIApp：`[UIApplication sharedApplication]`
* 定义变量：`var 变量名 = 变量值`
* 通过内存获得对象：`#内存地址`
* 查看对象的所有成员：`*对象`
* 获取所有已加载的OC类：`ObjectiveC.classes`
* 获取当前内存中所有UITableViewCell（包含子类）的实例：`choose(UITableViewCell)`
* 递归打印所有的子控件：`[view recursiveDescription].toString()`
* 查看 bundleId: `[[NSBundle mainBundle] bundleIdentifier]`

5. 函数
```js
function KenPrintIvars(objc){
    var x = {};
    for(i in *objc){
        try {
            x[i] = (*objc)[i];
        } catch(e) {
        }
    }
    return x;
}
```


## 引用外部脚本
这里使用[`mjcript`](https://github.com/CoderMJLee/mjcript)作为外部脚本引入，下载得到`mjcript.cy`

把文件拷贝到手机上
```sh
scp mjcript.cy root@xx.xx.xx.xx:/usr/lib/cycript0.9/mjcript.cy
```
把手机的文件拷贝到本地
```sh
scp root@xx.xx.xx.xx:/usr/lib/cycript0.9/mjcript.cy ~/Desktop/mjcript.cy
```
加载脚本
```sh
# 附加到进程
cycript -p XXX

# 加载
cy# @import mjcript

# 使用脚本
cy# MJFrontVc()
#"<ZTPersonCenterViewController: 0x10520ba00>"
```

`mjcript`功能列表

```objc
// 包名
MJAppId;

// bundle path
MJAppPath;

// document path
MJDocPath;

// caches path
MJCachesPath;

// 加载系统动态库（/System/Library/Frameworks/xxx.framework，/System/Library/Private/Frameworks/xxx.framework）
MJLoadFramework("BluetoothManager");

// keyWindow
MJKeyWin();

// 根控制器
MJRootVc();

// 找到显示在最前面的控制器
MJFrontVc();

// 递归打印UIViewController view的层级结构
MJVcSubviews(vc);

// 递归打印最上层UIViewController view的层级结构
MJFrontVcSubViews();

// 获取按钮绑定的所有TouchUpInside事件的方法名
MJBtnTouchUpEvent(btn);

// CG函数
MJPointMake(x, y);

MJSizeMake(w, h);

MJRectMake(x, y, w, h);

// 递归打印controller的层级结构
MJChildVcs(vc);

// 递归打印view的层级结构
MJSubviews(view);

// 判断是否为字符串 "str" @"str"
MJIsString(value);
		
// 判断是否为数组 []、@[]
MJIsArray(value);

// 判断是否为数字 666 @666
MJIsNumber(value);

// 打印所有的子类
MJSubclasses(className, reg);

// 打印所有的对象方法
MJInstanceMethods(className, reg);

// 打印所有的对象方法名字
MJInstanceMethodNames(className, reg);

// 打印所有的类方法
MJClassMethods(className, reg);

// 打印所有的类方法名字
MJClassMethodNames(className, reg);
		
// 打印所有的成员变量
MJIvars(obj, reg);

// 打印所有的成员变量名字
MJIvarNames(obj, reg);
```

> Cycript默认不支持中文，可以使用 unicode 字符表示中文`\**\**\**\**`
> 技巧：使用python把中文转成unicode字符
```py
unicode("登录", "UTF-8")
```


### 封装cycript脚本
```js
(function(exports) {
	exports.sum = function(a, b) {
		return a + b;
	};
	exports.minus = function(a, b) {
		return a - b;
	};
	exports.age = 10;
})(exports);
```
导入(`/usr/lib/cycript0.9/test.cy`)，并引用
```js
@import test

test.sum(1, 30)     // 31
test.minus(6, 2)    // 4
```

通过目录引用`/usr/lib/cycript0.9/com/mj/cycript.cy`
```js
@import com.mj.mjcript
```

## 小结
Cycript 可以直接附加到App 进行内存调试，可以查看和修改 UIViewController，UIView，可以动态修改和分析应用的业务逻辑，用起来非常方便
