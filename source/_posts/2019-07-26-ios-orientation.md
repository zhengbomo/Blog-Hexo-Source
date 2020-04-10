---
title: iOS旋转屏幕
tags: [iOS]
date: 2019-07-26 12:55:16
updated: 2019-07-26 12:55:16
categories: [iOS]
---

iOS屏幕旋转控制，自动旋转，手动旋转，锁定屏幕

<!-- more -->

## 1. 设置App支持的旋转方向（2种方式）

### 1. 通过工程设置

`General`->`Deployment Info`->`Device Orientation`，勾选支持的方向

### 2. 通过代码设置（AppDelegate）

```swift
func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
    return .allButUpsideDown
}
```

> 方式1存在一个问题：如果勾选了多个方向，如果横屏进入App，会出现首页横屏的情况，即使设置了`VC`只支持竖屏，推荐使用方式2

## 2. ViewController旋转控制

通常在需要旋转的ViewController，重写下面三个方法即可

```swift
/// 控制是否支持自动旋转，回根据设备方向自动调整布局，例如视频横屏播放，微信公众号文章横屏阅读等
override var shouldAutorotate: Bool {
    return true
}

/// 设置首次进入ViewController时的方向，之后再根据设备方向变动调整，例如可以保证无论设备是否横屏，首次进入一个ViewController的时候为竖屏
/// 注意：这里的设置仅对第二个页面有效，第一个页面无效
override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {
    return .portrait
}

/// 设置支持旋转的方向
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
    return .landscape
}
```

## 3. 容器

如果ViewController在容器控制器里面的话（`UINavigationController`和`UITabBarController`）需要重写容器，让其指向子控制器

`TabBarController`

```swift
import UIKit

class TabBarController: UITabBarController {
    override var shouldAutorotate: Bool {
        return self.selectedViewController?.shouldAutorotate ?? false
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return self.selectedViewController?.supportedInterfaceOrientations ?? UIInterfaceOrientationMask.portrait
    }

    override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {
        return self.selectedViewController?.preferredInterfaceOrientationForPresentation ?? .portrait
    }
}
```

`NavigationController`

```swift
import UIKit

class NavigationController: UINavigationController {
    override var shouldAutorotate: Bool {
        return self.topViewController?.shouldAutorotate ?? false
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return self.topViewController?.supportedInterfaceOrientations ?? UIInterfaceOrientationMask.portrait
    }

    override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {
        return self.topViewController?.preferredInterfaceOrientationForPresentation ?? .portrait
    }
}
```

## 4. 强制旋转

iOS没有提供公开的API直接修改屏幕方向，通常我们用`kvc`的方式实现

```swift
// 强制竖屏
UIDevice.current.setValue(UIInterfaceOrientation.portrait.rawValue, forKey: "orientation")

// 强制左横屏
UIDevice.current.setValue(UIInterfaceOrientation.landscapeLeft.rawValue, forKey: "orientation")
```

## 5. 方向锁定

通过控制VC支持`supportedInterfaceOrientations`的方向，就可以控制锁定了，只返回一种方向，就能实现锁定的功能

```swift
/// 定义锁定的方向
private var lookOrientation: UIInterfaceOrientation?

// 设置当前的方向（锁定屏幕方向）
self.lookOrientation = UIApplication.shared.statusBarOrientation
// 取消当前的方向（解锁）
self.lookOrientation = nil

/// 重载用于控制锁定
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
    if let orientation = self.lookOrientation {
        // 锁定方向
        return orientation.orientationMask
    } else {
        // 默认可选旋转
        return .allButUpsideDown
    }
}

extension UIInterfaceOrientation {
    // UIInterfaceOrientation转换为UIInterfaceOrientationMask
    var orientationMask: UIInterfaceOrientationMask {
        switch self {
        case .unknown:
            return .allButUpsideDown
        case .portrait:
            return .portrait
        case .portraitUpsideDown:
            return .portraitUpsideDown
        case .landscapeLeft:
            return .landscapeLeft
        case .landscapeRight:
            return .landscapeRight
        @unknown default:
            return .allButUpsideDown
        }
    }
}
```
