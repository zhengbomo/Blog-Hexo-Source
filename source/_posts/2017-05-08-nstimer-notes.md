---
title: NSTimer学习笔记
categories: iOS
tags: [iOS]
date: 2017-05-08 14:22:13
updated: 2017-05-08 14:22:13
---

NSTimer是iOS最常用的定时器工具之一，在使用的时候常常会遇到各种各样的问题，最常见的是内存泄漏，通常我们使用NSTimer的一般流程是这样的

1. 在ViewController初始化或加载的地方创建`NSTimer`，并且通过属性持有（为了关闭）
2. 在ViewController的`dealloc`方法关闭定时器（`invalidate`），并且把NSTimer置为`nil`

上面做法可能会造成内存泄漏，`invalidate`方法通常不能放在NStimer.target.dealloc里面，因为NSTimer会对target强引用，而如果target对NSTimer强引用就会造成循环引用

<!-- more -->

## 1. 构造函数
NSTimer只有被添加的Runloop才能生效，NSTimer有下面两种类型的构造函数
* initWithFireDate
* timerWithTimeInterval
* scheduledTimerWithTimeInterval

`scheduledTimerWithTimeInterval`除了构造timer，还会把timer添加到当前线程的runloop，所以我们通常使用`scheduledTimerWithTimeInterval`构造NSTimer而不是`timerWithTimeInterval`

1. 没有添加到runloop的timer，调用fire的时候会直接触发，并且只触发一次（如果repeat:YES）
```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    [self timer1];
    //[self timer2];
    //[self timer3];
    //[self timer4];
}

- (void)timer1 {
    self.timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
    // 不会触发
}

- (void)timer2 {
    self.timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
    // 正常触发
    [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSDefaultRunLoopMode];
}

- (IBAction)invalidate:(id)sender {
    [self.timer invalidate];
    self.timer = nil;
}

- (void)timerTest:(NSObject *)obj {
    NSLog(@"time fire");
}
```

2. 如果使用`timerWithTimeInterval`或`initWithFireDate`构造，需要手动添加到runloop上，使用`scheduledTimerWithTimeInterval`则不需要
```objc
- (void)timer3 {
    self.timer = [[NSTimer alloc] initWithFireDate:[NSDate dateWithTimeIntervalSinceNow:3] interval:3 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
    // 需要添加到runloop才能触发
    [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSDefaultRunLoopMode];
}

- (void)timer4 {
    // 自动添加到runloop
    self.timer = [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
}
```

## 2. NSTimer的触发
* NSTimer在添加到runloop时，timer开始计时，即使runloop没有开启（`run`），在构造NSTimer的时候，如果不是马上开始计时，可以先使用`timerWithTimeInterval`再手动加入runloop上
* 调用`fire`的时候，立即触发timer的方法，该方法触发不影响计时器原本的计时，只是新增一次触发
* 当NSTimer进入后台的时，NSTimer计时暂停，进入前台继续

## 3. NSTimer和Runloop
上面构造函数我们可以看到，当我们把timer添加到runloop的时候会指定NSRunLoopMode（scheduledTimerWithTimeInterval默认使用NSDefaultRunLoopMode），iOS支持的有下面两种模式

* NSDefaultRunLoopMode：默认的运行模式，用于大部分操作，除了NSConnection对象事件。
* NSRunLoopCommonModes：是一个模式集合，当绑定一个事件源到这个模式集合的时候就相当于绑定到了集合内的每一个模式。

下面三种是内部框架支持（AppKit）

* NSConnectionReplyMode：用来监控NSConnection对象的回复的，很少能够用到。
* NSModalPanelRunLoopMode：用于标明和Mode Panel相关的事件。
* NSEventTrackingRunLoopMode：用于跟踪触摸事件触发的模式（例如UIScrollView上下滚动）。


当timer添加到主线程的runloop时，某些UI事件（如：UIScrollView的拖动操作）会将runloop切换到`NSEventTrackingRunLoopMode`模式下，在这个模式下，`NSDefaultRunLoopMode`模式注册的事件是不会被执行的，也就是通过`scheduledTimerWithTimeInterval`方法添加到runloop的NSTimer这时候是不会被执行的

为了让NSTimer不被UI事件干扰，我们需要将注册到runloop的timer的mode设为`NSRunLoopCommonModes`，这个模式等效于NSDefaultRunLoopMode和NSEventTrackingRunLoopMode的结合
```objc
// 主线程
self.timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```

## 4. 循环引用
循环引用是最经常遇到的问题之一
* NSTimer在构造函数会对target强引用，在调用`invalidate`时，会移除去target的强引用
    ```objc
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)self));
    
    NSTimer *timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:@"ghi" repeats:YES];
    
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)self));

    [timer invalidate];
    
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)self));
    ```
    输出如下
    ```bash
    2017-05-09 10:41:45.071 NSTimerTest[6861:914021] Retain count is 6
    2017-05-09 10:41:46.056 NSTimerTest[6861:914021] Retain count is 7
    2017-05-09 10:41:47.848 NSTimerTest[6861:914021] Retain count is 6
    ```

