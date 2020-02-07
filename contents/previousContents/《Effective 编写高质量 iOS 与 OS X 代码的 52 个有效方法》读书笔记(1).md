# iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(1)

> **《Effective Objective-C 2.0: 编写高质量 iOS 与 OS X 代码的 52 个有效方法》是一本非常经典的 OC 书籍。这本书从语法、接口与 API 设计、内存管理、框架等 7 大方面总结和探讨了 OC 编程中的 52 个特性与陷阱，很值得读。**

##### 文章共分为三篇：

第一篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(1)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(1).md)

第二篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(2).md)

第三篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(3)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(3).md)



##### 文章目录：

> 第 1 章：熟悉 OC
第 2 章：对象、消息、运行期
第 3 章：接口与 API 设计
第 4 章：协议与分类
第 5 章：内存管理
第 6 章：块与大中枢派发（block 与 GCD）
第 7 章：系统框架

## 第 1 章：熟悉 OC
#### 第 1 条 - 在类的头文件中尽量少引入其他头文件
OC 中应尽量避免在头文件中引用其他类，即比如我们有两个类，`NNListViewController` 以及 `NNHomeViewController`，应避免在 `NNListViewController.h` 中引用 `NNHomeViewController`，如果非要在 `NNListViewController.h` 中引入 `NNHomeViewController`，如下面代码：

```
#import <UIKit/UIKit.h>

@interface NNListViewController : UIViewController
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *age;
@property (nonatomic, strong) NNHomeViewController *homeVC;
@end
```

这时程序会报错，因为 `NNHomeViewController` 类并不可见，这时常见的做法是在 `NNListViewController.h` 中加入下面这行代码

```
#import "NNHomeViewController.h"
```

这种方法可行，但是不够优雅。在编译 `NNListViewController` 类时，不需要知道 `NNHomeViewController` 类的全部细节，只需要知道有一个类名叫 `NNHomeViewController` 就好，所以我们应该这样写：

```
@class NNHomeViewController;
```

这叫做**“向前声明”该类**。现在 `NNListViewController.h` 变成了这样：
```
#import <UIKit/UIKit.h>

@class NNHomeViewController;

@interface NNListViewController : UIViewController
@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSString *age;
@property (nonatomic, strong) NNHomeViewController *homeVC;
@end
```
`NNListViewController` 的实现文件需要引入 `NNHomeViewController.h`，因为若要使用后者，就必须知道其所有接口细节。于是 `NNListViewController.m` 就是：

```
#import "NNListViewController.h"
#import "NNHomeViewController.h"

@implementation NNListViewController

@end
```

- 利用向前声明这种写法，有两个好处：
  1. 减少编译时间。在头文件中使用向前声明，不会引入 `NNHomeViewController` 中的所有内容，只在确有需要的时候才引入，这样会缩减编译时间。
  2. 解决了两个类互相引用的问题。假设 `NNHomeViewController` 类与 `NNListViewController` 类互相引用，使用 `#import` 而非 `#include` 指令虽然不会造成死循环，但会使两个类里有一个无法被正确编译。这个时候我们使用向前声明，就可以解决这个问题。

#### 第 2 条 - 多用字面量语法，少用与之等价的方法
编写 OC 代码总会用到几个类，`NSString`、`NSArray`、`NSNumber`、`NSDictionary`，从类名上即可看出各自所表达的数据结构。`NSString` 对象有一种简单的创建方式，叫做“字符串字面量”，其语法如下：

```
NSString string = @"Liu Zhong Ning";
```
如果不用这种语法，就要以常见的` alloc` 及 `init` 方法来分配并初始化 `NSString` 对象了。同样也能用字面量语法声明 `NSArray`、`NSNumber`、`NSDictionary` 类的实例。**使用字面量语法可以缩减代码长度，使其更为易读。**

- 非字面量语法：
```
    // NSNumber
    NSNumber *intNumber = [NSNumber numberWithInt:1];
    NSNumber *floatNumber = [NSNumber numberWithFloat:6.6f];
    NSNumber *charNumber = [NSNumber numberWithChar:'a'];

    // NSArray
    NSArray *animals = [NSArray arrayWithObjects:@"cat", @"dog", nil];
    // 取下标
    NSString *dog = [animals objectAtIndex:1];

    // NSDictionary
    NSDictionary *personDic = [NSDictionary dictionaryWithObjectsAndKeys:@"Liu Zhong Ning", @"name", 25, @"age", nil];
    // 访问字典
    NSString *name = [personDic objectForKey:@"name"];
```

