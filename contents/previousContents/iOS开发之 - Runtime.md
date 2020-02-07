# iOS开发之 - Runtime

![iOS开发之 - Runtime](http://upload-images.jianshu.io/upload_images/2665449-59759f8fb21bb6a9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 想总结 Runtime 不是一天两天了，前几天利用空闲时间已经打好文稿，但觉得还有需要改进的地方，现在已经改过来了，这里贴出来与大家分享下（如果还有什么小问题，还望各位朋友指出来哈）! 下面我们就开始进入运行时 Runtime 的世界吧！

## 一、Runtime简介
Runtime简称运行时，是系统在运行的时候的一些机制，其中最主要的是消息机制。它是一套比较底层的纯 C 语言 API, 属于一个 C 语言库，包含了很多底层的 C 语言 API。我们平时编写的 OC 代码，在程序运行过程时，其实最终都是转成了 Runtime 的 C 语言代码。如下所示:

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

- 1.利用Runtime遍历模型对象的所有属性
- 2.归档与解档
- 3.字典模型互转(利用 Runtime 遍历模型对象的所有属性, 根据属性名从字典中取出对的值,设置到模型的属性上)
- 4.动态添加/修改属性、动态添加/修改/替换方法(修改Person的身高体重等；替换 coding 方法为 eating 等)

## 四、演示代码

### 演示代码一：遍历对象的属性

新建一个Person类，继承自NSObject，在.h文件中设置它的属性如下

```

@interface NNPerson : NSObject {
    NSString * _str1;
}

@property NSString * str2;

@property (nonatomic, copy) NSString *str3;

@property (nonatomic, copy) NSDictionary * dict;

@property (nonatomic, copy) NSArray *ary;

```

在 ViewController 中获取它的属性，代码如下

```

#import "ViewController.h"
#import <objc/runtime.h>
#import "NNPerson.h"

@interface ViewController ()

@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

    // 获取 NNModel 中所有的变量并且输出变量名及类型
    unsigned int count = 0;
    Ivar * ivars = class_copyIvarList([NNPerson class], &count);
    for (unsigned int i = 0; i < count; i ++) {
        Ivar ivar = ivars[i];
        NSLog(@"类型: %s -名字: %s ", ivar_getTypeEncoding(ivar), ivar_getName(ivar));
  }
    free(ivars);
}
```

### 演示代码二：归档解档

新建一个类，继承自 NSObject 。.m中的代码如下：

```
#pragma mark -归档
- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int count = 0;

    //获取类中所有属性
    Ivar *vars = class_copyIvarList([self class], &count);
    for (unsigned int i = 0; i < count; i ++) {
        Ivar var = vars[i];
        const char *name = ivar_getName(var);
        NSString *key = [NSString stringWithUTF8String:name];

        //利用 KVC 进行取值，根据属性名称获取对应的值
        id value = [self valueForKey:key];
        [aCoder encodeObject:value forKey:key];
    }
    free(vars);
}


#pragma mark -解档
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int count = 0;

        //获取类中所有属性
        Ivar *vars = class_copyIvarList([self class], &count);
        for (unsigned int i = 0; i < count; i ++) {
            Ivar var = vars[i];
            const char *name = ivar_getName(var);
            NSString *key = [NSString stringWithUTF8String:name];

            //进行解档取值
            id value = [aDecoder decodeObjectForKey:key];

             //利用 KVC 对属性赋值
            [self setValue:value forKey:key];
        }
        free(vars);
    } 
    return self;
}
```

 

ViewController中的代码：

```

- (void)viewDidLoad {
    [super viewDidLoad];
    LZNArchive *archive = [LZNArchive objectWithKeyValues:self.Mydictionary];
    NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).lastObject;
    path = [path stringByAppendingPathComponent:@"hello"];
    [NSKeyedArchiver archiveRootObject:archive toFile:path];
    LZNArchive *m = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
}

```
### 示例代码三：字典与模型互转

首先应导入一个 plist 文件，将文件中的内容读取到字典中；然后新建一个分类如: NSObject+KeyValues

```
#pragma mark -字典转模型

+(instancetype)objectWithDict:(NSDictionary *)dict {
    id objc = [[self alloc] init];

    //遍历字典中的属性
    for (NSString *key in dict.allKeys) {
        id value = dict[key];
        objc_property_t property = class_getProperty(self, key.UTF8String);
        unsigned int count = 0;
        objc_property_attribute_t *attributeList = property_copyAttributeList(property, &count);
        objc_property_attribute_t attribute = attributeList[0];
        NSString *string = [NSString stringWithUTF8String:attribute.value];
        if ([string isEqualToString:@"@\"LZNArchive\""]) {
            value = [self objectWithDict:value];
        }

        //生成 setter 方法，并用 objc_msgSend 调用
        NSString *methodName = [NSString stringWithFormat:@"set%@%@:",[key substringToIndex:1].uppercaseString,[key substringFromIndex:1]];
        SEL setter = sel_registerName(methodName.UTF8String);
        if ([objc respondsToSelector:setter]) {
            ((void (*) (id,SEL,id)) objc_msgSend) (objc,setter,value);
        }
        free(attributeList);
    }
    return objc;
}

#pragma mark -模型转字典
-(NSDictionary *)keyValuesWithObject {
    unsigned int count = 0;
    objc_property_t *propertyList = class_copyPropertyList([self class], &count);
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];

    //遍历模型中属性
    for (int i = 0; i < count; i ++) {
        objc_property_t property = propertyList[i];

        //生成 getter 方法，并用 objc_msgSend 调用
        const char *propertyName = property_getName(property);
        SEL getter = sel_registerName(propertyName);
        if ([self respondsToSelector:getter]) {
            id value = ((id (*) (id,SEL)) objc_msgSend) (self,getter);

            //判断当前属性
            if ([value isKindOfClass:[self class]] && value) {
                value = [value keyValuesWithObject];
            }
            if (value) {
                NSString *key = [NSString stringWithUTF8String:propertyName];
                [dict setObject:value forKey:key];
            }
        }
    }
    free(propertyList);
    return dict;
}
```
### 演示代码四：objc_msgSend 消息传递 
新建一个类，继承自NSObject，.m中的代码如下:

```

- (void)sayHello {
    NSLog(@"Hello!");
}

-(void)showName:(NSString *)name {
    NSLog(@"My name is %@",name);
}

-(void)showAge:(NSInteger)age {
    NSLog(@"My age is %ld", age);
}

-(float)showHeight{
    return 180.0f;
}

-(NSString *)showInformation {
    return @"Nice to meet you!";
}
```
ViewController中的代码:

```objc
LZN_msgSend *msgSend = [[LZN_msgSend alloc] init];
    ((void (*) (id, SEL)) objc_msgSend) (msgSend, sel_registerName("sayHello"));

    ((void (*) (id, SEL, NSString *)) objc_msgSend) (msgSend, sel_registerName("showName:"), @"Liu Zhong Ning");

    ((void (*) (id, SEL, NSInteger)) objc_msgSend) (msgSend, sel_registerName("showAge:"), 23);

    float f = ((float (*) (id, SEL)) objc_msgSend_fpret) (msgSend, sel_registerName("showHeight"));

    NSLog(@"and height is %.2fcm",f);

    NSString *information = ((NSString* (*) (id, SEL)) objc_msgSend) (msgSend, sel_registerName("showInformation"));

    NSLog(@"%@",information);

```
*the end*

---

**结束语：Runtime 相关的知识就整理了这么些，希望对看到的朋友们有用！另外如果 demo 有什么问题，还烦请大家告知，一起进步！谢谢O(∩_∩)O~! 另祝大家周末愉快哈！**
