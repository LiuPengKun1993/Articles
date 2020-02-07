# iOS开发之 - 多线程

![iOS开发之 － 多线程](http://upload-images.jianshu.io/upload_images/2665449-72633919539a5942.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 最近利用晚上的空闲时间，整理了一下多线程相关的知识，几经更改，就变成你现在看到的模样了（如有失误之处，还请不吝赐教）。把多线程相关的知识总结在这里，只希望对看到的朋友们有所帮助！文章有些长，涵盖的知识点也比较多，希望朋友们能够耐心读完哈！我们这篇文章主要包括以下几个模块：

- **（一）iOS开发之多线程 基础知识**
- **（二）iOS开发之多线程 NSThread**
- **（三）iOS开发之多线程 NSOperation**
- **（四）iOS开发之多线程 GCD**
- **（五）iOS开发之多线程 线程间的通信**

*下面我们开始了解 iOS 多线程的基础知识，如果在此之前对多线程已经有所了解的朋友，大可跳过这一部分，直接看后边三种多线程编程的 demo 即可！*

## （一）iOS 开发之多线程 基础知识

### 1.多线程简介:
一个进程要想执行任务, 必须得有线程(每个进程至少要有一条线程), 线程是进程的基本执行单元, 一个进程（程序）的所有任务都在线程中执行.

### 2.多线程的原理:
 
同一时间, CPU 只能处理一条线程,只有一条线程在工作.
多线程并发(同时)执行, 其实是 CPU 快速地在多条线程之间调度.
如果 CPU 调度线程的时间足够快,就造成了多线程并发执行的假象.
 
### 3.多线程的优缺点:
 
#### 多线程的优点:
能适当提高程序的执行效率.
能适当提高资源利用率(CPU、内存利用率).
 
#### 多线程的缺点:
开启线程需要占用一定的内存空间(默认情况下，主线程占用1M，子线程占用512KB),如果开启大量的线程,会占用大量的内存空间,降低程序的性能.
 
### 4.多线程在 iOS 开发中的应用:

主线程:一个 iOS 程序运行后，默认会开启1条线程，称为“主线程”或“ UI 线程”.
主线程的主要作用:刷新 UI 界面、处理 UI 事件(比如点击事件、滚动事件、拖拽事件等).
 

### 5.iOS 中三种多线程编程的技术，分别是：
### （一）NSThread
### （二）Cocoa NSOperation
### （三）GCD（全称：Grand Central Dispatch）

这三种编程方式从上到下，抽象度层次是从低到高的，抽象度越高的使用越简单，也是Apple最推荐使用的。

#### 三种方式的优缺点介绍：

##### NSThread:
优点：NSThread 比其他两个轻量级
缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销。


##### Cocoa NSOperation
优点：不需要关心线程管理，数据同步的事情，可以把精力放在自己需要执行的操作上。
Cocoa operation 相关的类是 NSOperation ，NSOperationQueue。
NSOperation是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的两个子类：NSInvocationOperation 和 NSBlockOperation。
创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行。

##### GCD
Grand Central Dispatch (GCD)是Apple开发的一个多核编程的解决方法。在iOS4.0开始之后才能使用。GCD是一个替代诸如NSThread, NSOperationQueue, NSInvocationOperation等技术的很高效和强大的技术。

---

*对 iOS 基础知识有了一定了解之后，下边我们开始看多线程相关的一些 demo， 建议朋友们试试下边的 demo，这样更方便理解！如有疑问，或发现某些 demo 写的有误，还请朋友们告知！*

**下边我们开始了解NSThread，NSThread 其实用的真不多，主要还是用另外两种，不过作为 iOS 开发者，还是应该对它有些了解的，下边是 NSThread 相关的 demo ！**
## （二）iOS 开发之多线程 NSThread

### 一、NSThread 初始化

#### 1.动态方法

```objc
- (id)initWithTarget:(id)target selector:(SEL)selector object:(id)argument 
// 初始化线程 
NSThread* thread = [[NSThread alloc] initWithTarget:self selector:@selector(doSomething:)object:nil];
// 线程名字 
thread.name = @"MYThread";
 // 启动线程 
[thread start];
```

#### 2.静态方法
```objc
+ (void)detachNewThreadSelector:(SEL)aSelector toTarget:(id)aTarget withObject:(id)anArgument
[NSThread detachNewThreadSelector:@selector(doSomething:) toTarget:self withObject:nil];   
// 调用完毕后，会马上创建并开启新线程 
```

#### 3.隐式创建线程的方法：
```objc
- (void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg;
[self performSelectorInBackground:@selector(run) withObject:nil];  
```

**提示朋友们一下:第一种方式会直接创建线程并且开始运行线程，第二种方式是先创建线程对象，然后再运行线程操作，在运行线程操作前可以设置线程的优先级等线程信息**

### 二、线程间的通信
```objc
//在主线程上执行操作
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES];  
//在当前线程执行操作
[self performSelector:@selector(run) withObject:nil];  
//在指定线程上执行操作
[self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES]; 
```

### 三、一些简单方法
```objc
//获取当前线程
NSThread *thread = [NSThread currentThread];
//获取主线程
NSThread *main = [NSThread mainThread]; 
//睡眠，相当于暂停
[NSThread sleepForTimeInterval:3];
```


---

*NSThread  就说到这，一起再看下NSOperation， NSOperation 在开发中用的还是挺多的，虽然苹果建议使用 GCD ，但依然有开发者觉得 NSOperation  比 GCD 更好用！仁者见仁智者见智吧！本小白觉得如果能掌握还是都掌握的好！*
## （三）iOS开发之多线程 NSOperation

### 一、创建线程的方式

使用NSOperation创建线程的方式有3种:

（1）NSInvocationOperation 方式
（2）NSBlockOperation 方式
（3）自定义子类继承NSOperation,实现内部相应的⽅法

下面是这三种方式创建线程的具体方法：
####1.NSInvocationOperation

```objc
- (void)invocationOperation
{
//创建NSInvocationOperation对象
    NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];

//开始执行操作
    [operation start];
}
//一旦执行操作,就会调用target的run方法
```
提示:默认情况下,调用了start方法后并不会开一条新线程去执行操作,只有将NSOperation放到一个NSOperationQueue中,才会异步执行操作.


#### 2.NSBlockOperation
```objc
- (void)blockOperation
{
//创建NSBlockOperation对象
    NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        // 主线程
        NSLog(@"1---%@", [NSThread currentThread]);
    }];
    // 添加任务(在子线程执行)
    [operation addExecutionBlock:^{
        NSLog(@"2---%@", [NSThread currentThread]);
    }];
    [operation addExecutionBlock:^{
        NSLog(@"3---%@", [NSThread currentThread]);
    }];
    //开启执行操作
    [operation start];
}
```
提示:只要NSBlockOperation封装的操作数 > 1,就会异步执行操作


#### 3.自定义子类继承自NSOperation

自定义NSOperation子类需要重写 - (void)main方法

```objc
- (void)main
{
    // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
    @autoreleasepool {
    for (NSInteger i = 0; i<100; i++) {
        NSLog(@"1 - %ld - %@", i, [NSThread currentThread]);
    }
    //检测操作是否被取消,对取消做出响应
    if (self.isCancelled) return;
    
    for (NSInteger i = 0; i<100; i++) {
        NSLog(@"2 - %ld - %@", i, [NSThread currentThread]);
    }
    if (self.isCancelled) return;
    
    for (NSInteger i = 0; i<100; i++) {
        NSLog(@"3 - %ld - %@", i, [NSThread currentThread]);
    }
    if (self.isCancelled) return;
    }
}
```

### 二、NSOperationQueue的应用

一个NSOperation对象可以通过调用start方法来执行任务,默认是同步执行的.也可以将NSOperation添加到一个NSOperationQueue中去执行,而且是异步执行的.

#### 1.NSOperationQueue简单创建
```objc
//第一种方式
- (void)operationQueue
{
    // 创建NSOperationQueue队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 创建NSInvocationOperation对象
    NSInvocationOperation *operation1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run1) object:nil];
    NSInvocationOperation *operation2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run2) object:nil];
    
  // 创建NSBlockOperation对象
    NSBlockOperation *operation3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1 --- %@", [NSThread currentThread]);
    }];
    [operation3 addExecutionBlock:^{
        NSLog(@"2 --- %@", [NSThread currentThread]);
    }];
    NSBlockOperation *operation4 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3 --- %@", [NSThread currentThread]);
    }];
    // 添加任务到队列中
    [queue addOperation:operation1]; 
    [queue addOperation:operation2]; 
    [queue addOperation:operation3]; 
    [queue addOperation:operation4]; 
}


//第二种方式
- (void)operationQueue2
{
    // 创建NSOperationQueue队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    [queue addOperationWithBlock:^{
        NSLog(@"1 --- %@", [NSThread currentThread]);
    }];
    [queue addOperationWithBlock:^{
        NSLog(@"2 --- %@", [NSThread currentThread]);
    }];
}
```

#### 2.NSOperation中的操作依赖
NSOperation之间可以设置依赖来保证执行顺序,比如一定要让操作A执行完后,才能执行操作B,可以这么写[operationB addDependency:operationA]; // 操作B依赖于操作operationA,另外,通过removeDependency方法可以删除依赖对象.

```objc
#pragma mark - 依赖操作调整执行顺序
- (void)NSOperationTest1 {
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"~~1~~%@", [NSThread currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 5; i ++) {
            NSLog(@"~~2~~%@~~%d~~", [NSThread currentThread], i);
        }
    }];
    NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"~~3~~%@", [NSThread currentThread]);
    }];
    [op2 addExecutionBlock:^{
        NSLog(@"~~4~~%@", [NSThread currentThread]);
    }];
    op3.completionBlock = ^{
        NSLog(@"~~5~~%@", [NSThread currentThread]);
    };
      // 设置依赖
    [op1 addDependency:op2];
    [op1 addDependency:op3];
    [queue addOperation:op1];
    [queue addOperation:op2];
    [queue addOperation:op3];
}
```

#### 3.线程间的通信
```objc
#pragma mark - NSOperation实现线程间的通信
- (void)NSOperationTest2 {
    [[[NSOperationQueue alloc] init] addOperationWithBlock:^{
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://img.lanrentuku.com/img/allimg/1310/13822295641903.jpg"]];
        
        UIImage *image = [UIImage imageWithData:data];
        
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            self.imageview.image = image;
        }];
    }];
}
```

### 三、一些简单方法
```objc
// 取消单个操作  
[operation cancel];  
// 取消queue中所有的操作  
[queue cancelAllOperations]; 
//设置队列的最大并发操作数量
queue.maxConcurrentOperationCount = 1; 
// 会阻塞当前线程，等到某个operation执行完毕
[operation waitUntilFinished];  
// 阻塞当前线程，等待queue的所有操作执行完毕
[queue waitUntilAllOperationsAreFinished];
// 暂停queue YES代表暂停队列,NO代表恢复队列
[queue setSuspended:YES];
```
---
*NSOperation 常用的基本上也就上面那些了。。。GCD 就不用多说了，老东家苹果推荐的，不过我却觉得名字太长~~~，一开始总是记不住，多敲就好了！所有编程语言大概都有这么一个绝招O(∩_∩)O！*
## （四）iOS开发之多线程 GCD

## 一、GCD 术语
要理解GCD ，我们先来熟悉与线程和并发相关的几个概念。

#### 1.Serial vs. Concurrent 串行 vs. 并发
任务串行执行就是每次只有一个任务被执行，任务并发执行就是在同一时间可以有多个任务被执行。

#### 2.同步（Synchronous）与异步（Asynchronous）
同步，意味着在当前线程中执行任务，不具备开启新的线程的能力。
异步，在新的线程中执行任务，具备开启新的线程的能力。

#### 3.临界区（Critical Section）
就是一段代码不能被并发执行，即两个线程不能同时执行这段代码。

#### 4.死锁（Deadlock）
停止等待事情的线程会导致多个线程相互维持等待，即死锁。

#### 5.Thread Safe 线程安全
线程安全的代码能在多线程或并发任务中被安全的调用，而不会导致任何问题（数据损坏，崩溃等）。

#### 6.Queues 队列
GCD 提供有 dispatch queues 来处理代码块，这些队列管理你提供给 GCD 的任务并用 FIFO 顺序执行这些任务。

### 二、代码演示

### 1、dispatch_async(异步函数)

```objc
//异步函数 + 并发队列:可以同时开启多条线程,任务是并行的
// 1.创建并发队列
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
      //2.将任务加入队列
    dispatch_async(queue, ^{
        for (NSInteger i = 0 ; i < 5; i ++) {
            NSLog(@"%ld~~~~1~~~~%@", i, [NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"%ld~~~2~~~%@", i, [NSThread currentThread]);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"%ld~~~~3~~~~%@", i, [NSThread currentThread]);
        }
    });
//异步函数 + 串行队列:会开启新的线程，但是任务是串行的，执行完一个任务，再执行下一个任务
// 1.创建串行队列
    dispatch_queue_t queue = dispatch_queue_create("YMWM", DISPATCH_QUEUE_SERIAL);
// 2.将任务加入队列
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5 ; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"2~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"2~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
```
    
#### 2、dispatch_sync(同步函数)

```objc
   //同步函数 + 并发队列:不会开启新的线程
    // 1.获得全局的并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    // 2.将任务加入队列
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
});
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"2~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
});
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"3~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
});
//同步函数 + 串行队列:不会开启新的线程
    // 1.创建串行队列
    dispatch_queue_t queue = dispatch_queue_create("YMWM", DISPATCH_QUEUE_SERIAL);
    // 2.将任务加入队列
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"2~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"3~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
});
```
    
#### 3.两者在主队列中的应用
```objc
    //异步函数 + 主队列：只在主线程中执行任务
    dispatch_queue_t queue = dispatch_get_main_queue();
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"2~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_async(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"3~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    //同步函数 + 主队列：死循环
    dispatch_queue_t queue = dispatch_get_main_queue();
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
    dispatch_sync(queue, ^{
        for (NSInteger i = 0; i < 5; i ++) {
            NSLog(@"1~~~%@~~~~~%ld", [NSThread currentThread], i);
        }
    });
```
  
#### 4、dispatch_group_async的使用

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
// 创建一个队列组
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{
//添加操作...
   [NSThread sleepForTimeInterval:1];
   NSLog(@"1%@", [NSThread currentThread]);
});
dispatch_group_async(group, queue, ^{
//添加操作...
   [NSThread sleepForTimeInterval:1];
   NSLog(@"2%@", [NSThread currentThread]);
});
dispatch_group_async(group, queue, ^{
//添加操作...
   [NSThread sleepForTimeInterval:1];
   NSLog(@"3%@", [NSThread currentThread]);
});
// 回到主线程
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
   NSLog(@"回到主线程刷新UI");
});  
```
    
#### 5、dispatch_barrier_async的使用

```objc
dispatch_queue_t queue = dispatch_queue_create("YMWM", DISPATCH_QUEUE_CONCURRENT);   
dispatch_async(queue, ^{   
    [NSThread sleepForTimeInterval:.5];   
    NSLog(@"1%@", [NSThread currentThread]);  
});   
dispatch_async(queue, ^{   
    [NSThread sleepForTimeInterval:.5];   
    NSLog(@"2%@", [NSThread currentThread]);  
});   
dispatch_barrier_async(queue, ^{   
    NSLog(@"barrier --- %@", [NSThread currentThread]); 
    [NSThread sleepForTimeInterval:.5];  
});   
dispatch_async(queue, ^{   
    NSLog(@"3%@", [NSThread currentThread]);   
});
dispatch_async(queue, ^{   
    NSLog(@"4%@", [NSThread currentThread]);   
});
```

#### 6、dispatch_apply的使用
```objc
  //执行某个代码片段10次
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_apply(10, queue, ^(size_t index) {
        NSLog(@"%@", [NSThread currentThread]);
    });
