# iOS开发之 - 键盘处理神器 IQKeyboardManager

>   年后上班第一天，比较闲，上午的时候抽空整理了` iOS `开发中常用的易忘知识点：[iOS开发之 - 小冷易忘知识点总结](http://www.jianshu.com/p/328a4c65dc5c)，有兴趣的朋友们可以去看看。下午整理了之前用过的一个第三方库——键盘处理神器  `IQKeyboardManager`。

> 平常在开发中，用到输入框的地方不胜其数，当输入框位于屏幕底部时，弹起的键盘很可能覆盖输入框，导致用户看不到输入结果，体验较差...... `IQKeyboardManager` 可以很简单快捷的解决键盘遮盖输入框的问题，接下来就一起来学习一下吧。

- 先简单认识下 IQKeyboardManager
GitHub 地址：[GitHub 地址](https://github.com/hackiftekhar/IQKeyboardManager)

- 官方示意效果图如下：

![官方示意效果图](http://upload-images.jianshu.io/upload_images/2665449-85b8ad91640b23c3.gif?imageMogr2/auto-orient/strip)

--------
再贴一下自己做的简单效果图☺️
先说下我的 Xcode 版本是：Version 8.1 (8B62)，简单起见直接在 Main.storyboard 中拖入 7 个UITextField， 每个 UITextField 都设有占位文字。

![简单效果图☺️](http://upload-images.jianshu.io/upload_images/2665449-1cabc35287b06995.gif?imageMogr2/auto-orient/strip)


--------
### 以下是 IQKeyboardManager 的一些具体使用

#### 1. 用 Cocoapod  导入或直接下载拖进去，这里方便起见直接用 Cocoapod 导入。
IQKeyboardManager 的 GitHub地址：[IQKeyboardManager 的 GitHub 地址](https://github.com/hackiftekhar/IQKeyboardManager)
#### 2. 在 AppDelegate.m 中导入头文件 
```
#import <IQKeyboardManager/IQKeyboardManager.h> 
```
#### 3. 在 AppDelegate 中设置全局属性
```
 - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    IQKeyboardManager *keyboardManager = [IQKeyboardManager sharedManager]; // 获取类库的单例变量

    keyboardManager.enable = YES; // 控制整个功能是否启用

    keyboardManager.shouldResignOnTouchOutside = YES; // 控制点击背景是否收起键盘

    keyboardManager.shouldToolbarUsesTextFieldTintColor = YES; // 控制键盘上的工具条文字颜色是否用户自定义

    keyboardManager.toolbarManageBehaviour = IQAutoToolbarBySubviews; // 有多个输入框时，可以通过点击Toolbar 上的“前一个”“后一个”按钮来实现移动到不同的输入框

    keyboardManager.enableAutoToolbar = YES; // 控制是否显示键盘上的工具条

    keyboardManager.shouldShowTextFieldPlaceholder = YES; // 是否显示占位文字

    keyboardManager.placeholderFont = [UIFont boldSystemFontOfSize:17]; // 设置占位文字的字体

    keyboardManager.keyboardDistanceFromTextField = 10.0f; // 输入框距离键盘的距离

    return YES;
}
```
#### 4. 若某个类不需要使用 IQKeyboardManager，可以在这个类中这样设置
```
  - (void)viewWillAppear:(BOOL)animated {
  [super viewWillAppear:animated];
   [IQKeyboardManager sharedManager].enable = NO;
}

  - (void)viewWillDisappear:(BOOL)animated {
  [super viewWillDisappear:animated]; 
  [IQKeyboardManager sharedManager].enable = YES;
}
```
#### 5. 常用属性介绍

- `sharedManager`：获取类库的单例变量
- ` enable`：项目使用不使用 IQKeyboardManager 这个类库，当然，某些页面可以根据需要单独设置
- `shouldResignOnTouchOutside`：点击背景页面时是否收起键盘
- `shouldToolbarUsesTextFieldTintColor`：控制键盘上的工具条文字颜色是否用户自定义，默认为 NO
- `toolbarManageBehaviour`：有多个输入框时，可以通过点击Toolbar 上的“前一个” “后一个”按钮来实现移动到不同的输入框
- `enableAutoToolbar`：是否显示键盘上的工具条
- `shouldShowTextFieldPlaceholder`：是否显示占位文字（如果输入框有占位文字，那么在 Toolbar 中默认会显示出来）
- `placeholderFont`：占位文字的字体大小
- `keyboardDistanceFromTextField`：输入框距离键盘的距离

#### 6. 再推荐几篇不错的相关文章
   - [iOS开发第三方库一 IQKeyboardManager](http://www.jianshu.com/p/01c0682003a9)

- [自动处理键盘事件的第三方库 IQKeyboardManager](https://my.oschina.net/u/1418722/blog/384477)

- [iOS开发之处理键盘问题神器IQKeyboardManager](http://www.jianshu.com/p/3ea3db5429f3)
