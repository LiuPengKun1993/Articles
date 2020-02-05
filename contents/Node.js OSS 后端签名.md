# Node.js OSS 后端签名


> 我们后端使用的是 Node.js，用的框架式是阿里的 [Egg.js](https://github.com/eggjs/egg/)。这是 [Egg 官网](https://eggjs.org/zh-cn/intro/quickstart.html) 对 Egg.js 做的介绍：*Egg.js 为企业级框架和应用而生，我们希望由 Egg.js 孕育出更多上层框架，帮助开发团队和开发人员降低开发和维护成本。*

Egg.js 使用起来及其简单方便，作为一个移动端开发者，初次使用还算顺利，感兴趣的童鞋可以查看其文档：[Egg 快速入门](https://eggjs.org/zh-cn/intro/quickstart.html)。

我们的需求是前端上传日志文件到 OSS，在上传文件过程中需要签名这一步骤。在前端进行签名也可以调通阿里云的接口，但是把 accessKeyId、accessKeySecret 放在前端存在很大的安全隐患，因此开发中我们通常在后端做签名。


前端上传数据到 OSS 架构：

![](https://github.com/liuzhongning/Articles/blob/master/resources/NodeJS-OSS-01.jpg)

- 用户发送上传 Policy 请求到应用服务器
- 应用服务器返回上传 Policy 和签名给用户
- 用户直接上传数据到 OSS


这里是后端签名的核心代码：

```
'use strict';
'use aliyun-oss-sign'

const Controller = require('egg').Controller;
const crypto = require("crypto")

const config = {
  bucket: '', //oss桶名
  region: '', //oss节点名
  accessKeyId: '', //申请的osskey
  accessKeySecret: '', // 申请的osssecret
  callbackIp: "", // 回调ip,一定要能被外网访问的地址。
  callbackPort: "8080", // 回调端口
  callbackPath: "user/ossCallback", // 回调接口路径
  expAfter: 60000, // 签名失效时间
  maxSize: 1048576000 // 最大文件大小
}

class UserController extends Controller {
  // 获取token
  async getToken() {
    const { ctx } = this;
    const {
      bucket,
      region,
      expAfter,
      maxSize,
      accessKeyId,
      accessKeySecret,
      callbackIp,
      callbackPort,
      callbackPath
    } = config
    let dict = this.ctx.request.body; // oss 文件夹 不存在会自动创建
    const dirPath = dict.dirPath;
    console.log(dirPath);
    const host = `http://${bucket}.${region}.aliyuncs.com` //你的oss完整地址
    const expireTime = new Date().getTime() + expAfter
    console.log(expireTime);
    const expiration = new Date(expireTime).toISOString()
    console.log(expiration);
    const policyString = JSON.stringify({
      expiration,
      conditions: [
        ['content-length-range', 0, maxSize],
        ['starts-with', '$key', dirPath]
      ]
    })
    const policy = Buffer(policyString).toString("base64")
    const Signature = crypto.createHmac('sha1', accessKeySecret).update(policy).digest("base64")
    const callbackBody = {
      "callbackUrl": `http://${callbackIp}:${callbackPort}/${callbackPath}`,
      "callbackHost": `${callbackIp}`,
      "callbackBody": "{\"filename\": ${object},\"size\": ${size}}",
      "callbackBodyType": "application/json"
    }
    const callback = Buffer(JSON.stringify(callbackBody)).toString('base64')

    ctx.body = {
      statusCode: "200",
      message: 'oss签名成功',
      result: {
        Signature,
        policy,
        host,
        'OSSAccessKeyId': accessKeyId,
        'key': expireTime,
        'success_action_status': 200,
        dirPath,
        callback
      }
    };
    ctx.status = 200;
  }
}

module.exports = UserController;
```
后端在 config 里填写 OSS 相关的各种信息，前端调取 getToken 接口，会得到 Signature 等一系列返回结果，大致如下:

```
{
  "statusCode": "200",
  "message": "oss签名成功",
  "result": {
    "Signature": "********",
    "policy": "*************",
    "host": "http://ta-app-log.oss-cn-hangzhou.aliyuncs.com",
    "OSSAccessKeyId": "************",
    "key": 1580875747468,
    "success_action_status": 200,
    "dirPath": "test",
    "callback": "********************"
  }
}

```

这样前端就可以安全的上传数据到 OSS 了。

结束语： 本人本职工作是 iOS，前段时间因工作需要做了一段时间前端开发。如果哪里写的有问题，欢迎各位童鞋多多指教。

相关文章：

- [iOS 表单格式上传 OSS](https://github.com/liuzhongning/Articles/blob/master/contents/iOS%20表单格式上传%20OSS.md)
- [JS 表单格式直传 OSS](https://github.com/liuzhongning/Articles/blob/master/contents/JS%20表单格式直传%20OSS.md)

