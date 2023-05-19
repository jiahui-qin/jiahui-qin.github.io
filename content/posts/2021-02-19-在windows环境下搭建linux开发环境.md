---
title: 在windows环境下搭建linux+gvm开发环境
date: 2021-02-19 16:38:50
tags:
- Windows
- wsl2
- Ubuntu
categories:
- 指南
---

为什么要这么做呢？

主要是做go开发的时候发现要修改telegram之类的插件都是在linux系统下开发编译比较方便，然而我手头又没有linux机器，只好考虑在本机上搭建这么一套环境了。

其实这是一篇早就写好的文章，之前发表在了我的CSDN上，还是搬运了过来，以下是正文

<!--more-->
----

## 安装ubuntu16.04

这个没啥好说的，参考微软的文档绝对没问题

https://docs.microsoft.com/zh-cn/windows/wsl/install-win10

最简单的方法还是直接加入开发者计划，升级一下系统，一行命令就搞定

## ubuntu设置
Ubuntu安装好之后还是要做一些设置的：

1. 设置登陆账号密码 
2. 权限提升，其实可以直接用root来登录，要设置root的密码

		sudo passwd root
	
3. 升级apt-get，需要换源的可以参考这个[帖子](https://blog.csdn.net/weixin_39394526/article/details/87935449)
	
		apt-get update
	

## 安装GVM 

[在ubuntu上安装GVM](https://www.cnblogs.com/or2-/p/4160814.html)

	//安装gvm的必要软件
	sudo apt-get install curl git mercurial make binutils bison gcc build-essential　
	//使用bash安装GVM
	bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

如果这个时候报了403错误，则按照下列操作

	sudo vim /etc/hosts
	//在文件中加下边一句
	199.232.28.133 raw.githubusercontent.com

如果报了这个错误：

	curl: (35) gnutls_handshake() failed: Error in the pull function.
就按照这个[帖子](https://blog.csdn.net/anlian523/article/details/90729063)操作一下
## 安装GO
通过GVM来安装go，如果要安装1.13等高版本，需要首先安装1.4版本的go

那么就先安装1.4，再安装1.13：

	gvm install go1.4
	gvm use go1.4
	gvm install go1.13
安装1.4的是时候如果有报错：

	Downloading Go source...
	Installing go1.4...
	 * Compiling...
	/root/.gvm/scripts/install: line 84: go: command not found
	ERROR: Failed to compile. Check the logs at /root/.gvm/logs/go-go1.4-compile.log
	ERROR: Failed to use installed version
那么就设置环境变量不使能cgo，主要参考的是这篇[文章](https://blog.csdn.net/zoumy3/article/details/78440880?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-3.not_use_machine_learn_pai&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-3.not_use_machine_learn_pai)，一般就可以解决，接下来继续安装go1.13：

	export  CGO_ENABLED=0
	
如果报这个错：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210108161709483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzUzNzI4OQ==,size_16,color_FFFFFF,t_70)
就换成如下安装命令：
	
	gvm install go1.4 -B

## GO设置
go代理主要参考的是这一篇文章：

[goproxy 代理设置](https://www.sunzhongwei.com/problem-of-domestic-go-get-unable-to-download?from=sidebar_new)
	
	go env -w GO111MODULE=on
	go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct

## 其他参考：

[gvm更新以及安装go](https://travis-ci.org/github/monochromegane/the_platinum_searcher/jobs/30478610)


