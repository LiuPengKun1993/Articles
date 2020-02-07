# iOS开发之 - 支付宝支付

> 在开发中，很多时候我们都会用到支付宝支付和微信支付，前段时间已经总结过[微信支付流程](http://www.jianshu.com/p/a17b37cb8fe3)，这里再说下支付宝支付（相对来说觉得支付宝有点坑），先说下支付宝支付的流程，如下图：

### 一、支付流程理解

先看个图
![支付流程](http://upload-images.jianshu.io/upload_images/2665449-111551cb2f5ba892.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

流程和咱们平时在手机上买东西是一样的：
1.用户选好商品后，点击提交订单（一般是这样），选择使用支付宝付款。
2.手机客户端（你做的APP）把用户选择的商品的信息传给你们后台服务器。
3.后台的服务器将各种数据拼接签名后生成一个签名后的字符串，回传到客户端APP上。
4.用户点击确认支付按钮，调用手机支付宝客户端，利用后台传过来的那个参数调起支付宝，让支付宝客户端传给他们服务器交互，进行付款。(这一步是支付宝自己完成的，安全性高)
5.支付宝的服务器将支付的结果（可能成功也可能不成功）返回给手机支付宝客户端和你们公司的后台服务器。
6.你们公司后台服务器收到后一般是更新下数据信息，手机支付宝客户端会显示一下支付成功。


### 二、支付流程详解
1. 下载支付宝SDK
[App支付DEMO&SDK](https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.B7TwKK&treeId=54&articleId=104509&docType=1)

2. 进入支付平台注册应用
[创建应用](https://openhome.alipay.com/platform/appCreate.htm)
3. 获取支付相关的 '私钥' 和 '密钥'
[RSA私钥及公钥生成](https://doc.open.alipay.com/doc2/detail?treeId=44&articleId=103242&docType=1)

4. 集成支付宝SDK
可以先看下[官方集成文档](https://doc.open.alipay.com/doc2/detail.htm?spm=a219a.7386797.0.0.IGzFSR&treeId=48&articleId=103346&docType=1)，下面是一些具体步骤：

4.1 导入文件（如下图）

![导入相应的文件](http://upload-images.jianshu.io/upload_images/2665449-79cac6572336637c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里有一个注意点：如果不在客户端上签名，只需要发送订单和处理支付返回结果，只需要添加AlipaySDK.bundle和AlipaySDK.framework就行了。

4.2 导入相关的依赖库

![导入相关的依赖库](http://upload-images.jianshu.io/upload_images/2665449-df522e1f9642c24e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.3 为URL Types 添加支付宝回调scheme

![设置回调scheme](http://upload-images.jianshu.io/upload_images/2665449-39aa192999a9d489.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 identifier必须为 alipayShare
URL Schemes 命名规则：ap+AppID，需要和代码中的一致

4.4  代码部分
- 发送订单的方法

```
- (void)payOrder:(NSString *)orderStr
      fromScheme:(NSString *)schemeStr
        callback:(CompletionBlock)completionBlock;
```

- 在AppDelegate中处理事件回调

在 APAppDelegate.m 文件中，增加引用代码：

```
#import <AlipaySDK/AlipaySDK.h>
```

在@implementation AppDelegate中增加如下代码：

```
#pragma mark - 处理返回结果
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    
    //如果极简开发包不可用，会跳转支付宝钱包进行支付，需要将支付宝钱包的支付结果回传给开发包
    if ([url.host isEqualToString:@"safepay"]) {
        [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
            //【由于在跳转支付宝客户端支付的过程中，商户app在后台很可能被系统kill了，所以pay接口的callback就会失效，请商户对standbyCallback返回的回调结果进行处理,就是在这个方法里面处理跟callback一样的逻辑】
            NSLog(@"result = %@",resultDic);
        }];
    }
    if ([url.host isEqualToString:@"platformapi"]){//支付宝钱包快登授权返回authCode
        
        [[AlipaySDK defaultService] processAuthResult:url standbyCallback:^(NSDictionary *resultDic) {
            //【由于在跳转支付宝客户端支付的过程中，商户app在后台很可能被系统kill了，所以pay接口的callback就会失效，请商户对standbyCallback返回的回调结果进行处理,就是在这个方法里面处理跟callback一样的逻辑】
            NSLog(@"result = %@",resultDic);
        }];
    }
    return YES;
}
```

---
- 集成到这里就差不多了，另外再推荐几篇好文章：


1. 这篇是客户端做的时候遇到的坑：[点这里](http://www.mamicode.com/info-detail-1076023.html)。
2. 如果签名数据是在App上做的，可以参考下这篇文章，已经封装好的，[点这里](http://www.360doc.com/content/15/0703/10/20918780_482317876.shtml)。
3. 这个是官方的集成流程，很详细，[点这里](https://doc.open.alipay.com/doc2/detail.htm?spm=0.0.0.0.hEK6Hb&treeId=59&articleId=103675&docType=1)。
4. [iOS 集成支付宝](http://www.jianshu.com/p/4a6232d8294b)
5. [集成支付宝钱包支付iOS SDK的方法与经验](http://www.jianshu.com/p/fe56e122663e)
