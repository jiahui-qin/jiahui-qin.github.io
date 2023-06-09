---
title: mysql和redis中的数据一致性问题
date: 2021-12-08 10:12:48
tags:
- mysql
- redis
categories:
- develop
---

今天早上在地铁上翻腾讯技术工程的微信公众号，发现了一篇文章，发现讲的真不错，就想围绕这一篇文章写点东西。

这篇文章的地址是[认识 MySQL 和 Redis 的数据一致性问题](https://mp.weixin.qq.com/s/GU3cbUkI84IMwttDz16P3w)

主要讲的就是在有无并发的两种情况下，如何保证redis和mysql中的数据一致。围绕这个问题讲的是相当详细。因为我当前在做的模块也涉及到了redis和mysql的数据同步，看完之后感觉还是受益匪浅呀。这种偏向工程实践的文章可以再多来点！

<!--more-->

具体的当然还是看文章更好，我主要浓缩一下就好了：

mysql和redis保证数据一致性的最佳实践应该这么作：

* 只读缓存：
    * 无并发场景：
    * 高并发场景：

        先更新数据库再删除缓存，在 “更新数据库 + 删除缓存” 的方案中，推荐使用推荐用 “先更新数据库，再删除缓存” 策略，因为先删除缓存可能会导致大量请求落到数据库，而且延迟双删的时间很难评估。在 “先更新数据库，再删除缓存” 策略中，可以使用“消息队列+重试机制” 的方案保证缓存的删除。并通过 “订阅 binlog” 进行缓存比对，加上一层保障。



* 读写缓存：
    * 无并发场景
    * 高并发场景：
        更新数据库再更新缓存，可以保证缓存中一直有数据


保证缓存和数据库一致的主要策略还是通过消息队列+重试机制来保证。也就是将要删除的消息push到消息队列里边，如果删除、更新成功的话，就消费掉这一条消息，否则就不断重试。固定次数之后向客户端报错。

此外还需要注意数据读写的顺序，也就是通过加锁来保证

此外还可以通过binlog+canal来做数据一致性对比。

文章里关于缓存穿透、缓存击穿、缓存雪崩的描述和解决方案也非常有用，总之赞赞赞

---
## 附录：

### binlog
binlog是Mysql sever层维护的一种二进制日志，其主要是用来记录对mysql数据更新或潜在发生更新的SQL语句，并以"事务"的形式保存在磁盘中；

作用主要有：

* 复制：MySQL Replication在Master端开启binlog，Master把它的二进制日志传递给slaves并回放来达到master-slave数据一致的目的

* 数据恢复：通过mysqlbinlog工具恢复数据

* 增量备份


### canal

canal，是阿里开源的组件，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费

工作原理：

* canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
* MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
* canal 解析 binary log 对象(原始为 byte 流)

通过开启了binlog的mysql + canal，就可以订阅mysql的数据变化，和我们要做的数据变化做对比，确保数据有更新，同时也可以对canal的消息进行消费，比如用来同步缓存、其它数据库的数据