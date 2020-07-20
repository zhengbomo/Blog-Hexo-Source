---
title: 【OpenGL案例5】球的自转和公转
tags: [OpenGL]
date: 2020-07-20 16:47:49
updated:
categories: OpenGL
---

通过[案例4](/2020-07-20/opengl-demo4/)我们已经知道如何绘制`球`了，本案例绘制一个非常经典的案例，就是球的自转和公转（小球公转，大球自转）

<!-- more -->

{% img /images/post/opengl/sphere-world.gif 600 %}

为了方便看出自转，这里只画三角形线，不进行填充

## 绘制图形

基于前面的案例4的基础上来绘制

### 绘制地板

```cpp
// 地板
GLBatch                floorBatch;

// 设置地板顶点数据
floorBatch.Begin(GL_LINES, 324);
for(GLfloat x = -20.0; x <= 20.0f; x+= 0.5) {
    floorBatch.Vertex3f(x, -0.55f, 20.0f);
    floorBatch.Vertex3f(x, -0.55f, -20.0f);

    floorBatch.Vertex3f(20.0f, -0.55f, x);
    floorBatch.Vertex3f(-20.0f, -0.55f, x);
}
floorBatch.End();

// 在renderSence绘制地板
shaderManager.UseStockShader(GLT_SHADER_FLAT, transformPipeline.GetModelViewProjectionMatrix(), vGreen);
floorBatch.Draw();
```

### 大球和小球

自转公转小球

```cpp
// 设置大球模型
gltMakeSphere(torusBatch, 0.4f, 20, 40);

// 设置小球模型
gltMakeSphere(sphereBatch, 0.2f, 8, 16);
```

随机小球

```cpp
// 随机球个数
#define NUM_SPHERES 50
// 记录随机球位置
GLFrame spheres[NUM_SPHERES];


// 随机生成位置放置小球
for (int i = 0; i < NUM_SPHERES; i++) {
    //y轴不变，X,Z产生随机值
    GLfloat x = ((GLfloat)((rand() % 400) - 200 ) * 0.1f);
    GLfloat z = ((GLfloat)((rand() % 400) - 200 ) * 0.1f);

    // 在y方向，将球体设置为0.0的位置，这使得它们看起来是飘浮在眼睛的高度
    // 对spheres数组中的每一个顶点，设置顶点数据
    spheres[i].SetOrigin(x, 0.0f, z);
}
```

### renderSence

添加`视图矩阵`到矩阵堆栈

```cpp
// 模型视图矩阵，push单元矩阵
modelViewMatrix.PushMatrix();

// 模型变换
M3DMatrix44f mCamera;
cameraFrame.GetCameraMatrix(mCamera);
// push视图变换
modelViewMatrix.PushMatrix(mCamera);
```

由于地板不需要其他变换，这时候可以绘制地板

```cpp
shaderManager.UseStockShader(GLT_SHADER_FLAT, transformPipeline.GetModelViewProjectionMatrix(), vGreen);
floorBatch.Draw();
```

让大小球显示在观察者前面，这里添加一个`视图变换`

```cpp
// 平移（z轴）让小球显示到观察者前面，
modelViewMatrix.Translate(0.0f, 0.0f, -3.0f);
```

然后绘制大球，这里使用点`点光源着⾊器`，可以看到球的光照效果

```cpp
// 这里添加一个旋转角度(5°)，通过这个变量控制自转
float yRot = 5;

// 定义光源位置
M3DVector4f vLightPos = {0.0f,10.0f,5.0f,1.0f};

// 绘制随机球，这里直接用小球的批次类来绘制
for (int i = 0; i < NUM_SPHERES; i++) {
    modelViewMatrix.PushMatrix();
    modelViewMatrix.MultMatrix(spheres[i]);
    shaderManager.UseStockShader(
                                 GLT_SHADER_POINT_LIGHT_DIFF,
                                 transformPipeline.GetModelViewMatrix(),
                                 transformPipeline.GetProjectionMatrix(),
                                 vLightPos,
                                 vBlue);
    sphereBatch.Draw();
    modelViewMatrix.PopMatrix();
}

// 旋转（用于大球自转）
modelViewMatrix.PushMatrix();
modelViewMatrix.Rotate(yRot, 0.0f, 1.0f, 0.0f);

// 球上的三角形画线
glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
glLineWidth(2f);

// 指定合适的着色器(点光源着色器)
shaderManager.UseStockShader(
                             GLT_SHADER_POINT_LIGHT_DIFF,
                             transformPipeline.GetModelViewMatrix(),
                             transformPipeline.GetProjectionMatrix(),
                             vLightPos,
                             vRed);
// 画球
torusBatch.Draw();
// 弹出旋转矩阵（因为旋转只用于大球）
modelViewMatrix.PopMatrix();
```

接下来画`小球`，先旋转，再平移（先旋转到固定的角度，再移到外面）

```cpp
// 旋转（公转）
modelViewMatrix.Rotate(yRot * -2.0f, 0.0f, 1.0f, 0.0f);
// 平移
modelViewMatrix.Translate(0.8f, 0.0f, 0.0f);
shaderManager.UseStockShader(
                             GLT_SHADER_POINT_LIGHT_DIFF,
                             transformPipeline.GetModelViewMatrix(),
                             transformPipeline.GetProjectionMatrix(),
                             vLightPos,
                             vBlue);
// 画小球
sphereBatch.Draw();

// 前面一共push了三个矩阵，用完后pop
modelViewMatrix.PopMatrix();
modelViewMatrix.PopMatrix();
modelViewMatrix.PopMatrix();
```

> 注意，这里的小球没有自转，只是跟随者Y轴旋转

### 定时器

接下来是控制刷新，主要是修改`yRot`，就能控制自转和公转了，这里通过`CStopWatch`获取一个时间间隔，来生成一个动态的值

```cpp
static CStopWatch rotTimer;
// 可以通过流失的时间获取一个动态的值
float yRot = rotTimer.GetElapsedSeconds() * 60.0f;
```

完整代码见[这里](https://github.com/zhengbomo/OpenGLDemo/tree/master/005--%E5%B0%8F%E7%90%83%E8%87%AA%E8%BD%AC%E5%85%AC%E8%BD%AC)
