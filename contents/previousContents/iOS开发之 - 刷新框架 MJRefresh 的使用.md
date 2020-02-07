# iOS开发之 - 刷新框架 MJRefresh 的使用

![MJRefresh](http://upload-images.jianshu.io/upload_images/2665449-9e9c86ec592c3c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 下拉刷新控件目前比较火的有好几种，此前用过[MJRefresh](https://github.com/CoderMJLee/MJRefresh/blob/master/README.md) 和[ SVPullToRefresh](https://github.com/samvermette/SVPullToRefresh)，相对而言，还是觉得 MJRefresh 更好用！因此抽了些时间整理了 MJRefresh 的用法。以下是文章内容。。。

 # 【文章目录】
## 一、类结构图
- MJRefreshComponent.h
- MJRefreshHeader.h
- MJRefreshFooter.h
- MJRefreshAutoFooter.h

## 二、参考例子
- 下拉刷新01-默认
- 下拉刷新02-动画图片
- 下拉刷新03-隐藏时间
- 下拉刷新04-隐藏状态和时间
- 下拉刷新05-自定义文字
- 下拉刷新06-自定义刷新控件
- 上拉刷新01-默认
- 上拉刷新02-动画图片
- 上拉刷新03-隐藏刷新状态的文字
- 上拉刷新04-全部加载完毕
- 上拉刷新05-自定义文字
- 上拉刷新06-加载后隐藏
- 上拉刷新07-自动回弹的上拉01
- 上拉刷新08-自动回弹的上拉02
- 上拉刷新09-自定义刷新控件(自动刷新)
- 上拉刷新10-自定义刷新控件(自动回弹)
- UICollectionView01-上下拉刷新
- UIWebView01-下拉刷新

# 【文章内容】

## 1.类结构图
![类结构图](http://upload-images.jianshu.io/upload_images/2665449-4a80291bb05b5283.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### MJRefreshComponent.h
```
/** 刷新控件的基类 */
@interface MJRefreshComponent : UIView
#pragma mark - 刷新状态控制
/** 进入刷新状态 */
- (void)beginRefreshing;
/** 结束刷新状态 */
- (void)endRefreshing;
/** 是否正在刷新 */
- (BOOL)isRefreshing;

#pragma mark - 其它
/** 根据拖拽比例自动切换透明度 */
@property (assign, nonatomic, getter=isAutomaticallyChangeAlpha) BOOL automaticallyChangeAlpha;
@end
```

### MJRefreshHeader.h
```
@interface MJRefreshHeader : MJRefreshComponent
/** 创建 header */
+ (instancetype)headerWithRefreshingBlock:(MJRefreshComponentRefreshingBlock)refreshingBlock;
/** 创建 header */
+ (instancetype)headerWithRefreshingTarget:(id)target refreshingAction:(SEL)action;

/** 这个 key 用来存储上一次下拉刷新成功的时间 */
@property (copy, nonatomic) NSString *lastUpdatedTimeKey;
/** 上一次下拉刷新成功的时间 */
@property (strong, nonatomic, readonly) NSDate *lastUpdatedTime;

/** 忽略多少 scrollView 的 contentInset 的顶部 */
@property (assign, nonatomic) CGFloat ignoredScrollViewContentInsetTop;
@end
```
### MJRefreshFooter.h
```
@interface MJRefreshFooter : MJRefreshComponent
/** 创建 footer */
+ (instancetype)footerWithRefreshingBlock:(MJRefreshComponentRefreshingBlock)refreshingBlock;
/** 创建 footer */
+ (instancetype)footerWithRefreshingTarget:(id)target refreshingAction:(SEL)action;

/** 提示没有更多的数据 */
- (void)noticeNoMoreData;
/** 重置没有更多的数据（消除没有更多数据的状态） */
- (void)resetNoMoreData;

/** 忽略多少 scrollView 的 contentInset 的底部*/
@property (assign, nonatomic) CGFloat ignoredScrollViewContentInsetBottom;

/** 自动根据有无数据来显示和隐藏 */
@property (assign, nonatomic) BOOL automaticallyHidden;
@end
```

### MJRefreshAutoFooter.h

```
@interface MJRefreshAutoFooter : MJRefreshFooter
/** 是否自动刷新(默认为 YES) */
@property (assign, nonatomic, getter=isAutomaticallyRefresh) BOOL automaticallyRefresh;

/** 当底部控件出现多少时就自动刷新(默认为1.0，也就是底部控件完全出现时，才会自动刷新) */
@property (assign, nonatomic) CGFloat appearencePercentTriggerAutoRefresh;
@end
```
## 2.参考例子
ps: 可以下载 [MJRefresh](https://github.com/CoderMJLee/MJRefresh)试试
![MJRefresh 演示](http://upload-images.jianshu.io/upload_images/2665449-9bfe1621e9aea9f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 下拉刷新01-默认
```
    self.tableView.mj_header = [MJRefreshNormalHeader headerWithRefreshingBlock:^{
        // 进入刷新状态后会自动调用这个block
    }];
    // 或
    // 设置回调（一旦进入刷新状态，就调用 target 的 action，也就是调用 self 的 loadNewData 方法）
    self.tableView.mj_header = [MJRefreshNormalHeader headerWithRefreshingTarget:self refreshingAction:@selector(loadNewData)];
    // 马上进入刷新状态
    [self.tableView.mj_header beginRefreshing];
```

![下拉刷新01-默认](http://upload-images.jianshu.io/upload_images/2665449-a3bfaa0e6bfbc461.gif?imageMogr2/auto-orient/strip)

### 下拉刷新02-动画图片
```
    // 设置回调（一旦进入刷新状态，就调用 targett 的 action，即调用 self 的 loadNewData 方法）
    MJRefreshGifHeader *header = [MJRefreshGifHeader headerWithRefreshingTarget:self refreshingAction:@selector(loadNewData)];
    
    // 设置普通状态的动画图片
    NSArray *idleImages = @[@"图片1", @"图片2", @"图片3"];
    [header setImages:idleImages forState:MJRefreshStateIdle];
    
    // 设置即将刷新状态的动画图片（一松开就会刷新的状态）
    NSArray *pullingImages = @[@"图片1", @"图片2", @"图片3"];
    [header setImages:pullingImages forState:MJRefreshStatePulling];
    
    // 设置正在刷新状态的动画图片
    NSArray *refreshingImages = @[@"图片1", @"图片2", @"图片3"];
    [header setImages:refreshingImages forState:MJRefreshStateRefreshing];
    // 设置 header
    self.tableView.mj_header = header;
```
![下拉刷新02-动画图片](http://upload-images.jianshu.io/upload_images/2665449-392f24b00061ea83.gif?imageMogr2/auto-orient/strip)

### 下拉刷新03-隐藏时间
```
// 隐藏时间
header.lastUpdatedTimeLabel.hidden = YES;
```
![下拉刷新03-隐藏时间](http://upload-images.jianshu.io/upload_images/2665449-a3c38c436477cacc.gif?imageMogr2/auto-orient/strip)

### 下拉刷新04-隐藏状态和时间
```
// 隐藏时间
header.lastUpdatedTimeLabel.hidden = YES;

// 隐藏状态
header.stateLabel.hidden = YES;
```
![下拉刷新04-隐藏状态和时间](http://upload-images.jianshu.io/upload_images/2665449-bea00ff6a76a81fd.gif?imageMogr2/auto-orient/strip)

### 下拉刷新05-自定义文字

```
// 设置文字
[header setTitle:@"Pull down to refresh" forState:MJRefreshStateIdle];
[header setTitle:@"Release to refresh" forState:MJRefreshStatePulling];
[header setTitle:@"Loading ..." forState:MJRefreshStateRefreshing];

// 设置字体
header.stateLabel.font = [UIFont systemFontOfSize:15];
header.lastUpdatedTimeLabel.font = [UIFont systemFontOfSize:14];

// 设置颜色
header.stateLabel.textColor = [UIColor redColor];
header.lastUpdatedTimeLabel.textColor = [UIColor blueColor];
```

![下拉刷新05-自定义文字](http://upload-images.jianshu.io/upload_images/2665449-ad9e5a170567e3bb.gif?imageMogr2/auto-orient/strip)

### 下拉刷新06-自定义刷新控件

```
self.tableView.mj_header = [MJDIYHeader headerWithRefreshingTarget:self refreshingAction:@selector(loadNewData)];
// 具体实现参考 MJDIYHeader.h 和 MJDIYHeader.m
```
![下拉刷新06-自定义刷新控件](http://upload-images.jianshu.io/upload_images/2665449-02fca65a8c891905.gif?imageMogr2/auto-orient/strip)

### 上拉刷新01-默认
```
    self.tableView.mj_footer = [MJRefreshAutoNormalFooter footerWithRefreshingBlock:^{
        // 进入刷新状态后会自动调用这个 block
    }];
    // 或
    // 设置回调（一旦进入刷新状态，就调用 target 的 action，即调用 self 的 loadMoreData 方法）
    self.tableView.mj_footer = [MJRefreshAutoNormalFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
```
![上拉刷新01-默认](http://upload-images.jianshu.io/upload_images/2665449-475e06e1f29fb149.gif?imageMogr2/auto-orient/strip)

### 上拉刷新02-动画图片
```

    // 设置回调（一旦进入刷新状态，就调用 target 的 action，即调用 self 的 loadMoreData 方法）
    MJRefreshAutoGifFooter *footer = [MJRefreshAutoGifFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
    
    // 设置刷新图片
    NSArray *refreshingImages = @[@"图片1", @"图片2", @"图片3"];
    [footer setImages:refreshingImages forState:MJRefreshStateRefreshing];
    
    // 设置尾部
    self.tableView.mj_footer = footer;
```
![上拉刷新02-动画图片](http://upload-images.jianshu.io/upload_images/2665449-15a83363eff10b1e.gif?imageMogr2/auto-orient/strip)

### 上拉刷新03-隐藏刷新状态的文字
```
// 隐藏刷新状态的文字
footer.refreshingTitleHidden = YES;
// 如果没有上面的方法，就用footer.stateLabel.hidden = YES;
```
![上拉刷新03-隐藏刷新状态的文字](http://upload-images.jianshu.io/upload_images/2665449-7dcd46bd73d0423f.gif?imageMogr2/auto-orient/strip)

### 上拉刷新04-全部加载完毕
```
    // 变为没有更多数据的状态
    [footer endRefreshingWithNoMoreData];
```
![上拉刷新04-全部加载完毕](http://upload-images.jianshu.io/upload_images/2665449-3f0e9a88feb33219.gif?imageMogr2/auto-orient/strip)

### 上拉刷新05-自定义文字
```
// 设置文字
[footer setTitle:@"Click or drag up to refresh" forState:MJRefreshStateIdle];
[footer setTitle:@"Loading more ..." forState:MJRefreshStateRefreshing];
[footer setTitle:@"No more data" forState:MJRefreshStateNoMoreData];

// 设置字体
footer.stateLabel.font = [UIFont systemFontOfSize:17];

// 设置颜色
footer.stateLabel.textColor = [UIColor blueColor];
```
![上拉刷新05-自定义文字](http://upload-images.jianshu.io/upload_images/2665449-f9383ff9ab48a6b2.gif?imageMogr2/auto-orient/strip)

### 上拉刷新06-加载后隐藏
```
// 隐藏当前的上拉刷新控件
self.tableView.mj_footer.hidden = YES;
```
![上拉刷新06-加载后隐藏](http://upload-images.jianshu.io/upload_images/2665449-44824bc808564096.gif?imageMogr2/auto-orient/strip)

### 上拉刷新07-自动回弹的上拉01

```
self.tableView.mj_footer = [MJRefreshBackNormalFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
```
![上拉刷新07-自动回弹的上拉01](http://upload-images.jianshu.io/upload_images/2665449-ab778a9f00a22a4f.gif?imageMogr2/auto-orient/strip)

### 上拉刷新08-自动回弹的上拉02
```

    MJRefreshBackGifFooter *footer = [MJRefreshBackGifFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
    
    // 设置普通状态的动画图片
    NSArray *idleImages = @[@"图片1", @"图片2", @"图片3"];
    [footer setImages:idleImages forState:MJRefreshStateIdle];
    
    // 设置即将刷新状态的动画图片（一松开就会刷新的状态）
    NSArray *pullingImages = @[@"图片1", @"图片2", @"图片3"];

    [footer setImages:pullingImages forState:MJRefreshStatePulling];
    
    // 设置正在刷新状态的动画图片
    NSArray *refreshingImages = @[@"图片1", @"图片2", @"图片3"];
    [footer setImages:refreshingImages forState:MJRefreshStateRefreshing];
    
    // 设置尾部
    self.tableView.mj_footer = footer;
```
![上拉刷新08-自动回弹的上拉02](http://upload-images.jianshu.io/upload_images/2665449-7ff75c5dc70d3d33.gif?imageMogr2/auto-orient/strip)

### 上拉刷新09-自定义刷新控件(自动刷新)
```
self.tableView.mj_footer = [MJDIYAutoFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
// 具体实现参考MJDIYAutoFooter.h和MJDIYAutoFooter.m
```
![上拉刷新09-自定义刷新控件(自动刷新)](http://upload-images.jianshu.io/upload_images/2665449-504544818173e40a.gif?imageMogr2/auto-orient/strip)

### 上拉刷新10-自定义刷新控件(自动回弹)
```
self.tableView.mj_footer = [MJDIYBackFooter footerWithRefreshingTarget:self refreshingAction:@selector(loadMoreData)];
// 具体实现参考MJDIYBackFooter.h和MJDIYBackFooter.m
```
![上拉刷新10-自定义刷新控件(自动回弹)](http://upload-images.jianshu.io/upload_images/2665449-39f29fe70a310b80.gif?imageMogr2/auto-orient/strip)

### UICollectionView01-上下拉刷新
```
// 下拉刷新
self.collectionView.mj_header = [MJRefreshNormalHeader headerWithRefreshingBlock:^{
  // 进入刷新状态后会自动调用这个block
}];
// 上拉刷新
self.collectionView.mj_footer = [MJRefreshAutoNormalFooter footerWithRefreshingBlock:^{
  // 进入刷新状态后会自动调用这个block
}];
```
![UICollectionView01-上下拉刷新](http://upload-images.jianshu.io/upload_images/2665449-0d4fd9a97ffb9513.gif?imageMogr2/auto-orient/strip)

### UIWebView01-下拉刷新
```
// 添加下拉刷新控件
self.webView.scrollView.mj_header = [MJRefreshNormalHeader headerWithRefreshingBlock:^{
   // 进入刷新状态后会自动调用这个block
}];
```
![UIWebView01-下拉刷新](http://upload-images.jianshu.io/upload_images/2665449-ee3c092adb2b9561.gif?imageMogr2/auto-orient/strip)

- *over*
以上是 MJRefresh 的具体用法，有兴趣的朋友还可以参考下[MJRefresh/README.md](https://github.com/CoderMJLee/MJRefresh/blob/master/README.md)
