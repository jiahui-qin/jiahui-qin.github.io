---
title: SOLID设计原则-代码精进之路(2)
date: 2021-04-27 15:47:52
tags:
- 读后感
- java
categories:
- 代码精进之路
---

SOLID可以说是老生常谈了，秋招准备面试的时候就在看这个东西，无奈当时根本没有写代码，对于这些东西也完全不了解，最近写了一些代码，大概讲讲对这个的理解吧。

## SOLID概览

其中开闭原则、里氏替换原则是设计目标，单一职责、接口隔离、依赖倒置是设计方法

这里讲的是设计原则，不过SOLID并不是所有的设计原则，除了这几条之外还有DRY, YANGI, Rule of three等设计原则

## Single Responsibility Principle 单一职责原则

这个比较简单：一个软件模块只负责一个职责

## Open Close Principle 开闭原则

软件实体对扩展开放，对修改关闭

比如前段时间在用的装饰模式有一点开闭原则的感觉

## Liskov Substitution Principle 里氏替换原则

程序中所有的父类都可以被子类替换

LSP认为“程序中的对象应该是可以在不改变程序正确性的前提下被它的子类所替换的”，即子类应该可以替换任何基类能够出现的地方，并且经过替换后，代码还能正常工作。

## Interface Segregation Principle 接口隔离原则

多个特定用途的客户端接口好过一个宽泛用途的接口 --> 这个思想很适合用来指导做接口拆分

## Dependency Inverse Principle 依赖倒置原则

模块之间交互应该依赖抽象，而非实现。DIP要求高层模块不应该依赖于低层模块，二者都应该依赖于抽象。抽象不应该依赖细节，细节应该依赖抽象。

## Don'r repeat Yourself

同样的代码不要写太多次

## You Ain’t Gonna Need It

不一定需要提前做优化

## Rule of Three

针对上述两个设计原则，如果同样的代码出现了3次或者以上，就要考虑抽象出来了。