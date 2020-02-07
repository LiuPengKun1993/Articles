# iOS开发之 - AFNetworking - 网络请求

> 最近一直很忙也没有时间写博客，今天正好有点空闲时间，所以就对 [AFNetworking](https://github.com/AFNetworking/AFNetworking/) 做了一些简单的总结，特地记录下来，以便遇到的朋友们看看！！！

先说下这篇文章的大概内容，以便朋友们能够快速找到自己需要的。这篇文章主要介绍了三种网络请求的渠道，第一种是通过苹果自带的 NSURLSession 请求网络；第二种是通过 AFNetworking2.x 请求网络；最后一种是通过 AFNetworking3.0 请求网络。下边是这三种网络请求渠道的 demo.


##一、苹果自带的请求网络方法：
这里介绍了三种，GET 请求、POST 请求、代理方式请求。

1.GET 请求:

```
    // 获得 NSURLSession 对象
    NSURLSession *session =  [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:nil delegateQueue:[NSOperationQueue mainQueue]];
    
    // 创建任务
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@" "]] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"data ~~~ %@ ~~~ response ~~~ %@ ~~~ error~~~ %@", data, response , error);
    }];
    
    // 启动任务
    [dataTask resume];
```
2.POST 请求:

```
    // 获得NSURLSession对象
    NSURLSession *session = [NSURLSession sharedSession];
    
    // 创建请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@" "]];
    
    // 设置请求方法
    request.HTTPMethod = @"POST"; 
    
    // 设置请求体
    request.HTTPBody = [@" " dataUsingEncoding:NSUTF8StringEncoding]; 
    
    // 创建任务
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"data ~~~ %@ ~~~ response ~~~ %@ ~~~ error~~~ %@", data, response , error);
    }];
    
    // 启动任务
    [task resume];
```
3.代理:NSURLSessionDataDelegate

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@" "]];
    
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    
    // 如果创建task时，有block，代理会失效
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request];
    
    [task resume];
}

#pragma mark - NSURLSessionDataDelegate
#pragma mark - 接收到响应头调用的协议方法
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler {
    
    NSHTTPURLResponse *httpRes = (NSHTTPURLResponse *)response;
    NSLog(@"%@",httpRes.allHeaderFields);
    
    //通过设置 completionHandler, Block 可以设置后面的响应体中的数据是否继续发送,默认是不发送
    completionHandler(NSURLSessionResponseAllow);
    
}

#pragma mark - 数据请求完成后，调用的协议方法，data是响应体中的数据
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask
    didReceiveData:(NSData *)data {
    
    NSLog(@"%@",data);
}

#pragma mark - 不管请求成功还是失败，最终都会调用此方法
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
    
    if (error) {
        NSLog(@"请求失败");
    } else {
        NSLog(@"请求成功");
    }
}
```


##二、AFNetworking2.x

1.GET 请求

```
 AFHTTPRequestOperationManager *operationManager = [AFHTTPRequestOperationManager manager];
    
    [operationManager GET:@" " parameters:nil
     success:^(AFHTTPRequestOperation *operation, id responseObject) {
         NSLog(@"请求成功");
     } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
         NSLog(@"请求失败");
     }];
```
2.POST 请求

```
 AFHTTPRequestOperationManager *operationManager = [AFHTTPRequestOperationManager manager];
    
    [operationManager POST:@" " parameters:nil
      success:^(AFHTTPRequestOperation *operation, id responseObject) {
          NSLog(@"请求成功");
      } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
          NSLog(@"请求失败");
   }];
```  
      
##三、AFNetworking3.0

1.GET 请求

```
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    
    // 解析服务器返回的JSON数据
//        [AFJSONResponseSerializer serializer];
    // 解析服务器返回的XML数据
//        [AFXMLParserResponseSerializer serializer];

	// 参数1: 请求的网址
	// 参数2: 参数
	// 参数3: 当前的进度
	// 参数4: 请求成功
	// 参数5: 请求失败
   [sessionManager GET:@" " parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {
        NSLog(@"进度");
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"请求成功");
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"请求失败");
    }];
```

2.POST 请求

```
// 创建管理者对象
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    
    // 设置请求参数
    NSString *urlStr = @" ";
    
    // 需要设置 body 体
    NSDictionary *dic = @{@" " : @" ",@" " : @" "};

    [sessionManager POST:urlStr parameters:dic progress:^(NSProgress * _Nonnull downloadProgress) {
        NSLog(@"进度");
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"请求成功");
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"请求失败");
    }];
```
上面的一些代码都是基础性代码，因此也没怎么做注释，如果有疑问的话可以私信交流...另外代码有什么问题的话也希望大家能够指出！谢谢大家！