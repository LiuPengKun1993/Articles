> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 40 ReactNative、Flutter等，这些跨端方案怎么选?

目前， 主流的跨端方案，主要分为两种:一种是，将 JavaScriptCore 引擎当作虚拟机的方案，代表框架是 React Native;另一种是，使用非 JavaScriptCore 虚拟机的方案，代表框架是 Flutter。

使用跨端方案进行开发，必然会替代原有平台的开发技术，所以我们在选择跨端方案时，不能只依赖于某几 项指标，比如编程语言、性能、技术架构等，来判断是否适合自己团队和产品，更多的还要考虑开发效率、 社区支持、构建发布、 DevOps、 CI 支持等工程化方面的指标。

所以说，我们在做出选择时，既要着眼于团队现状和所选方案生态，还要考虑技术未来的发展走向。

#### React Native框架的优势

React Native 使用 JavaScript 语言来开发，Flutter 使用的是 [Dart 语言](https://dart.dev/guides/language/language-tour)。这两门编程语言，对 iOS 开发者来说都有一定的再学习成本，而使用何种编程语言，其实决定了团队未来的技术栈。

JavaScript 的历史和流行程度都远超 Dart ，生态也更加完善，开发者也远多于 Dart 程序员。所以，从编程 语言的角度来看，虽然 Dart 语言入门简单，但从长远考虑，还是选择React Native 会更好一些。

同时，从页面框架和自动化工具的角度来看，React Native也要领先于 Flutter。这，主要得益于 Web 技术这么多年的积累，其工具链非常完善。前端开发者能够很轻松地掌握 React Native，并进行移动端 App 的开发。

#### Flutter框架的优势

除了编程语言、页面框架和自动化工具以外，React Native 的表现就处处不如 Flutter 了。总体来说，相比于React Native框架，Flutter的优势最主要体现在性能、开发效率和体验这两大方面。

Flutter的优势，首先在于其性能。

React Native 所使用的 JavaScriptCore，原本用在浏览器中，用于解释执行网页中的JavaScript代码。为了兼容 Web 标准留下的历史包袱，无法专门针对移动端进行性能优化。

Flutter 却不一样。它一开始就抛弃了历史包袱，使用全新的 Dart 语言编写，同时支持 AOT 和 JIT两种编译 方式，而没有采用HTML/CSS/JavaScript 组合方式开发，在执行效率上明显高于 JavaScriptCore 。

除了编程语言的虚拟机，Flutter的优势还体现在UI框架的实现上。它重写了UI 框架，从 UI 控件到渲染，全部重新实现了，依赖 Skia 图形库和系统图形绘制相关的接口，保证了不同平台上能有相同的体验。

想要了解 Flutter 的布局和渲染，你可以看看这两个视频“[The Mahogany Staircase - Flutter’s Layered Design](https://www.youtube.com/watch?v=dkyY9WCGMi0)”和“[Flutter’s Rendering Pipeline](https://www.youtube.com/watch?v=UUfXWzp0-DU&t=1955s)”。

除了性能上的优势外，Flutter在开发效率和体验上也有很大的建树。

凭借热重载(Hot Reload)这种极速调试技术，极大地提升了开发效率，因此Flutter 吸引了大量开发者的眼球。

同时，Flutter 因为重新实现了UI框架，可以不依赖 iOS 和 Android 平台的原生控件，所以无需专门去处理平台差异，在开发体验上实现了真正的统一。

此外，Flutter 的学习资源也非常丰富。Flutter的[官方文档](https://flutter.dev/docs)，分门别类整理得仅仅有条。YouTube 上有一个专门的[频道](https://www.youtube.com/flutterdev)，提供了许多讲座、演讲、教程资源。

或许，你还会说 Flutter 包大小是个问题。Flutter 的渲染引擎是自研的，并没有用到系统的渲染，所以 App 包必然会大些。但是，我觉得从长远来看，App Store 对包大小的限制只会越来越小，所以说这个问题一定不会成为卡点。
