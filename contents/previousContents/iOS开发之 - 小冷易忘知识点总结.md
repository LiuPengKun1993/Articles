
# iOS开发之 - 小冷易忘知识点总结

> 看网上有人整理 iOS 开发中常用的易忘知识点，[iOS 开发小冷易忘知识点总结](http://www.jianshu.com/p/b3ea66bffc68)，觉得不错，于是自己也想着整理一些易忘的知识点。会持续更新......

- iOS 控制器的基类，搭建框架之初一般都会设计一个 ViewController 基类，这里设置的基类是 NNBaseViewController，继承自 UIViewController。

```
#import "NNBaseViewController.h"

@interface NNBaseViewController ()

@end

@implementation NNBaseViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupViews];
    // 判断是否有上级页面，有的话再设置返回按钮
    if ([self.navigationController.viewControllers indexOfObject:self] != 0) {
        [self setupLeftBarButton];
    }
}

- (void)setupViews {
    // 设置应用的背景色
    self.view.backgroundColor = [UIColor lightGrayColor];
    // 不允许 viewController 自动调整，我们自己布局；如果设置为YES，视图会自动下移 64 像素
    self.automaticallyAdjustsScrollViewInsets = NO;
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

- (void)leftBarButtonClick {
    [self.navigationController popViewControllerAnimated:YES];
}

@end
```


- 隐藏 navigationControll 左侧的返回按钮

因为项目需要，有时需要隐藏 navigationControll 左侧或右侧的按钮。

```
	// 这两句貌似不太管用
    self.navigationItem.leftBarButtonItems = nil;
    self.navigationItem.rightBarButtonItem = nil;
    // 可以试试这两句代码
    self.navigationItem.hidesBackButton = YES;
    self.navigationItem.rightBarButtonItem.customView.hidden = YES;
```
- 改变 button 内部的图片以及文字的偏移量

有时候需要在 button 的右部添加指示箭头，但 button 默认的是图片在左，文字在右，因此想要实现项目需求，我们需要改变 button 内部的偏移量。在 storyboard 或 xib 中也可以改变 button 内部的图片以及文字的偏移量，但貌似没有做适配的地方，那么在不同机型上的显示就会有问题（如果哪位道友发现可以在 storyboard 或 xib 中可以做适配，还请告知），下面是改变 button 内部图片以及文字偏移量的代码。

```
	// 已经做了适配，各位可以根据需要做屏幕适配
    self.changeSpecialistBtn.titleEdgeInsets = UIEdgeInsetsMake(0, -lengthFit(16), 0, 0);
    self.changeSpecialistBtn.imageEdgeInsets = UIEdgeInsetsMake(0, lengthFit(76), 0, 0);
```

- tableView 滚动时自动收起键盘

项目中某个 tableView 页面有文字搜索功能，输入文本完毕之后滚动 tableView，要求键盘自动收起。


```
    self.tableView.keyboardDismissMode = UIScrollViewKeyboardDismissModeOnDrag;
```

- 判断上级页面是哪个页面

```
	// 我这里的项目需求是，如果上一个页面是 KXOrderCenterViewController，就隐藏稍后支付按钮，否则不隐藏
	NSArray *array = self.navigationController.viewControllers;
    if ([array[0] isKindOfClass:[KXOrderCenterViewController class]]) {
        self.payLaterBtn.hidden = YES;
    } else {
        self.payLaterBtn.hidden = NO;
    }
      
```
- 从任意界面返回 NavigationController 中的任意一个

这个方法还是挺实用的，从某个界面跳转到任意一个界面。


```
	// 得到当前的 NavigationController
	[[APPDELEGATE getCurrentViewController].navigationController popToRootViewControllerAnimated:NO];
	
    // 跳转到哪个 NavigationController，这里下标为1，表示第二个
    [APPDELEGATE.tabBarController setSelectedIndex:1];
    
    // 退出当前界面并执行 block 中的内容
    [self.navigationController dismissViewControllerAnimated:YES completion:^{
            
        UINavigationController *navi = APPDELEGATE.tabBarController.viewControllers[1];
        
        // 保险起见，再判断下是否是将要跳转到的页面
        if ([navi.viewControllers[0] isKindOfClass:[KXOrderCenterViewController class]]) {
        
        	// 我这里需要刷新
            KXOrderCenterViewController *orderCenter = navi.viewControllers[0];
            [orderCenter beginRefreshing];
            }
    }];
        
```

- 设置导航栏右部点击按钮，并添加点击事件

```
// 添加文字按钮
	UIBarButtonItem *rightBarButton = [[UIBarButtonItem alloc] initWithTitle:@"编辑" style:UIBarButtonItemStylePlain target:self action:@selector(editor)];
        
	self.navigationItem.rightBarButtonItem = rightBarButton;
        
// 添加图片按钮
    UIBarButtonItem *rightBarBtn = [[UIBarButtonItem alloc] initWithImage:[[UIImage imageNamed:@"图片名"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] style:UIBarButtonItemStylePlain target:self action:@selector(rightBarButtonEvent)];
    
    [self.navigationItem setRightBarButtonItem:rightBarBtn];
```

- 给 view 添加手势，并加入点击事件

```
	UITapGestureRecognizer *operationTap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(gotoCapture:)];

	[self.operationView addGestureRecognizer:operationTap];
    
```

- textview 输入文本，服务器那边要求返回文本个数不超过32

服务器那边有限制，要求返回文本个数不超过32个。。。于是这里做了判断，如果超过32个就不再允许输入，允许删除。


```
- (void)textViewDidChange:(UITextView *)textView {
    
    [self updateViewHeight];
    // 该判断用于联想输入
    if (textView == self.secondDiagnosisTextView) {
        if (textView.text.length > 32) {
        	// 截取字符串
            textView.text = [textView.text substringToIndex:32];
            // HUD 提示
            [CommunityPublicClass showHUDwithLabelText:@"最多输入32个字" andDetailsLabelText:nil];
        }
    }
}

// 代理方法规定只能输入32个字
- (BOOL)textView:(UITextView *)textView shouldChangeTextInRange:(NSRange)range replacementText:(NSString *)text {

	// 输入32字之后，允许删除
    if ([text isEqualToString:@""]) {
        return YES;
    }
    
    if (textView == self.secondDiagnosisTextView) {
        if (textView.text.length == 0)
            return YES;
        NSInteger existedLength = textView.text.length;
        NSInteger selectedLength = range.length;
        NSInteger replaceLength = text.length;
        if (existedLength - selectedLength + replaceLength > 32) {
        	// HUD 提示
            [CommunityPublicClass showHUDwithLabelText:@"最多输入32个字" andDetailsLabelText:nil];
            return NO;
        }
    }
    return YES;
}
```

- textfield 中输入手机号，只允许输入数字

```
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string{
    
    if (textField == self.secondAgeTextField) {
        
        if ([self isPureInt:string] || [string isEqualToString:@""]) {
            return YES;
        }else{
            return NO;
        }
    }
    return YES;
}

// 判断输入的是否是纯数字
- (BOOL)isPureInt:(NSString*)string{
    NSScanner *scan = [NSScanner scannerWithString:string];
    int val;
    return [scan scanInt:&val] && [scan isAtEnd];
}
```

- 给 button 设置边框、圆角、边框设置颜色

```
 	[self.completeBtn.layer setBorderWidth:1.0];
    self.completeBtn.layer.cornerRadius = 1;
    [self.completeBtn.layer setBorderColor:KXColorTextBlack.CGColor];
    
```
- 设置图片拉伸

有时 UI 设计师给出的图片并不合适，比如我们这的 UI 设计师一边忙 web 端，一边忙我们移动端，有时候都不忍心让他重新作图，于是只好自己想办法

```
  // 设置图片拉伸
  UIImage *bgImage = [UIImage imageNamed:@""];
  _bgImageView.image = [bgImage stretchableImageWithLeftCapWidth:15 topCapHeight:15];
  
```

- 点击单元格时，默认的点击效果比较丑，大多时候不显示点击效果

```
  - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
  {
      [tableView deselectRowAtIndexPath:indexPath animated:YES] ;
  }
```

- 改变导航栏标题的颜色

```
	UIColor *color = [UIColor cyanColor];
    NSDictionary *dictionary = [NSDictionary dictionaryWithObject: color forKey:NSForegroundColorAttributeName];
    self.navigationController.navigationBar.titleTextAttributes = dictionary;
    
```
 
 - 延迟调用
 
```
  // 延迟两秒钟调用 beginAction 方法
 [self performSelector:@selector(beginAction) withObject:nil afterDelay:0.2] ;
```

 - 项目中用到了 UISwitch 控件，需要更改它的大小

```
// 改变 UISwitch  的大小，CGAffineTransformMakeScale(CGFloat x, CGFloat y) 对 view 的长和宽进行缩放，不改变 view 的中心点
self.orderSwitch.transform = CGAffineTransformMakeScale(0.7, 0.7); 

// 改变 UISwitch 开启时的颜色
self.orderSwitch.onTintColor = KXColorBlue;
```
- 在做明暗文切换(密码输入框)的时候遇见一个坑——就是切换 secureTextEntry 的时候，输入框的光标会偏移，下面是解决办法及明暗文切换的方法：

```
- (IBAction)btnClick:(UIButton *)sender {
    
    sender.selected = !sender.selected;
    NSString *passwordString = self.passwordText.text;
    // 这句代码可以防止切换的时候光标偏移
    self.passwordText.text = @"";

    if (sender.selected) { // 明文
        self.passwordText.secureTextEntry = NO;
    } else { // 暗文
        self.passwordText.secureTextEntry = YES;
    }
    
    self.passwordText.text = passwordString;
}
```

- 判断屏幕横屏/竖屏(在ViewController里面)

```
// 屏幕发生翻转的时候会调用
- (void)viewWillLayoutSubviews {
    [self shouldRotateToOrientation:(UIDeviceOrientation)[UIApplication sharedApplication].statusBarOrientation];
}

- (void)shouldRotateToOrientation:(UIDeviceOrientation)orientation {
    if (orientation == UIDeviceOrientationPortrait ||orientation == UIDeviceOrientationPortraitUpsideDown) {
        NSLog(@"这是竖屏");
    } else {
        NSLog(@"这是横屏");
    }
}
```
- 加载本地的 HTML 文件，比如文件名字是`serviceDescription.html`

```
NSString *resourcePath = [[NSBundle mainBundle] resourcePath];
    NSString *filePath = [resourcePath stringByAppendingPathComponent:@"serviceDescription.html"];
    NSString *htmlString = [[NSString alloc] initWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:nil];
    [self.webView loadHTMLString: htmlString baseURL: [NSURL fileURLWithPath:[[NSBundle mainBundle] bundlePath]]];
```

- iOS 项目开发中用的一部分宏定义

```
/** self的弱引用 */
#define NNWeakSelf __weak typeof(self) weakSelf = self;
/** self的强引用 */
#define NNStrongSelf __strong __typeof(weakSelf)strongSelf = weakSelf;

#define NNColorBlue [UIColor colorWithRed:0.0/255.0f green:135.0/255.0f blue:209.0/255.0f alpha:1]
#define NNFontBoldText [UIFont boldSystemFontOfSize:fontFit(16)]
#define NNFontText [UIFont systemFontOfSize:fontFit(16)]

#define SCREENWIDTH [UIScreen mainScreen].bounds.size.width
#define SCREENHEIGHT [UIScreen mainScreen].bounds.size.height

#define iPhone5 (([[UIScreen mainScreen] bounds].size.height*[[UIScreen mainScreen] bounds].size.width <= 320*568)?YES:NO)

#define iPhone6 (([[UIScreen mainScreen] bounds].size.height*[[UIScreen mainScreen] bounds].size.width <= 375*667)?YES:NO)

#define iPhone6p (([[UIScreen mainScreen] bounds].size.height*[[UIScreen mainScreen] bounds].size.width <= 414*736)?YES:NO)

#define iOS7 ([[[UIDevice currentDevice] systemVersion] floatValue] >= 7.0 ? YES : NO) // 是否IOS7

static inline float lengthFit(float iphone6PlusLength)
{
    if (iPhone5) {
        return iphone6PlusLength *320.0f/414.0f;
    }
    if (iPhone6) {
        return iphone6PlusLength *375.0f/414.0f;
    }
    return iphone6PlusLength;
}

static inline float fontFit(float iphone6PlusFont)
{
    if (iPhone5) {
        return iphone6PlusFont - 2;
    }
    if (iPhone6) {
        return iphone6PlusFont - 1;
    }
    return iphone6PlusFont;
}

#define PlaceholderAvatarImage [UIImage imageNamed:@"默认头像"]
```

- 一句代码隐藏 UITableView 中空的 cell

```
self.tableView.tableFooterView = [[UIView alloc] init];
```

- 自定义 leftBarButtonItem 返回按钮时，防止返回手势失效的办法

```
- (void)setupLeftBarButton {
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]
                                             initWithImage:[[UIImage imageNamed:@"Back-蓝"] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]
                                                style:UIBarButtonItemStylePlain
                                                target:self
                                                action:@selector(leftBarButtonClick)];
    self.navigationController.interactivePopGestureRecognizer.delegate = (id<UIGestureRecognizerDelegate>)self;
}

- (void)leftBarButtonClick {
    [self.navigationController popViewControllerAnimated:YES];
}
```

- 不允许 viewController 自动调整，我们自己布局；如果设置为YES，视图会自动下移 64 像素

```
    self.automaticallyAdjustsScrollViewInsets = NO;
```

- 去空格，开发中有些地方不需要空格，这里记录一下

```
        // 去掉两端的空格
        [trimString stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
        
        // 去掉所有空格
        [trimString stringByReplacingOccurrencesOfString:@" " withString:@""];
```

- 判断字符串中是否包含非法字符

```
if ([self.searchText.text rangeOfString:@"'"].location != NSNotFound) {
        // 用封装好的提示框提示
        [CommunityPublicClass showHUDwithLabelText:@"您输入的字符不合法" andDetailsLabelText:nil];
        return;
    }
```

- iOS 导航栏的正确隐藏方法

第一种方法

```
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];

    [self.navigationController setNavigationBarHidden:YES animated:YES];
}

- (void)viewWillDisappear:(BOOL)animated {
    [super viewWillDisappear:animated];

    [self.navigationController setNavigationBarHidden:NO animated:YES];
}
```

第二种方法

```
@interface NNViewController () <UINavigationControllerDelegate>

@end

@implementation NNViewController 

- (void)viewDidLoad {
    [super viewDidLoad];

    // 设置导航控制器的代理为self
    self.navigationController.delegate = self;
}

#pragma mark - UINavigationControllerDelegate
- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated {

    [self.navigationController setNavigationBarHidden: [viewController isKindOfClass:[self class]] animated:YES];
}
```

- 判断字符串是否为空

```
// 判断字符串是否为空
+ (BOOL)isBlankString:(NSString *)string {
    if (string == nil || string == NULL) {
        return YES;
    }
    if ([string isKindOfClass:[NSNull class]]) {
        return YES;
    }
    if ([[string stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet]] length]==0) {
        return YES;
    }
    return NO;
}
```


- 把一个 view 的所有subview清空

```
[view.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];

```