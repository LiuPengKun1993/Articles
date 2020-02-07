# iOS开发之 - MD5加密

> MD5（消息摘要算法第五版）是计算机安全领域广泛使用的一种散列函数，用以提供消息的完整性保护。

- MD5特点：
1、压缩性：任意长度的数据，算出的MD5值长度都是固定的。
2、容易计算：从原数据计算出MD5值很容易。
3、抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。
4、强抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。

- 事先介绍一个概念，因为下边有用到👇
加盐：所谓加盐， 就是在原本需要加密的信息基础上，糅入其它内容salt。

- 接下来是一些代码的实现，方便起见，这里创建了一个分类NSString+MD5。

分类 NSString+MD5.h 中的代码如下：

```
#import <Foundation/Foundation.h>

@interface NSString (MD5)
/** 直接加密 */
- (NSString *)MD5_32BIT;
/** 加盐 */
- (NSString *)MD5_saltStr:(NSString *)saltStr;
/** 多次加密 */
- (NSString *)MD5_againStr:(NSString *)againStr;
/** 乱序加密 */
- (NSString *)MD5_Chaos:(NSString *)chaosStr;

@end
```

这里是分类 NSString+MD5.m 中的代码，需要导入头文件 #import <CommonCrypto/CommonDigest.h>：

```
#import "NSString+MD5.h"
#import <CommonCrypto/CommonDigest.h>
#define salt @"qwertyuiop"

@implementation NSString (MD5)


// 直接加密
- (NSString *)MD5_32BIT {
    
    const char *cStr = [self UTF8String];
    
    unsigned char digest[CC_MD5_DIGEST_LENGTH];
    
    CC_MD5( cStr, (CC_LONG)self.length, digest );
    
    NSMutableString *result = [NSMutableString stringWithCapacity:CC_MD5_DIGEST_LENGTH * 2];
    
    for(int i = 0; i < CC_MD5_DIGEST_LENGTH; i++)
        [result appendFormat:@"%02X", digest[i]];
    
    return result;
}

// 加盐
- (NSString *)MD5_saltStr:(NSString *)saltStr {
    
    saltStr = [saltStr stringByAppendingString:salt];
    
    NSString *result =[saltStr MD5_32BIT];
    
    return result;
}

// 多次加密
- (NSString *)MD5_againStr:(NSString *)againStr {
    
    NSString *result = [againStr MD5_32BIT];
    
    result = [result MD5_32BIT];
    
    return result;
}

// 乱序加密
- (NSString *)MD5_Chaos:(NSString *)chaosStr {
    
    NSString *result =[chaosStr MD5_32BIT];
    
    NSString *header =[result substringFromIndex:2];
    
    NSString *footer=[result substringFromIndex:2];
    
    result =[footer stringByAppendingString:header];
    
    return result;
    
}

@end
```

接着在控制器中调用：

```
#import "ViewController.h"
#import "NSString+MD5.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString *string = @"10.24";
    // 调用 MD5 加密并打印结果
    NSLog(@"MD5Result = %@", [string MD5_32BIT]);
    NSLog(@"MD5_saltResult = %@", [string MD5_saltStr:string]);
    NSLog(@"MD5_againResult = %@", [string MD5_againStr:string]);
    NSLog(@"chaosResult = %@", [string MD5_Chaos:string]);
}
```

这里是打印结果：
```
2016-10-26 08:15:49.616 md5[92278:6185546] MD5Result = F7F942B87C2D0B62FC27AD465CF0D05B
2016-10-26 08:15:49.616 md5[92278:6185546] MD5_saltResult = 1C2140C07ADD1E334896D499E8C4968C
2016-10-26 08:15:49.617 md5[92278:6185546] MD5_againResult = 01AE5309B5A8E672AE4D427FE987E06A
2016-10-26 08:15:49.617 md5[92278:6185546] chaosResult = F942B87C2D0B62FC27AD465CF0D05BF942B87C2D0B62FC27AD465CF0D05B
```

---
总结：大家应该都知道有一个MD5破解网站，名字叫做 [md5在线解密加密](http://www.cmd5.com)，因此我们在使用MD5加密的时候，最好使用多次加密，加盐，乱序加密等，不然会很容易被破解。至于其它加密，等以后用到了会继续整理出来，如果文章有什么不足之处，还请朋友们能过指出来，谢谢大家！