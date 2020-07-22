---
title: 【OpenGL案例1】绘制一个三角形
tags: [OpenGL]
date: 2020-07-11 21:42:13
updated: 2020-07-11 21:42:13
categories: OpenGL
---

通过使用OpenGL绘制一个三角形，了解OpenGL的一些常用API和绘制流程，这里使用了[GLUT](https://www.opengl.org/resources/libraries/glut/)（OpenGL Utility Toolkit）的API，`GLUT`是OpenGL官方提供的OpenGL工具箱，封装了一些有用的工具，用于辅助操作OpenGL

<!-- more -->

## 工具API

本案例也使用`GLUT`工具进行绘制，使用最基本的固定着色器来绘制，本案例会用到下面几个头文件

|  头文件   | 说明  |
|  ----  | ----  |
| GLShaderManager.h  | 着色器管理类，用于创建和管理着色器，还提供一组内置的存储着色器（固定管线着色器） |
| GLTools.h  | 包含大部分类似C语⾔的独⽴函数 |
| GLUT.h  | OpenGL工具箱 |

## 相关函数

这里会注册两个函数用于绘制

* `renderSence`: 类似于iOS的类似于`drawRect`，每次View需要重新绘制的时候，会被调用
* `changeSize`: 窗口大小变化的时候被调用，通常用于调整glview的视口

定义两个变量用于`管理着色器`和`顶点数据`

```cpp
#include "GLShaderManager.h"
#include "GLTools.h"
#include <GLUT/GLUT.h>

// 定义一个，着色管理器
GLShaderManager shaderManager;
// 简单的批次容器，用于将顶点数据提交给着色器使用
GLBatch triangleBatch;
```

### main

程序启动的时候，我们需要做一些初始化操作

1. 初始化`glutInit`
2. 初始化双缓冲区，颜色模式，深度，模板
3. 设置窗口信息（大小，标题）
4. 注册生命周期函数：`renderSence`和`changeSize`
5. 测试驱动可用性: 通过`glewInit`结果判断
6. 初始化渲染数据`setupRC`
7. 开启事件循环

```cpp
int main(int argc,char *argv[]) {
    //初始化GLUT库,这个函数只是传说命令参数并且初始化glut库
    glutInit(&argc, argv);
    /*
     初始化双缓冲窗口，其中标志GLUT_DOUBLE、GLUT_RGBA、GLUT_DEPTH、GLUT_STENCIL分别指
     双缓冲窗口、RGBA颜色模式、深度测试、模板缓冲区

     --GLUT_DOUBLE`：双缓存窗口，是指绘图命令实际上是离屏缓存区执行的，然后迅速转换成窗口视图，这种方式，经常用来生成动画效果；
     --GLUT_DEPTH`：标志将一个深度缓存区分配为显示的一部分，因此我们能够执行深度测试；
     --GLUT_STENCIL`：确保我们也会有一个可用的模板缓存区。
     深度、模板测试后面会细致讲到
     */
    glutInitDisplayMode(GLUT_DOUBLE|GLUT_RGBA|GLUT_DEPTH|GLUT_STENCIL);

    //GLUT窗口大小、窗口标题
    glutInitWindowSize(800, 600);
    glutCreateWindow("Triangle");

    // 注册窗口改变事件
    glutReshapeFunc(changeSize);
    // 注册显示函数，当需要重新绘制的时候，会调用
    glutDisplayFunc(renderScene);

    // 初始化一个GLEW库，测试是否报错，确保OpenGL API对程序完全可用。
    GLenum status = glewInit();
    if (GLEW_OK != status) {
        printf("GLEW Error:%s\n",glewGetErrorString(status));
        return 1;
    }

    // 准备我们需要渲染的数据
    setupRC();

    // 开启事件循环，相当于iOS的runloop
    glutMainLoop();
    return  0;
}
```

### setupRC

我们要画一个`三角形`，可以在这里做一些准备工作

* 设置清屏颜色
* 初始化着色器
* 初始化顶点数据

```cpp
void setupRC() {
    // 设置清屏颜色（背景颜色，白色）
    glClearColor(1.0f, 1.0f, 1.0f, 1.0f);

    // 没有着色器，在OpenGL 核心框架中是无法进行任何渲染的。这里初始化一个渲染管理器，在renderSence会用到。这里使用固定管线着色器
    shaderManager.InitializeStockShaders();

    // 三角形顶点
    GLfloat vVerts[] = {
        -0.5f,0.0f,0.0f,
        0.5f,0.0f,0.0f,
        0.0f,0.5f,0.0f
    };

    // 将顶点数据传递到三角形批次类中
    triangleBatch.Begin(GL_TRIANGLES, 3);
    triangleBatch.CopyVertexData3f(vVerts);
    triangleBatch.End();
}
```

这里`triangleBatch`在Begin的时候设置了一个图元装配方式为`GL_TRIANGLES`，在OpenGL中，相同的顶点可以有不同的装配方式，如下图

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

### renderSence

准备好数据之后，就可以开始绘制了，核心代码在之前注册的`renderSence`里面

* 清空缓冲区（颜色缓冲区，深度缓冲区，模板缓冲区），避免脏数据
* 使用`着色器`填充颜色
* 批次类将`顶点数据`提交到`着色器`上绘制

```cpp
void RenderScene(void) {
    // 1.清除一个或者一组特定的缓存区，如果后面需要用到这些缓冲区，就需要清空，不然会出现之前使用的脏数据（如深度缓冲区，颜色缓冲区，模板缓冲区等）
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT|GL_STENCIL_BUFFER_BIT);

    // 2.设置一个颜色用来填充三角形
    GLfloat vRed[] = {1.0, 0.0, 0.0, 1.0f};

    // 3. 使用单元着色器，来对图形进行着色，即GLT_SHADER_IDENTITY着色器，这个着色器只是使用指定颜色以默认笛卡尔坐标第在屏幕上渲染几何图形
    shaderManager.UseStockShader(GLT_SHADER_IDENTITY, vRed);

    // 4. 提交顶点数据到着色器，进行绘制
    triangleBatch.Draw();

    // 在开始的设置openGL 窗口的时候，我们指定要一个双缓冲区的渲染环境。这就意味着将在后台缓冲区进行渲染，渲染结束后交换给前台。
    glutSwapBuffers();
}
```

### changeSize

当窗口大小改变的时候，我们需要重新调整视口大小

```cpp
void changeSize(int w,int h) {
    // x,y 参数代表窗口中视图的左下角坐标，而宽度、高度是像素为表示，通常x,y 都是为0
    glViewport(0, 0, w, h);
}
```

## 运行

{% img /images/post/opengl/triangle-demo.png 600 三角形 %}

完整demo在[这里](https://github.com/zhengbomo/OpenGLDemo/tree/master/001--%E5%AE%8C%E6%95%B4%E6%B8%B2%E6%9F%93%E4%B8%89%E8%A7%92%E5%BD%A2)
