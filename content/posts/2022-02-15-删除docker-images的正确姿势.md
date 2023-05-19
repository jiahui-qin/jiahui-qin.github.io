---
title: 删除docker images的正确姿势
date: 2022-02-15 12:10:35
tags:
- docker
categories:
- develop
---

今天看了一下机器里的docker image，发现居然有20来个，好多现在都不用了，各个image又挺大的，就想着干脆全删了得了，常用的image再下载一遍就好啦，于是就随手搜了一下怎么删掉所有的image， 所有的网站都给了同一条指令：

```
docker rmi $(docker images -q)
```

很好理解吧，`docker images -q`就是列出来所有image的id，然后再挨个删掉。然后我就报了一个错：

<!--more-->
```
Error response from daemon: conflict: unable to delete 3a5e93284781 (must be forced) - image is referenced in multiple repositories
Error response from daemon: conflict: unable to delete 3a5e93284781 (must be forced) - image is referenced in multiple repositories
```

好吧，这是因为一个镜像id指向了多个镜像，删除的时候只能用repository:tag的形式来删除，然而又没有人写怎么一次性全删除==那就只好我来写了：

```
docker rmi $(docker images --format "{{.Repository}}:{{.Tag}}")
```

删除images的时候用Repository:Tag的方式来删除就搞定了