```

#### 7、dispatch_once_t的使用
第一个参数predicate，该参数是检查后面第二个参数所代表的代码块是否被调用的谓词，第二个参数则是在整个应用程序中只会被调用一次的代码块
```objc
static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSLog(@"%@", [NSThread currentThread]);
    });
```
   
#### 8.dispatch_after的使用
```objc
  - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"-------");
    //延迟执行，较为精确
   dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
       NSLog(@"~~~~~~");
   });
}
```
#### 9.线程间的通信
```objc
#pragma mark - GCD-线程间的通信
- (void)downloadImage {
    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://img.lanrentuku.com/img/allimg/1310/13822295641903.jpg"]];
    
    UIImage *image = [UIImage imageWithData:data];
    
    dispatch_async(dispatch_get_main_queue(), ^{
        
        self.imageview.image = image;
    });
}
```

#### 10.文件剪切
```objc
//第一种方式
	//将文件从from剪切至to
    NSString *from = @"/Users/iOS/Desktop/Test";
    NSString *to = @"/Users/iOS/Desktop/test1";
    NSFileManager *mgr = [NSFileManager defaultManager];
    NSArray *subpaths = [mgr subpathsAtPath:from];
    for (NSString *subpath in subpaths) {
        NSString *fromFullpath = [from stringByAppendingPathComponent:subpath];
        NSString *toFullpath = [to stringByAppendingPathComponent:subpath];
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //剪切
            [mgr moveItemAtPath:fromFullpath toPath:toFullpath error:nil];
        });
    }
    //第二种方式
    //将文件从from剪切至to
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    NSString *from = @"/Users/iOS/Desktop/Test";
    NSString *to = @"/Users/iOS/Desktop/test1";
    NSFileManager *mag = [NSFileManager defaultManager];
    NSArray *subpaths = [mag subpathsAtPath:from];
    dispatch_apply(subpaths.count, queue, ^(size_t index) {
        NSString *subpath = subpaths[index];
        NSString *fromFullpath = [from stringByAppendingPathComponent:subpath];
        NSString *toFullpath = [to stringByAppendingPathComponent:subpath];
//        剪切
        [mag moveItemAtPath:fromFullpath toPath:toFullpath error:nil];
        NSLog(@"%@~~~~~~%@", [NSThread currentThread], subpath);
    });
