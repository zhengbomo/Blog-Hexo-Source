---
title: 合并分离.a和.framework库
date: 2016-04-29 18:50:52
updated: 2016-04-29 18:51:41
categories: iOS
tags: [iOS]   
---

xcode在生成库（.a或.framework）的时候，通常会生成两个版本，一个是用于真机的版本，一个是用于模拟器的版本

* 真机：`armv7`, `armv7s`, `arm64`架构
* 模拟器：`i386`, `x86_64`

<!-- more -->

如果在库和项目在同一个工程中，通常会自动根据当前是模拟器还是真机自动引用相关的库文件

在使用第三方库的时候通常只有一个.a库，这个库既能用于真机调试，又能用于模拟器调试，这个时候我们需要对不同的架构的库进行合并

使用`lipo`对不同架构的库进行合并，在编译的时候会自动识别

## 一、.a库合并与拆分
例如有两个不同架构的库`liba-arm64.a`, `liba-i386.a`
1. 查看库的架构信息
  ```
  lipo -info liba-arm64.a
  input file liba-arm64.a is not a fat file
  Non-fat file: liba-arm64.a is architecture: arm64
  ```
2. 合并两个库
  ```
  lipo -create liba-arm64.a liba-i386.a -output liba.a
  ```
  合并成`liba.a`到当前目录

3. 抽取出`arm64`库
  ```
  lipo liba.a -thin arm64 -output liba-arm64.a
  ```

## 二、.Framework库合并与拆分
.framework库与.a库类似，只是添加了头文件和资源，其实相当于一个目录，所以操作的是里面的库文件，而不是xxx.framework文件

例如有两个不同架构的库
  * `IJKMediaFramework_x86_64.framework`
  * `IJKMediaFramework_arm64.framework`  

1. 查看.framework信息
  ```
  lipo -info IJKMediaFramework_arm64.framework/IJKMediaFramework
  input file IJKMediaFramework_arm64.framework/IJKMediaFramework is not a fat file
  Non-fat file: IJKMediaFramework_arm64.framework/IJKMediaFramework is architecture: arm64
  ```
2. 合并库
  ```
  lipo -create IJKMediaFramework_x86_64.framework/IJKMediaFramework IJKMediaFramework_arm64.framework/IJKMediaFramework -output IJKMediaFramework
  ```

  得到通用的库`IJKMediaFramework`替换到`IJKMediaFramework_x86_64.framework/IJKMediaFramework`，这时候`IJKMediaFramework_x86_64.framework`就是通用framework

3. 抽取出`arm64`库
  ```
  lipo IJKMediaFramework_x86_64.framework/IJKMediaFramework -thin x86_64 -output IJKMediaFramework
  ```

## 三、xcode脚本自动合并库
如果是自己生成的库，有个技巧，xcode生成库的时候自动执行脚本完成合并的操作，需要选择真机和模拟器分别编译一遍

在Build Phases添加Run Script
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/73528845.jpg)
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/37235497.jpg)

```bash
# 编译的时候
if [ "${ACTION}" = "build" ]
then

# 生成通用framework的目录
INSTALL_DIR=${SRCROOT}/Products/${PRODUCT_NAME}.framework

# 需要合并的framework
DEVICE_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphoneos/${PRODUCT_NAME}.framework
SIMULATOR_DIR=${BUILD_ROOT}/${CONFIGURATION}-iphonesimulator/${PRODUCT_NAME}.framework

# 如果已经存在，则删除
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi

# 创建目录
mkdir -p "${INSTALL_DIR}"

# 拷贝Header到目标目录
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"

# 合并framework
lipo -create "${DEVICE_DIR}/${PRODUCT_NAME}" "${SIMULATOR_DIR}/${PRODUCT_NAME}" -output "${INSTALL_DIR}/${PRODUCT_NAME}"

# 编译完成后打开文件夹
#open "${DEVICE_DIR}"
open "${SRCROOT}/Products/${PRODUCT_NAME}.framework"
fi
```

切换到Release模式，分别切换到模拟器和真机编译一次，编译完成后会自动打开输出文件夹，通过`lipo`命令查看
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/39574369.jpg)

完成，接下来可以直接用了
