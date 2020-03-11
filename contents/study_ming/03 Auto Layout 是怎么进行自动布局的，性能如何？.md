> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记，课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)。

### 03 Auto Layout 是怎么进行自动布局的，性能如何？


#### Auto Layout 的来历

Cassoway 算法能够有效解析线性等式系统和线性不等式系统，用来表示用户界面中那些相等关系和不等关系，由此苹果公司将 Cassoway 算法运用到了 Auto Layout 中。

#### Auto Layout 的生命周期

拥有一整套的 Layout Engine 来统一管理布局的创建、更新和销毁，Layout Engine 是 Auto Layout 的核心，主导着整个界面布局。Layout Engine 在遇到 Constraints Change 时，会重新计算布局。

#### Auto Layout 的性能问题

- iOS12 之前，视图嵌套的数量对性能的影响是呈指数级增长的，而 iOS12 优化之后对性能的影响是线性的，与手写布局相差不大。

#### 使用 Auto Layout 时注意的问题

- 多使用 Compression Resistance Priority 和 Hugging Priority，利用优先级的设置，让布局更灵活，代码更少，更易于维护。


戴铭老师在这一节课上提到了 UIStackView，这里是我之前练习 UIStackView 时的一个 demo：[NNStackView](https://github.com/liuzhongning/NNLearn/tree/master/005.%20NNStackView)


#### WWDC 视频：
- [High Performance Auto Layout - WWDC 2018 220](https://developer.apple.com/videos/play/wwdc2018/220)
- [What’s New in Cocoa Touch - WWDC 2018 202](https://developer.apple.com/videos/play/wwdc2018/202)