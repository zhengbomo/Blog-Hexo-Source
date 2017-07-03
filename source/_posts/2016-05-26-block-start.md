---
title: block学习笔记
tags: [block, iOS]
date: 2017-07-03 00:22:13
updated: 2017-07-03 00:22:13
categories: iOS
---

## 引用计数内存管理
1. 所有的对象都存放在堆上，需要手动管理内存
2. iOS内存管理通过retainCount进行管理，通过引用计数+/-1来控制内存的的声明周期，通常来说，我们在一个代码块中，我们会对需要用到的对象的引用计数+1，在离开代码块时，对引用计数-1，通过这种机制，我们只需要关心在我们的代码中需要的时候retain，不需要的时候release，而不用关心对象什么时候释放，当引用计数为0，即之后再也没有对该内存的引用，对象内存就会被释放，这个由系统框架来做

<!-- more -->

## autorelease与runloop
runloop在每一个调用周期（消息循环）外部都会添加一个`autoreleasepool`，对于`autorelease`的对象，并不是在离开作用域就马上释放的，而是在离开`autoreleasepool`的时候才被释放



```objc
@property (nonatomic, weak) NSArray *array1;
@property (nonatomic, weak) NSArray *array2;

- (IBAction)btnClick:(id)sender {
    [self test];
    NSLog(@"%@", self.array1);    // 这里可以打印出来array1对象的值
    NSLog(@"%@", self.array2);    // 这里打印出来array2为nil
}

- (void)test {
    self.array1 = [NSMutableArray arrayWithObjects:@1, @2, @3, nil];
    // 1. self.array1指向的对象是用autorelease修饰的
    // 2. 虽然array使用weak修饰的，但是离开了test方法，array依然不会被释放，直到离开当前的autoreleasepool

    NSMutableArray *array = [[NSMutableArray alloc] initWithObjects:@1, @2, @3, nil];
    self.array2 = array;
}
```

## 函数与block
block就是对闭包的实现，block的本质其实就是函数，只是在函数的基础上加上了捕获变量列表
编译之后会变成一个结构体，包含捕获参数，和函数指针，后面我们看看block编译之后的代码

## block的编译
### 1. 不捕获变量
先定义一个test.m文件
```objc
#include <stdio.h>

int main() {
    void (^blk)(void) = ^{
    	printf("Block\n");
    };
    blk();
    return 0;
}
```

通过clang命令生成预编译后的代码
```shell
clang -rewrite-objc test.m
```

生成如下代码
```cpp
// block类描述，相当于OC的类描述
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

// block实现的结构体，可以看成__block_impl的子类
struct __main_block_impl_0 {
  struct __block_impl impl;						// block类的描述
  struct __main_block_desc_0* Desc;				// block的描述
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {		# 结构体构造方法
    impl.isa = &_NSConcreteStackBlock;			// block的类型，共有3种
    impl.Flags = flags;
    impl.FuncPtr = fp;							// block的实现，最后被赋值为block编译后的方法
    Desc = desc;								// block描述
  }
};

// block的描述，描述block实现的结构体的大小（框架用，我们用不到）
static struct __main_block_desc_0 {
	size_t reserved;
	size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

// block方法实现
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  printf("Block\n");
}

int main() {
  // 1. 构造block结构体__main_block_impl_0，并且结构体存放在栈上的
  void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));

  // 2. 通过__main_block_impl_0.FuncPtr调用block指向的方法
  ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

  return 0;
}
```
可以看到，block被编译成一个静态方法，和一个block描述的结构体


### 2. 捕获不可变变量
```objc
#include <stdio.h>
#import <Foundation/Foundation.h>

int main() {
	NSObject *obj = [[NSObject alloc] init];
	int a = 10;
    void (^blk)(void) = ^{
    	printf("Block：%d\n", a);
    	NSLog(@"%@", obj);
    };
    blk();
    return 0;
}
```

编译之后的代码
```cpp
// __block_impl 一样

// 生成的__main_block_impl_0多了一个捕获字段a和对象obj
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a;
  NSObject *obj;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, NSObject *_obj, int flags=0) : a(_a), obj(_obj) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

// block的实现方法，通过__cself拿到捕获的变量
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
	// 这里通过block捕获的结构体引用a字段
	int a = __cself->a; // bound by copy
  	NSObject *obj = __cself->obj; // bound by copy

    printf("Block：%d\n", a);
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_h8_fm423vjj5mlgb636xh5b44n80000gn_T_test_e9e5ff_mi_0, obj);
}

// block描述多了两个变量，copy和dispose方法，用于持有和释放block捕获的对象
static struct __main_block_desc_0 {
	size_t reserved;
	size_t Block_size;
	void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
	void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

int main() {
	// 1. 声明自由变量
    NSObject *obj = ((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)((NSObject *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("NSObject"), sel_registerName("alloc")), sel_registerName("init"));

	int a = 10;

	// 2. 构造block结构体__main_block_impl_0，并且结构体存放在栈上
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a, obj, 570425344));

    // 3. 通过__main_block_impl_0.FuncPtr调用block指向的方法
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```
这里用到了一个对象`obj`，`__main_block_desc_0`多了两个方法（`__main_block_copy_0`, `__main_block_dispose_0`），用于持有和释放block捕获的对象，如果只有int，没有对对象的捕获，则不会生成这两个成员

### 3. 捕获可变变量
```objc
#include <stdio.h>

int main() {
	__block int a = 10;
    void (^blk)(void) = ^{
    	printf("Block：%d\n", a);
    };
    blk();
    return 0;
}
```

