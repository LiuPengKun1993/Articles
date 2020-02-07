# iOS开发技巧 - UIViewController 基类设计

> 在项目开发中，所有控制器里面大概都有共同的属性，比如背景色、导航栏、tarBar 的设置等等，这时我们一般都会设计出来一个 **`UIViewController`** 的基类，通常叫做 **`baseViewController`**或**`rootViewController`**，在这个类里面设置所有控制器的共同的属性，然后项目中所有的控制器再继承自这个类。

#####一般来说这种基类控制器里面需要做的操作有以下几个：

- 为 APP 设置统一的背景色
- 设置是否允许控制器自动调整高度（一般是 NO）
- 自定义导航栏返回按钮
- 重新布局视图大小，及时更新视图的 frame

当然还有其它的一些，毕竟这和项目的具体需求息息相关，因此还要因项目而异。我这里主要就列出基类控制器中基本的常用的一些需求，和大家分享下，希望能帮到需要的人，少走弯路；另外文章如果有不足之处，也希望各位同行能多多的交流指点。

<br></br>
####UIViewController 基类控制器设计步骤

######首先创建一个 `UIViewController` 类，命名为 `NNBaseViewController`

######接着在 `NNBaseViewController.m`中做一些操作

1.设置应用的统一背景色

```
    // 设置应用的背景色
    self.view.backgroundColor = [UIColor lightGrayColor];
```

2.将 automaticallyAdjustsScrollViewInsets 设置为NO，不然视图会下移64像素

```
// 不允许 viewController 自动调整，我们自己布局；如果设置为YES，视图会自动下移 64 像素
    self.automaticallyAdjustsScrollViewInsets = NO;
```

3.有时候系统的返回按钮不符合应用的风格，所以就需要重写导航栏上的返回按钮

```
#pragma mark - 自定义返回按钮
- (void)setupLeftBarButton {
    // 自定义 leftBarButtonItem ，UIImageRenderingModeAlwaysOriginal 防止图片被渲染
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]
                                             initWithImage:[[UIImage imageNamed:@"Back-蓝"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]
                                             style:UIBarButtonItemStylePlain
                                             target:self
                                             action:@selector(leftBarButtonClick)];
    // 防止返回手势失效
    self.navigationController.interactivePopGestureRecognizer.delegate = (id<UIGestureRecognizerDelegate>)self;
}

#pragma mark - 返回按钮的点击事件
- (void)leftBarButtonClick {
    [self.navigationController popViewControllerAnimated:YES];
}

```

当然这里需要判断一下，不然首层控制器的导航栏上也会被设置上返回按钮 `leftBarButtonItem`，我是在 `- (void)viewDidLoad` 方法中判断的。

```
    // 判断是否有上级页面，有的话再调用
    if ([self.navigationController.viewControllers indexOfObject:self] > 0) {
        [self setupLeftBarButton];
    }
```

4.接下来我们一起设置下 `View` 中视图的布局，根据视图中控件的最大的 Y 值和最大的 X 值调整 `View` 的 `frame` 。这个主要用在视图中控件比较多的时候，也需要具体分析，比如有时候应用中全是 `UITableView`，那么就不需要设置了，因为 `UITableView` 可以根据 `cell` 自动调整 `frame`。不过也有用到的时候，比如应用中需要计算高度的类有很多，那么就可以用这个统一设置，主要还是看项目需求。另外注意这个方法要放在 `- (void)viewWillAppear:(BOOL)animated`中。

```
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    
    CGFloat baseViewHeight = 0;
    CGFloat baseViewWidth = 0;
    NSArray *subViews = self.baseView.subviews;
    
    // 遍历视图中的所有控件，求出最大的Y值和最大的X值
    for (UIView *view in subViews) {
        if (CGRectGetMaxY(view.frame) > baseViewHeight) {
            baseViewHeight = CGRectGetMaxY(view.frame);
        }
        if (CGRectGetMaxX(view.frame) > baseViewWidth) {
            baseViewWidth = CGRectGetMaxX(view.frame);
        }
    }
    
    // 三目运算方法求出最大的宽是否大于屏幕宽，以及最大的高是否大于屏幕高
    CGFloat NNHeight = baseViewHeight > NNBaseViewSizeHeight ? baseViewHeight:NNBaseViewSizeHeight;
    CGFloat NNWidth = baseViewWidth > NNBaseViewSizeWidth ? baseViewWidth:NNBaseViewSizeWidth;

    self.baseView.contentSize = CGSizeMake(NNWidth, NNHeight);
}
```

