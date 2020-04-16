---
title: iOS安全防护
tags: [iOS, 逆向]
date: 2020-04-14 22:35:28
updated: 2020-04-14 22:35:28
categories: iOS
---

对于安全性要求高的App，需要添加逆向成本，较少被破解和攻击的风险，防护的方式主要有`越狱检测`, `抓包检测`, `防反编译`, `防重签名`, `防hook`, `防动态调试`

<!-- more -->

## 越狱检测

```objc
+ (BOOL)isJailbroken {
    // 检查是否存在越狱常用文件
    NSArray *jailFilePaths = @[@"/Applications/Cydia.app",
                               @"/Library/MobileSubstrate/MobileSubstrate.dylib",
                               @"/bin/bash",
                               @"/usr/sbin/sshd",
                               @"/etc/apt"];
    for (NSString *filePath in jailFilePaths) {
        if ([[NSFileManager defaultManager] fileExistsAtPath:filePath]) {
            return YES;
        }
    }

    // 检查是否安装了越狱工具Cydia
    if([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"cydia://package/com.example.package"]]){
        return YES;
    }

    // 检查是否有权限读取系统应用列表
    if ([[NSFileManager defaultManager] fileExistsAtPath:@"/User/Applications/"]){
        NSArray *applist = [[NSFileManager defaultManager] contentsOfDirectoryAtPath:@"/User/Applications/"
                                                                               error:nil];
        NSLog(@"applist = %@",applist);
        return YES;
    }

    //  检测当前程序运行的环境变量
    char *env = getenv("DYLD_INSERT_LIBRARIES");
    if (env != NULL) {
        return YES;
    }

    return NO;
}
```

当然，上面方法很容易被hook，我们可以把它拆成多个方法，并且字符串加密处理，放到C方法中，增加hook的成本

## 防抓包

* 代理检测
* SSLPinning：校验

## 防反编译

这里主要是代码混淆，不做展开

* 类名方法名混淆，OC的话可以用宏占位和宏替换来做
* llvm编译器混淆

## 防止重签名

我们都知道，App在打包签名后在app里面会带上`embedded.mobileprovision`，系统会通过该文件校验应用是否合法，这个文件就是我们打包用的文件，我们我们可以在代码中校验该文件是不是我们自己的，如果不是，则退出程序（AppStore下载的包没有`embedded.mobileprovision`）

```objc
/// 判断签名
/// <string>9PCXXXXK5A.*</string>
/// <string>9PCXXXXK5A.com.bomo.demo</string>
void checkCodesign(NSString *identifier){
    // 描述文件路径
    NSString *embeddedPath = [[NSBundle mainBundle] pathForResource:@"embedded" ofType:@"mobileprovision"];
    if ([NSFileManager.defaultManager fileExistsAtPath:embeddedPath]) {
        NSString *embeddedProvisioning = [NSString stringWithContentsOfFile:embeddedPath encoding:NSASCIIStringEncoding error:nil];
        NSArray *embeddedProvisioningLines = [embeddedProvisioning componentsSeparatedByCharactersInSet:[NSCharacterSet newlineCharacterSet]];

        for (int i = 0; i < embeddedProvisioningLines.count; i++) {
            if ([embeddedProvisioningLines[i] rangeOfString:@"application-identifier"].location != NSNotFound) {
                NSString *value = embeddedProvisioningLines[i + 1];
                NSInteger start = [value rangeOfString:@"<string>"].location;
                NSInteger end = [value rangeOfString:@"</string>"].location;

                if (start != NSNotFound && end != NSNotFound) {
                    NSString *applicationIdentifier = [value substringWithRange:NSMakeRange(start + 8, end - start - 8)];
                    // <string>9PCXXXXK5A.*</string>
                    // <string>9PCXXXXK5A.com.bomo.demo</string>

                    // 对比签名ID
                    if (![applicationIdentifier isEqual:identifier]) {
                        // exit
                        asm(
                            "mov X0,#0\n"
                            "mov w16,#1\n"
                            "svc #0x80"
                            );
                    }
                }
            }
        }
    } else {
        // AppStore的包没有mobileprovision文件
    }
}
```

<!-- # https://blog.csdn.net/quwenjie/article/details/80353384 -->

## 防动态调试

### 使用`ptrace`函数反调试

`debugserver`之所以可以调试APP, 是依赖一个系统函数`ptrace`(process trace 进程跟踪). 此函数提供了一个进程监听控制另外一个进程, 并且可以检查被控制进程的内容和寄存器里面的数据. 可以用来实现断电调试和系统调用跟踪. iOS中没有提供此函数的头文件, 但不是私有API.

`ptrace`函数在iOS项目中不能找到，在MacOS工程可以引用到，我们把需要用到的函数声明搬过来

