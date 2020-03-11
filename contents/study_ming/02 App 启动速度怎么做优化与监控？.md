> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 02 App 启动速度怎么做优化与监控？

#### App 的启动主要包括三个阶段：
1. main() 函数执行前；
2. main() 函数执行后；
3. 首屏渲染完成后；

#### main() 函数执行之前：
- 减少动态库加载。
- 减少加载启动后不会去使用的类或者方法。
- +load() 方法里的内容可以放到首屏渲染完成后再执行，或使用 +initialize() 方法替换掉。
- 控制 C++ 全局变量的数量。

#### main() 函数执行之后：

这一阶段我们通常指的是从 main() 函数执行开始，到 appDelegate 的didFinishLaunchingWithOptions 方法里首屏渲染相关方法执行完成。优化手段包括：

- 将非首屏渲染需要的功能滞后，放到合适的阶段进行。
- 将首屏渲染前主线程上的耗时方法滞后或者异步执行。

#### 首屏渲染完成后：
从函数上来看，这个阶段指的就是截止到 didFinishLaunchingWithOptions 方法作用域内执行首屏渲染之后的所有方法执行完成。因为用户已经能够看到App的首页信息了，所以优化优先级最低。优化思路：

- 优先处理比较占主线程资源的操作，保证用户后续操作的流畅性。

#### 如何对 App 启动速度进行监控呢？

监控手段主要有两种：

- 定时抓取主线程上的方法调用堆栈，计算一段时间内各个方法的耗时，类似 Xcode 中的 Time Profiler。
- 对 objc_msgSend 方法进行 hook 来掌控方法的执行耗时。
	- hook objc_msgSend 方法优点是非常精确，缺点是只能针对 oc 方法，对于 c 方法和 block，可以使用 libffi 的 ffi_call 来达成 hook。
hook objc_msgSend 监控工具完整代码见[戴铭老师的github](https://github.com/ming1016/GCDFetchFeed/tree/f9d9650b264e720fea14920a3b1a046353a12690/GCDFetchFeed/GCDFetchFeed/Lib/SMLagMonitor)
