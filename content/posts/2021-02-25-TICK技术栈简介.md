---
title: TICK技术栈简介
date: 2021-02-25 11:28:05
tags:
- TICK
- 监控系统
categories:
- 指南
---

最近在看监控系统相关的文章，TICK的应用比较广泛，这是InfluxData开发的一套工具栈，主要包括以下四个组件： Telegraf  Influxdb Chronograf grafana Kapacitor

具体的可以看[图](http://zhoujinl.github.io/2018/02/27/tick/)

## telegraf

这是一个go开发的监控工具，可以用来收集和报告指标，通过输出、转换、输出插件等将数据从目标采集并发送给其他的数据存储、服务或者消息队列

<!--more-->

## influxDB

这是一个go开发的时序数据库，为带有时间戳的数据编写。

## chronograf

是一个开源可视化引擎 -- 可能会被替代？

## grafana

go和angular框架写的可视化工具，比较流行的时序数据可视化工具

## Kapacitor

是一个数据引擎，可以处理来自influxdb的流数据，并且可以监控和报警

核心还是telegraf & influxdb

influxdb更新了2.x版本，变化较大 [官方文档](https://docs.influxdata.com/influxdb/v2.0/get-started/)

[更新内容](https://www.infoq.cn/article/662MdX6QNzcL-5D4axKb)