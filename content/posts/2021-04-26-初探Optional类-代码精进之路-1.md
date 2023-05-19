---
title: 初探Optional类-代码精进之路(1)
date: 2021-04-26 10:21:55
tags:
- 读后感
- java
- 语法
categories:
- 代码精进之路
---

最近开始读张建飞的《代码精进之路：从码农到工匠》，感觉对于我这样没有开始太多编程的人是一本不错的书：没有经过系统的编码培训，需要从命名规范开始学起。

计划在这个分类下边记录一下读这本书的一些感悟&知识，其实应该是感悟多一些，但是无奈java基础比较差，所以第一篇就以知识开始。

Optional出现在书中的（3.7 精简辅助代码）中，这个看到同事也用过，但是现在还是不了解这个东西可以干嘛，所以就在这里记录一下。

<!--more-->

## 为什么会有Optional？

Optional是java8开始引入的，主要是为了解决一个问题：空指针异常（NullPointerException）

先看看java8之前是这么做的：

以前我们想要完成以下逻辑的时候：

````java
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
`````

就会考虑到：中间无论哪一个get如果是null的话，都不会得到最后的结果，所以代码只能这么写：

````java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
````

这样就把代码变得很麻烦。所以java8就推出了Optional来解决这个问题。

## Optioal是什么

>A container object which may or may not contain a non-null value. If a value is present, isPresent() returns true. If no value is present, the object is considered empty and isPresent() returns false.

>Additional methods that depend on the presence or absence of a contained value are provided, such as orElse() (returns a default value if no value is present) and ifPresent() (performs an action if a value is present).

>This is a value-based class; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization) on instances of Optional may have unpredictable results and should be avoided.

上边就是Optional代码的注释，简单的说就是一个容器可能包含了空值，使用isPresent()方法可以来判断值是否存在，还提供了orElse()这样的支持方法，同时禁止使用 identity-sensitive 类型的操作

## 如何使用Optional

如果对象可能为null也可以不能为null，那么应该这么做：

    Optional<User> opt = Optional.ofNullable(user)

想要访问对象的话，可以这么做：

    opt.ifPresent( u -> assertEquals(user.getEmail(), u.getEmail()));


此外还可以使用orElse()/orElseGet() 这样的方法来处理null的情况。

针对一开始举的例子，就可以这么使用：

    String isocode = Optional.ofNullable(user)
                    .map(User::getAddress)
                    .map(Address::getCountry)
                    .map(Country::getIsocode)
                    .orElse("test")

备注一下  :: 是[method reference](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)方法引用，也是java8更新的新特性。感觉像是为了优化代码结构而做，不用::也可以实现