---
title: Q001:influxdb学习-基础&重复值解决
date: 2021-06-01 18:51:17
tags:
- TICK
- influxdb
categories:
- 日常问题解决
---

为什么要写这篇文章呢？

因为这两天想要走一个查询接口按照时间来查询记录去插入到influxdb里边，但是考虑到程序运行时之类的问题，严格按照运行间隔来取时间查询肯定会存在漏掉记录的可能性，所以最好还是多查一段时间，然后再去掉重复值

思路呢，就按照这么来：

1. 先复习一下influxdb的数据格式
2. 看一下如何解决这个问题
3. 其他

<!--more-->

## influxdb的数据格式

这个很多[blog](https://www.cnblogs.com/shhnwangjian/p/6897216.html)里边都有，这里我大致重复一下

influxdb里边可以将一条数据看为一个虚拟的key和其对应的value

    cpu_usage,host=server01,region=us-west value=0.64 1434055562000000000

其中key包括以下部分

* database 数据库名，在influxdb中可以创建多个数据库，不同数据库的文件可以隔离存放

* retention policy: 存储策略，用于设置数据保留的时间，每个数据库刚开始会自动创建一个默认的存储策略 autogen，数据保留时间为永久，之后用户可以自己设置，例如保留最近2小时的数据。插入和查询数据时如果不指定存储策略，则使用默认存储策略，且默认存储策略可以修改。InfluxDB 会定期清除过期的数据。

* measurement: 测量指标名，例如 cpu_usage 表示 cpu 的使用率。

* tag sets: tags 在 InfluxDB 中会按照字典序排序，不管是 tagk 还是 tagv，只要不一致就分别属于两个 key，例如 host=server01,region=us-west 和 host=server02,region=us-west 就是两个不同的 tag set。

* tag--标签，在InfluxDB中，tag是一个非常重要的部分，表名+tag一起作为数据库的索引，是“key-value”的形式。

* field name: 例如上面数据中的 value 就是 fieldName，InfluxDB 中支持一条数据中插入多个 fieldName，这其实是一个语法上的优化，在实际的底层存储中，是当作多条数据来存储。

* timestamp: 每一条数据都需要指定一个时间戳，在 TSM 存储引擎中会特殊对待，以为了优化后续的查询操作。 

tag + field + field 合起来就是point，可以简单理解为：

1. tags是有索引的属性
2. time是主索引，存储进去的时间
3. fields是数据，没有索引的属性

## 如何应对重复值？

一般插入的时候不会写time，但是time+用户自定义的tag 才是完整的tag，而tag又是用来做索引的，这里我们就可以考虑一下：完全一样的tag能不能插入呢？

做以下测试：

先进入influxdb的命令行界面，插入以下三条一样的数据：

    INSERT dupltest,host=serverA,region=us_west value=0.64 1623146232869000000
    INSERT dupltest,host=serverA,region=us_west value=0.64 1623146232869000000
    INSERT dupltest,host=serverA,region=us_west value=0.64 1623146232869000000
                                    
然后再查询一下：

    name: dupltest
    time                host    region  value
    ----                ----    ------  -----
    1623146232869000000 serverA us_west 0.64

果然不能重复插入。到这里问题就解决了：只要tags完全一样，就不会重复插入。

那考虑其他的情况：如果time是自动生成的话要怎么办呢？这样就不能通过依赖重复tag来剔除掉重复数据了。

可以考虑一下自己插入一个时间数据，然后查询的时候查询最后一个时间，如果时间小于这个最后时间，就不插入。