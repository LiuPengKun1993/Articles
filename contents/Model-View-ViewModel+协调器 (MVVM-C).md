# Model-View-ViewModel+协调器 (MVVM-C)


![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVC-01.jpg)


此系列文章是《App架构》的读书笔记及心得。

### 前言

> 没有哪个单独的模式能在所有情景下都能做到最好。我们在开发中需要具体情况具体分析，最好的模式就是对你来说最为清晰的模式。


### MVVM 架构

Model-View-ViewModel (MVVM) 是一种基于 MVC 进行改进的模式，它将所有 model 相关的任务 (包括更新 model、观察变更、将 model 变形为可以显示的形式等) 从 controller 层抽离出来，放到新的叫做 view-model 的一层对象中。在通常的 iOS 实现中，view-model 位于 model 和 controller 之间：

![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVVM-01.jpg)


和所有好的模式一样，MVVM 所做的不仅仅是把代码移动到新的地方。加入一层新的 view-model 层的目的是双重的：

- 鼓励将 model 和 view 之间的关系构建为一系列的变形管道。
- 提供一套独立于 app 框架的接口，但是它在相当程度上代表了 view 应该展示的状态。

MVVM 模式可以被看作是 MVC 的精密版本。两者的模式很相似：controller 层充分了解程序的结构，它使用这些认知来对所有部件进行构建和连接。相比起 MVC，主要有三个不同：

- 必须创建 view-model。
- 必须建立起 view-model 和 view 之间的绑定。
- Model 由 view-model 拥有，而不是由 controller 所拥有。

为了保持 view 与 view-model 的同步，MVVM 强制使用某种形式的绑定，也就是说，需要一种保证一个对象上的属性与另一个对象上的属性同步的方式。Controller 负责构建这些绑定，将 view-model 所暴露的属性和场景中 view-model 所代表的 view 上的属性关联起来。

### 响应式编程

一般我们使用响应式编程的方式来实现绑定，响应式编程能够很好的展示 MVVM 的优势。在 iOS 开发中，我们可以使用 KVO 或者通知来实现响应式编程。另外我们还可以使用响应式编程的经典框架 [ReactiveCocoa](https://github.com/ReactiveCocoa)。

关于 RAC，在 iOS 开发中，当我们需要处理某些业务逻辑时，比如按钮的点击使用 delegate，属性值改变使用 KVO、通知等方式，现在都可以统一通过 RAC 进行处理。RAC 为事件提供了很多处理方法，而且利用 RAC 处理事件更方便，可以把要处理的事情，和监听的代码放在一起，这样非常方便开发人员进行管理，不需要跳到对应的方法里。很符合我们开发中高内聚，低耦合的思想。


### 协调器

在 MVVM 中，协调器并非是强制需要的部件，但是它可以帮助 controller 从两个额外的职责中解放出来：controller 可以不再需要管理其他 controller 的展示，它也不再需要去协调 model 数据和 controller 之间的通讯。协调器负责维护 controller 的层级，这样一来，controller 和 view-model 就只需要专注处理它们在特定场景的使用了。Model-View-ViewModel+协调器 (MVVM-C) 的框图如下：

![](https://github.com/liuzhongning/Articles/blob/master/resources/App-MVVM-02.jpg)

协调器也可以被运用到 Cocoa MVC 中，用来减少 controller 的职责，并对它们解耦。要是没有协调器，一个 controller 往往会需要通过将其他的 controller 推入导航栈，或是以 model 的方式来显示它们。当使用协调器时，controller 就可以调用协调器上的代码，而不需要直接显示其他 controller 了。



### 总结

因为 MVVM-C 加入了额外的一层来进行管理，看起来是比 Cocoa MVC 模式更加复杂。不过，在实现的层级，如果开发人员能够始终如一地贯彻这个模式，代码会变得更简单一些。这里说的简单并不意味着容易，只有当你对常见的响应式代码变形熟悉以后，才不会对书写代码感到无从下手，才不会对调试问题感到懊恼沮丧。不过，从令人高兴的一面来说，精心设计的数据管道通常不容易产生错误，在长期来看维护也更容易一些。

相关文章:

- [MVC 下的网络层架构](https://github.com/liuzhongning/Articles/blob/master/contents/MVC%20下的网络层架构.md)
- [Model-View-Controller (MVC)](https://github.com/liuzhongning/Articles/blob/master/contents/Model-View-Controller%20(MVC).md)