```



### 三、GCD与非GCD实现单粒设计模式

#### 1.GCD实现设计模式

类的.h文件

```objc
@interface PKPerson : NSObject
+ (instancetype)sharedCar;
@end
```

类的.m文件

```objc
@interface PKPerson() <NSCopying>

@end

@implementation PKPerson

// 实例变量，当前类
static id _instance;

// 重写allocWithZone方法
+ (instancetype)allocWithZone:(struct _NSZone *)zone{

    static dispatch_once_t onceToken;

    // 该方法整个应用程序只调用一次
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    return _instance;
}

+ (instancetype)sharedInstance{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [[self alloc] init];
    });
    return _instance;
}

// 保证每复制一个对象也是同一内存空间
- (id)copyWithZone:(NSZone *)zone{
    return _instance;
}
@end
```

#### 2.宏定义封装GCD单粒设计模式
通常当多个类之间有相同的属性和方法时，我们会考虑将相同的部分抽取出来，封装到父类中，但单粒模式不可以这样做，因为单粒模式采用继承方式的话，所有的类会共享一份内存。所以一般采取宏定义封装GCD单粒设计模式。注意，每行的结尾需要添加 "\"。

```objc
// .h文件
#define PKSingletonH + (instancetype)sharedInstance;

// .m文件
#define PKSingletonH \
static id _instace; \
 \
