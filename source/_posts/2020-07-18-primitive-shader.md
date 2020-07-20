---
title: OpenGL投影、图元和存储着色器
tags: [OpenGL]
date: 2020-07-18 21:01:31
updated: 2020-07-18 21:01:31
categories: OpenGL
---

## 投影方式

OpenGL有两种投影方式

|  投影方式   | 说明  | 函数 |
|  ----  | ----  | ----  |
| 正投影  | 相同的物体远近看起来都一样大 | GLFrustum::SetPerspective(float fFov, float fAspect, float fNear, float fFar) |
| 透视投影  | 近大远小 | GLFrustum::SetOrthographic(GLfloat xMin, GLfloat xMax, GLfloat yMin, GLfloat yMax, GLfloat zMin, GLfloat zMax) |

<!-- more -->

{% img /images/post/opengl/two-projection.png 600 %}

```cpp
#include "GLFrustum.h"

// 投影矩阵
GLFrustum              viewFrustum;

// 设置透视投影
viewFrustum.SetPerspective(35.0f, float(w) / float(h), 1.0f, 500.0f);
// 获取投影矩阵
M3DMatrix44f *projectionMatrix = viewFrustum.GetProjectionMatrix();
```

## 图元装配方式

在OpenGL中，相同的顶点可以有不同的装配方式，如下图

{% img /images/post/opengl/primitive-assembly.jpg 800 %}

|  图元   | 描述  |
|  ----  | ----  |
| GL_POINTS  | 每个顶点在屏幕上都是单独点 |
| GL_LINES  | 每⼀对顶点定义⼀个线段 |
| GL_LINE_STRIP  | 一个从第⼀个顶点依次经过每⼀个后续顶点而绘制的线条 |
| GL_LINE_LOOP  | 和GL_LINE_STRIP相同，但是最后⼀个顶点和第⼀个顶点连接起来了 |
| GL_TRIANGLES  | 每3个顶点定义⼀个新的三角形 |
| GL_TRIANGLE_STRIP  | 共⽤一个条带(strip)上的顶点的一组三⻆形 |
| GL_TRIANGLE_FAN  | 以⼀个圆点为中⼼呈扇形排列，共⽤相邻顶点的⼀组三⻆形 |

其中`GL_TRIANGLE_STRIP`和`GL_TRIANGLE_FAN`共享三角形的一条边，在绘制大量三角形的时候，可以节省存储空间和提高性能

如

```cpp
GLfloat vCoast[] = {
    3,3,0,
    0,3,0,
    3,0,0
};
// 三个点
pointBatch.Begin(GL_POINTS, 3);
pointBatch.CopyVertexData3f(vCoast);
pointBatch.End();

// 一条线
lineBatch.Begin(GL_LINES, 3);
lineBatch.CopyVertexData3f(vCoast);
lineBatch.End();

// 闭合三角形线段
lineLoopBatch.Begin(GL_LINE_LOOP, 3);
lineLoopBatch.CopyVertexData3f(vCoast);
lineLoopBatch.End();

// 闭合三角形，可以填充颜色
triangleBatch.Begin(GL_TRIANGLES, 3);
triangleBatch.CopyVertexData3f(vCoast);
triangleBatch.End();
```

## 存储着色器/固定管线着色器

在使用存储着色器之前需要先进行初始化

```cpp
GLShaderManager shaderManager;
shaderManager.InitializeStockShaders();
```

OpenGL内置了很多存储着色器可以使用

### 单元着色器

使⽤场景：绘制默认OpenGL 坐标系(-1,1)下图形。 图形所有片段都会以⼀种颜⾊填充。

```cpp
// 参数1: 存储着⾊器种类-单元着⾊器
// 参数2: 颜⾊值
GLShaderManager::UserStockShader(GLT_SHADER_IDENTITY,
                                 GLfloat vColor[4]);
```

### 平面着色器

使⽤场景：在绘制图形时, 可以应⽤矩阵变换(模型/投影变化)。

```cpp
// 参数1: 存储着⾊器种类-平⾯着⾊器
// 参数2: 允许变化的4*4矩阵
// 参数3: 颜⾊色值
GLShaderManager::UserStockShader(GLT_SHADER_FLAT,
                                 GLfloat mvp[16],
                                 GLfloat vColor[4]);
```

