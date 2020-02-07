# iOS开发之 - RunLoop

![RunLoop](http://upload-images.jianshu.io/upload_images/2665449-4d7042ab55fbe301.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 以下是 RunLoop 相关的内容，前半部分是理论知识，后半部分是代码。。。

## 一、RunLoop 基础知识

### 1.1 RunLoop  简介
一般来说，一个线程只能执行一个任务，执行完就会退出，如果我们需要一种机制，让线程能随时处理时间但并不退出，那么 RunLoop 就是这样的一个机制。Runloop是事件接收和分发机制的一个实现。所以它实际上是一个对象，这个对象管理了其需要处理的事件和消息。在iOS 系统中，有这样两个对象： NSRunLoop 和 CFRunLoopRef。 NSRunLoop 和 CFRunLoopRef都代表着 RunLoop 对象。


### 1.2 RunLoop的基本作用

 (1)保持程序的持续运行；
(2)处理 App 中的各种事件(比如触摸事件、定时器事件、Selector 事件)；
(3)节省 CPU 资源, 提高程序性能。

### 1.3 RunLoop的使用环境

仅当在为你的程序创建辅助线程的时候，你才需要显示运行 RunLoop 。对于辅助线程，你需要判断一个 RunLoop 是否是必须的。如果是必须的，那么你要自己配置并启动它，你不需要再任何情况下都去启动一个线程的 RunLoop 。 RunLoop 在你要和线程有更多的交互时才需要，比如以下情况：
(1)使用端口或者自定义输入源来和其他线程通信；
(2)使用线程的定时器；
(3) Cocoa 中使用任何 performSelector 的方法；
(4)使线程周期性工作。 

### 1.4 RunLoop工作的特点:

(1)当有时间发生时, RunLoop 会根据具体的事件类型通知应用程序作出响应；
(2)当没有事件发生时, RunLoop 会进入休眠状态；
(3)当事件再次发生时, RunLoop 会被重新唤醒, 处理事件。

### 1.5 runLoop的内部逻辑
runLoop其实是一个函数，其内部是一个 do-while 循环，当你调用CFRunLoop() 时，线程就会一直停留在这个循环中；直到超时或被手动停止，该函数才会返回。每次运行 runLoop，线程的 runLoop 会自动处理之前未处理的消息，并通知相关的观察者，具体的顺序如下：
(1)通知观察者 runLoop 已经启动；
(2)通知观察者处理将要开始的定时器；
(3)通知观察者处理即将启动的非基于端口的源；
(4)处理非基于端口的源；
(5)如果基于端口的源准备好并处于等待状态，立即启动进入步骤⑨；
(6)通知观察者线程即将进入休眠；
(7)线程处于休眠状态，直到下面的任意的一个事件发生：
A.某一时间到达基于端口的源；
B.定时器启动；
C.runLoop被外部手动唤醒；
(8)通知观察者线程刚被唤醒；
(9)处理接收到的事件，处理定时器并重启 runLoop，进入步骤2；
(10)通知观察者，即将退出 runLoop。

### 1.6 RunLoop与线程的关系

(1)每条线程都有唯一的一个与之对应的 RunLoop 对象；
(2)主线程的 RunLoop 已经自动创建好了, 子线程的 RunLoop 需要手动创建；
(3) RunLoop在第一次获取时创建, 在线程结束时销毁；
(4)直线线程与圆形线程：直线线程执行的任务是一条直线；而圆形线程不断循环，直到通过某种方式截止，在 iOS 中，圆形线程就是通过 runLoop 实现的。

 

### 1.7 RunLoop对象

(1) iOS中有2套 API 来访问和使用 RunLoop。
Foundation框架 : NSRunLoop；
Core Foundation框架: CFRunLoopRef。

(2) NSRunLoop 和 CFRunLoopRef 都代表着 RunLoop 对象 ,它们是等价的，可以互相转换；

(3) NSRunLoop 是基于 CFRunLoopRef 的一层 OC 包装，所以要了解 RunLoop 内部结构，需要多研究 CFRunLoopRef 层面的 API（Core Foundation层面）。

### 1.8获得 RunLoop 对象
苹果不允许直接创建RunLoop，它只提供了两个自动获取的函数。
(1) Foundation [NSRunLoop currentRunLoop]; //获得当前线程的 RunLoop 对象
[NSRunLoop mainRunLoop]; //获得主线程的 RunLoop 对象

(2) Core Foundation
CFRunLoopGetCurrent(); //获得当前线程的 RunLoop 对象
CFRunLoopGetMain(); //获得主线程的 RunLoop 对象

### 1.9 相关类:

Core Foundation中关于 RunLoop 的 5 个类 ,RunLoop 如果没有这些东西, 会直接退出。
(1) CFRunLoopRef
(2) CFRunLoopModeRef 该类并没有对外暴露；
(3) CFRunLoopSourceRef
(4) CFRunLoopTimerRef
(5) CFRunLoopObserverRef

 

### 1.10 CFRunLoopSourceRef 事件源(输入源)

时间产生的地方，source 有两个版本：
(1) source0：只包含一个回调，它并不能主动触发事件，使用时，需先调用 CFRunLoopSourceSignal 将这个 source 标记为待处理，后调用 CFRunLoopWakeUp 来唤醒 runLoop，让其处理这个事件；
(2) source1：包含一个 mach_port 和一个回调，被用于通过内核和其他线程互发送消息，这种 source 能主动唤醒 runLoop 线程。

### 1.11 CFRunLoopObserverRef 
CFRunLoopObserverRef是观察者,能够监听 RunLoop 的状态改变,当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：

kCFRunLoopEntry = (1UL << 0), // 即将进入 Loop

kCFRunLoopBeforeTimers = (1UL << 1), //即将处理 Timer

kCFRunLoopBeforeSources = (1UL << 2), //即将处理 Source

kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠

kCFRunLoopAfterWaiting = (1UL << 6), //刚从休眠中唤醒

kCFRunLoopExit = (1UL << 7), //即将退出 Loop

### 1.12 runLoop的模式
runLoop 中使用 mode 来指定时间在运行循环中的优先级，系统默认注册了 5 个Mode：
(1)NSDefaultRunLoopMode（kCFRunLoopDefaultMode）:默认Mode,通常主线程是在这个Mode下运行；

(2) UITrackingRunLoopMode:界面跟踪Mode,用于 ScrollView 追踪触摸滑动, 保证界面滑动时不受其他 Mode 影响；

(3) UIInitializationRunLoopMode:启动程序后的过渡 mode，启动完成后就不再使用；

(4) GSEventReceiveRunLoopMode:接受系统事件的内部 Mode,通常用不到；

(5)kCFRunLoopCommonModes（NSRunLoopCommonModes）: 这是一个占位用的 Mode ，作为标记 DefaultMode 和CommonMode 用。

## 二、演示代码
### 演示代码一:常驻线程

控制器.m中的代码:

```objc
- (void)viewDidLoad {
[super viewDidLoad];
self.thread = [[NNThread alloc] initWithTarget:self selector:@selector(run) object:nil];
[self.thread start];
}

#pragma mark -常驻线程方式一
- (void)run {
@autoreleasepool {

[[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
NSLog(@"-- run -- %@ --", [NSThread currentThread]);
[[NSRunLoop currentRunLoop] run];
NSLog(@"--不会执行--"); // 不会执行
}
}

#pragma mark -常驻线程方式二
- (void)run1{
@autoreleasepool {
while (1) {
[[NSRunLoop currentRunLoop] run];
NSLog(@"-- run1 -- %@ --", [NSThread currentThread]);
}
}
}

#pragma mark -常驻线程方式三
- (void)run2{
@autoreleasepool {
[NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(test) userInfo:nil repeats:YES];
NSLog(@"-- run2 -- %@ --", [NSThread currentThread]);
[[NSRunLoop currentRunLoop] run];
}
}

#pragma mark -在该线程中自定义事件
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
[self performSelector:@selector(test) onThread:self.thread withObject:nil waitUntilDone:NO];
}

- (void)test {
NSLog(@"-- test -- %@ --", [NSThread currentThread]);
}

```

线程子类化:

```objc

#import "NNThread.h"
@implementation NNThread

#pragma mark -重写了 dealloc 方法,查看线程是否销毁
- (void)dealloc {
NSLog(@"线程 NNThread 被销毁"); // 不会打印
}
@end
```

 

### 演示代码二: GCD 定时器:
```objc
#pragma mark - GCD 的 timer
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
//获得队列
dispatch_queue_t queue = dispatch_get_main_queue();
//创建一个定时器 (dispatch_source_t 本质还是个 OC 对象)

self.timer =dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
// GCD的时间参数，一般是纳秒（1秒== 10的9次方纳秒）
dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
uint64_t interval = (uint64_t)(1.0 * NSEC_PER_SEC);
dispatch_source_set_timer(self.timer, start, interval, 0);

//设置回调
dispatch_source_set_event_handler(self.timer, ^{
NSLog(@"------ %@ ------", [NSThread currentThread]);
count++;
if (count == 10) {
//取消定时器
dispatch_cancel(self.timer);
self.timer = nil;
}
});

//启动定时器
dispatch_resume(self.timer);
}

```

 

### 演示代码三:观察者

```objc
#pragma mark -观察者
-(void)observerTest{
/**
CFRunLoopObserverRef是观察者, 能够监听 RunLoop 的状态改变,当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。

      CFRunLoopObserverRef参数:
第一个参数:分配存储空间
第二个参数:监听状态
第三个参数:是否要持续监听
第四个参数:优先级
第五个参数:回调
*/

//创建observer
CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
switch (activity) {
case kCFRunLoopEntry:
NSLog(@"即将进入runloop");
break;
case kCFRunLoopBeforeTimers:
NSLog(@"RunLoop即将处理 timer");
break;
case kCFRunLoopBeforeSources:
NSLog(@"RunLoop即将处理Sources");
break;
case kCFRunLoopBeforeWaiting:
NSLog(@"RunLoop即将进入休眠");
break;
case kCFRunLoopAfterWaiting:
NSLog(@"RunLoop刚从休眠中唤醒");
break;
case kCFRunLoopExit:
NSLog(@"即将退出RunLoop ");
break;
default:
break;
}
});
/**
RunLoop的模式
1.NSDefaultRunLoopMode（kCFRunLoopDefaultMode）:默认Mode,通常主线程是在这个Mode下运行
2.UITrackingRunLoopMode:界面跟踪 Mode, 用于 ScrollView 追踪触摸滑动, 保证界面滑动时不受其他 Mode 影响
3.UIInitializationRunLoopMode:启动程序后的过渡 mode，启动完成后就不再使用
4.GSEventReceiveRunLoopMode:接受系统事件的内部 Mode,通常用不到
5.kCFRunLoopCommonModes（NSRunLoopCommonModes）: 这是一个占位用的 Mode, 作为标记 DefaultMode 和 CommonMode 用
*/

/**
CFRunLoopAddObserver参数:
第一个参数:要监听哪个RunLoop
第二个参数:监听者
第三个参数:要监听RunLoop在哪种运行模式下的状态
*/

// 添加观察者：监听RunLoop的状态
CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);

// 释放  Observer
CFRelease(observer);
}

- (void)timerTest
{
//调用了 scheduledTimer 返回的定时器，已经自动被添加到当前 RunLoop 中，默认是 NSDefaultRunLoopMode
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];

//修改模式
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
}
```
*end*

---

**结束语：RunLoop 的总结就到此为止了，以后如果想到或遇到新知识点，会再来补充！另外如果文章写的有什么错误，烦请各位能指出来！谢谢大家！**
