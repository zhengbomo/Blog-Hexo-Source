---
title: runloop学习笔记
categories: iOS
tags: [iOS, runloop]
date: 2017-05-17 10:01:26
updated: 2017-05-17 17:01:26
---


今天重新梳理了一下runloop的相关知识点，做一个总结
iOS上的runloop类似于Windows上的消息循环一样的东西，通过一个循环不断的接收事件处理

## 为什么要使用runloop

* 让程序一直活着，通过`do-while`循环
* 共享线程资源（一个程序多个任务）
* 解决同步问题（同一个线程）
* 节省CPU事件（挂起和唤醒机制）
* 调用解耦（对于调用方和被调用方而言）

<!--more-->

## runloop源代码
整个CoreFoundation都是开源的

CFRunLoop代码可以在这里找到：[http://opensource.apple.com/source/CF/CF-855.17/](http://opensource.apple.com/source/CF/CF-855.17/)
下载地址：[http://opensource.apple.com/tarballs/CF/CF-855.17.tar.gz](http://opensource.apple.com/tarballs/CF/CF-855.17.tar.gz)

## 涉及到runloop的框架类

### 系统级
* `GCD`：与runloop相互辅助，
* `mach内核`：通过内核进行进程间通信和线程调度
* `pthread`：runloop就是基于pthread线程的，线程锁

### 应用层
* `NSTimer`：基于runloop调用的
* `UIEvent`：通过消息抛给runloop调用
* `AutoreleasePool`：runloop在其循环周期和每个调用周期管理autoreleasepool的创建于释放
* `NSObject(NSDelayedPerforming)，NSObject(NSThreadPerformAddition)`
* `CADisplayLink，CATransition，CAAnimation`：动画执行用于动画帧计算并抛给runloop执行
* `dispatch_get_main_queue`：GCD中dispatch到main queue的block会被dispatch到main RunLoop执行
* `NSPort`：用于绑定端口用于进程间通信
* `NSURLConnection/AFNetworking`：通过线程的runloop处理回调，不影响主线程

## 构成
runloop在外层是通过NSRunloop来实现的，而NSRunloop内部是基于CFRunloop实现的，NSRunloop只是对CFRunloop的一个OC的封装，CFRunloop是通过C语言实现的，NSRunloop和CFRunloop的API基本上是一一对应的

> NSRunloop（Foundation） -> CFRunloop（CoreFoundation）

一个线程内部只可以有一个runloop，主线程的runloop是开启的，其他线程默认是不开启的
一个runloop内部可以包含多个`mode`
一个mode内部包含`source`, `timer`, `observer`三种类型的对象

    * source（CFRunLoopSource）: runloop的触发数据源，类似于接口，可以触发runloop执行相关的任务
    * timer（CFRunLoopTimer）: 定时器，由内核驱动
    * observer（CFRunLoopObserver）: 监听runloop的状态的对象



### runloop内部逻辑
```c
{
    /// 1. 通知Observers，即将进入RunLoop
    /// 此处有Observer会创建AutoreleasePool: _objc_autoreleasePoolPush();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopEntry);
    do {
        /// 2. 通知 Observers: 即将触发 Timer 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeTimers);
        /// 3. 通知 Observers: 即将触发 Source (非基于port的,Source0) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeSources);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 4. 触发 Source0 (非基于port的) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__(source0);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 6. 通知Observers，即将进入休眠
        /// 此处有Observer释放并新建AutoreleasePool: _objc_autoreleasePoolPop(); _objc_autoreleasePoolPush();
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeWaiting);
 
        /// 7. 进入休眠，等待新消息唤醒.
        mach_msg() -> mach_msg_trap();
 
        /// 8. 通知Observers，线程被唤醒
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopAfterWaiting);
 
        /// 9. 如果是被Timer唤醒的，回调Timer
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__(timer);
 
        /// 9. 如果是被dispatch唤醒的，执行所有调用 dispatch_async 等方法放入main queue 的 block
        __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(dispatched_block);
 
        /// 9. 如果如果Runloop是被 Source1 (基于port的) 的事件唤醒了，处理这个事件
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__(source1);
 
    } while (...);
 
    /// 10. 通知Observers，即将退出RunLoop
    /// 此处有Observer释放AutoreleasePool: _objc_autoreleasePoolPop();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopExit);
}
```

### Mode
一个runloop可以包含多个mode（CFRunLoopMode），而同一时间段内，只能有一个mode被运行（`_currentMode`），下面是runloopMode和runloop的结构

```c
struct __CFRunLoopMode {
    CFStringRef _name;            // ModeName, 例如：kCFRunLoopDefaultMode
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
 
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set<Source/Observer/Timer>
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```

iOS有下面几个NSRunLoopMode（对外开放的Mode有`kCFRunLoopDefaultMode`, `UITrackingRunLoopMode`和`kCFRunLoopCommonModes`三个）

1. kCFRunLoopDefaultMode: App的默认 Mode，通常主线程是在这个 Mode 下运行的
2. UITrackingRunLoopMode: 界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3. kCFRunLoopCommonModes: 这是一个占位的 Mode，没有实际作用，用于内部逻辑判断

4. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
5. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到


这里有个概念叫 "CommonModes"：一个 Mode 可以将自己标记为"Common"属性（通过将其 ModeName 添加到 RunLoop 的 "commonModes" 中）。每当 RunLoop 的内容发生变化时，RunLoop 都会自动将 _commonModeItems 里的 Source/Observer/Timer 同步到具有 "Common" 标记的所有Mode里。

例如：主线程默认会把`UITrackingRunLoopMode`和`kCFRunLoopDefaultMode`加到`_commonModes`上，当我们添加NSTimer到runloop并指定mode为`kCFRunLoopCommonModes`时，就可以同时出发UI响应和timer事件


CFRunLoop对外暴露的管理 Mode 接口只有下面2个，只能添加，不能删除，通过modeName操作mode，即上面的`kCFRunLoopDefaultMode`和`kCFRunLoopCommonModes`

> CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
> CFRunLoopRunInMode(CFStringRef modeName, ...);


Mode 暴露的管理 mode item 的接口有下面几个：

> CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
> CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
> CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
> CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
> CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
> CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);

> 引用：http://blog.ibireme.com/2015/05/18/runloop/

### CFRunLoopSource


### CFRunLoopTimer

### CFRunLoopTimer






### CFRunLoopObserver
runloop有下面集中状态
```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

## autorelease
我们都知道在MRC中autorelease对象会在当前autoreleasepool结束的时候被释放，并且通常情况下，autoreleasepool是自动创建并且自动销毁的，而autorelease对象也会在恰当的时候被释放，runloop会在下面几种情况下会创建和释放autoreleasepool

1. Observer监视到`kCFRunLoopEntry`状态时，其回调内会调用`_objc_autoreleasePoolPush`创建自动释放池（最外层）
2. Observer监视到`kCFRunLoopBeforeWaiting`状态时，其回调内会调用`_objc_autoreleasePoolPop`释放旧的自动释放池和`_objc_autoreleasePoolPush`创建自动释放池
3. Observer监视到`kCFRunLoopExit`状态时，其回调内会调用`_objc_autoreleasePoolPop`释放自动释放池（最外层）

故runloop的自动释放池在`kCFRunLoopBeforeWaiting`和`kCFRunLoopExit`两种状态下会被释放


