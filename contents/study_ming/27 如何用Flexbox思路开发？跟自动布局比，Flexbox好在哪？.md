> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 27 如何用Flexbox思路开发？跟自动布局比，Flexbox好在哪？

#### Flexbox 好在哪

与自动布局(Auto Layout)思路类似，Flexbox 使用的也是描述性的语言来布局。使用 Flexbox 布局的视图元素叫 Flex 容器 (flex container)，其子视图元素叫作 Flex 项目(flex item)。Flexbox 布局的主要思想是，通过 Flex 容器设定的属性来改变内部 Flex 项目的宽、高，并调整 flex 项目的位置来填充 flex 容器的可用空间。

关于Flexbox 的详细入门资料，你可以参看阮一峰老师的“[Flex 布局教程:语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)”一文。而Flexbox 在 W3C 上完整的定义，你可以点击[这个链接](https://www.w3.org/TR/css-flexbox-1/)查看。

#### Texture 如何使用 Flexbox 思路进行布局

基于 Flexbox 的布局思路，Texture 框架的布局方案考虑得十分长远，也已经十分成熟，虽然学习起来需要费些力气，但是性能远好于苹果的自动布局，而且写起来更简单。

Texture 框架的布局中，Texture 考虑到布局扩展性，提供了一个基类 ASLayoutSpec。这个基类提供了布局的基本能力，使 Texture 可以通过它扩展实现多种布局思路，比如 Wrapper、Inset、Overlay、Ratio、 Relative、Absolute 等布局思路，也可以继承 ASLayoutSpec 来自定义你的布局算法。

ASLayoutSpec的子类，及其具体的功能如下:

```
ASAbsoluteLayoutSpec // 绝对布局 
ASBackgroundLayoutSpec // 背景布局 
ASInsetLayoutSpec // 边距布局 
ASOverlayLayoutSpec // 覆盖布局 
ASRatioLayoutSpec // 比例布局 
ASRelativeLayoutSpec // 顶点布局 
ASCenterLayoutSpec // 居中布局 
ASStackLayoutSpec // 盒子布局 
ASWrapperLayoutSpec // 填充布局 
ASCornerLayoutSpec // ⻆标布局
```

ASLayoutSpec 子类实现了各种布局思路，ASLayoutSpec 会制定各种布局相通的协议方法，遵循这些协议后可以保证这些子类能够使用相同的规则去实现更丰富的布局。

通过 ASLayoutSpec 遵循的 ASLayoutElement 协议，可以知道 ASLayoutSpec 提供的基本能力有哪些。 ASLayoutElement 协议定义如下:

```
@protocol ASLayoutElement <ASLayoutElementExtensibility, ASTraitEnvironment, ASLayoutElementAsciiArtProtocol>
#pragma mark - Getter
@property (nonatomic, assign, readonly) ASLayoutElementType layoutElementType;
@property (nonatomic, strong, readonly) ASLayoutElementStyle *style;
- (nullable NSArray<id<ASLayoutElement>> *)sublayoutElements;
#pragma mark - Calculate layout
/**
 * 要求节点根据给定的大小范围返回布局
 */
- (ASLayout *)layoutThatFits:(ASSizeRange)constrainedSize;
/**
 * 在子 layoutElement 上调用它来计算它们在calculateLayoutThatFits: 方法里面实现的布局
 */
- (ASLayout *)layoutThatFits:(ASSizeRange)constrainedSize parentSize:(CGSize)parentSize;
/**
 * 重写此方法以计算 layoutElement 的 布局.
 */
- (ASLayout *)calculateLayoutThatFits:(ASSizeRange)constrainedSize;
/**
 * 重写此方法允许你接收 layoutElement 的大小。使用这些值可以计算最终的约束大小，但这个方法要尽量少用。
 */
- (ASLayout *)calculateLayoutThatFits:(ASSizeRange)constrainedSize
                     restrictedToSize:(ASLayoutElementSize)size
                 relativeToParentSize:(CGSize)parentSize;
- (BOOL)implementsLayoutMethod;
@end
```

通过上面代码可以看出，协议定义了 layoutThatFits 和 calculateLayoutThatFits 等回调方法。其中，layoutThatFits 回调方法用来要求节点根据给定的大小范围返回布局，重写 calculateLayoutThatFits 方法用 以计算 layoutElement 的布局。定义了统一的协议方法，能让 ASLayoutSpec 统一透出布局计算能力，统一 规范的协议方法，也有利于布局算法的扩展。

除了 Texture 用到 Flexbox 的布局思想以外，ReactNative 和 Weex 也用到了 Flexbox 布局思路，这两个框架对 Flexbox 布局思想的实现，通过一个叫 [Yoga](https://github.com/facebook/yoga) 的C++库。

除了 React Native、Weex 外，Yoga 还为很多其他的开源框架提供支持，比如 Litho、ComponentKit 等。

Yoga 布局库是对 Texture 布局思想的实现，是有 C/C++ 语言编写的，依赖少、编译后的二进制文件也小，基于此，Yoga 可以用于多平台，可以很方便地集成到 Android 和 iOS 上。

除了 Flexbox 思路的布局 ASStackLayoutSpec 以外，Texture 中还有 wrapper、inset、overlay、ratio、relative、absolute 等针对不同场景的布局思路，同时还支持自定义布局算法。

#### Flexbox 算法

Flexbox 算法的主要思想是：让 flex 容器能够改变其 flex 项目的宽高和顺序，以填充可用空间，flex 容器可以通过扩大 flex 项目来填充可用空间，或者缩小 flex 项目来使其不超出可用空间。

---

此篇课程只记录到这里，如果想要更多更深入的了解 Flexbox，还需查阅更多的资料，这里推荐几篇不错的博客：

- [A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [A Visual Guide to CSS3 Flexbox Properties](https://scotch.io/tutorials/a-visual-guide-to-css3-flexbox-properties)
