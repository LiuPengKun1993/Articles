# Mac 配置 Flutter 环境，运行 iOS Android 两端

1.在 [Flutter 官网](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos#macos) 下载其最新可用的安装包，解压之后，放在想安装的目录下，记住文件目录，比如：`/Users/liupengkun/Documents/flutter_learn`。

2.接着打开终端，输入命令行 `vim ~/.bash_profile`，打开文件后，添加 flutter 相关工具到 path 中：

```
// 路径要使用你上面放置Sdk的目录路径
export PATH=/Users/liupengkun/Documents/flutter_learn/flutter/bin:$PATH
```

3.使用镜像，由于在国内访问 Flutter 有时可能会受到限制，Flutter 官方为中国开发者搭建了临时镜像（此镜像为临时镜像，并不能保证一直可用，读者可以参考 [Using Flutter in China](https://flutter.io/community/china) 以获得有关镜像服务器的最新动态。）：

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

4.改完之后，关闭 bash_profile 文件，执行 `source ~/.bash_profile`。

5.接着终端执行 `flutter doctor` 检测环境：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_01.jpg)

每个人的电脑具体配置不一样，flutter doctor 检测出来的问题也不一样，因为我电脑之前使用过 Xcode、 Android Studio、VSCode，所以报的问题相对较少，IntelliJ IDEA 倒是用的不多。

接下来一一解决报的问题：

```
! Some Android licenses not accepted.  To resolve this, run: flutter doctor --android-licenses

```

这个简单，接受这些协议即可。直接终端执行 `flutter doctor --android-licenses`，一直输入 `y` 并换行，直至成功。

```
✗ Flutter plugin not installed; this adds Flutter specific functionality.
✗ Dart plugin not installed; this adds Dart specific functionality.
```

这个意思也很清楚，需要打开 AS，在 configure 里安装 Flutter 以及 Dart 插件：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_02.jpg)

接着重新在终端执行 flutter doctor：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_03.jpg)

可以看到刚刚的两个问题已经好了，关于 IntelliJ IDEA 的问题，暂时因为我这边很少用到 IntelliJ IDEA，先搁置，其实也很简单，在 IntelliJ IDEA 里安装 Flutter 以及 Dart 插件就可以了。

6.创建 Flutter 项目并运行

cd 到项目所在文件夹，然后执行命令：

```
flutter create hello_flutter
```

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_04.jpg)

打开上图 ios 或 android 项目，并使用模拟器运行。

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_05.jpg)


关于 `! No devices available` 的问题，我这里测试的结论是，用模拟器运行的话是没作用的，只有真机运行，才会有改变：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_06.jpg)
