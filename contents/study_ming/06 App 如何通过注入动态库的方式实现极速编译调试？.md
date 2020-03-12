> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 06 App 如何通过注入动态库的方式实现极速编译调试？

在这一节，为了提高编译速度从而提高开发效率，戴铭老师推荐了一个神器 Injection，利用这个神器，可以像开发 JS 一样，只 command + s 一下就可以查看更改效果。这里是 GitHub 地址：[https://github.com/johnno1962/InjectionIII](https://github.com/johnno1962/InjectionIII)


#### 在 App Store 搜索下载这个工具

#### 在工程的 AppDelegate.m 中加入代码：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    #if DEBUG
        
        [[NSBundle bundleWithPath:@"/Applications/InjectionIII.app/Contents/Resources/iOSInjection.bundle"] load];
        
    #endif
    // Override point for customization after application launch.
    return YES;
}
```

#### 选择模拟器运行程序（目前只能在模拟器里面使用），当程序加载完成后如果没选择工程路径的话会弹出一个选择工程目录的对话框，选择工程的目录就行了。

#### 在任意使用的 OC 类的 .m 文件里面添加方法

```
- (void)injected {
	NSLog(@"此处的代码写完之后，按下Ctrl+S保存一下就可以在模拟器里看到刚刚改的代码了");
    self.view.backgroundColor = [UIColor orangeColor];
}
```


戴铭老师讲解了其工作原理，有需要了解的可以点击[《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)。