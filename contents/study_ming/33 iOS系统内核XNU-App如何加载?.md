> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 33 iOS系统内核XNU-App如何加载?

#### iOS 系统架构

iOS 系统是基于 ARM 架构的，大致可以分为四层:

- 最上层是用户体验层，主要是提供用户界面。这一层包含了 SpringBoard、Spotlight、Accessibility。
- 第二层是应用框架层，是开发者会用到的。这一层包含了开发框架 Cocoa Touch。
- 第三层是核心框架层，是系统核心功能的框架层。这一层包含了各种图形和媒体核心框架、Metal 等。
- 第四层是 Darwin层，是操作系统的核心，属于操作系统的内核态。这一层包含了系统内核 XNU、驱动等。

![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_XNU.jpg)

其中，用户体验层、应用框架层和核心框架层，属于用户态，是上层 App 的活动空间。Darwin是用户态的 下层支撑，是iOS系统的核心。

##### XNU

XNU 内部由 Mach、BSD、驱动 API IOKit 组成，这些都依赖于 libkern、libsa、Platform Expert。如下图所示:

![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_XNU01.jpg)

其中，[Mach](https://developer.apple.com/library/archive/documentation/Darwin/Conceptual/KernelProgramming/Mach/Mach.html) 是作为 UNIX 内核的替代，主要解决 UNIX一切皆文件导致抽象机制不足的问题，为现代操作系 统做了进一步的抽象工作。 Mach 负责操作系统最基本的工作，包括进程和线程抽象、处理器调度、进程间 通信、消息机制、虚拟内存管理、内存保护等。

进程对应到 Mach 是 Mach Task，Mach Task 可以看做是线程执行环境的抽象，包含虚拟地址空间、IPC 空 间、处理器资源、调度控制、线程容器。

进程在 BSD 里是由 BSD Process 处理，BSD Process 扩展了 Mach Task，增加了进程 ID、信号信息等， BSD Process 里面包含了扩展 Mach Thread 结构的 Uthread。

Mach 的模块包括进程和线程都是对象，对象之间不能直接调用，只能通过 Mach Msg 进行通信，也就是 mach_msg() 函数。在用户态的那三层中，也就是在用户体验层、应用框架层和核心框架层中，你可以通过 mach_msg_trap() 函数触发陷阱，从而切至 Mach，由 Mach 里的 mach_msg() 函数完成实际通信，具体实 现可以参看 NSHipster 的这篇文章“[Inter-Process Communication](https://nshipster.com/inter-process-communication/)”。

总体来说，XNU 加载就是为 Mach-O 创建一个新进程，建立虚拟内存空间，解析Mach-O文件，最后映射到内存空间。流程可以概括 为:

1. fork 新进程;
2. 为 Mach-O 分配内存;
3. 解析 Mach-O;
4. 读取 Mach-O 头信息;
5. 遍历 load command 信息，将 Mach-O 映射到内存; 6. 启动 dyld。