---
title: WSL2使用问题记录
date: 2021-05-26 15:07:19
tags:
- wsl2
- 问题记录
categories:
- WSL
---

今天在wsl2中起了一个nodejs的服务，服务起了之后显示如下：

>2021-05-26 15:08:50,767 INFO 6103 [master] egg started on http://127.0.0.1:7002 (1032ms)

但是在windows里边用工具请求localhost却完全请求不到api，返回都报的是500

后来想了一下，大概是wsl2里边的网络和windows的网络不一样，所以宿主机的localhost并不是wsl2所显示的localhost···

引用别人[blog](https://www.jianshu.com/p/ba2cf239ebe0)里边的话讲，大概是这样的：

>  wsl2 则可以理解为宿主机完整虚拟出来的一个完整的 Linux 虚拟机，拥有自己的逻辑上独立的网卡，也即拥有属于自己的独立网络栈。与 VMware 的 bridge 模式和 docker 的 macvlan 模式类似。

于是问题就比较好解决了：

在wsl2里边用ifconfig查看一下 eth0 的ip地址，用eth0的ip地址来替换localhost，就可以成功查看了。

