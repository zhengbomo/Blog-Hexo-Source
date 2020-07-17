---
title: iOS图像渲染原理解析
tags: [OpenGL, iOS]
date: 2020-07-10 21:33:03
updated: 2020-07-10 21:33:03
categories: iOS
---


## CPU和GPU

{% img /images/post/opengl/cpu-gpu.png 800 %}

从图中结构可以看出

* `CPU`的处理单元少，由控制器和缓存单元，擅长处理复杂的逻辑以及数据结构，CPU中的并行其实是通过时间切片完成的。任务之间依赖性高，`擅长逻辑控制`  
* `GPU`是由许多计算单元组成，每个计算单元可以独立工作，任务之间依赖性低，擅长浮点运算，`擅长并发计算`

<!-- more -->

## 计算机渲染原理

### ⾼级光栅扫描显示系统结构

{% img /images/post/opengl/print-structure.png 700 %}

### 屏幕扫描

`视频控制器/显示控制器`从`帧缓冲区`中读取图像信息（位图），经过数模转换（数字信号处->模拟型号）后通过`逐行扫描`把图像显示到显示器上的

{% img /images/post/opengl/screen-scan.png 500 %}

#### 撕裂问题

显示一个完整画面是需要一定时间的，视频控制器在显示图像的过程中，如果这时候帧缓冲区的数据被刷新了，就会造成`撕裂`的问题，上面部分显示前面一帧的数据，下面部分为新的数据

{% img /images/post/opengl/image-tear.png 500 %}

#### 双缓冲区和垂直同步

为了解决撕裂，苹果引入了`垂直同步`（VSync） + `双缓存区`（DoubleBuffering）来解决撕裂的问题（苹果使用的就是这个策略）

1. 垂直同步Vsync：每隔1/60s就会发出一个信号，让GPU开始渲染图像，而这个时间间隔足够视频控制器显示图像了，App启动后，会在Runloop注册对应的CFRunLoopSource，通过mach_port，接受来自系统的Vsync事件（实际上是由硬件发出的，每秒钟发60次），CADisplayLink也是同样的原理
2. 双缓存区 DoubleBuffering，使用两个帧缓冲区，视频控制器使用的帧缓冲区和GPU使用的分开，避免视频控制器正在使用的缓冲区被修改，避免撕裂问题，在GPU把帧数据写到帧缓冲区后，会和视频控制器使用的帧缓冲区进行交换，然后等待下一帧的渲染

{% img /images/post/opengl/double-buffer.png 800 %}

#### 掉帧

上面解决了撕裂的问题，但是还有一个掉帧的问题，如下图

{% img /images/post/opengl/jank.png 800 %}

当CPU和GPU渲染图像的时间过长，在下一个垂直同步信号来的时候，GPU并没有处理完一帧的数据，帧缓冲区也就没有交换，视频控制器就会显示原来缓冲区的内容

#### 三缓冲区

从上图可以看出，CPU和GPU是在垂直同步信号到来的时候才开始渲染的工作，为了`减少掉帧`的情况，引入了`三缓冲区`

A：显示到屏幕
B：提前渲染号
C：正在渲染

其实相当于预加载，充分利用CPU和GPU的空闲时间，提前渲染好一帧B（同时也会带来画面延迟，当然1帧的延迟是可以接受的），多留出了一帧的时间，即使在渲染C的时候出现了一次掉帧，依然能刘畅渲染，这种情况大大减小了掉帧的可能

但如果渲染C的时间过长（掉多帧），依然会带来掉帧的问题，三缓冲区本质上并不解决掉帧的问题，只是缓解

> 为了解决掉帧的问题，我们只能尽可能优化我们的代码，减少CPU和GPU的渲染时间

## iOS的渲染框架

### 渲染框架

{% img /images/post/opengl/ios-render-structure.png 800 %}

可以看到在iOS中的`CoreGraphics`, `CoreAnimation`, `CoreImage`都是通过OpenGL/Metal进行渲染的，我们的App也可以使用OpenGL/Metal来操作GPU进行渲染

### CoreAnimation 渲染流⽔线

{% img /images/post/opengl/coreanimation-pipe.png 1000 %}

`CoreAnimation`会在`Runloop`注册一个`Observer`监听触摸事件，当点击事件到来的时候，Runloop会被唤醒处理相关的业务逻辑（UIView的创建，修改，添加动画等）

最终会在CALayer通过`CATransaction`提交到`RenderServer`中，RenderServer会对图片进行解码，并等待下一个`VSync`的到来

VSync信号到来后，`RenderService`会通过OpenGL/Metal做一些绘制操作，然后把处理完的数据（纹理，顶点，着色器等）提交给`GPU`

GPU通过下面渲染流程程（顶点数据->顶点着⾊器->⽚元着⾊器），渲染到`帧缓冲区`，然后交换`帧缓冲区`（双缓冲区）

下一个VSync信号到来的时候，视频控制器读取帧缓冲区的数据显示到屏幕上

如果此处有动画，CoreAnimation会通过`DisplayLink`等机制多次触发相关流程

{% img /images/post/opengl/renderservice.png 800 %}

### 渲染流程

1. `CPU`阶段
   * 布局（Frame）: `layoutSubviews`, `addSubview`
   * 显示（Core Graphics）: `drawRect`, 绘制字符串
   * 准备（QuartzCore/Core Animation）：图片`decode`
   * 提交：通过`IPC`提交(打包好的layers以及动画属性)给OpenGL/Metal，递归提交subview的layers

2. `OpenGL ES/Metal`阶段，主要是对图层进行取色，采样，生成纹理，绑定数据，生成前后帧缓存，为GPU渲染做准备
   * 生成(Generate)
   * 绑定(Bind)
   * 缓存数据(Buffer Data)
   * 启用(Enable)
   * 设置指针(Set Pointers)
   * 绘图(Draw)
   * 清除(Delete)

3. `GPU`阶段
   * 接收提交的纹理（Texture）和顶点描述（三角形）
   * 应用变换（transform）
   * 合并渲染（离屏渲染等）

## 参考

* [iOS界面渲染流程分析](https://www.jianshu.com/p/39b91ecaaac8)
