> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 08 如何利用 Clang 为 App 提质？

先贴出 [Clang 源码](https://code.woboq.org/llvm/clang/)。

#### Clang 有哪些优势：

- 对于使用者来说，Clang 编译的速度非常快，对内存的使用率非常低，并且兼容 GCC。
- 对于代码诊断来说，Clang 也非常强大。比如在 Xcode 编译iOS项目的时候，都是使用 LLVM 的，而 Clang 又是 LLVM 的编译前端，具体如下：代码的亮度（Clang）、实时代码检查（Clang）、代码提示（Clang）、debug断点调试（LLDB）。
- Clang 对 typedef 的保留和展开也处理得非常好。typedef 可以缩写很长的类型，保留 typedef 对于粗粒度诊断分析很有帮助。同时，需要了解细节时对 typedef 进行展开即可。
- 在宏处理上，很多宏都是深度嵌套的，Clang 会自动打印实力话信息和嵌套范围信息来帮助你进行宏的诊断和分析。
- Clang 的架构是模块化的。除了代码静态分析外，利用其输出的接口还可以开发用于代码转义、代码生成代码重构的工具。方便与IDE进行继承，这一点刚刚第二条也提到过。


#### Clang 做了哪些事?

- Clang 会对代码进行词法分析，将代码切分成 Token
- 接下来，词法分析完后就会进行语法分析

#### Clang 提供了什么能力?

Clang 为一些需要分析代码语法、语义信息的工具提供了基础设施。这些基础设施就是 LibClang、Clang Plugin 和 LibTooling。

这里是几篇戴铭老师几篇 clang 相关的文章，写的很详细：

- [深入剖析 iOS 编译 Clang LLVM](https://github.com/ming1016/study/wiki/深入剖析-iOS-编译-Clang---LLVM#llvm-工具链)
- [Clang Static Analyzer静态代码分析](https://github.com/ming1016/study/wiki/深入剖析-iOS-编译-Clang---LLVM#clang-static-analyzer静态代码分析)
- [LLVM 工具链](https://github.com/ming1016/study/wiki/深入剖析-iOS-编译-Clang---LLVM#llvm-工具链)