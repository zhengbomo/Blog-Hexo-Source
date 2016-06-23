---
title: Python多线程学习笔记
categories: Python
tags:
  - 多线程
  - Python
date: 2016-06-14 23:42:53
updated: 2016-06-14 23:42:53
---


关于多线程的理论，这里不做介绍，Python通过thread和threading两个标准库提供对多线程的支持。
  * thread提供了低级别的、原始的线程以及一个简单的锁。
  * threading基于Java的线程模型设计。锁（Lock）和条件变量（Condition）在Java中是对象的基本行为（每一个对象都自带了锁和条件变量），而在Python中则是独立的对象。

<!-- more -->

## thread模块
使用`start_new_thread`方法开启一个线程，第一个参数为线程函数，第二个参数为参数，如果函数没有参数，要传空元组
```python
import time
import thread

def test1():
    print 'start test1'
    # 休息3秒
    time.sleep(3)
    print 'end test1'

if __name__ == '__main__':
    thread.start_new_thread(test1, ())
    print 'main thread...'
    # start_new_thread创建的线程在主线程执行完成时会自动结束，这里等5秒
    time.sleep(5)
    print 'main thread end'
```

上面通过`sleep`防止主线程退出导致其他线程也跟着退出，显然不靠谱，这时候我们可以通过锁的方式控制线程执行顺序
```python
lock = thread.allocate_lock()  # 返回一个新的锁定对象。
lock.acquire()                 # 请求锁，如果该所没被占用，则成功返回，如果被占用，则等待直到锁被释放
lock.release()                 # 释放锁
```
例子：
```python
import time
import thread

def test1(thread_lock):
    print 'start test1'
    # 休息3秒
    time.sleep(3)
    print 'end test1'
    # 执行完后释放锁
    thread_lock.release()

if __name__ == '__main__':
    # 创建一个锁
    lock = thread.allocate_lock()
    # 请求锁
    lock.acquire()
    # 把锁传给函数
    thread.start_new_thread(test1, (lock,))

    print 'main thread...'
    # 只有被释放了才能请求到
    lock.acquire()
    print 'main thread end'
```

## threading模块
thread模块不支持守护线程，当主线程退出时，所有子线程不管是否工作都会被结束，而threading更强大，也支持守护线程
```python
import time
import threading

def test1():
    print 'start test1 '
    time.sleep(3)      # 休息3秒
    print 'end test1'

if __name__ == '__main__':
    # 创建一个线程
    t = threading.Thread(target=test1, args=())
    # 运行线程
    t.start()

    print 'main thread...'
    # join函数阻塞当前线程，直到t线程运行完成
    t.join()
    print 'main thread end'
```
使用Thread.start()运行的线程，在主线程执行完成后不会被强制结束，会一直运行至结束

常用属性
* threading.currentThread()：返回当前的线程变量。
* threading.enumerate()：返回一个包含正在运行的线程的list
* threading.activeCount()： 返回正在运行的线程数量

Thread对象
* start(): 启动线程
* join(): 阻塞直到线程完成
* isAlive(): 返回线程是否活动的
* getName(): 返回线程名
* setName(): 设置线程名
* run(): 表示线程活动的方法

当Thread对象调用start方法的时候，默认会调用run方法，所以我们可以封装线程函数到Thread对象里面，如下
```python
import time
import threading
class MyThread(threading.Thread):
    def __init__(self, name):
        super(MyThread, self).__init__()
        self.name = name

    def run(self):
        """线程执行函数"""
        time.sleep(2)
        print '%s run' % (self.name,)

t = MyThread('thread_name')
t.start()
```


## 线程同步问题
与其他语言一样，Python也提供了线程同步相关的支持，Python支持下面几种线程同步锁

线程锁的锁定释放的流程如下
> 请求锁定 —> 进入锁定池等待 —> 获取锁 —> 已锁定 —> 释放锁

### 1. Lock & RLock
1. Lock
指令锁，只有两种状态
```python
mutex = threading.Lock()    # 构造方法
mutex.acquire()             # 请求锁，成功则锁定，如果该锁已被锁定，则阻塞等待
# mutex.acquire()           # 会发生死锁
mutex.release()             # 释放锁，使用前该锁必须已被锁定
```

2. RLock
可重入锁，为了保证线程对共享资源的独占，又避免死锁的出现，允许在`同一线程`中多次请求锁，如下：
```python
mutex = threading.RLock()    # 构造方法
mutex.acquire()              # 请求锁
mutex.acquire()              # 请求锁，不会死锁
mutex.release()
mutex.release()              # 请求多少次就要释放多少次，成对出现
```

