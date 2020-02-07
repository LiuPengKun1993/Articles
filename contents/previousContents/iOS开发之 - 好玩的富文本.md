# iOS开发之 - 好玩的富文本

周末闲着没事，就想着不如把那些容易遗忘的知识点整理一下，一来可以让有需要的朋友少走弯路，二来自己以后再忘记的时候也可以回头看看......但 iOS 中小冷易忘的知识点实在太多了，不知道该从哪里开始整理，“百无聊赖”逛了下淘宝，发现里面好多都是富文本，就想着为什么不从富文本开始呢？反正闲着也是闲着......于是就有了下面这些东西，希望可以帮到看到的盆友们！

- 先写一个小引子
>在项目中，很多时候我们都需要把文字设置成倾斜、加粗、加下划线、加删除线、加阴影等等状态，就比如下面这张图：
![不是打广告哦](http://upload-images.jianshu.io/upload_images/2665449-efaea7b3b4c215d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但如果我们只是简单的设置字体
```
self.textView.text = @"很好玩的富文本";
```
显示出来大概是这个样子滴：
![普通文本](http://upload-images.jianshu.io/upload_images/2665449-ab7fd6348e681a88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了让文字变的好看一些，接下来我们尝试一下富文本！

- 改变字体颜色

```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    
    [string addAttribute:NSForegroundColorAttributeName value:[UIColor blueColor] range:NSMakeRange(1, 2)];
    [string addAttribute:NSForegroundColorAttributeName value:[UIColor redColor] range:NSMakeRange(3, 1)];
    [string addAttribute:NSForegroundColorAttributeName value:[UIColor blueColor] range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![改变字体颜色](http://upload-images.jianshu.io/upload_images/2665449-9ab05decdf730650.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 改变字体大小

```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:20] range:NSMakeRange(1, 2)];
    [string addAttribute:NSFontAttributeName value:[UIFont systemFontOfSize:30] range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![改变字体大小](http://upload-images.jianshu.io/upload_images/2665449-c85ab447f29b3333.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 给文字添加背景色
```
 NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSBackgroundColorAttributeName value:[UIColor cyanColor] range:NSMakeRange(1, 2)];
    [string addAttribute:NSBackgroundColorAttributeName value:[UIColor yellowColor] range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![给文字添加背景色](http://upload-images.jianshu.io/upload_images/2665449-5b3beb42cba98ef1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 给文字设置间距
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSKernAttributeName value:@(20) range:NSMakeRange(0, 6)];
    self.textView.attributedText = string;
```
![给文字设置间距](http://upload-images.jianshu.io/upload_images/2665449-faf5af7f1000d40e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 字体倾斜
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSObliquenessAttributeName value:@(0.5) range:NSMakeRange(0, 7)];
    self.textView.attributedText = string;
```
![字体倾斜](http://upload-images.jianshu.io/upload_images/2665449-84144fae563a02fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 字体加粗
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSExpansionAttributeName value:@(0.5) range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![字体加粗](http://upload-images.jianshu.io/upload_images/2665449-1a4c2b74f132931e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 加下划线并设置下划线颜色
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSUnderlineStyleAttributeName value:@(1) range:NSMakeRange(4, 3)];
    [string addAttribute:NSUnderlineColorAttributeName value:[UIColor blueColor] range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![加下划线并设置下划线颜色](http://upload-images.jianshu.io/upload_images/2665449-54fdb333b29af0a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 加删除线并设置删除线颜色
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"原价666💰，现价只要66💰！"];
    [string addAttribute:NSStrikethroughStyleAttributeName value:@(1) range:NSMakeRange(2, 3)];
    [string addAttribute:NSStrikethroughColorAttributeName value:[UIColor blueColor] range:NSMakeRange(2, 3)];
    self.textView.attributedText = string;
```
![加删除线并设置删除线颜色](http://upload-images.jianshu.io/upload_images/2665449-7660641928dfc070.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 设置文字方向
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSVerticalGlyphFormAttributeName value:@(1) range:NSMakeRange(0, 7)];
    [string addAttribute:NSWritingDirectionAttributeName value:@[@(2),@(3)] range:NSMakeRange(0, 7)];
    self.textView.attributedText = string;
```
![设置文字方向](http://upload-images.jianshu.io/upload_images/2665449-3e892743a9f112f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 给文字添加阴影
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    NSShadow *shadow = [[NSShadow alloc]init];
    shadow.shadowOffset = CGSizeMake(5, 5);
    shadow.shadowColor = [UIColor blueColor];
    shadow.shadowBlurRadius = 1;
    [string addAttribute:NSShadowAttributeName value:shadow range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![给文字添加阴影](http://upload-images.jianshu.io/upload_images/2665449-8f882d994febb18b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 图文混排

```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    // 创建图片图片附件
    NSTextAttachment *attach = [[NSTextAttachment alloc] init];
    attach.image = [UIImage imageNamed:@"1.jpg"];
    attach.bounds = CGRectMake(0, 0, 20, 20);
    NSAttributedString *attachString = [NSAttributedString attributedStringWithAttachment:attach];
    [string appendAttributedString:attachString];
    self.textView.attributedText = string;
```
![图文混排](http://upload-images.jianshu.io/upload_images/2665449-27adc1a27e0637ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 文字偏移
```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:@"很好玩的富文本"];
    [string addAttribute:NSBaselineOffsetAttributeName value:@(10) range:NSMakeRange(0, 1)];
    [string addAttribute:NSBaselineOffsetAttributeName value:@(20) range:NSMakeRange(1, 1)];
    [string addAttribute:NSBaselineOffsetAttributeName value:@(30) range:NSMakeRange(2, 1)];
    [string addAttribute:NSBaselineOffsetAttributeName value:@(40) range:NSMakeRange(3, 1)];
    [string addAttribute:NSBaselineOffsetAttributeName value:@(50) range:NSMakeRange(4, 3)];
    self.textView.attributedText = string;
```
![文字偏移](http://upload-images.jianshu.io/upload_images/2665449-4e0d6a0908bfb1e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上是对 iOS 中富文本相关的知识做的一些整理，如果有粗心写错的地方，还请朋友们指出！如果还有遗漏的地方，以后碰到了再来补充。。。