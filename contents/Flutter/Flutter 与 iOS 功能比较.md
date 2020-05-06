# Flutter 与 iOS 功能比较

> 此文档是学习过程中的总结，文章详情：[https://flutterchina.club/flutter-for-ios/#数据库和本地存储](https://flutterchina.club/flutter-for-ios/#数据库和本地存储)。


## Views

#### UIView 相当于 Flutter 中的什么？

Widget 类似于 UIView，两者区别如下：

- 生存时间不同，widgets 一直存在且保持不变，直到当它们需要被改变，当 widgets 和它们的状态被改变时，Flutter 会构建一颗新的 widgets 树； views 在改变时并不会被重新创建。
- Flutter 的 widgets 非常轻量。widgets 本身并不是什么控件，也不会被直接绘制出什么，而只是 UI 的描述。
- iOS 上更新 views，只需要直接改变它们就可以了。在 Flutter 中，widgets 是不可变的，而且不能被直接更新。你需要去操纵 widget 的 state。
- iOS 中可以通过约束或者 frame 来布局，Flutter 中，可以通过编写一个 widget 树来声明布局（可以给任何的 widget 添加 padding），这里是 [Flutter 提供的布局](https://flutter.dev/docs/development/ui/widgets/layout)。
- 在 iOS 中，可以在父 view 中调用 addSubview() 或在子 view 中调用 removeFromSuperview() 来动态地添加或移除子 views。在 Flutter 中，由于 widget 不可变，所以没有和 addSubview() 直接等价的东西。作为替代，可以向 parent 传入一个返回 widget 的函数，并用一个布尔值来控制子 widget 的创建。
- 在 iOS 中，可以通过调用 animate(withDuration:animations:) 方法来给一个 view 创建动画。在 Flutter 中，使用动画库来包裹 widgets，而不是创建一个动画 widget。在 Flutter 中，使用 AnimationController。这是一个可以暂停、寻找、停止、反转动画的 Animation<double> 类型。它需要一个 Ticker 当 vsync 发生时来发送信号，并且在每帧运行时创建一个介于 0 和 1 之间的线性插值（interpolation）。你可以创建一个或多个的 Animation 并附加给一个 controller。

#### 绘图

在 iOS 上，你通过 CoreGraphics 来在屏幕上绘制线条和形状。Flutter 有一套基于 Canvas 类的不同的 API，还有 CustomPaint 和 CustomPainter 这两个类来帮助你绘图。后者实现你在 canvas 上的绘图算法。

#### 怎么创建自定义的 widgets

在 iOS 中，你编写 UIView 的子类，或使用已经存在的 view 来重载并实现方法，以达到特定的功能。在 Flutter 中，你会组合（composing）多个小的 widgets 来构建一个自定义的 widget（而不是扩展它）。

## 导航

#### 页面之间跳转

在 iOS 中，可以使用管理了 view controller 栈的 UINavigationController 来在不同的 view controller 之间跳转。

Flutter 也有类似的实现，使用了 Navigator 和 Routes。一个路由是 App 中“屏幕”或“页面”的抽象，而一个 Navigator 是管理多个路由的 widget 。可以粗略地把一个路由对应到一个 UIViewController。Navigator 的工作原理和 iOS 中 UINavigationController 非常相似，当你想跳转到新页面或者从新页面返回时，它可以 push() 和 pop() 路由。

#### 跳转到其他 App

在 iOS 中，要跳转到其他 App，需要一个特定的 URL Scheme。对系统级别的 App 来说，这个 scheme 取决于 App。为了在 Flutter 中实现这个功能，你可以创建一个原生平台的整合层，或者使用现有的 plugin，例如 [url_launcher](https://pub.dartlang.org/packages/url_launcher)。

## 线程和异步

#### 怎么编写异步的代码

Dart 是单线程执行模型，但是它支持 Isolate（一种让 Dart 代码运行在其他线程的方式）、事件循环和异步编程。除非你自己创建一个 Isolate ，否则你的 Dart 代码永远运行在 UI 线程，并由 event loop 驱动。Flutter 的 event loop 和 iOS 中的 main loop 相似——Looper 是附加在主线程上的。

Dart 的单线程模型并不意味着你写的代码一定是阻塞操作，从而卡住 UI。相反，使用 Dart 语言提供的异步工具，例如 async / await ，来实现异步操作。

#### 把工作放到后台线程

由于 Flutter 是单线程并且跑着一个 event loop 的（就像 Node.js 那样），你不必为线程管理或是开启后台线程而操心。如果你正在做 I/O 操作，如访问磁盘或网络请求，安全地使用 async / await 就完事了。如果，在另外的情况下，你需要做让 CPU 执行繁忙的计算密集型任务，你需要使用 Isolate 来避免阻塞 event loop。

Isolates 是分离的运行线程，并且不和主线程的内存堆共享内存。这意味着你不能访问主线程中的变量，或者使用 setState() 来更新 UI。正如它们的名字一样，Isolates 不能共享内存。

#### 发起网络请求

使用 [http](https://pub.dartlang.org/packages/http) 做网络请求非常简单，类似于 AFNetworking 或 Alamofire。

#### 加载进度条

在 iOS 中，在后台运行耗时任务时你会使用 UIProgressView。在 Flutter 中，使用一个 ProgressIndicator widget。通过一个布尔 flag 来控制是否展示进度。在任务开始时，告诉 Flutter 更新状态，并在结束后隐去。

## 工程结构、本地化、依赖和资源

#### 怎么在 Flutter 中引入 image assets？多分辨率怎么办？

iOS 把 images 和 assets 作为不同的东西，而 Flutter 中只有 assets。Flutter 中的 assets 可以是任意类型的文件，而不仅仅是图片。例如，你可以把 json 文件放置到 my-assets 文件夹中。

对于图片，Flutter 像 iOS 一样，遵循了一个简单的基于像素密度的格式。Image assets 可能是 1.0x 2.0x 3.0x 或是其他的任何倍数。

#### 在哪里放置字符串？怎么做本地化？

不像 iOS 拥有一个 Localizable.strings 文件，Flutter 目前并没有一个用于处理字符串的系统。目前，最佳实践是把你的文本拷贝到静态区，并在这里访问。

默认情况下，Flutter 只支持美式英语字符串。如果你要支持其他语言，请引入 flutter_localizations 包。你可能也要引入 intl 包来支持其他的 i10n 机制，比如日期/时间格式化。

#### Cocoapods 相当于什么？如何添加依赖？

在 iOS 中，你把依赖添加到 Podfile 中。Flutter 使用 Dart 构建系统和 Pub 包管理器来处理依赖。这些工具将本机 Android 和 iOS 包装应用程序的构建委派给相应的构建系统。

如果你的 Flutter 工程中的 iOS 文件夹中拥有 Podfile，请仅在你为每个平台集成时使用它。总体来说，使用 pubspec.yaml 来在 Flutter 中声明外部依赖。一个可以找到优秀 Flutter 包的地方是 [Pub](https://pub.dev/flutter/packages)。

## ViewControllers

#### ViewController 相当于 Flutter 中的什么？

在 iOS 中，一个 ViewController 代表了用户界面的一部分，最常用于一个屏幕，或是其中一部分。它们被组合在一起用于构建复杂的用户界面，并帮助你拆分 App 的 UI。在 Flutter 中，这一任务回落到了 widgets 中。就像在界面导航部分提到的一样，一个屏幕也是被 widgets 来表示的，因为“万物皆 widget！”。使用 Navigator 在 Route 之间跳转，或者渲染相同数据的不同状态。

#### 怎么监听 iOS 中的生命周期事件？

在 iOS 中，你可以重写 ViewController 中的方法来捕获它的视图的生命周期，或者在 AppDelegate 中注册生命周期的回调函数。在 Flutter 中没有这两个概念，但你可以通过 hook WidgetsBinding 观察者来监听生命周期事件，并监听 didChangeAppLifecycleState() 的变化事件。

可观察的生命周期事件有：

- inactive - 应用处于不活跃的状态，并且不会接受用户的输入。这个事件仅工作在 iOS 平台，在 Android 上没有等价的事件。
- paused - 应用暂时对用户不可见，虽然不接受用户输入，但是是在后台运行的。
- resumed - 应用可见，也响应用户的输入。
- suspending - 应用暂时被挂起，在 iOS 上没有这一事件。

更多细节：[AppLifecycleState](https://docs.flutter.io/flutter/dart-ui/AppLifecycleState-class.html)

## 布局

#### UITableView 和 UICollectionView 相当于 Flutter 中的什么？

在 iOS 中，你可能用 UITableView 或 UICollectionView 来展示一个列表。在 Flutter 中，你可以用 ListView 来达到相似的实现。在 iOS 中，你通过代理方法来确定行数，每一个 index path 的单元格，以及单元格的尺寸。由于 Flutter 中 widget 的不可变特性，你需要向 ListView 传递一个 widget 列表，Flutter 会确保滚动是快速且流畅的。

#### 怎么知道列表的哪个元素被点击了？

iOS 中，你通过 tableView:didSelectRowAtIndexPath: 代理方法来实现。在 Flutter 中，使用传递进来的 widget 的 touch handle。

#### 怎么动态地更新 ListView？

一个更新 ListView 的简单方法是，在 setState() 中创建一个新的 list，并把旧 list 的数据拷贝给新的 list。虽然这样很简单，但当数据集很大时，并不推荐这样做。

一个推荐的、高效的且有效的做法是，使用 ListView.Builder 来构建列表。这个方法在你想要构建动态列表，或是列表拥有大量数据时会非常好用。

#### ScrollView 相当于 Flutter 里的什么？

在 iOS 中，你给 view 包裹上 ScrollView 来允许用户在需要时滚动你的内容。在 Flutter 中，最简单的方法是使用 ListView widget。它表现得既和 iOS 中的 ScrollView 一致，也能和 TableView 一致，因为你可以给它的 widget 做垂直排布。

## 手势检测及触摸事件处理

#### 怎么给 Flutter 的 widget 添加一个点击监听者？

在 iOS 中，你给一个 view 添加 GestureRecognizer 来处理点击事件。在 Flutter 中，有两种方法来添加点击监听者：

- 如果 widget 本身支持事件监测，直接传递给它一个函数，并在这个函数里实现响应方法。
- 如果 widget 本身不支持事件监测，则在外面包裹一个 GestureDetector，并给它的 onTap 属性传递一个函数。

#### 怎么处理 widget 上的其他手势？

使用 GestureDetector 你可以监听更广阔范围内的手势，比如：

- Tapping
	- onTapDown — 在特定位置轻触手势接触了屏幕。
	- onTapUp — 在特定位置产生了一个轻触手势，并停止接触屏幕。
	- onTap — 产生了一个轻触手势。
	- onTapCancel — 触发了 onTapDown 但没能触发 tap。
- Double tapping
	- onDoubleTap — 用户在同一个位置快速点击了两下屏幕。
- Long pressing
	- onLongPress — 用户在同一个位置长时间接触屏幕。
- Vertical dragging
	- onVerticalDragStart — 接触了屏幕，并且可能会垂直移动。
	- onVerticalDragUpdate — 接触了屏幕，并继续在垂直方向移动。
	- onVerticalDragEnd — 之前接触了屏幕并垂直移动，并在停止接触屏幕前以某个垂直的速度移动。
- Horizontal dragging
	- onHorizontalDragStart — 接触了屏幕，并且可能会水平移动。
	- onHorizontalDragUpdate — 接触了屏幕，并继续在水平方向移动。
	- onHorizontalDragEnd — 之前接触屏幕并水平移动的触摸点与屏幕分离。

## 主题和文字

#### 怎么给 App 设置主题？

Flutter 实现了一套漂亮的 MD 组件，并且开箱可用。它接管了一大堆你需要的样式和主题。

为了充分发挥你的 App 中 MD 组件的优势，声明一个顶级 widget，MaterialApp，用作你的 App 入口。MaterialApp 是一个便利组件，包含了许多 App 通常需要的 MD 风格组件。它通过一个 WidgetsApp 添加了 MD 功能来实现。

#### 怎么给 Text widget 设置自定义字体？

在 iOS 中，你在项目中引入任意的 ttf 文件，并在 info.plist 中设置引用。在 Flutter 中，在文件夹中放置字体文件，并在 pubspec.yaml 中引用它，然后在你的 Text widget 中指定字体。


## 表单输入

#### Flutter 中表单怎么工作？我怎么拿到用户的输入？

在表单处理的实践中，就像在 Flutter 中任何其他的地方一样，要通过特定的 widgets。如果你有一个 TextField 或是 TextFormField，你可以通过 [TextEditingController](https://docs.flutter.io/flutter/widgets/TextEditingController-class.html) 来获得用户输入。

#### Text field 中的 placeholder 相当于什么？

在 Flutter 中，你可以轻易地通过向 Text widget 的装饰构造器参数重传递 InputDecoration 来展示“小提示”，或是占位符文字。

#### 怎么展示验证错误信息？

就像展示“小提示”一样，向 Text widget 的装饰器构造器参数中传递一个 InputDecoration。然而，你并不想在一开始就显示错误信息。相反，当用户输入了验证信息，更新状态，并传入一个新的 InputDecoration 对象。

## 和硬件、第三方服务以及平台交互

#### 怎么和平台，以及平台的原生代码交互？

Flutter 提供了 [platform channels](https://flutter.io/platform-channels/) ，来和管理你的 Flutter view 的 ViewController 通信和交互数据。平台管道本质上是一个异步通信机制，桥接了 Dart 代码和宿主 ViewController，以及它运行于的 iOS 框架。你可以用平台管道来执行一个原生的函数，或者是从设备的传感器中获取数据。

除了直接使用平台管道之外，你还可以使用一系列预先制作好的 [plugins](https://flutter.io/using-packages/)。例如，你可以直接使用插件来访问相机胶卷或是设备的摄像头，而不必编写你自己的集成层代码。你可以在 [Pub](https://pub.dartlang.org) 上找到插件，这是一个 Dart 和 Flutter 的开源包仓库。其中一些包可能会支持集成 iOS 或 Android，或两者均可。

如果你在 Pub 上找不到符合你需求的插件，你可以[自己编写](https://flutter.io/developing-packages/)，并且发布在 [Pub](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) 上。

#### 怎么访问 GPS 传感器？

通过 [location](https://pub.dartlang.org/packages/location) 社区插件。

#### 怎么访问摄像头？

通过 [image_picker](https://pub.dartlang.org/packages/image_picker) 插件。

#### 怎么登录 Facebook？

登录 Facebook 可以使用 [flutter_facebook_login](https://pub.dartlang.org/packages/flutter_facebook_login) 社区插件。

#### 怎么创建自己的原生集成层？

如果有一些 Flutter 和社区插件遗漏的平台相关的特性，可以根据 [developing packages and plugins](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) 页面构建自己的插件。

## 数据库和本地存储

#### 怎么在 Flutter 中访问 UserDefaults？

在 iOS 中，你可以使用属性列表来存储键值对的集合，即我们熟悉的 UserDefaults。

在 Flutter 中，可以使用 [Shared Preferences plugin](https://pub.dartlang.org/packages/shared_preferences) 来达到相似的功能。它包裹了 UserDefaluts 以及 Android 上等价的 SharedPreferences 的功能。

#### CoreData 相当于 Flutter 中的什么？

在 iOS 中，你通过 CoreData 来存储结构化的数据。这是一个 SQL 数据库的上层封装，让查询和关联模型变得更加简单。

在 Flutter 中，使用 [SQFlite](https://pub.dartlang.org/packages/sqflite) 插件来实现这个功能。

## 通知

#### 怎么推送通知？

在 iOS 中，你需要向苹果开发者平台中注册来允许推送通知。

在 Flutter 中，使用 [firebase_messaging](https://pub.dartlang.org/packages/firebase_messaging) 插件来实现这一功能。