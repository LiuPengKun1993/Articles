# Cordova 使用 App Center 进行热更新

> App Center 是微软开发的云服务器，通过它开发者可以直接在用户的设备上部署手机应用更新。
> 
> 使用方法：首先注册一个 App Center 帐户，或通过 [https://appcenter.ms](https://appcenter.ms) 登录到现有的 App Center 帐户，项目需要安装的插件及代码配置可以直接参考官方文档：[Apache Cordova入门](https://docs.microsoft.com/en-us/appcenter/sdk/getting-started/cordova)。

### 常用技巧

#### 安装 code-push CLI

```
npm install -g code-push-cli 
// 安装成功后登陆
ode-push login 
```

login 时会打开浏览器，选择账号登录，在命令行粘贴 token，回车即可登录。

#### 创建部署

```
// 测试环境
code-push release-cordova MyGroup/MyAndroid android -m -d "Staging" --des “描述"
// 生产环境
code-push release-cordova MyGroup/MyAndroid android -m -d "Production" --des “描述"
```

#### 查看部署
部署成功后，输入以下命令行，查看部署状态，比如有多少触发了热更新等等：

```
appcenter codepush deployment list -a MyGroup/MyAndroid
```

![](https://github.com/liuzhongning/Articles/blob/master/resources/appcenter/appcenter01.jpg)

#### 查看所有发布的版本
下面两句命令可以查看所有历史热更新的版本：

```
// 测试环境
code-push deployment h MyGroup/MyAndroid Staging 
// 生产环境
code-push deployment h MyGroup/MyAndroid Production
```


### code-push 命令大全

![](https://github.com/liuzhongning/Articles/blob/master/resources/appcenter/appcenter02.jpg)

```
// 安装
npm install -g code-push-cli
// 注册账号
code-push register
// 登陆
code-push login
// 注销
code-push logout
// 添加项目
code-push app add [app名称]
// 删除项目
code-push app remove [app名称]
// 列出账号下的所有项目
code-push app list
// 显示登陆的token
code-push access-key ls
// 删除某个access-key
code-push access-key rm <accessKey>
// 添加协作人员
code-push collaborator add <appName> next@126.com
// 部署一个环境
code-push deployment add <appName> <deploymentName>
// 删除部署
code-push deployment rm <appName>
// 列出应用的部署
code-push deployment ls <appName>
// 查询部署环境的key
code-push deployment ls <appName> -k
// 查看部署的历史版本信息
code-push deployment history <appName> <deploymentNmae>
// 重命名一个部署
code-push deployment rename <appName> <currentDeploymentName> <newDeploymentName>
```

#### 相关资源：

- code push github:[https://github.com/Microsoft/cordova-plugin-code-push](https://github.com/Microsoft/cordova-plugin-code-push)
- code push官网：[http://microsoft.github.io/code-push/](http://microsoft.github.io/code-push/)
- code push cli手册：[http://microsoft.github.io/code-push/docs/cli.html](http://microsoft.github.io/code-push/docs/cli.html)
- code push javaScript Api: [https://github.com/Microsoft/react-native-code-push/blob/master/docs/api-js.md#codepush](https://github.com/Microsoft/react-native-code-push/blob/master/docs/api-js.md#codepush)

#### 参考的博客：

- [Code-Push使用](https://www.jianshu.com/p/cd7576af381f)
- [微软的React Native热更新 - 使用篇](https://www.jianshu.com/p/67de8aa052af)
- [react-native-code-push进阶篇](https://www.jianshu.com/p/6e96c6038d80?from=timeline)

