# HIVE 调优—— hive.fetch.task.conversion

Fetch 抓取是指，Hive 中对某些情况的查询可以不必使用 MapReduce 计算。

> 启用 MapReduce Job 是会消耗系统开销的。对于这个问题，从 Hive0.10.0 版本开始，对于简单的不需要聚合的类似 `select <col> from <table> limit n`语句，不需要起 MapReduce job，直接通过 Fetch task 获取数据。


比如：`select * from user_table;`在这种情况下，Hive 可以简单地读取 user_table 对应的存储目录下的文件，然后输出查询结果到控制台。

在 `hive-default.xml.template` 文件中`hive.fetch.task.conversion`默认是 more，老版本 hive 默认是 minimal，该属性修改为 more 以后，在全局查找、字段查找、limit 查找等都不走 MapReduce。

```
<property>
    <name>hive.fetch.task.conversion</name>
    <value>more</value>
    <description>
      Expects one of [none, minimal, more].
      Some select queries can be converted to single FETCH task minimizing latency.
      Currently the query should be single sourced not having any subquery and should not have
      any aggregations or distincts (which incurs RS), lateral views and joins.
      0. none : disable hive.fetch.task.conversion
      1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
      2. more  : SELECT, FILTER, LIMIT only (support TABLESAMPLE and virtual columns)
    </description>
  </property>
```

1）把hive.fetch.task.conversion设置成none，然后执行查询语句，都会执行mapreduce程序。

```
hive (default)> set hive.fetch.task.conversion=none;
hive (default)> select * from emp;
hive (default)> select ename from emp;
hive (default)> select ename from emp limit 3;
Time taken: 18.203 seconds, Fetched: 14 row(s)
```

2）把hive.fetch.task.conversion设置成more，然后执行查询语句，如下查询方式都不会执行mapreduce程序。

```
hive (default)> set hive.fetch.task.conversion=more;
hive (default)> select * from emp;
hive (default)> select ename from emp;
hive (default)> select ename from emp limit 3;
Time taken: 0.09 seconds, Fetched: 3 row(s)
```
