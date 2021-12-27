---
title: xx直播编译优化
tags: [iOS]
date: 2021-09-27 17:49:32
updated: 2021-09-27 17:49:32
categories: iOS
---

## 一、背景
`编译`对于开发者可以说是最为频繁的操作，编译速度的快慢会极大的影响我们的开发效率，对于xx直播App来说，由于代码量大，加上代码结构设计不合理等原因，导致编译速度非常慢，每次启动App（即使不改动代码）需要3分钟的时间才能跑在手机上，而一次完整的编译，则需要15分钟，严重影响日常的业务开发和调试，现阶段编译速度的优化显得尤为重要，主要的时间消耗有

<!-- more -->

* Compile
* Linking
* Run Script
* Sign
* Install（文件多可能是主因）

其中`Compile`时间占用`90%`以上，这里只关注Compile

## 二、调研

Xcode是基于`llvm`编译的，llvm编译器的编译过程主要分为3个部分

* 前端（Frontend）：负责解析源码，检查错误，生成抽象语法树（AST），并把 AST 转化成类汇编中间代码
* 优化器（Optimizer）：对中间代码进行架构无关的优化，提高运行效率，减少代码体积，例如无效代码，无用变量等
* 后端（Backend）：把中间代码转换成目标平台的机器码

其中我们代码影响最大的`前端`，这里我们只关注前端，前端处理主要有

1. 预处理：这阶段的工作主要是
2. 头文件展开处理
3. 宏展开/替换，预编译指令处理
4. 注释去除处理
5. 编译：这阶段做的事情比较多
6. 词法分析（Lexical Analysis）：将代码转换成一系列 token
7. 语法分析（Semantic Analysis）：将token树抽象语法树 AST
8. 静态分析（Static Analysis）：检查代码错误，例如参数类型是否错误，调用对象方法是否有实现
9. 中间代码生成（Code Generation）：将语法树自顶向下遍历逐步翻译成 LLVM IR

llvm9.0之后添加了一个新的编译参数可以用于统计编译过程中各个阶段的耗时（`-ftime-trace`），利用该参数和编译器我们可以得到项目中所有文件编译的详细时间信息

### 2.1 -ftime-trace测试

1. 下载llvm编译器，`9.0`以上的版本均可，解压后得到clang+llvm-9.0.0，我这里放到（~/development/clang+llvm-9.0.0）
2. 在需要统计编译时间的Target中修改clang前端编译器（CC和CXX），和编译参数（OtherCFlag）

    ```
    CC: /path/to/clang
    CXX: /path/to/clang++
    Other C Flag: 添加-ftime-trace
    ```

3. 关闭INDEX：COMPILER_INDEX_STORE_ENABLE=NO
4. 编译后，在目标文件（xxx.o）同个目录下，会生成xxx.json文件，如

    * YYTimer.o
    * YYTimer.json

