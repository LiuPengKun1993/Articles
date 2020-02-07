# iOS开发 - 链式编程思想

> 因为有 Masory 以及 Snapkit 这些知名开源库的存在，相信很多 iOS 开发者对链式编程都不会太陌生，先来看下面这句代码：

```
[imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.bottom.left.right.equalTo(self.view).insets(UIEdgeInsetsMake(250, 100, 250, 100));
    }];
```

这句代码就属于链式编程，而且 Masonry 框架本身也是通过链式语法对 NSLayoutConstraints 进行的封装。对 Masonry 感兴趣的童鞋可以读读这篇文章 [iOS开发之源码解析 - Masonry](http://www.jianshu.com/p/2fe9568fd3f5)。

####链式编程思想：
所谓链式编程就是通过点（.）将多个操作链接在一起成为一句代码，使代码更加紧凑，也提高了代码的可读性（如上面那句代码）。

####链式编程特点：
- 方法的返回值是 block，block 中必须有一个返回值，通常返回它本身，也可以是处理后的数据或对象。
- 返回值中的 block 具备两个功能，第一：可以作为类的属性被'点'出来。第二：可以当作函数直接调用。
- 通常会通过调用一个函数来给属性赋值，在函数内部封装赋值的语句，也可以加入一些判断逻辑等。

####链式编程练习
我这里写了一个 UILabel 的扩展类用来练习链式编程，叫做 `UILabel+NNCategory`。利用这个分类创建 UILabel，并为其设置`frame，text，font，textColor，backgroundColor`等属性，代码如下：

```
    [UILabel addToView:self.view createLabel:^(UILabel *label) {
        label.nn_frame(50, 200, 300, 100).nn_text(@"NNTreasure").nn_fontSize(50).nn_textColor([UIColor redColor]).nn_backgroundColorRGB(224, 224, 224, 1).nn_textAlignment(NSTextAlignmentCenter);
    }];
```
细心观察上面的代码，我们可以猜出 `nn_frame()`，`nn_text()`，`nn_fontSize()` 这些都是属性，不过为什么它们后面都带有括号呢？这是**因为这些属性都是 `block` 类型**。那么为什么它们可以点出来呢？即为什么可以 `label.nn_frame().nn_text().nn_fontSize()` 这么用？简单来说，就是**因为这些 `block` 类型的属性都带有返回值**！


下面是分类 `UILabel+NNCategory` 的详细实现过程

- 在 UILabel+NNCategory.h 中声明一些属性

```
@property (nonatomic, copy, readonly) UILabel *(^nn_text)(NSString *);
@property (nonatomic, copy, readonly) UILabel *(^nn_frame)(CGFloat, CGFloat, CGFloat, CGFloat);
@property (nonatomic, copy, readonly) UILabel *(^nn_attributedText)(NSAttributedString *);
@property (nonatomic, copy, readonly) UILabel *(^nn_textAlignment)(NSTextAlignment);
@property (nonatomic, copy, readonly) UILabel *(^nn_textColor)(UIColor *);
@property (nonatomic, copy, readonly) UILabel *(^nn_textColorRGB)(CGFloat, CGFloat, CGFloat, CGFloat);
@property (nonatomic, copy, readonly) UILabel *(^nn_backgroundColor)(UIColor *);
@property (nonatomic, copy, readonly) UILabel *(^nn_backgroundColorRGB)(CGFloat, CGFloat, CGFloat, CGFloat);
@property (nonatomic, copy, readonly) UILabel *(^nn_highlightTextColor)(UIColor *);
@property (nonatomic, copy, readonly) UILabel *(^nn_highlight)(BOOL);
@property (nonatomic, copy, readonly) UILabel *(^nn_enable)(BOOL);
@property (nonatomic, copy, readonly) UILabel *(^nn_font)(UIFont *);
@property (nonatomic, copy, readonly) UILabel *(^nn_fontSize)(NSInteger);
@property (nonatomic, copy, readonly) UILabel *(^nn_shadowColor)(UIColor *);
@property (nonatomic, copy, readonly) UILabel *(^nn_shadowOffset)(CGSize);
@property (nonatomic, copy, readonly) UILabel *(^nn_lineBreakMode)(NSLineBreakMode);
@property (nonatomic, copy, readonly) UILabel *(^nn_numberOfLine)(NSInteger);
@property (nonatomic, copy, readonly) UILabel *(^nn_adjustsFontSizeToFitWidth)(BOOL);
@property (nonatomic, copy, readonly) UILabel *(^nn_baselineAdjust)(UIBaselineAdjustment);
@property (nonatomic, copy, readonly) UILabel *(^nn_drawText)(CGRect);
```

我们在 UILabel 分类里面声明了这些属性，所以接下来只要是 UILabel 类型都可以点出来这些属性。这里也可以看出这些属性都是 `block` 类型。我们抽出一句代码简单说明一下：`@property (nonatomic, copy, readonly) UILabel *(^nn_text)(NSString *);`，这句代码声明了一个 block(^) 原型，名字叫做 nn_text，包含了一个 NSString 类型的参数，返回值是 UILabel 类型。

- 在 UILabel+NNCategory.m 中实现

```
- (UILabel *(^)(NSString *))nn_text {
    return ^(NSString *text) {
        self.text = text;
        return self;
    };
}

- (UILabel *(^)(CGFloat, CGFloat, CGFloat, CGFloat))nn_frame {
    return ^(CGFloat X, CGFloat Y, CGFloat W, CGFloat H) {
        self.frame = CGRectMake(X, Y, W, H);
        return self;
    };
}

- (UILabel *(^)(NSAttributedString *))nn_attributedText {
    return ^(NSAttributedString *attributedText) {
        self.attributedText = attributedText;
        return self;
    };
}

- (UILabel *(^)(NSTextAlignment))nn_textAlignment {
    return ^(NSTextAlignment textAlignment) {
        self.textAlignment = textAlignment;
        return self;
    };
}

- (UILabel *(^)(UIColor *))nn_textColor {
    return ^(UIColor *textColor) {
        self.textColor = textColor;
        return self;
    };
}

- (UILabel *(^)(CGFloat, CGFloat, CGFloat, CGFloat))nn_textColorRGB {
    return ^(CGFloat r, CGFloat g, CGFloat b, CGFloat a){
        self.textColor = [UIColor colorWithRed:r / 255.0 green:g / 255.0 blue:g / 255.0 alpha:a];
        return self;
    };
}

- (UILabel *(^)(UIColor *))nn_backgroundColor {
    return ^(UIColor *backgroundColor) {
        self.backgroundColor = backgroundColor;
        return self;
    };
}

- (UILabel *(^)(CGFloat, CGFloat, CGFloat, CGFloat))nn_backgroundColorRGB {
    return ^(CGFloat r, CGFloat g, CGFloat b, CGFloat a) {
        self.backgroundColor = [UIColor colorWithRed:r / 255.0 green:g / 255.0 blue:g / 255.0 alpha:a];
        return self;
    };
}

- (UILabel *(^)(UIColor *))nn_highlightTextColor {
    return ^(UIColor *color) {
        self.highlightedTextColor = color;
        return self;
    };
}

- (UILabel *(^)(BOOL))nn_highlight {
    return ^(BOOL isHighlighted) {
        self.highlighted = isHighlighted;
        return self;
    };
}

- (UILabel *(^)(BOOL))nn_enable {
    return ^(BOOL isEnabled) {
        self.enabled = isEnabled;
        return self;
    };
}

- (UILabel *(^)(UIFont *))nn_font {
    return ^(UIFont *font) {
        self.font = font;
        return self;
    };
}

- (UILabel *(^)(NSInteger))nn_fontSize {
    return ^(NSInteger size) {
        self.font = [UIFont systemFontOfSize:size];
        return self;
    };
}

- (UILabel *(^)(UIColor *))nn_shadowColor {
    return ^(UIColor *shadowColor) {
        self.shadowColor = shadowColor;
        return self;
    };
}

- (UILabel *(^)(CGSize))nn_shadowOffset {
    return ^(CGSize size) {
        self.shadowOffset = size;
        return self;
    };
}

- (UILabel *(^)(NSLineBreakMode))nn_lineBreakMode {
    return ^(NSLineBreakMode mode) {
        self.lineBreakMode = mode;
        return self;
    };
}

- (UILabel *(^)(NSInteger))nn_numberOfLine {
    return ^(NSInteger number) {
        self.numberOfLines = number;
        return self;
    };
}

- (UILabel *(^)(BOOL))nn_adjustsFontSizeToFitWidth {
    return ^(BOOL b) {
        self.adjustsFontSizeToFitWidth = b;
        return self;
    };
}

- (UILabel *(^)(UIBaselineAdjustment))nn_baselineAdjust {
    return ^(UIBaselineAdjustment adjustment) {
        self.baselineAdjustment = adjustment;
        return self;
    };
}

- (UILabel *(^)(CGRect))nn_drawText {
    return ^(CGRect rect) {
        [self drawTextInRect:rect];
        return self;
    };
}
```
`UILabel+NNCategory.m` 中的这些是 `UILabel+NNCategory.h` 文件中属性的 getter 方法。我们抽出一个简单说明一下

```
- (UILabel *(^)(NSString *))nn_text {
	// 返回临时变量的 block
    return ^(NSString *text) {
    	// block 执行的一些功能
        self.text = text;
        // block 执行完毕的返回值
        return self;
    };
}
```
这个属性的类型是 block，具体是 `UILabel *(^nn_text)(NSString *)` 类型，需要说明的是，我们用 block 并不是为了返回 block 对象本身，而是为了在 block 内部执行一些操作，所以我们在属性的 getter 方法中首先返回一个临时的 block 对象，主要是为了在 block 内部返回 UILabel 类型的对象。具体请看上面代码块中的一些注释。

**这时我们还需要定义一个方法方便外界调用，这个方法主要有两个功能，第一是用来创建 UILabel ，第二是为其“点”出各种属性。**

- 在 UILabel+NNCategory.h 中声明一个方法

```
+ (instancetype)addToView:(UIView *)superView createLabel:(void(^)(UILabel *label))block;
```
这里是模仿 Masonry，我们通常用 Masonry 加约束时首先会调用这个方法：`- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *make))block`，

```
- (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block {
    self.translatesAutoresizingMaskIntoConstraints = NO;
    MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView:self];
    block(constraintMaker);
    return [constraintMaker install];
}
```
我们用 Masonry 加约束时调用这个方法，是因为 Masonry 在这个方法内部早已做好一些所需要的操作。所以接下来我们也要在 UILabel+NNCategory.m 中做些事情。

- 在 UILabel+NNCategory.m 中实现这个方法

```
+ (instancetype)addToView:(UIView *)superView createLabel:(void(^)(UILabel *label))block {
	// 创建 UILabel
    UILabel *label = [[UILabel alloc] init];
    // 把 UILabel 添加到传过来的 superView 上
    [superView addSubview:label];
    // 通过 block 进行回调
    if (block) block(label);
    return label;
}
```

上面这个方法首先初始化一个 UILabel 对象，接着把 UILabel 添加到传过来的 superView 上，然后对 block 进行回调，执行 block，把外界对 UILabel 设置的属性添加上去，最后返回 UILabel 对象。

---

这就是一些简单的链式编程思维，外界调用的时候只需如下代码即可：

```
    [UILabel addToView:self.view createLabel:^(UILabel *label) {
        label.nn_frame(50, 200, 300, 100).nn_text(@"NNTreasure").nn_fontSize(50).nn_textColor([UIColor redColor]).nn_backgroundColorRGB(224, 224, 224, 1).nn_textAlignment(NSTextAlignmentCenter);
    }];
```

效果图如下：



![链式编程](http://upload-images.jianshu.io/upload_images/2665449-98cb658222344c24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)