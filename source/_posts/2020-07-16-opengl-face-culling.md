---
title: OpenGL正背面剔除和深度测试
tags: [OpenGL]
date: 2020-07-19 10:29:52
updated:
categories: OpenGL
---

## 背景

当我们在绘制3D图形的时候，由于观察者的角度问题，3D图形拥有多个面，假设所有的面是不透明的，在前面的面是可见的，而背后的面是不可见的，对于不可见的部分，不应该渲染出来，并且出于性能的考虑，应该丢弃（如下面正方体有6个面，而我们能看到的只有三个面），对于下面正方体，面比较少，影响不大，但对于一些复杂的3D图形，影响就比较大了，这种问题称为`隐藏面消除/找出可见面`（Hidden surface elimination/Visible surface detemination）

<!-- more -->

{% img /images/post/opengl/cube.png 200 正方体 %}

## 解决方案

### 油画算法

对于2D的图形，通常采用的是油画算法，远的物体先绘制，近的物体覆盖远的物体

{% img /images/post/opengl/painter-draw.png 600 油画算法 %}

对于3D图形，由于有深度的影响，无法简单分辨物体的远近，如下图，相互交叉的三角形，油画算法将无法处理

{% img /images/post/opengl/triangle-composition.png 300 多个三角形交叉叠加 %}

### 正背面剔除（Face Culling）

在OpenGL中所有的面都是三角形组成，而所有的平面都有两个面（`正面`和`背面`），我们在一个时刻只能看到一个正面。而看不到的背面，OpenGL会检查所有正面朝向观察者的面，并渲染它们，从而丢弃背面朝向的⾯面

在OpenGL规定

`正面`: 按照逆时针顶点连接顺序的三角形面
`背面`: 按照顺时针顶点连接顺序的三⻆形面

{% img /images/post/opengl/cube-face-culling.png 400 %}

从图中可以看出

左边三角形面相对于观察者是顺时针，所以面向观察者的是背面，右边的三角形面是逆时针，所以面向观察者的是正面，所有右边的三角形面被渲染，而左边的三角形面不会被渲染，
如果观察者在左边，则情况会反过来，左边的三角形面被渲染，而右边的三角形面被丢弃。三角形的正面还是背面，是根据观察者的观察方向而变动的。

#### 案例

我们通过画一个甜甜圈来看下开启和关闭正背面剔除的区别

```cpp
#include "GLTools.h"
#include "GLMatrixStack.h"
#include "GLFrame.h"
#include "GLFrustum.h"
#include "GLGeometryTransform.h"

#include <math.h>
#ifdef __APPLE__
#include <glut/glut.h>
#else
#define FREEGLUT_STATIC
#include <GL/glut.h>
#endif

////设置角色帧，作为相机
GLFrame             viewFrame;
//使用GLFrustum类来设置透视投影
GLFrustum           viewFrustum;
GLTriangleBatch     torusBatch;
GLMatrixStack       modelViewMatix;
GLMatrixStack       projectionMatrix;
GLGeometryTransform transformPipeline;
GLShaderManager     shaderManager;

// 标记：是否背面剔除
int iCull = 1;

// 渲染场景
void renderScene() {
    // 1.清除窗口和深度缓冲区
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

    // 开启/关闭正背面剔除功能
    if (iCull) {
        glEnable(GL_CULL_FACE);
        glFrontFace(GL_CCW);
        glCullFace(GL_BACK);
    } else {
        glDisable(GL_CULL_FACE);
    }

    // 2.把摄像机矩阵压入模型矩阵中
    modelViewMatix.PushMatrix(viewFrame);

    // 3.设置绘图颜色
    GLfloat vRed[] = { 1.0f, 0.0f, 0.0f, 1.0f };

    // 使用默认光源着色器
    // 通过光源、阴影效果跟提现立体效果
    // 参数1：GLT_SHADER_DEFAULT_LIGHT 默认光源着色器
    // 参数2：模型视图矩阵
    // 参数3：投影矩阵
    // 参数4：基本颜色值
    shaderManager.UseStockShader(GLT_SHADER_DEFAULT_LIGHT, transformPipeline.GetModelViewMatrix(), transformPipeline.GetProjectionMatrix(), vRed);

    // 5.绘制
    torusBatch.Draw();

    // 6.出栈 绘制完成恢复
    modelViewMatix.PopMatrix();

    // 7.交换缓存区
    glutSwapBuffers();
}

void setupRC() {
    //1.设置背景颜色
    glClearColor(0.3f, 0.3f, 0.3f, 1.0f );
    //2.初始化着色器管理器
    shaderManager.InitializeStockShaders();
    //3.将相机向后移动7个单元：肉眼到物体之间的距离
    viewFrame.MoveForward(5);
    //4.创建一个甜甜圈
    //void gltMakeTorus(GLTriangleBatch& torusBatch, GLfloat majorRadius, GLfloat minorRadius, GLint numMajor, GLint numMinor);
    //参数1：GLTriangleBatch 容器帮助类
    //参数2：外边缘半径
    //参数3：内边缘半径
    //参数4、5：主半径和从半径的细分单元数量
    gltMakeTorus(torusBatch, 1.0f, 0.3f, 52, 26);
    //5.点的大小(方便点填充时,肉眼观察)
    glPointSize(1.0f);
}

// 通过键盘↑↓←→修改旋转方向，控制Camera的移动，从而改变视口
void specialKeys(int key, int x, int y) {
    //1.判断方向
    if(key == GLUT_KEY_UP)
        //2.根据方向调整观察者位置
        viewFrame.RotateWorld(m3dDegToRad(-5.0), 1.0f, 0.0f, 0.0f);
    if(key == GLUT_KEY_DOWN)
        viewFrame.RotateWorld(m3dDegToRad(5.0), 1.0f, 0.0f, 0.0f);

    if(key == GLUT_KEY_LEFT)
        viewFrame.RotateWorld(m3dDegToRad(-5.0), 0.0f, 1.0f, 0.0f);

    if(key == GLUT_KEY_RIGHT)
        viewFrame.RotateWorld(m3dDegToRad(5.0), 0.0f, 1.0f, 0.0f);

    // 重新刷新
    glutPostRedisplay();
}

/// 窗口改变
void changeSize(int w, int h) {
    //1.防止h变为0
    if(h == 0) {
        h = 1;
    }

    //2.设置视口窗口尺寸
    glViewport(0, 0, w, h);

    //3.setPerspective函数的参数是一个从顶点方向看去的视场角度（用角度值表示）
    // 设置透视模式，初始化其透视矩阵
    viewFrustum.SetPerspective(35.0f, float(w)/float(h), 1.0f, 100.0f);

    //4.把透视矩阵加载到透视矩阵对阵中
    projectionMatrix.LoadMatrix(viewFrustum.GetProjectionMatrix());

    //5.初始化渲染管线
    transformPipeline.SetMatrixStacks(modelViewMatix, projectionMatrix);
}

int main(int argc, char* argv[]) {
    gltSetWorkingDirectory(argv[0]);

    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA | GLUT_DEPTH | GLUT_STENCIL);
    glutInitWindowSize(800, 600);
    glutCreateWindow("Geometry Test Program");
    glutReshapeFunc(changeSize);
    glutSpecialFunc(specialKeys);
    glutDisplayFunc(renderScene);

    GLenum err = glewInit();
    if (GLEW_OK != err) {
        fprintf(stderr, "GLEW Error: %s\n", glewGetErrorString(err));
        return 1;
    }

    setupRC();

    glutMainLoop();
    return 0;
}
```

