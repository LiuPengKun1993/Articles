# MVC 下的网络层架构


![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVC-01.jpg)


此系列文章是《App架构》的读书笔记及心得，主要详细讲解了以下五种最为主要的 app 设计模式：

- Model-View-Controller (MVC)
- Model-View-ViewModel+Coordinator (MVVM-C)
- Model-View-Controller+ViewState (MVC+VS)
- ModelAdapter-ViewBinder (MAVB)
- Elm 架构 (The Elm Architecture, TEA)

### 前言

> 没有哪个单独的模式能在所有情景下都能做到最好。我们在开发中需要具体情况具体分析，最好的模式就是对你来说最为清晰的模式。


在 MVC 模式下，我们会用两种方式为我们的 app 添加网络支持。第一种方式是 controller 拥有网络 —— 这会将 model 层移除，并让 view controller 来处理网络请求。第二种方式是 model 拥有网络，它保留了 MVC 版本的 model 层，并在它的下方加入了一个网络层。这篇文章主要介绍网络层由 controller 层拥有和由 model 层拥有的区别。

在 controller 拥有网络的情况下，数据直接从网络获取，而非从一个本地存储中取得。从网络获得的数据不会被持久化，而是由 view controller 负责将它们以 view state 的方式缓存在内存中。这样一来，app 就只能在有网络连接的情况下工作了。

对比通过 model 共享数据的方式来说，controller 拥有网络的方式让不同 view controller 之间进行数据共享变得困难，因为它们之间很大程度上是彼此独立的。现在，新的数据必须要主动地传递给其他依赖它的 view controller，而在 model 拥有网络的情况下，这些 view controller 可以使用观察 model 中变更的方式来获取新数据。

### Controller 持有网络

在 controller 持有网络的实现中，view controller 负责进行网络请求。它们同时也拥有通过这些请求所加载的数据。反过来，这也意味着 app 就没有 model 层了：每个 view controller 管理着它自己的数据。

让 controller 拥有网络，是为一个 app 添加网络支持的很容易的“临时”方案，它能快速给我们结果。虽然我们一般还是建议尽量小心不要使用这种方式，不过在 controller 中写网络确实有一些正确的使用场景，而且它在很多代码库中都很流行。几乎所有的开发者都或多或少写过这样的代码。

Controller 持有网络时，我们 app 中所有数据都是暂态的，也就是说，当 app 退出时它们不会被保存下来。保存数据的任务被完全外包给了服务器。所以，对数据的更改现在是一个通过网络执行的异步任务。

数据更改流程图：
![](https://github.com/liuzhongning/Articles/blob/master/resources/App-Network-01.jpg)

总体上不推荐使用 controller 拥有网络的方式。不过，为了理解在哪些特定情况下 controller 拥有网络的方式可行，在哪些情况下它不可行，需要再深入讨论一下这种方式中的权衡。结论：如果通过 view controller 加载的任何网络数据需要被 app 中其他部分所共享的话，我们需要重新引入 model 层。如果这些数据只是在本地使用的话，那么，原则上来说，我们可以使用 controller 拥有网络的方式，不需要 model 层也能构建 app。


### Model 持有网络
View controller 中所添加的唯一的职责是开始加载数据 (比如下拉刷新或者当导航至一个新的画面时)。然而，和上面 controller 拥有网络的方式不同，view controller 不负责处理网络返回的数据。这些数据被插入到 store 中，和处理本地变更的方式一样，view controller 通过监听 model 变更通知来接收这些变更。

这种方式的一大优势是，在 app 中分享数据和进行通讯会非常简单。所有的 view controller 从 store 中获取它们的数据，并且订阅 model 的变更通知。就算是 app 中多个部分独立地显示或者变更相同的数据，这些数据也会具有一贯性。

我们使用 Model 持有网络，可以使用 store 做离线缓存（理论上，controller 拥有网络同样可以做到离线工作，只是要正确实现会困难得多），这让 app 在没有网络连接的时候也能工作。在客户端，我们可以立即更新数据，而不用先去请求服务器。由于有这些改进，我们需要在客户端增加额外的代码，来追踪未提交的变更，并且在把变更提交给服务器时，对可能发生的冲突进行处理。

### 其他架构下的网络

上文介绍的两种网络方式不只适用于 MVC，对于像是 MVVM 这样的相关的模式依然适用。区别在于，controller 拥有的网络在 MVVM 中将会变成 view-model 拥有的网络。不过，它们的实现是非常相似的。

然而，在像是 MVC+VS，MAVB，或者 TEA 这样的模式中，controller 拥有网络和 model 拥有网络的差别就不适用了。举个例子，MVC+VS 的前提是，将 view state 明确地作为 model 的一部分来进行表示。由于 controller 拥有网络方式中，数据是 view state 的一部分，所以它将会被立即推入到 model 层中。这样一来，在这个上下文中，区分 controller 拥有网络还是 model 拥有网络，就变得没有意义了。

### 总结

从架构的角度来看，上述两种方法 (controller 拥有网络和 model 拥有网络) 之间的主要区别在于数据的所有权。在 controller 拥有网络的方式中，view controller 在本地持有数据，而在 model 拥有网络的方式中，model 层中的一些实体持有数据。

View controller 中的网络代码通常会被认为是反模式的，不过，原则上来说，如果网络过来的数据只作为本地 view state 使用，而不需要与其他部件进行通讯时，在 controller 里写网络代码并不是什么问题。如果我们可以把大块的网络代码正确地提取到辅助函数中去，这种方式甚至不会使我们的 view controller 臃肿很多。

真正关键的问题是，这些数据会需要被共享吗？一旦数据需要被共享，model 拥有网络就编程了天然的选择：由于数据是由 controller 层以外的实体所拥有，它将能够很容易地在整个 app 中进行共享，而不会遭遇数据不一致的问题。
