# iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(3)

#####文章共分为三篇：
第一篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(1)](https://www.jianshu.com/p/6985ec0666cb)
第二篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)](https://www.jianshu.com/p/72b5525e82e3)
第三篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(3)](https://www.jianshu.com/p/43fb6ccea4b2)

接上篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)](https://www.jianshu.com/p/72b5525e82e3)


## 第 6 章 块与大中枢派发
### 第 35 条：理解 “块”这一概念

`blcok`和函数类似，它是直接定义在另一个函数里的，和定义它的那个函数共享同一个范围的东西。用`^`来表示，后面接一对大括号，括号里是`blcok`的实现代码。

- 格式： 返回类型 （^blockName）(参数){实现代码};

```
    // 无返回值无参数
    void(^testBlcok)(void) = ^{
        NSLog(@"testBlcok");
    };
    testBlcok(); // print："testBlcok"

    // 无返回值有参数
    void(^testBlcok)(int a) = ^(int a) {
        NSLog(@"%d", a);
    };
    testBlcok(5); // print："5"

    // 有返回值有参数
    int (^addBlock)(int a, int b) = ^(int a, int b) {
        return a + b;
    };
    NSLog(@"%d", addBlock(13, 6)); // print："19"
```

- `block`可以访问局部变量，但是不能修改，如果修改局部变量，需要加`__block`
```
    __block NSInteger count = 0;
    int additional = 5;
    int (^addBlock)(int a, int b) = ^(int a, int b) {
        count = count + 10;
        return a + b + additional;
    };
    NSLog(@"addBlock = %d", addBlock(13, 6)); // print："24"
    NSLog(@"count = %ld", count); // print："10"
```

- 另外关于`block`的修饰符应该注意：
  1.如果用`copy`修饰`Block`，该`Block`就会存储在堆空间。则会对`Block`的内部对象进行强引用，导致循环引用。内存无法释放。
解决方法：新建一个指针`(__weak typeof(Target) weakTarget = Target )`指向`Block`代码块里的对象，然后用`weakTarget`进行操作。就可以解决循环引用问题。
  2.如果用`weak`修饰`Block`，该`Block`就会存放在栈空间。不会出现循环引用问题。

### 第 36 条：为常用的块类型创建 typedef

为常用的块类型创建 `typedef`，主要是为了代码的易读性，用的时候也较为方便。请看下面代码对比：

```
// 第一种写法
- (void)testWithBlockString:(NSString *)string withBlock:(void(^)(id dataSource))block;
/* 这种写法非常难记，也很难懂，用的时候不方便 */

// 第二种写法
typedef void (^testBlock)(id dataSource);
- (void)testWithBlockString:(NSString *)string withBlockName:(testBlock)block;
/* 用 typedef 关键字，为常用的块类型起个别名，方便易懂 */
```

### 第 37 条：用 handler 块降低代码分散程度
在`iOS`开发中，我们经常会异步处理一些任务，然后等任务执行结束后通知相关方法。实现此需求的方法有很多，比如可以选择代理委托，也可以选择`block`。`block `更轻型，使用更简单，能够直接访问上下文，这样类中不需要存储临时数据，使用` block `的代码通常会在同一个地方，这样使代码更连贯，可读性好。

```
typedef void (^testBlock)(id dataSource);
- (void)testWithBlockString:(NSString *)string withBlockName:(testBlock)block;

```

### 第 38 条：用块引用其所属对象时不要出现保留环
这条讲的比较基础，是`iOS`中`block`的循环引用问题。所谓循环引用，就是两个对象相互持有，这样就会造成循环引用。
- 请看下面代码：

```
typedef void (^testBlock)(id dataSource);
@property (copy, nonatomic) testBlock block;
@property (nonatomic, copy) NSString *blockString;

- (void)testBlock {
    self.block = ^(id dataSource) {
        NSString *blockString = self.blockString;
        NSLog(@"blockString = %@", blockString);
    };
}
```
- 代码截图：

![block 循环引用](http://upload-images.jianshu.io/upload_images/2665449-92afc35d076a4c15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 解决方法：
```
- (void)testBlock {
    __weak typeof(self) weakSelf = self;
    self.block = ^(id dataSource) {
        NSString *blockString = weakSelf.blockString;
        NSLog(@"blockString = %@", blockString);
    };
}
```

**但并非所有 block 都会造成循环引用**，在开发中，一些同学只要有`block`的地方就会用`__weak`来修饰对象，其实没有必要，以下几种`block`是不会造成循环引用的：

- 大部分`GCD`方法
```
    dispatch_async(dispatch_get_main_queue(), ^{
        NSString *blockString = self.blockString;
        NSLog(@"blockString = %@", blockString);
    });
```
代码解读：因为`self`并没有对`GCD`中的`block`进行持有，所以不会形成循环引用。

- `block` 属于另外一个类

```
    [NNHomeViewController testWithBlockName:^(id dataSource) {
        NSString *blockString = self.blockString;
        NSLog(@"blockString = %@", blockString);
    }];
```
代码解读：同上，`block`不是被`self`所持有的。

- `block`并不是属性值，而是临时变量

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self testWithBlock:^{
        NSString *blockString = self.blockString;
        NSLog(@"blockString = %@", blockString);
    }];
}

