> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 07 Clang、Infer 和 OCLint ，我们应该使用谁来做静态分析？

OCLint、Infer、Clang 是三款主流的静态分析器，可以帮助我们在编写代码的阶段发现潜在的问题，比如空指针访问、资源和内存泄露等等。

戴铭老师比较推荐的是 Infer。

Infer 是 Facebook 开源的、使用 OCaml 语言编写的静态分析工具，可以对 C、Java 和 Objective-C 代码进行静态分析，可以检查出空指针访问、资源泄露以及内存泄露。

关于使用方法，可以直接看官方文档，写的很清晰，这里是官方文档 [https://infer.liaohuqiu.net/docs/getting-started.html](https://infer.liaohuqiu.net/docs/getting-started.html)。

这里是 Infer 的 GitHub 地址：[https://github.com/facebook/infer](https://github.com/facebook/infer)


也可参考一下这篇博客：[infer使用的浅谈简析](https://www.jianshu.com/p/4667e36aadea)