如果不开启正背面剔除，背面的黑色面也会显示出来

{% img /images/post/opengl/face-culling.png 600 %}

在旋转到中间的过程时遇到下面情况

{% img /images/post/opengl/face-culling-deep-problem.png 300 %}

### 深度测试

出现上面缺口的原因是由于，在同一个位置，出现了两个正面，甜甜圈的内环和外环，都有一个正面，OpenGL不知道显示哪一个面，就出现了上面的问题，把内环的面显示出来了，覆盖了外环的面，解决问题之前，先来理解几个概念

#### 深度

深度是指OpenGL坐标系中，像素点的Z坐标距观察者的距离

* 如果观察者在Z轴的正方向，Z值越大则越靠近观察者
* 如果观察者在Z轴的负方向，Z值越小则越靠近观察者

#### 深度缓冲区（Depth Buffer）

深度缓存区是指一块专门内存区域，存储在显存中，用于存储屏幕上所绘制图形的每个像素点的深度值，深度越大，离观察者越远

#### 深度测试

在绘制物体的时候，像素点的深度会和之前的深度值做比较，如果 新值 > 旧值，则会被丢弃不绘制，反之，新值会更新到`深度缓冲区`，新的颜色值同样会更新到`颜色缓冲区`，这个过程称为深度测试，确保所有绘制的点都是距离观察者最近的

开启和关闭深度测试

```cpp
// 在RenderScene清空完缓冲区后设置

// 开启深度测试
glEnable(GL_DEPTH_TEST);
// 关闭深度测试
glDisable(GL_DEPTH_TEST);
```

{% img /images/post/opengl/face-culling-deep.png 600 %}

相比于`正背面剔除`，`深度测试`不仅可以解决正背面显示问题，还能解决上面隐藏面消除的问题

#### ZFighting闪烁

深度测试很好的解决了3D图形层次显示的问题，但是还存在一个误差的问题，开启深度测试后，由于深度缓冲区精度有限制，导致深度值相差很小时，OpenGL出现无法判断的情况，导致出现画面交错闪现的现象，这个问题成为`ZFighting闪烁`，如下图

{% img /images/post/opengl/z-fighting.png 600 %}

其问题产生的主要原因是由于图形靠的太近，导致无法区分出图层先后次序，针对该问题，OpenGL提供了一种多边形偏移（Polygon Offset）方案

```cpp
// 开启多边形偏移
glEnable(GL_POLYGON_OFFSET_FILL);
```

|  多边形偏移枚举值   | 对应光栅化模式  |
|  ----  | ----  |
| GL_POLYGON_OFFSET_POINT  | GL_POINT |
| GL_POLYGON_OFFSET_LINE  | GL_LINE |
| GL_POLYGON_OFFSET_FILL  | GL_FILL |

使用步骤

```cpp
// 1. 开启多边形偏移
glEnable(GL_POLYGON_OFFSET_FILL);
// 2. 指定偏移量glPolygonOffset (GLfloat factor, GLfloat units);，参数一般填 -1 和 -1
glPolygonOffset (GLfloat factor, GLfloat units);
// 3. 关闭多边形偏移
glDisable(GL_POLYGON_OFFSET_FILL)
```

预防ZFighting闪烁

* 避免两个物体靠的太近：在绘制时，插入一个小偏移
* 将近裁剪面（设置透视投影时设置）设置的离观察者远一些：提高裁剪范围内的精确度
* 使用更高位数的深度缓冲区：提高深度缓冲区的精确度
