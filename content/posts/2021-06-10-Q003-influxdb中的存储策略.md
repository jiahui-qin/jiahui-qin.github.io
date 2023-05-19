---
title: 'Q003:influxdb中的存储策略&存储相关'
date: 2021-06-10 18:49:03
tags:
- TICK
- influxdb
categories:
- 日常问题解决
---

为什么会发出这个问题呢？因为我在chronograf中查询influxdb数据的时候，发现要查询语句要这么写：

    select * from monitor.rp30.measurement1 order by time DESC limit 10
    select * from monitor.autogen.measurement1 order by time DESC limit 10

可以看到两个表分别是：*monitor.rp30.measurement1*、*monitor.autogen.measurement1*，可以很明确的知道monitor是database， measurement1是measurement，那么中间的rp30、autogen是什么意思呢？

这里可以快速做一个解答： 是influxdb中的存储策略

<!--more-->

## 存储策略
influxdb的存储策略定义数据在influxdb中的保留时间，可以使用以下命令来查询当前数据库的存储策略
    
    > show retention policies on monitor
    name    duration  shardGroupDuration replicaN default
    ----    --------  ------------------ -------- -------
    autogen 0s        168h0m0s           1        false
    rp30    2160h0m0s 24h0m0s            1        true

可以看到monitor的存储策略有两个：autogen和rp30， 每个字段意义如下：

+ duration 数据保留时间
+ shardGroupDuration shardGroup的保留时间，这是influxdb的一个基本结构，大于这个时间的数据查询效率比较低
+ replicaN 副本个数
+ default 是否是默认策略

新建一个存储策略也可以用以下语句：
    
    CREATE RETENTION POLICY "2_hours" ON "telegraf" DURATION 2h REPLICATION 1 DEFAULT

influxdb的存储策略的设置位置在：


## influxdb的存储引擎

TSM Tree

这个本来想要自己写，搜了一下发现有位大哥讲的很透彻，放链接吧

[时间序列数据库调研之InfluxDB](http://blog.fatedier.com/2016/07/05/research-of-time-series-database-influxdb/)

[InfluxDB详解之TSM存储引擎解析(一)](http://blog.fatedier.com/2016/08/05/detailed-in-influxdb-tsm-storage-engine-one/)


[InfluxDB详解之TSM存储引擎解析(二)](http://blog.fatedier.com/2016/08/15/detailed-in-influxdb-tsm-storage-engine-two/)

大概总结一下，TSM是LSM的升级版