### 2. Semaphore
信号量，比Lock多了计数器，可以记录多次请求和释放，技术器不能小于0，小于0则会阻塞，通常可以用在控制并发数的情况下，用法与Lock类似
```python
semaphore = threading.Semaphore(2)    # 构造一个信号量，容量为2
semaphore.acquire()                   # 请求信号，计数器-1，执行完后为1
semaphore.acquire()                   # 请求信号，计数器-1，执行完后为0
# semaphore.acquire()                   # 请求信号，计数器为0，阻塞，直到release让计数器+1
semaphore.release()                   # 请求信号，计数器+1，执行完后为1
semaphore.release()                   # 请求信号，计数器+1，执行完后为2
```

### 3. Event
与Lock相反，Event内部维护一个标志位，初始化为false，调用set置为true，调用clear置为flase

```python
import time
import threading

def test1(signal):
    print "I will sleep, wake me up 3 seconds later"
    signal.wait()
    print "I awake up..."

if __name__ == '__main__':
    sig = threading.Event()

    t = threading.Thread(target=test1, args=(sig,))
    t.start()
    # 睡3秒
    time.sleep(3)
    # 叫醒信号
    sig.set()
    print 'main thread end'
```

### 4. Condition
Condition称为条件变量，提供了Python多线程中复杂的同步支持，除了提供与Lock类似的`acquire`和`release`方法外，还提供了`wait`和`notify`方法，支持通知

* wait：release锁，阻塞，等待notify唤醒
* notify：唤醒由wait阻塞的线程


下面使用Condition来模拟一个捉迷藏的游戏
```python
import threading
import time
class Seeker(threading.Thread):
    def __init__(self, cond, name):
        super(Seeker, self).__init__()
        self.cond = cond
        self.name = name

    def run(self):
        self.cond.acquire()
        print self.name + ': 我已经把眼睛蒙上了'
        self.cond.wait()

        print self.name + ': 我找到你了 ^o^'
        self.cond.notify()
        self.cond.wait()

        print self.name + ': 我赢了'
        self.cond.release()


class Hider(threading.Thread):
    def __init__(self, cond, name):
        super(Hider, self).__init__()
        self.cond = cond
        self.name = name

    def run(self):
        time.sleep(1)           # 睡1秒确保Seeker先运行
        self.cond.acquire()
        print self.name + ': 我已经藏好了，你快来找我吧'
        self.cond.notify()
        self.cond.wait()

        print self.name + ': 被你找到了，哎~~~'
        self.cond.notify()

        self.cond.release()

cond = threading.Condition()
seeker = Seeker(cond, 'seeker')
hider = Hider(cond, 'hider')
seeker.start()
hider.start()
```
执行结果如下
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-6-17/91739824.jpg)

## 队列Queue
多线程很多时候可以与队列一起使用，把任务放到队列，保证线程任务的执行顺序
```python
import Queue
myqueue = Queue.Queue(maxsize = 10) # 指定容量，不指定则无限大

# 下面方法默认有个block参数和timeout参数，当容量不足或队列没有对象的时候会阻塞当前线程
myqueue.put(10)                     # 存入值进队列
myqueue.get(block=False)            # 取出队列中的第一个元素，如果没有对象，抛出Queue.Empty异常
```
可以利用Queue写一个线程安全的队列，如对数据库的操作可以放在一个队列里面进行，这样就可以省去线程同步带来的问题了

```python
import threading
import time
import Queue

class MyThread(threading.Thread):
    def __init__(self, queue):
        super(MyThread, self).__init__()
        self.queue = queue

    def run(self):
        (sql, args) = self.queue.get(block=True)
        while sql != 'q':
            # 假定退出操作为'q'
            time.sleep(0.5)
            print 'exe %s' % (sql,)
            (sql, args) = self.queue.get(block=True)


my_queue = Queue.Queue()
my_thread = MyThread(my_queue)
my_thread.start()

my_queue.put(('insert into user(name, age)', ('bobo', 23)))
my_queue.put(('update user set age=24 where name=?', ('bobo',)))
my_queue.put(('delete from user where name=?', ('bobo',)))

# 处理完
my_queue.put(('q', ()))

print 'wait for sqlite complete'
my_thread.join()
print 'complete'
```

## GIL
刚接触Python多线程的时候可能会经常遇到GIL这个词，并且GIL还经常与Python不能高效的实现多线程划上等号
GIL（global interpreter lock）不是Python的特性，而是CPython的特性，而CPython是通常是Python默认的解释器，而Python本身，不依赖于GIL

CPython编译器引入了GIL全局锁（进程）来解决多线程环境下的数据同步问题，即Python对象的内部是thread-safe的，并且被开发者广泛依赖，当然这种简单粗暴的锁不可避免也带来了一定的性能损耗，并且由于GIL的存在，同一时刻只能有一个线程在运行，Python无法充分的利用多核CPU带来的多核计算

