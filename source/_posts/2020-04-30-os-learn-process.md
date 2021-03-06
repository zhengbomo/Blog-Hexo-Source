---
title: 《操作系统》进程管理（二）
tags: [操作系统]
date: 2020-04-30 23:09:04
updated: 2020-04-30 23:09:04
categories: 操作系统
---

## 概念

程序：是静态的，存放在磁盘的一系列可执行的指令集合
进程：是动态的，程序加载到内存运行的一次执行过程

<!-- more -->

## 组成

* `进程控制块`(PCB): 操作系统用于管理进程所需要的相关信息，通常包含
  * 进程描述信息
    * 进程标识符PID
    * 用户标识符UID
  * 资源相关信息
    * I/O设备使用
    * 文件使用情况
    * 内存分配情况
  * 运行相关信息
    * CPU，磁盘使用，网络流量等）
    * 进程状态信息（就绪态/阻塞态/运行态）
  * 处理机相关信息
    * PSW，PC等寄存器等信息（用于进程切换保留现场）

* `程序段`：用于CPU执行的指令集合，只读
* `数据段`：用户存放程序运行过程中的产生的数据，可读写

## 特性

* 动态性
* 并发性
* 独立性
* 异步性
* 结构性

## 状态

* `就绪态`：具备运行的条件，等待CPU运行
* `运行态`：正在被CPU执行
* `阻塞态`：请求等待某个事件发生

* `创建态`：操作系统为新进程分配资源，创建PCB
* `终止态`：操作系统回收进程资源，撤销PCB

状态转换

* `就绪态->运行态`：进程被调度
* `运行态->就绪态`：CPU运行时间片到，或CPU被其他更高优先级的进程抢占了
* `运行态->阻塞态`：等待某个事件发生（主动行为）
* `阻塞态->就绪态`：等待的事件被触发（被动行为）

* `创建态->就绪态`：系统完成创建进程的相关工作
* `运行态->终止态`：进程运行结束，或运行遇到不可修复的错误

{% img /images/post/os/os-process-state-tran.png 1000 进程状态切换 %}

## 原语

原语的执行具有`原子性`，原子性通过`开中断指令`和`关中断指令`两个特权指令实现原子性

{% img /images/post/os/os-process-atomic.png 600 进程状态切换 %}

如果中断信号在关中断指令和开中断指令之间发生，则会推迟到之后

> 开关中断指令只能在内核进程使用，不能在用户进程使用

## 进程通信

进程的地址空间是相互独立的，一个进程无法直接访问另一个进程的地址空间，操作系统提供了下面三种通信方式

* `共享存储`: 独立于进程之外的空间，两个进程都能互斥访问
* `管道通信`: 其实是在内存中开辟的一个固定大小的缓冲区
  * 管道只能采用半双工通信（一个时间段只能实现单向传输），如果要实现双向通信，需要2个管道
  * 进程访问管道是互斥的
  * 管道必须写满才能被读，必须读完才能被写
* `消息传递`
  * 直接通信方式：每个进程会有一个`消息缓冲队列`，用于接受其他进程发送过来的消息
  * 间接通信方式：消息通过`信箱`集中管理进程的通信

## 线程

进程也有并发的需求，多进程并发，会有性能差，通信麻烦的问题，所以在进程内部引入`线程`

1. 解决进程并发问题
2. 解决进程切换开销大的问题
3. 解决进程通信的问题

`线程`：处理器的调度单位，为进程的执行过程，一个进程可以由多个线程，进程内的线程共享进程的地址空间，资源等

## 线程实现方式

* `用户级线程`: 用单线程模拟多线程，例如消息循环
  * 不支持多核并行
  * 一个线程阻塞会导致所有线程被阻塞
  * 不需要CPU切换内核态核用户态，效率高
* `内核级线程`: 由操作系统内核支持
  * 并发能力强
  * 需要CPU切换内核态核用户态，开销大

## 进程的调度

* 高级调度（作业调度）: 外存 ->  内存
* 中级调度（内存调度）: 内存 ⇌ 外存
* 低级调度（进程调度）: 内存 ⇌ CPU

### 时机

* 主动放弃
  * 进程正常终止
  * 运行过程发生异常而终止
  * 主动阻塞（如I/O等待）
* 被动放弃
  * CPU时间片用完
  * 有更紧急的事情要处理（如I/O中断）
  * 有更高优先级的进程进入就绪队列
* 不能进行线程调度
  * 处理中断过程中
  * 进程在操作系统内核程序临界区中
  * 院子操作过程中

### 调度

* 非抢占方式: 由进程主动放弃处理器, 可能CPU会被一个进程一直占用
* 抢占方式: 可以优先处理更紧急的进程

### 切换

1. 保存当前进程现场（保存寄存器）
2. 恢复即将切换进程现场（恢复寄存器）

### 临界区

`临界资源`：一个时间段只允许一个进程使用的资源，进程互斥访问临界资源
`临界区`：访问临界资源的代码
`内核程序临界区`：访问某种内核数据结构的代码，如进程的就绪队列，过程中进程不能被调度

## 优先级调度算法

* `系统进程`优先级高于`用户进程`
* `前台进程`优先级高于`后台进程`
* 操作系统更偏好`I/O型进程`（可以让I/O进程尽早投入工作，提高资源利用率，系统吞吐量）

优先级分为`静态优先级`和`动态优先级`

* 如果某进程在就绪队列等待了很长时间，则可以适当的提高优先级
* 如果某进程占用处理机运行了很长事件，则可以适当的降低优先级

## 进程同步

