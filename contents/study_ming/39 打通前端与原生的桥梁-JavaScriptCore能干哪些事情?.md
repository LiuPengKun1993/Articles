> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 39 打通前端与原生的桥梁-JavaScriptCore能干哪些事情?


JavaScriptCore 为原生编程语言 Objective-C、Swift 提供调用 JavaScript 程序的动态能力，还为 JavaScript 提供原生能力来弥补前端所缺能力。

#### JavaScriptCore 框架

这是苹果官方对JavaScriptCore框架的说明：[javascriptcore](https://developer.apple.com/documentation/javascriptcore)。从结构上看，JavaScriptCore 框架主要 由 JSVirtualMachine 、JSContext、JSValue类组成。

JSVirturalMachine的作用，是为 JavaScript 代码的运行提供一个虚拟机环境。在同一时间内， JSVirtualMachine只能执行一个线程。如果想要多个线程执行任务，你可以创建多个 JSVirtualMachine。每个 JSVirtualMachine 都有自己的 GC(Garbage Collector，垃圾回收器)，以便进行内存管理，所以多个 JSVirtualMachine 之间的对象无法传递。

JSContext 是 JavaScript 运行环境的上下文，负责原生和 JavaScript 的数据传递。

JSValue 是 JavaScript 的值对象，用来记录 JavaScript 的原始值，并提供进行原生值对象转换的接口方法。

三者关系：JSVirtualMachine 里包含了多个 JSContext， 同一个JSContext 中又可以有多个 JSValue。


JSVirtualMachine 、JSContext、JSValue 类提供的接口，能够让原生应用执行 JavaScript 代码，访问 JavaScript 变量，访问和执行 JavaScript 函数;也能够让 JavaScript 执行原生代码，使用原生输出的类。

那么，解释执行 JavaScript 代码的 JavaScriptCore 和原生应用是怎么交互的呢?

要理解这个问题，我们先来看看下面这张图:

![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_JavaScriptCore.jpg)

可以看到，每个 JavaScriptCore 中的 JSVirtualMachine 对应着一个原生线程，同一个 JSVirtualMachine 中 可以使用 JSValue 与原生线程通信，遵循的是JSExport协议:原生线程可以将类方法和属性提供给 JavaScriptCore 使用，JavaScriptCore 可以将JSValue提供给原生线程使用。

JavaScriptCore 和原生应用要想交互，首先要有 JSContext。JSContext 直接使用 init 初始化，会默认使用系 统创建的 JSVirtualMachine。如果 JSContext 要自己指定使用哪个 JSVirtualMachine，可以使用 initWithVirtualMachine 方法来指定，代码如下:

```
// 创建 JSVirtualMachine 对象 jsvm
JSVirtualMachine *jsvm = [[JSVirtualMachine alloc] init];
// 使用 jsvm 的 JSContext 对象 ct
JSContext *ct = [[JSContext alloc] initWithVirtualMachine:jsvm];
```


如上面代码所示，首先初始化一个 JSVirtualMachine 对象 jsvm，再初始化一个使用 jsvm 的 JSContext 对象 ct。

JSVirtualMachine 是一个抽象的 JavaScript 虚拟机，是提供给开发者进行开发的，而其核心的 JavaScriptCore 引擎则是一个真实的虚拟机，包含了虚拟机都有的解释器和运行时部分。其中，解释器主要 用来将高级的脚本语言编译成字节码，运行时主要用来管理运行时的内存空间。当内存出现问题，需要调试 内存问题时，你可以使用 JavaScriptCore 里的 Web Inspector，或者通过手动触发 Full GC 的方式来排查内 存问题。


#### JavaScriptCore 引擎的组成

JavaScriptCore内部是由 Parser、Interpreter、Compiler、GC 等部分组成，其中 Compiler 负责把字节码翻 译成机器码，并进行优化。你可以点击[这个链接](https://trac.webkit.org/wiki/JavaScriptCore)，来查看WebKit 官方对JavaScriptCore 引擎的介绍。

JavaScriptCore 解释执行 JavaScript 代码的流程，可以分为两步。 

第一步，由 Parser 进行词法分析、语法分析，生成字节码。

第二步，由 Interpreter 进行解释执行，解释执行的过程是先由 LLInt(Low Level Interpreter)来执行 Parser 生成的字节码，JavaScriptCore 会对运行频次高的函数或者循环进行优化。优化器有 Baseline JIT、 DFG JIT、FTL JIT。对于多优化层级切换， JavaScriptCore 使用 OSR(On Stack Replacement)来管理。

如果想更深入地理解 JavaScriptCore 引擎的内容，可以参考戴铭老师的一篇博文“[深入剖析 JavaScriptCore](https://ming1016.github.io/2018/04/21/deeply-analyse-javascriptcore/)”。

我之前写过一个原生JS互调的代码：[点击这里](https://github.com/liuzhongning/Swift30/blob/master/Swift30/Projects/Project%2006/NNWebViewController.swift)。