* NSTimer被加到Runloop的时候，会被runloop强引用持有，在调用invalidate的时候，会从runloop删除
    ```objc
    NSTimer *timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:@"ghi" repeats:YES];
    
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)timer));
    
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
    
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)timer));
    
    [timer invalidate];
    
    NSLog(@"Retain count is %ld", CFGetRetainCount((__bridge CFTypeRef)timer));
    ```
    输出如下
    ```
    2017-05-09 09:37:30.573 NSTimerTest[6505:883666] Retain count is 1
    2017-05-09 09:37:33.177 NSTimerTest[6505:883666] Retain count is 2
    2017-05-09 09:38:19.111 NSTimerTest[6505:883666] Retain count is 1
    ```
* 当定时器是不重复的（repeat=NO），在**执行完**触发函数后，会自动调用`invalidate`解除runloop的注册和接触对target的强引用

由于NSTimer被加到runloop的时候会被runloop强引用，故如果使用`scheduledTimerWithTimeInterval`构造函数时，我们可以在viewcontroller使用`weak`引用NSTimer
```objc
@property (nonatomic, weak) NSTimer *timer;
...

- (void)viewDidLoad {
    [super viewDidLoad];

    // 由于timer会被当前线程的runloop持有，故可以使用weak引用，而当调用invalidate时，self.timer会被自动置为nil
    self.timer = [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];

    // 或者
    NSTimer *timer2 = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer2 forMode:NSDefaultRunLoopMode];
    self.timer = timer;

}
```

所以通常我们不能在`dealloc`方法让`[timer invalidate]`, 因为timer在invalidate之前，会引用self（通常是ViewController），导致self无法释放，可以在`viewDidDisappear`或显式调用timer的`invalidate`方法

> invalidate是唯一让timer从runloop删除的方法，也是唯一去除对target强引用的方法


## 5. 多线程
如果我们不在主线程使用Timer的时候，即使我们把timer添加到runloop，也不能被触发，因为主线程的runloop默认是开启的，而其他线程的runloop默认没有实现runloop，并且在后台线程使用NSTimer不能通过fire启动定时器，只能通过runloop不断的运行下去

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    // 使用新线程
    [NSThread detachNewThreadSelector:@selector(startNewThread) toTarget:self withObject:nil];
}

- (void)startNewThread {
    self.timer = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];

    // 添加到runloop
    NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
    [runLoop addTimer:self.timer forMode:NSDefaultRunLoopMode];

    // 非主线程需要手动运行runloop，run方法会阻塞，直到没有输入源的时候返回（例如：timer从runloop中移除，invalidate）
    [runLoop run]
}
```

## 6. NSTimer准确性
通常我们使用NSTimer的时候都是在主线程使用的，主线程负责很多复杂的操作，例如UI处理，UI时间响应，并且iOS上的主线程是优先响应UI事件的，而NSTimer的优先级较低，有时候NSTimer的触发并不准确，例如当我们在滑动UIScrollView的时候，NSTimer就会延迟触发，主线优先响应UI的操作，只有UIScrollView停止了才触发NSTimer的事件
解决方案
NSTimer加入到runloop默认的Mode为`NSDefaultRunLoopMode`， 我们需要手动设置Mode为`NSRunLoopCommonModes`
这时候，NSTimer即使在UI持续操作过程中也能得到触发，当然，会降低流畅度

NSTimer触发是不精确的，如果由于某些原因错过了触发时间，例如执行了一个长时间的任务，那么NSTimer不会延后执行，而是会等下一次触发，相当于等公交错过了，只能等下一趟车，`tolerance`属性可以设置误差范围

```objc
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(timerTest:) userInfo:nil repeats:YES];
// 误差范围1s内
timer.tolerance = 1;
```

> 如果对精度有要求，可以使用GCD的定时器

## 7 NSTimer暂停/继续
NSTimer不支持暂停和继续，如果需要可以使用GCD的定时器

## 8. 后台运行
NSTimer不支持后台运行（真机），但是模拟器上App进入后台的时候，NSTimer还会持续触发

如果需要后台运行可以通过下面两种方式支持
1. 让App支持后台运行（运行音频）（在后台可以触发）
2. 记录离开和进入App的时间，手动控制计时器（在后台不能触发）

第一种控制起来比较麻烦，通常建议手动控制，不在后台触发计时

## 9. performSelector
NSObject对象有一个`performSelector`可以用于延迟执行一个方法，其实该方法内部是启用一个Timer并添加到当前线程的runloop，原理与NSTimer一样，所以在非主线程使用的时候，需要保证线程的runloop是运行的，否则不会得到执行

如下
```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    [NSThread detachNewThreadSelector:@selector(startNewThread) toTarget:self withObject:nil];
}

- (void)startNewThread {
    // test方法不会触发，因为runloop默认不开启
    [self performSelector:@selector(test) withObject:nil afterDelay:1];
}

- (void)test {
    NSLog(@"test trigger");
}
```

## 10. 总结
总的来说使用NSTimer有两点需要注意
1. NSTimer只有被注册到runloop才能起作用，fire不是开启定时器的方法，只是触发一次定时器的方法
2. NSTimer会强引用target
3. `invalidate`取消runloop的注册和target的强引用，如果是非重复的定时器，则在触发时会自动调用`invalidate`

通常我们自己封装GCD定时器使用起来更为方便，不会有这些问题