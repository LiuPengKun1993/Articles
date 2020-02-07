# iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)

##### 文章共分为三篇：


第一篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(1)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(1).md)

第二篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(2).md)

第三篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(3)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(3).md)

接上篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(1)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(1).md)

---

## 第 3 章：接口与 API 设计
### 第 13 条：用前缀避免命名空间冲突
`OC`没有其他语言那种内置的命名空间机制。因此我们起名时要设法避免潜在的命名冲突，否则很容易重名，引发命名冲突`（naming clash）`。避免此问题的唯一方法就是变相实现命名空间：为所有名称都加上适当前缀。所选前缀可以是与公司、应用程序或二者皆有关联之名。另外，`Apple`宣称其保留使用所有“两字母前缀”，所以我们一般还是三个字母开头比较好。

### 第 14 条：提供“全能初始化方法”

**能够为对象提供必要信息以便其完成工作的初始化方法叫做“全能初始化方法”。**

如果创建类实例的方式不止一种，那么这个类就会有多个初始化方法。这当然很好，不过仍然要在其中选定一个作为全能初始化方法，令其它初始化方法都来调用它。这样的话，当底层数据存储机制改变时，只需修改此方法的代码就好，无须改动其他初始化方法。

`NSDate`就是一个例子，其初始化方法如下：
```
- (instancetype)init NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTimeIntervalSinceReferenceDate:(NSTimeInterval)ti NS_DESIGNATED_INITIALIZER;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTimeIntervalSinceNow:(NSTimeInterval)secs;
- (instancetype)initWithTimeIntervalSince1970:(NSTimeInterval)secs;
- (instancetype)initWithTimeInterval:(NSTimeInterval)secsToBeAdded sinceDate:(NSDate *)date;
```
在上面几个初始化方法中，`initWithTimeIntervalSinceReferenceDate:`是全能初始化方法，也就是说，其余的初始化方法都要调用它。只有在全能初始化方法中，才会存储内部数据。

### 第 15 条：实现 description 方法
调试程序时，经常需要打印并查看对象信息。一种办法是编写代码把对象的全部属性都输出到日志中。不过最常用的做法还是像下面这样：

```
NSLog(@"object = %@", object);
```

这样直接打印我们的对象它会输出 `<Object:0x*****>`，这并不是我们想要的，所以当我们重写`description `的时候才可能满足我们调试的需求。

```
- (NSString *)description {
    return [NSString stringWithFormat:@"%@ : %@, %@", [self class], self, @"你需要的属性"];
}
```
若想在调试时打印出更详尽的对象描述信息，则应实现 `- (NSString *)debugDescription;`方法，再与`po`命令一起使用配合调试。

### 第 16 条：尽量使用不可变对象

这条主要讲的是，尽量使用不可变的对象。简单来说，我们设计出来一个类，其实是不希望别人更改其属性的，因此应该尽量把对外公布出来的属性设置为只读，而且只在确有必要时才将属性对外公布。

