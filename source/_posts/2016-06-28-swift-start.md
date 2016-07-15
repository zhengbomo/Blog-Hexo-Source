---
title: swift学习笔记
categories: iOS
date: 2016-06-28 15:00:04
updated: 2016-06-28 15:00:04
tags: swift
---

一直没有时间好好看一下swift，最近复习了一遍语法，这里记录swift学习过程中遇到的一些问题和要点，和Object-C的一些相关特性这里也不做介绍，只记录swift特有的一些特性

swift借鉴了很多语言的语法，特别是脚本语言，在swift里，可以看到python语言的一些影子，很多地方是借鉴了python的一些用法

<!-- more -->


## 一、基础语法
* swift语句结束不需要分号（写了也没有问题），有一种情况需要分号，如果一行代码中有多条语句，这时候就必须要分号隔开
* swift字符串，数组语法糖，字典语法糖不需要`@`标示
* swift是类型安全的语言，具有强大的类型推测，所以很多时候我们不需要声明类型


## 二、数据类型
* 与objc一样，swift支持以前（objc）使用的所有数据类型，swift的类型名字首字母大写，如Int, Float, NSInteger
* swift支持可选类型（Optionals）类型，相当于C#中的可空类型，标识变量可能为空，基础数据类型也可为空，可选类型不能直接赋非可选类型
  ```swift
  var a: Int? = 10
  var b: Int = a          // 报错，不同类型不能赋值
  ```
* swift的布尔类型使用`true/false`，而不用`YES/NO`
* swift支持使用`_`来分割数值来增强可读性而不影响值，如一亿可以表示为下面形式
  ```swift
  let oneMillion = 1_000_000
  ```
* swift数值类型进行运算符计算的时候不会自动进行类型转换
  ```swift
  var a: Int = 12
  var b: Float = 23
  var c = a + b         // 报错
  ```
* swift的基础数据类型与对象类型一视同仁，可以混用，不需要装箱和拆箱

## 三、常量变量
* 与`C/Obj-C`不同，swift的常量更为广义，支持**任意类型**
* swift的变量和常量在声明的时候类型就已经确定（由编译器自动识别或开发者指定）

```swift
// 常量：使用let声明
let a = NSMutableArray()
let b = 12
let c: Float = 12       // 类型标注(type annotation)
let d = b + 12
a.addObject(11)         // str == [11]
let e = a               // str == [11], d == [11]
a.addObject(12)         // str == [11, 12], d == [11, 12]

// 变量：使用var声明
var f: Double? = 12
var g = "hello world"
```

## 四、序列和集合
### 1. 数组Array
swift的数组可以是有类型的（通过泛型），存放同类型的数据，如果添加一个错误的类型会报编译错误，默认情况下编译器会自动识别
```swift
//1. 数组的写法为：Array<Int>，也可以简写成[Int]
//2. 数组初始化与NSArray类似，直接用中括号括起来，里面值用逗号隔开
var array0 = [Int]()
var array1: [Int] = [1, 3, 5, 7, 9]
var array2: Array<Int> = array1

array1.append(11)             // [1, 3, 5, 7, 9, 11]
array1.insert(0, atIndex: 0)  // [0, 1, 3, 5, 7, 9, 11]
array1.isEmpty                // False
array1.count                  // 7

// 3. 如果初始化时不指定类型，而编译器也不能识别出类型，这时候，会被当成NSArray处理
var array3 = []                       // array3 为 NSArray类型的空数组

// 4. 如果声明的时候使用不同的类型，编译器会把数组识别为NSObject类型
var array4 = ["fdsa", 121]            // array4 为 Array<NSObject> 类型

// 5. 集合支持加法运算，相当于NSMutableArray的addObjectsFromArray
array1 += [2, 4, 6, 8, 10]    // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]

// 6. 使用let声明的数组不可变，不能修改数组array3
let array5: [Int] = [1, 3, 5, 7, 9]
//array5.append(2)              // 报编译错误

// 7. 集合使用下标索引，支持区间索引，区间不可越界
var array6: [Int] = [1, 3, 5, 7, 9]
array6[1] = 4                       // [1, 3, 5, 7, 9]
array6[1...3] = [2, 3, 4]           // [1, 2, 3, 4, 9]
array6[0...2] = array6[1...3]       // [2, 3, 4, 4, 9]

// 8. 迭代数组的时候，如果需要索引，可以用enumerate方法
for (index, value) in array4.enumerate() {
    //do something
}
```

### 2. 字典Dictionary
与数组类型一样，字典也支持泛型，其键值类型都可以指定或有编译器识别，其中Key的类型，必须是可Hash的，swift中基础数据类型都是可hash的（String、Int、Double和Bool）
```swift
// 1. 用法与oc类似，初始化不需要@
var dict1 = ["key1": 1, "key2": 2, "key3": 3]

// 2. 声明方式
var dict2: Dictionary<String, Int> = dict1        //dict2与dict1不是一个对象
var dict3: [String: Int] = dict1


// 3. 不声明类型，编译器又无法识别，则为NSDictionary
var dict4 = [:]

// 4. 修改dict
dict1["key3"] = 4

// 5. key不存在不报错，返回可空类型nil
let value4 = dict1["key4"]

// 6. 字典迭代返回key/value元组
for (key, value) in dict1 {
    print("\(key) = \(value)")
}
```

> 数组（Array）或字典（Dictionary），如果声明为变量（var），则为可变，如果为常量（let），则为不可变
> 常量数组或字典编译器会对其进行优化，所以尽量把不可变的数组定义为常量数组

### 3. 元组Tuple
与python类似，swift也支持元组，可以很方便的使用元组包装多个值，也使得函数返回多个值变得更加方便，特别是临时组建值得时候
* 支持任意类型
* 支持同时赋值
* 支持自定义key，支持索引
* **元组不是对象，不是`AnyObject`类型，由于swift是强类型的，所以元组有时不能当做普通的对象使用，例如不能把元组加到数组里面，元组内的所有类型必须是明确的**

```swift
// 1. 声明一个元组，元组支持任意类型
let httpError1 = (404, "Not Found")
let point = (100, 50)

// 2. 可以分别赋值
let (x, y) = point
print(x)      // 100
print(y)      // 50

// 3. 取元组元素
print(httpError1.0)      // 404
print(httpError1.1)      // Not Found

// 4. 可以给数组元素取名
let httpError2 = (code: 404, errorMessage: "Not Found")
print(httpError2.code)               // 404
print(httpError2.errorMessage)       // Not Found

// 5. 可以用下划线表示忽略部分值
let (a, _) = point
```