- 字面量语法：
```
    // NSNumber
    NSNumber *intNumber = @1;
    NSNumber *floatNumber = @6.6f;
    NSNumber *charNumber = @'a';

    // NSArray
    NSArray *animals = @[@"cat", @"dog"];
    // 取下标
    NSString *dog = animals[1];

    // NSDictionary
    NSDictionary *personDic = @{@"name" : @"Liu Zhong Ning",
                                @"age" : @25};
    // 访问字典
    NSString *name = personDic[@"name"];
```

- 注意点：用字面量语法创建数组或字典时，若值中有 `nil`，则会抛出异常。因此，务必确保值里不包含 `nil`。

#### 第 3 条 - 多用类型常量，少用 #define 预处理指令
- 定义常量时，有时会用这种方法：

```
#define ANIMATION_DURATION 0.3
```
用预处理指令也可以达到我们想要的效果，但这样定义出来的常量 **没有类型信息**；
另外，预处理过程会把碰到的所有 `ANIMATION_DURATION` 换成 0.3，这样的话，假设此指令声明在某头文件，那么 **所有引入了这个头文件的代码，其` ANIMATION_DURATION` 都会被替换**。有个办法比用预处理指令来定义常量更好：


```
static const NSTimeInterval kAnimationDuration = 0.3;
```

可以看出，上面方式定义的常量包含类型信息，可知该常量类型为 `NSTimeInterval`，这种方式能令阅读代码的人更易理解其意图。另外还应注意常量名称：若常量只用在实现文件，则在前面加字母 `k`；若常量在类之外可见，则通常以类名为前缀。

**变量一定要同时用 `static` 与 `const` 来声明。如果试图修改由 `const` 修饰符所声明的变量，那么编译器就会报错。而 `static` 修饰符意味着该变量仅在定义此变量的实现文件中可见。**如果不加 `static`，则编译器会为它创建一个“外部符号”，此时若另一个类中也声明了同名变量，那么编译器就会抛出一条错误消息。

- 要点：
  - **不要用预处理指令定义常量。**这样定义出来的常量不含类型信息，编译器只是会在编译前执行查找与替换操作。即使有人重新定义了常量值，编译器也不会产生警告信息，这将导致应用程序中的常量值不一致。
  - **在实现文件中使用 `static const` 定义“只在编译单元内可见的常量”。**由于此类常量不再全局符号表中，所以无须为其名称加前缀。

#### 第 4 条 - 用枚举表示状态、选项、状态码
- **枚举是一种常量命名方式。当一个对象如果有多种状态时，就可以定义为一个枚举集。**

比如` UITableView` 中的 `UITableViewStyle` 就是一个枚举：

```
typedef NS_ENUM(NSInteger, UITableViewStyle) {
    UITableViewStylePlain,          // regular table view
    UITableViewStyleGrouped         // preferences style table view
};
```

**由于每种状态都用一个便于理解的值来表示，所以这样写出来的代码更易读懂。编译器会为枚举分配一个独有的编号，从 `0` 开始，每个枚举递增 `1`。**

- 在 switch 中使用枚举：

```
    // 定义
    typedef enum : NSUInteger {
        NNAccountStateInitial = 0,        // 初始
        NNAccountStateWaitReviewed,       // 待审核
        NNAccountStatePassed,             // 已通过
        NNAccountStateNoPassed,           // 未通过
        NNAccountStateDeactivated,        // 停用
        NNAccountStateFreeze,             // 冻结
    } NNAccountState;

    // 使用
    switch (self.accountState) {
        case NNAccountStateInitial:
            
            break;
        case NNAccountStateWaitReviewed:
            
            break;
        case NNAccountStatePassed:
            
            break;
        case NNAccountStateNoPassed:
            
            break;
        case NNAccountStateDeactivated:
            
            break;
        case NNAccountStateFreeze:
            
            break;
    }
```
- 要点：
  - 应该用枚举表示状态，给这些值起个易懂的名字。
  - 在处理枚举类型的 `switch` 语句中，不要实现 `default` 分支，这样，加入新枚举之后，编译器就会提示开发者：`switch` 语句并未处理所有的枚举。

## 第 2 章：对象、消息、运行期
#### 第 5 条 - 在对象内部尽量直接访问实例变量

请看下边这个类。

.h 文件
```
#import <UIKit/UIKit.h>

@interface NNListViewController : UIViewController

@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, copy) NSString *firstName;

- (NSString *)fullName;
- (void)setFullName:(NSString *)fullName;

@end
```