- (void)testWithBlock:(void(^)(void))block {
    block();
}
```


### 第 39 条：多用派发队列，少用同步锁
**在 OC 中，如果有多个线程要执行同一份代码，有时可能会出问题。这种情况下，通常要使用锁来实现某种同步机制。**

- GCD 出现之前通常使用两种方法：

第一种：内置的“同步块”

```
- (void)synchronizedMethod {
    @synchronized(self) {
        // safe
    }
}
```
这种写法会根据给定的对象，自动创建一个锁，并等待块中代码执行完毕。执行到这段代码结尾处，锁就释放了。但滥用`@synchronized(self)`会很大程度上降低代码效率，因此不推荐使用。

第二种：直接使用 NSLock 对象(也可以使用 NSRecursiveLock 递归锁)

```
_lock = [[NSLock alloc] init];

- (void)synchronizedMethod {
    [_lock lock];
    // safe
    [_lock unlock];
}
```
这种写法也有缺陷，在极端情况下，同步块会导致死锁，另外与 GCD 相比效率也很低。

- 推荐：GCD 方式

```
- (NSString *)testString {
    __block NSString *localTestString;
    dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        localTestString = self.testString;
    });
    return localTestString;
}

- (void)setTestString:(NSString *)testString {
    dispatch_barrier_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        self.testString = testString;
    });
}
```
使用同步队列及栅栏块，可以令同步行为更加高效。

### 第 40 条：多用 GCD，少用 performSelector 系列方法
在 `GCD` 出现之前，开发者延迟调用一些方法，或者指定运行方法的线程会用 `performSelector`，但是在 `GCD` 出来之后就不需要再使用`performSelector`了。

- performSelector 系列的方法：

```
- (id)performSelector:(SEL)aSelector;
- (id)performSelector:(SEL)aSelector withObject:(id)object;
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;

- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray<NSRunLoopMode> *)modes;
- (void)performSelector:(SEL)aSelector withObject:(nullable id)anArgument afterDelay:(NSTimeInterval)delay;

- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;

- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait modes:(nullable NSArray<NSString *> *)array;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg;
```

- performSelector 方法存在的缺点
  - 内存管理问题：在`ARC`下使用`performSelector`编译器经常出现警告，因为它无法确定将要执行的选择子具体是什么，因而`ARC`编译器也就无法插入适当的内存管理方法。
  - `performSelector`系列方法所能处理的选择子太过于局限，`performSelector`的返回值只能是`void`或对象类型；而且它无法处理带有多个参数的选择子，最多只能处理两个参数。

- 用 GCD 代替 performSelector 系列方法

1. 延迟调用方法：

```
// GCD
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)); dispatch_after(time, dispatch_get_main_queue(), ^(void){
    [self doSomething];
});

