# iOS开发 - 极光推送中生产证书和开发证书的生成

> 项目中用到了 [极光推送](https://www.jiguang.cn/push?source=bdjj#B_vid=1350498826687775136)，这里记录下极光推送中生产证书和开发证书的生成过程。

推送设置中需要配置生产证书以及开发证书~~
![极光推送中生产证书和开发证书的生成](http://upload-images.jianshu.io/upload_images/2665449-f6e129813e38c4b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###1.开发证书生成步骤
- 登录 [苹果开发者中心](https://developer.apple.com/account/#/overview/82W2K9C6T2)，点击 **Certificates, IDs & Profiles**

![开发证书生成步骤1](http://upload-images.jianshu.io/upload_images/2665449-b6de1efe8a9bc32d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 点击 **Development**，并选择 **Apple Push Notification service SSL (Sandbox)**，接着点击下边的 **Continue** 按钮

![开发证书生成步骤2](http://upload-images.jianshu.io/upload_images/2665449-140d40d381467beb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 来到 **App ID** 这个界面之后，继续点击下边的 **Continue** 按钮

![开发证书生成步骤3](http://upload-images.jianshu.io/upload_images/2665449-69cf10ec6fcb88c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 这里的说明是让我们生成一个 **CSR 文件**，继续点击 **Continue** 按钮

![开发证书生成步骤4](http://upload-images.jianshu.io/upload_images/2665449-c02bbf6bb568e09f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 这个界面需要上传 CSR 文件，上传 CSR 证书之后继续点击 **Continue** 按钮（ **CSR 文件** 生成很简单，后面会提一下）
![开发证书生成步骤5](http://upload-images.jianshu.io/upload_images/2665449-1bc222f798021465.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 点击 **Continue** 按钮之后，就会出现下面这个界面，直接点击 **Download** ，然后双击下载的证书，安装到钥匙串里。
![开发证书生成步骤6](http://upload-images.jianshu.io/upload_images/2665449-0bb19655f5e19139.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####1.1 CSR 文件生成步骤
- 首先打开电脑上的钥匙串，点击 **从证书颁发机构请求证书**，如下图所示：

![生成 CSR 文件步骤1](http://upload-images.jianshu.io/upload_images/2665449-153b4345af65eb1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 填写用户邮箱地址以及 CA 电子邮件地址，将请求改为“存储到磁盘”，点击 **继续**，接着点击 **存储** 即可。
![生成 CSR 文件步骤2](http://upload-images.jianshu.io/upload_images/2665449-b6141e6c50613449.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###2.生产证书生成步骤

- 生产证书的生成和开发证书的生成类似，先选择 **Production** ，点击“加号”后选择 **Production** 中的 ** Apple Push Notification service SSL (Sandbox & Production)** 这一栏，接着点击 **Continue** 按钮

![生产证书生成步骤1](http://upload-images.jianshu.io/upload_images/2665449-a8554f4b0b579a2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 来到 **App ID** 这个界面之后，继续点击下边的 **Continue** 按钮

![生产证书生成步骤2](http://upload-images.jianshu.io/upload_images/2665449-721072f3f4f498bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 然后依然会让上传一个 **CSR 文件**， 不再累述。生产证书生成后，下载安装到钥匙串中即可。

![生产证书生成步骤3](http://upload-images.jianshu.io/upload_images/2665449-7dda501137bad5df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###3. 导出 **p12** 文件

- 选中刚刚安装的生产证书和开发证书，然后导出~
![导出 **p12** 文件步骤1](http://upload-images.jianshu.io/upload_images/2665449-7b1b25481edad0e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 文件格式为 **个人信息交换(.p12)**
![导出 **p12** 文件步骤2](http://upload-images.jianshu.io/upload_images/2665449-cf0e2374a4886415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 接着设置下密码
![导出 **p12** 文件步骤3](http://upload-images.jianshu.io/upload_images/2665449-8c590c020fed9efe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####最后将两个p12文件上传到极光推送上，就OK了！

![极光推送中生产证书和开发证书的生成](http://upload-images.jianshu.io/upload_images/2665449-d8d78f18c48bcf0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
