---
title: sonor常见问题修改
date: 2021-08-11 11:52:20
tags:
- java
- bug修复
- 代码异味
categories:
- sonor
---

好像是很久没有写blog了，今天在用sonor检查了一遍现在写的模块，开始对异味做一下修复，由于很多异味都是重复的，所以这里记录一下异味的类型&修复方法

<!--more-->
### Provide the parametrized type for this generic

通用类声明时应该指定参数类型，否则可能在运行时捕获异常。 也就是说用 `List<>, R<>, Set<>` 这种的时候最好还是要指定一下里边参数的类型。

### Use the primitive boolean expression here

如果写了这样的代码： 
```
if (data.isExist()){
    doSomething
}
```
有可能会抛出一个NullPointerException错误。所以应该使用布尔值来和这个值进行比较: `Boolean.TRUE.equals(data.isExist())`

### Use isEmpty() to check whether the collection is empty or not

测试一个List是否为null的时候使用`isEmpty()`要优于`object.size()==0`，前者的时间复杂度时O(1),后者的时间复杂度时O(n)

### Boolean literals should not be redundant

这是纯粹的写法问题： `Boolean isEqual = 3.equals(2)? true:false;` 比较冗余，不如直接写 `Boolean isEqual = 3.equals(2)`

### Rename this method to prevent any misunderstanding or make it a constructor

一个类里的方法应该只有构造器的方法name可以和类name一样。

