---
title: 概览
---

InfluxDB是一个基于时间序列和度量值的分析型数据库。它是用go语言编写的，没有任何的依赖。这意味着你安装完InfluxDB之后，就没有额外的安装需要了(类似Redis, ZooKeeper, Cassandra, HBase等等)。

InfluxDB应用于以下场景，例如DevOps,度量值、传感数据以及需要数据实时分析的场景。它是基于以上场景专门开发的数据库，区别于我们目前已有的数据库。你可以[了解更多关于InfluxDB的使用场景从SaaS应用到开源时间序列数据库](/blog/2014/09/26/one-year-of-influxdb-and-the-road-to-1_0.html).

## 项目状态

InfluxDB目前的最新版本是0.9.4.1。集群、备份和高可用性等功能还在内测阶段。

## 主要功能

InfluxDB目前支持下面的一些主要功能，对处理基于时间序列的数据是一个绝佳的选择。

* 类似SQL的查询语法。
* HTTP(S)的接口支持数据的插入和查询。
* 内置支持其他数据协议，例如collectd。
* 存储数十亿的数据点。
* 支持对目标数据快速有效的查询。
* 数据库管理支持保留策略。
* 内置管理界面。
* 聚合查询语句如下:

```sql
SELECT mean(value) FROM cpu_user WHERE cpu=cpu6 GROUP BY time(5m)
```
* 支持以标签为标记存储和查询数十万的序列:

```sql
SELECT mean(value) FROM cpu
    WHERE region="uswest" AND az="1" AND server="server01"
    GROUP BY time(30s)
```

* 支持合并多个序列:

```sql
SELECT mean(value) FROM /cpu.*/ WHERE time > now() - 1h GROUP BY time(30m)
```

请参考[使用](getting_started.html)查看更多例子。

## 设计目的

下面是我们设计InfluxDB时想要实现的目标功能：

* 存储度量值数据 (例如响应时间和cpu负载等)。
* 存储事件数据(例如异常，用户分析和业务分析等)。
* HTTP（S）读写数据接口,支持浏览器操作无需额外的服务器。
* 安全模型，将使面向用户的分析仪表直接连接到HTTP接口。
* 水平可扩展。
* 数据保存在磁盘中。它不需要一个集群的机器将所有数据保存在内存中，因为大部分时间这些数据都用不到。
* 易于安装和管理。不像Zookeeper和Hadoop一样需要额外的依赖。
* 支持动态计算百分位和其他功能。
* 在不同的时间窗口下采样数据。
* 可以高效、自动地清除原数据，以腾出空间。数据库管理保留策略。
* 能够添加自定义的实时和批量分析处理。
* 支持向集群添加机器，能使节点的替换迅速简单。
* 在后台自动计算常用查询。
