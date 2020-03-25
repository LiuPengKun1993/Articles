> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 30 如何制定一套适合自己团队的iOS编码规范?

这一课程，戴铭老师主要从常量、变量、属性、条件语句、循环语句、函数、类，以及分类这8个方面说明了好的代码规范。

#### 常量

尽量使用类型常量，而不是使用宏定义。比如要定义一个字符串常量，可以写成:

```
static NSString * const STMProjectName = @"GCDFetchFeed"
```

#### 变量

- 变量名应该可以明确体现出功能，最好再加上类型做后缀。这样也就明确了每个变量都是做什么的，而不是把一个变量当作不同的值用在不同的地方。
- 在使用之前，需要先对变量做初始化，并且初始化的地方离使用它的地方越近越好。
- 不要滥用全局变量，尽量少用它来传递值，通过参数传值可以减少功能模块间的耦合。

```
let nameString = "Tom"
print("\(nameString)")
nameLabel.text = nameString
```

#### 属性

- Objective-C 里的属性，要尽量通过 get 方法来进行懒加载，以避免无用的内存占用和多余的计算。
- Swift 的计算属性如果是只读，可以省掉 get 子句。示例代码如下:

```
var rectangleArea: Double {
    return long * wide
}
```

#### 条件语句

在条件语句中，需要考虑到条件语句中可能涉及的所有分支条件，对于每个分支条件都需要考虑到，并进行处理，减少或不使用默认处理。特别是使用 Switch 处理枚举时，不要有 default 分支。

在iOS开发中，你使用 Swift 语言编写 Switch 语句时，如果不加default分支的话，当枚举有新增值时，编译器会提醒你增加分支处理。这样，就可以有效避免分支漏处理的情况。

另外，条件语句的嵌套分支不宜过多，可以充分利用 Swift 中的 guard 语法。比如，这一段处理登录的示例代码:

```
if let userName = login.userNameOK {
      if let password = login.passwordOK {
// 登录处理
... } else {
          fatalError("login wrong")
      }
  } else {
      fatalError("login wrong")
}
```
上面这段代码表示的是，当用户名和密码都没有问题时再进行登录处理。那么，我们使用 guard 语法时， 可以改写如下:

```
 guard
    let userName = login.userNameOK,
    let password = login.passwordOK,
    else {
      fatalError("login wrong")
  }
// 登录处理 ...
```

可以看到，改写后的代码更易读了，异常处理都在一个区域，guard 语句真正起到了守卫的职责。而且你一旦声明了 guard，编译器就会强制你去处理异常，否则就会报错。异常处理越完善，代码就会越健壮。所以，条件语句的嵌套处理，你可以考虑使用 guard 语法。

#### 循环语句

在循环语句中，我们应该尽量少地使用 continue 和 break，同样可以使用 guard 语法来解决这个问题。解 决方法是:所有需要 continue 和 break 的地方统一使用 guard 去处理，将所有异常都放到一处。这样做的 好处是，在维护的时候方便逻辑阅读，使得代码更加易读和易于理解。

#### 函数

对于函数来说，体积不宜过大，最好控制在百行代码以内。如果函数内部逻辑多，我们可以将复杂逻辑分解成多个小逻辑，并将每个小逻辑提取出来作为一个单独的函数。每个函数处理最小单位的逻辑，然后一层一层往上组合。

这样，我们就可以通过函数名明确那段逻辑处理的目的，提高代码的可读性。

拆分成多个逻辑简单的函数后，我们需要注意的是，要对函数的入参进行验证，guard 语法同样适用于检查入参。比如下面的这个函数:

```
func saveRSS(rss: RSS?, store: Store?) {
    guard let rss = rss else {
        return
    }
    guard let store = store else {
        return
    }
    // 保存 RSS
    return
}
```

如上面代码所示，通过 guard语法检查入参 rss 和 store 是否异常，提高函数的健壮性会来得更容易些。

另外，函数内尽量避免使用全局变量来传递数据，使用参数或者局部变量传递数据能够减少函数对外部的依赖，减少耦合，提高函数的独立性，提高单元测试的准确性。

#### 类

在 Objective-C 中，类的头文件应该尽可能少地引入其他类的头文件。你可以通过 class 关键字来声明，然后 在实现文件里引入需要的其他类的头文件。

对于继承和遵循协议的情况，无法避免引入其他类的头文件，所以你在代码设计时还是要尽量减少继承，特别是继承关系太多时不利于代码的维护和修改，比如说修改父类时还需要考虑对所有子类的影响，如果评估不全，影响就难以控制。

#### 分类

在写分类时，分类里增加的方法名要尽量加上前缀，而如果是系统自带类的分类的话，方法名就一定要加上前缀，来避免方法名重复的问题。

#### Code Review

好的代码规范首先要保证代码逻辑清晰，然后再考虑简洁、扩展、重用等问题。逻辑清晰的代码
几乎不需要注释来说明，通过命名和清晰地编写逻辑就能够让其他人快速读懂。

**不需要注释就能轻松读懂的代码，使用的语言特性也必然是通用和经典的，过新的语言特性和黑魔法不利于 代码逻辑的阅读，应该减少使用，即使使用也需要多加注释，避免他人无法理解。**

如果是 Swift 语言的话，你可以使用 [SwiftLint](https://github.com/realm/SwiftLint) 工具来检查代码规范。Swift 通过 Hook Clang 和 SourceKit 中 AST 的回调来检查源代码，如何使用SourceKit 开发工具可以参看这篇文章“[Uncovering SourceKit](https://www.jpsim.com/uncovering-sourcekit/)”。

SwiftLint 检查的默认规则，你可以参考[它的规则说明](https://realm.github.io/SwiftLint/rule-directory.html)。SwiftLint 也支持自定义检查规则，支持你添加自己制定的代码规范。你可以在 SwiftLint 目录下添加一个 .swiftlint.yml 配置文件来自定义基于正则表达式的自 定义规则。具体方法，你可以参看官方定义[自定义规则的说明](https://github.com/realm/SwiftLint/blob/master/README_CN.md)。

如果你是使用 Objective-C 语言开发的话，可以使用 OCLint 来做代码规范检查。关于 OCLint 如何定制自己 的代码规范检查，你可以参看杨萧玉的这篇博文“[使用 OCLint 自定义 MVVM 规则](http://yulingtianxia.com/blog/2019/01/27/MVVM-Rules-for-OCLint/)”。

然后，进行人工检查。人工检查的好处是：通过团队成员之间互相检查代码的方式，希望能够达到相互沟通交流，甚至相互学习 的效果。

---

- [Introduction to Coding Guidelines for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
- [Spotify 的 Objective-C 编码规范](https://github.com/spotify/ios-style)
- [纽约时报的 Objective-C 的编码规范](https://github.com/NYTimes/objective-c-style-guide)
- [Raywenderlich 的 Objective-C 编码规范](https://github.com/raywenderlich/objective-c-style-guide)
- [Raywenderlich 的 Swift 编码规范](https://github.com/raywenderlich/swift-style-guide)