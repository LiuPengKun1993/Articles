# 《Objective-C高级编程iOS与OS X多线程和内存管理》读书笔记

> 这两天重读了《Objective-C高级编程 iOS与OS X多线程和内存管理》，此书主要详细讲解了“自动引用计数”“Blocks”“GCD”这三大模块。

文章目录：

- 1. 自动引用计数
	- 1.1 自动引用计数概念
	- 1.2 内存管理的思考方式
	- 1.3 所有权修饰符
	- 1.4 ARC 的规则
- 2. Blocks
	- 2.1 Block 概念
	- 2.2 Block 本质
	- 2.3 Block 的三种类型
	- 2.4 Block 何时会复制到堆
	- 2.5 Block 循环引用
- 3. Grand Central Dispatch
	- 3.1 Grand Central Dispatch 概念
	- 3.2 多线程编程可能会出现的问题
	- 3.3 GCD API

### 1. 自动引用计数

#### 1.1 自动引用计数概念

在 LLVM 编译器中设置 ARC 为有效状态，就无需再次键入 retain 或者 release 代码，编译器将结合运行时基于引用计数自动进行内存管理。

#### 1.2 内存管理的思考方式

- 自己生成的对象，自己所持有。
- 非自己生成的对象，自己也能持有。
- 自己持有的对象不再需要时释放。
- 非自己持有的对象无法释放。

#### 1.3 所有权修饰符

- __strong
	- `__strong` 修饰符是 id 类型和对象类型默认的所有权修饰符。
	- `__strong` 修饰符表示对对象的“强引用”。持有强引用的变量在超出其作用域时被废弃，随着强引用的失效，引用的对象会随之释放。
- __weak
	- `__weak` 修饰符可以避免循环引用，是弱引用，`__weak` 不持有对象，在超出其变量作用域时，对象即被释放。
	- `__weak` 修饰符还有另一优点，在持有某对象的弱引用时，若该对象被废弃，则此弱引用将自动失效且处于 nil 被赋值的状态。
- __unsafe_unretained
	- `__unsafe_unretained` 修饰符类似于 `__weak`，不会对对象进行retain，但该对象销毁时，会依然指向之前的内存空间（野指针）。
- __autoreleasing
	- `__autoreleasing` 修饰符，在当前 autoreleasepool 作用域内有效，出了当前的 autoreleasepool 会被自动释放。

注意点：循环引用容易发生内存泄漏，内存泄漏是指应当废弃的对象在超出其生存周期后继续存在。

#### 1.4 ARC 的规则

- 不能使用 retain/release/retainCount/autorelease
	- 内存管理是编译器的工作，因此没必要使用内存管理的方法。
- 不能使用 NSAllocateObject/NSDeallocateObject
	- 同 retain 等方法一样，如果使用会引起编译错误。
- 须遵守内存管理的方法命名规则
- 不要显示调用 dealloc
	- ARC 会自动对此进行处理，因此不必书写 [super dealloc]。
- 使用 @autoreleasepool 块替代 NSAutoreleasePool
- 不能使用区域（NSZone）
- 对象型变量不能作为 C 语言结构体（struct/union）的成员
- 显示转换 "id" 和 "void *"


### 2. Blocks

#### 2.1 Block 概念

带有自动变量值的匿名函数。

#### 2.2 Block 本质

Block 本质是一个 OC 对象，它内部有一个 isa 指针。是封装了函数调用和函数调用环境的 OC 对象。

iOS 类的本质即 class_t 结构体：

```
struct class_t {
   struct class_t *isa;
   struct class_t *superclass;
   Cache cache;
   IMP *vtable;
   uintptr_t data_NEVER_USE;
}
```

该实例名称持有声明的成员变量、方法的名称、方法的实现（即函数指针）、属性以及父类的指针。

#### 2.3 Block 的三种类型

