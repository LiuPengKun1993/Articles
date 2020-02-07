
# iOS开发之 - 银联支付

> 前阵子看了很多篇博客，有很多朋友吐槽说银联支付怎么怎么坑，所以集成的时候小心翼翼，但集成完银联支付之后，觉得相对于支付宝支付微信支付而言，银联支付还可以说的过去。下面就来介绍一下怎样快速的集成银联支付。

- 首先，下载[银联支付SDK](https://open.unionpay.com/ajweb/help/search?category=aj&keyword=开发包)（这个有点不好找），里面包含需要的库文件和详细的文档；下载好开发包之后，进行解压，解压成下面这样的文件

![银联支付SDK](http://upload-images.jianshu.io/upload_images/2665449-02336a46b0ff2407.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 官方文档也在这个目录下：

![银联支付官方文档](http://upload-images.jianshu.io/upload_images/2665449-0602bf8cbd8aea0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 下面开始集成SDK


1. 导入文件（文件目录：app开发包/控件开发包/upmp_iphone/paymentcontrol）

![导入文件](http://upload-images.jianshu.io/upload_images/2665449-d3b3f6332abf8f6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：记得老版本是三个文件，现在是新版本，只有两个

到这里的话，其实银联支付就做了一半了，是不是觉得很轻松很 easy 😉，接下来像我们 iOS 客户端再简单调用一个方法就行啦

- 支付接口调用
商户App从商户服务器获取tn，当tn不为空时，调用支付接口。

```
  //当获得的tn不为空时，调用支付接口
  if (tn != nil && tn.length > 0)
  {
        [[UPPaymentControl defaultControl] 
                startPay:tn 
  fromScheme:@"UPPay" 
       mode:self.tnMode 
    viewController:self]; 
}
```
上边这个方法需要的几个参数文档上都写的有，tn 是交易流水号，fromScheme 是商户自定义协议， mode 是接入模式，viewController指的是发起调用的视图控制器。

- 检测是否已安装银联App接口调用(这个方法可写可不写)

```
 if([[UPPaymentControl defaultControl] isPaymentAppInstalled])   
  {
      //当判断用户手机上已安装银联App，商户客户端可以做相应个性化处理
}
```
---

到这里的话，银联支付就轻松愉快的搞定了。另外银联的开发文档中给我们提供的有测试帐号，大家可以试试。那里还有支付接口回调、检查是否安装银联App的接口、返回结果接口这三个方法，大家有兴趣的话也可以试试的。集成完银联支付之后，对比下支付宝和微信，觉得还是银联比较有业界良心。