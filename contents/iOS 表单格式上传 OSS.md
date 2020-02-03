#### 前言
> 最近的工作是负责写埋点 SDK，学到了很多知识，不仅仅是 iOS 方面的，还包括 `JavaScript`、`NodeJS`、`大数据`等方面的一些知识，有时间会把自己学到的知识全部记录下来，一则是加深印象，二是如果恰好遇到有需要的人，希望能提供一些思路。

#### 1.为什么要用表单格式上传 OSS

为什么要用表单格式上传数据到 OSS，大家知道，如果借用 [AliyunOSSiOS](https://github.com/aliyun/aliyun-oss-ios-sdk) 的话，会非常 easy 且轻松搞定 OSS 数据上传，那么为什么非要拼接成表单格式上传到 OSS 呢？

这就和具体需求有关了。

我们的需求是，为公司的几个 APP 做埋点 SDK，考虑到这几个 APP 都有可能使用 AliyunOSSiOS，那么为了避免冲突，兼容本身已集成 AliyunOSSiOS 的 APP，我们的埋点 SDK 就不能再集成 AliyunOSSiOS 了，于是就选择了表单格式上传。

#### 2.核心代码及思路

##### 2.1 先调用服务端接口获取签名

```
[NNNetworking requestWithURL:@"获取签名的接口" params:@{@"key":@"value"} success:^(id  _Nullable responseObject) {
    if (responseObject) {
        NSDictionary *origDict = responseObject;
        NSString *statusCode = origDict[@"statusCode"];
        if ([statusCode isEqualToString:@"200"]) {
            NSDictionary *resultDict = origDict[@"result"];
            NSString *policy = resultDict[@"policy"];
            NSString *OSSAccessKeyId = resultDict[@"OSSAccessKeyId"];
            NSString *signature = resultDict[@"signature"];
            NSString *key = resultDict[@"dirPath"];
            
            NSMutableDictionary *parameterDict = [[NSMutableDictionary alloc]init];
            [parameterDict setValue:OSSAccessKeyId forKey:@"OSSAccessKeyId"];
            [parameterDict setValue:policy            forKey:@"policy"];
            [parameterDict setValue:signature         forKey:@"signature"];
            [parameterDict setValue:key forKey:@"key"];
            // 上传数据到 OSS
            [NNNetworking uploadOSSWithParameter:parameterDict uploadDataArray:@[] success:^(id  _Nonnull obj) {
                
            } fail:^(NSError * _Nonnull error) {
                
            }];
        }
    }
} failure:^(NSError * _Nullable error) {
    
}];
```

由于自己做的本身就是 SDK，所以强制不能使用第三方框架，比如常用的网络框架 AFNetworking 或 Alamofire 都不能使用。


##### 2.2 接着拼接签名、策略、key 等字段，拼接成表单结构，然后上传至 OSS

```
+ (void)uploadOSSWithParameter:(NSDictionary *)parameterDict uploadDataArray:(NSArray *)dataArray success:(void (^)(id obj))success fail:(void (^)(NSError *error))fail {
    // 分界线的标识符
    NSString *TWITTERFON_FORM_BOUNDARY = @"IaX0GHJdM5gRIXY_S37dFzTjrYcygaI4lr8JjgHY_hUURrbjP8LbpzJ7KEQ2AbcY5yU3Jc";
    // 根据 url 初始化 request
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http://桶名.oss-cn-beijing.aliyuncs.com"] cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:10];
    // 分界线
    NSString *MPboundary = [[NSString alloc]initWithFormat:@"--%@",TWITTERFON_FORM_BOUNDARY];
    // 结束符
    NSString *endMPboundary=[[NSString alloc]initWithFormat:@"%@--",MPboundary];
    
    // http body 的字符串
    NSMutableString *body = [[NSMutableString alloc] init];
    
    // 参数的集合的所有key的集合
    NSArray *keys= [parameterDict allKeys];
    // 遍历keys
    for(int i = 0; i < [keys count]; i++) {
        // 得到当前key
        NSString *key=[keys objectAtIndex:i];
        // 添加分界线，换行
        [body appendFormat:@"%@\r\n",MPboundary];
        // 添加字段名称，换2行
        [body appendFormat:@"Content-Disposition: form-data; name=\"%@\"\r\n\r\n",key];
        // 添加字段的值
        [body appendFormat:@"%@\r\n",[parameterDict objectForKey:key]];
    }
    // 添加分界线，换行
    [body appendFormat:@"%@\r\n",MPboundary];
    // 声明pic字段，文件名为boris.png
    [body appendFormat:@"%@", [NSString stringWithFormat:@"Content-Disposition: form-data; name=\"file\"; filename=\"%@\"\r\n",@"MyFilename"]];
    // 声明上传文件的格式
    [body appendFormat:@"Content-Type: image/png\r\n\r\n"];
    NSString *paraString = [dataArray componentsJoinedByString:@","];
    
    NSString *end=[[NSString alloc]initWithFormat:@"\r\n%@",endMPboundary];
    
    // 声明myRequestData，用来放入http body
    NSMutableData *myRequestData = [NSMutableData data];
    // 将body字符串转化为UTF8格式的二进制
    [myRequestData appendData:[body dataUsingEncoding:NSUTF8StringEncoding]];
    // data加入
    [myRequestData appendData:[paraString dataUsingEncoding:NSUTF8StringEncoding]];
    // 加入结束符--AaB03x--
    [myRequestData appendData:[end dataUsingEncoding:NSUTF8StringEncoding]];
    
    // 设置HTTPHeader中Content-Type的值
    NSString *content = [[NSString alloc]initWithFormat:@"multipart/form-data; boundary=%@",TWITTERFON_FORM_BOUNDARY];
    [request setValue:content forHTTPHeaderField:@"Content-Type"];
    // 设置Content-Length
    [request setValue:[NSString stringWithFormat:@"%d", (int)[myRequestData length]] forHTTPHeaderField:@"Content-Length"];
    
    // 设置http body
    [request setHTTPBody:myRequestData];
    // http method
    [request setHTTPMethod:@"POST"];
    // 连接(NSURLSession)
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (error) {
            
        } else {
            if (success) {
                success(@"");
            }
        }
        NSString *str = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"body:%@", str);
    }];
    [dataTask resume];
}
```


#### 参考文档：

- [PostObject](https://help.aliyun.com/document_detail/31988.html?spm=5176.product31815.6.881.0b4gqm)
- [表单上传](https://helpcdn.aliyun.com/document_detail/31849.html)