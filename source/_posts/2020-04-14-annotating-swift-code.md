---
title: Swift Annotating
tags: [iOS, Swift]
date: 2020-04-14 09:51:28
updated: 2020-04-14 09:51:28
categories: iOS
---


除了普通的代码注释外，编译器通常也会提供一些辅助开发的注释，例如`TODO`，`FIXME`，`ERROR`之类的注释，这些注释可以被编译器识别到，并提供一些友好提示，我们应该多利用这些编译器的特性来辅助我们日常的开发

<!-- more -->

## TODO

```swift
func takePicture() -> Bool {
    // TODO: Do camera stuff
    return true
}
```

{% img /images/post/swift-annotating-todofixup.png 300 %}

## FIXME

```swift
func increment(_ num: Int) -> Int {
    // FIXME: Make this increment the input
    return num
}
```

{% img /images/post/swift-annotating-todofixup.gif 1000 %}

## warning

通过warning提示警告⚠️，可以编译通过

```swift
func increment(_ num: Int) -> Int {
    // FIXME: Make this increment the input
    #warning("Incorrect result returned")
    return num
}
```

{% img /images/post/swift-annotating-warning.png 500 %}

## error

`error`警告会编译不通过

```swift
func readArr(_ arr: [Int]) {
    for i in 0…arr.count {
        if i == arr.count {
            #error(“err”)
        }
    }
}
```

{% img /images/post/swift-annotating-error.png 500 %}

## 模拟器

在模拟器调试的特殊代码，相当于宏隔离，模拟器的代码不会编译到真机

```swift
@discardableResult
func takePicture() -> Bool {
    #if targetEnvironment(simulator)
         return false
    #else
        return true
    #endif
}
```



