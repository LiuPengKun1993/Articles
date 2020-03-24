> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 22 细说iOS响应式框架变迁，哪些思想可以为我所用？

主要谈了 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)，ReactiveCocoa 是将函数式编程和响应式编程结合起来的库，通过函数式编程思想建立了数据流的通道，数据流动时会经过各种函数的处理最终到达和数据绑定的界面，由此实现了数据变化响应界面变化的效果。

但 ReactiveCocoa 在 iOS 开发中相对使用的不多，原因是响应式编程没能提高 App 的性能，并且在调试上，由于 ReactiveCocoa 框架采用了 Monad 模式，导致其底层实现过于复杂，从而在方法调用堆栈里很难去定位到问题。

#### Monad

ReactiveCocoa 是采用号称纯函数式编程语言里的 Monad 设计模式搭建起来的，核心类是 RACStream。我 们使用最多的 RACSignal(信号类，建立数据流通道的基本单元) ，就是继承自RACStream。RACStream 的定义如下:

```

typedef RACStream * (^RACStreamBindBlock)(id value, BOOL *stop);
  /// An abstract class representing any stream of values.
  ///
  /// This class represents a monad, upon which many stream-based operations can
  /// be built.
  ///
  /// When subclassing RACStream, only the methods in the main @interface body need
  /// to be overridden.
  @interface RACStream : NSObject
  + (instancetype)empty;
  + (instancetype)return:(id)value;
  - (instancetype)bind:(RACStreamBindBlock (^)(void))block;
  - (instancetype)concat:(RACStream *)stream;
  - (instancetype)zipWith:(RACStream *)stream;
 @end
 
```

通过定义的注释可以看出，RACStream 的作者也很明确地写出了RACStream 类表示的是一个 Monad，所以我们在 RACStream 上可以构建许多基于数据流的操作；RACStreamBindBlock，就是用来处理 RACStream 接收到数据的函数。

---

我们之前有项目是使用 MVVM + RAC 模式开发的，有很多缺陷，比如代码侵入性太强、难于调试、RAC 方面的开发不好招等等，当然也有好处，比如 KVO、代理、block 这些都可以通过 RAC 来统一解决、会用的话使用起来还是很方便的。这里是一些 RAC 相关的博客：

- [函数式语言的宗教](http://www.yinwang.org/blog-cn/2013/03/31/purely-functional)
- [ReactiveCocoa 讨论会](http://blog.devtang.com/2016/01/02/reactive-cocoa-discussion/)
- [Functor、Applicative 和 Monad](http://blog.leichunfeng.com/blog/2015/11/08/functor-applicative-and-monad/)
- [ReactiveCocoa v2.5 源码解析之架构总览](http://blog.leichunfeng.com/blog/2015/12/25/reactivecocoa-v2-dot-5-yuan-ma-jie-xi-zhi-jia-gou-zong-lan/)
- [MVVM With ReactiveCocoa](http://blog.leichunfeng.com/blog/2016/02/27/mvvm-with-reactivecocoa/)
- [MVVMReactiveCocoa](https://github.com/leichunfeng/MVVMReactiveCocoa)
- [ReactiveCocoa中潜在的内存泄漏及解决方案](https://tech.meituan.com/2016/08/19/potential-memory-leak-in-reactivecocoa.html)
- [RACSignal的Subscription深入分析](https://tech.meituan.com/2015/06/30/rac-signal-subscription.html)
- [ReactiveCocoa核心元素与信号流](https://tech.meituan.com/2016/10/14/reactive-cocoa-signal-flow.html)
- [细说ReactiveCocoa的冷信号与热信号（一）](https://tech.meituan.com/2015/09/08/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-1.html)
- [细说ReactiveCocoa的冷信号与热信号（二）：为什么要区分冷热信号](https://tech.meituan.com/2015/09/28/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-2.html)
- [细说ReactiveCocoa的冷信号与热信号（三）：怎么处理冷信号与热信号](https://tech.meituan.com/2015/11/03/talk-about-reactivecocoas-cold-signal-and-hot-signal-part-3.html)
- [Reactive Cocoa Tutorial [1] = 神奇的Macros](http://blog.sunnyxx.com/2014/03/06/rac_1_macros/)
- [Draven 的 ReactiveObjC 系列](https://github.com/Draveness/analyze)
- [ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2](https://www.raywenderlich.com/2493-reactivecocoa-tutorial-the-definitive-introduction-part-1-2)
- [ReactiveCocoa Tutorial – The Definitive Introduction: Part 2/2](https://www.raywenderlich.com/2490-reactivecocoa-tutorial-the-definitive-introduction-part-2-2)
- [MVVM with Combine Tutorial for iOS](https://www.raywenderlich.com/4161005-mvvm-with-combine-tutorial-for-ios)
- [Haskell](http://learnyouahaskell.com)