![](http://7xqzvt.com1.z0.glb.clouddn.com/16-6-17/73225581.jpg)
* CPU密集型通常是计算为主，如图像处理，复杂的数学计算等
* IO密集型通常是与硬件相关的，如硬盘、网卡、声卡、显卡，计算机需要等待硬件的耗时处理，比较常见的有文件处理，网络流处理，这时候CPU负载比较低


在CPU密集型的场景下，由于GIL的存在，线程消耗CPU资源比较多，而在多线程下需要频繁的获取和释放锁，带来很大量的开销，所以通常在CPU密集型的场景下，多线程通常不如单线程来得快，建议使用多进程，而不是多线程

在IO密集型的场景下，由于GIL的存在，Python在遇到IO操作的时候回释放锁，建议使用多线程，而不是多进程

当然进程与线程又有自身的优缺点，进程不共享内存，多进程通讯比较麻烦，而线程共享所有内存，通讯更方便，具体如何取舍还是得看具体业务了

关于GIL的更多介绍，可以参见[这里](http://cenalulu.github.io/Python/gil-in-Python/)

## 测试CPU密集型和IO密集型场景下的多线程效果
1. CPU密集型:给一张图片创建1000张缩略图
2. IO密集型:给一个文件进行重复的读写和删除1000次操作

```python
import os
import time
import threading
import Queue
import uuid
from PIL import Image

class MyThread(threading.Thread):
    THUMB_SIZE = (75, 75)
    SAVE_DIRECTORY = 'thumbs'

    def __init__(self, queue, is_cpu=True):
        super(MyThread, self).__init__()
        self.queue = queue
        self.is_cpu = is_cpu

        if is_cpu:
            # 如果是CPU密集型,只打开一次图片,避免IO操作对测试的影响,图片大一些效果比较好
            self.path = u'/Users/zhengxiankai/Desktop/Python/test.jpg'
            self.image = Image.open(self.path)
        else:
            self.path = u'/Users/zhengxiankai/Desktop/Python/test2.png'

    def run(self):
        item = self.queue.get(block=True)
        while item != 'q':
            # 假定退出操作为'q'
            if self.is_cpu:
                self.test_cpu(self.image)
            else:
                self.test_io(self.path)
            self.queue.task_done()
            item = self.queue.get(block=True)

        self.queue.task_done()

    def test_cpu(self, img):
        # 模拟CPU密集型操作,只生成缩略图,不进行IO操作
        img.thumbnail(self.__class__.THUMB_SIZE, Image.ANTIALIAS)

    @staticmethod
    def test_io(filename):
        # 模拟IO密集型操作,进行文件读写和删除
        base, file_name = os.path.split(filename)
        file_id = uuid.uuid1()
        save_path = os.path.join(base, str(file_id) + ".jpg")
        # 保存文件
        open(save_path, "wb").write(open(filename, "rb").read())
        # 删除文件
        os.remove(save_path)


class MyQueue(Queue.Queue):
    def __init__(self, maxsize=0, thread_count=10, is_cpu=True):
        Queue.Queue.__init__(self, maxsize=maxsize)
        self.thread_count = thread_count
        self.threads = []
        self.is_cpu = is_cpu
        self.create_threads(thread_count)

    def create_threads(self, thread_count):
        for i in range(0, thread_count):
            thread = MyThread(self, self.is_cpu)
            thread.start()
            self.threads.append(thread)

    def finish(self):
        for _ in range(0, len(self.threads)):
            self.put('q')


def test():
    # 模拟1-10个线程的情况
    types = ['cpu', ' io']
    for t in types:
        for thread_count in range(1, 10):
            my_queue = MyQueue(thread_count=thread_count, is_cpu=(t == 'cpu'))
            # 模拟1000次操作
            for __ in range(0, 1000):
                my_queue.put(1)

            # 在处理完后结束所有线程
            my_queue.finish()
            start = time.time()
            my_queue.join()
            span = time.time() - start

            print '%s: thread_count=%d span=%s' % (t, thread_count, str(span))

test()
```
用我的MacbookPro测试结果如下：
```bash
cpu: thread_count=1 span=0.304748058319
cpu: thread_count=2 span=0.299258947372
cpu: thread_count=3 span=0.359387874603
cpu: thread_count=4 span=0.452157974243
cpu: thread_count=5 span=0.52136015892
cpu: thread_count=6 span=0.610749959946
cpu: thread_count=7 span=0.688009023666
cpu: thread_count=8 span=0.812467098236
cpu: thread_count=9 span=0.939681053162
 io: thread_count=1 span=2.01645898819
 io: thread_count=2 span=1.76048994064
 io: thread_count=3 span=1.45470404625
 io: thread_count=4 span=1.45363807678
 io: thread_count=5 span=1.10205197334
 io: thread_count=6 span=1.02844214439
 io: thread_count=7 span=1.0888478756
 io: thread_count=8 span=1.01435399055
 io: thread_count=9 span=1.0473139286

Process finished with exit code 0
```
上面可以看出，在CPU密集型的场景下，线程越多越慢，在IO密集型的场景下，线程数在6以下，提升效果明显，在6个以上，性能提升的效果不明显，所以Python得多线程也不是一无是处，看具体场景使用
