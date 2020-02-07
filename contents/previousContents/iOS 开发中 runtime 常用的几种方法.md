# iOS 开发中 runtime 常用的几种方法

> 公司项目中用了一些 runtime 相关的知识, 初看时有些蒙, 虽然用的并不多, 但还是想着系统的把 runtime 相关的常用方法整理一下, 自己以后用着方便, 也希望对看到的朋友有所帮助.

## 一、runtime 简介
runtime 简称运行时，是系统在运行的时候的一些机制，其中最主要的是消息机制。它是一套比较底层的纯 C 语言 API, 属于一个 C 语言库，包含了很多底层的 C 语言 API。我们平时编写的 OC 代码，在程序运行过程时，其实最终都是转成了 runtime 的 C 语言代码。如下所示:

```objc
// OC代码:
[Person coding];

//运行时 runtime 会将它转化成 C 语言的代码:
objc_msgSend(Person, @selector(coding));
```

## 二、相关函数

```objc
// 遍历某个类所有的成员变量
class_copyIvarList

// 遍历某个类所有的方法
class_copyMethodList

// 获取指定名称的成员变量
class_getInstanceVariable

// 获取成员变量名
ivar_getName

// 获取成员变量类型编码
ivar_getTypeEncoding

// 获取某个对象成员变量的值
object_getIvar

// 设置某个对象成员变量的值
object_setIvar

// 给对象发送消息
objc_msgSend
```
## 三、相关应用

- 更改属性值
- 动态添加属性
- 动态添加方法
- 交换方法的实现
- 拦截并替换方法
- 在方法上增加额外功能
- 归档解档
- 字典转模型

