# MaxCompute 自定义 Java UDF

> 公司大数据开发使用的是阿里云的 MaxCompute，MaxCompute 自身提供了很多 Hive SQL 函数，能够满足大部分需求，但是总有一些 Hive SQL 函数无法满足的需求，比如 base64 编解码等等，目前 MaxCompute 内部是没有相关函数的。解决办法是用户自定义函数，MaxCompute 提供了相关的文档，具体可以看 [这里](https://help.aliyun.com/document_detail/27866.html?spm=a2c4g.11186623.6.722.79fd612dqYS0p2)。

本篇文章主要记录使用 Java 来自定义函数。

#### 1. IDEA 里新建项目

在 IDEA 里新建一个项目，并创建一个 class 文件，命名为 data_mask。

#### 2. 添加 odps 依赖

在 pom 文件添加 odps 依赖：

```
<dependency>
	<groupId>com.aliyun.odps</groupId>
	<artifactId>odps-sdk-udf</artifactId>
	<version>0.29.10-public</version>
</dependency>
```

#### 3. 编写代码

在类文件里编写代码（data_mask 要继承自 阿里云的 UDF）：

```
package com.example.data_mask;

// 阿里云 UDF
import com.aliyun.odps.udf.UDF;

public class data_mask extends UDF {

     // 创建 evaluate 方法，与 HIVE 一样，MaxCompute 的 UDF 通常使用 evaluate 方法
     public String evaluate(String string) {
         return string + "UDF";
     }

}

```

#### 4.导出 JAR 包

此时 Java UDF 函数已自定义完毕，接着需要在 MaxCompute 的资源函数里添加 JAR 包。

#### 5. 新建资源

在 MaxCompute 里新建资源，并把刚刚导出的 JAR 包导入。

![](https://upload-images.jianshu.io/upload_images/2665449-d09f67f66713ab97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6. 注册函数

在 MaxCompute 里注册函数，类名里面填写 package 以及类名，资源列表里填写刚刚导入的 JAR 包名称。

![](https://upload-images.jianshu.io/upload_images/2665449-940a2da1c6552c50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 7.提交发布

将刚刚创建的资源和函数都提交，并发布。

![](https://upload-images.jianshu.io/upload_images/2665449-527c5e9ea8ae520c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

发布成功之后就可以直接调用了，直接根据定义的函数名称调用即可：

![](https://upload-images.jianshu.io/upload_images/2665449-db4e333f65f47651.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考：

- [Java UDF](https://help.aliyun.com/document_detail/27867.html?spm=a2c4g.11186623.6.723.1ccd612dbrlNl7)
- [JSON字符串获取示例](https://help.aliyun.com/document_detail/101960.html?spm=a2c4g.11186623.6.732.293f34d8YKI6IT)