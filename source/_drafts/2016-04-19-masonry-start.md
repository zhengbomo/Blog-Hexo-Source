---
title: Masonry学习笔记
tags:
  - masonry
categories: iOS
date: 2016-04-19 00:22:57
---

## 自动布局
在iOS6以后，苹果引入了自动布局，自动布局的好处不需要多讲，特别是适配越来越多不同的屏幕尺寸，不需要关心具体的坐标，而只关心相对位置

<!-- more -->


XCode提供的工具并没有那么好用，通过手动的拉线和拖控件有些时候非常低效，而且容易误操作，对于我这种喜欢用代码写布局的iOSer来说，使用Interface Builder进行布局有时候并不那么灵活，在IB上所有的操作，在代码中都有相关的API，这时候可以考虑使用代码编写，但是官方提供的API使用起来繁琐而臃肿，Masonry基于官方的API，提供了更优雅而简洁的写代码的方式，下面使用做个简单的演示

#### 简单使用

Masonry提供了使用链式编程的方式定义布局约束，可读性更强，基本上一眼就能看懂

```objc c
[secondView mas_makeConstraints:^(MASConstraintMaker *make) {
    make.left.equalTo(self.view.mas_leftMargin).offset(12);
    make.right.equalTo(self.view.mas_rightMargin).offset(-12);
    make.top.equalTo(self.top.mas_top).offset(12);
    make.height.equalTo(@100);
}];
```

如果布局约束已经存在，可以调用`mas_updateConstraints`更新约束



<!-- more -->

### 布局流程

### 自动布局新增API


### UILayoutSupport
  在iOS7以后ViewController添加了`topLayoutGuide`和`bottomLayoutGuide`可以不用考虑状态栏，导航栏，TabBar的情况，会根据实际情况而动态改变，Masonry也支持这两个属性，可以通过`length`属性拿到TopBar和TabBar的具体高度

### 问题
* 多个view层叠的时候，使用SB/Xib管理起来非常麻烦（容易误操作）
* 无法给约束设置float值
* 无法在在Xib设置LayoutGuide约束

## Masonry

### 简介

### 基本用法

### 要点

### 自动算宽高
1. 获取cell，并给cell赋值
2. 给cell的contentView添加宽度约束
3. 调用systemLayoutSizeFittingSize计算cell的宽高
4. 删除宽度约束
5. 补充分割线高度

```objc++
- (CGFloat)autoCalculateHeight:(UITableViewCell *)cell tableView:(UITableView *)tableView
{
    CGFloat contentViewWidth = CGRectGetWidth(tableView.frame);
    CGSize fittingSize = CGSizeZero;

    //add width constraint
    NSLayoutConstraint *widthFenceConstraint = [NSLayoutConstraint constraintWithItem:cell.contentView attribute:NSLayoutAttributeWidth relatedBy:NSLayoutRelationEqual toItem:nil attribute:NSLayoutAttributeNotAnAttribute multiplier:1.0 constant:contentViewWidth];
    [cell.contentView addConstraint:widthFenceConstraint];

    //calculate cell size
    fittingSize = [cell.contentView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];

    //remove width constraint
    [cell.contentView removeConstraint:widthFenceConstraint];

    //add separator height
    if (tableView.separatorStyle != UITableViewCellSeparatorStyleNone) {
        fittingSize.height += 1.0 / [UIScreen mainScreen].scale;
    }

    return fittingSize.height;
}
```

### 约束优先级
对于可能变动的约束，可能存在多种约束的情况，可以通过优先级来控制哪种约束生效

### 微博Cell布局

### 动画
使用Masonry做动画也非常简单，我们只需要关心最终状态下的约束，更新约束，然后执行LayoutIfNeed即可，并把LayoutIfNeed放在动画block里面，通常一些简单的动画可以用Autolayout方式来做，由于Autolayout并不是特别灵活，而一些比较复杂的还是考虑使用Frame模型来做




### 常见问题




* Content Hugging与Content Compression
  内容伸缩属性

*
