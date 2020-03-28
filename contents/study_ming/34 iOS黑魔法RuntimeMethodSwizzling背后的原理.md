> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 34 iOS黑魔法RuntimeMethodSwizzling背后的原理

Runtime Method Swizzling 编程方式，也可以叫作 AOP(Aspect-Oriented Programming，面向切面编程)。


AOP 是一种编程范式，也可以说是一种编程思想，使用 AOP 可以解决 OOP(Object Oriented Programming，面向对象编程)由于切面需求导致单一职责被破坏的问题。通过 AOP 可以不侵入 OOP 开发，非常方便地插入切面需求功能。

#### 直接使用 Runtime 方法交换开发的风险有哪些?

Runtime 不光能够进行方法交换，还能够在运行时处理 Objective-C 特性相关(比如类、成员函数、继承) 的增删改操作。

这里戴铭老师推荐了于德志 ([halfrost](https://github.com/halfrost)) 的三篇 Runtime 文章，[isa 和 Class](https://halfrost.com/objc_runtime_isa_class/)、[消息发送与转发](https://halfrost.com/objc_runtime_objc_msgsend/)，以及[如何正确使用 Runtime](https://halfrost.com/how_to_use_runtime/)。

直接使用 Runtime 进行方法交换非常简单，代码如下:

```
+ (void)hookClass:(Class)classObject fromSelector:(SEL)fromSelector toSelector:(SEL)toSelector {
    Class class = classObject;
    // 得到被交换类的实例方法
    Method fromMethod = class_getInstanceMethod(class, fromSelector);
    // 得到交换类的实例方法
    Method toMethod = class_getInstanceMethod(class, toSelector);
    // class_addMethod() 函数返回成功表示被交换的方法没实现，然后会通过 class_addMethod() 函数先实现;返回失败则表示被交换方法已存在，
    if(class_addMethod(class, fromSelector, method_getImplementation(toMethod), method_getTypeEncoding(toMethod))) {
        // 进行方法的交换
        class_replaceMethod(class, toSelector, method_getImplementation(fromMethod), method_getTypeEncoding(fromMethod));
    } else {
        // 交换 IMP 指针
        method_exchangeImplementations(fromMethod, toMethod);
      }
}
```

如代码所示:通过 class_getInstanceMethod() 函数可以得到被交换类的实例方法和交换类的实例方法。使 用 class_addMethod() 函数来添加方法，返回成功表示被交换的方法没被实现，然后通过 class_addMethod() 函数实现;返回失败则表示被交换方法已存在，可以通过 method_exchangeImplementations() 函数直接进行 IMP 指针交换以实现方法交换。

- 四个典型的直接使用 Runtime 方法进行方法交换的风险：
	- 第一个风险是，需要在 +load 方法中进行方法交换。因为如果在其他时候进行方法交换，难以保证另外一个 线程中不会同时调用被交换的方法，从而导致程序不能按预期执行。
	- 第二个风险是，被交换的方法必须是当前类的方法，不能是父类的方法，直接把父类的实现拷贝过来不会起作用。父类的方法必须在调用的时候使用，而不是方法交换时使用。
	- 第三个风险是，交换的方法如果依赖了 cmd，那么交换后，如果 cmd 发生了变化，就会出现各种奇怪问 题，而且这些问题还很难排查。特别是交换了系统方法，你无法保证系统方法内部是否依赖了 cmd。
	- 第四个风险是，方法交换命名冲突。如果出现冲突，可能会导致方法交换失败。

	
更多关于运行时方法交换的风险，可以查看 Stackoverflow 上的问题讨论“[What are the Dangers of Method Swizzling in Objective C?](https://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c)”。

#### 更安全的方法交换库[Aspects](https://github.com/steipete/Aspects)

Aspects 是一个通过 Runtime 消息转发机制来实现方法交换的库。它将所有的方法调用都指到 _objc_msgForward 函数调用上，按照自己的方式实现了消息转发，自己处理参数列表，处理返回值，最后 通过 NSInvocation 调用来实现方法交换。同时，Aspects 还考虑了一些方法交换可能会引发的风险，并进行了处理。

```
[UIViewController aspect_hookSelector:@selector(viewWillAppear:) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> aspectInfo, BOOL animated) {
    NSLog(@"View Controller %@ will appear animated: %tu", aspectInfo.instance, animated);
} error:NULL];
```

上面这段代码是 Aspects 通过运行时方法交换，按照 AOP 方式添加埋点的实现。代码简单，可读性高，接口使用 Block 也非常易用。按照这种方式，直接使用Aspects即可。

Aspects 的整体流程是，先判断是否可进行方法交换。这一步会进行安全问题的判断处理。如果没有风险的话，再针对要交换的是类对象还是实例对象分别进行处理。

- 对于类对象的方法交换，会先修改类的 forwardInvocation ，将类的实现转成自己的。然后，重新生成一 个方法用来交换。最后，交换方法的 IMP，方法调用时就会直接对交换方法进行消息转发。
- 对于实例对象的方法交换，会先创建一个新的类，并将当前实例对象的 isa 指针指向新创建的类，然后再 修改类的方法。


整个流程的入口是 aspect_add() 方法，这个方法里包含了 Aspects 的两个核心方法，第一个是进行安全判断的 aspect_isSelectorAllowedAndTrack 方法，第二个是执行类对象和实例对象方法交换的 aspect_prepareClassAndHookSelector 方法。

aspect_isSelectorAllowedAndTrack 方法，会对一些方法比如 retain、release、autorelease、 forwardInvocation 进行过滤，并对 dealloc 方法交换做了限制，要求只能使用 AspectPositionBefore 选项。同时，它还会过滤没有响应的方法，直接返回 NO。

安全判断执行完，就开始执行方法交换的 aspect_prepareClassAndHookSelector 方法，其实现代码如下:

```
static void aspect_prepareClassAndHookSelector(NSObject *self, SEL selector, NSError **error) {
    NSCParameterAssert(selector);
    //HookClass过程
    Class klass = aspect_hookClass(self, error);
    //此时的klass类为刚创建的具有_Aspects_后缀的子类，在创建的时候指定类他的父类，所以我们可以获取到selector这个方法
    Method targetMethod = class_getInstanceMethod(klass, selector);
    IMP targetMethodIMP = method_getImplementation(targetMethod);
    //判断是否为消息转发
    if (!aspect_isMsgForwardIMP(targetMethodIMP)) {
        //获得原生方法的类型编码
        const char *typeEncoding = method_getTypeEncoding(targetMethod);
        SEL aliasSelector = aspect_aliasForSelector(selector);
        if (![klass instancesRespondToSelector:aliasSelector]) {
            //为klass添加aspects__XX方法，方法的实现为原生方法的实现。
            __unused BOOL addedAlias = class_addMethod(klass, aliasSelector, method_getImplementation(targetMethod), typeEncoding);
            NSCAssert(addedAlias, @"Original implementation for %@ is already copied to %@ on %@", NSStringFromSelector(selector), NSStringFromSelector(aliasSelector), klass);
        }
        // 将原生方法实现替换为_objc_msgForward或_objc_msgForward_stret，用来实现消息转发
        class_replaceMethod(klass, selector, aspect_getMsgForwardIMP(self, selector), typeEncoding);
        AspectLog(@"Aspects: Installed hook for -[%@ %@].", klass, NSStringFromSelector(selector));
    }
}
```

可以看到，通过 aspect_hookClass()函数可以判断出 class 的 selector 是实例方法还是类方法，如果是实例 方法，会通过 class_addMethod 方法生成一个交换方法，这样在 forwordInvocation 时就能够直接执行交 换方法。aspect_hookClass 还会对类对象、元类、KVO 子类化的实例对象、class 和 isa 指向不同的情况进 行处理，使用 aspect_swizzleClassInPlace 混写 baseClass。

- [面向切面编程之疯狂的Aspect](https://www.jianshu.com/p/74457c9ffb68)
- [iOS面向切面编程AOP实践](https://www.jianshu.com/p/dbd2de44c03c)
- [iOS 开源库系列 Aspects核心源码分析---面向切面编程之疯狂的 Aspects](https://www.cnblogs.com/feng9exe/p/10386547.html)