编译后
```cpp
// 新增一个结构体，对可变成员进行包装
struct __Block_byref_a_0 {
    void *__isa;
    __Block_byref_a_0 *__forwarding;
    int __flags;
    int __size;
    int a;              // 原来的成员
};

// block实现，捕获的成员不是int a, 而是__Block_byref_a_0 *a, 其实就是`__block`修饰的成员包装成对象了
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    __Block_byref_a_0 *a; // by ref                 // 这里是捕获的成员，与上面一样
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
  }
};

int main() {
    // 1. 创建__Block_byref_a_0包装原来的成员a
    __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 10};

    // 2. 同上面捕获的
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
    return 0;
}
```
block捕获可变对象其实与捕获普通变量一样，只是对原有变量包装成一个新的对象而已

### 4. 捕获static变量
这里例子就省了，直接上结论

1. 如果是局部static变量，block会当成普通局部变量一样处理，即会捕获该局部变量
2. 如果是全局static变量，block会直接使用该全局变量，不进行变量捕获处理

## block类型
上面我们看到的block的实现`__main_block_impl_0`包含一个`_isa`字段，用于标识block的类型，block有三种类型

* _NSConcreteStackBlock：存放在Stack上
* _NSConcreteGlobalBlock：与全局变量一样，存放在全局区
* _NSConcreteMallocBlock：存放在堆上

看下面三种代码
```objc
typedef long (^BlkSum)(int, int);

BlkSum blk1 = ^ long (int a, int b) {
    return a + b;
};

// 当block不捕获外部变量，就会被声明成__NSGlobalBlock
NSLog(@"blk1 = %@", blk1);              // blk1 = <__NSGlobalBlock__: 0xce3347d0>
NSLog(@"blk1 = %@", [blk1 copy]);       // blk1 = <__NSGlobalBlock__: 0xce3347d0>

int base = 100;
BlkSum blk2 = ^ long (int a, int b) {
    return base + a + b;
};

// 当block捕获外部变量，就会被声明成__NSStackBlock__
NSLog(@"blk2 = %@", blk2);              // blk2 = <__NSStackBlock__: 0xbfffddf8>
NSLog(@"blk2 = %@", [blk2 copy]);       // blk2 = <__NSMallocBlock__: 0xbfffcef8>

BlkSum blk3 = [[blk2 copy] autorelease];
NSLog(@"blk3 = %@", blk3);              // blk3 = <__NSMallocBlock__: 0x902fda0>
NSLog(@"blk3 = %@", [blk3 copy]);       // blk3 = <__NSMallocBlock__: 0x902fda0>
```

当block存放在全局时，始终只有一份，调用copy/retain/release，返回的是同一个对象
当block存放在栈上时，如果进行copy，block会被拷贝到堆上，调用retain或release无效
当block存放在堆上时，如果进行copy/retain/release，内存管理与对象一样，引用计数+/-1

> 上面是MRC的行为，而在ARC上即使是block没有进行拷贝，也会被拷贝到堆上，所以在ARC上的block只有堆区和全局区，在ARC上，如果block会自动在需要的时候进行copy，如block作为返回值时，block作为参数传给另一个函数时

## 前向引用
当我们在block使用可变变量`__block`的时候，编译器会生成`__Block_byref_a_0`内部有一个`__forwarding`字段指向自己
```cpp
struct __Block_byref_a_0 {
    void *__isa;
    __Block_byref_a_0 *__forwarding;        // 指向自己
    int __flags;
    int __size;
    int a;              // 原来的成员
};
```
为什么要用一个变量指向自己，看下面这个例子
```objc
{
    __block int val = 0;
    void (^blk)(void) = [^{
        ++val;
    } copy];
    ++val;
    blk();
    NSLog(@"%d", val);
}
```
根据上面编译的代码，我们可以分析转换成的代码大致如下
```objc
{
    // 1. 创建byref对象包装int val
    __Block_byref_a_0 val;

    // 2. 创建block对象__main_block_impl_0，这个时候block是存放在栈上的
    __main_block_impl_0 blk = __main_block_impl_0(...)

    // 3. 拷贝block，返回新的block，新的block存放在堆上
    __main_block_impl_0 newBlk = _Block_copy(blk)

    // 对变量＋1
    ++(val->__forwarding->val)

    // 调用函数
    newBlk->FuncP();

    // 打印
    NSLog(@"%d", val);
}
```
上面有两个block（blk和newBlk）一个存放在栈上，一个存放在堆上
上面也有两个__Block_byref_a_0，当进行拷贝时，栈上的val也会被拷贝一份到堆上

当离开函数作用域时，栈上的内存会被释放，所以当block从栈拷贝到堆上时，会把堆上变量的__forwarding指针，指向堆，故后面及时我们使用局部变量val，实际上内部使用的已经是拷贝到堆上的变量了，这时候和block内部使用的变量是统一变量


## 循环引用常见问题
* NSTimer：[http://blog.bombox.org/2017-05-08/nstimer-notes/](http://blog.bombox.org/2017-05-08/nstimer-notes/)
* CADisplayLink: 注意事项和用法与NSTimer类似
* block：任何使用block的地方都需要考虑几个问题
    * 是否造成循环引用
    * 对于异步操作是否因为强持有导致延后释放（例如网路请求）
    * 除了self，被self强持有的变量也会导致循环引用
    * 多层循环引用
