# 浅谈 iOS 性能优化

#### iOS 中的 CPU 和 GPU

在屏幕成像过程中，CPU 和 GPU 起着很重要的作用。

- CPU（Central Processing Unit，中央处理器），负责对象的创建和销毁，对象属性的调整，布局计算，文本的计算和排版，图片的格式转换和解码，图像的绘制。
- GPU（Graphics Processing Unit，图形处理器），负责纹理的渲染。GPU 是一个专门为图形高并发计算而量身定做的处理单元，比 CPU 使用更少的电来完成工作并且 GPU 的浮点计算能力要超出 CPU 很多。GPU 的渲染性能要比 CPU 高效很多，同时对系统的负载和消耗也更低一些，所以在开发中，我们应该尽量让 **CPU 负责主线程的 UI 调动，把图形显示相关的工作交给 GPU 来处理**。


![](https://github.com/liuzhongning/Articles/blob/master/resources/Performance-Optimization-01.jpg)

#### iOS 缓冲机制

iOS 使用的是双缓冲机制。即 GPU 会预先渲染好一帧放入一个缓冲区内（前帧缓存），让视频控制器读取，当下一帧渲染好后，GPU 会直接把视频控制器的指针指向第二个缓冲器（后帧缓存）。当你视频控制器已经读完一帧，准备读下一帧的时候，GPU 会等待显示器的 VSync 信号发出后，前帧缓存和后帧缓存会瞬间切换，后帧缓存会变成新的前帧缓存，同时旧的前帧缓存会变成新的后帧缓存。


#### 屏幕成像原理

时钟信号：垂直同步信号V-Sync / 水平同步信号H-Sync。

屏幕成像就是垂直同步信号和水平同步信号一直发送。发出垂直同步信号（VSync）时，即将显示一页的数据。水平同步信号（HSync）发出时，就一行一行的显示。

![](https://github.com/liuzhongning/Articles/blob/master/resources/Performance-Optimization-02.jpg)

按照60FPS的刷帧率，每隔16ms就会有一次VSync信号。出现卡顿的原因就是发送VSync信号间隔时间过大。

#### 卡顿的原因

##### 屏幕内容是怎么显示到屏幕上的？

![](https://github.com/liuzhongning/Articles/blob/master/resources/Performance-Optimization-03.jpg)

在 VSync 信号到来后，系统图形服务会通过 CADisplayLink 等机制通知 App，App 主线程开始在 CPU 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 CPU 会将计算好的内容提交到 GPU 去，由 GPU 进行变换、合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待下一次 VSync 信号到来时显示到屏幕上。由于垂直同步的机制，如果在一个 VSync 时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

> 上面一段内容出自 [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)，你可以在上述文章中对卡顿的原因做更深入的了解。

在开发中，CPU 和 GPU 中任何一个压力过大，都会导致掉帧现象，所以在开发时，需要分别对 CPU 和 GPU 压力进行评估和优化。


#### 卡顿优化 - CPU

- 尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用CALayer取代UIView
- 不要频繁地调用UIView的相关属性，比如frame、bounds、transform等属性，尽量减少不必要的修改
- 尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
- Autolayout会比直接设置frame消耗更多的CPU资源
- 图片的size最好刚好跟UIImageView的size保持一致
- 控制一下线程的最大并发数量
- 尽量把耗时的操作放到子线程：文本处理（尺寸计算、绘制），图片处理（解码、绘制）

#### 卡顿优化 - GPU

- 尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示
- GPU能处理的最大纹理尺寸是4096x4096，一旦超过这个尺寸，就会占用CPU资源进行处理，所以纹理尽量不要超过这个尺寸
- 尽量减少视图数量和层次
- 减少透明的视图（alpha<1），不透明的就设置opaque为YES
- 尽量避免出现离屏渲染


#### 离屏渲染

##### 在OpenGL中，GPU有2种渲染方式

- On-Screen Rendering：当前屏幕渲染，在当前用于显示的屏幕缓冲区进行渲染操作
- Off-Screen Rendering：离屏渲染，在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

##### 离屏渲染消耗性能的原因

- 需要创建新的缓冲区
- 上下文切换，离屏渲染的整个过程，需要多次切换上下文环境（CPU渲染和GPU切换），先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的

##### 哪些操作会触发离屏渲染？

- 光栅化，layer.shouldRasterize = YES
- 遮罩，layer.mask
- 圆角，同时设置layer.masksToBounds = YES、layer.cornerRadius大于0
	- 考虑通过CoreGraphics绘制裁剪圆角，或者叫美工提供圆角图片
- 阴影，layer.shadowXXX
	- 如果设置了layer.shadowPath就不会产生离屏渲染

#### 卡顿检测

卡顿主要是因为在主线程执行了比较耗时的操作。我们可以使用 CADisplayLink 来监视 CPU 的卡顿问题，这是一个 [FPS 指示器](https://github.com/ibireme/YYText/blob/master/Demo/YYTextDemo/YYFPSLabel.m)。


##### 参考文档：

- [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
- [使用 ASDK 性能调优 - 提升 iOS 界面的渲染性能](https://github.com/draveness/analyze/blob/master/contents/AsyncDisplayKit/提升%20iOS%20界面的渲染性能%20.md#使用-asdk-性能调优---提升-ios-界面的渲染性能)