---
title: Xcode动态调试第三方App
tags: [iOS]
date: 2020-03-15 20:22:10
updated: 2020-03-15 20:22:10
categories: iOS
---

之前我们知道，我们可以使用lldb调试手机上的App，而Xcode就是用的lldb进行调试的，而且功能非常强大，例如查看UI层级结构，查看调用栈，打断点，lldb只能提示等等，使用起来比直接用lldb调试会方便很多，这里记录一下如何在XCode调试第三方App

<!-- more -->

## 直接启动调试

1. 我们先拿到`脱壳`的第三方App（`WeChat`），进行`重签名`，得到`WeChat.app`

    有很多种脱壳的方式，笔者用的是`iOS13`，用的是`CrackerXI+`工具

2. 新建一个新的工程，证书与重签名的一致，修改bundleId为`com.tencent.xin`，运行到手机上
3. 把`WeChat.app`放到工程根目录，在工程添加`Run Script`

   ```sh
   # 运行之前换成第三方app，达到偷梁换柱的目的
   cp -rf "${SRCROOT}/WeChat.app" "${BUILT_PRODUCTS_DIR}/"
   ```

4. 接下来就跟我们平常调试应用一样，只是没有源码可以看

### 添加断点

通过Hopper查看方法的地址为: `0x0000000102a9602c`

```txt
-[WCAccountLoginControlLogic onFirstViewLogin]:
0000000102a9602c         stp        x22, x21, [sp, #-0x30]!
0000000102a96030         stp        x20, x19, [sp, #0x10]
0000000102a96034         stp        x29, x30, [sp, #0x20]
```

打断点

```sh
# 1. 查看ASLR为：0x0000000000558000 = 0x0000000100558000 - 0x0000000100000000
(lldb) image list | grep WeChat
[  0] 7195B97E-9078-3119-9110-8BDA959283F0 0x0000000100558000 /Users/wendy/Library/Developer/Xcode/DerivedData/Test-haevfjompsameldsewkriqunrgfe/Build/Products/Debug-iphoneos/WeChat.app/WeChat

# 2. 所以方法的内存地址为: 0x0000000102fee02c = 0x0000000000558000 + 0x0000000102a9602c
(lldb) p/x 0x0000000000558000 + 0x0000000102a9602c
(long) $27 = 0x0000000102fee02c

# 3. 添加地址断点
(lldb) breakpoint set -a 0x0000000102fee02c
Breakpoint 5: where = WeChat`___lldb_unnamed_symbol137105$$WeChat, address = 0x0000000102fee02c

# 4. 添加符号断点（失败）
(lldb) breakpoint set -n "-[WCAccountLoginControlLogic onFirstViewLogin]"
Breakpoint 8: no locations (pending).
WARNING:  Unable to resolve breakpoint to any actual locations.
```

断点只能通过地址添加，并不能解析符号，另外Xcode也不能解析堆栈信息，我们可以用`restore-symbol`恢复符号表

### 恢复符号表

为了方便调试，打断点，查看堆栈调用信息，我们可以使用[`restore-symbol`](https://github.com/tobefuturer/restore-symbol)恢复符号表

使用方法，网站上很详细了

```sh
# 下载
git clone --recursive https://github.com/tobefuturer/restore-symbol.git
# 编译
cd restore-symbol && make

# 运行（恢复OC符号表）
./restore-symbol /pathto/origin_mach_o_file -o /pathto/mach_o_with_symbol
```

恢复block的符号表需要借助IDA导出映射关系，具体见网站说明，恢复符号表后需要`重签名`

### 打符号断点

![symbol breakpoint](/images/post/xcode-symbol-breakpoint.png)

### UI 层级

![uiview hierarchy](/images/post/xcode-ui-hierarchy.png)

## 远程附加调试

先安装第三方App(`脱壳`)，然后新建一个新工程，运行

在`XCode` -> `DEBGU` -> `Attach to Process`，然后选择对应的进程附加

![attach to process](/images/post/xcode-attachto-process.png)

用起来和直接调试是一样

![attach debug](/images/post/xcode-attach-debug.png)

> 附加调试有时候会很慢
