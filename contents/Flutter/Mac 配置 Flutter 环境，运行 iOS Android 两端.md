# Mac 配置 Flutter 环境，运行 iOS Android 两端

> Flutter 入门，从下载 Flutter SDK 到成功运行在 iOS Android 平台。编辑器：Xcode(11.3.1)、 Android Studio(3.2.1)、VSCode(1.43.2)。

#### 1.在 [Flutter 官网](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos#macos) 下载其最新可用的安装包，解压之后，放在想安装的目录下，记住文件目录，比如：`/Users/liupengkun/Documents/flutter_learn`。

#### 2.接着打开终端，输入命令行 `open ~/.bash_profile`，打开文件后，添加 flutter 相关工具到 path 中：

```
// 路径要使用你上面放置Sdk的目录路径
export PATH=/Users/liupengkun/Documents/flutter_learn/flutter/bin:$PATH
```

#### 3.使用镜像，由于在国内访问 Flutter 有时可能会受到限制，Flutter 官方为中国开发者搭建了临时镜像（此镜像为临时镜像，并不能保证一直可用，读者可以参考 [Using Flutter in China](https://flutter.io/community/china) 以获得有关镜像服务器的最新动态。）：

```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

#### 4.改完之后，关闭 bash_profile 文件，执行 `source ~/.bash_profile` 更新配置环境变量。

#### 5.接着终端执行 `flutter doctor` 检测环境，报了以下问题：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_01.jpg)

每个人的环境不太一样，因此 flutter doctor 检测出来的问题也会有差异。因为我电脑之前使用过 Xcode、 Android Studio、VSCode，所以报的问题相对较少，IntelliJ IDEA 倒是用的不多。

接下来开始解决报的问题：

```
! Some Android licenses not accepted.  To resolve this, run: flutter doctor --android-licenses

```

这个简单，接受这些协议即可。直接终端执行 `flutter doctor --android-licenses`，一直输入 `y` 并换行，直至成功。

```
✗ Flutter plugin not installed; this adds Flutter specific functionality.
✗ Dart plugin not installed; this adds Dart specific functionality.
```

这个意思也很清楚，打开 AS，在 configure 里安装 Flutter 以及 Dart 插件：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_02.jpg)

关于 `! No devices available` 的问题后面会讲到。

接着重新在终端执行 flutter doctor：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_03.jpg)

发现刚刚的两个问题已经好了，关于 IntelliJ IDEA，暂时我这边很少用到 IntelliJ IDEA，先搁置，其实想解决也很简单，在 IntelliJ IDEA 里安装 Flutter 以及 Dart 插件就可以了。

#### 6.创建 Flutter 项目并运行

cd 到存放项目的文件夹，然后执行命令：

```
flutter create hello_flutter
```

稍等一会儿，项目就构建成功了：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_04.jpg)

打开上图 ios 或 android 项目，并运行。

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_05.jpg)


关于 `! No devices available` 的问题，我这里测试的结论是，用模拟器运行的话是没作用的，只有真机运行，才会有改变：

![](https://github.com/liuzhongning/Articles/blob/master/resources/Flutter/flutter_06.jpg)

---


- 2020.04.08 更新
	- 问题 1：之前安装并配置好了 flutter，但是之后运行 `flutter doctor`时，报错 `zsh: command not found: flutter`，暂时性解决方案是执行`source ~/.bash_profile`；彻底解决方案是复制一份 `.bash_profile` 文件，改名为 `.zprofile`，终端执行 `source ~./zprofile`。
	- 问题 2：用 VSCode 运行项目到 iOS 模拟器时，一直 `Launching...`，用 Xcode 直接运行也不行，解决方案是手动删除 `~/Library/Developer/Xcode/DerivedData` 文件夹下的文件，重新运行。
	- 问题 3：执行命令时有时卡在了 `Waiting for another flutter command to release the startup lock`，解决方案是删除 flutter 的安装目录 /bin/cache/ 下的 lockfile 文件。