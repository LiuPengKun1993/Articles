# 混合开发 Cordova+Ionic+Angular 入门篇

### 前言

公司有项目是使用混合开发 Cordova+Ionic+Angular 模式编写的，因为我们部门接手了这个项目，因此不得不熟悉一下 Cordova+Ionic+Angular 模式。

> 先简单谈一下混合开发，我最早是在2016年年底接触的混合开发，当时组内考虑使用 RN 进行项目开发，于是我借着闲暇时间看了一些 RN 方面的技术，也尝试着做了一些小项目...混合开发的优势很明显，总所周知的是一份代码多处运行、开发成本低等等，混合开发的弊端主要是用户体验以及性能稍差，但随着技术的更新，相信混合开发在这方面也会逐渐改善。



**接下来步入正题，谈谈 Cordova+Ionic+Angular 入门：**

### 环境搭建（Mac 版）：

#### 1.安装 npm 、node.js

建议先安装 brew，终端输入 `brew -v` 查看是否安装了 homebrew，如没安装，可根据 [homebrew 官网](https://brew.sh) 说明进行安装；如已安装，执行 `brew install npm ` 与 `brew install node`，接着查看是否安装成功：

![](https://github.com/liuzhongning/Articles/blob/master/resources/hybrid-app/hybrid01.jpg)

#### 2.安装 ionic 和 cordova
输入命令行 `npm install -g cordova ionic`，如果安装不了，可以试试国内镜像安装，首先安装 cnpm，执行 `npm install -g cnpm --registry=https://registry.npm.taobao.org` 命令安装。然后执行命令 `cnpm install -g cordova ionic` 安装 ionic 和 cordova，接着查看是否安装成功：

![](https://github.com/liuzhongning/Articles/blob/master/resources/hybrid-app/hybrid02.jpg)

### 创建 Ionic 工程

#### 1. 创建 ionic 项目
```
ionic start <name> <template> [options]

例子：

// 生成名字为 ionic-test-App 的空项目
ionic start ionic-test-App blank 

// 生成名字为 ionic-test-App 的带有选项卡的项目
ionic start ionic-test-App tabs

// 生成名字为 ionic-test-App 的带有侧边菜单的项目
ionic start ionic-test-App sidemenu
```

终端执行 `ionic serve` 在浏览器打开该项目：

![](https://github.com/liuzhongning/Articles/blob/master/resources/hybrid-app/hybrid03.jpg)

#### 2. 创建安卓应用

```
$ cd ionic-test-App
$ ionic cordova platform add android
$ ionic cordova build android
$ ionic cordova emulate android
```

需打开 Android Studio，并打开模拟器，不然最后一步 `ionic cordova emulate android` 会报错找不到模拟器。最后运行如下：

![](https://github.com/liuzhongning/Articles/blob/master/resources/hybrid-app/hybrid04.jpg)

#### 3. 创建 iOS 应用

```
$ cd ionic-test-App
$ ionic cordova platform add ios
$ ionic cordova build ios
$ ionic cordova emulate ios
```

如果出现 `ios-sim was not found` 错误，可以执行命令：`npm install -g ios-sim`。最后运行如下：

![](https://github.com/liuzhongning/Articles/blob/master/resources/hybrid-app/hybrid05.jpg)

---

学习文档：[Ionic Framework](https://ionicframework.com/docs/)
