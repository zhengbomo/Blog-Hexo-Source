---
title: swift构造器总结
categories: iOS
tags: swift
date: 2016-07-07 21:17:17
updated: 2016-07-07 21:17:17
---

swift构造器的语法和规则比其他语言复杂，有很多限制和检查，刚开始学的时候经常搞懵，在这里做一个简单的总结

<!-- more -->

## 一、基本用法
构造器与函数类似，但不是函数，构造器无返回值，使用方式与函数类似，名称为`init`，可以带参数
```swift
class Person {
    var name: String
    var gender: Int?

    // 定义一个无参数的构造器
    init() {
        name = ""
    }

    // 定义一个带参数的构造器
    init(name: String) {
        self.name = name
    }
}
```

构造器的作用是保证所有的成员（变量/常量）初始化完成，所有常量被赋值，所有非`Optional`变量都要进行赋值
```swift
class Person {
    var name: String                // 非Optional类型变量，必须初始化
    var age: Int?                   // Optional类型变量，可以不显式初始化
    let idNumber: String            // 非Optional类型常量，必须初始化
    let gender: Int?                // Optional类型常量，必须初始化

    // 定义一个无参数的构造器
    init() {
        name = ""
        idNumber = ""
        gender = nil
    }

    // 定义一个带参数的构造器
    init(name: String) {
        self.name = name
        idNumber = ""
        gender = nil

    }
}
```

## 二、便利构造器
上面我们看到的构造器，都是指定构造器（designated），指定构造器必须保证所有的成员都完成初始化了，我们可以对初始化话方法做一些方便的扩展，并且可以重用原指定构构造器的逻辑，这时候可以定义便利构造器（convenience），定义便利构造器使用`convenience init()`，如上面Person类的例子
```swift
class Person {
    var name: String                // 非Optional类型变量，必须初始化
    var age: Int?                   // Optional类型变量，可以不显式初始化
    let idNumber: String            // 非Optional类型常量，必须初始化
    let gender: Int?                // Optional类型常量，必须初始化

    // 定义一个无参数的构造器
    init() {
        name = ""
        idNumber = ""
        gender = nil
    }

    // 定义便利构造器
    convenience init(name: String) {
        self.init()                 // 调用指定构造器
        self.name = name
    }
}
```

便利构造器不能被子类继承，子类不能通过`super`调用，如果希望便利构造器被子类继承，可以使用`required`声明子类必须重写该方法
```swift
class Person {
    var name: String                // 非Optional类型变量，必须初始化
    var age: Int?                   // Optional类型变量，可以不显式初始化
    let idNumber: String            // 非Optional类型常量，必须初始化
    let gender: Int?                // Optional类型常量，必须初始化

    // 定义一个无参数的构造器
    init() {
        name = ""
        idNumber = ""
        gender = nil
    }

    // 定义一个带参数的构造器
    required convenience init(name: String) {
        self.init()
        self.name = name
    }
}

class Student: Person {
    var classNo: String = ""

    required init(name: String) {

    }
}
```

便利构造器只能调用当前类的指定构造器，不能调用父类的指定构造器，类不能只有便利构造器，便利构造器是**横向**调用（当前类）而指定构造器是**纵向**调用的（父类）

> 开始使用`convenience`声明便利构造器感觉有点奇葩，因为编译器是可以识别到构造器是否调用了其他指定构造器（designated）的，也就是说可以，却还需要我们自己声明，当然自己声明可以方便我们区分构造器，以便更直观的看出指定构造器和便利构造器，这样可以很快的分辨是否可以被子类使用


## 三、可失败构造器（Failable initializer）
除了普通的构造器，swift还允许构造失败，如当我们传入一些不恰当的参数导致类构造出现一些异常的情况，这时候我们可以让构造器返回nil，说明构造失败，声明方式为`init?()`，在需要的地方`return nil`
```swift
class Person {
    var name: String
    var gender: Int
    // 定义一个无参数的构造器
    init?(name: String, gender: Int) {
        self.name = name
        if gender != 0 && gender != 1 {
            return nil      // 失败的地方返回nil即可
        }
        self.gender = gender
    }
}
var person = Person(name: "bomo", gender: 2)
if let p = person {
    print(p.name)
}
```
可失败构造这名字太难听了，我更喜欢叫为可空构造器，返回是一个`Optional`类型，使用的时候需要先取值

