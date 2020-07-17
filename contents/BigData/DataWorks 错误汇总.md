# DataWorks 错误汇总


- 报错日志

```
FAILED: ODPS-0420095: Access Denied - Authorization Failed [4093], You have NO privilege to do the restricted operation on  {acs:odps:*:projects/***}. Access Mode is AllDenied.  Context ID:afd84055-ef3c-4d8c-86ee-ff3695c15a34.
```

- 报错原因

线上脚本报以上错误，大多是欠费的原因，续费就好了，注意已报错的脚本需要重跑一下。

---

- 报错日志

```
2020-05-28 01:49:01.597 [job-156364196] ERROR RetryUtil - Exception when calling callable, 即将尝试执行第1次重试,共计重试9次.本次重试计划等待[1,000]ms,实际等待[1,000]ms, 异常Msg:[Code:[MYSQLErrCode-02], Description:[数据库服务的IP地址或者Port错误，请检查填写的IP地址和Port或者联系DBA确认IP地址和Port是否正确。如果是同步中心用户请联系DBA确认idb上录入的IP和PORT信息和数据库的当前实际信息是一致的].  - 具体错误信息为：Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server. - com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
    at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
    at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
    at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
    at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
    at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:990)
    at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:342)
    at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2188)
    at com.mysql.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:2221)
    at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2016)
    at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:776)
    at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
    at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
    at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
    at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
    at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
    at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:386)
    at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:330)
    at java.sql.DriverManager.getConnection(DriverManager.java:674)
    at java.sql.DriverManager.getConnection(DriverManager.java:217)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.connect(DBUtil.java:567)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.getMySQLConnection(DBUtil.java:460)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.connect(DBUtil.java:439)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.access$100(DBUtil.java:26)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil$3.call(DBUtil.java:378)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil$3.call(DBUtil.java:375)
    at com.alibaba.datax.common.util.RetryUtil$Retry.call(RetryUtil.java:163)
    at com.alibaba.datax.common.util.RetryUtil$Retry.doRetry(RetryUtil.java:111)
    at com.alibaba.datax.common.util.RetryUtil.executeWithRetry(RetryUtil.java:31)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.getConnection(DBUtil.java:375)
    at com.alibaba.datax.plugin.rdbms.util.DBUtil.getConnection(DBUtil.java:359)
    at com.alibaba.datax.plugin.rdbms.util.JdbcConnectionFactory.getConnecttion(JdbcConnectionFactory.java:29)
    at com.alibaba.datax.plugin.rdbms.writer.util.OriginalConfPretreatmentUtil.dealColumnConf(OriginalConfPretreatmentUtil.java:167)
    at com.alibaba.datax.plugin.rdbms.writer.util.OriginalConfPretreatmentUtil.dealColumnConf(OriginalConfPretreatmentUtil.java:250)
    at com.alibaba.datax.plugin.rdbms.writer.util.OriginalConfPretreatmentUtil.doPretreatment(OriginalConfPretreatmentUtil.java:43)
    at com.alibaba.datax.plugin.rdbms.writer.CommonRdbmsWriter$Job.init(CommonRdbmsWriter.java:68)
    at com.alibaba.datax.plugin.writer.mysqlwriter.MysqlWriter$Job.init(MysqlWriter.java:31)
    at com.alibaba.datax.core.job.JobContainer.initJobWriter(JobContainer.java:1064)
    at com.alibaba.datax.core.job.JobContainer.init(JobContainer.java:451)
    at com.alibaba.datax.core.job.JobContainer.start(JobContainer.java:210)
    at com.alibaba.datax.core.Engine.start(Engine.java:96)
    at com.alibaba.datax.core.Engine.entry(Engine.java:246)
    at com.alibaba.datax.core.Engine.main(Engine.java:279)
Caused by: java.net.ConnectException: Connection refused
    at java.net.PlainSocketImpl.socketConnect(Native Method)
    at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
    at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
    at java.net.Socket.connect(Socket.java:643)
    at com.mysql.jdbc.StandardSocketFactory.connect(StandardSocketFactory.java:211)
    at com.mysql.jdbc.MysqlIO.<init>(MysqlIO.java:301)
    ... 37 more
]
```


- 报错原因

![](https://github.com/liuzhongning/Articles/blob/master/resources/BigData/DataWorks-errors-01.jpg)

由上图可知，数据源有问题，链接 polardb 失败，导致无法导数据到 polardb，解决方法是，找运维连通数据源，然后重跑报错脚本。

---

- 跨工作空间取数据时报权限问题（比如在 A 空间内，访问 B 空间的表）

- 解决方案：主账号授权，授权链接：[https://help.aliyun.com/document_detail/27935.html?spm=a2c4g.11186623.6.927.da6a6847ZlA602](https://help.aliyun.com/document_detail/27935.html?spm=a2c4g.11186623.6.927.da6a6847ZlA602)

还有个解决方案，可以根据具体的表名、资源名，以 Package 的方式，将数据部分授权给其它工作空间。参考文档：[https://help.aliyun.com/document_detail/34603.html?spm=a2c4g.11186623.6.934.29d12eb6JDuE5y](https://help.aliyun.com/document_detail/34603.html?spm=a2c4g.11186623.6.934.29d12eb6JDuE5y)

---

- Maxcompute 访问 OSS 的权限问题

在读取外部表的时候，报了这样的错：

```
FAILED: Generating job conf failed, gen jobconf failed: Failed to obtain external data information, error msg: build/release64/common/io/oss/oss_client.cpp(97): OSSRequestException: req_id: 5F110AC115158031331734EA, http status code: 403, error code: UnknownError, message: Access denied, please make sure the oss host and oss bucket is matched, and the bucket should be authorized to the odps project with correct role_arn
```

这个报错信息不是很清晰，试了很多方法还是没搞定，无奈问了阿里云技术人员，说报错403是因为找不到地址（又是权限问题，运维的同学又修改东西了）。

接着我在新建外部表的时候，报了这样的错误：

```
<Code>AccessDenied</Code> <Message>The bucket you access does not belong to you.</Message>
```

这个就比较清晰了，解决方案见下文档：

[https://help.aliyun.com/document_detail/72777.html?spm=a2c4g.11186623.6.786.136e6d03xKJI2D](https://help.aliyun.com/document_detail/72777.html?spm=a2c4g.11186623.6.786.136e6d03xKJI2D)