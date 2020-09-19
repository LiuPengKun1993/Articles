# 浅谈 Hive 性能优化

> 总结了 Hive 的常用优化手段。

### 列裁剪及分区裁剪

这是最基本的操作。所谓列裁剪就是在查询时只读取需要的列，分区裁剪就是只读取需要的分区。

比如当列很多或者数据量很大时，如果使用 ` select * from order_table; ` 或者不指定分区，全列扫描和全表扫描效率都很低。

这个时候我们可以指定列：

```
select
	uid,
	price
from
	order_table;
```

或者当这个表是分区表的时候，指定分区：

```
select
	uid,
	price
from
	order_table
where
	pt_date = "20200901";
```

Hive 中与列裁剪优化相关的配置项是`hive.optimize.cp`，与分区裁剪优化相关的则是`hive.optimize.pruner`，默认都是true。

### 谓词下推

在关系型数据库如 MySQL 中，也有谓词下推（Predicate Pushdown，PPD）的概念。就是将 SQL 语句中的 where 谓词逻辑都尽可能提前执行，减少下游处理的数据量。

```
select
	a.uid,
	b.user_id,
	...
from
    user_table a
inner join
    (select
        uid,
        user_id,
        price
    from
        order_table
    where
        pt_date = "20200901"
    and
        paystate = 2)b
on
    a.user_id = b.user_id;

```

对 order_table 做过滤的 where 语句写在子查询内部，而不是外部。Hive 中有谓词下推优化的配置项`hive.optimize.ppd`，默认值true，与它对应的逻辑优化器是 PredicatePushDown。该优化器就是将 OperatorTree 中的 FilterOperator 向上提。

### sort by 代替 order by

HiveSQL 中的 order by 与其他 SQL 方言中的功能一样，就是将结果按某字段全局排序，这会导致所有 map 端数据都进入一个 reducer 中，在数据量大时可能会长时间计算不完。

如果使用 sort by，那么还是会视情况启动多个 reducer 进行排序，并且保证每个 reducer 内局部有序。为了控制 map 端数据分配到 reducer 的 key，往往还要配合 distribute by 一同使用。如果不加 distribute by 的话，map 端数据就会随机分配到 reducer。

举个例子，假如要以 uid 为 key，以订单时间倒序、订单金额倒序输出记录数据：

```
select
    uid,
    user_id,
    price
from
    order_table
where
    pt_date = "20200901"
and
    paystate = 2
distribute by 
    uid
sort by 
    order_time desc,
    price desc;
```

### group by 代替 distinct

数据量较大的情况下，慎用count(distinct)，count(distinct)容易产生倾斜问题

当要统计某一列的去重数时，如果数据量很大，count(distinct) 就会非常慢，原因与 order by 类似，count(distinct) 逻辑只会有很少的 reducer 来处理。这时可以用 group by 来改写：

```
select 
    count(1) 
from 
    (
        select 
            uid 
        from 
            order_table
        where 
            pt_date = "20200901"
        group by 
            uid
    );
```

但是这样写会启动两个 MR job（单纯 distinct 只会启动一个），所以要确保数据量大到启动 job 的 overhead 远小于计算耗时，才考虑这种方法。当数据集很小或者 key 的倾斜比较明显时，group by 还可能会比 distinct 慢。

### group by 配置调整

#### map 端预聚合

group by 时，如果先起一个 combiner 在 map 端做部分预聚合，可以有效减少 shuffle 数据量。预聚合的配置项是 `hive.map.aggr`，默认值 true，对应的优化器为 GroupByOptimizer，简单方便。
通过 `hive.groupby.mapaggr.checkinterval` 参数也可以设置 map 端预聚合的行数阈值，超过该值就会分拆 job，默认值 100000。

#### 倾斜均衡配置项
group by 时如果某些 key 对应的数据量过大，就会发生数据倾斜。Hive 自带了一个均衡数据倾斜的配置项`hive.groupby.skewindata`，默认值false。
其实现方法是在 group by 时启动两个 MR job。第一个 job 会将 map 端数据随机输入 reducer，每个 reducer 做部分聚合，相同的 key 就会分布在不同的 reducer 中。第二个 job 再将前面预处理过的数据按 key 聚合并输出结果，这样就起到了均衡的效果。

### join 基础优化

#### build table（小表）前置

在最常见的 hash join 方法中，一般总有一张相对小的表和一张相对大的表，小表叫 build table，大表叫 probe table。

Hive 在解析带 join 的 SQL 语句时，会默认将最后一个表作为 probe table，将前面的表作为 build table 并试图将它们读进内存。如果表顺序写反，probe table 在前面，引发 OOM 的风险就高了。

在维度建模数据仓库中，事实表就是 probe table，维度表就是 build table。

#### 多表 join 时 key 相同

