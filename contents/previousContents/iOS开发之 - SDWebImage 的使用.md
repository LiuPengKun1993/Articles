# iOS开发之 - SDWebImage 的使用

![SDWebImage](http://upload-images.jianshu.io/upload_images/2665449-6e9f0fccaeeca1e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> [SDWebImage](https://github.com/rs/SDWebImage) 是一个开源的第三方库，在我们 iOS 开发中用到 SDWebImage 的地方有很多，比如在 tableView 中，在 collectionView 中......不然它也不会高达 16000+ 这么多的 Star 了😄。下面是总结的一些 SDWebImage 相关的具体用法。

首先用 cocoapods 导入 SDWebImage 框架，网上有很多安装以及使用 cocoapods 的教程，在此就不累述了，这篇文章的内容是在导入 SDWebImage 基础之上的，主要是总结了一些 SDWebImage 的具体用法。


- 简单下载图片
```
 方式一：
     // 只下载图片
    [self.imageView sd_setImageWithURL:@"URL地址"]];

 方式二：
     /**
     第一个参数:图片的url
     第二个参数:占位图片
     */
    [self.imageView sd_setImageWithURL:[NSURL URLWithString:@"URL地址"]
                      placeholderImage:[UIImage imageNamed: @"占位图片名"]];
```
- 下载成功或失败之后做的事情
```
方式一：
     /**
     第一个参数:图片的url
     block:下载成功或失败之后的回调
     */
[self.imageView sd_setImageWithURL:[NSURL URLWithString:@"URL地址"] 
completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
        NSLog(@"下载成功或失败之后的回调");
    }];
方式二：
    /**
     第一个参数:图片的url
     第二个参数:占位图片
     block:下载成功或失败之后的回调
     */
    [self.imageView sd_setImageWithURL:[NSURL URLWithString:@"URL地址"]
                      placeholderImage:[UIImage imageNamed: @"占位图片名"]
                             completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
       NSLog(@"下载成功或失败之后的回调");
 }];
方式三：
    /**
     第一个参数:图片的url
     第二个参数:占位图片
     第三个参数:下载图片的策略
     第四个参数:progress进度
     第五个参数:completed 下载完成之后的回调
     */
    [self.imageView sd_setImageWithURL:[NSURL URLWithString:@"URL地址"]
                      placeholderImage:[UIImage imageNamed: @"占位图片名"]
                               options:SDWebImageRetryFailed
                              progress:^(NSInteger receivedSize, NSInteger expectedSize) {
        NSLog(@"progress进度");
    }
                             completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
        NSLog(@"下载成功或失败之后的回调");
    }];
```

- 借用一个方法简单的介绍一下 options 中的选项
```
     /**
     //失败后重试
     SDWebImageRetryFailed = 1 << 0,
     //UI交互期间开始下载，导致延迟下载比如UIScrollView减速。
     SDWebImageLowPriority = 1 << 1,
     //只进行内存缓存
     SDWebImageCacheMemoryOnly = 1 << 2,
     //这个标志可以渐进式下载,显示的图像是逐步在下载
     SDWebImageProgressiveDownload = 1 << 3,
     //刷新缓存
     SDWebImageRefreshCached = 1 << 4,
     //后台下载
     SDWebImageContinueInBackground = 1 << 5,
     //NSMutableURLRequest.HTTPShouldHandleCookies = YES;
     SDWebImageHandleCookies = 1 << 6,
     //允许使用无效的SSL证书
     //SDWebImageAllowInvalidSSLCertificates = 1 << 7,
     //优先下载
     SDWebImageHighPriority = 1 << 8,
     //延迟占位符
     SDWebImageDelayPlaceholder = 1 << 9,
     //改变动画形象
     SDWebImageTransformAnimatedImage = 1 << 10,
     */
    [self.imageView sd_setImageWithURL:[NSURL URLWithString:@"URL地址"] 
placeholderImage:[UIImage imageNamed: @"占位图片名"] 
options:SDWebImageRetryFailed];
```

- 使用 SDWebImageManager 下载图片
```
SDWebImageManager *imageManager = [SDWebImageManager sharedManager];
    /**
     第一个参数:图片的url
     第二个参数:下载图片的(策略)
     第三个参数:progress进度
     第四个参数:completed 下载完成之后的回调
     */
    [imageManager downloadImageWithURL:[NSURL URLWithString:@"URL地址"]
                               options:SDWebImageRetryFailed
                              progress:^(NSInteger receivedSize, NSInteger expectedSize) {
        NSLog(@"progress进度");
    }
                             completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
        NSLog(@"下载成功或失败之后的回调");
                                 if (image) {
                                     self.image = image;
                                 }
    }];
```

- 使用 SDWebImageDownloader 下载图片

```
SDWebImageDownloader *imageDownloader = [SDWebImageDownloader sharedDownloader];
    /*
     第一个参数:图片的url
     第二个参数:下载图片的策略
     第三个参数:progressBlock 进度
     第四个参数:completedBlock 下载完成之后的回调
     */
    [imageDownloader downloadImageWithURL:[NSURL URLWithString:@"URL地址"]
                                  options:SDWebImageDownloaderLowPriority
                                 progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                                     NSLog(@"progress进度");
                                 }
                                completed:^(UIImage *image, NSData *data, NSError *error, BOOL finished) {
                                    NSLog(@"下载成功或失败之后的回调");
                                    if (image) {
                                        self.image = image;
                                    }
                                }];
```
- 内存警告时调用

```
// 当程序收到内存警告时会调用这个方法
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // 清除缓存
    [[SDWebImageManager sharedManager].imageCache clearDisk]; // clean:删除过期缓存
    [[SDWebImageManager sharedManager].imageCache cleanDisk]; // clear:直接删除然后重新创建
    // 取消下载
    [[SDWebImageManager sharedManager] cancelAll];

}
```

- 其它一些很有用的方法

```
// 取消掉当前所有的下载
- (void)cancelAll;

// 检查是否有图片在下载
- (BOOL)isRunning;

// 将图片存入cache的方法
- (void)saveImageToCache:(UIImage *)image forURL:(NSURL *)url;

// 通过图片的url判断其是否已经存在
- (BOOL)cachedImageExistsForURL:(NSURL *)url;

// 检测一个image是否已经被缓存到磁盘(是否存且仅存在disk里).
- (BOOL)diskImageExistsForURL:(NSURL *)url;

// 如果检测到图片已经被缓存,那么执行回调block
- (void)cachedImageExistsForURL:(NSURL *)url
                     completion:(SDWebImageCheckCacheCompletionBlock)completionBlock;

 // 如果检测到图片已经在磁盘中,那么执行回调block
- (void)diskImageExistsForURL:(NSURL *)url
                   completion:(SDWebImageCheckCacheCompletionBlock)completionBlock;
```

以上是关于 SDWebImage 的一些用法总结，以后再用到其它的了，会再来补充。另外如果有写错的地方，欢迎朋友们指正！谢谢😊！