5. 可以用Chrome打开chrome://tracing，然后拖入该json文件，可以直观的看到编译时间 点击查看
6. 整个json文件如下

    ```json
    {"traceEvents":[{"pid":1,"tid":0,"ph":"X","ts":21778,"dur":43204,"name":"Module Load","args":{"detail":"UIKit"}},{"pid":1,"tid":0,"ph":"X","ts":21329,"dur":44253,"name":"Source","args":{"detail":"/Users/bomo/Documents/Code/iOS/Work/Analyze/4986-02-header2/FanXing/Pods/Target Support Files/KGThirdParty/KGThirdParty-prefix.pch"}},{"pid":1,"tid":0,"ph":"X","ts":66818,"dur":1868,"name":"Source","args":{"detail":"/Users/bomo/Documents/Code/iOS/Work/Analyze/4986-02-header2/FanXing/Pods/KGThirdParty/KGThirdParty/YYKit/Utility/YYTimer.h"}},{"pid":1,"tid":0,"ph":"X","ts":20679,"dur":70778,"name":"Frontend"},{"pid":1,"tid":0,"ph":"X","ts":91457,"dur":595,"name":"Frontend"},{"pid":1,"tid":0,"ph":"X","ts":94811,"dur":617,"name":"RunPass","args":{"detail":"AArch64 Assembly Printer"}},{"pid":1,"tid":0,"ph":"X","ts":94084,"dur":1370,"name":"OptFunction","args":{"detail":"\u0001+[YYTimer timerWithTimeInterval:target:selector:repeats:]"}},{"pid":1,"tid":0,"ph":"X","ts":95456,"dur":526,"name":"OptFunction","args":{"detail":"\u0001-[YYTimer init]"}},{"pid":1,"tid":0,"ph":"X","ts":95983,"dur":1123,"name":"OptFunction","args":{"detail":"\u0001-[YYTimer initWithFireTime:interval:target:selector:repeats:]"}},{"pid":1,"tid":0,"ph":"X","ts":93022,"dur":6873,"name":"OptModule","args":{"detail":"/Users/bomo/Documents/Code/iOS/Work/Analyze/4986-02-header2/FanXing/Pods/KGThirdParty/KGThirdParty/YYKit/Utility/YYTimer.m"}},{"pid":1,"tid":0,"ph":"X","ts":93013,"dur":6977,"name":"CodeGenPasses"},{"pid":1,"tid":0,"ph":"X","ts":92061,"dur":8108,"name":"Backend"},{"pid":1,"tid":0,"ph":"X","ts":66,"dur":100526,"name":"ExecuteCompiler"},{"pid":1,"tid":1,"ph":"X","ts":0,"dur":100526,"name":"Total ExecuteCompiler","args":{"count":1,"avg ms":100}},{"pid":1,"tid":2,"ph":"X","ts":0,"dur":71372,"name":"Total Frontend","args":{"count":2,"avg ms":35}},{"pid":1,"tid":3,"ph":"X","ts":0,"dur":46121,"name":"Total Source","args":{"count":2,"avg ms":23}},{"pid":1,"tid":4,"ph":"X","ts":0,"dur":43318,"name":"Total Module Load","args":{"count":3,"avg ms":14}},{"pid":1,"tid":5,"ph":"X","ts":0,"dur":8108,"name":"Total Backend","args":{"count":1,"avg ms":8}},{"pid":1,"tid":6,"ph":"X","ts":0,"dur":7094,"name":"Total OptModule","args":{"count":2,"avg ms":3}},{"pid":1,"tid":7,"ph":"X","ts":0,"dur":6977,"name":"Total CodeGenPasses","args":{"count":1,"avg ms":6}},{"pid":1,"tid":8,"ph":"X","ts":0,"dur":5819,"name":"Total OptFunction","args":{"count":40,"avg ms":0}},{"pid":1,"tid":9,"ph":"X","ts":0,"dur":5585,"name":"Total RunPass","args":{"count":729,"avg ms":0}},{"pid":1,"tid":10,"ph":"X","ts":0,"dur":479,"name":"Total DebugType","args":{"count":88,"avg ms":0}},{"pid":1,"tid":11,"ph":"X","ts":0,"dur":279,"name":"Total CodeGen Function","args":{"count":1,"avg ms":0}},{"pid":1,"tid":12,"ph":"X","ts":0,"dur":223,"name":"Total PerModulePasses","args":{"count":1,"avg ms":0}},{"pid":1,"tid":13,"ph":"X","ts":0,"dur":56,"name":"Total Module LoadIndex","args":{"count":1,"avg ms":0}},{"pid":1,"tid":14,"ph":"X","ts":0,"dur":21,"name":"Total PerFunctionPasses","args":{"count":1,"avg ms":0}},{"pid":1,"tid":15,"ph":"X","ts":0,"dur":3,"name":"Total PerformPendingInstantiations","args":{"count":1,"avg ms":0}},{"cat":"","pid":1,"tid":0,"ts":0,"ph":"M","name":"process_name","args":{"name":"clang-10"}}]}
    ```