+ (instancetype)allocWithZone:(struct _NSZone *)zone \
{ \
    static dispatch_once_t onceToken; \
    dispatch_once(&onceToken, ^{ \
        _instace = [super allocWithZone:zone]; \
    }); \
    return _instace; \
} \
 \
+ (instancetype)sharedInstance \
{ \
    static dispatch_once_t onceToken; \
    dispatch_once(&onceToken, ^{ \
        _instace = [[self alloc] init]; \
    }); \
    return _instace; \
} \
 \
- (id)copyWithZone:(NSZone *)zone \
{ \
    return _instace; \
}
```


#### 3.非GCD实现设计模式
类的.h文件

```objc
@interface PKPerson : NSObject

+ (instancetype)sharedInstance;

@end
```

类的.m文件

```objc
#import "PKPerson.h"

@interface PKPerson()

@end

@implementation PKPerson

static id _instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
	// 同步锁，防止多线程同时进入
    @synchronized(self) {
        if (_instance == nil) {
            _instance = [super allocWithZone:zone];
        }
    }
    return _instance;
}

+ (instancetype)sharedInstance
{
    @synchronized(self) {
        if (_instance == nil) {
            _instance = [[self alloc] init];
        }
    }
    return _instance;
}

- (id)copyWithZone:(NSZone *)zone
{
    return _instance;
}
@end
```

### 四、GCD定时器

```objc
@interface ViewController ()
/** 定时器(这里不用带*，因为dispatch_source_t就是个类，内部已经包含了*) */
@property (nonatomic, strong) dispatch_source_t timer;
@end

