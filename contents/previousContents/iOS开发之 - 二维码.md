# iOS开发之 - 二维码

> 在我们的日常生活中，二维码扫描越来越常见，比如支付宝支付扫描、微信支付扫描、信息获取等等，所以简单的整理了二维码相关的知识，总共分为两部分：生成二维码；扫描二维码。


#### 一、生成二维码
- 简单的生成二维码，先看效果图
如图所示，这个效果不太好，有些模糊
![简单生成二维码](http://upload-images.jianshu.io/upload_images/2665449-a0c118e13587ec11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 生成简单二维码的代码如下：

```
#import "ViewController.h"
// 需要导入此框架
#import <CoreImage/CoreImage.h>

@interface ViewController ()

@property (nonatomic, strong) UIImageView *imageView;
@end

@implementation ViewController

- (UIImageView *)imageView {

    if (!_imageView) {
        _imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 200, 200)];
        _imageView.center = self.view.center;
        [self.view addSubview:_imageView];
    }
    return _imageView;
}


- (void)viewDidLoad {
    [super viewDidLoad];
    [self creatQRCode];
}

// 生成二维码
- (void)creatQRCode {
    
    // 创建二维码滤镜
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    
    // 恢复滤镜的默认属性
    [filter setDefaults];
    
    // 给过滤器添加数据（纯文本，URL都可以）
    //    NSString *dataString = @"http://www.baidu.com";
    NSString *dataString = @"生成二维码";
    // 将字符串转换成NSData
    NSData *data = [dataString dataUsingEncoding:NSUTF8StringEncoding];
    
    // 设置滤镜 inputMessage 数据
    [filter setValue:data forKeyPath:@"inputMessage"];
    
    CIImage *outputImage = [filter outputImage];
    
    UIImage *image = [UIImage  imageWithCIImage:outputImage];
    
    // 把生成的二维码显示在 imageview 上
    self.imageView.image = image;
}
```

---
- 生成清晰二维码


![生成清晰二维码](http://upload-images.jianshu.io/upload_images/2665449-d5c8dcae91378594.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 代码如下：

```
#import "ViewController.h"
#import <CoreImage/CoreImage.h>

@interface ViewController ()

@property (nonatomic, strong) UIImageView *imageView;
@end

@implementation ViewController

- (UIImageView *)imageView {

    if (!_imageView) {
        _imageView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 200, 200)];
        _imageView.center = self.view.center;
        [self.view addSubview:_imageView];
    }
    return _imageView;
}


- (void)viewDidLoad {
    [super viewDidLoad];
    [self creatQRCode];
}

// 生成二维码
- (void)creatQRCode {
    
    // 创建二维码滤镜
    CIFilter *filter = [CIFilter filterWithName:@"CIQRCodeGenerator"];
    
    // 恢复滤镜的默认属性
    [filter setDefaults];
    
    // 给过滤器添加数据（纯文本，URL都可以）
    //    NSString *dataString = @"http://www.baidu.com";
    NSString *dataString = @"生成二维码";
    // 将字符串转换成NSData
    NSData *data = [dataString dataUsingEncoding:NSUTF8StringEncoding];
    
    // 设置滤镜 inputMessage 数据
    [filter setValue:data forKeyPath:@"inputMessage"];
    
    // 获取滤镜输出的二维码
    CIImage *outputImage = [filter outputImage];

    // 显示二维码
    self.imageView.image = [self creatClearImageFromCIImage:outputImage withSize:200];
}

- (UIImage *)creatClearImageFromCIImage:(CIImage *)image withSize:(CGFloat) size
{
    CGRect extent = CGRectIntegral(image.extent);
    CGFloat scale = MIN(size/CGRectGetWidth(extent), size/CGRectGetHeight(extent));
    
    size_t width = CGRectGetWidth(extent) * scale;
    size_t height = CGRectGetHeight(extent) * scale;
    // 创建灰度色调空间
    CGColorSpaceRef colorSpaceRef = CGColorSpaceCreateDeviceGray();
    CGContextRef bitmapRef = CGBitmapContextCreate(nil, width, height, 8, 0, colorSpaceRef, (CGBitmapInfo)kCGImageAlphaNone);
    CIContext *context = [CIContext contextWithOptions:nil];
    CGImageRef bitmapImage = [context createCGImage:image fromRect:extent];
    CGContextSetInterpolationQuality(bitmapRef, kCGInterpolationNone);
    CGContextScaleCTM(bitmapRef, scale, scale);
    CGContextDrawImage(bitmapRef, extent, bitmapImage);
    
    // 保存图片
    CGImageRef scaledImage = CGBitmapContextCreateImage(bitmapRef);
    CGContextRelease(bitmapRef);
    CGImageRelease(bitmapImage);
    return [UIImage imageWithCGImage:scaledImage];
}
```

#### 二、扫描二维码

```
#import "ViewController.h"
#import <AVFoundation/AVFoundation.h>

@interface ViewController () <AVCaptureMetadataOutputObjectsDelegate>

@property (nonatomic, strong) AVCaptureSession *session;
@property (nonatomic, strong) AVCaptureVideoPreviewLayer *layer;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // 创建拍摄会话
    self.session = [[AVCaptureSession alloc] init];
    
    // 添加输入设备
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:device error:nil];
    [self.session addInput:input];
    
    // 添加输出数据
    AVCaptureMetadataOutput *output = [[AVCaptureMetadataOutput alloc] init];
    [output setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];
    [self.session addOutput:output];
    
    // 设置输入元数据的类型
    [output setMetadataObjectTypes:@[AVMetadataObjectTypeQRCode]];
    
    // 添加扫描图层
    self.layer = [AVCaptureVideoPreviewLayer layerWithSession:self.session];
    self.layer.frame = self.view.bounds;
    [self.view.layer addSublayer:self.layer];
    
    // 启动会话
    [self.session startRunning];
}

#pragma mark - 扫描到数据就会执行该代理方法
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection
{
    if (metadataObjects.count > 0) {
         // 提示：如果需要对url或者文本等信息进行扫描，可以在此进行扩展
        AVMetadataMachineReadableCodeObject *object = [metadataObjects lastObject];
        NSLog(@"%@", object.stringValue);
        
        // 如果扫描完成，停止会话
        [self.session stopRunning];
        
        // 删除预览图层
        [self.layer removeFromSuperlayer];
    } else {
        NSLog(@"没有扫描到数据");
    }
}
```

最后说几个二维码相关的注意点：

 - 注意点：
 二维码都有一定的容错部分，意思就是有部分污损或者破损都没有关系，照常识别。但是也是有限度的，这根据生成时使用的纠错级别而定，可以有7％~30％左右的损坏。
 
- 二维码使用基本原则：
 1、三个角上的“回”及“回”字周围的底色不要动
 2、中间部分和不带“回”字的一角是可以填图片的（中间最好）
 3、如果中间有小的“回”字，能不变就不变，能少变就少变
 4、尽可能放大二维码后再添加图片，不要添加图片后放大
 5、生成时尽量选择较高的纠错级别

![二维码](http://upload-images.jianshu.io/upload_images/2665449-daf8123ea95c1514.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
