# canal 问题汇总

### 简介

[canal](https://github.com/alibaba/canal) 是阿里开源的 MySQL binlog 增量订阅&消费组件，基于 java 实现，整体已经在很多大型的互联网项目生产环境中使用，包括阿里、美团等都有广泛的应用，是一个非常成熟的数据库同步方案，基础的使用只需要进行简单的配置即可。

canal 是通过模拟成为 mysql 的 slave 的方式，监听 mysql 的 binlog 日志来获取数据，binlog 设置为 row 模式以后，不仅能获取到执行的每一个增删改的脚本，同时还能获取到修改前和修改后的数据，基于这个特性，canal 就能高性能的获取到 mysql 数据数据的变更。

### 使用

- canal 的部署主要分为 server 端和 client 端。
	- server 端部署好以后，可以直接监听 mysql binlog，因为 server 端是把自己模拟成了 mysql slave，所以，只能接受数据，没有进行任何逻辑的处理，具体的逻辑处理，需要 client 端进行处理。
	- client 端一般是需要大家进行简单的开发。[官网文档](https://github.com/alibaba/canal/wiki/ClientAPI)有一个简单的示例，很容易理解。

### 使用中遇到的问题

---

#### column size is not match 问题

见下图：

![](https://github.com/liuzhongning/Articles/blob/master/resources/BigData/DataWorks-canal-errors-01.jpg)

原因是 ddl 变更，导致列不匹配，报了 `column size is not match for table: xx , 12 vs 13` 的错误。解决方案是，删除 `conf/实例名` 目录下的 `h2.mv.db` 文件，如果没有 `h2.mv.db` 文件，删除 `meta.dat` 文件。`meta.dat` 文件内容如下图：

![](https://github.com/liuzhongning/Articles/blob/master/resources/BigData/DataWorks-canal-errors-02.jpg)

- 参考文档：
	- [TableMetaTSDB](https://github.com/alibaba/canal/wiki/TableMetaTSDB)
	- [https://github.com/alibaba/canal/issues/534](https://github.com/alibaba/canal/issues/534)

---