.m 文件
```
@implementation NNListViewController

- (void)viewDidLoad {
    [super viewDidLoad];
}

- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@", self.firstName, self.lastName];
}

- (void)setFullName:(NSString *)fullName {
    NSArray *compoents = [fullName componentsSeparatedByString:@" "];
    self.firstName = [compoents objectAtIndex:0];
    self.firstName = [compoents objectAtIndex:1];
}

@end
```

在 fullName 的获取方法与设置方法中，我们使用点语法，通过存取方法来访问相关的实例变量。现在假设重写这两个方法，不经由存取方法，而是直接访问实例变量：

```
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@", _firstName, _lastName];
}

- (void)setFullName:(NSString *)fullName {
    NSArray *compoents = [fullName componentsSeparatedByString:@" "];
    _firstName = [compoents objectAtIndex:0];
    _firstName = [compoents objectAtIndex:1];
}
```
- 这两种写法有几个区别：
  - 由于不经过 `OC` 方法派发步骤，所以直接访问实例变量的速度当然比较快，在这种情况下，编译器所生成的代码会直接访问保存对象实例变量的那块内存。
  - 直接访问实例变量不会调用其“设置方法”，这就绕过了为相关属性所定义的“内存管理语义”。比方说，如果在 `ARC` 下直接访问一个声明为 `copy` 的属性，那么并不会拷贝该属性，只会保留新值并释放旧值。
  - 如果直接访问实例变量，那么不会触发“键值观测”通知，这样做是否会产生问题，还取决于具体的对象行为。
  - 通过属性来访问有助于排查与之相关的错误，因为可以给“获取方法”或“设置方法”中新增“断点”，监控该属性的调用者及其访问时机。

#### 第 6 条 - 理解“对象等同性”这一概念
根据“等同性”来比较对象是一个非常有用的功能，请看下面这段代码，并思考会输出什么：
```
    NSString *foo = @"Badger 123";
    NSString *bar = [NSString stringWithFormat:@"Badger %d", 123];
    BOOL equalA = (foo == bar);
    BOOL equalB = [foo isEqual:bar];
    BOOL equalC = [foo isEqualToString:bar];
    NSLog(@"equalA = %d, equalB = %d, equalC = %d", equalA, equalB, equalC);
```

这里是输出结果：`equalA = 0, equalB = 1, equalC = 1`

大家可以看到 == 与等同性判断方法之间的差别(== 操作符只是比较了两个指针，而不是指针所指的对象)，NSString 实现了一个自己独有的判断方法，isEqualToString。调用该方法比调用 isEqual 方法快，后者还要执行额外的步骤，因此它不知道受测对象的类型。


#### 第 7 条 - 以“类族模式”隐藏实现细节
“类族”是一种很有用的模式，可以隐藏“抽象基类”背后的实现细节。`OC` 的系统框架中普遍使用此模式。比如创建 `UIButton` 时需要调用下面这个方法：
```
+ (instancetype)buttonWithType:(UIButtonType)buttonType;
```
该方法所返回的对象，其类型取决于传入的按钮类型，然而，不管返回什么类型的对象，它们都继承自同一个基类：`UIButton`。这么做的意义在于：`UIButton` 类的使用者无须关心创建出来的按钮具体属于哪个子类，也不用考虑按钮的绘制方式等实现细节。

- 举例创建类族

.h 文件
```
#import <UIKit/UIKit.h>

typedef enum : NSUInteger {
    NNAccountStateInitial = 0,        // 初始
    NNAccountStateWaitReviewed,       // 待审核
    NNAccountStatePassed,             // 已通过
    NNAccountStateNoPassed,           // 未通过
    NNAccountStateDeactivated,        // 停用
    NNAccountStateFreeze,             // 冻结
} NNAccountState;

@interface NNHomeViewController : UIViewController

+ (NNHomeViewController *)accountWithState:(NNAccountState)state;

@end
```

.m 文件
```
+ (NNHomeViewController *)accountWithState:(NNAccountState)state {
    switch (state) {
        case NNAccountStateInitial:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
        case NNAccountStateWaitReviewed:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
        case NNAccountStatePassed:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
        case NNAccountStateNoPassed:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
        case NNAccountStateDeactivated:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
        case NNAccountStateFreeze:
            // 做操作
            return [[NNHomeViewController alloc] init];
            break;
    }
}
```

