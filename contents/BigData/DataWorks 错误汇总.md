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