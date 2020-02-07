# iOS开发之 - WKWebView VS UIWebView

>WKWebView是苹果在 iOS 8 中推出的新框架，相比UIWebView来说，WKWebView 性能好，速度快，内存小，功能多，且支持了更多的 HTML5 特性......因此本小白禁不住“诱惑”利用了一晚上的时间，整理了一些关于 WKWebView 的资料，与大家分享下！在说 WKWebView 之前，我们先说说他的老大哥 UIWebView ！！！

UIWebView 是苹果在 iOS 2 之后推出的，由于资格比较“老”，大家对 UIWebView 应该也不会陌生，这里直接上代码：

### 1.UIWebView 基本使用
```
    // 创建webview
    UIWebView *webView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 20, NNWidth, NNHeight - 20)];
    // 网络请求
    NSURLRequest *request =[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
    // 加载网页
    [webView loadRequest:request];
    // 添加到界面
    [self.view addSubview:webView];
```
### 2.UIWebView 常用方法
```
    [webView reload]; // 刷新
    [webView stopLoading]; // 停止加载
    [webView goBack]; // 后退
    [webView goForward]; // 前进
    [webView canGoBack]; // 是否可以后退
    [webView canGoForward]; // 是否可以向前
    [webView isLoading]; // 是否正在加载
```
### 3.UIWebView 相关代理协议
```
#pragma mark - UIWebViewDelegate
	/// 是否允许加载网页
	- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
	    NSLog(@"允许加载网页");
	    return YES;
	}
	
	/// 开始加载网页时调用
	- (void)webViewDidStartLoad:(UIWebView *)webView {
	    NSLog(@"开始加载网页");
	}
	/// 网页加载完成时调用
	- (void)webViewDidFinishLoad:(UIWebView *)webView {
	    NSLog(@"网页加载完成");
	}
	
	/// 网页加载错误时调用
	- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error {
	    NSLog(@"网页加载错误时调用");
	}
```
 ### 这里是 UIWebView 的效果图！！！


![UIWebView加载网页](http://upload-images.jianshu.io/upload_images/2665449-ab410b93f7c4d753.gif?imageMogr2/auto-orient/strip)
![代理方法输出的顺序](http://upload-images.jianshu.io/upload_images/2665449-14ead660659b5a2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 因为主要想与大家分享的是 WKWebView，所以对 UIWebView 的介绍就到此为止了。接下来的 WKWebView 才是今晚的主角！！！    
  

***
**依然是先介绍 WKWebView 的基本使用**

### 1. WKWebView 基本使用
>这里加一个不用提示的提示：WKWebView 和 UIWebview 的基本使用方法相类似，但是需要导入头文件 #import <WebKit/WebKit.h>，其它步骤如下

```
// 创建webview
    WKWebView *webView = [[WKWebView alloc] initWithFrame:CGRectMake(0, 20, NNWidth, NNHeight - 20)];
    // 创建请求
    NSURLRequest *request =[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
    // 加载网页
    [webView loadRequest:request];
    // 将webView添加到界面
    [self.view addSubview:webView];
```
ps:这里就不再补充效果图了，除了速度内存优于 UIWebView，显示的效果差不多。。。
### 2. WKWebView 加载文件 

```
    // 创建webview
    WKWebView *webView = [[WKWebView alloc] initWithFrame:CGRectMake(0, 20, NNWidth, NNHeight - 20)];
    // 创建url(可以随便从桌面拉张图片)
    NSURL *url = [NSURL fileURLWithPath:@"/Users/ios/Desktop/图片/90416596477f347431a929dc73b9f404.jpg"];
    // 加载文件
    [webView loadFileURL:url allowingReadAccessToURL:url];
    // 最后将webView添加到界面
    [self.view addSubview:webView];
```
#### 这里是加载文件效果图！
![加载本地图片](http://upload-images.jianshu.io/upload_images/2665449-026af846596935f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3. WKWebView 与 JS 代码交互

```
// 图片缩放的 JS 代码
    NSString *JSImage = @"var count = document.images.length;for (var i = 0; i < count; i++) {var image = document.images[i];image.style.width=375;};window.alert('找到' + count + '张图');";
    // 初始化 WKUserScript
    WKUserScript *script = [[WKUserScript alloc] initWithSource:JSImage injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];
    // 初始化WKWebViewConfiguration
    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    [configuration.userContentController addUserScript:script];
    _webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:configuration];
// 这里需要从网上找到一张图片的地址
    [_webView loadHTMLString:@"<head></head>![](http://upload-images.jianshu.io/upload_images/2665449-f109bc70619e2a92.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)"baseURL:nil];
    [self.view addSubview:_webView];
```

![与 JS 交互效果图](http://upload-images.jianshu.io/upload_images/2665449-311830759271b58f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. WKWebView 常用的代理协议

*WKWebView 的代理方法还是蛮有趣的，大家可以试下哈！*
#### 4.1 WKNavigationDelegate 协议
这里简单罗列了几个常用的方法。以下四个与 UIWebView 中的方法相同：

```
#pragma mark - WKNavigationDelegate
// 页面开始加载时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation {

}
// 内容开始返回时调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation {

}
// 页面加载完成时调用
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {

}
// 页面加载失败时调用
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation {

}
```
新增的三个代理方法:

```

// 这个方法是服务器重定向时调用，即 接收到服务器跳转请求之后调用
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation {

}
// 在收到响应后，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler {

}
// 在发送请求之前，决定是否跳转
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {

}
```

#### 4.2 WKUIDelegate 协议

```
#pragma mark - WKUIDelegate
/// 创建一个新的WebView
- (WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures {
    return nil;
}

/**
 *  web界面中有弹出警告框时调用
 *
 *  @param webView           实现该代理的webview
 *  @param message           警告框中的内容
 *  @param frame             主窗口
 *  @param completionHandler 警告框消失调用
 */
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(void (^)())completionHandler {

}

/// 输入框
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * __nullable result))completionHandler {

}
/// 确认框
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler {

}
/// 警告框
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {

}
```

### 5.另外搜集了一些 WKWebView 常用的属性：

```
	•    WKBackForwardListItem: webview中后退列表里的某一个网页。
	•	WKFrameInfo: 包含一个网页的布局信息。
	•	WKNavigation: 包含一个网页的加载进度信息。
	•	WKPreferences: 概括一个 webview 的偏好设置。
	•	WKProcessPool: 表示一个 web 内容加载池。
	•	WKScriptMessage: 包含网页发出的信息。
	•	WKUserScript:表示可以被网页接受的用户脚本。
	•	WKWebViewConfiguration: 初始化 webview 的设置。
	•	WKWindowFeatures: 指定加载新网页时的窗口属性。
	•	WKScriptMessageHandler: 提供从网页中收消息的回调方法。
```
小尾巴:正像开头说的那样， WKWebView 相比UIWebView来说性能好，速度快，内存小，功能多......但现在开发中普遍用的还是 UIWebView，这是因为大多数App需要支持iOS7及以上的版本。但苹果之所以推出 WKWebView 就是为了弥补 UIWebView 的种种不足，逐渐取代 UIWebView，相信不久的将来，网页会成为 WKWebView 的天下！

*今天就简单的写到这里，以后如果用到其它的了，再和大家分享😄!*