### 4. 字符串String
swift字符串是由Character字符组成的集合，支持`+`操作符，可以与NSString无缝桥接
字符串是值类型（与Int, Float）一样，在传值的时候都会进行拷贝，当然这回带来一定的性能损耗，*swift编译器在编译的时候会进行优化，保证只在必要的情况下才进行拷贝*
```swift
// 1. 与NSString不同，声明不需要@前缀
let name1 = "bomo\n"

// 2. 空串（下面两种方式等价）
let name2 = ""
let name3 = String()

// 3. 字符串由字符Character组成，定义字符
let character1: Character = "!"

// 4. 常见属性，方法
name1.isEmpty                   // 判空
name1.characters.count          // 字符数
name1.uppercaseString
name1.lowercaseString
name1.hasPrefix("bo")
name1.hasSuffix("mo")


// 5. 加法运算
let hello = "hello " + name1   // hello bomo\n

// 6. 比较（比较值，而不是地址）
let name4 = "b" + "omo\n"
name4 == name1                 // True

// 7. 字符串插值
let city = "广州"
let hello2 = "I'm \(name1) from \(city)"

// 8. 格式化字符串
let f = 123.3233
var s = String(format: "%.2f", f)     //123.32
```


### 5. 集合的赋值和拷贝行为
swift的集合通常有Array和Dictionary，他们在赋值或传递的时候，行为上有所不同，字典类型Dictionary或数组类型Array在赋值给变量或常量的时候，只要有做修改，就会进行值拷贝，并且不会作用到原来变量上
```swift
var dict1 = ["a": 1, "b": 2]
var dict2 = dict1
print(dict1 == dict2)         // true
dict2["a"] = 3                // 修改dict2
print(dict1 == dict2)         // false


var arr1 = ["a", "b"]
var arr2 = arr1
print(arr1 == arr2)           // true
arr1[0] = "c"                 // 修改arr1
// arr1.append("c")
print(arr1 == arr2)           // false
```
当数组或字典作为参数传递给函数的时候，由于在Swift3中不推荐使用变量参数，故所有函数参数不可变，故也不进行拷贝

## 五、可选类型（可空类型）
swift加入了可空类型让我们使用数据的时候更为安全，我们需要在可空的地方使用可选类型声明该变量可为空，不能给非可选类型设值`nil`值

### 1. 声明
```swift
// 1. 声明可选类型，在类型后面加上?
var obj1: NSObject?
obj1 = NSObject()
obj1 = nil

// 2. 不能给一个可选类型赋nil，下面会报错，
var obj = NSObject()
obj = nil

// 3. 如果声明可选变量时没有赋值，则默认为nil
var i: Int?

// 4. 一个函数返回一个可选类型
func getdog() -> String? {
    return "wangcai"
}

// 5. 假定我们通过getdog方法返回的值一定不为空
var dog: String? = getdog()

// 6. 不能把可选类型赋值给非可选类型，下面会报错
let cat: String = dog
```

### 2. 强制解析
可选类型不能直接使用，需要通过取值操作符`!`取得变量的值，才能使用，如果变量有值，则返回该值，如果变量为空，则会运行时错误
```swift
var b: Int?
var a: Int
a = 12
b = 13
let c = a + b!              // 先对b取值，再运算

var b: Bool? = nil
if b! {                     // b为空，编译不报错，运行时报错
    print("true")
} else {
    print("false")
}
```

### 3. 可选绑定
使用可选绑定可以判断一个可选类型是否有值，如果有值，则绑定到变量上，如果没有值，返回false，使用`if-let`组合实现
```swift
var i: Int? = nil
if let number = i {
    print("\(number)")
} else {
    print("nil")
}
```
可选绑定还支持绑定条件
```swift
var i: Int? = nil
if let number = i where i > 10 {
    print("i不为空且大于10 \(number)")
} else {
    print("nil")
}
```

### 4. 隐式解析
声明类型的时候可以使用隐式解析，即在使用可选变量的时候自动取值，不需要调用`!`操作符，
```swift
// 一个函数返回一个可选类型
func getdog() -> String? {
    return "wangcai"
}

//假定我们通过getdog方法返回的值一定不为空
var dog: String? = getdog()
let cat: String = dog!          // 使用前需要通过!强制取值
```
使用dog的时候都需要取值我们觉得太麻烦了，可以声明成隐式可选类型，使用的时候自动取值
```swift
var dog: String! = getdog()     // 实际上dog还是可选类型，只是使用的时候回自动取值
let cat: String = dog           // 在使用dog的时候会自动进行取值，不需要取值操作符
```

### 5. 可选类型自判断链接
在使用可选类型之前，需要进行判断其是否有值，才能使用，通过`!`操作符取值后使用（保证有值的情况下），或通过`if-let`可选绑定的方式，swift提供了一种类似C#语言的语法糖可以让代码更为简洁，可以自动判断值，如果有值，则操作，无值则不操作，并返回nil，在使用前加上`?`
```swift
class Person {
    var favDog: Dog?
}
class Dog {
    var name: String?
}

var p = Person()
var d = Dog()
// p.favDog = d
p.favDog?.name = "tobi"   // 如果p.favDog为空，不设置name

if let name = p.favDog?.name {
    // p.favDog不为空且p.favDog.name不为空
} else {
    // p.favDog为空或p.favDog.name为空
}
```
自判断链接还支持多连接如
```swift
let identifier = john.residence?.address?.buildingIdentifier
```

### 6. 可选关联运算符
可选关联运算符可对可选类型进行拆包，如果可选类型对象为nil，返回第二个操作数，第二个操作数类型必须和第一个操作数同类型（可选或不可选）

```swift
let defaultColorName = "red"
var userDefinedColorName: String?   // defaults to nil

var colorNameToUse = userDefinedColorName ?? defaultColorName
```
* defaultColorName和userDefinedColorName必须是同类型（String或String?）
* 如果userDefinedColorName不为空，返回其值，如果userDefinedColorName为空，返回defaultColorName
* 返回值colorNameToUse的类型同`??`的第二个操作数的类型，为`String`




