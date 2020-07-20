---
title: 【OpenGL案例4】球，环，圆柱，磁盘的绘制
tags: [OpenGL]
date: 2020-07-20 12:55:22
updated:
categories: OpenGL
---

在之前的案例中，我们通过定义三角形顶点绘制了一些几何图形，本案例使用OpenGL内置的图形绘制，查看`GLTool.h`，可以看到内置了下面图形

<!-- more -->

{% img /images/post/opengl/object-render-demo.gif 600 %}

## 内置图形

### 球

```cpp
//参数1：sphereBatch，三角形批次类对象
//参数2：fRadius，球体半径
//参数3：iSlices，从球体底部堆叠到顶部的三角形带的数量；其实球体是一圈一圈三角形带组成
//参数4：iStacks，围绕球体一圈排列的三角形对数
gltMakeSphere(GLTriangleBatch& sphereBatch, GLfloat fRadius, GLint iSlices, GLint iStacks);
```

### 环

```cpp
//参数1：torusBatch，三角形批次类对象
//参数2：majorRadius,甜甜圈中心到外边缘的半径
//参数3：minorRadius,甜甜圈中心到内边缘的半径
//参数4：numMajor,沿着主半径的三角形数量
//参数5：numMinor,沿着内部较小半径的三角形数量
gltMakeTorus(GLTriangleBatch& torusBatch, GLfloat majorRadius, GLfloat minorRadius, GLint numMajor, GLint numMinor);
```

### 圆柱/圆锥

设置顶部和底部面的半径，相同为`圆住`，其中一个面的`半径为0`则为`圆锥`，半径不同类似一把中间镂空的伞

```cpp
//参数1：cylinderBatch，三角形批次类对象
//参数2：baseRadius,底部半径
//参数3：topRadius,头部半径
//参数4：fLength,圆形长度
//参数5：numSlices,围绕Z轴的三角形对的数量
//参数6：numStacks,圆柱底部堆叠到顶部圆环的三角形数量
void gltMakeCylinder(GLTriangleBatch& cylinderBatch, GLfloat baseRadius, GLfloat topRadius, GLfloat fLength, GLint numSlices, GLint numStacks);
```

### 磁盘

2D平面图形，有两个圆组成一个平面环

```cpp
//参数1:diskBatch，三角形批次类对象
//参数2:innerRadius,内圆半径
//参数3:outerRadius,外圆半径
//参数4:nSlices,圆盘围绕Z轴的三角形对的数量
//参数5:nStacks,圆盘外网到内围的三角形数量
void gltMakeDisk(GLTriangleBatch& diskBatch, GLfloat innerRadius, GLfloat outerRadius, GLint nSlices, GLint nStacks);
```

### 立方体

```cpp
//参数1:cubeBatch，立方体批次类对象
//参数2:fRadius，每个方向到原点距离都为20个单位长度的立方体
void gltMakeCube(GLBatch& cubeBatch, GLfloat fRadius);
```

## 案例

本案例基于[案例3](/2020-07-19/opengl-demo3/)，原来的部分不做介绍，定义图形批次类

```cpp
// 球
GLTriangleBatch     sphereBatch;
// 环
GLTriangleBatch     torusBatch;
// 圆柱
GLTriangleBatch     cylinderBatch;
// 锥
GLTriangleBatch     coneBatch;
// 磁盘
GLTriangleBatch     diskBatch;
// 立方体
GLBatch             cubeBatch;
```

生成批次类数据

```cpp
// 球
gltMakeSphere(sphereBatch, 3.0, 10, 20);
// 环面
gltMakeTorus(torusBatch, 3.0f, 0.75f, 15, 15);
// 圆柱
gltMakeCylinder(cylinderBatch, 2.0f, 2.0f, 3.0f, 15, 2);
// 锥
gltMakeCylinder(coneBatch, 2.0f, 0.0f, 3.0f, 13, 2);
// 磁盘
gltMakeDisk(diskBatch, 1.5f, 3.0f, 13, 3);
// 正立方体
gltMakeCube(cubeBatch, 1);
```

在renderSence判断显示哪一个图形

```cpp
switch(nStep) {
    case 0:
        drawWireFramedBatch(&sphereBatch);
        break;
    case 1:
        drawWireFramedBatch(&torusBatch);
        break;
    case 2:
        drawWireFramedBatch(&cylinderBatch);
        break;
    case 3:
        drawWireFramedBatch(&coneBatch);
        break;
    case 4:
        drawWireFramedBatch(&diskBatch);
        break;
    case 5:
        drawWireFramedBatch(&cubeBatch);
        break;
}
```

代码见[这里](https://github.com/zhengbomo/OpenGLDemo/tree/master/004--OpenGL%E7%BB%98%E5%88%B6%E5%87%A0%E4%BD%95%E5%9B%BE%E5%BD%A2)