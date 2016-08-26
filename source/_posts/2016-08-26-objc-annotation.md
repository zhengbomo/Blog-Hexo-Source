---
title: Object-C注释
categories: iOS
tags: iOS
date: 2016-08-26 08:26:26
updated: 2016-08-26 08:26:26
---


与其他语言一样，Object-C的注释也分为两种，一种是普通注释，一种是文档注释，普通注释通常给阅读代码的人看，而文档注释除了可以给阅读的文件的人看还可以被appledoc识别，在使用的时候xcode能给出智能提示，有些工具还可以把文档注释生成文档

<!-- more -->

![](http://7xqzvt.com1.z0.glb.clouddn.com/445476-20150917114303758-1998047181.png)

## 一、简单注释
1. 单行注释
  单行注释不能被文档识别，通常用于函数内部
  ```objc
  //学生信息
  ```
2. 多行注释
  可以被工具识别
  ```objc
  /*
   *　多行注释内容1
   *　多行注释内容2
   */
  ```

## 二、文档注释（appledoc可识别成文档）
1. 单行注释
  ```objc
  @interface Student : NSObject

  ///名字
  @property (nonatomic, copy) NSString *name;
  ///年龄
  @property (nonatomic, assign) NSInteger age;
  ///校园卡Id
  @property (nonatomic, copy) NSString *schoolId;
  ///年纪
  @property (nonatomic, copy) NSString *grade;

  @end
  ```

  如果安装了[VVDocument](https://github.com/onevcat/VVDocumenter-Xcode)，当输入`///`的时候回自动生成多行注释，通常我们通过`/** 注释内容 */`进行注释
  ```objc
  @interface Student : NSObject

  /** 名字 */
  @property (nonatomic, copy) NSString *name;
  /** 年龄 */
  @property (nonatomic, assign) NSInteger age;
  /** 校园卡Id */
  @property (nonatomic, copy) NSString *schoolId;
  /** 年纪 */
  @property (nonatomic, copy) NSString *grade;

  @end
  ```

2. 多行注释
```objc
/** 简要描述.
 *
 * 详细描述或其他.
 */
```

3. 行尾注释
有时候一些简短的注释可以可以用行尾注释来减少代码的行数，通常用在枚举变量的注释上
```objc
@interface Student : NSObject

@property (nonatomic, copy) NSString *name;         /**< 名字 */
@property (nonatomic, assign) NSInteger age;        /**< 年龄 */
@property (nonatomic, copy) NSString *schoolId;     /**< 校园卡Id */
@property (nonatomic, copy) NSString *grade;        /**< 年纪 */

@end
```

4. 函数注释
  函数注释也属于多行注释，通常我们使用 [VVDocument](https://github.com/onevcat/VVDocumenter-Xcode) 插件辅助

  ```objc
  /**
   *  获取状态描述
   *
   *  @param state 状态值
   *
   *  @return 返回状态描述
   */
  - (NSString *)getState:(NSInteger)state
  {
      switch (state) {
          case 1:
              return @"待确认";
              break;
          case 2:
              return @"确认";
              break;
          case 3:
              return @"驳回";
              break;
      }
  }
  ```

## 三、总结
上面介绍了objc中所有的注释样式，在实际开发中，我们应该多使用文档注释，使用文档注释可以获得xcode的智能提示，在用appledoc生成文档的时候也可以被识别

## 四、参考链接
* [http://www.cnblogs.com/zyl910/archive/2013/06/07/objcdoc.html](http://www.cnblogs.com/zyl910/archive/2013/06/07/objcdoc.html)
