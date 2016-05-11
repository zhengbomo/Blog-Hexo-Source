---
title: iOS获取当前ViewController
categories: iOS
tags:
  - iOS
date: 2016-05-11 14:36:16
updated: 2016-05-11 14:36:16
---


做iOS开发的时，很多时候我们需要对获取当前所在的ViewController，搜了一下，大多数答案都不靠谱，都不能正确获取到当前的控制器，不一定能获取到当前的ViewController，于是自己写了一个方法

<!-- more -->

iOS自带的ViewController有两种
  * 一种是普通控制器：UIViewController, UITableViewController, UISearchBarController等
  * 一种是容器控制器：
    * UINavigationController：
      通过childViewController.lastObject获取当前控制器
    * UITabBarController
      通过selectedViewController 获取当前控制器

页面跳转有push和present，如果有present控制器，则直接取弹出的控制器，如果是导航控制器，则取最后一个，然后一层一层往下取，代码如下

```objc
/** 获取当前控制器 */
+ (UIViewController *)currentVC
{
    UIWindow *window = [[UIApplication sharedApplication] keyWindow];

    //当前windows的根控制器
    UIViewController *controller = window.rootViewController;

    //通过循环一层一层往下查找
    while (YES) {
        //先判断是否有present的控制器
        if (controller.presentedViewController) {
            //有的话直接拿到弹出控制器，省去多余的判断
            controller = controller.presentedViewController;
        } else {
            if ([controller isKindOfClass:[UINavigationController class]]) {
                //如果是NavigationController，取最后一个控制器（当前）
                controller = [controller.childViewControllers lastObject];
            } else if ([controller isKindOfClass:[UITabBarController class]]) {
                //如果TabBarController，取当前控制器
                UITabBarController *tabBarController = (UITabBarController *)controller;
                controller = tabBarController.selectedViewController;
            } else {
                if (controller.childViewControllers.count > 0) {
                    //如果是普通控制器，找childViewControllers最后一个
                    controller = [controller.childViewControllers lastObject];
                } else {
                    //没有present，没有childViewController，则表示当前控制器
                    return controller;
                }
            }
        }
    }
}
```

上面代码只处理了NavigationController和TabBarController，如果你没有修改过NavigationController和TabBarController的默认行为，页面跳转使用默认的push和present，那么是可以正常获取到的
