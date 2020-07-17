---
title: iOS离屏渲染原理和优化
tags: [iOS]
date: 2020-07-14 10:07:11
updated: 2020-07-14 10:07:11
categories: iOS
---

几乎做iOS开发的人都知道，设置`圆角`会触发`离屏渲染`，那么什么情况下设置圆角不会触发离屏渲染呢，为什么会触发离屏渲染。

<!-- more -->

## 画家算法

计算机图层的叠加绘制大概遵循`画家算法`，在这种算法下会按层绘制，首先绘制距离较远的场景，然后用绘制距离较近的场景覆盖较远的部分，如下图。

{% img /images/post/opengl/painter-draw.png 800 画家算法 %}

这样就不会导致远的物体挡住近的物体，但是有个局限，就是无法在后面一层渲染完成后，再回去修改前面图层，因为前面的图层已经被覆盖了

## 离屏渲染

对于有前后依赖的图层（如全局剪切，阴影等），画家算法无法满足我们的需求，对于有前后依赖的图层，我们可以再另开辟一个空间，用于临时渲染，渲染完成后再渲染到当前的缓冲区上，这个临时渲染，就是离屏渲染，由于需要开辟一个新的内存空间，由于共享同一个上下文，所以还需要做上下文切换，并且渲染完成后还要进行拷贝操作

1. `开辟临时缓存空间`
2. `上下文切换`
3. `内存拷贝`
4. `额外的渲染`（没有进一步考证）

上面4项带来的开销会很大，并且每一帧渲染都需要执行，如果屏幕上触发离屏渲染的操作过多，会导致GPU渲染时间过长造成卡顿，应该避免触发离屏渲染

{% img /images/post/opengl/offscreen-flow.png 600 %}

## iOS圆角问题

官方文档关于`layer.cornerRadius`的描述

{% img /images/post/opengl/layer-cornerradius.png 600 %}

> `layer.cornerRadius`只作用`backgroundColor`和`border`，不会作用于`content`，支持`动画`

离屏渲染是系统无法按画家算法一次性渲染完我们的视图才会触发，我们先来看几个iOS的例子，模拟器打开`Color Off-screen Rendered`

```swift
// 1. UIImageView
let imageView = UIImageView(frame: CGRect(x: 50, y: 100, width: 300, height: 200))
self.view.addSubview(imageView)
imageView.image = UIImage.init(named: "test.jpg")

// image + cornerRadius + masksToBounds 不会触发离屏渲染
imageView.layer.cornerRadius = 10
imageView.layer.masksToBounds = true

// 触发离屏渲染
imageView.backgroundColor = UIColor.green
// 添加一个空的UIView不会触发离屏渲染
// imageView.addSubview(UIView(frame: CGRect(x: 0, y: 0, width: 10, height: 10)))

// 2. UIButton
let button = UIButton(type: .custom)
button.frame = CGRect(x: 50, y: 300 + 50, width: 300, height: 50)
self.view.addSubview(button)
button.setTitle("Test", for: .normal)
button.setTitleColor(UIColor.blue, for: .normal)
button.layer.cornerRadius = 10
button.layer.masksToBounds = true

// 触发离屏渲染
button.backgroundColor = UIColor.green
// 触发离屏渲染
button.setBackgroundImage(UIImage(named: "test.jpg"), for: .normal)

// 3. UIView
let view = UIView(frame: CGRect(x: 50, y: 400 + 50, width: 300, height: 50))
self.view.addSubview(view)
view.backgroundColor = UIColor.red
view.layer.cornerRadius = 10
view.layer.masksToBounds = true

// label如果被渲染，则会触发渲染，如果text为空不会被渲染
let label = UILabel(frame: CGRect(x: 10, y: 10, width: 1, height: 1))
label.text = "1"
view.addSubview(label)
```

{% img /images/post/opengl/offscreen-demo.png 300 出现离屏渲染的地方被标记为黄色 %}

如果设置了`cornerRadius+masksToBounds`（裁切），并且用于渲染的图层大于1，就会触发离屏渲染，其中如果设置`backgroundColor`，背景颜色相当于一个单独一个图层，`subviews`的图层也算，UILabel如果text为空（subviews为空，backgroundColor为空），则不会生成渲染图层

所以设置了`cornerRadius+masksToBounds`的

* `UIImageView`设置图片不会触发离屏渲染
* `UIView`设置了背景颜色，但不添加subview，不会触发离屏渲染
* `UILabel`设置了文字，并且设置了backgroundColor，会触发离屏渲染
* `UIButton`只设置文字和背景，会触发离屏渲染

## 优化圆角问题

基于上面的问题，我们可以有几个优化方向

1. 避免使用裁切操作，如果我们能确保View里面的内容不会溢出，就可以不用`masksToBounds`
2. 即使要用到裁切的操作，尽量放到子view里面，不要在上层view使用masksToBounds，因为裁切需要对所有的layer和subview所有图层都进行裁切，这样离屏渲染会需要更大的空间，裁切更多的图层，应该只对必要的view/layer进行裁切
3. 提前切好需要的圆角，避免渲染的时候再切

## 其他触发离屏渲染的情况

* 使用了遮罩的 layer (`layer.mask`)
* 需要进行裁剪的 layer (`layer.masksToBounds` / `view.clipsToBounds`)
* 设置了组透明度为 YES，并且透明度不为 1 的layer (`layer.allowsGroupOpacity` / `layer.opacity`)
* 添加了投影的 layer (`layer.shadow`)
* 采用了光栅化的 layer (`layer.shouldRasterize`)，光栅化也可以优化离屏渲染问题
* 绘制了文字的 layer (`UILabel`, `CATextLayer`, `CoreText`等)

## 毛玻璃

在iOS系统中，毛玻璃效果可以说应用的非常广泛，从上面分析也可以知道，这个肯定会触发离屏渲染的，图层之间存在依赖，下面是`UIBlurEffect`在GPU上的渲染过程

{% img /images/post/opengl/uiblureffect-render.png 1000  %}

GPU在渲染完Content之后，会另外开辟一个`Off-screen buffer`，执行下面步骤，最后再做合并处理，最后再拷贝回`On-screen buffer`上

{% img /images/post/opengl/uiblureffect-render2.png 600  %}
