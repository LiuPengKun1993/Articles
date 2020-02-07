# iOS开发之 - 微信支付

![iOS开发之 - 微信支付](http://upload-images.jianshu.io/upload_images/2665449-0d0a74ab4c5830ac.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 作为 iOS 开发者，支付无疑是很重要的，特别是这两年几乎到处都是微信支付、支付宝支付。相比支付宝支付，觉得微信支付还算简单，但还是建议预先看下[文档](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=9_1)。开始说文章的内容吧，这篇文章主要分为四个部分：创建应用、了解支付的流程、集成微信SDK，最后是实现支付的demo。


- **第一步：创建应用**

*通常来说注册APP这样的事情是部门经理做的，所以这一步也可以忽略～～～*

在 [微信开放平台](https://open.weixin.qq.com/cgi-bin/index?t=home/index&lang=zh_CN&token=0ebf4697e5671e8567a5c5f4682a4c6cb0fcaa92) 的管理中心 [创建一个应用。](https://open.weixin.qq.com/cgi-bin/frame?t=home/app_tmpl&lang=zh_CN)此应用官方称七日内审核通过，一般都是三四天！而关于填写经营信息、填写商户信息、填写对公帐号信息在这里就忽略不写了。有兴趣的朋友可以了解下[这篇文章](http://494075592.blog.51cto.com/10682677/1701260)。

![创建应用](http://upload-images.jianshu.io/upload_images/2665449-723d6f42f6b41ba2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- **第二步：了解支付的流程**

如果做过其它支付，就会发现这些流程其实都差不多，就像我们平时用手机支付买东西一样。下边的流程两分钟大概就能读完，很容易理解！

1> 用户使用 APP 客户端，选择商品下单。
2> APP 客户端将用户的商品数据传给商户服务器，请求生成支付订单。
3> 商户后台调用统一下单 API 向微信的服务器发送请求，微信服务器生成预付单，并生成一个 prepay_id 返回给商户后台。
4> 商户后台将这个 prepay_id 返回给商户客户端。
5> 用户点击确认支付，这时候商户客户端调用 SDK 打开微信客户端，进行微信支付。
6> 微信客户端向微信服务器发起支付请求并返回支付结果（他们之间交互用的就是prepay_id这个参数，微信的服务器要验证微信客户端传过去的参数是否跟第三步中生成的那个id一致）。
7> 用户输入支付密码后，微信客户端提交支付授权，跟微信服务器交互，完成支付
8> 微信服务器给微信客户端发送支付结果提示，并异步给商户服务器发送支付结果通知。
9> 商户客户端通过支付结果回调接口查询支付结果，并向后台检查支付结果是否正确，后台返回支付结果。
10> 商户客户端显示支付结果，完成订单，发货。
- **第三步：集成SDK**

其实严格来说，这一步才是微信支付的真正开始。首先我们应该下载 [微信SDK](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319164&lang=zh_CN)，如有需要，还可把[【微信支付】APP支付示例](https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=11_1_)下载下来看看。下面是集成 SDK 的具体步骤：

1.把下载好的 SDK 导入工程（如下图）（我下的是OpenSDK1.7.4版本）

![把下载好的 SDK 导入工程](http://upload-images.jianshu.io/upload_images/2665449-a0dfa21d98e8d045.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.导入微信支付SDK依赖库（如下图）

![导入微信支付SDK依赖库](http://upload-images.jianshu.io/upload_images/2665449-f4bd0524a4a2d18f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.设置URL Scheme
在注册微信平台APP的时候，会给一个唯一识别标识符（APPID）,需要填写在工程中。（如下图）

![设置URL Scheme](http://upload-images.jianshu.io/upload_images/2665449-8c7b570a18293fc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
*配置好上述参数后就可以写代码了～～～*
- **第四部：代码**

代码分为以下步骤：

1.在 Appdelegate 中注册微信 APPID
这是首先要做的事情。
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    // 向微信终端注册ID
    [WXApi registerApp:@"wx0f8d5e0eadd2d4b7" withDescription:@"demo test"];
    
    return YES;
}
```
2.获取微信支付所必要的参数
接着，在微信支付的.m文件中获取微信支付所必要的参数。为了提高数据安全性，下单、签名等操作一般都是在后台完成的，因此以下这些参数后台会给我们。我们主要得知道这些参数如何设置：
```
    // 发起微信支付，设置参数
    PayReq *request   = [[PayReq alloc] init];  // 创建支付对象
    request.openID = @""; // 由用户微信号和AppID组成的唯一标识
    request.partnerId = @""; // 商家ID
    request.prepayId  = @""; // 预支付订单ID
    request.package   = @"Sign=WXPay"; // 数据和签名
    request.nonceStr  = @""; // 随机编码
    NSDate *date = [NSDate date];
    NSString * timeSp = [NSString stringWithFormat:@"%ld", (long)[date timeIntervalSince1970]];
    UInt32 timeStamp = [timeSp intValue];
    request.timeStamp = timeStamp; // 时间戳
    request.sign = @""; // 签名，签名一般都会加密
    
    //发送请求到微信，等待微信返回 onResp
    [WXApi sendReq:request];
```

3.微信支付回调
最后要在Appdelegate.m文件中添加微信支付结果 onResp 回调方法
```
// 判断发起的请求是否为微信支付，如果是就回调
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options {
    return [WXApi handleOpenURL:url delegate:self];
}

#pragma mark - WXApiDelegate
// 微信支付结果回调方法
- (void)onResp:(BaseResp *)resp {
    NSString *payResoult = [NSString stringWithFormat:@"%d", resp.errCode];
    
    if([resp isKindOfClass:[PayResp class]]){

        /**
         WXSuccess           = 0,    成功
        WXErrCodeCommon     = -1,    普通错误类型
        WXErrCodeUserCancel = -2,    用户点击取消并返回
        WXErrCodeSentFail   = -3,    发送失败
        WXErrCodeAuthDeny   = -4,    授权失败
        WXErrCodeUnsupport  = -5,    微信不支持
         */
        //支付返回结果，实际支付结果需要去微信服务器端查询
        switch (resp.errCode) {
            case WXSuccess:
                payResoult = @"支付成功";
                break;
            case WXErrCodeCommon:
                payResoult = @"支付失败";
                break;
            case WXErrCodeUserCancel:
                payResoult = @"用户点击取消并返回";
                break;
            default:
                // 错误码 以及 错误提示字符串
                payResoult = [NSString stringWithFormat:@"支付结果：失败！retcode = %d, retstr = %@", resp.errCode,resp.errStr];
                break;
        }
    }
}
```

---

结束语：在开发中，微信支付常常与其它技术配合使用，比如MD5加密，AFN等等。有空闲的话会一一整理出来与大家分享！另外文章写的如果有什么错误，还请大家指出来，谢谢大家！