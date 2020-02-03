# JS 表单格式直传 OSS

> 公司的几个 APP 是使用 `cordova+angularjs+ionic` 的模式开发的，因此要有一套 JS 端的埋点 SDK，这里主要说一下 JS 表单格式上传 OSS。

和移动端整体思想一样，首先调取服务端接口获取签名信息（服务端我是使用 node.js 写的，接下来如有时间也会介绍一下签名这一块），获取到签名信息后，把 key、OSSAccessKeyId、policy、Signature 以及需要上传的数据等拼接成表单结构上传至 OSS，核心代码如下：

```
function __uploadOss() {
    let startIndex = __token.indexOf('_') + 1;
    let endIndex = __token.lastIndexOf('_');
    let appName = JSON.parse(JSON.stringify(__token)).substring(startIndex, endIndex);
    axios({
        // 获取签名的接口
        url: 'http://127.0.0.1:7002/user/getToken',
        method: 'post',
        data: {
            // 上传文件的目录
            "dirPath": 'test/' + __getDate() + '/' + appName + '/' + new Date().getTime()
        }
    }).then(res => {
        console.log(res.data);
        console.log(res.data.statusCode);
        if (res.data.statusCode == 200) {
            // cordova 原生插件的方法
            cordova.plugins.buried_track.readData("获取数据", result => {
                console.log(res.data);
                // 拼接成表单结构
                let data = new FormData();
                data.append("key", res.data.result.dirPath);
                data.append("OSSAccessKeyId", res.data.result.OSSAccessKeyId);
                data.append("policy", res.data.result.policy);
                data.append("Signature", res.data.result.Signature);
                data.append("file", result);
                data.append("success_action_status", 200);

                let url = 'http://桶名.oss-cn-beijing.aliyuncs.com';
                let config = {
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                }
                axios.post(url, data, config).then((res) => {
                    if (res.status == 200) {
                        console.log('上传成功');
                        // 上传成功之后删除原生插件里的日志
                        cordova.plugins.buried_track.deleteData("删除数据", result => { }, error => { });
                        cordova.plugins.buried_track.writeTime(JSON.stringify(new Date().getTime()), result => { }, error => { });
                    } else {
                        console.log('上传失败');
                    }
                }).catch((err) => {
                    console.log(err);
                    console.log('上传失败1');
                })
            }, error => alert('error'));
        }
    })
}
```
这里需要说明一下，`cordova.plugins.buried_track.readData` 是调取 cordova 原生插件的方法，`buried_track` 是插件名字，`readData` 是方法名字，关于 `cordova+angularjs+ionic`，它是混合开发模式的一种，类似于 Flutter，可以兼容 iOS、Android、web 等多个平台，正所谓一份代码，多端运行，有兴趣的童鞋可以看看这里[Build Your First Ionic React App](https://ionicframework.com/docs/react/your-first-app)。

去掉 cordova 相关的代码之后就更简单了：

```
function __uploadOss() {
    axios({
      url: '获取签名的接口',
      method: 'post',
      data: {
        "key": 'value'
      }
    }).then(res => {
      console.log(res.data);
      console.log(res.data.statusCode);
      if (res.data.statusCode == 200) {
        console.log(res.data);
        let data = new FormData();
        data.append("key", res.data.result.dirPath);
        data.append("OSSAccessKeyId", res.data.result.OSSAccessKeyId);
        data.append("policy", res.data.result.policy);
        data.append("Signature", res.data.result.Signature);
        data.append("file", result);
        data.append("success_action_status", 200);
        let url = res.data.result.host;
        let config = {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        }

        axios.post(url, data, config).then((res) => {
          if (res.status == 200) {

          } else {

          }
        }).catch((err) => {
          console.log(err);
        })
      }
    })
  }
```

结束语：
本人本职工作是 iOS，前端只是前段时间因工作需要做了几个月，搞定了这个需求之后的感觉是，JS 的表单格式上传比 iOS 简单很多。如果哪里写的有问题，欢迎各位童鞋多多指教。


相关文章

[iOS 表单格式上传 OSS](https://juejin.im/post/5df8d65d6fb9a01600533fad)

参考文档

[JavaScript客户端签名直传](https://help.aliyun.com/document_detail/31925.html?spm=a2c4g.11186623.6.1386.6dd76e28s5N09r#title-k6a-4w3-kkg)