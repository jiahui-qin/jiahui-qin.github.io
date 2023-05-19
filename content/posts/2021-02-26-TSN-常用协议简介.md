---
title: TSN 常用协议简介
date: 2021-02-26 15:59:24
tags:
- 协议
categories:
- 网络
---

在做这个项目的时候，总是有很多协议不太清楚是什么意思，这里先记录一下四个比较重要的协议，大概讲一下每个协议是做什么的，有什么意义

* P802.1Qci – Per-Stream Filtering and Policing

    * 基于stream的流量监管


* 802.1Qbu - Frame Preemption

    * 抢占式mac？

* 802.1Qbv - Enhancements for Scheduled Traffic

    * 时间感知整形器

* P802.1CB – Frame Replication and Elimination for Reliability

    * 帧的复制和高可靠性消除

* 802.1AS Timing and Synchronization

    * 时间同步

<!--more-->

## QCI 基于stream的流量监管

类似与防火墙的机制，对转发前的数据进行筛选和过滤，对特定标识的数据帧加以控制 -- 检测和缓解破坏性传输，提高系统的健壮性

Q： 依据什么来筛选和过滤呢？stream是什么概念？

## QBV 时间感知整形

QBV是TSN的一个核心协议，时间感知队列通过时间感知整形器(Time Aware Shaper，TAS)使TSN交换机能够来控制队列流量（queued traffic），以太网帧被标识并指派给基于优先级的VLAN Tag，每个队列在一个时间表中定义，然后这些数据队列报文在预定时间窗口在出口执行传输。其它队列将被锁定在规定时间窗口里。因此消除了周期性数据被非周期性数据所影响的结果。这意味着每个交换机的延迟是确定的，可知的。而在TSN网络的数据报文延时被得到保障。

[参考](https://www.sdnlab.com/22868.html)

## QBU 抢占式mac

这个是给端口配置的，主要的输入就两个： `发送帧抢占时pmac的最小帧长`, ``


## AS 时间同步

在802.1AS中，时间同步是按照“域”（domain）划分的，包含多个PTP节点。在这些PTP节点中，有且仅有一个全局主节点（GrandMaster PTP Instance），其负责提供时钟信息给所有其他从节点。

PTP节点又分为两类：PTP End Instance（PTP端节点）和PTP Relay Instance（PTP交换节点）其中：

PTP End Instance或者作为GrandMaster，或者接收来自GrandMaster的时间同步信息

PTP Relay Instance从某一接口接收时间同步信息，修正时间同步信息后，转发到其他接口

**如何选择GrandMaster？**

现在看是只按照优先级来选，而不看其他的指标，另外可以通过端口上的port-state结合实际物理拓扑来构造出一副图来指示整个网络的时间拓扑