### 上⾊着⾊器

使⽤场景：在绘制图形时, 可以应⽤变换(模型/投影变化)。颜色将会平滑地插入到顶点之间，称为平滑着色。

```cpp
// 参数1: 存储着⾊器种类-上⾊着⾊器
// 参数2: 允许变化的4*4矩阵
GLShaderManager::UserStockShader(GLT_SHADER_SHADED,
                                 GLfloat mvp[16]);
```

### 默认光源着色器

使⽤场景：在绘制图形时, 可以应⽤变换(模型/投影变化)。这种着⾊器会使绘制的图形产生阴影和光照的效果。

```cpp
// 参数1: 存储着⾊器种类-默认光源着⾊器
// 参数2: 模型4*4矩阵
// 参数3: 投影4*4矩阵
// 参数4: 颜⾊值
GLShaderManager::UserStockShader(GLT_SHADER_DEFAULT_LIGHT,
                                 GLfloat mvMatrix[16],
                                 GLfloat pMatrix[16],
                                 GLfloat vColor[4]);
```

### 点光源着⾊器

使⽤场景：在绘制图形时, 可以应用变换(模型/投影变化)。这种着⾊器会使绘制的图形产⽣阴影和光照的效果。它与默认光源着⾊器⾮常类似，区别只是光源位置可能是特定的。

```cpp
// 参数1: 存储着⾊器种类-点光源着⾊器
// 参数2: 模型4*4矩阵
// 参数3: 投影4*4矩阵
// 参数4: 点光源的位置
// 参数5: 漫反射颜⾊值
GLShaderManager::UserStockShader(GLT_SHADER_POINT_LIGHT_DIEF,
                                 GLfloat mvMatrix[16],
                                 GLfloat pMatrix[16],
                                 GLfloat vLightPos[3],
                                 GLfloat vColor[4]);
```

### 纹理替换矩阵着⾊器

使⽤场景：在绘制图形时, 可以应⽤变换(模型/投影变化)。这种着⾊器通过给定的模型视图投影矩阵，使⽤纹理单元来进⾏颜⾊填充。其中每个像素点的颜⾊是从纹理中获取。

```cpp
// 参数1: 存储着⾊器种类-纹理替换矩阵着⾊器
// 参数2: 模型4*4矩阵
// 参数3: 纹理单元
GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_REPLACE,
                                 GLfloat mvMatrix[16],
                                 GLint nTextureUnit);
```

### 纹理调整着⾊器

使⽤场景：在绘制图形时, 可以应⽤变换(模型/投影变化)。这种着⾊器通过给定的模型视图投影矩阵。着⾊器将⼀个基本⾊乘以⼀个取⾃纹理单元nTextureUnit 的纹理，将颜⾊与纹理进⾏颜⾊混合后才填充到⽚段中。

```cpp
// 参数1: 存储着⾊器种类-纹理调整着⾊器
// 参数2: 模型4*4矩阵
// 参数3: 颜⾊值
// 参数4: 纹理单元
GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_MODULATE,
                                 GLfloat mvMatrix[16],
                                 GLfloat vColor[4],
                                 GLint nTextureUnit);
```

### 纹理光源着⾊器

使⽤用场景：在绘制图形时, 可以应⽤变换(模型/投影变化)。这种着⾊器通过给定的模型视图投影矩阵，着⾊器将⼀个纹理通过漫反射照明计算进⾏调整(相乘)。

```cpp
// 参数1: 存储着⾊器种类-纹理光源着⾊器
// 参数2: 模型4*4矩阵
// 参数3: 投影4*4矩阵
// 参数4: 点光源位置
// 参数5: 颜⾊值
// 参数6: 纹理单元
GLShaderManager::UserStockShader(GLT_SHADER_TEXTURE_POINT_LIGHT_DIEF,
                                 GLfloat mvMatrix[16],
                                 GLfloat pMatrix[16],
                                 GLfloat vLightPos[3],
                                 GLfloat vBaseColor[4],
                                 GLint nTextureUnit);
```

## 综合案例

图元绘制完整代码见[这里](https://github.com/zhengbomo/OpenGLDemo/tree/master/003--OpenGL%E5%9B%BE%E5%85%83%E7%BB%98%E5%88%B6(%E7%BB%BC%E5%90%88))
