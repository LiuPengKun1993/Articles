# CALayer 及 CALayer 的 contents 系列属性

> 对性能比较挑剔的 APP, 我们一般会用轻量级的控件代替重量级的控件, 比如在代码中, 对于一些没有交互事件的 UIView 或者 UIImageView, 我们经常会用 CALayer 来替代两者.

关于 UIView 和 CALayer 的区别, 这里不做过多的介绍, 只简单说一点: **当我们在视图里添加 UIView 时, 这个视图的 sublayers 及 subviews 的数量都会有所增加; 而当我们在视图里添加 CALayer 时, 这个视图的 sublayers 数量会增加, subviews 的数量则不会改变.**

### CALayer 替代 UIView
用 CALayer 替代 UIView 很简单, 在视图里添加 CALayer 的方法与在视图里添加 UIView 的方法类似, 只需以下几行代码即可:

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view.layer addSublayer:self.redLayer];
}

- (CALayer *)redLayer {
    if (!_redLayer) {
        _redLayer = [[CALayer alloc] init];
        _redLayer.backgroundColor = [UIColor redColor].CGColor;
        _redLayer.frame = CGRectMake(self.view.center.x - 100, self.view.center.y - 100, 200, 200);
    }
    return _redLayer;
}
```

这样我们就在视图上添加了一个红色的 CALayer 了. 上边的代码有一点还需要说下: **与 UIView 不同, CALayer 的 backgroundColor 属性是 CGColorRef 类型, CGColorRef 指向 CGColor, 另外 CGColorRef 不属于 Cocoa 对象, 它是 Core Foundation 类型; 而在 UIColor 里有一个 CGColor 属性, 它返回一个 CGColorRef, 因此我们可以把这个属性直接赋给 CALayer 的 backgroundColor, 即 `_redLayer.backgroundColor = [UIColor redColor].CGColor`.**


### contents 属性

CALayer 有一个 contents 属性, 这个属性是 id 类型的. 

![](https://upload-images.jianshu.io/upload_images/2665449-aeef3a30893980ab.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


理论上来讲, 既然是 id 类型, 那么应该可以接收任何类型的对象, 但实际上并不是如此, 实际上是, 如果我们给 contents 赋的不是 CGImage, 那么图层将会是空白的(我这里试了给 contents 赋 UILabel, 编译并运行成功, 但是并未显示), 我们先用 contents 属性添加图片(CGImage), 代码如下:

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view.layer addSublayer:self.redLayer];
    [self.redLayer addSublayer:self.imageLayer];
}

- (CALayer *)redLayer {
    if (!_redLayer) {
        _redLayer = [[CALayer alloc] init];
        _redLayer.backgroundColor = [UIColor redColor].CGColor;
        _redLayer.frame = CGRectMake(self.view.center.x - 100, self.view.center.y - 100, 200, 200);
    }
    return _redLayer;
}

- (CALayer *)imageLayer {
    if (!_imageLayer) {
        _imageLayer = [[CALayer alloc] init];
        _imageLayer.frame = CGRectMake(50, 40, 100, 120);
        UIImage *tempImage = [UIImage imageNamed:@"header"];
        _imageLayer.contents = CFBridgingRelease(tempImage.CGImage);
    }
    return _imageLayer;
}
```

与上面说过的 CGColorRef 相似, **我们在这句代码里`_imageLayer.contents = CFBridgingRelease(tempImage.CGImage)`, 也要用 UIImage 的 CGImage 赋值, 因为 CGImageRef 是一个指向 CGImage 结构的指针. 另外关于 CFBridgingRelease(), 这里还需要说一下, 由于 ARC 只支持管理 OC 对象, 不支持 Core Foundation 类型, 为了避免内存泄漏以及 double free 引起的程序崩溃, 这里我们用 CFBridgingRelease() 将非 OC 对象转换为 OC 对象, 给予 ARC 管理权(实际上写`tempImage.CGImage`的话, Xcode 会直接报错, 我们还可以这样写`_imageLayer.contents = (__bridge id _Nullable)(tempImage.CGImage)`).**


这是运行效果图:

![](https://upload-images.jianshu.io/upload_images/2665449-ffc4b9bf98edeb75.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**这里需要说明, 我们并没有直接用 UIImageView 控件展示图片, 而是通过 CALayer 相关的一些代码, 让图片显示在视图上.**

### contentsGravity 属性

不过图片本来是正方形的, 这里明显被拉伸了, 当 UIImageView 被拉伸时, 我们会设置它的 contentMode 属性, 而当 CALayer 展示的图片被拉伸时, 我们可以使用它的 contentsGravity 属性, contentsGravity 是 NSString 类型的, 它有以下一些常量值:

- kCAGravityCenter : 将内容图像放置在图层的中心
- kCAGravityTop : 将内容图像沿图层的上边缘水平放置
- kCAGravityBottom : 将内容图像定位在图层的底部边缘
- kCAGravityLeft : 将内容图像垂直地置于图层的左边缘
- kCAGravityRight : 将内容图像垂直放置在图层的右边缘
- kCAGravityTopLeft : 将内容图像放置在图图层的左上角
- kCAGravityTopRight : 将内容图像放置在图层的右上角
- kCAGravityBottomLeft : 将内容图像放置在图层的左下角
- kCAGravityBottomRight : 将内容图像放置在图层的右下角
- kCAGravityResize : 调整内容图像的大小以完全填充层边界, 可能会忽略内容的自然方面, 这是默认的
- kCAGravityResizeAspect : 调整内容图像的大小, 使其在层边界内尽可能地显示大, 但仍然保持其自然的外观
- kCAGravityResizeAspectFill : 调整内容图像的大小, 使其显示为填充层边界, 同时保持其自然的外观, 这可能导致内容扩展到层边界之外

我们使用 kCAGravityResizeAspect, 它的效果等同于枚举 contentMode 里面的 UIViewContentModeScaleAspectFit, 给工程加上这行代码`_imageLayer.contentsGravity = kCAGravityResizeAspect`, 为了看的更清晰, 我们另外再加一行代码, 给 _imageLayer 添加一下背景色, `_imageLayer.backgroundColor = [UIColor blueColor].CGColor`, 运行看看效果:

![](https://upload-images.jianshu.io/upload_images/2665449-8b6b74b7df781b7a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样图片就居中显示了,  contentsGravity 的其它属性也可以一一尝试.

### contentsRect 属性

contentsRect 属性也是一个比较好玩的属性, contentsRect 属性默认的大小是 {0, 0, 1, 1}, 我们可以根据需求, 给它设置不同的大小进行图片裁剪或多张图片拼合, 我们在工程里写下这行代码`_imageLayer.contentsRect = CGRectMake(0, 0, 0.5, 0.5)`, 图片就被裁剪了, 只显示左上方的四分之一部分, 如下图: 
![](https://upload-images.jianshu.io/upload_images/2665449-a2164be0da191a77.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们还可以进行多张图片合成:
![](https://upload-images.jianshu.io/upload_images/2665449-7283417be0655296.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### contentsCenter 属性

contentsCenter 也是一个 CGRect, 但它和图片显示的位置无关, contentsCenter 默认情况下也是 {0, 0, 1, 1}, 它定义了图层的可拉伸的区域, 比如我们设置 contentsCenter 为 {0.2, 0.2, 0.6, 0.6}, 那么它的效果图大概如下:
![区域 1 垂直水平拉伸; 区域 2 只能水平拉伸; 区域 3 只能垂直拉伸; 区域 4 不拉伸.](https://upload-images.jianshu.io/upload_images/2665449-f4802abb0b24720e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

简单的演示效果图如下: 
![](https://upload-images.jianshu.io/upload_images/2665449-001695e34f0f7864.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


CALayer 也可以注册代理, CALayerDelegate, 我们也可以在它的代理方法里做一些操作, 我这里简单试用了一下: [一个小 demo: NNLayerTest](https://github.com/liuzhongning/NNLearn) . 

---

- 参考文章: 
  - [CALayer的contents属性的应用](https://blog.csdn.net/u013915422/article/details/51531325)
  - [CALayer的几个属性](https://www.jianshu.com/p/d951cdd0f7c0)
  - [关于CALayer的Content属性 ](https://blog.csdn.net/Lee_lisa520/article/details/47721159)