@implementation ViewController

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    // 获得队列
    dispatch_queue_t queue = dispatch_get_main_queue();
    
    // 创建一个定时器(dispatch_source_t本质还是个OC对象)
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    
    // GCD的时间参数一般是纳秒（1秒 == 10的9次方纳秒）
    dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
    uint64_t interval = (uint64_t)(1.0 * NSEC_PER_SEC);
    dispatch_source_set_timer(self.timer, start, interval, 0);
    
    // 设置回调
    dispatch_source_set_event_handler(self.timer, ^{
        NSLog(@"------%@-------", [NSThread currentThread]);
    });
    
    // 启动定时器
    dispatch_resume(self.timer);
}
```
---

*三种多线程编程都说完了，代码有些多，不过希望朋友们有所收获，另外再补充一些关于线程间通信的。。。*
## （五）iOS开发之多线程 线程间的通信
### 一、简单介绍
在一个进程中，线程往往不是孤立存在的，多个线程之间需要经常进行通信。以下是线程之间进行通信的方法:

### 二、常用方法:

#### 代码1:performSelector
performSelector常用方法的常用方法主要有以下几种:

```objc
//在主线程上执行操作
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES];  

//在当前线程执行操作
[self performSelector:@selector(run) withObject:nil];  

//在指定线程上执行操作
[self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES];
```

具体代码:

```objc
//点击屏幕开始执行以下方法
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    [self performSelectorInBackground:@selector(download) withObject:nil];
}
- (void)download
{
    // 图片的网络路径
    NSURL *url = [NSURL URLWithString:@"http://img.lanrentuku.com/img/allimg/1310/13822295641903.jpg"];
    
    // 根据地址加载图片
    NSData *data = [NSData dataWithContentsOfURL:url]; //耗时操作
    
    // 生成图片
    UIImage *image = [UIImage imageWithData:data];
    
    // 回到主线程，刷新UI界面
    //方式一
    [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:NO];
    
    //方式二
//    [self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:NO];

    //方式三
//    [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:NO];
}