// performSelector
[self performSelector:@selector(doSomething) withObject:nil  afterDelay:5.0];
```
2. 指定运行方法的线程：

```
// GCD
dispatch_async(dispatch_get_main_queue(), ^{
        [self doSomething];
});

// performSelector
[self performSelectorOnMainThread:@selector(doSomething) withObject:nil waitUntilDone:NO];
```

### 第 41 条：掌握 GCD 及操作队列的使用时机
这条讲的是什么时候该用 `GCD`，什么时候不该用`GCD`。`GCD`技术确实很棒，但`GCD`并不总是最佳解决方案。比如**当我们想取消队列中的某个操作时，或者需要后台执行任务时**，这时我们可以用`NSOperationQueue`，其实`NSOperationQueue`跟`GCD`有很多相像之处。`NSOperationQueue`在`GCD`之前就已经有了，`GCD`就是在其某些原理上构建的。**`GCD`是`C`层次的`API`，而`NSOperation`是重量级的`OC`对象。**


- 使用`NSOperation`及`NSOperationQueue`的好处如下：
  - 取消某个操作。
  - 指定操作间的依赖关系。
  - 通过键值观察机制监控NSOperation对象的属性。
  - 指定操作的优先级。
  - 重用NSOperation对象。

### 第 42 条：通过 Dispatch Group 机制，根据系统资源状况来执行任务
`dispatch group`是`GCD`的一项特性，能够把任务分组。调用者可以等待这组任务执行完毕，也可以在提供回调函数之后继续往下执行，这组任务完成时，调用者会得到通知，开发者可以拿到结果然后继续下一步操作。

请看以下代码：

```
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    // 创建一个队列组
    dispatch_group_t group = dispatch_group_create();
    dispatch_group_async(group, queue, ^{
        // 添加操作...
        NSLog(@"1%@", [NSThread currentThread]);
    });
    dispatch_group_async(group, queue, ^{
        // 添加操作...
        NSLog(@"2%@", [NSThread currentThread]);
    });
    dispatch_group_async(group, queue, ^{
        // 添加操作...
        NSLog(@"3%@", [NSThread currentThread]);
    });
    // 收到通知，回到主线程刷新UI
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"回到主线程刷新UI");
    });
```

多个任务可归入一个`dispatch group`之中。开发者可以在这组任务执行完毕时获得通知。

### 第 43 条：使用 dispatch_once 来执行只需运行一次的线程安全代码
这条讲的单例模式，即常用的`dispatch_once`。使用 `dispatch_once` 可以简化代码并且彻底保证线程安全，我们根本无须担心加锁或同步，另外它没有使用重量级的同步机制，所以也更高效。

```
+ (id)shareInstance {
    static EOCClass *sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}