7. 这里摘取`YYTimer.json`里面的一个片段

    ```json
    {
        "pid": 1,
        "tid": 0,
        "ph": "X",
        "ts": 66818,
        "dur": 1868,
        "name": "Source",
        "args": {
            "detail": "/Users/bomo/Documents/Code/iOS/Work/FanXing/Pods/KGThirdParty/KGThirdParty/YYKit/Utility/YYTimer.h"
        }
        },
        {
        "pid": 1,
        "tid": 1,
        "ph": "X",
        "ts": 0,
        "dur": 100526,
        "name": "Total ExecuteCompiler",
        "args": {
            "count": 1,
            "avg ms": 100
        }
    }
    ```

    这里的name=Source为头文件YYTimer.h预编译处理的时间，dur为时间，单位为微秒，name=Total ExecuteCompiler为该文件的编译时间

## 三、项目测试

通过上面的方法，编译整个项目，并统计所有文件的预编译处理时间，我这里用`python`扫描所有编译文件，然后做汇总统计

### 3.1 各编译阶段耗时

| 编译类型 | 耗时 | 
| ---- | ---- | 
| Total ExecuteCompiler: | 5077.09 秒 | 
| Total Frontend: | 3571.56 秒 | 
| Total Source: | 2479.86 秒 | 
| Total Module Load: | 1598.68 秒 | 
| Total Backend: | 396.09 秒 | 
| Total CodeGenPasses: | 373.86 秒 |
| Total OptModule: | 371.34 秒 | 
| Total OptFunction: | 272.03 秒 |
| Total RunPass: | 263.66 秒 | 
| Total Module Compile: | 142.07 秒 |

从上面数据看出`Source`耗时最长，占用`2479.86s`，占比较大

 ### 3.2 头文件引用次数（TOP10） 

 通过上面生成的json文件可以得到 
 
| 头文件 | 引用次数 | 平均耗时 | 
| ---- | ---- | --- | 
| ******Common-prefix.pch | 1619次 | 51.91毫秒 | 
| ******Singleton.h | 712次 | 6.01毫秒 |
| ******User-prefix.pch | 701次 | 50.23毫秒 |
| ******GiftList.h | 623次 | 10.22毫秒 |
| ******ModelObject.h | 617次 | 3.42毫秒 | 
| ******Constants.h | 617次 | 1.86毫秒 | 
| ******LiveInfo.h | 612次 | 3.34毫秒 |
| ******ProgramInfo.h | 612次 | 5.58毫秒 | 
| ******AnimationView.h | 612次 | 2.27毫秒 |
| ******ViewDefine.h | 611次 | 2.75毫秒 |

> 注：这里的引用，包含间接引用 * A引用C * B引用A * D引用B

则C被引用3次，会参与3次预编译处理，当D被引用100次时，A,B,C也会被引用处理100次，编译器在编译的时候会有其他优化策略，具体次数可能会细微差异 

### 3.3 头文件单次预处理耗时（TOP10）

| 头文件 | 耗时（平均） | 引用次数 | 
| ---- | ---- | ---- |
| ******ResultVC.h | 31971.25毫秒 | 1 | 
| ******ListView.h | 17175.35毫秒 | 1 | 
| ******ItemEntity.h | 16750.17毫秒 | 1 |
| ******GuideView.h | 16623.81毫秒 | 1 |
| ******RecommendView.h | 15125.93毫秒 | 1 | 
| ******DrawerCell.h | 13072.67毫秒 | 2 | 
| ******HelperMsgContainer.h | 13055.01毫秒 | 2 | 
| ******RecordCell.h | 10176.65毫秒 | 2 | 
| ******VerifyDao.h | 7482.75毫秒 | 2 |
| ******AppealModel.h | 7247.25毫秒 | 4 | 
    
> 注：由于头文件会多级引用，所以处理时间会叠加，这里仅供参考