由于操作系统的并发性，导致不同进程中业务的执行顺序不可预测，为了满足特地执行流程（例如，先进程A读数据要在进程B写数据之后才能继续），就需要使用进程同步的技术，解决异步的问题

`临界资源`：一个时间段内只允许一个进程使用的资源，进程对于临界资源的访问是互斥的

四个部分

* `进入区`：检查是否可以进入临界区，负责上锁
* `临界区`：访问临界区的代码
* `退出区`：负责解锁
* `剩余区`：其余代码部分

遵循原则

* 空闲进入
* 忙则等待
* 优先等待
* 让权等待，等待的时候让出CPU

进程互斥软件实现方法

* 单标志法
* 双标志先检查法
* 双标志后检查法
* Peterson算法

进程互斥的硬件实现

* 中断屏蔽方法：通过开/关中断指令实现，简单高效，只是用于单处理器，只是用于内核进程
* TestAndSet指令：检查临界区和上锁的操作为原子操作
* Swap指令：与TestAndSet指令类似

## 信号量机制

### 整型信号量

`信号量`：一个变量，可以表示系统某种资源的数量

使用`wait`和`signal`原语对信号量进行操作，也称为`PV操作`

```c
// 初始化整形信号量S，可用资源为1
int S = 1;

// wait原语，相当于进入区
void wait(int S) {
    while(S <= 0) ;     // 资源不够时，进入忙等待
    S = S - 1;          // 占用资源
}

// signal原语，相当于退出区
void signal(int S) {  
    S = S + 1;          // 释放资源
}
```

使用资源

```c
// P操作
wait(S);

// 使用资源

// V操作
signal(S)
```

存在问题：不满足让权等待的原则，会出现CPU空转的情况

### 记录型信号量

解决整形信号量`让权等待`的问题，通过`block`和`wakeup`原语，避免忙等待的情况，提高CPU的利用率

```c
typedef struct {
    int value;
    struct process *L;      // 等待队列
} semaphore;

// 原子操作
void wait(semaphore S) {
    S.value--;
    if (S.value < 0) {
        // 没有空闲的资源， 阻塞当前进程进程，并把进程放到等待队列中
        block(S.L);
    }
}

// 原子操作
void signal(semaphore S) {
    S.value++;
    if (S.value <= 0) {
        // 说明有进程在等待，唤醒队列中的进程
        wakeup(S.L);
    }
}
```

使用信号量解决`互斥问题`

```cpp
semaphore mutex = 1;

// 进程P1
P1() {
    ...
    // P操作
    P(mutex);
    临界区代码
    // V操作
    V(mutex)
    ...
}

// 进程P2
P2() {
...
    P(mutex);
    临界区代码
    V(mutex)
    ...
}
```

使用信号量解决`同步问题`

```c
semaphore S=0;

P1() {
    代码1;
    代码2;
    // 释放资源
    V(S);
    代码3;
}

P2() {
    // 在代码2执行完成后执行
    P(S);
    代码4;
    代码5;
    代码6;
}
```

### 生产者消费者问题

```c
semaphore mutex = 1;        // 互斥信号量
semaphore empty = n;        // 缓冲区大小
semaphore full = 0;         // 产品数量

producer() {
    while(1) {
        // 生产一个产品

        P(empty);       // 消耗一个空闲缓冲区

        P(mutex);
        将产品放入缓冲区
        V(mutex);

        V(full);        // 增加一个产品
    }
}

consumer() {
    while(1) {
        P(full);        // 消耗一个产品

        P(mutex);
        从缓冲区取出一个产品
        V(mutex);

        V(empty)        // 增加一个空闲缓存区

        // 使用产品
    }
}
```

### 读者写者问题

读并发，写互斥

```c
semaphore rw = 1;       // 文件互斥访问
int count = 0;          // 读文件进程个数
semaphore mutex = 1;    // 用户对count互斥
semaphore w = 1;        // 实现当写进程进入的时候，新的读进程进不来，防止写饥饿


writer() {
    while(1) {
        P(w);

        P(rw);
        写文件
        V(rw);

        V(w)
    }
}

reader() {
    while(1) {
        P(w);
        P(mutex);
        if (count == 0) {
            P(rw);
        }
        count++;
        V(mutex);
        V(w);
        读文件

        P(mutex);
        count--;
        if (count == 0) {
            V(rw);              // 没有读操作，释放
        }
        V(mutex);
    }
}
```

### 哲学家进餐问题

```c
semaphore chopstick[5] = {1, 1, 1, 1, 1};
semaphore mutex = 1;
Pi() {
    while(1) {
        P(mutex);
        P(chopstick[i]);        // 拿起左边筷子
        P(chopstick[(i+1)%5]);  // 拿起右边筷子
        V(mutex)

        吃饭

        V(chopstick[i]);        // 放下左边筷子
        V(chopstick[(i+1)%5]);  // 放下右边筷子
    }
}
```

其他问题

* 多生产者消费者问题
* 吸烟者问题

### 管程

信号量机制存在问题：编写程序困难，易出错，容易发生死锁，管程的引入是为了解决该问题
管程：通过封装同步数据的`方法`，由`编译器实现`方法的互斥，从而简单化信号量的使用，对于管程的方法，同一时间只有一个进程能进入方法，如下面`JAVA的管程`

```java
static class monitor {
    private Item buffer[] = new Item[N];
    private int count = 0;

    public synchronized void insert(Item item) {
        // 每次只能有一个线程进入，由编译器实现互斥
    }
}
```

管程其实是利用了封装的思想，把进程同步互斥的逻辑封装到方法里面，通过编译器实现`方法的互斥`，达到线程同步的目的
