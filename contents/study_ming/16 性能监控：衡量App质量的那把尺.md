> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 16 性能监控：衡量App质量的那把尺

> 对App的性能监控，主要是从线下和线上两个维度展开。

#### 线下 Instruments

Instruments的功能非常强大，比如说Energy Log就是用来监控耗电量的，Leaks就是专门用来监控内存泄露 问题的，Network就是用来专门检查网络情况的，Time Profiler就是通过时间采样来分析页面卡顿问题的。

关于 Instruments 的各种性能检测工具，戴铭老师总结了一个很棒的图：

![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_Instruments.jpg)

开发人员还可以使用 Instruments 10 还可以开发一款自定义 Instruments 工具。创建自定义 Instruments 工具可参考：[WWDC 2018：创建自定义的 Instrument](https://juejin.im/post/5b1cd1025188257d72709d7f)。

或者还可以参考 WeRead 团队开源的一款检测 iOS 内存泄漏的框架：[MLeaksFinder](https://github.com/Tencent/MLeaksFinder)。

#### 线上性能监控

线上性能监控的两个原则:

1. 监控代码不要侵入到业务代码中; 
2. 采用性能消耗最小的监控方案。

#### CPU 使用率的线上监控方法

需要明白一点，各个线程对 CPU 使用率的总和，就是当前 App 对 CPU 的使用率。iOS 系统中，在 usr/include/mach/thread_info.h 里看到线程基本信息的结构体，其中的 cpu_usage 就是 CPU 使用率。结构体的完整代码如下所示：

```
struct thread_basic_info {
    time_value_t user_time;  // 用户运行时长
    time_value_t system_time;  // 系统运行时长
    integer_t  cpu_usage;  // CPU 使用率
    policy_t policy; // 调度策略
    integer_t  run_state;  // 运行状态
    integer_t  flags;  // 各种标记
    integer_t  suspend_count; // 暂停线程的计数
    integer_t  sleep_time; // 休眠的时间
};
```

每个线程都会有 thread_basic_info 结构体，定时（比如，将定时间隔设置为 2s）去遍历每个线程，累加每个线程的 cpu_usage 字段的值，就能够得到当前 App 所在进程的 CPU 使用率了。实现代码如下：

```
+ (integer_t)cpuUsage {
    thread_act_array_t threads; //int 组成的数组比如 thread[1] = 5635
    mach_msg_type_number_t threadCount = 0; //mach_msg_type_number_t 是 int 类型
    const task_t thisTask = mach_task_self();
    // 根据当前 task 获取所有线程
    kern_return_t kr = task_threads(thisTask, &threads, &threadCount);
    if (kr != KERN_SUCCESS) {
        return 0;
    }
    integer_t cpuUsage = 0;
    // 遍历所有线程
    for (int i = 0; i < threadCount; i++) {
        thread_info_data_t threadInfo;
        thread_basic_info_t threadBaseInfo;
        mach_msg_type_number_t threadInfoCount = THREAD_INFO_MAX;
        if (thread_info((thread_act_t)threads[i], THREAD_BASIC_INFO, (thread_info_t)threadInfo, &threadInfoCount) == KERN_SUCCESS) {
            // 获取 CPU 使用率（遍历数组获取单个线程的基本信息。其中，线程基本信息的结构体是 thread_basic_info_t，这个结构体里就包含了我们需要的 CPU 使用率的字段 cpu_usage。然后累加这个字段就能够获取到当前的整体 CPU 使用率。）
            threadBaseInfo = (thread_basic_info_t)threadInfo;
            if (!(threadBaseInfo->flags & TH_FLAGS_IDLE)) {
                cpuUsage += threadBaseInfo->cpu_usage;
            }
        }
    }
    assert(vm_deallocate(mach_task_self(), (vm_address_t)threads, threadCount * sizeof(thread_t)) == KERN_SUCCESS);
    return cpuUsage;
}

```

#### FPS 线上监控方法

FPS 是指图像连续在显示设备上出现的频率。FPS 低，表示 App 不够流畅。和前面对 CPU 使用率的监控不同，iOS 系统中没有一个专门的结构体，用来记录与 FPS 相关的数据。但是，对 FPS 的监控也可以比较简单的实现：通过注册 CADisplayLink 得到屏幕的同步刷新率， 记录每次刷新时间，然后就可以得到 FPS。具体的实现代码如下:

```
- (void)start {
    self.dLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(fpsCount:)];
    [self.dLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
}
// 方法执行帧率和屏幕刷新率保持一致
- (void)fpsCount:(CADisplayLink *)displayLink {
    if (lastTimeStamp == 0) {
        lastTimeStamp = self.dLink.timestamp;
    } else {
        total++;
        // 开始渲染时间与上次渲染时间差值
        NSTimeInterval useTime = self.dLink.timestamp - lastTimeStamp;
        if (useTime < 1) return;
        lastTimeStamp = self.dLink.timestamp;
        // fps 计算
        fps = total / useTime;
        total = 0;
    }
}
```

还可以参考 [ibireme](https://github.com/ibireme) 的 [YYFPSLabel](https://github.com/ibireme/YYText/blob/master/Demo/YYTextDemo/YYFPSLabel.m)。

#### 内存使用量的线上监控方法

通常情况下，我们在获取 iOS 应用内存使用量时，都是使用 task_basic_info 里的 resident_size 字段信息。但是，我们发现这样获得的内存使用量和 Instruments 里看到的相差很大。后来，在 2018 WWDC Session 416 iOS Memory Deep Dive中，苹果公司介绍说 phys_footprint 才是实际使用的物理内存。

内存信息存在 task_info.h （完整路径 usr/include/mach/task.info.h）文件的 task_vm_info 结构体中，其中 phys_footprint 就是物理内存的使用，而不是驻留内存 resident_size。结构体里和内存相关的代码如下：

```
struct task_vm_info {
    mach_vm_size_t virtual_size;  // 虚拟内存大小
    integer_t region_count;  // 内存区域的数量
    integer_t page_size;
    mach_vm_size_t resident_size; // 驻留内存大小
    mach_vm_size_t resident_size_peak; // 驻留内存峰值
    /* added for rev1 */
    mach_vm_size_t phys_footprint;  // 物理内存
}
```

类似于对 CPU 使用率的监控，我们只要从这个结构体里取出 phys_footprint 字段的值，就能够监控到实际物理内存的使用情况了。具体实现代码如下：

```
uint64_t memoryUsage() {
    task_vm_info_data_t vmInfo;
    mach_msg_type_number_t count = TASK_VM_INFO_COUNT;
    kern_return_t result = task_info(mach_task_self(), TASK_VM_INFO, (task_info_t) &vmInfo, &count);
    if (result != KERN_SUCCESS){return 0;}
    return vmInfo.phys_footprint;
}
```

从以上三个线上性能监控方案可以看出，它们的代码和业务逻辑是完全解耦的，监控时基本都是直接获取系统本身提供的数据，没有额外的计算量，因此对 App 本身的性能影响也非常小，满足了我们要考虑的两个原则。