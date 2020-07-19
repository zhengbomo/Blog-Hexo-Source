---
title: OpenGL正方形键位控制
tags: [OpenGL]
date: 2020-07-12 19:40:12
updated: 2020-07-12 19:40:12
categories: OpenGL
---

上一篇完成了[三角形的绘制](/2020-07-11/opengl-triangle-demo/)，今天来添加一个变换控制，通过键盘的↑↓←→控制图形的移动

<!-- more -->

## 画正方形

基于原来的工程，把三角形改为正方形，`GL_TRIANGLES`修改为`GL_TRIANGLE_FAN`

```cpp
// 正方形边长
GLfloat blockSize = 0.2f;
// 正方形的4个点坐标
GLfloat vVerts[] = {
        -blockSize,-blockSize,0.0f,
        blockSize,-blockSize,0.0f,
        blockSize,blockSize,0.0f,
        -blockSize,blockSize,0.0f
};

...

// 三角形顶点改为正方形，图元装配改为GL_TRIANGLE_FAN
void setupRC() {
    ...

    // 修改为GL_TRIANGLE_FAN ，4个顶点构成四边形
    triangleBatch.Begin(GL_TRIANGLE_FAN, 4);
    triangleBatch.CopyVertexData3f(vVerts);
    triangleBatch.End();
}
```

{% img /images/post/opengl/square-demo.png 600 绘制正方形 %}

## 平移变换

### 修改坐标

平移变换可以直接修改正方形的4个顶点，然后重新刷新

```cpp
// x轴步长
GLfloat xStepSize = 0.025f;
// y轴步长
GLfloat yStepSize = 0.025f;

// 重新生成顶点
GLfloat vVerts[] = {
    -blockSize + xStepSize, -blockSize + yStepSize, 0.0f,
    blockSize + xStepSize, -blockSize + yStepSize, 0.0f,
    blockSize + xStepSize, blockSize + yStepSize, 0.0f,
    -blockSize + xStepSize, blockSize + yStepSize, 0.0f
};

// 新的顶点重新设置到批次类中
triangleBatch.CopyVertexData3f(vVerts);

// 触发重新绘制
glutPostRedisplay();
```

### 矩阵变换

上面直接通过修改坐标的方式过于麻烦，对于复杂的图形做变换会非常麻烦，而推荐使用`矩阵变换`，基于`平面着色器`，可以作用于任何的图形，不需要手动计算坐标，不仅可以做平移，还能做旋转缩放等

`renderScene`操作流程

1. 清理特定缓存区
2. 根据平移距离生成`平移矩阵`
3. 如果有多个矩阵变换，通过`叉乘`得到最终矩阵
4. 将`矩阵结果`交给存储着色器（`平面着色器`）中绘制

```cpp
// x轴平移距离
GLfloat xPos = 0.1f;
// y轴平移距离
GLfloat yPos = 0.1f;
// 旋转(旋转5度)
GLfloat rotate = 5.0f;


void renderScene(void) {
    // 清空缓冲区
    glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT|GL_STENCIL_BUFFER_BIT);

    // 定义2个矩阵（平移矩阵，旋转矩阵）
    M3DMatrix44f mTransfromMatrix, mRotationMartix;

    // 平移(xPos, yPos)
    m3dTranslationMatrix44(mTransfromMatrix, xPos, yPos, 0.0f);

    // 旋转(rotate)
    m3dRotationMatrix44(mRotationMartix, m3dDegToRad(rotate), 0.0f, 0.0f, 1.0f);

    // 将旋转和移动的矩阵结果 合并到mFinalTransform （矩阵相乘）
    M3DMatrix44f mFinalTransform;
    m3dMatrixMultiply44(mFinalTransform, mTransfromMatrix, mRotationMartix);

    // 正方形填充颜色
    GLfloat vRed[] = {1.0f,0.0f,0.0f,0.0f};

    //将矩阵结果 提交给固定着色器（平面着色器）中绘制
    shaderManager.UseStockShader(GLT_SHADER_FLAT, mFinalTransform, vRed);

    // 提交顶点到着色器绘制
    triangleBatch.Draw();

    //执行交换缓存区
    glutSwapBuffers();
}
```

## 键盘控制

GLUT提供了键盘监听事件

```cpp
// 注册特殊函数
glutSpecialFunc(onSpecialKeys);
```

实现上下左右键的监听

```cpp
void SpecialKeys(int key, int x, int y){
    if (key == GLUT_KEY_UP) {
        yPos += stepSize;
    }
    if (key == GLUT_KEY_DOWN) {
        yPos -= stepSize;
    }
    if (key == GLUT_KEY_LEFT) {
        xPos -= stepSize;
    }
    if (key == GLUT_KEY_RIGHT) {
        xPos += stepSize;
    }

    // 边缘检测
    if (xPos < (-1.0f + blockSize)) {
        xPos = -1.0f + blockSize;
    }
    if (xPos > (1.0f - blockSize)) {
        xPos = 1.0f - blockSize;
    }
    if (yPos < (-1.0f + blockSize)) {
        yPos = -1.0f + blockSize;
    }
    if (yPos > (1.0f - blockSize)) {
        yPos = 1.0f - blockSize;
    }

    // 重新绘制
    glutPostRedisplay();
}
```

{% img /images/post/opengl/square-demo.gif 600 %}

完整代码见[这里](https://github.com/zhengbomo/OpenGLDemo/tree/master/002--%E9%94%AE%E7%9B%98%E6%8E%A7%E5%88%B6%E6%AD%A3%E6%96%B9%E5%BD%A2)