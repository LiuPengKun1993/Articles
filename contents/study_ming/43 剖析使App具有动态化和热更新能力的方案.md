> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 43 剖析使App具有动态化和热更新能力的方案

热更新能力的初衷是，能够及时修复线上问题，减少 Bug 对用户的伤害。而动态化的目的，除了修复线上问 题外，还要能够灵活更新 App 版本。

要实现动态化，就需要具备在运行时动态执行程序的能力。同时，实现了动态化，也就具备了热更新能力。 通常情况下，实现动态化的方案有三种，分别是 JavaScriptCore 解释器方案、代码转译方案、自建解释器方案。

#### JavaScriptCore 解释器方案

iOS 系统内置的 JavaScriptCore，是能够在 App 运行过程中解释执行脚本的解释器。

JavaScriptCore 提供了易用的原生语言接口，配合 iOS 运行时提供的方法替换能力，出现了使用 JavaScript 语言修复线上问题的 [JSPatch](https://github.com/bang590/JSPatch)，以及把 JavaScriptCore 作为前端和原生桥梁的 [React Native](https://github.com/facebook/react-native) 和 [Weex](https://github.com/apache/incubator-weex) 开发框架。这些库，让 App 具有了动态化能力。

但是，对于原生开发者来说，只能解释执行 JavaScript 语言的解释器 JSPatch、React Native 等，我们用起 来不是很顺手，还是更喜欢用原生语言来开发。那么，有没有办法能够解决语言栈的问题呢?


#### 代码转译方案

DynamicCocoa 方案将 Objective-C 转换成 JavaScript 代码，然后下发动态执行。这样一来，原生开发者只要使用原生语言去开发调试即可，避免了使用 JavaScript 开发不畅的问题，也就解决了语言栈的问题。

当然，语言之间的转译过程需要解决语言差异的问题，比如 Objective-C 是强类型，而 JavaScript 是弱类型，这两种语言间的差异点就很多。但，好在 JavaScriptCore 解释执行完后，还会对应到原生代码上，所以我们只要做好各种情况的规则匹配，就可以解决这个问题。

手段上，语言转译可以使用现有的成熟工具，比如类 C 语言的转译，可以使用LLVM 套件中 Clang 提供的 LibTooling，通过重载 HandleTranslationUnit() 函数，使用 RecursiveASTVistor 来遍历 AST，获取代码的完整信息，然后转换成新语言的代码。

在这里，无法穷尽两种编程语言间的转译，但是如果你想要快速了解转译过程的话，最好的方法就是看一 个实现的雏形。

比如，我以前用 Swift 写过一个 Lisp 语言到 C 语言转译的雏形。你可以点击这个链接，查看具体的代码。通过这个代码，你能够了解到完成转译依次需要用到词法分析器、语法分析器、遍历器、转换器和代码生成器。它们的实现分别对应 LispToC 里的 JTokenizer.swif、JParser.swift、JTraverser.swift、JTransformer.swift 和 CodeGenerator.swift。

再比如，你可以查看 SwiftRewrite 项目的完整转译实现。SwiftRewriter 使用 Swift 开发，可以完成 Objective-C 到 Swift 的转换。

#### 自建解释器方案

可以发现，前面提到的 JSPatch、React Native 等库，到最后能够具有动态性，用的都是系统内置的 JavaScriptCore 来解释执行 JavaScript 语言。

虽然直接使用内置的 JavaScriptCore 非常方便，但却限制了对性能的优化。比如，系统限制了第三方 App 对 JavaScriptCore JIT(即时编译)的使用。再比如，由于 JavaScript 使用的是弱类型，而类型推断只能在 LLInt 这一层进行，无法得到足够的优化。

再加上 JSContext 多线程的处理也没有原生多线程处理得高效、频繁的 JavaScriptCore 和原生间的切换、内存管理方式不一致带来的风险、线程管理不一致的风险、消息转发时的解析转换效率低下等等原因，使得 JavaScriptCore 作为解释器的方案，始终无法比拟原生。

虽然通过引入前端技术栈和利用转译技术能够满足大部分动态化和热修复的需求，但一些对性能要求高的团队，还是会考虑使用性能更好的解释器。


#### 小结

动态化方案的选择主要由团队人员自身情况决定，比如原生开发者居多时可以选择代码转译或自建解释器方案;前端开发者居多或者原生开发者有意转向前端开发时，可以选择 JavaScriptCore 方案。

另外，动态化方案本身，对大团队的意义会更加明显。因为大团队一般会根据业务分成若干小团队，由这些不同团队组成的超级大 App 每次发版，都会相互掣肘，而动态化就能够解决不同团队灵活发版的问题，让各个小团队按照自己的节奏来迭代业务。