### 3.4 头文件预处理总耗时（TOP10） 

| 头文件 | 总耗时 | 引用次数 | 
| ---- | ---- | ---- | 
| ****Common.h | 939.64秒 | 610 |
| ****AlertView.h | 267.89秒 | 608 |
| ****Common-prefix.pch | 84.04秒 | 1619 | 
| ****MesageParse.h | 58.98秒 | 600 | 
| ****RoomManager.h | 43.77秒 | 598 | 
| ****User-prefix.pch | 35.21秒 | 701 | 
| ****VideoInfoModel.h | 34.40秒 | 10 | 
| ****VideoModel.h | 33.91秒 | 8 | 
| ****ResultVC.h | 31.97秒 | 1 | 
| ****Data-umbrella.h | 31.47秒 | 64 | 

> 注：`xxxxCommon.h`文件引用次数不是最多，但总耗时最长

## 4. 方案

通过上面测试数据可以看出，`xxxxCommon.h`预处理耗时最长，通过查看该文件可以看出 

* 该文件引用了非常多头文件，完全展开的话会非常大，符合耗时的预期 
* 项目中很多地方引用`xxxxCommon.h`文件可能只是用到里面其中一个或几个类，而引用整个`xxxxCommon.h`导致而预处理却花去了大量的时间 

这里决定给`xxxxCommon.h`文件进行瘦身，从而提高编译速度，

**`头文件按需引用，减少不必要的预编译处理`** 方案详情

## 5. 成效 

* 机器：iMac (Retina 4K, 21.5-inch, 2019)， 
* CPU: 3 GHz 六核Intel Core i5 * 内存：16 GB 2667 MHz DDR4 * 显卡：Radeon Pro 560X 4 GB 
* Configuration: `DEBUG` 
* 编译架构：`x86_64`

### 5.1宏观统计（6核）（减少110.9s） 

测试方法：

1. 关闭Xcode，关闭Chrome等大进程 
2. 删除所有Xcode缓存文件（`~/Library/Developer/Xcode/DerivedData/`） 
3. 打开Xcode编译 
4. 编译完成后，查看Xcode显示的编译总时间 
5. 测试结果去掉最高最低值

| 宏观统计 | 测试1 | 测试2 | 测试3 | 测试4 | 平均 | 
| ---- | ---- | ---- | ---- | ---- | ---- |
| 基础工程 | 447.9 | 446.3 | 452.7 | 453.2 | 450.025 | 
| 优化后工程 | 339.8 | 336.8 | 338.1 | 341.7 | 339.1 |

 > 注：部分子库用了二进制，这里更多关注差值

 ### 5.2 微观统计（减少372.7s） 

 测试方法： 
 
 1. 使用上面提到的llvm自带统计工具，然后汇总结果统计所有文件的`Total ExecuteCompiler` 
 2. 测试结果去掉最高最低值 
 3. 去掉所有的clang插件 
| 微观统计 | 测试1 | 测试2 | 测试3 | 平均 |
| ---- | ---- | ---- | ---- | ---- |
| 基础工程 | 1449.3 | 1445.8 | 1456.3 | 1450.4 | 
| 优化后工程 | 1076.8 | 1092.3 | 1064.1 | 1077.7 |

## 总结

1. 上面优化修改了2115个文件，工作量还是比较大的，主要还是编码习惯和历史包袱带来量变积累，导致质变，平时养成良好的编码习惯，减少量变积累 
2. Xcode在编译的时候会把`#import <AAA/BBB.h>`自动转成`@import AAA.BBB`，为了统一风格，建议统一使用`#import <AAA/BBB.h>`方式引用，详情见[WWDC2013-Advances in Objective-C](https://developer.apple.com/videos/play/wwdc2013/404/) 
3. 坚持一个原则：**按需引用**，**按需引用**（最小import原则），请引用`#import <FAFuncUnit/XXX.h>`，而不是`#import <LibA/LibA-umbrella.h>`