## 六、运算符
swift运算符在原有的基础上做了一些改进，还添加了一下更高级的用法，还有新的运算符
* `=`运算符不返回值
* 字符串String，字符Character支持`+`运算符
* 浮点数支持`%`求余运算
  ```swift
  8 % 2.5 // 等于 0.5
  ```
* `++/--`运算在swift3被抛弃，用`+=/-=`代替
* 支持溢出运算符（`&+`, `&-`, `&*`），可以在溢出时进行(高位)截断
* 支持位运算符（`>>`, `<<`）
* 与其他高级语言类似，swift运算符支持重载，可以为类添加自定义的运算符逻辑，后面会讲到
* `!=`, `==`, `===`, `!==`（恒等于/不恒等于）
      `===`：这两个操作符用于引用类型，用于判断两个对象是否指向同一地址
      `!===`：与`===`相反，表示两个变量/常量指向的的地址不同
      `==`：表示两个对象逻辑相等，可以通过重载运算符实现相等的逻辑，两个值相等的对象可以是不同地址的对象
      `!=`：与`==`相反，表示两个对象逻辑不等
* 范围运算符
    可以使用`a...b`表示一个范围，有点类似于Python的`range(a, b)`
    ```swift
    for i in 1...5 {
        print(i)          // 1, 2, 3, 4, 5
    }
    ```
    `a...b`: 从a到b并包含a和b
    `a..<b`: 包含a不包含b

    > `a..b`表示半闭区间的用法已经被放弃

    范围运算符也可以作用于字符串
    ```swift
    let az = "a"..."z"      // 返回的是CloseInteval或HalfOpenInterval
    az.contains("e")        // True
    ```
* 可选关联运算符`??`
    对于可选类型取值，如果不为空则返回该值，如果为空则去第二个操作数

## 七、流程控制
swift使用三种语句控制流程：`for-in`、`for`、`switch-case`、`while`和`do-while`，且判断条件的括号可以省略
```swift
if a > b {
    print(a)
} else {
    print(b)
}
```
流程控制语句的条件返回值必须是Bool，下面会报错
```swift
var dd: Bool? = true
if dd {
    print("fd")
}
```
条件判断可以与`let`结合使用，当值为nil时，视为false（即：`可选绑定`）
```swift
var dd: Bool? = true
if let ee = dd {
    print("fd")
}
```

### guard-else
guard翻译为保镖模式，在执行操作前，进行检查，如果不符合，过滤掉，使用方式与if有些类似，如果与let结合使用，可以对可选类型解包，先看看普通的`if-else`模式
```swift
func test(i: Int?) {
    if let i = i where i > 0 {
        // 符合条件的处理
        return
    }

    // 不符合条件的处理
}
```

上面的处理把条件放在了条件判断内部，使用guard与之相反，把正确的情况放在最外部，而异常情况放在条件判断内部
```swift
func test(i: Int?) {
    guard let i = i where i > 0 else {
        // 在这里拦截，处理不符合条件的情况
        return
    }

    // 符合条件的处理，这个时候已经对i进行了拆包，i是非可选类型，可以直接使用
    print(i)
}
```