//显示图片(如果performSelector调用的是setImage:方法，会直接调用系统的方法)
//- (void)showImage:(UIImage *)image
//{
//    self.imageView.image = image;
//}
```
#### 代码2:NSOperationQueue方式

一个NSOperation对象可以通过调用start方法来执行任务,默认是同步执行的.也可以将NSOperation添加到一个NSOperationQueue中去执行,而且是异步执行的.

```objc
#pragma mark - NSOperation实现线程间的通信
- (void)NSOperationTest {
	 [[[NSOperationQueue alloc] init] addOperationWithBlock:^{
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://img.lanrentuku.com/img/allimg/1310/13822295641903.jpg"]];
        UIImage *image = [UIImage imageWithData:data];
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            self.imageView.image = image;
        }];
    }];
}
```

#### 代码3:GCD方式
GCD，全称Grand Central Dispath，是苹果开发的一种支持并行操作的机制。它的主要部件是一个FIFO队列和一个线程池，前者用来添加任务，后者用来执行任务。

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

 // 图片的网络路径
        NSURL *url = [NSURL URLWithString:@"http://img.lanrentuku.com/img/allimg/1310/13822295641903.jpg"];
        
 // 加载图片
        NSData *data = [NSData dataWithContentsOfURL:url];
        
 // 生成图片
        UIImage *image = [UIImage imageWithData:data];
        
 // 回到主线程显示图片
        dispatch_async(dispatch_get_main_queue(), ^{
            self.imageView.image = image;
        });
    });
```