这种情况会将多个 join 合并为一个 MR job 来处理：

```
select
	a.uid,
	b.user_id,
	...
from
    user_table a
inner join
    (select
        uid,
        user_id,
        price
    from
        order_table
    where
        pt_date = "20200901"
    and
        paystate = 2)b
on
    a.user_id = b.user_id
inner join
    parent_table c
on  
    a.user_id = c.user_id;

```

如果上面两个 join 的条件不相同，比如改成 `a.uid = c.uid`，就会拆成两个 MR job 计算。
负责这个的是相关性优化器 CorrelationOptimizer，它的功能除此之外还非常多，逻辑复杂，可以参考 [Hive官方的文档](https://cwiki.apache.org/confluence/display/Hive/Correlation+Optimizer)。


#### 利用 map join 特性

map join 特别适合大小表 join 的情况。Hive 会将 build table 和 probe table 在 map 端直接完成 join 过程，消灭了 reduce，效率很高。

```
select
    /*+ mapjoin(a) */
	a.uid,
	b.user_id,
	...
from
    user_table a
inner join
    (select
        uid,
        user_id,
        price
    from
        order_table
    where
        pt_date = "20200901"
    and
        paystate = 2)b
on
    a.user_id = b.user_id;
```

### 优化 SQL 处理 join 数据倾斜

#### 空值或无意义值

空值或无意义值很常见，比如日志类型的数据，要统计每天的活跃手机号，但是总有一些日志数据没有收集到手机号，或为空、为 NULL 等，这个时候就需要提前将这些无意义的数据过滤掉，避免消耗。

#### build table 过大

有时，build table 会大到无法直接使用 map join 的地步，比如全量用户维度表，而使用普通 join 又有数据分布不均的问题。这时就要充分利用 probe table 的限制条件，削减 build table 的数据量，再使用 map join 解决。代价就是需要进行两次 join。举个例子：

```
select
    /*+ mapjoin(aa) */
    aa.uid,
    aa.user_id,
    ...
from
    (select
        /*+ mapjoin(a) */
        a.uid,
        b.user_id,
        ...
    from
        user_table a
    inner join
        (select
            uid,
            user_id,
            price
        from
            order_table
        where
            pt_date = "20200901"
        and
            paystate = 2)b
    on
        a.user_id = b.user_id)aa
inner join
    (select
        uid,
        user_id,
        price
    from
        order_table
    where
        pt_date = "20200902"
    and
        paystate = 2)bb;
```

### MapReduce 优化

#### 调整 mapper 数

mapper 数量与输入文件的 split 数息息相关，在 Hadoop 源码`org.apache.hadoop.mapreduce.lib.input.FileInputFormat` 类中可以看到 split 划分的具体逻辑。

可以直接通过参数 `mapred.map.tasks（默认值2）`来设定 mapper 数的期望值，但它不一定会生效，下面会提到。
设输入文件的总大小为 total_input_size。HDFS 中，一个块的大小由参数 dfs.block.size 指定，默认值 64MB 或 128MB。在默认情况下，mapper数就是：
`default_mapper_num = total_input_size / dfs.block.size`。

参数 `mapred.min.split.size（默认值1B）`和`mapred.max.split.size（默认值64MB）`分别用来指定 split 的最小和最大大小。split 大小和 split 数计算规则是：
`split_size = MAX(mapred.min.split.size, MIN(mapred.max.split.size, dfs.block.size))`；
`split_num = total_input_size / split_size`。

得出 mapper 数：
`mapper_num = MIN(split_num, MAX(default_num, mapred.map.tasks))`。

可见，如果想减少 mapper 数，就适当调高 mapred.min.split.size，split 数就减少了。如果想增大 mapper 数，除了降低 mapred.min.split.size 之外，也可以调高 mapred.map.tasks。


一般来讲，如果输入文件是少量大文件，就减少 mapper 数；如果输入文件是大量非小文件，就增大 mapper 数；至于大量小文件的情况，得参考下面“合并小文件”一节的方法处理。


#### 调整 reducer 数

reducer 数量的确定方法比 mapper 简单得多。使用参数`mapred.reduce.tasks`可以直接设定 reducer 数量，不像 mapper 一样是期望值。但如果不设这个参数的话，Hive 就会自行推测，逻辑如下：

参数`hive.exec.reducers.bytes.per.reducer`用来设定每个 reducer 能够处理的最大数据量，默认值 1G（1.2版本之前）或 256M（1.2版本之后）。

参数 `hive.exec.reducers.max` 用来设定每个 job 的最大 reducer 数量，默认值 999（1.2版本之前）或 1009（1.2版本之后）。

得出 reducer 数：
`reducer_num = MIN(total_input_size / reducers.bytes.per.reducer, reducers.max)`。

reducer 数量与输出文件的数量相关。如果 reducer 数太多，会产生大量小文件，对 HDFS 造成压力。如果 reducer 数太少，每个 reducer 要处理很多数据，容易拖慢运行时间或者造成 OOM。


#### 合并小文件

##### 输入阶段合并

需要更改 Hive 的输入文件格式，即参数`hive.input.format`，默认值是`org.apache.hadoop.hive.ql.io.HiveInputFormat`，我们改成`org.apache.hadoop.hive.ql.io.CombineHiveInputFormat`。这样比起上面调整 mapper 数时，又会多出两个参数，分别是`mapred.min.split.size.per.node`和`mapred.min.split.size.per.rack`，含义是单节点和单机架上的最小 split 大小。如果发现有 split 大小小于这两个值（默认都是 100MB），则会进行合并。具体逻辑可以参看 Hive 源码中的对应类。


##### 输出阶段合并

直接将 `hive.merge.mapfiles` 和 `hive.merge.mapredfiles` 都设为 true 即可，前者表示将 map-only 任务的输出合并，后者表示将 map-reduce 任务的输出合并。另外，`hive.merge.size.per.task` 可以指定每个task输出后合并文件大小的期望值，`hive.merge.size.smallfiles.avgsize` 可以指定所有输出文件大小的均值阈值，默认值都是 1GB。如果平均大小不足的话，就会另外启动一个任务来进行合并。

#### 启用压缩

压缩 job 的中间结果数据和输出数据，可以用少量 CPU 时间节省很多空间。压缩方式一般选择 Snappy，效率最高。

要启用中间压缩，需要设定`hive.exec.compress.intermediate` 为 true，同时指定压缩方式`hive.intermediate.compression.codec` 为`org.apache.hadoop.io.compress.SnappyCodec`。另外，参数`hive.intermediate.compression.type` 可以选择对块（BLOCK）还是记录（RECORD）压缩，BLOCK的压缩率比较高。
输出压缩的配置基本相同，打开`hive.exec.compress.output`即可。

#### JVM 重用

在 MR job 中，默认是每执行一个 task 就启动一个 JVM。如果 task 非常小而碎，那么 JVM 启动和关闭的耗时就会很长。可以通过调节参数`mapred.job.reuse.jvm.num.tasks`来重用。例如将这个参数设成 5，那么就代表同一个 MR job 中顺序执行的 5 个 task 可以重复使用一个 JVM，减少启动和关闭的开销。但它对不同 MR job 中的 task 无效。

### 并行执行与本地模式

#### 并行执行

Hive 中互相没有依赖关系的 job 间是可以并行执行的，最典型的就是多个子查询 union all。在集群资源相对充足的情况下，可以开启并行执行，即将参数hive.exec.parallel设为true。另外hive.exec.parallel.thread.number可以设定并行执行的线程数，默认为8，一般都够用。

#### 本地模式

Hive 也可以不将任务提交到集群进行运算，而是直接在一台节点上处理。因为消除了提交到集群的 overhead，所以比较适合数据量很小，且逻辑不复杂的任务。
设置 `hive.exec.mode.local.auto` 为 true 可以开启本地模式。但任务的输入数据总量必须小于 `hive.exec.mode.local.auto.inputbytes.max（默认值128MB）` ，且 mapper 数必须小于 `hive.exec.mode.local.auto.tasks.max（默认值4）` ，reducer 数必须为 0 或 1，才会真正用本地模式执行。

### 严格模式

所谓严格模式，就是强制不允许用户执行 3 种有风险的 HiveSQL 语句，一旦执行会直接失败。这 3 种语句是：

- 查询分区表时不限定分区列的语句；
- 两表 join 产生了笛卡尔积的语句；
- 用 order by 来排序但没有指定 limit 的语句。

要开启严格模式，需要将参数`hive.mapred.mode`设为 strict。

#### 采用合适的存储格式

在 HiveSQL 的 create table 语句中，可以使用 stored as ... 指定表的存储格式。Hive 表支持的存储格式有 TextFile、SequenceFile、RCFile、Avro、ORC、Parquet 等。

存储格式一般需要根据业务进行选择，在我们的实操中，绝大多数表都采用 TextFile 与 Parquet 两种存储格式之一。

TextFile 是最简单的存储格式，它是纯文本记录，也是 Hive 的默认格式。虽然它的磁盘开销比较大，查询效率也低，但它更多地是作为跳板来使用。RCFile、ORC、Parquet 等格式的表都不能由文件直接导入数据，必须由 TextFile 来做中转。

Parquet 和 ORC 都是 Apache 旗下的开源列式存储格式。列式存储比起传统的行式存储更适合批量 OLAP 查询，并且也支持更好的压缩和编码。我们选择 Parquet 的原因主要是它支持 Impala 查询引擎，并且我们对 update、delete 和事务性操作需求很低。