### 第 17 条：使用清晰而协调的命名方式
类、方法及变量的命名是 `OC` 编程的重要环节。这点就不用多说了，具体可以看下官方的命名规则：[Coding Guidelines for Cocoa](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html#//apple_ref/doc/uid/10000146-SW1)

### 第 18 条：为私有方法名加前缀

这样有助于调试，也很容易让我们将公共方法和私有方法区分开，另外便于修改方法名。但是要注意不要但用一个下划线做私有方法的前缀，因为这种做法是预留给苹果公司的。

### 第 19 条：理解 Objective-C 错误模型

当前很多种编程语言都有异常处理机制，`OC`也不例外。在`OC`中异常只用于处理`严重错误(fatal error)`，出现`“不那么严重的错误(nonfatal error)”`时，OC语言的处理方式是：令方法返回 `nil/0`，或者使用`NSError`，以表明有错误发生。

### 第 20 条：理解 NSCopying 协议
在  `OC` 中，拷贝对象通过`copy`方法完成，如果想令自己的类支持拷贝操作，那就要实现` NSCopying `协议，该协议只有一个方法：
```
- (id)copyWithZone:(NSZone*)zone
```

当然如果要求返回对象是可变的类型就要用到`NSMutableCopying`协议，相应方法：

```
- (id)mutableCopyWithZone:(NSZone *)zone
```
- 在拷贝对象时，需要注意拷贝执行的是深拷贝还是浅拷贝。
  - 深拷贝的意思就是：在拷贝对象自身时，会将对象的底层数据也进行复制。
  - 浅拷贝是只拷贝容器对象本身，而不拷贝其中数据。`Foundation` 框架中所有的`collection` 类在默认情况下都执行浅拷贝。

- 要点：
  - 若想令自己所写的对象具有拷贝功能，则需实现`NSCopying`协议。
  - 如果自定义的对象分为可变版本与不可变版本，那么就要同时实现`NSCopying`与`NSMutableCopying`协议。
  - 复制对象时需决定采用浅拷贝还是深拷贝，一般情况下应该尽量执行浅拷贝。
  - 如果你所写的对象需要深拷贝，那么可考虑新增一个专门执行深拷贝的方法。

## 第 4 章：协议与分类

### 第 21 条：通过委托与数据源协议进行对象间通信

简单来说，这条讲的是使用`delegate`，`protocal`进行数据传递。此模式可将数据与业务逻辑解耦。需要注意的是：修饰`delegate`的属性一定要定义成`weak`，而非`strong`，因为两者之间必须为“非拥有关系”。

### 第 22 条：将类的实现代码分散到便于管理的数个分类之中
这条讲的是使用分类。类中经常容易填满各种方法，而这些方法的代码则全部堆在一个巨大的实现文件里。有时这么做是合理的，因为即便通过重构把这个类打散，效果也不会更好。在此情况下，可以通过`OC`的“分类”机制，将类代码按逻辑划入几个分区中，这对开发与调试都有好处。

### 第 23 条：总是为第三方类的分类名称加前缀
这条也很容易理解，就是给分类名加前缀，比如`NSString`分类，

```
@interface NSString (HTTP)

@end

// 加前缀
@interface NSString (ABC_HTTP)

@end
```

给分类加上前缀，易于区分，并且能避免不必要的错误。

### 第 24 条：勿在分类中声明属性
除了 `Class-continuation`分类，其他分类无法向类中新增实例变量，这样就无法把实现属性所需要的实例变量合成出来，即无法生成实例变量，如下图：
![](http://upload-images.jianshu.io/upload_images/2665449-83e1a4ac405c62c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 第 25 条：使用 “Class-continuation分类” 隐藏实现细节

`Class-continuation`分类和普通的分类不同，它必须定义在其所接续的那个类的实现文件里。

```
#import "NNTestClass.h"

@interface NNTestClass ()
// 写你所需要的私有变量或方法
@end

@implementation NNTestClass
// 实现
@end
```
这条和之前的描述也有点类似，总之就是在说尽量不要在公共接口中暴露太多内容，隐藏其实现细节。

### 第 26 条：通过协议提供匿名对象

- 协议可以在某种程度上提供匿名对象，例如`id<someProtocal> object`。`object`对象的类型不限，只要能遵从这个协议即可，在这个协议里面定义了这个对象所应该实现的方法。
- 使用匿名对象来隐藏类型名称（或类名）。
- 如果具体类型不重要，重要的是对象能否处理好一些特定的方法，那么就可以使用这种协议匿名对象来完成。

## 第 5 章：内存管理
### 第 27 条：理解引用计数
`OC` 使用引用计数器来管理内存，每个对象都有个可以递增或递减的计数器。对象创建好之后，其保留计数至少为`1`，若保留计数为正，则对象继续存活，当保留计数降为`0`时，对象被销毁。
在对象的生命周期中，其余对象通过引用来保留或释放此对象。保留或释放操作分别会递增或递减保留计数。

注意`retain`,`release`,`autorelease`,这三个方法操作计数器。
- `retain`递增保留计数。
- `release`递减保留计数。
- `autorelease`待稍后清理“自动释放池”时，再递减保留计数。

### 第 28 条：以 ARC 简化引用计数
`ARC`，即自动管理计数器，理解起来很容易。
`ARC`会自动执行`retain`,`release`,`autorelease`等操作，所以直接在`ARC`中调用这些内存管理方法是非法的。具体来说，`ARC`不能调用以下方法:
- retain
- release
- autorelease
- dealloc

在`ARC`中，变量的内存管理语义可以通过修饰符指明，而原来则需手工执行”保留“及释放操作;(`__strong`,`__weak`,一个保留值，一个不保留值)。
注意的是`ARC`只负责管理`OC`对象的内存。注意：`CoreFoundation`对象不归`ARC`管理，必须适时调用`CFRetain/CFRelease`。

### 第 29 条：在 dealloc 方法中只释放引用并解除监听
不要在`delloc`方法中调用其它方法，尤其是需要异步执行某些任务又要回调的方法，这样是很危险的行为，很可能异步执行完回调的时候该对象已经被销毁了，`crash`了。

在`delloc`方法里，应该做的事情就是释放指向其他对象的引用，并取消原来订阅的键值观测（`KVO`）或`NSNotificationCenter`等通知，不要做其它事情。

```
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

### 第 30 条：编写 “异常安全代码” 时留意内存管理问题

这条也和上面说过的类似，有两点需要注意：
- 捕获异常时，一定要注意将`try`块内所创立的对象清理干净。
```
    @try { // 可能会出现崩溃的代码
    }
    @catch (NSException *exception) { // 捕获到的异常 exception
    }
    @finally { // 结果处理,也可以去掉
    }
```
- 在默认情况下，`ARC` 不生成安全处理异常所需的清理代码。开启编译器标志后，可生成这种代码，不过会导致应用程序变大，而且会降低运行效率。

### 第 31 条：以弱引用避免保留环
这个也很容易理解，保留环就是我们常说的循环引用。就是有时候我们需要将某些引用设为`weak`，避免出现“保留环”。

### 第 32 条：以“自动释放池块”降低内存峰值
**合理运用自动释放池，可降低应用程序的内存峰值。**

- 请看下面几行代码：
```
    NSArray *dataArray = /*****/;
    NSMutableArray *testArray = [NSMutableArray array];
    for(NSDictionary *recordDic in dataArray) {
        TestPerson *person = [[TestPerson alloc] initWithRecord:recordDic];
        [testArray addObject:person];
    }
```
当循环长度无法预估，需要创建很多对象时，上面代码会创造数量无法预估的临时对象，即内存中会出现很多不必要的临时对象，他们本该提早回收的。这时我们可以给这段代码增加一个释放池：

```
    NSArray *dataArray = /*****/;
    NSMutableArray *testArray = [NSMutableArray array];
    for(NSDictionary *recordDic in dataArray) {
        @autoreleasepool {
        TestPerson *person = [[TestPerson alloc] initWithRecord:recordDic];
        [testArray addObject:person];
        }
    }
```
这样临时对象就可以及时回收了，有一点需要注意：**不要把`for`循环放到释放池里面，特别是循环长度特别长的时候。**

### 第 33 条：用“僵尸对象”调试内存管理问题
![僵尸对象](http://upload-images.jianshu.io/upload_images/2665449-0d8e02375074cf0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图，启动这项调试功能之后，运行期系统在回收对象时，会把它的`isa`指针指向特殊的僵尸类，而不是真正的回收他们。系统会修改对象的`isa`指针，令其指向特殊的僵尸类，从而使该对象变为僵尸对象。僵尸类能够响应所有的选择子，响应方式为：打印一条包含消息内容以及其接收者的消息，然后终止应用程序。


### 第 34 条：不要使用 retainCount

`retainCount`，对象的保留计数，`ARC`环境下已经正式废弃，在`MRC`下还可以正常使用。


<br><br>

---
下一篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(3)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(3).md)