### 三、NSPort方法:
NSPort有3个子类，NSSocketPort、NSMessagePort、NSMachPort，但在iOS下只有NSMachPort可用。代码如下:

UIViewController中的代码:

```objc
#import "ViewController.h"
#import "MyPort.h"

@interface ViewController () <NSPortDelegate>
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    //创建主线程的port
    NSPort *myPort = [NSPort port];
    
    //设置代理
    myPort.delegate = self;
    
    //把port加入runloop
    [[NSRunLoop currentRunLoop] addPort:myPort forMode:NSDefaultRunLoopMode];
    
    //启动子线程,并传入主线程的port
    MyPort *work = [[MyPort alloc] init];
    
    [NSThread detachNewThreadSelector:@selector(launchThreadWithPort:)
                             toTarget:work
                           withObject:myPort];
}

- (void)handlePortMessage:(NSMessagePort*)message{
    NSLog(@"已接到子线程的消息 --- %@", message);
}
@end
```

MyPort中的代码:

.h文件:

```objc
#import <Foundation/Foundation.h>

@interface MyPort : NSPort

@end
```

.m文件:

```objc
#import "MyPort.h"
#define PKMessage 10

@interface MyPort ()<NSMachPortDelegate> {
    NSPort *firstPort;
    NSPort *secondPort;
}
@end

@implementation MyPort

- (void)launchThreadWithPort:(NSPort *)port {
    @autoreleasepool {
    
        //接收主线程传入的port

        //设置子线程名字
        [[NSThread currentThread] setName:@"MyThread"];
        
        //开启runloop
        [[NSRunLoop currentRunLoop] run];
        
        //创建自己port
        secondPort = [NSPort port];
        secondPort.delegate = self;
        
        //将自己的port添加到runloop,防止runloop执行完毕之后退出,接收主线程发送过来的port消息
        [[NSRunLoop currentRunLoop] addPort:secondPort forMode:NSDefaultRunLoopMode];
        
        //完成向主线程发送消息
        [self sendPortMessage];
    }
}
/**
 *   完成向主线程发送消息
 */
- (void)sendPortMessage {
    //发送消息到主线程
    [firstPort sendBeforeDate:[NSDate date]
                         msgid:PKMessage
                    components:nil
                          from:secondPort
                      reserved:0];
}
```

提示:通常来说, NSPort可以做的事情通过performSelector也同样可以搞定，因此线程间通信完全可以通过performSelector来实现。
  **<end>**

---

**结束语：以上是多线程相关的知识总结，本人也是小白级别，如果有写错的地方，还请朋友们能够指出来，谢谢！**
