---
title: go并发基础知识
date: 2021-02-20 11:21:49
tags:
- go
- 并发
categories:
- go初学者教程
---

既然标题是并发，先看看什么是并发：

**多线程程序在单核心的CPU上运行是并发**， 多线程程序在多核心CPU上运行是并行

并发是如何实现的呢？ 依靠的是CPU的时间分片来完成的。

什么是进程、线程、协程呢？

进程是程序的一次执行过程


<!--more-->

线程是进程的一个执行实体，是CPU调度和分派的基本单位。一个进程可以创建和撤销多个线程。

协程： 独立的栈空间，共享堆空间，调度由用户自己控制。类似用户级线程。协程是轻量级的线程

----

go并发设计的核心是 **goroutine**

使用go关键词可以创建goroutine，将go声明放在一个函数前，运行这个函数，就会作为一个并发的独立线程。

以下是三种用法：一个方法、匿名方法、直接写一个代码块

````go
//go 关键字放在方法调用前新建一个 goroutine 并执行方法体
go GetThingDone(param1, param2);

//新建一个匿名方法并执行
go func(param1, param2) {
}(val1, val2)

//直接新建一个 goroutine 并在 goroutine 中执行代码块
go {
    //do someting...
}
````

进程间的通信可以使用channel，使用make离开创建channel

````go
ci := make(chan int)
cs := make(chan string)
cf := make(chan interface{})
````


有一点没有搞明白：

java可以先创建多个线程，再一起执行，在go里边怎么创建多个线程再一起执行呢？

要通过channel拿结果的话就要一直等待这个线程执行完毕，那要如何实现呢？