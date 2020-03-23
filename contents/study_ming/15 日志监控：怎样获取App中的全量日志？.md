> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 15 日志监控：怎样获取App中的全量日志？

#### 获取 NSLog 的日志

##### iOS 10 以前可以通过 ASL 接口来获取

NSLog 的作用是，输出信息到标准的 Error 控制台和系统日志(syslog)中。在内部实现上，它其实使用的是 ASL(Apple System Logger，是苹果公司自己实现的一套输出日志的接口)的 API，将日志消息直接存储在磁盘上。

ASL 会提供接口去查找所有的日志，参考 [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack) 中的 [DDASLLogCapture](https://github.com/CocoaLumberjack/CocoaLumberjack/blob/d55964a756fece9131a0fb77a4ffe61c3dc5b91e/Sources/CocoaLumberjack/DDASLLogCapture.m) 实现。通过 CocoaLumberjack 这个第三方日志库里的 DDASLLogCapture 这个类，我们可以找到实时捕获 NSLog 的方法。DDASLLogCapture 会在 start 方法里开启一个异步全局队列去捕获 ASL 存储的日志。

CocoaLumberjack 的用法可以参考其[官方文档](https://github.com/CocoaLumberjack/CocoaLumberjack/tree/master/Documentation)，或者这篇博客:[CocoaLumberjack：简单好用的Log库](https://www.jianshu.com/p/7b799bef0107)。

##### 通过 fishhook 库hook NSLog 方法重定向 NSLog 函数

iOS10 之后，新的日志系统(Unified Logging System)全面取代 ASL，没有 ASL 那样的接口可以让我们取出全部日志，所以为了兼容新的统一日志系统，可以对 NSLog 日志的输出进行重定向。NSLog 本身就是一个 C 函数，而不是 Objective-C 方法，所以我们可以使用 fishhook 来完成重定向的工作。

fishhook 原理课参考博客：[iOS逆向工程 - fishhook原理](https://www.jianshu.com/p/4d86de908721)

利用 fishhook hook NSLog 函数：

```
static void (&orig_nslog)(NSString *format, ...);

void redirect_nslog(NSString *format, ...) {
    // 可以在这里先进行自己的处理
    // 继续执行原 NSLog
    va_list va;
    va_start(va, format);
    NSLogv(format, va);
    va_end(va);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        struct rebinding nslog_rebinding = {"NSLog",redirect_nslog,(void*)&orig_nslog};
        NSLog(@"try redirect nslog %@,%d",@"is that ok?");
   }
    return;
}
```

#####  使用 dup2 函数和 STDERR 句柄重定向 NSLog 函数

NSLog 最后重定向的句柄是 STDERR，NSLog 输出的日志内容，最终都通过 STDERR 句柄来记录，而 dup2 函数式专门进行文件重定向的；可以使用 dup2 重定向 STDERR 句柄，将内容重定向指定的位置，如写入文件，上传服务器，显示到 View 上。

实现重定向，需要通过 NSPipe 创建一个管道，pipe 有读端和写端，然后通过 dup2 将标准输入重定向到 pipe 的写端。再通过 NSFileHandle 监听 pipe 的读端，最后再处理读出的信息。之后通过 printf 或者 NSLog 写数据，都会写到 pipe 的写端，同时 pipe 会将这些数据直接传送到读端，最后通过 NSFileHandle 的监控函数取出这些数据。

```
- (void)redirectSTD:(int )fd {
    
    NSPipe * pipe = [NSPipe pipe] ;
    NSFileHandle *pipeReadHandle = [pipe fileHandleForReading] ;
    int pipeFileHandle = [[pipe fileHandleForWriting] fileDescriptor];
    dup2(pipeFileHandle, fd) ;
    
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(redirectNotificationHandle:)
                                                 name:NSFileHandleReadCompletionNotification
                                               object:pipeReadHandle] ;
    [pipeReadHandle readInBackgroundAndNotify];
}
 
- (void)redirectNotificationHandle:(NSNotification *)nf {
    NSData *data = [[nf userInfo] objectForKey:NSFileHandleNotificationDataItem];
    NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding] ;
    //可以添加自己的处理,可以将内容显示到View,或者是存放到另一个文件中等等
    //todo
    
    
    [[nf object] readInBackgroundAndNotify];
}
 
//使用
[self redirectSTD:STDERR_FILENO];
```
