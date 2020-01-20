# Model-View-Controller (MVC)


![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVC-01.jpg)


此系列文章是《App架构》的读书笔记及心得，主要详细讲解了以下五种最为主要的 app 设计模式：

- Model-View-Controller (MVC)
- Model-View-ViewModel+Coordinator (MVVM-C)
- Model-View-Controller+ViewState (MVC+VS)
- ModelAdapter-ViewBinder (MAVB)
- Elm 架构 (The Elm Architecture, TEA)

### 前言

> 没有哪个单独的模式能在所有情景下都能做到最好。我们在开发中需要具体情况具体分析，最好的模式就是对你来说最为清晰的模式。


### MVC 概述

![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVC-02.jpg)


MVC 是 App 开发中所有设计模式的基础，Apple 在所有的实例项目中都使用了这种模式，加上 Cocoa 本身就是针对这种模式设计的，所以 MVC 成为了 iOS、macOS、tvOS 和 watchOS 上官方认证的 app 架构模式。

MVC 的核心思想是，controller 层负责将 model 层和 view 层撮合到一起工作。Controller 对另外两层进行构建和配置，并对 model 对象和 view 对象之间的双向通讯进行协调。所以，在一个 MVC app 中，controller 层是作为核心来参与形成 app 的反馈回路的。


### 三者的职责

![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVC-03.jpg)


- Model： 负责封装数据、存储和处理数据运算等工作
- View： 负责数据展示、监听用户触摸等工作
- Controller： 负责业务逻辑、事件响应、数据加工等工作

我们编写 iOS 程序，其实就是编写这三个部分，同时管理这三个部分之间的交互和通信。一组一组的 MVC 进一步组合起来就形成了复杂的应用程序。


### MVC 优缺点

MVC 有其优点，它是 iOS 开发中阻力最低的架构模式。Cocoa 中的每个类都是在 MVC 的条件下进行测试的。像 storyboard 这样的特性，是与框架和类高度集成的，它可以和使用 MVC 的程序更顺滑地一同工作。在网络上搜索时，相比于其他任何一种设计模式，可以找到更多按照 MVC 进行实现的例子。而且，在所有模式中，MVC 通常都是代码量最少，设计开销最小的模式。

#### Controller 臃肿问题

在开发中，MVC 的最大的问题，就是 Controller 臃肿问题，什么样的内容才应该放到 Controller 中？


#### 较差的可测试性
关于可测试性，虽然接口测试和数据驱动的回归测试已经有四十年甚至更长的历史了，而现代的单元测试仅仅是从上世纪 90 年代才真正起步，不过单元测试只花了十多年的时间就成为了 app 中最常见的测试方式。不过即使是现在，许多 app 除了人工测试以外并没有进行常规的测试。

所以已经有超过 20 年历史的 Cocoa MVC 模式在被创建的时候并没有考虑单元测试这件事情就一点都不起怪了。

### Controller 臃肿问题如何解决

臃肿的 controller 通常进行了它们的主要工作 (观察 model，展示 view，为它们提供数据，以及接收 view action) 之外的无关工作；它们要么应该被打散成多个各自管理一个较小部分的 view 层级的 controller；要么就是因为接口和抽象没能将一段程序的复杂度封装起来， view controller 做了太多打扫垃圾的工作。

要解决这个问题，首先应该严格遵守 MVC 的设计模式，比如一个界面上有很多控件，我们要避免在 Controller 中把这些控件一个一个的往 self.view 上用 addSubView 的方法存放，而应该交给其子类 View 统一管理，然后再与 Controller 做关联，另外要主动地将尽可能多的功能移动到 model 层中。比如排序、数据的获取和处理等方法，因为不是 app 的存储状态的一部分，所以通常会被放到 controller 中。但是它们依然与 app 的数据和专用逻辑相关，把它们放在 model 中会是更好的选择。


### 总结

MVC 是最常用的一种模式，它有很多优点，但也有其弊端。我们所需要做的，是解决其 Controller 臃肿的问题，通过抽取代码，将原本 MVC 设计模式中的 ViewController 进一步拆分，构造出 网络请求层、ViewModel 层、Service 层等其它类，来配合 Controller 工作，从而使 Controller 更加简单，从而使我们的 App 更容易维护。

---


这里是我之前写的一个 MVC 相关的 demo：[瘦身后的 MVC](https://github.com/liuzhongning/NNLearn/tree/master/009.MVCDemo)