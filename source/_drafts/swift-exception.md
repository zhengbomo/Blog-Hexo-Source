---
title: swift-exception
date: 2016-07-22 19:42:07
updated: 2016-07-22 19:42:07
categories:
tags:
---

同其他高级语言一样，swift也提供了异常处理功能，并且更为安全

## 如何使用异常
在objc中，通常都使用error赋值或回调的方式处理异常，而在swift 2.0之后，处理异常最好的方式是定义自己的异常类型，继承自`ErrorType`

```swift
enum MyError: ErrorType {
  case NotExist
  case OutOfRange
}
```

如果一个函数可能会抛出异常，则需要在函数声明的时候加上`throws`声明（在参数后面）
```swift
func test() throws {
    guard let item = item else {
        // 抛出异常
        throw MyError.NotExist
    }
}
```

## 捕获异常
swift捕获异常通过`do-catch`，与其他语言的`try-catch`类似，在可能跑出异常的方法前加上`try`声明，并放在`do`代码块里面，`catch`的用法与`swift-case`的`case`类似，如果不声明捕获类型，则表示可以捕获任何类型
```swift
do {
    try test()
} catch MyError.NotExist(let a) {
  
} catch MyError.OutOfRange {
    // deal with not exist
}
```

## 不处理异常
默认情况下，所有的可能发生异常的方法必须放在`do-catch`里面，如果可以确保方法不会产生异常，可以使用`try!`声明，这样可以不处理异常