以上八种用法用代码都实现了, 文末会贴出代码地址.
![runtime](https://upload-images.jianshu.io/upload_images/2665449-964abd831b594da7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 四、代码实现

要使用runtime，要先引入头文件`#import <objc/runtime.h>`
#### 4.1 更改属性值

用 runtime 修改一个对象的属性值
```
    unsigned int count = 0;
    // 动态获取类中的所有属性(包括私有)
    Ivar *ivar = class_copyIvarList(_person.class, &count);
    // 遍历属性找到对应字段
    for (int i = 0; i < count; i ++) {
        Ivar tempIvar = ivar[i];
        const char *varChar = ivar_getName(tempIvar);
        NSString *varString = [NSString stringWithUTF8String:varChar];
        if ([varString isEqualToString:@"_name"]) {
            // 修改对应的字段值
            object_setIvar(_person, tempIvar, @"更改属性值成功");
            break;
        }
    }
```

#### 4.2 动态添加属性

用 runtime 为一个类添加属性, iOS 分类里一般会这样用, 我们建立一个分类, `NSObject+NNAddAttribute.h`, 并添加以下代码:
```
- (void)setName:(NSString *)name {
    objc_setAssociatedObject(self, @"name", name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (NSString *)name {
    return objc_getAssociatedObject(self, @"name");
}
```

这样只要引用 `NSObject+NNAddAttribute.h`, 用 NSObject 创建的对象就会有一个 name 属性, 我们可以直接这样写:
```
    NSObject *person = [NSObject new];
    person.name = @"以梦为马";
```

#### 4.3 动态添加方法

person 类中没有 `coding ` 方法，我们用 runtime 给 person 类添加了一个名字叫 `coding` 的方法，最终再调用coding方法做出相应. 下面代码的几个参数需要注意一下:


```
- (void)buttonClick:(UIButton *)sender {
    /*
     动态添加 coding 方法
     (IMP)codingOC 意思是 codingOC 的地址指针;
     "v@:" 意思是，v 代表无返回值 void，如果是 i 则代表 int；@代表 id sel; : 代表 SEL _cmd;
     “v@:@@” 意思是，两个参数的没有返回值。
     */
    class_addMethod([_person class], @selector(coding), (IMP)codingOC, "v@:");
    // 调用 coding 方法响应事件
    if ([_person respondsToSelector:@selector(coding)]) {
        [_person performSelector:@selector(coding)];
        self.testLabelText = @"添加方法成功";
    } else {
        self.testLabelText = @"添加方法失败";
    }
}

// 编写 codingOC 的实现
void codingOC(id self,SEL _cmd) {
    NSLog(@"添加方法成功");
}
```


#### 4.4 交换方法的实现

某个类有两个方法, 比如 person 类有两个方法, coding 方法与 eating 方法, 我们用 runtime 交换一下这两个方法, 就会出现这样的情况, 当我们调用 coding 的时候, 执行的是 eating, 当我们调用 eating 的时候, 执行的是 coding, 如下面的动态效果图.

```
    Method oriMethod = class_getInstanceMethod(_person.class, @selector(coding));
    Method curMethod = class_getInstanceMethod(_person.class, @selector(eating));
    method_exchangeImplementations(oriMethod, curMethod);
```

![交换方法的实现](https://upload-images.jianshu.io/upload_images/2665449-8683e26cc60e7612.gif?imageMogr2/auto-orient/strip)

#### 4.5 拦截并替换方法

这个功能和上面的其实有些类似, 拦截并替换方法可以拦截并替换同一个类的, 也可以在两个类之间进行, 我这里用了两个不同的类, 下面是简单的代码实现.

```
    _person = [NNPerson new];
    _library = [NNLibrary new];
    self.testLabelText = [_library libraryMethod];
    Method oriMethod = class_getInstanceMethod(_person.class, @selector(changeMethod));
    Method curMethod = class_getInstanceMethod(_library.class, @selector(libraryMethod));
    method_exchangeImplementations(oriMethod, curMethod);
```

#### 4.6 在方法上增加额外功能

这个使用场景还是挺多的, 比如我们需要记录 APP 中某一个按钮的点击次数, 这个时候我们便可以利用 runtime 来实现这个功能. 我这里写了个 UIButton 的子类, 然后在 `+ (void)load` 中用 runtime 给它增加了一个功能, 核心代码及实现效果图如下:

```
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method oriMethod = class_getInstanceMethod(self.class, @selector(sendAction:to:forEvent:));
        Method cusMethod = class_getInstanceMethod(self.class, @selector(customSendAction:to:forEvent:));
        // 判断自定义的方法是否实现, 避免崩溃
        BOOL addSuccess = class_addMethod(self.class, @selector(sendAction:to:forEvent:), method_getImplementation(cusMethod), method_getTypeEncoding(cusMethod));
        if (addSuccess) {
            // 没有实现, 将源方法的实现替换到交换方法的实现
            class_replaceMethod(self.class, @selector(customSendAction:to:forEvent:), method_getImplementation(oriMethod), method_getTypeEncoding(oriMethod));
        } else {
            // 已经实现, 直接交换方法
            method_exchangeImplementations(oriMethod, cusMethod);
        }
    });
}
```

![在方法上增加额外功能](https://upload-images.jianshu.io/upload_images/2665449-745774fb35fbf332.gif?imageMogr2/auto-orient/strip)


#### 4.7 归档解档

当我们使用 NSCoding 进行归档及解档时, 如果不用 runtime, 那么不管模型里面有多少属性, 我们都需要对其实现一遍 `encodeObject` 和 `decodeObjectForKey` 方法, 如果模型里面有 10000 个属性, 那么我们就需要写 10000 句`encodeObject` 和 `decodeObjectForKey` 方法, 这个时候用 runtime, 便可以充分体验其好处(以下只是核心代码, 具体代码请见 demo).
```
- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int count = 0;
    // 获取类中所有属性
    Ivar *ivars = class_copyIvarList(self.class, &count);
    // 遍历属性
    for (int i = 0; i < count; i ++) {
        // 取出 i 位置对应的属性
        Ivar ivar = ivars[i];
        // 查看属性
        const char *name = ivar_getName(ivar);
        NSString *key = [NSString stringWithUTF8String:name];
        // 利用 KVC 进行取值，根据属性名称获取对应的值
        id value = [self valueForKey:key];
        [aCoder encodeObject:value forKey:key];
    }
    free(ivars);
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int count = 0;
        // 获取类中所有属性
        Ivar *ivars = class_copyIvarList(self.class, &count);
        // 遍历属性
        for (int i = 0; i < count; i ++) {
            // 取出 i 位置对应的属性
            Ivar ivar = ivars[i];
            // 查看属性
            const char *name = ivar_getName(ivar);
            NSString *key = [NSString stringWithUTF8String:name];
            // 进行解档取值
            id value = [aDecoder decodeObjectForKey:key];
            // 利用 KVC 对属性赋值
            [self setValue:value forKey:key];
        }
    }
    return self;
}
```


#### 4.8 字典转模型

字典转模型我们通常用的都是第三方, MJExtension, YYModel 等, 但也有必要了解一下其实现方式: 遍历模型中的所有属性，根据模型的属性名，去字典中查找对应的 key，取出对应的值，给模型的属性赋值。

```
/** 字典转模型 **/
+ (instancetype)modelWithDict:(NSDictionary *)dict {
    id objc = [[self alloc] init];
    unsigned int count = 0;
    // 获取成员属性数组
    Ivar *ivarList = class_copyIvarList(self, &count);
    // 遍历所有的成员属性名
    for (int i = 0; i < count; i ++) {
        // 获取成员属性
        Ivar ivar = ivarList[i];
        // 获取成员属性名
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(ivar)];
        NSString *key = [ivarName substringFromIndex:1];
        // 从字典中取出对应 value 给模型属性赋值
        id value = dict[key];
        // 获取成员属性类型
        NSString *ivarType = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
        // 判断 value 是不是字典
        if ([value isKindOfClass:[NSDictionary class]]) {
            ivarType = [ivarType stringByReplacingOccurrencesOfString:@"@" withString:@""];
            ivarType = [ivarType stringByReplacingOccurrencesOfString:@"\"" withString:@""];
            Class modalClass = NSClassFromString(ivarType);
            // 字典转模型
            if (modalClass) {
                // 字典转模型
                value = [modalClass modelWithDict:value];
            }
        }
        if ([value isKindOfClass:[NSArray class]]) {
            // 判断对应类有没有实现字典数组转模型数组的协议
            if ([self respondsToSelector:@selector(arrayContainModelClass)]) {
                // 转换成id类型，就能调用任何对象的方法
                id idSelf = self;
                // 获取数组中字典对应的模型
                NSString *type = [idSelf arrayContainModelClass][key];
                // 生成模型
                Class classModel = NSClassFromString(type);
                NSMutableArray *arrM = [NSMutableArray array];
                // 遍历字典数组，生成模型数组
                for (NSDictionary *dict in value) {
                    // 字典转模型
                    id model =  [classModel modelWithDict:dict];
                    [arrM addObject:model];
                }
                // 把模型数组赋值给value
                value = arrM;
            }
        }
        // KVC 字典转模型
        if (value) {
            [objc setValue:value forKey:key];
        }
    }
    return objc;
}
```

--- 

上面的所有代码都可以在这里下载:  [runtime 练习: NNRuntimeTest](https://github.com/liuzhongning/NNLearn/tree/master/002.%20NNRuntimeTest)