### switch-case
* switch语句支持更多数据类型（String，Int, Float, 元组, 枚举），理论上switch支持任意类型的对象（需要实现`~=`方法或`Equatable`协议，详情参见[这里](http://www.jianshu.com/p/ff660a3e3d8a)）
* case可以带多个值，用逗号隔开
* case可以支持区间（`a...b`），支持元组，区间可以嵌套在元组内使用
* case多条语句不需要用大括号包起来
* case语句不需要break，除了空语句，如果需要执行下面的case，可以使用`fallthrough`
* 如果case不能命中所有的情况，必须要default，如Int，String类型
* 可以用`fallthrough`关键字声明接着执行下一条case语句，注意，如果case语句有赋值语句（`let`），则`fallthrough`无效

```swift
// 定义一个枚举
enum HttpStatus {
    case ServerError
    case NetworkError
    case Success
    case Redirect
}

var status = HttpStatus.Redirect
switch status {
// case可以接收多个值
case HttpStatus.ServerError, HttpStatus.NetworkError:
    print("error")
    // case语句结束显式写break，除非是空语句

case .Redirect:             // 如果编译器可以识别出枚举类型，可以省略枚举名
    print ("redirect")
fallthrough             //继续执行下一条case
case HttpStatus.Success:
    print("success")
}

//元组，区间
let request = (0, "https://baidu.com")
switch request {
case (0, let a):                  // 支持绑定
    a
case let (a, b) where a == 1:      // 绑定可以卸载元组外面，支持where判断
    "cancel \(b)"
case (2...10, _):                 // 支持区间，支持忽略值
    "error"
default:
    "unknown"
}

// case可以与where进行进一步判断
let request2 = (0, 10)
switch request2 {
case (0, let y) where y < 5:
"success"   //被输出
case (0, let y) where y >= 5:
"error"   //被输出
default:
    "unknown"
}
```

## 八、函数
### 1. 基本形式
```swift
//有返回值
func 函数名(参数名1:参数类型1, 参数名2:参数类型2) -> 返回值类型 {
    // 函数体
}

//多个返回值（元组）
func getPoint() -> (x: Int, y: Int) {
    return (1, 3)
}
var p = getPoint()
p.x

//无参数无返回值
func sayHello() {
    // 函数体
}

//egg
func add(a: Int, b: Int) -> Int {
    return a + b
}

// 调用
add(12, b: 232)
```
函数调用除了第一个参数，后面所有的参数必须带上参数名，符合Objc的函数命名规则，如果是调用构造器，第一个参数也需要显示声明
```swift
class A {
    var name: String
    init(name: String) {
        self.name = name
    }

    func sayHello(msg: String, count: Int) {
        for _ in 1...count {
            print (msg)
        }
    }
}

let a = A(name: "bomo")               // 构造器所有参数都必须显示声明参数名
a.sayHello("hello", count: 2)         // 函数参数除了第一个其他都需要显示声明参数名
```

### 2. 可变参数
可变参数只能作为最后一个参数，一个方法最多只有一个可变参数
```swift
func sum(numbers: Int...) -> Int {
    var sum = 0
    for number in numbers {
        sum += number
    }
    return sum
}
```

### 3. 外部参数名
默认情况下，如果不指定外部参数名，swift编译器会自动为函数参数声明与内部参数名同名的外部参数名（格式为：`外部参数名 内部参数名: 类型名`）
```swift
//默认情况下，外部参数名与内部参数名一样
func add(first a: Int, second b: Int) -> Int {
    return a + b
}

// 调用
add(first: 10, second: 20)
```
如果函数在第一个参数定义外部参数名，必须显示指定，当然我们还可以通过`_`符号让函数忽略参数名
```swift
func add(a: Int, _ b: Int) -> Int {
    return a + b
}
add(1, 2)
```

### 4. 函数默认值
函数还支持声明默认值，（格式为：`外部参数名 内部参数名: 类型名 = 默认值`）
```swift
func log(msg: String, isDebug: Bool = true) {
    if isDebug {
        print(msg)
    }
}
log("fail")
log("success", isDebug: false)
```
如果使用默认值并且默认值不是出现在最后，那调用的时候必须写全所有参数
> 建议把默认参数放到最后面，这样可以确保非默认参数的赋值顺序，减少参数混乱的情况


### 5. 闭包
* 函数作为变量
* 函数作为函数参数
* 函数作为函数返回值
* 闭包函数声明
```swift
func add(a: Int, b: Int) -> Int {
    return a + b
}

//函数作为变量，函数hello赋给somefunc，并调用
let somefunc: (Int, Int) -> Int = add
somefunc(10, 20)      // 30

//函数作为参数
func logAdd(a:Int, b:Int, function: (Int, Int) -> Int) {
    // 函数内容
    print("begin")
    function(a, b)
    print("end")
}
logAdd(12, b: 23, function: add)

//函数作为返回值（包装一个函数，在执行前后输出信息），函数作为参数又作为返回值
func addWrapper(addFunc: (Int, Int) -> Int) -> ((Int, Int) -> Int) {
    // 函数内容
    func wrapper(a: Int, b: Int) -> Int {
        print("begin")
        let res = addFunc(a, b)
        print("end")
        return res
    }
    return wrapper
}
var newAdd = addWrapper(add)
newAdd(12, 32)
```

闭包函数声明形式
```swift
{ (parameters) -> returnType in
    statements      // 可以有多行
}
```
‘’
闭包函数
```swift
//定义一个函数变量
var addfunc: (Int, Int) -> Int

//闭包的写法
// 1. 完整写法
addfunc = {(a: Int, b: Int) -> (Int) in
    //var c = a + 1       //函数体可以有多条语句，如果在同一行，需要用分号隔开，函数体不需要大括号
    return a + b
}
// 2. 前面的addfunc变量可以推断出后面函数的参数类型和返回值类型，故可以省略
addfunc = {(a, b) in return a + b}

// 3. 参数列表括号可以省去，函数只有一条语句时，return可以省略
addfunc = {a, b in a + b}

// 4. 参数和in可以省去，通过$和索引取得参数
addfunc = {$0 + $1}

// 操作符需要的参数与函数参数一致，可以省去参数，并使用括号括起来，作为参数时，可不用括号
addfunc = (+)
```

### 6. Trailing(尾行)闭包
如果函数作为另一个函数的参数，并且是最后一个参数时，可以通过Trainling闭包来增强函数的可读性
```swift
func someFunctionThatTakesAClosure(a: Int, closure: () -> ()) {
    // 函数体部分
}

// 1. 一般形式
someFunctionThatTakesAClosure(10, closure: {
    // 闭包主体部分
})

// 2. Trainling闭包的方式
someFunctionThatTakesAClosure(10) {
    // 闭包主体部分
}

// 3. 如果没有其他参数时，可以省略括号
someFunctionThatTakesAClosure {
    // 闭包主体部分
}
```

### 7. 常量参数和变量参数
默认情况下所有函数参数都是常量，意味着参数是不可变的，我们可以显式的声明参数为变量
```swift
func log(msg: String) {
    msg = "begin " + msg + " end"       // 会报错，因为msg为常量
    print(msg)
}
func log(var msg: String) {
    msg = "begin " + msg + " end"       // 变量参数正常运行
    print(msg)
}
```
> 注：变量参数在swift3被抛弃

### 8. 输入输出参数
在c语言里有指针，可以通过传址直接修改外部变量的值，在swift通过`inout`关键字声明函数内部可直接修改外部变量，外部通过`&`操作符取得变量地址
```swift
func swap(inout a: Int, inout b: Int) {
    let temp = a
    a = b
    b = temp
}
var a = 19, b = 3
swap(&a, &b)
```

## 九、枚举
swift的枚举比C语言的枚举更为强大，支持更多特性，swift的枚举更像类和结构体，支持类和结构体的一些特性，与`ObjC`不同，如果不声明枚举的值，编译器不会给枚举设置默认值
### 1. 声明和使用
```swift
// 1. 定义枚举
enum CompassPoint {
    case North
    case South
    case East
    case West
}
// 2. 可以把枚举值定义在一行，用逗号隔开
enum CompassPoint2 {
    case North, South, East, West
}

// 3. 像对象一样使用枚举，代码结构更为清晰，枚举更为简短
let direction = CompassPoint.East

// 4. 如果编译器可以识别出枚举的类型，可以省略枚举名
let direction2: CompassPoint
direction2 = .East

// 5. 如果编译器能确定case命中所有的情况，可以不需要default
switch direction {
case .East:
    print("east")
case .West:
    print("west")
case .South:
    print("south")
case .North:
    print("north")
    //所有值都被枚举，则不需要default
}
```

### 2. 嵌套枚举
swift的枚举定义支持嵌套，在使用的时候一层一层引用
```swift
enum Character {
    enum Weapon {
        case Bow
        case Sword
        case Lance
        case Dagger
    }
    enum Helmet {
        case Wooden
        case Iron
        case Diamond
    }
    case Thief
    case Warrior
    case Knight
}

let character = Character.Thief
let weapon = Character.Weapon.Bow
let helmet = Character.Helmet.Iron
```


### 3. 原始值
与C语言一样，可以为每个枚举指定值，并且可以支持更多类型（`Int`, `Float`, `Character`, `String`）
```swift
// 定义枚举，并初始化原始值
enum ASCIIControlCharacter: Character {
    case Tab = "\t"
    case LineFeed = "\n"
    case CarriageReturn = "\r"
}

// 2. 通过两个属性获得原始值
var ch = ASCIIControlCharacter.Tab
ch.hashValue    // 获取是否有原始值
ch.rawValue     // 获得原始值

// 3. 通过原始值构造枚举，如果不存在，则返回nil
var tab = ASCIIControlCharacter.init(rawValue: "\t")

// 4. 如果是原始值是整形值，后面的值默认自增1
enum Planet: Int {
    case Mercury = 1, Venus         // Venus = 2
    case Neptune                    // Neptune = 3
}
```

### 4. 关联值
上面我们说到，枚举与类和结构体类似，swift的枚举可以给不同的枚举值绑定关联值，如下
```swift
enum Barcode {
    case UPCA(Int, Int, Int)        //条形码，关联一个元组
    case QRCode(String)             //二维码，关联一个字符串
}

var productBarcode = Barcode.UPCA(8, 85909_51226, 3)
// var productBarcode = .QRCode("http://www.baidu.com")

switch productBarcode {
case .UPCA(let a, let b, let c):        //在枚举的时候可以取得关联值
    print("barcode: \(a)\(b)\(c)")
case let .QRCode(value):
    print("qrcode: \(value)")
}
```
如上面这种轻量的数据，一般来说我们可能需要定义两个类实现，而swift的枚举可以轻松的处理这种轻量数据，而减少项目中类的定义和维护


## 十、类与结构体
先来看看结构体和类的一些差异
* 类是引用类型，结构体为值类型
* 类使用引用计数管理内存，结构体分配在栈上，有系统管理内存，变量传递的时候，结构体整个拷贝，而类默认只传递引用地址（有些类会进行一些额外的拷贝，详见[深拷贝和浅拷贝]()）
* 结构体不支持继承，类支持继承
* 结构体是值类型
* 与ObjC不同，swift的结构体可以定义方法

### 1. 结构体，类定义
```swift
struct Point {
    let x: Int
    let y: Int

    func printPoint() {
        print("x=\(x), y=\(y)")
    }
}

class Person {
    var someObj = NSObject()          // 定义属性，并初始化
    var name: String                  // 定义属性，并指定类型

    init(name: String) {              // 构造函数
        self.name = name
    }

    func hello() {
        print("hello \(self.name)")
    }

    //析构函数
    deinit {
        print("dealloc")
    }
}
```

### 2. 静态属性，静态方法
swift中有两个`static`和`class`声明静态变量或方法，其中`class`只能用在类的方法和计算属性上，其他的都使用`static`，由于类支持继承，所以使用`class`声明的静态方法可以被继承，而static声明的静态方法不能被继承
```swift
class Person {
    static var instanceCount: Int = 0       // 声明一个类属性
    init () {
        Person.instanceCount += 1           // 通过类名引用类属性，子类可以访问基类的类属性
    }

    // 使用class声明的静态方法可以被继承
    class func overrideableComputedTypeProperty() {
        print("\(Person.instanceCount)")
    }

    // 使用static声明的静态方法不能被继承
    static func printInstanceCount() {      // 声明一个静态方法
        print("\(Person.instanceCount)")
    }
}
```
类和结构体的声明和用法与类类似，使用`static`
> 注意：`class`只能用来声明计算属性和方法，不能用来声明普通属性

### 3. 构造器和析构器
swift的构造器规则和限制比较多，关于构造器可以参见：[这里](/2016-07-07/swift-constructor/)

析构器相当于objc里面的`dealloc`方法，做一些需要手动释放资源的操作，析构器与构造器不同，没有参数，定义的时候不需要括号，类在释放的之前会自动调用父类的析构器，不需要主动调用
```swift
class Person {
    deinit {
        print("释放额外的资源，如通知")
    }
}
```

### 4. 类型判断
在objc中，我们通常使用`isKindOfClass`, `isMemberOfClass`, `isSubclassOfClass`等方法进行类型判断，swift使用`is`和`as`判断类型
```swift
class Parent {
}
class Son: Parent {
}

var s = Son()
// isKindOfClass
son is Son                // true
son is Parent             // true

// isMemberOfClass
son.dynamicType == Son.self         // true
son.dynamicType == Parent.self      // false

// isSubclassOfClass 暂时没找到相关的API
```
//TODO: swift动态性，反射

### 5. 弱引用
与`ObjC`一样，swift的内存管理也使用引用计数管理，也使用weak声明弱引用变量
```swift
class Person {
    weak var person: Person? = nil
}
```

### 6. 访问级别
在swift中，framework和bundle都被处理成模块

    * public：公开，可以被外部访问
    * internal：内部，在模块（framework）内部使用，模块外访问不到
    * private：只能在当前源文件中使用

swift默认的访问级别为Internal，使用的时候只需要在类/变量/函数前面加上访问级别即可
```swift
public class Person {
    class public var peopleCount: Int = 0    // 类变量，通过class声明，类变量使用时使用类名引用
    internal var age: Int                    // 实例变量
    var name: String                         // 不声明，则为internal

    init() {
        self.age = 0
        self.name = ""
        Person.peopleCount++              // 使用静态变量
    }

    private func sayHello() {
        print("hello")
    }
}
```
外层访问级别的必须是比成员更高，下面会报警告
```swift
class Person {                      // 默认为internal
    public var age: Int = 0         // 为public，比类访问级别高，会有警告
    private var gender: Int = 10
    private func sayHello() {
          print("hello")
    }
}
```

函数的访问级别要比参数(或泛型类型)的访问级别低，否则会报警告
```swift
private class PrivatePerson {
    private var age: Int = 0
    var gender: Int = 10          // 报警告

    private func sayHello() {

    }
}

public class Test {
    public func test(person:PrivatePerson) {    //报编译错误：这里参数访问级别为private，所以函数访问级别不能高于private，则只能为private
    }
}
```

`枚举类型`的成员访问级别跟随枚举类型，嵌套类型默认最高访问级别为internal（外层为public，内层默认为internal）
```swift
public enum CompassPoint {
    case North            // 四个枚举成员访问级别都为public
    case South
    case East
    case West
}
```

子类访问级别不能高于父类（包括泛型类型），协议继承也同理，子协议访问级别不能高于父协议
```swift
class Parent {

}

public class Son: Parent {       // 报编译错误：Son访问级别必须低于Parent，应该为internal或private

}
```

`元组`的访问级别为元组内所有类型访问级别中最低级的
```swift
class Parent {
}

private class Son: Parent {
}

public class SomeClass {
    internal let sometuple = (Son(), Parent())  // 报编译错误：sometuple的访问级别不能高于成员类型的访问级别，由于Son为private，故sometuple必须为private
}
```

变量的访问级别不能高于类型
```swift
private class PrivateClass {

}

public class SomeClass {
      public var value: PrivateClass        // 报编译错误：变量value的访问级别不能高于其类型，故value必须声明为private
}
```

属性的 Setter 访问级别不能高于 Getter访问级别
```swift
public class SomeClass {
    private(set) var num = 1_000_000      // 声明属性num，getter访问级别没有声明，默认为Internal，setter访问级别为private

    private internal(set) var name = "bomo"   // 报编译错误：属性name的setter访问级别为internal，高于getter访问级别private
}
```

协议与类的访问级别关系
* 协议中所有必须实现的成员的访问级别和协议本身的访问级别相同
* 其子协议的访问级别不高于父协议（与类相同）
* 如果类实现了协议，那类的访问级别必须低于或等于协议的访问级别

类型别名访问级别与类型的关系
* 类型别名的访问级别不能高于原类型的访问级别；


函数构造函数默认访问级别为internal，如果需要给其他模块使用，需显式声明为public


> 注意：swift的访问级别是作用于文件（private）和模块的（internal）的，而不只是类，所以只要在同一个文件内，private访问级别在不同类也可以直接访问，例如我们可以通过子类包装父类的方法以改变访问级别
```swift
public class A {
    private func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {   // 在同一个文件，改变someMethod的访问级别
        super.someMethod()
    }
}
```

### 7. 属性
* 使用关键字`lazy`声明一个懒加载属性，当属性被使用的时候（get），才会进行初始化
* set方法的访问级别必须必get方法低
* 声明属性的时候可以使用`private(set)`和`internal(set)`改变set方法默认的访问级别

```swift
class DataImporter {
}

class DataManager {
    // 1. 只有第一次调用importer的get方法的时候才会初始化
    lazy var importer = DataImporter()
    var data = [String]()
}

class Rectangle {
    var width: Double = 0.0
    var height: Double = 0.0

    // 2. 声明get方法和set方法的访问级别
    private private(set) var weight: Double = 0

    // 3. 自定义get/set方法
    var square: Double {
        get {
            return (self.width + self.height)/2;
        }
        //set {                 //如果不指定名称，默认通过newValue使用新值
        set(newValue) {
            self.width = newValue/2.0;
            self.height = newValue/2.0
        }
    }

    // 4. 只读属性，可以省略get
    var perimeter: Double {
        return (self.width + self.height) * 2
    }

    // 5. 属性监视器，在初始化的时候不会触发
    var someInt: Int = 0 {
        willSet {
            print("set方法之前触发")
        }
        didSet {
            print("set方法完成后触发，可以在这里设置obj的值覆盖set方法设置的值")
            self.someInt = 0      // someInt的值永远为0
        }
    }
}
```

### 8. 继承
我们都知道，在oc里所有的类都继承自NSObject/NSProxy，而在swift中的类并不是从一个通用的基类继承的，所有没有继承其他父类的类都称为基类
```swift
class Parent {
    final var gender = "unknown"
    init(gender: String) {
        self.gender = gender
    }
    private func hello() {
        print("parent hello")
    }
}
class Son: Parent {
    // 重写可以改变父类方法的访问级别
    internal override func hello() {                  // 重写父类方法必须加上override，否则会报编译错误
        //super.hello()                               //可以通过super调用父类发方法
        print("son hello")
    }
}
```

父类的属性，方法，类方法，附属脚本，包括类本身都可以被子类继承和重写，可以通过`final`约束限制子类的重写（`final class`, `final var`, `final func`, `final class func`, 以及 `final subscript`）
```swift
class Parent {
    final var gender = "unknown"        // 不允许被子类重写
    var name: String                    // 可以被子类重写
    init(gender: String) {
        self.gender = gender
        self.name = ""
    }
    final func hello() {                // 不允许被重写
        print("parent hello")
    }
}
```

swift编译器在识别数组类型的时候，如果数组元素有相同的基类，会被自动识别出来
```swift
class Person {
}
class Teacher: Person {
}
class Student: Person {
}

let t1 = Teacher()
let t2 = Teacher()
let s1 = Student()
let s2 = Student()

let people = [t1, t2, s1, s2]      // people会被识别为[Person]类型
```

向下类型转换`as!`, `as?`，`as!`返回非可选类型，如果类型不匹配会报错，`as?`返回可选类型，如果类型不匹配返回nil
```swift
for person in people {
    if let teacher = person as? Teacher {
        println("teacher")
    } else if let student = person as? Student {
        println("student")
    }
}
```

### 9. 附属脚本subscript
附属脚本可以让类、结构体、枚举对象快捷访问集合或序列，而不需要调用使用对象内的实例变量引用，看下面实例
```swift
class DailyMeal {
    enum MealTime {
        case Breakfast
        case Lunch
        case Dinner
    }

    var meals: [MealTime : String] = [:]
}

// 如果需要使用DailyMeal的meals对象的，需要这么用
var dailyMeal = DailyMeal()
dailyMeal.meals[MealTime.Breakfast] = "Toast"
```

使用附属脚本可以直接通过类对象索引访问meals的值
```swift
class DailyMeal {
    enum MealTime {
        case Breakfast
        case Lunch
        case Dinner
    }

    var meals: [MealTime : String] = [:]

    // 定义附加脚本，类似属性
    subscript(realMealTime: MealTime) -> String {
        get {
            if let value = meals[realMealTime] {
                return value
            } else {
                return "unknown"
            }
        }
        set(newValue) {
            meals[realMealTime] = newValue
        }
    }
}


var dailyMeal = DailyMeal()
dailyMeal[.Breakfast] = "sala"
print(dailyMeal[.Breakfast])
```

附加脚本类似属性，拥有get/set方法，支持只读和读写两种方式，附加脚本也支持多个参数，附属脚本可以屏蔽外部对内部对象的直接访问，隐藏对象内部的细节，提高封装度，使得代码更加健壮和简洁

### 10. 类型嵌套
与枚举一样，结构体和类都支持类型嵌套，可以在类里面再定义类/结构体/枚举
```swift
class SomeClass {
    // 类里面嵌套定义枚举
    enum Suit: Character {
        case Spades = "♠", Hearts = "♡", Diamonds = "♢", Clubs = "♣"

        // 枚举里面嵌套定义结构体
        struct Values {
            let first: Int, second: Int
        }
    }

    // 类里面嵌套定义结构体
    struct Point {
        let x: Int
        let y: Int
    }

    // 类里面嵌套定义类
    class InnerClass {
        var name: String = ""
        var id: Int = 0
    }
}

// 使用的时候像属性一样引用
let values = SomeClass.Suit.Values(first: 1, second: 2)
```

### 11. 类型别名
swift类型别名与c语言中取别名有点像，通过关键字`typealias`声明别名
```swift
public typealias MyInt = Int

func add(a: MyInt, b: MyInt) -> MyInt {
    return a + b
}
```
在容易出现命名冲突的情况下会考虑使用类型别名

## 十一、扩展Extension
与oc一样，扩展就是对已有的类添加新的功能，与oc的category类似，swift的扩展可以：
* 提供新的构造器（需要符合构造器的基本规则）
* 添加实例计算型属性和类计算性属性
* 添加实例方法和类方法
* 添加附加脚本
* 添加新的嵌套类型
* 使一个已有类型符合某个接口

swift扩展不可以：
* 不可以添加存储属性
* 不可以向已有属性添加属性观测器(willSet, didSet)

```swift
class Person {
    func hello() {
        print("hello")
    }
}

// 定义扩展
extension Person {
    func fly() {
        print("fly")
    }
}

let p = Person()
p.fly()
```

扩展也可以作用在结构体和枚举上
```swift
struct Rectangle {
    let width: Double
    let height: Double
}

extension Rectangle {
    var perimeter: Double {
        return 2 * (self.width + self.height)
    }
}

let rect = Rectangle(width: 100, height: 200)
print(rect.perimeter)
```
扩展内的成员定义与类类似，这里不再说明

### 扩展属性
由于swift不能扩展新的属性，有时候我们希望给类添加属性，在oc里可以用关联属性新增存储属性，在swift也可以，需要引入`ObjectiveC`模块
```swift
import ObjectiveC

class Point {
    var x: Int = 0
    var y: Int = 1
}

private var xoTag: UInt = 0
extension Point {
    var z: Int {
        get {
            return objc_getAssociatedObject(self, &xoTag) as! Int
        } set(newValue) {
            objc_setAssociatedObject(self, &xoTag, newValue, objc_AssociationPolicy.OBJC_ASSOCIATION_ASSIGN)
        }
    }
}
```

## 十二、协议Protocal
swift的协议在oc的基础上加了更多的支持，可以支持属性，方法，附加脚本，操作符等，协议的属性必须为变量`var`
```swift
protocol SomeProtocol {
    // 属性要求
    var mustBeSettable: Int { get set }
    // 只读属性
    var doesNotNeedToBeSettable: Int { get }
    // 只读静态属性
    static var staticProperty: Int { get }
    // 静态方法
    static func hello()
}
```

### 1. mutating
在swift中的结构体和枚举都是值类型，默认情况下不能对其进行修改，编译不通过，如果需要修改值类型的属性，需要在方法声明前加上`mutating`
```swift
struct Point {
    var x: Int
    var y: Int

    func moveToPoint(point: Point) {
        self.x = point.x        // 报错：不能对值类型的属性进行修改
        self.y = point.y        // 报错：不能对值类型的属性进行修改
    }

    mutating func moveToPoint2(point: Point) {
        self.x = point.x        // 编译通过
        self.y = point.y        // 编译通过
    }
}
```
特别是在定义Protocal的时候，需要考虑到协议可能作用于枚举或结构体，在定义协议的时候需要在方法前加上`mutating`
```swift
protocol SomeProtocol {
    mutating func moveToPoint(point: Point)
}
```

### 2. 协议类型
协议虽然没有任何实现，但可以当做类型来用，与oc的protocal类似，用协议类型表示实现了该协议的对象，与oc的`id<SomeProtocol>`一样

### 3. 协议组合
有时候我们需要表示一个对象实现多个协议，可以使用协议组合来表示，如下
```swift
protocol SwimProtocal {
    func fly()
}
protocol WalkProtocal {
    func walk()
}

func through(animal: protocol<WalkProtocal, SwimProtocal>) {
    animal.walk()
    animal.fly()
}
```

### 4. 自身类型
有时候我们需要表示实现协议的类型，可以使用`Self`代替，如下
```swift
protocol CompareProtocal {
    // Self表示实现协议自己的类型本身
    func compare(other: Self) -> Bool
}

class Product: CompareProtocal {
    var id: Int = 0
    func compare(other: Product) -> Bool {
        return self.id == other.id
    }
}
```

### 5. @objc协议
swift声明的协议是不能直接被oc的代码桥接调用的，如果需要，需要在声明前加上`@objc`，使用`@objc`声明的协议不能被用于结构体和枚举
```swift
import Foundation
@objc protocol HasArea {            // 协议可以被桥接到oc中使用
    var area: Double { get }
}
```

### 6. Optional要求
在oc中的protocal可以定义可选方法，在swift默认不支持可选方法，swift只有在添加了`@objc`声明的协议才能定义可选方法，在定义前添加`optional`声明
```swift
import Foundation
@objc protocol HasArea {
    optional var area: Double { get }     // 定义可选属性
}
```


## 十三、泛型
关于泛型的介绍，这里不进行说明，swift的泛型是我认为最酷的特性之一，当然其他语言也有，可以让类或函数更大程度的重用，swift的泛型与其他语言的泛型有点类似
### 1. 定义
在类或函数声明的时候，指定一个泛型类型参数（通常为T）然后使用的时候直接把T当成类型使用
```swift
//泛型函数定义
func swapTwoValues<T>(inout a: T, inout b: T) {
    let temporaryA = a
    a = b
    b = temporaryA
}

//泛型类定义
class Result<T> {
    var code: Int = 0
    var errorMessage: String?
    var data: T?
}

//多个泛型类型参数
class Result<T, TK> {
    var code: Int = 0
    var errorMessage: String?
    var data: T?
    var subData: TK?
}
```
### 2. 泛型约束
我们还可以对泛型进行约束，泛型类型参数只能是某些类型的子类，或实现了某些协议
```swift
func findIndex<T>(array: [T], valueToFind: T) -> Int? {
    for (index, value) in array.enumerate() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```
上面函数会报编译错误，因为在swift里，并不是所有的类都能用`==`操作符比较，只有实现了Equatable协议的类才能用`==`操作符，修改为
```swift
func findIndex<T: Equatable>(array: [T], valueToFind: T) -> Int? {
    for (index, value) in array.enumerate() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

### 3. 多泛型类型参数
有时候我们需要用多个协议进行约束，可以使用下面方式（类与函数的使用方式类似）
```swift
func someFunc<T : protocol<StudyProtocal, RunProtocal>>(arg: T) {
    // do stuff
}
```

如果约束既有类又有协议的话可以使用`where`添加限制条件
```swift
func someFunc<T, TK where T:Student, T: StudyProtocal>(t: T, tk: TK) {
    // do stuff
}
```

### 4. 泛型是不可变的
```swift
var dog1 = SomeClass<Parent>()
var dog2 = SomeClass<Son>()

dog1 = dog2       // 报错
```
> 关于可变，不可变，逆变，协变参考这里：[http://swift.gg/2015/12/24/friday-qa-2015-11-20-covariance-and-contravariance/](http://swift.gg/2015/12/24/friday-qa-2015-11-20-covariance-and-contravariance/)

### 5. 泛型协议
swift的协议不支持泛型，不能像类一样定义泛型，而是通过类型参数定义泛型
```swift
protocol GenericProtocol {
    associatedtype T1
    associatedtype T2
    func someFunc(t2: T2) -> T1
}

class SomeClass<T> : GenericProtocol {
    // 设置泛型类型
    typealias T1 = String
    typealias T2 = T

    func someFunc(t2: T2) -> T1 {
        return ""
    }
}
```

## 十四、运算符重载
与其他高级语言的一样，swift也提供了运算符重载的功能，我们可以自定义运算符的实现，运算符通常分为三种类型
* 单目运算符：`<运算符><操作数>`或`<操作数><运算符>`，如`!a`
* 双目运算符：`<操作数><运算符><操作数>`，如：`1 + 1`
* 三元运算符：`<操作数><运算符><操作数><运算符><操作数>`，如：`a ? b : c`

swift的运算符重载
* 支持自定义运算符`/`, `=`, `-`, `+`, `*`, `%`, `<`, `>`, `!`, `&`, `|`, `^`, `.`, `~`的任意组合。可以脑洞大开创造颜文字。
* 不能对默认的赋值运算符`=`进行重载。组合赋值运算符可以被重载，如`==`，`!==!`
* 无法对三元运算符`a ? b : c`进行重载
* 运算符声明和定义只能定义在全局作用域，不能定义在类/结构体/枚举内
*

### 1. 前缀，中缀，后缀运算符
* 前缀`prefix`：默认的有-，!，~等
* 中缀`infix`：默认的有+，*，==等
* 后缀`postfix`：默认的有：++，--等


### 1.1 声明运算符
如果实现不存在的运算符需要添加运算符声明（系统的提供的，可以不需要声明），声明必须放在全局作用域
```swift
// 前缀运算符
prefix operator +++ {}

// 中缀运算符（二元运算符）
infix operator +++ {}

// 后缀运算符
postfix operator +++ {}
```

### 1.2 实现上面三个运算符
```swift
// 定义Point结构体
struct Point {
    var x: Int
    var y: Int
}

// 重载操作符要放在全局作用域
func +++ (left: Point, right: Point) -> Point {
    return Point(x: left.x + right.x, y: left.y + right.y)
}

// 如果需要修改操作数，需要添加inout关键字
prefix func +++ (inout left: Point) {
    left.x += 1
    left.y += 1
}

postfix func --- (right: Point) -> Point {
    return Point(x: right.x - 1, y: right.y - 1)
}
```

### 1.3 使用
```swift
var p1 = Point(x: 12, y: 21)
var p2 = Point(x: 12, y: 2)

let p3 = p1+++p2            // p3.x = 24, p3.y = 23
+++p1                       // p1.x = 13, p1.y = 3
p1---                       // p1.x = 12, p1.y = 2
```

### 2. 优先级
这个很好理解，就是优先级高的运算符先执行，声明运算符的时候可以指明优先级
```swift
infix operator ^ {
    associativity left        // 结合性，后面说
    precedence 140            // 指定运算符优先级
}
```
[这里](https://developer.apple.com/reference/swift/1851035-swift_standard_library_operators)可以查看默认运算符的优先级

### 3. 结合性
运算符还可以定义结合性，对于双目运算符，当优先级一样的时候，可以定义运算符优先进行左结合还是右结合，运算符的结合性有下面三种
* left：左结合
* right：右结合
* none：无

结合性设置为`left`
```swift
// 定义一个双目操作符
infix operator ^ {
    associativity left         // 结合性
    precedence 140             // 指定运算符优先级
}

func ^ (left: Int, right: Int) -> Int {
    return Int(pow(Double(left), Double(right)))
}

let a = 2 ^ 2 ^ 2 ^ 2           // 执行结果为256
// 相当于
let aa = ((2 ^ 2) ^ 2) ^ 2
```

如果我们设置结合性为`right`
```swift
// 定义一个双目操作符
infix operator ^ {
    associativity right         // 结合性
    precedence 140              // 指定运算符优先级
}

func ^ (left: Int, right: Int) -> Int {
    return Int(pow(Double(left), Double(right)))
}

let a = 2 ^ 2 ^ 2 ^ 2           // 执行结果为65536
// 相当于
let aa = 2 ^ (2 ^ (2 ^ 2))
```
如果结合性设置为`none`，则会报错，无法判断

## 十五、命名空间
在很多语言里面，都有命名空间的概念，可以分离代码，防止命名冲突，而swift也有类似命名空间的概念，通过访问级别实现命名空间
//TODO

## 十六、参考链接
* [运算符结合性](https://en.wikipedia.org/wiki/Operator_associativity#Right-associativity_of_assignment_operators)
* [Swift高级运算符](https://developer.apple.com/library/prerelease/content/documentation/Swift/Conceptual/Swift_Programming_Language/AdvancedOperators.html)
