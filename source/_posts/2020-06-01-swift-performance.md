---
title: Swift性能优化（一）
tags: [iOS, Swift]
date: 2020-06-01 21:31:37
updated: 2020-06-01 21:31:37
categories: iOS
---

最近学习了Swift底层原理相关的视频和文章，收获颇丰，趁热打铁，记录和总结对Swift的理解，对于Swift性能优化主要从下面三个方面入手

<!-- more -->

* 内存分配
* 引用计数
* 方法派发方式

## 内存分配

在程序运行过程中，我们控制的内存主要有两个下面两个区域（DATA段也能修改，但对性能影响不大）

* 栈(Stack)：由操作系统管理，通常用来执行函数，存放局部变量和临时变量
  * 对于在栈上分配内存和释放只是堆栈指针的移动（入栈和出栈），并且不需要增加额外的数据
* 堆(Heap): 由开发者自行管理内存的生命周期，通常用于存放类对象
  * 对于在堆上分配内存，需要更`高级的数据结构`
  * 申请内存的时候需要`搜索堆空间`，寻找合适的闲置内存块
  * 需要添加额外的数据用于管理内存（如`引用计数`）
  * 对于引用计数的操作需要具备`原子性`（线程安全）
  * 堆上的内存还存在`线程安全`的问题

Swift 中的数据类型可以分成两种：`值类型`（Struct, Enum）、`引用类型`（Class）。两者的内存分配区域是不同的，值类型默认分配在栈区，引用类型默认分配在堆区

### 栈分配

{% img /images/post/swift/struct-on-stack.png 800%}

### 堆分配

{% img /images/post/swift/class-on-stack.png 800%}

### 优化

在值类型和引用类型的选择上，应该更多使用值类型，对于调用频繁的方法，应该减少在堆创建对象，如下

```swift
enum Color { case blue, green, gray }
enum Orientation { case left, right }
enum Tail { case none, tail, bubble }

var cache = [String: UIImage]()

/// 创建聊天气泡
func makeBalloon(_ color: Color, orientation: Orientation, tail: Tail) -> UIImage {
    let key = "\(color):\(orientation):\(tail)"
    if let image = cache[key] {
        return image
    }
    ...
}
```

上面`key`由于是动态创建的，会被分配到堆上，考虑用结构体包装，可以避免频繁在堆创建对象

```swift
struct Attribute: Hashable {
    var color: Color
    var orientation: Orientation
    var tail: Tail
}

var cache = [Attribute: UIImage]()

func makeBalloon(_ color: Color, orientation: Orientation, tail: Tail) -> UIImage {
    let key = Attribute(color: color, orientation:orientation, tail:tail)
    if let image = cache[key] {
        return image
    }
    ...
}
```

### 小结

对于需要频繁分配内存的需求，应尽量使用 `Struct`/`Enum` 代替 `Class`。因为栈区的内存分配速度更快，更安全。

## 引用计数

上面例子可以看到，class Point在堆分配时候，会额外分配两个字段，第一个是函数表，用来实现多态，另一个就是`引用计数`，用于内存管理，上面的Point类可以看成下面代码

```swift
class Point {
    var refCount: Int
    var x, y: Double
    func draw() {}
}

let point1 = Point(x: 0, y: 0)      // 引用计数=1
let point2 = point1
retain(point2)                      // 引用计数+1
point2.x = 5
// use point1
release(point1)                     // 引用计数-1
// use point2
release(point2)                     // 引用计数-1，引用计数==0，释放Point在堆中的内存
```

* 引用计数是间接的管理内存，当引用计数为0时，Swift会将对应的内存释放
* 引用计数的操作是高频率的
* 引用计数的操作具备原子性（考虑线程安全），会带来一定的开销

虽然栈上的内存分配会比堆上块，但是有时候，使用栈会增加引用计数的操作（栈上的结构体使用了类对象，类对象在堆上分配），从而影响性能，如下

```swift
struct Label {
    var text: String
    var font: UIFont
}

let label1 = Label(text: "Hi", font: font)
let label2 = label1
retain(label2.text._storeage)
retain(label2.font)
// use label1
release(label1.text._storeage)
release(label1.font)
// use label2
release(label2.text._storeage)
release(label2.font)
```

上面可以看到，每次label拷贝的时候，都会带来所有引用变量retain（上面例子是2个，如果多的话影响会更大）

### 小结

对于如果结构体包含多个引用成员的时候，需要考虑把结构体改成类，以减少在结构体拷贝的时候，对所有引用成员进行retain

## 派发方式

Swift的函数派发有

* 直接派发
  * 全局方法
  * 使用`static`和`final`修饰的类和方法
  * 使用`private`修饰的属性和方法会隐式添加`final`
  * `值类型`(struct, enum)的方法
  * `extension`里面没有用`@objc`修饰的方法
* 函数表派发
  * 使用protocol调用的方法
  * class的实例方法
  * 使用
* 消息派发
  * class中使用`dynamic`修饰的方法
  * 继承自OC对象的方法

性能：直接派发 > 函数表派发 > 消息派发

### 小结

出于性能的考虑，我们尽量

* 使用`final`来修饰不会被重载的方法，如果class不会被重载，可以设置为final
* 使用`private`来修饰不会被外部访问到的属性和方法
* 从而提高函数的派发性能

## 引用

* [Understanding Swift Performance](https://developer.apple.com/videos/play/wwdc2016/416/)

未完待续~