上面代码块中用到了一个属性 `baseView` ， `baseView` 属于 `UIScrollView` 类，相当于一个子视图容器，所有继承自 `NNBaseViewController` 的控制器都应该把子视图添加到  `baseView`上，这样才能更新它的 `frame`。

```
#import <UIKit/UIKit.h>

@interface NNBaseViewController : UIViewController

/** 子视图容器 */
@property (nonatomic, strong) UIScrollView *baseView;

@end
```

另外别忘了在 `- (void)viewDidLoad` 这个方法中创建`baseView`并设置它的属性。

```
    self.baseView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 64, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height - 64)];
    // 是否反弹
    self.baseView.bounces = NO;
    // 是否显示滚动指示器
    self.baseView.showsVerticalScrollIndicator = NO;
    self.baseView.showsHorizontalScrollIndicator = NO;
    
    [self.view addSubview:self.baseView];
    self.baseView.contentSize = NNContentSize;
    
```
上面便是一个基本的 `UIViewController` 的基类，具体还是要根据项目的需要来设计。

---
<br></br>
####接下来是完整的代码：
######NNBaseViewController.h

```
#import <UIKit/UIKit.h>

@interface NNBaseViewController : UIViewController

/** 子视图容器 */
@property (nonatomic, strong) UIScrollView *baseView;

@end
```

######NNBaseViewController.m

```
#import "NNBaseViewController.h"

#define NNBaseViewSize self.baseView.bounds.size
#define NNBaseViewSizeHeight self.baseView.bounds.size.height
#define NNBaseViewSizeWidth self.baseView.bounds.size.width

@interface NNBaseViewController ()

@end

@implementation NNBaseViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupViews];
    // 判断是否有上级页面，有的话再调用
    if ([self.navigationController.viewControllers indexOfObject:self] > 0) {
        [self setupLeftBarButton];
    }
}

- (void)setupViews {
    // 设置应用的背景色
    self.view.backgroundColor = [UIColor lightGrayColor];
    // 不允许 viewController 自动调整，我们自己布局；如果设置为YES，视图会自动下移 64 像素
    self.automaticallyAdjustsScrollViewInsets = NO;
    self.baseView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 64, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height - 64)];
    // 是否反弹
    self.baseView.bounces = NO;
    // 是否显示滚动指示器
    self.baseView.showsVerticalScrollIndicator = NO;
    self.baseView.showsHorizontalScrollIndicator = NO;
    
    [self.view addSubview:self.baseView];
    self.baseView.contentSize = NNBaseViewSize;
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    
    CGFloat baseViewHeight = 0;
    CGFloat baseViewWidth = 0;
    NSArray *subViews = self.baseView.subviews;
    
    // 遍历视图中的所有控件，求出最大的Y值和最大的X值
    for (UIView *view in subViews) {
        if (CGRectGetMaxY(view.frame) > baseViewHeight) {
            baseViewHeight = CGRectGetMaxY(view.frame);
        }
        if (CGRectGetMaxX(view.frame) > baseViewWidth) {
            baseViewWidth = CGRectGetMaxX(view.frame);
        }
    }
    
    // 三目运算方法求出最大的宽和最大的高
    CGFloat NNHeight = baseViewHeight > NNBaseViewSizeHeight ? baseViewHeight:NNBaseViewSizeHeight;
    CGFloat NNWidth = baseViewWidth > NNBaseViewSizeWidth ? baseViewWidth:NNBaseViewSizeWidth;

    self.baseView.contentSize = CGSizeMake(NNWidth, NNHeight);
}

#pragma mark - 自定义返回按钮
- (void)setupLeftBarButton {
    // 自定义 leftBarButtonItem ，UIImageRenderingModeAlwaysOriginal 防止图片被渲染
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]
                                             initWithImage:[[UIImage imageNamed:@"Back-蓝"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]
                                             style:UIBarButtonItemStylePlain
                                             target:self
                                             action:@selector(leftBarButtonClick)];
    // 防止返回手势失效
    self.navigationController.interactivePopGestureRecognizer.delegate = (id<UIGestureRecognizerDelegate>)self;
}

#pragma mark - 返回按钮的点击事件
- (void)leftBarButtonClick {
    [self.navigationController popViewControllerAnimated:YES];
}
```