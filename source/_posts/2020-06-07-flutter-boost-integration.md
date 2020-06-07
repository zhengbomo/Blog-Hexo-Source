---
title: flutter_boost混编实践
tags: [flutter]
date: 2020-06-07 10:15:43
updated: 2020-06-07 10:15:43
categories: flutter
---


基于现有的项目接入flutter，官方提供的一个比较完整的接入方案，但是存在一个问题，由于`FlutterEngine`很重，而多个`FlutterViewController`共享一个Engine，并且同一时间下，一个engine只能与一个viewcontroller绑定，在管理起来，特别是对于多层跳转（native->flutter->native->flutter）非常麻烦，而如果创建多个`FlutterEngine`，就会带来很大的开销，导致内存暴涨，而`flutter_boost`提供了像WebView的方式操作FlutterViewController，可以存在多个FlutterViewController，这里基于一个现有的demo继承flutter_boost并解决侧滑手势冲突的问题

<!-- more -->

## 环境配置

[官方](https://flutter.dev/docs/development/add-to-app/ios/project-setup)的流程的教程很详细，这里简单说明

1. 创建flutter模块（这里取为`my_flutter`）

    ```sh
    cd some/path/
    flutter create --template module my_flutter
    ```

2. 从原来的项目的`Podfile`添加引用

    ```rb
    source 'https://github.com/CocoaPods/Specs.git'

    platform:ios,'10.0'
    # 不用动态库，动态库多了影响启动速度
    # use_frameworks!
    use_modular_headers!
    inhibit_all_warnings!

    # 加上flutter脚本
    flutter_application_path = '../my_flutter'
    load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')


    target 'Demo' do
        # 其他native用到的第三方库
        pod 'SnapKit'

        # flutter pod
        install_all_flutter_pods(flutter_application_path)
    end
    ```

3. 先运行`flutter pub get`，再运行pod（最好先运行一下`my_flutter`）

    ```sh
    cd /path/to/flutter_module
    flutter pub get

    cd /path/to/iosproject
    pod install
    ```

4. 接下来就可以直接在Xcode运行了

> 当然还有其他的接入方式，笔者认为这个方式比较简单，具体可以参考[官方教程](https://flutter.dev/docs/development/add-to-app/ios/project-setup)

## 开始接入

为了更方便的管理`FlutterEngine`和`FlutterViewController`，这里使用`flutter_boost`来管理

1. 为了方便管理，我们对`FLBFlutterViewContainer`进行封装

    ```swift
    class FlutterVC: FLBFlutterViewContainer {
        override func loadView() {
            super.loadView()
            // 隐藏navigationbar
            self.navigationController?.navigationBar.isHidden = true
        }
    }
    ```

2. 实现flutter到本地的路由（`NavigationHelper`为帮助类，具体见后面源码链接）

    ```swift
    class PlatformRouterImp: NSObject, FLBPlatform {
        // 从flutter传过来的push页面需求
        // await FlutterBoost.singleton.open("nativePage", urlParams: {"a": 1}, exts: {"b": 2});
        func open(_ url: String, urlParams: [AnyHashable : Any], exts: [AnyHashable : Any], completion: @escaping (Bool) -> Void) {
            switch url {
            case "nativePage":
                // 跳转到本地页面
                let vc = HomeVC()
                NavigationHelper.nav2VC(vc)
                completion(true);
            default:
                completion(false);
            }
        }

        func present(_ url: String, urlParams: [AnyHashable : Any], exts: [AnyHashable : Any], completion: @escaping (Bool) -> Void) {
            switch url {
            case "nativeFlutterPage":
                // 创建新的FlutterVC
                let vc = FlutterVC();
                vc.setName("home", params: urlParams);
                let navVC = UINavigationController(rootViewController: vc)
                NavigationHelper.presentVC(navVC)
                completion(true);
            default:
                completion(false);
            }
        }

        func close(_ uid: String, result: [AnyHashable : Any], exts: [AnyHashable : Any], completion: @escaping (Bool) -> Void) {
            let topVC = NavigationHelper.getTopVC()
            let presentVC = topVC.presentingViewController
            // 判断是否时present
            if let _ = presentVC {
                topVC.dismiss(animated: true, completion: { [unowned topVC] in
                    topVC.removeFromParent()
                })
            } else {
                topVC.navigationController?.popViewController(animated: true)
            }
        }
    }
    ```

3. 初始化，可以放在`application:didFinishLaunchingWithOptions:`，定义一个名为`myflutter_method_channel`的MethodChannel，用于通信

    ```swift
    // 用于处理flutter发送的页面跳转消息
    let router = PlatformRouterImp.init();

    FlutterBoostPlugin.sharedInstance().startFlutter(with: router, onStart: { (engine) in
        // 配置channel，用于通信
        let batteryChannel = FlutterMethodChannel(name: "myflutter_method_channel",
                                                    binaryMessenger: engine.binaryMessenger)
        batteryChannel.setMethodCallHandler({ (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            switch call.method {
            case "alert":
                // native弹窗
                let vc = UIAlertController(title: "测试弹窗", message: "弹窗内容", preferredStyle: .alert)
                vc.addAction(UIAlertAction(title: "取消", style: .cancel, handler: nil))
                NavigationHelper.getTopVC().present(vc, animated: true, completion: nil)
                result(true)
            default:
                result(false)
            }
        })
    })
    ```

4. 在Dart初始化`flutter_boost`

    ```swift
    void main() {
        runApp(MyApp());
    }

    class MyApp extends StatefulWidget {
        @override
        _MyAppState createState() => _MyAppState();
    }

    class _MyAppState extends State<MyApp> {
        @override
        void initState() {
            // 注册Page，用于native跳转
            FlutterBoost.singleton.registerPageBuilders(<String, PageBuilder>{
                'home': (String pageName, Map params, String uniqueId) {
                    return HomePage();
                },
                "me": (String pageName, Map params, String uniqueId) {
                    return MePage();
                }
            });
            super.initState();
        }

        @override
        Widget build(BuildContext context) {
            return MaterialApp(
            title: 'Demo',
            theme: ThemeData(
                primarySwatch: Colors.blue,
            ),
            // 使用FlutterBoost作为builder
            builder: FlutterBoost.init(),
            home: HomePage(),
            );
        }
    }
    ```

5. flutter给native发消息

    ```dart
    try {
        // 给native发消息
        final bool result = await method_channel.invokeMethod("alert", <String, dynamic>{ "a": 1 });
        print(result);
    } on Exception catch (e) {
        print(e);
    }
    ```

6. TODO: native给flutter发消息


### 兼容侧滑返回

iOS的`UINavigationController`默认支持侧滑返回，flutter的`MaterialApp`在iOS上也支持侧滑返回，由于flutter运行在FlutterViewController上，所以默认情况下侧滑走的是UINavigationController，为了让侧滑可以衔接起来，我们需要

* 当`MaterialApp`的只有一个页面的时候，使用native侧滑
* 当`MaterialApp`有不止一个页面的时候，禁用native的侧滑操作

我们在`MaterialApp`添加一个observer

```dart
import 'package:flutter/material.dart';

// native通信channel
const method_channel = const MethodChannel('myflutter_method_channel');

class MyNavigatorObserver extends NavigatorObserver {
    // 在push和pop的时候，更新native的侧滑操作

    @override
    void didPush(Route<dynamic> route, Route<dynamic> previousRoute) {
        method_channel.invokeMethod("flutter_page_changed", {
            'type': 'push',
            'canPop': route.navigator.canPop()
        });
    }

    @override
    void didPop(Route<dynamic> route, Route<dynamic> previousRoute) {
        method_channel.invokeMethod("flutter_page_changed", {
            'type': 'pop',
            'canPop': route.navigator.canPop()
        });
    }
}
```

由于`flutter_boost`会接管`MeterialApp`自带的observer，并且提供了对应的方法（设置在MaterialApp会无效）

```dart
// 添加navigationObserver，用于控制返回操作
FlutterBoost.singleton.addBoostNavigatorObserver(myObserver);
```

在`FlutterVC`添加`popGestureRecognizerEnabled`用于设置native侧滑功能

```swift
class FlutterVC: FLBFlutterViewContainer {
    var popGestureRecognizerEnabled = true {
        didSet {
            self.navigationController?.interactivePopGestureRecognizer?.isEnabled = popGestureRecognizerEnabled
        }
    }
}
```

由于`UINavigationController`是多个`viewController`共用的，所以在本地页面跳转的时候，也需要更新`popGestureRecognizerEnabled`

```swift
protocol PopGestureEnable {
    var popGestureRecognizerEnabled: Bool { get set }
}

class FlutterVC: FLBFlutterViewContainer, PopGestureEnable {
    var popGestureRecognizerEnabled = true {
        didSet {
            self.navigationController?.interactivePopGestureRecognizer?.isEnabled = popGestureRecognizerEnabled
        }
    }

    // 由于navigationController是多个viewController共享的，所以在页面跳转的时候，也需要进行更新
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
         self.navigationController?.interactivePopGestureRecognizer?.isEnabled = self.popGestureRecognizerEnabled
    }
}

class ViewController: UIViewController, PopGestureEnable{
    var popGestureRecognizerEnabled = true {
        didSet {
            self.navigationController?.interactivePopGestureRecognizer?.isEnabled = popGestureRecognizerEnabled
        }
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // 页面显示的时候更新一下
        self.navigationController?.interactivePopGestureRecognizer?.isEnabled = self.popGestureRecognizerEnabled
    }
}
```

添加消息handle

```swift
// 配置channel
let batteryChannel = FlutterMethodChannel(name: "myflutter_method_channel",
                                            binaryMessenger: engine.binaryMessenger)
batteryChannel.setMethodCallHandler({ (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
    switch call.method {
    case "flutter_page_changed":
        // 修改native页面的侧滑
        if let argument = call.arguments as? [String: Any],
            let canPop = argument["canPop"] as? Bool {
            // 取出最上面的Controller
            if let vc = NavigationHelper.getTopVC() as? FlutterVC {
                // 修改侧滑enabled
                vc.popGestureRecognizerEnabled = !canPop
            }
        }
        result(nil)
    default:
        break;
    }
})
```

{% img /images/post/flutter_boost_demo.gif 300 %}

代码在这里: [https://github.com/zhengbomo/flutter_boost_demo](https://github.com/zhengbomo/flutter_boost_demo)
