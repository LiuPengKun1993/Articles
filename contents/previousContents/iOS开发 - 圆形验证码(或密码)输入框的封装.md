# iOS开发 - 圆形验证码(或密码)输入框的封装

> 项目中用到了圆形验证码输入框，输入框之间要求有一定的距离，UI图如下：


![UI图](http://upload-images.jianshu.io/upload_images/2665449-7b41dd80189fc084.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 上面的UI图主要有以下几个要求：
  - 输入框为圆形
  - 输入框之间有适当距离
  - 输入框颜色在输入文本时有变化


刚开始想着用固定的几个 UITextField 实现，但转念一想，用 UITextField 实现有点麻烦（输入框多的话，它们之间的响应事件不太容易控制，需要来回变换），于是开始想其它办法，最后用了以下的思路：

- 创建一个 UITextField，用  UILabel 显示 UITextField 上输入的数字，需要监听文本的输入，同时对 UILabel 进行一些操作，再用 block 将输入的文本传出来。


#####封装的圆形输入框主要实现了以下功能：输入框的数量、距离、颜色、大小等都可以自行设定，用起来也很方便，只需以下几行代码即可：

```
    NNValidationCodeView *view = [[NNValidationCodeView alloc] initWithFrame:CGRectMake(80, 100, 300, 45) andLabelCount:4 andLabelDistance:10];
    [self.view addSubview:view];
    view.changedColor = [UIColor yellowColor];
    view.codeBlock = ^(NSString *codeString) {
        NSLog(@"验证码:%@", codeString);
    };
```
这是 demo 的效果图：

![效果图1](http://upload-images.jianshu.io/upload_images/2665449-a1ab2114dbc942f9.gif?imageMogr2/auto-orient/strip)


![效果图2](http://upload-images.jianshu.io/upload_images/2665449-7d2566ca5c78135b.gif?imageMogr2/auto-orient/strip)

由于代码很容易看懂，另外代码中也写了注释，因此这里不再对项目做过多的陈述，这是 [demo地址](https://github.com/liuzhongning/NNValidationCodeView)，需要的话可以拿去借鉴，**有什么不足之处希望能留下宝贵意见或建议！**