# iOS开发之 - 广告植入

> 应用中植入广告是iOS开发者获得收益的方式之一，其实有很多 app 的绝大部分盈利都来自于广告收入......下面就开始介绍两种主流的植入广告的方法：iAd 方式和 Admob方式。

## 一、iAd 方式

-   iAd 方式简介
iAd 移动广告业务是乔布斯在2010年推出来的，怀念～～～
![iAd](http://upload-images.jianshu.io/upload_images/2665449-d2c8ede84e6f61e5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
刚开始的时候，iAd 的确有很大的发展，但是随着时间的推移，更是在老乔辞世😢之后，iAd 的使用人数便越来越少。以至于今年年初，苹果公司宣布将要放弃在 iAd 平台上直接出售广告，而是将其转交给出版商。。。但是作为 iOS 开发者的我们，还是有了解的必要， iAd 广告平台依在。

- 使用步骤如下：

1.1 导入iAd.framework 依赖库
![iAd.framework](http://upload-images.jianshu.io/upload_images/2665449-636c983733641509.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.2 方便起见，我是在 Main.storyboard 中拉入ADBannerView
![ADBannerView](http://upload-images.jianshu.io/upload_images/2665449-57e14a8fb97ab7c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.3 代码实现

```
#import "ViewController.h"
#import <iAd/iAd.h>

@interface ViewController () <ADBannerViewDelegate>
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *bannerViewBottom;
@property (nonatomic,strong)ADBannerView *bannerView;


@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
}

#pragma mark - ADBannerViewDelegate 协议一共有五个方法
// 广告即将被加载
- (void)bannerViewWillLoadAd:(ADBannerView *)banner{
    NSLog(@"bannerViewWillLoadAd");
}

// 广告已被加载
- (void)bannerViewDidLoadAd:(ADBannerView *)banner
{
    NSLog(@"bannerViewDidLoadAd");
    
    self.bannerViewBottom.constant = 0;
    
    [UIView animateWithDuration:0.5 animations:^{
        [self.view layoutIfNeeded];
    }];
}

// 是否出现全屏的广告
- (BOOL)bannerViewActionShouldBegin:(ADBannerView *)banner willLeaveApplication:(BOOL)willLeave {
// return YES: 用户点击 Banner 之后，会出现全屏广告
//    如果返回的是 NO，用户点击 Banner 之后, 不会出现全屏广告
//    另外如果返回 YES， 用户其它的活动都会被暂停
    return YES;
}

// 全屏的广告退出时调用，用户其它的活动都会被唤醒
- (void)bannerViewActionDidFinish:(ADBannerView *)banner {

    NSLog(@"bannerViewActionDidFinish");
}


// 如果 app 上没有广告，就会调用这个方法
- (void)bannerView:(ADBannerView *)banner didFailToReceiveAdWithError:(NSError *)error
{
    NSLog(@"didFailToReceiveAdWithError");
}
@end
```

## 二、Admob方式

- Admob方式简介
Admob 是 Google 推出的，用的人比较多，它提供了目前为止最方便的 iPhone app 广告集成库。

![Admob](http://upload-images.jianshu.io/upload_images/2665449-a4d5bb0d7fb0e5b0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面说一下它的基本使用～～～

2.1 首先需要注册Admob、设置账户信息、创建应用等等，这里不再详细介绍，如果有朋友需要可以借鉴[这篇文章](http://blog.csdn.net/songrotek/article/details/8501557)。

2.2  集成Admob SDK.
2.21 下载Admob SDK：[Admob SDK](http://www.cocoachina.com/bbs/read.php?tid=284847)
2.22 下载完成后，将这两个文件导入项目：

![Admob SDK](http://upload-images.jianshu.io/upload_images/2665449-bb165f16616a1c81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.23 添加依赖库

![添加依赖库](http://upload-images.jianshu.io/upload_images/2665449-717cd15cdd022342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.24 导入头文件
**注意需要导入的事这两个头文件！**

```
#import "GoogleMobileAds/GADInterstitial.h"
#import "GoogleMobileAds/GADBannerView.h"
```

2.3 代码实现
- 在屏幕底部植入广告

```
#import "ViewController.h"

#import "GoogleMobileAds/GADInterstitial.h"
#import "GoogleMobileAds/GADBannerView.h"

@interface ViewController (){
    GADBannerView *bannerView;
    
}
@end

static NSString *MyId = @"********";
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 在屏幕底部创建标准尺寸的视图
    bannerView = [[GADBannerView alloc] initWithFrame:CGRectMake(0.0, self.view.frame.size.height -GAD_SIZE_320x50.height, GAD_SIZE_320x50.width, GAD_SIZE_320x50.height)];
    bannerView.backgroundColor = [UIColor redColor];
    
    // 指定广告的标识符
    bannerView.adUnitID = MyId;
    
    // 转至广告的展示位置之后恢复哪个 UIViewController
    bannerView.rootViewController = self;
    [self.view addSubview:bannerView];
    
    // 请求并加载广告
    [bannerView loadRequest:[GADRequest request]];
}
@end
```

- 也可以在屏幕顶部植入广告

```
#import "ViewController.h"

#import "GoogleMobileAds/GADInterstitial.h"
#import "GoogleMobileAds/GADBannerView.h"

@interface ViewController (){
    GADBannerView *bannerView;
    
}
@end

static NSString *MyId = @"********";
@implementation ViewController

- (void)viewDidLoad {

    [super viewDidLoad];
    // 导航栏位置添加广告
    float y =self.navigationController.navigationBar.frame.size.height + [[UIApplication sharedApplication] statusBarFrame].size.height;
    float x = ([UIScreen mainScreen].bounds.size.width - 320) / 2;
    bannerView=[[GADBannerView alloc]initWithAdSize:kGADAdSizeBanner origin:CGPointMake(x, y)];
    bannerView.adUnitID = MyId;
    bannerView.backgroundColor = [UIColor redColor];
    bannerView.rootViewController = self;
    [self.view addSubview:bannerView];
    GADRequest *request = [GADRequest request];
    request.testDevices = @[MyId,MyId];
    [bannerView loadRequest:request];
}
```

---

当然，你可以在任何你希望的地方植入广告，大家可以试试的😊！以后要是用的多了，我会继续在这里补充，另外如果代码书写有误，还请大家能够指出来！互相交流学习！谢谢！
