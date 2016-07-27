---
title: swift反射
date: 2016-07-14 11:51:24
updated: 2016-07-14 11:51:24
categories: iOS
tags: swift
---

## 简介

熟悉C#和Java的开发者应该对于反射的概念很熟悉了，就是可以运行时动态的获取类信息，包括成员属性，变量，方法，还可以动态调用属性，方法

而在oc开发中，很少提到反射这个概念，而通常说的是runtime，其实oc的runtime就是为oc提供反射特性的支持，甚至比其他语言更为强大

swift其实支持反射的一些特性，swift不推荐使用oc中的runtime实现反射，而是推荐像其他高级语言的方式使用反射，swift中通过一个Mirror结构体对象实现反射

## Mirror包含哪些数据
### 1. displayStyle
描述对象的类型
```swift
public enum DisplayStyle {
    case Struct
    case Class
    case Enum
    case Tuple
    case Optional
    case Collection
    case Dictionary
    case Set
}
```

如果不在下面枚举，则会返回nil，如闭包类型
```swift
let closure = { (a: Int) -> Int in return a * 2 }
let mirror = Mirror(reflecting: closure)
mirror.displayStyle           // nil
```

### 2. subjectType
获取当前对象的类型
```swift
class Parent {
    var name: String?
}
class Son: Parent {
    var age: Int = 0
}

let son = Son()
let mirror = Mirror(reflecting: son)
mirror.subjectType            // Son.Type
```

### 3. children
对象的成员属性（不包含函数，计算属性，嵌套类），swift通过一个元组`Child`描述成员属性的名字和类型，第一个成员描述变量名，第二个描述变量的值
```swift
public typealias Child = (label: String?, value: Any)
```
变量名是可能为空的，例如元组，元组的成员没有具体的变量名
```swift
let son = Son()
let mirror = Mirror(reflecting: son)
for (key, value) in mirror.children {
    print("key=\(key), value=\(value)")
}
```

### 4. superclassMirror方法
Mirror可以通过superclassMirror获取到父类的Mirror对象，如果没有父类，返回nil
```swift
let son = Son()
let mirror = Mirror(reflecting: son)
let parentMirror = mirror.superclassMirror()
```

### 4. AncestorRepresentation枚举
TODO:

## 反射的基本方法
* 获取获取类型
* 动态构造对象
* 动态调用方法
* 获取类属性，方法，父类，协议


### 1. 获取对象的类型对象
```swift
let son = Son()
// 1. 通过dynamicType获取
let sonType1 = son.dynamicType

// 2. 通过Mirror对象获取
let mirror = Mirror(reflecting: son)
let sonType2 = mirror.subjectType

// 3. 硬编码通过类名获取
let sonType = Son.self

// 4. 如果是oc的对象（继承自NSObject）可以NSClassFromString获取
let stringType = NSClassFromString("NSString")
```

### 2. 根据类型构造对象
```swift


```