|类|设置对象的存储域|备注|
|---|---|---|
| `_NSConcreteStackBlock` |栈|捕获局部变量|
| `_NSConcreteGlobalBlock` |程序的数据区域（.data 区）|不捕获自动变量|
| `_NSConcreteMallocBlock` |堆|捕获成员变量|

#### 2.4 Block 何时会复制到堆

- 调用 Block 的 copy 实例方法时
- Block 作为函数返回值返回时
- 将 Block 赋值给附有 __strong 修饰符 id 类型的类或 Block 类型成员变量时
- 在方法名中含有 usingBlock 的 Cocoa 框架方法或 GCD 的 API 中传递 Block 时。

#### 2.5 Block 循环引用

- 原因：Block中附有__strong修饰符的对象类型自动变量在从栈复制到堆上时，该对象会被Block所持有。
- 解决方案：通过 `__weak` 或 `__unsafe_unretained` 修饰符来替代 `__strong` 类型的被截获的自动变量
通过 `__block` 说明符和设置nil来打破循环


### 3. Grand Central Dispatch

#### 3.1 Grand Central Dispatch 概念

Grand Central Dispatch （GCD） 是异步执行任务的技术之一，一般将应用程序中记述的线程管理用的代码在系统级中实现，开发者只需要定义想执行的任务并追加到适当的 Dispatch Queue 中，GCD 就能生成必要的线程并计划执行任务。

#### 3.2 多线程编程可能会出现的问题

- 多个线程更新相同的资源会导致数据的不一致（数据竞争）- 解决：使用 Serial Dispatch Queue （串行队列）
- 停止等待事件的线程会导致多个线程相互持续等待（死锁）
- 使用太多线程会消耗大量内存

Dispatch Queue 按照追加的顺序（先进先出 FIFO）执行处理，另在执行处理时存在两种 Dispatch Queue ：一种是等待现在执行中处理的 Serial Dispatch Queue，另一种是不等待现在执行中处理的 Concurrent Dispatch Queue。

#### 3.3 GCD API

- 可使用 `dispatch_set_target_queue` API 设置 `Dispatch Queue` 的优先级，同时也可以使多个本应并行执行的多个 `Serial Dispatch Queue`，在目标 `Serial Dispatch Queue` 上串行执行。
- `dispatch_after` 函数并不准时，因为 `Main Dispatch Queue` 在主线程的 Runloop 中执行，所以在比如每隔 1/60 秒自行的 Runloop 中，Block 最快3秒后自行，最慢在3秒 + 1/60 秒后执行，并且在 `Main Dispatch Queue` 有大量处理追加或主线程的处理本身有延迟时，这个时间会更长。
- `Dispatch Group` 在追加到 `Dispatch Queue` 中的多个处理全部结束后想执行结束处理可使用 `Dispatch Group` 实现。
- `dispatch_sync` 如同简易版的 `dispatch_group_wait` 函数，会在指定队列中同步执行任务，在任务执行结束之前不会返回。如果在主线程同步执行 Block 就会出现死锁。
- `dispatch_apply` 函数可进行快速遍历。由于 `dispatch_apply` 函数与 `dispatch_sync` 函数相同，会等待处理执行结束，因此推荐在 `dispatch_async` 函数中非同步地执行 `dispatch_apply` 函数。
- `dispatch_suspend` 函数可暂时挂起指定的 `Dispatch Queue`。
- `dispatch_resume` 函数可恢复指定的 `Dispatch Queue`。
- `dispatch_semaphore` 函数可对操作进行更细粒度的排他控制。
- `dispatch_semaphore_wait`(semaphore, `DISPATCH_TIME_FOREVER`)，`dispatch_semaphore_wait` 函数等待 `Dispatch Semaphore` 的计数值达到大于或等于1。该处理结束是使用 `dispatch_semaphore_signal` 函数将 `Dispatch Semaphore` 的计数值加1.
- `dispatch_once` 函数是保证再应用程序执行中只执行一次指定处理的 API。
- `dispatch_IO` 函数可多线程并发处理大文件，以提高文件读取速度。