> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 12 iOS崩溃千奇百怪，如何全面监控？

#### 常见的崩溃情况

- 数组越界:在取数据索引时越界，App 会发生崩溃。还有一种情况，就是给数组添加了 nil 会崩溃。这个可以扩展为容器越界（NSArray，NSDictionary等）。
- 多线程问题:在子线程中进行 UI 更新可能会发生崩溃。多个线程进行数据的读取操作，因为 处理时机不一致，比如有一个线程在置空数据的同时另一个线程在读取这个数据，可能会出现出崩溃情况。
- 主线程无响应:如果主线程超过系统规定的时间无响应，就会被 Watchdog 杀掉。这时，崩溃 问题对应的异常编码是 0x8badf00d。关于这个异常编码，我还会在后文和你说明。
- 野指针:指针指向一个已删除的对象访问内存区域时，会出现野指针崩溃。野指针问题是需要 我们重点关注的，因为它是导致 App 崩溃的最常见，也是最难定位的一种情况。

#### 常见的崩溃情况分类

##### 信号可捕获的分类

- KVO问题
- NSNotification线程问题
- 数组越界
- 野指针等

##### 信号不可捕获的分类

- 后台任务超时
- 内存打爆
- 主线程卡顿超阈值等

#### 信号可捕获的崩溃信息收集工具

戴铭老师推荐了三款崩溃信息收集工具，可以选择 [plcrashreporter](https://github.com/microsoft/plcrashreporter)，上传到自己服务器上进行整体监控，如果公司没有服务端开发能力，或者对数据不敏感，可以直接使用 [Fabric](https://get.fabric.io) 或者 [bugly](https://bugly.qq.com/v2/)。

或者还可以参考这篇博客：[漫谈iOS Crash收集框架](http://www.cocoachina.com/articles/12301)

#### 信号不可捕获的崩溃信息收集

##### 后台容易崩溃的原因

iOS 后台保活的 5 种方式:Background Mode、Background Fetch、Silent Push、PushKit、Background Task。

其中 Background Task 方式，是使用最多的。App 退后台后，默认都会使用这种方式。程序退到后台以后，只有几秒钟的时间可以执行代码，接下来就会被系统挂起。进程挂起 后所有线程都会暂停，不管这个线程是文件读写还是内存读写都会被暂停。但是，数据读写过程 无法暂停只能被中断，中断时数据读写异常而且容易损坏文件，所以系统会选择主动杀掉 App 进程。


而 Background Task 这种方式，就是系统提供了 beginBackgroundTaskWithExpirationHandler 方法来延长后台执行时间，可以解决你退后台后还需要一些时间去处理一些任务的诉求。

beginBackgroundTaskWithExpirationHandler 方法最多执行 3 分钟，3 分钟内任务运行完成，App 就会挂起。 如果需要执行的任务在 3 分钟之内没有执行完的话，系统会强制杀掉进程，从而造成崩溃，这就是为什么 App 退后台容易出现崩溃的原因。

##### 如何避免后台崩溃

严格控制后台数据的读写操作。比如，可以先判断需要处理的数据的大小，如果数据过大，也就是在后台限制时间内或延长后台执行时间后也处理不完的话，可以考虑在程序下次启动或后台唤醒时再进行处理。

##### 如何收集崩溃信息

采用 Background Task 方式时，我们可以根据 beginBackgroundTaskWithExpirationHandler 会 让后台保活 3 分钟这个阈值，先设置一个计时器，在接近 3 分钟时判断后台程序是否还在执行。 如果还在执行的话，我们就可以判断该程序即将后台崩溃，进行上报、记录，以达到监控的效果。

> 内存打爆和主线程卡顿时间超过阈值被 watchdog 杀掉这两种情况，也可以根据上述思路，先找到它们的阈值，然后在临近阈值时还在执行的后台程序，判断为将要崩溃，收集信息并上报。