---
title: go中...语法糖的应用
date: 2021-02-19 18:04:57
tags:
- go
- 语法糖
categories:
- go初学者教程
---

今天第一次用go刷leetcode，看到有这么一句：

    ans = append(ans, append([]int(nil), comb...))

对go的了解属实不深，不了解comb后边这...是干啥的，于是就查了下，以下是详细内容

<!--more-->

首先是函数定义的时候，可以做如下定义：

    func test1(args ...string){}

这里就可以传入多个string类型的变量，args也是一个可迭代的对象。这个用法有点像python里的变量前边加一个*就可以接受多个变量

第二中用法是

    var strss= []string{"qwr","234","yui"}
    var strss2= []string{"qqq","aaa","zzz","zzz",}
    strss=append(strss,strss2...) //strss2的元素被打散一个个append进strss
    fmt.Println(strss)

    output:
    [qwr 234 yui qqq aaa zzz zzz]

    这里... 相当于把strss2做了切片，挨个传进了strss

    毕竟append的第二个入参是...string 不是[]string，所以用这个还是相当方便的。