- Cocoa 里的类族
系统框架中有许多类族。大部分` collection` 类都是类族，例如 `NSArray` 与其可变版本 `NSMutableArray`。这样看来，实际上有两个抽象基类，一个用于不可变数组，一个用于可变数组。尽管具备公共接口的类有两个，但仍然可以合起来算作一个类族。不可变的类定义了对所有数组都通用的方法，而可变的类则定义了那些只适用于可变数组的方法。两个类共属于一类族，这意味着二者在实现各自类型的数组时可以共用实现代码，此外，还能够把可变数组复制为不可变数组，反之亦然。

- 请看下面这句代码:
```
    id maybeAnArray;
    // 可以对 maybeAnArray 赋任何值
    if ([maybeAnArray class] == [NSArray class]) { // 判断永远不会为真
        // 不会执行
    }
```
解析代码：`NSArray` 是个类族，`[maybeAnArray class]` 所返回的绝不可能是 `NSArray` 本身，因为由 `NSArray` 的初始化方法所返回的那个实例其类型是隐藏在类族公共接口后面的某个内部类型。
不过仍然有办法可以判断出某个实例所属的类是否位于类族之中。若想判断某对象是否位于类族中，不要直接检测两个类对象是否相同，而应该采用下列代码：
```
    id maybeAnArray;
    // 对 maybeAnArray 赋任何值
    if ([maybeAnArray isKindOfClass:[NSArray class]]) {
        // 会执行
    }
```


#### 第 8 条：在既有类中使用关联对象存放自定义数据

在 OC 中可以通过 Category 给一个现有的类添加属性，但却不能添加实例变量，这似乎成为了 OC 的一个短板。值得庆幸的是，`OC` 中有一项强大的特性可以解决此问题，这就是关联对象`（Associated Object）`。
- 看下这几个方法：
```
    // 需要导入头文件
    #import <objc/runtime.h>

    // 根据给定的键和策略为某对象设置关联对象值
    OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
    // 根据给定的键从某对象获取相应的关联对象值
    OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key);
    // 移除指定对象的全部关联对象
    OBJC_EXPORT void objc_removeAssociatedObjects(id object);
```

- 解析代码：`id object`给谁设置关联对象；`const void *key`关联对象唯一的 key ，获取时会用到的主键；`id value`关联对象；`objc_AssociationPolicy`关联策略，关联策略以下几种：

