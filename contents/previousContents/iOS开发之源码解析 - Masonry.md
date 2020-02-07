# iOS开发之源码解析 - Masonry

**这是 *GitHub* 上，[Masonry](https://github.com/SnapKit/Masonry) 官方对 *Masonry* 的介绍：**

> [Masonry](https://github.com/SnapKit/Masonry)  is a light-weight layout framework which wraps AutoLayout with a nicer syntax. Masonry has its own layout [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) which provides a chainable way of describing your NSLayoutConstraints which results in layout code that is more concise and readable. Masonry supports iOS and Mac OS X.

译文如下：
> [Masonry](https://github.com/SnapKit/Masonry) 是一个轻量级的布局框架，它通过一种友好的语法封装了自动布局。Masonry 通过链式语法 [DSL(Domain-specific language)](https://en.wikipedia.org/wiki/Domain-specific_language) 来封装 NSLayoutConstraints，使布局代码更加地简洁易读。Masonry 支持 iOS 和 Mac OS X。

在分析 Masonry 源码之前，有必要先说一下 Masonry 的基本使用，而在说 Masonry 的基本使用之前，我们还是先来看看 storyboard 以及 xib 中是如何进行 AutoLayout 的，我截了两张图：

![](http://upload-images.jianshu.io/upload_images/2665449-a305784e0ac30193.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2665449-a97902c3ba89e176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图我们可以看出，在 storyboard 和 xib 中，我们可以在可视化界面对控件进行 AutoLayout，操作较为简单方便。Masonry 的实现原理与这很相似，接下来我们就一起看看如何使用 Masonry 进行自动布局。

先看一下 Masonry 支持哪些属性：

```
/**
 *	以下属性返回一个新的 MASViewConstraint
 */
@property (nonatomic, strong, readonly) MASConstraint *left;      // 左侧
@property (nonatomic, strong, readonly) MASConstraint *top;       // 上侧
@property (nonatomic, strong, readonly) MASConstraint *right;     // 右侧
@property (nonatomic, strong, readonly) MASConstraint *bottom;    // 下侧
@property (nonatomic, strong, readonly) MASConstraint *leading;   // 首部
@property (nonatomic, strong, readonly) MASConstraint *trailing;  // 尾部
@property (nonatomic, strong, readonly) MASConstraint *width;     // 宽
@property (nonatomic, strong, readonly) MASConstraint *height;    // 高
@property (nonatomic, strong, readonly) MASConstraint *centerX;   // 横向居中
@property (nonatomic, strong, readonly) MASConstraint *centerY;   // 纵向居中
@property (nonatomic, strong, readonly) MASConstraint *baseline;  // 文本基线

/**
 *  返回一个 block 对象，block 的接收参数是 MASAttribute 类型,返回一个 MASCompositeConstraint 对象
 */
@property (nonatomic, strong, readonly) MASConstraint *(^attributes)(MASAttribute attrs);

/**
 *	返回一个 MASConstraint 对象，包含上下左右的布局信息
 */
@property (nonatomic, strong, readonly) MASConstraint *edges;

/**
 *	返回一个 MASConstraint 对象，包含宽高的布局信息
 */
@property (nonatomic, strong, readonly) MASConstraint *size;

/**
 *	返回一个 MASConstraint 对象，包含 centerX 和 centerY 信息
 */
@property (nonatomic, strong, readonly) MASConstraint *center;
```

有了属性之后，怎样添加约束呢？**使用 Masonry 添加约束的函数有三个**，这三个方法我们在文件 `View+MASAdditions` 中可以查看到：

```
/// 新增约束
- (NSArray *)mas_makeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;

/// 更新约束
- (NSArray *)mas_updateConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;

/// 清除旧约束，只保留新约束
- (NSArray *)mas_remakeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
```

上面这三个方法中最常用的是 `- (NSArray *)mas_makeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block`。即第一个，接下来我们就尝试一下。

举个栗子：

导入 Masonry 框架之后，添加以下代码：

```
- (void)layoutViews
{
    UIImageView *imageView = [[UIImageView alloc] init];
    [imageView setImage:[UIImage imageNamed:@"123"]];
    [self.view addSubview:imageView];
    
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view).offset(100);
        make.right.equalTo(self.view).offset(-100);
        make.top.equalTo(self.view).offset(250);
        make.bottom.equalTo(self.view).offset(-250);
    }];
}
```

上面这几句代码便可以对 UIImageView 控件进行约束，这里还有两点值得一提。第一点：**下面这三种方式会实现相同的效果**

```
    /// 第一种
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view).offset(100);
        make.right.equalTo(self.view).offset(-100);
        make.top.equalTo(self.view).offset(250);
        make.bottom.equalTo(self.view).offset(-250);
    }];
    
    /// 第二种
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.bottom.left.right.equalTo(self.view).insets(UIEdgeInsetsMake(250, 100, 250, 100));
    }];
    
    /// 第三种
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view).insets(UIEdgeInsetsMake(250, 100, 250, 100));
    }];
```
三种方式得出的效果图一样：


![](http://upload-images.jianshu.io/upload_images/2665449-ecf24222fa369f6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二点：**必须先把控件添加到视图上，才能对控件进行布局，否则程序会崩**。即先 `addSubview:`，再 `mas_makeConstraints:` 。
<br></br>

--- 
**以上是对 Masonry 使用的简单介绍。接下来我们开始分析它的源码**。




通过对 Masonry  使用方法的了解，我们可以看出 Masonry 的使用过程还是很简洁的

```
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view).offset(100);
        make.right.equalTo(self.view).offset(-100);
        make.top.equalTo(self.view).offset(250);
        make.bottom.equalTo(self.view).offset(-250);
    }];
```

那我们就从 `mas_makeConstraints: ` 这个方法开始探寻 Masonry 的源码。上文说到，Masonry 中设置约束最常用的方法是

```
/// 新增约束
- (NSArray *)mas_makeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;

```

同时，Masonry 还提供两个类方法用于更新和重建约束

```
/// 更新约束
- (NSArray *)mas_updateConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;

/// 清除旧约束，只保留新约束
- (NSArray *)mas_remakeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block;
```

这里我们就以  `mas_makeConstraints:`  为切入点开始分析 Masonry 这个框架。`mas_makeConstraints:`  这个方法位于分类`View+MASAdditions`中，方法的实现如下：

```
/// 新增约束
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
    /// 我们是手动添加约束，因此将自动转换关闭
    self.translatesAutoresizingMaskIntoConstraints = NO;
    /// 创建 MASConstraintMaker 对象
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    /// 通过 block 进行值的回调
    block(constraintMaker);
    /// 调用 install 方法
    return [constraintMaker install];
}
```
该方法先去掉 AutoResizing 的自动转换（如果这个属性没有被正确设置，那么视图的约束不会被成功添加），接着初始化一个 MASConstraintMaker 对象，传递到 block 中，执行 block，最后调用 install 方法。

第一次点击进来看到这个方法之后，我有几处疑问。
- **第一，MASConstraintMaker 类内部做了什么操作？**
- **第二，回调 `block(constraintMaker) `有什么用？**
- **第三，调用 `[constraintMaker install]` 方法实现了什么？**

<br></br>

我们先来分析一下 MASConstraintMaker 这个类。MASConstraintMaker 是 Masonry 框架整个 DSL 过程的控制中心，它控制着整个添加过程，上文我们总结 Masonry 支持哪些属性时，总结的那些属性就来自 MASConstraintMaker 类。我们知道 Masonry 是基于 AutoLayout 进行的封装，所以接着我们一起来看下 MASConstraintMaker 是如何发挥作用的。下面是 MASConstraintMaker 的初始化

```
///  这里的 MAS_VIEW 是一个宏，#define MAS_VIEW UIView
- (id)initWithView:(MAS_VIEW *)view {
    self = [super init];
    if (!self) return nil;
    
    self.view = view;
    self.constraints = NSMutableArray.new;
    
    return self;
}
```
从上边代码中我们可以清晰的看出，**Masonry 在初始化 MASConstraintMaker 时，将当前的 view 赋给 MASConstraintMaker 类，并初始化一个  constraints 的空可变数组，作为约束数组**。

除此之外 MASConstraintMaker 还做了什么呢？

先来到 MASConstraintMaker 的头文件 `MASConstraintMaker.h` 中，下面是一些比较常规的属性

```
/**
 *    以下属性返回一个新的 MASViewConstraint
 */
@property (nonatomic, strong, readonly) MASConstraint *left;      // 左侧
@property (nonatomic, strong, readonly) MASConstraint *top;       // 上侧
@property (nonatomic, strong, readonly) MASConstraint *right;     // 右侧
@property (nonatomic, strong, readonly) MASConstraint *bottom;    // 下侧
@property (nonatomic, strong, readonly) MASConstraint *leading;   // 首部
@property (nonatomic, strong, readonly) MASConstraint *trailing;  // 尾部
@property (nonatomic, strong, readonly) MASConstraint *width;     // 宽
@property (nonatomic, strong, readonly) MASConstraint *height;    // 高
@property (nonatomic, strong, readonly) MASConstraint *centerX;   // 横向居中
@property (nonatomic, strong, readonly) MASConstraint *centerY;   // 纵向居中
@property (nonatomic, strong, readonly) MASConstraint *baseline;  // 文本基线
```

另外可以在 MASConstraintMaker 的 `MASConstraintMaker.m` 中看到另一个属性 `@property (nonatomic, strong) NSMutableArray *constraints` ，用来存储 MASConstraint 

```
@interface MASConstraintMaker () <MASConstraintDelegate>

@property (nonatomic, weak) MAS_VIEW *view;
/// 存储 MASConstraint
@property (nonatomic, strong) NSMutableArray *constraints;

@end
```

> 接下来我们通过下面这个例子来分析上面这些属性：

```
 [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.bottom.left.right.equalTo(self.view).insets(UIEdgeInsetsMake(250, 100, 250, 100));
    }];
```

- block 中首先会执行 `make.top` ，会先调用一个增加约束的通用方法`- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute`  ，接着会调用 MASConstraintMaker 中 MASConstraintDelegate 的 `- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute` 方法。具体代码如下：

```
/// 重写 getter 方法
- (MASConstraint *)top {
    return [self addConstraintWithLayoutAttribute:NSLayoutAttributeTop];
}

/// 增加约束的通用方法
- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    return [self constraint:nil addConstraintWithLayoutAttribute:layoutAttribute];
}

/// 通过 NSLayoutAttribute 添加约束
- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    
    /// 构造 MASViewAttribute
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
    
    /// 通过 MASViewAttribute 构造第一个 MASViewConstraint
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
    
    /// 如果存在 contraint，则把 constraint 和 newConstraint 组合成 MASCompositeConstraint
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;
        
        /// 替换原来的 constraint 成新的 MASCompositeConstraint
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }
    
    /// 不存在则设置 constraint 到 self.constraints
    if (!constraint) {
        /// 设置delegate
        newConstraint.delegate = self;
        /// 将约束添加到self.constraints
        [self.constraints addObject:newConstraint];
    }
    /// 返回刚刚创建的 MASViewConstraint 对象
    return newConstraint;
}
```

值得一提的是，当调用 `make.top` 的时候会创建一个只有 firstViewAttribute 的 MASViewConstraint 对象，并且**进入不存在 constraint 的代码部分**，详情见代码块中的注释。

```
 /// 不存在则设置 constraint 到 self.constraints
    if (!constraint) {
        /// 设置delegate
        newConstraint.delegate = self;
        /// 将约束添加到self.constraints
        [self.constraints addObject:newConstraint];
    }
    /// 返回刚刚创建的 MASViewConstraint 对象
    return newConstraint;
```

**在这个方法的实现过程中，`make.top` 的返回值是 MASViewConstraint 对象**。

- 当执行到 `make.top.bottom` 的时候，其实**是对 MASViewConstraint 对象 .bottom 的调用**，会走到 MASViewConstraint 中重写的 `- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute `方法，然后最终还是会调用这个代理方法 `- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute`

```
- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    NSAssert(!self.hasLayoutRelation, @"Attributes should be chained before defining the constraint relation");

    return [self.delegate constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
}

/// 通过 NSLayoutAttribute 添加约束
- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {

    /// 构造 MASViewAttribute
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];

    /// 通过 MASViewAttribute 构造第一个 MASViewConstraint
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];

    /// 如果存在 contraint，则把 constraint 和 newConstraint 组合成 MASCompositeConstraint
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;

        /// 替换原来的 constraint 成新的 MASCompositeConstraint
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }

    /// 不存在则设置 constraint 到 self.constraints
    if (!constraint) {
        /// 设置delegate
        newConstraint.delegate = self;
        /// 将约束添加到self.constraints
        [self.constraints addObject:newConstraint];
    }
    /// 返回刚刚创建的 MASViewConstraint 对象
    return newConstraint;
}
```

和前面的执行过程不同，因为 MASViewConstraint 的 delegate 对象是刚才设置过的 MASConstraintMaker 对象，并且因为 constraint 不是 nil，所以会进入

```
/// 如果存在 contraint，则把 constraint 和 newConstraint 组合成 MASCompositeConstraint
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;

        /// 替换原来的 constraint 成新的 MASCompositeConstraint
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }
```

因此**调用 make.top.bottom 返回的是一个 MASCompositeConstraint 对象**。

- 当程序执行到 `make.top.bottom.left` 时，就**是对 MASCompositeConstraint 中 .left 的调用**，会走 MASCompositeConstraint 中的重写方法 `- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute`，接着会调用 `- (MASConstraint *)constraint:(MASConstraint __unused *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute`方法，最终还是会调用 `- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute`。代码如下：

```
- (MASConstraint *)addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    [self constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
    return self;
}

- (MASConstraint *)constraint:(MASConstraint __unused *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    id<MASConstraintDelegate> strongDelegate = self.delegate;
    MASConstraint *newConstraint = [strongDelegate constraint:self addConstraintWithLayoutAttribute:layoutAttribute];
    newConstraint.delegate = self;
    [self.childConstraints addObject:newConstraint];
    return newConstraint;
}

/// 通过 NSLayoutAttribute 添加约束
- (MASConstraint *)constraint:(MASConstraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute {
    
    /// 构造 MASViewAttribute
    MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute];
    
    /// 通过 MASViewAttribute 构造第一个 MASViewConstraint
    MASViewConstraint *newConstraint = [[MASViewConstraint alloc] initWithFirstViewAttribute:viewAttribute];
    
    /// 如果存在 contraint，则把 constraint 和 newConstraint 组合成 MASCompositeConstraint
    if ([constraint isKindOfClass:MASViewConstraint.class]) {
        //replace with composite constraint
        NSArray *children = @[constraint, newConstraint];
        MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
        compositeConstraint.delegate = self;
        
        /// 替换原来的 constraint 成新的 MASCompositeConstraint
        [self constraint:constraint shouldBeReplacedWithConstraint:compositeConstraint];
        return compositeConstraint;
    }
    
    /// 不存在则设置 constraint 到 self.constraints
    if (!constraint) {
        newConstraint.delegate = self;
        [self.constraints addObject:newConstraint];
    }
    return newConstraint;
}
```

**但这次 `if ([constraint isKindOfClass:MASViewConstraint.class])` 与 `if (!constraint)` 都不会进入，只直接返回 MASViewConstraint 对象，然后回到 `- (MASConstraint *)constraint:(MASConstraint __unused *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute)layoutAttribute` 方法中设置它的 delegate，并且将对象存入 MASCompositeConstraint 的 childConstraints 中。**

- 之后再有更多的链式 MASConstraint 的组合（比如执行到 `make.top.bottom.left.right`），也只是 MASCompositeConstraint 的调用，直接加入 childConstraints 中即可。

- 至于 `equalTo(self.view)` 的调用过程，这里有必要说明一下，`equalTo(self.view)` 在文件 MASConstraint 中执行 `- (MASConstraint * (^)(id))equalTo` 方法，接着调用 MASViewConstraint 中的 `- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation` 方法。代码如下：

```

/// MASConstraint 是一个抽象类，其中有很多的方法都必须在子类中覆写。Masonry 中有两个 MASConstraint 的子类，分别是 MASViewConstraint 和 MASCompositeConstraint

/// MASConstraint.m
- (MASConstraint * (^)(id))equalTo {
    return ^id(id attribute) {
        return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
    };
}

/// MASViewConstraint.m
- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
    return ^id(id attribute, NSLayoutRelation relation) {
        if ([attribute isKindOfClass:NSArray.class]) {
            NSAssert(!self.hasLayoutRelation, @"Redefinition of constraint relation");
            NSMutableArray *children = NSMutableArray.new;
            
            for (id attr in attribute) {
                MASViewConstraint *viewConstraint = [self copy];
                viewConstraint.layoutRelation = relation;
                viewConstraint.secondViewAttribute = attr;
                [children addObject:viewConstraint];
            }
            MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
            compositeConstraint.delegate = self.delegate;
            [self.delegate constraint:self shouldBeReplacedWithConstraint:compositeConstraint];
            return compositeConstraint;
        } else {
            NSAssert(!self.hasLayoutRelation || self.layoutRelation == relation && [attribute isKindOfClass:NSValue.class], @"Redefinition of constraint relation");
            self.layoutRelation = relation;
            self.secondViewAttribute = attribute;
            return self;
        }
    };
}

```

`- (MASConstraint * (^)(id))equalTo` 方法提供了参数 attribute 和布局关系 NSLayoutRelationEqual，这两个参数会传递到 `- (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation` 中，设置 constraint 的布局关系和 secondViewAttribute 属性，为 `[constraintMaker install]` 做准备。
<br></br>

到这里基本上就把上文的那个例子说完了，通过例子让我们对 MASConstraintMaker 中的一些常规属性有了一定的了解，同时也明白了 `block(constraintMaker)` 这个方法的作用——**在调用  `block(constraintMaker)` 时，对 constraintMaker 进行配置**。 MASConstraintMaker 中还有一些别的属性，我们再一起来看看吧

```
/**
 *  返回一个 block 对象，block 的接收参数是 MASAttribute 类型，返回一个 MASCompositeConstraint 对象
 */
@property (nonatomic, strong, readonly) MASConstraint *(^attributes)(MASAttribute attrs);

/**
 *    返回一个 MASConstraint 对象，包含上下左右的布局
 */
@property (nonatomic, strong, readonly) MASConstraint *edges;

/**
 *    返回一个 MASConstraint 对象，包含宽高的布局
 */
@property (nonatomic, strong, readonly) MASConstraint *size;

/**
 *    返回一个 MASConstraint 对象，包含 centerX 和 centerY
 */
@property (nonatomic, strong, readonly) MASConstraint *center;
```

这里只简单的介绍一下上面的几个属性
> 1. `attributes`：返回一个 block 对象，block 的接收参数是 MASAttribute 类型，返回 MASCompositeConstraint 对象
2. `edges`：返回一个 MASConstraint 对象，同时包含了上下左右的布局
3. `size`：返回一个 MASConstraint 对象，同时包含了宽高的布局
4. `center`：返回一个 MASConstraint 对象，同时包含了 centerX 和centerY


<br></br>

**接下来我们来分析 ` [constraintMaker install]` 方法**。

我们在`- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block ` 方法的最后会调用 ` [constraintMaker install]`  方法来添加所有存储在 `self.constraints` 数组中的所有约束。

```
- (NSArray *)install {
    /// 是否需要删除原来的约束
    if (self.removeExisting) {
        /// 获得所有约束
        NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView:self.view];
        /// 删除所有约束
        for (MASConstraint *constraint in installedConstraints) {
            [constraint uninstall];
        }
    }
    NSArray *constraints = self.constraints.copy;
    for (MASConstraint *constraint in constraints) {
        constraint.updateExisting = self.updateExisting;
        [constraint install];
    }
    /// 去除所有缓存的约束结构体
    [self.constraints removeAllObjects];
    return constraints;
}

```

这个方法会先判断当前视图的约束是否需要删除，如果我们之前调用过 `- (NSArray *)mas_remakeConstraints:(void(NS_NOESCAPE ^)(MASConstraintMaker *make))block` 这个方法（它会把 removeExisting 的 BOOL 值设为 YES），那么视图中的原有约束就会被全被删除。接着往下走，程序会遍历 constraints 数组，发送 install 消息。

```
/// MASViewConstraint.m
- (void)install {
    /// 已经有约束
    if (self.hasBeenInstalled) {
        return;
    }
    
    /// 支持 active 且已经有约束
    if ([self supportsActiveProperty] && self.layoutConstraint) {
        /// 激活约束
        self.layoutConstraint.active = YES;
        /// 添加约束缓存
        [self.firstViewAttribute.view.mas_installedConstraints addObject:self];
        return;
    }
    
    /// 获得 firstLayoutItem, firstLayoutAttribute, secondLayoutItem, secondLayoutAttribute
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;

    // alignment attributes must have a secondViewAttribute
    // therefore we assume that is refering to superview
    // eg make.left.equalTo(@10)
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
    
    /// NSLayoutConstraint 的创建，生成约束，MASLayoutConstraint 其实就是 NSLayoutConstraint 的别名
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    
    /// 设置 priority 和 mas_key
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
    
    /// 如果 secondViewAttribute 有 view 对象
    if (self.secondViewAttribute.view) {
        /// 取得两个 view 的最小公共 view
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        
        /// 设置约束 view 为此 view
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }

    /// 已经存在的约束
    MASLayoutConstraint *existingConstraint = nil;
    if (self.updateExisting) { // 需要更新
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    }
    if (existingConstraint) { // 如果存在则替换约束
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        /// 其它情况则直接添加约束
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
}
```

上面这个方法是为当前视图添加约束的最后的方法，首先这个方法会先获取即将用于初始化 NSLayoutConstraint 的子类的几个属性

```
    /// MASViewConstraint.m

    /// 获得 firstLayoutItem, firstLayoutAttribute, secondLayoutItem, secondLayoutAttribute
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;
```

然后判断当前即将添加的约束，如果不是 size 类型并且没有提供 self.secondViewAttribute，会自动将约束添加到 superview 上

```
    /// MASViewConstraint.m
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
```

接着创建 MASLayoutConstraint 对象

```
/// MASViewConstraint.m
/// NSLayoutConstraint 的创建，生成约束，MASLayoutConstraint 其实就是 NSLayoutConstraint 的别名
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];

    /// 设置 priority 和 mas_key
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
```

创建完约束对象后，我们要寻找该约束添加到那个 View 上。下方的代码段就是获取接收该约束对象的视图。如果是两个视图相对约束，就获取两种的公共父视图。如果添加的是 Width 或者 Height，那么就添加到当前视图上。如果既没有指定相对视图，也不是 Size 类型的约束，那么就将该约束对象添加到当前视图的父视图上。代码实现如下：

```
/// MASViewConstraint.m
/// 如果 secondViewAttribute 有 view 对象
    if (self.secondViewAttribute.view) {
        /// 取得两个 view 的最小公共 view
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);

        /// 设置约束 view 为此 view
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }
```

加约束时我们要判断是否需要对约束进行更新，如果需要，就替换约束，如果不需要就直接添加约束即可。添加成功后我们将通过 mas_installedConstraints 属性记录一下本次添加的约束。

```
   /// 已经存在的约束
    MASLayoutConstraint *existingConstraint = nil;
    if (self.updateExisting) { // 需要更新
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    }
    if (existingConstraint) { // 如果存在则替换约束
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        /// 其它情况则直接添加约束
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
```


> 到此为止，对 [Masonry](https://github.com/SnapKit/Masonry)  源码的简单介绍已接近尾声了。。。[Masonry](https://github.com/SnapKit/Masonry)  的代码流程简单来讲就是提供给我们一个 MASConstraintMaker，然后我们根据 Masonry 提供的语法，添加约束。最后 Masonry 解析约束，将真正的约束关系添加到相应的视图上。
