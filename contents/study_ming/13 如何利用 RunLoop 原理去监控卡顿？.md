> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 13 如何利用 RunLoop 原理去监控卡顿？


#### 导致卡顿问题的几种原因:

- 复杂 UI 、图文混排的绘制量过大; 
- 在主线程上做网络同步请求; 
- 在主线程做大量的 IO 操作; 
- 运算量过大，CPU 持续高占用; 
- 死锁和主子线程抢锁。


#### RunLoop 原理
RunLoop 内部实际上就是一个 do-while 循环，它在循环监听着各种事件源、消息，对他们进行管理并分发给线程来执行。

##### 可以参考以下代码：

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

其中，UIApplicationMain 函数内部启动了 RunLoop，因此 UIApplicationMain 函数是一直都没有返回的，保持了程序的持续运行。

##### RunLoop 特点：

- 保证程序不退出；
- 负责处理各种事件（source、timer、observer）；
- 有事做就去处理，没事做就休息。这样可以节省 CPU 资源，提高程序性能。

#### 监测卡顿的思路

##### RunLoop 的 6 个状态

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry , // 进入 loop
    kCFRunLoopBeforeTimers , // 触发 Timer 回调
    kCFRunLoopBeforeSources , // 触发 Source0 回调
    kCFRunLoopBeforeWaiting , // 等待 mach_port 消息
    kCFRunLoopAfterWaiting ), // 接收 mach_port 消息
    kCFRunLoopExit , // 退出 loop
    kCFRunLoopAllActivities  // loop 所有状态改变
}
```

如果 RunLoop 的线程，进入睡眠前方法的执行时间过长而导致无法进入睡眠，或者线程唤醒后 接收消息时间过长而无法进入下一步的话，就可以认为是线程受阻了。如果这个线程是主线程的 话，表现出来的就是出现了卡顿。

#### 如何检查卡顿

这里只简单说下思路：

- 创建一个 CFRunLoopObserverContext 观察者；
- 将创建好的观察者 runLoopObserver 添加到主线程 RunLoop 的 common 模式下观察；
- 创建一个持续的子线程专门用来监控主线程的 RunLoop 状态；
- 一旦发现进入睡眠前的 kCFRunLoopBeforeSources 状态，或者唤醒后的状态 kCFRunLoopAfterWaiting，在设置的时间阈值内一直没有变化，即可判定为卡顿；
- dump 出堆栈的信息，从而进一步分析出具体是哪个方法的执行时间过长；

代码可以看戴铭老师的GitHub：[SMLagMonitor.m](https://github.com/ming1016/DecoupleDemo/blob/496276c323681db239ea5ab5352dcc03d90cd4a6/DecoupleDemo/SMLagMonitor.m)

#### 强烈推荐 ✨✨✨✨✨：

- [深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)
- [iOS线下分享《RunLoop》by 孙源@sunnyxx](https://v.youku.com/v_show/id_XODgxODkzODI0.html)
