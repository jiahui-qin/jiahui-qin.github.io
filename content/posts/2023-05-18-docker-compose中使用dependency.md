---
title: docker-compose中使用dependency
date: 2023-05-18 08:07:38
tags:
- docker
- docker-compose
categories:
- develop
---

一般来说，我们部署的时候无所谓启动顺序啦，只要加上这样的标签

```
services:
    mycontainer:
        restart:always
```

就可以让docker启动失败的时候不断重启，直到他的依赖都正常起来。

但是当你遇到一台老破机器的时候可能就不那么靠谱了··毕竟周围还有其他人在等你的服务起来，起不来就尴尬了

那么就只能给docker加上启动依赖了。

假设我们有2个docker，分别是redis,javaApp，javaApp需要依赖redis，那么就要这么写docker-compose文件了：
```
services:
    redis:
        image:redis
        ports:
            - '3379:3379'
        restart: always
    javaApp:
        image:java
        depends_on:
            redis
```
如果真这么写了，就会发现有的时候javaApp还是比redis先起来，尤其是redis的aof文件比较大的时候··

查了查文档，才发现原因：如果只写`depends_on`的话，默认只要依赖的容器在运行状态就可以了，所以需要指定redis的状态是正在运行的，也就是`service_healthy`，那么我们如何知道这个容器是健康的呢？

那就引入了另外一个关键字：`healthcheck`,这个时候我们就可以重写上边的docker-compose文件如下：

```
services:
    redis:
        image:redis
        ports:
            - '3379:3379'
        restart: always
        healthcheck:
            test: ["CMD", "redis-cli", "ping"]
            interval: 1m30s
            timeout: 10s
            retries: 3
  start_period: 40s
    javaApp:
        image:java
        depends_on:
            redis:
                condition: service_healthy
```



最后总结一下具体内容：

depends_on 默认状态为`service_started`,保证以下两点：
1. 创建的时候按照顺序
2. 销毁的时候按照顺序

手动指定depends_on为`service_healthy`,就会引入健康检查，状态为健康的时候才会启动

参考docker-compose的官方文档：[healthcheck](https://docs.docker.com/compose/compose-file/05-services/#healthcheck) [depends_on](https://docs.docker.com/compose/compose-file/05-services/#depends_on)