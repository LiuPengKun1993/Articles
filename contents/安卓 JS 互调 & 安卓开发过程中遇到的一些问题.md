# 安卓 JS 互调 & 安卓开发过程中遇到的一些问题

> 在 APP 开发过程中，为了实现页面的灵活性与开发的高效性，原生与 H5 互调一个很常用的手段。这里简单介绍下安卓端的原生 JS 互调：


### 一、JS 调安卓

#### 安卓 WebView 初始化：

```
webView = new WebView(this);
WebSettings settings = webView.getSettings();
// 设置支持 javaScript 脚步语言
settings.setJavaScriptEnabled(true);
// 支持双击-前提是页面要支持才显示
settings.setUseWideViewPort(true);
// 支持缩放按钮-前提是页面要支持才显示
settings.setBuiltInZoomControls(true);
// 设置客户端-不跳转到默认浏览器中
webView.setWebViewClient(new WebViewClient());
// 加载网页地址
webView.loadUrl("");

```

#### 安卓注册 JavascriptInterface

```
/**
 * Js 中调用 Android 原生代码核心代码:
 * 该方法传入两个参数: 1.被调用的类; 2.JS 中被调用类的实例对象名字
 */
webView.addJavascriptInterface(new JsCallAndroid(), "Android");
```
注意：对象名必须和 H5 页面中 onclick 的名字对应。

#### 安卓端被调用 JsCallAndroid

```
private class JsCallAndroid {
/**
 * JS 中调用的方法
 */
@JavascriptInterface
public void hello() {
    Toast.makeText(JsCallAndroid.this, "JS 调用原生", Toast.LENGTH_SHORT).show();
}
}
```

#### H5 代码示例

```
<!--调用 Android 原生代码:onclick = "window.+Android 传过来的对象名.+对象中的方法"-->
<input type="button" value="JS 调用 Android" onclick="window.Android.hello()"/>
```

### 二、原生调 JS


#### 安卓代码示例

```
/**
 * 通过这种形式：javascript:Js文件中的方法名(参数) 
 * 下面的 AndroidCallJs();必须是JS代码中方法
 * 如果参数是变量,就使用拼接字符串方式传参
 */
webView.loadUrl("javascript:window.AndroidCallJs" + "("+hello+")");
```

#### H5 代码示例

```
<script type="text/javascript">
    function AndroidCallJs(arg){
         console.log("原生调 JS");
    }
</script>
```

<br>

> 小结：安卓 H5 互调，大体上和 iOS H5 互调类似。这里只抽出了安卓 H5 互调的简单代码，有疑问的童鞋欢迎讨论，或者你也可以参考下这篇 [博客](https://blog.csdn.net/carson_ho/article/details/64904691)，或许可以帮助到你。
> 
> 接着记一下安卓开发过程中遇到的几个小问题，之前开发安卓的时候也遇到过很多问题，但都没有记录，很快便忘记了，以至于现在每次用到的时候还得再查，因此这里简单记录一下...

---

### 安卓开发过程中遇到的问题

- 因为之前安装的模拟器比较老，在与 JS 交互时出现了问题，于是需要更新，但是在更新模拟器时，一直报 waiting for target deviceto come online 的问题，解决办法：

```
这个问题是由虚拟机引起的，所以要到 AVD manager 解决
第一步：先关掉模拟器；
第二步：打开 AVD manager,找到新的模拟器，里面有一个选项 Cold Boot Now
点击一下，模拟器打开并提示一行信息，直接 dismiss 掉就行了；
第三步：重新运行 app 就可以了。
```

- 在写方法时报错：“错误: 未报告的异常错误Exception; 必须对其进行捕获或声明以便抛出”，在方法后加 throws Exception。

```
public void getUserInfo(String msg) throws Exception {
    
}

```

- 遇到“错误: 从内部类中访问本地变量onSuccess; 需要被声明为最终类型”，在属性前加 final 关键字。

- 遇到“错误: 未报告的异常错误JSONException; 必须对其进行捕获或声明以便抛出”

```
try {
    // String 转 JSONObject
    JSONObject result = new JSONObject(str);
    // 取数据
    result.get("command_type");
                
} catch (JSONException e) {
    e.printStackTrace();
}
```