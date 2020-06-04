---
title: Swift方法派发机制
tags: [Swift, iOS]
date: 2020-05-23 19:02:12
updated: 2020-05-23 19:02:12
categories: iOS
---

方法派发机制是程序判断如何去调用函数或方法的机制，每次调用方法时都会触发，了解派发机制的工作原理，对于写出高性能的代码来说非常重要，派发机制也能解释一些Swift中的奇怪的现象，和Objective-C中的黑魔法

<!-- more -->

## 静态派发 vs 动态派发

根据函数调用能否在编译时或运行时确定，可以将派发机制分成两种类型：

* `静态派发`：在编译期的时候，编译器就知道要为某个方法调用某种实现。因此, 编译器可以执行某些优化，甚至在可能的情况下，可以将某些代码转换成inline函数，从而使整体执行速度异常快。
* `动态派发`：一定量的运行时开销为代价，提高了语言的灵活性。在动态派发机制下，对于每个方法的调用，编译器必须在方法列表(`witness table`或`virtial table`)中查找执行方法的实现，如在运行时判断选择父类的实现，还是子类的实现。由于对象的内存都是在运行时分配的，因此只能在运行时执行检查。

编译型语言有通常有三种基本的函数派发方式:

1. `直接派发`（Direct Dispatch）
   * 编译后就确定了方法的调用地址（也叫`静态派发`），汇编代码中，直接跳到方法的地址执行，生成的汇编指令最少，速度最快
   * 例如C语言，C++默认也是直接派发
   * 由于缺乏动态性，无法实现多态

2. `函数表派发`（Table Dispatch）
   * 在运行时通过一个函数表查找需要执行的方法，多一次查表的过程，速度比直接派发慢
   * C++的虚函数（Virtual Table），维护一个虚函数表，对象创建的时候会保存虚表的指针，调用方法之前，从对象中取出虚表地址，根据编译时的方法偏移量从虚表取出方法的地址，跳到方法的地址执行

3. `消息派发`（Message Dispatch）
   * Objective-C: 方法调用包装成消息，发给运行时（相当于`中间人`），运行时会找到类对象，类对象会保存类的数据信息，其中就包含方法列表（类方法在元类对象存储），或通过父类查找，直到命中执行，如果没找到方法，抛出异常，运行时提供了很多动态的方法用于改变消息派发的行为，相比函数表派发有很强的`动态性`，由于运行时支持的功能很多，方法查找的过程比较长，性能比较低

性能：直接派发 > 函数表派发 > 消息机制派发

函数表派发和消息派发属于`动态派发`

Swift支持上面三种函数派发方式，Swift编译器会根据不同的情况选择不同的派发方式，基于性能考虑优先选择性能高的派发方式

## Swift方法派发机制

这里先只讨论纯Swift对象（非继承自NSObject），继承自OC类的比较特殊，放到后面讨论

### 直接派发

在Swift中，下面方法会被编译为直接派发，在ARM64上调用方法会被编译为`bl 函数地址`

1. `全局函数`
2. 使用`static`声明的所有方法
3. 使用`final`声明的所有方法，使用final声明的类里面的所有方法

    ```swift
    class ParentClass {
        func method1() {}
        func method2() {}
    }
    final class ChildClass: ParentClass {
        override func method2() { }
        func method3() {}
    }

    let obj = ChildClass()
    // 下面调用都是直接派发
    obj.method1()
    obj.method2()
    obj.method3()
    ```

4. 使用`private`声明的方法和属性，会隐式`final`声明
5. `值类型`的方法，`struct`和`enum`都是值类型
6. 纯Swift（没有继承自NSObject的类）的`extension`中定义的所有方法

### 函数表派发

只有引用类型才支持函数表派发，在Swift中，类的方法默认使用函数派发的方式，Swift的函数表叫`witness table`（其他语言叫`virtual table`）

* 每个子类都有它自己的表结构
* 对于类中每个重写的方法，都有不同的函数指针
* 当子类添加新方法时，这些方法指针会添加在表数组的末尾
* 最后，编译器在运行时使用此表来查找调用函数的实现

```swift
class ParentClass {
    func method1() {}
    func method2() {}
}
class ChildClass: ParentClass {
    override func method2() {}
    func method3() {}
}

let obj = ChildClass()
obj.method2()
```

`obj`对象调用`method2`的过程

{% img /images/post/swift-table-dispatch.png 500 %}

* 读取对象`0xB00`的函数表.
* 读取函数指针的索引，在这里`method2`的索引是1(偏移量)，也就是`0xB00 + 1`
* 跳到`0x222`

### 消息派发

Swift支持和OC混编，支持有限的runtime运行时（主要是为了和OC混编），对了纯Swift类，为了可以给OC调用，可以在方法前面加上`dynamic`来支持消息派发（注意`@objc`只是用于把方法暴露给ObjectiveC）

```swift
class ParentClass {
    dynamic func method2() {}
}
```

当消息被派发时，运行时会顺着继承关系向上查找被调用的方法，为了能够提升消息派发的性能，一般都会将查找进行缓存

### 协议Protocol

协议所指向的对象，只有在运行时才能确定类型，Swift对于协议默认都使用`函数表派发`，协议可以为struct提供多态的支持

```swift
protocol Drawable {
    func draw()
}

struct Line: Drawable {
    func draw() {}
}

struct Point: Drawable {
    func draw() {}
}

let drawable1: Drawable = Line()
let drawable2: Drawable = Point()

drawable1.draw()        // 使用函数表派发
drawable1.draw()        // 使用函数表派发
```

### NSObject类

这里指继承自NSObject的类（包括UIView, UIButton等）

* 对于普通的实例方法，使用函数表派发
* 对于使用`@objc`声明的方法，会暴露给ObjectiveC，还是使用函数表派发
* 对于`override`的OC方法，使用消息派发
* 对于`extension`方法，默认使用直接派发
* 使用`dynamic`修饰的方法使用消息派发

```swift
class MyButton: UIButton {
    // 直接派发
    final func method1() {}

    // 直接派发
    static func method2() {}

    // 函数表派发
    func method3() {}

    // 函数表派发
    @objc
    func method4() {}

    // 消息派发派发
    @objc dynamic
    func method5() {}

    // 消息派发
    override func layoutSubviews() {
        super.layoutSubviews()
    }
}

extension MyButton {
    // 直接派发
    func method6() {}

    // 消息派发
    @objc func method7() {}

    // 直接派发
    dynamic func method8() {}
}
```

> 以上基于XCode11+Swift5测试，讨论的是未被编译器优化的情况，编译器会根据方法的使用情况做优化，函数表派发可能被优化成直接派发，部分方法会被优化城`inline`形式

## 引用

* [Understanding Swift Performance](https://developer.apple.com/videos/play/wwdc2016/416/)
