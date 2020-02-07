# iOS开发之 - AFNetworking - 上传、下载、网络监听

这一篇是接着上一篇总结的，如果有朋友需要了解网络请求的内容，可以[到这里](http://www.jianshu.com/p/d23f4f913af1)。和上篇一样，先说下这篇文章的大概内容，以便朋友们能够快速找到自己需要的。这篇文章主要介绍了 AFNetworking 的三个很广泛的应用：上传、下载以及网络监听。如果有朋友想了解苹果自带的 NSURLSession 相关的上传与下载，可以[到这里](http://www.jianshu.com/p/be94889824d9)。接下来是使用 AFNetworking 上传、下载以及网络监听的 demo.


##一、上传：
这里介绍了 AFN 的两种上传方式：

1.通过工程中的文件上传

```
     // 创建管理者对象
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    
    // 请求参数
    NSString *urlStr = @" ";
    
    NSDictionary *dic = @{@" " : @" ",@" " : @" "};
    
    // 发送 post 请求，上传一张图片
    [sessionManager POST:urlStr parameters:dic constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {
        
        NSString *filePath = [[NSBundle mainBundle] pathForResource:@"a.jpg" ofType:nil];
        NSData *imgData = [NSData dataWithContentsOfFile:filePath];
        
        // 将图片数据拼接，进行上传
        [formData appendPartWithFileData:imgData name:@"pic" fileName:@"filename" mimeType:@"image/jpg"];
        
    } progress:^(NSProgress * _Nonnull uploadProgress) {
        
        // 获取上传的进度
        NSLog(@"%.2f",uploadProgress.fractionCompleted);
        NSLog(@"%@",[NSThread currentThread]); //子线程
        
        
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        // 请求成功
        NSLog(@"请求成功：%@",responseObject);
        NSLog(@"%@",[NSThread currentThread]); //主线程
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        
        // 请求失败
        NSLog(@"请求失败：%@",error);
    }];
```

2.根据URL路径上传

```
     // 创建管理者对象
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    
    [sessionManager POST:@" " parameters:@{@" " : @" "} constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
        
    // 在block中设置需要上传的文件
    [formData appendPartWithFileURL:[NSURL fileURLWithPath:@"文件路径"] name:@"file" error:nil];
    
    } success:^(NSURLSessionDataTask *task, id responseObject) {
        
        NSLog(@"成功：%@", responseObject);
        
    } failure:^(NSURLSessionDataTask *task, NSError *error) {
        NSLog(@"失败：%@", error);
	}];
```	
	
	
##二、下载

```
	// 创建管理者对象
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    
    // 确定请求的URL地址
    NSURL *url = [NSURL URLWithString:@" "];
    
    // 创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    
    // 下载文件
    NSURLSessionDownloadTask *task = [manager downloadTaskWithRequest:request progress:^(NSProgress * _Nonnull downloadProgress) {
        
        // block会实时调用
        NSLog(@"%.2f",downloadProgress.fractionCompleted);
        
    } destination:^NSURL * _Nonnull(NSURL * _Nonnull targetPath, NSURLResponse * _Nonnull response) {

        // NSHomeDirectory()可以得到应用程序目录的路径
        // 返回一个URL存储文件
        NSString *filePath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents/***.mp3"];
        
        NSURL *url = [NSURL fileURLWithPath:filePath];
        
        return url;
        
    } completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nullable filePath, NSError * _Nullable error) {
        
        if (error) {
            NSLog(@"下载失败 ~~~ %@",error);
        }else {
            
            NSLog(@"下载成功");
        }
    }];
    
    // 开始下载任务
    [task resume];
```
 
##三、网络监听（监听手机网络的不同状态）


```
	 // 获得网络监控的管理者
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager manager];
    
    // 只要网络环境发生变化，就会调用此 block
    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        
        /* 枚举里面的四个状态
         AFNetworkReachabilityStatusUnknown          = -1,  未知
         AFNetworkReachabilityStatusNotReachable     = 0,   不可用
         AFNetworkReachabilityStatusReachableViaWWAN = 1,   手机自带网络
         AFNetworkReachabilityStatusReachableViaWiFi = 2,   wifi
         */
        
        switch (status) {
            case AFNetworkReachabilityStatusUnknown:
                NSLog(@"未知");
                break;
                
            case AFNetworkReachabilityStatusNotReachable:
                NSLog(@"不可用");
                break;
                
            case AFNetworkReachabilityStatusReachableViaWWAN:
                NSLog(@"手机自带网络");
                break;
                
            case AFNetworkReachabilityStatusReachableViaWiFi:
                NSLog(@"Wifi");
                break;
                
            default:
                break;
        }
        
    }];
    
    // 开始监听
    [manager startMonitoring];
```
这是今天做的一些关于 AFNetworking 的总结，如果有错误或疏忽的地方，希望大家能够指出！谢谢大家！！！