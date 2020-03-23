> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 18 怎么减少App电量消耗？

#### 诊断电量问题的方式

##### CPU 计算

首先查看当前 CPU 使用率，然后获取当前方法堆栈，就可以定位耗电原因。

获取线程信息：

```
thread_act_array_t threads;
mach_msg_type_number_t threadCount = 0;
const task_t thisTask = mach_task_self();
kern_return_t kr = task_threads(thisTask, &threads, &threadCount);
```

threads 结构体：

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

判断 cpu_usage 就可以知道当前 CPU 的使用百分比，较高的情况即是耗电较多的情况：

```
// 轮询检查多个线程 CPU 情况
+ (void)updateCPU {
    thread_act_array_t threads;
    mach_msg_type_number_t threadCount = 0;
    const task_t thisTask = mach_task_self();
    kern_return_t kr = task_threads(thisTask, &threads, &threadCount);
    if (kr != KERN_SUCCESS) {
        return;
    }
    for (int i = 0; i < threadCount; i++) {
        thread_info_data_t threadInfo;
        thread_basic_info_t threadBaseInfo;
        mach_msg_type_number_t threadInfoCount = THREAD_INFO_MAX;
        if (thread_info((thread_act_t)threads[i], THREAD_BASIC_INFO, (thread_info_t)threadInfo, &threadInfoCount) == KERN_SUCCESS) {
            threadBaseInfo = (thread_basic_info_t)threadInfo;
            if (!(threadBaseInfo->flags & TH_FLAGS_IDLE)) {
                integer_t cpuUsage = threadBaseInfo->cpu_usage / 10;
                if (cpuUsage > 90) {
                    //cup 消耗大于 90 时打印和记录堆栈
                    NSString *reStr = smStackOfThread(threads[i]);
                    //记录数据库中
                    [[[SMLagDB shareInstance] increaseWithStackString:reStr] subscribeNext:^(id x) {}];
                    NSLog(@"CPU useage overload thread stack：\n%@",reStr);
                }
            }
        }
    }
}
```

- 优化电量
	- 对 CPU 的使用要精打细算，要避免让 CPU 做多余的事情。对于大量数据的复杂计算，应该把数据传到服务 器去处理，如果必须要在 App 内处理复杂数据计算，可以通过 GCD 的 dispatch_block_create_with_qos_class 方法指定队列的 Qos 为 QOS_CLASS_UTILITY，将计算工作放到这个 队列的 block 里。在 QOS_CLASS_UTILITY 这种 Qos 模式下，系统针对大量数据的计算，以及复杂数据处理专门做了电量优化。

	
##### I/O 操作

I/O 操作会破坏低功耗状态，所以要减少操作次数。可以先使用 NSCache 来存储，等数据存储达到阈值之后再将数据进行 I/O 操作。比如我们常见的 SDWebImage 就是这样实现的：

```
- (UIImage *)imageFromMemoryCacheForKey:(NSString *)key {
      return [self.memCache objectForKey:key];
}
- (UIImage *)imageFromDiskCacheForKey:(NSString *)key {
// 检查 NSCache 里是否有
UIImage *image = [self imageFromMemoryCacheForKey:key]; if (image) {
          return image;
      }
// 从磁盘里读
UIImage *diskImage = [self diskImageForKey:key]; if (diskImage && self.shouldCacheImagesInMemory) {
          NSUInteger cost = SDCacheCostForImage(diskImage);
          [self.memCache setObject:diskImage forKey:key cost:cost];
      }
      return diskImage;
  }
```

可以看出，SDWebImage 将获取的图片数据都放到了 NSCache 里，利用 NSCache 缓存策略进行图片缓存 内存的管理。每次读取图片时，会检查 NSCache 是否已经存在图片数据:如果有，就直接从 NSCache 里读 取;如果没有，才会通过 I/O 读取磁盘缓存图片。

使用了 NSCache 内存缓存能够有效减少 I/O 操作，在写类似功能时也可以采用这样的思路，让 App 更省电。

#### 推荐博客

- [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/MonitorEnergyWithInstruments.html)
- [Writing Energy Efficient Apps - 2017 - 238](https://developer.apple.com/videos/play/wwdc2017/238/)
- [iOS 电量消耗改善：一招套路及相关姿势](https://juejin.im/post/5bfce9305188255362443ac5)