```c
/**
 * request: 要做的事情
 * pid: 要监听/操作的id
 * addr: 为request代表的操作提供的地址
 */
int ptrace(int _request, pid_t _pid, caddr_t _addr, int _data);

#define PT_TRACE_ME 0   /* child declares it's being traced */
#define PT_READ_I   1   /* read word in child's I space */
#define PT_READ_D   2   /* read word in child's D space */
#define PT_READ_U   3   /* read word in child's user structure */
#define PT_WRITE_I  4   /* write word in child's I space */
#define PT_WRITE_D  5   /* write word in child's D space */
#define PT_WRITE_U  6   /* write word in child's user structure */
#define PT_CONTINUE 7   /* continue the child */
#define PT_KILL     8   /* kill the child process */
#define PT_STEP     9   /* single step the child */
#define PT_ATTACH   ePtAttachDeprecated /* trace some running process */
#define PT_DETACH   11  /* stop tracing a process */
#define PT_SIGEXC   12  /* signals as exceptions for current_proc */
#define PT_THUPDATE 13  /* signal for thread# */
#define PT_ATTACHEXC    14  /* attach to running process with signal exception */
#define PT_FORCEQUOTA   30  /* Enforce quota for root */
#define PT_DENY_ATTACH  31
#define PT_FIRSTMACH    32  /* for machine-specific requests */
```

找地方执行，可以在load方法

```objc
+ (void)load {
    // PT_DENY_ATTACH 表示拒绝调试
    ptrace(PT_DENY_ATTACH, getpid(), 0, 0);
}
```

当我打开debugserver的时候会失败(`Segmentation fault: 11`)

```sh
root# debugserver 127.0.0.1:3333 -a Test
debugserver-@(#)PROGRAM:LLDB  PROJECT:lldb-900.3.98
 for arm64.
Attaching to process Test...
Segmentation fault: 11
laboshi:~ root#
```

### 使用`sysctl`函数反调试

使用`sysctl`函数可以判断当前程序是否正在被调试，需要隔一段时间检测一下

```c
#import <sys/sysctl.h>

bool isDebuging() {
    // 控制码
    int name[4];                // 里面放字节码, 查询信息
    name[0] = CTL_KERN;         // 内核
    name[1] = KERN_PROC;        // 查询进程
    name[2] = KERN_PROC_PID;    // 通过id查询, 传递的参数是进程id
    name[3] = getpid();         // 拿到当前进程id

    struct kinfo_proc info;     // 结束进程查询结果的结构体
    size_t info_size = sizeof(info);    // 结构体的大小

    int error = sysctl(name, sizeof(name)/sizeof(*name), &info, &info_size, 0, 0);

    if (!error) {
        // p_flag 的值转换为二进制, 假如从低位到高位第12位的值为1(0x800), 则正在被调试
        if (info.kp_proc.p_flag & P_TRACED) {
            return true;
        } else {
            return false;
        }
    }
    return false;
}
```

### 反反调试

上面反调试方法都是C语言的方法，而我们知道[`fishhook`](https://github.com/facebook/fishhook)可以 hook C方法，所以上面两个方法可以被fishhook替换掉

这时候我们就需要`让系统的C方法不被hook`，我们可以在别人hook之前换成我们自己的实现，然后别人再hook的时候就只是hook我们替换过的实现了，如何`确保我们的hook在别人之前调用`呢

我们知道，dyld加载App的时候，动态库是先加载的，而动态库的加载顺序是根据MachO文件描述的顺序（XCode中编译的顺序一样，也就是Frameworks,Libraries,and Embedded Content配置的顺序），我们可以用一个`防护的动态库`让我们的动态库先执行

当然如果MachO文件的动态链接库的顺序被改变了，还是会被别人先hook，这个成本就比较高了

## 反hook

对于OC的方法的hook通常是使用runtime的方法交换来实现`method_exchangeImplementations`，所以我们确保这个方法是安全的，就能很大程度上降低OC方法被hook

由于dyld加载程序时候，对于外部符号（例如系统函数）是`lazybind`加载的，编译的时候并不是绑定真实的地址，而是在运行时动态绑定的，所以`fishhook`可以hook系统方法

我们可以先把`method_exchangeImplementations`先换成我们的函数，然后别人在交换该方法的时候，就无法拿到原本的实现了

如何让我们的hook先调用呢

> dyld在加载程序的时候，会先加载动态库，并且是按照MachO文件存储的顺序加载（也就是Xcode链接库的顺序），所以我们可以把我们的hook代码放到动态库放到最前面，就可以然后在load方法交换方法

当然，如果MachO文件的动态库链接顺序也被修改了，那么就没办法了，这时候可以通过一些逻辑判断来增加hook难度，例如如果调用次数多了，就退出程序

> 上面只做了`method_exchangeImplementations`方法的防护，还有其他一些潜在的危险方法也需要做防护，例如：`method_setImplementation`和`method_getImplementation`，通常我们没有用到这两个方法，如果没有用到，就直接替换掉

另外由于程序库内部的`C方法`比较难被hook，对于一些敏感的方法可以放到C方法中（在命名也做一些混淆处理）

## 小结

安全防护之后更好，没有最好，我们只能增加攻击者的成本，增加逆向难度。
