# iOS开发之 - 内购

> 电商之类的 app 大多用的都是第三方支付（支付宝微信等支付），但是在某种特殊情况之下，我们却不得不使用内购。比如商品是在本 app 中使用和消耗的（游戏币游戏道具等），就一定要用内购，苹果强制的，否则会被拒绝上线。因此作为 iOS 开发者，不光要了解第三方支付，对内购也要有一定的认识！

##一、什么是内购？
内购是苹果推出的一种支付方式。说白了内购就是苹果获取利润的方式之一。如果我们使用第三方支付，只会支付少量的💰，但使用内购却要支付很多💰💰💰💰💰💰💰💰，要和苹果三七分帐（以前是四六分😢）。

##二、内购产品的类别

官方的注释写的很清楚了，内购有五种产品类别，简介如下：
- 非消耗品（Nonconsumable）
指的是在游戏中一次性购买并拥有永久访问权的物品或服务。非消耗品物品可以被用户再次下载，并且能够在用户的所有设备上使用。
- 消耗品（Consumable）
消耗品购买不可被再次下载,根据其特点,消耗品不能在用户的设备之间跨设备使用,除非自定义服务在用户的账号之间共享这些信息。
**以下三种类别在iBooks中使用，目前iBooks不支持大陆市场**
- 免费订阅（Free subscriptions）
- 自动续费订阅（Auto-renewing subscriptions）
- 非自动续费订阅（Nonrenewing subscriptions）

##三、苹果官方的内购流程

![内购流程](http://upload-images.jianshu.io/upload_images/2665449-0614b70007393038.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 白话文如下⬇️


1. 程序向服务器发送请求，获得一份产品列表。
2. 服务器返回包含产品标识符的列表。
3. 程序向App Store发送请求，得到产品的信息。
4. App Store返回产品信息。
5. 程序把返回的产品信息显示给用户（App的store界面）
6. 用户选择某个产品
7. 程序向App Store发送支付请求
8. App Store处理支付请求并返回交易完成信息。
9. 程序从信息中获得数据，并发送至服务器。
10. 服务器纪录数据，并进行审(我们的)查。
11. 服务器将数据发给App Store来验证该交易的有效性。
12. App Store对收到的数据进行解析，返回该数据和说明其是否有效的标识。
13. 服务器读取返回的数据，确定用户购买的内容。
14. 服务器将购买的内容传递给程序。

##四、代码实现
4.1 项目的Bundle identifier需要与您申请AppID时填写的bundleID一致，不然会无法请求到商品信息。

![Bundle identifier](http://upload-images.jianshu.io/upload_images/2665449-4a5e5e9889d10179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.2要使用内购,需要导入 StoreKit 依赖库
![入StoreKit](http://upload-images.jianshu.io/upload_images/2665449-c68598a75f514fda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.3代码部分

```
#import "UITableController.h"
#import <StoreKit/StoreKit.h>

@interface UITableController () <SKProductsRequestDelegate, UITableViewDelegate, UITableViewDataSource, SKPaymentTransactionObserver>

/** 所有商品 */
@property (nonatomic, strong) NSArray *products;
@end

@implementation UITableController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self requestProducts];

}

// 请求可卖商品
- (void)requestProducts {
    
    // 1.从服务器请求所有商品的ProductIds
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"iapdemo.plist" ofType:nil];
    NSArray *productArray = [NSArray arrayWithContentsOfFile:filePath];
    
    // 2.获取productId
    NSArray *productIdArray = [productArray valueForKeyPath:@"productId"];
    
    // 3.请求可卖的商品
    NSSet *productIdSet = [NSSet setWithArray:productIdArray];
    SKProductsRequest *request = [[SKProductsRequest alloc] initWithProductIdentifiers:productIdSet];
    request.delegate = self;
    [request start];
}

- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    
    // 添加观察者
    [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    // 移除观察者
    [[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
}


#pragma mark - 实现SKProductsRequest的代理方法
// 请求到可卖商品的结果
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
{
    // 1.展示商品
    self.products = [response.products sortedArrayWithOptions:NSSortConcurrent usingComparator:^NSComparisonResult(SKProduct *obj1, SKProduct *obj2) {
        return [obj1.price compare:obj2.price];
    }];
    
    // 2.刷新表格
    [self.tableView reloadData];
}

// 请求失败
- (void)request:(SKRequest *)request didFailWithError:(NSError *)error {

    NSLog(@"%@", [error localizedDescription]);
}

// 请求结束
- (void)requestDidFinish:(SKRequest *)request {

    NSLog(@"请求结束");
}


#pragma mark - 实现tableView的数据源和代理方法
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.products.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *ID = @"productCellId";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ID];
    }
    
    // 1.取出模型
    SKProduct *product = self.products[indexPath.row];
    
    // 2.给cell设置数据
    cell.textLabel.text = product.localizedTitle;
    cell.detailTextLabel.text = [NSString stringWithFormat:@"价格:%@", product.price];
    
    return cell;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 1.取出模型
    SKProduct *product = self.products[indexPath.row];
    
    // 2.购买商品
    [self buyProduct:product];
}

#pragma mark - 购买商品
- (void)buyProduct:(SKProduct *)product
{
    // 1.创建票据
    SKPayment *payment = [SKPayment paymentWithProduct:product];
    
    // 2.将票据加入到交易队列中
    [[SKPaymentQueue defaultQueue] addPayment:payment];
}

#pragma mark - 实现SKPaymentQueue的回调方法
// 监听购买结果
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions
{
    for (SKPaymentTransaction *transation in transactions) {
        switch (transation.transactionState) {
            case SKPaymentTransactionStatePurchasing:
                NSLog(@"正在交易");
                break;
                
            case SKPaymentTransactionStatePurchased:
                NSLog(@"交易成功");
                
                // 将交易从交易队列中移除, 否则会验证不通过,以下几个都得移除
                [queue finishTransaction:transation];
                break;
                
            case SKPaymentTransactionStateFailed:
                NSLog(@"交易失败");
                [queue finishTransaction:transation];
                break;
                
            case SKPaymentTransactionStateRestored:
                NSLog(@"购买过该商品");
                [queue finishTransaction:transation];
                break;
                
            case SKPaymentTransactionStateDeferred:
                NSLog(@"未决定");
                break;
            default:
                break;
        }
    }
}

- (void)dealloc {

    [[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
}
```

另外使用这个方法即可恢复购买

```
[[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
```



---