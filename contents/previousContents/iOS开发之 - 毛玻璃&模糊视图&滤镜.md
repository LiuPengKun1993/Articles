# iOS开发之 - 毛玻璃&模糊视图&滤镜

> 为了让界面更加美观，有时候我们需要将图片设置成模糊，比如下边这张图片：

![CoreImage/CoreImage初窥](http://upload-images.jianshu.io/upload_images/2665449-6946c7d084d2e1ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这篇文章主要演示了三种模糊效果，如下：
####一、简单的毛玻璃效果：
- 原图
![原图](http://upload-images.jianshu.io/upload_images/2665449-cb2dfce94d622fc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 毛玻璃效果
![毛玻璃效果](http://upload-images.jianshu.io/upload_images/2665449-df954aee7a52a2ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 代码如下：

```
#import "ViewController.h"
#import <CoreImage/CoreImage.h>

@interface ViewController ()
@property (nonatomic, strong) UIImageView *imageView;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.imageView = [[UIImageView alloc]initWithFrame:self.view.frame];
    self.imageView.image = [UIImage imageNamed:@"1.jpg"];
    self.imageView.contentMode = UIViewContentModeScaleAspectFill;
    [self.view addSubview:self.imageView];
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    button.frame = CGRectMake(0, self.view.frame.size.height - 80, self.view.frame.size.width, 40);
    [button setTitle:@"蒙奇·D·路飞" forState:UIControlStateNormal];
    button.backgroundColor = [UIColor brownColor];
    [button addTarget:self action:@selector("点击调用的方法") forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:button];
}

```
button 被点击时调用以下代码：

```
    self.imageView = [[UIImageView alloc] initWithFrame:self.view.frame];
    self.imageView.image = [UIImage imageNamed:@"1.jpg"];
    UIBlurEffect *blur = [UIBlurEffect effectWithStyle:UIBlurEffectStyleLight];
    UIVisualEffectView *visualEffectView = [[UIVisualEffectView alloc] initWithEffect:blur];
    visualEffectView.frame = self.view.frame;
    [self.imageView addSubview:visualEffectView];

    UIVibrancyEffect *vibrancyEffect = [UIVibrancyEffect effectForBlurEffect:blur];
    UIVisualEffectView *ano = [[UIVisualEffectView alloc] initWithEffect:vibrancyEffect];
    ano.frame = self.view.frame;

    UILabel *label = [[UILabel alloc] init];
    label.font = [UIFont systemFontOfSize:40];
    label.frame = CGRectMake(0, self.view.frame.size.height - 120, self.view.frame.size.width, 80);
    label.textAlignment = NSTextAlignmentCenter;
    label.text = @"蒙奇·D·路飞";
    [visualEffectView.contentView addSubview:ano];
    [ano.contentView addSubview:label];
    
    [self.view addSubview:self.imageView];
```

####二、高斯模糊运动模糊等，先看效果图：
- 原图

![原图](http://upload-images.jianshu.io/upload_images/2665449-954940694c6fed53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 运动模糊效果图
![运动模糊](http://upload-images.jianshu.io/upload_images/2665449-d27b1bd30c08f616.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 代码如下

viewDidLoad 中的代码和“简单的毛玻璃效果”中的 viewDidLoad 是一样的，这里只简单贴出 button 被点击调用的代码

```
    CIImage *inputImage = [CIImage imageWithCGImage:self.imageView.image.CGImage];

    // CIGaussianBlur   高斯模糊
    // CIBoxBlur        均值模糊
    // CIDiscBlur       环形卷积模糊
    // CIMotionBlur     运动模糊
    CIFilter *filter = [CIFilter filterWithName:@"CIMotionBlur"];
    [filter setValue:inputImage forKey:kCIInputImageKey];
    [filter setValue:@5 forKey:kCIInputRadiusKey];
    CIContext *context = [CIContext contextWithOptions:nil];
    CIImage *outupImage = filter.outputImage;
    CGImageRef imageRef = [context createCGImage:outupImage fromRect:outupImage.extent];
    self.imageView.image= [UIImage imageWithCGImage:imageRef];
```

####三、滤镜效果，先看下效果图

- 原图
![原图](http://upload-images.jianshu.io/upload_images/2665449-503d38d5e10a65b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- “怀旧”效果图
![“怀旧”效果图](http://upload-images.jianshu.io/upload_images/2665449-f1196625bd86712d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- 代码如下

viewDidLoad 中的代码和“简单的毛玻璃效果”中的 viewDidLoad 是一样的，这里也只贴出 button 被点击调用的代码

```
    CIContext *context = [CIContext contextWithOptions:nil];
    CIImage *inputImage = [[CIImage alloc] initWithImage:self.imageView.image];
    
    // 怀旧  CIPhotoEffectInstant
    // 单色  CIPhotoEffectMono
    // 黑白  CIPhotoEffectNoir
    // 褪色  CIPhotoEffectFade
    // 色调  CIPhotoEffectTonal
    // 冲印  CIPhotoEffectProcess
    // 岁月  CIPhotoEffectTransfer
    // 铬黄  CIPhotoEffectChrome
    CIFilter *filter = [CIFilter filterWithName:@"CIPhotoEffectInstant"]; 
    [filter setValue:inputImage forKey:kCIInputImageKey];
    CIImage *result = [filter valueForKey:kCIOutputImageKey];
    CGImageRef cgImage = [context createCGImage:result fromRect:[result extent]];
    UIImage *resultImage = [UIImage imageWithCGImage:cgImage];
    self.imageView.image= [UIImage imageWithCGImage:resultImage.CGImage];
    
```


这是今晚整理的 CoreImage 相关的知识，如果哪里写的有问题，欢迎大家指正！另外给大家分享一篇文章，里面有很多 iOS 9 出来的 Core Image新滤镜，[在这里。](http://www.cocoachina.com/ios/20151118/14253.html)