```

### 第 44 条：不要使用 dispatch_get_current_queue

- `dispatch_get_current_queue` 函数的行为常常与开发者所预期的不同，此函数已经废弃，只应做调试之用。
- 由于`GCD`是按层级来组织的，所以无法单用某个队列对象来描述"当前队列"这一概念。
- `dispatch_get_current_queue` 函数用于解决由不可以重入的代码所引发的死锁，然后能用此函数解决的问题，通常也可以用"队列特定数据"来解决。

## 第 7 章 系统框架
### 第 45 条：熟悉系统框架
开发者会碰到的主要框架就是`Fundation`。另外还有个与`Fundation`相伴的框架，叫做`CoreFoundation`。除了`Fundation`与`CoreFoundation`，还有很多系统库，其中包括但不限于下面列出的这些：

- CFNetwork：此框架提供了 C 语言级别的网络通信能力，它将 BSD 套接字`(BSD socket)`抽象成易于使用的网络接口。而 `Foundation` 则将该框架里的部分内容封装为`OC`语言的接口，以便进行网络通信。
- CoreAudio：此框架所提供的`C`语言`API`可以用来操作设备上的音频硬件。
- AVFoundation：此框架所提供的`OC`对象可用来回访并录制音频及视频，比如能够在UI视图类里播放视频。
- CoreData：此框架所提供的`OC`接口可以将对象放入数据库，将数持久保存。
- CoreText：此框架提供的C语言接口可以高效执行文字排版以及渲染操作。

- 要点：
  - 许多系统框架都可以直接使用。其中最重要的是`Fundation`与`CoreFoundation`，这两个框架提供了构架应用程序所需的许多核心功能。
  - 很多常见任务都能用框架来做。例如音频与视频处理、网络通信、数据管理等。
  - 用纯`C`写成的框架与用`OC`写成的一样重要，若想成为优秀的`OC`开发者，应该掌握`C`语言的核心概念。

### 第 46 条：多用块枚举，少用 for 循环
遍历`collection`有四种方式。最基本的办法是`for循环`，其次是`NSEnumerator遍历法`及`快速遍历法`，最新最先进的方式则是`“块枚举法”`。

- 四种遍历方法：

```
    NSArray *testArray = @[@1, @2, @3, @4, @5];
    // for 循环遍历
    for (int i = 0; i < testArray.count; i ++) {
        NSLog(@"testArray[%d] = %@", i, testArray[i]);
    }
    
    // NSEnumerator遍历法
    NSEnumerator *enumerator = [testArray objectEnumerator];
    id object;
    while ((object = [enumerator nextObject]) != nil) {
        NSLog(@"object = %@", object);
    }
    
    // 快速遍历
    for (NSObject *obj in testArray) {
        NSLog(@"obj = %@", obj);
    }
    
    // 块枚举遍历数组
    [testArray enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSLog(@"idx = %zd, obj = %@", idx, obj);
    }];

    // 块枚举遍历字典
    NSDictionary *testDic = @{@"name":@"liu zhong ning",@"age":@"25"};
    [testDic enumerateKeysAndObjectsUsingBlock:^(NSString * key,id object,BOOL * stop){
        NSLog(@"testDic[%@] = %@", key, object);
    }];
```
块枚举法拥有其他遍历方式都具备的优势，而且还能带来更多好处，在遍历字典的时候，还可以同时提供键和值，而且还有选项可以开启并发迭代功能，所以多写点代码还是值的。

### 第 47 条：对自定义其内存管理语义的`collection`使用无缝桥接
通过无缝桥接技术，可以在`Foundation`框架中的`OC`对象与`CoreFoundation`框架中的`C`语言数据结构之间来回转换。

- 简单的无缝桥接示例：

```
    NSArray *testNSArray = @[@1, @2, @3, @4, @5];
    CFArrayRef testCFArray = (__bridge CFArrayRef)testNSArray;
    NSLog(@"Size of array = %li", CFArrayGetCount(testCFArray));
    // Output：Size of array = 5