## 四、必须构造器
类可以定义`required`构造器，当子类继承父类是，必须实现`required`构造器，当然，如果子类不实现任何构造器时，也可以不实现`required`构造器
```swift
class Parent {
    var name: String
    required init(name: String) {
        self.name = name
    }
}
class Son: Parent {
    required init(name: String) {       //子类必须实现
        super.init(name: name)
    }
    init() {
        super.init(name: "bomo")
    }
}

class Son2: Parent {      // 如果子类不实现任何构造器，则不可以不实现required
}
```

## 五、闭包初始化属性
出行的初始化可以通过构造器设置，也可以在声明的时候直接设置，swift还支持通过执行闭包函数初始化属性
```swift
class SomeClass {
    let someProperty: SomeType = {
        // 这里不能使用类相关的属性，因为在这里类还没有初始化完成
        return someValue
    }()   // 最后需要执行闭包，否则返回的是闭包，而不是闭包函数运行后返回的值
}
```

## 六、继承
    * 子类只能继承父类的指定构造器和`required`声明的构造器，required声明构造器表示子类必须重写该构造器
    * 如果子类实现了父类的**部分**指定构造器，则不会继承父类的构造器
    * 如果子类实现了父类的**所有**指定构造器，则会继承父类
    * 如果子类没有实现任何构造器，则会继承父类的**所有构造器**
    * 子类的指定构造器最终必须调用父类的指定构造器
    * 子类调用父类构造器之前必须保证当前类的所有属性已初始化为适当的值（非可选变量都已赋值，所有常量都已赋值）
    * 子类使用父类的属性时，必须在调用父类指定构造器之后


所有的构造器都必须保证，在执行完构造器时，所有的属性都初始化为适当的值

## 七、两段式构造
swift的构造器有点奇葩，分为两段式构造
    * 第一阶段：为每个属性初始化一个初始值
    * 第二阶段：为每个属性定制化值

在其他语言中C#/Java中，系统默认会为对象的所有属性和字段初始化值，如值类型初始化为0，引用类型初始化为null，结构体成员遵循前两条，系统自动为我们完成了第一阶段，我们直接进入第二阶段，直接定制化属性/字段值，而swift把这两个阶段分开给我们实现了

## 八、外部参数名
所有带参数的构造器在调用的时候必须显式使用外部参数名，如果不显式声明外部参数名，外部参数名与内部参数名相同（与函数的规则一样）
```swift
class Person {
    var name: String
    // 定义一个带参数的构造器
    init(outName name: String) {
        self.name = name
    }
}
let p1 = Person("bomo")            // 报编译错误，需要指定外部参数名
var p2 = Person(outName: "")
```

如果想忽略外部参数名，可以使用`_`代替外部参数名
```swift
class Person {
    var name: String
    // 定义一个带参数的构造器
    init(_ name: String) {
        self.name = name
    }
}
let p = Person("bomo")            // 编译通过
```

## 九、构造器总结
### 属性
* 如果所有成员都进行了初始化，并且没有提供构造器，编译器会自动为类生成一个无参数构造器，如果是结构体，那么还会生成一个带所有参数的构造器

### 指定构造器和便利构造器
* 指定构造器总是纵向向上代理，便利构造器总是横向代理
* 便利构造器只能调用当前类内的构造器
* 便利构造器最终需调用指定构造器

### 4种检查
* 指定构造器必须保证所有的成员属性都被初始化
* 指定构造器（如果有父类）必须最终调用父类的构造器
* 指定构造器调用父类构造器之前必须保证当前类的所有成员已完成初始化

### 继承
* 父类构造器通常情况下不会被子类继承，只有子类无构造器的时候才会
* 子类可以通过`super`调用父类的指定构造器，不能调用父类的便利构造器
* 如果有父类，所有的构造器最终都必须调用父类的指定构造器
* 使用父类的属性之前，必须先调用父类的构造器，再使用
