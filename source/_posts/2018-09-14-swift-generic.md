---
title: swift泛型笔记
tags: [iOS, Swift]
date: 2018-06-14 07:41:49
updated: 2018-06-14 07:41:49
categories: [iOS]
---


泛型可以让代码处理类型更加灵活，在某些场景下可以很大程度的重用代码，泛型是什么，使用泛型的好处，这里不多说，网上有很多文章介绍的很详细，这里只讨论用法，Swift的泛型与其他语言有些类型，又有些不一样，搜了一下发现，网上的文章只描述了一点，并不全面，看完后依然没能很全面的说明泛型的用法，在这里记录完整的用法

<!-- more -->

## 定义泛型

swift的泛型定义方式有两种，一种通过`<T>`指定泛型参数，而在属性或方法，使用指定的泛型类型，下面是一个泛型方法的声明

### 泛型方法

```swift
/// 交换两个同类型的变量
func swap<T>(a: inout T, b: inout T) {
    let c = a
    a = b
    b = c
}
var (a, b) = (2, 1)

swap(&a, &b)
print(a, b)			// 1, 2

swap(&a, &b)        
print(a, b)			// 2, 1
```

### 泛型类
上面是泛型方法的定义，如果在类里面，可以对类声明泛型
```swift
class Test<T> {
    /// 泛型属性
    var testObj: T?

    /// 泛型方法，可以直接使用类的泛型参数T
    func swap<T>(a: inout T, b: inout T) {
        let c = a
        a = b
        b = c
    }

    /// 方法的泛型定义可以不依赖于类的泛型参数
    func swap2<TK>(a: inout TK, b: inout TK) {
        let c = a
        a = b
        b = c
    }
}

// 属性
let t = Test<Int>()
t.testObj = 12
print(t.testObj)        // 12

var a: Int? = 2
var b: Int? = 1
t.swap(a: &a, b: &b)
print(a, b)            // 1, 2

var s1: String? = "a"
var s2: String? = "b"
t.swap2(a: &s1, b: &s2)
print(s1, s2)            // b, a
```

### 泛型参数
泛型可以支持多个类型参数，在声明处用逗号分开
```swift
class Test {
	/// 多个泛型参数
    func test<T1, T2>(t1: T1, t2: T2) -> Int {
        print("t1: \(t1), t2: \(t2)")
        return 1
    }
}

let t = Test()
let result = t.test(t1: NSObject(), t2: "abc")
```

## 泛型约束
类型约束定义在泛型声明处，支持类约束和`protocol`约束，如下
```swift
protocol Run {
    func run()
}

protocol Fly {
    func fly()
}

/// 单协议约束
func test<T: Run>(animal: T) {
    animal.run()
}

/// 类型和多协议约束
func test<T: NSObject & Fly & Run>(bird: T) {
    bird.fly()
    bird.run()
}
```

泛型可以用在方法参数，方法返回值，属性上


## 协议泛型
除了类，结构体和枚举也支持泛型，用法与类一样，但是协议protocol不能像上面一样使用
协议只能通过关联类型`associatedtype`来实现泛型的功能，相当于泛型声明为协议的成员，而泛型的成员在实现的时候指定

```swift
protocol Write {
    /// 关联类型Element，相当于上面的T
    associatedtype Element
    func write(_ element: Element)
}

class File: Write {
    // 在实现关联类型的协议的时候，需要指定关联类型
    typealias Element = String

    // 实现协议方法
    func write(_ element: File.Element) {
        print(element)
    }
}
```
### 关联类型的约束
关联类型的约束与普通泛型约束一样，在声明的地方后面添加，通过逗号隔开
```swift
 protocol Write {
    associatedtype Element: NSObject, Encodable
    func write(_ element: Element)
}
```

### 关联自身类型
protocol利用`associatedtype`关联自身类型
```swift
protocol Equalable {
    func equal(_ a: Self) -> Bool
}

class Test: Equalable {
    var id: Int = 0
    
    init(id: Int) {
        self.id = id
    }
    
    func equal(_ a: Test) -> Bool {
        return self.id == a.id
    }
}

let a = Test(id: 1)
let b = Test(id: 2)
a.equal(b)              // false
```
### associatedtype冲突
associatedtype有另一个问题没有解决，就是类型冲突，如下
```swift
protocol Read {
    associatedtype Element
    func read() -> Element
}
protocol Write {
    associatedtype Element
    func write(a: Element)
}

// 两个Element名字相同，无法指定
class ReadWrite: Read, Write {
    func read() -> Int {
        return 5
    }
    func write(a: String) {
        print("writing \(a)")
    }
}
```

在stackoverflow上有个不完美的解决方案:
[https://stackoverflow.com/questions/37736457/protocol-with-same-associated-type-name ](https://stackoverflow.com/questions/37736457/protocol-with-same-associated-type-name )

```swift
protocol ReadInt: Read {
    associatedtype Element = Int
}
protocol WriteString: Write {
    associatedtype Element = String
}

class ReadWrite: ReadInt, WriteString {
    func read() -> Int {
        return 5
    }
    func write(a: String) {
        print("writing \(a)")
    }
}
```


### 关于associatedtype
对于其他语言大多都使用直接指定类型`<T>`的方式声明泛型，而swift为什么还要搞出一个`associatedtype`呢，这篇文章有分析：[http://www.cocoachina.com/swift/20160726/17188.html](http://www.cocoachina.com/swift/20160726/17188.html)


使用关联类型，类似于类型当成成员属性使用，可以解耦依赖，如果需要使用对象的关联的类型，而类型与当前对象并没有直接关系，则通过成员的关联类型引用可以避免直接依赖对象类型，如下例子：机动车(Automobile)、燃料(Fuel)、尾气(Exhaust)

```swift
/// 机动车
public protocol Automobile {
  	associatedtype FuelType
  	associatedtype ExhaustType
  	func drive(fuel: FuelType) -> ExhaustType
}

/// 燃料
public protocol Fuel {
  	associatedtype ExhaustType
  	func consume() -> ExhaustType
}

/// 尾气
public protocol Exhaust {  
	func emit()
}
```

实现

```swift
public struct UnleadedGasoline: Fuel {
  	public func consume() -> E {    
  		print("consuming unleaded gas...")  
  		return E()
  	}
}

public struct CleanExhaust: Exhaust {
  	public func emit() {  
  		print("this is some clean exhaust...")
  	}
}

public class Car: Automobile {
 	public func drive(fuel: F) -> E {    
 		return fuel.consume()
  	}
}
```


实际依赖关系：  
机动车 -> 燃料  
燃料 -> 尾气  

但是代码有个问题是在定义Car的时候，必须声明尾气的类型E，而实际上机动车是不依赖于尾气的类型的，尾气的类型取决于燃料，如果Car支持两种燃料FuelA和FuelB，而两种燃料产生的尾气E1和E2，`drive`方法无法同时满足，只能定义两个方法

```swift
public class Car: Automobile {

 	public func drive(fuel: F1) -> E1 {    
 		return fuel.consume()
  	}
    public func drive(fuel: F2) -> E2 {    
 		return fuel.consume()
  	}
}
```

如果使用关联类型的话可以解决这个问题，把尾气关联到燃料上

```swift
public class Car: Automobile {
 	public func drive(fuel: F) -> F.ExhaustType {    
 		return fuel.consume()
    }
}
```

以上就是Swift泛型的全部要点