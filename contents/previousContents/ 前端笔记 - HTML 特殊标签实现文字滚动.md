#  前端笔记 - HTML 特殊标签实现文字滚动

> 想用 HTML 做个文字滚动效果，可以用特殊标签 marquee 实现，当然也可以用 JS 实现， 今天试了下用特殊标签 marquee 实现文字滚动。先声明下本人从事于 iOS 开发，之前零零星星的学过 HTML ，但目前尚处于小白阶段😂，因此文章如果有不当之处还请各位大神不吝指出！


### 先一起看下 marquee 标签存在哪些属性：
- behavior：滚动方式，三种：scroll（循环滚动） slide（单次滚动）、alternate（来回滚动）
- bgcolor：滚动文本框的背景颜色
- direction：滚动方向，四种：left（从右到左）、right（从左到右）、up（从下到上）、down（从上到下）
- scrolldelay：每轮滚动之间的延迟时间，数字越大滚动越慢
- scrollamount：一次滚动总的时间量，数字越小滚动越慢
- align：文字的对齐方式，三种： middle(居中)、bottom(居下)、top(居上)
- width：滚动文本框的宽度
- height：滚动文本框的高度
- loop：滚动次数，默认为 infinite（无限）
- hspace：水平方向的空白距离
- vspace：垂直方向的空白距离

### 下面便是 marquee 属性的一些应用

#### 一、滚动方式 behavior ：scroll（循环滚动）
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="right"
         behavior="scroll"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```

![循环滚动](http://upload-images.jianshu.io/upload_images/2665449-47605cbed1e7bf3a.gif?imageMogr2/auto-orient/strip)

#### 二、滚动方式 behavior：alternate（来回滚动）
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="right"
         behavior="alternate"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```

![来回滚动](http://upload-images.jianshu.io/upload_images/2665449-2ef9e8f7f7097a05.gif?imageMogr2/auto-orient/strip)

### 三、滚动方向 direction：up（从下到上）

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="up"
         behavior="scroll"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```
![从下到上](http://upload-images.jianshu.io/upload_images/2665449-afc2adaff655f9b5.gif?imageMogr2/auto-orient/strip)

### 四、滚动方向 direction：left（从右到左）

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```

![从右到左](http://upload-images.jianshu.io/upload_images/2665449-59c59ad94d252736.gif?imageMogr2/auto-orient/strip)

### 五、滚动速度（scrollamount）
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"
         scrollamount="80px"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```

![滚动速度](http://upload-images.jianshu.io/upload_images/2665449-8c084cd3ab2119e7.gif?imageMogr2/auto-orient/strip)

### 六、循环次数（loop）

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"
         scrollamount="80px"
         loop="5"><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```
![循环次数](http://upload-images.jianshu.io/upload_images/2665449-1e7e0b115db0402f.gif?imageMogr2/auto-orient/strip)

### 七、当鼠标停留在文字上，文字停止滚动

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"
         scrollamount="80px"
         onmouseover=stop()
         onmouseout=start()><br>
    金风玉露一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```

![当鼠标停留在文字上，文字停止滚动](http://upload-images.jianshu.io/upload_images/2665449-a9a929dcfc80b834.gif?imageMogr2/auto-orient/strip)

### 八、给滚动字幕加超链接

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"
         scrollamount="80px"
         onmouseover=stop()
         onmouseout=start()><br>
    <a href=#>金风玉露</a>一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```


![给滚动字幕加超链接](http://upload-images.jianshu.io/upload_images/2665449-5668ad5af1df16db.gif?imageMogr2/auto-orient/strip)

### 九、综合其它属性

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
<marquee direction="left"
         behavior="scroll"
         scrollamount="30"
         scrolldelay="0"
         loop="-1"
         width="500"
         height="100"
         bgcolor="#FFCCFF"
         hspace="30"
         vspace="10"
         onmouseover=this.stop()
         onmouseout=start()><br>
    <a href=#>金风玉露</a>一相逢, 便胜却人间无数.<br>
    两情若是久长时, 又岂在朝朝暮暮.<br>
</marquee>
</body>
</html>
```


![综合其它属性](http://upload-images.jianshu.io/upload_images/2665449-d8ef46a6806d1d63.gif?imageMogr2/auto-orient/strip)


小结：试了 HTML 实现文字滚动效果之后，发现比 iOS 简单方便许多，以后在工作之余会多学学前端知识，求大神带哈😀！