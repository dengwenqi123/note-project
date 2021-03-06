## Druid简介

Druid是一个快速的**列式**分布式的**支持实时分析**的**数据存储系统**。它在处理PB级数据、毫秒级查询、数据实时处理方面，比传统的OLAP(On-Line Analytical Processing,联机分析处理)系统有了显著的性能改进。(时序数据库)

Druid是一种高性能、列式存储、分布式数据存储的时序数据分析引擎。能支持实时数据摄入，“PB”级数据的秒级查询。类似的产品有kylin/clickhouse。Druid典型的应用是OLAP场景下的cube组合查询分析，如数据钻取(Drill-down)、上卷(Roll-up)、切片(Dice)、以及旋转(Pivot).

#### Druid的特点

1. 列式存储格式，Druid 使用面向列的存储，它只需要加载特定查询所需的列。查询速度迅速快。
2. 可扩展的分布式系统。Druid通常部署在数十到数百台服务器的集群中，并且提供数百万条/秒的摄取率，保留数百万条记录，以及亚秒级到几秒钟的查询延迟。
3. 大规模的并行处理。Druid可以在整个集群中进行大规模的并行查询。
4. 实时或批量摄取。Druid可以实时摄取数据(实时获取的数据可立即用于查询)或批量处理数据。
5. 自愈、自平衡、易操作。集群扩展和缩小，只需添加或删除服务器，集群将在后台自动重新平衡。无需任何停机时间。
6. 数据进行了有效的预聚合或预计算，查询速度快。
7. 数据的结果应用了Bitmap压缩算法。

#### Druid应用场景

1. 适用于清洗好的记录实时录入，但不需要更新操作
2. 适用于支持宽表，不用join的方式(换句话说就是一张单表)
3. 适用于可以总结出基础的统计指标，用一个字段表示
4. 适用于实时性要求高的场景
5. 适用于对数据质量的敏感度不高的场景。

**Druid**是一个实时处理时序数据的OLAP数据库，因为它的索引首先按照时间分片，查询的时候也是按照时间线去路由索引。

```
select md,max('loading_time') from "topic_start" group by md;
```

Druid的DataSource相当于关系型数据库中的表(Table),DataSource的结构包括：

1. 时间列，表明每行数据的时间值，默认使用UTC时间格式且精确到毫秒级别。
2. 纬度列：纬度来自OLAP的概念，用来标识数据行的各个类别信息。
3. 指标列：是用来聚合和计算的列，通常是一些数字，计算操作通常包括Count,Sum等

无论是实时数据消费还是批量数据处理，Druid在基于DataSource结构存储数据时即可选择对任意的指标列进行聚合操作，该聚合操作主要基于维度列与时间范围两方面的情况，下图显示的是执行聚合操作后DataSource的数据情况。





































