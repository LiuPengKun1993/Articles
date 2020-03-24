> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 21 除了Cocoa，iOS还可以用哪些GUI框架开发？

现在流行的 GUI 框架除了 Cocoa Touch 外，还有 WebKit、Flutter、Texture(原名 AsyncDisplayKit)、 Blink、Android GUI 等。其中，WebKit、Flutter、Texture 可以用于 iOS 开发。

#### WebKit

关于 WebKit 可以参考戴铭老师的博客：[深入剖析 WebKit](https://ming1016.github.io/2017/10/11/deeply-analyse-webkit/)。

#### Flutter
Flutter 是 Google公司于2017年推出的一个移动应用开发的 GUI 框架，使用 Dart 语言编写程序，一套代码 可以同时运行在iOS和Android平台。

#### Texture

Texture 框架的基本单元，是基于 UIView 抽象的节点 ASDisplayNode。和 UIView 不同的是 ， ASDisplayNode 是线程安全的，可以在后台线程上并行实例化和配置整个层级结构。Texture框架的开发语 言，使用的是苹果公司自家的 Objective-C 和 Swift。

GUI框架对比图：
![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_GUI.jpg)

#### 渲染流程

GUI 框架中的渲染，一般都会经过布局、渲染、合成这三个阶段。 

WebKit 用 CSS 来布局，CSS 会提供 Frame 布局和 FlexBox 布局；Flutter 也支持 Frame 布局和 FlexBox 布局；Cocoa Touch 框架本身不支持 FlexBox 布局，但是通过 Facebook 的 [Yoga 库](https://yogalayout.com) 也能够使用 FlexBox 布局（这里有一篇博客介绍 iOS  FlexBox 布局：[iOS 上的 FlexBox 布局](https://juejin.im/post/5a33a6926fb9a045104a8d3c)）。

渲染阶段的主要工作，是利用图形函数计算出界面的内容。一般情况下，对于 2D 平面的渲染都是使用 CPU 计算，对 3D 空间的渲染会使用 GPU 计算。Cocoa Touch 和 Texture 框架使用的是 Core Animation，3D 使用的是 Metal 引擎。Flutter 使用的是 Skia， 3D 使用的是 OpenGL(ES)。WebKit 将渲染接口抽象了出来，实现层 根据平台进行区分，比如在 iOS 上就用 CoreGraphics 来渲染，在 Android 就用 Skia 渲染。

#### Texture 里 Node 的异步绘制

Texture 开发了线程安全的 ASDisplayNode，可以进行异步绘制，极大的提高 APP 性能，而且还可以与 UIView 共生，这样在开发时我们只需要将性能要求高的界面使用 Texture 即可。

关于 Texture 最核心的线程安全节点 ASDisplayNode，戴铭老师也在课程里介绍了一些原理，但我觉得，要想真正搞懂 Texture 的原理，只看这些是不行的。最好看官方文档：

- [Texture 官方文档](http://texturegroup.org/docs/installation.html)
- [Texture Github](https://github.com/texturegroup/texture)

或者这里也有一些讲的很好的博客：

- [iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
- [使用 ASDK 性能调优 - 提升 iOS 界面的渲染性能](https://draveness.me/asdk-rendering)
- [从 Auto Layout 的布局算法谈性能](https://draveness.me/layout-performance)
- [预加载与智能预加载（iOS）](https://draveness.me/preload)
- [AsyncDisplayKit 2.0 Tutorial: Getting Started](https://www.raywenderlich.com/818-asyncdisplaykit-2-0-tutorial-getting-started)
- [Texture 布局篇](https://bawn.github.io/2017/12/Texture-Layout/)