```

- __bridge：ARC仍然具备这个Objective对象的所有权。
- __bridge_retained:ARC将交出对象的所有权。
- __bridge_transfer：C转化为OC

想深入了解无缝桥接技术的童鞋可以点击这里：[iOS无缝桥接官方文档](https://developer.apple.com/library/content/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html)

### 第 48 条：构建缓存时选用 NSCache 而非 NSDictionary
- 实现缓存时应选用`NSCache`而非`NSDictionary`对象。因为`NSCache`可以提供优雅的自动删减功能，而且是“线程安全的”，此外，它与字典不同，并不会拷贝键。
- 可以给`NSCache`对象设置上限，用以限制缓存中的对象总个数及“总成本”，而这些尺度则定义了缓存删减其中对象的时机。但是绝对不要把这些尺度当成可靠的“硬限制”，他们仅对`NSCache`起指导作用。
- 将`NSPurgeableData`与`NSCache`搭配使用，可实现自动清除数据的功能，也就是说，当`NSPurgeableData`对象所占内存为系统所丢弃时，该对象自身也会从缓存中移除。
- 如果缓存使用得当，那么应用程序的相应速度就能提高。只有那种“重新计算起来很费事的”数据，才值得放入缓存，比如那些需要从网络获取或从磁盘读取的数据。

### 第 49 条：精简 initialize 与 load 的实现代码

- 当程序启动的时候，类和分类，必定会调动且仅调用一次`load`方法。先调用类的`load`方法，再调用分类的`load`方法。先调用超类的`load`方法，再调用子类的`load`方法。`load`方法需要实现得精简一些，因为整个应用程序会在执行`load`方法时都会阻塞。
- `initialize`方法会在程序首次用该类之前调用，且只调用一次。`initialize`是“懒加载”的，如果某个类一直都没有使用，就不会执行该类的`initialize`方法。`initialize`方法可以安全使用并调用任意类中的任意方法。`initialize`方法只应该用来设置内部数据，不应该在其中调用其他方法。
- 在加载阶段，如果类实现了`load`方法，那么系统就会调用它。分类里也可以定义此方法，类的`load`方法要比分类中的先调用。与其他方法不同，`load`方法不参与覆写机制。
- 首次使用某个类之前，系统会向其发送`initialize`消息。由于此方法遵从普通的覆写规则，所以通常应该在里面判断当前要初始化的是哪个类。
- `load`与`initialize` 方法都应该实现的精简一点，这样有助于保持应用程序的响应能力，也可以减少引入依赖环的几率。
- 无法在编译器设定的全局变量，可以放在`initialize`方法里初始化。

### 第 50 条：别忘了 NSTimer 会保留其目标对象

开发中经常会用到NSTimer，由于定时器NSTimer会保留其目标对象，所以反复执行任务通常会导致应用程序出问题，也就是说很容易造成循环引用。请看以下代码：

```
#import <Foundation/Foundation.h>

@interface NNTimer: NSObject

- (void)startPolling;
- (void)stopPolling;

@end

@implementation NNTimer {
    NSTimer *_pollTimer;
}

- (id)init {
    return [super init];
}

- (void)dealloc {
    [_pollTimer invalidate];
}

- (void)stopPolling {
    
    [_pollTimer invalidate];
    _pollTimer = nil;
}

- (void)startPolling {
    _pollTimer = [NSTimer scheduledTimerWithTimeInterval:5.0
                                                  target:self
                                                selector:@selector(p_doPoll)
                                                userInfo:nil
                                                 repeats:YES];
}

- (void)p_doPoll {
    // Poll the resource
}

@end
```

上面这段代码是存在问题的。如果创建了本类的实例，并调用其startPolling方法，那么会如何呢？创建计时器的时候，由于目标对象是self，所以要保留此实例。然而，因为计时器是用实例变量存放的，所以实例也保留了计时器，于是就产生了保留环。

单从计时器本身入手，你会发现很难解决这个问题，那么如何解决这个问题呢？我们可以通过“块”来解决。虽然计时器当前不直接支持块，但是可以用下面这段代码为其添加此功能：
.h文件：

```
@interface NSTimer (NNBlocksSupport)
+ (NSTimer *)nn_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                          block:(void(^)(void))block
                                       repeats:(BOOL)repeats;
@end
```
.m文件：

```
#import "NSTimer+NNBlocksSupport.h"

@implementation NSTimer (NNBlocksSupport)

+ (NSTimer *)nn_scheduledTimerWithTimeInterval:(NSTimeInterval)interval
                                         block:(void(^)(void))block
                                       repeats:(BOOL)repeats {
    return [self scheduledTimerWithTimeInterval:interval
                                         target:self
                                       selector:@selector(nn_blockInvoke:)
                                       userInfo:[block copy]
                                        repeats:repeats];
    
}

+ (void)nn_blockInvoke:(NSTimer *)timer {
    void (^block)(void) = timer.userInfo;
    if (block) {
        block();
    }
}

@end
```

结束语：由于个人能力有限，这三篇读书笔记难免有错误或不足之处，还望各位道友能不吝赐教，谢谢。

最后安利一下这本书：[PDF版](https://pan.baidu.com/s/1eT3gfzs)