# iOS 开发中导航栏渐变的两种方法

> 很多 APP 都有导航栏颜色渐变的效果，各位开发者也都八仙过海各显神通，各有各的方法，我这里提供两种思路：**第一种是通过设置 navigationBar 的 alpha；第二种是在 navigationBar 上加入一个 view，再设置 view 的 alpha。**



![导航栏渐变](http://upload-images.jianshu.io/upload_images/2665449-d8f9e18b30189a3b.gif?imageMogr2/auto-orient/strip)
两种方法的核心代码：

#### 一、设置 navigationBar 的 alpha
向上滑和向下滑时分别对 navigationBar 的透明度进行设置。

```
#pragma mark - UIScrollViewDelegate
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    
    self.currentPostion = scrollView.contentOffset.y;
    
    if (self.currentPostion > 0) {
        if (self.currentPostion - self.lastPosition >= 0) {
                if (self.topBool) {
                    self.topBool = NO;
                    self.bottomBool = YES;
                self.stopPosition = self.currentPostion + 64;
            }
            self.lastPosition = self.currentPostion;
            self.navigationController.navigationBar.alpha = 1 - self.currentPostion / 500;
        } else {
            if (self.bottomBool) {
                self.bottomBool = NO;
                self.topBool = YES;
                self.stopPosition = self.currentPostion + 64;
            }
            self.lastPosition = self.currentPostion;
            self.navigationController.navigationBar.alpha = (self.stopPosition - self.currentPostion) / 200;
        }
    }
}
```
这里要说明一点，**如果你不希望自己 navigationBar 上的按钮也被改变透明度，可以通过自定义 UINavigationController 实现。用系统的 navigationBar 时，navigationBar上的按钮的透明度也会被更改。**

---
#### 二、在 navigationBar 上加入一个 view
在 navigationBar 上加一个 view，然后设置 view 的 alpha。

```
CGSize customSize = self.navigationController.navigationBar.frame.size;
    self.customView = [[UIView alloc] initWithFrame:CGRectMake(0, -20, customSize.width, customSize.height + 20)];
    self.customView.backgroundColor = [UIColor orangeColor];
    self.customView.userInteractionEnabled = NO;
    [self.navigationController.navigationBar insertSubview: self.customView atIndex:0];
```
这里也要说明一点，**View 的 userInteractionEnabled 属性要设置为 NO，不然会出现问题。**


UIScrollViewDelegate 的这个代理方法与上面的一样。

```
#pragma mark - UIScrollViewDelegate
- (void)scrollViewDidScroll:(UIScrollView *)scrollView {
    
    self.currentPostion = scrollView.contentOffset.y;
    
    if (self.currentPostion > 0) {
        if (self.currentPostion - self.lastPosition >= 0) {
            if (self.topBool) {
                self.topBool = NO;
                self.bottomBool = YES;
                self.stopPosition = self.currentPostion + 64;
            }
            self.lastPosition = self.currentPostion;
            self.customView.alpha = 1 - self.currentPostion / 500;
        } else {
            if (self.bottomBool) {
                self.bottomBool = NO;
                self.topBool = YES;
                self.stopPosition = self.currentPostion + 64;
            }
            self.lastPosition = self.currentPostion;
            self.customView.alpha = (self.stopPosition - self.currentPostion) / 200;
        }
    }
}
```

上面的两种方法都可以实现导航栏渐变，只是思路不同，一个是改变系统的导航栏，一个是在导航栏上插入 View，另外还可以自定义导航控制器这个类来实现，各位道友可以试试哦！

####[代码传送门](https://github.com/liuzhongning/NNJaneBook) 欢迎各位道友交流或指出错误！
