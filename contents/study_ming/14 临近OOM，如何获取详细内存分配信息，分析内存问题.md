> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 14 临近OOM，如何获取详细内存分配信息，分析内存问题


OOM：Out of Memory，指的是 App 内存达到上限被系统强杀。OOM 是由 iOS 的 Jetsam 机制导致的一种“另类”崩溃，日志无法通过信号捕捉到。

Jestam 机制： 操作系统为了防止内存资源的过度使用，而采用的一种管控机制。

#### 通过 JetsamEvent 日志计算内存限制值

通过手机的设置 —> 隐私 —> 分析可以查看系统日志，在系统日志中找到 JetsamEvent 开头的日志，日志中的 PageSize 代表内存页大小 ，per-process-limit 部分的 rpages 代表内存页数量。pagesize *rpages / 1024 / 1024 就是内存大小。

#### iOS怎么发现 Jetsam？

iOS 会开启优先级最高的线程 vm_pressure_monitor 来监控系统的内存压力情况，通过一个堆栈来维护所有App的进程。而且 iOS 还会维护一个内存快照表，用于保存每个进程内存页的消耗情况。

当监控系统内存的线程某 App 内存有压力，就会发通知让 App 执行 DidReceiveMemoryWarning 代理。通过此代理，可以获得最后一个机会去释放内存。

系统在强杀 App 前，会先做优先级判断。优先级判断依据是：内核用线程的优先级最高，其次是操作系统，最后是 App。前台 App 优先级高于后台 App。若相同优先级的线程中，哪个线程 CPU 占用高，优先级会比降低。

iOS 因为系统占用原因强杀 App 前，至少有 6 秒钟的时间用来做优先级判断。也就是这 6 秒内生成 JetSamEvent 日志。

除了 JetSamEvent 日志，还可以通过 XNU 来获取内存的限制值。

#### 通过 XNU 获取内存限制值

XNU：iOS 系统的内核。

在 XNU 中，有专门获取内存上限值的函数和宏。也可以通过 memorystatus_priority_entry 这个结构体得到进程的优先级与内存限制值。

通过 XNU 的宏获取内存限制，需要由 root 权限，非越狱情况下想获取这个权限，可以利用didReceivedMemoryWarning 这个代理事件来动态获取。

#### 如何获取当前内存使用情况？

iOS提供了一个函数task_info，可以帮我们获取到当前任务的信息。

```
	// 需要导入 mach.h 文件
    struct mach_task_basic_info info;
    mach_msg_type_number_t size = sizeof(info);
    task_info(mach_task_self(), MACH_TASK_BASIC_INFO, (task_info_t)&info, &size);
   
    float used_memory = info.resident_size;
    NSLog(@"使用了 %f MB内存",used_memory / 1024.0f / 1024.0f);
```

#### 定位内存问题信息收集

**现在有三种方法获取内存上限：JetSam 日志自己算、XNU 越狱才有 root 权限或者通过 DidreceivedMemory 动态获取、task_info 只需要导入 mach.h 文件。**

想精确定位问题，就需要 dump 出完整的内存信息，包括所有对象和其内存占用值，然后在内存接近上限时都记录下来并找机会上传。不过不仅要知道对象的内存使用，还要知道是谁分配的这块内存，也就是具体在哪个函数里被创建出来的。

内存分配函数 malloc（分配后不清理缓存）与 calloc（分配后清理缓存）等默认使用 nano_zone 分配 256B 以下的内存，大于 256B 的使用 scalable_zone 来分配。

scalable_zone 分配内存的函数都会调用 malloc_logger 函数，系统通过 malloc_logger 来统计并管理内存的分配情况。

iOS 通过堆栈管理所有的 app 进程，通过一个优先级最高的线程去监控系统内存的压力，还有一个快速对照表记录所有 app 的内存使用情况。如果内存有压力了，就按照优先级去释放优先级低还使用内存多的。

所以 app 的内存阈值是没有固定大小的。

#### iOS 获取内存信息：

```
#import <sys/types.h>
#import <sys/sysctl.h>
#import <mach/host_info.h>
#import <mach/mach_host.h>
#import <mach/task_info.h>
#import <mach/task.h>
 
-(void)logMemoryInfo {
    
    
    int mib[6];
    mib[0] = CTL_HW;
    mib[1] = HW_PAGESIZE;
    
    int pagesize;
    size_t length;
    length = sizeof (pagesize);
    if (sysctl (mib, 2, &pagesize, &length, NULL, 0) < 0)
    {
        fprintf (stderr, "getting page size");
    }
    
    mach_msg_type_number_t count = HOST_VM_INFO_COUNT;
    
    vm_statistics_data_t vmstat;
    if (host_statistics (mach_host_self (), HOST_VM_INFO, (host_info_t) &vmstat, &count) != KERN_SUCCESS)
    {
        fprintf (stderr, "Failed to get VM statistics.");
    }
    task_basic_info_64_data_t info;
    unsigned size = sizeof (info);
    task_info (mach_task_self (), TASK_BASIC_INFO_64, (task_info_t) &info, &size);
    
    double unit = 1024 * 1024;
    double total = (vmstat.wire_count + vmstat.active_count + vmstat.inactive_count + vmstat.free_count) * pagesize / unit;
    double wired = vmstat.wire_count * pagesize / unit;
    double active = vmstat.active_count * pagesize / unit;
    double inactive = vmstat.inactive_count * pagesize / unit;
    double free = vmstat.free_count * pagesize / unit;
    double resident = info.resident_size / unit;
    NSLog(@"===================================================");
    NSLog(@"Total:%.2lfMb", total);
    NSLog(@"Wired:%.2lfMb", wired);
    NSLog(@"Active:%.2lfMb", active);
    NSLog(@"Inactive:%.2lfMb", inactive);
    NSLog(@"Free:%.2lfMb", free);
    NSLog(@"Resident:%.2lfMb", resident);
}

```

#### 推荐博客：

- [质量监控-资源使用](https://www.jianshu.com/p/22a077fd51f1)