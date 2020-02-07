# iOS开发之 - DZNEmptyDataSet 空白页占位图

> 发现了一个很有用的第三方，[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet)，用起来也极其方便，这里给大家分享下！先说下其用途：`DZNEmptyDataSet` 主要用于 `UITableView` 以及 `UICollectionView` 页面空白时展示（据说也可以用于 `UIScrollView`， 有兴趣的童鞋可以试试）。开发中之所以用占位图，是为了提高用户体验；而为什么选 `DZNEmptyDataSet` 而不自行设计占位图（做过占位图的童鞋应该会觉得做占位图并不难，但没有对比就没有伤害，你如果试用了 `DZNEmptyDataSet` ，就会顿悟自己做占位图是多么地繁琐）。坦白说，`DZNEmptyDataSet` 是一个为了让开发人员“变懒”的第三方（貌似每个第三方都是这样的“宗旨”），接下来我们就一起看下 `DZNEmptyDataSet` 吧！

###一、先来认识一下 DZNEmptyDataSet
- GitHub 地址：[DZNEmptyDataSet](https://github.com/dzenbot/DZNEmptyDataSet)
- DZNEmptyDataSet 效果图


![DZNEmptyDataSet效果图1](http://upload-images.jianshu.io/upload_images/2665449-a0171c63c0ebeaa1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![DZNEmptyDataSet效果图2](http://upload-images.jianshu.io/upload_images/2665449-9b7b23c881b34deb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###二、用 DZNEmptyDataSet 实现占位图效果
不得不说，其使用方法极其简单，用 cocoapods 导入可以，直接拖进项目也行。。。接着~~~

- 导入头文件

```
#import "UIScrollView+EmptyDataSet.h"
```

- 添加代理协议

```
@interface NNViewController ()<UITableViewDataSource, DZNEmptyDataSetSource, DZNEmptyDataSetDelegate>
```
- 设置代理协议

```
    _tableView = [[UITableView alloc] initWithFrame:CGRectMake(0, 0, NNSCREENWIDTH, NNSCREENHEIGHT) style:UITableViewStylePlain];
    [self.view addSubview:self.tableView];
    
    self.tableView.dataSource = self;
    self.tableView.emptyDataSetSource = self;
    self.tableView.emptyDataSetDelegate = self;
    self.tableView.tableFooterView = [[UIView alloc] init];
   
```

- 设置空白页展示图片

```
#pragma mark - DZNEmptyDataSetSource
// 返回图片
- (UIImage *)imageForEmptyDataSet:(UIScrollView *)scrollView{
    return [UIImage imageNamed:@"Screenshot_Slack"];
}
```

到这里简单的占位图已经做好了，是不是觉得 so 轻松 so easy！下面是效果图：
![效果图](http://upload-images.jianshu.io/upload_images/2665449-ec13c6b78895b11f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###三、接下来是 DZNEmptyDataSet 的其它使用方法
DZNEmptyDataSet 框架扩展性极强，可以在占位图上添加图片、标题文字、详情文字等等一堆东西，总有一款适合你！下边是一些具体用法及效果图。

####3.1 添加标题文字
```
// 返回标题文字
- (NSAttributedString *)titleForEmptyDataSet:(UIScrollView *)scrollView {
    NSString *text = @"这是一张空白页";
    NSDictionary *attribute = @{NSFontAttributeName: [UIFont boldSystemFontOfSize:18.0f], NSForegroundColorAttributeName: [UIColor darkGrayColor]};
    return [[NSAttributedString alloc] initWithString:text attributes:attribute];
}
```
![返回标题文字](http://upload-images.jianshu.io/upload_images/2665449-8d76ca82b929405b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3.2 添加详情文字
```
// 返回详情文字
- (NSAttributedString *)descriptionForEmptyDataSet:(UIScrollView *)scrollView { NSString *text = @"哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈";
    NSMutableParagraphStyle *paragraph = [NSMutableParagraphStyle new];
    paragraph.lineBreakMode = NSLineBreakByWordWrapping;
    paragraph.alignment = NSTextAlignmentCenter;
    NSDictionary *attribute = @{NSFontAttributeName: [UIFont systemFontOfSize:14.0f], NSForegroundColorAttributeName: [UIColor lightGrayColor], NSParagraphStyleAttributeName: paragraph};
    return [[NSAttributedString alloc] initWithString:text attributes:attribute];
}
```

![详情文字](http://upload-images.jianshu.io/upload_images/2665449-f0154c472bfe6c61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3.3 添加可以点击的按钮 上面带文字
```
// 返回可以点击的按钮 上面带文字
- (NSAttributedString *)buttonTitleForEmptyDataSet:(UIScrollView *)scrollView forState:(UIControlState)state {
    NSDictionary *attribute = @{NSFontAttributeName: [UIFont boldSystemFontOfSize:17.0f]};
    return [[NSAttributedString alloc] initWithString:@"哈喽" attributes:attribute];
}

//#pragma mark - DZNEmptyDataSetDelegate
// 处理按钮的点击事件
- (void)emptyDataSet:(UIScrollView *)scrollView didTapButton:(UIButton *)button {
    NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"https://www.baidu.com"]];
    if ([[UIApplication sharedApplication] canOpenURL:url]) {
        [[UIApplication sharedApplication] openURL:url];
    }
}
```
![可以点击的按钮](http://upload-images.jianshu.io/upload_images/2665449-d2ae69bb76fd8299.gif?imageMogr2/auto-orient/strip)

####3.4 空白区域点击事件

```
// 空白区域点击事件
- (void)emptyDataSet:(UIScrollView *)scrollView didTapView:(UIView *)view {
    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"哈哈" message:@"最近咋样" delegate:nil cancelButtonTitle:@"别点我" otherButtonTitles:@"点我干啥", nil];
    [alert show];
}
```
![空白区域点击事件](http://upload-images.jianshu.io/upload_images/2665449-bc977dd868779f9a.gif?imageMogr2/auto-orient/strip)

####3.5 改变标题文字与详情文字的距离
```
// 标题文字与详情文字的距离
- (CGFloat)spaceHeightForEmptyDataSet:(UIScrollView *)scrollView {
    return 100;
}
```
![标题文字与详情文字的距离](http://upload-images.jianshu.io/upload_images/2665449-c7e36364da637b6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3.6 空白区域的颜色自定义
```
// 返回空白区域的颜色自定义
- (UIColor *)backgroundColorForEmptyDataSet:(UIScrollView *)scrollView {
    return [UIColor cyanColor];
}
```
![空白区域的颜色自定义](http://upload-images.jianshu.io/upload_images/2665449-e6e70559dd5033fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3.7 标题文字与详情文字同时调整垂直偏移量
```
// 标题文字与详情文字同时调整垂直偏移量
- (CGFloat)verticalOffsetForEmptyDataSet:(UIScrollView *)scrollView {
    return -100;
}
```

![标题文字与详情文字同时调整垂直偏移量](http://upload-images.jianshu.io/upload_images/2665449-659f044350373024.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####3.8 添加动画效果
```
#pragma mark - DZNEmptyDataSetSource
// 返回图片
- (UIImage *)imageForEmptyDataSet:(UIScrollView *)scrollView{
    return [UIImage imageNamed:@"icon_wwdc"];
}

- (CAAnimation *)imageAnimationForEmptyDataSet:(UIScrollView *)scrollView {
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath: @"transform"];
    animation.fromValue = [NSValue valueWithCATransform3D:CATransform3DIdentity];
    animation.toValue = [NSValue valueWithCATransform3D:CATransform3DMakeRotation(M_PI_2, 0.0, 0.0,   1.0)];
    animation.duration = 0.25;
    animation.cumulative = YES;
    animation.repeatCount = MAXFLOAT;
    return animation;
}

#pragma mark - DZNEmptyDataSetDelegate
// 图片是否要动画效果，默认NO
- (BOOL)emptyDataSetShouldAnimateImageView:(UIScrollView *)scrollView {
    return YES;
}
```
![动画效果](http://upload-images.jianshu.io/upload_images/2665449-8a61fbb540b61cfa.gif?imageMogr2/auto-orient/strip)

####3.9 其它方法
```
#pragma mark - DZNEmptyDataSetSource
// 返回图片的 tintColor
- (UIColor *)imageTintColorForEmptyDataSet:(UIScrollView *)scrollView {
    return [UIColor yellowColor];
}

// 返回可点击按钮的 image
- (UIImage *)buttonImageForEmptyDataSet:(UIScrollView *)scrollView forState:(UIControlState)state {
    return [UIImage imageNamed:@"icon_wwdc"];
}

// 返回可点击按钮的 backgroundImage
- (UIImage *)buttonBackgroundImageForEmptyDataSet:(UIScrollView *)scrollView forState:(UIControlState)state {
    return [UIImage imageNamed:@"icon_wwdc"];
}

// 返回自定义 view
- (UIView *)customViewForEmptyDataSet:(UIScrollView *)scrollView {
    return nil;
}

#pragma mark - DZNEmptyDataSetDelegate
// 是否显示空白页，默认YES
- (BOOL)emptyDataSetShouldDisplay:(UIScrollView *)scrollView {
    return YES;
}
// 是否允许点击，默认YES
- (BOOL)emptyDataSetShouldAllowTouch:(UIScrollView *)scrollView {
    return YES;
}
// 是否允许滚动，默认NO
- (BOOL)emptyDataSetShouldAllowScroll:(UIScrollView *)scrollView {
    return YES;
}
// 图片是否要动画效果，默认NO
- (BOOL)emptyDataSetShouldAnimateImageView:(UIScrollView *)scrollView {
    return YES;
}
// 空白页将要出现
- (void)emptyDataSetWillAppear:(UIScrollView *)scrollView {
    
}
// 空白页已经出现
- (void)emptyDataSetDidAppear:(UIScrollView *)scrollView {
    
}
// 空白页将要消失
- (void)emptyDataSetWillDisappear:(UIScrollView *)scrollView {
    
}
// 空白页已经消失
- (void)emptyDataSetDidDisappear:(UIScrollView *)scrollView {
    
}
```
------------------华丽的分割线------------------

小结：本小白只在 UITableView 上试用了 DZNEmptyDataSet 框架。。。公司要求年前再迭代一个版本，时间不太充裕因此没逐一去试。。。哪位大神如果试了 UICollectionView 或 UIScrollView 发现有问题，还请不吝赐教！