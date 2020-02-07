# iOS开发之 - CADisplayLink 实现酷炫动画


> 偶然发现了一个好玩的类， CADisplayLink，出于好奇所以就尝试了一下，用 CADisplayLink 做了个类似云飘的效果。由于对 CADisplayLink 的认识还比较浅，如果哪里写的不正确，还请各位大大能够指出来！看效果图先，~~就是有点丑~~，😂

![ning.gif](http://upload-images.jianshu.io/upload_images/2665449-da509b22ab7071c0.gif?imageMogr2/auto-orient/strip)

- 核心代码如下：

#### 一、创建 CADisplayLink，添加事件，绑定 Runloop。

```
// 创建 CADisplayLink
- (CADisplayLink *)displayLink {
    if (!_displayLink) {
        _displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(makeCloud)];

        // 当把 CADisplayLink 对象添加到 Runloop 中后，selector就能被周期性调用
        [self.displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
    }
    return _displayLink;
}

// 周期调用的方法
- (void)makeCloud {
    self.cloudOffsetX += self.cloudSpeed;
    
    [self cloudLayerName:self.firstCloudLayer];
    [self cloudLayerName:self.secondCloudLayer];
    [self cloudLayerName:self.thirdCloudLayer];
    [self cloudLayerName:self.fourthCloudLayer];
    [self cloudLayerName:self.fifthCloudLayer];
}
```
#### 二、配置参数，加载图层

```
    self.cloudWidth = self.frame.size.width;            // 云彩宽度
    self.cloudColor = RGBA(255, 255, 255, 0.3);         // 云彩颜色
    self.cloudSpeed = 0.05 / M_PI;                      // 云彩飘动的速度
    self.cloudPointY = 100;                             // 云彩Y坐标
    self.cloudOffsetX = 0;                              // 云彩位移X
    self.cloudAmplitude = 30;                           // 振幅大小
    self.cloudCycle =  1.03 * M_PI / self.cloudWidth;   // 周期大小

    // 添加图层
    [self.layer addSublayer:self.firstCloudLayer];
    [self.layer addSublayer:self.secondCloudLayer];
    [self.layer addSublayer:self.thirdCloudLayer];
    [self.layer addSublayer:self.fourthCloudLayer];
    [self.layer addSublayer:self.fifthCloudLayer];

```

#### 三、创建 CAShapeLayer 动画

```
// 五个图层动画
- (void)cloudLayerName:(CAShapeLayer *)cloudLayerName {
    // 创建一个Path句柄
    CGMutablePathRef path = CGPathCreateMutable();
    CGFloat y = self.cloudPointY;
   // 初始化该path到一个初始点
    CGPathMoveToPoint(path, nil, 0, y);
    for (float x = 0.0f; x <= self.cloudWidth; x++) {
        if (cloudLayerName == self.firstCloudLayer) {
          // 云彩的 Y 值
            y = self.cloudAmplitude * sin(self.cloudCycle * x + self.cloudOffsetX - 10) + self.cloudPointY + 10;
        } else if (cloudLayerName == self.secondCloudLayer) {
            y = (self.cloudAmplitude + 15) * sin(self.cloudCycle * x + self.cloudOffsetX ) + self.cloudPointY ;
        } else if (cloudLayerName == self.thirdCloudLayer) {
            y = (self.cloudAmplitude + 30)* sin(self.cloudCycle * x + self.cloudOffsetX + 20) + self.cloudPointY + 10;
        } else if (cloudLayerName == self.fourthCloudLayer) {
            y = (self.cloudAmplitude + 20)* sin(self.cloudCycle * x + self.cloudOffsetX - 20) + self.cloudPointY - 10;
        } else if (cloudLayerName == self.fifthCloudLayer) {
            y = (self.cloudAmplitude + 10)* sin(self.cloudCycle * x + self.cloudOffsetX - 10) + self.cloudPointY + 2;
        }
        // 添加一条直线
        CGPathAddLineToPoint(path, nil, x, y);
    }

    // 添加一条直线
    CGPathAddLineToPoint(path, nil, self.cloudWidth, self.frame.size.height);
    // 添加一条直线
    CGPathAddLineToPoint(path, nil, 0, self.frame.size.height);
    // 关闭该path
    CGPathCloseSubpath(path);
    cloudLayerName.path = path;
    // 释放该path
    CGPathRelease(path);
}
```


- 推荐几篇和 CADisplayLink 相关的深度好文！ 

  - [官方文档](https://developer.apple.com/reference/quartzcore/cadisplaylink)
  -  [iOS 核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/chapter11/frame-timing.html) 
  - [CADisplayLink 结合 UIBezierPath的神奇妙用](http://kittenyang.com/cadisplaylinkanduibezierpath/)
