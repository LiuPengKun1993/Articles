# 算法-几种排序算法 OC 版

> 如果你交给某人一个程序，你将“折磨”他一整天；如果你教某人如何编写程序，你将“折磨”他一辈子。——《大话数据结构》

这阵子抽空看了一些算法与数据结构相关的东西，不得不说算法数据结构这些思想真的是博大精深，作为 iOS 开发人员，为什么要学习算法？说实话很多开发人员可能不会算法，但照样能做 App，有位大家都熟悉的大神在几个月前写了篇文章，我也是无意间读到的，[搞 iOS 的学算法有意义吗？](http://chuansong.me/n/1331276728735)，大家有兴趣的话可以去读读。

> 这两天看了八大排序算法，并试着用 OC 语言写了几种，给大家分享下！

### 算法一：冒泡排序

个人觉得冒泡排序是最简单的排序了

```
#pragma mark - 冒泡排序
+ (void)bubbleSort:(NSMutableArray *)mutableArray {
    if(mutableArray == nil || mutableArray.count == 0)
        return;
    NSLog(@"冒泡排序之前: %@", mutableArray);
    for (NSInteger i = 0; i < mutableArray.count; i++) {
        for (NSInteger j = 0; j < mutableArray.count - i - 1; j++) {
            if ([mutableArray[j + 1] floatValue] < [mutableArray[j] floatValue]) {
                CGFloat tempFloat = [mutableArray[j] floatValue];
                mutableArray[j] = mutableArray[j + 1];
                mutableArray[j + 1] = [NSNumber numberWithFloat:tempFloat];
            }
        }
    }
    NSLog(@"冒泡排序之后: %@", mutableArray);
}
```

- 算法步骤：
1）比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2）对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3）针对所有的元素重复以上的步骤，除了最后一个。
4）持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

### 算法二：直接插入排序

```
#pragma mark - 直接插入排序
+ (void)insertSort:(NSMutableArray *)mutableArray {
    if(mutableArray == nil || mutableArray.count == 0)
        return;
    NSLog(@"直接插入排序之前: %@", mutableArray);
    for (NSInteger i = 0; i < mutableArray.count; i++) {
        CGFloat tempFloat = [mutableArray[i] floatValue];
        for (NSInteger j = i - 1; j >= 0 && tempFloat < [mutableArray[j] floatValue]; j--) {
            mutableArray[j + 1] = mutableArray[j];
            mutableArray[j] = [NSNumber numberWithFloat:tempFloat];
        }
    }
    NSLog(@"直接插入排序之后: %@", mutableArray);
}
```
- 算法步骤：
1）将第一待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列。
2）从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面。）


### 算法三：选择排序

```
#pragma mark - 选择排序
+ (void)chooseSort:(NSMutableArray *)mutableArray {
    if(mutableArray == nil || mutableArray.count == 0)
        return;
    NSLog(@"选择排序之前: %@", mutableArray);
    for (NSInteger i = 0; i < mutableArray.count; i++) {
        for (NSInteger j = i + 1; j < mutableArray.count; j++) {
            if ([mutableArray[i] floatValue] > [mutableArray[j] floatValue]) {
                CGFloat tempFloat = [mutableArray[i] floatValue];
                mutableArray[i] = mutableArray[j];
                mutableArray[j] = [NSNumber numberWithFloat:tempFloat];
            }
        }
    }
    NSLog(@"选择排序之后: %@", mutableArray);
}
```
- 算法步骤：
1）首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置
2）再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
3）重复第二步，直到所有元素均排序完毕。

### 算法四：折半插入排序

```
#pragma mark - 折半插入排序
+ (void)binaryInsertSort:(NSMutableArray *)mutableArray {
    if(mutableArray == nil || mutableArray.count == 0)
        return;
    NSLog(@"折半插入排序之前: %@", mutableArray);
    for(NSInteger i = 1 ; i < mutableArray.count ; i++) {
        CGFloat tempFloat = [[mutableArray objectAtIndex:i] floatValue];
        NSInteger left = 0;
        NSInteger right = i - 1;
        
        while (left <= right) {
            CGFloat middle = (left + right) / 2;
            
            if(tempFloat < [[mutableArray objectAtIndex:middle] floatValue]){
                right = middle - 1;
            } else {
                left = middle + 1;
            }
        }
        for(NSInteger j = i ; j > left; j--) {
            [mutableArray replaceObjectAtIndex:j withObject:[mutableArray objectAtIndex:j - 1]];
        }
        [mutableArray replaceObjectAtIndex:left withObject:[NSNumber numberWithFloat:tempFloat]];
    }
    NSLog(@"折半插入排序之后: %@", mutableArray);
}
```
- 折半插入排序是对插入排序算法的一种改进，排序算法过程，就是不断的依次将元素插入前面已排好序的序列中。


### 算法五：希尔排序

```
#pragma mark - 希尔排序
+ (void)shellSort:(NSMutableArray *)mutableArray {
    if(mutableArray == nil || mutableArray.count == 0)
        return;
    NSLog(@"希尔排序之前: %@", mutableArray);
    NSInteger shellValue = mutableArray.count / 2;
    while (shellValue >= 1) {
        for(NSInteger i = shellValue; i < mutableArray.count; i++) {
            CGFloat tempFloat = [[mutableArray objectAtIndex:i] floatValue];
            NSInteger j = i;
            while (j >= shellValue && tempFloat < [[mutableArray objectAtIndex:(j - shellValue)] floatValue]) {
                [mutableArray replaceObjectAtIndex:j withObject:[mutableArray objectAtIndex:j - shellValue]];
                j -= shellValue;
            }
            [mutableArray replaceObjectAtIndex:j withObject:[NSNumber numberWithFloat:tempFloat]];
        }
        shellValue = shellValue / 2;
    }
    NSLog(@"希尔排序之后: %@", mutableArray);
}
```
- 算法步骤：
1）选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
2）按增量序列个数k，对序列进行k 趟排序；
3）每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

> 排序算法就先总结到这里，需要看代码打印效果的童鞋可以到这里下载代码：[代码传送门](https://github.com/liuzhongning/NNSort)

另外分享一篇很不错的博文，[八大排序算法](http://blog.csdn.net/hguisu/article/details/7776068)，下面是各种排序算法的时间复杂度空间复杂度以及稳定性的展示图表：

![图片来自博文八大排序算法](http://upload-images.jianshu.io/upload_images/2665449-60505d0e355b8bc1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