```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```
关联对象有很多细节很多坑需要注意，想了解更多关联对象的童鞋请看这篇文章：[Objective-C Associated Objects 的实现原理。](http://blog.csdn.net/shaobo8910/article/details/47748727)

#### 第 9 条：理解 objc_msgSend 的作用

在 OC 中，如果向某个对象传递消息，那就会在运行时使用动态绑定`（dynamic binding）`机制来决定需要调用的方法。但是到了底层具体实现，却是普通的C语言函数实现的。这个实现的函数就是`objc_msgSend`,该函数定义如下：

```
void objc_msgSend(id self, SEL cmd, ...) 
```

第一个参数代表`接收者`，第二个参数代表`选择子`（`SEL是选择子的类型，选择子即方法名字`），后续参数就是消息中的那些`参数`，其顺序不变。

- 举个例子:

```
id returnValue = [someObject messageName:parameter]; 
```

代码解析：`objc_msgSend` 函数会依据接收者与选择子的类型来调用适当的方法。函数首先会在接收者所属的类中搜寻其方法列表，如果能找到这个跟选择子名称相同的方法，就跳转至其实现代码。若是当前类没找到，那就沿着继承体系向上查找，等找到合适方法之后再跳转 ，如果最终还是找不到，那就进入消息转发的流程去进行处理了。


### 第 10 条：理解消息转发机制
**当对象接收到无法解读的消息后，就会启动“消息转发”`（message forwarding）`机制**，我们可以通过代码在消息转发的过程中告诉对象应该如何处理未知的消息，系统默认是抛出异常，控制台给出提示代码如下：

```
unrecognized selector sent to instance 0x7f8c8a70a380
```

**消息转发机制所讲的就是在抛出异常之前也就是消息转发过程中经过的一些步骤。**

##### 消息转发机制分为两个阶段：
- 第一阶段先征询接收者，所属的类，看其是否能动态添加方法，以处理当前这个“未知的选择子”`（unknown selector）`，这叫做“动态方法解析”`（dynamic method resolution）`。
- 第二阶段涉及“完整的消息转发机制”`（full forwarding mechanism）`。如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增方法的手段来响应包含该选择子的消息了。此时，运行期系统会请求接收者以其他手段来处理与消息相关的方法调用。这又细分为两小步。
  - 首先，请接收者看看有没有其他对象能处理这条消息。
  - 若有，则运行期系统会把消息转给那个对象，于是消息转发过程结束，一切如常。若没有“备援的接收者”`（replacement receiver）`，则启动完整的消息转发机制，运行期系统会把与消息有关的全部细节都封装到 `NSInvocation` 对象中，再给接收者最后一次机会，令其设法解决当前还未处理的这条消息。

- 消息转发全流程：
![消息转发全流程](http://upload-images.jianshu.io/upload_images/2665449-22dab42fdda6386d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 第 11 条：用“方法调配技术”调试“黑盒方法”

这条讲的主要内容就是方法调配`( Method Swizzling)`，通过运行时用另外一种方法实现来替换掉原有的方法实现，往往被应用在向原有实现中添加新功能，打印信息等。

- 如何互换两个方法实现？具体主要用下面两个方法

```
class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name)
method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2)
```

首先我们写一个分类，比如为`NSString`写一个“分类”`（category）`：

```
#import "NSString+extension.h"
#import <objc/runtime.h>

@implementation NSString (extension)

+ (void)load {
    Method oldMethod = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method newMethod = class_getInstanceMethod([NSString class], @selector(new_lowercaseString));
    method_exchangeImplementations(oldMethod, newMethod);
}

- (NSString *)new_lowercaseString {
    NSString *lowerString = [self new_myLowercaseString];
    NSLog(@"%@ - %@", self, lowerString);
    return lowerString;
}

@end
```
解析代码：`+ (void)load`中会调换`lowercaseString`方法与`new_lowercaseString`方法；在方法`- (NSString *)new_lowercaseString`中，` [self new_myLowercaseString]`，我们调用`new_myLowercaseString`方法，此时不会引起死循环，因为这个方法是与`lowercaseString`方法调换的。

此时我们在`NSString`实例中调用`lowercaseString`方法：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    NSString *string = @"Talk is cheap, Show me the code.";
    NSString *lowercaseString = [string lowercaseString];
    NSLog(@"%@", lowercaseString);
}
```

打印效果图如下：
![打印效果图](http://upload-images.jianshu.io/upload_images/2665449-fc0ce6698f16a1a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过此方案，开发者可以为那些“完全不知道其具体实现的”`（completely opaque，“完全不透明的”）`黑盒方法增加日志记录功能，这非常有助于程序调试。然而，此做法只在调试程序时有用。很少有人在调试程序之外的场合用上述“方法调配技术”来永久改动某个类的功能。不能仅仅因为`OC`语言里有这个特性就一定要用它。若是滥用，反而会令代码变得不易读懂且难于维护。

### 第 12 条：理解“类对象”的用意

**类是一个对象，是` Class` 类型的对象，简称“类对象”。**

```
typedef struct objc_class *Class; 
```

- 类型查询方法：

可以用类型信息查询方法来检视类继承体系。`“isMemberOfClass:”`能够判断出对象是否为某个特定类的实例，而`“isKindOfClass:”`则能够判断出对象是否为某类或其派生类的实例。例如：

```
NSMutableDictionary *dict = [NSMutableDictionary new];  
[dict isMemberOfClass:[NSDictionary class]]; ///< NO 
[dict isMemberOfClass:[NSMutableDictionary class]]; ///< YES 
[dict isKindOfClass:[NSDictionary class]]; ///< YES 
[dict isKindOfClass:[NSArray class]]; ///< NO 
```

比较类对象是否等同的办法来判断，使用`==操作符`，而不要使用比较`OC`对象时常用的`“isEqual:”`方法。原因在于，类对象是`“单例”（singleton）`，在应用程序范围内，每个类的`Class`仅有一个实例。


- 要点：
  - 每个实例都有一个指向`Class` 对象的指针，用以表明其类型，而这些 `Class`对象则构成了类的继承体系。
  - 如果对象类型无法在编译期确定，那么就应该使用类型信息查询方法来探知。
  - 尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能。

<br><br>

---

下一篇：[iOS 开发 -《Effective Objective-C 2.0:编写高质量 iOS 与 OS X 代码的 52 个有效方法》读书笔记(2)](https://github.com/liuzhongning/Articles/blob/master/contents/previousContents/《Effective%20编写高质量%20iOS%20与%20OS%20X%20代码的%2052%20个有效方法》读书笔记(2).md)
