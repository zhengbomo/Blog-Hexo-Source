---
title: block-start
date: 2016-05-26 00:22:13
updated: 2016-05-26 00:22:13
categories:
tags:
---

# block
> 1. 什么是block
> 2. block基本用法
> 2. block有几种类型
> 5. block与copy
> 6. block循环引用问题


## 什么是block
block是跟随GCD被引入的一个特性，用于表示线程队列中的一个任务，相当于C语言中的函数指针，Java的匿名函数，C#的lambda表达式，block是Object-C的闭包实现

同Class一样，block也是对象，器结构体定义为




### 闭包
objc的block还实现了闭包的功能  
> 闭包：可以在函数内访问局部自由变量

当声明或定义一个block的时候，
* 创建的闭包会捕获（retain）在他的域以外的变量，并在内存持有他们，并在block执行完成后release该变量，这样才能保证，在block执行过程中，变量不会因为引用计数为0而被释放：解释block循环引用问题
* 并且捕获的变量不允许修改：如上addtional不允许在block内部修改
*  

## block的基本用法
语法形式
```objc
return_type (^block_name)(parameters)
```
示例
```objc
//声明一个block
void (^myBlock)();
int (^myBlock)(int,int);

//block作为临时变量
int addtional = 5;
int (^addBlock)(int a,int b) = ^(int a, int b){
    return a + b + addtional;
}

//block作为函数参数
- (void)func:(int (^)(int a, int b))block;

//通过typedef把block定义成类型
typedef int (^myBlock)(int a, int b);

- (void)func:(myBlock)block;
myBlock addBlock = ^(int a, int b){
    return a + b + addtional;
};
```

### block的实现
block的本质也是对象，所以block也拥有一般对象的一些特性，下面是block类的一些定义
[http://www.jianshu.com/p/abeb5848b57a](http://www.jianshu.com/p/abeb5848b57a)

## block类型

* `__NSGlobalBlock__`：当一个block没有访问任何外部变量的时候，这时候block会被当做全局变量保存在全局内存中，相当于单例的存在，使用retain、copy、release操作都是无效的（编译器优化）
```objc
void (^globalBlock) () = ^ () {
  NSLog(@"global block");
};
```

* `__NSStackBlock__`：栈block，在block内部引用外部变量，Block是默认建立在栈上, 所以如果离开方法作用域, Block就会被丢弃（释放），系统管理内存，所以retain和release都无效
```objc
int base = 100;
long (^block) (int, int) = ^ long (int a, int b) {
    return base + a + b;
};
```
> 上面代码在MRC上，block类型为`__NSStackBlock__`，在ARC上，则为`__NSMallocBlock__`，因为在ARC上，block赋值给一个strong变量的时候，会自动的copy

在ARC下, 以下几种情况, Block会自动被从stack复制到heap：
1. 被执行copy方法
2. 作为方法返回值
3. 将Block赋值给附有__strong修饰符的id类型的类或者Blcok类型成员变量时
4. 在方法名中含有usingBlock的Cocoa框架方法或者GDC的API中传递的时候.

* `__NSMallocBlock__`：堆block，如果NSStackBlock需要在其作用域外部使用的时候，需要拷贝到堆上，与普通的对象一样，支持retain/release

  * 使用copy与retain类似，引用计数加一，内容不会重新拷贝一份
  * 尽量不要使用retain：使用retain不会把block拷贝到heap，引用计数也不会增加，当然，现在在ARC上不会用到retain

## block与copy
* copy方法可以把栈上的block拷贝到堆上，如果block已经在栈上了，copy让引用计数+1
* 在ARC下，block复制给一个strong修饰的变量时，自动调用copy方法
* block没有retain方法，对block发送retain消息实际上是什么都不做
*

## block特性
1. 捕获自由变量，通过copy的方式(待验证)
2. 使用`__block`修饰的变量，block赋值其引用地址来实现访问和修改

  使用`__block`修饰

```objc
//    __block int i = 1024;        //i分配在栈上

//    上面代码会在编译时被转换成
//    __Block_byref_val_0 i = {
//        0,
//        &val,
//        0,
//        sizeof(__Block_byref_val_0),
//        10
//    };
```

在访问block的时候，使用内部的一个叫__forwarding的指针访问，
![](http://upload-images.jianshu.io/upload_images/1862021-100fdd59e5b0c03a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过__forwarding, 无论实在block中, block外访问__block变量, 也不管该变量在栈上或堆上, 都能顺利地访问同一个__block变量.


## 代码
```objc
- (void)foo
{
  __ block int i = 1024;        //i分配在栈上
  int j = 1;                    //j分配在栈上
  void (^block)(void) = ^{      //block位于栈上
    NSLog(@"%d, %d\n", i, j);
  };

  block();                      
  j++;                    
  int (^newBlock)(int, int) = [block copy];   //调用Block_copy()方法，返回在堆上的新的block实例，由于
}
```
