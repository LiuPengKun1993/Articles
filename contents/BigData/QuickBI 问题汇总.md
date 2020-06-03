# QuickBI 问题汇总

### Quick BI 连接 MaxCompute 数据报错：

Quick BI 连接 MaxCompute 数据源，参考了阿里云文档：[云数据源MaxCompute](https://helpcdn.aliyun.com/document_detail/166109.html?spm=a2c4g.11174283.6.603.1dca7151UCWdWL)，检查了参数，但还是连接不通。

```
显示名称：****
数据库地址：http://service.cn-hangzhou.maxcompute.aliyun-inc.com/api
项目名称：maxcompute 项目名
```

研究半天是 ak 账号的原因，子账户的 ak 需要自己创建，子账户又没有权限创建，所以只能主账户给子账户开权限，子账户创建 ak，这样就连通了。

---

### Table(***) is full scan with all partitions, please specify partition predicates

仪表板报错：[46] execute failed: ODPS-0130071:[0,0] Semantic analysis exception - physical plan generation failed: java.lang.RuntimeException: Table(***) is full scan with all partitions, please specify partition predicates.

这个问题是由于分区表引起的，有两个解决方案，第一是给该数据集开启全表扫描，参考文档：[优化数据集性能](https://help.aliyun.com/document_detail/169395.html?spm=a2c4g.11186623.6.630.2b6a74efA9S6X5#title-mdr-mh0-buh)，第二是给数据集设置过滤条件，只查询某一个分区的。